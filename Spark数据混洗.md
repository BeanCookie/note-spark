#### shuffle概述
Spark shuffle是Spark Job中某些算子触发的操作, 更详细点说,当RDD依赖中出现宽依赖的时候就会触发shuffle操作, shuffle操作通常会伴随磁盘IO、数据序列号以及网络IO等操作, 也正因如此导致shuffle操作需要耗费大量的资源, 这些耗时操作往往会成为一个分布式计算任务的瓶颈。
举个最简单的例子Spark中的reduceByKey算子, 该算子会生成一个新的RDD, 这个子RDD中会对父RDD中相同key的value按照指定的函数操作形成一个新的value。复杂的地方在于,相同的key数据可能存在于父RDD的多个partition中,这就需要我们读取所有partition中相同key值的数据然后聚合再做计算,这就是一个典型的shuffle操作。

#### Spark与Hadoop中的shuffle的联系
在Hadoop中shuffle是连接Map和Reduce之间的桥梁,Map的输出要用到Reduce中必须经过shuffle这个环节。通常shuffle分为两部分：Map阶段的数据准备和Reduce阶段的数据拷贝处理。一般将在Map端的Shuffle称之为Shuffle Write,在Reduce端的Shuffle称之为Shuffle Read。
Spark的shuffle过程与Hadoop的Shuffle过程有着诸多类似, 一些概念可直接套用,例如,Shuffle过程提供数据的一端被称作Map端, Map端每个生成数据的任务称为Mapper, 对应的接收数据的一端被称作Reduce端, Reduce端每个拉取数据的任务称为Reducer, Shuffle 过程本质上都是将Map端获得的数据使用分区器进行划分, 并将数据发送给对应的Reducer的过程
#### Spark shuffle的进化之旅

##### Hash Shuffle
相对于传统的MapReduce, Spark假定大多数情况下shuffle的数据不需要排序,例如 WordCount, 强制排序反而会降低性能。因此不在 Shuffle Read时做Merge Sort, 如果需要合并的操作的话,则会使用聚合(agggregator), 即用了一个 HashMap(实际上是一个 AppendOnlyMap)来将数据进行合并。在Map Task过程按照Hash的方式重组Partition的数据不进行排序。每个Map Task为每个Reduce Task生成一个文件。在此过程中通常会产生大量的文件(即对应为M*R个中间文件,其中M表示Map Task个数,R表示Reduce Task个数), 如果在Reduce Task时需要合并操作的话,会把数据放在一个HashMap中进行合并,如果数据量较大很容易引发OOM。

针对上面的第一个问题,Spark做了改进, 引入了File Consolidation机制。一个Executor上所有的Map Task生成的分区文件只有一份, 即将所有的Map Task相同的分区文件合并,这样每个Executor上最多只生成N个分区文件。但是假如下游Stage的分区数N很大, 还是会在每个Executor上生成N个文件,同样, 如果一个Executor上有K个Core, 还是会开K*N个Writer Handler, 所以这里仍然容易导致OOM。

##### Sort Shuffle
为了更好地解决上面的问题, Spark参考了MapReduce中Shuffle的处理方式,引入基于排序的Shuffle写操作机制。

每个Task不会为后续的每个Task创建单独的文件, 而是将所有对结果写入同一个文件。该文件中的记录首先是按照Partition Id排序, 每个Partition内部再按照Key进行排序, Map Task运行期间会顺序写每个Partition的数据, 同时生成一个索引文件记录每个 Partition的大小和偏移量。

在Reduce阶段, Reduce Task拉取数据做Combine时不再是采用HashMap, 而是采用ExternalAppendOnlyMap, 该数据结构在做Combine时如果内存不足会刷写磁盘, 很大程度的保证了鲁棒性, 避免大数据情况下的OOM。

##### Tungsten-Sort Based Shuffle
从Spark 1.5.0开始, Spark开始了钨丝计划(Tungsten), 目的是优化内存和CPU的使用, 进一步提升Spark的性能。由于使用了堆外内存, 而它基于JDK Sun Unsafe API, 故Tungsten-Sort Based Shuffle也被称为Unsafe Shuffle。

它的做法是将数据记录用二进制的方式存储, 直接在序列化的二进制数据上Sort而不是在 Java对象上, 这样一方面可以减少内存的使用和GC的开销, 另一方面避免shuffle过程中频繁的序列化以及反序列化。在排序过程中, 它提供cache-efficient sorter, 使用一个8bytes的指针, 把排序转化成了一个指针数组的排序,极大的优化了排序性能。