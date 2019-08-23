#### 1. 问题描述
 * Spark2.2.0读取不到Hive1.2.1事务表数据(通过metastore)

#### 2. 问题重现

 * 事务表数据准备

   * sql

     ```sql
     CREATE TABLE tm (
       id int,
       name string
     )
     CLUSTERED BY (id) INTO 2 BUCKETS STORED AS ORC
     TBLPROPERTIES ("transactional"="true");
     
     insert into tm values(1,'aa');
     ```
     
   * 数据展示
     ![](http://pic.carryzheng.xin/zx_md/20190823141723.png)

     ![](http://pic.carryzheng.xin/zx_md/20190823160726.png)

 * spark读取tm表中数据

    * 代码

   ```scala
   import org.apache.spark.sql.SparkSession
   /**
     * @author zhengxin
     *         2019-08-06 17:43:54
     */
   object HiveOnSpark {
   
     def main(args: Array[String]): Unit = {
       val spark = SparkSession.builder().enableHiveSupport()
         .appName("hiveOnSpark").master("local[2]").getOrCreate()
       spark.table("tm").printSchema()
       spark.table("tm").show
     }
   }
   ```
   
 * 结果
   
      ![](http://pic.carryzheng.xin/zx_md/20190823160925.png)


#### 3. 原因

 * hive每次进行事务操作(增、删、改)时，会在hdfs表目录上新建一个以delta开头的目录，将增量数据写入该目录内，但spark不会处理只有增量数据的事务表（我认为这应该是spark的一个bug）。


#### 4. 解决方案

 * 进行一次Major Compactor
   ```sql
   ALTER TABLE tm COMPACT 'major';
   ```
   
 * 查看合并进度
   ```sql
   SHOW COMPACTIONS;
   ```
   
 * 查看结果

    ![](http://pic.carryzheng.xin/zx_md/20190823161037.png)

    ![](http://pic.carryzheng.xin/zx_md/20190823161156.png)

    ![](http://pic.carryzheng.xin/zx_md/20190823161334.png)

[reference](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-SHOWCOMPACTIONS)

