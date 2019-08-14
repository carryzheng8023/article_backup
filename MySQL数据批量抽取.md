## MySQL数据批量抽取

#### 1.新建QueryDatabaseTable Processor

![](http://pic.carryzheng.xin/zx_md/20190712153551.png)



#### 2.填写配置信息

![](http://pic.carryzheng.xin/zx_md/20190712153713.png)

> 其中：
>
> * **Database Connection Pooling Service：** 数据库连接信息
> * **Database Type：**数据库类型
> * **Table Name：**表名
> * **Custom Query：**查询语句
> * <font color=#FF0000 >**Fetch Size：**</font>从MySQL数据库一次查出的最大条数(相当于在查询语句后加了limit n)
> * <font color=#FF0000 >**Max Rows Per Flow File：**</font>一个FlowFile中最大存放记录数