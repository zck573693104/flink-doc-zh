---
标题: Graph API
nav-parent_id: graphs
nav-pos: 1
---
<!--
获得Apache软件基金会（ASF）的许可或更多贡献者许可协议。 请参阅此工作分发的NOTICE文件，
以获取更多关于版权所有权的信息。 ASF许可此文件是根据Apache的许可证2.0版（
“执照”）; 除非符合规定，否则您不得使用此文件与许可证。 您可以在以下位置获取许可证副本
http://www.apache.org/licenses/LICENSE-2.0
除非适用法律要求或书面同意，否则根据许可证分发的软件分发是在“原样”的基础上，没有任何
担保或条件，无论是明示的还是隐含的。 请参阅许可证上的特定语言管理的权限与限制。
-->

* This will be replaced by the TOC
{:toc}

Graph Representation
-----------
//图的节点由`Vertex`类型表示，“Vertex”类型由唯一ID和值定义。`Vertex`类型的ID应该实现`Comparable`接口。可以通过将值类型设置为“NullValue”来表示没有值的顶点。


<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// 创建一个具有long ID和字符串值的新顶点
Vertex<Long, String> v = new Vertex<Long, String>(1L, "foo");

// 创建一个具有长ID且无值的新顶点
Vertex<Long, NullValue> v = new Vertex<Long, NullValue>(1L, NullValue.getInstance());
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 创建一个具有长ID和字符串值的新顶点
val v = new Vertex(1L, "foo")

// 创建一个具有长ID且无值的新顶点
val v = new Vertex(1L, NullValue.getInstance())
{% endhighlight %}
</div>
</div>

图形的边由“边”类型表示。“边”由源ID(源“顶点”的ID)、目标ID(目标“顶点”的ID)和一个可选值定义。源id和目标id的类型应该与“顶点”id相同。没有值的边具有' NullValue '值类型。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Edge<Long, Double> e = new Edge<Long, Double>(1L, 2L, 0.5);

// 反转这条边的源和目标
Edge<Long, Double> reversed = e.reverse();

Double weight = e.getValue(); // weight = 0.5
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val e = new Edge(1L, 2L, 0.5)

// 反转这条边的源和目标
val reversed = e.reverse

val weight = e.getValue // weight = 0.5
{% endhighlight %}
</div>
</div>

在Gelly中，“边”总是从源顶点指向目标顶点。一个“图”可以是无向的
每条“边”都包含从目标顶点到源顶点的匹配“边”。

{% top %}

Graph Creation
-----------

你可以用以下方法创建一个“图表”:

* 从边的“数据集”和可选的顶点的“数据集”:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<Vertex<String, Long>> vertices = ...

DataSet<Edge<String, Double>> edges = ...

Graph<String, Long, Double> graph = Graph.fromDataSet(vertices, edges, env);
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

val vertices: DataSet[Vertex[String, Long]] = ...

val edges: DataSet[Edge[String, Double]] = ...

val graph = Graph.fromDataSet(vertices, edges, env)
{% endhighlight %}
</div>
</div>

* 从' Tuple2 '的'数据集'表示边缘。Gelly将每个“Tuple2”转换为“Edge”，其中第一个字段是源ID，第二个字段是目标ID。顶点和边缘值都将设置为“NullValue”。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<Tuple2<String, String>> edges = ...

Graph<String, NullValue, NullValue> graph = Graph.fromTuple2DataSet(edges, env);
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

val edges: DataSet[(String, String)] = ...

val graph = Graph.fromTuple2DataSet(edges, env)
{% endhighlight %}
</div>
</div>

* 从“Tuple3”的“数据集”和“Tuple2”的可选“数据集”。在本例中，Gelly将每个“Tuple3”转换为“Edge”，其中第一个字段是源ID，第二个字段是目标ID，第三个字段是边缘值。等价地，每个“Tuple2”将被转换为一个“顶点”，其中第一个字段将是顶点ID，第二个字段将是顶点值:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<Tuple2<String, Long>> vertexTuples = env.readCsvFile("path/to/vertex/input").types(String.class, Long.class);

DataSet<Tuple3<String, String, Double>> edgeTuples = env.readCsvFile("path/to/edge/input").types(String.class, String.class, Double.class);

Graph<String, Long, Double> graph = Graph.fromTupleDataSet(vertexTuples, edgeTuples, env);
{% endhighlight %}

* 从边缘数据的CSV文件和顶点数据的可选CSV文件。在本例中，Gelly将把Edge CSV文件中的每一行转换为“Edge”，其中第一个字段将是源ID，第二个字段将是目标ID，第三个字段(如果存在)将是边缘值。等价地，可选顶点CSV文件中的每一行都将转换为“顶点”，其中第一个字段将是顶点ID，第二个字段(如果存在)将是顶点值。为了从“GraphCsvReader”获取“图形”，必须指定类型，使用以下方法之一:

- `types(Class<K> vertexKey, Class<VV> vertexValue,Class<EV> edgeValue)`: both vertex and edge values are present.
- `edgeTypes(Class<K> vertexKey, Class<EV> edgeValue)`: the Graph has edge values, but no vertex values.
- `vertexTypes(Class<K> vertexKey, Class<VV> vertexValue)`: the Graph has vertex values, but no edge values.
- `keyType(Class<K> vertexKey)`: the Graph has no vertex values and no edge values.

{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//创建一个具有字符串顶点id、长顶点值和双边缘值的图形
Graph<String, Long, Double> graph = Graph.fromCsvReader("path/to/vertex/input", "path/to/edge/input", env)
					.types(String.class, Long.class, Double.class);


// 创建一个既没有顶点也没有边值的图形
Graph<Long, NullValue, NullValue> simpleGraph = Graph.fromCsvReader("path/to/edge/input", env).keyType(Long.class);
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

val vertexTuples = env.readCsvFile[String, Long]("path/to/vertex/input")

val edgeTuples = env.readCsvFile[String, String, Double]("path/to/edge/input")

val graph = Graph.fromTupleDataSet(vertexTuples, edgeTuples, env)
{% endhighlight %}

* 从边缘数据的CSV文件和顶点数据的可选CSV文件。
  在本例中，Gelly将把每一行从Edge CSV文件转换为“Edge”。
  每行的第一个字段将是源ID，第二个字段将是目标ID，第三个字段(如果存在)将是边缘值。
  如果边缘没有关联的值，将边缘值类型参数(第三种类型参数)设置为“NullValue”。
  您还可以指定使用顶点值初始化顶点。
  如果您通过“pathVertices”提供CSV文件的路径，则该文件的每一行都将转换为“Vertex”。
  每行的第一个字段是顶点ID，第二个字段是顶点值。
  如果您通过“vertexValueInitializer”参数提供顶点值初始化器“MapFunction”，那么这个函数将用于生成顶点值。
  顶点集将从边输入中自动创建。
  如果顶点没有关联的值，将顶点值类型参数(第2类型参数)设置为“NullValue”。
  然后，顶点将从顶点值类型为“NullValue”的边输入中自动创建。

{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

// 创建一个具有字符串顶点id、长顶点值和双边缘值的图形
val graph = Graph.fromCsvReader[String, Long, Double](
		pathVertices = "path/to/vertex/input",
		pathEdges = "path/to/edge/input",
		env = env)


// 创建一个既没有顶点也没有边值的图形
val simpleGraph = Graph.fromCsvReader[Long, NullValue, NullValue](
		pathEdges = "path/to/edge/input",
		env = env)

// 创建一个由顶点值初始化器生成且没有边值的双顶点值的图
val simpleGraph = Graph.fromCsvReader[Long, Double, NullValue](
        pathEdges = "path/to/edge/input",
        vertexValueInitializer = new MapFunction[Long, Double]() {
            def map(id: Long): Double = {
                id.toDouble
            }
        },
        env = env)
{% endhighlight %}
</div>
</div>


* 从边的“集合”和可选的顶点的“集合”:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

List<Vertex<Long, Long>> vertexList = new ArrayList...

List<Edge<Long, String>> edgeList = new ArrayList...

Graph<Long, Long, String> graph = Graph.fromCollection(vertexList, edgeList, env);
{% endhighlight %}

如果在图形创建过程中没有提供顶点输入，Gelly将自动从边缘输入生成“顶点”“数据集”。在这种情况下，创建的顶点将没有值。或者，您可以提供一个“MapFunction”作为创建方法的参数，以便初始化“顶点”值:
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// 初始化顶点值使其等于顶点ID
Graph<Long, Long, String> graph = Graph.fromCollection(edgeList,
				new MapFunction<Long, Long>() {
					public Long map(Long value) {
						return value;
					}
				}, env);
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

val vertexList = List(...)

val edgeList = List(...)

val graph = Graph.fromCollection(vertexList, edgeList, env)
{% endhighlight %}

如果在图形创建过程中没有提供顶点输入，Gelly将自动从边缘输入生成“顶点”“数据集”。在这种情况下，创建的顶点将没有值。或者，您可以提供一个“MapFunction”作为创建方法的参数，以便初始化“顶点”值:

{% highlight java %}
val env = ExecutionEnvironment.getExecutionEnvironment

// 初始化顶点值使其等于顶点ID
val graph = Graph.fromCollection(edgeList,
    new MapFunction[Long, Long] {
       def map(id: Long): Long = id
    }, env)
{% endhighlight %}
</div>
</div>

{% top %}

图的属性
------------

Gelly包括以下用于检索各种图形属性和指标的方法:
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// Gelly包括以下用于检索各种图形属性和度量的方法:获取顶点数据集
DataSet<Vertex<K, VV>> getVertices()

// Gelly包括以下用于检索各种图形属性和度量的方法:获取顶点数据集获取顶点数据集获取边缘数据集
DataSet<Edge<K, EV>> getEdges()

// 以数据集的形式获取顶点的id
DataSet<K> getVertexIds()

// 将边缘id的源-目标对作为数据集获取
DataSet<Tuple2<K, K>> getEdgeIds()

//获取一个<顶点ID，所有顶点的in-degree>对的数据集
DataSet<Tuple2<K, LongValue>> inDegrees()

// 获取一个数据集<顶点ID，所有顶点的内度>对;获取一个数据集<顶点ID，所有顶点的外度>对
DataSet<Tuple2<K, LongValue>> outDegrees()

// 获取一个<顶点ID，所有顶点的度>对的数据集，其中度是进出度的和
DataSet<Tuple2<K, LongValue>> getDegrees()

// 得到顶点的个数
long numberOfVertices()

// 得到边的数量
long numberOfEdges()

//获取一个三元组<srcVertex, trgVertex, edge>的数据集
DataSet<Triplet<K, VV, EV>> getTriplets()

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 获取顶点数据集
getVertices: DataSet[Vertex[K, VV]]

//获取边缘数据集
getEdges: DataSet[Edge[K, EV]]

//以数据集的形式获取顶点的id
getVertexIds: DataSet[K]

// 将边缘id的源-目标对作为数据集获取
getEdgeIds: DataSet[(K, K)]

// 获取一个<顶点ID，所有顶点的in-degree>对的数据集
inDegrees: DataSet[(K, LongValue)]

// 获取一个数据集<顶点ID，所有顶点的输出度>对
outDegrees: DataSet[(K, LongValue)]

//获取一个<顶点ID，所有顶点的度>对的数据集，其中度是进出度的和
getDegrees: DataSet[(K, LongValue)]

// 得到顶点的个数
numberOfVertices: Long

// 得到边的数量
numberOfEdges: Long

// 得到属于 Triplets<srcVertex, trgVertex, edge>
getTriplets: DataSet[Triplet[K, VV, EV]]

{% endhighlight %}
</div>
</div>

{% top %}

图的转换
-----------------

* <strong>MAP</strong>: Gelly提供了在顶点值或边缘值上应用映射转换的专门方法。“mapVertices”和“mapEdges”返回一个新的“Graph”，其中顶点(或边)的id保持不变，而值则根据提供的用户定义的map函数进行转换。映射函数还允许更改顶点或边缘值的类型。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
Graph<Long, Long, Long> graph = Graph.fromDataSet(vertices, edges, env);

// 每个顶点值增加1
Graph<Long, Long, Long> updatedGraph = graph.mapVertices(
				new MapFunction<Vertex<Long, Long>, Long>() {
					public Long map(Vertex<Long, Long> value) {
						return value.getValue() + 1;
					}
				});
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment
val graph = Graph.fromDataSet(vertices, edges, env)

//每个顶点值增加1
val updatedGraph = graph.mapVertices(v => v.getValue + 1)
{% endhighlight %}
</div>
</div>

* <strong>转化</strong>: Gelly提供了翻译顶点和边缘id (' translateGraphIDs ')、顶点值(' translateVertexValues ')或边缘值(' translateedgevalvalues ')的值和/或类型的专门方法。翻译由用户定义的map函数执行，其中几个函数在“org.apache.flink.graph.asm”中提供。翻译的包。这三种翻译方法都可以使用相同的“MapFunction”。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
Graph<Long, Long, Long> graph = Graph.fromDataSet(vertices, edges, env);

// 将每个顶点和边缘ID遍历到一个字符串中
Graph<String, Long, Long> updatedGraph = graph.translateGraphIds(
				new MapFunction<Long, String>() {
					public String map(Long id) {
						return id.toString();
					}
				});

// 将顶点id、边缘id、顶点值和边缘值转换为LongValue
Graph<LongValue, LongValue, LongValue> updatedGraph = graph
                .translateGraphIds(new LongToLongValue())
                .translateVertexValues(new LongToLongValue())
                .translateEdgeValues(new LongToLongValue())
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment
val graph = Graph.fromDataSet(vertices, edges, env)

// 将每个顶点和边缘ID转换为字符串
val updatedGraph = graph.translateGraphIds(id => id.toString)
{% endhighlight %}
</div>
</div>


* <strong>Filter</strong>: 过滤器转换在“图”的顶点或边缘上应用用户定义的过滤器函数。' filteronedge '将创建原始图的子图，只保留满足所提供谓词的边。注意顶点数据集不会被修改。' filterOnVertices '分别对图中的顶点应用筛选器。源和/或目标不满足顶点谓词的边将从结果边缘数据集中删除。“子图”方法可用于同时对顶点和边缘应用一个过滤函数。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, Long, Long> graph = ...

graph.subgraph(
		new FilterFunction<Vertex<Long, Long>>() {
			   	public boolean filter(Vertex<Long, Long> vertex) {
					// keep only vertices with positive values
					return (vertex.getValue() > 0);
			   }
		   },
		new FilterFunction<Edge<Long, Long>>() {
				public boolean filter(Edge<Long, Long> edge) {
					// keep only edges with negative values
					return (edge.getValue() < 0);
				}
		})
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val graph: Graph[Long, Long, Long] = ...

// 只保留具有正值的顶点
// 只有带负值的边
graph.subgraph((vertex => vertex.getValue > 0), (edge => edge.getValue < 0))
{% endhighlight %}
</div>
</div>

<p class="text-center">
    <img alt="Filter Transformations" width="80%" src="{{ site.baseurl }}/fig/gelly-filter.png"/>
</p>

* <strong>Join</strong>: Gelly提供了将顶点和边缘数据集与其他输入数据集连接起来的专门方法。' joinWithVertices '使用' Tuple2 '输入数据集连接顶点。该连接使用顶点ID和' Tuple2 '输入的第一个字段作为连接键执行。该方法返回一个新的“图”，其中顶点值已根据提供的用户定义的转换函数进行更新。
                         类似地，可以使用三种方法中的一种将输入数据集与边缘连接起来。“joinWithEdges”要求输入“Tuple3”的“数据集”，并在源顶点id和目标顶点id的组合键上进行连接。' joinWithEdgesOnSource '希望得到' Tuple2 '的'数据集'，并连接到边的源键和输入数据集的第一个属性上;' joinWithEdgesOnTarget '希望得到' Tuple2 '的'数据集'，并连接到边的目标键和输入数据集的第一个属性上。这三种方法都在边缘上应用一个转换函数和输入数据集的值。
                         注意，如果输入数据集多次包含键，那么所有Gelly连接方法将只考虑遇到的第一个值。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, Double, Double> network = ...

DataSet<Tuple2<Long, LongValue>> vertexOutDegrees = network.outDegrees();

// 将转换概率赋值为边缘权值
Graph<Long, Double, Double> networkWithWeights = network.joinWithEdgesOnSource(vertexOutDegrees,
				new VertexJoinFunction<Double, LongValue>() {
					public Double vertexJoin(Double vertexValue, LongValue inputValue) {
						return vertexValue / inputValue.getValue();
					}
				});
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val network: Graph[Long, Double, Double] = ...

val vertexOutDegrees: DataSet[(Long, LongValue)] = network.outDegrees

// 将转换概率赋值为边缘权值
val networkWithWeights = network.joinWithEdgesOnSource(vertexOutDegrees, (v1: Double, v2: LongValue) => v1 / v2.getValue)
{% endhighlight %}
</div>
</div>

* <strong>Reverse</strong>: “reverse()”方法返回一个新的“图”，其中所有边的方向都已反转。

* <strong>Undirected</strong>: “reverse()”方法返回一个新的“图”，其中所有边的方向都已反转。在Gelly中，“图”总是有向的。无向图可以通过向图中添加所有相反方向的边来表示。为此，Gelly提供了“getundirect()”方法。

* <strong>Union</strong>: “reverse()”方法返回一个新的“图”，其中所有边的方向都已反转。在Gelly中，“图”总是有向的。无向图可以通过向图中添加所有相反方向的边来表示。为此，Gelly提供了“getundirect()”方法。Gelly的' union() '方法对指定图和当前图的顶点和边缘集执行联合操作。重复的顶点从产生的“图”中删除，而如果重复的边存在，这些将被保留。

<p class="text-center">
    <img alt="Union Transformation" width="50%" src="{{ site.baseurl }}/fig/gelly-union.png"/>
</p>

* <strong>Difference</strong>: Gelly的' difference() '方法在当前图和指定图的顶点和边缘集上执行一个差值。

* <strong>Intersect</strong>: Gelly的' intersect() '方法在边缘执行一个交集
                              当前图和指定图的集合。结果是一个包含所有内容的新“图”
                              存在于两个输入图中的边。两条边被认为是相等的，如果它们有相同的源
                              标识符、目标标识符和边缘值。结果图中的顶点没有
                              价值。例如，如果需要顶点值，可以使用以下方法从输入图中检索它们
                              “joinWithVertices()的方法。
                              根据参数“distinct”的不同，结果中会包含一次相等的边
                              "图形"或者输入图形中有一对相等的边。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// create first graph from edges {(1, 3, 12) (1, 3, 13), (1, 3, 13)}
List<Edge<Long, Long>> edges1 = ...
Graph<Long, NullValue, Long> graph1 = Graph.fromCollection(edges1, env);

// create second graph from edges {(1, 3, 13)}
List<Edge<Long, Long>> edges2 = ...
Graph<Long, NullValue, Long> graph2 = Graph.fromCollection(edges2, env);

// Using distinct = true results in {(1,3,13)}
Graph<Long, NullValue, Long> intersect1 = graph1.intersect(graph2, true);

// Using distinct = false results in {(1,3,13),(1,3,13)} as there is one edge pair
Graph<Long, NullValue, Long> intersect2 = graph1.intersect(graph2, false);

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

// create first graph from edges {(1, 3, 12) (1, 3, 13), (1, 3, 13)}
val edges1: List[Edge[Long, Long]] = ...
val graph1 = Graph.fromCollection(edges1, env)

// create second graph from edges {(1, 3, 13)}
val edges2: List[Edge[Long, Long]] = ...
val graph2 = Graph.fromCollection(edges2, env)


// Using distinct = true results in {(1,3,13)}
val intersect1 = graph1.intersect(graph2, true)

// Using distinct = false results in {(1,3,13),(1,3,13)} as there is one edge pair
val intersect2 = graph1.intersect(graph2, false)
{% endhighlight %}
</div>
</div>

-{% top %}

图的变化
-----------

GraGelly包括以下方法，用于从输入“图”中添加和删除顶点和边缘:ph突变

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
// 向图中添加一个顶点。如果顶点已经存在，就不会再添加它。
Graph<K, VV, EV> addVertex(final Vertex<K, VV> vertex)

// 向图中添加顶点列表。如果图中的顶点已经存在，则不再添加它们。
Graph<K, VV, EV> addVertices(List<Vertex<K, VV>> verticesToAdd)

// 向图形添加一条边。如果源顶点和目标顶点在图中不存在，它们也将被添加。
Graph<K, VV, EV> addEdge(Vertex<K, VV> source, Vertex<K, VV> target, EV edgeValue)

// 将边列表添加到图中。当为不存在的顶点集添加边时，该边被认为无效且被忽略。
Graph<K, VV, EV> addEdges(List<Edge<K, EV>> newEdges)

// 从图中删除给定的顶点及其边缘。
Graph<K, VV, EV> removeVertex(Vertex<K, VV> vertex)

// 从图中删除给定的顶点及其边缘。
Graph<K, VV, EV> removeVertices(List<Vertex<K, VV>> verticesToBeRemoved)

// 从图中删除与给定边匹配的所有边。
Graph<K, VV, EV> removeEdge(Edge<K, EV> edge)

// 从图中删除与给定边匹配的所有边。
Graph<K, VV, EV> removeEdges(List<Edge<K, EV>> edgesToBeRemoved)
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// 向图中添加一个顶点。如果顶点已经存在，就不会再添加它。
addVertex(vertex: Vertex[K, VV])

// 向图中添加一个顶点。如果顶点已经存在，就不会再添加它。向图中添加一个顶点。如果顶点已经存在，就不会再添加它。
addVertices(verticesToAdd: List[Vertex[K, VV]])

// 向图形添加一条边。如果源顶点和目标顶点在图中不存在，它们也将被添加。
addEdge(source: Vertex[K, VV], target: Vertex[K, VV], edgeValue: EV)

// 向图形添加一条边。如果源顶点和目标顶点在图中不存在，它们也将被添加。
addEdges(edges: List[Edge[K, EV]])

// 从图中删除给定的顶点及其边缘。
removeVertex(vertex: Vertex[K, VV])

// 从图中删除给定的顶点及其边缘。
removeVertices(verticesToBeRemoved: List[Vertex[K, VV]])

// 从图中删除所有定的顶点及其边缘。
removeEdge(edge: Edge[K, EV])

// 从图中删除所有定的顶点及其边缘。
removeEdges(edgesToBeRemoved: List[Edge[K, EV]])
{% endhighlight %}
</div>
</div>

Neighborhood Methods
-----------
邻域方法允许顶点在其第一跳邻域上执行聚合。
' reduceonedge() '可用于计算一个顶点相邻边的值的聚合，' reduceOnNeighbors() '可用于计算一个顶点相邻边的值的聚合。这些方法假设关联和交换聚合，并在内部利用组合器，显著提高了性能。
邻域范围由“EdgeDirection”参数定义，该参数接受“IN”、“OUT”或“ALL”值。' IN '将收集一个顶点的所有进来的边(邻边)，' OUT '将收集所有出去的边(邻边)，而' all '将收集所有的边(邻边)。
例如，假设您希望在下面的图中为每个顶点选择所有外缘的最小权值:

<p class="text-center">
    <img alt="reduceOnEdges Example" width="50%" src="{{ site.baseurl }}/fig/gelly-example-graph.png"/>
</p>

下面的代码将收集每个顶点的外边缘，并将“SelectMinWeight()”用户定义的函数应用于每个结果邻域:
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, Long, Double> graph = ...

DataSet<Tuple2<Long, Double>> minWeights = graph.reduceOnEdges(new SelectMinWeight(), EdgeDirection.OUT);

//用户定义的函数来选择最小权值
static final class SelectMinWeight implements ReduceEdgesFunction<Double> {

		@Override
		public Double reduceEdges(Double firstEdgeValue, Double secondEdgeValue) {
			return Math.min(firstEdgeValue, secondEdgeValue);
		}
}
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val graph: Graph[Long, Long, Double] = ...

val minWeights = graph.reduceOnEdges(new SelectMinWeight, EdgeDirection.OUT)

// 用户定义的函数来选择最小权值
final class SelectMinWeight extends ReduceEdgesFunction[Double] {
	override def reduceEdges(firstEdgeValue: Double, secondEdgeValue: Double): Double = {
		Math.min(firstEdgeValue, secondEdgeValue)
	}
 }
{% endhighlight %}
</div>
</div>

<p class="text-center">
    <img alt="reduceOnEdges Example" width="50%" src="{{ site.baseurl }}/fig/gelly-reduceOnEdges.png"/>
</p>

类似地，假设你想计算每个顶点的所有相邻点的和。下面的代码将收集每个顶点的传入邻域，并在每个邻域上应用用户定义的“SumValues()”函数:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, Long, Double> graph = ...

DataSet<Tuple2<Long, Long>> verticesWithSum = graph.reduceOnNeighbors(new SumValues(), EdgeDirection.IN);

// 用户定义的函数，用于对邻居值求和
static final class SumValues implements ReduceNeighborsFunction<Long> {

	    	@Override
	    	public Long reduceNeighbors(Long firstNeighbor, Long secondNeighbor) {
		    	return firstNeighbor + secondNeighbor;
	  	}
}
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val graph: Graph[Long, Long, Double] = ...

val verticesWithSum = graph.reduceOnNeighbors(new SumValues, EdgeDirection.IN)

// user-defined function to sum the neighbor values
final class SumValues extends ReduceNeighborsFunction[Long] {
   	override def reduceNeighbors(firstNeighbor: Long, secondNeighbor: Long): Long = {
    	firstNeighbor + secondNeighbor
    }
}
{% endhighlight %}
</div>
</div>

<p class="text-center">
    <img alt="reduceOnNeighbors Example" width="70%" src="{{ site.baseurl }}/fig/gelly-reduceOnNeighbors.png"/>
</p>

当聚集函数不是结合式和可交换的，或者希望每个顶点返回多个值时，可以使用更一般的方法
' groupreduceonedge() '和' groupReduceOnNeighbors() '方法。
这些方法返回每个顶点的0个、1个或多个值，并提供对整个邻域的访问。
例如，下面的代码将输出所有与权重为0.5或以上的边相连的顶点对:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
Graph<Long, Long, Double> graph = ...

DataSet<Tuple2<Vertex<Long, Long>, Vertex<Long, Long>>> vertexPairs = graph.groupReduceOnNeighbors(new SelectLargeWeightNeighbors(), EdgeDirection.OUT);

// user-defined function to select the neighbors which have edges with weight > 0.5
static final class SelectLargeWeightNeighbors implements NeighborsFunctionWithVertexValue<Long, Long, Double,
		Tuple2<Vertex<Long, Long>, Vertex<Long, Long>>> {

		@Override
		public void iterateNeighbors(Vertex<Long, Long> vertex,
				Iterable<Tuple2<Edge<Long, Double>, Vertex<Long, Long>>> neighbors,
				Collector<Tuple2<Vertex<Long, Long>, Vertex<Long, Long>>> out) {

			for (Tuple2<Edge<Long, Double>, Vertex<Long, Long>> neighbor : neighbors) {
				if (neighbor.f0.f2 > 0.5) {
					out.collect(new Tuple2<Vertex<Long, Long>, Vertex<Long, Long>>(vertex, neighbor.f1));
				}
			}
		}
}
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val graph: Graph[Long, Long, Double] = ...

val vertexPairs = graph.groupReduceOnNeighbors(new SelectLargeWeightNeighbors, EdgeDirection.OUT)

// 用户定义的函数，用于选择具有权值为> 0.5的边的邻居
final class SelectLargeWeightNeighbors extends NeighborsFunctionWithVertexValue[Long, Long, Double,
  (Vertex[Long, Long], Vertex[Long, Long])] {

	override def iterateNeighbors(vertex: Vertex[Long, Long],
		neighbors: Iterable[(Edge[Long, Double], Vertex[Long, Long])],
		out: Collector[(Vertex[Long, Long], Vertex[Long, Long])]) = {

			for (neighbor <- neighbors) {
				if (neighbor._1.getValue() > 0.5) {
					out.collect(vertex, neighbor._2)
				}
			}
		}
   }
{% endhighlight %}
</div>
</div>

当聚合计算不需要访问顶点值(执行聚合的顶点值)时，建议对用户定义的函数使用更有效的“EdgesFunction”和“NeighborsFunction”。当需要访问顶点值时，应该使用“EdgesFunctionWithVertexValue”和“NeighborsFunctionWithVertexValue”来代替。
{% top %}

图表验证
-----------

Gelly提供了一个用于对输入图执行验证检查的简单实用程序。根据特定的条件，图可能有效，也可能无效，这取决于应用程序上下文。例如，用户可能需要验证他们的图是否包含重复的边，或者它的结构是否是二部分的。为了验证一个图，可以定义一个定制的“GraphValidator”并实现它的“validate()”方法。InvalidVertexIdsValidator是Gelly预定义的验证器。它检查边缘集是否包含有效的顶点id，即所有的边缘id
也存在于顶点id集中。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// 创建一个id ={1, 2, 3, 4, 5}的顶点列表
List<Vertex<Long, Long>> vertices = ...

// 创建一个id ={(1,2)(1,3)，(2,4)，(5,6)}的边列表
List<Edge<Long, Long>> edges = ...

Graph<Long, Long, Long> graph = Graph.fromCollection(vertices, edges, env);

// 返回false: 6是无效的ID
graph.validate(new InvalidVertexIdsValidator<Long, Long, Long>());

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val env = ExecutionEnvironment.getExecutionEnvironment

//创建一个id ={1, 2, 3, 4, 5}的顶点列表
val vertices: List[Vertex[Long, Long]] = ...

// 创建一个id ={(1,2)(1,3)，(2,4)，(5,6)}的边列表
val edges: List[Edge[Long, Long]] = ...

val graph = Graph.fromCollection(vertices, edges, env)

// 创建一个ID ={(1,2)(1,3)，(2,4)，(5,6)}的边列表将返回false: 6是一个无效的ID
graph.validate(new InvalidVertexIdsValidator[Long, Long, Long])

{% endhighlight %}
</div>
</div>

{% top %}
