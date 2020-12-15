#### 从批处理到流处理
传统批处理方法是持续收取数据，然后按照时间将持续的数据切分成多个批次再周期性地执行批次运算。但是这种处理方式在很多业务需求中会产生明细的短板，假设需要计算单位时间出现某种事件转换的次数，如果某个事件转换跨越了所定义的时间划分，传统批处理会将这个实践运算结果带到下一个批次进行计算。更严重的是当出现接收到的事件顺序颠倒情况下，传统批处理仍会将顺序错乱的事件带到下一批次的运算结果中。
#### Dataflow模型
Google在2013年公布了一篇名为《MillWheel: Fault-Tolerant Stream Processing at InternetScale》的论文，用于构建一种高吞吐、低延迟、数据不重不丢、可以有效处理乱序数据且具有容错性保证的分布式流处理框架。
2015年Google有公布了《The Dataflow Model: A PracticalApproach to Balancing Correctness, Latency, and Cost inMassive-Scale, Unbounded, Out-of-Order DataProcessing》用于阐述如何用统一的方式将流处理和批处理进行统一。Spark中的Structured Streaming和现在如日中天的Flink都借鉴了Google MillWheel和Dataflow中的很多设计思想。
Dataflow从四个角度对解析数据处理的四个维度：
1. What results are being computed（计算的结果是什么）
2. Where in event time they are been computed（在何处计算时间）
3. When in processing time they are materialized（什么时候触发计算）
4. How earlier results relate to later refinements（如何修正乱序数据）
#### Unbounded Stream和Bounded Stream
在Dataflow之前对于有限/无限数据集合的描述，一般使用批/流 （Batch/Streaming），对于批处理和流处理，一般情况下是可以互相转化的，比如Spark用微批来模拟流。而Dataflow模型一般将有限/无限数据集合称为Bounded/Unbounded Dataset，而 Streaming/Batch用来特指执行引擎
#### Event Time和Processing Time
在流式处理中通常使用Event Time和Processing Time来定义时间，传感器采集事件时对应的系统时间就是Event Time，然后事件发送到流式系统进行处理，处理的时候对应的系统时间就是Processing Time。
#### Window
因为大多数的业务需求都是要对一段时间的数据进行统计，比如多种时间维度的数据报表、数量增长量和变化曲线等。流计算中引入了Window的概念，这里的Window可以是按时间间隔或事件数量进行定义。
#### Watermarks
因为多种原因数据源产生的数据可能存在延时的情况，这样就会导致当前的Window可能接收到上个Window的数据，但是此时前一个Window已经触发的计算操作，这样就会导致Window数据丢失。这种情况下流计算引入了水位线的概念，设置了水位线之后呢可以延长窗口计算的时间。
#### 如何保障消息可靠性
当我们考虑状态容错时难免会想到精确一次的状态容错，应用在运算时累积的状态，每笔输入的事件反映到状态，更改状态都是精确一次，如果修改超过一次的话也意味着数据引擎产生的结果是不可靠的。

参考:
https://zhuanlan.zhihu.com/p/54739130

https://weread.qq.com/web/reader/483326b071a52591483e940kc81322c012c81e728d9d180