---
标题:图创建
nav-parent_id:图
nav-pos: 5
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

* 这将被TOC替换
{:toc}


<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

boolean wrapEndpoints = false;

int parallelism = 4;

Graph<LongValue, NullValue, NullValue> graph = new GridGraph(env)
    .addDimension(2, wrapEndpoints)
    .addDimension(4, wrapEndpoints)
    .setParallelism(parallelism)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.GridGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

wrapEndpoints = false

val parallelism = 4

val graph = new GridGraph(env.getJavaEnv).addDimension(2, wrapEndpoints).addDimension(4, wrapEndpoints).setParallelism(parallelism).generate()
{% endhighlight %}
</div>
</div>

# #循环图
[circulant graph](http://mathworld.wolfram.com/CirculantGraph.html)是一个
(有向图)(http://mathworld.wolfram.com/OrientedGraph.html)配置
具有一个或多个连续范围的偏移量。边连接整数顶点id
其差值等于配置偏移量。没有偏移量的循环图
是否[空图](#empty-graph)和具有最大范围的图是
(完全图)(#完全图)。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5;

Graph<LongValue, NullValue, NullValue> graph = new CirculantGraph(env, vertexCount)
    .addRange(1, 2)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.CirculantGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5

val graph = new CirculantGraph(env.getJavaEnv, vertexCount).addRange(1, 2).generate()
{% endhighlight %}
</div>
</div>

# #完全图
连接每一对不同顶点的无向图。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5;

Graph<LongValue, NullValue, NullValue> graph = new CompleteGraph(env, vertexCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.CompleteGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5

val graph = new CompleteGraph(env.getJavaEnv, vertexCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="540"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="270" y1="40" x2="489" y2="199" />
    <line x1="270" y1="40" x2="405" y2="456" />
    <line x1="270" y1="40" x2="135" y2="456" />
    <line x1="270" y1="40" x2="51" y2="199" />

    <line x1="489" y1="199" x2="405" y2="456" />
    <line x1="489" y1="199" x2="135" y2="456" />
    <line x1="489" y1="199" x2="51" y2="199" />

    <line x1="405" y1="456" x2="135" y2="456" />
    <line x1="405" y1="456" x2="51" y2="199" />

    <line x1="135" y1="456" x2="51" y2="199" />

    <circle cx="270" cy="40" r="20" />
    <text x="270" y="40">0</text>

    <circle cx="489" cy="199" r="20" />
    <text x="489" y="199">1</text>

    <circle cx="405" cy="456" r="20" />
    <text x="405" y="456">2</text>

    <circle cx="135" cy="456" r="20" />
    <text x="135" y="456">3</text>

    <circle cx="51" cy="199" r="20" />
    <text x="51" y="199">4</text>
</svg>

# #循环图
一种无向图，其中一组边通过连接形成一个循环
每个顶点到链循环中的两个相邻顶点。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5;

Graph<LongValue, NullValue, NullValue> graph = new CycleGraph(env, vertexCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.CycleGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5

val graph = new CycleGraph(env.getJavaEnv, vertexCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="540"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="270" y1="40" x2="489" y2="199" />
    <line x1="489" y1="199" x2="405" y2="456" />
    <line x1="405" y1="456" x2="135" y2="456" />
    <line x1="135" y1="456" x2="51" y2="199" />
    <line x1="51" y1="199" x2="270" y2="40" />

    <circle cx="270" cy="40" r="20" />
    <text x="270" y="40">0</text>

    <circle cx="489" cy="199" r="20" />
    <text x="489" y="199">1</text>

    <circle cx="405" cy="456" r="20" />
    <text x="405" y="456">2</text>

    <circle cx="135" cy="456" r="20" />
    <text x="135" y="456">3</text>

    <circle cx="51" cy="199" r="20" />
    <text x="51" y="199">4</text>
</svg>
##回波图
一个[echo graph](http://mathworld.wolfram.com/EchoGraph.html)是一个
[循环图](#circulant-graph)，由a的宽度定义n个顶点
以' n/2 '为中心的单范围偏移量。一个顶点连接到“远”
连接到“近”顶点的顶点，连接到“远”顶点的顶点，…
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5;
long vertexDegree = 2;

Graph<LongValue, NullValue, NullValue> graph = new EchoGraph(env, vertexCount, vertexDegree)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.EchoGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5
val vertexDegree = 2

val graph = new EchoGraph(env.getJavaEnv, vertexCount, vertexDegree).generate()
{% endhighlight %}
</div>
</div>
##空图
没有边的图。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5;

Graph<LongValue, NullValue, NullValue> graph = new EmptyGraph(env, vertexCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.EmptyGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5

val graph = new EmptyGraph(env.getJavaEnv, vertexCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="80"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <circle cx="30" cy="40" r="20" />
    <text x="30" y="40">0</text>

    <circle cx="150" cy="40" r="20" />
    <text x="150" y="40">1</text>

    <circle cx="270" cy="40" r="20" />
    <text x="270" y="40">2</text>

    <circle cx="390" cy="40" r="20" />
    <text x="390" y="40">3</text>

    <circle cx="510" cy="40" r="20" />
    <text x="510" y="40">4</text>
</svg>

# #网格图
一种无向图，在一个或多个维度中连接规则平铺中的顶点。
每个维度都是单独配置的。当尺寸大小至少为3时
端点通过设置“wrapendpoint”可选地连接。改变以下
例如“addDimension(4, true)”将“0”连接到“3”，“4”连接到“7”。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

boolean wrapEndpoints = false;

Graph<LongValue, NullValue, NullValue> graph = new GridGraph(env)
    .addDimension(2, wrapEndpoints)
    .addDimension(4, wrapEndpoints)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.GridGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val wrapEndpoints = false

val graph = new GridGraph(env.getJavaEnv).addDimension(2, wrapEndpoints).addDimension(4, wrapEndpoints).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="200"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="30" y1="40" x2="510" y2="40" />
    <line x1="30" y1="160" x2="510" y2="160" />

    <line x1="30" y1="40" x2="30" y2="160" />
    <line x1="190" y1="40" x2="190" y2="160" />
    <line x1="350" y1="40" x2="350" y2="160" />
    <line x1="510" y1="40" x2="510" y2="160" />

    <circle cx="30" cy="40" r="20" />
    <text x="30" y="40">0</text>

    <circle cx="190" cy="40" r="20" />
    <text x="190" y="40">1</text>

    <circle cx="350" cy="40" r="20" />
    <text x="350" y="40">2</text>

    <circle cx="510" cy="40" r="20" />
    <text x="510" y="40">3</text>

    <circle cx="30" cy="160" r="20" />
    <text x="30" y="160">4</text>

    <circle cx="190" cy="160" r="20" />
    <text x="190" y="160">5</text>

    <circle cx="350" cy="160" r="20" />
    <text x="350" y="160">6</text>

    <circle cx="510" cy="160" r="20" />
    <text x="510" y="160">7</text>
</svg>

# #超立方体图
一种无向图，其中边形成一个' n '维超立方体。每个顶点
在超立方体中，每个维度都与另一个顶点相连。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long dimensions = 3;

Graph<LongValue, NullValue, NullValue> graph = new HypercubeGraph(env, dimensions)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.HypercubeGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val dimensions = 3

val graph = new HypercubeGraph(env.getJavaEnv, dimensions).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="320"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="190" y1="120" x2="350" y2="120" />
    <line x1="190" y1="200" x2="350" y2="200" />
    <line x1="190" y1="120" x2="190" y2="200" />
    <line x1="350" y1="120" x2="350" y2="200" />

    <line x1="30" y1="40" x2="510" y2="40" />
    <line x1="30" y1="280" x2="510" y2="280" />
    <line x1="30" y1="40" x2="30" y2="280" />
    <line x1="510" y1="40" x2="510" y2="280" />

    <line x1="190" y1="120" x2="30" y2="40" />
    <line x1="350" y1="120" x2="510" y2="40" />
    <line x1="190" y1="200" x2="30" y2="280" />
    <line x1="350" y1="200" x2="510" y2="280" />

    <circle cx="190" cy="120" r="20" />
    <text x="190" y="120">0</text>

    <circle cx="350" cy="120" r="20" />
    <text x="350" y="120">1</text>

    <circle cx="190" cy="200" r="20" />
    <text x="190" y="200">2</text>

    <circle cx="350" cy="200" r="20" />
    <text x="350" y="200">3</text>

    <circle cx="30" cy="40" r="20" />
    <text x="30" y="40">4</text>

    <circle cx="510" cy="40" r="20" />
    <text x="510" y="40">5</text>

    <circle cx="30" cy="280" r="20" />
    <text x="30" y="280">6</text>

    <circle cx="510" cy="280" r="20" />
    <text x="510" y="280">7</text>
</svg>

# #路径图
一种无向图，其中一组边通过连接形成一条路径
两个度为1的端点顶点和所有度为1的中点顶点
“2”。路径图可以通过从循环图中删除一条边来形成。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 5

Graph<LongValue, NullValue, NullValue> graph = new PathGraph(env, vertexCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.PathGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 5

val graph = new PathGraph(env.getJavaEnv, vertexCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="80"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="30" y1="40" x2="510" y2="40" />

    <circle cx="30" cy="40" r="20" />
    <text x="30" y="40">0</text>

    <circle cx="150" cy="40" r="20" />
    <text x="150" y="40">1</text>

    <circle cx="270" cy="40" r="20" />
    <text x="270" y="40">2</text>

    <circle cx="390" cy="40" r="20" />
    <text x="390" y="40">3</text>

    <circle cx="510" cy="40" r="20" />
    <text x="510" y="40">4</text>
</svg>

# # RMat图
使用。生成的有向幂律多图
[递归矩阵(R-Mat)](http://www.cs.cmu.edu/~christos/PUBLICATIONS/siam04.pdf)模型。
RMat是一个随机生成器，配置了实现
“RandomGenerableFactory”界面。提供的实现是“JDKRandomGeneratorFactory”
和“MersenneTwisterFactory”。它们生成一个随机值的初始序列
然后用作产生边缘的种子。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

RandomGenerableFactory<JDKRandomGenerator> rnd = new JDKRandomGeneratorFactory();

int vertexCount = 1 << scale;
int edgeCount = edgeFactor * vertexCount;

Graph<LongValue, NullValue, NullValue> graph = new RMatGraph<>(env, rnd, vertexCount, edgeCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.RMatGraph

val env = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 1 << scale
val edgeCount = edgeFactor * vertexCount

val graph = new RMatGraph(env.getJavaEnv, rnd, vertexCount, edgeCount).generate()
{% endhighlight %}
</div>
</div>

默认的RMat常量可以被覆盖，如下面的例子所示。
这些常量定义了来自每个生成边缘源的比特之间的相互依赖关系
和目标标签。可以启用RMat噪声，并逐步干扰
生成每条边时的常数。
RMat生成器可以配置为通过删除自循环来生成一个简单的图
和重复的边缘。对称可以通过“剪切-翻转”丢弃来执行
对角线以上的半矩阵或全部“翻转”保留和镜像所有边

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

RandomGenerableFactory<JDKRandomGenerator> rnd = new JDKRandomGeneratorFactory();

int vertexCount = 1 << scale;
int edgeCount = edgeFactor * vertexCount;

boolean clipAndFlip = false;

Graph<LongValue, NullValue, NullValue> graph = new RMatGraph<>(env, rnd, vertexCount, edgeCount)
    .setConstants(0.57f, 0.19f, 0.19f)
    .setNoise(true, 0.10f)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.RMatGraph

val env = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 1 << scale
val edgeCount = edgeFactor * vertexCount

clipAndFlip = false

val graph = new RMatGraph(env.getJavaEnv, rnd, vertexCount, edgeCount).setConstants(0.57f, 0.19f, 0.19f).setNoise(true, 0.10f).generate()
{% endhighlight %}
</div>
</div>

## 单边缘图
   一个无向图，包含孤立的两条路径，其中每个顶点都有度数
`1`.

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexPairCount = 4

// note: configured with the number of vertex pairs
Graph<LongValue, NullValue, NullValue> graph = new SingletonEdgeGraph(env, vertexPairCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.SingletonEdgeGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexPairCount = 4

// note: configured with the number of vertex pairs
val graph = new SingletonEdgeGraph(env.getJavaEnv, vertexPairCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="200"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="30" y1="40" x2="190" y2="40" />
    <line x1="350" y1="40" x2="510" y2="40" />
    <line x1="30" y1="160" x2="190" y2="160" />
    <line x1="350" y1="160" x2="510" y2="160" />

    <circle cx="30" cy="40" r="20" />
    <text x="30" y="40">0</text>

    <circle cx="190" cy="40" r="20" />
    <text x="190" y="40">1</text>

    <circle cx="350" cy="40" r="20" />
    <text x="350" y="40">2</text>

    <circle cx="510" cy="40" r="20" />
    <text x="510" y="40">3</text>

    <circle cx="30" cy="160" r="20" />
    <text x="30" y="160">4</text>

    <circle cx="190" cy="160" r="20" />
    <text x="190" y="160">5</text>

    <circle cx="350" cy="160" r="20" />
    <text x="350" y="160">6</text>

    <circle cx="510" cy="160" r="20" />
    <text x="510" y="160">7</text>
</svg>

## Star Graph
   一个无向图，包含一个与所有其它叶顶点相连的中心顶点。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

long vertexCount = 6;

Graph<LongValue, NullValue, NullValue> graph = new StarGraph(env, vertexCount)
    .generate();
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.flink.api.scala._
import org.apache.flink.graph.generator.StarGraph

val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val vertexCount = 6

val graph = new StarGraph(env.getJavaEnv, vertexCount).generate()
{% endhighlight %}
</div>
</div>

<svg class="graph" width="540" height="540"
    xmlns="http://www.w3.org/2000/svg"
    xmlns:xlink="http://www.w3.org/1999/xlink">

    <line x1="270" y1="270" x2="270" y2="40" />
    <line x1="270" y1="270" x2="489" y2="199" />
    <line x1="270" y1="270" x2="405" y2="456" />
    <line x1="270" y1="270" x2="135" y2="456" />
    <line x1="270" y1="270" x2="51" y2="199" />

    <circle cx="270" cy="270" r="20" />
    <text x="270" y="270">0</text>

    <circle cx="270" cy="40" r="20" />
    <text x="270" y="40">1</text>

    <circle cx="489" cy="199" r="20" />
    <text x="489" y="199">2</text>

    <circle cx="405" cy="456" r="20" />
    <text x="405" y="456">3</text>

    <circle cx="135" cy="456" r="20" />
    <text x="135" y="456">4</text>

    <circle cx="51" cy="199" r="20" />
    <text x="51" y="199">5</text>
</svg>

{% top %}
