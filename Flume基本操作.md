#  一、监听端口将结果输出到console
1. 编写配置文件job_flume_netcat.conf
```properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

2. 启动flume进程
```shell
flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a1 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/job_flume_netcat.conf -Dflume.root.logger==INFO,console
```

3. 查看指定端口是否被flume监听
```shell
sudo netstat -tunlp | grep 44444
```

4. 用telnet发送数据
```shell
telnet localhost 44444
```
<br/>
# 二、动态读取日志文件并写入到hdfs
1. 编写配置文件job_flume_2hdfs.conf
```properties
# Name the components on this agent 
a2.sources = r2
a2.sinks = k2
a2.channels = c2
# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /home/hadoop/app/hive-1.2.2/logs/hive.log
a2.sources.r2.shell = /bin/bash -c

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop01:9000/flume/hive-logs/%Y%m%d/%H

#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-

#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true

#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1

#重新定义时间单位 
a2.sinks.k2.hdfs.roundUnit = hour

#是否使用本地时间戳 
a2.sinks.k2.hdfs.useLocalTimeStamp = true

#积攒多少个Event才flush到HDFS 一次
a2.sinks.k2.hdfs.batchSize = 1000

#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream

#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 600

#设置每个文件的滚动大小 
a2.sinks.k2.hdfs.rollSize = 134217700

#文件的滚动与Event数量无关 
a2.sinks.k2.hdfs.rollCount = 0

#最小冗余数
a2.sinks.k2.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory 
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel 
a2.sources.r2.channels = c2 
a2.sinks.k2.channel = c2
```
2. 启动flume进程
```shell
flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a2 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/job_flume_2hdfs.conf
```
<br/>
# 三、实时读取目录文件到HDFS
1. 编写配置文件flume-dir.conf
```properties
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /home/hadoop/app/apache-flume-1.7.0/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true

#忽略所有以.tmp 结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop01:9000/flume/upload/%Y%m%d/%H

#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-

#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true

#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1

#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour

#是否使用本地时间戳 
a3.sinks.k3.hdfs.useLocalTimeStamp = true

#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100

#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream

#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 600

#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700

#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

#最小冗余数
a3.sinks.k3.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory 
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```
2. 启动flume进程
```shell
flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a3 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-dir.conf
```
<br/>
# 四、Flume与Flume之间数据传递:单Flume多Channel、 Sink
1. 编写配置文件
   * flume-1.conf
   ```properties
   # Name the components on this agent
   a1.sources = r1
   a1.sinks = k1 k2
   a1.channels = c1 c2

   # 将数据流复制给多个 channel 
   a1.sources.r1.selector.type = replicating

   # Describe/configure the source
   a1.sources.r1.type = exec
   a1.sources.r1.command = tail -F /home/hadoop/app/hive-1.2.2/logs/hive.log
   a1.sources.r1.shell = /bin/bash -c

   # Describe the sink 
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop01
   a1.sinks.k1.port = 4141
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop01
   a1.sinks.k2.port = 4142

   # Describe the channel 
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100

   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 1000
   a1.channels.c2.transactionCapacity = 100

   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1 c2
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ```

   * flume-2.conf
   ```properties
   # Name the components on this agent
   a2.sources = r1
   a2.sinks = k1
   a2.channels = c1

   # Describe/configure the source
   a2.sources.r1.type = avro
   a2.sources.r1.bind = hadoop01
   a2.sources.r1.port = 4141

   # Describe the sink
   a2.sinks.k1.type = hdfs
   a2.sinks.k1.hdfs.path = hdfs://hadoop01:9000/flume2/%Y%m%d/%H
   #上传文件的前缀
   a2.sinks.k1.hdfs.filePrefix = flume2-

   #是否按照时间滚动文件夹
   a2.sinks.k1.hdfs.round = true

   #多少时间单位创建一个新的文件夹
   a2.sinks.k1.hdfs.roundValue = 1

   #重新定义时间单位
   a2.sinks.k1.hdfs.roundUnit = hour

   #是否使用本地时间戳
   a2.sinks.k1.hdfs.useLocalTimeStamp = true

   #积攒多少个Event才flush到HDFS 一次
   a2.sinks.k1.hdfs.batchSize = 100

   #设置文件类型，可支持压缩
   a2.sinks.k1.hdfs.fileType = DataStream

   #多久生成一个新的文件
   a2.sinks.k1.hdfs.rollInterval = 600

   #设置每个文件的滚动大小大概是128M
   a2.sinks.k1.hdfs.rollSize = 134217700

   #文件的滚动与Event数量无关
   a2.sinks.k1.hdfs.rollCount = 0

   #最小冗余数 
   a2.sinks.k1.hdfs.minBlockReplicas = 1

   # Describe the channel
   a2.channels.c1.type = memory
   a2.channels.c1.capacity = 1000
   a2.channels.c1.transactionCapacity = 100

   # Bind the source and sink to the channel
   a2.sources.r1.channels = c1
   a2.sinks.k1.channel = c1
   ```

   * flume-3.conf
   ```properties
   # Name the components on this agent
   a3.sources = r1
   a3.sinks = k1
   a3.channels = c1

   # Describe/configure the source 
   a3.sources.r1.type = avro
   a3.sources.r1.bind = hadoop01
   a3.sources.r1.port = 4142

   # Describe the sink 文件夹必须存在
   a3.sinks.k1.type = file_roll
   a3.sinks.k1.sink.directory = /home/hadoop/flume2

   # Describe the channel 
   a3.channels.c1.type = memory
   a3.channels.c1.capacity = 1000
   a3.channels.c1.transactionCapacity = 100

   # Bind the source and sink to the channel
   a3.sources.r1.channels = c1
   a3.sinks.k1.channel = c1
   ```

2. 启动flume进程
```shell
flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a1 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-1.conf

flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a2 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-2.conf

flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a3 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-3.conf
```
<br/>
# 五、Flume与Flume之间数据传递，多Flume汇总数据到单Flume
1. 编写配置文件
   * flume-a.conf
   ```properties
   # Name the components on this agent 
   a1.sources = r1
   a1.sinks = k1 
   a1.channels = c1

   # Describe/configure the source
   a1.sources.r1.type = exec
   a1.sources.r1.command = tail -F /home/hadoop/app/hive-1.2.2/logs/hive.log
   a1.sources.r1.shell = /bin/bash -c

   # Describe the sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop01
   a1.sinks.k1.port = 4141

   # Describe the channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100

   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ```

   * flume-b.conf
   ```properties
   # Name the components on this agent
   a2.sources = r1
   a2.sinks = k1
   a2.channels = c1

   # Describe/configure the source
   a2.sources.r1.type = netcat
   a2.sources.r1.bind = hadoop01
   a2.sources.r1.port = 44444

   # Describe the sink
   a2.sinks.k1.type = avro
   a2.sinks.k1.hostname = hadoop01
   a2.sinks.k1.port = 4141

   # Use a channel which buffers events in memory
   a2.channels.c1.type = memory
   a2.channels.c1.capacity = 1000
   a2.channels.c1.transactionCapacity = 100

   # Bind the source and sink to the channel
   a2.sources.r1.channels = c1
   a2.sinks.k1.channel = c1
   ```

   * flume-c.conf
   ```properties
   # Name the components on this agent 
   a3.sources = r1
   a3.sinks = k1
   a3.channels = c1

   # Describe/configure the source
   a3.sources.r1.type = avro
   a3.sources.r1.bind = hadoop01
   a3.sources.r1.port = 4141

   # Describe the sink
   a3.sinks.k1.type = hdfs
   a3.sinks.k1.hdfs.path = hdfs://hadoop01:9000/flume3/%Y%m%d/%H

   #上传文件的前缀
   a3.sinks.k1.hdfs.filePrefix = flume3-

   #是否按照时间滚动文件夹
   a3.sinks.k1.hdfs.round = true

   #多少时间单位创建一个新的文件夹
   a3.sinks.k1.hdfs.roundValue = 1

   #重新定义时间单位
   a3.sinks.k1.hdfs.roundUnit = hour

   #是否使用本地时间戳
   a3.sinks.k1.hdfs.useLocalTimeStamp = true

   #积攒多少个 Event 才 flush 到 HDFS 一次
   a3.sinks.k1.hdfs.batchSize = 100

   #设置文件类型，可支持压缩
   a3.sinks.k1.hdfs.fileType = DataStream

   #多久生成一个新的文件
   a3.sinks.k1.hdfs.rollInterval = 600

   #设置每个文件的滚动大小大概是 128M
   a3.sinks.k1.hdfs.rollSize = 134217700

   #文件的滚动与 Event 数量无关
   a3.sinks.k1.hdfs.rollCount = 0

   #最小冗余数
   a3.sinks.k1.hdfs.minBlockReplicas = 1

   # Describe the channel
   a3.channels.c1.type = memory
   a3.channels.c1.capacity = 1000
   a3.channels.c1.transactionCapacity = 100

   # Bind the source and sink to the channel
   a3.sources.r1.channels = c1
   a3.sinks.k1.channel = c1
   ```

2. 启动flume进程
```shell
flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a1 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-a.conf

flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a2 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-b.conf

flume-ng agent --conf /home/hadoop/app/apache-flume-1.7.0/conf/ --name a3 --conf-file /home/hadoop/app/apache-flume-1.7.0/job/flume-c.conf
```
3. 用telnet发送数据
```shell
telnet localhost 44444
```
