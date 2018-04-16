---
标题: "应用分析"
nav-parent_id: monitoring 监控
nav-pos: 15
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

* ToC
{:toc}

## 自定义日志 Apache Flink 

每一个 standalone JobManager, TaskManager, HistoryServer, and ZooKeeper 守护程序重定向 `stdout` and `stderr` to a file
with a `.out` filename suffix and writes 内部的 logging to a file with a `.log` suffix. 在java里面配置
 `env.java.opts`, `env.java.opts.jobmanager`, and `env.java.opts.taskmanager` 同样可以定义日志文件
使用这些变量 `FLINK_LOG_PREFIX` and by enclosing the options in 进行引导设置. Log files
using `FLINK_LOG_PREFIX` are rotated along with the default `.out` and `.log` files.

# java运行记录
通过Oracle JDK来分析记录的 通过java收集数据，可以高效有效的分析并处理数据进行记录
Java Flight Recorder  
[Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html)
is an advanced set of tools that enables efficient and detailed analysis of the extensive of data collected by Java
Flight Recorder. Example configuration:

~~~
env.java.opts: "-XX:+UnlockCommercialFeatures -XX:+UnlockDiagnosticVMOptions -XX:+FlightRecorder -XX:+DebugNonSafepoints -XX:FlightRecorderOptions=defaultrecording=true,dumponexit=true,dumponexitpath=${FLINK_LOG_PREFIX}.jfr"
~~~

# Profiling with JITWatch
日志分析可视化系统 java HotSpot 即时编译系统 用来检查内部关系，热加载，装配字节码。
配置如下:
[JITWatch](https://github.com/AdoptOpenJDK/jitwatch/wiki) is a log analyser and visualizer for the Java HotSpot JIT
compiler used to inspect inlining decisions, hot methods, bytecode, and assembly. Example configuration:

~~~
env.java.opts: "-XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompilation -XX:LogFile=${FLINK_LOG_PREFIX}.jit -XX:+PrintAssembly"
~~~

{% top %}
