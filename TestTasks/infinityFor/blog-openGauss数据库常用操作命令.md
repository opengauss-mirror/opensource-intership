# openGauss数据库常用操作命令
## 1. 以操作系统用户omm登录数据库主节点
```c
su - omm
```
### 1.1 启动服务
```c
分布式openGauss：
gs_om -t start    启动服务
gs_om -t restart  重启服务
集中式openGauss：
gs_om -t stop   关闭服务
gs_om -t start  启动服务
```
### 1.2 使用“gs_om -t status –detail”命令查询openGauss各实例状态情况
```c
gs_om -t status --detail
```
如下部署了集中式openGauss集群，数据库主节点实例的服务器IP地址为172.20.73.178

数据库主节点数据路径为/opt/gaussdb/master1/

集中式没有CN，通过主DN访问

```c
[omm@openGauss01 ~]$ gs_om -t status --detail
[  CMServer State   ]

node           node_ip         instance                             state
---------------------------------------------------------------------------
2  openGauss02 172.20.73.179   1    /opt/gaussdb/cmserver/cm_server Standby
3  openGauss03 172.20.74.210   2    /opt/gaussdb/cmserver/cm_server Primary

[    ETCD State     ]

node           node_ip         instance                       state
---------------------------------------------------------------------------
1  openGauss01 172.20.73.178   7001 /opt/huawei/xuanyuan/etcd StateFollower
2  openGauss02 172.20.73.179   7002 /opt/huawei/xuanyuan/etcd StateLeader
3  openGauss03 172.20.74.210   7003 /opt/huawei/xuanyuan/etcd StateFollower

[   Cluster State   ]

cluster_state   : Normal
redistributing  : No
balanced        : Yes
current_az      : AZ_ALL

[  Datanode State   ]

node           node_ip         instance                   state            | node           node_ip         instance                   state            | node           node_ip         instance                   state
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1  openGauss01 172.20.73.178   6001 /opt/gaussdb/master1  P Primary Normal | 2  openGauss02 172.20.73.179   6002 /opt/gaussdb/slave1_1 S Standby Normal | 3  openGauss03 172.20.74.210   6003 /opt/gaussdb/slave1_2 S Standby Normal
[omm@openGauss01 ~]$
```

### 1.3 检查数据库性能
```c
gs_checkperf

1. 以简要格式在屏幕上显示性能统计结果。
gs_checkperf -i pmk -U omm 
2. 以详细格式在屏幕上显示性能统计结果。
gs_checkperf -i pmk -U omm --detai
```

### 1.4 确认数据库主节点的端口号

在1.2查到的数据库主节点数据路径下的postgresql.conf文件中查看端口号信息。示例如下：
cat /opt/gaussdb/master1/postgresql.conf |grep port
```c
[omm@openGauss01 ~]$ cat /opt/gaussdb/master1/postgresql.conf |grep port
port = '36000'                          # (change requires restart)
#ssl_renegotiation_limit = 0            # amount of data between renegotiations, no longer supported
                                        # supported by the operating system:
```
36000为数据库主节点的端口号

端口号在安装数据库时，会在xml文件中配置，查看安装时的xml配置文件也可以找到端口。

### 1.5列出所有可用的数据库
```c
gsql -d postgres -p 36000 -l
[omm@openGauss01 ~]$ gsql -d postgres -p 36000 -l
                          List of databases
   Name    | Owner | Encoding  | Collate | Ctype | Access privileges
-----------+-------+-----------+---------+-------+-------------------
 db1       | song  | SQL_ASCII | C       | C     |
 db2       | song  | SQL_ASCII | C       | C     |
 kwdb      | kw    | SQL_ASCII | C       | C     |
 mydb      | song  | GBK       | C       | C     |
 postgres  | omm   | SQL_ASCII | C       | C     |
 song_suse | song  | SQL_ASCII | C       | C     |
 template0 | omm   | SQL_ASCII | C       | C     | =c/omm           +
           |       |           |         |       | omm=CTc/omm
 template1 | omm   | SQL_ASCII | C       | C     | =c/omm           +
           |       |           |         |       | omm=CTc/omm
(8 rows)

```
其中，postgres为openGauss安装完成后默认生成的数据库。初始可以连接到此数据库进行新数据库的创建。

## 2. 查看数据库对象
```c
1. 登陆默认数据库postgres：
gsql -d postgres -p 36000
[omm@openGauss01 ~]$ gsql -d postgres -p 36000
gsql ((GaussDB Kernel V500R002C00 build fab4f5ea) compiled at 2021-10-24 11:58:09 commit 3086 last mr 6592 release)
Non-SSL connection (SSL connection is recommended when requiring high-security)
Type "help" for help.

openGauss=#
2. 登陆自建数据库song_suse:
gsql -d 数据库名 -p 36000 -U 用户名 -W 密码  -r
[omm@openGauss01 ~]$ gsql -d song_suse -p 36000 -U song -W Info1234  -r
gsql ((GaussDB Kernel V500R002C00 build fab4f5ea) compiled at 2021-10-24 11:58:09 commit 3086 last mr 6592 release)
Non-SSL connection (SSL connection is recommended when requiring high-security)
Type "help" for help.

song_suse=>
```
### 1）查看帮助信息：
```c
postgres=# \?
```
### 2）切换数据库：
```c
postgres=# \c dbname
```
### 3）列举数据库：
```c
使用\l元命令查看数据库系统的数据库列表。
postgres=# \l
使用如下命令通过系统表pg_database查询数据库列表。
postgres=# select dataname from pg_database;
```
### 4)列举表：
```c
postgres=# \dt
postgres=# \d
```
### 5)列举所有表、视图和索引：
```c
postgres=# \d+
```
### 6）使用gsql的\d+命令查询表的属性：
```c
postgres=# \d+ tablename
```
### 7）查看表结构：
```c
postgres=# \d tablename
```
### 8）列举schema：
```c
postgres=# \dn
```
### 9）查看索引：
```c
postgres=# \di
```
### 10）查询表空间：
```c
使用gsql程序的元命令查询表空间。postgres=# \db
检查pg_tablespace系统表。如下命令可查到系统和用户定义的全部表空间。
postgres=# select spcname from pg_tablespace;
```
### 11）查看数据库用户列表：
```c
postgres=# select * from pg_user;
```
### 12）要查看用户属性：
```c
postgres=# select * from pg_authid;
```
### 13）查看所有角色：
```c
postgres=# select * from PG_ROLES;
```
### 14）切换用户：
```c
postgres=# \c – username
```
### 15）退出数据库：
```c
postgres=# \q
```
## 3. 语言：

常用的sql语句在openGauss都是能够使用的，但查询表分布列仅在openGauss才能使用：
```c
select getdistributekey ('schemaName.tableName');
select getdistributekey ('tableName');
```