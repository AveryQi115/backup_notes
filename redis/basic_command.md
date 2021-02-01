## 字符串类型操作 
  
  `SET key value` 键和值都是字符串类型  
  
  `GET key` 获取键名对应的值  
    
  `INCR key`
  `INCRBY key num` 对于值可转换成整数的类型，增加1或num（这里注意：本身值的属性还是字符串类型，如果值字符串不能转换为整数类型，则错误）  
    
  `APPEND key word` 在值字符串之后追加拼接word字符串  
    
  `MSET key value [key2 value2 key3 value3 ...]` 同时设置多个键值对  
    
  `MGET key [key2 key3 key4...]` 同时返回多个键对应的值（在命令行返回多行，每一行前面有序号；检查在api中是否会返回序号）  
    
## 散列类型操作  
  
在redis中只有散列类型是可以嵌套的，其他的类型比如集合它的元素的值不能是集合  
  
  `HSET key field value` 设置键-散列类型值对  
    
  `HGET key field` 散列类型获取值时需要说明字段  
    
  `HMSET key field value [key2 field2 value2...]`  
    
  `HMGET key field [key2 field2...]`  
    
  `HGETALL key` 只知道键名不知道字段名时候可以获取字段名和值组成的列表，顺序为field1 value1 field2 value2...  
    
  `HKEYS key` 获取当前key全部字段名  
    
  `HVALUES key` 获取当前key全部字段值  
    
## 列表类型操作
  
在redis中的列表类型底层使用双端列表实现，向列表的两端添加删除和查询都是O(1)的时间复杂度  
  
这样redis能实现很多关系型数据库中非常麻烦的操作，代价是使用索引查询列表类型很慢  
  
  `LPUSH key value [key2 value2 key3 value3...]` 左边添加，可一次性添加多个  
    
  `RPUSH key value [key2 value2 key3 value3...]` 右边添加，可一次性添加多个
  
  `LPOP key` 左边弹出  
    
  `RPOP key` 右边弹出
  
  `LRANGE key start stop` 返回列表【start...stop】元素，左闭右闭区间  
   
  `LREM key count value` 删除key列表中前count个值为value的元素  
    
## 集合类型操作  
  
底层实现是使用值为空的hash表  
  
  `SADD key member [member2...]`增加元素，返回成功增加的元素个数  
  
  `SREM key member [member2...]`删除元素，返回成功删除的元素个数，可能不存在  
  
  `SMEMBERS key`获取集合中所有元素  
  
  `SISMEMBER key member`判断元素member是否在集合key中
  
  `SDIFF key1 key2`对称差
  
  `SINTER key1 key2`交集  
  
  `SUNION key1 key2`并集  
  
## 有序集合类型操作  
  
底层实现采用hash表和跳跃表，所以与列表不同，读取中间的元素也可以很快O(logN)，可以简单调整元素的位置。  
  
但是更加耗费内存  
  
  `ZADD key score member [score2 member2...]` 向有序集合中加入一个元素和元素的分数，这个命令也可以用来修改已经存在的元素的分数  
  
  `ZSCORE key member` 获得元素的分数  
  
  `ZRANGE key start stop [WITHSCORE]` 获得排名在【start..stop】之间的元素列表（及分数）  
  
  `ZRANGEBYSCORE key min max [WITHSCORE][LIMIT offset count]` 获得分数在min，max之间及两端的元素列表  
  
  
