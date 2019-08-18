#### 1. 问题描述
 * Spark2.2.0读取不到Hive1.2.1事务表数据(通过metastore)

#### 2. 原因

 * hive每次进行事务操作(增、删、改)时，会在hdfs表目录上新建一个以delta开头的目录，将增量数据写入该目录内，但spark不会处理只有增量数据的事务表（我认为这应该是spark的一个bug）。


#### 3. 解决方案

 * 进行一次Major Compactor
   ```sql
   ALTER TABLE table_name COMPACT 'major';
   ```
* 查看合并进度
   ```sql
   SHOW COMPACTIONS;
   ```

[reference](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-SHOWCOMPACTIONS)

