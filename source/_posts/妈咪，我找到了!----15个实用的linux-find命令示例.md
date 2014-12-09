title: '妈咪，我找到了! -- 15个实用的Linux find命令示例'
date: 2014-05-14 14:20:06

---
除了在一个目录结构下查找文件这种基本的操作，你还可以用find命令实现一些实用的操作，使你的命令行之旅更加简易。

本文将介绍15种无论是于新手还是老鸟都非常有用的Linux find命令。

首先，在你的home目录下面创建下面的空文件，来测试下面的find命令示例。

<!-- more -->

	# vim create_sample_files.sh
	touch MybashProgram.sh
	touch mycprogram.c
	touch MyCProgram.c
	touch Program.c
	
	mkdir backup
	cd backup
	
	touch MybashProgram.sh
	touch mycprogram.c
	touch MyCProgram.c
	touch Program.c
	
	# chmod +x create_sample_files.sh
	
	# ./create_sample_files.sh
	
	# ls -R
	.:
	backup                  MybashProgram.sh  MyCProgram.c
	create_sample_files.sh  mycprogram.c      Program.c
	
	./backup:
	MybashProgram.sh  mycprogram.c  MyCProgram.c  Program.c

**1. 用文件名查找文件**

这是find命令的一个基本用法。下面的例子展示了用MyCProgram.c作为查找名在当前目录及其子目录中查找文件的方法。

	# find -name "MyCProgram.c"
	./backup/MyCProgram.c
	./MyCProgram.c

**2.用文件名查找文件，忽略大小写**

这是find命令的一个基本用法。下面的例子展示了用MyCProgram.c作为查找名在当前目录及其子目录中查找文件的方法，忽略了大小写。

	# find -iname "MyCProgram.c"
	./mycprogram.c
	./backup/mycprogram.c
	./backup/MyCProgram.c
	./MyCProgram.c

**3. 使用mindepth和maxdepth限定搜索指定目录的深度**

在root目录及其子目录下查找passwd文件。

	# find / -name passwd
	./usr/share/doc/nss_ldap-253/pam.d/passwd
	./usr/bin/passwd
	./etc/pam.d/passwd
	./etc/passwd

在root目录及其1层深的子目录中查找passwd. (例如root — level 1, and one sub-directory — level 2)

	# find -maxdepth 2 -name passwd
	./etc/passwd

在root目录下及其最大两层深度的子目录中查找passwd文件. (例如 root — level 1, and two sub-directories — level 2 and 3 )

	# find / -maxdepth 3 -name passwd
	./usr/bin/passwd
	./etc/pam.d/passwd
	./etc/passwd

在第二层子目录和第四层子目录之间查找passwd文件。

	# find -mindepth 3 -maxdepth 5 -name passwd
	./usr/bin/passwd
	./etc/pam.d/passwd

**4. 在find命令查找到的文件上执行命令**

下面的例子展示了find命令来计算所有不区分大小写的文件名为“MyCProgram.c”的文件的MD5验证和。{}将会被当前文件名取代。

	find -iname "MyCProgram.c" -exec md5sum {} \;
	d41d8cd98f00b204e9800998ecf8427e  ./mycprogram.c
	d41d8cd98f00b204e9800998ecf8427e  ./backup/mycprogram.c
	d41d8cd98f00b204e9800998ecf8427e  ./backup/MyCProgram.c
	d41d8cd98f00b204e9800998ecf8427e  ./MyCProgram.c

**5. 相反匹配**

显示所有的名字不是MyCProgram.c的文件或者目录。由于maxdepth是1，所以只会显示当前目录下的文件和目录。

	find -maxdepth 1 -not -iname "MyCProgram.c"
	.
	./MybashProgram.sh
	./create_sample_files.sh
	./backup
	./Program.c

**6. 使用inode编号查找文件**

任何一个文件都有一个独一无二的inode编号，借此我们可以区分文件。创建两个名字相似的文件，例如一个有空格结尾，一个没有。

	touch "test-file-name"

	# touch "test-file-name "
	[Note: There is a space at the end]
	
	# ls -1 test*
	test-file-name
	test-file-name

从ls的输出不能区分哪个文件有空格结尾。使用选项-i，可以看到文件的inode编号，借此可以区分这两个文件。

	ls -i1 test*
	16187429 test-file-name
	16187430 test-file-name

你可以如下面所示在find命令中指定inode编号。在此，find命令用inode编号重命名了一个文件。

	find -inum 16187430 -exec mv {} new-test-file-name \;
	
	# ls -i1 *test*
	16187430 new-test-file-name
	16187429 test-file-name

你可以在你想对那些像上面一样的糟糕命名的文件做某些操作时使用这一技术。例如，名为file?.txt的文件名字中有一个特殊字符。若你想执行“rm file?.txt”，下面所示的所有三个文件都会被删除。所以，采用下面的步骤来删除"file?.txt"文件。

	ls
	file1.txt  file2.txt  file?.txt

找到每一个文件的inode编号。

	ls -i1
	804178 file1.txt
	804179 file2.txt
	804180 file?.txt

如下所示： 使用inode编号来删除那些具有特殊符号的文件名。

	find -inum 804180 -exec rm {} \;
	
	# ls
	file1.txt  file2.txt
	[Note: The file with name "file?.txt" is now removed]

**7. 根据文件权限查找文件**

下面的操作时合理的：

* 找到具有指定权限的文件
* 忽略其他权限位，检查是否和指定权限匹配
* 根据给定的八进制/符号表达的权限搜索

此例中，假设目录包含以下文件。注意这些文件的权限不同。

	ls -l
	total 0
	-rwxrwxrwx 1 root root 0 2009-02-19 20:31 all_for_all
	-rw-r--r-- 1 root root 0 2009-02-19 20:30 everybody_read
	---------- 1 root root 0 2009-02-19 20:31 no_for_all
	-rw------- 1 root root 0 2009-02-19 20:29 ordinary_file
	-rw-r----- 1 root root 0 2009-02-19 20:27 others_can_also_read
	----r----- 1 root root 0 2009-02-19 20:27 others_can_only_read

找到具有组读权限的文件。使用下面的命令来找到当前目录下对同组用户具有读权限的文件，忽略该文件的其他权限。

	find . -perm -g=r -type f -exec ls -l {} \;
	-rw-r--r-- 1 root root 0 2009-02-19 20:30 ./everybody_read
	-rwxrwxrwx 1 root root 0 2009-02-19 20:31 ./all_for_all
	----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read
	-rw-r----- 1 root root 0 2009-02-19 20:27 ./others_can_also_read

找到对组用户具有只读权限的文件。

	find . -perm g=r -type f -exec ls -l {} \;
	----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read
	
找到对组用户具有只读权限的文件(使用八进制权限形式)。

	find . -perm 040 -type f -exec ls -l {} \;
	----r----- 1 root root 0 2009-02-19 20:27 ./others_can_only_read

**8. 找到home目录及子目录下所有的空文件(0字节文件)**

下面命令的输出文件绝大多数都是锁定文件盒其他程序创建的place hoders

	find ~ -empty

只列出你home目录里的空文件。

	find . -maxdepth 1 -empty

只列出当年目录下的非隐藏空文件。

	find . -maxdepth 1 -empty -not -name ".*"

**9. 查找5个最大的文件**

下面的命令列出当前目录及子目录下的5个最大的文件。这会需要一点时间，取决于命令需要处理的文件数量。

	find . -type f -exec ls -s {} \; | sort -n -r | head -5

**10. 查找5个最小的文件**

方法同查找5个最大的文件类似，区别只是sort的顺序是降序。

	find . -type f -exec ls -s {} \; | sort -n  | head -5

上面的命令中，很可能你看到的只是空文件(0字节文件)。如此，你可以使用下面的命令列出最小的文件，而不是0字节文件。

	find . -not -empty -type f -exec ls -s {} \; | sort -n  | head -5

**11. 使用-type查找指定文件类型的文件**

只查找socket文件

	find . -type s

查找所有的目录

	find . -type d

查找所有的一般文件

	find . -type f

查找所有的隐藏文件

	find . -type f -name ".*"

查找所有的隐藏目录

	find -type d -name ".*"

**12. 通过和其他文件比较修改时间查找文件**

显示在指定文件之后做出修改的文件。下面的find命令将显示所有的在ordinary_file之后创建修改的文件。

	ls -lrt
	total 0
	-rw-r----- 1 root root 0 2009-02-19 20:27 others_can_also_read
	----r----- 1 root root 0 2009-02-19 20:27 others_can_only_read
	-rw------- 1 root root 0 2009-02-19 20:29 ordinary_file
	-rw-r--r-- 1 root root 0 2009-02-19 20:30 everybody_read
	-rwxrwxrwx 1 root root 0 2009-02-19 20:31 all_for_all
	---------- 1 root root 0 2009-02-19 20:31 no_for_all
	
	# find -newer ordinary_file
	.
	./everybody_read
	./all_for_all
	./no_for_all

**13. 通过文件大小查找文件**

使用-size选项可以通过文件大小查找文件。

查找比指定文件大的文件

	find ~ -size +100M

查找比指定文件小的文件

	find ~ -size -100M

查找符合给定大小的文件

	find ~ -size 100M

注意: – 指比给定尺寸小，+ 指比给定尺寸大。没有符号代表和给定尺寸完全一样大。

**14. 给常用find操作取别名**

若你发现有些东西很有用，你可以给他取别名。并且在任何你希望的地方执行。

常用的删除a.out文件。

	alias rmao="find . -iname a.out -exec rm {} \;"
	# rmao

删除c程序产生的core文件。

	alias rmc="find . -iname core -exec rm {} \;"
	# rmc

**15. 用find命令删除大型打包文件**

下面的命令删除大于100M的*.zip文件。

	find / -type f -name *.zip -size +100M -exec rm -i {} \;"

用别名rm100m删除所有大雨100M的*.tar文件。使用同样的思想可以创建rm1g,rm2g,rm5g的一类别名来删除所有大于1G,2G,5G的文件。

	alias rm100m="find / -type f -name *.tar -size +100M -exec rm -i {} \;"
	# alias rm1g="find / -type f -name *.tar -size +1G -exec rm -i {} \;"
	# alias rm2g="find / -type f -name *.tar -size +2G -exec rm -i {} \;"
	# alias rm5g="find / -type f -name *.tar -size +5G -exec rm -i {} \;"
	
	# rm100m
	# rm1g
	# rm2g
	# rm5g

Find命令示例(第二部分)

若你喜欢这篇关于find命令的Mommy文章，别忘了看看第二部分的关于find命令的Daddy文章。[爹地，我找到了!, 15个极好的Linux find命令示例](http://www.oschina.net/translate/15-practical-unix-linux-find-command-examples-part-2)