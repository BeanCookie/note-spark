#### 架构描述
Driver是用户编写的数据处理逻辑，这个逻辑中包含用户创建的SparkContext。SparkContext是用户逻辑与Spark集群主要的交互接口，它会和Cluster Manager交互，包括向它申请计算资源等。ClusterManager负责集群的资源管理和调度，现在支持Standalone、Apache Mesos、Hadoop的YARN以及K8S。Worker Node是集群中可以执行计算任务的节点。Executor是在一个Worker Node上为某应用启动的一个进程，该进程负责运行任务，并且负责将数据存在内存或者磁盘上。Task是被送到某个Executor上的计算单元。每个应用都有各自独立的Executor，计算最终在计算节点的Executor中执行。

![spark001](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark001.jpg)

#### 相关词汇表

| 关键词          | 含义描述                                                     |
| --------------- | ------------------------------------------------------------ |
| Driver program  | 运行应用程序的main()函数并创建SparkContext的过程             |
| Cluster manager | 用于在群集上获取资源的外部服务(例如: Standalone、Mesos和YARN) |
| Worker node     | 可以在集群中运行应用程序代码的任何节点                       |
| Executor        | 为工作节点上的应用程序启动的进程, 该进程运行任务并将数据跨任务存储在内存或磁盘存储中, 每个应用程序都有自己的执行程序。 |
| Task            |                                                              |
| Job             |                                                              |
| Stage           |                                                              |

#### 应用生命周期

1. 用户程序创建SparkContext时，新创建的SparkContext实例会连接到Cluster Manager。Cluster Manager会根据用户提交时设置的CPU和内存等信息为本次提交分配计算资源启动Executor
2. Driver会将用户程序划分为不同的执行阶段，每个执行阶段由一组完全相同的Task组成，这些Task分别作用于待处理数据的不同分区。在阶段划分完成和Task创建后，Driver会向Executor发送Task
3. Executor在接收到Task后，会下载Task的运行时依赖，在准备好Task的执行环境后，会开始执行Task，并且将Task的运行状态汇报给Driver
4. Driver会根据收到的Task的运行状态来处理不同的状态更新。Task分为两种：一种是Shuffle Map Task，它实现数据的重新洗牌，洗牌的结果保存到Executor所在节点的文件系统中；另外一种是Result Task，它负责生成结果数据
5. Driver会不断地调用Task，将Task发送到Executor执行，在所有的Task都正确执行或者超过执行次数的限制仍然没有执行成功时停止