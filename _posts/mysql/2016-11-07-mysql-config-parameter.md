---
layout: post
title: MySQL配置文件先关参数
categories: mysql
description: 只有了解了MySQL相关参数的含义，才能更好的维护MySQL
keywords: MySQL
---

## MySQL配置文件相关参数
**[mysqld]**

**bind-address=10.217.x.x**

**port=20000**

**socket=/data/mysqldata/20000/mysql.sock**

**server\_id=2217xxxxx**

**default-storage-engine=innodb**

**alter\_query\_log=ON**

**datadir=/data/mysqldata/20000/data/**

**character\_set\_server=utf8**

**init\_connect**="set @user=user(),@cur\_user=current\_user();insert into test.conn\_log values(connection\_id(),now(),@user,@cur\_user,'10.217.x.x')"

**tmpdir**=/data/mysqldata/20000/tmp/

**log\_warnings=0**

设定是否将警告信息记录进错误日志。默认设定为1，表示启用；可以将其设置为0以禁用；而其值为大于1的数值时表示将新发起连接时产生的“失败的连接”和“拒绝访问”类的错误信息也记录进错误日志。

**innodb\_io\_capacity=1000**

设置innodb后台进程I/O活动的上限，如从buffer pool中刷新页和从change buffer中合并脏数据。innodb\_io\_capacity的大小应该设置得和每秒进行的I/O操作次数相当。理想情况下是保持低实用，但不至于低到后台活动落后。如果值很高，则数据会被很快地从buffer pool中移除，数据插入buffer也会很快。
```
默认值:200;可使用super权限动态调整，如：set global innodb_io_capacity = 2000;
```
**innodb\_read\_io\_threads=8**

Innodb使用后天线程处理数据页上读I/O请求的数量

**innodb\_write\_io\_threads=8**

Innodb使用后天线程处理数据页上写I/O请求的数量

**innodb\_file\_io\_threads**=4 已经被上面两个参数取代

**innodb\_file\_per\_table=1**

 独立表空间:  每一个表都将会生成以独立的文件方式来进行存储，每一个表都有一个.frm表描述文件，还有一个.ibd文件。 其中这个文件包括了单独一个表的数据内容以及索引内容.

**innodb\_additional\_mem\_pool\_size=20M**

innodb用于存储数据目录信息和其他内部数据结构的内存池大小。在应用程序中的表越多，在这里所分配的内存就越多。**默认为8M，从5.6.3版本后弃用。**

**innodb\_buffer\_pool\_instances=4**

innodb\_buffer\_pool缓冲池复制管理着free list（初始化空闲页，为每一个page指定一个block头结构，并初始化各种mutex与rw-lock，将page加入Buffer_ Pool的free list链表，等待分配）、flush list（缓冲池产生的脏页（数据库被修改，但未写入磁盘），当**innodb\_max\_dirty\_pages\_pct**超过设置的值时，会把修改时间较早的page刷进磁盘）、 LRU（在内存中但最近又不用的数据块，按照最近最少使用算法，MySQL会根据哪些数据属于LRU而将其移出内存，从而腾出空间来加载另外的数据）等，当InnoDB_Buffer_Pool缓冲池达到好几十GB时，如果某个线程正在更新缓冲池，将会造成其他线程必须等待的瓶颈。

**在MySQL5.5里，可以通过innodb\_buffer\_pool\_instances参数来增加InnoDB\_Buffer\_Pool实例的个数，**并使用哈希函数将读取缓存的数据页随机分配到一个缓冲池里面，这样每个缓冲区实例就可以分别管理着自己的free list、flush list和LRU，也就可以解决上面的问题了。

可以将在my.cnf.20000中将innodb\_buffer\_pool\_instances设置为4，restart MySQL，show engine innodb status\G即可看到三个相应的free list、flush list了。[http://book.2cto.com/201402/40305.html](http://book.2cto.com/201402/40305.html)

**innodb\_thread\_concurrency=8**

**innodb\_thread\_concurrency定义的是进入到innodb层的并发线程数。**

如果innodb\_thread\_concurrency=0,则表示并发线程数不受限制。
innodb\_thread\_concurrency>0,则表示检查机制开启，如已经超过innodb_thread_concurrency设置的限制，则该请求线程会等待innodb_thread_sleep_delay微秒后尝试重新请求，如果第二次请求还是无法获得，则该线程会进入FIFO队列休眠。重试两次的机制是为了减少CPU的上下文切换的次数，以降低CPU消耗。

如果请求被Innodb接受，则会获得一个次数为innodb_concurrency_tickets(默认500次)的通行证，在次数用完之前，该线程重新请求时无须再进行前面的检查。
[http://www.gpfeng.com/?p=488](http://www.gpfeng.com/?p=488)

**thread\_concurrency=8** 这个参数5.6不用了，而且只对Solaris系统有效。

**innodb\_data\_file\_path**=ibdata1:1G:autoextend

表空间文件路径,设置方法如：```innodb_data_file_path=/db/ibdata1:2000M;/dr2/db/ibdata2:2000M:autoextend```

**innodb\_data\_home\_dir**=/data/mysqldata/20000/innodb/data和上面一个参数做组合，设置mysql表空间文件所在位置。

**innodb\_flush\_log\_at\_trx\_commit=0**

 [官网解释](http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit)
- innodb\_flush\_log\_at\_trx\_commit的默认值是1，设置为1时,每一次事务提交都会将InnoDB log buffer中的内容写入到log file并刷新到磁盘。<font color="#ff00ff">我觉得应该是InnoDB log buffer中的全部内容，而不单指提交了的这个事务的日志。</font>所以刷新时当然也包含了许多未提交事务产生的日志。
- innodb\_flush\_log\_at\_trx\_commit=0时,InnoDB log buffer大约每一秒写入到InnoDB log file并刷新到磁盘。
- innodb\_flush\_log\_at\_trx\_commit=2时,每一次事务提交都将InnoDB log buffer中的内容写入到磁盘，并等着系统自动更新到磁盘(大约每秒钟一次)。
![innodb_flush_log_at_trx_commit](/images/posts/mysql/innodb_flush_log_at_trx_commit.jpg)

**innodb\_log\_buffer\_size=32M** 重做日志缓冲区大小

**innodb\_log\_file\_size=256M** 设置每个重做日志文件的大小

**innodb\_log\_files\_in\_group=4** 设置重做日志组中有多少个重做日志文件

**innodb\_log\_group\_home\_dir**=/data/mysqldata/20000/log/ 指定重做日志文件路径

**skip-name-resolve**

禁用dns解析，避免网络DNS解析服务引发访问MYSQL的错误，一般应当启用。启用"skip-name-resolve"后，在MySQL的授权表中就不能使用主机名了，只能使用IP。

**skip-external-locking**

 看看[官网解释](http://dev.mysql.com/doc/refman/5.7/en/external-locking.html)吧!相关人士[博客](http://www.kuqin.com/database/20120815/328905.html)。

**skip_slave_start** mysql服务启动后跳过自动启动复制


**log\_bin**=/data/mysqllog/20000/binlog/binlog20000.bin 设置binlog日志的位置以及其相关名称

**log\_bin\_trust\_function\_creators**=1

log_bin_trust_function_creators参数缺省0，是不允许function的同步的，一般我们在配置repliaction的时候，都忘记关注这个参数，这样在master更新funtion后，slave就会报告错误。所以这里需要打开。

**log\_slave\_updates**=1

该参数就是为了让从库从主库复制数据时可以写入到binlog日志。那么为什么不是从库开启了log_bin就可以了呢？
答：从库开启log-bin参数，如果直接往从库写数据，是可以记录log-bin日志的，但是从库通过I0线程读取主库二进制日志文件，然后通过SQL线程写入的数据，是不会记录binlog日志的。也就是说从库从主库上复制的数据，是不写入从库的binlog日志的。所以从库做为其他从库的主库时需要在配置文件中添加log-slave-updates参数。


**max\_binlog\_size**=256M 设置binlog文件的大小。

**relay-log**=/data/mysqldata/20000/relay-log/relay-log.bin 设置relay-log的文件名称和位置

**relay_log_recovery=1**  
该参数主要是在slave服务器启动后,会根据上一次SQL thread处理到relay log的位置重新初始化一个relay log,并初始化IO thread和SQL Thread到同样的位置。再从master上拉取日志。

换一句说法就是：relay_log_recovery=1,当slave宕机后,如果relay-log损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的relay-log,并从最后执行的relay log位置重新从master上获取日志,该参数默认关闭。同时该参数不能动态改变。

对应错误:

```
161128 16:40:28 [ERROR] Slave SQL: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave. Error_code: 1594
161128 16:40:28 [ERROR] Error running query, slave SQL thread aborted. Fix the problem, and restart the slave SQL thread with "SLAVE START". We stopped at log 'binlog.134374' position 198324449
```

**slow\_query\_log\_file**=/data/mysqllog/20000/slow-query.log 慢查询日志位置和名称

**long\_query\_time**=1 设置慢查询时间，如果sql语句执行超过这个时间则记录慢查询日志

**log\_queries\_not\_using\_indexes**=on 没有使用索引的查询语句会被记录到慢查询日志中

**query_response_time_stats**=ON

参数query_response_time_stats用于在服务器端观察数据库的响应时间。set global query_response_time_stats = 'ON';后一段时间执行:select * from QUERY_RESPONSE_TIME; 看看

**max\_connections**=3000 设置mysql的最大连接数

**max\_connect\_errors**=9999999 设置mysql连接出错的最大重复次数

**myisam\_sort\_buffer\_size**=64M

myisam表在修复表或者使用create index 、alter table创建索引时，排序表索引所分配的缓存大小。

**innodb\_buffer\_pool\_size**=1280M

innodb_buffer_pool_size参数表示缓冲池字节大小，InnoDB缓存表和索引数据的内存区域。mysql默认的值是128M。

**query\_cache\_size**=0

通过对 Query 语句进行特定的 Hash 计算之后与结果集对应存放在 Query Cache 中，以提高完全相同的 Query 语句的相应速度。当我们打开 MySQL 的 Query Cache 之后，MySQL 接收到每一个 SELECT 类型的 Query 之后都会首先通过固定的 Hash 算法得到该 Query 的 Hash 值，然后到 Query Cache 中查找是否有对应的 Query Cache。如果有，则直接将 Cache 的结果集返回给客户端。如果没有，再进行后续操作，得到对应的结果集之后将该结果集缓存到 Query Cache 中，再返回给客户端。

<font color="red">当数据表的数据发生任何更改之后，与该表相关的所有 Query Cache 全部会失效，所以 Query Cache对变更比较频繁的表并不是非常适用。这里“数据表更改”包括: INSERT, UPDATE,DELETE, TRUNCATE, ALTER TABLE, DROP TABLE, or DROP DATABASE等。</font>

在select语句中禁用query cache的方法是：select SQL_NO_CACHE ...

**query\_cache\_type**=0

关闭query cache的方法是将:query_cache_size=0,query_cache_type=0；

**read\_buffer\_size**=2M

每个线程对MyISAM表的顺序扫描都会对每个扫描的表分配read\_buffer\_size大小(单位为byte)的缓存。如果你会做许多顺序扫描，你可以增加这个值的大小，该值的默认大小是128KB。
read\_buffer\_size在下列情况中会用于所有存储引擎的上下文:
- 使用order by进行排序时，cache临时文件(不是临时表)的索引;
- 做数据批量导入分区;
- cache嵌套查询结果;

**table\_cache**=5120

table\_cache后续改名字为table\_open\_cache,该参数限制所有线程打开表的数量,或者说所有线程缓存表的数量。如果当前已经缓存的表未达到table_cache，则会将新表添加进来；若已经达到此值，MySQL将根据缓存表的最后查询时间、查询率等规则释放之前的缓存。

可以通过```show global status like 'open%_tables'```查看mysql中open\_tables、opened\_tables的值。open_tables是当前正在打开表的数量，opened_tables是所有已经打开表的数量。如果opened_tables值很大，且你不经常用```flush tables```。那么就应该增加table\_cache参数的大小。

**thread\_cache**=8

MySQL为了提高客户端请求性能，提供了一个线程池，即Thread_Cache池，将空闲的线程放在线程池中，而不是立即销毁。好处：当新的请求来到时，mysql不会立即去创建新的线程，而是先去Thread_Cache中去查找空闲的线程，如果存在则直接使用，不存在才创建新的线程。

参数:thread_cache_size 表示mysql server能够cache并重复使用的线程数量(可以说成是连接池大小);如果server有大量的新连接，增加这个变量的值能显著提高性能。

``` sql
show global status like 'thread%';
Threads_cached 57 /*线程池中线程数*/
Threads_connected 1268 /*当前已打开的链接数，等于show processlist*/
Threads_created 31715
Threads_running 1 /*因为刚刚执行了show global status语句*/
```

**key\_buffer**=64M

MyISAM表的索引块需要被缓存和被所有线程共享。key\_buffer\_size就是该缓存的大小。

**replicate-ignore-db**=mysql

**replicate-wild-ignore-table**=mysql.%

**slave\_compressed\_protocol**=1 该参数设置为1时，主从间的通讯在两者都支持的情况下会以压缩方式进行。

**interactive\_timeout**=86400

服务器关闭**交互式连接**前等待活动的秒数。

**wait\_timeout**=86400

服务器关闭**非交互连接**之前等待活动的秒数。

客户端是交互式还是非交互式主要取决于mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义。interactive\_timeout和wait\_timeout一般设置为相等。且需要同时设置。

**[mysql]**

default-character-set=utf8

no-auto-rehash

port=20000

socket=/data/mysqldata/20000/mysql.sock
