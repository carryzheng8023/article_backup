#### 1、概述

> 窗口函数分为聚合函数和分析函数。分析函数用于计算基于组的某种聚合值，它和聚合函数的区别在于：对于每个组返回多行，而聚合函数对于每个组值返回一行。

#### 2、聚合函数

> count(column_name)：返回指定列的值的数目
> sum(column_name)：返回数值列的总数
> min(column_name)：返回一列中的最小值，null值不包括在计算中
> max(column_name)：返回一列中的最大值，null值不包括在计算中
> avg(column_name)：返回数值列的平均值，null值不包括在计算中

#### 3、分析函数

> row_number()：从1开始，按照顺序，生成分组内记录的序列
> rank()：生成数据项在分组中的排名，排名相等会在名次中留下空位
> dense_rank()：生成数据项在分组中的排名，排名相等会在名次中不会留下空位
> lag(col,n)：向前第n行数据
> lead(col,n)：向后第n行数据
> first_value(col)：第一行数据
> last_value(col)：最后一行数据
> ntile(n)：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，ntile返回此行所属组的编号。

#### 4、over()从句：用来添加条件

> distribute by：用于分区
> sort by：用于排序
> between row_a and row_b*：窗口取值范围
> current row：当前行
> n preceding：往前n行数据
> n following：往后n行数据
> unbounded：无界限，unbounded preceding表示从起点开始，unbounded following表示到终点

#### 5、示例

* 数据准备
   ```
   jack,2017-01-01,10
   tony,2017-01-02,15
   jack,2017-02-03,20
   tony,2017-02-04,43
   jack,2017-03-05,25
   jack,2017-01-07,62
   tony,2017-01-04,34
   mart,2017-05-07,45
   mart,2017-01-02,62
   neil,2017-06-06,72
   mart,2017-01-08,35
   neil,2017-03-09,64
   mart,2017-01-10,26
   neil,2017-04-22,45
   ```

   ```sql
   create table business(
   name string,
   orderdate string,
   cost int
   )
   row format delimited
   fields terminated by ',';
   ```

* 查询在2017年1月购买过的顾客总人数

   ```sql
select name, count(1) over() from business where substring(orderdate,1,7)='2017-01' group by name;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827115929.png)
   
* 查询顾客的购买明细及月购买总额
   ```sql
select *, sum(cost) over(partition by month(orderdate)) from business;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827120138.png)
* 将cost按照日期进行累加
   ```sql
select *, sum(cost) over(sort by orderdate rows between unbounded preceding and current row) from business;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827120304.png)
* 查询顾客上次的购买时间
   ```sql
select *, lag(orderdate, 1) over(distribute by name sort by orderdate) from business;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827120546.png)
* 查询前20%时间的订单信息
   ```sql
select * from(select name, orderdate, cost, ntile(5) over(sort by orderdate) gid from business) t where gid = 1;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827120717.png)

* 按照每个人的消费金额进行分组排序
   ```sql
select *,rank() over(partition by name order by cost desc) from business;
   ```
   ![](http://pic.carryzheng.xin/zx_md/20190827121203.png)