1. 是什么

   GFS(google file system)是Google2003年设计的分布式文件管理系统。

   Google公司观察到在实际的分布式系统的实现中：

   - Crash down is the norm other than the exception
   - KB sized operation 不好操作
   - 许多操作是append而不是existing data
   - weak consistency makes system more flexible

   他们假设：

   - client只发起两种读操作：
     - large streaming reads
     - small random reads

   - client发起随机读取之前会自行排序这些读操作，offset不会go back and forth
   - 大量writes为sequential append，少量为random writes不用专门优化
   - 数据processed in a batch manner，可以接受延迟，但对带宽的需求较大

2. 架构

   - 2.1 基本数据结构
     - 文件：client请求文件的读写，文件被分成多个chunk存储在chunkserver里
     - chunk：固定64MB，每个chunk带一个全局唯一且不可变的64B的chunkHandle，一般每个chunk共有3个replica分布在3个chunkserver上
     - log：log存储command并决定command的顺序
     - lease：chunk粒度，一个chunk只有0个或1个chunkserver含有租约，该chunkserver为primary，负责处理所有client发起的writes command，含ttl，过期需重新申请

   - 2.2 master维护的metaInfo
     - 持久化保存：FileNameSpace(不是目录树结构！更像一个LookupTable，每个路径一个读写锁)
     - 持久化保存：File->chunk mapping
     - 持久化保存：latest chunk version
     - 非持久化保存：chunk location相关信息(即使master restart也可以根据heartbeat msg获得)
     - 非持久化保存：chunk lease

   - 2.3 角色架构

     在读操作中，系统存在三个角色：

     - client
     - master
     - chunkserver

     写操作中，存在四个角色：

     - client
     - master
     - primary chunkserver
     - secondary chunkserver

   - 2.4 读写流程

     - 2.4.1 读操作：
       - client----(fileName,offset)---->master
       - client<----(chunkHandle,[]chunkserver)----master
       - client缓存metaInfo，减少与master的交互次数
       - client-----(offset,length)------->chunkserver(与具体哪一个交流取决于网络拓扑)
       - client<------(data)----------chunkserver

     - 2.4.2 追加写操作
       - client----(file,append record request)--->master
       - master:
         - find up-to-date replica(according to version number)
         - pick the primary(according to lease and ttl)
         - increment version
         - tells primary,secondary,new version
         - write new version to disk
       - client<-----(chunkHandle,[]chunkserver,primary)---master
       - Client------(data)---->primary(其实是把数据送给全部chunkserver，chunkserver内部维护一个LRU队列缓存数据，但chunkserver会把请求重定向到primary)
       - primary-----(append log,offset)--->secondary
       - all secondary reply yes: return yes to client; else: return no to client
       - if got no:client retry