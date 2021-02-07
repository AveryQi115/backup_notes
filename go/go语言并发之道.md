## go的并发和其他语言中并发的区别  
  
其他语言的并发：需要利用操作系统提供的线程，在操作系统层上再做一层抽象  
  
go：利用go协程，不在OS层上面抽象，而是替代OS的线程  
  
## go的并发哲学  
  
- CSP的理念：输入和输出对于一个程序来说是重要的  
  
- 不要用共享内存的方式来沟通（线程间通信），用沟通的方式来共享内存  
  
- structure ur code to make sure 一个时刻一份资源只有一个go协程在管理  
  
## goroutine概念：  
  
- thread：线程，所线程间共享内存  
  
- green thread/virtual thread：被运行库或者虚拟机管理而非OS调度的线程，他们不在核心态被管理而是在用户态  
  
- coroutine：协程，不可抢占；但是会有多个停止点（再进入点)  
  
- goroutine  
  
> - 本质上是一种协程，但是不定义自己的停止点和再进入点，而是通过go runtime管理  
> - goruntime会关注goroutine的运行状态并在goroutine阻塞时挂起他们；在他们不阻塞的时候恢复他们  
> - 所以在操作系统层面goroutine变成了可抢占的，但是他们只会在阻塞的时候被抢占（我不知道为什么这里还算是抢占调度）  
  
## goroutine代码示例  
  
1. 关于引用值  
  
下述代码示例最后的结果是会打印三次good day  

    var wg sync.WaitGroup
    for _, salutation := range []string{"hello", "greetings", "good day"} {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println(salutation)
        }()
    }
    wg.Wait()  
  
出现这样结果的原因是循环遍历大概率会在goroutine开始之前结束，  
  
所以这个时候三个goroutine中的salutation都会指向一个range结束之后的单元  
  
但是由于go的垃圾回收机制，它会监测到salutation还有引用  
  
所以将salutation变量的内存搬到堆空间，并恢复其最后一个值也就是good day  
  
2. 关于go协程泄漏  
  
因为go的垃圾回收机制是不会回收goroutine的  
  
所以如果有一个goroutine内部是一个永远都被block的操作，它就一直占用内存直到程序终止  
  
