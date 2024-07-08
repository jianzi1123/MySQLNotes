### 第02章 MySQL的数据目录

#### **1. MySQL8的主要目录结构**

```shell
find / -name mysql
```

##### **1.1** **数据库文件的存放路径** 

MySQL数据库文件的存放路径：

```shell
/var/lib/mysql/
```

MySQL服务器程序在启动时会到文件系统的某个目录下加载一些文件，之后在运行过程中产生的数据也都会存储到这个目录下的某些文件中，这个目录就称为 **数据目录。**

MySQL把数据都存到哪个路径下呢？其实 数据目录 对应这一个系统变量 datadir，我们在使用客户端与服务器建立连接之后查看这个系统变量的值就可以了。

```mysql
mysql> show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.04 sec)
```

从结果中可以看出，在我的计算机上MySQL的数据目录就是 /var/lib/mysql/ 。

```mysql
show variables like 'datadir'; # /var/lib/mysql/
```

##### **1.2** **相关命令目录**

相关命令目录：/usr/bin（mysqladmin、mysqlbinlog、mysqldump等命令）和/usr/sbin。

![image-20240319173438503](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/image-20240319173438503.png)

##### **1.3** **配置文件目录**

**配置文件目录：/usr/share/mysql-8.0（命令及配置文件），/etc/mysql（如my.cnf）**

<img src="./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/image-20240319173642560.png" alt="image-20240319173642560" style="zoom:80%;" />

```mysql
[root@master mysql-8.3]# vim /etc/my.cnf
```



#### **2.** **数据库和文件系统的关系**

像InnoDB、MyISAM 这样的存储引擎都是把表存储在磁盘上的，操作系统用来管理磁盘的结构被称为文件系统，所以用专业一点的话来表述就是:像InnoDB、MyISAM 这样的存储引擎都是把表存储在文件系统上的。当我们想读取数据的时候，这些存储引擎会从文件系统中把数据读出来返回给我们，当我们想写入数据的时候,这些存储引擎会把这些数据又写回文件系统。本章学习一下InnoDB和MyISAM这两个存储引起的数据如何在文件系统中存储。

##### 2.1 查看默认数据库

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.04 sec)
```

可以看到有 4 个数据库是属于 MySQL 自带的系统数据库。

- mysql


MySQL 系统自带的核心数据库，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定 义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。

```mysql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
+-----------+------------------+
4 rows in set (0.00 sec)

mysql> select * from user/G;
```

- information_schema


MySQL 系统自带的数据库，这个数据库保存着 MySQL 服务器 维护的所有其他数据库的信息 ，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为 元数据 。在系统数据库 information_schema 中提供了一些以 innodb_sys 开头的表，用于表示内部系统表。

- performance_schema


MySQL 系统自带的数据库，这个数据库里主要保存 MySQL 服务器运行过程中的一些状态信息，可以用来 监控 MySQL 服务的各类性能指标 。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。

- sys


MySQL 系统自带的数据库，这个数据库主要是通过 视图 的形式把 information_schema 和 performance_schema 结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。

##### 2.2 数据库在文件系统中的表示

使用CREATE DATABASE 数据库名语句创建一个数据库的时候， 在文件系统上实际发生了什么呢?其实很简单,每个数据库都对应数据目录下的一个子目录，或者说对应一个文件夹，每当新建一 个数据库时, MySQL会帮我们做这两件事儿:

1. 在数据目录下创建一个和数据库名同名的子目录。
2. 在与该数据库名同名的子目录下创建一个名为 db.opt的文件(仅限MySQL5.7及之前版本)，这个文件中包含了该数据库的各种属性，比如该数据库的字符集和比较规则。（8.0版本没有这个dp.opt文件）



##### **2.3** **表在文件系统中的表示** 

###### **2.3.1 InnoDB存储引擎模式** ==直接看3图解==

**1.** **表结构**

为了保存表结构，`InnoDB`在`数据目录`下对应的数据库子目录下创建了一个专门用于`描述表结构的文件`

```
表名.frm
```

比方说我们在 `test`数据库下创建一个名为`test`的表

那在数据库 `test` 对应的子目录下就会创建一个名为 `test.frm` 的用于描述表结构的文件。`.frm` 文件的格式在不同的平台上都是相同的。这个后缀名为 `.frm` 是以 `二进制格式` 存储的，我们直接打开是乱码的

**2.** **表中数据和索引**

**① 系统表空间（system tablespace）**

默认情况下，InnoDB会在数据目录下创建一个名为`ibdata1`、大小为`12M`的`自拓展`文件，这个文件就是对应的`系统表空间`在文件系统上的表示。注意这个文件是 `自扩展文件` ，当不够用的时候它会自己增加文件大小。

当然，如果你想让系统表空间对应文件系统上多个实际文件，或者仅仅觉得原来的 `ibdata1` 这个文件名难听，那可以在MySQL启动时配置对应的文件路径以及它们的大小，比如我们这样修改一下my.cnf 配置文件:

```mysql
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

这样在MySQL启动之后就会创建这两个512M大小的文件作为系统表空间，其中的autoextend表明这两个文件如果不够用会自动扩展data2文件的大小。

需要注意的一点是,在一个MySQL服务器中，系统表空间只有一份。从MySQL5.5.7到MySQL5.6.6之间的各个版本中，我们表中的数据都会被默认存储到这个系统表空间


**② 独立表空间(file-per-table tablespace)** 

**在MySQL5.6.6以及之后的版本中**，InnoDB并不会默认的把各个表的数据存储到系统表空间中，而是为`每一个表建立一个独立表空间`，也就是说我们创建了多少个表，就有多少个独立表空间。使用`独立表空间`来存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表名相同。

```
表名.ibd
```

> MySQL8.0中不再单独提供`表名.frm`，而是合并在`表名.ibd`文件中。

```mysql
cd /var/lib/mysql
```

![image-20240319180105423](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/image-20240319180105423.png)

> 如果问到`.frm` 在MySQL8中不存在了，去哪里了？

这就需要解析 ibd 文件。Oracle官方将 frm文件 的信息及更多信息移动到叫做序列化字典信息（Serialized Dictionary Information,SDI），SDI 被写在 ibd 文件内部，MySQL 8.0 属于 Oracle 旗下，同理。

为了从 IBD 文件中提取 SDI 信息，Oracle 提供了一个应用程序 ibd2sdi

这个工具不需要下载，MySQL8自带的有，只需要你配好环境变量就能到处用。

查看表结构：到存储ibd文件的目录下，执行下面的命令

```mysql
ibd2sdi --dump-file=student.txt student.ibd
```

```mysql
[root@master dbtest1]# ibd2sdi --dump-file=emp1.txt emp1.ibd
[root@master dbtest1]# ll
总用量 124
-rw-r-----. 1 mysql mysql 114688 3月  20 15:22 emp1.ibd
-rw-------. 1 root  root   11111 3月  20 15:40 emp1.txt
[root@master dbtest1]# vim ./emp1.txt  # 该文件下可以查看创建的表结构信息

```



这样 ibd2sdi 就会把 xxx.ibd 里存储的表结构以 json 的格式保存在 student.txt 中


**③ 系统表空间与独立表空间的设置**

我们可以自己指定使用`系统表空间`还是`独立表空间`来存储数据，这个功能由启动参数`innodb_file_per_table`控制,比如说我们想刻意将表数据都存储到 `系统表空间` 时，可以在启动MySQL服务器的时候这样配置:

```ini
[server] 
innodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间

```

默认情况下：

```mysql
mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.01 sec) # on：代表独立表空间
```

**④ 其他类型的表空间**

随着MySQL的发展，除了上述两种老牌表空间之外，现在还新提出了一些不同类型的表空间，比如通用表空间（general tablespace）、临时表空间（temporary tablespace）等。



==3、图解：直接看这里的图解，比上面的描述更加清楚一点==

![img](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/02ecd704bcc1eb3c9f05534f81a28865.png)

![img](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/18ff8b9726f1bc96619cda6c58b07025.png)



###### **2.3.2 MyISAM存储引擎模式** 

**1.** **表结构**

在存储表结构方面， MyISAM 和 InnoDB 一样，也是在`数据目录`下对应的数据库子目录下创建了一个专门用于描述表结构的文件

```
表名.frm
```

**2.** **表中数据和索引**

在MyISAM中的索引全部都是`二级索引`，该存储引擎的`数据和索引是分开存放`的。所以在文件系统中也是使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。

```
test.frm 存储表结构 #MySQL8.0 改为了 b.xxx.sdi
test.MYD 存储数据 (MYData) 
test.MYI 存储索引 (MYIndex
```

**3、图解**

![img](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/373e8af95b94c6b110887cbee62b4de2.png)



##### ==2.4 小结==

举例：数据库 a ， 表b 。

- 如果表 b 采用 InnoDB ，data/a 中会产生 1 个或者 2 个文件:

  - ​    b.frm：描述表结构文件，字段长度等
  - ​    如果采用 系统表空间 模式的，数据信息和索引信息都存储在 ibdata1 中
  - ​    如果采用 独立表空间 存储模式，data/a中还会产生 b.ibd 文件（存储数据信息和索引信息）

  > MySQL 5.7 中会在 data/a 的目录下生成 db.opt 文件用于保存数据库的相关配置。比如：字符集、比较规则。而 MySQL 8.0 不再提供 db.opt 文件。
  >
  > MySQL 8.0 中 不再单独提供 b.frm，而是合并在 b.ibd 文件中。



- 如果表 b 采用 MyISAM ，data/a中会产生 3 个文件:
  - ​    MySQL5.7 中 b.frm ：描述表结构文件，字段长度等。
  - ​    MySQL8.0 中 b.xxx.sdi ：描述表结构文件，字段长度等
  - ​    b.MYD (MYData)：数据信息文件，存储数据信息(如果采用独立表存储模式)
  - ​    b.MYI (MYIndex)：存放索引信息文件



##### 2.5 视图在文件系统中的表示

我们知道MySQL中的视图其实是虚拟的表，也就是某个查询语句的一一个别名而已，所以在存储视图的时候是不要存储真实的数据的，只需要把它的结构存储起来就行了。和表一样，描述视图结构的文件也会被存储到所属数据库对应的子目录下边，只会存储一个视图名. frm的文件。如下图中的: emp_details_view.frm

![bd62b80ee6d743fff7cbd7ea9c5de61e](./%E7%AC%AC02%E7%AB%A0%20MySQL%E7%9A%84%E6%95%B0%E6%8D%AE%E7%9B%AE%E5%BD%95.assets/bd62b80ee6d743fff7cbd7ea9c5de61e.png)



##### 2.6 其他的文件

除了我们上边说的这些用户自己存储的数据以外，数据目录下还包括为了更好运行程序的一些额外文件,主要包括这几种类型的文件:

- 服务器进程文件

  我们知道每运行一个MySQL服务器程序,都意味着启动一个进程。MySQL服务器会把自己的进程ID写入到一个文件中。

- 服务器日志文件

  在服务器运行过程中，会产生各种各样的日志，比如常规的查询日志、错误日志、二进制日志、redo日志等。这些日志各有各的用途。后面讲解。

- 默认/自动生成的SSL和RSA证书和密钥文件

  主要是为了客户端和服务器安全通信而创建的一些文件。

