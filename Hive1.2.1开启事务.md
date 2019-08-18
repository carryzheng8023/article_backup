### Hive1.2.1开启事务



#### 一、在hive-site.xml中开启添加如下配置

```xml
<property>
  <name>hive.support.concurrency</name>
  <value>true</value>
</property>
  <property>
  <name>hive.enforce.bucketing</name>
  <value>true</value>
</property>
  <property>
  <name>hive.exec.dynamic.partition.mode</name>
  <value>nonstrict</value>
</property>
<property>
  <name>hive.txn.manager</name>
  <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
  <property>
  <name>hive.compactor.initiator.on</name>
  <value>true</value>
</property>
<property>
  <name>hive.compactor.worker.threads</name>
  <value>1</value>
</property>
```



#### 二、进行事务操作

```sql
create table tm(id int,name string) clustered by (id) into 3 buckets
row format delimited
fields terminated by '\t'
lines terminated by '\n'
stored as orc
tblproperties('transactional'='true');

insert into tm values(1,'aa');
update tm set name = 'bb' where id = 1;
delete from tm where id = 1;

ALTER TABLE tm COMPACT 'minor'; -- 压缩delta
ALTER TABLE tm COMPACT 'major'; -- 合并base
```



#### 三、遇到的问题(各种表不存在)

```
ERROR [main]: metastore.RetryingHMSHandler (RetryingHMSHandler.java:invoke(159)) - MetaException(message:Unable to update transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.TXNS' doesn't exist
```

```sql
create table TXNS
(
  TXN_ID bigint not null
    primary key,
  TXN_STATE char not null,
  TXN_STARTED bigint not null,
  TXN_LAST_HEARTBEAT bigint not null,
  TXN_USER varchar(128) not null,
  TXN_HOST varchar(128) not null,
  TXN_AGENT_INFO varchar(128) null,
  TXN_META_INFO varchar(128) null,
  TXN_HEARTBEAT_COUNT int null
)
engine=InnoDB;
```

```
Caused by: MetaException(message:Unable to select from transaction database, com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.NEXT_TXN_ID' doesn't exist
```

```sql
create table NEXT_TXN_ID
(
  NTXN_NEXT bigint not null
)
engine=InnoDB;

insert into NEXT_TXN_ID(NTXN_NEXT) values(1);
```

```
Caused by: MetaException(message:Unable to update transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.HIVE_LOCKS' doesn't exist
```

```sql
create table HIVE_LOCKS
(
  HL_LOCK_EXT_ID bigint not null,
  HL_LOCK_INT_ID bigint not null,
  HL_TXNID bigint null,
  HL_DB varchar(128) not null,
  HL_TABLE varchar(128) null,
  HL_PARTITION varchar(767) null,
  HL_LOCK_STATE char not null,
  HL_LOCK_TYPE char not null,
  HL_LAST_HEARTBEAT bigint not null,
  HL_ACQUIRED_AT bigint null,
  HL_USER varchar(128) not null,
  HL_HOST varchar(128) not null,
  HL_HEARTBEAT_COUNT int null,
  HL_AGENT_INFO varchar(128) null,
  HL_BLOCKEDBY_EXT_ID bigint null,
  HL_BLOCKEDBY_INT_ID bigint null,
  primary key (HL_LOCK_EXT_ID, HL_LOCK_INT_ID)
)
engine=InnoDB;

create index HIVE_LOCK_TXNID_INDEX
  on HIVE_LOCKS (HL_TXNID)
;

create index HL_TXNID_IDX
  on HIVE_LOCKS (HL_TXNID)
;
```

```
Caused by: MetaException(message:Unable to update transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.NEXT_LOCK_ID' doesn't exist
```

```sql
create table NEXT_LOCK_ID
(
  NL_NEXT bigint not null
)
engine=InnoDB;
insert into NEXT_LOCK_ID(NL_NEXT) values (1);
```

```
Caused by: MetaException(message:Unable to update transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.TXN_COMPONENTS' doesn't exist
```

```sql
create table TXN_COMPONENTS
(
  TC_TXNID bigint not null,
  TC_DATABASE varchar(128) not null,
  TC_TABLE varchar(128) not null,
  TC_PARTITION varchar(767) null,
  TC_OPERATION_TYPE char null,
  constraint txn_components_ibfk_1
    foreign key (TC_TXNID) references TXNS (TXN_ID)
)
engine=InnoDB
;

create index TC_TXNID
  on TXN_COMPONENTS (TC_TXNID)
;
```

```
Caused by: MetaException(message:Unable to update transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.COMPLETED_TXN_COMPONENTS' doesn't exist
```

```sql
create table COMPLETED_TXN_COMPONENTS
(
  CTC_TXNID bigint not null,
  CTC_DATABASE varchar(128) not null,
  CTC_TABLE varchar(128) null,
  CTC_PARTITION varchar(767) null
)
engine=InnoDB;
```


```
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:Unable to select from transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.NEXT_COMPACTION_QUEUE_ID' doesn't exist
```

```sql
create table NEXT_COMPACTION_QUEUE_ID
(
  NCQ_NEXT bigint not null
)
engine=InnoDB;
insert into NEXT_COMPACTION_QUEUE_ID values(1);
```



```
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:Unable to select from transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive_metadata.COMPACTION_QUEUE' doesn't exist
```

```sql
create table COMPACTION_QUEUE
(
  CQ_ID bigint not null
    primary key,
  CQ_DATABASE varchar(128) not null,
  CQ_TABLE varchar(128) not null,
  CQ_PARTITION varchar(767) null,
  CQ_STATE char not null,
  CQ_TYPE char not null,
  CQ_TBLPROPERTIES varchar(2048) null,
  CQ_WORKER_ID varchar(128) null,
  CQ_START bigint null,
  CQ_RUN_AS varchar(128) null,
  CQ_HIGHEST_TXN_ID bigint null,
  CQ_META_INFO varbinary(2048) null,
  CQ_HADOOP_JOB_ID varchar(32) null
)
engine=InnoDB
;
```

**注：**如果你是报一次错才创建一个表，请在所有表创建完后将表中的数据清空，否则会卡死并得到以下错误

```
[main]: lockmgr.DbLockManager (DbLockManager.java:lock(85)) - Response to queryId=hadoop_20190813061155_57cfa30c-b1f5-4f47-89d7-87f8f8615aa9 LockResponse(lockid:8, state:WAITING)

Caused by: MetaException(message:Unable to select from transaction database com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Duplicate entry '1' for key 'PRIMARY'
```