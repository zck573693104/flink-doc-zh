---
title: "Gelly: Flink 图的 API"
nav-id: graphs
nav-show_overview: true
nav-title: "Graphs: Gelly"
nav-parent_id: libs
nav-pos: 3
---
<!--
这将授权给Apache软件基金会(ASF)
或者更多贡献者许可协议。参见通知文件
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
根据许可证。我被TOC替换了
-->
Gelly是Flink的图形API。它包含了一组旨在简化Flink中图形分析应用程序开发的方法和实用程序。在Gelly中，可以使用与批处理API提供的类似的高级函数转换和修改图。Gelly提供了创建、转换和修改图形的方法，以及图形算法库。

{:#markdown-toc}
* [Graph API](graph_api.html)
* [Iterative Graph Processing](iterative_graph_processing.html)
* [Library Methods](library_methods.html)
* [Graph Algorithms](graph_algorithms.html)
* [Graph Generators](graph_generators.html)
* [Bipartite Graphs](bipartite_graph.html)

使用 Gelly
-----------

Gelly目前是*libraries* Maven项目的一部分。所有相关类都位于*org.apache.flink中。图*包。
  将以下依赖项添加到您的“pom”。Gelly使用的xml。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight xml %}
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-gelly{{ site.scala_version_suffix }}</artifactId>
    <version>{{site.version}}</version>
</dependency>
{% endhighlight %}
</div>
<div data-lang="scala" markdown="1">
{% highlight xml %}
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-gelly-scala{{ site.scala_version_suffix }}</artifactId>
    <version>{{site.version}}</version>
</dependency>
{% endhighlight %}
</div>
</div>


Gelly 运行列子
----------------------

Gelly库jar在[Flink发行版]中提供(https://flink.apache.org/downloads.html“Apache Flink: Downloads”)。
在**opt**目录中(对于Flink 1.2以上的版本，可以手动从该目录下载
ga(Maven中央)(http://search.maven.org/搜索| | 1 | flink % 20葛里炸药))。要运行Gelly示例，请使用**flink-gelly** (for
(对于Scala) jar必须复制到Flink的**lib**目录中。

~~~bash
cp opt/flink-gelly_*.jar lib/
cp opt/flink-gelly-scala_*.jar lib/
~~~

Gelly的examples jar包含每个库方法的驱动程序，并在**examples**目录中提供。
配置并启动集群后，列出可用的算法类:
~~~bash
./bin/start-cluster.sh
./bin/flink run examples/gelly/flink-gelly-examples_*.jar
~~~


~~~bash
./bin/flink run examples/gelly/flink-gelly-examples_*.jar --algorithm JaccardIndex
~~~

Display [graph metrics](./library_methods.html#metric) for a million vertex graph:

~~~bash
./bin/flink run examples/gelly/flink-gelly-examples_*.jar \
    --algorithm GraphMetrics --order directed \
    --input RMatGraph --type integer --scale 20 --simplify directed \
    --output print
~~~
图的大小由*\-\-scale*和*\-\-edge_factor*参数调整。的
[library generator](./graph_generator .html#rmat-graph)提供对其他配置的访问，以调整
幂律歪斜和随机噪声。

示例社交网络数据由[Stanford network Analysis Project](http://snap.stanford.edu/data/index.html)提供。
[com-lj](http://snap.stanford.edu/data/bigdata/communties/com-lj.ungraph.txt.gz)数据集是一个很好的初始大小。
在Flink的Web UI中运行一些算法并监控工作进程:

~~~bash
wget -O - http://snap.stanford.edu/data/bigdata/communities/com-lj.ungraph.txt.gz | gunzip -c > com-lj.ungraph.txt

./bin/flink run -q examples/gelly/flink-gelly-examples_*.jar \
    --algorithm GraphMetrics --order undirected \
    --input CSV --type integer --simplify undirected --input_filename com-lj.ungraph.txt --input_field_delimiter $'\t' \
    --output print

./bin/flink run -q examples/gelly/flink-gelly-examples_*.jar \
    --algorithm ClusteringCoefficient --order undirected \
    --input CSV --type integer --simplify undirected --input_filename com-lj.ungraph.txt --input_field_delimiter $'\t' \
    --output hash

./bin/flink run -q examples/gelly/flink-gelly-examples_*.jar \
    --algorithm JaccardIndex \
    --input CSV --type integer --simplify undirected --input_filename com-lj.ungraph.txt --input_field_delimiter $'\t' \
    --output hash
~~~
请提交特性请求并在用户[邮件列表]上报告问题(https://flink.apache.org/community.html#邮件列表)
或(Flink Jira)(https://issues.apache.org/jira/browse/FLINK)。我们欢迎对新的算法和功能的建议
以及[代码贡献](https://flink.apache.org/contribut-code.html)。

{% top %}
