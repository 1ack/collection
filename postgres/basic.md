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

每列数据单行展示
```
postgres=# \x
Expanded display is on.
postgres=# select * from oc_activity;
-[ RECORD 1 ]-+------------------------------------------------------------------------------------------------------------------------------------------------
activity_id   | 1
timestamp     | 1597145540
priority      | 30
type          | file_created
user          | 18800200553
affecteduser  | 18800200553
app           | files
subject       | created_self
subjectparams | [{"6":"\/Documents"}]
message       | 
messageparams | []
file          | /Documents
link          | http://119.45.227.222:8000/index.php/apps/files/?dir=/
object_type   | files
object_id     | 6
-[ RECORD 2 ]-+------------------------------------------------------------------------------------------------------------------------------------------------
activity_id   | 2
timestamp     | 1597145540
priority      | 30
type          | file_created
user          | 18800200553
affecteduser  | 18800200553
app           | files
subject       | created_self
subjectparams | [{"7":"\/Documents\/Welcome to Nextcloud Hub.docx"}]
message       | 
messageparams | []
file          | /Documents/Welcome to Nextcloud Hub.docx
link          | http://119.45.227.222:8000/index.php/apps/files/?dir=/Documents
object_type   | files
object_id     | 7
```

## postgres 数据类型
### JSON和JSONB
PostgreSQL支持丰富的NOSQL特性。
PostgreSQL支持两种JSON数据类型：json和jsonb，两种类型在使用上几乎完全相同，两者主要区别为以下：json存储格式为文本而jsonb存储格式为二进制 ，由于存储格式的不同使得两种json数据类型的处理效率不一样，json类型以文本存储并且存储的内容和输入数据一样，当检索json数据时必须重新解析，而jsonb以二进制形式存储已解析好的数据，当检索jsonb数据时不需要重新解析，因此json写入比jsonb快，但检索比jsonb慢。

操作符 #> 返回 json 数据字段指定的元素   
```
postgres=# select '{"a":[1,2,3],"b":[4,5,6]}'::json#>'{b,1}';
 ?column? 
----------
 5
(1 row)

```

通过 jsonb || jsonb (concatenate / overwrite) 操作符可以覆盖元素值
```
postgres=# select '{"name":"francs","age":"31"}'::jsonb || '{"age":"32"}'::jsonb; 
            ?column?             
---------------------------------
 {"age": "32", "name": "francs"}
(1 row)
```

通过操作符删除元素
```
postgres=# SELECT '{"name": "James", "email": "james@localhost"}'::jsonb - 'email';
     ?column?      
-------------------
 {"name": "James"}
(1 row)
```
### 复合类型
![复合类型](images/pg_type.png)

### 数组类型
```
postgres=# create table test_array(id serial primary key, phone int8[]);  
CREATE TABLE
postgres=# \d test_array
                          Table "public.test_array"
 Column |   Type   |                        Modifiers                        
--------+----------+---------------------------------------------------------
 id     | integer  | not null default nextval('test_array_id_seq'::regclass)
 phone  | bigint[] | 
Indexes:
    "test_array_pkey" PRIMARY KEY, btree (id)

postgres=# insert into test_array(phone) values ('{1,2}');
INSERT 0 1
postgres=# insert into test_array(phone) values (array[3,4,5]); 
INSERT 0 1
postgres=# select * From test_array; 
 id |  phone  
----+---------
  1 | {1,2}
  2 | {3,4,5}
(2 rows)

postgres=#  select phone[1],phone[2] from test_array where id=1;  
 phone | phone 
-------+-------
     1 |     2
(1 row)
```
数组操作符
![数组操作符](images/pg_array.png)

## PostgreSQL的模式、表空间、用户间的关系
### 模式
PostgreSQL 模式（SCHEMA）可以看着是一个表的集合。

一个模式可以包含视图、索引、据类型、函数和操作符等。

相同的对象名称可以被用于不同的模式中而不会出现冲突，例如 schema1 和 myschema 都可以包含名为 mytable 的表。

使用模式的优势：

* 允许多个用户使用一个数据库并且不会互相干扰。

* 将数据库对象组织成逻辑组以便更容易管理。

* 第三方应用的对象可以放在独立的模式中，这样它们就不会与其他对象的名称发生冲突。

通常情况下，创建和访问表时，如果不知道模式，默认是访问 public 模式。

可以针对模式进行权限控制。

### 表空间
表空间是实际的数据存储的地方。一个数据库schema可能存在于多个表空间，相似地，一个表空间也可以为多个schema服务。一个表空间可以让多个数据库使用，而一个数据库也可以使用多个表空间。

通过使用表空间，管理员可以控制磁盘的布局。表空间的最常用的作用是优化性能，例如，一个最常用的索引可以建立在非常快的硬盘上，而不太常用的表可以建立在便宜的硬盘上，比如用来存储用于进行归档文件的表。

postgres 自带了两个表空间，pg_default， pg_global

表空间pg_default是用来存储系统目录对象、用户表、用户表index、和临时表、临时表index、内部临时表的默认空间。对应存储目录$PADATA/base/

表空间pg_global用来存放系统字典表；对应存储目录$PADATA/global/

```
postgres=# CREATE TABLESPACE tsp OWNER postgres LOCATION '/var/lib/postgresql/tsp';
CREATE TABLESPACE
postgres=# CREATE DATABASE dbtsp OWNER postgres TEMPLATE template1 TABLESPACE tsp;
CREATE DATABASE
postgres=# \l
                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |     Access privileges     
-----------+----------+----------+------------+------------+---------------------------
 dbtsp     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | postgres=CTc/postgres    +
           |          |          |            |            | oc_18800200553=c/postgres
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres              +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \db
               List of tablespaces
    Name    |  Owner   |        Location         
------------+----------+-------------------------
 pg_default | postgres | 
 pg_global  | postgres | 
 tsp        | postgres | /var/lib/postgresql/tsp
(3 rows)
```

### 角色和用户的关系
CREATE USER除了默认具有LOGIN权限之外，其他与CREATE ROLE是完全相同的
```
postgres=# CREATE ROLE custom PASSWORD 'custom';
CREATE ROLE
postgres=# CREATE USER guest PASSWORD 'guest';
CREATE ROLE
postgres=# \du
                                      List of roles
   Role name    |                         Attributes                         | Member of 
----------------+------------------------------------------------------------+-----------
 custom         | Cannot login                                               | {}
 guest          |                                                            | {}
 oc_18800200553 | Create DB                                                  | {}
 postgres       | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 replica        | Replication                                                | {}

```
## PostgreSQL TOAST 技术
[PostgreSQL TOAST 技术理解](https://cloud.tencent.com/developer/article/1004455)

PG 不允许一行数据跨页存储，对于超长的行数据，PG 会启动 TOAST ，具体就是采用压缩和切片的方式。如果启用了切片，实际数据存储在另一张系统表的多个行中，这张表就叫 TOAST 表，这种存储方式叫行外存储。

四种 TOAST 的策略：

* PLAIN ：避免压缩和行外存储。只有那些不需要 TOAST 策略就能存放的数据类型允许选择（例如 int 类型），而对于 text 这类要求存储长度超过页大小的类型，是不允许采用此策略的
* EXTENDED ：允许压缩和行外存储。一般会先压缩，如果还是太大，就会行外存储
* EXTERNA ：允许行外存储，但不许压缩。类似字符串这种会对数据的一部分进行操作的字段，采用此策略可能获得更高的性能，因为不需要读取出整行数据再解压。
* MAIN ：允许压缩，但不许行外存储。不过实际上，为了保证过大数据的存储，行外存储在其它方式（例如压缩）都无法满足需求的情况下，作为最后手段还是会被启动。因此理解为：尽量不使用行外存储更贴切。

```
postgres=# create table blog(id int, title text, content text);
CREATE TABLE
postgres=# \d+ blog;
                          Table "public.blog"
 Column  |  Type   | Modifiers | Storage  | Stats target | Description 
---------+---------+-----------+----------+--------------+-------------
 id      | integer |           | plain    |              | 
 title   | text    |           | extended |              | 
 content | text    |           | extended |              | 
```


## postgres 主备配置

## postgres docker部署

## postgres 索引