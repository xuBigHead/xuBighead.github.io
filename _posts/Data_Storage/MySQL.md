# MySQL

## 基本概念

### 数据库三范式

第一范式（1NF）：对属性的原子性，要求属性具有原子性，不可再分解；

第二范式（2NF）：对记录的唯一性，要求记录有唯一标识，即实体的唯一性，即不存在部分依赖；

第三范式（3NF）：对字段的冗余性，要求任何字段不能由其他字段派生出来，它要求字段没有冗余，即不存在传递依赖。



三大范式的作用是为了控制数据库的冗余，是对空间的节省，实际上，一般互联网公司的设计都是反范式的，通过冗余一些数据，避免跨表跨库，利用空间换时间，提高性能。



## 架构
MySQL主要分为 Server 层和存储引擎层：

![](../../Image/2022/04/220407.png)



![img](../../Image/2022/09/220921-3.jpg)

- 客户端

最上层的服务并不是MySQL所独有的，大多数基于网络的客户端/服务器的工具或者服务都有类似的架构。比如连接处理、授权认证、安全等等。



- Server 层

主要包括连接器、查询缓存、分析器、优化器、执行器等，所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图，函数等，还有一个通用的日志模块 binglog 日志模块。

`连接器`：TCP握手后服务器来验证登陆用户身份，A用户创建连接后，管理员对A用户权限修改了也不会影响到已经创建的链接权限，必须重新登陆。

`查询缓存`：查询后的结果存储位置，MySQL8.0版本以后已经取消，因为查询缓存失效太频繁，得不偿失。

`分析器`：根据语法规则，判断你输入的这个SQL语句是否满足MySQL语法。

`优化器`：多种执行策略可实现目标，系统自动选择最优进行执行。

`执行器`：判断是否有权限，将最终任务提交到存储引擎。




- 存储引擎

主要负责数据的存储和读取。其架构模式是`插件式`的，支持`InnoDB`、`MyISAM`、`Memory`等多个存储引擎。现在最常用的存储引擎是`InnoDB`，它从MySQL 5.5.5版本开始成为了默认存储引擎(经常用的也是这个)。server 层通过api与存储引擎进行通信。



### Server 层

MySQL 大多数的核心功能模块都在这实现，主要包括连接器，查询缓存、解析器、预处理器、优化器、执行器等。另外，所有的内置函数（如日期、时间、数学和加密函数等）和所有跨存储引擎的功能（如存储过程、触发器、视图等。）都在 Server 层实现。



#### 连接器

第一步要先连接 MySQL 服务，然后才能执行 SQL 语句，连接的过程需要先经过 TCP 三次握手，因为 MySQL 是基于 TCP 协议进行传输的。

如果 MySQL 服务正常运行，完成 TCP 连接的建立后，连接器就要开始验证你的用户名和密码，如果用户名或密码不对，就收到一个"Access denied for user"的错误，然后客户端程序结束执行。

如果用户密码都没有问题，连接器就会获取该用户的权限，然后保存起来，后续该用户在此连接里的任何操作，都会基于连接开始时读到的权限进行权限逻辑的判断。

所以，如果一个用户已经建立了连接，即使管理员中途修改了该用户的权限，也不会影响已经存在连接的权限。修改完成后，只有再新建的连接才会使用新的权限设置。



##### 数据库连接池

池化设计应该不是一个新名词。我们常见的如java线程池、jdbc连接池、redis连接池等就是这类设计的代表实现。这种设计会初始预设资源，解决的问题就是抵消每次获取资源的消耗，如创建线程的开销，获取远程连接的开销等。就好比你去食堂打饭，打饭的大妈会先把饭盛好几份放那里，你来了就直接拿着饭盒加菜即可，不用再临时又盛饭又打菜，效率就高了。除了初始化资源，池化设计还包括如下这些特征：池子的初始值、池子的活跃值、池子的最大值等，这些特征可以直接映射到java线程池和数据库连接池的成员属性中。

数据库连接本质就是一个 socket 的连接。数据库服务端还要维护一些缓存和用户权限信息之类的 所以占用了一些内存。我们可以把数据库连接池是看做是维护的数据库连接的缓存，以便将来需要对数据库的请求时可以重用这些连接。为每个用户打开和维护数据库连接，尤其是对动态数据库驱动的网站应用程序的请求，既昂贵又浪费资源。**在连接池中，创建连接后，将其放置在池中，并再次使用它，因此不必建立新的连接。如果使用了所有连接，则会建立一个新连接并将其添加到池中。 **连接池还减少了用户必须等待建立与数据库的连接的时间。



#### 查询缓存 

> 执行查询语句的时候，会先查询缓存。不过，MySQL 8.0 版本后移除，因为这个功能不太实用



连接器得工作完成后，客户端就可以向 MySQL 服务发送 SQL 语句了，MySQL 服务收到 SQL 语句后，就会解析出 SQL 语句的第一个字段，看看是什么类型的语句。

如果 SQL 是查询语句（select 语句），MySQL 就会先去查询缓存（ Query Cache ）里查找缓存数据，看看之前有没有执行过这一条命令，这个查询缓存是以 key-value 形式保存在内存中的，key 为 SQL 查询语句，value 为 SQL 语句查询的结果。

如果查询的语句命中查询缓存，那么就会直接返回 value 给客户端。如果查询的语句没有命中查询缓存中，那么就要往下继续执行，等执行完后，查询的结果就会被存入查询缓存中。

这么看，查询缓存还挺有用，但是其实**查询缓存挺鸡肋**的。

对于更新比较频繁的表，查询缓存的命中率很低的，因为只要一个表有更新操作，那么这个表的查询缓存就会被清空。如果刚缓存了一个查询结果很大的数据，还没被使用的时候，刚好这个表有更新操作，查询缓冲就被清空了，相当于缓存了个寂寞。

所以，MySQL 8.0 版本直接将查询缓存删掉了，也就是说 MySQL 8.0 开始，执行一条 SQL 查询语句，不会再走到查询缓存这个阶段了。

对于 MySQL 8.0 之前的版本，如果想关闭查询缓存，我们可以通过将参数 query_cache_type 设置成 DEMAND。



**缓存虽然能够提升数据库的查询性能，但是缓存同时也带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。** 因此，开启缓存查询要谨慎，尤其对于写密集的应用来说更是如此。如果开启，要注意合理控制缓存空间大小，一般来说其大小设置为几十MB比较合适。此外，**还可以通过sql_cache和sql_no_cache来控制某个查询语句是否需要缓存：**



my.cnf加入以下配置，重启MySQL开启查询缓存

```bash
query_cache_type=1
query_cache_size=600000
```

MySQL执行以下命令也可以开启查询缓存

```mysql
set global query_cache_type=1;
set global query_cache_size=600000;
```



#### 分析器

没有命中缓存的话，SQL 语句就会经过分析器，主要分为两步，词法分析和语法分析，先看 SQL 语句要做什么，再检查 SQL 语句语法是否正确。

第一件事情，**词法分析**。MySQL 会根据你输入的字符串识别出关键字出来，构建出 SQL 语法树，这样方面后面模块获取 SQL 类型、表名、字段名、 where 条件等等。

第二件事情，**语法分析**。根据词法分析的结果，语法解析器会根据语法规则，判断你输入的这个 SQL 语句是否满足 MySQL 语法。

如果输入的 SQL 语句语法不对，就会在解析器这个阶段报错。比如，我下面这条查询语句，把 from 写成了 form，这时 MySQL 解析器就会给报错。



#### 预处理器

预处理阶段做了两件事。

- 检查 SQL 查询语句中的表或者字段是否存在，不存在则抛出异常；
- 将 `select *` 中的 `*` 符号，扩展为表上的所有列；



#### 优化器

优化器对查询进行优化，包括重写查询、决定表的读写顺序以及选择合适的索引等，生成执行计划。

**优化器主要负责将 SQL 查询语句的执行方案确定下来**，比如在表里面有多个索引的时候，优化器会基于查询成本的考虑，来决定选择使用哪个索引。



#### 执行器 

经历完优化器后，就确定了执行方案，接下来 MySQL 就真正开始执行语句了，这个工作是由「执行器」完成的。在执行的过程中，执行器就会和存储引擎交互了，交互是以记录为单位的。



### 存储引擎

支持 InnoDB、MyISAM、Memory 等多个存储引擎，不同的存储引擎共用一个 Server 层。现在最常用的存储引擎是 InnoDB，从 MySQL 5.5 版本开始， InnoDB 成为了 MySQL 的默认存储引擎。我们常说的索引数据结构，就是由存储引擎层实现的，不同的存储引擎支持的索引类型也不相同，比如 InnoDB 支持索引类型是 B+树 ，且是默认使用，也就是说在数据表中创建的主键索引和二级索引默认使用的是 B+ 树索引。




## 分区表
分区表是一个独立的逻辑表，但是底层由多个物理子表组成。

当查询条件的数据分布在某一个分区的时候，查询引擎只会去某一个分区查询，而不是遍历整个表。在管理层面，如果需要删除某一个分区的数据，只需要删除对应的分区即可。

### 按照范围分区。
```mysql
CREATE TABLE test_range_partition(
       id INT auto_increment,
       createdate DATETIME,
       primary key (id,createdate)
   ) 
   PARTITION BY RANGE (TO_DAYS(createdate) ) (
      PARTITION p201801 VALUES LESS THAN ( TO_DAYS('20180201') ),
      PARTITION p201802 VALUES LESS THAN ( TO_DAYS('20180301') ),
      PARTITION p201803 VALUES LESS THAN ( TO_DAYS('20180401') ),
      PARTITION p201804 VALUES LESS THAN ( TO_DAYS('20180501') ),
      PARTITION p201805 VALUES LESS THAN ( TO_DAYS('20180601') ),
      PARTITION p201806 VALUES LESS THAN ( TO_DAYS('20180701') ),
      PARTITION p201807 VALUES LESS THAN ( TO_DAYS('20180801') ),
      PARTITION p201808 VALUES LESS THAN ( TO_DAYS('20180901') ),
      PARTITION p201809 VALUES LESS THAN ( TO_DAYS('20181001') ),
      PARTITION p201810 VALUES LESS THAN ( TO_DAYS('20181101') ),
      PARTITION p201811 VALUES LESS THAN ( TO_DAYS('20181201') ),
      PARTITION p201812 VALUES LESS THAN ( TO_DAYS('20190101') )
   )
```


在/var/lib/mysql/data/可以找到对应的数据文件，每个分区表都有一个使用##分隔命名的表文件：

```
  -rw-r----- 1 MySQL MySQL    65 Mar 14 21:47 db.opt
  -rw-r----- 1 MySQL MySQL  8598 Mar 14 21:50 test_range_partition.frm
  -rw-r----- 1 MySQL MySQL 98304 Mar 14 21:50 test_range_partition##P##p201801.ibd
  -rw-r----- 1 MySQL MySQL 98304 Mar 14 21:50 test_range_partition##P##p201802.ibd
  -rw-r----- 1 MySQL MySQL 98304 Mar 14 21:50 test_range_partition##P##p201803.ibd
```



### list分区

对于`List`分区，分区字段必须是已知的，如果插入的字段不在分区时枚举值中，将无法插入。

```mysql
create table test_list_partiotion
   (
       id int auto_increment,
       data_type tinyint,
       primary key(id,data_type)
   )partition by list(data_type)
   (
       partition p0 values in (0,1,2,3,4,5,6),
       partition p1 values in (7,8,9,10,11,12),
       partition p2 values in (13,14,15,16,17)
   );
```



### hash分区

可以将数据均匀地分布到预先定义的分区中。

```mysql
create table test_hash_partiotion
   (
       id int auto_increment,
       create_date datetime,
       primary key(id,create_date)
   )partition by hash(year(create_date)) partitions 10;
```





## 变量查询
### datadir
查询存储数据地址。

```mysql
show global variables like "datadir";
```



## 主从同步

### 什么是MySQL主从同步？

主从同步使得数据可以从一个数据库服务器复制到其他服务器上，在复制数据时，一个服务器充当主服务器（`master`），其余的服务器充当从服务器（`slave`）。

因为复制是异步进行的，所以从服务器不需要一直连接着主服务器，从服务器甚至可以通过拨号断断续续地连接主服务器。通过配置文件，可以指定复制所有的数据库，某个数据库，甚至是某个数据库上的某个表。



### 为什么要做主从同步？

1. 读写分离，使数据库能支撑更大的并发。
2. 在主服务器上生成实时数据，而在从服务器上分析这些数据，从而提高主服务器的性能。
3. 数据备份，保证数据的安全。
4. 更新完成。



## 参考资料
- [MySQL 5.7版本版本版本版本官方文档](https://dev.mysql.com/doc/refman/5.7/en/【【【)
- [MySQL 8.0版本版本版本版本官方文档](https://dev.mysql.com/doc/refman/8.0/en/)



# MySQL 数据类型

## 时间日期类型

### date

日期  如：2019-10-26 不带时分秒

范围: '1000-01-01' to '9999-12-31'

表示的日期值, 格式`yyyy-mm-dd`,范围`1000-01-01 到 9999-12-31`，3字节。



### time

表示的时间值，格式 `hh:mm:ss`，范围`-838:59:59 到 838:59:59`，3字节。



### datetime

日期时间 如：2019-10-26 10:53:00 带时分秒 

范围: '1000-01-01 00:00:00' to '9999-12-31 23:59:59'.

表示的日期时间值，格式`yyyy-mm-dd hh:mm:ss`，范围`1000-01-01 00:00:00到`9999-12-31 23:59:59```,8字节，跟时区无关。



**推荐优先使用`datetime`类型来保存日期和时间，因为存储范围更大，且跟时区无关。**



### timestamp

时间戳， 指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。

范围: '1970-01-01 00:00:01' UTC to '2038-01-19 03:14:07' UTC 

Coordinated Universal  Time，协调世界时，又称世界统一时间、世界标准时间、国际协调时间。由于英文（CUT）和法文（TUC）的缩写不同，作为妥协，简称UTC。

表示的时间戳值，格式为`yyyymmddhhmmss`，范围`1970-01-01 00:00:01到2038-01-19 03:14:07`，4字节，跟时区有关。



#### datetime和timestamp比较

**相同点**：

1. 两个数据类型存储时间的表现格式一致。均为 `YYYY-MM-DD HH:MM:SS`
2. 两个数据类型都包含「日期」和「时间」部分。
3. 两个数据类型都可以存储微秒的小数秒（秒后6位小数秒）



**不同点：**

1. **日期范围**：DATETIME 的日期范围是 `1000-01-01 00:00:00.000000` 到 `9999-12-31 23:59:59.999999`；TIMESTAMP 的时间范围是`1970-01-01 00:00:01.000000` UTC` 到 ``2038-01-09 03:14:07.999999` UTC
2. **存储空间**：DATETIME 的存储空间为 8 字节；TIMESTAMP 的存储空间为 4 字节
3. **时区相关**：DATETIME 存储时间与时区无关；TIMESTAMP 存储时间与时区有关，显示的值也依赖于时区
4. **默认值**：DATETIME 的默认值为 null；TIMESTAMP 的字段默认不为空(not null)，默认值为当前时间(CURRENT_TIMESTAMP)



### year

年份值，格式为`yyyy`。范围`1901到2155`，1字节。



## 数值类型

### int



### bigint



### float

float和double是以二进制存储的，所以有一定的误差。



### double



## 字符串类型

### char

- char表示定长字符串，长度是固定的；
- 如果插入数据的长度小于char的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比varchar快很多，甚至能快50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于char来说，最多能存放的字符个数为255，和编码无关



### varchar

- varchar表示可变长字符串，长度是可变的；
- 插入的数据是多长，就按照多长来存储；
- varchar在存取方面与char相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于varchar来说，最多能存放的字符个数为65532



#### 字段长度

字段长度在`varchar`和`char`类型表示字符长度，而其他类型表示的长度都表示字节长度。比如`char(10)`表示字符长度是10，而`bigint（4）`表示显示长度是`4`个字节，但是因为bigint实际长度是`8`个字节，所以bigint（4）的实际长度就是8个字节。



### text

text用于存储大字符串，有一个字符集，并且根据字符集的校对规则对值进行排序和比较。



### decimal

货币在数据库中MySQL常用Decimal和Numric类型表示，这两种类型被MySQL实现为同样的类型。他们被用于保存与货币有关的数据。

例如salary DECIMAL(9,2)，9(precision)代表将被用于存储值的总的小数位数，而2(scale)代表将被用于存储小数点后的位数。存储在salary列中的值的范围是从-9999999.99到9999999.99。

DECIMAL和NUMERIC值作为字符串存储，而不是作为二进制浮点数，以便保存那些值的小数精度。

之所以不使用float或者double的原因：因为float和double是以二进制存储的，所以有一定的误差。



### numeric



### 总结

#### 尽量使用数值替代字符串类型

因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了，字符会降低查询和连接的性能，并会增加存储开销。



#### 使用varchar代替char

`varchar`变长字段按数据内容实际长度存储，存储空间小，可以节省存储空间；`char`按声明大小存储，不足补空格。

其次对于查询来说，在一个相对较小的字段内搜索，效率更高；



`char`和`varchar2`是一对矛盾的统一体，两者是互补的关系，`varchar2`比`char`节省空间，在效率上比`char`会稍微差一点，既想获取效率，就必须牺牲一点空间，这就是我们在数据库设计上常说的“以空间换效率”。

`varchar2`虽然比`char`节省空间，但是假如一个`varchar2`列经常被修改，而且每次被修改的数据的长度不同，这会引起“行迁移”现象，而这造成多余的I/O，是数据库设计中要尽力避免的，这种情况下用`char`代替`varchar2`会更好一些。`char`中还会自动补齐空格，因为你`insert`到一个`char`字段自动补充了空格的,但是`select`后空格没有删除，因此`char`类型查询的时候一定要记得使用`trim`。

如果开发人员细化使用`rpad()`技巧将绑定变量转换为某种能与`char`字段相比较的类型（当然，与截断`trim`数据库列相比，填充绑定变量的做法更好一些，因为对列应用函数`trim`很容易导致无法使用该列上现有的索引），可能必须考虑到经过一段时间后列长度的变化。如果字段的大小有变化，应用就会受到影响，因为它必须修改字段宽度。

正是因为以上原因，定宽的存储空间可能导致表和相关索引比平常大出许多，还伴随着绑定变量问题，所以无论什么场合都要避免使用char类型。



## 其它类型

### blob

blob用于存储二进制数据，没有字符集。




# MySQL 字符集

字符集指的是一种从二进制编码到某类字符符号的映射。校对规则则是指某种字符集下的排序规则。MySQL中每一种字符集都会对应一系列的校对规则。

MySQL采用的是类似继承的方式指定字符集的默认值，每个数据库以及每张数据表都有自己的默认值，他们逐层继承。比如：某个库中所有表的默认字符集将是该数据库所指定的字符集（这些表在没有指定字符集的情况下，才会采用默认字符集）。



## utf8



## utf8mb4

MySQL可以直接使用字符串存储emoji。但是需要注意的，utf8 编码是不行的，MySQL中的utf8是阉割版的 utf8，它最多只用 3 个字节存储字符，所以存储不了表情。需要使用utf8mb4编码。



# MySQL 命令

- 启动：

net start mySql;



- 进入：

mysql -u root -p/mysql -h localhost -u root -p databaseName;



- 备份数据库：(将数据库test备份)：

mysqldump -u root -p test>c:\test.txt



- 将备份数据导入到数据库：(导回test数据库)：

mysql -u root -p test<c:\test.txt



- 删除student_course数据库中的students数据表：

rm -f student_course/students.*



## mysqldump



# MySQL 语法

**SQL主要分成四部分**：
（1）数据定义。（SQL DDL）用于定义SQL模式、基本表、视图和索引的创建和撤消操作。
（2）数据操纵。（SQL DML）数据操纵分成数据查询和数据更新两类。数据更新又分成插入、删除、和修改三种操作。
（3）数据控制。包括对基本表和视图的授权，完整性规则的描述，事务控制等内容。
（4）嵌入式SQL的使用规定。涉及到SQL语句嵌入在宿主语言程序中使用的规则。



## DCL

### 概念

DCL（Data Control Language）数据库控制语言  授权，角色控制等。



### 语法

#### GRANT 授权

```mysql
# 增加一个管理员帐户
grant all on . to user@localhost identified by "password";
# 创建一个可以从任何地方连接服务器的一个完全的超级用户，但是必须使用一个口令something做这个
grant all privileges on . to identified by ’something’ with；
# 增加新用户，格式：grant select on 数据库.* to 用户名@登录主机 identified by “密码”
GRANT ALL PRIVILEGES ON . TO IDENTIFIED BY ’something’ WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON . TO ” IDENTIFIED BY ’something’ WITH GRANT OPTION;
# 创建一个用户custom在特定客户端it363.com登录，可访问特定数据库fangchandb
grant select, insert, update, delete, create,drop on fangchandb.* to custom@ it363.com identified by ‘ passwd’
```



#### REVOKE 取消授权

删除授权：

```mysql
revoke all privileges on . from ”;
delete from user where user=”root” and host=”%”;
flush privileges;
```



#### UPDATE

```mysql
# 修改mysql中root的密码
update user set password=password(”xueok654123″) where user=’root’;
# 刷新数据库
flush privileges;
```



## DDL
### 概念

**DDL**（Data Definition Language）数据库定义语言

>  statements are used to define the database structure or schema.



用于定义数据库的三级结构，包括外模式、概念模式、内模式及其相互之间的映像，定义数据的完整性、安全控制等约束。DDL不需要commit。



### 语法
#### 索引相关

##### 查询索引

```mysql
## 查询指定表的索引
SHOW INDEX FROM it_blog;
```



##### 查询结果：

| Table   | Non_unique | Key_name              | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
| ------- | ---------- | --------------------- | ------------ | ----------- | --------- | ----------- | -------- | ------ | ---- | ---------- | ------- | ------------- | ------- | ---------- |
| it_blog | 0          | PRIMARY               | 1            | id          | A         | 2141        |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 0          | uk_url                | 1            | url         | A         | 2172        |          |        |      | BTREE      |         | url唯一索引   | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 1            | user_id     | A         | 2           |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 2            | tag_id      | A         | 4           |          |        |      | BTREE      |         |               | YES     |            |
| it_blog | 1          | idx_user_tag_classify | 3            | classify_id | A         | 8           |          |        |      | BTREE      |         |               | YES     |            |



##### 添加索引

```mysql
# 添加普通索引
ALTER TABLE `it_blog` ADD KEY `idx_user`(`user_id`);
alter table table1 add index ind_id (id);
CREATE INDEX `idx_user` ON `it_blog`(`user_id`);
# 添加联合索引
ALTER TABLE `it_blog` ADD KEY `idx_user_classiyf`(`user_id`, `classify_id`);
CREATE INDEX `idx_user_classiyf` ON `it_blog`(`user_id`, `classify_id`);
# 添加唯一索引
ALTER TABLE `it_blog` ADD UNIQUE KEY `uk_url`(`url`);
CREATE UNIQUE INDEX `uk_url` ON `it_blog`(`url`);
# 添加主键索引
ALTER TABLE `it_blog` ADD PRIMARY KEY `temp`(`id`);
# 移除主键并设置新的主键
ALTER TABLE `it_blog` DROP PRIMARY KEY, ADD PRIMARY KEY `it_blog`(`id`) USING BTREE;
# 添加FULLTEXT(全文索引)
ALTER TABLE `table_name` ADD FULLTEXT (`column`) 
```



##### 删除索引

```mysql
# 删除索引
ALTER TABLE `it_blog` DROP INDEX `uk_url`;
# 删除索引
DROP INDEX `idx_user` ON `it_blog`;
# 删除主键索引
ALTER TABLE `it_blog` DROP PRIMARY KEY;
```



如果单纯移除自增主键而不设置新的主键时，会执行失败。因为表中只能有一列自增列并且改自增列必须要是主键。此时会返回如下结果：

```
ALTER TABLE `it_blog` DROP PRIMARY KEY
> 1075 - Incorrect table definition; there can be only one auto column and it must be defined as a key
> 时间: 0.031s
```



#### CREATE

```MySQL
# 创建临时表
create temporary table zengchao(name varchar(10));
# 创建表是先判断表是否存在
create table if not exists students(……);
# 从已经有的表中复制表的结构
create table table2 select * from table1 where 1<>1;
# 复制表
create table table2 select * from table1;
```



##### 存储引擎

```mysql
# innodb引擎
CREATE TABLE TESTIdentity(
ID int,
key(id))engine=INNODB auto_increment=100;
```



##### 默认约束

```MySQL
# id字段默认值为12
CREATE TABLE emp
(
 id INT DEFAULT 12
)
```



##### 自增列

无论innodb引擎还是MYISAM引擎的表中，只能有一个自增列，并且自增列一定是索引列，无论是二级索引还是主键索引MySQL字符串函数



```MySQL
# 设置自增ID并从100开始
CREATE TABLE emp (
ID INT  PRIMARY KEY AUTO_INCREMENT
) AUTO_INCREMENT = 100;
```



###### MyISAM和INNODB的区别

两种类型的存储引擎所存储的最大ID记录的方式不同，MyISAM表将最大的ID记录到了数据文件里，重启mysql自增主键的最大ID值也不会丢失；而InnoDB则是把最大的ID值记录到了内存中，所以重启mysql或者对表进行了OPTIMIZE操作后，最大ID值将会丢失。

所以如果最大自增ID值被删除后重启MySQL，此时MySIAM会在删除前的最大ID的基础上进行自增，而INNODB会在当前表中的最大ID的基础上进行自增。



###### 获取自增值方法

(1) SELECT MAX(id) FROM person  针对特定表

(2) SELECT LAST_INSERT_ID()  函数  针对任何表

(3) SELECT @@identity   针对任何表

@@identity 是表示的是最近一次向具有identity属性(即自增列)的表插入数据时对应的自增列的值，是系统定义的全局变量。一般系统定义的全局变量都是以@@开头，用户自定义变量以@开头。使用@@identity的前提是在进行insert操作后，执行select @@identity的时候连接没有关闭，否则得到的将是NULL值。

(4)  SHOW TABLE STATUS LIKE 'person'

如果针对特定表，建议使用这一种方法得出的结果里边对应表名记录中有个Auto_increment字段，里边有下一个自增ID的数值就是当前该表的最大自增ID.



#### ALTER

##### 表相关

```MySQL
# 对表重新命名
alter table table1 rename as table2;
alter table t1 rename t2;
```



##### 表字段相关

```MySQL
# 增加一个字段
alter table tabelName add column fieldName dateType;
# 增加多个字段
alter table tabelName add column fieldName1 dateType, add columns fieldName2 dateType;
# 修改列id的类型为int unsigned
alter table table1 modify id int unsigned;
# 修改列id的名字为sid，而且把属性修改为int unsigned
alter table table1 change id sid int unsigned;
# 如果修改字段名前后一致，则可以通过CHANGE实现只修改字段类型的效果
ALTER TABLE emp2 CHANGE id id BIGINT
# 删除字段
ALTER TABLE emp2 DROP NAME;
```



##### 外键相关

```MySQL
# 删除外键约束
ALTER TABLE emp2 DROP FOREIGN KEY fk_emp_dept
# 删除主键约束
ALTER TABLE emp2 DROP PRIMARY KEY pk_emp_dept
```



#### DROP

```MySQL
# 删除表
DROP TABLE emp2
# 删除多个表或者删除之前要先判断一下
DROP TABLE IF EXISTS emp1 ,emp2
```



#### TRUNCATE

删除表中所有数据。

```mysql
TRUNCATE TABLE person
```



#### COMMENT



#### RENAME



#### SHOW

```mysql
# 列出数据库。
SHOW DATABASES;
# 查询表状态信息。
SHOW TABLE STATUS LIKE 'person'
# 列出所有数据表。
SHOW TABLES;
# 查看表结构
SHOW CREATE TABLE `it_blog`;
```



#### USE

```mysql
# 选择数据库
USE DATABASES;
```



#### DESCRIBE

```MySQL
# 表的详细描述
DESCRIBE tablename; 
# DESCRIBE 可以简写为 DESC
DESC tablename
```



## DML
### 概念

DML（Data Manipulation Language）数据操纵语言statements are used for managing data within schema objects.

由DBMS提供，用于让用户或程序员使用，实现对数据库中数据的操作。
DML分成交互型DML和嵌入型DML两类。
依据语言的级别，DML又可分成过程性DML和非过程性DML两种。
需要commit.



### 语法

#### SELECT

```mysql
select 属性列表
from 表名和视图列表
[where 条件表达式]
[group by 属性名[having 条件表达式]]
[order by 属性名[asc|desc]]
[limit <offset>,row count]
```



- where子句：按照“条件表达式”指定的条件进行查询。
- group by子句：按照“属性名”指定的字段进行分组。
- having子句：有group by才能having子句，只有满足“条件表达式”中指定的条件的才能够输出。
- group by子句通常和count()、sum()等聚合函数一起使用。
- order by子句：按照“属性名”指定的字段进行排序。排序方式由“asc”和“desc”两个参数指出，默认是按照“asc”来排序，即升序。



##### 数据去重

```mysql
# 去除重复数据
SELECT DISTINCT FROM table_name;
```



##### 强制指定索引

```mysql
# FORCE INDEX
SELECT * FROM tableName FORCE INDEX(indexName) WHERE `id` > 1;
```



#### 更新

##### INSERT

**单条插入**

```mysql
INSERT INTO table_name1(column_list1) SELECT (column_list2) FROM table_name2 WHERE (condition);
```

table_name1指定待插入数据的表；column_list1指定待插入表中要插入数据的哪些列；table_name2指定插入数据是从哪个表中查询出来的；column_list2指定数据来源表的查询列，该列表必须和column_list1列表中的字段个数相同，数据类型相同。



**批量插入**

默认新增SQL有事务控制，导致每条都需要事务开启和事务提交，而批量处理是一次事务开启和提交，效率提升明显。



###### IGNORE

当要插入的数据中有重复值的时候，可以使用IGNORE关键字来忽略该条插入语句。

```mysql
INSERT IGNORE INTO person(id,NAME,age,info)
SELECT id,NAME,age,info FROM person_old;
```



###### 自增列

插入时可以不指定自增列，或指定自增列为NULL，此时MySQL会自动自增。

```mysql
# 可以不指定自增列或指定值为NULL
INSERT INTO person(id,NAME,age,info) VALUES (NULL,'feicy',33,'student');
INSERT INTO person(NAME,age,info) VALUES ('amy',12,'bb');
# 也可以指定自增列的值
INSERT INTO person(id,NAME,age,info) VALUES (16,'tom',88,'student');
```



##### UPDATE

```MySQL
UPDATE person SET info ='police' WHERE id BETWEEN 14 AND 17;
```



##### DELETE

```mysql
DELETE FROM person WHERE id BETWEEN 14 AND 17;
```



删除WHERE后面指定条件的数据，如果没有WHERE条件则表示删除表中所有数据。



###### delete in 不走索引

MySQL对select in子查询做了优化，把子查询改成join的方式，所以可以走索引。对于`delete in`子查询，MySQL却没有对它做这个优化。

```mysql
explain select * from account where name in (select name from old_account);
show WARNINGS; //可以查看优化后,最终执行的sql
```



结果如下：

```mysql
select `test2`.`account`.`id` AS `id`,`test2`.`account`.`name` AS `name`,`test2`.`account`.`balance` AS `balance`,`test2`.`account`.`create_time` AS `create_time`,`test2`.`account`.`update_time` AS `update_time` from `test2`.`account` 
semi join (`test2`.`old_account`)
where (`test2`.`account`.`name` = `test2`.`old_account`.`name`)
```



##### 总结

###### TRUNCATE、DELETE与DROP区别？

|          | delete                                                 | truncate                                   | drop                                               |
| :------- | :----------------------------------------------------- | :----------------------------------------- | :------------------------------------------------- |
| 类型     | 属于DML                                                | 属于DDL                                    | 属于DDL                                            |
| 回滚     | 可回滚                                                 | 不可回滚                                   | 不可回滚                                           |
| 删除内容 | 表结构还在，删除表的全部或者一部分数据行，不重置自增列 | 表结构还在，删除表中的所有数据，重置自增列 | 从数据库中删除表，所有数据行，索引和权限也会被删除 |
| 删除速度 | 删除速度慢，需要逐行删除                               | 删除速度快                                 | 删除速度最快                                       |



**相同点：**

1. TRUNCATE和不带WHERE子句的DELETE、以及DROP都会删除表内的所有数据。
2. DROP、TRUNCATE都是DDL语句（数据定义语言），执行后会自动提交。



**不同点：**

1. TRUNCATE和DELETE只删除数据不删除表的结构；DROP语句将删除表的结构被依赖的约束、触发器、索引；
2. DELETE删除不会影响自增列的增长，而TRUNCATE删除会重置自增列为1；
3. 一般来说，执行速度: DROP> TRUNCATE> DELETE。
4. 但 TRUNCATE 比 DELETE 速度快，且使用的系统和事务日志资源少。



`delete`语句每次删除一行，并在事务日志中为所删除的每行记录一项。`truncate table`通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。

`truncate table`删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 `drop table`语句。

对于由 `foreign key`约束引用的表，不能使用 `truncate table`，而应使用不带  `where`子句的 DELETE 语句。由于 `truncate table`不记录在日志中，所以它不能激活触发器。

`truncate table`不能用于参与了索引视图的表。



###### 操作delete或者update语句，加个limit或者循环分批次删除

1、降低写错SQL的代价，如果加limit，删错也只是丢失部分数据，可以通过binlog日志快速恢复的。

2、SQL效率很可能更高，加了`limit 1`，如果第一条就命中目标`return`， 没有`limit`，还会继续执行扫描表。

3、避免长事务，delete执行时,如果age加了索引，MySQL会将所有相关的行加写锁和间隙锁，所有执行相关行会被锁住，如果删除数量大，会直接影响相关业务无法使用。

4、删除数据量很大时，不加limit限制一下记录数，容易把`cpu`打满，导致越删越慢。

5、锁表，一次性删除太多数据，可能造成锁表，会有lock wait timeout exceed的错误，所以建议分批操作。



#### 条件

##### LIKE

模糊匹配查询

- 百分号通配符“%”，匹配任意长度的字符，甚至包括零字符。
- 下划线通配符“_”,一次只能匹配任意一个字符。



##### IS NULL和IS NOT NULL

MySQL中NULL和NULL值是不能通过=符来比较的，要通过IS NULL和IS NOT NULL来判断是否为NULL和是否不为NULL。



##### EXIST 和 NOT EXIST

```mysql
SELECT ...FROM table WHERE EXISTS(subquery);
```



将主查询的数据放到子查询中做条件验证，根据验证结果（TRUE或者FALSE）来决定朱查询的数据结果是否得意保留。

- EXISTS(subquery)只返回TRUE或者FALSE，因此子查询中的SELECT * 也可以是SELECT 1或者SELECT 'X'，官方说法是实际执行时会忽略SELECT清单，因此没有区别。
- EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际校验。
- EXISTS子查询可以用条件表达式，其他子查询或者JOIN来替代，何种最优需要具体问题具体分析。



##### IN 和 NOT IN

`in`查询在MySQL底层是通过`n*m`的方式去搜索，类似`union`。

in查询在进行cost代价计算时（代价 = 元组数 * IO平均值），是通过将in包含的数值，一条条去查询获取元组数的，因此这个计算过程会比较的慢，所以MySQL设置了个临界值(eq_range_index_dive_limit)，5.6之后超过这个临界值后该列的cost就不参与计算了。因此会导致执行计划选择不准确。默认是200，即in条件超过了200个数据，会导致in的代价计算存在问题，可能会导致Mysql选择的索引不准确。



##### WHERE



##### AND



##### OR

OR表示满足两个条件中的一个即可。



##### 总结

###### EXIST和IN的区别

`exists`用于对外表记录做筛选。`exists`会遍历外表，将外查询表的每一行，代入内查询进行判断。当`exists`里的条件语句能够返回记录行时，条件就为真，返回外表当前记录。反之如果`exists`里的条件语句不能返回记录行，条件为假，则外表当前记录被丢弃。

`in`是先把后边的语句查出来放到临时表中，然后遍历临时表，将临时表的每一行，代入外查询去查找。

**子查询的表比较大的时候**，使用`exists`可以有效减少总的循环次数来提升速度；**当外查询的表比较大的时候**，使用`in`可以有效减少对外查询表循环遍历来提升速度。



1、in查询时首先查询子查询的表，然后将内表和外表做一个`笛卡尔积`，然后按照条件进行筛选。

2、子查询使用 exists，会先进行主查询，将查询到的每行数据`循环带入`子查询校验是否存在，过滤出整体的返回数据。

3、两表大小相当，in 和 exists 差别不大。`内表大，用 exists 效率较高；内表小，用 in 效率较高`。

4、查询用not in 那么内外表都进行全表扫描，没有用到索引；而not exists 的子查询依然能用到表上的索引。`not exists比not in要快`。



###### NOT EXIST 和 NOT IN的区别

not in 和not exists如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。



#### GROUP BY

分组



**在需要进行分组过滤时，应尽量先过滤再分组。**



###### WITH ROLLUP

```mysql
SELECT s_id ,COUNT(1) AS total FROM fruits GROUP BY s_id WITH ROLLUP
```



WITH ROLLUP表示将分组统计的结果进行汇总。**当使用ROLLUP时，不能同时使用ORDER BY子句进行结果排序，即ROLLUP和ORDER BY是互相排斥的！**



```mysql
select city ,count(*) as num from staff group by city;
```

1. 创建内存临时表，表里有两个字段`city和num`；

2. 全表扫描staff的记录，依次取出city = 'X'的记录。

	- 判断临时表中是否有为`city	='X'`的行，没有就插入一个记录` (X,1)`;

	- 如果临时表中有`city='X'`的行，就将X这一行的num值加 1；

3. 遍历完成后，再根据字段`city`做排序，得到结果集返回给客户端。这个流程的执行图如下：

![图片](../../Image/2022/10/221011-3.png)



**临时表的排序是怎样的呢？**

就是把需要排序的字段，放到sort buffer，排完就返回。在这里注意一点哈，排序分全字段排序和rowid排序

- 如果是全字段排序，需要查询返回的字段，都放入sort buffer，根据排序字段排完，直接返回
- 如果是rowid排序，只是需要排序的字段放入sort buffer，然后多一次回表操作，再返回。



`group by`使用不当，很容易就会产生慢`SQL`问题。因为它既用到临时表，又默认用到排序。有时候还可能用到磁盘临时表。

- 如果执行过程中，会发现内存临时表大小到达了上限（控制这个上限的参数就是`tmp_table_size`），会把内存临时表转成磁盘临时表。
- 如果数据量很大，很可能这个查询需要的磁盘临时表，就会占用大量的磁盘空间。



##### 优化

**从哪些方向去优化呢？**

- 方向1：既然它默认会排序，我们不给它排是不是就行啦。
- 方向2：既然临时表是影响group by性能的X因素，我们是不是可以不用临时表？

我们一起来想下，执行`group by`语句为什么需要临时表呢？`group by`的语义逻辑，就是统计不同的值出现的个数。如果这个这些值一开始就是有序的，我们是不是直接往下扫描统计就好了，就不用临时表来记录并统计结果啦?

可以有这些优化方案：

- group by 后面的字段加索引
- order by null 不用排序
- 尽量只使用内存临时表
- 使用SQL_BIG_RESULT



#### HAVING

###### having和where的区别？

- 二者作用的对象不同，`where`子句作用于表和视图，`having`作用于组。
- `where`在数据分组前进行过滤，`having`在数据分组后进行过滤。



#### ORDER BY

##### 文件排序

`order by`排序，分为`全字段排序`和`rowid排序`。它是拿`max_length_for_sort_data`和结果行数据长度对比，如果结果行数据长度超过`max_length_for_sort_data`这个值，就会走`rowid`排序，相反，则走全字段排序。



###### rowid排序

```mysql
select name,age,city from staff where city = '深圳' order by age limit 10;
```

rowid排序，一般需要回表去找满足条件的数据，所以效率会慢一点。以下这个SQL，使用rowid排序，执行过程是这样：

1. `MySQL`为对应的线程初始化`sort_buffer`，放入需要排序的`age`字段，以及`主键id`；
2. 从索引树`idx_city`， 找到第一个满足 `city='深圳’`条件的`主键id`,假设`id`为`X`；
3. 到主键`id索引树`拿到`id=X`的这一行数据， 取age和主键id的值，存到`sort_buffer`；
4. 从索引树`idx_city`拿到下一个记录的`主键id`，假设`id=Y`；
5. 重复步骤 3、4 直到`city`的值不等于深圳为止；
6. 前面5步已经查找到了所有`city`为深圳的数据，在`sort_buffer`中，将所有数据根据`age`进行排序；遍历排序结果，取前10行，并按照id的值回到原表中，取出`city、name 和 age`三个字段返回给客户端。

![图片](../../Image/2022/10/221011-1.png)



###### 全字段排序

同样的SQL，如果是走全字段排序是这样的：

```mysql
select name,age,city from staff where city = '深圳' order by age limit 10;
```

1. MySQL 为对应的线程初始化`sort_buffer`，放入需要查询的`name、age、city`字段；
2. 从索引树`idx_city`， 找到第一个满足 `city='深圳’`条件的主键 id，假设找到`id=X`；
3. 到主键id索引树拿到`id=X`的这一行数据， 取`name、age、city`三个字段的值，存到`sort_buffer`；
4. 从索引树`idx_city` 拿到下一个记录的主键`id`，假设`id=Y`；
5. 重复步骤 3、4 直到`city`的值不等于深圳为止；
6. 前面5步已经查找到了所有`city`为深圳的数据，在`sort_buffer`中，将所有数据根据age进行排序；
7. 按照排序结果取前10行返回给客户端。



![图片](../../Image/2022/10/221011-2.png)



`sort_buffer`的大小是由一个参数控制的：`sort_buffer_size`。

- 如果要排序的数据小于`sort_buffer_size`，排序在`sort_buffer`内存中完成
- 如果要排序的数据大于`sort_buffer_size`，则借助磁盘文件来进行排序。



> 借助磁盘文件排序的话，效率就更慢一点。因为先把数据放入`sort_buffer`，当快要满时。会排一下序，然后把`sort_buffer`中的数据，放到临时磁盘文件，等到所有满足条件数据都查完排完，再用归并算法把磁盘的临时排好序的小文件，合并成一个有序的大文件。



###### 文件排序优化

- 因为数据是无序的，所以就需要排序。如果数据本身是有序的，那就不会再用到文件排序啦。而索引数据本身是有序的，我们通过建立索引来优化`order by`语句。
- 还可以通过调整`max_length_for_sort_data、sort_buffer_size`等参数优化；



#### 分页

##### LIMIT

LIMIT[位置偏移量]，行数

第一个“位置偏移量”参数指示MYSQL从哪一行开始显示，是一个可选参数，如果不指定“位置偏移量”，将会从表中第一条记录开始（第一条记录的位置偏移量是0，第二天记录的位置偏移量是1......以此类推）。

第二个参数“行数”指示返回的记录条数。



###### 深度分页问题

limit深分页，导致SQL变慢原因有两个：

- `limit`语句会先扫描`offset+n`行，然后再丢弃掉前`offset`行，返回后`n`行数据。也就是说`limit 100000,10`，就会扫描`100010`行，而`limit 0,10`，只扫描`10`行。
- `limit 100000,10` 扫描更多的行数，也意味着回表更多的次数。



**标签记录法**



**延迟关联法**



#### 子查询

##### ANY和ALL

ANY关键字接在一个比较操作符的后面，表示若与子查询返回的任何值比较为TRUE，则返回TRUE。返回tbl2表的所有num2列，然后将tbl1中的num1的值与之进行比较，只要大于num2的任何一个值，即为符合查询条件的结果。

```mysql
SELECT num1 FROM tbl1 WHERE num1>ANY(SELECT num2 FROM tbl2)
```



ALL关键字接在一个比较操作符的后面，表示与子查询返回的所有值比较为TRUE，则返回TRUE。

```mysql
SELECT num1 FROM tbl1 WHERE num1>ALL(SELECT num2 FROM tbl2)
```



#### 联合查询

##### UNION

`UNION`在进行表链接后会筛选掉重复的记录，所以在表链接后会对所产生的结果集进行排序运算，删除重复的记录再返回结果。

使用UNION关键字，合并结果时，两个查询对应的列数和数据类型必须相同。各个SELECT语句之间使用UNION或UNION ALL关键字分隔。

UNION：执行的时候删除重复的记录，所有返回的行都是唯一的。

UNION ALL：不删除重复行也不对结果进行自动排序。



##### UNION ALL 

采用`UNION ALL`操作符替代`UNION`，因为`UNION ALL`操作只是简单的将两个结果合并后就返回。

- 如果使用UNION ALL，不会合并重复的记录行
- 效率 UNION 高于 UNION ALL



#### 连接查询

![图片](../../Image/2022/09/220921-6.jpg)

##### 内连接

取得两张表中满足存在连接匹配关系的记录。



###### INNER JOIN



###### LEFT JOIN



###### RIGHT JOIN



##### 外连接

不只取得两张表中满足存在连接匹配关系的记录，还包括某张表（或两张表）中不满足匹配关系的记录。



##### 交叉连接

显示两张表所有记录一一对应，没有匹配关系进行筛选，它是笛卡尔积在SQL中的实现，如果A表有m行，B表有n行，那么A和B交叉连接的结果就有m*n行。



##### 总结

- inner join 内连接，只保留两张表中完全匹配的结果集；
- left join会返回左表所有的行，即使在右表中没有匹配的记录；
- right join会返回右表所有的行，即使在左表中没有匹配的记录；



三种连接如果结果相同，优先使用inner join，如果使用left join左边表尽量小。

- 如果inner join是等值连接，返回的行数比较少，所以性能相对会好一点；
- 使用了左连接，左边表数据结果尽量小，条件尽量放到左边处理，意味着返回的行数可能比较少；
- 这是mysql优化原则，就是小表驱动大表，小的数据集驱动大的数据集，从而让性能更优；



1、表连接不宜太多，一般5个以内

1. 关联的表个数越多，编译的时间和开销也就越大

2. 每次关联内存中都生成一个临时表

3. 应该把连接表拆开成较小的几个执行，可读性更高

4. 如果一定需要连接很多表才能得到数据，那么意味着这是个糟糕的设计了

	


## TCL
### 概念

TCL（Transaction Control Language）事务控制语言。



### 语法

#### SAVEPOINT 设置保存点

#### ROLLBACK  回滚

#### SET TRANSACTION



#### 查看和设置

```mysql
## 查看隔离级别
select @@transaction_isolation;
## 设置隔离级别
set session transaction isolation level read uncommitted;
set global transaction isolation level repeatable read;
```



## 变量

mysql服务器维护着2种mysql的系统参数（系统变量）：全局变量（global variables）和会话变量（session variables）。它们的含义与区别如其各占的名称所示，session variables是在session级别的，对其的变更只会影响到本session；global variables是系统级别的，对其的变更会影响所有新session（变更时已经存在session不受影响）至下次mysql server重启动。

注意它的变更影响不能跨重启，要想再mysql server重启时也使用新的值，那么就只有通过在命令行指定变量选项或者更改选项文件来定，而通过SET变更是达不到跨重启的。

每一个系统变量都有一个默认值，这个默认值是在编译mysql系统的时候确定的。对系统变量的指定，一般可以在server启动的时候在命令行指定选项或者通过选项文件来指定当然，大部分的系统变量，可以在系统的运行时，通过set命令指定其值。



```mysql
# 查看系统当前默认的自增列种子值和步长值
SHOW GLOBAL VARIABLES LIKE 'auto_incre%';
```



## 函数

### 统计相关函数

#### count

count() 是一个聚合函数，函数的参数不仅可以是字段名，也可以是其他任意表达式，该函数作用是**统计符合查询条件的记录中，函数指定的参数不为 NULL 的记录有多少个**。



```mysql
select count(name) from t_order;
```

统计「 t_order 表中，name 字段不为 NULL 的记录」有多少个。也就是说，如果某一条记录中的 name 字段的值为 NULL，则就不会被统计进去。



```mysql
select count(1) from t_order;
```

统计「 t_order 表中，1 这个表达式不为 NULL 的记录」有多少个。

1 这个表达式就是单纯数字，它永远都不是 NULL，所以上面这条语句，其实是在统计 t_order 表中有多少个记录。



count函数使用方式按照性能排序如下：

**count(*) = count(1) > count(主键字段) > count(字段)**



- count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL
- count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
- count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

**执行速度**：

- 列名为主键，count(列名)会比count(1)快
- 列名不为主键，count(1)会比count(列名)快
- 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）
- 如果有主键，则 select count（主键）的执行效率是最优的
- 如果表只有一个字段，则 select count（*）最优。



##### 执行原理

###### count(主键字段)

在通过 count 函数统计有多少个记录时，MySQL 的 server 层会维护一个名叫 count 的变量。

server 层会循环向 InnoDB 读取一条记录，如果 count 函数指定的参数不为 NULL，那么就会将变量 count 加 1，直到符合查询的全部记录被读完，就退出循环。最后将 count 变量的值发送给客户端。

InnoDB 是通过 B+ 树来保持记录的，根据索引的类型又分为聚簇索引和二级索引，它们区别在于，聚簇索引的叶子节点存放的是实际数据，而二级索引的叶子节点存放的是主键值，而不是实际数据。

用下面这条语句作为例子：

```sql
//id 为主键值
select count(id) from t_order;
```

如果表里只有主键索引，没有二级索引时，那么，InnoDB 循环遍历聚簇索引，将读取到的记录返回给 server 层，然后读取记录中的 id 值，就会 id 值判断是否为 NULL，如果不为 NULL，就将 count 变量加 1。

但是，如果表里有二级索引时，InnoDB 循环遍历的对象就不是聚簇索引，而是二级索引。

这是因为相同数量的二级索引记录可以比聚簇索引记录占用更少的存储空间，所以二级索引树比聚簇索引树小，这样遍历二级索引的 I/O 成本比遍历聚簇索引的 I/O 成本小，因此「优化器」优先选择的是二级索引。



###### count(1)

用下面这条语句作为例子：

```text
select count(1) from t_order;
```

如果表里只有主键索引，没有二级索引时。

那么，InnoDB 循环遍历聚簇索引（主键索引），将读取到的记录返回给 server 层，**但是不会读取记录中的任何字段的值**，因为 count 函数的参数是 1，不是字段，所以不需要读取记录中的字段值。参数 1 很明显并不是 NULL，因此 server 层每从 InnoDB 读取到一条记录，就将 count 变量加 1。

可以看到，count(1) 相比 count(主键字段) 少一个步骤，就是不需要读取记录中的字段值，所以通常会说 count(1) 执行效率会比 count(主键字段) 高一点。

但是，如果表里有二级索引时，InnoDB 循环遍历的对象就二级索引了。



###### count(*)

看到 `*` 这个字符的时候，是不是大家觉得是读取记录中的所有字段值？

对于 `selete *` 这条语句来说是这个意思，但是在 count(*) 中并不是这个意思。

**count(*) 其实等于 count(`0`)**，也就是说，当你使用 count(`*`) 时，MySQL 会将 `*` 参数转化为参数 0 来处理。所以，**count(\*) 执行过程跟 count(1) 执行过程基本一样的**，性能没有什么差异。

而且 MySQL 会对 count(*) 和 count(1) 有个优化，如果有多个二级索引的时候，优化器会使用key_len 最小的二级索引进行扫描。

只有当没有二级索引的时候，才会采用主键索引来进行统计。



###### count(字段)

count(字段) 的执行效率相比前面的 count(1)、 count(*)、 count(主键字段) 执行效率是最差的。

用下面这条语句作为例子：

```sql
// name不是索引，普通字段
select count(name) from t_order;
```

对于这个查询来说，会采用全表扫描的方式来计数，所以它的执行效率是比较差的。



##### 扩展

###### 为什么要通过遍历的方式来计数？

基于 Innodb 存储引擎来说，count 函数需要通过遍历的方式来统计记录个数。但是在 MyISAM 存储引擎里，执行 count 函数的方式是不一样的，通常在没有任何查询条件下的 count(*)，MyISAM 的查询速度要明显快于 InnoDB。

使用 MyISAM 引擎时，执行 count 函数只需要 O(1 )复杂度，这是因为每张 MyISAM 的数据表都有一个 meta 信息有存储了row_count值，由表级锁保证一致性，所以直接读取 row_count 值就是 count 函数的执行结果。

而 InnoDB 存储引擎是支持事务的，同一个时刻的多个查询，由于多版本并发控制（MVCC）的原因，InnoDB 表“应该返回多少行”也是不确定的，所以无法像 MyISAM一样，只维护一个 row_count 变量。

举个例子，假设表 t_order 有 100 条记录，现在有两个会话并行以下语句：

<img src="../../Image/2022/07/220727-29.png" alt="图片" style="zoom:50%;" />

在会话 A 和会话 B的最后一个时刻，同时查表 t_order 的记录总个数，可以发现，显示的结果是不一样的。所以，在使用 InnoDB 存储引擎时，就需要扫描表来统计具体的记录。

而当带上 where 条件语句之后，MyISAM 跟 InnoDB 就没有区别了，它们都需要扫描表来进行记录个数的统计。



##### 总结

count(1)、 count(*)、 count(主键字段)在执行的时候，如果表里存在二级索引，优化器就会选择二级索引进行扫描。

所以，如果要执行 count(1)、 count(*)、 count(主键字段) 时，尽量在数据表上建立二级索引，这样优化器会自动采用 key_len 最小的二级索引进行扫描，相比于扫描主键索引效率会高一些。

再来，就是不要使用 count(字段) 来统计记录个数，因为它的效率是最差的，会采用全表扫描的方式来统计。如果你非要统计表中该字段不为 NULL 的记录个数，建议给这个字段建立一个二级索引。



##### 优化

###### 取近似值

如果你的业务对于统计个数不需要很精确，比如搜索引擎在搜索关键词的时候，给出的搜索结果条数是一个大概值。

这时，我们就可以使用 show table status 或者 explain 命令来表进行估算。

执行 explain 命令效率是很高的，因为它并不会真正的去查询，下图中的 rows 字段值就是 explain 命令对表 t_order 记录的估算值。



###### 额外冗余计数值

如果是想精确的获取表的记录总数，我们可以将这个计数值保存到单独的一张计数表中。

当我们在数据表插入一条记录的同时，将计数表中的计数字段 + 1。也就是说，在新增和删除操作时，我们需要额外维护这个计数表。



#### sum

#### avg

#### max

#### min



### 字符串相关

#### 字符串拼接

```MySQL
# 联合字符或者多个列(将列id与":"和列name和"="连接)
select concat(id,':',name,'=') from students;
```



#### 大小写转换

MySQL 字符串大小写转化函数有两对：lower(), uppper() 和 lcase(), ucase()。一般使用lower(), upper() 来转换字符串大小写，因为这和其他数据库中函数相兼容。

```mysql
select lower('DDD');
select upper('ddd');
select lcase('DDD');
select ucase('ddd');
```



#### 首尾清除

- 清除首尾空格

清除字符串首尾空格函数有三个：ltrim(), rtrim(), trim()。

```mysql
# 结果为 .ddd .
select concat('.', ltrim(' ddd '), '.'); 
# 结果为 . ddd.
select concat('.', rtrim(' ddd '), '.'); 
# 结果为 .ddd.
select concat('.', trim(' ddd '), '.'); 
```



- trim扩展

trim([{both | leading | trailing} [remstr] from] str) 

trim([remstr from] str) 

```mysql
# 清除字符串首部字符，结果为 ddd..
select trim(leading '.' from '..ddd..'); 
# 清除字符串尾部字符，结果为 ..ddd
select trim(trailing '.' from '..ddd..'); 
# 清除字符串首尾部字符，结果为 ddd 
select trim(both '.' from '..ddd..'); 
select trim('.' from '..ddd..'); 
```



#### 内容截取

MySQL 字符串截取函数：left(), right(), substring(), substring_index()。还有 mid(), substr()。其中，mid(), substr() 等价于 substring() 函数，substring() 的功能非常强大和灵活。



##### left和right

left(str, length)和right(str, length)

```mysql
# 结果为 sql
select left('sqlstudy.com', 3); 
# 结果为 com
select right('sqlstudy.com', 3);
```



##### substring

substring(str, pos); substring(str, pos, len)

```mysql
# 从字符串的第 4 个字符位置开始取，直到结束。结果为 study.com
select substring('sqlstudy.com', 4);
# 从字符串的第 4 个字符位置开始取，只取 2 个字符。结果为 st
select substring('sqlstudy.com', 4, 2); 
# 从字符串的第 4 个字符位置（倒数）开始取，直到结束。结果为 .com 
select substring('sqlstudy.com', -4); 
# 从字符串的第 4 个字符位置（倒数）开始取，只取 2 个字符。结果为 .c
select substring('sqlstudy.com', -4, 2);
```



##### substring_index

substring_index(str,delim,count)

```mysql
# 截取第二个 '.' 之前的所有字符。结果为 www.sqlstudy
select substring_index('www.sqlstudy.com.cn', '.', 2);
# 截取第二个 '.' （倒数）之后的所有字符。结果为 com.cn
select substring_index('www.sqlstudy.com.cn', '.', -2); 
# 如果在字符串中找不到 delim 参数指定的值，就返回整个字符串。结果为 www.sqlstudy.com.cn
select substring_index('www.sqlstudy.com.cn', '.coc', 1); 
```



#### 字符串长度

char_length函数可以用于获取字符串长度

```mysql
select * from brand where name like '%keyword%' 
order by char_length(name) asc limit 5,5;
```



#### 字符串定位

`locate`函数可以匹配的关键字，在字符串中的位置。

```mysql
select * from brand where name like '%苏三%' 
order by char_length(name) asc, locate('苏三',name) asc limit 5,5;
```



`instr`和`position`函数，它们的功能跟`locate`函数类似。



### 数据库相关

```mysql
# 查询数据库版本
select version();
# 查询当前使用的数据库
select database();
```



### 索引相关

```mysql
# 强制使用指定索引查询
SELECT * FROM FORCE INDEX(`index_name`) WHERE `age` = 10;
```



### 其它

```mysql
# 查询时间
select now();
# 查询当前用户
select user();
```



### 总结

| 函数名称 | 描述     |
| -------- | -------- |
| sum      | 求和     |
| avg      | 求平均值 |
| max      | 求最大值 |
| min      | 求最小值 |
| count    | 计数     |
| md5      | 取MD5    |



## 运算符

### 安全等于**<=>**

安全等于运算符（<=>），这个操作符和=操作符执行相同的比较操作，不过<=>可以用来判断NULL值。

在两个操作数均为NULL时，其返回值为1而不为NULL；而当一个操作数为NULL时，其返回值为0而不为NULL。

```mysql
# 结果为0
SELECT 1 <=> 0;
# 结果为0
SELECT 1 <=> NULL;
# 结果为1
SELECT NULL <=> NULL;
```



### LEAST

语法格式为：LEAST（值1,值2,...值n），其中值n表示参数列表中有n个值。在有两个或多个参数的情况下，返回最小值。

任意一个自变量为NULL，则LEAST()的返回值为NULL。

```mysql
# 结果分别为0，a和NULL
SELECT LEAST(2,0), LEAST('a','b','c'), LEAST(10,NULL);
```



### **GREATEST**

语法格式为：GREATEST(值1，值2，...值n)，其中n表示参数列表中有n个值。

在有两个或多个参数的情况下，返回最大值。假如任意一个自变量为NULL，则GREATEST()的返回值为NULL

```mysql
# 结果分别为2，c和NULL
SELECT GREATEST(2,0),GREATEST('a','b','c'),GREATEST(10,NULL);
```



### **REGEXP**

用来匹配字符串，语法格式为：字符串 REGEXP  匹配条件，如果expr满足匹配条件，返回1；如果不满足，则返回0；若expr或匹配条件任意一个为NULL，则结果为NULL。

常用的几种通配符：

（1）'^'匹配以该字符后面的字符开头的字符串

（2）'$'匹配以该字符后面的字符结尾的字符串

（3）'.'匹配任何一个单字符

（4）'[...]'匹配在方括号内的任何字符。例如，“[abc]" 匹配a、b或c。

字符的范围可以使用一个'-'，“[a-z]”匹配任何字母，而“[0-9]”匹配任何数字

（5）`'*' 匹配零个或多个在他前面的字符。例如，“x*”匹配任何数量的'*'字符，“[0-9]*”匹配任何数量的数字，而“.*”匹配任何数量的任何字符。`



```mysql
# 结果为1，1，1，0
SELECT 'ssky' REGEXP '^s','ssky' REGEXP 'y$' ,'ssky' REGEXP '.sky','ssky' REGEXP '[ab]';
```



1、查询以特定字符或字符串开头的记录

字符“^”匹配以特定字符或者字符串开头的文本

```mysql
# 返回f_name字段以b开头的记录
SELECT * FROM fruits WHERE f_name REGEXP '^b'
```



2、查询以特定字符或字符串结尾的记录

字符“$”匹配以特定字符或者字符串结尾的文本

```mysql
# 返回f_name字段以y结尾的记录
SELECT * FROM fruits WHERE f_name REGEXP 'y$'
```



3、用符号“.”来代替字符串中的任意一个字符

```mysql
# 字符“.”匹配任意一个字符，会查询出f_name为orange的数据
SELECT * FROM fruits WHERE f_name REGEXP 'a.g'
```



4、使用“*”和“+”来匹配多个字符

```mysql
# 星号“*”匹配前面的字符任意多次，包括0次。加号“+”匹配前面的字符至少一次
# 查询b开头，a出现任意多次的数据
SELECT * FROM fruits WHERE f_name REGEXP '^ba*'；
# 查询b开头，a出现至少一次的数据
SELECT * FROM fruits WHERE f_name REGEXP '^ba+'
```



5、匹配指定字符串

正则表达式可以匹配指定字符串，只要这个字符串在查询文本中即可，如要匹配多个字符串，多个字符串之间使用分隔符“|”隔开

```mysql
# 查询包含on或ap的数据
SELECT * FROM fruits WHERE f_name REGEXP 'on|ap'
```



6、匹配指定字符中的任意一个

方括号“[]”指定一个字符集合，只匹配其中任何一个字符，即为所查找的文本，方括号[]还可以指定数值集合。

```mysql
# 查询包含o或t的数据
SELECT * FROM fruits WHERE f_name REGEXP '[ot]'
# 查询包含4、5或6的数据，[456]也可以写成[4-6]即指定集合区间
SELECT * FROM fruits WHERE s_id REGEXP '[456]'
```



7、匹配指定字符以外的字符

```mysql
# “[^字符集合]”匹配不在指定集合中的任何字符
# 返回开头不在a-e  1-2字母的记录，例如a1，b1这些记录就不符合要求
SELECT * FROM fruits WHERE f_id REGEXP '[^a-e1-2]'
```



8、使用{n,} 或者{n,m}来指定字符串连续出现的次数

“字符串{n,}”，表示至少匹配n次前面的字符；“字符串{n,m}”表示匹配前面的字符串不少于n次，不多于m次。

```mysql
# 查询b至少出现1次的数据
SELECT * FROM fruits WHERE f_name REGEXP 'b{1,}';
# 查询b至少出现1次，之多出现3次的数据
SELECT * FROM fruits WHERE f_name REGEXP 'ba{1,3}';
```



### 逻辑运算符

- 逻辑与运算符：AND或者&&
- 逻辑或运算符：OR或者||
- 异或运算符：XOR

当任意一个操作数为NULL时，返回值为NULL;对于非NULL的操作数，如果两个操作数都是非0值或者都是0值，则返回结果为0；

```mysql
# 结果为0，0，1，NULL，1
SELECT 1 XOR 1, 0 XOR 0,1 XOR 0,1 XOR NULL,1 XOR 1 XOR 1
```



### 位运算符

位运算符是用来对二进制字节中的位进行测试、移位或者测试处理

MYSQL中提供的位运算有

- 按位或(|)
- 按位与(&)
- 按位异或(^)
- 按位左移(<<)
- 按位右移(>>)
- 按位取反(~)：反转所有比特



# MySQL 数据类型

### 字符串形

#### 字符串形类型

##### char

##### varchar

#### 字段长度

```MySQL
# 获取字段长度
SELECT LENGTH(vb) FROM tmp13;
```



### 整形

#### 整形类型

##### int

##### bigint

#### 类型宽度

MYSQL中的整数型数据类型都可以指定显示宽度. 创建一个表。

```mysql
CREATE TABLE tb_emp( id BIGINT(1))
```



id字段的数据类型为BIGINT(1)，注意到后面的数字1，这表示的是该数据类型指定的显示宽度，指定能够显示的数值中数字的个数。

例如，假设声明一个INT类型的字段 YEAR INT(4) ，该声明指明，在year字段中的数据一般只显示4位数字的宽度。

显示宽度和数据类型的取值范围是无关的。显示宽度只是指明MYSQL最大可能显示的数字个数，数值的位数小于指定的宽度时会有空格填充，如果插入了大于显示宽度的值，只要该值不超过该类型整数的取值范围，数值依然可以插入，而且能显示出来。

例如，向year字段插入一个数值19999，当使用select查询的时候，MYSQL显示的将是完整带有5位数字的19999，而不是4位数字的值 如果不指定显示宽度，则MYSQL为每一种类型指定默认的宽度值。

注意：显示宽度只用于显示，并不能限制取值范围和占用空间，例如：INT(3)会占用4个字节的存储空间，并且允许的最大值也不会是999，而是INT整型所允许的最大值。



# MySQL 配置

## 配置变量

### profiling

记录SQL执行流程配置，可以查看SQL执行具体耗时。



```mysql
# 查看profiling配置是否打开
show variables like 'profiling';
# 开启profiling配置
set profiling=ON;
```



```mysql
# 查看所有SQL执行耗时
show profiles;
```

执行结果如下：

| Query_ID | Duration   | Query                            |
| -------- | ---------- | -------------------------------- |
| 1        | 0.06811025 | select * from user where age>=60 |



```mysql
# 查看指定SQL执行耗时
show profile for query <Query_ID>;
```

执行结果如下：

| Status               | Duration |
| -------------------- | -------- |
| starting             | 0.000074 |
| checking permissions | 0.000010 |
| Opening tables       | 0.000034 |
| init                 | 0.000032 |
| System lock          | 0.000027 |
| optimizing           | 0.000020 |
| statistics           | 0.000058 |
| preparing            | 0.000018 |
| executing            | 0.000013 |
| Sending data         | 0.067701 |
| end                  | 0.000021 |
| query end            | 0.000015 |
| closing tables       | 0.000014 |
| freeing items        | 0.000047 |
| cleaning up          | 0.000027 |



耗时大部分时候都在`Sending data`阶段，而这一阶段里如果慢的话，最容易想到的还是索引相关的原因。



### max_connections

max_connections表示数据库连接数，Mysql的最大连接数默认是`100`, 最大可以达到`16384`。

```mysql
show variables like 'max_connections';

set global max_connections= 500;
```



mysql的server层里有个**连接管理**，它的作用是管理客户端和mysql之间的长连接。

正常情况下，客户端与server层如果只有**一条**连接，那么在执行sql查询之后，只能阻塞等待结果返回，如果有大量查询同时并发请求，那么**后面的请求都需要等待前面的请求执行完成**后，才能开始执行。

SQL执行较慢时可以查看是否是因为连接数过小导致。



### innodb_buffer_pool_size

```mysql
show global variables like 'innodb_buffer_pool_size';
set global innodb_buffer_pool_size = 536870912;
```



# MySQL 索引

索引有很多种类型，可以为不同的场景提供更好的性能。在 MySQL 中，**索引是存储引擎层实现的**，并没有统一的索引标准，即使多个存储引擎支持同一种类型的索引，其底层的实现也可能不同。

索引的根本原理就是降低硬盘的io次数。



> 其实简单来说，索引就是一个排好序的数据结构



**索引优势**

1. 加快查询和分组、排序的速度，降低数据库的IO成本以及CPU的消耗。
2. 通过创建唯一索引，可以保证每一行数据的唯一性。
3. 加速表与表的连接。
4. 显著的减少查询中分组和排序的时间



**索引劣势**

1. 增删改操作时需要更新索引，会降低数据操作的性能：
  2. 新增：自然需要在索引树中新增节点；
  3. 删除：索引树中指向的记录可能会失效，意味着这棵索引树很多节点，都是失效的；
  4. 改动：索引树中节点的**指向**可能需要改变。
5. 创建索引时需要对表加锁，在锁表的同时，可能会影响到其他的数据操作。
6. 索引需要磁盘的空间进行存储，如果针对单表创建了大量的索引，可能比数据文件更快达到大小上限。
7. 当对表中的数据进行CRUD的时，也会触发索引的维护，而维护索引需要时间，可能会降低数据操作的性能。
8. 创建索引需要时间，后期创建的索引，创建开销时间与表数据量呈正相关
9. 占用磁盘空间；
10. 增加查询优化器的负担；



## 索引类型

### 数据结构角度

#### B+Tree 索引

>  B-Tree能加快数据的访问速度，因为存储引擎不再需要进行全表扫描来获取数据，数据分布在各个节点之中。最常见的索引类型，大部分索引都支持 B 树索引。



MySQL 中以 B+Tree 的有序的数据结构存储索引数据。

本身是一种平衡的二叉树，每一个叶子节点到根的距离都是相同的，并且记录的所有节点是按键值的大小、顺序放在同一层的叶子节点上的，并且每一个叶子节点是通过指针来连接的。

是B-Tree的改进版本，同时也是数据库索引索引所采用的存储结构。数据都在叶子节点上，并且增加了顺序访问指针，每个叶子节点都指向相邻的叶子节点的地址。相比B-Tree来说，进行范围查找时只需要查找两个节点，进行遍历即可。而B-Tree需要获取所有节点，相比之下B+Tree效率更高。

B+Tree的每个非叶子节点存储了多个索引数据，每个非叶子节点大小为4kb。

MySql索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个**指向相邻叶子节点的链表指针(整体类似一个双向链表的结构)**，就形成了带有顺序指针的B+Tree，提高区间访问的性能。



MyISAM的B+Tree索引的叶子节点上面存储的是主键id的内存地址。

MyISAM的B+Tree索引如果是主键，则叶子节点上面存储的是该主键对应的所有列；如果不是主键，则叶子节点上面存储的是主键数据。

自适应 HASH 索引：自动维护





##### 优点

Btree 索引适用于全值匹配的查询。如

Btree 索引适合处理范围查找。

Btee 索引从索引的最左侧列开始匹配查找列

对于多个列组合成的索引，只能从左开始查找，如



##### 缺陷

- 只能从最左侧开始按索引键的顺序使用索引，不能跳过索引键

    如一个 a_b_c 的联合索引，在过滤的时候使用 a 和 c 列，那么就只能用到 a 列的索引

- NOT IN 和 `<>`（不等于） 操作无法使用索引

- 索引列上不能使用表达式或是函数



##### 和B-Tree对比

- B+Tree的磁盘读写代价更低

B+Tree非叶子结点存储索引数据，仅仅包含索引列和地址指针，同一个结点（磁盘页）中，B+Tree包含的索引数量比B-Tree更多，同样的数据量下，B+Tree会比B-Tree更加“矮胖”，查询时需要的**IO次数更少**。



- B+树的查询效率更加稳定

B+Tree所有索引或行数据都在叶子结点，而叶子结点都属于同一层级，所有查询都是从根结点遍历到叶子结点，时间复杂度相比B-Tree查询**更加稳定**。



- B+树更有利对数据的扫描

B+树中**更有利于对数据扫描**，可以避免B树的回溯扫描。

B-Tree如果需要查询一串相邻的数值，可能需要来回扫描或是从根结点多次中序遍历。而B+Tree的所有元素都存储叶子结点，每个叶子结点都有指向下一个结点的指针，直接线性遍历即可。B+Tree更加利于做范围查询。



##### 扩展

###### 为什么索引结构默认使用B+Tree，而不是链表、hash、二叉树、红黑树？

- **HASH**

    虽然可以快速定位，但是没有顺序，IO复杂度高。

    

- **二叉树**

    树的高度不均匀，不能自平衡，查找效率跟树的高度有关，并且IO代价高。在极端情况下等同于链表，此时查询相当于全表扫描。

    

- **红黑树**

    树的高度不可控，随着数据量增加而增加，IO代价高。



###### 为什么官方建议使用自增长主键作为索引

结合B+Tree的特点，自增主键是连续递增的，在插入过程中能减少页分裂，即使要进行页分裂，也只会分裂很少一部分。并且能减少数据的移动，每次插入都是插入到最后。总之就是减少分裂和移动的频率。



###### B+Tree索引页能存储多少数据

一个索引页默认大小16kb，整数（`bigint`）字段的长度为8B，另外还跟着6B的指向其子树的指针，这意味着一个索引页可以存储接近1200条数据(16kb/14B ≈ 1170)。

- 树的根节点存储1200条索引目录，占用16kb内存。
- 树的第二层总共是1200个索引页，每个索引页存放1200条索引目录，就有144w条索引目录，占用1200 \* 16KB = 20M内存。
- 树的第三层1200 \* 1200 = 144w页，144w \* 16kB = 23G，此时不适合存放到内存中。



###### 根据主键ID查询流程

- 内存中直接获取树根索引页，对树根索引页内的目录进行二分查找，定位到第二层的索引页
- 内存中直接获取第二层的索引页，对索引页内的目录进行二分查找，定位到第三层的索引页
- 从磁盘加载第三层的索引页到内存中，对索引页内的目录进行二分查找，定位到第四层数据页
- 从磁盘加载第四层的数据页到内存中，数据页变成缓存页，对缓存页中的目录进行二分查找，定位到具体的行数据



#### Hash索引

基于哈希表实现，只有精确匹配索引所有列的查询才有效，对于每一行数据，存储引擎都会对所有的索引列计算一个哈希码（hash code），并且Hash索引将所有的哈希码存储在索引中，同时在索引表中保存指向每个数据行的指针。



##### 优点

- 通过对Key进行散列值计算，可以直接得到对应数据的存放位置，它的查询效率能够达到O(1)。



##### 缺陷

- Hash索引仅仅能满足“=”，“in”查询条件，不能使用范围查询。
- 不能用部分索引键来搜索，因为组合索引在计算哈希值的时候是一起计算的。
- 哈希索引也没办法利用索引完成排序
- 不支持最左匹配原则
- 存在大量哈希冲突（Hash碰撞）的情况下，哈希索引的效率也是极低的


#### R-Tree索引

空间索引，是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少。



### 主键角度

#### 主键索引

MySQL在创建表时如果设置了主键（Primary Key），则会默认在主键列上设置索引，即主键索引（Primary Index）。

**在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。**



> 在InnoDB里，主键索引也被称为**聚簇索引**（Clustered Index）。



##### 特点

1. 数据唯一，不能为NULL



#### 辅助索引

与主键索引相对的即是**辅助索引**（Secondary Index），辅助索引也被称为**二级索引**、**次级索引**。



### 应用层次角度

#### 普通索引

即一个索引只包含单个列，一个表可以有多个单列索引。

普通索引的唯一作用就是为了快速查询数据，一张表允许创建多个普通索引，并允许数据重复和 NULL。



#### 组合索引

一个索引包含多个列。组合索引和字符串列上的索引都有一个最左匹配原则，查询条件中必须要有索引最左一列才会生效，而如果中间跳过一列则跳过的列及之后的列都不会生效。

有时有可能因为多个列名过长，导致组合索引树的键大小过大，降低了存储和查询的效率。所以为了避免出现这样的情况，可以适当的保证每一列的名字不要太长，或只取组合索引每一列前几个字符组成索引。

**组合索引实际还是一个索引，并非真的创建了多个索引，只是产生的效果等价于产生多个索引。**



**示例**

| 索引名称           | 字段               | 索引类型 | 索引方法 |
| ------------------ | ------------------ | -------- | -------- |
| idx_classify_title | classify_id, title | NORMAL   | BTREE    |



##### 优点

- 减少开销

组合索引(col1,col2,col3)实际产生的作用等价于建了(col1),(col1,col2),(col1,col2,col3)三个索引。索引也需要磁盘空间，每多一个索引，不仅增加磁盘空间的开销，还多了一棵索引的查询和维护。对于大量数据的表，使用联合索引会大大的减少磁盘空间和执行开销！



- 覆盖索引

如果查询的列都包括在组合索引中，那么MySQL可以只遍历一次该组合索引，便可以取到`col1,col2,col3`三列的数据，而无需回表，这就减少了很多IO操作。



- 效率高

索引列越多，通过索引筛选出的数据越少。



#### 唯一索引

唯一索引也是一种约束。**唯一索引的属性列不能出现重复的数据，一张表允许创建多个唯一索引。** 建立唯一索引的目的大部分时候都是为了该属性列的数据的唯一性，而不是为了查询效率。

在数据库的角度中，`NULL` != `NULL` , 所以唯一索引列，可以有多个空值。

每张表可以有多个唯一索引，但是只能有一个Primary索引。



#### 前缀索引

前缀索引只适用于字符串类型的数据。前缀索引是对文本的前几个字符创建索引，相比普通索引建立的数据更小， 因为只取前几个字符。



#### 空间索引

> 不常用

空间索引，空间索引是对空间数据类型的字段建立的索引，MYSQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。



#### 全文索引

> 不常用

全文索引（MySQL 5.6 之后InnoDB才支持），它是模糊匹配的一种更好的解决方案，它的效率要比使用`like %`更高，并且它还支持多种匹配方式，灵活性也更加强大。只有字段的数据类型为 char、varchar、text 及其系列才可以建全文索引。

全文索引主要是为了检索大文本数据中的关键字的信息，是目前搜索引擎数据库使用的一种技术。Mysql5.6 之前只有 MYISAM 引擎支持全文索引，5.6 之后 InnoDB 也支持了全文索引

全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从Mysql5.6版本开始支持全文索引。

说是全文索引，其实它只代表可以实现全文检索的功能，而全文检索的底层实现，实际就是倒排索引，所以你也可以把全文索引的本质，当做是倒排索引。再了解全文检索之前，务必要先了解倒排索引

为什么叫倒排索引？

因为英文单词Inverted有颠倒的意思，然后可能就被翻译为倒排索引，也有很多地方叫反向索引。相对而言的索引就是正排索引（forward index）
倒排索引和正排索引，我的个人理解是，它与我们之前说的B+索引，哈希索引的应用不同。拿这些索引的知识去理解正排索引和倒排索引，感觉有些难以联系起来，容易混乱。因为正排，倒排是属于搜索引擎范畴的概念，所以理解正排，倒排最好还是先对搜索引擎有一些了解。比如使用过ElasticSearch
什么是倒排索引（Inverted index）？

说实话，倒排索引也不是一个简单的概念，很多搜索引擎的底层原理都是倒排索引，比如ElasticSearch等。
学习倒排索引的时候，最好还是需要一些搜索引擎的知识会更容易理解。

倒排索引就是相对正排索引相反而言的模型。正排索引是通过文档ID来遍历文档内容，找到关键字（文档 -> 关键字）。而倒排索引则是通过关键字，找到所在的文档（关键字 -> 文档）

以上正排索引，假如我们的搜索引擎是基于正排索引实现的，那么我们要在索引库中查询"python"关键字, 就很可能需要一个一个文档的进行遍历，直到在某个文档的内容中找到"python"单词。这样的一个时间复杂度是很大的，尤其是数据量很大的情况下。所以基于正排索引实现的搜索引擎是不现实的

如果在每一文档录入搜索引擎索引库的时候，我们就对文档的内容进行分词，统计，分析。建立倒排索引，记录分词得到的每个单词对应文档的出现位置和出现次数。 那么我们在搜索引擎查询"python"关键字，就会非常的快速，只需要通过关键字就可以马上查询到已经预处理的统计分析结果。也就知道了'python'关键字在文档A和文档B出现过。展示给用户即可。

这也是为什么是正排和倒排的原因，正排是通过文档查找关键字，倒排是通过关键字查到对应的文档。详细的倒排索引知识以后有时间再在ES的知识点中重点说明。


### 数据存储角度

#### 聚簇索引

聚簇索引（Clustered Index）又称`聚集索引`，将根据键值对数据库表中的数据进行排序存储，并将相关的信息聚簇在一起索引就叫聚簇索引（**该顺序是物理上连续的存储空间的顺序**）。

只有InnoDB支持聚簇索引，若存在主键，则以该主键列生聚簇索引树；若没有主键，则以**第一个唯一非空列**的索引作为聚簇索引；以上条件皆不满足，InnoDB会在内部生成一个名为`GEN_CLUST_INDEX`隐式聚簇索引。该索引是基于一个名为`DB_ROW_ID`的隐藏字段，通常称之隐式主键。



##### 特点

1. 定义了数据存储在表中的顺序，该表的数据有且仅以一种方式排序。因此，**每个表只能有一个聚簇索引**；
2. 聚簇索引其中一个大特征就是将索引和数据存储在同一个文件中，叶子结点不仅保存键的信息，还保存了位于同一行其他列的信息，**聚簇索引的叶子结点保存的是一个完整行记录数据**；
3. 聚簇索引是一种有序索引，它的具体实现可以是稠密索引，也可以是稀疏索引。



##### 优点

- **聚簇索引可以将相关的数据紧密的关联起来，存储在相邻的连续物理空间，利于范围查询**，比如将相关的数据存放在一个叶子结点上，既一个结点的多个关键字对应的数据都存储在一个数据页中，范围查询时，磁盘一次Load出即可，降低IO操作次数，比如针对MAX, MIN, COUNT等聚集函数都有很好的作用；

- **聚簇索引将数据和索引存储在同一个数据文件。** 既聚簇索引的叶子结点不仅存放键的信息，还存储相关其他列的完全数据。当查询走聚簇索引，不需要中间人跳转，直接就可以获得目的数据，查询效率更快。



##### 缺陷

- 因为聚簇索引是顺序存储的，如果多次的插入操作是以非顺序的方式执行，那么最终聚簇索引需要不断的维护这个顺序，这是需要一定性能消耗的；
- 当聚簇索引中的主键发生更改时，可能需要重新维护顺序，迫使物理空间的交换，所以聚簇索引需要更长的时间来更新记录；
- 支持聚簇索引的存储引擎的辅助键索引的查询结果只是一个中间结果，还需要通过中间结果到聚簇索引上二次查询，即回表，操作相对繁琐。



##### 扩展

###### 为什么InnoDB存储引擎一定要有聚簇索引呢？

如果SQL条件是一个非主键列的数据，那么在索引查询中，很可能需要跨树查询，既两次查询。

因为InnoDB的辅助索引的叶子结点并不存储行数据，而是对应的主键值。查询时需要根据辅助键索引查询到的主键值，再去聚簇索引中查询。



#### 非聚簇索引

非聚簇索引（No-Clustered Index）将数据存储在一个位置，将索引存储在另一个位置，索引包含指向该数据位置的指针。



##### 特点

1. 一个表中可以包含多个非聚簇索引；
2. InnoDB的辅助键索引，以及MyISAM的主、辅索引都是非聚簇索引；
3. 非聚簇索引只存储键与指针，不存储数据，所以非聚簇索引的叶子结点仅保存数据的地址(`MyISAM`)或是其主键信息(`InnoDB`)。



##### 优点

- **因为一张表只能有一个聚簇索引，而非聚簇索引则可以有多个**；
- **非聚簇索引占用空间小**，因为叶子结点不存储真实数据，所以非聚簇索引相比聚簇索引更小；
- 在部分查询中，**可以利用覆盖索引的特性，加快查询速度**，直接从辅助键索引中获得想要的数据，而不需要做二次查询；
- **非聚簇索引只需要一次遍历，便可得到数据地址。**支持聚簇索引的存储引擎会导致其辅助键索引查询的结果只是一个中间结果，还需要通过该中间结果在聚簇索引再遍历一次。而不支持聚簇索引的存储引擎，只有非聚簇索引，非聚簇索引只需要一次遍历，即可得到真实数据的地址。



##### 缺陷

1. 非聚集索引只能按逻辑顺序存储数据，并不允许以物理空间连续的方式对数据行进行顺序存储。既非聚簇索引一个叶子结点内部的所有关键字仅仅是逻辑顺序的维护。一个结点对应真实数据在数据文件中可能并非按连续物理空间存储的。 相对聚簇索引的查询，IO次数可能更多，查询性能更低；
2. 相比聚簇索引，范围查询更慢，因为聚簇索引的范围查询可以让磁盘一次load出整个结点的数据线性遍历。虽然非聚簇索引的同叶子结点之间的关键字也是逻辑顺序存储，也可以线性遍历，但每线性遍历一个关键字都需要中间再跳转到另一个地方(InnoDB下的聚簇索引)遍历或(MyISAM下的数据文件)访问 。这个中间过程实际都是不同的IO操作，可能触发磁盘不同盘块的数据读取。所以本质还是会造成大量的IO操作；
3. 每当聚簇索引的主键值更新时，可能会触发非聚簇索引的更新，因为非聚簇索引的叶子结点可能存放的是主键信息（比如InnoDB）；
4. 每当数据文件中的数据发生更新时, 也可能会触发非聚簇索引的更新，因为可能会导致非聚簇索引叶子结点的数据地址发生改变（比如MyISAM）。



### 索引密度角度
#### 稠密索引

稠密索引（Dense Index）是有序的，包含所有数据对应的索引，索引数据包含关键字段值和指向数据的指针。

>**Dense Index :**
>
>- An index record appears for every search key value in file.
>- This record contains search key value and a pointer to the actual record.



##### 特点

- 稠密索引的真实数据是按顺序储存的；
- 为每一个键都创建一个索引记录；
- 每个索引记录都包含键本身和指向实际数据的指针；
- 因为每个键都有索引，所以可以直接通过索引就找到目的键对应的数据。



#### 稀疏索引

稀疏索引（Sparse Index）是有序的，包含部分数据对应的索引，索引数据包含关键字段值和指向数据的指针。

>**Sparse Index :**
>
>- Index records are created only for some of the records.
>- To locate a record, we find the index record with the largest search key value less than or equal to the search key value we are looking for.
>- We start at that record pointed to by the index record, and proceed along the pointers in the file (that is, sequentially) until we find the desired record.



##### 特点

- 稀疏索引的真实数据是按顺序存储的；
- 只为部分的键创建索引记录；
- 当在稀疏索引中查找某个目的键时，通常会通过索引，先找到小于或等于目的键的其他键的数据项，既通过索引找到比目的键值要小的数据项（如果目的键有索引，就直接找到目的键的数据）。然后在数据项按顺序遍历(线性)，直到找到目的键的数据记录。



##### 稠密索引和稀疏索引的优缺点

- 相对某列键而言，稠密索引对每个数据都建有索引，要查询起来，直接快速。但是因为要为每个数据都建立对应的索引，所以需要比较大的空间资源
- 而稀疏索引因为只针对部分数据建立索引，所以空间资源占用小，但是查询效率相对比较慢

## 存储引擎支持

### 数据结构角度

| 索引        | INNODB                               | MYISAM | MEMORY | HEAP | NDB  |
| ----------- | ------------------------------------ | ------ | ------ | ---- | ---- |
| BTREE索引   | 支持                                 | 支持   | 支持   |      |      |
| HASH 索引   | 不支持<br />（支持自适应的HASH索引） | 不支持 | 支持   | 支持 | 支持 |
| R-tree 索引 | 不支持                               | 支持   | 不支持 |      |      |



### 主键角度

| 索引     | INNODB引擎 | MYISAM引擎 | MEMORY引擎 |
| -------- | ---------- | ---------- | ---------- |
| 主键索引 | 支持       | 支持       | 支持       |
| 辅助索引 | 支持       | 支持       | 支持       |



### 应用层次角度

| 索引               | INNODB引擎      | MYISAM引擎 | MEMORY引擎 |
| ------------------ | --------------- | ---------- | ---------- |
| 全文索引(FULLTEXT) | 5.6版本之后支持 | 支持       | 不支持     |



### 数据存储角度

| 索引       | INNODB引擎 | MYISAM引擎 | MEMORY引擎 |
| ---------- | ---------- | ---------- | ---------- |
| 聚簇索引   | 支持       | 不支持     | ？         |
| 非聚簇索引 | 支持       | 支持       | ？         |



## 索引的维护

### B+Tree索引

B+Tree索引是一个有序的树形结构，因此每次增删改涉及到索引的数据时，都会对索引的数据页和索引页进行重新排列（回收和分裂）。



> 删除操作并不会立即进行数据页内记录的重排列，而是会给被删除的记录打上一个删除的标识，等到合适的时候，再把记录从链表中移除，但是总归需要涉及到排序的维护，势必要消耗性能。



## 索引配置

### 索引下推

索引下推能够减少**二级索引**在查询时的回表操作，提高查询的效率，因为它将 Server 层部分负责的事情，交给存储引擎层去处理了。

联合索引当遇到范围查询 (>、<、between、like) 就会停止匹配，也就是 **a 字段能用到联合索引，但是 reward 字段则无法利用到索引**。

那么，不使用索引下推（MySQL 5.6 之前的版本）时，执行器与存储引擎的执行流程是这样的：

- Server 层首先调用存储引擎的接口定位到满足查询条件的第一条二级索引记录，也就是定位到 age > 20 的第一条记录；
- 存储引起根据二级索引的 B+ 树快速定位到这条记录后，获取主键值，然后**进行回表操作**，将完整的记录返回给 Server 层；
- Server 层在判断该记录的 reward 是否等于 100000，如果成立则将其发送给客户端；否则跳过该记录；
- 接着，继续向存储引擎索要下一条记录，存储引擎在二级索引定位到记录后，获取主键值，然后回表操作，将完整的记录返回给 Server 层；
- 如此往复，直到存储引擎把表中的所有记录读完。

可以看到，没有索引下推的时候，每查询到一条二级索引记录，都要进行回表操作，然后将记录返回给 Server，接着 Server 再判断该记录的 reward 是否等于 100000。

而使用索引下推后，判断记录的 reward 是否等于 100000 的工作交给了存储引擎层，过程如下 ：

- Server 层首先调用存储引擎的接口定位到满足查询条件的第一条二级索引记录，也就是定位到 age > 20 的第一条记录；
- 存储引擎定位到二级索引后，**先不执行回表**操作，而是先判断一下该索引中包含的列（reward列）的条件（reward 是否等于 100000）是否成立。如果**条件不成立**，则直接**跳过该二级索引**。如果**成立**，则**执行回表**操作，将完成记录返回给 Server 层。
- Server 层在判断其他的查询条件（本次查询没有其他条件）是否成立，如果成立则将其发送给客户端；否则跳过该记录，然后向存储引擎索要下一条记录。
- 如此往复，直到存储引擎把表中的所有记录读完。

可以看到，使用了索引下推后，虽然 reward 列无法使用到联合索引，但是因为它包含在联合索引（age，reward）里，所以直接在存储引擎过滤出满足 reward = 100000 的记录后，才去执行回表操作获取整个记录。相比于没有使用索引下推，节省了很多回表操作。

当你发现执行计划里的 Extr 部分显示了 “Using index condition”，说明使用了索引下推。



#### ICP索引下推

> To see how this optimization works, consider first how an index scan proceeds when Index Condition Pushdown is not used:
>
> 1. Get the next row, first by reading the index tuple, and then by using the index tuple to locate and read the full table row.
> 2. Test the part of the `WHERE` condition that applies to this table. Accept or reject the row based on the test result.
>
> When Index Condition Pushdown is used, the scan proceeds like this instead:
>
> 1. Get the next row's index tuple (but not the full table row).
> 2. Test the part of the `WHERE` condition that applies to this table and can be checked using only index columns. If the condition is not satisfied, proceed to the index tuple for the next row.
> 3. If the condition is satisfied, use the index tuple to locate and read the full table row.
> 4. Test the remaining part of the `WHERE` condition that applies to this table. Accept or reject the row based on the test result.



##### ICP实践

现有如下联合索引：

| 索引名称           | 字段               | 索引类型 | 索引方法 |
| ------------------ | ------------------ | -------- | -------- |
| idx_classify_title | classify_id, title | NORMAL   | BTREE    |



执行如下SQL获取 `classify_id` 为1且 `title` 中包含'异常设计'的数据：

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` = 1 AND `title` LIKE '%异常设计%'
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra                 |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :-------------------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 11.11    | Using index condition |



由于联合索引的叶子节点的记录是先按照 `classify_id` 字段排序，`classify_id` 字段相同的情况下再按照 `title` 字段排序，因为把`%`加在 `title` 字段前面时无法利用索引的顺序性来进行快速比较的，也就是说这条查询语句中只有 `classify_id` 字段可以使用索引进行快速比较和过滤。正常情况下查询过程是这个样子的：

1. InnoDB使用联合索引查出所有 `classify_id` 为1的二级索引数据，得到主键值；
2. 拿到主键索引进行回表，到聚簇索引中拿到完整记录；
3. InnoDB把完整记录返回给MySQL的Server层，在Server层过滤出 `title` 包含'异常设计'的数据。



此时如果 `classify_id` 查询出来的结果较多，就需要将大量数据返回给Server层进行过滤。但是实际上是可以在InnoDB存储引擎中过滤出`title` 包含'异常设计'的数据主键的。这种在存储引擎中提前过滤数据的行为就称为索引下推（Index Condition Pushdown，**ICP**），或索引条件下推。

准确来说就是过滤的动作由下层的存储引擎层通过使用索引来完成，而不需要上推到Server层进行处理，使用ICP的方式能有效减少回表的次数。

ICP是在MySQL5.6之后完善的功能，是默认开启的，对于二级索引，只要能把条件甩给下面的存储引擎，存储引擎就会进行过滤，不需要进行干预。



> 即使满足索引下推的使用条件，查询优化器也未必会使用索引下推，因为可能存在更高效的方式。



##### ICP设置

- 查看索引下推设置

```mysql
## 查看优化器开关
SHOW VARIABLES LIKE 'optimizer_switch';
```

​	

| Variable_name    | Value                                 |
| ---------------- | ------------------------------------- |
| optimizer_switch | ..., index_condition_pushdown=on, ... |



- 开启索引下推

```mysql
SET optimizer_switch="index_condition_pushdown=on";
```



- 关闭索引下推

```mysql
SET optimizer_switch="index_condition_pushdown=off";
```



关闭索引下推后再执行同样的SQL，其执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 11.11    | Using where |



和打开索引下推时的执行计划区别在于Extra信息发生变化，没有再使用索引条件。



### 索引潜水

> MySQL预估扫描行数更准确，可以选择更合适的索引。



**索引潜水（Index dive）**指预估扫描行数的方式。



```mysql
# 获取IN条件参数个数临界值，是否触发索引潜水获取扫描行数
show variables like '%eq_range_index_dive_limit%';
# 配置临界值大小
set eq_range_index_dive_limit=200;
```



是否发生索引潜水根据参数**eq_range_index_dive_limit**和查询条件IN的参数决定。

**MySQL5.7.3**之前的版本，**eq_range_index_dive_limit**值默认是10，之后的版本，这个值默认是200。

当where语句in条件中参数个数小于**eq_range_index_dive_limit**值的时候，MySQL就采用**索引潜水（Index dive）**的方式预估扫描行数，非常准确。当where语句in条件中参数个数大于等于这个值的时候，MySQL就采用另一种方式**索引统计**（Index statistics）预估扫描行数，误差较大。MySQL预估扫描行数更准确，可以选择更合适的索引。

基于成本的考虑，**索引潜水**估算成本较高，适合小数据量。**索引统计**估算成本较低，适合大数据量。一般情况下，where语句的in条件的参数不会太多，适合使用**索引潜水**预估扫描行数。



## 最佳实践

### 正确姿势

#### 离散度高的列

离散度公式：`COUNT(DISTINCT(column_name)) / COUNT(*)`，列的不重复值的个数与所有数据行的比例。简而言之，如果列的重复值越多，列的离散度越低。重复值越少，离散度就越高。

为离散度高的列创建索引时会在二级索引中搜索到大量的重复数据，然后进行大量回表操作。



#### 搜索、排序或分组的列

只为出现在`WHERE`子句中的列或者出现在`ORDER BY`和`GROUP BY`子句中的列创建索引即可。仅出现在查询列表中的列不需要创建索引。



#### 联合索引

**不要为联合索引的第一个索引列单独创建索引**，因为联合索引本身就是先按照第一个索引列进行排序，因此联合索引对第一个索引列的搜索是有效的，不需要单独为`name`再创建索引了。

因此在建立联合索引的时候，要注意如下几点：

1. 区分度最高的列放在联合索引的最左侧
2. 使用最频繁的列放到联合索引的最左侧
3. 尽量把字段长度小的列放在联合索引列的最左侧



#### 过长的列

如果一个字符串格式的列占用的空间比较大（就是说允许存储比较长的字符串数据），为该列创建索引，就意味着该列的数据会被完整地记录在每个数据页的每条记录中，会占用相当大的存储空间。

对此，可以为该列的前几个字符创建索引，也就是在二级索引的记录中只会保留字符串的前几个字符。比如可以为`phone`列创建索引，索引只保留手机号的前3位：

```sql
ALTER TABLE user_innodb ADD INDEX IDX_PHONE_3 (phone(3));
```

然后执行下面的SQL语句：

```sql
EXPLAIN SELECT * FROM user_innodb WHERE phone = '1320';
```

由于在`IDX_PHONE_3`索引中只保留了手机号的前3位数字，所以我们只能定位到以132开头的二级索引记录，然后在遍历所有的这些二级索引记录时再判断它们是否满足第4位数为0的条件。



**当列中存储的字符串包含的字符较多时，为该字段建立前缀索引可以有效节省磁盘空间**。



#### 较短的列

索引创建之后也是使用硬盘来存储的，因此提升索引访问的I/O效率，也可以提升总体的访问效率。假如构成索引的字段总长度比较短，那么在给定大小的存储块内可以存储更多的索引值，相应的可以有效的提升MySQL访问索引的I/O效率。



#### 定长的列

尽可能使用定长数据类型，用char代替varchar，固定长度的数据处理比变长的快些。

万一出现数据表崩溃，使用固定长度数据行的表更容易重新构造。使用固定长度的数据行，每个记录的开始位置都是固定记录长度的倍数，可以很容易被检测到，但是使用可变长度的数据行就不一定了。

对于MyISAM类型的数据表，虽然转换成固定长度的数据列可以提高性能，但是占据的空间也大。



#### 设置默认值而非NULL

不是说使用了is null或者 is not null就会不走索引了，这个跟mysql版本以及查询成本都有关；

如果`mysql`优化器发现，走索引比不走索引成本还要高，就会放弃索引，这些条件 `!=，<>，is null，is not null`经常被认为让索引失效；

其实是因为一般情况下，查询的成本高，优化器自动放弃索引的；

如果把`null`值，换成默认值，很多时候让走索引成为可能，同时，表达意思也相对清晰一点；



#### 组合索引排序（未验证）

排序时应按照组合索引中各列的顺序进行排序，即使索引中只有一个列是要排序的，否则排序性能会比较差。



```MySQL
create index IDX_USERNAME_TEL on user(deptid,position,createtime);
select username,tel from user where deptid= 1 and position = 'java开发' order by deptid,position,createtime desc;
```

实际上只是查询出符合 `deptid= 1 and position = 'java开发'`条件的记录并按createtime降序排序，但写成order by createtime desc性能较差。



### 错误姿势
#### 索引越多越好

不是越多越好，维护也需要时间和空间代价，建议单张表索引不超过 5 个。

1. 索引并不是越多越好，虽其提高了查询的效率，但却会降低插入和更新的效率，`insert`或`update`时有可能会重建索引，如果数据量巨大，重建将进行记录的重新排序，所以建索引需要慎重考虑，视具体情况来定；；
2. 索引可以理解为一个就是一张表，其可以存储数据，其数据就要占空间；
3. 索引表的数据是排序的，排序也是要花时间的；



##### 空间上的代价

索引就是一棵B+数，每创建一个索引都需要创建一棵B+树，每一棵B+树的节点都是一个数据页，每一个数据页默认会占用16KB的磁盘空间，每一棵B+树又会包含许许多多的数据页。所以，大量创建索引会消耗大量磁盘空间。



##### 时间上的代价

MySQL 执行优化器在选择如何优化查询时，会根据统一信息，对每一个可以用到的索引来进行评估，以生成出一个最好的执行计划，如果同时有很多个索引都可以用于查询，就会增加 MySQL 优化器生成执行计划的时间，同样会降低查询性能。

数据更新时会维护过多索引导致效率降低。



#### 数据量小的表建立索引

数据量过小，建索引等于多此一举，还增加了操作复杂度。



#### 频繁更新的列

数据频繁更新，触发索引频频维护（数据页回收和分裂），降低写速度。



#### 随机无序的列

随机无序的值，不建议作为索引，例如身份证、UUID。



### 索引失效

现有如下联合索引：

| 名                 | 字段               | 索引类型 | 索引方法 |
| ------------------ | ------------------ | -------- | -------- |
| idx_classify_title | classify_id, title | NORMAL   | BTREE    |
| uk_url             | url                | UNIQUE   | BTREE    |



#### 最左前缀原则

MySQL根据联合索引查询时，会按从左往右的字段依次进行排序查询。查询条件中的字段依次匹配上索引字段，就能根据对应的索引查询，如果中间有中断，没有匹配上，则从中断位置及后面的索引字段都无法根据索引查询。

因此如果联合索引最左边的字段没有匹配上，则不会根据该联合索引进行查询。



##### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE `title` LIKE '异常设计%';
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1995 | 11.11    | Using where |



联合索引的B+树数据页内的记录首先按照 `classify_id` 字段进行排序，`classify_id` 字段相同的情况下，再按照 `title` 字段进行排序。所以，如果我们直接使用 `title` 字段进行搜索，无法利用索引的顺序性。



##### 正确用法

```mysql
SELECT * FROM `it_blog` WHERE `title` LIKE '异常设计%' AND `classify_id` = 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys      | key                | key_len | ref  | rows | filtered | Extra                 |
| :--- | :---------- | :------ | :--------- | :---- | :----------------- | :----------------- | :------ | :--- | :--- | :------- | :-------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | idx_classify_title | idx_classify_title | 1030    | NULL | 1    | 100.00   | Using index condition |



加上 `classify_id` 的搜索条件，就会使用到联合索引，而且不需要在意 `classify_id` 在 `WHERE` 子句中的位置，因为查询优化器会进行优化。



#### 使用反向查询（!=, <>,NOT LIKE）

MySQL在使用反向查询（!=, <>, NOT LIKE）的时候无法使用索引，会导致全表扫描，覆盖索引除外。



##### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` <> 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | idx_classify_title | NULL | NULL    | NULL | 1995 | 50.38    | Using where |

查询 `classify_id` 不为1的数据时，MySQL会进行全表扫描，获取所有的数据。



##### 正确用法

```mysql
SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` <> 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys      | key                | key_len | ref  | rows | filtered | Extra                    |
| :--- | :---------- | :------ | :--------- | :---- | :----------------- | :----------------- | :------ | :--- | :--- | :------- | :----------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | idx_classify_title | idx_classify_title | 8       | NULL | 1005 | 100.00   | Using where; Using index |

同样查询 `classify_id` 不为1的数据时，但由于此时查询的所有数据都在索引页中，因此MySQL会进行优化，在索引页中进行查询，而不是全表扫描。



##### 主键<>查询

```mysql
SELECT * FROM `it_blog` WHERE `id` <> 1
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :---- | :------------ | :------ | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL | 1057 | 100.00   | Using where |

在主键反向查询这种特殊的场景下，也是能够使用主键索引的。



#### LIKE以通配符开头

在进行LIKE模糊匹配时，如果匹配条件中的通配符在左边，则无法使用索引查询。因为字符串的模糊匹配和联合索引类似，也是从左往右将字符串中的字符依次匹配，如果通配符在左边，就无法匹配最左边的字符，因此会导致索引失效。

和反向查询一样，覆盖索引除外，因为所有数据都可以在索引页中获取。



##### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE `url` LIKE '%https://blog.csdn.net%';
SELECT * FROM `it_blog` WHERE `url` LIKE '%https://blog.csdn.net';
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1995 | 11.11    | Using where |



当使用 `url LIKE '%https://blog.csdn.net'` 或者 `url LIKE '%https://blog.csdn.net%'` 这两种方式都会使索引失效，因为联合索引的B+树数据页内的记录首先按照 `url ` 字段进行排序，这两种搜索方式无法从 `url ` 字段最左边开始匹配，自然就无法使用索引，只能通过全表扫描的方式进行查询。



##### 正确用法

```mysql
SELECT * FROM `it_blog` WHERE `url` LIKE 'https://blog.csdn.net%';
SELECT * FROM `it_blog` WHERE `url` LIKE 'https://blog.csdn.net';
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key    | key_len | ref  | rows | filtered | Extra                 |
| :--- | :---------- | :------ | :--------- | :---- | :------------ | :----- | :------ | :--- | :--- | :------- | :-------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | uk_url        | uk_url | 1022    | NULL | 9    | 100.00   | Using index condition |



模糊匹配通配符不在左边时就可以使用索引，根据 `url` 字段从最左边开始匹配从索引页中获取数据。



#### 对索引列做任何操作

如果不是单纯使用索引列，而是对索引列做了其他操作，例如数值计算、使用函数、（手动或自动）类型转换等操作，会导致索引失效。



##### 使用函数

###### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE LEFT(`url`, 24) = 'https://mp.weixin.qq.com';
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 2020 | 100.00   | Using where |



因为对索引字段 `url` 做了函数操作，导致索引失效。



###### 正确用法

```mysql
SELECT * FROM `it_blog` WHERE `url` LIKE 'https://mp.weixin.qq.com%';
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key    | key_len | ref  | rows | filtered | Extra                 |
| :--- | :---------- | :------ | :--------- | :---- | :------------ | :----- | :------ | :--- | :--- | :------- | :-------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | uk_url        | uk_url | 1022    | NULL | 402  | 100.00   | Using index condition |



##### 使用表达式

###### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE `id` + 1 = 2;
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 2020 | 100.00   | Using where |



因为在主键id查询时，对id字段使用了表达式，导致索引时效。



###### 正确用法

```mysql
SELECT * FROM `it_blog` WHERE `id` = 2 - 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
| :--- | :---------- | :------ | :--------- | :---- | :------------ | :------ | :------ | :---- | :--- | :------- | :---- |
| 1    | SIMPLE      | it_blog | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL  |



##### 使用类型转换

###### 错误用法

```mysql
SELECT * FROM `it_blog_classify` WHERE `title` = 3;
```

​	执行计划如下：

| id   | select_type | table            | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :--------------- | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog_classify | NULL       | ALL  | idx_title     | NULL | NULL    | NULL | 10   | 10.00    | Using where |



使用整型条件查询索引字段 `title` 时，由于会发生类型转换，导致无法使用索引。



###### 正确用法

```mysql
SELECT * FROM `it_blog_classify` WHERE `title` = '3';
```

执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key       | key_len | ref   | rows | filtered | Extra |
| :--- | :---------- | :--------------- | :--------- | :---- | :------------ | :-------- | :------ | :---- | :--- | :------- | :---- |
| 1    | SIMPLE      | it_blog_classify | NULL       | const | idx_title     | idx_title | 1022    | const | 1    | 100.00   | NULL  |



```mysql
SELECT * FROM `it_blog_classify` WHERE `id` = '3';
SELECT * FROM `it_blog_classify` WHERE `id` = 3;
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
| :--- | :---------- | :------ | :--------- | :---- | :------------ | :------ | :------ | :---- | :--- | :------- | :---- |
| 1    | SIMPLE      | it_blog | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL  |



因此当索引字段类型为字符串时，使用数字类型进行搜索不会用到索引；而索引字段类型为数字类型时，使用字符串类型进行搜索会使用到索引。



###### 原理解析

MySQL遇到类型转换时，会自动将字符串类型转换为数字类型。

```mysql
SELECT 10 > 9 ## result 1
SELECT '10' > 9 ## result 1
SELECT '10' > '9' ## result 0
```

如上所示，在比较字符串类型10和数字类型9的大小时，结果与数字类型10和数字类型9的比较结果一致。因此可以得知MySQL在字符串类型比较数字类型时，会将字符串类型转换为数字类型。



```mysql
SELECT * FROM `it_blog_classify` WHERE `title` = 3;
## 等价于
SELECT * FROM `it_blog_classify` WHERE CAST(`title` AS signed int) = 3;
```

使用数字类型查询字符串类型字段时，因为在索引字段使用了函数，导致索引失效。



```mysql
SELECT * FROM `it_blog_classify` WHERE `id` = '3';
## 等价于
SELECT * FROM `it_blog_classify` WHERE `id` = CAST('3' AS signed int);
```

使用字符串类型查询数字类型字段时，没有在索引字段添加任何操作，因此不会导致索引失效。



#### OR连接

`OR` 查询时只有条件两边字段都走索引才不会发生索引失效。



##### 错误用法

```mysql
SELECT * FROM `it_blog` WHERE `tag_id` = 1 OR `classify_id` = 1
```

执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | idx_classify_title | NULL | NULL    | NULL | 2112 | 39.70    | Using where |

因为OR查询时，只有 `classify_id` 字段有索引，而 `tag_id` 字段没有设置索引，所以SQL执行了全表扫面。



##### 正确用法

```mysql
SELECT * FROM `it_blog` WHERE `user_id` = 1 OR `classify_id` = 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type        | possible_keys                            | key                                      | key_len | ref  | rows | filtered | Extra                                                        |
| :--- | ----------- | ------- | ---------- | ----------- | ---------------------------------------- | ---------------------------------------- | ------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| 1    | SIMPLE      | it_blog | NULL       | index_merge | idx_user_tag_classify,idx_classify_title | idx_user_tag_classify,idx_classify_title | 8,8     | NULL | 6    | 100.00   | Using sort_union(idx_user_tag_classify,idx_classify_title); Using where |

因为 `user_id` 和 `classify_id` 字段都设置了索引，因此没有发生索引失效，而实根据两个索引条件去查询数据。



## 查看索引使用情况

```sql
-- 查看当前会话索引使用情况
show status like 'Handler_read%';
-- 查看全局索引使用情况
show global status like 'Handler_read%';
```

**Handler_read_first**：索引中第一条被读的次数。如果较高，表示服务器正执行大量全索引扫描（这个值越低越好）。

**Handler_read_key**：如果索引正在工作，这个值代表一个行被索引值读的次数，如果值越低，表示索引得到的性能改善不高，因为索引不经常使用（这个值越高越好）。

**Handler_read_next** ：按照键顺序读下一行的请求数。如果你用范围约束或如果执行索引扫描来查询索引列，该值增加。

**Handler_read_prev**：按照键顺序读前一行的请求数。该读方法主要用于优化ORDER BY ... DESC。

**Handler_read_rnd** ：根据固定位置读一行的请求数。如果你正执行大量查询并需要对结果进行排序该值较高。你可能使用了大量需要MySQL扫描整个表的查询或你的连接没有正确使用键。这个值较高，意味着运行效率低，应该建立索引来补救。

**Handler_read_rnd_next**：在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明你的表索引不正确或写入的查询没有利用索引。



# MySQL 执行计划
## 为什么要关注执行计划？
- 了解 SQL 如何访问表中的数据，是使用全表扫描、还是索引等方式来获取的
- 了解 SQL 如何使用表中的索引，是否使用到了正确的索引
- 了解 SQL 锁使用的查询类型，是否使用到了子查询、关联查询等信息



重点关注这4列就能找到索引问题：

1、key（查看有没有使用索引） 

2、key_len（查看索引使用是否充分）

3、type（查看索引类型） 

4、Extra（查看附加信息：排序、临时表、where条件为false等）



## explain关键字

可通过 EXPLAIN 来获取到 SQL的执行计划

| id   | select_type | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | :---------- | :--------- | :--------- | :--- | :------------ | :--- | :------ | :--- | :--- | :------- | :---------- |
| 1    | SIMPLE      | table_name | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 100  | 33.33    | Using where |



## 执行计划表

### id列

id 会有两种值：

- 数值：查询中的 SQL 数据对数据库对象操作的顺序
	- 当 ID 相同时由上到下执行。
	- 当 ID 不同时，由大到小执行

- NULL：这一行数据是由另外两个查询进行 union 后产生的



#### 数值

##### id数值相同时

```mysql
EXPLAIN SELECT * FROM `it_blog` ib INNER JOIN `it_blog_classify` ibc ON ib.`classify_id` = ibc.`id`
```



执行计划如下：

| id   | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                            | rows | filtered | Extra |
| :--- | ----------- | ----- | ---------- | ------ | ------------- | ------- | ------- | ------------------------------ | ---- | -------- | ----- |
| 1    | SIMPLE      | ib    | NULL       | ALL    | idx_classify  | NULL    | NULL    | NULL                           | 1699 | 100.00   | Null  |
| 1    | SIMPLE      | ibc   | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | l-sixth-service.ib.classify_id | 1    | 100.00   | Null  |



此时SQL执行循序是从上到下执行，先执行`it_blog`表的查询，再执行`it_blog_classify`表的查询。



##### id数值不同时

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



此时SQL执行顺序是由大到小执行，先执行`it_blog_classify`表子查询，再执行外面的`it_blog  `表查询。



#### NULL

```mysql
EXPLAIN SELECT `id`, `title` FROM `it_blog` WHERE `id` = 10
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 20
```



执行计划如下：

| id   | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
| :--- | ------------ | ---------- | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | --------------- |
| 1    | PRIMARY      | it_blog    | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| 2    | UNION        | it_blog    | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL | NULL     | Using temporary |



此时执行计划结果中id为NULL的执行计划表示这一行数据是由另外两个查询进行 union 后产生的。



### select_type列

查询的类型，根据关联、union、子查询等等分类，有以下几种类型：

- SIMPLE：不包含子查询或是 UNION 操作的查询
- PRIMARY：查询中如果包含任何子查询，那么最外层的查询则被标记为 PRIMARY
- SUBQUERY：SEL ECT 列表中的子查询
- DEPENDENT SUBQUERY：依赖外部结果的子查询
- UNION：union 操作的第二个或是之后的查询的值为 union
- UNION RESULT：UNION 产生的结果集
- DEPENDENT UNION：当 UNION 作为子查询时，第二或是第二个后的查询的 select_type 值
- DERIVED：出现在 FROM 子句中的子查询
- MATERIALIZED：



#### SIMPLE

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `id` = 10
```



执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL  |



`select_type` 值为 `SIMPLE` 时表示该查询是简单查询，不包括任何子查询或UNION查询。



#### PRIMARY & SUBQUERY

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



当一个查询语句中包括子查询时，`PRIMARY` 表示该表的查询时子查询的外层查询，`SUBQUERY` 则表示该表是子查询。



#### DEPENDENT SUBQUERY



#### UNION & UNION RESULT

```mysql
EXPLAIN SELECT `id`, `title` FROM `it_blog` WHERE `id` IN (10, 30)
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 20
UNION 
SELECT `id`, `title` FROM `it_blog` WHERE `id` = 40
```



执行计划如下：

| id   | select_type  | table        | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
| :--- | ------------ | ------------ | ---------- | ----- | ------------- | ------- | ------- | ----- | ---- | -------- | --------------- |
| 1    | PRIMARY      | it_blog      | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL  | 2    | 100.00   | Using where     |
| 2    | UNION        | it_blog      | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| 3    | UNION        | it_blog      | NULL       | const | PRIMARY       | PRIMARY | 8       | const | 1    | 100.00   | NULL            |
| NULL | UNION RESULT | <union1,2,3> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL | NULL     | Using temporary |



`UNION` 表示该表的查询操作是联合查询的第二个或是之后的查询；`UNION RESULT` 则表示是联合查询产生的结果集。



#### DEPENDENT UNION



#### DERIVED

```mysql
EXPLAIN SELECT drived_table.* FROM (
	SELECT COUNT(`id`), `classify_id` FROM `it_blog` GROUP BY `classify_id`) drived_table 
WHERE drived_table.`classify_id` = 1
```



执行计划如下：

| id   | select_type | table      | partitions | type  | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ---------- | ---------- | ----- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | PRIMARY     | <derived2> | NULL       | ref   | <auto_key0>   | <auto_key0>  | 8       | const | 10   | 100.00   | NULL        |
| 2    | DERIVED     | it_blog    | NULL       | index | idx_classify  | idx_classify | 8       | NULL  | 1718 | 100.00   | Using index |



`DERIVED` 表示出现在 `FROM` 子句中的子查询。



#### MATERIALIZED

```mysql
EXPLAIN SELECT * FROM `it_blog_classify` WHERE `id` IN (
	SELECT `classify_id` FROM `it_blog` WHERE `classify_id` NOT IN (
		SELECT `id` FROM `it_blog_classify`))
```



执行计划如下：

| id   | select_type  | table            | partitions | type   | possible_keys | key          | key_len | ref                                 | rows | filtered | Extra                    |
| :--- | ------------ | ---------------- | ---------- | ------ | ------------- | ------------ | ------- | ----------------------------------- | ---- | -------- | ------------------------ |
| 1    | PRIMARY      | it_blog_classify | NULL       | ALL    | PRIMARY       | NULL         | NULL    | NULL                                | 10   | 100.00   | Using where              |
| 1    | PRIMARY      | <subquery2>      | NULL       | eq_ref | <auto_key>    | <auto_key>   | 8       | l-sixth-service.it_blog_classify.id | 1    | 100.00   | NULL                     |
| 2    | MATERIALIZED | it_blog          | NULL       | index  | idx_classify  | idx_classify | 8       | NULL                                | 1718 | 100.00   | Using where; Using index |
| 3    | SUBQUERY     | it_blog_classify | NULL       | index  | PRIMARY       | idx_user     | 8       | NULL                                | 10   | 100.00   | Using index              |



### table列

表示执行计划表中的数据是由哪个表输出的

- 表名：指明从哪个表中获取数据，有原始表名，或则别名
- `<unionM,N>`：表示由 ID 为 M.N 查询 union 产生的结果集
- `<derived N>/<subquery N>`：由 ID 为 N 的查询产生的结果集



### partitions列

只有在查询分区表的时候，才会显示分区表的 ID



### type列

通常通过 type 可以知道查询使用的连接类型。



>在 MySQL 中，并不是只有通过 join 产生的关联查询才叫关联查询。
>
>就算只查询一个表，也会认为是一个连接查询



type 按照性能从高到低排列如下：

| 类型            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| system          | 表仅有一行，基本用不到。                                     |
| const           | 表中有且只有一个匹配的行时使用，如对主键或是唯一索引的查询，或者被连接的部分是一个常量值，这是效率最高的连接方式。 |
| eq_ref          | 唯一索引或主键引查找，对于每个索引键，表中只有一条记录与之匹配；是以前面表返回的数据行为基础，对于每一行数据都会从本表中读取一行数据。 |
| fulltext        |                                                              |
| ref             | 非唯一索引查找、返回匹配某个单独值的所有行。                 |
| ref_or_null     | 类似于 ref 类型的查询，但是附加了对 NULL 值列的查询。        |
| index_merge     | 该连接类型表示使用了索引合并优化方法；mysql 5.6 之前，一般只支持一个索引查询，后来支持索引合并之后，可以支持多个索引查询。 |
| unique_subquery | 替换下面的 `IN`子查询，子查询返回不重复的集合。              |
| index_subquery  | 区别于`unique_subquery`，用于非唯一索引，可以返回重复值。    |
| range           | 只检索给定范围的行，使用一个索引来选择行。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时。 |
| index           | 全索引扫描，同 ALL 的区别是，遍历的是索引树。该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小； |
| ALL             | FULL TABLE Scan 权标扫表这是效率最差的连接方式。             |
| NULL            | MySQL不访问任何表或索引，直接返回结果。                      |



#### system



#### const

#### eq_ref



#### ref

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` = 2
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify  | idx_classify | 8       | const | 3    | 100.00   | NULL  |



`ref` 表示非唯一索引查找、返回匹配某个单独值的所有行。



#### fulltext



#### ref_or_null



#### index_merge

```mysql
SELECT * FROM `it_blog` WHERE `user_id` = 1 OR `classify_id` = 1;
```

执行计划如下：

| id   | select_type | table   | partitions | type        | possible_keys                            | key                                      | key_len | ref  | rows | filtered | Extra                                                        |
| :--- | ----------- | ------- | ---------- | ----------- | ---------------------------------------- | ---------------------------------------- | ------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| 1    | SIMPLE      | it_blog | NULL       | index_merge | idx_user_tag_classify,idx_classify_title | idx_user_tag_classify,idx_classify_title | 8,8     | NULL | 6    | 100.00   | Using sort_union(idx_user_tag_classify,idx_classify_title); Using where |



#### unique_subquery



#### index_subquery



#### range

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` IN (1, 2, 3, 4, 5, 6)
```

执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------------ | ------- | ---- | ---- | -------- | --------------------- |
| 1    | SIMPLE      | it_blog | NULL       | range | idx_classify  | idx_classify | 8       | NULL | 9    | 100.00   | Using index condition |



`range` 表示索引范围扫描，常见于 between、`>`、`<` 和 `IN` 查询这样的查询条件。



#### index



#### ALL

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `title` = 'Java'
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1725 | 10.00    | Using where |



`ALL` 表示全表扫描，是效率最低的方式。



#### NULL

```mysql
EXPLAIN SELECT 1
```



执行计划如下：



| id   | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
| :--- | ----------- | ----- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | -------------- |
| 1    | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | NULL     | No tables used |



此时type列的值为NULL，表示没有访问任何表或索引，直接返回结果。



### possible_keys列

指出查询中可能会用到的索引，优化器会把这里边的索引都试一遍，然后选一个开销最小的，如果都不太行，那就直接全表扫描。



> 如果表中建立的索引过多，执行SQL时，优化器就会花费更多的时间在选择索引上。因此一般来说表中的索引个数不要超过5个。



### key列

possible_keys 列出的是可能使用到的索引，key 则是表示实际使用到的索引

- 如果为 NULL，则表示该表中没有使用到索引
- 如果出现的值，没有存在 possible_keys 中，查询可能使用到的是覆盖索引





### key_len列

实际使用索引的最大长度。

**注意：**比如在一个联合索引中，如果有 3 列且总字节长度是 100 字节，key_len 可能少于 100 字节的，比如只有 30 个字节，这就说明在查询中没有到联合索引中的所有列，而是只使用到了联合索引中的部分列。

**注意：** 这一列的字节计算方式是以表中定义列的字节方式，而不是数据的实际长度



### ref列

指出哪些列或常量被用于索引查找；





### rows列

有两方面的含义：

- 根据统计信息预估的扫描的行数

- 在关联查询中，也表示内嵌循环的次数

	每获取一个值，就要对目标表进行一次查找，循环越多，性能就越差



```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `classify_id` NOT IN (SELECT `id` FROM `it_blog_classify`)
```



执行计划如下：

| id   | select_type | table            | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ---------------- | ---------- | ----- | ------------- | -------- | ------- | ---- | ---- | -------- | ----------- |
| 1    | PRIMARY     | it_blog          | NULL       | ALL   | NULL          | NULL     | NULL    | NULL | 1699 | 100.00   | Using where |
| 2    | SUBQUERY    | it_blog_classify | NULL       | index | PRIMARY       | idx_user | 8       | NULL | 10   | 100.00   | Using index |



### filtered列

与 rows 有一定的关系，表示返回结果的行数占需要读取行数（rows）的百分比。也是一个预估值，不太准确，但是可以在一定程度上可以预估 mysql 的查询成本。该值越高，效率也越高。



```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `title` = 'Java' AND `classify_id` = 1
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify  | idx_classify | 8       | const | 2    | 10.00    | Using where |



### Extra列

包含了不适合在其他列中显示的一些额外信息。常见的值有如下：

| 值                                 | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| NULL                               |                                                              |
| Distinct                           | 优化 distinct 操作，在找到第一匹配的元组后即停止找同样值的动作 |
| Not exists                         | 使用 not exists 来优化查询                                   |
| Using filesort                     | 表示按文件排序，一般是在指定的排序和索引排序不一致的情况才会出现。通常会出现在 order by 或 group by 查询中。 |
| Using index                        | 使用了覆盖索引进行查询。                                     |
| Using index condition; Using where | 表示使用了索引下推，在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据减少回表的数据。 |
| Using where                        | 表示使用了where条件过滤。                                    |
| Using where; Using index           |                                                              |
|                                    |                                                              |
| Using temporary                    | 表示使用了临时表，性能特别差，需要重点优化。一般多见于group by语句，或者union语句。 |
| select tables optimizedaway        | 直接通过索引来获得数据，不用访问标                           |



#### NULL

```mysql
EXPLAIN SELECT * FROM `it_blog` 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1779 | 100.00   | NULL  |



#### Distinct





#### Not exists  





#### Using filesort





#### Using index

```mysql
EXPLAIN SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys                      | key          | key_len | ref   | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ---------------------------------- | ------------ | ------- | ----- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify,idx_user_tag_classify | idx_classify | 8       | const | 2    | 100.00   | Using index |



`Using index` 表示where 和select中需要的字段都能够直接通过一个索引字段获取，无需再回表查询，当查询涉及的列都是某一单独索引的组成部分时即为此种情况，这实际上就是索引类型中覆盖索引。

上述查询的SQL条件和结果列都在 `idx_classify` 索引树中，因此无需再回表查询。



#### Using index condition; Using where





#### Using where

- 非索引where匹配查询

```mysql
EXPLAIN SELECT * FROM `it_blog` WHERE `read_status` = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
| :--- | ----------- | ------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----------- |
| 1    | SIMPLE      | it_blog | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1779 | 100.00   | Using where |



`Using where` 表示where条件中的查询没有使用到索引来限制要返回的select结果，执行计划中 `type` 是ALL类型。



#### Using where; Using index

```mysql
EXPLAIN SELECT `classify_id` FROM `it_blog` WHERE `classify_id`%2 = 1 
```



执行计划如下：

| id   | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                    |
| :--- | ----------- | ------- | ---------- | ----- | ------------- | ------------ | ------- | ---- | ---- | -------- | ------------------------ |
| 1    | SIMPLE      | it_blog | NULL       | index | NULL          | idx_classify | 8       | NULL | 1779 | 100.00   | Using where; Using index |



`Using where; Using index` 表示sql使用了覆盖索引，所有字段均从索引中获取，同时在从索引中查询出初步结果后，还需要使用组成索引的部分字段进一步进行条件筛选，而不是说需要回表获取完整行数据。



#### 	Using temporary





> 当该列的值出现了 Using temporary 时，就需要重点关注了，通常来说性能不会太好





#### select tables optimizedaway



## Profile

`explain`只是看到`SQL`的预估执行计划，如果要了解`SQL`**真正的执行线程状态及消耗的时间**，需要使用`profiling`。开启`profiling`参数后，后续执行的`SQL`语句都会记录其资源开销，包括`IO，上下文切换，CPU，内存`等等，可以根据这些开销进一步分析当前慢SQL的瓶颈再进一步进行优化。

```mysql
# 查看是否开启profile功能
show variables like '%profil%'
```



| Variable_name          | Value |
| ---------------------- | ----- |
| have_profiling         | YES   |
| profiling              | ON    |
| profiling_history_size | 15    |



### Show Profile

> Version 5.0.37

Show Profile 与 EXPLAIN 的区别是，前者主要是在外围分析；后者则是深入到 MySQL 内核，从执行线程的状态和时间来分析。

show profiles会显示最近发给服务器的多条语句，条数由变量profiling_history_size定义，默认是15。如果需要看单独某条SQL的分析，可以show profile查看最近一条SQL的分析，也可以使用show profile for query id（其中id就是show profiles中的QUERY_ID）查看具体一条的SQL语句分析。



```mysql
# 查看分析功能是否开启，YES表示功能已开启
select @@have_profiling;
```



```mysql
# 开启profiles分析功能
SET profiling=1;
```



```mysql
# 显示为空，说明profiles功能是关闭的。
SHOW PROFILES;
```



**查询结果：**

结果为空时说明需要开启profiles功能。正常结果如下：

| Query_ID | Duration   | Query                                                        |
| -------- | ---------- | ------------------------------------------------------------ |
| 16       | 0.001233   | SHOW STATUS                                                  |
| 17       | 0.00115825 | SELECT QUERY_ID, SUM(DURATION) AS SUM_DURATION FROM INFORMATION_SCHEMA.PROFILING GROUP BY QUERY_ID |



```mysql
# 查看SQL语句在执行过程中线程的每个状态所消耗的时间，16表示的是上述结果中的Query_ID
SHOW PROFILE FOR QUERY 16;
```



## Optimizer Trace

profile只能查看到SQL的执行耗时，但是无法看到SQL真正执行的过程信息，即不知道MySQL优化器是如何选择执行计划。这时可以使用`Optimizer Trace`，它可以跟踪执行语句的解析优化执行的全过程。

使用set optimizer_trace="enabled=on"打开开关，接着执行要跟踪的SQL，最后执行select * from information_schema.optimizer_trace跟踪，如下：

```mysql
set optimizer_trace="enabled=on";
SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` = 1;
select * from information_schema.optimizer_trace;
```



| QUERY      | TRACE         | MISSING_BYTES_BEYOND_MAX_MEM_SIZE | INSUFFICIENT_PRIVILEGES |
| ---------- | ------------- | --------------------------------- | ----------------------- |
| SELECT ... | {"steps": ... | 0                                 | 0                       |



```json
{
  "steps": [
    {
      "join_preparation": {}
    },
    {
      "join_optimization": {}
    },
    {
      "join_execution": {}
    }
  ]
}
```



分析其执行树，会包括三个阶段：

- join_preparation：准备阶段
- join_optimization：分析阶段
- join_execution：执行阶段



# MySQL 执行过程

## Server层

**Server 层负责建立连接、分析和执行 SQL**。MySQL 大多数的核心功能模块都在这实现，主要包括连接器，查询缓存、解析器、预处理器、优化器、执行器等。另外，所有的内置函数（如日期、时间、数学和加密函数等）和所有跨存储引擎的功能（如存储过程、触发器、视图等。）都在 Server 层实现。

- 连接器：建立连接，管理连接、校验用户身份，没有权限则返回错误；
- 查询缓存：查询语句如果命中查询缓存则直接返回，否则继续往下执行。MySQL 8.0 已删除该模块；
- **分析器**解析 SQL，通过解析器对 SQL 查询语句进行词法分析、语法分析，检查语法是否有错误。然后构建语法树，方便后续模块读取表名、字段、语句类型；
- 执行 SQL：执行 SQL 共有三个阶段：

	- **预处理器**预处理阶段：检查表或字段是否存在；将 `select *` 中的 `*` 符号扩展为表上的所有列。
	- **优化器**优化阶段：基于查询成本的考虑， 选择查询成本最小的执行计划；
	- **执行器**执行阶段：根据执行计划执行 SQL 查询语句，从存储引擎读取记录，返回给客户端；



查询SQL到了InnoDB中。会根据前面优化器里计算得到的索引，去**查询相应的索引页**，如果不在buffer pool里则从磁盘里加载索引页。**再通过索引页加速查询，得到数据页**的具体位置。如果这些数据页不在buffer pool中，则从磁盘里加载进来。



### 优化器

mysql会在**优化器阶段**里看下选择哪个索引，查询速度会更快。一般主要考虑几个因素，比如：

- 选择这个索引大概要扫描**多少行**（rows）
- 为了把这些行取出来，需要读**多少个16kb的页**
- 走普通索引需要回表，主键索引则不需要，**回表成本**大不大？



## 查询SQL

查询语句的执行流程如下：权限校验、查询缓存、分析器、优化器、权限校验、执行器、引擎。

![查询语句执行流程](../../Image/2022/07/220720-50.png)



举个例子，查询语句如下：

```mysql
select * from user where id > 1 and name = '大彬';
```



- 先检查该语句`是否有权限`，如果没有权限，直接返回错误信息，如果有权限会先查询缓存 (MySQL8.0 版本以前)。
- 如果没有缓存，分析器进行`语法分析`，提取 sql 语句中 select 等关键元素，然后判断 sql 语句是否有语法错误，比如关键词是否正确等等。
- 语法解析之后，MySQL的服务器会对查询的语句进行优化，确定执行的方案。
- 完成查询优化后，按照生成的执行计划`调用数据库引擎接口`，返回执行结果。




#### 执行SQL

#####  主键索引查询

查询条件用到了主键索引，而且是等值查询，同时主键 id 是唯一，不会有 id 相同的记录，所以优化器决定选用访问类型为 const 进行查询，也就是使用主键索引查询一条记录，那么执行器与存储引擎的执行流程是这样的：

- 执行器第一次查询，会调用 read_first_record 函数指针指向的函数，因为优化器选择的访问类型为 const，这个函数指针被指向为 InnoDB 引擎索引查询的接口，把条件 `id = 1` 交给存储引擎，**让存储引擎定位符合条件的第一条记录**。
- 存储引擎通过主键索引的 B+ 树结构定位到 id = 1的第一条记录，如果记录是不存在的，就会向执行器上报记录找不到的错误，然后查询结束。如果记录是存在的，就会将记录返回给执行器；
- 执行器从存储引擎读到记录后，接着判断记录是否符合查询条件，如果符合则发送给客户端，如果不符合则跳过该记录。
- 执行器查询的过程是一个 while 循环，所以还会再查一次，但是这次因为不是第一次查询了，所以会调用 read_record 函数指针指向的函数，因为优化器选择的访问类型为 const，这个函数指针被指向为一个永远返回 - 1 的函数，所以当调用该函数的时候，执行器就退出循环，也就是结束查询了。



##### 全表扫描

查询条件没有用到索引，所以优化器决定选用访问类型为 ALL 进行查询，也就是全表扫描的方式查询，那么这时执行器与存储引擎的执行流程是这样的：

- 执行器第一次查询，会调用 read_first_record 函数指针指向的函数，因为优化器选择的访问类型为 all，这个函数指针被指向为 InnoDB 引擎全扫描的接口，**让存储引擎读取表中的第一条记录**；
- 执行器会判断读到的这条记录的 name 是不是 iphone，如果不是则跳过；如果是则将记录发给客户的（是的没错，Server 层每从存储引擎读到一条记录就会发送给客户端，之所以客户端显示的时候是直接显示所有记录的，是因为客户端是等查询语句查询完成后，才会显示出所有的记录）。
- 执行器查询的过程是一个 while 循环，所以还会再查一次，会调用 read_record 函数指针指向的函数，因为优化器选择的访问类型为 all，read_record 函数指针指向的还是 InnoDB 引擎全扫描的接口，所以接着向存储引擎层要求继续读刚才那条记录的下一条记录，存储引擎把下一条记录取出后就将其返回给执行器（Server层），执行器继续判断条件，不符合查询条件即跳过该记录，否则发送到客户端；
- 一直重复上述过程，直到存储引擎把表中的所有记录读完，然后向执行器（Server层） 返回了读取完毕的信息；
- 执行器收到存储引擎报告的查询完毕的信息，退出循环，停止查询。



### 执行顺序

**MySQL执行顺序如下**

FROM > ON > JOIN > WHERE > GROUP BY > HAVING > SELECT > DISTINCT > UNION > ORDER BY > LIMIT	

1. **FROM**：对FROM子句中的左表<left_table>和右表<right_table>执行笛卡儿积（Cartesianproduct），产生虚拟表VT1
2. **ON**：对虚拟表VT1应用ON筛选，只有那些符合<join_condition>的行才被插入虚拟表VT2中
3. **JOIN**：如果指定了OUTER JOIN（如LEFT OUTER JOIN、RIGHT OUTER JOIN），那么保留表中未匹配的行作为外部行添加到虚拟表VT2中，产生虚拟表VT3。如果FROM子句包含两个以上表，则对上一个连接生成的结果表VT3和下一个表重复执行步骤1）～步骤3），直到处理完所有的表为止
4. **WHERE**：对虚拟表VT3应用WHERE过滤条件，只有符合<where_condition>的记录才被插入虚拟表VT4中
5. **GROUP BY**：根据GROUP BY子句中的列，对VT4中的记录进行分组操作，产生VT5
6. **CUBE|ROLLUP**：对表VT5进行CUBE或ROLLUP操作，产生表VT6
7. **HAVING**：对虚拟表VT6应用HAVING过滤器，只有符合<having_condition>的记录才被插入虚拟表VT7中。
8. **SELECT**：第二次执行SELECT操作，选择指定的列，插入到虚拟表VT8中
9. **DISTINCT**：去除重复数据，产生虚拟表VT9
10. **ORDER BY**：将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10。11）
11. **LIMIT**：取出指定行的记录，产生虚拟表VT11，并返回给查询用户



- 先执行from,join来确定表之间的连接关系，得到初步的数据
- where对数据进行普通的初步的筛选
- group by 分组
- 各组分别执行having中的普通筛选或者聚合函数筛选。
- 然后把再根据我们要的数据进行select，可以是普通字段查询也可以是获取聚合函数的查询结果，如果是集合函数，select的查询结果会新增一条字段
- 将查询结果去重distinct
- 最后合并各组的查询结果，按照order by的条件进行排序



from&join&where用于确定我们要查询的表的范围，涉及哪些表。

group by按照我们的分组条件，将数据进行分组，但是不会筛选数据。

having中可以是普通条件的筛选，也能是聚合函数。而where只能是普通函数，一般情况下，有having可以不写where，把where的筛选放在having里，SQL语句看上去更丝滑。

使用where再group by和使用group by再having几乎没有区别，不同的是，having语法支持聚合函数,其实having的意思就是针对每组的条件进行筛选。普通的筛选条件是不影响的，但是having还支持聚合函数，这是where无法实现的。

分组结束之后，再执行select语句，将查询结果去重distinct。

执行order by 将数据按照一定顺序排序。

limit是最后查询的。



### 存储引擎

#### 索引选择

通过**索引的区分度**来判断走什么索引，一个索引上不同的值越多，意味着出现相同数值的索引越少，意味着索引的区分度越高。我们也把区分度称之为**基数**，即区分度越高，基数越大。

索引系统是通过遍历部分数据，也就是通过**采样**的方式，来预测索引的基数的。**因此可能会由于统计的失误，导致系统没有走索引，而是走了全表扫描。**

系统判断是否走索引，扫描行数的预测其实只是原因之一，这条查询语句是否需要使用使用临时表、是否需要排序等也是会影响系统的选择的。



##### Innodb

根据主键查询时直接走主键索引，从主键索引的根结点开始遍历，然后查到叶子结点，直接得到行数据；非主键索引查询时，先从非主键索引树的叶子节点中获取到主键，再根据主键回表查询主键索引，得到行数据；全表扫描时直接遍历主键索引的叶子节点，得到行数据。



##### MyISAM

MyISAM下的表数据都是单独存储一个文件进行存储的，并不像InnoDB一样，存储在主键索引中，所以MyISAM不管查询是走主键索引，还是辅助键索引，通通都要遍历到叶子结点，从叶子结点获得真实数据的地址，再通过地址找到真实数据。



### 相关概念

#### 回表

```mysql
SELECT * FROM `it_blog` WHERE `classify_id` = 1 
```

查询上述SQL时，先通过索引 `classif_id` 获取到叶子节点上的主键id，此时表中的其他数据还需要通过主键去查询。



执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra |
| :--- | :---------- | :------ | :--------- | :--- | :----------------- | :----------------- | :------ | :---- | :--- | :------- | :---- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_classify_title | idx_classify_title | 9       | const | 2    | 100.00   | NULL  |



通过二级索引找到B+树中的叶子结点，但是二级索引的叶子节点的内容并不全，只有索引列的值和主键值。需要拿着主键值再去聚簇索引（主键索引）的叶子节点中去拿到完整的用户记录，这个过程叫做回表。

二级索引叶子节点中的主键id的排布就没有任何规律了，进行回表的时候，极有可能出现主键`id`所在的记录在聚簇索引叶子节点中反复横跳的情况，也就是随机IO。如果目标数据页恰好在内存中的话效果倒也不会太差，但如果不在内存中，还要从磁盘中加载一个数据页的内容（16KB）到内存中，这个速度就太慢了。

因此要尽量减少回表操作带来的损耗，总结起来就是两点：

1. 能不回表就不回；
2. 必须回表就减少回表的次数。



#### 覆盖索引

如果辅助索引中已经包含了所有需要读取的列数据，就不需要回表，这种查询方式称为**覆盖索引**（或**索引覆盖**），如下SQL所示：

```mysql
SELECT `id`, `classify_id` FROM `it_blog` WHERE `classify_id` = 1 
```



此时索引页中就包含了所查询的所有数据，无需回表查询，执行计划如下：

| id   | select_type | table   | partitions | type | possible_keys                            | key                | key_len | ref   | rows | filtered | Extra       |
| :--- | :---------- | :------ | :--------- | :--- | :--------------------------------------- | :----------------- | :------ | :---- | :--- | :------- | :---------- |
| 1    | SIMPLE      | it_blog | NULL       | ref  | idx_user_tag_classify,idx_classify_title | idx_classify_title | 9       | const | 2    | 100.00   | Using index |



覆盖索引在MySQL中，仅仅是针对InnoDB存储引擎而言的。准确的说，是针对聚簇索引和非聚簇索引共存的情况下才能起作用的。



## 更新SQL

![图片](../../Image/2022/09/220926-2.jpg)



1. 执行器先找引擎获取ID=2这一行。ID是主键，存储引擎检索数据，找到这一行。如果ID=2这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
2. 执行器拿到引擎给的行数据，把这个值加上1，比如原来是N，现在就是N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare状态。然后告知执行器执行完成了，随时可以提交事务。
4. 执行器生成这个操作的binlog，并把binlog写入磁盘。
5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成提交（commit）状态，更新完成。



具体更新一条记录 `UPDATE t_user SET name = 'xiaolin' WHERE id = 1;` 的流程如下:

1. 执行器负责具体执行，会调用存储引擎的接口，通过主键索引树搜索获取 id = 1 这一行记录：
	- 如果 id=1 这一行所在的数据页本来就在 buffer pool 中，就直接返回给执行器更新；
	- 如果记录不在 buffer pool，将数据页从磁盘读入到 buffer pool，返回记录给执行器。
2. 执行器得到聚簇索引记录后，会看一下更新前的记录和更新后的记录是否一样：
	- 如果一样的话就不进行后续更新流程；
	- 如果不一样的话就把更新前的记录和更新后的记录都当作参数传给 InnoDB 层，让 InnoDB 真正的执行更新记录的操作；
3. 开启事务， InnoDB 层更新记录前，首先要记录相应的 undo log，因为这是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面，不过在修改该 Undo 页面前需要先记录对应的 redo log，所以**先记录修改 Undo 页面的 redo log ，然后再真正的修改 Undo 页面**。
4. InnoDB 层开始更新记录，根据 WAL 技术，**先记录修改数据页面的 redo log ，然后再真正的修改数据页面**。修改数据页面的过程是修改 Buffer Pool 中数据所在的页，然后将其页设置为脏页，为了减少磁盘I/O，不会立即将脏页写入磁盘，后续由后台线程选择一个合适的时机将脏页写入到磁盘。
5. 至此，一条记录更新完了。
6. 在一条更新语句执行完成后，然后开始记录该语句对应的 binlog，此时记录的 binlog 会被保存到 binlog cache，并没有刷新到硬盘上的 binlog 文件，在事务提交时才会统一将该事务运行过程中的所有 binlog 刷新到硬盘。
7. 事务提交（为了方便说明，这里不说组提交的过程，只说两阶段提交）：
	- **prepare 阶段**：将 redo log 对应的事务状态设置为 prepare，然后将 redo log 刷新到硬盘；
	- **commit 阶段**：将 binlog 刷新到磁盘，接着调用引擎的提交事务接口，将 redo log 状态设置为 commit（将事务设置为 commit 状态后，刷入到磁盘 redo log 文件）；
8. 至此，一条更新语句执行完成。



查询语句的那一套流程，更新语句也是同样会走一遍：

- 客户端先通过连接器建立连接，连接器自会判断用户身份；
- 因为这是一条 update 语句，所以不需要经过查询缓存，但是表上有更新语句，是会把整个表的查询缓存情空的，所以说查询缓存很鸡肋，在 MySQL 8.0 就被移除这个功能了；
- 解析器会通过词法分析识别出关键字 update，表名等等，构建出语法树，接着还会做语法分析，判断输入的语句是否符合 MySQL 语法；
- 预处理器会判断表和字段是否存在；
- 优化器确定执行计划，因为 where 条件中的 id 是主键索引，所以决定要使用 id 这个索引；
- 执行器负责具体执行，找到这一行，然后更新。

不过，更新语句的流程会涉及到 undo log（回滚日志）、redo log（重做日志） 、binlog （归档日志）这三种日志。



更新语句执行流程如下：分析器、权限校验、执行器、引擎、`redo log`（`prepare`状态）、`binlog`、`redo log`（`commit`状态）



举个例子，更新语句如下：

```mysql
update user set name = '大彬' where id = 1;
```



1. 先查询到 id 为1的记录，有缓存会使用缓存。
2. 拿到查询结果，将 name 更新为大彬，然后调用引擎接口，写入更新数据，innodb 引擎将数据保存在内存中，同时记录`redo log`，此时`redo log`进入 `prepare`状态。
3. 执行器收到通知后记录`binlog`，然后调用引擎接口，提交`redo log`为`commit`状态。
4. 更新完成。



### **change buffer**

普通索引用在更新过程的加速，更新的字段如果在缓存中，如果是普通索引则直接更新即可。如果是唯一索引需要将所有数据读入内存来确保不违背唯一性，所以尽量用普通索引。



## 总结

- 查询流程如下：权限校验---》查询缓存---》分析器---》优化器---》权限校验---》执行器---》引擎

- 更新流程如下：分析器----》权限校验----》执行器---》引擎---redo log prepare---》binlog---》redo log commit



# MySQL 内存结构

### 脏页

当我们要往数据库插入一条数据、或者要更新一条数据的时候，我们知道数据库会在**内存**中把对应字段的数据更新了，但是更新之后，这些更新的字段并不会马上同步持久化到**磁盘**中去，而是把这些更新的记录写入到 redo log 日记中去，等到空闲的时候，在通过 redo log 里的日记把最新的数据同步到**磁盘**中去。

当内存数据页跟磁盘数据页内容不一致的时候，称这个内存页为“脏页”。内存数据写入到磁盘后，内存和磁盘上的数据页的内容就一致了，称为“干净页”。



#### 刷新脏页

引入了 Buffer Pool 后，当修改数据时，首先是修改 Buffer Pool 中数据所在的页，然后将其页设置为脏页，但是磁盘中还是原数据。

因此，脏页需要被刷入磁盘，保证缓存和磁盘数据一致，但是若每次修改数据都刷入磁盘，则性能会很差，因此一般都会在一定时机进行批量刷盘。

可能大家担心，如果在脏页还没有来得及刷入到磁盘时，MySQL 宕机了，不就丢失数据了吗？

这个不用担心，InnoDB 的更新操作采用的是 Write Ahead Log 策略，即先写日志，再写入磁盘，通过 redo log 日志让 MySQL 拥有了崩溃恢复能力。



**刷脏页有下面4种场景（后两种不用太关注“性能”问题）：**

- **redolog写满了：**redo log 里的容量是有限的，如果数据库一直很忙，更新又很频繁，这个时候 redo log 很快就会被写满了，这个时候就没办法等到空闲的时候再把数据同步到磁盘的，只能暂停其他操作，全身心来把数据同步到磁盘中去的，而这个时候，**就会导致我们平时正常的SQL语句突然执行的很慢**，所以说，数据库在在同步数据到磁盘的时候，就有可能导致我们的SQL语句执行的很慢了。
- **内存不够用了：**如果一次查询较多的数据，恰好碰到所查数据页不在内存中时，需要申请内存，而此时恰好内存不足的时候就需要淘汰一部分内存数据页，如果是干净页，就直接释放，如果恰好是脏页就需要刷脏页。
- **MySQL 认为系统“空闲”的时候：**这时系统没什么压力。
- **MySQL 正常关闭的时候：**这时候，MySQL 会把内存的脏页都 flush 到磁盘上，这样下次 MySQL 启动的时候，就可以直接从磁盘上读数据，启动速度会很快。



#### 脏页管理

设计 Buffer Pool 除了能提高读性能，还能提高写性能，也就是更新数据的时候，不需要每次都要写入磁盘，而是将 Buffer Pool 对应的缓存页标记为**脏页**，然后再由后台线程将脏页写入到磁盘。

那为了能快速知道哪些缓存页是脏的，于是就设计出 **Flush 链表**，它跟 Free 链表类似的，链表的节点也是控制块，区别在于 Flush 链表的元素都是脏页。

![img](../../Image/2022/07/220727-3.png)



有了 Flush 链表后，后台线程就可以遍历 Flush 链表，将脏页写入到磁盘。



### BufferPool

![图片](../../Image/2022/09/220921-5.jpg)

MySQL 的数据都是存在磁盘中的，那么我们要更新一条记录的时候，得先要从磁盘读取该记录，然后在内存中修改这条记录。那修改完这条记录是选择直接写回到磁盘，还是选择缓存起来呢？

当然是缓存起来好，这样下次有查询语句命中了这条记录，直接读取缓存中的记录，就不需要从磁盘获取数据了。

为此，Innodb 存储引擎设计了一个**缓冲池（Buffer Pool）**，来提高数据库的读写性能。



<img src="../../Image/2022/07/220723-7.png" alt="Buffer Poo" style="zoom:50%;" />

有了 Buffer Pool 后：

- 当读取数据时，如果数据存在于 Buffer Pool 中，客户端就会直接读取 Buffer Pool 中的数据，否则再去磁盘中读取。
- 当修改数据时，如果数据存在于 Buffer Pool 中，那直接修改 Buffer Pool 中数据所在的页，然后将其页设置为脏页（该页的内存数据和磁盘上的数据已经不一致），为了减少磁盘I/O，不会立即将脏页写入磁盘，后续由后台线程选择一个合适的时机将脏页写入到磁盘。



应用系统分层架构，为了加速数据访问，会把最常访问的数据，放在缓存(cache)里，避免每次都去访问数据库。操作系统，会有缓冲池(buffer pool)机制，避免每次访问磁盘，以加速数据的访问。MySQL作为一个存储系统，同样具有缓冲池(buffer pool)机制，以避免每次查询数据都进行磁盘IO，主要作用：

> 1、存在的意义是加速查询 
>
> 2、缓冲池(buffer pool) 是一种常见的**降低磁盘访问** 的机制；
>
> 3、缓冲池通常以页(page **16K**)为单位缓存数据；
>
> 4、缓冲池的常见管理算法是**LRU**，memcache，OS，InnoDB都使用了这种算法；
>
> 5、InnoDB对普通LRU进行了优化：将缓冲池分为`老生代`和`新生代`，入缓冲池的页，优先进入老生代，该页被访问，才进入新生代，以解决预读失效的问题页被访问。且在老生代**停留时间超过配置阈值**的，才进入新生代，以解决批量数据访问，大量热数据淘汰的问题



#### 缓存池大小

Buffer Pool 是在 MySQL 启动的时候，向操作系统申请的一片连续的内存空间，默认配置下 Buffer Pool 只有 `128MB` 。

可以通过调整 `innodb_buffer_pool_size` 参数来设置 Buffer Pool 的大小，一般建议设置成可用物理内存的 60%~80%。



#### 缓存池内容

InnoDB 会把存储的数据划分为若干个「页」，以页作为磁盘和内存交互的基本单位，一个页的默认大小为 16KB。因此，Buffer Pool 同样需要按「页」来划分。

InnoDB中，因为直接操作磁盘会比较慢，所以加了一层内存提提速，叫**buffer pool**，这里面，放了很多内存页，每一页16KB，有些内存页放的是数据库表里看到的那种一行行的数据，有些则是放的索引信息。

在 MySQL 启动的时候，**InnoDB 会为 Buffer Pool 申请一片连续的内存空间，然后按照默认的`16KB`的大小划分出一个个的页， Buffer Pool 中的页就叫做缓存页**。此时这些缓存页都是空闲的，之后随着程序的运行，才会有磁盘上的页被缓存到 Buffer Pool 中。

所以，MySQL 刚启动的时候，你会观察到使用的虚拟内存空间很大，而使用到的物理内存空间却很小，这是因为只有这些虚拟内存被访问后，操作系统才会触发缺页中断，申请物理内存，接着将虚拟地址和物理地址建立映射关系。

Buffer Pool 除了缓存「索引页」和「数据页」，还包括了 Undo 页，插入缓存、自适应哈希索引、锁信息等等。

<img src="../../Image/2022/07/220723-8.png" alt="img" style="zoom:50%;" />



为了更好的管理这些在 Buffer Pool 中的缓存页，InnoDB 为每一个缓存页都创建了一个**控制块**，控制块信息包括「缓存页的表空间、页号、缓存页地址、链表节点」等等。

控制块也是占有内存空间的，它是放在 Buffer Pool 的最前面，接着才是缓存页，如下图：

![img](../../Image/2022/07/220727-1.png)



上图中控制块和缓存页之间灰色部分称为碎片空间。



##### 碎片空间

每一个控制块都对应一个缓存页，那在分配足够多的控制块和缓存页后，可能剩余的那点儿空间不够一对控制块和缓存页的大小，自然就用不到喽，这个用不到的那点儿内存空间就被称为碎片了。

当然，如果把 Buffer Pool 的大小设置的刚刚好的话，也可能不会产生碎片。



##### Undo页

开启事务后，InnoDB 层更新记录前，首先要记录相应的 undo log，如果是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面。



**查询一条记录，就只需要缓冲一条记录吗？**

不是的。当我们查询一条记录时，当我们查询一条记录时，InnoDB 是会把整个页的数据加载到 Buffer Pool 中，因为，通过索引只能定位到磁盘中的页，而不能定位到页中的一条记录。将页加载到 Buffer Pool 后，再通过页里的页目录去定位到某条具体的记录。



#### 缓存池管理

Buffer Pool 是一片连续的内存空间，当 MySQL 运行一段时间后，这片连续的内存空间中的缓存页既有空闲的，也有被使用的。

那当我们从磁盘读取数据的时候，总不能通过遍历这一片连续的内存空间来找到空闲的缓存页吧，这样效率太低了。

所以，为了能够快速找到空闲的缓存页，可以使用链表结构，将空闲缓存页的「控制块」作为链表的节点，这个链表称为 **Free 链表**（空闲链表）。

![img](../../Image/2022/07/220727-2.png)



Free 链表上除了有控制块，还有一个头节点，该头节点包含链表的头节点地址，尾节点地址，以及当前链表中节点的数量等信息。

Free 链表节点是一个一个的控制块，而每个控制块包含着对应缓存页的地址，所以相当于 Free 链表节点都对应一个空闲的缓存页。

有了 Free 链表后，每当需要从磁盘中加载一个页到 Buffer Pool 中时，就从 Free链表中取一个空闲的缓存页，并且把该缓存页对应的控制块的信息填上，然后把该缓存页对应的控制块从 Free 链表中移除。



#### 缓存命中率

Buffer Pool 的大小是有限的，对于一些频繁访问的数据我们希望可以一直留在 Buffer Pool 中，而一些很少访问的数据希望可以在某些时机可以淘汰掉，从而保证 Buffer Pool 不会因为满了而导致无法再缓存新的数据，同时还能保证常用数据留在 Buffer Pool 中。

要实现这个，最容易想到的就是 LRU（Least recently used）算法。

该算法的思路是，链表头部的节点是最近使用的，而链表末尾的节点是最久没被使用的。那么，当空间不够了，就淘汰最久没被使用的节点，从而腾出空间。

简单的 LRU 算法的实现思路是这样的：

- 当访问的页在 Buffer Pool 里，就直接把该页对应的 LRU 链表节点移动到链表的头部。
- 当访问的页不在 Buffer Pool 里，除了要把页放入到 LRU 链表的头部，还要淘汰 LRU 链表末尾的节点。



到这里我们可以知道，Buffer Pool 里有三种页和链表来管理数据。

![img](../../Image/2022/07/220727-4.png)

图中：

- Free Page（空闲页），表示此页未被使用，位于 Free 链表；
- Clean Page（干净页），表示此页已被使用，但是页面未发生修改，位于LRU 链表。
- Dirty Page（脏页），表示此页「已被使用」且「已经被修改」，其数据和磁盘上的数据已经不一致。当脏页上的数据写入磁盘后，内存数据和磁盘数据一致，那么该页就变成了干净页。脏页同时存在于 LRU 链表和 Flush 链表。

简单的 LRU 算法并没有被 MySQL 使用，因为简单的 LRU 算法无法避免下面这两个问题：

- 预读失效；
- Buffer Pool 污染；



##### 查看缓存命中率

```
SHOW STATUS LIKE 'Innodb_buffer_pool_%'
```



`Innodb_buffer_pool_read_requests`表示读请求的次数。

`Innodb_buffer_pool_reads` 表示从物理磁盘中读取数据的请求次数。

所以buffer pool的命中率就可以这样得到：

命中率 = 1 - (Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests) * 100%



一般情况下**buffer pool命中率**都在`99%`以上，如果低于这个值，才需要考虑加大innodb buffer pool的大小。



##### 预读失效

由于预读(Read-Ahead)，提前把页放入了缓冲池，但最终MySQL并没有从页中读取数据，称为预读失效。

程序是有空间局部性的，靠近当前被访问数据的数据，在未来很大概率会被访问到。所以，MySQL 在加载数据页时，会提前把它相邻的数据页一并加载进来，目的是为了减少磁盘 IO。但是可能这些**被提前加载进来的数据页，并没有被访问**，相当于这个预读是白做了，这个就是**预读失效**。

如果使用简单的 LRU 算法，就会把预读页放到 LRU 链表头部，而当 Buffer Pool空间不够的时候，还需要把末尾的页淘汰掉。如果这些预读页如果一直不会被访问到，就会出现一个很奇怪的问题，不会被访问的预读页却占用了 LRU 链表前排的位置，而末尾淘汰的页，可能是频繁访问的页，这样就大大降低了缓存命中率。



###### LRU算法改进

要避免预读失效带来影响，最好就是**让预读的页停留在 Buffer Pool 里的时间要尽可能的短，让真正被访问的页才移动到 LRU 链表的头部，从而保证真正被读取的热数据留在 Buffer Pool 里的时间尽可能长**。

MySQL 改进了 LRU 算法，将 LRU 划分了 2 个区域：**old 区域 和 young 区域**。young 区域在 LRU 链表的前半部分，old 区域则是在后半部分，如下图：

![img](../../Image/2022/07/220727-6.png)



old 区域占整个 LRU 链表长度的比例可以通过 `innodb_old_blocks_pc` 参数来设置，默认是 37，代表整个 LRU 链表中 young 区域与 old 区域比例是 63:37。

**划分这两个区域后，预读的页就只需要加入到 old 区域的头部，当页被真正访问的时候，才将页插入 young 区域的头部**。如果预读的页一直没有被访问，就会从 old 区域移除，这样就不会影响 young 区域中的热点数据。



###### 示例

现在有个编号为 20 的页被预读了，这个页只会被插入到 old 区域头部，而 old 区域末尾的页（10号）会被淘汰掉。

![img](../../Image/2022/07/220727-7.png)

如果 20 号页一直不会被访问，它也没有占用到 young 区域的位置，而且还会比 young 区域的数据更早被淘汰出去。

如果 20 号页被预读后，立刻被访问了，那么就会将它插入到 young 区域的头部，young 区域末尾的页（7号），会被挤到 old 区域，作为 old 区域的头部，这个过程并不会有页被淘汰。

![img](../../Image/2022/07/220727-8.png)



##### Buffer Pool 污染

当某一个SQL语句，要批量扫描大量数据时，可能导致把缓冲池的所有页都替换出去，导致大量热数据被换出，MySQL性能急剧下降，这种情况叫缓冲池污染。解决办法：加入`老生代停留时间窗口`策略后，短时间内被大量加载的页，并不会立刻插入新生代头部，而是优先淘汰那些，短期内仅仅访问了一次的页。



当某一个 SQL 语句**扫描了大量的数据**时，在 Buffer Pool 空间比较有限的情况下，可能会将 **Buffer Pool 里的所有页都替换出去，导致大量热数据被淘汰了**，等这些热数据又被再次访问的时候，由于缓存未命中，就会产生大量的磁盘 IO，MySQL 性能就会急剧下降，这个过程被称为 **Buffer Pool 污染**。

注意， Buffer Pool 污染并不只是查询语句查询出了大量的数据才出现的问题，即使查询出来的结果集很小，也会造成 Buffer Pool 污染。

比如，在一个数据量非常大的表，执行了这条语句：

```mysql
select * from t_user where name like "%xiaolin%";
```

可能这个查询出来的结果就几条记录，但是由于这条语句会发生索引失效，所以这个查询过程是全表扫描的，接着会发生如下的过程：

- 从磁盘读到的页加入到 LRU 链表的 old 区域头部；
- 当从页里读取行记录时，也就是页被访问的时候，就要将该页放到 young 区域头部；
- 接下来拿行记录的 name 字段和字符串 xiaolin 进行模糊匹配，如果符合条件，就加入到结果集里；
- 如此往复，直到扫描完表中的所有记录。

经过这一番折腾，原本 young 区域的热点数据都会被替换掉。



###### 访问时间校验

像前面这种全表扫描的查询，很多缓冲页其实只会被访问一次，但是它却只因为被访问了一次而进入到 young 区域，从而导致热点数据被替换了。

LRU 链表中 young 区域就是热点数据，只要我们提高进入到 young 区域的门槛，就能有效地保证 young 区域里的热点数据不会被替换掉。

MySQL 是这样做的，进入到 young 区域条件增加了一个**停留在 old 区域的时间判断**。

具体是这样做的，在对某个处在 old 区域的缓存页进行第一次访问时，就在它对应的控制块中记录下来这个访问时间：

- 如果后续的访问时间与第一次访问的时间**在某个时间间隔内**，那么**该缓存页就不会被从 old 区域移动到 young 区域的头部**；
- 如果后续的访问时间与第一次访问的时间**不在某个时间间隔内**，那么**该缓存页移动到 young 区域的头部**；

这个间隔时间是由 `innodb_old_blocks_time` 控制的，默认是 1000 ms。

也就说，**只有同时满足「被访问」与「在 old 区域停留时间超过 1 秒」两个条件，才会被插入到 young 区域头部**，这样就解决了 Buffer Pool 污染的问题 。

另外，MySQL 针对 young 区域其实做了一个优化，为了防止 young 区域节点频繁移动到头部。young 区域前面 1/4 被访问不会移动到链表头部，只有后面的 3/4被访问了才会。



###### 示例

举个例子，假设需要批量扫描：21，22，23，24，25 这五个页，这些页都会被逐一访问（读取页里的记录）。

![img](../../Image/2022/07/220727-9.png)

在批量访问这些数据的时候，会被逐一插入到 young 区域头部。

![img](../../Image/2022/07/220727-10.png)

可以看到，原本在 young 区域的热点数据 6 和 7 号页都被淘汰了，这就是 Buffer Pool 污染的问题。





## 总结

Innodb 存储引擎设计了一个**缓冲池（\*Buffer Pool\*）**，来提高数据库的读写性能。

Buffer Pool 以页为单位缓冲数据，可以通过 `innodb_buffer_pool_size` 参数调整缓冲池的大小，默认是 128 M。

Innodb 通过三种链表来管理缓页：

- Free List （空闲页链表），管理空闲页；
- Flush List （脏页链表），管理脏页；
- LRU List，管理脏页+干净页，将最近且经常查询的数据缓存在其中，而不常查询的数据就淘汰出去。；

InnoDB 对 LRU 做了一些优化，我们熟悉的 LRU 算法通常是将最近查询的数据放到 LRU 链表的头部，而 InnoDB 做 2 点优化：

- 将 LRU 链表 分为**young 和 old 两个区域**，加入缓冲池的页，优先插入 old 区域；页被访问时，才进入 young 区域，目的是为了解决预读失效的问题。
- 当**「页被访问」且「 old 区域停留时间超过 `innodb_old_blocks_time` 阈值（默认为1秒）」**时，才会将页插入到 young 区域，否则还是插入到 old 区域，目的是为了解决批量数据访问，大量热数据淘汰的问题。

可以通过调整 `innodb_old_blocks_pc` 参数，设置 young 区域和 old 区域比例。

在开启了慢 SQL 监控后，如果你发现「偶尔」会出现一些用时稍长的 SQL，这可因为脏页在刷新到磁盘时导致数据库性能抖动。如果在很短的时间出现这种现象，就需要调大 Buffer Pool 空间或 redo log 日志的大小。



# MySQL MVCC
MVCC（Multiversion concurrency control）就是同一份数据保留多版本的一种方式，进而实现并发控制。MVCC是在存储引擎层进行实现的，不同存储引擎对MVCC有不同的实现标准。在查询的时候，通过read view和版本链找到对应版本的数据。

作用：提升并发性能。对于高并发场景，MVCC比行级锁开销更小。

通过一定机制生成一个数据请求**时间点的一致性数据快照（Snapshot)**，并用这个快照来提供一定级别（**语句级或事务级**）的**一致性读取**。从用户的角度来看，好像是**数据库可以提供同一数据的多个版本**。

- MVCC是被Mysql中 `事务型存储引擎InnoDB` 所支持的;
- **应对高并发事务, MVCC比`单纯的加锁`更高效**;
- MVCC只在 `READ COMMITTED` 和 `REPEATABLE READ` 两个隔离级别下工作;
- MVCC可以使用 `乐观(optimistic)锁` 和 `悲观(pessimistic)锁`来实现;
- 各数据库中MVCC实现并不统一
- InnoDB存储引擎在数据库每行数据的后面添加了**三个字段**



1、全称`Multi-Version Concurrency Control`，即`多版本并发控制`。MVCC是一种并发控制的`理念`，维持一个数据的多个版本，使得读写操作没有冲突。

2、MVCC在MySQL InnoDB中实现目的主要是为了**提高数据库并发性能**，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读。



## 基础概念

### Read View

Read View 主要是用来做可见性判断的，**因为Read View生成时机的不同, 造成了RC，RR两种隔离级别的不同可见性**。

- RR级别时, 事务在begin/start transaction之后的第一条select读操作后, 会创建一个快照Read View, 将当前系统中活跃的其他事务记录记录起来;
- RC级别时，事务中每条select语句都会创建一个Read View



> With REPEATABLE READ isolation level, the snapshot is based on the time when the first read operation is performed.
>
> With READ COMMITTED isolation level, the snapshot is reset to the time of each consistent read operation.



![img](../../Image/2022/07/220727-21.png)



Read View 有四个重要的字段：

- m_ids ：指的是在创建 Read View 时，当前数据库中「活跃事务」的**事务 id 列表**，注意是一个列表，**“活跃事务”指的就是，启动了但还没提交的事务**。
- min_trx_id ：指的是在创建 Read View 时，当前数据库中「活跃事务」中事务 **id 最小的事务**，也就是 m_ids 的最小值。
- max_trx_id ：这个并不是 m_ids 的最大值，而是**创建 Read View 时当前数据库中应该给下一个事务的 id 值**，也就是全局事务中最大的事务 id 值 + 1；
- creator_trx_id ：指的是**创建该 Read View 的事务的事务 id**。



### Undo Log快照

每一次记录变更之前都会先存储一份快照到`Und oLog`中，那么这几个隐式字段也会跟着记录一起保存在`Und oLog`中，每一个快照中都有有一个`DB_TRX_ID`字段记录了本次变更的事务ID，以及一个`DB_ROLL_PTR`字段指向了上一个快照的地址。

![图片](../../Image/2022/10/221001-1.png)

### 聚簇索引隐藏列

InnoDB存储引擎在数据库每行数据的后面添加了三个字段：



#### DB_TRX_ID

`DB_TRX_ID`（6字节）表示最后一次增删改该条记录的事务ID。



#### DB_ROLL_PTR

`DB_ROLL_PTR`（7字节）回滚指针表示指向该条记录在`Undo Log`中的上一个版本地址。



#### DB_ROW_ID

数据表没有主键和唯一索引，`InnoDB`会自动为每条记录添加`DB_ROW_ID`（6字节）来产生一个聚簇索引。



#### FLAG

一个删除flag隐藏字段, 既记录被更新或删除并不代表真的删除，而是删除flag变了。



### 当前读和快照读

1.MySQL的InnoDB存储引擎默认事务隔离级别是RR(可重复读), 是通过 "行排他锁+MVCC" 一起实现的, 不仅可以保证可重复读, 还可以**部分**防止幻读, 而非完全防止;

2.为什么是部分防止幻读, 而不是完全防止?

- 效果: 在如果事务B在事务A执行中, insert了一条数据并提交, 事务A再次查询, 虽然读取的是undo中的旧版本数据(防止了部分幻读), 但是事务A中执行update或者delete都是可以成功的!!
- 因为在innodb中的操作可以分为`当前读(current read)`和`快照读(snapshot read)`:

3.快照读(snapshot read)

```lasso
简单的select操作(不包括 select ... lock in share mode, select ... for update)
```

4.当前读(current read)

- select ... lock in share mode
- select ... for update
- insert
- update
- delete

在RR级别下，快照读是通过MVVC(多版本控制)和undo log来实现的，当前读是通过加record lock(记录锁)和gap lock(间隙锁)来实现的。
innodb在快照读的情况下并没有真正的避免幻读, 但是在当前读的情况下避免了不可重复读和幻读!!!



#### 当前读

1、像`select lock in share mode`(共享锁)、`select for updat`e 、`update`、`insert`、`delete`(排他锁)这些操作都是一种`当前读`，就是它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对`读取的记录进行加锁`。

2、当前读可以认为是`悲观锁`的具体功能实现



#### 快照读

1、不加锁的select就是快照读，即不加锁的非阻塞读；快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读；之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于多版本并发控制，即`MVCC`，可以认为`MVCC是行锁的一个变种`，但它在很多情况下，`避免了加锁操作`，降低了开销；既然是基于多版本，即快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本。

2、快照读就是MVCC思想在MySQL的具体非阻塞读功能实现，MVCC的目的就是为了实现读-写冲突不加锁，提高并发读写性能，而这个读指的就是`快照读`。

3、快照读就是MySQL为我们实现MVCC理想模型的其中一个具体非阻塞读功能。



##  原理

在创建 Read View 后，我们可以将记录中的 trx_id 划分这三种情况：

<img src="../../Image/2022/07/220727-22.png" alt="img" style="zoom:50%;" />

一个事务去访问记录的时候，除了自己的更新记录总是可见之外，还有这几种情况：

- 如果记录的 trx_id 值小于 Read View 中的 `min_trx_id` 值，表示这个版本的记录是在创建 Read View **前**已经提交的事务生成的，所以该版本的记录对当前事务**可见**。
- 如果记录的 trx_id 值大于等于 Read View 中的 `max_trx_id` 值，表示这个版本的记录是在创建 Read View **后**才启动的事务生成的，所以该版本的记录对当前事务**不可见**。
- 如果记录的 trx_id 值在 Read View 的min_trx_id和max_trx_id之间，需要判断 trx_id 是否在 m_ids 列表中：
	- 如果记录的 trx_id **在** `m_ids` 列表中，表示生成该版本记录的活跃事务依然活跃着（还没提交事务），所以该版本的记录对当前事务**不可见**。
	- 如果记录的 trx_id **不在** `m_ids`列表中，表示生成该版本记录的活跃事务已经被提交，所以该版本的记录对当前事务**可见**。

**这种通过「版本链」来控制并发事务访问同一个记录时的行为就叫 MVCC（多版本并发控制）。**



### 可重复读实现

**可重复读隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View**。

假设事务 A （事务 id 为51）启动后，紧接着事务 B （事务 id 为52）也启动了，那这两个事务创建的 Read View 如下：

<img src="../../Image/2022/07/220727-23.png" alt="img" style="zoom:50%;" />

事务 A 和 事务 B 的 Read View 具体内容如下：

- 在事务 A 的 Read View 中，它的事务 id 是 51，由于它是第一个启动的事务，所以此时活跃事务的事务 id 列表就只有 51，活跃事务的事务 id 列表中最小的事务 id 是事务 A 本身，下一个事务 id 则是 52。
- 在事务 B 的 Read View 中，它的事务 id 是 52，由于事务 A 是活跃的，所以此时活跃事务的事务 id 列表是 51 和 52，**活跃的事务 id 中最小的事务 id 是事务 A**，下一个事务 id 应该是 53。

接着，在可重复读隔离级别下，事务 A 和事务 B 按顺序执行了以下操作：

- 事务 B 读取小林的账户余额记录，读到余额是 100 万；
- 事务 A 将小林的账户余额记录修改成 200 万，并没有提交事务；
- 事务 B 读取小林的账户余额记录，读到余额还是 100 万；
- 事务 A 提交事务；
- 事务 B 读取小林的账户余额记录，读到余额依然还是 100 万；

接下来，跟大家具体分析下。

事务 B 第一次读小林的账户余额记录，在找到记录后，它会先看这条记录的 trx_id，此时**发现 trx_id 为 50，比事务 B 的 Read View 中的 min_trx_id 值（51）还小，这意味着修改这条记录的事务早就在事务 B 启动前提交过了，所以该版本的记录对事务 B 可见的**，也就是事务 B 可以获取到这条记录。

接着，事务 A 通过 update 语句将这条记录修改了（还未提交事务），将小林的余额改成 200 万，这时 MySQL 会记录相应的 undo log，并以链表的方式串联起来，形成**版本链**，如下图：

<img src="../../Image/2022/07/220727-24.png" alt="img" style="zoom:50%;" />

你可以在上图的「记录的字段」看到，由于事务 A 修改了该记录，以前的记录就变成旧版本记录了，于是最新记录和旧版本记录通过链表的方式串起来，而且最新记录的 trx_id 是事务 A 的事务 id（trx_id = 51）。

然后事务 B 第二次去读取该记录，**发现这条记录的 trx_id 值为 51，在事务 B 的 Read View 的 min_trx_id 和 max_trx_id 之间，则需要判断 trx_id 值是否在 m_ids 范围内，判断的结果是在的，那么说明这条记录是被还未提交的事务修改的，这时事务 B 并不会读取这个版本的记录。而是沿着 undo log 链条往下找旧版本的记录，直到找到 trx_id 「小于」事务 B 的 Read View 中的 min_trx_id 值的第一条记录**，所以事务 B 能读取到的是 trx_id 为 50 的记录，也就是小林余额是 100 万的这条记录。

最后，当事物 A 提交事务后，**由于隔离级别时「可重复读」，所以事务 B 再次读取记录时，还是基于启动事务时创建的 Read View 来判断当前版本的记录是否可见。所以，即使事物 A 将小林余额修改为 200 万并提交了事务， 事务 B 第三次读取记录时，读到的记录都是小林余额是 100 万的这条记录**。

就是通过这样的方式实现了，「可重复读」隔离级别下在事务期间读到的记录都是事务启动前的记录。



### 读已提交实现

**读提交隔离级别是在每次读取数据时，都会生成一个新的 Read View**。

也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。

那读提交隔离级别是怎么工作呢？我们还是以前面的例子来聊聊。

假设事务 A （事务 id 为51）启动后，紧接着事务 B （事务 id 为52）也启动了，接着按顺序执行了以下操作：

- 事务 B 读取数据（创建 Read View），小林的账户余额为 100 万；
- 事务 A 修改数据（还没提交事务），将小林的账户余额从 100 万修改成了 200 万；
- 事务 B 读取数据（创建 Read View），小林的账户余额为 100 万；
- 事务 A 提交事务；
- 事务 B 读取数据（创建 Read View），小林的账户余额为 200 万；

那具体怎么做到的呢？我们重点看事务 B 每次读取数据时创建的 Read View。前两次 事务 B 读取数据时创建的 Read View 如下图：

<img src="../../Image/2022/07/220727-25.png" alt="img" style="zoom:50%;" />

我们来分析下为什么事务 B 第二次读数据时，读不到事务 A （还未提交事务）修改的数据？

事务 B 在找到小林这条记录时，会看这条记录的 trx_id 是 51，在事务 B 的 Read View 的 min_trx_id 和 max_trx_id 之间，接下来需要判断 trx_id 值是否在 m_ids 范围内，判断的结果是在的，那么说明**这条记录是被还未提交的事务修改的，这时事务 B 并不会读取这个版本的记录**。而是，沿着 undo log 链条往下找旧版本的记录，直到找到 trx_id 「小于」事务 B 的 Read View 中的 min_trx_id 值的第一条记录，所以事务 B 能读取到的是 trx_id 为 50 的记录，也就是小林余额是 100 万的这条记录。

我们来分析下为什么事务 A 提交后，事务 B 就可以读到事务 A 修改的数据？

在事务 A 提交后，**由于隔离级别是「读提交」，所以事务 B 在每次读数据的时候，会重新创建 Read View**，此时事务 B 第三次读取数据时创建的 Read View 如下：

![img](../../Image/2022/07/220727-26.png)

事务 B 在找到小林这条记录时，**会发现这条记录的 trx_id 是 51，比事务 B 的 Read View 中的 min_trx_id 值（52）还小，这意味着修改这条记录的事务早就在创建 Read View 前提交过了，所以该版本的记录对事务 B 是可见的**。

正是因为在读提交隔离级别下，事务每次读数据时都重新创建 Read View，那么在事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。



### 幻读解决原理

在可重复读隔离级别下，**普通的查询是快照读，是不会看到别的事务插入的数据的**。

可重复读隔离级是由 MVCC（多版本并发控制）实现的，实现的方式是启动事务后，在执行第一个查询语句后，会创建一个 Read View，然后后续的查询语句利用这个 Read View，通过 Read View 就可以在 undo log 版本链找到事务开始时的数据，所以每次查询的数据都是一样的。

MySQL 里除了普通查询是快照读，其他都是**当前读**，比如update、insert、delete，这些语句执行前都会查询最新版本的数据，然后再做进一步的操作。另外，`select ... for update` 这种查询语句是当前读，每次执行的时候都是读取最新的数据。

**要讨论「可重复读」隔离级别的幻读现象，是要建立在「当前读」的情况下。**



假设`select ... for update`当前读是不会加锁的（实际上是会加锁的）：

![img](../../Image/2022/07/220727-27.png)



这时候，事务 B 插入的记录，就会被事务 A 的第二条查询语句查询到（因为是当前读），这样就会出现前后两次查询的结果集合不一样，这就出现了幻读。

所以，**Innodb 引擎为了解决「可重复读」隔离级别使用「当前读」而造成的幻读问题，就引出了 next-key 锁**，就是记录锁和间隙锁的组合。

- 记录锁，锁的是记录本身；
- 间隙锁，锁的就是两个值之间的空隙，以防止其他事务在这个空隙间插入新的数据，从而避免幻读现象。

比如，执行这条语句的时候，会锁住，然后期间如果有其他事务在这个锁住的范围插入数据就会被阻塞。

![img](../../Image/2022/07/220727-28.png)



next-key 锁的加锁规则其实挺复杂的，在一些场景下会退化成记录锁或间隙锁。

需要注意的是，如果 update 语句的 where 条件没有用到索引列，那么就会全表扫描，在一行行扫描的过程中，不仅给行数据加上了行锁，还给行两边的空隙也加上了间隙锁，相当于锁住整个表，然后直到事务结束才会释放锁。

所以，在线上千万不要执行没有带索引的 update 语句，不然会造成业务停滞。



### 可见性比较算法

在可重复度隔离级别下，设要读取的行的最后提交事务id(即当前数据行的稳定事务id)为 `trx_id_current`，当前新开事务id为 `new_id`，当前新开事务创建的快照`read view` 中最早的事务id为`up_limit_id`, 最迟的事务id为`low_limit_id`(注意这个low_limit_id=未开启的事务id=当前最大事务id+1)。

比较:

- 1.`trx_id_current < up_limit_id`, 这种情况比较好理解, 表示, 新事务在读取该行记录时, 该行记录的稳定事务ID是小于, 系统当前所有活跃的事务, 所以当前行稳定数据对新事务可见, 跳到步骤5.
- 2.`trx_id_current >= trx_id_last`, 这种情况也比较好理解, 表示, 该行记录的稳定事务id是在本次新事务创建之后才开启的, 但是却在本次新事务执行第二个select前就commit了，所以该行记录的当前值不可见, 跳到步骤4。
- 3.`trx_id_current <= trx_id_current <= trx_id_last`, 表示: 该行记录所在事务在本次新事务创建的时候处于活动状态，从up_limit_id到low_limit_id进行遍历，如果trx_id_current等于他们之中的某个事务id的话，那么不可见, 调到步骤4,否则表示可见。
- 4.从该行记录的 DB_ROLL_PTR 指针所指向的回滚段中取出最新的undo-log的版本号, 将它赋值该 `trx_id_current`，然后跳到步骤1重新开始判断。
- 5.将该可见行的值返回。



MVCC 实现原理如下：

MVCC 的实现依赖于版本链，版本链是通过表的三个隐藏字段实现。

DB_TRX_ID：当前事务id，通过事务id的大小判断事务的时间顺序。
DB_ROLL_PRT：回滚指针，指向当前行记录的上一个版本，通过这个指针将数据的多个版本连接在一起构成undo log版本链。
DB_ROLL_ID：主键，如果数据表没有主键，InnoDB会自动生成主键。
每条表记录大概是这样的：

图片
使用事务更新行记录的时候，就会生成版本链，执行过程如下：

用排他锁锁住该行；
将该行原本的值拷贝到undo log，作为旧版本用于回滚；
修改当前行的值，生成一个新版本，更新事务id，使回滚指针指向旧版本的记录，这样就形成一条版本链。
下面举个例子方便大家理解。

1、初始数据如下，其中DB_ROW_ID和DB_ROLL_PTR为空。

图片
2、事务A对该行数据做了修改，将age修改为12，效果如下：

图片
3、之后事务B也对该行记录做了修改，将age修改为8，效果如下：

图片
4、此时undo log有两行记录，并且通过回滚指针连在一起。

接下来了解下read view的概念。

read view可以理解成将数据在每个时刻的状态拍成“照片”记录下来。在获取某时刻t的数据时，到t时间点拍的“照片”上取数据。

在read view内部维护一个活跃事务链表，表示生成read view的时候还在活跃的事务。这个链表包含在创建read view之前还未提交的事务，不包含创建read view之后提交的事务。

不同隔离级别创建read view的时机不同。

read committed：每次执行select都会创建新的read_view，保证能读取到其他事务已经提交的修改。

repeatable read：在一个事务范围内，第一次select时更新这个read_view，以后不会再更新，后续所有的select都是复用之前的read_view。这样可以保证事务范围内每次读取的内容都一样，即可重复读。

read view的记录筛选方式

前提：DATA_TRX_ID 表示每个数据行的最新的事务ID；up_limit_id表示当前快照中的最先开始的事务；low_limit_id表示当前快照中的最慢开始的事务，即最后一个事务。

图片
如果DATA_TRX_ID < up_limit_id：说明在创建read view时，修改该数据行的事务已提交，该版本的记录可被当前事务读取到。
如果DATA_TRX_ID >= low_limit_id：说明当前版本的记录的事务是在创建read view之后生成的，该版本的数据行不可以被当前事务访问。此时需要通过版本链找到上一个版本，然后重新判断该版本的记录对当前事务的可见性。
如果up_limit_id <= DATA_TRX_ID < low_limit_i：
需要在活跃事务链表中查找是否存在ID为DATA_TRX_ID的值的事务。
如果存在，因为在活跃事务链表中的事务是未提交的，所以该记录是不可见的。此时需要通过版本链找到上一个版本，然后重新判断该版本的可见性。
如果不存在，说明事务trx_id 已经提交了，这行记录是可见的。
总结：InnoDB 的MVCC是通过 read view 和版本链实现的，版本链保存有历史版本记录，通过read view 判断当前版本的数据是否可见，如果不可见，再从版本链中找到上一个版本，继续进行判断，直到找到一个可见的版本。



### 总结

InnoDB 中，MVCC 就是通过 Undo Log + Read View 进行数据读取，**Undo Log 保存了历史快照，而 Read View 规则帮我们判断当前版本的数据是否可见。**从而不需要通过加锁的方式，就可以实现提交读和可重复读这两种隔离级别。



## 相关特性

### 快照读和当前读
表记录有两种读取方式。

快照读：读取的是快照版本。普通的SELECT就是快照读。通过mvcc来进行并发控制的，不用加锁。

当前读：读取的是最新版本。UPDATE、DELETE、INSERT、SELECT … LOCK IN SHARE MODE、SELECT … FOR UPDATE是当前读。

快照读情况下，InnoDB通过mvcc机制避免了幻读现象。而mvcc机制无法避免当前读情况下出现的幻读现象。因为当前读每次读取的都是最新数据，这时如果两次查询中间有其它事务插入数据，就会产生幻读。

下面举个例子说明下：

1、首先，user表只有两条记录，具体如下：
2、事务a和事务b同时开启事务start transaction；

3、事务a插入数据然后提交；
4、事务b执行全表的update；
5、事务b然后执行查询，查到了事务a中插入的数据。（下图左边是事务b，右边是事务a。事务开始之前只有两条记录，事务a插入一条数据之后，事务b查询出来是三条数据）
以上就是当前读出现的幻读现象。

那么MySQL是如何避免幻读？

在快照读情况下，MySQL通过mvcc来避免幻读。
在当前读情况下，MySQL通过next-key来避免幻读（加行锁和间隙锁来实现的）。
next-key包括两部分：行锁和间隙锁。行锁是加在索引上的锁，间隙锁是加在索引之间的。

Serializable隔离级别也可以避免幻读，会锁住整张表，并发性极低，一般不会使用。



## 事务隔离级别控制

![img](../../Image/2022/07/220723-4.png)

### 读已提交

就是把**释放锁的位置调整到事务提交之后**，此时在事务提交前，其他进程是无法对该行数据进行读取的，包括任何操作。

不可重复读：**一个事务读取到另外一个事务已经提交的数据，也就是说一个事务可以看到其他事务所做的修改**



### 可重复读

- **和不可重复读类似，但虚读(幻读)会读到其他事务的插入的数据，导致前后读取不一致**
- MySQL的`Repeatable read`隔离级别加上GAP间隙锁**已经处理了幻读了**。



## 总结

1. 一般我们认为MVCC有下面几个特点：
	- 每行数据都存在一个版本，每次数据更新时都更新该版本
	- 修改时Copy出当前版本, 然后随意修改，各个事务之间无干扰
	- 保存时比较版本号，如果成功(commit)，则覆盖原记录, 失败则放弃copy(rollback)
	- 就是每行都有版本号，保存时根据版本号决定是否成功，**听起来含有乐观锁的味道, 因为这看起来正是，在提交的时候才能知道到底能否提交成功**
2. 而InnoDB实现MVCC的方式是:
	- 事务以排他锁的形式修改原始数据
	- 把修改前的数据存放于undo log，通过回滚指针与主数据关联
	- 修改成功（commit）啥都不做，失败则恢复undo log中的数据（rollback）
3. **二者最本质的区别是**: 当修改数据时是否要`排他锁定`，如果锁定了还算不算是MVCC？

- Innodb的实现真算不上MVCC, 因为并没有实现核心的多版本共存, `undo log` 中的内容只是串行化的结果, 记录了多个事务的过程, 不属于多版本共存。但理想的MVCC是难以实现的, 当事务仅修改一行记录使用理想的MVCC模式是没有问题的, 可以通过比较版本号进行回滚, 但当事务影响到多行数据时, 理想的MVCC就无能为力了。
- 比如, 如果事务A执行理想的MVCC, 修改Row1成功, 而修改Row2失败, 此时需要回滚Row1, 但因为Row1没有被锁定, 其数据可能又被事务B所修改, 如果此时回滚Row1的内容，则会破坏事务B的修改结果，导致事务B违反ACID。 这也正是所谓的 `第一类更新丢失` 的情况。
- 也正是因为InnoDB使用的MVCC中结合了排他锁, 不是纯的MVCC, 所以第一类更新丢失是不会出现了, 一般说更新丢失都是指第二类丢失更新。



事务是在 MySQL 引擎层实现的，我们常见的 InnoDB 引擎是支持事务的，事务的四大特性是原子性、一致性、隔离性、持久性，我们这次主要讲的是隔离性。

当多个事务并发执行的时候，会引发脏读、不可重复读、幻读这些问题，那为了避免这些问题，SQL 提出了四种隔离级别，分别是读未提交、读已提交、可重复读、串行化，从左往右隔离级别顺序递增，隔离级别越高，意味着性能越差，InnoDB 引擎的默认隔离级别是可重复读。

要解决脏读现象，就要将隔离级别升级到读已提交以上的隔离级别，要解决不可重复读现象，就要将隔离级别升级到可重复读以上的隔离级别。

而对于幻读现象，不建议将隔离级别升级为串行化，因为这会导致数据库并发时性能很差。InnoDB 引擎的默认隔离级别虽然是「可重复读」，但是它通过 next-key lock 锁（行锁+间隙锁的组合）来锁住记录之间的“间隙”和记录本身，防止其他事务在这个记录之间插入新的记录，这样就避免了幻读现象。

对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 Read View 来实现的，它们的区别在于创建 Read View 的时机不同：

- 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
- 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

这两个隔离级别实现是通过「事务的 Read View 里的字段」和「记录中的两个隐藏列」的比对，来控制并发事务访问同一个记录时的行为，这就叫 MVCC（多版本并发控制）。

在可重复读隔离级别中，普通的 select 语句就是基于 MVCC 实现的快照读，也就是不会加锁的。而 select .. for update 语句就不是快照读了，而是当前读了，也就是每次读都是拿到最新版本的数据，但是它会对读到的记录加上 next-key lock 锁。



# MySQL 事务

**事务是逻辑上的一组操作，要么都执行，要么都不执行。**

数据库事务（简称：事务）是数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成。数据库事务通常包含了一个序列的对数据库的读/写操作。包含有以下两个目的：

- 为数据库操作序列提供了一个**从失败中恢复到正常状态**的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
- 当多个应用程序在**并发访问数据库**时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

当事务被提交给了数据库管理系统（DBMS），则DBMS需要确保该事务中的所有操作都成功完成且其结果被永久保存在数据库中，如果事务中有的操作没有成功完成，则事务中的所有操作都需要回滚，回到事务执行前的状态；同时，该事务对数据库或者其他事务的执行无影响，所有的事务都好像在独立的运行。



## 事务特性

### 原子性

**原子性**（`Atomicity`）：一个事务必须是一系列操作的最小单元，这系列操作的过程中，要么整个执行，要么整个回滚，不存在只执行了其中某一个或者某几个步骤。

**一致性**（`Consistency`）：事务要保证数据库整体数据的完整性和业务数据的一致性，事务成功提交整体数据修改，事务错误则回滚到数据回到原来的状态。

**隔离性**（`Isolation`）：隔离性是说两个事务的执行都是独立隔离开来的，事务之前不会相互影响，多个事务操作一个对象时会以串行等待的方式保证事务相互之间是隔离的。

**持久性**（`Durability`）：持久性是指一旦事务成功提交后，只要修改的数据都会进行持久化（通常是指数据成功保存到磁盘），不会因为异常、宕机而造成数据错误或丢失。



### 实现ACID的方式

影响数据库ACID实现的因素有两个：并发和系统故障，相应地，数据库系统通过并发控制技术和日志恢复技术来实现数据库的ACID特性。

- **持久性是通过 redo log （重做日志）来保证的；**
- **原子性是通过 undo log（回滚日志） 来保证的；**
- **隔离性是通过 MVCC（多版本并发控制） 或锁机制来保证的；**
- **一致性则是通过持久性+原子性+隔离性来保证；**



#### 数据库的并发控制

并发控制技术是实现事务隔离性以及不同隔离级别的关键,实现方式有很多,按照其对可能冲突的操作采取的不同策略可以分为乐观并发控制和悲观并发控制两大类。

- 乐观并发控制:对于并发执行可能冲突的操作,假定其不会真的冲突,允许并发执行,直到真正发生冲突时才去解决冲突,比如让事务回滚。
- 悲观并发控制:对于并发执行可能冲突的操作,假定其必定发生冲突,通过让事务等待(锁)或者中止(时间戳排序)的方式使并行的操作串行执行。

其实现方式有多种： 基于封锁的并发控制、基于时间戳的并发控制、基于有效性检查的并发控制、基于快照隔离的并发控制。



#### 数据库的日志

数据库运行过程中可能会出现故障,这些故障包括事务故障和系统故障两大类

- 事务故障:比如非法输入,系统出现死锁,导致事务无法继续执行。
- 系统故障:比如由于软件漏洞或硬件错误导致系统崩溃或中止。

这些故障可能会对事务和数据库状态造成破坏,因而必须提供一种技术来对各种故障进行恢复,保证数据库一致性,事务的原子性以及持久性。数据库通常以日志的方式记录数据库的操作从而在故障时进行恢复,因而可以称之为日志恢复技术。数据库日志包含undo和redo日志。



## 异常场景

### 写-写异常

- **一类丢失更新**：两个事物读同一数据，一个修改字段1，一个修改字段2，后提交的恢复了先提交修改的字段。**事务A的事务回滚覆盖了事务B已提交的结果。**
- **二类丢失更新（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 **事务A的提交覆盖了事务B已提交的结果。**



### 读-写异常

- **脏读**（dirty read）是指在一个事务处理过程里读取了另一个未提交的事务中的数据。
- **不可重复读**（non-repeatable read）是指在对于数据库中的某行记录，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，另一个事务修改了数据并提交了。
- **幻读**（phantom read）是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录或删除了记录，当之前的事务再次读取该范围的记录时，会产生幻行，就像产生幻觉一样，这就是发生了幻读。

**不可重复读和脏读的区别**是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

幻读和不可重复读都是读取了另一条已经提交的事务，不同的是不可重复读的重点是修改，幻读的重点在于新增或者删除。



## 隔离级别

MySQL数据库为了解决上面提到的脏读、不可重复读、幻读这几个问题，提供了四种隔离级别：

数据库的事务隔离越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使事务在一定程度上 “串行化”进行，这显然与“并发”是矛盾的。同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力。 



**读未提交(Read Uncommitted)**：所有事务都可以看到其他未提交事务的执行结果。能够读取到其他事务中未提交的内容，存在脏读问题。

**读已提交(Read Committed)**：一个事务只能看见已经提交事务所做的改变。可避免脏读的发生。只能读取其他事务已经提交的内容，存在不可重复读问题。

**可重复读(Repeated Read)**：MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行，解决了不可重复读的问题。在读取某行后不允许其他事务操作此行，直到事务结束，但是依然存在幻读问题。

**串行读(Serializable)**：通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。一个事务的开始必须等待另一个事务的完成。



MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**。

与 SQL 标准不同的地方在于 InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用的是Next-Key Lock 锁算法，因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server) 是不同的。所以说InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）** 已经可以完全保证事务的隔离性要求，即达到了 SQL标准的 **SERIALIZABLE(可串行化)** 隔离级别。因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是 **READ-COMMITTED(读取提交内容)** ，但是你要知道的是InnoDB 存储引擎默认使用 **REPEAaTABLE-READ（可重读）** 并不会有任何性能损失。

**InnoDB 引擎的默认隔离级别虽然是「可重复读」，但是它通过next-key lock 锁（行锁和间隙锁的组合）来锁住记录之间的“间隙”和记录本身，防止其他事务在这个记录之间插入新的记录，这样就避免了幻读现象。**



### 实现原理

#### 读未提交

对于「读未提交」隔离级别的事务来说，因为可以读到未提交事务修改的数据，所以直接读取最新的数据就好了；



#### 读已提交

对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 **Read View \**来实现的，它们的区别在于创建 Read View 的时机不同，大家可以把 Read View 理解成一个数据快照，就像相机拍照那样，定格某一时刻的风景。\**「读提交」隔离级别是在「每个语句执行前」都会重新生成一个 Read View，而「可重复读」隔离级别是「启动事务时」生成一个 Read View，然后整个事务期间都在用这个 Read View**。



##### 半一致读

在数据库的 **RC 这种隔离级别中，还支持"半一致读"** ，一条update语句，如果 where 条件匹配到的记录已经加锁，那么InnoDB会返回记录最近提交的版本，由MySQL上层判断此是否需要真的加锁。



#### 可重复读

- 针对**快照读**（普通 select 语句），是**通过 MVCC 方式解决了幻读**，因为可重复读隔离级别下，事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，即使中途有其他事务插入了一条数据，是查询不出来这条数据的，所以就很好了避免幻读问题。
- 针对**当前读**（select ... for update 等语句），是**通过 next-key lock（记录锁+间隙锁）方式解决了幻读**，因为当执行 select ... for update 语句的时候，会加上 next-key lock，如果有其他事务在 next-key lock 锁范围内插入了一条记录，那么这个插入语句就会被阻塞，无法成功插入，所以就很好了避免幻读问题。



#### 串行读

对于「串行化」隔离级别的事务来说，通过加读写锁的方式来避免并行访问；



### 总结

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| -------- | ---- | ---------- | ---- |
| 读未提交 | ✓    | ✓          | ✓    |
| 读已提交 | ×    | ✓          | ✓    |
| 可重复读 | ×    | ×          | ✓    |
| 串行读   | ×    | ×          | ×    |



#### RC和RR对比

RR 和 RC 两种事务隔离级别。他们主要在加锁机制、主从同步以及一致性读方面存在一些差异。

**为了提升并发度和降低死锁发生的概率，会把数据库的隔离级别从默认的 RR 调整成 RC。**

当然，这样做也不是完全没有问题，首先使用 RC 之后，就需要自己解决幻读的问题，这个其实还好，很多时候幻读问题其实是可以忽略的，或者可以用其他手段解决。



**RC 在加锁的过程中，是不需要添加Gap Lock和 Next-Key Lock 的，只对要修改的记录添加行级锁就行了。**这就使得并发度要比 RR 高很多。

**因为 RC 还支持"半一致读"，可以大大的减少了更新语句时行锁的冲突；对于不满足更新条件的记录，可以提前释放锁，提升并发度。**

RR这种事务隔离级别会增加Gap Lock和 Next-Key Lock，这就使得锁的粒度变大，那么就会使得死锁的概率增大。



使用 RC 的时候，不能使用statement格式的 binlog，这种影响其实可以忽略不计了，因为MySQL是在5.1.5版本开始支持row的、在5.1.8版本中开始支持mixed，后面这两种可以代替 statement格式。



## 语法

查看当前数据库的事务隔离级别：

```mysql
show variables like 'tx_isolation';
```

查看隔离级别

```mysql
# 查看当前会话隔离级别（v8.0之前）
SELECT @@tx_isolation;
# 查看系统当前隔离级别（v8.0之前）
SELECT @@global.tx_isolation;
# 查看当前会话隔离级别（v8.0之后）
SELECT @@transaction_isolation;
```

设置当前会话隔离级别

```mysql
set session transaction isolation level read uncommitted;
set session transaction isolation level read committed;
set session transaction isolation level repeatable read;
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```



设置全局事务隔离级别

```mysql
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```



事务开启、提交和回滚

```mysql
# 开启事务
start transaction;
# 提交事务
commit;
# 回滚事务
rollback;
```



注意，执行「开始事务」命令，并不意味着启动了事务。在 MySQL 有两种开启事务的命令，分别是：

- 第一种：begin/start transaction 命令；
- 第二种：start transaction with consistent snapshot 命令；

这两种开启事务的命令，事务的启动时机是不同的：

- 执行了 begin/start transaction 命令后，并不代表事务启动了。只有在执行这个命令后，执行了增删查改操作的 SQL 语句，才是事务真正启动的时机；
- 执行了 start transaction with consistent snapshot 命令，就会马上启动事务。



# MySQL 锁

**当一个事务请求的锁模式与当前的锁兼容，InnoDB就将请求的锁授予该事务；反之如果请求不兼容，则该事物就等待锁释放。**

![img](../../Image/2022/07/220725-12.png)



![图片](../../Image/2022/09/220921-4.jpg)



## 锁种类

![img](../../Image/2022/07/220722-4.png)

### 操作读写来分

#### 读锁（共享锁）

也叫共享锁，当一个事务添加了读锁后，其他的事务也可以添加读锁或是读取数据，但是不能进行写操作，只能等到所有的读锁全部释放。

共享锁（S锁）允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。也叫做**读锁**：读锁是**共享**的，多个客户可以**同时读取同一个**资源，但**不允许其他客户修改**。

共享锁（Share Locks，简记为S）又被称为读锁，其他用户可以并发读取数据，但任何事务都不能获取数据上的排他锁，直到已释放所有共享锁。

共享锁(S锁)又称为读锁，若事务T对数据对象A加上S锁，则事务T只能读A；其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这就保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。



为了确保自己查到的数据没有被其他的事务正在修改，也就是说确保查到的数据是`最新的数据`，并且不允许其他人来修改数据。但是自己不一定能够修改数据，因为有可能其他的事务也对这些数据使用了 `in share mode` 的方式上了`S` 锁。如果不及时的commit 或者rollback 也可能会**造成大量的事务等待**。



#### 写锁（排他锁）

也叫排他锁，当一个事务添加了写锁后，其他事务不能读不能写也不能添加任何锁，只能等待当前事务释放锁。

使用InnoDB的情况下，在执行更新、删除、插入操作时，数据库也会自动为所涉及的行添加写锁（排他锁），直到事务提交时，才会释放锁。

执行普通的查询操作时，不会添加任何锁。

使用MyISAM的情况下，在执行更新、删除、插入操作时，数据库会对涉及的表添加写锁，在执行查询操作时，数据库会对涉及的表添加读锁。

排他锁（X锁)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。也叫做**写锁**：写锁是排他的，**写锁会阻塞其他的写锁和读锁**。

排它锁（(Exclusive lock,简记为X锁)）又称为写锁，若事务T对数据对象A加上X锁，则只允许T读取和修改A，其它任何事务都不能再对A加任何类型的锁，直到T释放A上的锁。它防止任何其它事务获取资源上的锁，直到在事务的末尾将资源上的原始锁释放为止。在更新操作(INSERT、UPDATE 或 DELETE)过程中始终应用排它锁。



为了让自己查到的数据确保是最新数据，并且查到后的数据只允许自己来修改的时候，需要用到`for update`。相当于一个 update 语句。在业务繁忙的情况下，如果事务没有及时的commit或者rollback 可能会造成其他事务长时间的等待，从而影响数据库的并发使用效率。



##### FOR UPDATE

```mysql
# 先表上加上意向独占锁，然后对读取的记录加独占锁
SELECT * FROM `table_name` WHERE `condition` FOR UPDATE;
```



查询条件用了索引或主键，那么`FOR UPDATE`就会进行行锁；如果是普通字段(没有索引或主键)，那么`FOR UPDATE`就会进行锁表。



#### 总结

##### **共享锁和排他锁的区别**

共享锁（S锁）：如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不 能加排他锁。获取共享锁的事务只能读数据，不能修改数据。

排他锁（X锁）：如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获取排他锁的事务既能读数据，又能修改数据。



##### SELECT申请锁

SELECT 的读取锁定主要分为两种方式：共享锁和排他锁。

普通的 select 语句是不会对记录加锁的，如果要在查询时对记录加行锁，可以使用下面这两个方式：

```mysql
# 先在表上加上意向共享锁，然后对读取的记录加共享锁
SELECT * FROM `table_name` WHERE `condition` LOCK IN SHARE MODE;
# 先表上加上意向独占锁，然后对读取的记录加独占锁
SELECT * FROM `table_name` WHERE `condition` FOR UPDATE;
```

上面这两条语句必须在一个事务中，**因为当事务提交了，锁就会被释放**，所以在使用这两条语句的时候，要加上 begin、start transaction 或者 set autocommit = 0。



这两种方式主要的不同在于LOCK IN SHARE MODE多个事务同时更新同一个表单时很容易造成死锁。

申请排他锁的前提是，没有线程对该结果集的任何行数据使用排它锁或者共享锁，否则申请会受到阻塞。在进行事务操作时，MySQL会对查询结果集的每行数据添加排它锁，其他线程对这些数据的更改或删除操作会被阻塞（只能读操作），直到该语句的事务被commit语句或rollback语句结束为止。

SELECT... FOR UPDATE 使用注意事项：

for update 仅适用于innodb，且必须在事务范围内才能生效。根据主键进行查询，查询条件为like或者不等于，主键字段产生表锁。根据非索引字段进行查询，会产生表锁。



### 操作意向来分

> 如果另一个任务试图在该表级别上应用共享或排它锁，则受到由第一个任务控制的表级别意向锁的阻塞。第二个任务在锁定该表前不必检查各个页或行锁，而只需检查表上的意向锁。



当一个事务需要给自己需要的某个资源加锁的时候，如果遇到一个共享锁正锁定着自己需要的资源的时候，自己可以再加一个共享锁，不过不能加排他锁。但是，如果遇到自己需要锁定的资源已经被一个排他锁占有之后，则只能等待该锁定释放资源之后自己才能获取锁定资源并添加自己的锁定。

而意向锁的作用就是当一个事务在需要获取资源锁定的时候，如果遇到自己需要的资源已经被排他锁占用的时候，该事务可以需要锁定行的表上面添加一个合适的意向锁。

如果自己需要一个共享锁，那么就在表上面添加一个意向共享锁。而如果自己需要的是某行（或者某些行）上面添加一个排他锁的话，则先在表上面添加一个意向排他锁。**意向共享锁和意向排它锁可以同时并存多个。**

**为了允许行锁和表锁共存，实现多粒度锁机制**，InnoDB还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是**表锁**。

`意向锁是有数据引擎自己维护的，用户无法手动操作意向锁`，在为数据行加共享 / 排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。



#### 意向锁原理

因为共享锁与排他锁`互斥`，所以事务 B 在对 `users`表加共享锁的时候，必须保证：

- 当前没有其他事务持有 users 表的表排他锁。
- 当前没有其他事务持有 users 表中任意一行的排他锁  。

为了检测是否满足第二个条件，事务 B 必须在确保 `users`表不存在任何排他锁的前提下，去检测表中的每一行是否存在排他锁。很明显这是一个效率很差的做法，但是有了**意向锁**之后，情况就不一样了：



**意向锁之间是互相兼容的**，它会与普通的**表级排他 / 共享锁**互斥。



`事务 A` 先获取了某一行的**排他锁**，并未提交：

```mysql
SELECT * FROM users WHERE id = 6 FOR UPDATE;
```

1. `事务 A` 获取了 `users` 表上的**意向排他锁**。
2. `事务 A` 获取了 id 为 6 的数据行上的**排他锁**。

之后`事务 B` 想要获取 `users` 表的**共享锁**：

```mysql
LOCK TABLES users READ;
```

1. `事务 B` 检测到`事务 A` 持有 `users` 表的**意向排他锁**。
2. `事务 B` 对 `users` 表的加锁请求被阻塞（排斥）。

最后`事务 C` 也想获取 `users` 表中某一行的**排他锁**：

```mysql
SELECT * FROM users WHERE id = 5 FOR UPDATE;
```

1. `事务 C` 申请 `users` 表的**意向排他锁**。
2. `事务 C` 检测到`事务 A` 持有 `users` 表的**意向排他锁**。
3. 因为意向锁之间并不互斥，所以`事务 C` 获取到了 `users` 表的**意向排他锁**。
4. 因为id 为 5 的数据行上不存在任何**排他锁**，最终`事务 C` 成功获取到了该数据行上的**排他锁**。



#### 意向共享锁

意向共享锁（IS）指事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。



#### 意向排它锁

意向排他锁（IX）指事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。



#### 总结

也就是，当执行插入、更新、删除操作，需要先对表加上「意向独占锁」，然后对该记录加独占锁。

而普通的 select 是不会加行级锁的，普通的 select 语句是利用 MVCC 实现一致性读，是无锁的。

**意向共享锁和意向独占锁是表级锁，不会和行级的共享锁和独占锁发生冲突，而且意向锁之间也不会发生冲突，只会和共享表锁（\*lock tables ... read\*）和独占表锁（\*lock tables ... write\*）发生冲突。**



表锁和行锁是满足读读共享、读写互斥、写写互斥的。

如果没有「意向锁」，那么加「独占表锁」时，就需要遍历表里所有记录，查看是否有记录存在独占锁，这样效率会很慢。

那么有了「意向锁」，由于在对记录加独占锁前，先会加上表级别的意向独占锁，那么在加「独占表锁」时，直接查该表是否有意向独占锁，如果有就意味着表里已经有记录被加了独占锁，这样就不用去遍历表里的记录。

所以，**意向锁的目的是为了快速判断表里是否有记录被加锁**。



|                | 意向共享锁 | 意向排他锁 | 表共享锁 | 表排他锁 | 行共享锁 | 行排他锁 |
| -------------- | ---------- | ---------- | -------- | -------- | -------- | -------- |
| **意向共享锁** | 兼容       | 兼容       | 兼容     | 互斥     | 兼容     | 兼容     |
| **意向排他锁** | 兼容       | 兼容       | 互斥     | 互斥     | 兼容     | 兼容     |



1. **这里的意向锁是表级锁，表示的是一种意向，仅仅表示事务正在读或写某一行记录，在真正加行锁时才会判断是否冲突。意向锁是InnoDB自动加的，不需要用户干预。**
2. **IX，IS是表级锁，不会和行级的X，S锁发生冲突，只会和表级的X，S发生冲突。**
3. InnoDB 支持`多粒度锁`，特定场景下，行级锁可以与表级锁共存。
4. 意向锁之间互不排斥，但除了 IS 与 S 兼容外，`意向锁会与 共享锁 / 排他锁 互斥`。
5. IX，IS是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突。
6. 意向锁在保证并发性的前提下，实现了`行锁和表锁共存`且`满足事务隔离性`的要求。



### 作用范围来分

#### 全局锁

锁作用于全局，整个数据库的所有操作全部受到锁限制。



##### 语法

```mysql
# 获取全局锁
flush tables with read lock
# 释放全局锁
unlock tables
```



执行获取全局锁后，**整个数据库就处于只读状态了**，这时其他线程执行以下操作，都会被阻塞：

- 对数据的增删改操作，比如 insert、delete、update等语句；
- 对表结构的更改操作，比如 alter table、drop table 等语句。



执行释放全局锁命令来释放全局锁，或当会话断开了，全局锁会被自动释放。



##### 应用场景

###### 全库逻辑被封

全局锁主要应用于做**全库逻辑备份**，这样在备份数据库期间，不会因为数据或表结构的更新，而出现备份文件的数据与预期的不一样。

加上全局锁，意味着整个数据库都是只读状态。如果数据库里有很多数据，备份就会花费很多的时间，关键是备份期间，业务只能读数据，而不能更新数据，这样会造成业务停滞。



如果数据库的引擎支持的事务支持**可重复读的隔离级别**，那么在备份数据库之前先开启事务，会先创建 Read View，然后整个事务执行期间都在用这个 Read View，而且由于 MVCC 的支持，备份期间业务依然可以对数据进行更新操作。

因为在可重复读的隔离级别下，即使其他事务更新了表的数据，也不会影响备份数据库时的 Read View，这就是事务四大特性中的隔离性，这样备份期间备份的数据一直是在开启事务时的数据。

备份数据库的工具是 mysqldump，在使用 mysqldump 时加上 `–single-transaction` 参数的时候，就会在备份数据库之前先开启事务。这种方法只适用于支持「可重复读隔离级别的事务」的存储引擎。

InnoDB 存储引擎默认的事务隔离级别正是可重复读，因此可以采用这种方式来备份数据库。

但是，对于 MyISAM 这种不支持事务的引擎，在备份数据库时就要使用全局锁的方法。



#### 表级锁

Mysql中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM和 InnoDB引擎都支持表级锁。

锁作用于整个表，所有对表的操作都会收到锁限制。

MySQL中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM和 InnoDB引擎都支持表级锁。

- 开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低。

**虽然使用行级索具有粒度小、并发度高等特点，但是表级锁有时候也是非常必要的**：

- 事务更新大表中的大部分数据直接使用表级锁效率更高；
- 事务比较复杂，使用行级索很可能引起死锁导致回滚。



##### 表锁

表锁可以分为**表读锁**和**表写锁**。

需要注意的是，表锁除了会限制别的线程的读写外，也会限制本线程接下来的读写操作。

也就是说如果本线程对学生表加了「共享表锁」，那么本线程接下来如果要对学生表执行写操作的语句，是会被阻塞的，当然其他线程对学生表进行写操作时也会被阻塞，直到锁被释放。



在表读锁和表写锁的环境下：**读读不阻塞，读写阻塞，写写阻塞**！

- 读读不阻塞：当前用户在读数据，其他的用户也在读数据，不会加锁
- 读写阻塞：当前用户在读数据，其他的用户**不能修改当前用户读的数据**，会加锁！
- 写写阻塞：当前用户在修改数据，其他的用户**不能修改当前用户正在修改的数据**，会加锁！



**读锁和写锁是互斥的，读写操作是串行**。

- 如果某个进程想要获取读锁，**同时**另外一个进程想要获取写锁。在mysql里边，**写锁是优先于读锁的**！
- 写锁和读锁优先级的问题是可以通过参数调节的：`max_write_lock_count`和`low-priority-updates`



尽量避免在使用 InnoDB 引擎的表使用表锁，因为表锁的颗粒度太大，会影响并发性能。



###### 语法

```mysql
# 表级别的共享锁，也就是读锁；
lock tables t_student read;

# 表级别的独占锁，也就是写锁；
lock tables t_stuent write;

# 释放当前会话的所有表锁，另外，当会话退出后，也会释放所有表锁。
unlock tables;
```



##### 元数据锁

###### 创建元数据锁

当我们对数据库表进行操作时，会自动给这个表加上**元数据锁**MDL：

- 对一张表进行 CRUD 操作时，加的是 **MDL 读锁**；
- 对一张表做结构变更操作的时候，加的是 **MDL 写锁**；

MDL 是为了保证当用户对表执行 CRUD 操作时，防止其他线程对这个表结构做了变更。

当有线程在执行 select 语句（ 加 MDL 读锁）的期间，如果有其他线程要更改该表的结构（ 申请 MDL 写锁），那么将会被阻塞，直到执行完 select 语句（ 释放 MDL 读锁）。

反之，当有线程对表结构进行变更（ 加 MDL 写锁）的期间，如果有其他线程执行了 CRUD 操作（ 申请 MDL 读锁），那么就会被阻塞，直到表结构变更完成（ 释放 MDL 写锁）。



###### 释放元数据锁

MDL 是在事务提交后才会释放，这意味着**事务执行期间，MDL 是一直持有的**。

如果数据库有一个长事务（所谓的长事务，就是开启了事务，但是一直还没提交），那在对表结构做变更操作的时候，可能会发生意想不到的事情，比如下面这个顺序的场景：

1. 首先，线程 A 先启用了事务（但是一直不提交），然后执行一条 select 语句，此时就先对该表加上 MDL 读锁；
2. 然后，线程 B 也执行了同样的 select 语句，此时并不会阻塞，因为「读读」并不冲突；
3. 接着，线程 C 修改了表字段，此时由于线程 A 的事务并没有提交，也就是 MDL 读锁还在占用着，这时线程 C 就无法申请到 MDL 写锁，就会被阻塞，

那么在线程 C 阻塞后，后续有对该表的 select 语句，就都会被阻塞，如果此时有大量该表的 select 语句的请求到来，就会有大量的线程被阻塞住，这时数据库的线程很快就会爆满了。



> 为什么线程 C 因为申请不到 MDL 写锁，而导致后续的申请读锁的查询操作也会被阻塞？

这是因为申请 MDL 锁的操作会形成一个队列，队列中**写锁获取优先级高于读锁**，一旦出现 MDL 写锁等待，会阻塞后续该表的所有 CRUD 操作。

所以为了能安全的对表结构进行变更，在对表结构变更前，先要看看数据库中的长事务，是否有事务已经对表加上了 MDL 读锁，如果可以考虑 kill 掉这个长事务，然后再做表结构的变更。



##### 意向锁

###### 意向共享锁

见前文。



###### 意向排它锁

见前文。



##### AUTO-INC锁

在为某个字段声明 `AUTO_INCREMENT` 属性时，之后可以在插入数据时，可以不指定该字段的值，数据库会自动给该字段赋值递增的值，这主要是通过 AUTO-INC 锁实现的。

AUTO-INC 锁是特殊的表锁机制，锁**不是再一个事务提交后才释放，而是再执行完插入语句后就会立即释放**。

**在插入数据时，会加一个表级别的 AUTO-INC 锁**，然后为被 `AUTO_INCREMENT` 修饰的字段赋值递增的值，等插入语句执行完成后，才会把 AUTO-INC 锁释放掉。

那么，一个事务在持有 AUTO-INC 锁的过程中，其他事务的如果要向该表插入语句都会被阻塞，从而保证插入数据时，被 `AUTO_INCREMENT` 修饰的字段的值是连续递增的。

但是， AUTO-INC 锁再对大量数据进行插入的时候，会影响插入性能，因为另一个事务中的插入会被阻塞。

因此， 在 MySQL 5.1.22 版本开始，InnoDB 存储引擎提供了一种**轻量级的锁**来实现自增。

一样也是在插入数据的时候，会为被 `AUTO_INCREMENT` 修饰的字段加上轻量级锁，**然后给该字段赋值一个自增的值，就把这个轻量级锁释放了，而不需要等待整个插入语句执行完后才释放锁**。

InnoDB 存储引擎提供了个 innodb_autoinc_lock_mode 的系统变量，是用来控制选择用 AUTO-INC 锁，还是轻量级的锁。

- 当 innodb_autoinc_lock_mode = 0，就采用 AUTO-INC 锁；
- 当 innodb_autoinc_lock_mode = 2，就采用轻量级锁；
- 当 innodb_autoinc_lock_mode = 1，这个是默认值，两种锁混着用，如果能够确定插入记录的数量就采用轻量级锁，不确定时就采用 AUTO-INC 锁。

不过，当 innodb_autoinc_lock_mode = 2 是性能最高的方式，但是会带来一定的问题。因为并发插入的存在，在每次插入时，自增长的值可能不是连续的，**这在有主从复制的场景中是不安全的**。



##### 扩展

**MyISAM可以**支持查询和插入操作的**并发**进行。可以通过系统变量`concurrent_insert`来指定哪种模式，在**MyISAM**中它默认是：如果MyISAM表中没有空洞（即表的中间没有被删除的行），MyISAM允许在一个进程读表的同时，另一个进程从**表尾**插入记录。

但是**InnoDB存储引擎是不支持的**！



#### 页级锁

 MySQL中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。页级进行了折衷，一次锁定相邻的一组记录。BDB存储引擎支持页级锁。开销和加锁时间界于表锁和行锁之间，会出现死锁。锁定粒度界于表锁和行锁之间，并发度一般。



#### 行级锁

Mysql中锁定 **粒度最小** 的一种锁，只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，**会出现死锁。**

锁作用于表中的某一行，只会通过锁限制对某一行的操作（仅InnoDB支持）

InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**，也就是说，**InnoDB的行锁是基于索引的**！



##### 记录锁

（Record Locks）记录锁, **仅仅锁住索引记录的一行**，在单条索引记录上加锁。Record lock锁住的永远是索引，而非记录本身，即使该表上没有任何索引，那么innodb会在后台创建一个隐藏的聚集主键索引，那么锁住的就是这个隐藏的聚集主键索引。所以说当一条sql没有走任何索引时，那么将会在每一条聚合索引后面加写锁，这个类似于表锁，但原理上和表锁应该是完全不同的。

对索引项加锁，锁定符合条件的行。其他事务不能修改和删除加锁项；



##### 间隙锁

对索引项之间的“间隙”加锁，锁定记录的范围（对第一条记录前的间隙或最后一条将记录后的间隙加锁），不包含索引项本身。其他事务不能在锁范围内插入数据，这样就防止了别的事务新增幻影行。

（Gap Locks）**仅仅锁住一个索引区间**（开区间，不包括双端端点）。在索引记录之间的间隙中加锁，或者是在某一条索引记录之前或者之后加锁，并不包括该索引记录本身。比如在 1、2中，间隙锁的可能值有 (-∞, 1)，(1, 2)，(2, +∞)，间隙锁可用于防止幻读，保证索引间的不会被插入数据。

当我们**用范围条件检索数据**而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给**符合范围条件的已有数据记录的索引项加锁**；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

值得注意的是：间隙锁只会在`Repeatable read`隔离级别下使用~

- **为了防止幻读**(上面也说了，`Repeatable read`隔离级别下再通过GAP锁即可避免了幻读)
- 满足恢复和复制的需要，MySQL的恢复机制要求：**在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读**。




> *Gap locks in InnoDB are “purely inhibitive”, which means that their only purpose is to prevent other transactions from Inserting to the gap. Gap locks can co-exist. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap. There is no difference between shared and exclusive gap locks. They do not conflict with each other, and they perform the same function.*



间隙锁与间隙锁之间是兼容的，间隙锁在本质上是不区分共享间隙锁或互斥间隙锁的，而且间隙锁是不互斥的，即两个事务可以同时持有包含共同间隙的间隙锁。

这里的共同间隙包括两种场景：

- 其一是两个间隙锁的间隙区间完全一样；
- 其二是一个间隙锁包含的间隙区间是另一个间隙锁包含间隙区间的子集。



**间隙锁本质上是用于阻止其他事务在该间隙内插入新记录，而自身事务是允许在该间隙内插入数据的**。也就是说间隙锁的应用场景包括**并发读取、并发更新、并发删除和并发插入**。



###### 插入意向锁

> *An Insert intention lock is a type of gap lock set by Insert operations prior to row Insertion. This lock signals the intent to Insert in such a way that multiple transactions Inserting into the same index gap need not wait for each other if they are not Inserting at the same position within the gap. Suppose that there are index records with values of 4 and 7. Separate transactions that attempt to Insert values of 5 and 6, respectively, each lock the gap between 4 and 7 with Insert intention locks prior to obtaining the exclusive lock on the Inserted row, but do not block each other because the rows are nonconflicting.*



**插入意向锁是一种特殊的间隙锁，但不同于间隙锁的是，该锁只用于并发插入操作**。如果说间隙锁锁住的是一个区间，那么「插入意向锁」锁住的就是一个点。因而从这个角度来说，插入意向锁确实是一种特殊的间隙锁。

插入意向锁与间隙锁的另一个非常重要的差别是：尽管「插入意向锁」也属于间隙锁，但两个事务却不能在同一时间内，一个拥有间隙锁，另一个拥有该间隙区间内的插入意向锁（当然，插入意向锁如果不在间隙锁区间内则是可以的）。

- 每插入一条新记录，都需要看一下待插入记录的下一条记录上是否已经被加了间隙锁，如果已加间隙锁，那 Insert 语句应该被阻塞，并生成一个插入意向锁 。

	

`插入意向锁`是在插入一条记录行前，由 **INSERT** 操作产生的一种`间隙锁`。该锁用以表示插入**意向**，当多个事务在**同一区间**（gap）插入**位置不同**的多条数据时，事务之间**不需要互相等待**。假设存在两条值分别为 4 和 7 的记录，两个不同的事务分别试图插入值为 5 和 6 的两条记录，每个事务在获取插入行上独占的（排他）锁前，都会获取（4，7）之间的`间隙锁`，但是因为数据行之间并不冲突，所以两个事务之间并**不会产生冲突**（阻塞等待）。

总结来说，`插入意向锁`的特性可以分成两部分：

1. `插入意向锁`是一种特殊的`间隙锁`  —— `间隙锁`可以锁定**开区间**内的部分记录。
2. `插入意向锁`之间互不排斥，所以即使多个事务在同一区间插入多条记录，只要记录本身（`主键`、`唯一索引`）不冲突，那么事务之间就不会出现**冲突等待**。
3. **插入意向锁与间隙锁是冲突的，所以当其它事务持有该间隙的间隙锁时，需要等待其它事务释放间隙锁之后，才能获取到插入意向锁。而间隙锁与间隙锁之间是兼容的，所以所以两个事务中 `select ... for update` 语句并不会相互影响**。

需要强调的是，虽然`插入意向锁`中含有`意向锁`三个字，但是它并不属于`意向锁`而属于`间隙锁`，因为`意向锁`是**表锁**而`插入意向锁`是**行锁**。



1. **MySql InnoDB** 在 `Repeatable-Read` 的事务隔离级别下，使用`插入意向锁`来控制和解决并发插入。
2. `插入意向锁`是一种特殊的`间隙锁`。
3. `插入意向锁`在**锁定区间相同**但**记录行本身不冲突**的情况下**互不排斥**。



##### 临键锁

临键锁（Next-Key Locks）**Record lock + Gap lock，左开右闭区间**。默认情况下，`InnoDB`正是使用Next-key Locks来锁定记录（如select … for update语句）它还会根据场景进行灵活变换：

| 场景                                       | 转换                           |
| :----------------------------------------- | :----------------------------- |
| 使用唯一索引进行精确匹配，但表中不存在记录 | 自动转换为 Gap Locks           |
| 使用唯一索引进行精确匹配，且表中存在记录   | 自动转换为 Record Locks        |
| 使用非唯一索引进行精确匹配                 | 不转换                         |
| 使用唯一索引进行范围匹配                   | 不转换，但是只锁上界，不锁下界 |

锁定索引项本身和索引范围。即Record Lock和Gap Lock的结合。可解决幻读问题。



##### 总结

**InnoDB** 中的`行锁`的实现依赖于`索引`，一旦某个加锁操作没有使用到索引，那么该锁就会退化为`表锁`。

**记录锁**存在于包括`主键索引`在内的`唯一索引`中，锁定单条索引记录。

**间隙锁**存在于`非唯一索引`中，锁定`开区间`范围内的一段间隔，它是基于**临键锁**实现的。

**临键锁**存在于`非唯一索引`中，该类型的每条记录的索引上都存在这种锁，它是一种特殊的**间隙锁**，锁定一段`左开右闭`的索引区间。



### 乐观锁和悲观锁

数据库中的并发控制是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。乐观锁和悲观锁是并发控制主要采用的技术手段。



#### 悲观锁

悲观锁：假定会发生并发冲突，在查询完数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制。

```mysql
select * from `table_name` for update
```



在select 语句后边加了 `for update`相当于加了排它锁(写锁)，加了写锁以后，其他的事务就不能对它修改了！需要等待当前事务修改完之后才可以修改。



`优点`：适合在写多读少的并发环境中使用，虽然无法维持非常高的性能，但是在乐观锁无法提更好的性能前提下，可以做到数据的安全性

`缺点`：加锁会增加系统开销，虽然能保证数据的安全，但数据处理吞吐量低，不适合在读书写少的场合下使用



#### 乐观锁

乐观锁：假设不会发生并发冲突，只在提交操作时检查是否数据是否被修改过。给表增加`version`字段，在修改提交之前检查`version`与原来取到的`version`值是否相等，若相等，表示数据没有被修改，可以更新，否则，数据为脏数据，不能更新。实现方式：乐观锁一般使用版本号机制或`CAS`算法实现。



`优点`：在读多写少的并发场景下，可以避免数据库加锁的开销，提高DAO层的响应性能，很多情况下ORM工具都有带有乐观锁的实现，所以这些方法不一定需要我们人为的去实现。

`缺点`：在写多读少的并发场景下，即在写操作竞争激烈的情况下，会导致CAS多次重试，冲突频率过高，导致开销比悲观锁更高。

`实现`：数据库层面的乐观锁其实跟`CAS`思想类似， 通`数据版本号`或者`时间戳`也可以实现。



### 死锁

#### 死锁条件

**互斥、占有且等待、不可强占用、循环等待。**



**InnoDB的行级锁是基于索引实现的，如果查询语句为命中任何索引，那么InnoDB会使用表级锁.** 此外，InnoDB的行级锁是针对索引加的锁，不针对数据记录，因此即使访问不同行的记录，如果使用了相同的索引键仍然会出现锁冲突，还需要注意的是，在通过

```mysql
SELECT ...LOCK IN SHARE MODE;
# 或
SELECT ...FOR UPDATE;
```

使用锁的时候，如果表没有定义任何索引，那么InnoDB会创建一个隐藏的聚簇索引并使用这个索引来加记录锁。

此外，不同于MyISAM总是一次性获得所需的全部锁，InnoDB的锁是逐步获得的，当两个事务都需要获得对方持有的锁，导致双方都在等待，这就产生了死锁。 发生死锁后，InnoDB一般都可以检测到，并使一个事务释放锁回退，另一个则可以获取锁完成事务。



#### 死锁排查

当数据库发生死锁时，可以通过以下命令获取死锁日志：

```mysql
show engine innodb status
```



#### 避免死锁

但一般来说MySQL通过回滚帮我们解决了不少死锁的问题了，但死锁是无法完全避免的，可以通过以下的经验参考，来尽可能少遇到死锁：

- 以**固定的顺序**访问表和行。比如对两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；将两个事务的sql顺序调整为一致，也能避免死锁。
- **大事务拆小**。大事务更倾向于死锁，如果业务允许，将大事务拆小。
- 在同一个事务中，尽可能做到**一次锁定**所需要的所有资源，减少死锁概率。
- **降低隔离级别**。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。
- **为表添加合理的索引**。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。



##### 事务等待锁的超时时间

当一个事务的等待时间超过该值后，就对这个事务进行回滚，于是锁就释放了，另一个事务就可以继续执行了。在 InnoDB 中，参数 `innodb_lock_wait_timeout` 是用来设置超时时间的，默认值时 50 秒。

当发生超时后，就出现下面这个提示：

```base
ERROR 1205 (HY000): Lock wait timeout exceed; try restarting transaction
```



##### 主动开启死锁检测

主动死锁检测在发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数 `innodb_deadlock_detect` 设置为 on，表示开启这个逻辑，默认就开启。

当检测到死锁后，就会出现下面这个提示：

```
ERROR 1213 (40001): Deadlock fount when trying to get lock; try restarting transaction
```



我们可以采取以上方式避免死锁：

- 通过表级锁来减少死锁产生的概率；
- 多个程序尽量约定以相同的顺序访问表（这也是解决并发理论中哲学家就餐问题的一种思路）；
- 同一个事务尽可能做到一次锁定所需要的所有资源。



## 锁原理

> 以V8.0.26为参照



1、加锁的基本单位是 NextKeyLock，是前开后闭区间。

2、查找过程中访问到的对象才会加锁。

3、索引上的等值查询，给`唯一索引`加锁的时候，NextKeyLock退化为行锁。

4、索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，NextKeyLock退化为间隙锁。

5、唯一索引上的范围查询会访问到不满足条件的第一个值为止。



### 唯一索引等值查询

- **当查询的记录是存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「记录锁」**。
- **当查询的记录是不存在的，在用「唯一索引进行等值查询」时，next-key lock 会退化成「间隙锁」**。



#### 案例

案例表数据如下：

| id   | a    | b    |
| ---- | ---- | ---- |
| 0    | 0    | 0    |
| 4    | 4    | 4    |
| 8    | 8    | 8    |
| 16   | 16   | 16   |
| 32   | 32   | 32   |



##### 记录存在

```MySQL
# 会话1 查询一个存在的记录
SELECT * FROM t_test WHERE id = 16 FOR UPDATE;
# 会话2 （阻塞）
UPDATE t_test SET a = 100 WHERE id = 16;
# 会话3 （正常）
INSERT INTO t_test VALUE(9, 9, 9);
```



会话1加锁变化过程如下：

1. 加锁的基本单位是 next-key lock，因此会话1的加锁范围是(8, 16];
2. 但是由于是用唯一索引进行等值查询，且查询的记录存在，所以 **next-key lock 退化成记录锁，因此最终加锁的范围是 id = 16 这一行**。

所以，会话 2 在修改 id=16 的记录时会被锁住，而会话 3 插入 id=9 的记录可以被正常执行。



##### 记录不存在

```mysql
# 会话1 查询一个不存在的记录
SELECT * FROM t_test WHERE id = 10 FOR UPDATE;
# 会话2 （阻塞）
INSERT INTO t_test VALUE(9, 9, 9);
# 会话3 （正常）
UPDATE t_test SET a = 100 WHERE id = 16;
```



会话1加锁变化过程如下：

1. 加锁的基本单位是 next-key lock，因此主键索引 id 的加锁范围是(8, 16];
2. 但是由于查询记录不存在，next-key lock 退化成间隙锁，因此最终加锁的范围是 (8,16)。

所以，会话 2 要往这个间隙里面插入 id=9 的记录会被锁住，但是会话 3 修改 id =16 是可以正常执行的，因为 id = 16 这条记录并没有加锁。



### 唯一索引范围查询

```mysql
# 会话1 查询一个范围的记录
SELECT * FROM t_test WHERE id >= 8 AND id < 9 FOR UPDATE;
# 会话2 （阻塞）
INSERT INTO t_test VALUE(9, 9, 9);
# 会话3 （阻塞）
UPDATE t_test SET a = 100 WHERE id = 8;
# 会话4 （正常）
UPDATE t_test SET a = 100 WHERE id = 16;
```



会话 1 加锁变化过程如下：

1. 最开始要找的第一行是 id = 8，因此 next-key lock(4,8]，但是由于 id 是唯一索引，且该记录是存在的，因此会退化成记录锁，也就是只会对 id = 8 这一行加锁；
2. 由于是范围查找，就会继续往后找存在的记录，也就是会找到 id = 16 这一行停下来，然后加 next-key lock (8, 16]，但由于 id = 16 不满足 id < 9，所以会退化成间隙锁，加锁范围变为 (8, 16)。

所以，会话 1 这时候主键索引的锁是记录锁 id=8 和间隙锁(8, 16)。

会话 2 由于往间隙锁里插入了 id = 9 的记录，所以会被锁住了，而 id = 8 是被加锁的，因此会话 3 的语句也会被阻塞。

由于 id = 16 并没有加锁，所以会话 4 是可以正常被执行。



### 非唯一索引等值查询

- **当查询的记录存在时，除了会加 next-key lock 外，还额外加间隙锁，也就是会加两把锁**。
- **当查询的记录不存在时，只会加 next-key lock，然后会退化为间隙锁，也就是只会加一把锁。**



#### 案例

##### 记录存在

```mysql
# 会话1 查询一个存在的记录
SELECT * FROM t_test WHERE id = 8 FOR UPDATE;
# 会话2 （阻塞）
INSERT INTO t_test VALUE(9, 9, 9);
# 会话3 （阻塞）
INSERT INTO t_test VALUE(5, 5, 5);
# 会话4 （阻塞）
UPDATE t_test SET a = 100 WHERE id = 8;
# 会话5 （正常）
UPDATE t_test SET a = 100 WHERE id = 16;
```



会话 1 加锁变化过程如下：

1. 先会对普通索引 b 加上 next-key lock，范围是(4,8];
2. 然后因为是非唯一索引，且查询的记录是存在的，所以还会加上间隙锁，规则是向下遍历到第一个不符合条件的值才能停止，因此间隙锁的范围是(8,16)。

所以，会话1的普通索引 b 上共有两个锁，分别是 next-key lock (4,8] 和间隙锁 (8,16) 。

那么，当会话 2 往间隙锁里插入 id = 9 的记录就会被锁住，而会话 3 和会话 4 是因为更改了 next-key lock 范围里的记录而被锁住的。

然后因为 b = 16 这条记录没有加锁，所以会话 5 是可以正常执行的。



> 之前有读者反馈说，他自己做实验，发现插入 b = 4 这条记录会被阻塞，和我说的 next-key lock (4,8] 有点矛盾。
>
> 其实测试锁的范围是开区间还是闭区间不能用 insert 语句测试，而是要用 update 语句去测试。
>
> 因为 insert 加的锁是比较特殊的，**每插入一条新记录，都需要看一下待插入记录的下一条记录上是否已经被加了间隙锁，如果已加间隙锁，那 Insert 语句应该被阻塞，并生成一个插入意向锁**。
>
> 插入意向锁名字虽然有意向锁，但是它并不是意向锁，它是一种特殊的间隙锁，然后插入意向锁和间隙锁是冲突的，所以插入 b = 4 这条记录就发生阻塞了。
>
> 而用 update 语句来更新 b = 4 的这条记录，加的是记录锁，你测试的时候，会发现更新 b = 4 的这条记录是能更新成功的，所以 b = 4 这条并没有加锁，因此 next-key lock 的范围是 (4,8] ，是没问题。



##### 记录不存在

```mysql
# 会话1 查询一个不存在的记录
SELECT * FROM t_test WHERE b = 10 FOR UPDATE;
# 会话2 （阻塞）
INSERT INTO t_test VALUE(9, 9, 9);
# 会话3 （正常）
UPDATE t_test SET a = 100 WHERE id = 16;
```



会话 1 加锁变化过程如下：

1. 先会对普通索引 b 加上 next-key lock，范围是(8,16];
2. 但是由于查询的记录是不存在的，所以不会再额外加个间隙锁，但是 next-key lock 会退化为间隙锁，最终加锁范围是 (8,16)。

会话 2 因为往间隙锁里插入了 b = 9 的记录，所以会被锁住，而 b = 16 是没有被加锁的，因此会话 3 的语句可以正常执行。



### 非唯一索引范围查询

非唯一索引和主键索引的范围查询的加锁也有所不同，不同之处在于**普通索引范围查询，next-key lock 不会退化为间隙锁和记录锁**。



```mysql
# 会话1 查询一个范围的记录
SELECT * FROM t_test WHERE id >= 8 AND id < 9 FOR UPDATE;
# 会话2 （阻塞）
UPDATE t_test SET a = 100 WHERE id = 8;
# 会话3 （阻塞）
INSERT INTO t_test VALUE(9, 9, 9);
# 会话4 （阻塞）
UPDATE t_test SET a = 100 WHERE id = 16;
```



会话 1 加锁变化过程如下：

1. 最开始要找的第一行是 b = 8，因此 next-key lock(4,8]，但是由于 b 不是唯一索引，并不会退化成记录锁。
2. 但是由于是范围查找，就会继续往后找存在的记录，也就是会找到 b = 16 这一行停下来，然后加 next-key lock (8, 16]，因为是普通索引查询，所以并不会退化成间隙锁。

会话 1 的普通索引 b 有两个 next-key lock，分别是 (4,8] 和(8, 16]。所以会话 2 、会话 3 、会话 4 的语句都会被锁住了。



### 无索引更新

InnoDB 存储引擎的默认事务隔离级别是「可重复读」，但是在这个隔离级别下，在多个事务并发的时候，会出现幻读的问题，所谓的幻读是指在同一事务下，连续执行两次同样的查询语句，第二次的查询语句可能会返回之前不存在的行。

因此 InnoDB 存储引擎自己实现了行锁，通过 next-key 锁（记录锁和间隙锁的组合）来锁住记录本身和记录之间的“间隙”，防止其他事务在这个记录之间插入新的记录，从而避免了幻读现象。

当我们执行 update 语句时，实际上是会对记录加独占锁（X 锁）的，如果其他事务对持有独占锁的记录进行修改时是会被阻塞的。另外，这个锁并不是执行完 update 语句就会释放的，而是会等事务结束时才会释放。

在 InnoDB 事务中，对记录加锁带基本单位是 next-key 锁，但是会因为一些条件会退化成间隙锁，或者记录锁。加锁的位置准确的说，锁是加在索引上的而非行上。

比如，在 update 语句的 where 条件使用了唯一索引，那么 next-key 锁会退化成记录锁，也就是只会给一行记录加锁。



但是，**在 update 语句的 where 条件没有使用索引，就会全表扫描，于是就会对所有记录加上 next-key 锁（记录锁 + 间隙锁），相当于把整个表锁住了**。

```mysql
# 事务A
UPDATE t_stu SET score = 100 WHERE name = 'Bob';
# 事务B（阻塞）
UPDATE t_stu SET score = 77 WHERE id = 5;
```



可以看到，这次事务 B 的 update 语句被阻塞了。

这是因为事务 A的 update 语句中 where 条件没有索引列，所有记录都会被加锁，相当于锁住了全表。

因此，当在数据量非常大的数据库表执行 update 语句时，如果没有使用索引，就会给全表的加上 next-key 锁， 那么锁就会持续很长一段时间，直到事务结束，而这期间除了 `select ... from`语句，其他语句都会被锁住不能执行，业务会因此停滞。



update 语句的 where 带上索引能否避免全表记录加锁**关键还得看这条语句在执行过程种，优化器最终选择的是索引扫描，还是全表扫描，如果走了全表扫描，就会对全表的记录加锁了**。



>  innodb 不会对select、insert、delete、update语句加表锁的。因此这里锁全表的原因不是因为表锁。



#### 解决方式

将 MySQL 里的 `sql_safe_updates` 参数设置为 1，开启安全更新模式。当 sql_safe_updates 设置为 1 时，update 语句必须满足如下条件之一才能执行成功：

- 使用 where，并且 where 条件中必须有索引列；
- 使用 limit；
- 同时使用 where 和 limit，此时 where 条件中可以没有索引列；

delete 语句必须满足以下条件能执行成功：

- 同时使用 where 和 limit，此时 where 条件中可以没有索引列；



如果 where 条件带上了索引列，但是优化器最终扫描选择的是全表，而不是索引的话，我们可以使用 `force index([index_name])` 可以告诉优化器使用哪个索引，以此避免有几率锁全表带来的隐患。



### Insert加锁

Insert 语句在正常执行时是不会生成锁结构的，它是靠聚簇索引记录自带的 trx_id 隐藏列来作为**隐式锁**来保护记录的。



#### 隐式锁

当事务需要加锁的时，如果这个锁不可能发生冲突，InnoDB会跳过加锁环节，这种机制称为隐式锁。隐式锁是 InnoDB 实现的一种延迟加锁机制，其特点是只有在可能发生冲突时才加锁，从而减少了锁的数量，提高了系统整体性能。

隐式锁就是在 Insert 过程中不加锁，只有在特殊情况下，才会将隐式锁转换为显示锁，这里我们列举两个场景。

- 如果记录之间加有间隙锁，为了避免幻读，此时是不能插入记录的；
- 如果 Insert 的记录和已有记录存在唯一键冲突，此时也不能插入记录；



#### 记录间有间隙锁

每插入一条新记录，都需要看一下待插入记录的下一条记录上是否已经被加了间隙锁，如果已加间隙锁，那 Insert 语句应该被阻塞，并生成一个插入意向锁。



##### 示例

举个例子，现在 t_order 表中，只有这些数据，**order_no 是二级索引**。

| id   | order_no | create_date |
| ---- | -------- | ----------- |
| 1    | 1001     |             |
| 2    | 1002     |             |
| 3    | 1003     |             |
| 4    | 1004     |             |
| 5    | 1005     |             |



现在，事务 A 执行了下面这条语句。

```sql
# 事务 A
mysql> begin;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from t_order where order_no = 1006 for update;
Empty set (0.01 sec)
```

接着，我们执行 `select * from performance_schema.data_locks\G;` 语句 ，确定事务 A 加了什么类型的锁，这里只关注在记录上加锁的类型。

![img](../../Image/2022/07/220727-11.png)

可以看到，加的是 X 型的锁（排他锁），但是具体是记录锁、间隙锁、next-key 锁呢？注意，这里 LOCK_TYPE 中的 RECORD 表示行级锁，而不是记录锁的意思。

首先通过 LOCK_MODE 可以确认是 next-key 锁，还是间隙锁，还是记录锁：

- 如果 LOCK_MODE 为 `X`，说明是 next-key 锁；
- 如果 LOCK_MODE 为 `X, REC_NOT_GAP`，说明是记录锁；
- 如果 LOCK_MODE 为 `X, GAP`，说明是间隙锁；

因此，本次的例子加的是 next-key 锁（记录锁+间隙锁），锁范围是`（1005, +∞]`。

然后，有个事务 B 在这个间隙锁中，插入了一个记录，那么此时该事务 B 就会被阻塞：

```sql
# 事务 B 插入一条记录
mysql> begin;
Query OK, 0 rows affected (0.01 sec)

mysql> insert into t_order(order_no, create_date) values(1010,now());
### 阻塞状态。。。。
```

接着，我们执行 `select * from performance_schema.data_locks\G;` 语句 ，确定事务 B 加了什么类型的锁，这里只关注在记录上加锁的类型。

![img](../../Image/2022/07/220727-12.png)

可以看到，事务 B 的状态为等待状态（LOCK_STATUS: WAITING），因为向事务 A 生成的 next-key 锁（记录锁+间隙锁）范围`（1005, +∞]` 中插入了一条记录，所以事务 B 的插入操作生成了一个插入意向锁（`LOCK_MODE: X,INSERT_INTENTION`）。



#### 主键或唯一索引冲突

如果在插入新记录时，插入了一个与「已有的记录的主键或者唯一二级索引列值相同」的记录」（不过可以有多条记录的唯一二级索引列的值同时为NULL，这里不考虑这种情况），此时插入就会失败，然后对于这条记录加上了 **S 型的锁**。

至于是行级锁的类型是记录锁，还是 next-key 锁，跟是「主键冲突」还是「唯一二级索引冲突」有关系。

如果主键值重复：

- 当隔离级别为**读已提交**时，插入新记录的事务会给已存在的主键值重复的聚簇索引记录**添加 S 型记录锁**。
- 当隔离级别是**可重复读**（默认隔离级别），插入新记录的事务会给已存在的主键值重复的聚簇索引记录**添加 S 型记录锁**。

如果唯一二级索引列重复：

- **不论是哪个隔离级别**，插入新记录的事务都会给已存在的二级索引列值重复的二级索引记录**添加 S 型 next-key 锁**。对的，没错，即使是读已提交隔离级别也是加 next-key 锁，这是读已提交隔离级别中为数不多的给记录添加间隙锁的场景。因为如果不添加间隙锁的话，会让唯一二级索引中出现多条唯一二级索引列值相同的记录，这就违背了 UNIQUE 的约束。



##### 示例

###### 唯一索引冲突

下面举个「唯一二级索引冲突」的例子，MySQL 8.0 版本，事务隔离级别为可重复读（默认隔离级别）。

t_order 表中的 order_no 字段为唯一二级索引，并且已经存在 order_no 值为 1001 的记录，此时事务 A，插入了 order_no 为 1001 的记录，就出现了报错。

![img](../../Image/2022/07/220727-13.png)

但是除了报错之外，还做一个很重要的事情，就是对 order_no 值为 1001 这条记录加上了 **S 型的 next-key 锁**。

我们可以执行 `select * from performance_schema.data_locks\G;` 语句 ，确定事务加了什么类型的锁，这里只关注在记录上加锁的类型。

![img](../../Image/2022/07/220727-14.png)

可以看到，index_order 二级索引中的 1001（LOCK_DATA） 记录的锁类型为 S 型的 next-key 锁。注意，这里 LOCK_TYPE 中的 RECORD 表示行级锁，而不是记录锁的意思。如果是记录锁的话，LOCK_MODE 会显示 `S, REC_NOT_GAP`。

此时，事务 B 执行了 select * from t_order where order_no = 1001 for update; 就会阻塞，因为这条语句想加 X 型的锁，是与 S 型的锁是冲突的，所以就会被阻塞。

![img](../../Image/2022/07/220727-15.png)

我们也可以从 performance_schema.data_locks 这个表中看到，事务 B 的状态（LOCK_STATUS）是等待状态，加锁的类型 X 型的记录锁（LOCK_MODE: X,REC_NOT_GAP ）。

![img](../../Image/2022/07/220727-16.png)

上面的案例是针对唯一二级索引重复而插入失败的场景。



###### 执行了相同的 insert 语句

在隔离级别可重复读的情况下，开启两个事务，前后执行相同的 Insert 语句，此时**事务 B 的 Insert 语句会发生阻塞**。

![img](../../Image/2022/07/220727-20.png)

两个事务的加锁过程：

- 事务 A 先插入 order_no 为 1006 的记录，可以插入成功，此时对应的唯一二级索引记录被「隐式锁」保护，此时还没有实际的锁结构（执行完这里的时候，你可以看查 performance_schema.data_locks 信息，可以看到这条记录是没有加任何锁的）；
- 接着，事务 B 也插入 order_no 为 1006 的记录，由于事务 A 已经插入 order_no 值为 1006 的记录，所以事务 B 在插入二级索引记录时会遇到重复的唯一二级索引列值，此时事务 B 想获取一个 S 型 next-key 锁，但是事务 A 并未提交，**事务 A 插入的 order_no 值为 1006 的记录上的「隐式锁」会变「显示锁」且锁类型为 X 型的记录锁，所以事务 B 向获取 S 型 next-key 锁时会遇到锁冲突，事务 B 进入阻塞状态**。

我们可以执行 `select * from performance_schema.data_locks\G;` 语句 ，确定事务加了什么类型的锁，这里只关注在记录上加锁的类型。

先看事务 A 对 order_no 为 1006 的记录加了什么锁？

从下图可以看到，**事务 A 对 order_no 为 1006 记录加上了类型为 X 型的记录锁**（*注意，这个是在执行事务 B 之后才产生的锁，没执行事务 B 之前，该记录还是隐式锁*）。

![img](../../Image/2022/07/220727-19.png)

然后看事务 B 想对 order_no 为 1006 的记录加什么锁？

从下图可以看到，**事务 B 想对 order_no 为 1006 的记录加 S 型的 next-key 锁，但是由于事务 A 在该记录上持有了 X 型的记录锁，这两个锁是冲突的，所以导致事务 B 处于等待状态**。

![img](../../Image/2022/07/220727-18.png)

从这个实验可以得知，并发多个事务的时候，第一个事务插入的记录，并不会加锁，而是会用隐式锁保护唯一二级索引的记录。

但是当第一个事务还未提交的时候，有其他事务插入了与第一个事务相同的记录，第二个事务就会**被阻塞**，**因为此时第一事务插入的记录中的隐式锁会变为显示锁且类型是 X 型的记录锁，而第二个事务是想对该记录加上 S 型的 next-key 锁，X 型与 S 型的锁是冲突的**，所以导致第二个事务会等待，直到第一个事务提交后，释放了锁。

如果 order_no 不是唯一二级索引，那么两个事务，前后执行相同的 Insert 语句，是不会发生阻塞的，就如前面的这个例子。

![img](../../Image/2022/07/220727-17.png)



### 记录锁

记录锁就是为**某行**记录加锁，它`封锁该行的索引记录`：

```sql
-- id 列为主键列或唯一索引列
SELECT * FROM table WHERE id = 1 FOR UPDATE;
```

id 为 1 的记录行会被锁住。

需要注意的是：`id` 列必须为`唯一索引列`或`主键列`，否则上述语句加的锁就会变成`临键锁`。同时查询语句必须为`精准匹配`（`=`），不能为 `>`、`<`、`like`等，否则也会退化成`临键锁`。




在通过 `主键索引` 与 `唯一索引` 对数据行进行 UPDATE 操作时，也会对该行数据加`记录锁`：

```mysql
-- id 列为主键列或唯一索引列
UPDATE SET age = 50 WHERE id = 1;
```



### 间隙锁

**间隙锁**基于`非唯一索引`，它`锁定一段范围内的索引记录`。**间隙锁**基于下面将会提到的`Next-Key Locking` 算法，请务必牢记：**使用间隙锁锁住的是一个区间，而不仅仅是这个区间中的每一条数据**。

```sql
SELECT * FROM table WHERE id BETWEN 1 AND 10 FOR UPDATE;
复制代码
```

即所有在`（1，10）`区间内的记录行都会被锁住，所有id 为 2、3、4、5、6、7、8、9 的数据行的插入会被阻塞，但是 1 和 10 两条记录行并不会被锁住。

除了手动加锁外，在执行完某些 SQL 后，InnoDB 也会自动加**间隙锁**。



### 插入意向锁

如果只是使用普通的`间隙锁`会怎么样呢？还是使用我们文章开头的数据表为例：

MySql，InnoDB，Repeatable-Read：users(id PK, name, age KEY)

| id   | name | age  |
| ---- | ---- | ---- |
| 1    | Mike | 10   |
| 2    | Jone | 20   |
| 3    | Tony | 30   |

首先`事务 A` 插入了一行数据，并且没有 `commit`：

```sql
INSERT INTO users SELECT 4, 'Bill', 15;
```

此时 `users` 表中存在**三把锁**：

1. id 为 4 的记录行的`记录锁`。
2. age 区间在（10，15）的`间隙锁`。
3. age 区间在（15，20）的`间隙锁`。

最终，`事务 A` 插入了该行数据，并锁住了（10，20）这个区间。

随后`事务 B` 试图插入一行数据：

```sql
INSERT INTO users SELECT 5, 'Louis', 16;
```

因为 16 位于（15，20）区间内，而该区间内又存在一把`间隙锁`，所以`事务 B` 别说想申请自己的`间隙锁`了，它甚至不能获取该行的`记录锁`，自然只能乖乖的等待 `事务 A` 结束，才能执行插入操作。

很明显，这样做事务之间将会频发陷入**阻塞等待**，**插入的并发性**非常之差。这时如果我们再去回想我们刚刚讲过的`插入意向锁`，就不难发现它是如何优雅的解决了**并发插入**的问题。



### 临键锁

Next-Key 可以理解为一种特殊的**间隙锁**，也可以理解为一种特殊的**算法**。通过**临建锁**可以解决`幻读`的问题。 每个数据行上的`非唯一索引列`上都会存在一把**临键锁**，当某个事务持有该数据行的**临键锁**时，会锁住一段**左开右闭区间**的数据。需要强调的一点是，`InnoDB` 中`行级锁`是基于索引实现的，**临键锁**只与`非唯一索引列`有关，在`唯一索引列`（包括`主键列`）上不存在**临键锁**。



假设有如下表：
 **MySql**，**InnoDB**，**Repeatable-Read**：table(id PK, age KEY, name)

| id   | age  | name   |
| ---- | ---- | ------ |
| 1    | 10   | Lee    |
| 3    | 24   | Soraka |
| 5    | 32   | Zed    |
| 7    | 45   | Talon  |



该表中 `age` 列潜在的`临键锁`有：(-∞, 10], (10, 24], (24, 32], (32, 45], (45, +∞],



在`事务 A` 中执行如下命令：

```sql
-- 根据非唯一索引列 UPDATE 某条记录
UPDATE table SET name = Vladimir WHERE age = 24;
-- 或根据非唯一索引列 锁住某条记录
SELECT * FROM table WHERE age = 24 FOR UPDATE;
```



不管执行了上述 SQL 中的哪一句，之后如果在`事务 B` 中执行以下命令，则该命令会被阻塞：

```sql
INSERT INTO table VALUES(100, 26, 'Ezreal');
# 或
INSERT INTO table VALUES(100, 30, 'Ezreal');
```



很明显，`事务 A` 在对 `age` 为 24 的列进行 UPDATE 操作的同时，也获取了 `(24, 32]` 这个区间内的临键锁。

不仅如此，在执行以下 SQL 时，也会陷入阻塞等待：



那最终我们就可以得知，在根据`非唯一索引` 对记录行进行 `UPDATE \ FOR UPDATE \ LOCK IN SHARE MODE` 操作时，InnoDB 会获取该记录行的 `临键锁` ，并同时获取该记录行下一个区间的`间隙锁`。

即`事务 A `在执行了上述的 SQL 后，最终被锁住的记录区间为 `(10, 32)`。



### 锁原理总结

对记录加锁时，**加锁的基本单位是 next-key lock**，它是由记录锁和间隙锁组合而成的，**next-key lock 是前开后闭区间，而间隙锁是前开后开区间**。



这里需要注意的是，不同的版本加锁规则可能会有所不同。我这里总结下， 我这个 MySQL 版本的行级锁的加锁规则。

唯一索引等值查询：

- 当查询的记录是存在的，next-key lock 会退化成「记录锁」。
- 当查询的记录是不存在的，next-key lock 会退化成「间隙锁」。

非唯一索引等值查询：

- 当查询的记录存在时，除了会加 next-key lock 外，还额外加间隙锁，也就是会加两把锁。
- 当查询的记录不存在时，只会加 next-key lock，然后会退化为间隙锁，也就是只会加一把锁。

非唯一索引和主键索引的范围查询的加锁规则不同之处在于：

- 唯一索引在满足一些条件的时候，next-key lock 退化为间隙锁和记录锁。
- 非唯一索引范围查询，next-key lock 不会退化为间隙锁和记录锁。



## 查看加锁

通过 `select * from performance_schema.data_locks\G;` 语句，查看事务执行 SQL 过程中加了什么锁。

执行结果如下：

```bash

```



### 参数含义

#### **ENGINE**

执行引擎。

- INNODB：InnoDB执行引擎



#### **LOCL_TYPE**

锁类型，有TABLE和RECORD两种。

- TABLE：表锁
- RECORD：行锁



#### **LOCK_MODE**

加锁类型。

TABLE级别：

- IX：X类型的意向锁



RECORD级别

- X：X类型的临键锁。
- X, REC_NOT_GAP：记录锁
- X, GAP：间隙锁



#### **LOCK_DATA**

表示next-key 锁的范围。如果 LOCK_MODE 是 next-key 锁或者间隙锁，那么 **LOCK_DATA 就表示锁的范围最右值**，而锁范围的最左值为 LOCK_DATA 的上一条记录的值。



## 行锁原理

### 行锁升级

- 如果一个表批量更新，大量使用行锁，可能导致其他事务长时间等待，严重影响事务的执行效率。此时，MySQL会将 `行锁` 升级为 `表锁`
- 行锁是针对索引加的锁，如果 `条件索引失效`，那么 `行锁` 也会升级为 `表锁`



### 事务隔离级别

#### 读已提交

**在 RC 中，只会对索引增加Record Lock，不会添加Gap Lock和Next-Key Lock。**



#### 可重复读

**在 RR 中，为了解决幻读的问题，在支持Record Lock的同时，还支持Gap Lock和Next-Key Lock；**



## 总结

| 锁名称     | 操作类型 | 作用范围 | 支持的存储引擎 |
| ---------- | -------- | -------- | -------------- |
| 全局锁     | 所有操作 | 全局     | InnoDB、MyISAM |
| 表读锁     | 读       | 表级     | InnoDB、MyISAM |
| 表写锁     | 写       | 表级     | InnoDB         |
| 元数据读锁 | 读       | 表级     | InnoDB、MyISAM |
| 元数据写锁 | 写       | 表级     | InnoDB、MyISAM |
| 意向共享锁 | 读       | 表级     | InnoDB         |
| 意向排它锁 | 写       | 表级     | InnoDB         |
| AUTO-INC锁 | 写       | 表级     | InnoDB、MyISAM |
| 记录锁     | 写       | 行级     | InnoDB         |
| 间隙锁     | 写       | 行级     | InnoDB         |
| 临键锁     | 写       | 行级     | InnoDB         |




# MySQL 日志
MySQL日志主要包括查询日志、慢查询日志、事务日志、错误日志、二进制日志等。其中比较重要的是 bin log（二进制日志）和 redo log（重做日志）和 undo log（回滚日志）。

MySQL日志文件有很多，包括 ：

- **错误日志**（error log）：错误日志文件对MySQL的启动、运行、关闭过程进行了记录，能帮助定位MySQL问题。
- **慢查询日志**（slow query log）：慢查询日志是用来记录执行时间超过 long_query_time 这个变量定义的时长的查询语句。通过慢查询日志，可以查找出哪些查询语句的执行效率很低，以便进行优化。
- **一般查询日志**（general log）：一般查询日志记录了所有对MySQL数据库请求的信息，无论请求是否正确执行。
- **二进制日志**（bin log）：关于二进制日志，它记录了数据库所有执行的DDL和DML语句（除了数据查询语句select、show等），以事件形式记录并保存在二进制文件中。

还有两个InnoDB存储引擎特有的日志文件：

- **重做日志**（redo log）：重做日志至关重要，因为它们记录了对于InnoDB存储引擎的事务日志。
- **回滚日志**（undo log）：回滚日志同样也是InnoDB引擎提供的日志，顾名思义，回滚日志的作用就是对数据进行回滚。当事务对数据库进行修改，InnoDB引擎不仅会记录redo log，还会生成对应的undo log日志；如果事务执行失败或调用了rollback，导致事务需要回滚，就可以利用undo log中的信息将数据回滚到修改之前的样子。



## error log

## slow query log

通过**慢查询日志**来查看慢SQL。默认的情况下，MySQL数据库是不开启慢查询日志（`slow query log`）。需要手动把它打开。

```MySQL
# 查看慢查询日志配置
show variables like 'slow_query_log%'
```



| Variable_name       | Value                                                  |
| ------------------- | ------------------------------------------------------ |
| slow_query_log      | OFF                                                    |
| slow_query_log_file | /usr/local/mysql/data/izuf6e56zt7ebjhby2hxboz-slow.log |



slow_query_log表示慢查询日志开启状态，slow_query_log_file表示慢查询日志存储位置。



```mysql
# 查看慢查询时间长度定义
show variables like 'long_query_time'
```



| Variable_name   | Value     |
| --------------- | --------- |
| long_query_time | 10.000000 |



## general log



## 基础概念

### binlog
MySQL 在完成一条更新操作后，Server 层还会生成一条 binlog，等之后事务提交的时候，会将该事物执行过程中产生的所有 binlog 统一写 入 binlog 文件。binlog主要用于恢复数据库和同步数据库。

binlog 文件是记录了所有数据库表结构变更（例如create、alter table）和表数据修改（insert、update、delete）的日志，不会记录查询类的操作，比如 SELECT 和 SHOW 操作。

最开始 MySQL 里并没有 InnoDB 引擎，MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档。

而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 crash-safe 能力的，所以 InnoDB 使用 redo log 来实现 crash-safe 能力。

主从数据库同步用到的都是BinLog文件。BinLog日志文件有三种模式。



#### BinLog模式

##### **STATEMENT 模式**

`内容`：binlog 只会记录可能引起数据变更的 sql 语句

`优势`：该模式下，因为没有记录实际的数据，所以日志量和 IO 都消耗很低，性能是最优的

`劣势`：但有些操作并不是确定的，比如 uuid() 函数会随机产生唯一标识，当依赖 binlog 回放时，该操作生成的数据与原数据必然是不同的，此时可能造成无法预料的后果。



##### **ROW 模式**

`内容`：在该模式下，binlog 会**记录每次操作的源数据与修改后的目标数据**，StreamSets就要求该模式。

`优势`：可以绝对精准的还原，从而保证了数据的安全与可靠，并且复制和数据恢复过程可以是并发进行的

`劣势`：缺点在于 binlog 体积会非常大，同时，对于修改记录多、字段长度大的操作来说，记录时性能消耗会很严重。阅读的时候也需要特殊指令来进行读取数据。



##### **MIXED 模式**

`内容`：是对上述STATEMENT 跟 ROW  两种模式的混合使用。

`细节`：对于绝大部分操作，都使用 STATEMENT 来进行 binlog 的记录，只有以下操作使用 ROW 来实现：表的存储引擎为 NDB，使用了uuid() 等不确定函数，使用了 insert delay 语句，使用了临时表



#### binlog刷盘

事务执行过程中，先把日志写到 binlog cache（Server 层的 cache），事务提交的时候，再把 binlog cache 写到 binlog 文件中。

MySQL 给 binlog cache 分配了一片内存，每个线程一个，参数 binlog_cache_size 用于控制单个线程内 binlog cache 所占内存的大小。如果超过了这个参数规定的大小，就要暂存到磁盘。



在事务提交的时候，执行器把 binlog cache 里的完整事务写入到 binlog 文件中，并清空 binlog cache。如下图：

![image-20220725211411126](../../Image/2022/07/220725-9.png)



MySQL提供一个 sync_binlog 参数来控制数据库的 binlog 刷到磁盘上的频率：

- sync_binlog = 0 的时候，表示每次提交事务都只 write，不 fsync，后续交由操作系统决定何时将数据持久化到磁盘；
- sync_binlog = 1 的时候，表示每次提交事务都会 write，然后马上执行 fsync；
- sync_binlog =N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

在MySQL中系统默认的设置是 sync_binlog = 0，也就是不做任何强制性的磁盘刷新指令，这时候的性能是最好的，但是风险也是最大的。因为一旦主机发生异常重启，在 binlog cache 中的所有 binlog 日志都会被丢失。

而当 sync_binlog 设置为 1 的时候，是最安全但是性能损耗最大的设置。因为当设置为1的时候，即使主机发生异常重启，也最多丢失 binlog cache 中未完成的一个事务，对实际数据没有任何实质性影响，就是对写入性能影响太大。

如果能容少量事务的 binlog 日志丢失的风险，为了提高写入的性能，一般会 sync_binlog 设置为 100~1000 中的某个数值。



#### 扩展

##### redo log 和 binlog 有什么区别？

这两个日志有四个区别。

*1、**适用对象不同**：*

- binlog 是 MySQL 的 Server 层实现的日志，所有存储引擎都可以使用；
- redo log 是 Innodb 存储引擎实现的日志；

*2、**文件格式不同**：*

- binlog 有 3 种格式类型，分别是 STATEMENT（默认格式）、ROW、 MIXED，区别如下：
	- STATEMENT：每一条修改数据的 SQL 都会被记录到 binlog 中（相当于记录了逻辑操作，所以针对这种格式， binlog 可以称为逻辑日志），主从复制中 slave 端再根据 SQL 语句重现。但 STATEMENT 有动态函数的问题，比如你用了 uuid 或者 now 这些函数，你在主库上执行的结果并不是你在从库执行的结果，这种随时在变的函数会导致复制的数据不一致；
	- ROW：记录行数据最终被修改成什么样了（这种格式的日志，就不能称为逻辑日志了），不会出现 STATEMENT 下动态函数的问题。但 ROW 的缺点是每行数据的变化结果都会被记录，比如执行批量 update 语句，更新多少行数据就会产生多少条记录，使 binlog 文件过大，而在 STATEMENT 格式下只会记录一个 update 语句而已；
	- MIXED：包含了 STATEMENT 和 ROW 模式，它会根据不同的情况自动使用 ROW 模式和 STATEMENT 模式；
- redo log 是物理日志，记录的是在某个数据页做了什么修改，比如对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了AAA 更新；

*3、**写入方式不同**：*

- binlog 是追加写，写满一个文件，就创建一个新的文件继续写，不会覆盖以前的日志，保存的是全量的日志。
- redo log 是循环写，日志空间大小是固定，全部写满就从头开始，保存未被刷入磁盘的脏页日志。

*4、**用途不同**：*

- binlog 用于备份恢复、主从复制；
- redo log 用于掉电等故障恢复。





### Redo Log

1、 记录更新时，InnoDB引擎就会先把记录写到RedoLog（粉板）里面，并更新内存。同时，InnoDB引擎会在空闲时将这个操作记录更新到磁盘里面。

2、 如果更新太多RedoLog处理不了的时候，需先将RedoLog部分数据写到磁盘，然后擦除RedoLog部分数据。RedoLog类似转盘。



Redo Log 记录的是新数据的备份（和 Undo Log 相反）。在事务提交前，只要将 Redo Log 持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是 Redo Log 已经持久化。系统可以根据 Redo Log 的内容，将所有数据恢复到崩溃之前的状态。

redo log 是物理日志，记录了某个数据页做了什么修改，对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了AAA 更新，每当执行一个事务就会产生这样的一条物理日志。

在事务提交时，只要先将 redo log 持久化到磁盘即可，可以不需要将缓存在 Buffer Pool 里的脏页数据持久化到磁盘。当系统崩溃时，虽然脏页数据没有持久化，但是 redo log 已经持久化，接着 MySQL 重启后，可以根据 redo log 的内容，将所有数据恢复到最新的状态。



- **实现事务的持久性，让 MySQL 有 crash-safe 的能力**，能够保证 MySQL 在任何时间段突然崩溃，重启后之前已提交的记录都不会丢失；
- **将写操作从「随机写」变成了「顺序写」**，提升 MySQL 写入磁盘的性能。



redo log是innodb引擎级别，用来记录innodb存储引擎的事务日志，不管事务是否提交都会记录下来，用于数据恢复。当数据库发生故障，innoDB存储引擎会使用redo log恢复到发生故障前的时刻，以此来保证数据的完整性。将参数innodb_flush_log_at_tx_commit设置为1，那么在执行commit时会将redo log同步写到磁盘。

Buffer Pool 是提高了读写效率没错，但是问题来了，Buffer Pool 是基于内存的，而内存总是不可靠，万一断电重启，还没来得及落盘的脏页数据就会丢失。

为了防止断电导致数据丢失的问题，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log 里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，由后台线程将缓存在 Buffer Pool 的脏页刷新到磁盘里，这就是 **WAL （Write-Ahead Logging）技术**，**指的是 MySQL 的写操作并不是立刻更新到磁盘上，而是先记录在日志上，然后在合适的时间再更新到磁盘上**。



![img](../../Image/2022/07/220723-9.png)

#### 基础概念

##### write pos

是当前记录的位置，一边写一边后移，写到第3号文件末尾后就回到0号文件开头。



##### check point

是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。



#### 实现原理

write pos和check point之间的是粉板上还空着的部分，可以用来记录新的操作。如果write pos追上checkpoint，表示粉板满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把checkpoint推进一下。

有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为`crash-safe`。



##### 刷入磁盘

redo log的写入不是直接落到磁盘，而是在内存中设置了一片称之为`redo log buffer`的连续内存空间，也就是`redo 日志缓冲区`。

在如下的一些情况中，log buffer的数据会刷入磁盘：

- log buffer 空间不足时

log buffer 的大小是有限的，如果不停的往这个有限大小的 log buffer 里塞入日志，很快它就会被填满。如果当前写入 log buffer 的redo 日志量已经占满了 log buffer 总容量的大约**一半**左右，就需要把这些日志刷新到磁盘上。

- 事务提交时

在事务提交时，为了保证持久性，会把log buffer中的日志全部刷到磁盘。注意，这时候，除了本事务的，可能还会刷入其它事务的日志。

- 后台线程输入

有一个后台线程，大约每秒都会刷新一次`log buffer`中的`redo log`到磁盘。

- 正常关闭服务器时
- **触发checkpoint规则**



重做日志缓存、重做日志文件都是以**块（block）**的方式进行保存的，称之为**重做日志块（redo log block）**,块的大小是固定的512字节。我们的redo log它是固定大小的，可以看作是一个逻辑上的 **log group**，由一定数量的**log block** 组成。

它的写入方式是从头到尾开始写，写到末尾又回到开头循环写。

其中有两个标记位置：

`write pos`是当前记录的位置，一边写一边后移，写到第3号文件末尾后就回到0号文件开头。`checkpoint`是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到磁盘。

当`write_pos`追上`checkpoint`时，表示redo log日志已经写满。这时候就不能接着往里写数据了，需要执行`checkpoint`规则腾出可写空间。

所谓的**checkpoint规则**，就是checkpoint触发后，将buffer中日志页都刷到磁盘。



#### 扩展

##### 被修改 Undo 页面，需要记录对应 redo log 吗？

需要的。

开启事务后，InnoDB 层更新记录前，首先要记录相应的 undo log，如果是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面。

不过，在修改该 Undo 页面前需要先记录对应的 redo log，所以**先记录修改 Undo 页面的 redo log ，然后再真正的修改 Undo 页面**。



##### redo log 和 undo log 区别在哪？

这两种日志是属于 InnoDB 存储引擎的日志，它们的区别在于：

- redo log 记录了此次事务**「完成后」**的数据状态，记录的是更新之**「后」**的值；
- undo log 记录了此次事务**「开始前」**的数据状态，记录的是更新之**「前」**的值；

事务提交之前发生了崩溃，重启后会通过 undo log 回滚事务，事务提交之后发生了崩溃，重启后会通过 redo log 恢复事务，如下图：

![事务恢复](../../Image/2022/07/220723-10.png)





所以有了 redo log，再通过 WAL 技术，InnoDB 就可以保证即使数据库发生异常重启，之前已提交的记录都不会丢失，这个能力称为 **crash-safe**（崩溃恢复）。可以看出来， **redo log 保证了事务四大特性中的持久性**。



##### bin log和redo log有什么区别？

最开始 MySQL 并没与 InnoDB 引擎( InnoDB 引擎是其他公司以插件形式插入 MySQL 的) ，MySQL 自带的引擎是 MyISAM，但是我们知道 redo log 是 InnoDB 引擎特有的，其他存储引擎都没有，这就导致会没有 crash-safe 的能力(crash-safe 的能力即使数据库发生异常重启，之前提交的记录都不会丢失)，binlog 日志只能用来归档。

bin log会记录所有日志记录，包括InnoDB、MyISAM等存储引擎的日志；redo log只记录innoDB自身的事务日志。

bin log只在事务提交前写入到磁盘，一个事务只写一次；而在事务进行过程，会有redo log不断写入磁盘。

bin log是逻辑日志，记录的是SQL语句的原始逻辑；redo log是物理日志，记录的是在某个数据页上做了什么修改。



1. redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用。
2. redo log是物理日志，记录的是在某个数据页上做了什么修改；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如给ID=2这一行的c字段加1。
3. redo log是循环写的，空间固定会用完；binlog是可以追加写入的。追加写是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



- bin log会记录所有与数据库有关的日志记录，包括InnoDB、MyISAM等存储引擎的日志，而redo log只记InnoDB存储引擎的日志。
- 记录的内容不同，bin log记录的是关于一个事务的具体操作内容，即该日志是逻辑日志。而redo log记录的是关于每个页（Page）的更改的物理情况。
- 写入的时间不同，bin log仅在事务提交前进行提交，也就是只写磁盘一次。而在事务进行的过程中，却不断有redo ertry被写入redo log中。
- 写入的方式也不相同，redo log是循环写入和擦除，bin log是追加写入，不会覆盖已经写的文件。



##### redo log 要写到磁盘，数据也要写磁盘，为什么要多此一举？

写入 redo log 的方式使用了追加操作， 所以磁盘操作是**顺序写**，而写入数据需要先找到写入位置，然后才写到磁盘，所以磁盘操作是**随机写**。

磁盘的「顺序写 」比「随机写」 高效的多，因此 redo log 写入磁盘的开销更小。

针对「顺序写」为什么比「随机写」更快这个问题，可以比喻为你有一个本子，按照顺序一页一页写肯定比写一个字都要找到对应页写快得多。

可以说这是 WAL 技术的另外一个优点：**MySQL 的写操作从磁盘的「随机写」变成了「顺序写」**，提升语句的执行性能。这是因为 MySQL 的写操作并不是立刻更新到磁盘上，而是先记录在日志上，然后在合适的时间再更新到磁盘上 。



##### 产生的 redo log 是直接写入磁盘的吗？

实际上， 执行一个事务，产生的 redo log 也不是直接写入磁盘的，因为这样会产生大量的 I/O 操作，而且磁盘的运行速度远慢于内存。

所以，redo log 也有自己的缓存—— **redo log buffer**，每当产生一条 redo log 时，会先写入到 redo log buffer，后续在持久化到磁盘如下图：

![事务恢复](../../Image/2022/07/220723-11.png)

redo log buffer 默认大小 16 MB，可以通过 `innodb_log_Buffer_size` 参数动态的调整大小，增大它的大小可以让 MySQL 处理「大事务」是不必写入磁盘，进而提升写 IO 性能。



##### redo log 什么时候刷盘？

缓存在 redo log buffe 里的 redo log 还是在内存中，它什么时候刷新到磁盘？

主要有下面几个时机：

- MySQL 正常关闭时；
- 当 redo log buffer 中记录的写入量大于 redo log buffer 内存空间的一半时，会触发落盘；
- InnoDB 的后台线程每隔 1 秒，将 redo log buffer 持久化到磁盘。
- 每次事务提交时都将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘（这个策略可由 innodb_flush_log_at_trx_commit 参数控制，下面会说）。



##### innodb_flush_log_at_trx_commit 参数控制的是什么？

单独执行一个更新语句的时候，InnoDB 引擎会自己启动一个事务，在执行更新语句的过程中，生成的 redo log 先写入到 redo log buffer 中，然后等事务提交的时候，再将缓存在 redo log buffer 中的 redo log 按组的方式「顺序写」到磁盘。

上面这种 redo log 刷盘时机是在事务提交的时候，这个默认的行为。

除此之外，InnoDB 还提供了另外两种策略，由参数 `innodb_flush_log_at_trx_commit` 参数控制，可取的值有：0、1、2，默认值为 1，这三个值分别代表的策略如下：

- 当设置该**参数为 0 时**，表示每次事务提交时 ，还是**将 redo log 留在 redo log buffer 中** ，该模式下在事务提交时不会主动触发写入磁盘的操作。
- 当设置该**参数为 1 时**，表示每次事务提交时，都**将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘**，这样可以保证 MySQL 异常重启之后数据不会丢失。
- 当设置该**参数为 2 时**，表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log **写到 redo log 文件，注意写入到「 redo log 文件」并不意味着写入到了磁盘**，因为操作系统的文件系统中有个 Page Cache（如果你想了解 Page Cache，可以看[这篇 (opens new window)](https://xiaolincoding.com/os/6_file_system/pagecache.html)），Page Cache 是专门用来缓存文件数据的，所以写入「 redo log文件」意味着写入到了操作系统的文件缓存。



![img](../../Image/2022/07/220723-12.png)



##### innodb_flush_log_at_trx_commit 为 0 和 2 的时候，什么时候才将 redo log 写入磁盘？

InnoDB 的后台线程每隔 1 秒：

- 针对参数 0 ：会把缓存在 redo log buffer 中的 redo log ，通过调用 `write()` 写到操作系统的 Page Cache，然后调用 `fsync()` 持久化到磁盘。**所以参数为 0 的策略，MySQL 进程的崩溃会导致上一秒钟所有事务数据的丢失**;
- 针对参数 2 ：调用 fsync，将缓存在操作系统中 Page Cache 里的 redo log 持久化到磁盘。**所以参数为 2 的策略，较取值为 0 情况下更安全，因为 MySQL 进程的崩溃并不会丢失数据，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失**。

加入了后台现线程后，innodb_flush_log_at_trx_commit 的刷盘时机如下图：



![img](../../Image/2022/07/220723-13.png)

##### 这三个参数的应用场景是什么？

这三个参数的数据安全性和写入性能的比较如下：

- 数据安全性：参数 1 > 参数 2 > 参数 0
- 写入性能：参数 0 > 参数 2> 参数 1

所以，数据安全性和写入性能是熊掌不可得兼的，**要不追求数据安全性，牺牲性能；要不追求性能，牺牲数据安全性**。

- 在一些对数据安全性要求比较高的场景中，显然 `innodb_flush_log_at_trx_commit` 参数需要设置为 1。
- 在一些可以容忍数据库崩溃时丢失 1s 数据的场景中，我们可以将该值设置为 0，这样可以明显地减少日志同步到磁盘的 I/O 操作。
- 安全性和性能折中的方案就是参数 2，虽然参数 2 没有参数 0 的性能高，但是数据安全性方面比参数 0 强，因为参数 2 只要操作系统不宕机，即使数据库崩溃了，也不会丢失数据，同时性能方便比参数 1 高。



##### redo log 文件写满了怎么办？

默认情况下， InnoDB 存储引擎有 1 个重做日志文件组( redo log Group），「重做日志文件组」由有 2 个 redo log 文件组成，这两个 redo 日志的文件名叫 ：`ib_logfile0` 和 `ib_logfile1` 。

在重做日志组中，每个 redo log File 的大小是固定且一致的，假设每个 redo log File 设置的上限是 1 GB，那么总共就可以记录 2GB 的操作。

重做日志文件组是以**循环写**的方式工作的，从头开始写，写到末尾就又回到开头，相当于一个环形。

所以 InnoDB 存储引擎会先写 ib_logfile0 文件，当 ib_logfile0 文件被写满的时候，会切换至 ib_logfile1 文件，当 ib_logfile1 文件也被写满时，会切换回 ib_logfile0 文件。



![重做日志文件组写入过程](../../Image/2022/07/220723-14.png)



我们知道 redo log 是为了防止Buffer Pool 中的脏页丢失而设计的，那么如果随着系统运行，Buffer Pool 的脏页刷新到了磁盘中，那么 redo log 对应的记录也就没用了，这时候我们擦除这些旧记录，以腾出空间记录新的更新操作。

redo log 是循环写的方式，相当于一个环形，InnoDB 用 write pos 表示 redo log 当前记录写到的位置，用 checkpoint 表示当前要擦除的位置，如下图：

<img src="../../Image/2022/07/220723-15.png" alt="img" style="zoom:50%;" />



图中的：

- write pos 和 checkpoint 的移动都是顺时针方向；
- write pos ～ checkpoint 之间的部分（图中的红色部分），用来记录新的更新操作；
- check point ～ write pos 之间的部分（图中蓝色部分）：待落盘的脏数据页记录；

如果 write pos 追上了 checkpoint，就意味着 **redo log 文件满了，这时 MySQL 不能再执行新的更新操作，也就是说 MySQL 会被阻塞**（*因此所以针对并发量大的系统，适当设置 redo log 的文件大小非常重要*），此时**会停下来将 Buffer Pool 中的脏页刷新到磁盘中，然后标记 redo log 哪些记录可以被擦除，接着对旧的 redo log 记录进行擦除，等擦除完旧记录腾出了空间，checkpoint 就会往后移动（图中顺时针）**，然后 MySQL 恢复正常运行，继续执行新的更新操作。

所以，一次 checkpoint 的过程就是脏页刷新到磁盘中变成干净页，然后标记 redo log 哪些记录可以被覆盖的过程。



##### 如果不小心整个数据库的数据被删除了，能使用 redo log 文件恢复数据吗？

不可以使用 redo log 文件恢复，只能使用 binlog 文件恢复。

因为 redo log 文件是循环写，是会边写边擦除日志的，只记录未被刷入磁盘的数据的物理日志，已经刷入磁盘的数据都会从 redo log 文件里擦除。

binlog 文件保存的是全量的日志，也就是保存了所有数据变更的情况，理论上只要记录在 binlog 上的数据，都可以恢复，所以如果不小心整个数据库的数据被删除了，得用 binlog 文件恢复数据。



### Undo Log

在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为 Undo Log），然后进行数据的修改。如果出现了错误或者用户执行了 Rollback 语句，系统可以利用 Undo Log 中的备份将数据恢复到事务开始之前的状态。**通俗易懂地说，它只关心过去的数据。**

除了记录redo log外，当进行数据修改时还会记录undo log，undo log用于数据的撤回操作，它保留了记录修改前的内容。通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC。



- Undo log是InnoDB MVCC事务特性的重要组成部分。当我们对记录做了变更操作时就会产生undo记录，Undo记录默认被记录到系统表空间(ibdata)中，但从5.6开始，也可以使用独立的Undo 表空间。

- Undo记录中存储的是老版本数据，当一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。当版本链很长时，通常可以认为这是个比较耗时的操作（例如bug#69812）。

- 大多数对数据的变更操作包括INSERT/DELETE/UPDATE，其中INSERT操作在事务提交前只对当前事务可见，因此产生的Undo日志可以在事务提交后直接删除（谁会对刚插入的数据有可见性需求呢！！），而对于UPDATE/DELETE则需要维护多版本信息，在InnoDB里，UPDATE和DELETE操作产生的Undo日志被归成一类，即update_undo

- 另外, 在回滚段中的undo logs分为: `insert undo log` 和 `update undo log`

	- insert undo log : 事务对insert新记录时产生的undolog, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
	- update undo log : 事务对记录进行delete和update操作时产生的undo log, 不仅在事务回滚时需要, 一致性读也需要，所以不能随便删除，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除




我们在执行执行一条“增删改”语句的时候，虽然没有输入 begin 开启事务和 commit 提交事务，但是 MySQL 会**隐式开启事务**来执行“增删改”语句的，执行完就自动提交事务的，这样就保证了执行完“增删改”语句后，我们可以及时在数据库表看到“增删改”的结果了。

执行一条语句是否自动提交事务，是由 `autocommit` 参数决定的，默认是开启。所以，执行一条 update 语句也是会使用事务的。

那么，考虑一个问题。一个事务在执行过程中，在还没有提交事务之前，如果MySQL 发生了崩溃，要怎么回滚到事务之前的数据呢？

如果我们每次在事务执行过程中，都记录下回滚时需要的信息到一个日志里，那么在事务执行中途发生了 MySQL 崩溃后，就不用担心无法回滚到事务之前的数据，我们可以通过这个日志回滚到事务之前的数据。

实现这一机制就是 undo log（回滚日志），它保证了事务的 ACID 特性中的原子性（Atomicity）。

undo log 是一种用于撤销回退的日志。在事务没提交之前，MySQL 会先记录更新前的数据到 undo log 日志文件里面，当事务回滚时，可以利用 undo log 来进行回滚。如下图：

![回滚事务](../../Image/2022/07/220723-6.png)

每当 InnoDB 引擎对一条记录进行操作（修改、删除、新增）时，要把回滚时需要的信息都记录到 undo log 里，比如：

- 在**插入**一条记录时，要把这条记录的主键值记下来，这样之后回滚时只需要把这个主键值对应的记录**删掉**就好了；
- 在**删除**一条记录时，要把这条记录中的内容都记下来，这样之后回滚时再把由这些内容组成的记录**插入**到表中就好了；
- 在**更新**一条记录时，要把被更新的列的旧值记下来，这样之后回滚时再把这些列**更新为旧值**就好了。

在发生回滚时，就读取 undo log 里的数据，然后做原先相反操作。比如当 delete 一条记录时，undo log 中会把记录中的内容都记下来，然后执行回滚操作的时候，就读取 undo log 里的数据，然后进行 insert 操作。

不同的操作，需要记录的内容也是不同的，所以不同类型的操作（修改、删除、新增）产生的 undo log 的格式也是不同的，具体的每一个操作的 undo log 的格式我就不详细介绍了，感兴趣的可以自己去查查。

一条记录的每一次更新操作产生的 undo log 格式都有一个 roll_pointer 指针和一个 trx_id 事务id：

- 通过 trx_id 可以知道该记录是被哪个事务修改的；
- 通过 roll_pointer 指针可以将这些 undo log 串成一个链表，这个链表就被称为版本链；

版本链如下图：

![版本链](../../Image/2022/07/220723-5.png)

另外，**undo log 还有一个作用，通过 ReadView + undo log 实现 MVCC（多版本并发控制）**。

对于「读提交」和「可重复读」隔离级别的事务来说，它们的快照读（普通 select 语句）是通过 Read View + undo log 来实现的，它们的区别在于创建 Read View 的时机不同：

- 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
- 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

这两个隔离级别实现是通过「事务的 Read View 里的字段」和「记录中的两个隐藏列（trx_id 和 roll_pointer）」的比对，如果不满足可见行，就会顺着 undo log 版本链里找到满足其可见性的记录，从而控制并发事务访问同一个记录时的行为，这就叫 MVCC（多版本并发控制）。

因此，undo log 两大作用：

- **实现事务回滚，保障事务的原子性**。事务处理过程中，如果出现了错误或者用户执 行了 ROLLBACK 语句，MySQL 可以利用 undo log 中的历史数据将数据恢复到事务开始之前的状态。
- **实现 MVCC（多版本并发控制）关键因素之一**。MVCC 是通过 ReadView + undo log 实现的。undo log 为每条记录保存多份历史数据，MySQL 在执行快照读（普通 select 语句）的时候，会根据事务的 Read View 里的信息，顺着 undo log 的版本链找到满足其可见性的记录。



### Relay Log



## 实现原理

### 两阶段提交

> 1 RedoLog prepare阶段 -->  2 写binlog  --> 3 RedoLog commit

1. 写入更新数据，innodb 引擎将数据保存在内存中，同时记录`redo log`，此时`redo log`进入 `prepare`状态。
2. 执行器收到通知后记录`binlog`，然后调用引擎接口，提交`redo log`为`commit`状态。



为什么记录完`redo log`，不直接提交，而是先进入`prepare`状态？

假设先写`redo log`直接提交，然后写`binlog`，写完`redo log`后，机器挂了，`binlog`日志没有被写入，那么机器重启后，这台机器会通过`redo log`恢复数据，但是这个时候`binlog`并没有记录该数据，后续进行机器备份的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。

**先写 binlog，然后写 redo log**，假设写完了 binlog，机器异常重启了，由于没有 redo log，本机是无法恢复这一条记录的，但是 binlog 又有记录，那么和上面同样的道理，就会产生数据不一致的情况。



如果采用 redo log 两阶段提交的方式就不一样了，写完 binglog 后，然后再提交 redo log 就会防止出现上述的问题，从而保证了数据的一致性。那么问题来了，有没有一个极端的情况呢？假设 redo log 处于预提交状态，binglog 也已经写完了，这个时候发生了异常重启会怎么样呢？ 这个就要依赖于 MySQL 的处理机制了，MySQL 的处理过程如下：

- 判断 redo log 是否完整，如果判断是完整的，就立即提交。

- 如果 redo log 只是预提交但不是 commit 状态，这个时候就会去判断 binlog 是否完整，如果完整就提交 redo log, 不完整就回滚事务。



事务提交后，redo log 和 binlog 都要持久化到磁盘，但是这两个是独立的逻辑，可能出现半成功的状态，这样就造成两份日志之间的逻辑不一致。

举个例子，假设 id = 1 这行数据的字段 name 的值原本是 'jay'，然后执行 `UPDATE t_user SET name = 'xiaolin' WHERE id = 1;` 如果在持久化 redo log 和 binlog 两个日志的过程中，出现了半成功状态，那么就有两种情况：

- **如果在将 redo log 刷入到磁盘之后， MySQL 突然宕机了，而 binlog 还没有来得及写入**。MySQL 重启后，通过 redo log 能将 Buffer Pool 中 id = 1 这行数据的 name 字段恢复到新值 xiaolin，但是 binlog 里面没有记录这条更新语句，在主从架构中，binlog 会被复制到从库，由于 binlog 丢失了这条更新语句，从库的这一行 name 字段是旧值 jay，与主库的值不一致性；
- **如果在将 binlog 刷入到磁盘之后， MySQL 突然宕机了，而 redo log 还没有来得及写入**。由于 redo log 还没写，崩溃恢复以后这个事务无效，所以 id = 1 这行数据的 name 字段还是旧值 jay，而 binlog 里面记录了这条更新语句，在主从架构中，binlog 会被复制到从库，从库执行了这条更新语句，那么这一行 name 字段是新值 xiaolin，与主库的值不一致性；

可以看到，在持久化 redo log 和 binlog 这两份日志的时候，如果出现半成功的状态，就会造成主从环境的数据不一致性。这是因为 redo log 影响主库的数据，binlog 影响从库的数据，所以 redo log 和 binlog 必须保持一致才能保证主从数据一致。

**MySQL 为了避免出现两份日志之间的逻辑不一致的问题，使用了「两阶段提交」来解决**，两阶段提交其实是分布式事务一致性协议，它可以保证多个逻辑操作要不全部成功，要不全部失败，不会出现半成功的状态。

**两阶段提交把单个事务的提交拆分成了 2 个阶段，分别是分别是「准备（Prepare）阶段」和「提交（Commit）阶段」**，每个阶段都由协调者（Coordinator）和参与者（Participant）共同完成。注意，不要把提交（Commit）阶段和 commit 语句混淆了，commit 语句执行的时候，会包含提交（Commit）阶段。



**数据恢复**

1. 当在2之前崩溃时，重启恢复后发现没有commit，回滚。备份恢复：没有binlog 。一致
2. 当在3之前崩溃时，重启恢复发现虽没有commit，但满足prepare和binlog完整，所以重启后会`自动`commit。备份：有binlog. 一致



#### 实现过程

在 MySQL 的 InnoDB 存储引擎中，开启 binlog 的情况下，MySQL 会同时维护 binlog 日志与 InnoDB 的 redo log，为了保证这两个日志的一致性，MySQL 使用了**内部 XA 事务**（是的，也有外部 XA 事务，跟本文不太相关，我就不介绍了），内部 XA 事务由 binlog 作为协调者，存储引擎是参与者。

当客户端执行 commit 语句或者在自动提交的情况下，MySQL 内部开启一个 XA 事务，**分两阶段来完成 XA 事务的提交**，如下图：

![image-20220725220656484](../../Image/2022/07/220725-10.png)

从图中可看出，事务的提交过程有两个阶段，就是**将 redo log 的写入拆成了两个步骤：prepare 和 commit，中间再穿插写入binlog**，具体如下：

- **prepare 阶段**：将 XID（内部 XA 事务的 ID） 写入到 redo log，同时将 redo log 对应的事务状态设置为 prepare，然后将 redo log 刷新到硬盘；
- **commit 阶段**：把 XID 写入到 binlog，然后将 binlog 刷新到磁盘，接着调用引擎的提交事务接口，将 redo log 状态设置为 commit（将事务设置为 commit 状态后，刷入到磁盘 redo log 文件，所以 commit 状态也是会刷盘的）；



#### 异常情况

在两阶段提交的不同时刻，MySQL 异常重启会出现什么现象？下图中有时刻 A 和时刻 B 都有可能发生崩溃：

![image-20220725221145874](../../Image/2022/07/220725-11.png)

不管是时刻 A（已经 redo log，还没写入 binlog），还是时刻 B （已经写入 redo log 和 binlog，还没写入 commit 标识）崩溃，**此时的 redo log 都处于 prepare 状态**。

在 MySQL 重启后会按顺序扫描 redo log 文件，碰到处于 prepare 状态的 redo log，就拿着 redo log 中的 XID 去 binlog 查看是否存在此 XID：

- **如果 binlog 中没有当前内部 XA 事务的 XID，说明 redolog 完成刷盘，但是 binlog 还没有刷盘，则回滚事务**。对应时刻 A 崩溃恢复的情况。
- **如果 binlog 中有当前内部 XA 事务的 XID，说明 redolog 和 binlog 都已经完成了刷盘，则提交事务**。对应时刻 B 崩溃恢复的情况。

可以看到，**对于处于 prepare 阶段的 redo log，即可以提交事务，也可以回滚事务，这取决于是否能在 binlog 中查找到与 redo log 相同的 XID**，如果有就提交事务，如果没有就回滚事务。这样就可以保证 redo log 和 binlog 这两份日志的一致性了。

所以说，**两阶段提交是以 binlog 写成功为事务提交成功的标识**，因为 binlog 写成功了，就意味着能在 binlog 中查找到与 redo log 相同的 XID。



**先写入redo log，后写入binlog**

在写完redo log之后，数据此时具有`crash-safe`能力，因此系统崩溃，数据会恢复成事务开始之前的状态。但是，若在redo log写完时候，binlog写入之前，系统发生了宕机。此时binlog没有对上面的更新语句进行保存，导致当使用binlog进行数据库的备份或者恢复时，就少了上述的更新语句。从而使得`id=2`这一行的数据没有被更新。



**先写入binlog，后写入redo log**

写完binlog之后，所有的语句都被保存，所以通过binlog复制或恢复出来的数据库中id=2这一行的数据会被更新为a=1。但是如果在redo log写入之前，系统崩溃，那么redo log中记录的这个事务会无效，导致实际数据库中`id=2`这一行的数据并没有更新。



#### 存在问题

两阶段提交虽然保证了两个日志文件的数据一致性，但是性能很差，主要有两个方面的影响：

- **磁盘 I/O 次数高**：对于“双1”配置，每个事务提交都会进行两次 fsync（刷盘），一次是 redo log 刷盘，另一次是 binlog 刷盘。
- **锁竞争激烈**：两阶段提交虽然能够保证「单事务」两个日志的内容一致，但在「多事务」的情况下，却不能保证两者的提交顺序一致，因此，在两阶段提交的流程基础上，还需要加一个锁来保证提交的原子性，从而保证多事务的情况下，两个日志的提交顺序一致。



##### 磁盘 I/O 次数高

binlog 和 redo log 在内存中都对应的缓存空间，binlog 会缓存在 binlog cache，redo log 会缓存在 redo log buffer，它们持久化到磁盘的时机分别由下面这两个参数控制。一般我们为了避免日志丢失的风险，会将这两个参数设置为 1：

- 当 sync_binlog = 1 的时候，表示每次提交事务都会将 binlog cache 里的 binlog 直接持久到磁盘；
- 当 innodb_flush_log_at_trx_commit = 1 时，表示每次事务提交时，都将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘；

可以看到，如果 sync_binlog 和 当 innodb_flush_log_at_trx_commit 都设置为 1，那么在每个事务提交过程中， 都会至少调用 2 次刷盘操作，一次是 redo log 刷盘，一次是 binlog 落盘，所以这会成为性能瓶颈。



##### 锁竞争激烈

在早期的 MySQL 版本中，通过使用 prepare_commit_mutex 锁来保证事务提交的顺序，在一个事务获取到锁时才能进入 prepare 阶段，一直到 commit 阶段结束才能释放锁，下个事务才可以继续进行 prepare 操作。

通过加锁虽然完美地解决了顺序一致性的问题，但在并发量较大的时候，就会导致对锁的争用，性能不佳。



#### 扩展

##### 处于 prepare 阶段的 redo log 加上完整 binlog，重启就提交事务，MySQL 为什么要这么设计?

binlog 已经写入了，之后就会被从库（或者用这个 binlog 恢复出来的库）使用。

所以，在主库上也要提交这个事务。采用这个策略，主库和备库的数据就保证了一致性。



##### 事务没提交的时候，redo log 会被持久化到磁盘吗？

会的。事务执行中间过程的 redo log 也是直接写在 redo log buffer 中的，这些缓存在 redo log buffer 里的 redo log 也会被「后台线程」每隔一秒一起持久化到磁盘。

也就是说，事务没提交的时候，redo log 也是可能被持久化到磁盘的。

有的同学可能会问，如果 mysql 崩溃了，还没提交事务的 redo log 已经被持久化磁盘了，mysql 重启后，数据不就不一致了？

放心，这种情况 mysql 重启会进行回滚操作，因为事务没提交的时候，binlog 是还没持久化到磁盘的。

所以， redo log 可以在事务没提交之前持久化到磁盘，但是 binlog 必须在事务提交之后，才可以持久化到磁盘。



### 组提交

**MySQL 引入了 binlog 组提交（group commit）机制，当有多个事务提交的时候，会将多个 binlog 刷盘操作合并成一个，从而减少磁盘 I/O 的次数**，如果说 10 个事务依次排队刷盘的时间成本是 10，那么将这 10 个事务一次性一起刷盘的时间成本则近似于 1。

引入了组提交机制后，prepare 阶段不变，只针对 commit 阶段，将 commit 阶段拆分为三个过程：

- **flush 阶段**：多个事务按进入的顺序将 binlog 从 cache 写入文件（不刷盘）；
- **sync 阶段**：对 binlog 文件做 fsync 操作（多个事务的 binlog 合并一次刷盘）；
- **commit 阶段**：各个事务按顺序做 InnoDB commit 操作；

上面的**每个阶段都有一个队列**，每个阶段有锁进行保护，因此保证了事务写入的顺序，第一个进入队列的事务会成为 leader，leader领导所在队列的所有事务，全权负责整队的操作，完成后通知队内其他事务操作结束。

对每个阶段引入了队列后，锁就只针对每个队列进行保护，不再锁住提交事务的整个过程，可以看的出来，**锁粒度减小了，这样就使得多个阶段可以并发执行，从而提升效率**。



MySQL 5.6 没有 redo log 组提交，MySQL 5.7 有 redo log 组提交。

在 MySQL 5.6 的组提交逻辑中，每个事务各自执行 prepare 阶段，也就是各自将 redo log 刷盘，这样就没办法对 redo log 进行组提交。

所以在 MySQL 5.7 版本中，做了个改进，在 prepare 阶段不再让事务各自执行 redo log 刷盘操作，而是推迟到组提交的 flush 阶段，也就是说 prepare 阶段融合在了 flush 阶段。

这个优化是将 redo log 的刷盘延迟到了 flush 阶段之中，sync 阶段之前。通过延迟写 redo log 的方式，为 redolog 做了一次组写入，这样 binlog 和 redo log 都进行了优化。

接下来介绍每个阶段的过程，注意下面的过程针对的是“双 1” 配置（sync_binlog 和 innodb_flush_log_at_trx_commit 都配置为 1）。



#### flush 阶段

第一个事务会成为 flush 阶段的 Leader，此时后面到来的事务都是 Follower ：

接着，获取队列中的事务组，由事务组的 Leader 对 rodo log 做一次 write + fsync，即一次将同组事务的 redolog 刷盘：

完成了 prepare 阶段后，将绿色这一组事务执行过程中产生的 binlog 写入 binlog 文件（调用 write，不会调用 fsync，所以不会刷盘，binlog 缓存在操作系统的文件系统中）。

从上面这个过程，可以知道 flush 阶段队列的作用是**用于支撑 redo log 的组提交**。

如果在这一步完成后数据库崩溃，由于 binlog 中没有该组事务的记录，所以 MySQL 会在重启后回滚该组事务。



#### sync 阶段

一组事务的 binlog 写入到 binlog 文件后，并不会马上执行刷盘的操作，而是**会等待一段时间**，这个等待的时长由 `Binlog_group_commit_sync_delay` 参数控制，**目的是为了组合更多事务的 binlog，然后再一起刷盘**。

不过，在等待的过程中，如果事务的数量提前达到了 `Binlog_group_commit_sync_no_delay_count` 参数设置的值，就不用继续等待了，就马上将 binlog 刷盘。

从上面的过程，可以知道 sync 阶段队列的作用是**用于支持 binlog 的组提交**。

如果想提升 binlog 组提交的效果，可以通过设置下面这两个参数来实现：

- `binlog_group_commit_sync_delay= N`，表示在等待 N 微妙后，直接调用 fsync，将处于文件系统中 page cache 中的 binlog 刷盘，也就是将「 binlog 文件」持久化到磁盘。
- `binlog_group_commit_sync_no_delay_count = N`，表示如果队列中的事务数达到 N 个，就忽视binlog_group_commit_sync_delay 的设置，直接调用 fsync，将处于文件系统中 page cache 中的 binlog 刷盘。

如果在这一步完成后数据库崩溃，由于 binlog 中已经有了事务记录，MySQL会在重启后通过 redo log 刷盘的数据继续进行事务的提交。



#### commit 阶段

最后进入 commit 阶段，调用引擎的提交事务接口，将 redo log 状态设置为 commit。

commit 阶段队列的作用是承接 sync 阶段的事务，完成最后的引擎提交，使得 sync 可以尽早的处理下一组事务，最大化组提交的效率。



## 总结

| 日志                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **binlog （归档日志）**  | 是 Server 层生成的日志，主要**用于数据备份和主从复制**；     |
| **redo log（重做日志）** | 是 Innodb 存储引擎层生成的日志，实现了事务中的**持久性**，**将写操作从「随机写」变成了「顺序写」**，主要**用于掉电等故障恢复**； |
| **undo log（回滚日志）** | 是 Innodb 存储引擎层生成的日志，实现了事务中的**原子性**，主要**用于事务回滚和 MVCC**。 |



### MySQL 磁盘 I/O 很高，有什么优化的方法？

现在我们知道事务在提交的时候，需要将 binlog 和 redo log 持久化到磁盘，那么如果出现 MySQL 磁盘 I/O 很高的现象，我们可以通过控制以下参数，来 “延迟” binlog 和 redo log 刷盘的时机，从而降低磁盘 I/O 的频率：

- 设置组提交的两个参数： binlog_group_commit_sync_delay 和 binlog_group_commit_sync_no_delay_count 参数，延迟 binlog 刷盘的时机，从而减少 binlog 的刷盘次数。这个方法是基于“额外的故意等待”来实现的，因此可能会增加语句的响应时间，但即使 MySQL 进程中途挂了，也没有丢失数据的风险，因为 binlog 早被写入到 page cache 了，只要系统没有宕机，缓存在 page cache 里的 binlog 就会被持久化到磁盘。
- 将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000），表示每次提交事务都 write，但累积 N 个事务后才 fsync，相当于延迟了 binlog 刷盘的时机。但是这样做的风险是，主机掉电时会丢 N 个事务的 binlog 日志。
- 将 innodb_flush_log_at_trx_commit 设置为 2。表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log 写到 redo log 文件，注意写入到「 redo log 文件」并不意味着写入到了磁盘，因为操作系统的文件系统中有个 Page Cache，专门用来缓存文件数据的，所以写入「 redo log文件」意味着写入到了操作系统的文件缓存，然后交由操作系统控制持久化到磁盘的时机。但是这样做的风险是，主机掉电的时候会丢数据。




# MySQL 存储引擎
MySQL中常用的四种存储引擎分别是：MyISAM、InnoDB、MEMORY、ARCHIVE。MySQL 5.5版本后默认的存储引擎为InnoDB。

| 功能         | MylSAM | MEMORY | InnoDB |
| :----------- | :----- | :----- | :----- |
| 存储限制     | 256TB  | RAM    | 64TB   |
| 支持事务     | No     | No     | Yes    |
| 支持全文索引 | Yes    | No     | Yes    |
| 支持树索引   | Yes    | Yes    | Yes    |
| 支持哈希索引 | No     | Yes    | Yes    |
| 支持数据缓存 | No     | N/A    | Yes    |
| 支持外键     | No     | No     | Yes    |



大多数情况下，使用默认的InnoDB就够了。如果要提供提交、回滚和恢复的事务安全（ACID 兼容）能力，并要求实现并发控制，InnoDB 就是比较靠前的选择了。

如果数据表主要用来插入和查询记录，则 MyISAM 引擎提供较高的处理效率。

如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 MEMORY 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。



## 基础概念

### 相关语法

```mysql
# 查看MySQL提供的所有存储引擎
show engines;

# 查看默认的存储引擎
show variables like '%storage_engine%';

# 查看表的存储引擎
show table status like "table_name";
```



## InnoDB
InnoDB是MySQL默认（V5.5之后）的事务型存储引擎，使用最广泛，基于聚簇索引建立的。InnoDB内部做了很多优化，如能够自动在内存中创建自适应hash索引，以加速读操作。

优点：支持事务和崩溃修复能力；引入了行级锁和外键约束。

缺点：占用的数据空间相对较大。

适用场景：需要事务支持，并且有较高的并发读写频率。



### 数据存储

#### 存储方式

InnoDB下，一张表的所有数据都是存放在该表的聚簇索引，即主键索引的叶子结点中的。



#### 数据文件

```
[root@izuf6e56zt7ebjhby2hxboz l@002dsixth@002dservice]## ll
total 10488
-rw-r----- 1 mysql mysql  147456 Mar 28 12:28 it_blog_classify.ibd
-rw-r----- 1 mysql mysql  114688 Dec  2 14:15 it_blogger.ibd
-rw-r----- 1 mysql mysql 9437184 Mar 28 23:18 it_blog.ibd
-rw-r----- 1 mysql mysql  114688 Dec  7 10:19 it_blog_tag.ibd
```



| 文件格式 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| .frm     | 存储表定义的文件                                             |
| .ibd     | 存储索引和数据的文件。既该表的所有索引树，所有行记录数据都存储在该文件中。 |



### 索引支持

- 根据索引类型来分：

| 索引名称 | 描述     | 最早支持版本 | 是否常用 |
| -------- | -------- | ------------ | -------- |
| NORMAL   | 普通索引 |              | 是       |
| UNIQUE   | 唯一索引 |              | 是       |
| SPATIAL  | 空间索引 |              | 否       |
| FULLTEXT | 全文索引 | MySQL 5.6    | 否       |



- 根据索引结构来分

| 结构名称 | 备注                                                   |
| -------- | ------------------------------------------------------ |
| B+Tree   | 支持                                                   |
| HASH     | Innodb不支持真正的hash结构，这里是一种自适应的hash索引 |



#### 数据结构角度

##### B+Tree

- **主键索引**

叶子节点存储与主键对应的行数据。



![](../../Image/2022/04/220406-3.png)



- 非主键索引

叶子节点存储的是主键，查找数据时先根据索引找到匹配的主键，再根据主键索引进行查询。

![](../../Image/2022/04/220406-2.png)



##### 自适应HASH

Innodb不支持真正的hash结构，这里是一种自适应的hash索引。

- 虽然InnoDB不支持哈希索引，但是它依然有曲线救国的手段，就是支持一种伪哈希索引的方式，变相支持哈希索引。
- 这样的伪哈希索引，我们叫它为**自适应哈希索引**。但这个**自适应哈希索引**并不是由我们人为控制建立的。而InnoDB存储引擎引擎自动优化创建，不受人为干预的



什么是哈希表，相信我们大家都知道，通过O(1)的时间复杂度，我们就可以查询到想要的数据，但是需要付出O(n)的空间复杂度代价。这也是InnoDB不支持哈希索引的原因之一
那么什么是自适应哈希索引呢？说白了，它也是哈希索引，但是它不为表中的所有数据都建立索引。而是有选择性的为一些热点数据建立哈希索引。
既Innodb存储引擎会监控对某表的辅助键索引查找情况，如果发现某辅助键索引被频繁访问，既代表某些关键字是热数据，于是这些数据则会被放入哈希索引中，由此让特定频繁被访问的热点数据可以享受到哈希索引O(1)的速度。



#### 主键角度

##### 主键索引

主键索引采用的是聚簇索引，叶子结点的每个关键字对应的数据，**存放的都是完整的行数据**。既InnoDB下，一张表的所有数据都是存放在该表的聚簇索引叶子结点中的。



##### 辅助索引

辅助键索引采用的是非聚簇索引，叶子结点的每个关键字对应的数据，**存放的都是主键信息**。


## MyISAM
数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，可以使用MyISAM引擎。MyISAM会将表存储在两个文件中，数据文件.MYD和索引文件.MYI。

优点：访问速度快。

缺点：MyISAM不支持事务和行级锁，不支持崩溃后的安全恢复，也不支持外键。

适用场景：对事务完整性没有要求；表的数据都会只读的。

### 数据存储

#### 存储方式

**在MyISAM下，主键索引和辅助键索引都属于非聚簇索引。**非聚簇索引的叶子结点的关键字的值都是对应数据在数据文件中的地址，就是叶子结点存储的不是数据本身，而仅仅是一个指针。

MyISAM存储引擎的数据文本和索引是分开存储的，索引是索引，索引的叶子结点存储的也仅仅是指针。



#### 数据文件

```
[root@izuf6e56zt7ebjhby2hxboz springboot]## ll
total 120
-rw-r----- 1 mysql mysql  10605 Mar 28 23:45 jpa_user_1732.sdi
-rw-r----- 1 mysql mysql    736 Mar 28 23:45 jpa_user.MYD
-rw-r----- 1 mysql mysql   2048 Mar 28 23:45 jpa_user.MYI
-rw-r----- 1 mysql mysql  10608 Mar 28 23:45 mybatis_user_1733.sdi
-rw-r----- 1 mysql mysql    220 Mar 28 23:45 mybatis_user.MYD
-rw-r----- 1 mysql mysql   2048 Mar 28 23:45 mybatis_user.MYI
```



| 文件格式 | 描述                   |
| -------- | ---------------------- |
| .frm     | 存储表定义的文件       |
| .MYD     | 存储所有行数据的文件   |
| .MYI     | 存储索引相关数据的文件 |



### 索引支持

#### 数据结构角度

##### B+Tree索引

MyISAM中的主键索引和非主键索引一样，都是非聚簇索引，存储的是行数据地址的指针。

![1](../../Image/2022/04/220406-1.png)



### MyISAM和InnoDB的区别

**是否支持行级锁** : MyISAM 只有表级锁，而InnoDB 支持行级锁和表级锁，默认为行级锁。

**是否支持事务和崩溃后的安全恢复**：MyISAM 不提供事务支持。而InnoDB提供事务支持，具有事务、回滚和崩溃修复能力。

**是否支持外键**： MyISAM不支持，而InnoDB支持。

**是否支持MVCC** ：MyISAM不支持，InnoDB支持。应对高并发事务，MVCC比单纯的加锁更高效。

**MyISAM不支持聚集索引，InnoDB支持聚集索引。**

叶子节点存储的是行数据的内存地址。



**1.  存储结构**：每个MyISAM在磁盘上存储成三个文件；InnoDB所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB表的大小只受限于操作系统文件的大小，一般为2GB。

**2. 事务支持**：MyISAM不提供事务支持；InnoDB提供事务支持事务，具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全特性。

**3  最小锁粒度**：MyISAM只支持表级锁，更新时会锁住整张表，导致其它查询和更新都会被阻塞InnoDB支持行级锁。

**4. 索引类型**：MyISAM的索引为聚簇索引，数据结构是B树；InnoDB的索引是非聚簇索引，数据结构是B+树。

**5.  主键必需**：MyISAM允许没有任何索引和主键的表存在；InnoDB如果没有设定主键或者非空唯一索引，**就会自动生成一个6字节的主键(用户不可见)**，数据是主索引的一部分，附加索引保存的是主索引的值。

**6. 表的具体行数**：MyISAM保存了表的总行数，如果select count(*) from table;会直接取出出该值; InnoDB没有保存表的总行数，如果使用select count(*) from table；就会遍历整个表;但是在加了wehre条件后，MyISAM和InnoDB处理的方式都一样。

**7.  外键支持**：MyISAM不支持外键；InnoDB支持外键。



## MEMORY
MEMORY引擎将数据全部放在内存中，访问速度较快，但是一旦系统奔溃的话，数据都会丢失。

MEMORY引擎默认使用哈希索引，将键的哈希值和指向数据行的指针保存在哈希索引中。

优点：访问速度较快。

缺点：

哈希索引数据不是按照索引值顺序存储，无法用于排序。
不支持部分索引匹配查找，因为哈希索引是使用索引列的全部内容来计算哈希值的。
只支持等值比较，不支持范围查询。
当出现哈希冲突时，存储引擎需要遍历链表中所有的行指针，逐行进行比较，直到找到符合条件的行。

## ARCHIVE

ARCHIVE存储引擎非常适合存储大量独立的、作为历史记录的数据。ARCHIVE提供了压缩功能，拥有高效的插入速度，但是这种引擎不支持索引，所以查询性能较差。



# MySQL 配置

## 数据库连接

### 查看数据库连接

```mysql
# 查看当前 MySQL 连接情况
show processlist;
show full processlist;
```



返回参数如下：

1. **id**：线程ID，可以用`kill id`杀死某个线程

2. **db**：数据库名称

3. **user**：数据库用户

4. **host**：数据库实例的IP

5. **command**：当前执行的命令，比如`Sleep`，`Query`，`Connect`等

6. **time**：消耗时间，单位秒

7. **state**：执行状态，主要有以下状态：

8. - `Sleep`，线程正在等待客户端发送新的请求
	- `Locked`，线程正在等待锁
	- `Sending data`，正在处理`SELECT`查询的记录，同时把结果发送给客户端
	- `Kill`，正在执行`kill`语句，杀死指定线程
	- `Connect`，一个从节点连上了主节点
	- `Quit`，线程正在退出
	- `Sorting for group`，正在为`GROUP BY`做排序
	- `Sorting for order`，正在为`ORDER BY`做排序

9. **info**：正在执行的`SQL`语句



### 空闲连接

当连接详情的 Command 列的状态为 Sleep ，这意味着该连接没有再执行过任何命令，也就是说这是一个空闲的连接，并且空闲的时长是 736 秒（ Time 列）。

MySQL 定义了空闲连接的最大空闲时长，由 wait_timeout 参数控制的，默认值是 8 小时（28880秒），如果空闲连接超过了这个时间，连接器就会自动将它断开。

```mysql
> show variables like 'wait_timeout'
```



查询结果：

| Variable_name | Value |
| ------------- | ----- |
| wait_timeout  | 28800 |



也可以手动断开空闲的连接，使用的是 kill connection + id 的命令。

```mysql
> kill connection +6;
```



一个处于空闲状态的连接被服务端主动断开后，这个客户端并不会马上知道，等到客户端在发起下一个请求的时候，才会收到这样的报错“ERROR 2013 (HY000): Lost connection to MySQL server during query”。



### 连接限制

MySQL 服务支持的最大连接数由 max_connections 参数控制，默认是 151 个，超过这个值，系统就会拒绝接下来的连接请求，并报错提示“Too many connections”。

```mysql
> show variables like 'max_connections';
```



### 长连接占用

MySQL 的连接也跟 HTTP 一样，有短连接和长连接的概念，它们的区别如下：

```bash
// 短连接
连接 mysql 服务（TCP 三次握手）
执行sql
断开 mysql 服务（TCP 四次挥手）

// 长连接
连接 mysql 服务（TCP 三次握手）
执行sql
执行sql
执行sql
....
断开 mysql 服务（TCP 四次挥手）
```



使用长连接的好处就是可以减少建立连接和断开连接的过程，所以一般是推荐使用长连接。

但是，使用长连接后可能会占用内存增多，因为 MySQL 在执行查询过程中临时使用内存管理连接对象，这些连接对象资源只有在连接断开时才会释放。如果长连接累计很多，将导致 MySQL 服务占用内存太大，有可能会被系统强制杀掉，这样会发生 MySQL 服务异常重启的现象。

有两种解决方式。

第一种，**定期断开长连接**。既然断开连接后就会释放连接占用的内存资源，那么我们可以定期断开长连接。

第二种，**客户端主动重置连接**。MySQL 5.7 版本实现了 `mysql_reset_connection()` 函数的接口，注意这是接口函数不是命令，那么当客户端执行了一个很大的操作后，在代码里调用 mysql_reset_connection 函数来重置连接，达到释放内存的效果。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。



# MySQL 分库分表

## 分库

## 分表


### 水平分表
水平分表指的是将表中的数据，按不同的规则分别存储到对应的分表中。分表后表的结构不变，数据量变小。

**保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。 水平拆分可以支撑非常大的数据量。**

水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。

水平拆分可以支持非常大的数据量。需要注意的一点是：分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以 **水平拆分最好分库** 。

水平拆分能够 **支持非常大的数据量存储，应用端改造也少**，但 **分片事务难以解决** ，跨节点Join性能较差，逻辑复杂。《Java工程师修炼之道》的作者推荐 **尽量不要对数据进行分片，因为拆分会带来逻辑、部署、运维的各种复杂度** ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。



#### 原理

水平划分是根据一定规则，例如时间或id序列值等进行数据的拆分。比如根据年份来拆分不同的数据库。每个数据库结构一致，但是数据得以拆分，从而提升性能。



##### 优点

- 单库（表）的数据量得以减少，提高性能；切分出的表结构相同，程序改动较少。
- 提高了系统的稳定性和负载能力



##### 缺点

- 分片事务一致性难以解决
- 跨节点join性能差，逻辑复杂
- 数据分片在扩容时需要迁移



#### 常用方式

##### 主键取模或Hash



##### 时间范围取值



#### 水平分表规范

##### 禁止使用自增主键

- 资源浪费
- 范围分表时会产生尾部热点问题，最新的分表的压力较大。



##### 禁止使用UUID替代自增主键

UUID是无序的128位长的唯一字符串，较为浪费空间，而且会产生频繁的索引重排。主键索引在B+Tree索引中是有序的数据结构，UUID每次生成的都是无序的。

因此推荐使用雪花算法来生成递增唯一的主键ID。



**下面补充一下数据库分片的两种常见方案：**

- **客户端代理：** **分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。** 当当网的 **Sharding-JDBC** 、阿里的TDDL是两种比较常用的实现。
- **中间件代理：** **在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。** 我们现在谈的 **Mycat** 、360的Atlas、网易的DDB等等都是这种架构的实现。



### 垂直分表

垂直分表指将字段较多的表拆成多张字段较少的表，分表后表的结构发生改变，表的数据量不变。分表之间以主键关联。

垂直划分数据库是根据业务进行划分，例如购物场景，可以将库中涉及商品、订单、用户的表分别划分出成一个库，通过降低单库的大小来提高性能。同样的，分表的情况就是将一个大表根据业务功能拆分成一个个子表，例如商品基本信息和商品描述，商品基本信息一般会展示在商品列表，商品描述在商品详情页，可以将商品基本信息和商品描述拆分成两张表。

**根据数据库里面数据表的相关性进行拆分。** 例如，用户表中既有用户的登录信息又有用户的基本信息，可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

**简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。** 如下图所示，这样来说大家应该就更容易理解了。



#### 原理

MySQL中默认每一页数据的大小是16K，然后跨页获取数据的效率较低。因此为了减少跨页获取数据，就要尽可能多的增加每一页的数据行数。然后查询获取数据后通过主键到其他分表中获取其他字段。



- **垂直拆分的优点：** 可以使得列数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。
- **垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；



##### 优点

- 行记录变小，数据页可以存放更多记录，在查询时减少I/O次数。
- 可以达到最大化利用Cache的目的，具体在垂直拆分的时候可以将不常变的字段放一起，将经常改变的放一起
- 数据维护简单



##### 缺点

- 主键出现冗余，需要管理冗余列；
- 会引起表连接JOIN操作，可以通过在业务服务器上进行join来减少数据库压力；
- 依然存在单表数据量过大的问题。
- 事务处理复杂



#### 字段拆分规范

表拆分时，应将数据查询、排序时需要的字段和高频访问的小字段作为主要查询表。将低频访问的字段和大字段作为通过主键查询的表。



## 存在问题

### 事务问题

### 跨库关联

### 排序问题

### 分页问题

### 分布式ID

因为要是分成多个表之后，每个表都是从 1 开始累加，这样是不对的，我们需要一个全局唯一的 id 来支持。

生成全局 id 有下面这几种方式：

- **UUID**：不适合作为主键，因为太长了，并且无序不可读，查询效率低。比较适合用于生成唯一的名字的标示比如文件的名字。
- **数据库自增 id** : 两台数据库分别设置不同步长，生成不重复ID的策略来实现高可用。这种方式生成的 id 有序，但是需要独立部署数据库实例，成本高，还会有性能瓶颈。
- **利用 redis 生成 id :** 性能比较好，灵活方便，不依赖于数据库。但是，引入了新的组件造成系统更加复杂，可用性降低，编码更加复杂，增加了系统成本。
- **Twitter的snowflake算法** ：Github 地址：https://github.com/twitter-archive/snowflake。
- **美团的[Leaf](https://tech.meituan.com/2017/04/21/mt-leaf.html)分布式ID生成系统** ：Leaf 是美团开源的分布式ID生成器，能保证全局唯一性、趋势递增、单调递增、信息安全，里面也提到了几种分布式方案的对比，但也需要依赖关系数据库、Zookeeper等中间件。感觉还不错。美团技术团队的一篇文章：https://tech.meituan.com/2017/04/21/mt-leaf.html 。



## 解决方案

由于水平拆分牵涉的逻辑比较复杂，当前也有了不少比较成熟的解决方案。这些方案分为两大类：客户端架构和代理架构。



### 客户端架构

通过修改数据访问层，如JDBC、Data Source、MyBatis，通过配置来管理多个数据源，直连数据库，并在模块内完成数据的分片整合，一般以Jar包的方式呈现。

![Alt text](../../Image/2022/07/220722-5.png)

分片的实现是和应用服务器在一起的，通过修改Spring JDBC层来实现

客户端架构的优点是：

- 应用直连数据库，降低外围系统依赖所带来的宕机风险
- 集成成本低，无需额外运维的组件

缺点是：

- 限于只能在数据库访问层上做文章，扩展性一般，对于比较复杂的系统可能会力不从心
- 将分片逻辑的压力放在应用服务器上，造成额外风险



### 代理架构

通过独立的中间件来统一管理所有数据源和数据分片整合，后端数据库集群对前端应用程序透明，需要独立部署和运维代理组件。

![Alt text](../../Image/2022/07/220722-6.png)



代理组件为了分流和防止单点，一般以集群形式存在，同时可能需要Zookeeper之类的服务组件来管理

代理架构的优点是：

- 能够处理非常复杂的需求，不受数据库访问层原来实现的限制，扩展性强
- 对于应用服务器透明且没有增加任何额外负载

缺点是：

- 需部署和运维独立的代理中间件，成本高
- 应用需经过代理来连接数据库，网络上多了一跳，性能有损失且有额外风险



### 解决方案总结

如此多的方案，如何进行选择？可以按以下思路来考虑：

1. 确定是使用代理架构还是客户端架构。中小型规模或是比较简单的场景倾向于选择客户端架构，复杂场景或大规模系统倾向选择代理架构
2. 具体功能是否满足，比如需要跨节点`ORDER BY`，那么支持该功能的优先考虑
3. 不考虑一年内没有更新的产品，说明开发停滞，甚至无人维护和技术支持
4. 最好按大公司->社区->小公司->个人这样的出品方顺序来选择
5. 选择口碑较好的，比如github星数、使用者数量质量和使用者反馈
6. 开源的优先，往往项目有特殊需求可能需要改动源代码

按照上述思路，推荐以下选择：

- 客户端架构：ShardingJDBC
- 代理架构：MyCat或者Atlas



## 总结

### 分表原则

- 能不分就不分，参考单表优化
- 分片数量尽量少，分片尽量均匀分布在多个数据结点上，因为一个查询SQL跨分片越多，则总体性能越差，虽然要好于所有数据在一个分片的结果，只在必要的时候进行扩容，增加分片数量
- 分片规则需要慎重选择做好提前规划，分片规则的选择，需要考虑数据的增长模式，数据的访问模式，分片关联性问题，以及分片扩容问题，最近的分片策略为范围分片，枚举分片，一致性Hash分片，这几种分片都有利于扩容
- 尽量不要在一个事务中的SQL跨越多个分片，分布式事务一直是个不好处理的问题
- 查询条件尽量优化，尽量避免Select * 的方式，大量数据结果集下，会消耗大量带宽和CPU资源，查询尽量避免返回大量结果集，并且尽量为频繁使用的查询语句建立索引。
- 通过数据冗余和表分区赖降低跨库Join的可能

这里特别强调一下分片规则的选择问题，如果某个表的数据有明显的时间特征，比如订单、交易记录等，则他们通常比较合适用时间范围分片，因为具有时效性的数据，我们往往关注其近期的数据，查询条件中往往带有时间字段进行过滤，比较好的方案是，当前活跃的数据，采用跨度比较短的时间段进行分片，而历史性的数据，则采用比较长的跨度存储。

总体上来说，分片的选择是取决于最频繁的查询SQL的条件，因为不带任何Where语句的查询SQL，会遍历所有的分片，性能相对最差，因此这种SQL越多，对系统的影响越大，所以我们要尽量避免这种SQL的产生。



# MySQL 主从同步

## 实现原理

MySQL 的主从复制依赖于 binlog ，也就是记录 MySQL 上的所有变化并以二进制形式保存在磁盘上。复制的过程就是将 binlog 中的数据从主库传输到从库上。

这个过程一般是**异步**的，也就是主库上执行事务操作的线程不会等待复制 binlog 的线程同步完成。



![image-20220725210812936](../../Image/2022/07/220725-7.png)



1、主节点必须启用二进制日志，记录任何修改了数据库数据的事件。

2、从节点开启一个线程（I/O Thread)把自己扮演成 mysql 的客户端，通过 mysql 协议，请求主节点的二进制日志文件中的事件 。

3、主节点启动一个线程（dump Thread），检查自己二进制日志中的事件，跟对方请求的位置对比，如果不带请求位置参数，则主节点就会从第一个日志文件中的第一个事件一个一个发送给从节点。

4、从节点接收到主节点发送过来的数据把它放置到中继日志（Relay log）文件中。并记录该次请求到主节点的具体哪一个二进制日志文件内部的哪一个位置（主节点中的二进制文件会有多个）。

5、从节点启动另外一个线程（sql Thread ），把 Relay log 中的事件读取出来，并在本地再执行一次。



MySQL 集群的主从复制过程梳理成 3 个阶段：

- **写入 Binlog**：主库写 binlog 日志，提交事务，并更新本地存储数据。
- **同步 Binlog**：把 binlog 复制到所有从库上，每个从库把 binlog 写到暂存日志中。
- **回放 Binlog**：回放 binlog，并更新存储引擎中的数据。

具体详细过程如下：

- MySQL 主库在收到客户端提交事务的请求之后，会先写入 binlog，再提交事务，更新存储引擎中的数据，事务提交完成后，返回给客户端“操作成功”的响应。
- 从库会创建一个专门的 I/O 线程，连接主库的 log dump 线程，来接收主库的 binlog 日志，再把 binlog 信息写入 relay log 的中继日志里，再返回给主库“复制成功”的响应。
- 从库会创建一个用于回放 binlog 的线程，去读 relay log 中继日志，然后回放 binlog 更新存储引擎中的数据，最终实现主从的数据一致性。

在完成主从复制之后，你就可以在写数据时只写主库，在读数据时只读从库，这样即使写请求会锁表或者锁记录，也不会影响读请求的执行。



![image-20220725210936405](../../Image/2022/07/220725-8.png)



因为从库数量增加，从库连接上来的 I/O 线程也比较多，**主库也要创建同样多的 log dump 线程来处理复制的请求，对主库资源消耗比较高，同时还受限于主库的网络带宽**。

所以在实际使用中，从库是不是越多越好，一个主库一般跟 2～3 个从库（1 套数据库，1 主 2 从 1 备主），这就是一主多从的 MySQL 集群结构。



### binlog

在数据主从同步时，不同格式的 binlog 也对事务隔离级别有要求。

MySQL的binlog主要支持三种格式，分别是statement、row以及mixed，但是，RC 隔离级别只支持row格式的binlog。如果指定了mixed作为 binlog 格式，**那么如果使用RC，服务器会自动使用基于row 格式的日志记录。**

**而 RR 的隔离级别同时支持statement、row以及mixed三种。**



## 复制模型

mysql默认的复制方式是`异步`的，并且复制的时候是有`并行复制能力`的。主库把日志发送给从库后不管了，这样会产生一个问题就是假设主库挂了，从库处理失败了，这时候从库升为主库后，**日志就丢失了**。由此产生两个概念。



- **全同步复制**：MySQL 主库提交事务的线程要等待所有从库的复制成功响应，才返回客户端结果。这种方式在实际项目中，基本上没法用，原因有两个：一是性能很差，因为要复制到所有节点才返回响应；二是可用性也很差，主库和所有从库任何一个数据库出问题，都会影响业务。

	主库写入binlog后强制同步日志到从库，**所有的从库都执行完成后才返回给客户端**，但是很显然这个方式的话性能会受到严重影响。

	

- **异步复制**（默认模型）：MySQL 主库提交事务的线程并不会等待 binlog 同步到各从库，就返回客户端结果。这种模式一旦主库宕机，数据就会发生丢失。



- **半同步复制**：MySQL 5.7 版本之后增加的一种复制方式，介于两者之间，事务线程不用等待所有的从库复制成功响应，只要一部分复制成功响应回来就行，比如一主二从的集群，只要数据成功复制到任意一个从库上，主库的事务线程就可以返回给客户端。这种**半同步复制的方式，兼顾了异步复制和同步复制的优点，即使出现主库宕机，至少还有一个从库有最新的数据，不存在数据丢失的风险**。

	半同步复制的逻辑是这样，从库写入日志成功后返回`ACK`确认给主库，主库收到至少一个从库的确认就认为写操作完成。



# MySQL 规范和优化

## 使用规范

### 表规范

**1:N 关系的设计**

在从表（`N`的这一方）创建一个字段，以字段作为外键指向主表（`1`的这一方）的主键。如学生表是多（`N`）的一方，会有个字段`class_id`保存班级表的主键。



**N:N关系的设计**

通过增加第三张表，把`N:N`修改为两个 `1:N`。



### 命名规范

- 表名、字段名必须使用小写字母或者数字并用下划线分割，禁止使用数字开头，禁止使用拼音，并且一般不使用英文缩写。
- 主键索引名为`pk_字段名`；唯一索引名为`uk_字段名`；普通索引名则为`idx_字段名`。
- 临时库表必须以tmp_为前缀并以日期为后缀，备份表必须以bak_为前缀并以日期(时间戳)为后缀
- 所有数据库对象名称禁止使用MySQL保留关键字
- 数据库对象的命名要能做到见名识意，并且最好不要超过32个字符
- 所有存储相同数据的列名和类型必须一致



### 基本设计规范

**所有表必须使用Innodb存储引擎**

```
没有特殊要求（即Innodb无法满足的功能如：列存储，存储空间数据等）的情况下，所有表必须使用Innodb存储引擎（mysql5.5之前默认使用Myisam，5.6以后默认的为Innodb）
Innodb 支持事务，支持行级锁，更好的恢复性，高并发下性能更好
```



**数据库和表的字符集统一使用UTF8**

```
兼容性更好，统一字符集可以避免由于字符集转换产生的乱码，不同的字符集进行比较前需要进行转换会造成索引失效，如果数据库中有存储emoji表情的需要，字符集需要采用utf8mb4字符集
```



**所有表和字段都需要添加注释**

```
使用comment从句添加表和列的备注
从一开始就进行数据字典的维护
```



**尽量控制单表数据量的大小，建议控制在500万以内**

500万并不是Mysql数据库的限制，过大会造成修改表结构，备份，恢复都会有很大的问题
可以用历史数据归档（应用于日志数据），分库分表（应用于业务数据）等手段来控制数据量大小



**谨慎使用Mysql分区表**

```
分区表在物理上表现为多个文件，在逻辑上表现为一个表
谨慎选择分区键，跨分区查询效率可能更低
建议采用物理分表的方式管理大数据
```



**尽量做到冷热数据分离，减小表的宽度**

```
Mysql限制每个表最多存储4096列，并且每一行数据的大小不能超过65535字节

减少磁盘IO,保证热数据的内存缓存命中率（表越宽，把表装载进内存缓冲池时所占用的内存也就越大,也会消耗更多的IO）
更有效的利用缓存，避免读入无用的冷数据
经常一起使用的列放到一个表中（避免更多的关联操作）
```



**禁止在表中建立预留字段**

```
预留字段的命名很难做到见名识义
预留字段无法确认存储的数据类型，所以无法选择合适的类型
对预留字段类型的修改，会对表进行锁定
```



**禁止在数据库中存储图片，文件等大的二进制数据**

```
通常文件很大，会短时间内造成数据量快速增长，数据库进行数据库读取时，通常会进行大量的随机IO操作，文件很大时，IO操作很耗时
通常存储于文件服务器，数据库只存储文件地址信息
```

**禁止在线上做数据库压力测试**

**禁止从开发环境，测试环境直接连接生成环境数据库**



### 字段设计规范

- 尽可能选择存储空间小的字段类型，就好像数字类型的，从`tinyint、smallint、int、bigint`从左往右开始选择
- 小数类型如金额，则选择 `decimal`，禁止使用 `float` 和 `double`。
- 如果存储的字符串长度几乎相等，使用 `char` 定长字符串类型。
- `varchar`是可变长字符串，不预先分配存储空间，长度不要超过`5000`。
- 如果存储的值太大，建议字段类型修改为`text`，同时抽出单独一张表，用主键与之对应。
- 同一表中，所有`varchar`字段的长度加起来，不能大于`65535`. 如果有这样的需求，请使用`TEXT/LONGTEXT `类型。
- 选择合适的字段长度，一般是2的幂；



**优先选择符合存储需要的最小的数据类型**

```
原因是：列的字段越大，建立索引时所需要的空间也就越大，这样一页中所能存储的索引节点的数量也就越少也越少，在遍历时所需要的IO次数也就越多，
索引的性能也就越差
方法：
```

- 将字符串转换成数字类型存储，如：将IP地址转换成整形数据

mysql提供了两个方法来处理ip地址

inet_aton 把ip转为无符号整型(4-8位)
inet_ntoa 把整型的ip转为地址

插入数据前，先用inet_aton把ip地址转为整型，可以节省空间
显示数据时，使用inet_ntoa把整型的ip地址转为地址显示即可。

- 对于非负型的数据（如自增ID、整型IP）来说，要优先使用无符号整型来存储

因为：无符号相对于有符号可以多出一倍的存储空间
SIGNED INT -2147483648~2147483647
UNSIGNED INT 0~4294967295

VARCHAR(N)中的N代表的是字符数，而不是字节数
使用UTF8存储255个汉字 Varchar(255)=765个字节

过大的长度会消耗更多的内存



**避免使用TEXT、BLOB数据类型，最常见的TEXT类型可以存储64k的数据**

- 建议把BLOB或是TEXT列分离到单独的扩展表中

Mysql内存临时表不支持TEXT、BLOB这样的大数据类型，如果查询中包含这样的数据，在排序等操作时，就不能使用内存临时表，必须使用磁盘临时表进行
而且对于这种数据，Mysql还是要进行二次查询，会使sql性能变得很差，但是不是说一定不能使用这样的数据类型

如果一定要使用，建议把BLOB或是TEXT列分离到单独的扩展表中，查询时一定不要使用select * 而只需要取出必要的列，不需要TEXT列的数据时不要对该列进行查询

- TEXT或BLOB类型只能使用前缀索引

因为MySQL对索引字段长度是有限制的，所以TEXT类型只能使用前缀索引，并且TEXT列上是不能有默认值的



**避免使用ENUM类型**

修改ENUM值需要使用ALTER语句，ENUM类型的ORDER BY操作效率低，需要额外操作，禁止使用数值作为ENUM的枚举值。



**尽可能把所有列定义为NOT NULL**

索引NULL列需要额外的空间来保存，所以要占用更多的空间，进行比较和计算时要对NULL值做特别的处理。

- 首先，` NOT NULL` 可以防止出现空指针问题。
- 其次，`NULL`值存储也需要额外的空间的，它也会导致比较运算更为复杂，使优化器难以优化SQL。
- `NULL`值有可能会导致索引失效
- 如果将字段默认设置成一个空字符串或常量值并没有什么不同，且都不会影响到应用逻辑， 那就可以将这个字段设置为`NOT NULL`。



**使用TIMESTAMP（4个字节）或DATETIME类型（8个字节）存储时间**

TIMESTAMP 存储的时间范围 1970-01-01 00:00:01 ~ 2038-01-19-03:14:07，TIMESTAMP 占用4字节和INT相同，但比INT可读性高，超出TIMESTAMP取值范围的使用DATETIME类型存储。

经常会有人用字符串存储日期型的数据（不正确的做法）

- 缺点1：无法用日期函数进行计算和比较

- 缺点2：用字符串存储日期要占用更多的空间



**同财务相关的金额类数据必须使用decimal类型**

- 非精准浮点：float,double
- 精准浮点：decimal

Decimal类型为精准浮点数，在计算时不会丢失精度

占用空间由定义的宽度决定，每4个字节可以存储9位数字，并且小数点要占用一个字节

可用于存储比bigint更大的整型数据



**添加通用字段**

表必备一般来说，或具备这几个字段：

- id：主键，一个表必须得有主键，必须
- create_time：创建时间，必须
- modifed_time/update_time: 修改时间，必须，更新记录时，需要更新它
- version : 数据记录的版本号，用于乐观锁，非必须
- remark ：数据记录备注，非必须
- modified_by :修改人，非必须
- creator ：创建人，非必须



**一张表的字段不宜过多**

如果一张表的字段过多，表中保存的数据可能就会很大，查询效率就会很低。如果业务需求，实在需要很多字段，可以把一张大的表，拆成多张小的表，它们的主键相同即可。当表的字段数非常多时，可以将表分成两张表，一张作为条件查询表，一张作为详细内容表 (主要是为了性能考虑)。



**不要创建外键关联**

- 使用外键存在性能问题、并发死锁问题、使用起来不方便等等。每次做`DELETE`或者`UPDATE`都必须考虑外键约束，会导致开发的时候很难受,测试数据造数据也不方便。
- 还有一个场景不能使用外键，就是分库分表。



### 索引设计规范

**限制每张表上的索引数量，建议单张表索引不超过5个**

```
索引并不是越多越好！索引可以提高效率同样可以降低效率

索引可以增加查询效率，但同样也会降低插入和更新的效率，甚至有些情况下会降低查询效率

因为mysql优化器在选择如何优化查询时，会根据统一信息，对每一个可以用到的索引来进行评估，以生成出一个最好的执行计划，如果同时有很多个
索引都可以用于查询，就会增加mysql优化器生成执行计划的时间，同样会降低查询性能 
```



**禁止给表中的每一列都建立单独的索引**

```
5.6版本之前，一个sql只能使用到一个表中的一个索引，5.6以后，虽然有了合并索引的优化方式，但是还是远远没有使用一个联合索引的查询方式好
```



**每个Innodb表必须有个主键**

```
Innodb是一种索引组织表：数据的存储的逻辑顺序和索引的顺序是相同的
每个表都可以有多个索引，但是表的存储顺序只能有一种
Innodb是按照主键索引的顺序来组织表的

不要使用更新频繁的列作为主键，不适用多列主键（相当于联合索引）
不要使用UUID,MD5,HASH,字符串列作为主键（无法保证数据的顺序增长）
主键建议使用自增ID值
```



**常见索引列建议**

出现在SELECT、UPDATE、DELETE语句的WHERE从句中的列；包含在ORDER BY、GROUP BY、DISTINCT中的字段。并不要将符合1和2中的字段的列都建立一个索引， 通常将1、2中的字段建立联合索引效果更好。多表join的关联列。



**如何选择索引列的顺序**

建立索引的目的是：希望通过索引进行数据查找，减少随机IO，增加查询性能 ，索引能过滤出越少的数据，则从磁盘中读入的数据也就越少

区分度最高的放在联合索引的最左侧（区分度=列中不同值的数量/列的总行数）

尽量把字段长度小的列放在联合索引的最左侧（因为字段长度越小，一页能存储的数据量越大，IO性能也就越好）

使用最频繁的列放到联合索引的左侧（这样可以比较少的建立一些索引）



**避免建立冗余索引和重复索引（增加了查询优化器生成执行计划的时间）**

重复索引示例：primary key(id)、index(id)、unique index(id)

冗余索引示例：index(a,b,c)、index(a,b)、index(a)



**对于频繁的查询优先考虑使用覆盖索引**

覆盖索引：就是包含了所有查询字段(where,select,ordery by,group by包含的字段)的索引

覆盖索引的好处:

- 避免Innodb表进行索引的二次查询

Innodb是以聚集索引的顺序来存储的，对于Innodb来说，二级索引在叶子节点中所保存的是行的主键信息，如果是用二级索引查询数据的话，在查找到相应的键值后，还要通过主键进行二次查询才能获取我们真实所需要的数据。而在覆盖索引中，二级索引的键值中可以获取所有的数据，避免了对主键的二次查询 ，减少了IO操作，提升了查询效率。

- 可以把随机IO变成顺序IO加快查询效率

由于覆盖索引是按键值的顺序存储的，对于IO密集型的范围查找来说，对比随机从磁盘读取每一行的数据IO要少的多，因此利用覆盖索引在访问时也可以把磁盘的随机读取的IO转变成索引查找的顺序IO。



### 索引SET规范

**尽量避免使用外键约束**

不建议使用外键约束（foreign key），但一定要在表与表之间的关联键上建立索引，外键可用于保证数据的参照完整性，但建议在业务端实现。外键会影响父表和子表的写操作从而降低性能。



### SQL开发规范

**建议使用预编译语句进行数据库操作**

```
预编译语句可以重复使用这些计划，减少SQL编译所需要的时间，还可以解决动态SQL所带来的SQL注入的问题
只传参数，比传递SQL语句更高效
相同语句可以一次解析，多次使用，提高处理效率
```



**避免数据类型的隐式转换**

```
隐式转换会导致索引失效
如:  select name,phone from customer where id = '111';
```



**充分利用表上已经存在的索引**

- 避免使用双%号的查询条件。

如 a like '%123%'，（如果无前置%,只有后置%，是可以用到列上的索引的）



- 一个SQL只能利用到复合索引中的一列进行范围查询

```
如 有 a,b,c列的联合索引，在查询条件中有a列的范围查询，则在b,c列上的索引将不会被用到，
在定义联合索引时，如果a列要用到范围查找的话，就要把a列放到联合索引的右侧
```



- 使用left join 或 not exists 来优化not in 操作

```
因为not in 也通常会使用索引失效
```



**数据库设计时，应该要对以后扩展进行考虑**

**程序连接不同的数据库使用不同的账号，禁止跨库查询**

```
为数据库迁移和分库分表留出余地
降低业务耦合度
避免权限过大而产生的安全风险
```



**禁止使用SELECT * 必须使用SELECT <字段列表> 查询**

```
    消耗更多的CPU和IO以网络带宽资源
    无法使用覆盖索引
    可减少表结构变更带来的影响
```



**禁止使用不含字段列表的INSERT语句**

```
如： insert into values ('a','b','c');
应使用 insert into t(c1,c2,c3) values ('a','b','c');
```



**避免使用子查询，可以把子查询优化为join操作**

```
通常子查询在in子句中，且子查询中为简单SQL(不包含union、group by、order by、limit从句)时,才可以把子查询转化为关联查询进行优化

子查询性能差的原因：

 子查询的结果集无法使用索引，通常子查询的结果集会被存储到临时表中，不论是内存临时表还是磁盘临时表都不会存在索引，所以查询性能会受到一定的影响
 特别是对于返回结果集比较大的子查询，其对查询性能的影响也就越大
 由于子查询会产生大量的临时表也没有索引，所以会消耗过多的CPU和IO资源，产生大量的慢查询
```



**避免使用JOIN关联太多的表**

```
对于Mysql来说，是存在关联缓存的，缓存的大小可以由join_buffer_size参数进行设置
在Mysql中，对于同一个SQL多关联（join）一个表，就会多分配一个关联缓存，如果在一个SQL中关联的表越多，
所占用的内存也就越大

如果程序中大量的使用了多表关联的操作，同时join_buffer_size设置的也不合理的情况下，就容易造成服务器内存溢出的情况，
就会影响到服务器数据库性能的稳定性

同时对于关联操作来说，会产生临时表操作，影响查询效率
Mysql最多允许关联61个表，建议不超过5个
```



**减少同数据库的交互次数**

```
数据库更适合处理批量操作
合并多个相同的操作到一起，可以提高处理效率
```



**对应同一列进行or判断时，使用in代替or**

```
in 的值不要超过500个
in 操作可以更有效的利用索引，or大多数情况下很少能利用到索引
```



**禁止使用order by rand() 进行随机排序**

```
会把表中所有符合条件的数据装载到内存中，然后在内存中对所有数据根据随机生成的值进行排序，并且可能会对每一行都生成一个随机值，如果满足条件的数据集非常大，
就会消耗大量的CPU和IO及内存资源
推荐在程序中获取一个随机值，然后从数据库中获取数据的方式
```



**WHERE从句中禁止对列进行函数转换和计算**

```
对列进行函数转换或计算时会导致无法使用索引

不推荐：
where date(create_time)='20190101'
推荐：
where create_time >= '20190101' and create_time < '20190102'
```



**在明显不会有重复值时使用UNION ALL 而不是UNION**

```
UNION 会把两个结果集的所有数据放到临时表中后再进行去重操作
UNION ALL 不会再对结果集进行去重操作
```



**拆分复杂的大SQL为多个小SQL**

```
大SQL:逻辑上比较复杂，需要占用大量CPU进行计算的SQL
MySQL 一个SQL只能使用一个CPU进行计算
SQL拆分后可以通过并行执行来提高处理效率
```



**如果知道查询结果只有一条或者只要最大/最小一条记录，建议用limit 1**

- 加上limit 1后,只要找到了对应的一条记录,就不会继续向下扫描了,效率将会大大提高。
- 当然，如果name是唯一索引的话，是不必要加上limit 1了，因为limit的存在主要就是为了防止全表扫描，从而提高性能,如果一个语句本身可以预知不用全表扫描，有没有limit ，性能的差别并不大。



**优化limit分页**

当偏移量最大的时候，查询效率就会越低，因为Mysql并非是跳过偏移量直接去取后面的数据，而是先把偏移量+要取的条数，然后再把前面偏移量这一段的数据抛弃掉再返回的。

```mysql
//方案一 ：返回上次查询的最大记录(偏移量)
select id，name from employee where id>10000 limit 10.

//方案二：orderby + 索引
select id，name from employee order by id limit 10000，10

//方案三：在业务允许的情况下限制页数：
```



- 如果使用优化方案一，返回上次最大查询记录（偏移量），这样可以跳过偏移量，效率提升不少。
- 方案二使用order by+索引，也是可以提高查询效率的。
- 方案三的话，建议跟业务讨论，有没有必要查这么后的分页啦。因为绝大多数用户都不会往后翻太多页。



**使用where条件限定要查询的数据，避免返回多余的行**

- 需要什么数据，就去查什么数据，避免返回不必要的数据，节省开销。



**Inner join 、left join、right join，优先使用Inner join，如果是left join，左边表结果尽量小**

Inner join 内连接，在两张表进行连接查询时，只保留两张表中完全匹配的结果集；left join 在两张表进行连接查询时，会返回左表所有的行，即使在右表中没有匹配的记录；right join 在两张表进行连接查询时，会返回右表所有的行，即使在左表中没有匹配的记录。



- 如果inner join是等值连接，或许返回的行数比较少，所以性能相对会好一点。
- 同理，使用了左连接，左边表数据结果尽量小，条件尽量放到左边处理，意味着返回的行数可能比较少。



### 操作行为规范

**超100万行的批量写（UPDATE、DELETE、INSERT）操作，要分批多次进行操作**

- 大批量操作可能会造成严重的主从延迟

主从环境中,大批量操作可能会造成严重的主从延迟，大批量的写操作一般都需要执行一定长的时间，
而只有当主库上执行完成后，才会在其他从库上执行，所以会造成主库与从库长时间的延迟情况

- binlog日志为row格式时会产生大量的日志

大批量写操作会产生大量日志，特别是对于row格式二进制数据而言，由于在row格式中会记录每一行数据的修改，我们一次修改的数据越多，
产生的日志量也就会越多，日志的传输和恢复所需要的时间也就越长，这也是造成主从延迟的一个原因

- 避免产生大事务操作

大批量修改数据，一定是在一个事务中进行的，这就会造成表中大批量数据进行锁定，从而导致大量的阻塞，阻塞会对MySQL的性能产生非常大的影响
特别是长时间的阻塞会占满所有数据库的可用连接，这会使生产环境中的其他应用无法连接到数据库，因此一定要注意大批量写操作要进行分批



**对于大表使用pt-online-schema-change修改表结构**

1. 避免大表修改产生的主从延迟
2. 避免在对表字段进行修改时进行锁表

对大表数据结构的修改一定要谨慎，会造成严重的锁表操作，尤其是生产环境，是不能容忍的

pt-online-schema-change它会首先建立一个与原表结构相同的新表，并且在新表上进行表结构的修改，然后再把原表中的数据复制到新表中，并在原表中增加一些触发器。把原表中新增的数据也复制到新表中，在行所有数据复制完成之后，把新表命名成原表，并把原来的表删除掉，把原来一个DDL操作，分解成多个小的批次进行。



**禁止为程序使用的账号赋予super权限**

```
当达到最大连接数限制时，还运行1个有super权限的用户连接
super权限只能留给DBA处理问题的账号使用
```



**对于程序连接数据库账号，遵循权限最小原则**

```
程序使用数据库账号只能在一个DB下使用，不准跨库
程序使用的账号原则上不准有drop权限
```



**慎用distinct关键字**

distinct 关键字一般用来过滤重复记录，以返回不重复的记录。在查询一个字段或者很少字段的情况下使用时，给查询带来优化效果。但是在字段很多的时候使用，却会大大降低查询效率。

带distinct的语句cpu时间和占用时间都高于不带distinct的语句。因为当查询很多字段时，如果使用distinct，数据库引擎就会对数据进行比较，过滤掉重复数据，然而这个比较，过滤的过程会占用系统资源，cpu时间。



**where子句中考虑使用默认值代替null。**

并不是说使用了is null 或者 is not null 就会不走索引了，这个跟mysql版本以及查询成本都有关。

如果把null值，换成默认值，很多时候让走索引成为可能，同时，表达意思会相对清晰一点。

如果mysql优化器发现，走索引比不走索引成本还要高，肯定会放弃索引，这些条件`！=，>is null，is not null`经常被认为让索引失效，其实是因为一般情况下，查询的成本高，优化器自动放弃的。



**exist & in的合理利用**

选择最外层循环小的，也就是，如果**B的数据量小于A，适合使用in，如果B的数据量大于A，即适合选择exist**。



## 使用优化

### 服务器优化

当服务器的内存够多时，配制线程数量 = 最大连接数+5，这样能发挥最大的效率；否则使用配制线程数量<最大连接数启用SQL SERVER的线程池来解决，如果还是数量 = 最大连接数+5，严重的损害服务器的性能；



### 建表优化

- 避免在创建表时NULL是默认值，然后在where子句中对字段进行null值判断，应该使用NOT NULL，或者用0作为默认值；
- 使用数字型字段，若只含数值信息的字段不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销；
- 创建表时字段类型在满足需求的情况下越小越好；使用varchar/nvarchar代替char/nchar，变长字段存储空间小，可以节省存储空间，而且在一个相对较小的字段内查询效率要高些；
- 将需要查询的结果预先计算好放在表中，查询的时候再Select



### 查询优化

- 避免select *，将需要查找的字段列出来；
- 尽量做单表查询，避免任何方式的多表查询；进行多表查询时可将连接（join）查询来代替子查询；
- sql语句尽可能简单：一条sql只能在一个cpu运算；大语句拆小语句，减少锁时间；一条大sql可以堵死整个库；
- 不用函数和触发器，在应用程序实现；
- 使用同类型进行比较，比如用'123'和'123'比，123和123比；
- OR改写成IN：OR的效率是n级别，IN的效率是log(n)级别，in的个数建议控制在200以内；
- 对于连续数值，使用BETWEEN不用IN；
- 列表数据不要拿全表，要使用LIMIT来分页限定，每页数量也不要太大；
- 可以通过将不需要的记录在GROUP BY之前过滤掉来提高GROUP BY语句的效率；
- 当只要一行数据时使用LIMIT 1；
- 不做列运算：SELECT id WHERE age + 1 = 10，任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移至等号右边；
- 应尽量避免在WHERE子句中对字段进行NULL值判断，否则将导致引擎放弃使用索引而进行全表扫描；
- Like左模糊查询也会导致全表扫描；
- 尽量避免在WHERE子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描；
- 应尽量避免在where子句中使用or来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，可以使用UNION合并查询：



#### **利用覆盖索引**

InnoDB使用非主键索引查询数据时会回表，但是如果索引的叶节点中已经包含要查询的字段，那它没有必要再回表查询了，这就叫覆盖索引



#### **低版本避免使用or查询**

在 MySQL 5.0 之前的版本要尽量避免使用 or 查询，可以使用 union 或者子查询来替代，因为早期的 MySQL 版本使用 or 查询可能会导致索引失效，高版本引入了索引合并，解决了这个问题。



#### **避免使用 != 或者 <> 操作符**

SQL中，不等于操作符会导致查询引擎放弃查询索引，引起全表扫描，即使比较的字段上有索引

解决方法：通过把不等于操作符改成or，可以使用索引，避免全表扫描



#### **适当使用前缀索引**

适当地使用前缀所云，可以降低索引的空间占用，提高索引的查询效率。

比如，邮箱的后缀都是固定的“`@xxx.com`”，那么类似这种后面几位为固定值的字段就非常适合定义为前缀索引

需要注意的是，前缀索引也存在缺点，MySQL无法利用前缀索引做order by和group by 操作，也无法作为覆盖索引



#### **避免列上函数运算**

要避免在列字段上进行算术运算或其他表达式运算，否则可能会导致存储引擎无法正确使用索引，从而影响了查询的效率



#### **正确使用联合索引**

使用联合索引的时候，注意最左匹配原则。



#### 禁止3表JOIN查询

- MySQL的SQL优化器在针对超过3张表的关联查询时，NLJ多级嵌套性能差。
- 多张表关联的数据迁移时改造困难。



##### 解决方式

- 单表查出结果后作为条件in查询关联的表。但此时只适用于数据量小且要是INNER JOIN的情况。
- 通过反范式表来查询，反范式表指的是冗余所有的需要的字段到一张大表中。
- 通过数据仓库来获取数据。
- 通过倒排表来获取数据。



#### **优化子查询**

尽量使用 Join 语句来替代子查询，因为子查询是嵌套查询，而嵌套查询会新创建一张临时表，而临时表的创建与销毁会占用一定的系统资源以及花费一定的时间，同时对于返回结果集比较大的子查询，其对查询性能的影响更大



#### **小表驱动大表**

关联查询的时候要拿小表去驱动大表，因为关联的时候，MySQL内部会遍历驱动表，再去连接被驱动表。

比如left join，左表就是驱动表，A表小于B表，建立连接的次数就少，查询速度就被加快了。



#### **适当增加冗余字段**

增加冗余字段可以减少大量的连表查询，因为多张表的连表查询性能很低，所有可以适当的增加冗余字段，以减少多张表的关联查询，这是以空间换时间的优化策略。



#### **利用索引扫描做排序**

MySQL有两种方式生成有序结果：其一是对结果集进行排序的操作，其二是按照索引顺序扫描得出的结果自然是有序的

但是如果索引不能覆盖查询所需列，就不得不每扫描一条记录回表查询一次，这个读操作是随机IO，通常会比顺序全表扫描还慢

因此，在设计索引时，尽可能使用同一个索引既满足排序又用于查找行

只有当索引的列顺序和ORDER BY子句的顺序完全一致，并且所有列的排序方向都一样时，才能够使用索引来对结果做排序



#### **条件下推**

MySQL处理union的策略是先创建临时表，然后将各个查询结果填充到临时表中最后再来做查询，很多优化策略在union查询中都会失效，因为它无法利用索引

最好手工将where、limit等子句下推到union的各个子查询中，以便优化器可以充分利用这些条件进行优化

此外，除非确实需要服务器去重，一定要使用union all，如果不加all关键字，MySQL会给临时表加上distinct选项，这会导致对整个临时表做唯一性检查，代价很高。



### 更新优化

- 当有一批处理的插入或更新时，用批量插入或批量更新，绝不会一条条记录的去更新



### 索引优化

- 索引并不是越多越好，要根据查询有针对性的创建，考虑在WHERE和ORDER BY命令上涉及的列建立索引，可根据EXPLAIN来查看是否用了索引还是全表扫描；
- 值分布很稀少的字段不适合建索引，例如"sex"；
- 尽量不用UNIQUE，由程序保证约束；
- 索引的创建要与应用结合考虑，建议大的OLTP表不要超过6个索引；
- 字符字段只建前缀索引
- 字符字段最好不要做主键
- 使用多列索引时主意顺序和查询条件保持一致，同时删除不必要的单列索引
- 频繁进行数据操作的表，不要建立太多的索引； 



### 其它优化

- 避免对大表查询时进行table scan，必要时考虑新建索引；

​	

### 大表优化

- 限定数据的范围

​	务必禁止不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内；



- 读/写分离

经典的数据库拆分方案，主库负责写，从库负责读；



- 垂直拆分和水平拆分



### 表数据优化

#### optimize table

对于经常修改的表，容易产生碎片，使在查询数据库时必须读取更多的磁盘块，降低查询性能。具有可变长的表都存在磁盘碎片问题，这个问题对blob数据类型更为突出，因为其尺寸变化非常大。

可以通过使用optimize table来整理碎片，保证数据库性能不下降，优化那些受碎片影响的数据表。optimize table可以用于MyISAM和BDB类型的数据表。实际上任何碎片整理方法都是用mysqldump来转存数据表，然后使用转存后的文件并重新建数据表；



## 系统调优

可以使用下面几个工具来做基准测试：

- [sysbench](https://link.segmentfault.com/?enc=K4C%2FJWzWGetm0SjR%2FXLViw%3D%3D.CvVtBn6W0WBRTP6yYQBcyN2ffg%2B%2FU04yb6vKLmbN6FMUbvgKPznfOeBp%2FMEIay3m)：一个模块化，跨平台以及多线程的性能测试工具
- [iibench-mysql](https://link.segmentfault.com/?enc=WYWyQrcDLK9pjdTz9IVu6A%3D%3D.vNPyeW%2BgGlu8TwHhX%2F6p9UmViYFPlGNu5bgzRLmdiI%2FiKtaQ31S2lG0ash%2Fuf1%2Bu)：基于 Java 的 MySQL/Percona/MariaDB 索引进行插入性能测试工具
- [tpcc-mysql](https://link.segmentfault.com/?enc=3udMCZz5B61kAvb%2FA9znuw%3D%3D.TftoccNADM%2FxGy6ndsIsFteGtzs7ruXuBJ1gA%2FQwWV885ZqP14HjGjVyWCLd90i8)：Percona开发的TPC-C测试工具

具体的调优参数内容较多，具体可参考官方文档，这里介绍一些比较重要的参数：

- back_log：back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果MySql的连接数据达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。可以从默认的50升至500
- wait_timeout：数据库连接闲置时间，闲置连接会占用内存资源。可以从默认的8小时减到半小时
- max_user_connection: 最大连接数，默认为0无上限，最好设一个合理上限
- thread_concurrency：并发线程数，设为CPU核数的两倍
- skip_name_resolve：禁止对外部连接进行DNS解析，消除DNS解析时间，但需要所有远程主机用IP访问
- key_buffer_size：索引块的缓存大小，增加会提升索引处理速度，对MyISAM表性能影响最大。对于内存4G左右，可设为256M或384M，通过查询`show status like 'key_read%'`，保证`key_reads / key_read_requests`在0.1%以下最好
- innodb_buffer_pool_size：缓存数据块和索引块，对InnoDB表性能影响最大。通过查询`show status like 'Innodb_buffer_pool_read%'`，保证` (Innodb_buffer_pool_read_requests – Innodb_buffer_pool_reads) / Innodb_buffer_pool_read_requests`越高越好
- innodb_additional_mem_pool_size：InnoDB存储引擎用来存放数据字典信息以及一些内部数据结构的内存空间大小，当数据库对象非常多的时候，适当调整该参数的大小以确保所有数据都能存放在内存中提高访问效率，当过小的时候，MySQL会记录Warning信息到数据库的错误日志中，这时就需要该调整这个参数大小
- innodb_log_buffer_size：InnoDB存储引擎的事务日志所使用的缓冲区，一般来说不建议超过32MB
- query_cache_size：缓存MySQL中的ResultSet，也就是一条SQL语句执行的结果集，所以仅仅只能针对select语句。当某个表的数据有任何任何变化，都会导致所有引用了该表的select语句在Query Cache中的缓存数据失效。所以，当我们的数据变化非常频繁的情况下，使用Query Cache可能会得不偿失。根据命中率`(Qcache_hits/(Qcache_hits+Qcache_inserts)*100))`进行调整，一般不建议太大，256MB可能已经差不多了，大型的配置型静态数据可适当调大.
	可以通过命令`show status like 'Qcache_%'`查看目前系统Query catch使用大小
- read_buffer_size：MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。如果对表的顺序扫描请求非常频繁，可以通过增加该变量值以及内存缓冲区大小提高其性能
- sort_buffer_size：MySql执行排序使用的缓冲大小。如果想要增加`ORDER BY`的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。如果不能，可以尝试增加sort_buffer_size变量的大小
- read_rnd_buffer_size：MySql的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
- record_buffer：每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，可能想要增加该值
- thread_cache_size：保存当前没有与连接关联但是准备为后面新的连接服务的线程，可以快速响应连接的线程请求而无需创建新的
- table_cache：类似于thread_cache_size，但用来缓存表文件，对InnoDB效果不大，主要用于MyISAM



## MySQL优化

SQL优化主要分4个方向：`SQL语句跟索引`、`表结构`、`系统配置`、`硬件`。

总优化思路就是**最大化利用索引**、**尽可能避免全表扫描**、**减少无效数据的查询**：

1、减少数据访问：设置合理的字段类型，启用压缩，通过索引访问等减少磁盘 IO。

2、返回更少的数据：只返回需要的字段和数据分页处理，减少磁盘 IO 及网络 IO。

3、减少交互次数：批量 DML 操作，函数存储等减少数据连接次数。

4、减少服务器 CPU 开销：尽量减少数据库排序操作以及全表查询，减少 CPU 内存占用 。

5、分表分区：使用表分区，可以增加并行操作，更大限度利用 CPU 资源。



### SQL语句和索引

1、合理建立覆盖索引：可以有效减少回表。

2、union，or，in都能命中索引，建议使用in 

3、负向条件(!=、<>、not in、not exists、not like 等) 索引不会使用索引，建议用in。

4、在列上进行运算或使用函数会使索引失效，从而进行全表扫描 

5、小心隐式类型转换，原字符串用整型会触发CAST函数导致索引失效。原int用字符串则会走索引。

6、不建议使用%前缀模糊查询。

7、多表关联查询时，小表在前，大表在后。在 MySQL 中，执行 from 后的表关联查询是从左往右执行的(Oracle 相反)，第一张表会涉及到全表扫描。

8、调整 Where 字句中的连接顺序，MySQL 采用从左往右，自上而下的顺序解析 where 子句。根据这个原理，应将过滤数据多的条件往前放，最快速度缩小结果集。



### 表结构

1、尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED 。

2、VARCHAR的长度只分配真正需要的空间 。

3、尽量使用TIMESTAMP而非DATETIME 。

4、单表不要有太多字段，建议在20以内。

5、避免使用NULL字段，设置默认值。



### 系统配置

#### 读写分离

只在主服务器上写，只在从服务器上读。对应到数据库集群一般都是一主一从、一主多从。业务服务器把需要写的操作都写到主数据库中，读的操作都去从库查询。主库会同步数据到从库保证数据的一致性。一般 读写分离 的实现方式有两种：代码封装跟数据库中间件。



#### 分库分表

分库分表：分库分表分为垂直和水平两个方式，一般是先垂直后水平。

1、垂直分库：将应用分为若干模块，比如订单模块、用户模块、商品模块、支付模块等等。其实就是微服务的理念。

2、垂直分表：一般将不常用字段跟数据较大的字段做拆分。

3、水平分表：根据场景选择什么字段作分表字段，比如淘宝日订单1000万，用userId作分表字段，数据查询支持到最近6个月的订单，超过6个月的做归档处理，那么6个月的数据量就是18亿，分1024张表，每个表存200W数据，hash(userId)%100找到对应表格。

4、ID生成器：分布式ID 需要跨库全局唯一方便查询存储-检索数据，确保唯一性跟数字递增性。



目前主要流行的分库分表工具 就是`Mycat`和`sharding-sphere`。



### 硬件



# MySQL 生产场景

## 使用场景

### 慢SQL

#### 定位

- **慢查询日志**：开启MySQL的慢查询日志，再通过一些工具比如mysqldumpslow去分析对应的慢查询日志，当然现在一般的云厂商都提供了可视化的平台。
- **服务监控**：可以在业务的基建中加入对慢SQL的监控，常见的方案有字节码插桩、连接池扩展、ORM框架过程，对服务运行中的慢SQL进行监控和告警。



#### 排查

慢SQL排查主要从一下几个角度来进行排查：

- 索引使用情况
- 数据库连接数过小
- 应用侧连接数过小
- buffer pool太小



### 表廋身

MySQL执行`delete`命令其实只是把记录的位置，或者数据页标记为了`可复用`，但磁盘文件的大小是不会变的。通过delete命令是不能回收表空间的。这些可以复用，而没有被使用的空间，看起来就像是`空洞`。插入时候引发分裂同样会产生空洞。



**重建表思路**：

1、新建一个跟A表结构相同的表B 

2、按照主键ID将A数据一行行读取同步到表B 

3、用表B替换表A实现效果上的瘦身。



### 隐藏敏感信息

#### 隐藏姓名

```mysql
SELECT `name` AS `before_encrypt`,
	CASE CHAR_LENGTH(`name`)
	WHEN 2 THEN CONCAT('*',SUBSTRING(`name`, 2, 1))
	WHEN 3 THEN CONCAT(SUBSTRING(`name`, 1, 1),'*',SUBSTRING(`name`, -1, 1))
	ELSE CONCAT(SUBSTRING(`name`, 1, CHAR_LENGTH(`name`) - 2),'**')
	END AS `after_encrypt`
FROM `user`
```



#### 隐藏手机号

```mysql
SELECT `mobile` AS `before_encrypt`, 
INSERT(`mobile`, CHAR_LENGTH(`mobile`) - 7, 4, '****') AS `after_encrypt`
FROM `user`
```



#### 隐藏身份证号

```mysql
SELECT `id_card_no` AS `before_encrypt`, 
INSERT(`id_card_no`, 7, 8, '********') AS `after_encrypt`
FROM `user`
```



## 问题排查

### 慢SQL原因排查

大多数情况下很正常，偶尔很慢，则有如下原因

- 数据库在刷新脏页，例如 redo log 写满了需要同步到磁盘。

- 执行的时候，遇到锁，如表锁、行锁。可以用 **show processlist**这个命令来查看当前的状态。

这条 SQL 语句一直执行的很慢，则有如下原因。

- 没有用上索引：例如该字段没有索引；由于对字段进行运算、函数操作导致无法用索引。

- 数据库选错了索引，可以使用强制指定索引来解决。

