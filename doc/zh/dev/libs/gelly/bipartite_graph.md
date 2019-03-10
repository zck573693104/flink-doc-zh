---
标题:两偶图
nav-parent_id:图
nav-pos: 6
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

<span class="label label-danger">Attention</span> Bipartite Graph currently only supported in Gelly Java API.

* This will be replaced by the TOC
{:toc}

两偶图
---------------
二部图(也称为双模图)是顶点被分成两个不相交集的图的一种类型。这些集合通常称为顶部和底部顶点。此图中的边只能连接来自相反集合的顶点(即底部顶点到顶部顶点)，而不能连接同一集合中的两个顶点。
这些图在实践中有广泛的应用，对于特定的领域可以是更自然的选择。例如，要表示科学论文的作者，顶部的顶点可以表示科学论文，而底部的节点将表示作者。自然，顶部和底部节点之间的边缘表示特定科学论文的作者。应用二部图的另一个常见例子是演员和电影之间的关系。在本例中，边表示在电影中扮演的特定演员。
使用二部图代替常规图(单模式)的实际原因如下(http://www.complexnetworks.fr/wp-content/uploads/2011/01/socnet07.pdf):
*它们保存了更多关于顶点之间连接的信息。例如，在表示他们共同撰写了一篇论文的图中，两个研究人员之间没有一个单一的链接，而是一个二部图保存了他们所撰写论文的信息
二部图可以比单模图更紧凑地编码相同的信息
图表示
--------------------

“双分形图”表示为:
*顶部节点的“数据集”
*底部节点的“数据集”
*顶部和底部节点之间的边缘的“数据集”
与在“Graph”中一样，类节点由“Vertex”类型表示，其类型和值也适用相同的规则。
图的边缘由“BipartiteEdge”类型表示。“BipartiteEdge”由顶部ID(顶部“顶点”的ID)、底部ID(底部“顶点”的ID)和一个可选值定义。“Edge”和“BipartiteEdge”之间的主要区别在于，它链接的节点的id可以是不同类型的。没有值的边具有“NullValue”值类型。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
BipartiteEdge<Long, String, Double> e = new BipartiteEdge<Long, String, Double>(1L, "id1", 0.5);

Double weight = e.getValue(); // weight = 0.5
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// Scala API is not yet supported
{% endhighlight %}
</div>
</div>
{% top %}


图的构建
--------------

你可以用以下方法创建一个“双分图”:
*来自顶部顶点的“数据集”、底部顶点的“数据集”和边缘的“数据集”:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<Vertex<String, Long>> topVertices = ...

DataSet<Vertex<String, Long>> bottomVertices = ...

DataSet<Edge<String, String, Double>> edges = ...

Graph<String, String, Long, Long, Double> graph = BipartiteGraph.fromDataSet(topVertices, bottomVertices, edges, env);
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// Scala API is not yet supported
{% endhighlight %}
</div>
</div>


图的转换
---------------------


* <strong>Projection</strong>: Projection is a common operation for bipartite graphs that converts a bipartite graph into a regular graph. There are two types of projections: top and bottom projections. Top projection preserves only top nodes in the result graph and creates a link between them in a new graph only if there is an intermediate bottom node both top nodes connect to in the original graph. Bottom projection is the opposite to top projection, i.e. only preserves bottom nodes and connects a pair of nodes if they are connected in the original graph.

<p class="text-center">
    <img alt="Bipartite Graph Projections" width="80%" src="{{ site.baseurl }}/fig/bipartite_graph_projections.png"/>
</p>

Gelly支持两种类型的投影:简单投影和完整投影。它们之间唯一的区别是什么数据与结果图中的边相关联。
在简单投影的情况下，结果图中的每个节点都包含一对连接原始图中节点的二部边缘值:
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
// Vertices (1, "top1")
DataSet<Vertex<Long, String>> topVertices = ...

// Vertices (2, "bottom2"); (4, "bottom4")
DataSet<Vertex<Long, String>> bottomVertices = ...

// Edge that connect vertex 2 to vertex 1 and vertex 4 to vertex 1:
// (1, 2, "1-2-edge"); (1, 4, "1-4-edge")
DataSet<Edge<Long, Long, String>> edges = ...

BipartiteGraph<Long, Long, String, String, String> graph = BipartiteGraph.fromDataSet(topVertices, bottomVertices, edges, env);

// Result graph with two vertices:
// (2, "bottom2"); (4, "bottom4")
//
//和一条包含底边id的边和一个带
//原始二部图中中间边的值:
// (2, 4, ("1-2-edge", "1-4-edge"))
Graph<Long, String, Tuple2<String, String>> graph bipartiteGraph.projectionBottomSimple();

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// Scala API is not yet supported
{% endhighlight %}
</div>
</div>

完全投影保存了关于两个顶点之间连接的所有信息，并将其存储在“投影”实例中。这包括中间顶点的值和id，源和目标顶点值，源和目标边值:
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
// Vertices (1, "top1")
DataSet<Vertex<Long, String>> topVertices = ...

// Vertices (2, "bottom2"); (4, "bottom4")
DataSet<Vertex<Long, String>> bottomVertices = ...

// Edge that connect vertex 2 to vertex 1 and vertex 4 to vertex 1:
// (1, 2, "1-2-edge"); (1, 4, "1-4-edge")
DataSet<Edge<Long, Long, String>> edges = ...

BipartiteGraph<Long, Long, String, String, String> graph = BipartiteGraph.fromDataSet(topVertices, bottomVertices, edges, env);

// Result graph with two vertices:
// (2, "bottom2"); (4, "bottom4")
// and one edge that contains ids of bottom edges and a tuple that 
// contains id and value of the intermediate edge, values of connected vertices
// and values of intermediate edges in the original bipartite graph:
// (2, 4, (1, "top1", "bottom2", "bottom4", "1-2-edge", "1-4-edge"))
Graph<String, String, Projection<Long, String, String, String>> graph bipartiteGraph.projectionBottomFull();

{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
// Scala API is not yet supported
{% endhighlight %}
</div>
</div>

{% top %}
