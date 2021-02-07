# CSP  
  
在go的并发逻辑中，提倡用沟通的方式去共享内存而不是用共享内存的方式去沟通  
  
虽然提供了一般的通过内存控制进行并发的原语（lock，unlock，wait等）但是还是推荐用CSP类型原语（chan，select等）  
  
## channels  
```go 
var dataStream chan interface{}     //声明channel dataStream可以传输和接受任何类型
dataStream = make(chan interface{}) //初始化channel  

var receiveChan <-chan interface{}  //只读类型的channel，只支持读出数据
var sendChan chan<- interface{}     //只写类型的channle，只支持写入数据
```  
上述代码中的单向channel声明其实不常见，这种channel更多的作为函数传入参数和返回结果存在  
  
单向和双向channel的转换就像const xx类型和xx类型的转换一样  
  
```go
intStream := make(chan int) 
close(intStream)
integer, ok := <- intStream 
fmt.Printf("(%v): %v", ok, integer)
```
在读取channel中的值时可以一次性读两个，其中第二个值是一个bool变量  
  
代表当前的值是否是从一个open的channel里面取出的  
  
当上游决定已经没有其他消息要传递的时候，可以关闭这个channel  
  
注意在上述的代码中，有一个tricky的地方：该channel没有生产任何数据，但是关闭的时候也可以读  
  
```go
begin := make(chan interface{}) 
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
  wg.Add(1)
  go func(i int) { 
    defer wg.Done()
    <-begin
    fmt.Printf("%v has begun\n", i) 
  }(i)
}
fmt.Println("Unblocking goroutines...") 
close(begin)
wg.Wait()
```
上述的代码显示了channel类似sync.BroadCast的用法  
  
关闭一个channel会向所有等待这个channel中消息的协程发送信号  
  
因为channel更易组合，所以利用channel来恢复数个被阻塞的进程是本书作者更喜欢的方式  
  
### buffered channels VS unbuffered channels  
  
突然发现我对这两个概念理解的不是很清楚，所以总结一下  
  
```go
var unbuffered chan interface{}
unbuffered = make(chan interface{}, 0)    //这是一个unbuffered channel

var buffered chan interface{}
buffered = make(chan interface{},4)       //这是一个unbuffered channel

go func(c chan interface{}){
  c<- "Hello"                             //向一个unbuffered channel里面写入数据，在receiver没发生之前会被阻塞
  DoSthNext()                             //这里DoSthNext不会立即执行，直到str := <-unbuffered执行
}(unbuffered)

str := <-unbuffered                       //这句执行时触发了匿名函数中的sender
```  
但是如果上述代码使用的是buffered channel，DoSthNext就会先于str:=<-buffered执行  
  
所以根据上述原因，下面这段代码会造成死锁(验证了确实会死锁)  
```go
var unbuffered chan interface{}
unbuffered = make(chan interface{}, 0)    //这是一个unbuffered channel
unbuffered <- "Hello"
str := <- unbuffered
```
这个时候我们再回头看一下原来的代码  
```go
intStream := make(chan int) 
close(intStream)
integer, ok := <- intStream 
fmt.Printf("(%v): %v", ok, integer)
```
与造成死锁的不同的是close操作并不会阻塞，而关闭一个channel也不是代表该channel的内存就被回收了  
  
仍然可以从中读到关闭的信号  
  
对于有缓存的channel，一般是在知道会向这个channel中传入多少数据时使用  
  
比如，我们需要传输n个数据，那么可以设置一个capacity为n-1的channel  
  
这样n个数据相当于不会阻塞的一起传输到了reader端  
  
最后一个数据的传输实际上是会阻塞后面的内容  
  
但是从reader的角度看，它实行了一次read之后，最后一个数据马上会补入buffer，
  
所以它相当于一次性有n个数据可读
  
### channel的使用框架  
  
channel的使用有下述可能的危险：  
  
- 未初始化channel导致读写或者close nil于是panic  
- 重复close一个channel导致panic  
- 向已经close的channel中写入数据  
- 没有读取两个变量导致不知道channel什么时候关闭，一直重复读取已经关闭的channel  
  
本书作者建议使用下述框架来规范对channel的使用：  
  
- 对于channel的owner(初始化channel，写数据的func)，下述三个阶段写在一个context里
> - 初始化channel  
> - 向channel写数据或者传递channel的ownership  
> - close channel  
  
- 对于读channel的func  
> - 用range读channel或者一次读两个以便知道channel什么时候关闭的  
> - 如果被阻塞时适当的处理（不处理代表该程序可以被阻塞无限长的时间也没关系）  
  
## select  
  
select语句也是CSP并发概念中一个很重要的原语  
  
如果说channel是把goroutine联系在一起的胶水，那么select就是把channel联系在一起的胶水  
  
```go
var c1, c2 <-chan interface{}
var c3 chan<- interface{}
select {
  case <- c1:
        // Do something
  case <- c2:
        // Do something
  case c3<- struct{}{}:
        // Do something
}
```
上述代码跟switch-case语句比较类似，但是它们有几个较大的不同：  
- 每一个case的判断几乎是同时进行的，而不是顺序进行  
- 当一个case准备好了之后（可read，可write，读closed channel），会执行相应代码
- 所有case都没准备好的话会阻塞  
  
那么这里就有三个问题：  
- 如果有case同时成立会如何？  
- 如果不想永远阻塞怎么办？  
- 如果所有case都没准备好，希望程序执行另一种逻辑怎么办？  
  
- 如果有case同时成立，go runtime随机执行其中一个case  
```go
c1 := make(chan interface{});
close(c1)
c2 := make(chan interface{});
close(c2)
var c1Count, c2Count int for i := 1000; i >= 0; i-- {
  select { 
    case <-c1:
      c1Count++              //c1Count: 505
    case <-c2:
      c2Count++              //c2Count: 496
  }
}
fmt.Printf("c1Count: %d\nc2Count: %d\n", c1Count, c2Count)
```
- 如果不想永远被阻塞就设置阻塞到一定时间时，执行...  
```go
var c <-chan int
select {
  case <-c:
  case <-time.After(1 * time.Second):
    fmt.Println("Timed out.")
}
```
- 对于所有case都没准备好，可以用default来执行相应逻辑：
```go
done := make(chan interface{})
go func() {
  time.Sleep(5*time.Second)
  close(done)
}()
workCounter := 0 
loop:
for {
  select { 
    case <-done:
      break loop 
    default:
  }
  // Simulate work
  workCounter++
  time.Sleep(1*time.Second) 
}
fmt.Printf("Achieved %v cycles of work before signalled to stop.\n", workCounter)
```
`select{}`会一直阻塞
