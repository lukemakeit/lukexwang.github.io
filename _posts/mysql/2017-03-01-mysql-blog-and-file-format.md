---
layout: post
title: MySQL blog字段和文件格式
categories: mysql
description: 对于有很多blob字段的表来说,文件格式是必须设置为Barracuda的。
keywords: MySQL
---
## MySQL blog字段和文件格式

年前上线的某业务,因为有许多mediumblob,所有我们在MySQL(5.5)侧开启了参数innodb_file_format=Barracuda;

该参数也支持动态修改,**set global innodb_file_format=Barracuda;**

<font color="ff00ff">那么这个参数的作用是什么呢？</font>

针对这个参数，遇到过下面这个错误信息的同学应该不陌生:

```
Row size too large (> 8126). Changing some columns to TEXT or BLOB or using ROW_FORMAT=DYNAMIC or ROW_FORMAT=COMPRESSED may help. In current row format, BLOB prefix of 768 bytes is stored inline.
```

关于innodb的文件格式也就是innodb\_file\_format,主要有下面两个值:

**Antelope** 和 **Barracuda**

**在MySQL5.5以及以前版本,innodb_file_format默认值是Antelops;从MySQL5.7开始,innodb_file_format的默认值是Barracuda。**

### Antelope

在innodb存储引擎中,记录时以行格式存储的。这也就意味着页中保存了一行行的数据。  

在InnoDB 1.0.x版本之前，InnoDB存储引擎提供了**Compact**和**Redundant**两种行格式(row_format)来存放行记录数据。

- **Redundant**格式是为兼容之前版本而保留的。
- **Compact**是目前使用最多的一种行格式，在MySQL5.6中,默认row_format依然是Compact,可以通过下面语句查看
```
show table status like 'tb_name';
```

**MySQL5.7.9以后,row_format由innodb_default_row_format变量决定,默认值是DYNAMIC。**

Compact行记录是在 MySQL 5.0 中引入的，其设计目标是高效地存储数据。简单来说，一个页中存放的行数据越多，其性能就越高。

Compact行记录格式(row\_format)具体设计可以参考:[MySQL技术内幕:InnoDB存储引擎](https://book.douban.com/subject/24708143/)

当row\_format=Compact时,对于InnoDB来说,**一般认为BLOB、text这类的大对象列类型的存储会把数据存放在数据页面之外,也就是溢出页面中。**

(这里的理解存在一定的偏差，BLOB可以不将数据存储在溢出页面,而即便是varchar列数据类型,也有可能将数据存储在溢出页面。)

因为innodb存储引擎使用的是B+Tree的结构,那么每个数据页至少需要两行(否则失去了B+Tree的意义，变成链表了)。**默认一个数据页是16K,也就是说数据页中每行最多只能8K。如果一行超过8K(8098)，就会报Row size too large (> 8126)错误。**

而**Compact行记录,存储blob，text，varchar(8099) 这样的大字段,innodb只会存放前768字节在数据页中**,剩余的数据则会存储在溢出页。
![mysql_char_set_chart](/images/posts/mysql/mysql_file_format_compact.png)

按着这种算法,如果表中有40个blob字段(上面提到的业务role表确实有40个blob字段,除了blob当然还有其他字段),同时所有字段保存的内容超过8k，就会报Row size too large (> 8126)错误。

对于40个blob字段来说,存储内容超过8K是非常容易的。

### Barracuda

  Barracuda文件格式(file_format)是从innodb plugin引入的。该文件格式拥有两种行格式:**compressed**和**Dynamic,两种格式对blob字段采用完全溢出的方式，数据页中只存放20字节**，其余的都存放在溢出页中:
  ![mysql_char_set_chart](/images/posts/mysql/mysql_file_format_dynamic.png)

  **同时行格式 Compressed 的另一个功能就是对存储的行数据会以zlib算法进行压缩,因此对于 BLOB、TEXT、VARCHAR 这类大长度类型的数据能够进行非常有效的存储。**

关于修改文件格式(file format)为 Barracuda,修改表行格式 row_format为 COMPRESSED。特别需要注意:

**<font color="red">优先修改 innodb_file_format,然后再修改innodb表row_format;</font>**

```
+--------------------------+----------+
| Variable_name            | Value    |
+--------------------------+----------+
| innodb_file_format       | Antelope |
| innodb_file_format_check | ON       |
| innodb_file_format_max   | Antelope |
+--------------------------+----------+
rows in set (0.00 sec)


test> create table test_1 (x int) ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
Query OK, 0 rows affected, 4 warnings (0.07 sec)

test> show warnings;
+---------+------+-----------------------------------------------------------------------+
| Level   | Code | Message                                                               |
+---------+------+-----------------------------------------------------------------------+
| Warning | 1478 | InnoDB: KEY_BLOCK_SIZE requires innodb_file_format > Antelope.        |
| Warning | 1478 | InnoDB: ignoring KEY_BLOCK_SIZE=8.                                    |
| Warning | 1478 | InnoDB: ROW_FORMAT=COMPRESSED requires innodb_file_format > Antelope. |
| Warning | 1478 | InnoDB: assuming ROW_FORMAT=COMPACT.                                  |
+---------+------+-----------------------------------------------------------------------+
rows in set (0.00 sec)

show create table test_1;
test_1 | CREATE TABLE `test_1` (
  `x` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8

show table status like 'test_1'\G;
*************************** 1. row ***************************
           Name: test_1
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2013-09-27 15:59:13
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: row_format=COMPRESSED KEY_BLOCK_SIZE=8
        Comment:
row in set (0.00 sec)
```

**参考文章**

[MySQL大字段溢出导致数据回写失败](http://blog.opskumu.com/mysql-blob.html)

[Change limit for “Mysql Row size too large](http://stackoverflow.com/questions/15585602/change-limit-for-mysql-row-size-too-large)
