1. 是什么

   MapReduce是Google2003年设计的用于分布式系统的框架。

   - 输入：mapf，reducef，inputFiles
   - 架构：master，worker
   - 实现过程：inputFiles -> chunks -> map -> intermediate files(key, value pairs)
   - intermediate files%R -> reducef -> final results

2. 实现细节

   2.1 Worker职责：

   - ask for tasks periodically
   - map task: 若存在mapNo对应临时文件，先删除临时文件，inputfiles喂给mapf生成中间变量键值对，hash(key)%R将键值对分为R份，存储在R个临时文件mr-mapTaskNo-ri中
   - reduce task: 读取reduceNo对应的所有临时文件mr-*-ri将他们的values按照key聚合起来，喂给reducef，生成最终结果，生成成功后删除临时文件

   2.2 Master职责：

   - reply for issueTask Request：先遍历Map Task，有Idle状态task分配Idle状态的；全部Map Task成功后，分配reduce task
   - 维护Map Task和Reduce Task列表和每个Task的状态
   - 超时的Task被重新分配
   - reply for UpdateTaskStatus Request

3. 实际实现：

   - 实际实现不是通过临时文件传递中间变量，而是通过Google GFS

   - 部署Google GFS的worker可能和接收到map任务的worker一致，所以读input files可以在本地读
   - reducef可能受到switch bottleneck的限制

