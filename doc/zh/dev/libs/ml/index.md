---
题目:“FlinkML - Flink机器学习”
nav-id:机器学习
nav-show_overview:true
nav-title:机器学习
nav-parent_id:库
nav-pos: 4
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
FlinkML是Flink的机器学习(ML)库。这是Flink社区的一项新努力，
越来越多的算法和贡献者。我们的目标是与FlinkML提供
可伸缩的ML算法、直观的API和帮助最小化端到端ML中的粘合代码的工具
系统。你可以看到更多关于我们目标的细节，以及图书馆在我们的愿景中的走向
和路线图)(https://cwiki.apache.org/confluence/display/FLINK/FlinkML%3A +视觉+和+路线图)。
*这将由TOC代替
{:toc}

## Supported Algorithms

FlinkML目前支持以下算法:

### 监督式学习

* [SVM using Communication efficient distributed dual coordinate ascent (CoCoA)](svm.html)
* [Multiple linear regression](multiple_linear_regression.html)
* [Optimization Framework](optimization.html)

### 无监督式学习

* [k-Nearest neighbors join](knn.html)

### 数据处理

* [Polynomial Features](polynomial_features.html)
* [Standard Scaler](standard_scaler.html)
* [MinMax Scaler](min_max_scaler.html)

### 推荐系统

* [Alternating Least Squares (ALS)](als.html)

### 离群值选择

* [Stochastic Outlier Selection (SOS)](sos.html)

### 公用程式

* [Distance Metrics](distance_metrics.html)
* [Cross Validation](cross_validation.html)

## 开始

您可以查看我们的[快速入门指南](quickstart.html)来获得全面的入门知识
的例子。
如果您想直接进入，您必须[设置一个Flink程序]({{site。baseurl}} / dev / linking_with_flink.html)。
接下来，您必须将FlinkML依赖项添加到“pom”中。项目的xml '。

{% highlight xml %}
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-ml{{ site.scala_version_suffix }}</artifactId>
  <version>{{site.version }}</version>
</dependency>
{% endhighlight %}

请注意，FlinkML目前不是二进制发行版的一部分。
有关集群执行的链接[这里]({{site.baseurl}}/dev/link .html)。
现在可以开始解决分析任务了。
下面的代码片段显示了训练一个多元线性回归模型是多么容易。
{% highlight scala %}
// 带标签(类或实值)的特征向量
val trainingData: DataSet[LabeledVector] = ...
val testingData: DataSet[Vector] = ...

// 另一种方法是使用拆分器将数据集分解为培训和测试数据.
val dataSet: DataSet[LabeledVector] = ...
val trainTestData: DataSet[TrainTestDataSet] = Splitter.trainTestSplit(dataSet)
val trainingData: DataSet[LabeledVector] = trainTestData.training
val testingData: DataSet[Vector] = trainTestData.testing.map(lv => lv.vector)

val mlr = MultipleLinearRegression()
  .setStepsize(1.0)
  .setIterations(100)
  .setConvergenceThreshold(0.001)

mlr.fit(trainingData)

// 拟合后的模型现在可以用来进行预测
val predictions: DataSet[LabeledVector] = mlr.predict(testingData)
{% endhighlight %}

## 管道

FlinkML的一个关键概念是其[scikit-learn](http://scikit-learn.org)启发的管道机制。
它允许您快速构建复杂的数据分析管道，了解它们如何出现在每个数据科学家的日常工作中。
关于FlinkML管道及其内部工作原理的深入描述[这里](pipelins .html)。
下面的示例代码展示了使用FlinkML设置分析管道是多么容易。

{% highlight scala %}
val trainingData: DataSet[LabeledVector] = ...
val testingData: DataSet[Vector] = ...

val scaler = StandardScaler()
val polyFeatures = PolynomialFeatures().setDegree(3)
val mlr = MultipleLinearRegression()

// 构建标准定标器、多项式特征和多元线性回归的流水线
val pipeline = scaler.chainTransformer(polyFeatures).chainPredictor(mlr)

// 训练管道
pipeline.fit(trainingData)

// 计算预测
val predictions: DataSet[LabeledVector] = pipeline.predict(testingData)
{% endhighlight %}
通过调用方法chainTransformer，可以将一个“Transformer”链接到另一个“Transformer”或一组链接的“Transformer”。
如果要将一个“预测器”链接到一个“Transformer”或一组链接的“Transformer”，则必须调用方法“chainPredictor”。


## 如何构建

Flink社区欢迎所有希望参与开发Flink及其库的贡献者。
为了快速开始对FlinkML的贡献，请阅读我们的官方文章
[contribution guide]({{site.baseurl}}/dev/libs/ml/contribution_guide.html).
