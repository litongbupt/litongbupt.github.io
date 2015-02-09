---
layout: post
title: "列式数据库infobright"
description: ""
category: 
tags: []
---
{% include JB/setup %}

##1.简介

infobright是一个基于MySQL的数据仓库系统，内部是没有索引，采用的Knowledge Grid来组织数据。基本特征如下：

- 查询性能高：百万、千万、亿级记录数条件下，同等的SELECT查询语句，速度比MyISAM、InnoDB等普通的MySQL存储引擎快5～60倍 
- 存储数据量大：TB级数据大小，几十亿条记录 
- 高压缩比：理论上是40：1，在我们的项目中为10:1，极大地节省了存储空间 
- 基于列存储：无需要物化视图、复杂的数据分区策略、索引 
- 适合复杂的分析性SQL查询：SUM, COUNT, AVG, GROUP BY
- 没有特殊的数据仓库摸（比如星形模型、雪花模型）要求
- 和众多的BI套件相容，比如Pentaho、Cognos、Jaspersoft。
- 随着数据库的逐渐增大，查询和装载性能基本保持稳定,实施和管理简单，需要极少的管理


说infobright之前，先回顾下mysql:


###1.1 Mysql逻辑视图

![infobirght1](http://xiangguo.qiniudn.com/img/posts/infobright/infobright1.jpg "infobright1")

1.连接管理与安全性
每个客户端一个线程也可以用第三方的线程池；认证可以使用安全套接字或者X.509
2.优化执行
包括重写查询、决定表的读取顺序，以及选择合适的索引等


###1.2 Mysql的存储引擎

####基于MVCC的InnoDB引擎
MVCC是行锁的一种变种，在很多情况下避免加锁操作，降低开销，例如非阻塞的读操作，以及写操作只锁定必要的行。    
MVCC的实现是通过保存数据在某个时间点的快照来实现。    
5.5以后的版本是默认引擎。

####MyISAM引擎
5.1以及以前版本的默认引擎
缺点：不支持事务；没有行级锁；崩溃后无法恢复
优点：可全文索引，压缩表，空间函数

####CSV引擎
优点：可以将普通的CSV文件作为MySQL的表来处理
缺点：不支持索引

####brighthoust
即infobright


##1.3 ICE和IEE版本的比较    

infobright有两个版本ICE和IEE，支持64位和32位的Linux和windows。

![infobirght2](http://xiangguo.qiniudn.com/img/posts/infobright/infobright2.png "infobright2")   

最主要的：ICE不支持insert、update等操作。


##2.Infobright的总体构架图如下:

![infobirght3](http://xiangguo.qiniudn.com/img/posts/infobright/infobright3.png  "infobright3")   

如上图所示，Infobright采用了和MySQL一致的构架，分为两层。上层是服务及应用管理，下层是存储引擎。Infobright的默认存储引擎是brighthouse，但是Infobright还可以支持其他的存储引擎，比如MyISAM、MRG_MyISAM、Memory、CSV。Infobright通过三层来组织数据，分别是DP(Data Pack)、DPN（Data Pack Node）、KN（Knowledge Node）。而在这三层之上就是无比强大的知识网络（Knowledge Grid）。

　　数据块（DP）是存储的最低层，列中每64K个单元组成一个DP。DP比列更小，具有更好的压缩比率；又比单个数据单元更大，具有更好的查询性能。

　　数据块节点（DPN），DPN和DP之间是一对一的关系。DPN记录着每一个DP里面存储和压缩的一些统计数据，包括最大值、最小值、null的个数、单元总数count、sum等等。

　　KN里面存储着指向DP之间或者列之间关系的一些元数据集合，比如值发生的范围（MIin_Max）、列数据之间的关联。大部分的KN数据是装载数据的时候产生的，另外一些事是查询的时候产生。

　　在这三层之上是知识网络（Knowledge Grid），Knowledge Grid构架是Infobright高性能的重要原因。
　　
![infobirght4](http://xiangguo.qiniudn.com/img/posts/infobright/infobright4.jpg "infobright4")   　　

Knowledge Grid可分为四部分，DPN、Histogram、CMAP、P-2-P。

　　DPN如上所述。Histogram用来提高数字类型（比如date，time，decimal）的查询的性能。Histogram是装载数据的时候就产生的。DPN中有mix、max，Histogram中把Min-Max分成1024段，如果Mix_Max范围小于1024的话，每一段就是就是一个单独的值。这个时候KN就是一个数值是否在当前段的二进制表示。

![infobirght5](http://xiangguo.qiniudn.com/img/posts/infobright/infobright5.jpg "infobright5")   　　

　　Histogram的作用就是快速判断当前DP是否满足查询条件。如上图所示，比如select id from customerInfo where id>50 and id<70。那么很容易就可以得到当前DP不满足条件。所以Histogram对于那种数字限定的查询能够很有效地减少查询DP的数量。

　　CMAP是针对于文本类型的查询，也是装载数据的时候就产生的。CMAP是统计当前DP内，ASCII在1-64位置出现的情况。如下图所示

![infobirght6](http://xiangguo.qiniudn.com/img/posts/infobright/infobright6.jpg "infobright6") 

比如上面的图说明了A在文本的第二个、第三个、第四个位置从来没有出现过。0表示没有出现，1表示出现过。查询中文本的比较归根究底还是按照字节进行比较，所以根据CMAP能够很好地提高文本查询的性能。

　　Pack-To-Pack是Join操作的时候产生的，它是表示join的两个DP中操作的两个列之间关系的位图，也就是二进制表示的矩阵。

　　Knowledge Grid还是比较复杂的，里面还有很多细节的东西，可以参考官方的白皮书和Brighthouse: an analytic data warehouse for ad-hoc queries这篇论文。

##3.工作原理

前面已经简要分析了Infobright的构架，现在来介绍Infobright的工作原理。

　　粗糙集（Rough Sets）是Infobright的核心技术之一。Infobright在执行查询的时候会根据知识网络（Knowledge Grid）把DP分成三类：

- 相关的DP（Relevant Packs），满足查询条件限制的DP
- 不相关的DP（Irrelevant Packs），不满足查询条件限制的DP
- 可疑的DP（Suspect Packs），DP里面的数据部分满足查询条件的限制

下面是一个案例：

![infobirght7](http://xiangguo.qiniudn.com/img/posts/infobright/infobright7.jpg "infobright7") 

如图所示，每一列总共有5个DP，其中限制条件是A>6。所以A1、A2、A4就是不相关的DP，A3是相关的DP，A5是可疑的DP。那么执行查询的时候只需要计算B5中满足条件的记录的和然后加上Sum（B3），Sum（B3）是已知的。此时只需要解压缩B5这个DP。从上面的分析可以知道，Infobright能够很高效地执行一些查询，而且执行的时候where语句的区分度越高越好。where区分度高可以更精确地确认是否是相关DP或者是不相关DP亦或是可以DP，尽可能减少DP的数量、减少解压缩带来的性能损耗。在做条件判断的使用，一般会用到上一章所讲到的Histogram和CMAP，它们能够有效地提高查询性能。

　　多表连接的的时候原理也是相似的。先是利用Pack-To-Pack产生join的那两列的DP之间的关系。

　　比如：SELECT MAX(X.D) FROM T JOIN X ON T.B = X.C WHERE T.A > 6。Pack-To-Pack产生T.B和X.C的DP之间的关系矩阵M。假设T.B的第一个DP和X.C的第一个DP之间有元素交叉，那么M[1,1]=1，否则M[1,1]=0。这样就有效地减少了join操作时DP的数量。


　　
##5. Infobright的数据类型　

Infobright里面支持所有的MySQL原有的数据类型。其中Integer类型比其他数据类型更加高效。尽可能使用以下的数据类型：

- TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT
- DECIMAL（尽量减少小数点位数）
- DATE ，TIME
- 效率比较低的、不推荐使用的数据类型有：
- BINARY VARBINARY
- FLOAT
- DOUBLE
- VARCHAR
- TINYTEXT TEXT

Infobright数据类型使用的一些经验和注意点：

- Infobright的数值类型的范围和MySQL有点不一样，比如Infobright的Int的最小值是-2147483647，而MySQl的Int最小值应该是-2147483648。其他的数值类型都存在这样的问题。
- 能够使用小数据类型就使用小数据类型，比如能够使用SMALLINT就不适用INT，这一点上Infobright和MySQL保持一致。
- 避免效率低的数据类型，像TEXT之类能不用就不用，像FLOAT尽量用DECIMAL代替，但是需要权衡毕竟DECIMAL会损失精度。
- 尽量少用VARCHAR，在MySQL里面动态的Varchar性能就不强，所以尽量避免VARCHAR。如果适合的话可以选择把VARCHAR改成CHAR存储甚至专程INTEGER类型。VARCHAR的优势在于分配空间的长度可变，既然Infobright具有那么优秀的压缩性能，个人认为完全可以把VARCHAR转成CHAR。CHAR会具有更好的查询和压缩性能。
- 能够使用INT的情况尽量使用INT，很多时候甚至可以把一些CHAR类型的数据往整型转化。比如搜索日志里面的客户永久id、客户id等等数据就可以用BIGINT存储而不用CHAR存储。其实把时间分割成year、month、day三列存储也是很好的选择。在我能见到的系统里面时间基本上是使用频率最高的字段，提高时间字段的查询性能显然是非常重要的。当然这个还是要根据系统的具体情况，做数据分析时有时候很需要MySQL的那些时间函数。
- varchar和char字段还可以使用comment lookup，comment lookup能够显著地提高压缩比率和查询性能。


##4. 压缩
　　第三节讲到了解压缩，这里提一提DP的压缩。每个DP中的64K个元素被当成是一个序列，其中所有的null的位置都会被单独存储，然后其余的non-null的数据会被压缩。数据的压缩跟数据的类型有关，infobright会根据数据的类型选择压缩算法。infobright会自适应地调节算法的参数以达到最优的压缩比。
　　
实验的压缩：

![infobirght8](http://xiangguo.qiniudn.com/img/posts/infobright/infobright8.png "infobright8") 

压缩率：

![infobirght9](http://xiangguo.qiniudn.com/img/posts/infobright/infobright9.png "infobright9") 

###comment lookup的使用

前面的章节一直涉及到comment lookup，这里将简单介绍comment lookup的使用。    
comment lookup只能显式地使用在char或者varchar上面。Comment Lookup可以减少存储空间，提高压缩率，对char和varchar字段采用comment lookup可以提高查询效率。    
Comment Lookup实现机制很像位图索引，实现上利用简短的数值类型替代char字段已取得更好的查询性能和压缩比率。CommentLookup的使用除了对数据类型有要求，对数据也有一定的要求。一般要求数据类别的总数小于10000并且当前列的单元数量/类别数量大于10。Comment Lookup比较适合年龄，性别，省份这一类型的字段。
comment lookup使用很简单，在创建数据库表的时候如下定义即可：    

- act   char(15)   comment 'lookup',
- part  char(4) comment 'lookup',


##5. 查询优化
前面已经分析了Infobright的构架，简要介绍了Infobright的压缩过程和工作原理。现在来讨论查询优化的问题。

（1）配置环境    
在Linux下面，Infobright环境的配置可以根据README里的要求，配置brighthouse.ini文件。

（2） 选取高效的数据类型    
参见前面章节。

（3）使用comment lookup    
参见前面章节。

（4）尽量有序地导入数据    
前面分析过Infobright的构架，每一列分成n个DP，每个DPN列面存储着DP的一些统计信息。有序地导入数据能够使不同的DP的DPN内的数据差异化更明显。比如按时间date顺序导入数据，那么前一个DP的max（date）<=下一个DP的min(date)，查询的时候就能够减少可疑DP，提高查询性能。换句话说，有序地导入数据就是使DP内部数据更加集中，而不再那么分散。

（5）使用高效的查询语句。    

- 这里涉及的内容比较多了，总结如下：
- 尽量不适用or，可以采用in或者union取而代之
- 减少IO操作，原因是infobright里面数据是压缩的，解压缩的过程要消耗很多的时间。
- 查询的时候尽量条件选择差异化更明显的语句
- Select中尽量使用where中出现的字段。原因是Infobright按照列处理的，每一列都是单独处理的。所以避免使用where中未出现的字段可以得到较好的性能。
- 限制在结果中的表的数量，也就是限制select中出现表的数量。
- 尽量使用独立的子查询和join操作代替非独立的子查询
- 尽量不在where里面使用MySQL函数和类型转换符
- 尽量避免会使用MySQL优化器的查询操作
- 使用跨越Infobright表和MySQL表的查询操作
- 尽量不在group by 里或者子查询里面使用数学操作，如sum（a*b）。
- select里面尽量剔除不要的字段。

　　Infobright执行查询语句的时候，大部分的时间都是花在优化阶段。Infobright优化器虽然已经很强大，但是编写查询语句的时候很多的细节问题还是需要程序员注意。　　


##6. infobright的中文编码问题

中文乱码的问题的终极解决方案就是所有地方都用同一个字符集，gbk或者utf8，我选用了utf8
infobright的设置方法和mysql自身的大同小异

    1.新建数据库时设置default character set 为utf8，defualt collation为utf8_bin
    2.新建表时也指定为utf8
    3.设置/etc/my-ib.cnf中
        collation_server=utf8_bin
        character_set_server=utf8
        然后再最后加入
        init-connect=SET NAMES utf8



## 7.参考资料：    

维基：www.infobright.org/wiki    
论坛：www.infobright.org/Forums    
博客：[共工的不周山的blog](http://www.wentrue.net/blog/?tag=infobright)
开源贡献：www.infobright.org/Downloads/Contributed-Software/    
我的贡献：[往infobright数据load数据的开源shell脚本](https://github.com/litongbupt/infobright_load_tool)    





