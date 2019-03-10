---
mathjax:包括
标题:快速入门指南
nav-title:快速入门
nav-parent_id:机器学习
nav-pos: 0
---
< !
授权给Apache软件基金会(ASF)
或者更多贡献者许可协议。查看通知文件
与此工作一起分发，以获取更多信息
关于版权的所有权。ASF许可此文件
在Apache许可下，2.0版
“许可”);除非符合规定，否则您不能使用此文件
的许可证。你可于
http://www.apache.org/licenses/LICENSE-2.0
除非适用法律要求或经书面同意，
在授权下发布的软件是在
无任何保证或条件
善意的，明示的或暗示的。参见许可证
管理权限和限制的特定语言
根据许可证。
-->
*这将由TOC代替
{:toc}

## 介绍

FlinkML的设计目的是使从数据中学习成为一个直接的过程，抽象出来
大数据学习任务的复杂性。在这个
快速入门指南我们将展示解决一个简单的监督学习问题是多么容易
使用FlinkML。但首先是一些基础知识，如果已经学过了，可以跳过下面几行
熟悉机器学习(ML)。
Murphy [[1]](# Murphy) ML定义的是检测数据中的模式，并使用这些模式
学习预测未来的模式。我们可以把大多数ML算法分类
两大类:监督学习和非监督学习。
** *监督学习**处理从一组输入中学习一个函数(映射)
(特征)到一组输出。学习是使用*训练集*(输入，
输出)我们用来近似映射函数的对。监督学习问题包括
进一步分为分类和回归问题。在分类问题中，我们尝试
预测示例所属的*class*，例如用户是否要单击
广告与否。另一方面，回归问题是关于预测(实际)数值的
值，通常称为因变量，例如明天的温度。
** *无监督学习**处理发现数据中的模式和规律。一个例子
其中之一是“集群”，在这里我们试图发现来自
描述性的功能。例如，无监督学习也可以用于特征选择
通过[主成分分析](https://en.wikipedia.org/wiki/Principal_component_analysis)。
##链接FlinkML
为了在项目中使用FlinkML，首先必须这样做
[设置一个Flink程序]({{站点。baseurl}} / dev / linking_with_flink.html)。
接下来，您必须将FlinkML依赖项添加到“pom”中。项目的xml ':
{% highlight xml %}
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-ml{{ site.scala_version_suffix }}</artifactId>
  <version>{{site.version }}</version>
</dependency>
{% endhighlight %}

## 数据加载

要加载与FlinkML一起使用的数据，我们可以使用Flink的ETL功能，或者专门使用它
用于格式化数据的函数，如LibSVM格式。对于监督学习的问题就是这样
通常使用' LabeledVector '类来表示'(标签，特性)'示例。“LabeledVector”
对象将有一个FlinkML“Vector”成员表示示例的特性，以及一个“Double”
成员，它表示标签，标签可以是分类问题中的类，也可以是从属类
回归问题的变量。
举个例子，我们可以使用哈伯曼生存数据集，你可以
[从UCI ML存储库下载](http://archive.ics.uci.edu/ml/computerlearing-databases/haberman/haberman.data)。
该数据集*“包含了一项关于患者生存的研究的案例
乳腺癌手术”*。数据来自一个逗号分隔的文件，前3列在其中
是特征，最后一列是类，第四列表示患者是否
存活5年或更长时间(标签1)，或在5年内死亡(标签2)。您可以查看[UCI]
(https://archive.ics.uci.edu/ml/datasets/Haberman%27s+Survival)获取更多关于数据的信息。
我们可以先加载数据作为‘DataSet[String]’:
{% highlight scala %}

import org.apache.flink.api.scala._

val env = ExecutionEnvironment.getExecutionEnvironment

val survival = env.readCsvFile[(String, String, String, String)]("/path/to/haberman.data")

{% endhighlight %}

我们现在可以将数据转换为“DataSet[LabeledVector]”。这将允许我们使用
数据集与FlinkML分类算法。我们知道数据集的第四个元素
是类标签，其余的是特性，所以我们可以像这样构建“LabeledVector”元素:
{% highlight scala %}

import org.apache.flink.ml.common.LabeledVector
import org.apache.flink.ml.math.DenseVector

val survivalLV = survival
  .map{tuple =>
    val list = tuple.productIterator.toList
    val numList = list.map(_.asInstanceOf[String].toDouble)
    LabeledVector(numList(3), DenseVector(numList.take(3).toArray))
  }

{% endhighlight %}

然后我们可以用这些数据来训练学习者。不过，我们将使用另一个数据集来举例说明
建立一个学习者;这将允许我们展示如何导入其他数据集格式。
* * * * LibSVM文件
ML数据集的一种常见格式是LibSVM格式，使用该格式的许多数据集可以是
发现[在LibSVM数据集网站](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/)。FlinkML提供了加载实用程序
通过' MLUtils '提供的' readLibSVM '函数使用LibSVM格式的数据集
对象。
您还可以使用' writeLibSVM '函数以LibSVM格式保存数据集。
让我们导入svmguide1数据集。你可下载
(训练集)(http://www.csie.ntu.edu.tw/ ~ cjlin libsvmtools /数据/二进制/ svmguide1)
(http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary/svmguide1.t)。
这是一个天文粒子二分类数据集，由Hsu等[[3]](# Hsu)在他们的
实用支持向量机(SVM)指南。它包含4个数值特征，以及类标签。
我们可以简单地导入数据集，然后使用:
{% highlight scala %}

import org.apache.flink.ml.MLUtils

val astroTrain: DataSet[LabeledVector] = MLUtils.readLibSVM(env, "/path/to/svmguide1")
val astroTest: DataSet[(Vector, Double)] = MLUtils.readLibSVM(env, "/path/to/svmguide1.t")
      .map(x => (x.vector, x.label))

{% endhighlight %}

This gives us two `DataSet` objects that we will use in the following section to
create a classifier.

# #分类
一旦我们导入了数据集，我们就可以训练一个“预测器”，比如线性SVM分类器。
我们可以为分类器设置一些参数。这里我们设置了block参数，
它由底层CoCoA算法[[2]](#jaggi)使用来分割输入。的
正则化参数决定所应用的$l_2$正则化的数量
避免过度拟合。步长决定权重向量更新到的贡献
下一个权向量的值。此参数设置初始步长。
{% highlight scala %}

import org.apache.flink.ml.classification.SVM

val svm = SVM()
  .setBlocks(env.getParallelism)
  .setIterations(100)
  .setRegularization(0.001)
  .setStepsize(0.1)
  .setSeed(42)

svm.fit(astroTrain)

{% endhighlight %}

现在我们可以对测试集进行预测，并使用“evaluate”函数创建(真值、预测)对。
{% highlight scala %}

val evaluationPairs: DataSet[(Double, Double)] = svm.evaluate(astroTest)

{% endhighlight %}

接下来，我们将了解如何预处理数据，并使用FlinkML的ML管道功能。
##数据预处理和管道
在使用SVM分类时，通常鼓励[[3]](#hsu)的预处理步骤是缩放
将输入的特性设置为[0,1]范围，以避免具有极值的特性
的休息。
FlinkML有许多“变形金刚”，比如用于预处理数据的“MinMaxScaler”，
其中一个关键功能是将“变形金刚”和“预测器”连接在一起。这允许
我们可以运行相同的转换管道，并对火车和测试数据进行预测
一种直接且类型安全的方式。您可以阅读更多关于FlinkML管道系统的信息
[在pipeline文档中](pipelins .html)。
让我们首先为数据集中的特性创建一个规范化转换器，并将其链接到a
新的支持向量机分类器。
{% highlight scala %}

import org.apache.flink.ml.preprocessing.MinMaxScaler

val scaler = MinMaxScaler()

val scaledSVM = scaler.chainPredictor(svm)

{% endhighlight %}

现在，我们可以使用新创建的管道对测试集进行预测。
首先我们再次调用fit来训练定标器和SVM分类器。
然后将测试集的数据自动缩放，然后将其传递给SVM
作出预测。
{% highlight scala %}

scaledSVM.fit(astroTrain)

val evaluationPairsScaled: DataSet[(Double, Double)] = scaledSVM.evaluate(astroTest)

{% endhighlight %}

The scaled inputs should give us better prediction performance.

##从这里去哪里
这个快速入门指南可以作为FlinkML基本概念的介绍，但是还有很多
你能做的更多。
我们建议浏览[FlinkML文档]({{站点)。baseurl}}/dev/libs/ml/index.html)，并尝试不同的方法
算法。
一个非常好的开始方法是使用来自UCI ML的有趣数据集
库和LibSVM数据集。
从[Kaggle](https://www.kaggle.com)或
[DrivenData](http://www.drivendata.org/)也是通过与他人竞争来学习的好方法
数据科学家。
如果你想贡献一些新的算法，看看我们的
贡献指南(contribution_guide.html)。
** 引用**
<a name="murphy"></a>[1] Murphy, Kevin P. *Machine learning: a probabilistic perspective.* MIT
press, 2012.

<a name="jaggi"></a>[2] Jaggi, Martin, et al. *Communication-efficient distributed dual
coordinate ascent.* Advances in Neural Information Processing Systems. 2014.

<a name="hsu"></a>[3] Hsu, Chih-Wei, Chih-Chung Chang, and Chih-Jen Lin.
 *A practical guide to support vector classification.* 2003.

{% top %}
