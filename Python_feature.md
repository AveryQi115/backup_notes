之前面试的时候被问到了，没答上来，记录一下  
  
## 生成器  
  
generator就是一个惰性计算的序列。  
  
同样一个序列，`f = [x**2 for x in range(10000)]`是将f在声明时就计算好，消耗的空间也很大。  
  
而`f=(x**2 for x in range(10000))`是一个生成器，规定了后续数据产生的规则，但此时不计算数据。  
  
生成器f需要通过调用`next(f)`来得到下一个数据，当没有下一个数据时，抛出异常StopIteration。  
  
一般不用`next(f)`而是直接循环`for i in f:`就可以了，到没有数据的时候循环终止不抛出异常。  
  
## 函数中的生成器  
  
将下述计算斐波那契数列的函数修改一下  
  
    def f(n):
      i,a,b = 0,0,1
      while(i<n)
        print(b)
        a,b = b,a+b
        i = i+1
      return 'done'  
          
将其中的print改为`yield b`,则该函数变成了一个迭代器  
  
    def f(n):
      i,a,b = 0,0,1
      while(i<n)
        yield b
        a,b = b,a+b
        i = i+1
      return 'done'  
    g = f()
    next(g)
    
每次调用`next(g)`函数都会走到yield处停止然后下一次调用从上一次yield之后开始  
  
## map/reduce  
  
map/reduce就是接受一个函数和一个iterable,然后只是生成计算每个元素的方法或根据前一个元素和当前元素生成新元素的方法，并不立刻计算，输出一个iterator  
  
## 函数作为返回值  
  
    def lazy_sum(*args):
      def sum():
        ax = 0
        for n in args:
          ax += n
        return ax
      return sum
      
    f = lazy_sum(1,3,5,7,9)
    f() # compute1+3+5+7+9
    
在上面的代码中lazy_sum给sum函数提供了一个闭包环境，使得sum函数接受了lazy_sum的参数和变量  
  
但是不要在这种闭包中做循环，并将循环的局部变量传给内部函数当做参数使用，因为每次lazy_sum返回的都是一个新生成的函数了(有空可以看看怎么实现的)  
  
## 装饰器  
    
    def log(func):
      def wrapper(*args, **kw):
        print("call %s():" %func.__name__)
        return func(*args,**kw)
      return wrapper
      
    @log
    def now():
      return 1  
     
装饰器就是一个函数作为参数，再返回函数的函数。如果在now前面加了装饰器log，其作用就是在运行now()之前做了一次`now = log(now)`  
  
装饰器的作用是为了在运行时动态的改变函数的功能，flask框架中的app路由就是通过装饰器实现的
