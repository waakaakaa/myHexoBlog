title: 'SQL 中不同类型的表连接'
date: 2014-05-15 11:15:40

---
**1、简介**

在关系型数据库中，join操作是将不同的表中的数据联合在一起时非常通用的一种做法。首先让我们看看join是如何操作的，然后我们探索一下当join和where语句同时存在的时候的执行顺序问题，最后来谈一谈不同类型的join的顺序问题。

<!-- more -->

**2、建立初始的测试表结构（[建表语句到这里下载](http://www.codeproject.com/Articles/435694/Understanding-Table-Joins-using-SQL)）**

表建立完之后，将会看到如下三个表。

![image](http://waakaakaa.qiniudn.com/103306_e9CO_82993.jpg)

我们将通过以上三个表来演示join操作。这三个表都是用来做演示的，所以我并没有使用主键和外键。

**3、表的笛卡尔乘积**

一般情况下，我们使用两个表中的相关字段进行join操作，例如，employee表中的DeptId字段对应于Department表中的DepId字段，通过这种方式进行join。

下面的一个例子是不使用关联字段来做表的连接。这里TableA和TableB以笛卡尔乘积的方式连在了一起。笛卡尔乘积就是先从第一个表中取出一条记录，和第二章表中的每一条记录配合，然后再取出第二条记录，同样和第二张表的所有记录配合，直至第一张表中的所有记录都取完。所以最终的结果数量将是两张表的乘积。

![image](http://waakaakaa.qiniudn.com/103407_CPoq_82993.jpg)

**4、join两张表**

当我们想做两个表的连接，而不是像上面的例子一样得到大量的无用的结果的时候，我们就得从两张表中选取一个join列。我下面给出的例子是使用id作为join列的。我们可以通过这种方式使得结果只映射我们需要的那一部分，从而过滤掉了无用数据。

![image](http://waakaakaa.qiniudn.com/103435_FDRs_82993.jpg)

注意：在笛卡尔乘积表中的第一行和第五行满足了join的映射关系，从而被作为结果，其他的都被过滤掉了。

**5、join多张表**

上面的例子是join两张表，如果想join多张表，我们需要在上面的结果中选择一列，然后再在新表中选择一列，将这两者作为join字段，然后指定join的规则，这样我们理论上可以join任意多张表。

![image](http://waakaakaa.qiniudn.com/103524_uGSe_82993.jpg)

首先，Table_A和Table_B做了连接，就上上面的join两张表的例子，然后将join的结果作为一张表AB。再将AB与Table_C连接。

![image](http://waakaakaa.qiniudn.com/103553_ofCc_82993.jpg)

**6、join类型**

在两张不同的表做连接有3中join类型。

1. full join
2. inner join
3. outer join(left outer join、right outer join)

在上面两个例子中我们看到的是inner join。如果我们连接表自身就叫做self join。这个特殊类型不会混淆连接类型。

**7、full join**

full join和笛卡尔有些不同，笛卡尔积会获取所有可能的结果。而full join将匹配的结果与所有左边的表中不匹配右边的行和右边的表中所有不匹配左边的行加在一起，在不匹配的地方使用NULL代替。结果行数=匹配行数+左表剩余行数+右表剩余行数。

![image](http://waakaakaa.qiniudn.com/103707_a5Lt_82993.jpg)

![image](http://waakaakaa.qiniudn.com/103715_XlOu_82993.jpg)

在上面的图片中，蓝色的行是两个表匹配的行。

第二行，左边绿色，右边红色的是不匹配的，左表中的行是存在的，而右表中的字段则被null填充。

第三行，左边红色，右边绿色的同样是不匹配的，右表中的行是存在的，而左表中的字段则被null填充。

**8、left join**

左连接（left join）保证左表中的所有行都有，而当不匹配的时候以NULL填充右表字段。

![image](http://waakaakaa.qiniudn.com/103812_omnc_82993.jpg)

蓝色匹配，红色和绿色不匹配

**9、right join**

反过来，右连接（right join）保证右表中所有的行都有，而当不匹配的时候以NULL填充左表字段。

![image](http://waakaakaa.qiniudn.com/103918_9ZGB_82993.jpg)

蓝色匹配，红色和绿色不匹配

**10、inner join**

inner join就是只列出匹配的行。

**11、self join**

表连接自身叫做self join。为了解释一下这个让我们看如下图中的employee表。EmployeeID是此表的主键，ReportsTo引用了此表的主键。我们可以想象成这样，ReportTo字段引用代表该雇员的上司，其上司同样也是雇员。

![image](http://waakaakaa.qiniudn.com/104034_6UfA_82993.jpg)

看如下例子

 

这里，有ReportTo指向的行是Manager，所以employee是左表，Manager是右表。

**12、执行顺序**

当连接中有where语句的时候我们需要注意连接和where的执行顺序问题。

1. 将where语句先于join执行，因为执行完where查询的结果将会比较少，从而join操作性能会提升。
2. 将where语句后与join执行。

以上两者将在inner join的时候返回同样的结果，但是当使用outer join的时候至少有一种连接操作的返回结果不同。看下面例子。

![image](http://waakaakaa.qiniudn.com/104203_qlXI_82993.jpg)

![image](http://waakaakaa.qiniudn.com/104332_G4d9_82993.jpg)

![image](http://waakaakaa.qiniudn.com/104345_t1jS_82993.jpg)

所以记住当外连接的时候尽量先执行join操作然后执行where语句。

**13、连接的顺序**

当你想将inner join和outer join同时使用的时候join的顺序也是非常重要的。

什么是连接的顺序？如果我像这样连接三张表【X inner Y】left Z，顺序就是先inner join再left join。

让我们回到上面的例子中，你想得到的结果是获取所有客户的名字，不管他们是否有订单。如果他们确实有一些订单，还要列出了客户订购的数量。

看如下的查询【先outer join再inner join】

![image](http://waakaakaa.qiniudn.com/104419_CpoO_82993.jpg)

1. 在Orders和Customers中进行了right join。右连接能保证你获取所有Customer的信息，不管他是否有order。
2. 现在上面的结果将和Order Details连接。但是我们需要注意的是，在右连接的结果中有两行roderid为null的，因为这两个customer并没有任何order，而在后面做inner join的时候，由于orderid为null，inner join将跳过这两行，从而导致这两个customer的信息被过滤掉了。
再让我们看看下面的这个查询【先inner join再outer join】

![image](http://waakaakaa.qiniudn.com/104459_m47U_82993.jpg)

让我们分析一下为什么这才是我们想要的结果。

首先Order和Order Details表做inner join，所有匹配的结果都将被列出来，然后将此结果作为左表，Customer表作为右表，右表的所有行都将被列出来，不管其匹配与否（言外之意，那两个没有order的customer也将被列出来）。

所以，在我们同时使用inner join和outer join的时候一定要对连接的顺序做慎重考虑。

**14、获取同样数据的其他办法**

看如下查询

![image](http://waakaakaa.qiniudn.com/104551_IGzJ_82993.jpg)

1. 首先查询出Customers将其作为左表
2. 然后将Orders表查询出来，仍然作为左表
3. 然后查询出Order Details表将其作为右表与Orders表进行inner join。
4. 最后Customers表将于第三步查询出的结果进行左连接。别忘了左连接将保证Customers表不丢失任何记录。

[OSChina.NET](http://oschina.net/)原创翻译/[原文链接](http://www.codeproject.com/Articles/435694/Understanding-Table-Joins-using-SQL)