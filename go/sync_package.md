# sync package  
  
这个包是用来实现基本的基于内存共享来实现协程间通讯的功能。  
  
## WaitGroup  
  
- sync.WaitGroup.Add(n)  
> 表示此时有n个goroutines开始了  
> 在主进程中调用，在goroutine中调用会引发竞争条件  
> 最好和goroutine set as close as possible  
  
- sync.WaitGroup.Done  
> 在goroutine中传入waitGroup并调用done方法表示goroutine结束  
> 一般加上defer关键词确保goroutine结束一定调用  
  
- sync.WaitGroup.Wait  
> 主进程中调用，在所有协程退出之前会阻塞  
  
## Mutex  
  
- sync.Mutex.Lock()  
  
- defer sync.Mutex.Unlock()  
  
- sync.RWMutex.RLocker().Lock() 读锁，如果当前没有其他协程在写资源，则允许多个读锁开放  
  
## cond
  
cond是为了实现协程间的通信功能，协程间发送一个不带任何数据的信号  

```go 
c := sync.NewCond(&sync.Mutex{})                    //NewCond接受传入一个Locker接口类型
queue := make([]interface{}, 0, 10)
removeFromQueue := func(delay time.Duration) {
    time.Sleep(delay)
    c.L.Lock()
    queue = queue[1:]
    fmt.Println("Removed from queue") 
    c.L.Unlock()
    c.Signal()                                      //c.Signal发送信号
}
for i := 0; i < 10; i++{ 
    c.L.Lock()
    for len(queue) == 2 {
        c.Wait()                                    //c.Wait会将当前协程标为block状态并挂起当前协程，其他协程此时可调度上台
                                                    //开始c.Wait之前会自动c.L.UnLock打开申请的锁  
                                                    //当出现c.Signal时会唤醒此协程  
                                                    //c.Wait结束之前自动调用c.L.Lock申请关锁
    }
    fmt.Println("Adding to queue")
    queue = append(queue, struct{}{})
    go removeFromQueue(1*time.Second)
    c.L.Unlock()
}
```
- go runtime会维护一个FIFO的队列记录当前wait状态的协程  
  
- c.Signal()会唤醒等待时间最久的协程  
  
- c.Signal()功能也可以通过channel来实现，但是channel要略低性能一点  
  
- c.BroadCast()功能会唤醒所有等待的协程  
  
```go
type Button struct { Clicked *sync.Cond}
button := Button{ Clicked: sync.NewCond(&sync.Mutex{}) }
subscribe := func(c *sync.Cond, fn func()) {
  var goroutineRunning sync.WaitGroup
  goroutineRunning.Add(1)
  go func() {
    goroutineRunning.Done()
    c.L.Lock()
    defer c.L.Unlock()
    c.Wait()
    fn() }()
  goroutineRunning.Wait() 
}
var clickRegistered sync.WaitGroup
clickRegistered.Add(3)
subscribe(button.Clicked, func() {
  fmt.Println("Maximizing window.")
  clickRegistered.Done()
})
subscribe(button.Clicked, func() {
  fmt.Println("Displaying annoying dialog box!")
  clickRegistered.Done()
})
subscribe(button.Clicked, func() {
  fmt.Println("Mouse clicked.")
  clickRegistered.Done()
})
button.Clicked.Broadcast()
clickRegistered.Wait()
```
上述代码展示了broadcast的使用  
  
一个GUI程序的鼠标点击时间会触发三个wait状态的handler  
  
模拟了鼠标点击，窗口放大，展示对话框的事件  
  
## Once  
  
sync.Once保证了Once.Do()只被执行一次，即使在多个goroutine里面调用了Once.Do方法  
  
```go
var onceA, onceB sync.Once
var initB func()
initA := func() { onceB.Do(initB) } //1
initB = func() { onceA.Do(initA) }  //2
onceA.Do(initA)                     //3
```  
调用了3之后会执行initA，initA会执行initB  
  
initB在2处想要执行initA，但是onceA.Do正在执行，无法执行2，所以无法完成3，造成死锁  
  
## Pool  
  
```go
var numCalcsCreated int calcPool := &sync.Pool {
  New: func() interface{} {                       //创建一个Pool要说明它的new方法
    numCalcsCreated += 1                          //new方法用来申请资源，只有当当前pool中资源不够时才新申请
    mem := make([]byte, 1024)
    return &mem
  },
}
// Seed the pool with 4KB
calcPool.Put(calcPool.New())                      //初始化一个pool，pool中有4份资源
calcPool.Put(calcPool.New())
calcPool.Put(calcPool.New())
calcPool.Put(calcPool.New())

const numWorkers = 1024*1024
var wg sync.WaitGroup 
wg.Add(numWorkers)                                //做numWorkers次申请资源的动作
for i := numWorkers; i > 0; i-- {
  go func() {
    defer wg.Done()                               //申请完资源再Done
    mem := calcPool.Get().(*[]byte)
    defer calcPool.Put(mem)                       //申请完资源做了一些事情后调用Put将当前资源返回池中
  }() 
}
wg.Wait()
fmt.Printf("%d calculators were created.", numCalcsCreated)
```
上述代码统计得利用Pool申请1024*1024次资源，实际只新申请了8次  
  
当申请的资源为同质类型，且有大量并发的申请，申请了之后会马上释放时，使用Pool比较合算  
  
但是如果申请的资源不同质，可能花在类型转换上的时间更多  
  
Pool也适合用做cache  
  

  


