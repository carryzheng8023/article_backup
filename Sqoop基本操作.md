## 一、MySql到HDFS
### 1.初始化MySql数据
```sql
create database company;
create table company.staff(id int(4) primary key not null auto_increment, name varchar(255), sex varchar(255));
insert into company.staff(name, sex) values('Thomas', 'Male');
insert into company.staff(name, sex) values('Catalina', 'FeMale');
```
### 2.导入数据
#### (1)全部导入
```shell
sqoop import \
--connect jdbc:mysql://hadoop01:3306/company \
--username root \
--password 123456 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"
```
#### (2)查询导入
```shell
sqoop import \
--connect jdbc:mysql://hadoop01:3306/company \
--username root \
--password 123456 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query 'select name,sex from staff where id <=1 and $CONDITIONS;'
```
#### (3)导入指定列
```shell
sqoop import \
--connect jdbc:mysql://hadoop01:3306/company \
--username root \
--password 123456 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--columns id,sex \
--table staff
```
#### (4)使用sqoop关键字筛选查询导入数据
```shell
sqoop import \
--connect jdbc:mysql://hadoop:3306/company \
--username root \
--password 123456 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--table staff \
--where "id=1"
```
<br/>
## 二、MySql到Hive
```shell
sqoop import \
--connect jdbc:mysql://hadoop01:3306/company \
--username root \
--password 123456 \
--table staff \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```
<br/>
## 三、HIVE/HDFS到Mysql
```shell
sqoop export \
--connect jdbc:mysql://hadoop01:3306/company \
--username root \
--password 123456 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"
```
<br/>
## 四、常用命令
序号|命令|类|说明
--|:--:|--:|--:
1|import|ImportTool|将数据导入到集群
2|export|ExportTool|将集群数据导出
3|codegen|CodeGenTool|获取数据库中某张表数据生成Java并打Jar包
4|create-hive-table|CreateHiveTableTool|创建Hive表
5|eval|EvalSqlTool|查看SQL执行结果
6|import-all-tables|ImportAllTablesTool|导入某个数据库下所有表到HDFS中
7|job|JobTool|用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务。
8|list-databases|ListDatabasesTool|列出所有数据库名
9|list-tables|ListTablesTool|列出某个数据库下所有表
10|merge|MergeTool|将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中。
11|metastore|MetastoreTool|记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为:~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改。
12|help|HelpTool|打印sqoop帮助信息
13|version|VersionTool|打印sqoop版本信息
<br/>
## 五、命令&参数
### 1.公用参数:数据库连接
序号|参数|说明
--|:--:|--:
1|--connect|连接关系型数据库的URL
2|--connection-manager|指定要使用的连接管理类
3|--driver|连接数据库的驱动类
4|--help|打印帮助信息
5|--password|连接数据库的密码
6|--username|连接数据库的用户名
7|--verbose|在控制台打印出详细信息

### 2.公用参数:import
序号|参数|说明
--|:--:|--:
1|--enclosed-by <char>|给字段值前后加上指定的字符
2|--escaped-by <char>|对字段中的双引号加转义符
3|--fields-terminated-by <char>|设定每个字段是以什么符号作为结束，默认为逗号。
4|--lines-terminated-by <char>|设定每行记录之间的分隔符，默认是\n
5|--mysql-delimiters|Mysql 默认的分隔符设置，字段之间以逗号分隔，行之间以\n 分隔，默认转义符是\，字段值以单引号包裹。
6|--optionally-enclosed-by <char>|给带有双引号或单引号的字段值前后加上指定字符

### 3.公用参数:export
序号|参数|说明
--|:--:|--:
1|--input-enclosed-by <char>|对字段值前后加上指定字符
2|--input-escaped-by <char>|对含有转移符的字段做转义处理
3|--input-fields-terminated-by <char>|字段之间的分隔符
4|--input-lines-terminated-by <char>|行之间的分隔符
5|--input-optionally-enclosed-by <char>|给带有双引号或单引号的字段前后加上指定字符

### 4.公用参数:hive
序号|参数|说明
--|:--:|--:
1|--input-enclosed-by <char>|用自定义的字符串替换掉数据中的\r\n和\013\010等字符
2|--hive-drop-import-delims|在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符
3|--map-column-hive <map>|生成hive表时，可以更改生成字段的数据类型
4|--hive-partition-key|创建分区，后面直接跟分区名，分区字段的默认类型为string
5|--hive-partition-value <v>|导入数据时，指定某个分区的值
6|--hive-home <dir>|hive的安装目录，可以通过该参数覆盖之前默认配置的目录
7|--hive-import|将数据从关系数据库中导入到hive表中
8|--hive-overwrite|覆盖掉在hive表中已经存在的数据
9|--create-hive-table|默认是false，即：如果目标表已经存在了，那么创建任务失败。
10|--hive-table|后面接要创建的 hive 表,默认使用MySQL的表名
11|--table|指定关系数据库的表名

### 5.命令&参数:import
序号|参数|说明
--|:--:|--:
1|--append|将数据追加到HDFS中已经存在的DataSet中，如果使用该参数，sqoop 会把数据先导入到临时文件目录，再合并。
2|--as-avrodatafile|将数据导入到一个avro数据文件中
3|--as-sequencefile|将数据导入到一个sequence文件中
4|--as-textfile|将数据导入到一个普通文本文件中
5|--boundary-query <statement>|边界查询，导入的数据为该参数的值(一条sql语句)所执行的结果区间内的数据。
6|--columns <col1, col2, col3>|指定要导入的字段
7|--direct|直接导入模式，使用的是关系数据库自带的导入导出工具，以便加快导入导出过程。
8|--direct-split-size|在使用上面direct直接导入的基础上，对导入的流按字节分块，即达到该阈值就产生一个新的文件。
9|--inline-lob-limit|设定大对象数据类型的最大值
10|--m 或--num-mappers|启动N个map来并行导入数据，默认4个。
11|--query 或--e <statement>|将查询结果的数据导入，使用时必须伴随参--target-dir，--hive-table，如果查询中有where 条件，则条件后必须加上$CONDITIONS关键字。
12|--split-by <column-name>|按照某一列来切分表的工作单元，不能与--autoreset-to-one-mapper连用。
13|--table <table-name>|关系数据库的表名
14|--target-dir <dir>|指定HDFS路径
15|--warehouse-dir <dir>|与14参数不能同时使用，导入数据到HDFS时指定的目录。
16|--where|从关系数据库导入数据时的查询条件
17|--z或--compress|允许压缩
18|--compression-codec|指定hadoop压缩编码类，默认为gzip
19|--null-string <null-string>|string类型的列如果null，替换为指定字符串。
20|--null-non-string <null-string>|非string类型的列如果null，替换为指定字符串。
21|--check-column <col>|作为增量导入判断的列名
22|--incremental <mode>|mode:append或lastmodified
23|--last-value <value>|指定某一个值，用于标记增量导入的位置。

### 6.命令&参数:export
序号|参数|说明
--|:--:|--:
1|--direct|利用数据库自带的导入导出工具，以便于提高效率。
2|--export-dir <dir>|存放数据的 HDFS 的源目录
3|--m 或--num-mappers <n>|启动N个map来并行导入数据，默认4个。
4|--table <table-name>|指定导出到哪个RDBMS中的表
5|--update-key <col-name>|对某一列的字段进行更新操作
6|--update-mode <mode>|updateonly或allowinsert(默认)
7|--input-null-string <null-string>|请参考import该类似参数说明
8|--input-null-non-string <null-string>|请参考import该类似参数说明
9|--staging-table <staging-table-name>|创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。
10|--clear-staging-table|如果第 9 个参数非空，则可以在导出操作执行前，清空临时事务结果表。

### 7.命令&参数:codegen
序号|参数|说明
--|:--:|--:
1|--bindir <dir>|指定生成的Java文件、编译成的class文件及将生成文件打包为jar的文件输出路径。
2|--class-name <name>|设定生成的Java文件指定的名称
3|--outdir <dir>|生成Java文件存放的路径
4|--package-name <name>|包名，如 com.z，就会生成com和z两级目录。
5|--input-null-non-string <null-str>|在生成的Java文件中，可以将 null 字符串或者不存在的字符串设置为想要设定的值(例如空字符串)。
6|--input-null-string <null-str>|将null字符串替换成想要替换的值(一般与5同时使用)。
7|--map-column-java <arg>|数据库字段在生成的Java文件中会映射成各种属性，且默认的数据类型与数据库类型保持对应关系。该参数可以改变默认类型，例如:--map-column-java id=long,name=String。
8|--null-non-string <null-str>|在生成Java文件时，可以将不存在或者null的字符串设置为其他值。
9|--null-string <null-str>|在生成Java文件时，将null字符串设置为其他值(一般与8 同时使用)。
10|--table <table-name>|对应关系数据库中的表名，生成的Java文件中的各个属性与该表的各个字段一一对应。

### 8.命令&参数:create-hive-table
序号|参数|说明
--|:--:|--:
1|--hive-home <dir>|Hive的安装目录，可以通过该参数覆盖掉默认的Hive目录。
2|--hive-overwrite|覆盖掉在Hive表中已经存在的数据
3|--create-hive-table|默认是false，如果目标表已经存在了，那么创建任务会失败。
4|--hive-table|后面接要创建的hive表
5|--table|指定关系数据库的表名

### 9.命令&参数:eval
序号|参数|说明
--|:--:|--:
1|--query或--e|后跟查询的SQL语句

### 10.命令&参数:job
序号|参数|说明
--|:--:|--:
1|--create <job-id>|创建一个job
2|--delete <job-id>|删除一个job
3|--exec <job-id>|执行一个job
4|--help|显示job帮助
5|--list|显示job列表
6|--meta-connect <jdbc-uri>|用来连接metastore服务
7|--show <job-id>|显示一个 job 的信息
8|--verbose|打印命令运行时的详细信息

### 11.命令&参数:merge
序号|参数|说明
--|:--:|--:
1|--new-data <path>|HDFS待合并的数据目录，合并后在新的数据集中保留。
2|--onto <path>|HDFS合并后，重复的部分在新的数据集中被覆盖。
3|--merge-key <col>|合并键，一般是主键ID。
4|--jar-file <file>|合并时引入的jar包，该jar包是通过Codegen工具生成的jar包。
5|--class-name <class>|对应的表名或对象名，该class 类是包含在 jar 包中的。
6|--target-dir <path>|合并后的数据在HDFS里存放的目录

### 12.命令&参数:metastore
序号|参数|说明
--|:--:|--:
1|--shutdown|关闭metastore
