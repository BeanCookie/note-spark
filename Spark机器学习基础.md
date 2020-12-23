#### 什么是机器学习

##### AlphaGo

AlphaGo 的研究计划于 2014 年引导，此后和之前的围棋程序相比表现出显著提升。在和 Crazy Stone 和 Zen 等其他围棋程序的 500 局比赛中[[15\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-15)，单机版AlphaGo（运行于一台电脑上）仅输一局[[16\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-16)。而在其后的对局中，分布式版AlphaGo（以[分布式运算](https://zh.wikipedia.org/wiki/分散式運算)运行于多台电脑上）在500局比赛中全部获胜，且对抗运行在单机上的AlphaGo约有77%的胜率。2015年10月的分布式运算版本AlphaGo使用了1,202块[CPU](https://zh.wikipedia.org/wiki/CPU)及176块[GPU](https://zh.wikipedia.org/wiki/GPU)。[[10\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-DeepMindnature2016-10)

- 2015 年 10 月，AlphaGo 击败[樊麾](https://zh.wikipedia.org/wiki/樊麾)，成为第一个无需[让子](https://zh.wikipedia.org/wiki/圍棋的讓子)即可在 19 路棋盘上击败围棋[职业棋手](https://zh.wikipedia.org/wiki/職業棋士)的[电脑围棋](https://zh.wikipedia.org/wiki/电脑围棋)程序，写下了历史，并于 2016 年 1 月发表在知名期刊《[自然](<https://zh.wikipedia.org/wiki/自然_(期刊)>)》。[[8\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-googlego-8)[[11\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-bbcgo-11)
- 2016 年 3 月，透过自我对弈数以万计盘进行练习强化，AlphaGo 在一场[五番棋](https://zh.wikipedia.org/wiki/AlphaGo李世石五番棋)比赛中 4:1 击败顶尖职业棋手[李世石](https://zh.wikipedia.org/wiki/李世石)，成为第一个不借助让子而击败[围棋职业九段](https://zh.wikipedia.org/wiki/围棋段位制)棋手的电脑围棋程序，立下了里程碑。[[17\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-17)五局赛后[韩国棋院](https://zh.wikipedia.org/wiki/韓國棋院)授予AlphaGo有史以来第一位[名誉职业九段](https://zh.wikipedia.org/wiki/圍棋段位制)[[18\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-18)。
- 2016 年 7 月 18 日，因[柯洁](https://zh.wikipedia.org/wiki/柯洁)那段时间状态不佳，其在[Go Ratings](https://zh.wikipedia.org/wiki/Go_Ratings)网站上的 WHR 等级分下滑，AlphaGo 得以在 Go Ratings 网站的排名中位列世界第一，但几天之后，柯洁便又反超了 AlphaGo[[19\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-19)。2017年2月初，Go Ratings 网站删除了 AlphaGo、[DeepZenGo](https://zh.wikipedia.org/wiki/DeepZenGo)等围棋人工智能在该网站上的所有信息。
- 2016 年 12 月 29 日至 2017 年 1 月 4 日，再度强化的 AlphaGo 以“[Master](<https://zh.wikipedia.org/wiki/Master_(围棋软件)>)”为账号名称，在未公开其真实身份的情况下，借非正式的网络快棋对战进行测试，挑战中韩日台的一流高手，测试结束时 60 战全胜[[20\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-20)。
- 2017 年 5 月 23 至 27 日在[乌镇围棋峰会](https://zh.wikipedia.org/wiki/乌镇围棋峰会)上，最新的强化版 AlphaGo 和世界第一棋手柯洁比试、并配合八段棋手协同作战与对决五位顶尖九段棋手等五场比赛，获取三比零全胜的战绩，团队战与组队战也全胜，此次 AlphaGo 利用谷歌 TPU 运行，加上快速进化的机器学习法，运算资源消耗仅李世石版本的十分之一。[[21\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-21)在与柯洁的比赛结束后，[中国围棋协会](https://zh.wikipedia.org/wiki/中国围棋协会)授予AlphaGo[职业围棋九段](https://zh.wikipedia.org/wiki/围棋段位制)的称号。[[22\]](https://zh.wikipedia.org/wiki/AlphaGo#cite_note-AlphaGo_9-dan-22)

AlphaGo 在没有人类对手后，AlphaGo 之父[杰米斯·哈萨比斯](https://zh.wikipedia.org/wiki/杰米斯·哈萨比斯)宣布 AlphaGo 退役。而从业余棋手的水平到世界第一，AlphaGo 的棋力获取这样的进步，仅仅花了两年左右。

##### 人工智能、机器学习和深度学习

提起 AlphaGo 大家自然会联想到人工智能、机器学习和深度学习，同样想要解释什么是机器学习就先要搞清楚这三者之间的关系。用北京市的环线来形容三者的关系实在最形象不过了（见图 1-1）：人工智能（Artificial Intelligence）涵盖范围最广，三环以内都可以叫人工智能，它关注的问题和方法也最杂，包括知识推理、逻辑规划以及机器人等方面。机器学习（Machine Learning）住在二环，是人工智能的核心区域，也是当前发展最迅猛的一部分，子算法流派枝繁叶茂，但思想比较统一，本书就是对机器学习“查户口”。至于当下“网红”——深度学习（Deep Learning），其实原本是从机器学习的神经网络子算法分支发展出来的一系列成果，知识体系一脉相承，只不过近年大出风头，干脆重新起了个名字“单飞”了。

![p01](https://miro.medium.com/max/700/0*KOKNUnsgpbLau8i6.png)

那么机器学习到底是什么呢？没有什么特别的，形象地说，就是一个“苦命”的外包程序员。以前我们写程序时，先要知道输入什么数据、输出什么数据，然后设计功能函数，最后一行一行敲代码来实现。现在有了机器学习，我们写程序就简便多了，即把输入输出数据一股脑儿全扔过去，这位外包程序员二话不说就吭哧吭哧把后面的活全干了。我们给这个过程起了一个好听的名字——“机器学习”。

#### 机器学习基本原理

人工智能相关的各种技术命名都是非常的有创意就显得很高大上，导致我们理解起来可能会有偏差。如何我们把“机器学习”翻译成“统计模型训练”或许可以更容易的描述实际概念，机器学习的过程更接近于马戏团里的动物训练，譬如训练海豹，训练员给海豹一个信号要它拍手，最开始海豹当然不知道要做什么，它可能做出各种动作，如点头、扭动身体，但只要它无意中做出了拍手的动作，训练员就会奖励它一条小鱼。海豹希望吃到小鱼，但它没有那么聪明，无法立即明白听到信号只要拍手就能吃小鱼，需要训练员花费大量的时间，不断给它反馈。久而久之，海豹形成了条件反射，听到信号就拍手，训练就成功了。

#### 机器学习基本概念

假如我们正在教小朋友识字（一、二、三）。我们首先会拿出 3 张卡片，然后便让小朋友看卡片，一边说“一条横线的是一、两条横线的是二、三条横线的是三”

![p02](https://miro.medium.com/max/700/0*RBG1kxYGm_htOsOF.png)

当重复的次数足够多时，小朋友就学会了一个新技能 — — 认识汉字：一、二、三

![p03](https://miro.medium.com/max/700/0*0MDazJ2HkR0yxufU.png)

##### 数据集

上面提到的认字的卡片

##### 特征

上面提到的“一条横线，两条横线”这种区分不同汉字的属性

##### 模型

学会了识字后总结出来的规律

#### 机器学习分类

##### 有监督学习

有监督学习是指我们给算法一个数据集，并且给定正确答案。机器通过数据来学习正确答案的计算方法。

我们准备了一大堆猫和狗的照片，我们想让机器学会如何识别猫和狗。当我们使用监督学习的时候，我们需要给这些照片打上标签

![p04](https://miro.medium.com/max/700/0*Z7IUjdlZ8lszau2v.png)

我们给照片打的标签就是“正确答案”，机器通过大量学习，就可以学会在新照片中认出猫和狗

![p06](https://miro.medium.com/max/700/0*I9GoVNCGE6fnwj6e.png)

##### 无监督学习

我们把一堆猫和狗的照片给机器，不给这些照片打任何标签，但是我们希望机器能够将这些照片分分类

![p07](https://miro.medium.com/max/700/0*dtaNnmvi23kWkOlb.png)

通过学习，机器会把这些照片分为 2 类，一类都是猫的照片，一类都是狗的照片。虽然跟上面的监督学习看上去结果差不多，但是有着本质的差别：**非监督学习中，虽然照片分为了猫和狗，但是机器并不知道哪个是猫，哪个是狗。对于机器来说，相当于分成了 A、B 两类**

![p08](https://miro.medium.com/max/700/0*NiBVCmnH_-IolGwx.png)

#### 实施机器学习 7 个步骤

1. 收集数据

   鸢【音：yuān】尾花（Iris）是单子叶百合目花卉，是一种比较常见的花，可能不经意间你就能在某个公园里碰见它，而且鸢尾花的品种较多。鸢尾花数据集可能是模式识别、机器学习等领域里被使用最多的一个数据集。

2. 数据准备

   在这个例子中，我们的数据是很工整的，但是在实际情况中，我们收集到的数据会有很多问题，所以会涉及到数据清洗等工作。

   为了检验、评价神经网络我们通常会把收集到的数据分为训练数据和测试数据，一般用于训练的数据可以是所有数据的 70%，剩下的 30%可以拿来测试学习结果。如果你想问为什么要分开成两批，那就想想我们读书时的日子，考试题和作业题大部分都是不一样的这也是同一个道理。

3. 选择一个模型

   研究人员和数据科学家多年来创造了许多模型。有些非常适合图像数据，有些非常适合于序列（如文本或音乐），有些用于数字数据，有些用于基于文本的数据。

4. 训练

   大部分人都认为这个是最重要的部分，其实并非如此~ 数据数量和质量、还有模型的选择比训练本身重要更多（训练知识台上的 3 分钟，更重要的是台下的 10 年功）。这个过程就不需要人来参与的，机器独立就可以完成，整个过程就好像是在做算术题。因为机器学习的本质就是**将问题转化为数学问题，然后解答数学题的过程**

5. 评估

   一旦训练完成，就可以评估模型是否有用。这是我们之前预留的验证集和测试集发挥作用的地方。

6. 参数调整

   完成评估后，您可能希望了解是否可以以任何方式进一步改进训练。我们可以通过调整参数来做到这一点。当我们进行训练时，我们隐含地假设了一些参数，我们可以通过认为的调整这些参数让模型表现的更出色。

7. 预测

   我们上面的 6 个步骤都是为了这一步来服务的，这也是机器学习的价值。有了上诉操作得到的模型我们就可以判断一朵新的鸢尾花属于什么类型了。

#### 机器学习实战

##### KNN（K-近邻算法）

**多数表决**：这个词也许会让你联想到举手、表决器或者投票箱，但这都只是形式，其实质是数人数。物以类聚，人以群分，你是哪一类人，查查你的朋友圈也许就能知道答案

**表决权**：表决权就是谁可以参与表决，换句话说就是朋友圈中的“人”可以参与表决，对于 KNN 这个圈就是由距离决定的。具体来说，根据样本各个维度的值可以作出一个个数据点，我们只需要度量点与点之间的距离，然后若想给某个点划分“朋友圈”，只需要以这个点为圆心，就可以找到与它临近的点有哪些，从而构成它的“朋友圈”。

KNN 是 K-Nearest Neighbor 的英文简写，中文直译就是 K 个最近邻。KNN 的核心在于多数表决，而谁有投票表决权呢？就是这个“最近邻”，也就是以待分类样本点为中心，距离最近的 K 个点。这 K 个点中什么类别的占比最多，待分类样本点就属于什么类别

![p09](https://pic3.zhimg.com/80/v2-c3f1d2553e7467d7da5f9cd538d2b49a_720w.png)

##### 使用 Sklearn 进行机器学习

Sklearn 本身就有很多数据库可以用来练习。 以 Iris 的数据为例，这种花有四个属性：萼长度，花萼宽度，花瓣长度，花瓣宽度，根据这些属性把花分为三类（Setosa，Versicolour，Virginica）。

![p10](https://mofanpy.com/static/results-small/sklearn/2_2_1.png)

| sepal_length        | sepal_width         | petal_length        | petal_width         | species |
| ------------------- | ------------------- | ------------------- | ------------------- | ------- |
| 花萼长度（单位 cm） | 花萼宽度（单位 cm） | 花瓣长度（单位 cm） | 花瓣宽度（单位 cm） | 种类    |

```python
# 导入模块
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier

# 数据准备
iris = datasets.load_iris()
iris_X = iris.data
iris_y = iris.target

X_train, X_test, y_train, y_test = train_test_split(
    iris_X, iris_y, test_size=0.3)

# 选择一个模型
knn = KNeighborsClassifier()

# 训练
knn.fit(X_train, y_train)

# 预测
print(knn.predict(X_test))
print(y_test)
```

#### 参考

- 《机器学习实战》

- 《机器学习算法的数学解析与 Python 实现》

- https://mofanpy.com/tutorials/machine-learning/sklearn/general-pattern/

- https://medium.com/@pkqiang49/%E8%BF%99%E5%8F%AF%E8%83%BD%E6%98%AF2019%E5%B9%B4%E6%9C%80%E5%A5%BD%E7%9A%84-%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0-%E7%A7%91%E6%99%AE%E6%96%87-8c150efb7c39
