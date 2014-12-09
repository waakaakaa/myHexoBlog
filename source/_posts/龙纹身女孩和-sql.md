title: '龙纹身女孩和SQL'
date: 2014-05-14 11:39:02

---
我喜欢大卫·芬奇(David Fincher)拍的电影《龙纹身女孩》，他成功的把小说《龙纹身女孩》搬上了荧幕，超出了我的预期。我本以为这又是一部肤浅的、愤世嫉俗的用来敛钱的好莱坞电影，事实情况却是，这是一部情节紧张，能引起共鸣的电影，只是里面的淫杀犯罪让人毛骨悚然。我最喜欢的一个情节是龙纹身女孩用SQL来查找40年前的凶杀案的过程。

![image](/img/龙纹身女孩和SQL/1.jpg)

我们从电影里可以看到她使用笔记本电脑，轻而易举的进入瑞典警察局数据库，当她敲入像‘unsolved(未破案)’和‘decapitation(斩首)’等关键词时，屏幕上翻滚着绿色的检索出的信息，虽然我们看不清她使用的完整的查询语句：

![image](/img/龙纹身女孩和SQL/2.jpg)

<!-- more -->

处于一种天生的好奇，我忍不住截取了这些镜头画面，用Photoshop拼接了一下，下面是我得到的结果：

![image](/img/龙纹身女孩和SQL/3.jpg)

你马上能发现，这不是Oracle SQL——很显然 AS 关键字在Oracle里不能用在表假名上。事实上，如果我们回去看看她那个令人兴奋的查询结果输出时，你会看到 mysql 的提示符，而且还有 use [dbname] 连接数据库的语法，下面是一个更详细的画面：

![image](/img/龙纹身女孩和SQL/4.jpg)

我们实际上可以把她用的left join关键词表的SQL语句整理出来。

最终我们获得了一个全屏的输出结果信息：

![image](/img/龙纹身女孩和SQL/5.jpg)

下面就是我们Oracle“WTF研究会”部门重新构造出的她使用的SQL：

	SELECT DISTINCT v.fname, v.lname, i.year, i.location, i.report_file
	FROM   Incident AS i
    	   LEFT JOIN V(ictim?)...  -- presumably v.incident_id = i.id
    	   LEFT JOIN Keyword AS k ON k.incident_id = i.id
	WHERE  i.year BETWEEN 1947 AND 1966
	AND    i.type = 'HOMICIDE'
	AND    v.sex = 'F'
	AND    i.status = 'UNSOLVED'
	AND    ...
    	   OR v.fname IN ('Mari', 'Magda')
    	   OR SUBSTR ...
	AND    (k.keyword IN ('rape', 'decapitation', 'dismemberment', 'fire', 	'altar', 'priest', 'prostitute')
    	   ...
    	   AND SUBSTR(v.fname, 1, 1) = 'R' AND SUBSTR(v.lname, 1, 1) = 'L');

	+--------+---------+------+-----------+----------------------------------+
	| fname  | name    | year | location  | report_file                      |
	+--------+---------+------+-----------+----------------------------------+
	| Anna   | Wedin   | 1956 | Mark      | FULL POLICE REPORT NOT DIGITIZED |
	| Linda  | Janson  | 1955 | Mariestad | FULL POLICE REPORT NOT DIGITIZED |
	| Simone | Grau    | 1958 | Goteborg  | FULL POLICE REPORT NOT DIGITIZED |
	| Lea    | Persson | 1962 | Uddevalla | FULL POLICE REPORT NOT DIGITIZED |
	| Kajsa  | Severin | 1962 | Dals-Ed   | FULL POLICE REPORT NOT DIGITIZED |
	+--------+---------+------+-----------+----------------------------------+

你也许会很惊讶，很奇怪，这样一个顶级的黑客为什么要outer-join的方式连接Victims(被害人)表和Keywords(关键词)表呢，还使用这样的文字过滤方式，岂不知MySQL里是有 like 语法的，更奇怪的是输出结果里根本没有姓和名分别以’R L’打头的受害人。

[本文英文原文链接：[The Girl With The ANSI Tattoo](http://oracle-wtf.blogspot.co.uk/2012/05/girl-with-ansi-tattoo.html) ]