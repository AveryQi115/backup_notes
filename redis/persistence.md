## 持久化  
  
因为redis是一个使用内存的数据库，所以在重启或者复制的时候数据都会丢失，需要把数据实时写入磁盘避免数据丢失。  
  
redis有两种持久化方式，RMD持久化和AOF持久化，两种方式可以同时存在。  
  
## RMD持久化  
  
RMD持久化采用快照的方式，它将当前内存中的数据存储为一次快照，快照生成结束时替代磁盘上的原快照。  
  
快照生成的时间节点：  
  
1. 按照规则生成自动快照 `save m n`规则表示在m时间内若有n个键发生了改动则保存一次快照  
  
2. 手动生成快照  `SAVE`该命令为同步生成快照，生成快照期间数据库不会接受其他请求，生产中不要用  
               `BSAVE`该命令为异步生成快照，数据库在后台生成快照，可接受其他请求  
               `FLUSHALL`清空数据库中所有数据，如果当前有自动快照规则，则触发快照，否则不触发快照  
              
3. 复制时生成快照  
  
### RDB快照生成的具体过程  
  
父进程调用fork创建子进程，子进程和父进程此时共享同一块内存空间  
  
子进程将内存空间中的内容写入到磁盘上的临时文件中  
  
父进程继续接受请求，如果在此时出现了写父进程空间的请求，使用copy-on-write的机制，也就是shadowing，复制一份写的数据，所以不会改变子进程的空间  
  
临时文件写完毕后，临时文件代替磁盘上的原快照，一般是dump.rdb文件  
  
## AOF持久化  
  
对于用redis作缓存的系统来说，数据库中的数据没有那么重要，所以可以接受丢失  
  
但是对于数据很重要的系统来说，RDB持久化的同步时间间隔还是太久了  
  
AOF持久化会写入每一条命令到磁盘中的appendonly.aof文件中  
  
可以设置对aof文件中的命令进行定时优化  
  
默认情况下，aof写入的文件并不是磁盘文件，而是磁盘缓存文件（操作系统决定），由操作系统定时写磁盘  
  
这种情况下数据会有30s左右的丢失，可以通过开启强制立即写来避免  
  
  