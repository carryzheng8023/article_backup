## nifi压力测试

### 一、测试要求

> 1. 设计ta、tb、tc、td、te五张表，每张表不少于10个字段，包括整型、字符串、日期等类型；
> 2. 将ta同步到tc、tb同步到td；
> 3. 用nifi调用jar执行ta join tb，结果插入到te；
> 4. 更新td符合要求的100条记录；
> 5. ta表5000条记录、tb表10000条记录；
> 6. 以上为1份作业，copy300份同时运行。



### 二、准备工作

#### 设置nifi内存

```shell
vim $NIFI_HOME/conf/bootstrap.conf
```

```properties
java.arg.2=-Xms8192m
java.arg.3=-Xmx8192m
```

#### 设置执行jar时jvm内存

```shell
$JAVA_HOME/bin/java -Xms8m -Xmx8m -jar $JAR_PATH
```

#### SQL


```sql
# 1. 建库
create database if not exists test_nifi;

# 2. 建表
use test_nifi;

create table ta(
  id int not null primary key auto_increment,
  join_key_a varchar(255),
  join_key_b varchar(255),
  join_key_c int,
  filter_col_a varchar(255),
  filter_col_b int,
  filter_col_c bigint,
  create_time timestamp,
  col_a int,
  col_b char(10),
  col_c bigint,
  col_d varchar(30)
);

create table tb(
  id int not null primary key auto_increment,
  join_key_a varchar(255),
  join_key_b varchar(255),
  join_key_c int,
  filter_col_a varchar(255),
  filter_col_b int,
  filter_col_c bigint,
  create_time timestamp,
  col_a int,
  col_b char(10),
  col_c bigint,
  col_d varchar(30)
);

create table tc(
  join_key_a varchar(255),
  join_key_b varchar(255),
  join_key_c int,
  filter_col_a varchar(255),
  filter_col_b int,
  filter_col_c bigint,
  create_time timestamp,
  col_a int,
  col_b char(10),
  col_c bigint,
  col_d varchar(30)
);

create table td(
  join_key_a varchar(255),
  join_key_b varchar(255),
  join_key_c int,
  filter_col_a varchar(255),
  filter_col_b int,
  filter_col_c bigint,
  create_time timestamp,
  col_a int,
  col_b char(10),
  col_c bigint,
  col_d varchar(30)
);

create table te(

  col1 varchar(255),
  col2 varchar(255),
  col3 int,
  col4 varchar(255),
  col5 int,
  col6 bigint,
  col7 timestamp,
  col8 int,
  col9 char(10),
  col10 bigint,
  col11 varchar(30)

);


# 3. 数据插入
delimiter //
create procedure batchInsertTA1()
begin
    declare num int;
    set num=1;
    while num<=1000 do
        insert into ta(join_key_a,join_key_b,join_key_c, filter_col_a, filter_col_b, filter_col_c, create_time, col_a, col_b, col_c, col_d)
        values(concat('join_key_a', num), concat('join_key_b', num), num, concat('filter_col_a', num), num, num, now(), 1, '1', 1, '1');
        set num=num+1;
    end while;
end
//
delimiter ;

delimiter //
create procedure batchInsertTA2()
begin
    declare num int;
    set num=1;
    while num<=9000 do
        insert into ta(join_key_a,join_key_b,join_key_c, filter_col_a, filter_col_b, filter_col_c, create_time, col_a, col_b, col_c, col_d)
        values('join_key_a1000000', 'join_key_b1000000', 1000000, 'filter_col_a1000000', 1000000, 1000000, now(), 1000000, '1000000', 1000000, '1000000');
        set num=num+1;
    end while;
end
//
delimiter ; 

delimiter //
create procedure batchInsertTB1()
begin
    declare num int;
    set num=1;
    while num<=1000 do
        insert into tb(join_key_a,join_key_b,join_key_c, filter_col_a, filter_col_b, filter_col_c, create_time, col_a, col_b, col_c, col_d)
        values(concat('join_key_a', num), concat('join_key_b', num), num, concat('filter_col_a', num), num, num, now(), 2, '2', 2, '2');
        set num=num+1;
    end while;
end
//
delimiter ; 

delimiter //
create procedure batchInsertTB2()
begin
    declare num int;
    set num=1;
    while num<=49000 do
        insert into tb(join_key_a,join_key_b,join_key_c, filter_col_a, filter_col_b, filter_col_c, create_time, col_a, col_b, col_c, col_d)
        values('join_key_a2000000', 'join_key_b2000000', 2000000, 'filter_col_a2000000', 2000000, 2000000, now(), 2000000, '2000000', 2000000, '2000000');
        set num=num+1;
    end while;
end
//
delimiter ; 

call batchInsertTA1();
call batchInsertTA2();
call batchInsertTB1();
call batchInsertTB2();
```
#### nifi template

> 链接:https://pan.baidu.com/s/18-hcDdDTUMbVqFiCGs-azQ  密码:mii6

#### join code

> 链接:https://pan.baidu.com/s/1tdri8nsT82T7xmLz6p4i-Q  密码:jz0t



### 三、测试结果

|              | 100份作业 | 200份作业 | 300份作业 |
| ------------ | --------- | --------- | --------- |
| 4核+8G(11G)  | 15分钟    | 18分钟    | \         |
| 6核+16G(32G) | 9分钟     | 11分钟    | 62分钟    |



### 四、附录A：运行环境

1. 配置一：
```properties
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             4
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz
Stepping:              4
CPU MHz:               2299.340
BogoMIPS:              4600.00
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              16896K
NUMA node0 CPU(s):     0-3
```
```properties
              total        used        free      shared  buff/cache   available
Mem:          11855        5526        2884         297        3443        5664
Swap:          4095          59        4036
```
2. 配置二：

```properties
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                6
On-line CPU(s) list:   0-5
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             6
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz
Stepping:              4
CPU MHz:               2299.134
BogoMIPS:              4600.00
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              16896K
NUMA node0 CPU(s):     0-5
```

```properties
              total        used        free      shared  buff/cache   available
Mem:          32013       18762        2669          39       10580       12761
Swap:          4095          37        4058
```




3. Software Version

```properties
MySQL: 5.7.27
Nifi: 1.9.2
```


