## 将MySQL同步到HIVE并分区



### 一、初始化(第一次将mysql全量写入hive)

#### 0. 整体流程图 

![](http://pic.carryzheng.xin/zx_md/20190711142834.png)

#### 1. 流程说明

* 从mysql中查询出数据
* 写入到hive

#### 2. 查询mysql数据库(配置ExecuteSQL Processor)

![](http://pic.carryzheng.xin/zx_md/20190711143051.png)

> 其中必填项:
> * **Database Connection URL**
> * **SQL select query**
> ```sql
> select * from test_user
> ```



![](http://pic.carryzheng.xin/zx_md/20190711143151.png)

> 其中必填项:
>
> * **Database Connection URL**
>
> ```properties
> jdbc:mysql://10.0.31.239:3309/csust
> ```
>
> * **Database Driver Class Name**
>
> ```properties
> com.mysql.jdbc.Driver
> ```
>
> * **Database Driver Location(s)**
>
> ```properties
> /opt/monica/nifi-1.8/lcylibs/mysql-connector-java-5.1.47.jar
> ```




#### 3. 写入hive数据仓库(配置PutHiveStreaming Processor)

![](http://pic.carryzheng.xin/zx_md/20190711143643.png)

> 其中必填项:
>
> * **Hive Metastore URI**
>
> ```properties
> thrift://10.0.31.239:9083
> ```
>
> * **Hive Configuration Resources**
>
> ```properties
> /usr/hdp/2.6.4.0-91/hive2/conf/hive-site.xml
> ```
>
> * **Database Name**
>
> ```properties
> default
> ```
>
> * **Table Name**
>
> ```properties
> test_user2019
> ```

### 二、增量写入

#### 0. 整体流程图

![](http://pic.carryzheng.xin/zx_md/20190711144015.png)

#### 1. 流程说明

* 先从hive中查询当前最大id
* 将查询出的数据类型从avro转成json
* 从json中提取该id
* 根据id从mysql查出未写入到hive的数据
* 写入hive

#### 2. SelectMaxIdFromHive(配置SelectHiveQL Processor)

![](http://pic.carryzheng.xin/zx_md/20190711144820.png)

> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> select max(id) as maxid from default.test_user2019
> ```

![](http://pic.carryzheng.xin/zx_md/20190711145744.png)
> 其中必填项:
>
> * **Database Connection URL**
>
> ```properties
> jdbc:hive2://10.0.31.239:10000
> ```
>
> * **Hive Configuration Resources**
>
> ```properties
> /usr/hdp/2.6.4.0-91/hive2/conf/hive-site.xml
> ```
>
> * **Database User**
>
> ```properties
> hive
> ```

#### 3. 将查询出的数据类型从avro转成json(配置ConvertAvroToJSON Processor)

![](http://pic.carryzheng.xin/zx_md/20190711150153.png)

#### 4. 从json中提取该id(配置EvaluateJsonPath Processor)

![](http://pic.carryzheng.xin/zx_md/20190711150259.png)

> 其中，需添加属性maxid，$.maxid命名与json字符串中一致。

#### 5. SelectDataFromMysql(配置ExecuteSQL Processor)

![](http://pic.carryzheng.xin/zx_md/20190711150630.png)

> 其中必填项:
>
> * **Database Connection Pooling Service**
> * **SQL select query**
>
> ```sql
> select id,name,age,create_time from test_user where id > ${maxid} and ${maxid} is not null
> ```



#### 6. InsertHive(配置PutHiveStreaming Processor)

![](http://pic.carryzheng.xin/zx_md/20190711150849.png)
> 其中必填项:
>
> * **Hive Metastore URI**
>
> ```properties
> thrift://10.0.31.239:9083
> ```
>
> * **Hive Configuration Resources**
>
> ```properties
> /usr/hdp/2.6.4.0-91/hive2/conf/hive-site.xml
> ```
>
> * **Database Name**
>
> ```properties
> default
> ```
>
> * **Table Name**
>
> ```properties
> test_user2019
> ```



### 三、分区

#### 0. 整体流程图

![](http://pic.carryzheng.xin/zx_md/20190711151320.png)

#### 1. 流程说明

* 清空分区表数据
* 从原始表导入到分区表

#### 2. TruncateTable(配置ExecuteSQL Processor)

![](http://pic.carryzheng.xin/zx_md/20190711181214.png)

> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> alter table test_user2019a drop if exists partition(create_time='20190707')
> ```

#### 3. TablePartition(配置ExecuteSQL Processor)

![](http://pic.carryzheng.xin/zx_md/20190711181245.png)

> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> insert into table test_user2019a partition(create_time) select id, name, age, create_time from test_user2019 where create_time = '20190707'
> ```



### 四、注意事项

1. 定时调度(Scheduling)

* CORN表达式和Quartz一样，最小单位为秒。

2. hive需开启非严格模式和动态分区表

```shell
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.dynamic.partition=true;
```





### 五、附录

#### 1. MySQL建表语句

```sql
create table test_user
(
	id int auto_increment
		primary key,
	name varchar(20) null,
	age int null,
	create_time varchar(8) null
)
engine=InnoDB;
```



#### 2. Hive建表语句

```sql
create table test_user2019(
id int,
name string,
age int,
create_time string
)
clustered by (id) into 3 buckets
row format delimited
fields terminated by '\t'
stored as orc
tblproperties('transactional'='true');
```



#### 3. Hive分区表建表语句

```sql
create table test_user2019a(
id int,
name string,
age int
)
partitioned by(create_time string)
clustered by (id) into 3 buckets
row format delimited
fields terminated by '\t'
stored as orc
tblproperties('transactional'='true');
```