### 一、初始化(第一次将mysql全量写入hive)

#### 0. 整体流程图 

![image](https://upload-images.jianshu.io/upload_images/17802167-bee6d82634de685a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1. 流程说明

* 从mysql中查询出数据
* 写入到hive

#### 2. 查询mysql数据库(配置ExecuteSQL Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-ba3c109af46aeff3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其中必填项:
> * **Database Connection URL**
> * **SQL select query**
> ```sql
> select * from test_user
> ```



![image](https://upload-images.jianshu.io/upload_images/17802167-34e7a4c221b07647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

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

![image](https://upload-images.jianshu.io/upload_images/17802167-a442e41631a7d119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

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

![image](https://upload-images.jianshu.io/upload_images/17802167-2b8659364cd775e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1. 流程说明

* 先从hive中查询当前最大id
* 将查询出的数据类型从avro转成json
* 从json中提取该id
* 根据id从mysql查出未写入到hive的数据
* 写入hive

#### 2. SelectMaxIdFromHive(配置SelectHiveQL Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-1a2e7e864424b6e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> select max(id) as maxid from default.test_user2019
> ```

![image](https://upload-images.jianshu.io/upload_images/17802167-f1f7c89b1e644444.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
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

![image](https://upload-images.jianshu.io/upload_images/17802167-05d73c3e13b18ea7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4. 从json中提取该id(配置EvaluateJsonPath Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-4bffc0b86f49e39d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其中，需添加属性maxid，$.maxid命名与json字符串中一致。

#### 5. SelectDataFromMysql(配置ExecuteSQL Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-1343a6d15c3b6794.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其中必填项:
>
> * **Database Connection Pooling Service**
> * **SQL select query**
>
> ```sql
> select id,name,age,create_time from test_user where id > ${maxid} and ${maxid} is not null
> ```



#### 6. InsertHive(配置PutHiveStreaming Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-2301fb7c0929cfa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
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

![image](https://upload-images.jianshu.io/upload_images/17802167-755bd7b12d2e78a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1. 流程说明

* 清空分区表数据
* 从原始表导入到分区表

#### 2. TruncateTable(配置ExecuteSQL Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-a741c7542cd2fd4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> truncate table default.test_user2019a
> ```

#### 3. TablePartition(配置ExecuteSQL Processor)

![image](https://upload-images.jianshu.io/upload_images/17802167-f4ca0c064be144d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 其中必填项:
> * **Hive Database Connection Pooling Service**
> * **HiveQL Select Query**
> ```sql
> insert into table test_user2019a partition(create_time) select id, name, age, create_time from test_user2019
> ```



### 四、注意事项

1. 定时调度(Scheduling)

* CORN表达式和Quartz一样，最小单位为秒。



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
