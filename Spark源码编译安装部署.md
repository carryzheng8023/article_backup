### Spark源码编译安装部署



#### 一、下载并解压

[下载地址]([http://spark.apache.org/downloads.html](http://spark.apache.org/downloads.html))

```shell
wget http://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.4.3/spark-2.4.3.tgz
tar -zxvf spark-2.4.3.tgz
```



#### 二、编译

[源码编译文档说明](http://spark.apache.org/docs/latest/building-spark.html)

##### 1. 修改 dev/make-distribution.sh(加速编译过程)

```properties
#VERSION=$("$MVN" help:evaluate -Dexpression=project.version $@ 2>/dev/null\
#    | grep -v "INFO"\
#    | grep -v "WARNING"\
#    | tail -n 1)
#SCALA_VERSION=$("$MVN" help:evaluate -Dexpression=scala.binary.version $@ 2>/dev/null\
#    | grep -v "INFO"\
#    | grep -v "WARNING"\
#    | tail -n 1)
#SPARK_HADOOP_VERSION=$("$MVN" help:evaluate -Dexpression=hadoop.version $@ 2>/dev/null\
#    | grep -v "INFO"\
#    | grep -v "WARNING"\
#    | tail -n 1)
#SPARK_HIVE=$("$MVN" help:evaluate -Dexpression=project.activeProfiles -pl sql/hive $@ 2>/dev/null\
#    | grep -v "INFO"\
#    | grep -v "WARNING"\
#    | fgrep --count "<id>hive</id>";\
#    # Reset exit status to 0, otherwise the script stops here if the last grep finds nothing\
#    # because we use "set -o pipefail"
#    echo -n)

VERSION=2.4.3
SCALA_VERSION=2.11
SPARK_HADOOP_VERSION=2.7.2
SPARK_HIVE=1
```

##### 2. 执行编译

* 修改scala version

```shell
./dev/change-scala-version.sh 2.11
```
* 执行编译脚本

```shell
./dev/make-distribution.sh \
--name hadoop2.7.2 \
--pip --tgz \
-Pscala-2.11 \
-Phadoop-2.7 -Dhadoop.version=2.7.2 \
-Phive -Phive-thriftserver \
-Pmesos -Pyarn -Pkubernetes
```



#### 三、部署(local|standalone|yarn)

##### 1. 解压编译后的压缩包

```shell
tar -zxvf spark-2.4.3-bin-hadoop2.7.2.tgz
```

##### 2. 修改配置文件

* $SPARK_HOME/conf/spark-env.sh

```properties
HADOOP_CONF_DIR=/home/hadoop/app/hadoop-2.7.2/etc/hadoop
SPARK_MASTER_HOST=hadoop01
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
```

* $SPARK_HOME/conf/slaves

```properties
hadoop01
```

* 将hive-site.xml引用到$SPARK_HOME/conf/

```shell
ln -s $HIVE_HOME/conf/hive-site.xml $SPARK_HOME/conf/hive-site.xml
```

* 将mysql的jar包添加到$SPARK_HOME/jars/目录



#### 四、启动Spark

* local模式

```shell
$SPARK_HOME/bin/spark-shell --master local[2]
```

* standalone模式

```shell
$SPARK_HOME/sbin/start-all.sh
$SPARK_HOME/bin/spark-shell --master spark://host:port
```

* yarn模式

```shell
$SPARK_HOME/bin/spark-shell --master yarn --deploy-mode client|cluster
```

![](http://pic.carryzheng.xin/zx_md/20190808110417.png)



