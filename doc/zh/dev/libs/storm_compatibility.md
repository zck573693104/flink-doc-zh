**Storm 兼容性接口测试**
Flink流与Apache Storm接口兼容，因此允许重用为Storm实现的代码。
列如:
 1.在Flink中执行整个Storm Topology。
 2.在Flink Streaming中,使用Storm Spout/Bolt 作为数据源/操作符。
本文档介绍了如何在Flink中使用现有的Storm代码。

**项目配置**
支持Storm包含在flink-stormMaven模块中。代码在org.apache.flink.storm包中。
如果要在Flink中执行Storm代码，请将以下依赖项添加到您的pom里面
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-storm_2.11</artifactId>
	<version>1.7.1</version>
</dependency>
请注意：不要添加storm-core为依赖项。它已包含在内flink-storm。
请注意:Flink -storm不是提供的二进制Flink发行版的一部分。因此，您需要在提交给Flink的JobManager的程序jar(也称为uber-jar或fat-jar)中包含Flink -storm类(及其依赖项)。
有关如何正确打包jar的示例，请参见flink-storm-examples/pom.xml中的WordCount Storm。



