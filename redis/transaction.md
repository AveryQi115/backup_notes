## 事务  
  
redis中的事务以MULTI开头，EXEC结尾  
  
redis中事务的执行可能出现两种错误：  
  
1. 指令书写错误：在执行前就能发现，一个事务中出现了这种错误，其中正常的指令也不会执行  
  
2. 执行时才能发现的错误：比如指令和键类型不匹配等，一个事务中出现了这种错误，其他正常的指令会执行  
  
redis中没有事务回滚这个说法，所以如果出现了第2中错误，要自己擦屁股  
  
## WATCH指令  
  
这个指令我说不清楚，举个例子  
  
如果分开使用get和set会导致多个指令并发执行时出现race condition  
  
但是如果放在事务中的话，get要到事务执行时才会执行，而set需要拿到get返回的值才能执行+1操作，所以也不能把他俩放一个事务里  
  
`WATCH key`指令会监视key，如果在事务执行前，WATCH监视的键发生了改变的话，事务就不会执行  
  
    WATCH key
    SET key 2
    MULTI
    SET key 3
    EXEC
    GET key  
    
 最后返回的key是2，中间`SET key 3`的事务因为watch的键出现了改变，所以不会执行  
   
 避免出现race condition的使用watch的做法：  
   
    WATCH key
    $value = GET key  
    MULTI
    SET key,$value+1  
    EXEC
