## ES  
  
- 分布式文件存储，全文搜索引擎  
  
- 输入数据格式：json  
  
- 通过inverted index支持全文搜索，索引中保存每一个出现过的单词，标识单词出现的每一个文件  
  
- 数值类型用BKD树存储  
  
## 概念  
  
- index(indices) --> type --> document  
  
### ES中的index  
  
- index（名词）：类似关系型数据库中的数据库，一个ES集群中可能部署了多个数据库  
  
- index a document：存储一个文件，ES中的文件类似关系型数据库中的一条记录；ES中的type类似关系型数据库中的表  
  
- inverted index：类似关系型数据库中的索引，ES通过对document中每一个字段添加索引实现全文搜索；没有索引的字段就搜不到  
  
