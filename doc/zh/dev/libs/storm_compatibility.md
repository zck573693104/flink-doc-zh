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
可以使用实际的运行时代码,ie, Spouts and Bolts,可以使用*unmodified*
topology 是否在远程集群中执行,参数`nimbus.host` 和 `nimbus.thrift.port`，分别使用`jobmanger.rpc.address` 和 `jobmanger.rpc.port`，
如果未指定参数，则从“flink-con .yaml”获取该值。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
TopologyBuilder builder = new TopologyBuilder(); // 构建Storm topology

// 实际topology 能够像原来一样使用Spouts/Bolts
builder.setSpout("source", new FileSpout(inputFilePath));
builder.setBolt("tokenizer", new BoltTokenizer()).shuffleGrouping("source");
builder.setBolt("counter", new BoltCounter()).fieldsGrouping("tokenizer", new Fields("word"));
builder.setBolt("sink", new BoltFileSink(outputFilePath)).shuffleGrouping("counter");

Config conf = new Config();
if(runLocal) { // submit to test cluster
	// replaces: LocalCluster cluster = new LocalCluster();
	FlinkLocalCluster cluster = new FlinkLocalCluster();
	cluster.submitTopology("WordCount", conf, FlinkTopology.createTopology(builder));
} else { // submit to remote cluster
	// optional
	// conf.put(Config.NIMBUS_HOST, "remoteHost");
	// conf.put(Config.NIMBUS_THRIFT_PORT, 6123);
	// replaces: StormSubmitter.submitTopology(topologyId, conf, builder.createTopology());
	FlinkSubmitter.submitTopology("WordCount", conf, FlinkTopology.createTopology(builder));
}
~~~
</div>
</div>

# Flink Streaming 操作Storm

作为替代方案, Spouts and Bolts 可以嵌入到stream.
Storm 兼容性层为每个类提供一个包装器类, 就是 `SpoutWrapper` 和 `BoltWrapper` (`org.apache.flink.storm.wrappers`).

默认情况下，两个包装器都会进行转换 Storm 输出 tuples 到 Flink's [Tuple]({{site.baseurl}}/dev/api_concepts.html#tuples-and-case-classes) types (ie, `Tuple0` to `Tuple25` according to the number of fields of the Storm tuples).
对于单个字段输出Tuple，也可以转换为字段的数据类型 (eg, `String` instead of `Tuple1<String>`).

因为 Flink 无法推断输出属于的 Storm 操作字段类型 , 需要手动指定输出类型。
为了得到正确的结果`TypeInformation` object, 可以使用Flink's `TypeExtractor` 

## 嵌入说明

为了使用Flink source作为管道流, 使用 `StreamExecutionEnvironment.addSource(SourceFunction, TypeInformation)`.
这个Spout object 的构造函数 `SpoutWrapper<OUT>` `addSource(...)`作为第一个参数.
泛型类型声明 `OUT` 指定源输出流的类型.

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// 流只有 `raw` 一个类型 (只有单个字段输出流)
DataStream<String> rawInput = env.addSource(
	new SpoutWrapper<String>(new FileSpout(localFilePath), new String[] { Utils.DEFAULT_STREAM_ID }), // emit default output stream as raw type
	TypeExtractor.getForClass(String.class)); // 输出类型

// 输出过程流
[...]
~~~
</div>
</div>
如果一个Spout发出有限数量的元组，那么可以通过在构造函数中设置“numberOfInvocations”参数将“spoutrapper”配置为自动终止。
这允许Flink程序在处理完所有数据后自动关闭。
默认情况下，程序将一直运行，直到手动[取消]({{site.baseurl}}/ops/cli.html)。
## 嵌入 Bolts
为了使用Bolt作为Flink的操作符, 使用`DataStream.transform(String, TypeInformation, OneInputStreamOperator)`.
Bolt使用`BoltWrapper<IN,OUT>`作为`transform(...)`的最后一个参数
泛型类型声明“IN”和“OUT”分别指定操作符的输入流和输出流的类型。
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream<String> text = env.readTextFile(localFilePath);

DataStream<Tuple2<String, Integer>> counts = text.transform(
	"tokenizer", // 操作名
	TypeExtractor.getForObject(new Tuple2<String, Integer>("", 0)), // 输出类型
	new BoltWrapper<String, Tuple2<String, Integer>>(new BoltTokenizer())); // Bolt 操作

// 做进一步处理
[...]
~~~
</div>
</div>

### Embedded Bolts的命名属性访问 

Bolts 通过名称访问输入元组字段(另外通过索引访问).
要使用Bolts, 需要下面操作

 1. [POJO]({{site.baseurl}}/dev/api_concepts.html#pojos) 输入流或
 2. [Tuple]({{site.baseurl}}/dev/api_concepts.html#tuples-and-case-classes) 输入流并指定输入模式 (i.e. name-to-index-mapping)

对于POJO输入类型，Flink通过反射访问字段。
对于这种情况，Flink需要一个对应的公共成员变量或公共getter方法。
例如，如果一个Bolt通过名称' sentence '访问一个字段(例如，' String s = input. getstringbyfield ("sentence"); ')，输入POJO类必须有一个成员变量' public String sentence; '或方法' public String getSentence(){…(注意驼峰的命名)。

为了 `Tuple` 输入类型, 需要使用Storm的“Fields”类指定输入模式。
对于这种情况，“BoltWrapper”的构造函数接受一个额外的参数: `new BoltWrapper<Tuple1<String>, ...>(..., new Fields("sentence"))`.
输入类型 `Tuple1<String>` 和 `Fields("sentence")` 比较 `input.getStringByField("sentence")` is equivalent to `input.getString(0)`.

看 [BoltTokenizerWordCountPojo](https://github.com/apache/flink/tree/master/flink-contrib/flink-storm-examples/src/main/java/org/apache/flink/storm/wordcount/BoltTokenizerWordCountPojo.java) and [BoltTokenizerWordCountWithNames](https://github.com/apache/flink/tree/master/flink-contrib/flink-storm-examples/src/main/java/org/apache/flink/storm/wordcount/BoltTokenizerWordCountWithNames.java) for examples.

## 配置 Spouts 和 Bolts

在Storm中，Spouts和Bolts可以配置一个全局分布的“Map”对象，该对象被赋予“LocalCluster”或“StormSubmitter”的“submitTopology(…)”方法。
此“MAP”由Topology的用户提供，并作为参数转发到调用“sp .open(…)”和“Bolt.prepare(…)”。
如果整个Topology使用FlinkTopologyBuilder等在Flink中执行，则不需要特别注意&ndash;它和storm一样有效。

对于嵌入式使用，必须使用Flink的配置机制。
全局配置可以通过“. getconfig (). setglobaljobparameters(…)”在“StreamExecutionEnvironment”中设置。
Flink的常规“配置”类可用于配置Spouts 和 Bolts
然而，' Configuration '不像Storm那样支持任意键数据类型(只允许' String '键)。
因此，Flink还提供了“StormConfig”类，可以像原始的“Map”那样使用它来提供对Storm的完全兼容性。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

StormConfig config = new StormConfig();
// 设置配置值
[...]

// storm全局配置
env.getConfig().setGlobalJobParameters(config);

// 嵌入式汇编程序 Spouts and/or Bolts
[...]
~~~
</div>
</div>
## 多个输出流
Flink还可以处理多个Spouts and Bolts输出流的声明。
使用flink执行完整的topology `FlinkTopologyBuilder` 等等., 不需要特别注意。它和storm一样有效。

对于嵌入式使用，输出流将是数据类型的 `SplitStreamType<T>`必须使用分割`DataStream.split(...)` 和 `SplitStream.select(...)`.
Flink已经为`.split(...)`提供预定义的输出选择器`StormStreamSelector<T>` 
此外，包装器类型 `SplitStreamTuple<T>` 可以被移除`SplitStreamMapper<T>`.
<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
[...]
//从Spout或Bolt获取数据流，它们声明两个输出流s1和s2，输出类型为SomeType
DataStream<SplitStreamType<SomeType>> multiStream = ...

SplitStream<SplitStreamType<SomeType>> splitStream = multiStream.split(new StormStreamSelector<SomeType>());

// 使用SplitStreamMapper删除SplitStreamType以获得SomeType类型的数据流 
DataStream<SomeType> s1 = splitStream.select("s1").map(new SplitStreamMapper<SomeType>()).returns(SomeType.class);
DataStream<SomeType> s2 = splitStream.select("s2").map(new SplitStreamMapper<SomeType>()).returns(SomeType.class);

// 对s1和s2做进一步的处理
[...]
~~~
</div>
</div>

看 [SpoutSplitExample.java](https://github.com/apache/flink/tree/master/flink-contrib/flink-storm-examples/src/main/java/org/apache/flink/storm/split/SpoutSplitExample.java) for a full example.

# Flink Extensions

## Finite Spouts

在Flink中，流源可以是有限的，即发出有限数量的记录，并在发出最后一条记录后停止。然而，喷口通常发出无限的流。
这两种方法之间的桥梁是“FiniteSpout”接口，除了“IRichSpout”之外，它还包含一个“reachedEnd()”方法，用户可以在其中指定一个停止条件。
用户可以通过实现这个接口来创建一个有限的喷口，而不是(或添加到)' IRichSpout '，并另外实现' reachedEnd() '方法。
与配置为发出有限数量元组的“spoutrapper”不同，“FiniteSpout”接口允许实现更复杂的终止条件。

虽然在Flink流程序中嵌入Spouts或将整个Storm拓扑提交给Flink时，并不需要使用有限的spout，但在某些情况下，它们可能会派上用场:

*要实现这一点，Spout行为方式与有限Flink源相同，只需进行最小的修改
*用户只想处理一个流一段时间;之后，喷口可以自动停止将文件读入流
*供测试用

一个有限Spout例子，只发出10秒的记录:

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
~~~java
public class TimedFiniteSpout extends BaseRichSpout implements FiniteSpout {
	[...] // implement open(), nextTuple(), ...

	private long starttime = System.currentTimeMillis();

	public boolean reachedEnd() {
		return System.currentTimeMillis() - starttime > 10000l;
	}
}
~~~
</div>
</div>

# storm兼容性的例子

您可以在Maven模块中找到更多示例 `flink-storm-examples`.
对于不同版本的WordCount 可以看 [README.md](https://github.com/apache/flink/tree/master/flink-contrib/flink-storm-examples/README.md).
要运行这些示例，您需要组装一个正确的jar文件。
`flink-storm-examples-{{ site.version }}.jar` is **no** 有效的用于作业执行的jar文件(它只是一个标准的maven工件).
这里有几个jar是为了分别嵌入 Spout and Bolt, namely `WordCount-SpoutSource.jar` and `WordCount-BoltTokenizer.jar`, respectively.
比较pom查看如何构建
此外，还有一个关于整个storm topologies的例子 (`WordCount-StormTopology.jar`).
您可以通过以下方式运行每个示例  `bin/flink run <jarname>.jar`.您可以通过以下方式运行eacc:在每个jar的清单文件中包含正确的入口点类
{% top %}
