### Lambda架构解决了哪些问题
批处理技术虽然能够对全量数据进行批量处理，但因为存在一定的数据延迟，所以这里的全量数据并不能反应外部系统的真实情况。
流处理技术能够很好地对当前时间窗口的数据进行处理，但是得到的结果只能反映当前视图并不能反应全量数据的情况。

例如大型的搜索引擎会定期去外部爬取互联网的数据然后生成倒排索引，假设一天计算一次那么就会出现一个问题，在两次生成倒排索引的期间爬取到的网页就不能被索引。事实上像谷歌这样的公司，基本上网页在被爬取后十分钟之内就能被索引到。这就是因为他们采用了类似Lambda的架构，利用流处理维护一个较小的倒排索引，批处理定时生成全量的倒排索引，查询会请求这两类倒排索引并将其结果进行合并即可，返回的结果反映的几乎可以说是实时的。

### 什么是Lambda架构
Lambda架构将整个系统分为3层：

#### 批处理层（Batch Layer）

  理想状态下，任何数据访问都可以从用`query= function(all data)`表示，但是当数据量到达一定程度的时候执行全局查询会耗费相当长的时间与资源。一种比较通用的方式就是对这些查询进行预处理并生成**Batch View**，以往的全局查询就会转移到对应的**Batch View**。

  上述的表达式就转化为：

  `batch view = function(all data)`

  `query = function(batch view)` 

![lambda002](http://git.nuozhilin.site/luzhong/images/raw/branch/master/lambda002.png)

#### 速度层（Speed Layer）

批处理层更新间隔期间系统接收到的数据无法实时反映到**Batch View**，**速度层**就是为了解决这一问题而诞生的，它会持续处理批处理层更新期间的实时数据并生成`real time view`，这一过程可以概括为`real time view = function(real time view, new data)`。`real time view`的数据会在**批处理层**生成`batch view`后丢弃，因为最新的`batch view`中已经包含了`real time view`中的全部数据。

![lambda003](http://git.nuozhilin.site/luzhong/images/raw/branch/master/lambda003.jpg)

#### 服务层（Serving Layer）

**服务层**所做的工作就是加载**批处理层**和**速度层**生成的数据并对外提供查询服务，合并`batch view`和`real time view`中的数据最终得到正确的、实时的结果。

![lambda001](http://git.nuozhilin.site/luzhong/images/raw/branch/master/lambda001.jpg)

#### 总结

`batch view = function(all data) `

`realtime view = function(realtime view, new data) `

`query = function(batch view, realtime view)`

##### 特点

- 容错性

  Lambda架构的每个组件都应该具有很好的容错性，在任何情况下，都不会存在数据丢失的情况，此外，对于那些数据处理的错误，也要能很好地恢复。

- 数据不可变

  这一点其实是Lambda的灵魂，Lambda架构之所有能够这样架构，最重要的原因也是因为数据是不可变的，这样的系统具有很小的复杂性，也更易于管理。它只允许查询和插入数据，不允许删除与更改数据。

- 重算

  重算可以使架构充满灵活性，由于主数据集总是可用的，因此总可以重新定义计算来满足新需求，这就意味着主数据集并没有和什么模式绑定在一起。

##### 优点

一旦批处理层重算生成新的批处理视图以后，那么当前实时视图的结果立刻可以丢弃，也就是说，如果当前的实时视图有什么问题的话，只需要丢弃掉当前的实时视图，并在几小时甚至更短的时间内，整个系统就可以重新回到正常状态