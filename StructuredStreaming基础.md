#### DStream 的不足

相较于 Dataflow 模型中的 Unbounded Stream，DStream 更准确的来说只是微型批处理。

- 缺少 Event Time
  DStream 只有 Processing Time 的概念，Event Time 是数据自带的属性，一般表示数据产生于数据源的时间，在 IoT 以及大多数实时处理领域中 Event Time 都具有更重要的位置。
- 难以处理乱序数据
  数据生成以及将数据移交给处理引擎时可能会有延迟，但是 DStream 没有类型水位线的概念去处理这些无法准时到达的数据。
- 缺乏端到端恰好一次的数据一致性保障
  DStream 虽然有检查点来保存作业进度，但是无法确保恰好一次的处理数据
- 缺乏高级 API
  DStream 只提供了类似 RDD 的 API，可能会降低开发效率和增加维护成本。

#### StructuredStreaming 做出的改进

Structured Streaming 在 Spark 2.0 版本于 2016 年引入，设计思想参考很多 Dataflow、Flink 中的设计思路。

与 DStream 相比 Structured Streaming 它采用的是 Google Dataflow 的思想，将数据流抽象成无边界输入表（unbounded table），然后我们通过对输入表进行查询的方式得到流处理的结果并生成结果表。每当到达一个触发点实时数据流中的数据就会被追加到输入表中，然后触发计算并更新结果表。每次更新结果表时，开发人员都希望将更改写入外部系统，因此 Structured Streaming 提供了三种输出模式：完全模式、追加模式与更新模式。

1. 完全模式：整个更新的结果表将被写入外部存储，这由存储连接器来决定如何写入整张表
2. 追加模式：只将上次触发后追加到结果表的数据写到外部存储。这种模式适用于不希望修改已经存在于结果表的数据的场景。这是默认的输出模式。在这种模式下，窗口内定时触发生成的中间结果会保存到中间状态，最后一次触发生成的结果才会写到结果表
3. 更新模式：只将上次触发后更新到结果表的数据写到外部存储。这与完全模式不同，它只输出上次触发后变更的结果记录。如果窗口中不包含聚合逻辑，则意味着，不需要修改以前结果表中的数据，其实与追加模式无异

![spark005](https://github.com/BeanCookie/note-images/blob/main/spark005.png)

![spark006](https://github.com/BeanCookie/note-images/blob/main/spark006.png)

- 添加 Event Time
  当事件的时间戳包含在接收到的数据中时，StructuredStreaming 提供了基于事件时间处理数据的功能。这是 StructuredStreaming 中引入的一项主要功能，该功能根据现实世界中数据生成的时间提供了一种不同的数据处理方式。这样我们可以处理迟到的数据并获得更准确的结果。
- 实现端到端的恰好一次
  为了实现恰好一次的目标，Structured Streaming 引入 Source、Sink 和 StreamExecution 等组件，可以准确地追踪处理进度，以便从重启或重新处理中处理任何类型的故障。假设每个 Source 都有偏移量（类似于 Kafka 偏移量或 Kinesis 序列号）用于跟踪读取进度，StreamExecution 可以使用检查点和预写日志机制记录来记录数据源偏移量，当发生故障时可以重新计算丢失的数据，并且每个 Sink 都是支持幂等写入或者回滚。在上诉两个条件都具备的前提下就可以实现端到端的恰好一次，即使发送某些不可预测故障时也可以保证数据完整性。
- 水位线
  在 Structured Streaming 中水位线定义为当前最大的事件时间减去晚到时间最大容忍值。对一个 T 时刻的特定窗口，计算单元会维护结果状态，当计算单元接收到的晚到数据满足观察到的最大事件时间减去晚到时间最大容忍值大于 T 的条件时，都允许晚到数据修改结果状态。换句话说，在水位线前面的数据会被处理，后面的数据则会被丢弃
![spark007](https://github.com/BeanCookie/note-images/blob/main/spark007.png)
- DataFrame 和 Dataset API
  Structured Streaming 中可以使用 DataFrame 和 Dataset 表示 Unbounded Stream。
