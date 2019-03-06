---
标题: 迭代的图像处理
nav-parent id:图
nav-pos: 2
---
<!--
授权给Apache软件基金会(ASF)
或者更多的贡献者许可协议。参见通知文件
随本著作一起分发以获取更多信息
关于版权的所有权。ASF许可这个文件
根据Apache许可证，2.0版
“许可”);除非符合规定，否则不得使用此文件
的许可证。你可于
http://www.apache.org/licenses/LICENSE-2.0
除非适用法律要求或经书面同意，
根据许可协议发布的软件在
“按现状”的基础，没有任何保证或条件
善良，无论是明示的还是暗示的。的许可证
控制权限和限制的特定语言
根据许可证。
-->

Gelly利用Flink的高效迭代算子来支持大规模的迭代图处理。目前，我们提供了以顶点为中心、分散-聚集和聚集-应用模型的实现。在下面的部分中，我们将描述这些抽象，并展示如何在Gelly中使用它们。

* 这个将被TOC替换
{:toc}

## Vertex-Centric迭代
以顶点为中心的模型，也称为“像顶点一样思考”或“Pregel”，从图中顶点的角度表达计算。
计算以同步的迭代步骤(称为超步骤)进行。在每个超步骤中，每个顶点执行一个用户定义的函数。
顶点通过消息与其他顶点通信。一个顶点可以向图中的任何其他顶点发送消息，只要它知道自己的唯一ID。
计算模型如下图所示。虚线框对应的是并行化单元。
在每个超步中，所有活动顶点都执行
相同的用户定义并行计算。超级步骤是同步执行的，因此在一个超级步骤中发送的消息可以保证在下一个超级步骤的开始交付。
<p class="text-center">
    <img alt="Vertex-Centric Computational Model" width="70%" src="{{ site.baseurl }}/fig/vertex-centric supersteps.png"/>
</p>

要在Gelly中使用以顶点为中心的迭代，用户只需要定义顶点计算函数' ComputeFunction '。
该函数和要运行的最大迭代次数作为Gelly的“runVertexCentricIteration”的参数给出。此方法将对输入图执行以顶点为中心的迭代，并返回一个具有更新顶点值的新图。可以定义一个可选的消息组合器“MessageCombiner”，以减少通信成本。
让我们考虑使用以顶点为中心的迭代计算单源最短测试路径。最初，每个顶点都有一个无限远的值，除了从源点到源点的距离为0。在第一个超步骤中，源将距离传播到它的邻居。在以下超步骤中，每个顶点检查其接收到的消息并选择其中的最小距离。如果该距离小于当前值，则更新其状态并为其邻居生成消息。如果一个顶点在超阶跃过程中没有改变它的值，那么它不会为下一个超阶跃产生任何消息给它的邻居。当没有值更新或达到最大超步数时，算法收敛。在该算法中，可以使用消息组合器来减少发送到目标顶点的消息数量。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// 读取输入图
Graph<Long, Double, Double> graph = ...

// 定义最大迭代次数
int maxIterations = 10;

// 执行以顶点为中心的迭代
Graph<Long, Double, Double> result = graph.runVertexCentricIteration(
            new SSSPComputeFunction(), new SSSPCombiner(), maxIterations);

// 结果提取顶点
DataSet<Vertex<Long, Double>> singleSourceShortestPaths = result.getVertices();


// - - -  UDFs - - - //

public static final class SSSPComputeFunction extends ComputeFunction<Long, Double, Double, Double> {

public void compute(Vertex<Long, Double> vertex, MessageIterator<Double> messages) {

    double minDistance = (vertex.getId().equals(srcId)) ? 0d : Double.POSITIVE_INFINITY;

    for (Double msg : messages) {
        minDistance = Math.min(minDistance, msg);
    }

    if (minDistance < vertex.getValue()) {
        setNewVertexValue(minDistance);
        for (Edge<Long, Double> e: getEdges()) {
            sendMessageTo(e.getTarget(), minDistance + e.getValue());
        }
    }
}

// 消息组合器
public static final class SSSPCombiner extends MessageCombiner<Long, Double> {

    public void combineMessages(MessageIterator<Double> messages) {

        double minMessage = Double.POSITIVE_INFINITY;
        for (Double msg: messages) {
           minMessage = Math.min(minMessage, msg);
        }
        sendCombinedMessage(minMessage);
    }
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 读取输入图
val graph: Graph[Long, Double, Double] = ...

// 定义最大迭代次数
val maxIterations = 10

// 执行以顶点为中心的迭代
val result = graph.runVertexCentricIteration(new SSSPComputeFunction, new SSSPCombiner, maxIterations)

// 结果提取顶点
val singleSourceShortestPaths = result.getVertices


// - - -  UDFs - - - //

final class SSSPComputeFunction extends ComputeFunction[Long, Double, Double, Double] {

    override def compute(vertex: Vertex[Long, Double], messages: MessageIterator[Double]) = {

    var minDistance = if (vertex.getId.equals(srcId)) 0 else Double.MaxValue

    while (messages.hasNext) {
        val msg = messages.next
        if (msg < minDistance) {
            minDistance = msg
        }
    }

    if (vertex.getValue > minDistance) {
        setNewVertexValue(minDistance)
        for (edge: Edge[Long, Double] <- getEdges) {
            sendMessageTo(edge.getTarget, vertex.getValue + edge.getValue)
        }
    }
}

// 消息组合器
final class SSSPCombiner extends MessageCombiner[Long, Double] {

    override def combineMessages(messages: MessageIterator[Double]) {

        var minDistance = Double.MaxValue

        while (messages.hasNext) {
          val msg = inMessages.next
          if (msg < minDistance) {
            minDistance = msg
          }
        }
        sendCombinedMessage(minMessage)
    }
}
{% endhighlight %}
</div>
</div>

{% top %}

## 配置以顶点为中心的迭代
可以使用“VertexCentricConfiguration”对象配置以顶点为中心的迭代。
目前可以指定以下参数::

* <strong>Name</strong>: 以顶点为中心的迭代的名称。名称显示在日志和消息中，可以使用' setName() '方法指定

* <strong>Parallelism</strong>: 迭代的并行性。它可以使用' setParallelism() '方法进行设置。

* <strong>非托管内存中的解决方案集</strong>: 定义解决方案集是保存在托管内存中(Flink以序列化形式保存对象的内部方法)还是作为简单的对象映射。默认情况下，解决方案集在托管内存中运行。可以使用' setSolutionSetUnmanagedMemory() '方法设置此属性。

* <strong>Aggregators</strong>: 迭代聚合器可以使用“registerAggregator()”方法注册。迭代聚合器组合
                                每个超级步骤全局聚合一次，并使它们在下一个超级步骤中可用。可以在用户内部访问注册的聚合器-defined `ComputeFunction`.

* <strong>Broadcast Variables</strong>: DataSets can be added as [Broadcast Variables]({{site.baseurl}}/dev/batch/index.html#broadcast-variables) to the `ComputeFunction`, using the `addBroadcastSet()` method.

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}

Graph<Long, Double, Double> graph = ...

// 配置迭代
VertexCentricConfiguration parameters = new VertexCentricConfiguration();

// 设置迭代名称
parameters.setName("Gelly Iteration");

// 设置并行性
parameters.setParallelism(16);

// 注册一个聚合器
parameters.registerAggregator("sumAggregator", new LongSumAggregator());

// 运行以顶点为中心的迭代，同时传递配置参数
Graph<Long, Long, Double> result =
            graph.runVertexCentricIteration(
            new Compute(), null, maxIterations, parameters);

// user-defined function
public static final class Compute extends ComputeFunction {

    LongSumAggregator aggregator = new LongSumAggregator();

    public void preSuperstep() {

        // retrieve the Aggregator
        aggregator = getIterationAggregator("sumAggregator");
    }


    public void compute(Vertex<Long, Long> vertex, MessageIterator inMessages) {

        //do some computation
        Long partialValue = ...

        // aggregate the partial value
        aggregator.aggregate(partialValue);

        // update the vertex value
        setNewVertexValue(...);
    }
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}

val graph: Graph[Long, Long, Double] = ...

val parameters = new VertexCentricConfiguration

// 设置迭代名称
parameters.setName("Gelly Iteration")

// 设置并行性
parameters.setParallelism(16)

// 注册一个聚合器
parameters.registerAggregator("sumAggregator", new LongSumAggregator)

// 运行以顶点为中心的迭代，同时传递配置参数
val result = graph.runVertexCentricIteration(new Compute, new Combiner, maxIterations, parameters)

// user-defined function
final class Compute extends ComputeFunction {

    var aggregator = new LongSumAggregator

    override def preSuperstep {

        // retrieve the Aggregator
        aggregator = getIterationAggregator("sumAggregator")
    }


    override def compute(vertex: Vertex[Long, Long], inMessages: MessageIterator[Long]) {

        //do some computation
        val partialValue = ...

        // aggregate the partial value
        aggregator.aggregate(partialValue)

        // update the vertex value
        setNewVertexValue(...)
    }
}

{% endhighlight %}
</div>
</div>

{% top %}

## 散集迭代
散射-聚集模型，又称“信号/收集”模型，是从图中一个顶点的角度来表达计算。计算以同步的迭代步骤(称为超步骤)进行。在每个超步骤中，顶点为其他顶点生成消息，并根据接收到的消息更新其值。要在Gelly中使用分散-聚集迭代，用户只需要定义顶点在每个超步中的行为:

* <strong>Scatter</strong>: 生成一个顶点将发送给其他顶点的消息。
* <strong>Gather</strong>: 使用接收到的消息更新顶点值。

Gelly提供了分散-聚集迭代的方法。用户只需要实现两个功能，分别对应于分散阶段和聚集阶段。第一个函数是“ScatterFunction”，它允许一个顶点向其他顶点发送消息。在与发送消息相同的父步骤中接收消息。第二个函数是' aggregfunction '，它定义顶点如何根据接收到的消息更新其值。
这些函数和要运行的最大迭代次数作为Gelly的“runscattergather迭代”的参数给出。此方法将在输入图上执行分散-聚集迭代，并返回具有更新顶点值的新图。
分散-聚集迭代可以通过顶点总数、输入次数和输出次数等信息进行扩展。
此外，可以指定要在其上运行分散-收集迭代的邻居类型(in/out/all)。默认情况下，来自内邻居的更新用于修改当前顶点的状态，消息被发送到外邻居。
让我们考虑在下面的图中使用散点-聚集迭代计算单源最短测试路径，并让顶点1作为源。在每个超步中，每个顶点都向其所有邻居发送一个候选距离消息。消息值是该顶点的当前值和连接该顶点与其邻居的边权值的和。在接收候选距离消息时，每个顶点计算最小距离，如果发现较短的路径，则更新其值。如果一个顶点在超阶跃过程中没有改变它的值，那么它就不会为下一个超阶跃生成它的邻居的消息。当没有值更新时，算法收敛
<p class="text-center">
    <img alt="Scatter-gather SSSP superstep 1" width="70%" src="{{ site.baseurl }}/fig/gelly-vc-sssp1.png"/>
</p>

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// 读取输入图
Graph<Long, Double, Double> graph = ...

// 定义最大迭代次数
int maxIterations = 10;

// 执行分散-收集迭代
Graph<Long, Double, Double> result = graph.runScatterGatherIteration(
			new MinDistanceMessenger(), new VertexDistanceUpdater(), maxIterations);

// 结果提取顶点
DataSet<Vertex<Long, Double>> singleSourceShortestPaths = result.getVertices();


// - - -  UDFs - - - //

// scatter: messaging
public static final class MinDistanceMessenger extends ScatterFunction<Long, Double, Double, Double> {

	public void sendMessages(Vertex<Long, Double> vertex) {
		for (Edge<Long, Double> edge : getEdges()) {
			sendMessageTo(edge.getTarget(), vertex.getValue() + edge.getValue());
		}
	}
}

// gather: 顶点更新
public static final class VertexDistanceUpdater extends GatherFunction<Long, Double, Double> {

	public void updateVertex(Vertex<Long, Double> vertex, MessageIterator<Double> inMessages) {
		Double minDistance = Double.MAX_VALUE;

		for (double msg : inMessages) {
			if (msg < minDistance) {
				minDistance = msg;
			}
		}

		if (vertex.getValue() > minDistance) {
			setNewVertexValue(minDistance);
		}
	}
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 读取输入图
val graph: Graph[Long, Double, Double] = ...

// 定义最大迭代次数
val maxIterations = 10

// 执行分散-收集迭代
val result = graph.runScatterGatherIteration(new MinDistanceMessenger, new VertexDistanceUpdater, maxIterations)

// 结果提取顶点
val singleSourceShortestPaths = result.getVertices


// - - -  UDFs - - - //

// messaging
final class MinDistanceMessenger extends ScatterFunction[Long, Double, Double, Double] {

	override def sendMessages(vertex: Vertex[Long, Double]) = {
		for (edge: Edge[Long, Double] <- getEdges) {
			sendMessageTo(edge.getTarget, vertex.getValue + edge.getValue)
		}
	}
}

// 顶点更新
final class VertexDistanceUpdater extends GatherFunction[Long, Double, Double] {

	override def updateVertex(vertex: Vertex[Long, Double], inMessages: MessageIterator[Double]) = {
		var minDistance = Double.MaxValue

		while (inMessages.hasNext) {
		  val msg = inMessages.next
		  if (msg < minDistance) {
			minDistance = msg
		  }
		}

		if (vertex.getValue > minDistance) {
		  setNewVertexValue(minDistance)
		}
	}
}
{% endhighlight %}
</div>
</div>

{% top %}

##配置分散-聚集迭代
可以使用“scattergather配置”对象配置分散-聚集迭代。
目前可以指定以下参数:

* <strong>Name</strong>: 分散-聚集迭代的名称。名称显示在日志和消息中
                         可以使用' setName() '方法指定。

* <strong>Parallelism</strong>: 迭代的并行性。它可以使用' setParallelism() '方法进行设置。

* <strong>Solution set in unmanaged memory</strong>: 定义解决方案集是保存在托管内存中(Flink以序列化形式保存对象的内部方法)还是作为简单的对象映射。默认情况下，解决方案集在托管内存中运行。可以使用' setSolutionSetUnmanagedMemory() '方法设置此属性。

* <strong>Aggregators</strong>:迭代聚合器可以使用“registerAggregator()”方法注册。迭代聚合器组合
                               每个超级步骤全局聚合一次，并使它们在下一个超级步骤中可用。可以在用户内部访问注册的聚合器-defined `ScatterFunction` and `GatherFunction`.

* <strong>Broadcast Variables</strong>: DataSets can be added as [Broadcast Variables]({{site.baseurl}}/dev/batch/index.html#broadcast-variables) to the `ScatterFunction` and `GatherFunction`, using the `addBroadcastSetForUpdateFunction()` and `addBroadcastSetForMessagingFunction()` methods, respectively.

* <strong>Number of Vertices</strong>: 访问迭代中顶点的总数。可以使用' setOptNumVertices() '方法设置此属性。
                                       顶点的数量可以顶中访问点更新函数和传递函数使用的getNumberOfVertices()的方法。如果未设置该选项 in the configuration, this method will return -1.

* <strong>Degrees</strong>:
* <strong>Messaging Direction</strong>: 默认情况下，顶点向它的外邻居发送消息，并根据从内邻居接收到的消息更新它的值。此配置选项允许用户将消息传递方向更改为“EdgeDirection”或“EdgeDirection”。”、“EdgeDirection。”、“EdgeDirection.ALL”。消息传递方向还指示更新方向，即“EdgeDirection”。”、“EdgeDirection。在”和“EdgeDirection。所有的分别的。可以使用' setDirection() '方法设置此属性。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}

Graph<Long, Double, Double> graph = ...

// 配置迭代
ScatterGatherConfiguration parameters = new ScatterGatherConfiguration();

// 设置迭代名称
parameters.setName("Gelly Iteration");

// 设置并行性
parameters.setParallelism(16);

// 注册一个聚合器
parameters.registerAggregator("sumAggregator", new LongSumAggregator());

// 顶点更新运行分散-聚集迭代，同时传递配置参数
Graph<Long, Double, Double> result =
			graph.runScatterGatherIteration(
			new Messenger(), new VertexUpdater(), maxIterations, parameters);

// user-defined functions
public static final class Messenger extends ScatterFunction {...}

public static final class VertexUpdater extends GatherFunction {

	LongSumAggregator aggregator = new LongSumAggregator();

	public void preSuperstep() {

		// retrieve the Aggregator
		aggregator = getIterationAggregator("sumAggregator");
	}


	public void updateVertex(Vertex<Long, Long> vertex, MessageIterator inMessages) {

		//do some computation
		Long partialValue = ...

		// aggregate the partial value
		aggregator.aggregate(partialValue);

		// update the vertex value
		setNewVertexValue(...);
	}
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}

val graph: Graph[Long, Double, Double] = ...

val parameters = new ScatterGatherConfiguration

// 设置迭代名称
parameters.setName("Gelly Iteration")

// 设置并行性
parameters.setParallelism(16)

// 注册一个聚合器
parameters.registerAggregator("sumAggregator", new LongSumAggregator)

// 顶点更新运行分散-聚集迭代，同时传递配置参数
val result = graph.runScatterGatherIteration(new Messenger, new VertexUpdater, maxIterations, parameters)

// user-defined functions
final class Messenger extends ScatterFunction {...}

final class VertexUpdater extends GatherFunction {

	var aggregator = new LongSumAggregator

	override def preSuperstep {

		// retrieve the Aggregator
		aggregator = getIterationAggregator("sumAggregator")
	}


	override def updateVertex(vertex: Vertex[Long, Long], inMessages: MessageIterator[Long]) {

		//do some computation
		val partialValue = ...

		// aggregate the partial value
		aggregator.aggregate(partialValue)

		// update the vertex value
		setNewVertexValue(...)
	}
}

{% endhighlight %}
</div>
</div>

//下面的例子说明了度数的使用以及顶点选项的数量。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}

Graph<Long, Double, Double> graph = ...

// 配置迭代
ScatterGatherConfiguration parameters = new ScatterGatherConfiguration();

// set the number of vertices option to true
parameters.setOptNumVertices(true);

// set the degree option to true
parameters.setOptDegrees(true);

// 顶点更新运行分散-聚集迭代，同时传递配置参数
Graph<Long, Double, Double> result =
			graph.runScatterGatherIteration(
			new Messenger(), new VertexUpdater(), maxIterations, parameters);

// user-defined functions
public static final class Messenger extends ScatterFunction {
	...
	// retrieve the vertex out-degree
	outDegree = getOutDegree();
	...
}

public static final class VertexUpdater extends GatherFunction {
	...
	// get the number of vertices
	long numVertices = getNumberOfVertices();
	...
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}

val graph: Graph[Long, Double, Double] = ...

// 配置迭代
val parameters = new ScatterGatherConfiguration

// set the number of vertices option to true
parameters.setOptNumVertices(true)

// set the degree option to true
parameters.setOptDegrees(true)

// 顶点更新运行分散-聚集迭代，同时传递配置参数
val result = graph.runScatterGatherIteration(new Messenger, new VertexUpdater, maxIterations, parameters)

// user-defined functions
final class Messenger extends ScatterFunction {
	...
	// retrieve the vertex out-degree
	val outDegree = getOutDegree
	...
}

final class VertexUpdater extends GatherFunction {
	...
	// get the number of vertices
	val numVertices = getNumberOfVertices
	...
}

{% endhighlight %}
</div>
</div>

下面的示例演示了edge direction选项的用法。顶点更新它们的值以包含其所有内邻节点的列表。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, HashSet<Long>, Double> graph = ...

// 配置迭代
ScatterGatherConfiguration parameters = new ScatterGatherConfiguration();

// set the messaging direction
parameters.setDirection(EdgeDirection.IN);

// 顶点更新运行分散-聚集迭代，同时传递配置参数
DataSet<Vertex<Long, HashSet<Long>>> result =
			graph.runScatterGatherIteration(
			new Messenger(), new VertexUpdater(), maxIterations, parameters)
			.getVertices();

// user-defined functions
public static final class Messenger extends GatherFunction {...}

public static final class VertexUpdater extends ScatterFunction {...}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val graph: Graph[Long, HashSet[Long], Double] = ...

// 配置迭代
val parameters = new ScatterGatherConfiguration

// set the messaging direction
parameters.setDirection(EdgeDirection.IN)

// 顶点更新运行分散-聚集迭代，同时传递配置参数
val result = graph.runScatterGatherIteration(new Messenger, new VertexUpdater, maxIterations, parameters)
			.getVertices

// user-defined functions
final class Messenger extends ScatterFunction {...}

final class VertexUpdater extends GatherFunction {...}

{% endhighlight %}
</div>
</div>

{% top %}

## Gather-Sum-Apply迭代
就像在散点-收集模型中一样，收集-和-应用也在同步的迭代步骤中进行，称为超步骤。每一个超步骤包括以下三个阶段:

* <strong>Gather</strong>: 在每个顶点的边和邻边并行调用用户定义的函数，生成一个局部值。
* <strong>Sum</strong>: 使用用户定义的简化程序，将在收集阶段生成的部分值聚合为单个值。
* <strong>Apply</strong>:  通过对当前值和Sum阶段生成的聚合值应用一个函数来更新每个顶点值。

让我们考虑在下面的图中使用GSA计算单源最短测试路径，并让顶点1作为源。在“聚集”阶段，我们计算新的候选距离，通过添加每个顶点值和边缘权重。在“和”中，候选距离按顶点ID分组，选择最小距离。在“应用”中，将新计算的距离与当前顶点值进行比较，并将两者中的最小值指定为该顶点的新值。
<p class="text-center">
    <img alt="GSA SSSP superstep 1" width="70%" src="{{ site.baseurl }}/fig/gelly-gsa-sssp1.png"/>
</p>

注意，如果一个顶点在超阶跃过程中没有改变它的值，它将不会在下一个超阶跃过程中计算候选距离。当没有顶点改变值时，算法收敛。
要在Gelly GSA中实现这个示例，用户只需要调用输入图上的“runGatherSumApplyIteration”方法，并提供“aggregfunction”、“SumFunction”和“ApplyFunction”udf。迭代同步、分组、值更新、收敛由系统处理:
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// 读取输入图
Graph<Long, Double, Double> graph = ...

// 定义最大迭代次数
int maxIterations = 10;

// 执行GSA迭代
Graph<Long, Double, Double> result = graph.runGatherSumApplyIteration(
				new CalculateDistances(), new ChooseMinDistance(), new UpdateDistance(), maxIterations);

// 结果提取顶点
DataSet<Vertex<Long, Double>> singleSourceShortestPaths = result.getVertices();


// - - -  UDFs - - - //

// Gather
private static final class CalculateDistances extends GatherFunction<Double, Double, Double> {

	public Double gather(Neighbor<Double, Double> neighbor) {
		return neighbor.getNeighborValue() + neighbor.getEdgeValue();
	}
}

// Sum
private static final class ChooseMinDistance extends SumFunction<Double, Double, Double> {

	public Double sum(Double newValue, Double currentValue) {
		return Math.min(newValue, currentValue);
	}
}

// Apply
private static final class UpdateDistance extends ApplyFunction<Long, Double, Double> {

	public void apply(Double newDistance, Double oldDistance) {
		if (newDistance < oldDistance) {
			setResult(newDistance);
		}
	}
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 读取输入图
val graph: Graph[Long, Double, Double] = ...

// 定义最大迭代次数
val maxIterations = 10

// 执行GSA迭代
val result = graph.runGatherSumApplyIteration(new CalculateDistances, new ChooseMinDistance, new UpdateDistance, maxIterations)

// 结果提取顶点
val singleSourceShortestPaths = result.getVertices


// - - -  UDFs - - - //

// Gather
final class CalculateDistances extends GatherFunction[Double, Double, Double] {

	override def gather(neighbor: Neighbor[Double, Double]): Double = {
		neighbor.getNeighborValue + neighbor.getEdgeValue
	}
}

// Sum
final class ChooseMinDistance extends SumFunction[Double, Double, Double] {

	override def sum(newValue: Double, currentValue: Double): Double = {
		Math.min(newValue, currentValue)
	}
}

// Apply
final class UpdateDistance extends ApplyFunction[Long, Double, Double] {

	override def apply(newDistance: Double, oldDistance: Double) = {
		if (newDistance < oldDistance) {
			setResult(newDistance)
		}
	}
}

{% endhighlight %}
</div>
</div>

注意，' gather '接受' neighbour '类型作为参数。这是一种方便的类型，它简单地将一个顶点与其相邻的边包装起来。
更多的列子实现{% gh_link /flink-libraries/flink-gelly/src/main/java/org/apache/flink/graph/library/GSAPageRank.java "GSAPageRank" %} and {% gh_link /flink-libraries/flink-gelly/src/main/java/org/apache/flink/graph/library/GSAConnectedComponents.java "GSAConnectedComponents" %} library methods of Gelly.

{% top %}

## 配置集合-和-应用迭代
可以使用“GSAConfiguration”对象配置GSA迭代。
目前可以指定以下参数:

* <strong>Name</strong>: 可以使用“GSAConfiguration”对象配置GSA迭代。GSA迭代的名称。名称显示在日志和消息中，可以使用' setName() '方法指定。
                         目前可以指定以下参数:

* <strong>Parallelism</strong>: 迭代的并行性。它可以使用' setParallelism() '方法进行设置。

* <strong>Solution set in unmanaged memory</strong>: 定义解决方案集是保存在托管内存中(Flink以序列化形式保存对象的内部方法)还是作为简单的对象映射。默认情况下，解决方案集在托管内存中运行。可以使用' setSolutionSetUnmanagedMemory() '方法设置此属性

* <strong>Aggregators</strong>: 迭代聚合器可以使用“registerAggregator()”方法注册。迭代聚合器对每个超步骤全局地组合所有聚合一次，并使它们在下一个超步骤中可用。注册的聚合器可以在用户定义的“aggregfunction”、“SumFunction”和“ApplyFunction”中访问。

* <strong>Broadcast Variables</strong>: DataSets can be added as [Broadcast Variables]({{site.baseurl}}/dev/index.html#broadcast-variables) to the `GatherFunction`, `SumFunction` and `ApplyFunction`, using the methods `addBroadcastSetForGatherFunction()`, `addBroadcastSetForSumFunction()` and `addBroadcastSetForApplyFunction` methods, respectively.

* <strong>Number of Vertices</strong>: 
* <strong>Neighbor Direction</strong>: 
下面的例子演示了顶点数选项的用法。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}

Graph<Long, Double, Double> graph = ...

// 配置迭代
GSAConfiguration parameters = new GSAConfiguration();

// set the number of vertices option to true
parameters.setOptNumVertices(true);

// run the gather-sum-apply iteration, also passing the configuration parameters
Graph<Long, Long, Long> result = graph.runGatherSumApplyIteration(
				new Gather(), new Sum(), new Apply(),
			    maxIterations, parameters);

// user-defined functions
public static final class Gather {
	...
	// get the number of vertices
	long numVertices = getNumberOfVertices();
	...
}

public static final class Sum {
	...
    // get the number of vertices
    long numVertices = getNumberOfVertices();
    ...
}

public static final class Apply {
	...
    // get the number of vertices
    long numVertices = getNumberOfVertices();
    ...
}

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}

val graph: Graph[Long, Double, Double] = ...

// 配置迭代
val parameters = new GSAConfiguration

// 设置顶点数选项为真
parameters.setOptNumVertices(true)

// 运行集合-和-应用迭代，同时传递配置参数
val result = graph.runGatherSumApplyIteration(new Gather, new Sum, new Apply, maxIterations, parameters)

// user-defined functions
final class Gather {
	...
	// get the number of vertices
	val numVertices = getNumberOfVertices
	...
}

final class Sum {
	...
    // get the number of vertices
    val numVertices = getNumberOfVertices
    ...
}

final class Apply {
	...
    // get the number of vertices
    val numVertices = getNumberOfVertices
    ...
}

{% endhighlight %}
</div>
</div>

下面的示例演示了edge direction选项的用法。<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}

Graph<Long, HashSet<Long>, Double> graph = ...

// 配置迭代
GSAConfiguration parameters = new GSAConfiguration();

//设置消息传递方向
parameters.setDirection(EdgeDirection.IN);

// 运行集合-和-应用迭代，同时传递配置参数
DataSet<Vertex<Long, HashSet<Long>>> result =
			graph.runGatherSumApplyIteration(
			new Gather(), new Sum(), new Apply(), maxIterations, parameters)
			.getVertices();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}

val graph: Graph[Long, HashSet[Long], Double] = ...

// 配置迭代
val parameters = new GSAConfiguration

// set the messaging direction
parameters.setDirection(EdgeDirection.IN)

// 运行集合-和-应用迭代，同时传递配置参数
val result = graph.runGatherSumApplyIteration(new Gather, new Sum, new Apply, maxIterations, parameters)
			.getVertices()
{% endhighlight %}
</div>
</div>
{% top %}

## 迭代比较抽象
尽管Gelly中的三个迭代抽象看起来非常相似，但是理解它们的不同之处可以提高程序的性能和可维护性。
在这三个模型中，以顶点为中心的模型是最通用的模型，它支持每个顶点的任意计算和消息传递。在散集模型中，生成消息的逻辑与更新顶点值的逻辑是解耦的。因此，使用散点收集编写的程序有时更易于跟踪和维护。
将消息传递阶段与顶点值更新逻辑分离不仅使某些程序更容易理解，而且还可能对性能产生积极影响。分散-聚集实现通常具有较低的内存需求，因为不需要对收件箱(接收到的消息)和发件箱(要发送的消息)数据结构进行并发访问。然而，这种特性也限制了表达性，使得一些计算模式不直观。当然，如果一个算法需要一个顶点并发地访问它的收件箱和发件箱，那么这个算法在散点-聚集中的表达式可能会有问题。强连通分量和近似最大值
权重匹配就是这样的图算法的例子。这种限制的直接后果是顶点不能在同一阶段生成消息和更新它们的状态。因此，决定是否基于其内容传播消息需要将其存储在顶点值中，以便收集阶段能够在接下来的迭代步骤中访问它。同样,如果顶点更新逻辑包括计算值的相邻的边缘,这些必须被包括在一个特殊的消息从分散到收集的阶段。这种变通方法通常会导致更高的内存需求和非优雅的、难以理解的算法实现。
收集-和-应用迭代也非常类似于分散-收集迭代。事实上，任何可以表示为GSA迭代的算法也可以在散集模型中编写。分散-聚集模型的消息传递阶段相当于GSA的收集和求和步骤:收集可以看作是产生消息的阶段，求和可以看作是将消息路由到目标顶点的阶段。类似地，值更新阶段对应于应用步骤。
这两种实现之间的主要区别是GSA的收集阶段在边缘上并行化计算，而消息传递阶段在顶点上分布计算。使用上面的SSSP示例，我们看到在散点-聚集情况的第一个超步骤中，顶点1、2和3并行地生成消息。顶点1产生3条消息，而顶点2和3分别产生一条消息。另一方面，在GSA情况下，计算是在边缘上并行进行的:顶点1的三个候选距离值是并行生成的。因此，如果集合步骤包含“繁重的”计算，那么使用GSA并分散计算可能是更好的方法，而不是加重单个顶点的负担。另一种情况是，当输入图是倾斜的(一些顶点比其他顶点有更多的邻居)时，边缘并行化可能会被证明是更有效的。
这两种实现之间的另一个区别是分散-聚集实现在内部使用“coGroup”操作符，而GSA使用“reduce”。因此，如果组合邻居值(消息)的函数需要整个值组进行计算，则应该使用散点-聚集。如果更新函数是结合式和交换式的，那么GSA的约简器将提供更有效的实现，因为它可以使用组合器。
需要注意的另一件事是，GSA严格地在邻域上工作，而在以顶点为中心和分散-聚集模型中，一个顶点可以向任何一个顶点发送消息，前提是它知道自己的顶点ID，而不管它是否是一个邻居。最后，在Gelly的分散-聚集实现中，可以选择消息传递方向，即更新传播的方向。GSA还不支持这一点，因此每个顶点将只根据其邻居的值进行更新。
Gelly迭代模型之间的主要区别如下表所示。

<table class="table table-bordered">
  <thead>
    <tr>
      <th class="text-left" style="width: 25%">Iteration Model</th>
      <th class="text-center">Update Function</th>
      <th class="text-center">Update Logic</th>
      <th class="text-center">Communication Scope</th>
      <th class="text-center">Communication Logic</th>
    </tr>
  </thead>
  <tbody>
 <tr>
  <td>Vertex-Centric</td>
  <td>arbitrary</td>
  <td>arbitrary</td>
  <td>any vertex</td>
  <td>arbitrary</td>
</tr>
<tr>
  <td>Scatter-Gather</td>
  <td>arbitrary</td>
  <td>based on received messages</td>
  <td>any vertex</td>
  <td>based on vertex state</td>
</tr>
<tr>
  <td>Gather-Sum-Apply</td>
  <td>associative and commutative</td>
  <td>based on neighbors' values</td>
  <td>neighborhood</td>
  <td>based on vertex state</td>
</tr>
</tbody>
</table>

{% top %}
