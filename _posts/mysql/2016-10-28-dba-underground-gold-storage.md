---
layout: post
title: DBA地下库
categories: mysql
description: DBA的一些小心得，来自于别人的总结
keywords: MySQL
---
## DBA地下库

### 紧急情况下DR切DB
紧急情况下DB故障，如磁盘只读、硬件故障。需要将DR切换成DB，需要检查的项有下面这些:
- 如果DB还能启动，则DB上show master status、DR上show slave status，查看DB的binlog file、binlog pos与DR的Relay\_Masster\_Log\_File、Exec\_Master\_Log\_Pos是否一致;
- 如果DB不能启动了则DR show slave status查看**Master\_Log\_File**和**Read\_Mater\_Log\_Pos**,**Relay\_Master\_Log\_File**和**Exec\_Master\_Log\_Pos**是否一致；
- 检查DB和DR每天的数据检校(表db\_infobase.checksum)，确认是否有异常；(依赖于每天的数据检校)
- 将DB的权限克隆到DR上；(依赖每天DB权限的保存上报到其他地方);
- 如果以上的步骤都没问题则可以将DB切换到DR。

### MySQL character set相关
参考文章:[http://www.jb51.net/article/30864.htm](http://www.jb51.net/article/30864.htm)

**latin1**：对ASCII的拉丁语扩展，向下兼容ASCII,其编码范围0x00-0xFF,0x00-0x7F之间完全和ASCII一致，0x80-0x9F之间是控制字符,0xA0-0xFF是文字字符。

**Unicode码**:兼容了所有语言编码，有两个字节或者四个字节表示。utf8是Unicode表的一种落地实现。Unicode转换为utf8后，范围是1——6字节。

**MySQL字符集**

- **character\_set\_client**:客户端来源字符集，客户端告诉服务器所发送请求的字符集编码;
- **character\_set\_connection**:连接层字符集，MySQL使用character\_set\_connection字符集，将客户端的请求转换为character\_set\_connection所表示的字符集;
- **character\_set\_results**:查询字符集，mysql将数据存储的字符集转换为character\_set\_results返回给前端;
- **character\_set\_database**:当前选中数据库的默认字符集;
- **character\_set\_server**:默认的内部操作字符集(**基本可以忽略**)。创建一个数据库时，除非明确指定，这个数据库的字符集缺省会被设定为character\_set\_server;
- **character\_set\_system**:系统元数据(字段名等)存储时候使用的字符集(**可以忽略**)。

create table在没有明确指定字符集的时候，mysql使用的默认字符集：**字段字符集->表字符集->数据库字符集->服务器字符集**

**mysql字符集转换过程**

1. mysql server收到请求时将请求数据从character\_set\_client转为character\_set\_connection;
2. 进行内部操作前将请求数据从character\_set\_connection转换为内部操作字符集，mysql所使用的方法如下:
    - 使用每个数据**字段**的CHARACTER SET设定值;
    - 若上述值不存在，则使用对应**数据表**的DEFAULT CHARACTER SET设定值;
    - 若上述值不存在，则使用对应**数据库**的DEFAULT CHARACTER SET设定值;
    - 若上述值不存在，则使用**character\_set\_server**设定值;
3. 将操作结果从内部操作字符集转换为**character\_set\_results**:
![mysql_char_set_chart](/images/posts//mysql/mysql_char_set_chart.jpg)

**本地连接mysql指定--default-character-set=utf8或者php访问页面前面加上mysql_query("SET NAMES utf8");均可使:character\_set\_client、character\_set\_connection和character\_set\_results变为utf8。**

**常见问题解析**

- **情况一**:表字符集指定为utf8,character\_set\_client、character\_set\_connection和character\_set\_results均为latin1。出错情况具体参考:[文章](http://www.jb51.net/article/30864.htm)
- **情况二**:表字符集指定为latin1,character\_set\_client、character\_set\_connection和character\_set\_results均为utf8。出错情况具体参考:[文章](http://www.jb51.net/article/30864.htm)
