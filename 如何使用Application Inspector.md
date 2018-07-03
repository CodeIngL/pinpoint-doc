## 如何使用Application Inspector

1. 介绍
2. 架构
3. 配置
4. 监控流式作业
5. 其他

## 应用程序检查器 ##

### 1. 介绍 ###

应用程序检查器提供了在同一应用程序名称下注册的所有`agent`资源数据（cpu，内存，tps，数据源连接数等）的聚合视图。 为应用程序检查员提供了与`agent`检查器类似的统计图的单独视图。

要访问应用程序检查器，请单击屏幕左侧的应用程序检查器菜单。

	- 1：应用程序检查器菜单，2：应用程序统计数据
	
![](http://naver.github.io/pinpoint/images/inspector_view.jpg)

例如，上面的堆使用情况图显示了在同一应用程序名称下注册的代理程序的平均（平均），最小（最小），最大（最大）堆使用率以及具有最小/最大的堆使用率的代理程序的ID 在某个时间点。 应用程序检查员还以类似的方式提供`agent`检查员中发现的其他统计信息。

![](http://naver.github.io/pinpoint/images/graph.jpg)

应用程序检查器需要[flink](https://flink.apache.org/)和[zookeeper](https://zookeeper.apache.org/)。 请阅读更多细节。

### 2. 架构 ###

![](http://naver.github.io/pinpoint/images/execute_flow.jpg)

**A.** 在[flink](https://flink.apache.org/)上运行流式`job`。  

**B.** `task manager`服务在作业开始后作为数据节点注册到`zookeeper`。

C. 收集器从`zookeeper`获取`flink`服务器信息以创建一个`tcp`连接并开始发送`agent`数据。

D. `flink`服务聚合`Collector`发送的数据并将它们存储到`hbase`中

### 3. 配置 ###

为了启用应用程序检查器，您需要执行以下操作并运行`pinpoint`。

**A.** 创建**ApplicationStatAggre**表（参考[create table script](https://github.com/naver/pinpoint/tree/master/hbase/scripts)），该表存储应用程序统计数据。

**B.** 在[pinpoint-flink.properties](https://github.com/naver/pinpoint/blob/master/flink/src/main/resources/pinpoint-flink.properties)中配置`zookeeper`地址，该地址将用于存储`flink`的`task manager`服务信息。

    flink.cluster.enable=true
    flink.cluster.zookeeper.address=YOUR_ZOOKEEPER_ADDRESS
    flink.cluster.zookeeper.sessiontimeout=3000
    flink.cluster.zookeeper.retry.interval=5000
    flink.cluster.tcp.port=19994

**C.** 在[pinpoint-flink.properties](https://github.com/naver/pinpoint/blob/master/flink/src/main/resources/pinpoint-flink.properties)中配置作业执行类型和从`Collector `接收数据的监听器的数量。

- 如果您正在运行`flink`群集，请将*flink.StreamExecutionEnvironment*设置为**server**，将*flink.sourceFunction.Parallel*设置为`task manager`服务的数量。
- 如果您将`flink`作为独立运行，请将*flink.StreamExecutionEnvironment*设置为**local**，并将*flink.sourceFunction.Parallel*设置为**1**。

	  flink.StreamExecutionEnvironment=server
	  flink.sourceFunction.Parallel=1

**D.** 在[hbase.properties](https://github.com/naver/pinpoint/blob/master/flink/src/main/resources/hbase.properties)中配置将用于存储聚合应用程序数据的hbase地址。

**E.** 构建[pinpoint-flink](https://github.com/naver/pinpoint/tree/master/flink)并运行`flink`服务上`target`目录下创建的流式作业文件。

- 流式作业的名称是`pinpoint-flink-job.2.0.jar`。
- 有关如何执行作业的详细信息，请参阅[flink网站](https://flink.apache.org/)。
**F.** 在[pinpoint-collector.properties](https://github.com/naver/pinpoint/blob/master/collector/src/main/resources/pinpoint-collector.properties)中配置zookeeper地址，以便收集器可以连接到flink服务。

	    flink.cluster.enable=true
		flink.cluster.zookeeper.address=YOUR_ZOOKEEPER_ADDRESS
	    flink.cluster.zookeeper.sessiontimeout=3000

**G.** 通过在[pinpoint-web.properties](https://github.com/naver/pinpoint/blob/master/web/src/main/resources/pinpoint-web.properties)中启用以下配置，在web-ui中启用应用程序检查器。

    config.show.applicationStat=true

### 4. 监控流式作业 ###

有一个批处理作业监视Pinpoint流作业如何运行。 要启用此批处理作业，请为`pinpoint-web`配置以下文件。

**batch.properties**

	batch.flink.server=FLINK_MANGER_SERVER_IP_LIST
	# Flink job manager server IPs, separated by ','.
	# ex) batch.flink.server=123.124.125.126,123.124.125.127

**applicationContext-batch-schedule.xml**

	<task:scheduled-tasks scheduler="scheduler">
		...
		<task:scheduled ref="batchJobLauncher" method="flinkCheckJob" cron="0 0/10 * * * *" />
	</task:scheduled-tasks>

如果想在发生批处理作业失败时发送警报，则必须实现`com.navercorp.pinpoint.web.batch.JobFailMessageSender`类并将其注册为Spring bean。

### 5. 其他 ###

有关如何安装和操作flink的更多详细信息，请参阅[flink网站](https://flink.apache.org/)。

