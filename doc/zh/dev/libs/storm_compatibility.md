---
标题: "Storm 兼容测试"
is_beta: true
nav-parent_id: libs
nav-pos: 2
---
<!--
授权给Apache软件基金会(ASF)
或者更多的贡献者许可协议。参见通知文件
随本著作一起分发以获取更多信息
关于版权的所有权。ASF许可这个文件
根据Apache许可证，2.0版
“许可”);除非符合规定，否则不得使用此文件
的许可证。详情可见
http://www.apache.org/licenses/LICENSE-2.0

除非适用法律要求或经书面同意，根据许可协议发布的软件在
“按现状”的基础，没有任何保证或可能条件，无论是明示的还是暗示的。的许可证
控制权限和限制的特定语言根据许可证。
-->
[Flink streaming]({{ site.baseurl }}/dev/datastream_api.html) 
Flink流与Apache Storm接口兼容，因此允许重用为Storm实现的代码.
列如:
 1.在Flink中执行整个Storm Topology。
 2.在Flink Streaming中,使用Storm Spout/Bolt 作为数据源/操作符。
本文档介绍了如何在Flink中使用现有的Storm代码。
* 这个将被TOC替换
{:toc}

# 项目配置
支持Storm包含在flink-stormMaven模块中。代码在org.apache.flink.storm包中。
如果要在Flink中执行Storm代码，请将以下依赖项添加到您的pom里面
~~~xml
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-storm_2.11</artifactId>
	<version>1.7.1</version>
</dependency>
~~~

**请注意**：不要添加storm-core为依赖项。它已包含在内flink-storm。
**请注意**:Flink -storm不是提供的二进制Flink发行版的一部分。因此，您需要在提交给Flink的JobManager的程序jar(也称为uber-jar或fat-jar)中包含Flink -storm类(及其依赖项)。
有关如何正确打包jar的示例，请参见flink-storm-examples/pom.xml中的WordCount Storm。

# 运行 Storm Topologies
Flink提供了一个与Storm兼容的API (' org.apache.flink.storm.api ')，它提供了以下类的替代品:
- `StormSubmitter` 替代 `FlinkSubmitter`
- `NimbusClient` and `Client` 替代`FlinkClient`
- `LocalCluster` 替代 `FlinkLocalCluster`
为了将Storm topology 提交给Flink,在topology Storm *client代码中，用它们的Flink替换使用过的Storm类就足够了




