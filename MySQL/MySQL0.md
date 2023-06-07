[toc]

# 数据库设计范式、
如何设计合理的表结构、


# 01 存储引擎（P0）
## InnoDB和MyISAM的区别？



11、如果从查询等层面，MyIsam和InnoDB的区别
4、复习数据库不同引擎的各种区别(原理、构造、查询、索引、优化等等)



## 事务隔离级别（P0）
ACID：原子性、一致性、隔离性、
* 读未提交 Read Uncommited
* 读已经提交
* 可重复读
* Searializable：顺序读？

https://blog.csdn.net/zl1zl2zl3/article/details/86702635

如何处理MySQL事务隔离级别错误
使用
数据库事务的隔离级别包含哪些？

事物隔离级别
无锁变成wcs
mysql for update

## InnoDB锁的类型
锁是和事物隔离级别一起的.

https://blog.csdn.net/zl1zl2zl3/article/details/86702635
https://www.jianshu.com/p/af2f5fed3b6d
https://segmentfault.com/a/1190000014133576

1. 意向锁、
2. 共享和排他锁
5. 意向插入锁
6. 自增锁


* 记录锁: 锁住单个索引记录,防止其他事物插入、更新或者删除索引
* 区间锁(间隙锁):锁住两个key之间的记录,防止其他事物插入数据,引起幻读问题
* 邻键锁Next-Key Locks

# 03 索引和查询优化（P0）
聚簇索引和非剧簇索引、

join、 left join、right join、
https://www.guru99.com/joins.html


# 04 高可用与稳定性（P1）
集群、MGR和HAProxy\
介绍性文章： https://tmcdcgeek.club/2019/05/24/mgr_haproxy/

# 06 安全和SQL注入
严格验证数据
* 类型、长度、字符集、模式、取值范围
* 例如：qq号，email地址，时间日期
对数据进行转义
* 拼接SQL语句前转义输入中的特殊字符
验证与转义的时机
屏蔽程序错误信息


用PDO吧，prepared statment才是王道
PDO (PHP Data Objects)提供一个通用接口访问多种数据库，即抽象的数据模型支持连接多种数据库。


* addslashes() 是强行加\；
* mysql_real_escape_string() 会判断字符集，但是对PHP版本有要求；
* mysql_escape_string不考虑连接的当前字符集。

dz中的防止sql注入就是用addslashes这个函数，同时在dthmlspecialchars这个函数中有进行一些替换$string = preg_replace('/&((#(\d{3,5}|x[a-fA-F0-9]{4}));)/', '&\\1',这个替换解决了注入的问题，同时也解决了中文乱码的一些问题



# 08 存储过程和函数（P2）
见具体文档、

# 09 字符集和编码

# 10 安装实践（P3）
见分文档、

# binlog
- [ ] binlog三种模式


# 业界学习和最新进展
MySQL索引原理及慢查询优化 - 美团点评技术团队
http://tech.meituan.com/mysql-index.html?from=timeline&isappinstalled=0
http://tech.meituan.com/mysql-index.html?from=timeline&isappinstalled=0

Mysql数据库HA、事务一致性保证
https://www.cnblogs.com/zejin2008/p/7705473.html

## 12 TiDB
https://pingcap.com/docs-cn/
参考分享：https://docs.google.com/presentation/d/1Os5uQcCNEwsTIXVEPx8uIw8d63WAnH9QYqu5Urr4gaI/edit#slide=id.g5d32adf1da_2_50

# 性能调优

一、发现系统的瓶颈在哪里？系统在做什么？
1、磁盘搜索 现代磁盘的平均时间通常小于10ms

mysql客户端程序用BENCHMARK()函数执行定时测试。
语法：BENCHMARK(loop_count,expression)。

用Analyze Table更新表的统计

二、估算查询性能
大多数情况下，小表可以使用1次搜索找到行，


数据库部分：
1、数据备份的方式？
2、SQLSERVER的默认端口是？
3、为何使用索引查询速度加快？




1. 查看分区表上索引使用情况:
explain  partitions  SELECT *....


--------------------------
Show  table  status like 'usr'  \G;

Checktable mytable
Repair  table mytable

查看存储过程
show procedure status;
drop procedure load_part_tab;


函数实现：
show function status like ’fun_s’;
Show processlist; 显示当前运行的进程


数据库操作经验
目录/usr/local/mysql/bin/
mysqladmin -shutdown 关闭数据库
mysqld_safe --user=root & 启动数据库
数据库log位置
/data/log/

 

1、
存储IP地址用unsigned int
INET_ATON();
INET_NTOA();


2、垂直分表：
a、将常用的数据和不常用的数据分开存储（如个人详细信息和基本信息）
b、将不变的数据和经常修改的数据、统计数据分开存储（如商品描述和统计表）
而且这里可能会影响cache效果。 不需要cache的可以加hint sql_no_cache不写cache提高性能

c、日记表，将长正文数据和基本数据分开

水平分表：

3、query_cache严格按select的字符串hash计算
写多的应用不要使用query_cache

 


4、理解各存储引擎中索引记录的组织形式
InnoDB的索引类型是：
cluster类型索引（主键索引和数据存放在一起）
第二索引需要先映射为主键值，然后才能找到数据，所以尽量使用主键索引

主键字段越小越好，所有的二级索引都会存储主键索引！


5. coding原则：
1、不要考虑迭代，例如（for loop,while loop这类的实现）
2、thingking in terms of set
3、把复杂的sql语句或者业务逻辑拆分成小的、可管理的模块
4、把几个语句合并在一个事务中提交，但要控制事务的大小

样例：查询每个客户的最后的支付信息
为啥第二个方式会快那么多？？Why？


二、索引列上的函数操作：
日期查询，
注意：日期函数是个不确定的值，不能使用query cache！
例子：
select * from order where to_days(current_date) - to_days(order_created) <=7;
-->
select * from order where order_created >= current_date() - interval 7 day;
-->
select * from orders where order_created >= '2008-01-11' - interval 7 day;
-->
select order_id,customer_id,order_total,order_created
from orders where order_created >= '2008-01-11' - interval 7 day;


三、用索引避免全表查询，
1、对于字符串需要%xx 查找的时候，使用逆转字段值，  顺便创建一个两个触发器

 

2.取sql的sum数量；
qb1=$(/usr/local/mysql/bin/mysql dbX5FileUpload -sBe "select count(*) as sum from tbDemo where iSubType=25");


MYSQL日期时间格式转换
SELECT UNIX_TIMESTAMP() ; 


mysql>SELECT FROM_UNIXTIME( 1249488000, '%Y%m%d' )  
->20071120

mysql> SELECT UNIX_TIMESTAMP('2009-08-06') ;
->1249488000

 

Mysql显示运行中的进程:

 

mysql show processlist命令 详解 

 


1.TRUNCATE 效率比delete高,但是对于数据量大的表依然不能适用,此时清空表的最佳办法是
create table tmp1 like tmp2;
drop table tmp1;
rename table tmp2 to tmp1;


2，
CREATE TABLE test (
  id VARCHAR(20) NOT NULL,
  name VARCHAR(20) NOT NULL,
  submit_time DATETIME NOT NULL,
  index time_index (submit_time),
  index id_index (id)
)ENGINE=MyISAM
PARTITION BY RANGE COLUMNS(submit_time)
(
PARTITION p1 VALUES LESS THAN ('2010-02-01'),
PARTITION p2 VALUES LESS THAN ('2010-03-01'),
PARTITION p3 VALUES LESS THAN ('2010-04-01'),
PARTITION p4 VALUES LESS THAN ('2010-05-01'),
PARTITION p5 VALUES LESS THAN ('2010-06-01'),
PARTITION p6 VALUES LESS THAN ('2010-07-01'),
PARTITION p7 VALUES LESS THAN ('2010-08-01'),
PARTITION p8 VALUES LESS THAN ('2010-09-01'),
PARTITION p9 VALUES LESS THAN ('2010-10-01'),
PARTITION p10 VALUES LESS THAN ('2010-11-01'),
PARTITION p11 VALUES LESS THAN ('2010-12-01') 
);
------------
delimiter // 
CREATE PROCEDURE mark_test()
begin
    declare v int default 0;
    while v < 8000
    do
       insert into test values (v,'testing partitions',adddate('2010-01-01', INTERVAL v hour));
       set v = v + 1;
    end while;
end //
delimiter ;

测试用例，写mysql存储过程。


# 分区
1. 查看表的分区信息
SHOW CREATE TABLE tr\G

 EXPLAIN PARTITIONS SELECT 

查看select语句怎样使用分区 


高性能WEB服务器
explain extended  查看Mysql

## 查看mysql状态：
```
1、show status
2、show innodb status
3、mysql report 工具
4、show processlist 显示系统当前运行的线程状态
5、mysqldumpslow\mysqlsla 慢查询分析工具
```

一定要根据查询的需要来设计有针对性的索引。量身定做的索引比全表扫描来的快捷。

使用慢查询分析工具来分析慢查询。
my.cnf配置文件中添加：
long_query_time = 1
long-slow-queries = /data/var/mysql_slow.log
long_queries-not-using-indexes =1

Innodb查用预写日志的方式来实现事物，也就是说只有事务日志写入磁盘后才更新数据和索引。



什么是独立表空间\什么是共享表空间
2013年12月11日
17:54
drop 大量数据的表?

分区表过多是buffer pool导致的？
mingoxu(徐晓明) 12-11 17:57:26
分区表多 删除慢 是buffer pool导致的
这边正在优化


Mysql查询缓存不会存储有不确定结果的查询，如不确定函数、用户自定义函数、存储函数、用户自定义变量、临时表等。
如NOW(),CURRENT_DATE(),CONNECTION_ID(),CURRENT_USER()

从缓存中受益最多的是需要很多资源来产生结果，但是不需要很多空间来保存的类型。如聚集查询，count
检查缓存中受益的最简单的办法是检查缓存命中率。 


掌握核心技术很难的说