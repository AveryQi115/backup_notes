之前面试的时候被问到了，没答上来，记录一下  
  
## 生成器  
  
generator就是一个惰性计算的序列。  
  
同样一个序列，'f = [x**2 for x in range(10000)]' 是将f在声明时就计算好，消耗的空间也很大。  
  
而'f=(x**2 for x in range(10000))'是一个生成器，规定了后续数据产生的规则，但此时不计算数据。  
  
生成器f需要通过调用'next(f)'来得到下一个数据，当没有下一个数据时，抛出异常StopIteration。  
  
一般不用'next(f)'而是直接循环'for i in f:'就可以了，到没有数据的时候循环终止不抛出异常。  
  
## 函数中的生成器  
  
将下述计算斐波那契数列的函数修改一下  
    def f(n):
      i,a,b = 0,0,1
      while(i<n)
        print(b)
        a,b = b,a+b
        i = i+1
      return 'done'
将其中的print改为yield b,则该函数变成了一个迭代器  
    def f(n):
      i,a,b = 0,0,1
      while(i<n)
        yield b
        a,b = b,a+b
        i = i+1
      return 'done'  
    g = f()
    next(g)
每次调用'next(g)'函数都会走到yield处停止然后下一次调用从上一次yield之后开始  
  
