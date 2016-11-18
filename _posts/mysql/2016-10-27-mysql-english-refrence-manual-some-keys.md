---
layout: post
title: MySQL英文参考手册some keys
categories: mysql
description: MySQL英文参考手册一些平时没注意的关键点小计
keywords: MySQL
---

### 参考手册信息

MySQL参考手册版本：refman-5.6-en.a4.pdf

下载地址：[http://dev.mysql.com/doc/index-archive.html](http://dev.mysql.com/doc/index-archive.html)

### some keys

1. Options on the command line take precedence over values specified in option files and environmentvariables, and values in option files take precedence over values in environment variables. [2.12]

2. On character type columns, sorting—like all other comparison operations—is normally performed in a case-insensitive fashion.This means that the order is undefined for columns that are identical except for their case. You can force a case-sensitive sort for a column by using BINARY like so: ORDER BY BINARY col_name. [3.3.4.4]
简译：对于字符型列来说，排序是不区分大小写的。如果想让字符型列排序区
分大小写可以使用BINARY，如：ORDER BY BINARY col\_name

3. TIMESTAMPDIFF(YEAR,birth,CURDATE()) as age =》计算年龄

4. MONTH(birth) = **MOD**(MONTH(CURDATE()), 12) + 1;=》选择得到生日在下个月。**MOD()进行取模运算**[3.3.4.6]

5. You cannot use arithmetic comparison operators such as =, <, or <> to test for NULL. [3.3.4.6] =>计算符号对NULL是没有任何意义的

6. In MySQL, 0 or NULL means false and anything else means true. The default truth value from a boolean operation is 1. =》在MySQL中，0或者NULL都是false的意思，其他的则为true。从bool运算中得到true值是用1表示

7. Two NULL values are regarded as equal in a GROUP BY.

8. When doing an ORDER BY, NULL values are presented first if you do ORDER BY ... ASC and last if you do ORDER BY ... DESC.

9. A common error when working with NULL is to assume that it is not possible to insert a zero or an empty string into a column defined as NOT NULL, but this is not the case. =》使用NULL一个常见的错误就是以为对声明为NOT NULL的列不能插入0或者空字符串，其实是可以的。

10. SQL pattern matching enables you to use “_” to match any single character and “%” to match an arbitrary number of characters (including zero characters).=>SQL正则表达式允许使用"\_"来匹配任意单个字符,使用“%”来匹配任意个数字符，包括0个。[3.3.4.7]

11. In MySQL, SQL patterns are caseinsensitive by default.=》MySQL中正则表达式对大小写并不敏感。  

12. SELECT * FROM pet WHERE name REGEXP '^b';

13. If you really want to force a REGEXP comparison to be case sensitive, use the BINARY keyword to make one of the strings a binary string.**SELECT * FROM pet WHERE name REGEXP BINARY '^b';**

14. If you want to get the interactive output format in batch mode, use mysql -t. To echo to the output the statements that are executed, use mysql -v. =》如果想在批处理模式也就是source file这种模式下的输出和登录MySQL的输出一样是表状的，那么就是用mysql -t选项。如果你想同时在输出时同时打印执行的sql，使用-v选项。

15. MySQL does not perform any sort of CHECK to make sure that col\_name actually exists in tbl\_name (or even that tbl_name itself exists).[3.6.6]=>MySQL的references tbl\_name(col_name)语法不会做任何col\_name是否存在的检查，即使tal\_name是存在的。**这里只是说references这个语法，并不是说创建外键时不检查目标表和目标列的存在.**  

16. When you insert any other value into an AUTO_INCREMENT column, the column is set to that value and the sequence is reset so that the next automatically generated value follows sequentially from the largest column value.

17. For a multiple-row insert, LAST_INSERT_ID() and mysql_insert_id() actually return the AUTO_INCREMENT key from the first of the inserted rows.

18. --column-names option that determines whether or not to display a row of columnnames at the beginning of query results. [4.2.5 ] =>这样子来说的话，--column-names这个参数在用来执行select一个表数据时还是蛮好用的。因为默认是会把列名一起导出来的

```SQL
#enalbled
--disable-column-names
--skip-column-names
--column-names=0
#disabled
--column-names
--enable-column-names
--column-names=1
```

19. A MySQL program started with the **--no-defaults** option reads no optionfiles other than .mylogin.cnf [4.2.6 ]

20. ```!include``` directives in option files to include other option files and ```!includedir``` to search specific directories for option files. **Any files to be found and included using the ```!includedir``` directive on Unixoperating systems must have file names ending in .cnf.**

```SQL
!include /home/mydir/myopt.cnf
!includedir /home/mydir
```

21. If you execute mysqld_safe with the **--defaults-file** or **--defaults-extra-file** option to name an option file, the option must be the first one given on the command line or the option file will not be used. **For example, this command will not use the named option file[4.3.2]:**

```shell
mysql> mysqld_safe --port=port_num --defaults-file=file_name
```

Instead, use the following command:

```shell
mysql> mysqld_safe --defaults-file=file_name --port=port_num
```

22. ```mysql_install_db``` **initializes the MySQL data directory and creates the system tables that it contains, if they do not exist.** It also initializes the system tablespace and related data structures needed to manage InnoDB tables.
Because the MySQL server, mysqld, must access the data directory when it runs later, you should **either run mysql_install_db from the same system account that will be used for running mysqld, or run it as root and specify the --user option to indicate the user name that mysqld will run as.**

23. 设置密码的几种方法
```shell
shell> mysql -u root
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_password');

shell> mysql -u root
mysql> UPDATE mysql.user SET Password = PASSWORD('new_password') WHERE User = 'root' and Host='localhost';
mysql> FLUSH PRIVILEGES;

shell> mysqladmin -u root -h localhost password "new_password"
```
