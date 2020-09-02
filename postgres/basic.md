# postgres knowledge
## postgres 的特点
* 开源
* 支持丰富的数据类型和自定义类型(JSON JSONB ARRAY)
* 提供丰富的接口，很容易扩展功能 (Extension)
* 支持使用流行的语言写自定义函数 (PL/Perl PL/Python PL/pgSQL)

[PostgreSQL优于其他开源数据库的特性：Part I](https://postgres.fun/20151209093052.html)

[PostgreSQL优于其他开源数据库的特性：Part II](https://postgres.fun/20151211151702.html)

## 架构
1.进程和内存结构图  
   
![进程和内存结构图](images/pg_arch.jpg)

![进程列表](images/pg_ps.jpg)

2.主进程  
postgres 常驻进程  
也成为Postmaster，是整个数据库实例的总控进程，负责启动和关闭该数据库实例。

默认监听UNIX Domain Socket和TCP/IP（Windows等一部分的平台只监听tcp/ip）的5432端口，等待来自前端的的连接处理。监听的端口号可以在PostgreSQL的设置文件postgresql.conf里面可以改。

一旦有前端连接过来，postgres会通过fork(2)生成子进程。没有Fork(2)的windows平台的话，则利用createProcess()生成新的进程。这种情形的话，和fork(2)不同的是，父进程的数据不会被继承过来，所以需要利用共享内存把父进程的数据继承过来。

postgres 子进程  
子进程根据pg_hba.conf定义的安全策略来判断是否允许进行连接，根据策略，会拒绝某些特定的IP及网络，或者也可以只允许某些特定的用户或者对某些数据库进行连接。

Postgres会接受前端过来的查询，然后对数据库进行检索，最好把结果返回，有时也会对数据库进行更新。更新的数据同时还会记录在事务日志里面（PostgreSQL称为WAL日志），这个主要是当停电的时候，服务器当机，重新启动的时候进行恢复处理的时候使用的。另外，把日志归档保存起来，可在需要进行恢复的时候使用。在PostgreSQL 9.0以后，通过把WAL日志传送其他的postgreSQL，可以实时得进行数据库复制，这就是所谓的‘数据库复制’功能。

3.Syslogger（系统日志）进程  
需要在Postgres.conf中logging_collection设置为on，此时主进程才会启动Syslogger辅助进程。

4.BgWriter(后台)进程  
把共享内存中的脏页写到磁盘上的进程。主要是为了提高插入、更新和删除数据的性能。

5.WalWrite（预写式日志）进程  
WAL：Write Ahead Log（预写式日志）  
在修改数据之前把修改操作记录到磁盘中，以便后面更新实时数据时就不需要数据持久化到文件中。

6.PgArch（归档）进程  
WAL日志会被循环使用，PgArch在归档前会把WAL日志备份出来。通过PITY（Point in Time Recovery）技术，可以对数据库进行一次全量备份后，该技术将备份时间点之后的WAL日志通过归档进行备份，使用数据库的全量备份再加上后面产生的WAL日志，即可把数据库向前推到全量备份后的任意一个时间点。

7.AutoVacuum（自动清理）进程  
在PostgreSQL数据库中，对表进行DELETE操作后，旧的数据并不会立即被删除，并且，在更新数据时，也并不会在旧的数据上做更新，而是新生成一行数据。旧的数据只是被标识为删除状态，只有在没有并发的其他事务读到这些就数据时，它们才会被清楚。这个清除工作就有AutoVacuum进程完成。

8.PgStat（统计数据收集）进程  
做数据的统计收集工作。主要用于查询优化时的代价估算，包括一个表和索引进行了多少次的插入、更新、删除操作，磁盘块读写的次数、行的读次数。pg_statistic中存储了PgStat收集的各类信息。

9.共享内存  
PostgreSQL启动后，会生成一块共享内存，用于做数据块的缓冲区，以便提高读写性能。WAL日志缓冲区和Clog缓冲区也存在共享内存中，除此之外还有全局信息比如进程、锁、全局统计等信息也保存在共享内存中。

使用"mmap()"方式的共享内存，使用此内存的好处是不在需要配置"System V"共享内存的参数"kernel.shmmax"和"kernel.shmall”，就能使用较大的共享内存。

10.本地内存  
非全局存储的数据都存在本地内存中，主要包括：

* 临时缓冲区：用于访问临时表的缓冲区

* work_mem: 内部排序操作和Hash表在使用临时操作文件之前使用的存储缓冲区。

* manintance_work_mem: 在维护操作比如：VACUUM、CREATE INDEX、ALTER TABLE ADD FOREIGN Key等中使用的内存缓冲区。


## VACUUM



## JSON和JSONB
PostgreSQL支持丰富的NOSQL特性。
PostgreSQL支持两种JSON数据类型：json和jsonb，两种类型在使用上几乎完全相同，两者主要区别为以下：json存储格式为文本而jsonb存储格式为二进制 ，由于存储格式的不同使得两种json数据类型的处理效率不一样，json类型以文本存储并且存储的内容和输入数据一样，当检索json数据时必须重新解析，而jsonb以二进制形式存储已解析好的数据，当检索jsonb数据时不需要重新解析，因此json写入比jsonb快，但检索比jsonb慢。
## extension
通过extension扩展能力，可以动态加载到系统空间
fdw系列插件，使得pg可以从任意数据库上读取数据（ORACLE,SQL SERVER,MYSQL,MONDODB)
[awesome-postgres]https://github.com/dhamaniasad/awesome-postgres

## postgres 常用命令
### 命令行操作
初始化数据库，可以指定默认的超级管理员名字，不指定时会使用当前的操作系统用户名 
```
initdb -D /opt/pgdata -U postgres 
```


### psql命令
执行psql命令连接数据库进入psql命令行，psql的命令都以 \ 开头

psql -l 可以查看有哪些数据库
```
postgres@9b3f17cb2d8c:~$ psql -l
                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |     Access privileges     
-----------+----------+----------+------------+------------+---------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | postgres=CTc/postgres    +
           |          |          |            |            | oc_18800200553=c/postgres
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

```

```
postgres=# \l
                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |     Access privileges     
-----------+----------+----------+------------+------------+---------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | postgres=CTc/postgres    +
           |          |          |            |            | oc_18800200553=c/postgres
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

```

切换到另一个数据库
```
postgres=# \c skylar
You are now connected to database "skylar" as user "insight".
```

显示当前数据库所有表
```
postgres=# \d
                             List of relations
 Schema |                 Name                 |   Type   |     Owner      
--------+--------------------------------------+----------+----------------
 public | oc_accounts                          | table    | oc_18800200553
 public | oc_activity                          | table    | oc_18800200553
 public | oc_activity_activity_id_seq          | sequence | oc_18800200553
 public | oc_activity_mq                       | table    | oc_18800200553
 public | oc_activity_mq_mail_id_seq           | sequence | oc_18800200553
 public | oc_addressbookchanges                | table    | oc_18800200553
 public | oc_addressbookchanges_id_seq         | sequence | oc_18800200553
 public | oc_addressbooks                      | table    | oc_18800200553
 public | oc_addressbooks_id_seq               | sequence | oc_18800200553
 public | oc_appconfig                         | table    | oc_18800200553
 public | oc_authtoken                         | table    | oc_18800200553
......
```

显示指定表的结构定义
```
postgres=# \d oc_accounts
                       Table "public.oc_accounts"
 Column |         Type          |               Modifiers                
--------+-----------------------+----------------------------------------
 uid    | character varying(64) | not null default ''::character varying
 data   | text                  | not null default ''::text
Indexes:
    "oc_accounts_pkey" PRIMARY KEY, btree (uid)

```

显示索引信息
```
postgres=# \d oc_accounts_pkey
       Index "public.oc_accounts_pkey"
 Column |         Type          | Definition 
--------+-----------------------+------------
 uid    | character varying(64) | uid
primary key, btree, for table "public.oc_accounts"

```



## postgres 主备配置

## postgres docker部署

## postgres 索引