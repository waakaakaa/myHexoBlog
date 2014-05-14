title: '爹地，我找到了! -- 15个极好的Linux find命令示例'
date: 2014-05-14 14:34:41

---
前阵子，我们审查了15件实事 find命令的例子（第一部分）。查找命令可以做很多比只是在寻找基于名称的文件 （第2部分）在这篇文章中，让我们来讨论15高级find命令的例子， 包括-根据它访问，修改或改变的时间查找文件，查找文件相比之下，执行操作找到的文件等， 拉梅什纳塔拉詹：这是我的照片中的可爱的小女儿。她很高兴地发现在加州长滩水族馆海狮。 

**基于访问/修改/更改时间查找文件**

你可以找到基于以下三个文件的时间属性的文件。

* 访问时间的文件。文件访问时，访问时间得到更新。
* 的文件的修改时间。文件内容修改时，修改时间得到更新。
* 更改文件的时间。更改时间时，被更新的inode数据的变化。

在下面的例子中，min选项之间的差异和时间选项是参数。

* 分论点将它的参数为分钟。例如，60分钟（1小时）= 60分钟。
* 时间参数，将它的参数为24小时。例如，时间2 = 2 * 24小时（2天）。
* 虽然这样做的24个小时计算，小数部分都将被忽略，所以25小时为24小时，和47小时取为24小时，仅48小时为48小时。要获得更清晰的参考atime的部分find命令的手册页。

**例1：找到在1个小时内被更改的文件**

想要通过文件修改时间找出文件，可以使用参数 -mmin -mtime。下面是man手册中有关mmin和mtime的定义。

* -mmin n 文件最后一次修改是在n分钟之内
* -mtime n 文件最后一次修改是在 n*24小时之内（译者注：也就是n天了呗）

执行下面例子中的命令，将会找到当前目录以及其子目录下，最近一次修改时间在1个小时（60分钟）之内的文件或目录

	# find . -mmin -60

同样的方式，执行下面例子中的命令，将会找到24小时（1天）内修改了的文件（文件系统根目录 / 下）

	# find / -mtime -1

**例2：找到1个小时内被访问过的文件**

想要通过文件访问时间找出文件，可以使用参数 -amin -atime。下面是man手册中有关amin和atime的定义。

* -amin n 文件最后一次访问是在n分钟之内
* -atime n 文件最后一次访问是在 n*24小时之内

执行下面例子中的命令，将会找到当前目录以及其子目录下，最近一次访问时间在1个小时（60分钟）之内的文件或目录

	# find . -amin -60

同样的方式，执行下面例子中的命令，将会找到24小时（1天）内被访问了的文件（文件系统根目录 / 下）

	# find / -atime -1

**例3：查找一个小时内状态被改变的文件**

（译者注：这里的改变更第1个例子的更改文件内容时间是不同概念，这里是更改的是文件inode的数据，比如文件的权限，所属人等等信息）

要查找文件的inode的更改时间，使用-cmin和-ctime选项

* -cmin n  文件的状态在n分钟内被改变
* -ctime n  文件状态在n*24小时内（也就是n天内）被改变

（译者注：如果上面的n为-n形式，则表示n分钟/天之内，n为+n则表示n分钟/天之前）

下面的例子在当前目录和其子目录下面查找一个小时内文件状态改变的文件（也就是60分钟内）：

	# find . -cmin -60

同样的道理，下面的例子在根目录/及其子目录下一天内（24小时内）文件状态被改变的文件列表：

	# find / -ctime -1

**例4：搜索仅仅限定于文件，不显示文件夹**

上面的例子搜索出来不仅仅有文件，还会显示文件夹。因为当一个文件被访问的时候，它所处的文件夹也会被访问，如果你对文件夹不感兴趣，那么可以使用 -type f 选项

下面的例子会显示30分钟内被修改过的文件，文件夹不显示：

	# find /etc/sysconfig -amin -30
	.
	./console
	./network-scripts
	./i18n
	./rhn
	./rhn/clientCaps.d
	./networking
	./networking/profiles
	./networking/profiles/default
	./networking/profiles/default/resolv.conf
	./networking/profiles/default/hosts
	./networking/devices
	./apm-scripts
	[注: 上面的输出包含了文件和文件夹]
	
	# find /etc/sysconfig -amin -30 -type f
	./i18n
	./networking/profiles/default/resolv.conf
	./networking/profiles/default/hosts
	[注: 上面的输出仅仅包含文件]

**例5： 仅仅查找非隐藏的文件（不显示隐藏文件）：**

如果我们查找的时候不想隐藏文件也显示出来，可以使用下面的正则式查找：

下面的命令会显示当前目录及其子目录下15分钟内文件内容被修改过的文件，并且只列出非隐藏文件。也就是说，以.开头的文件时不会显示出来的

	# find . -mmin -15 \( ! -regex ".*/\..*" \)

基于文件比较的查找命令

我们平时通过更别的东西进行比较，会更容易记住一些事情。比如说我想找出在我编辑test文件之后编辑过的文件。你可以通过test这个文件的编辑时间作为比较基准去查找之后编辑过的文件：

**例6： 查找文件修改时间在某一文件修改后的文件：**

	语法： find -newer FILE

下面的例子显示在/etc/passwd修改之后被修改过的文件。对于系统管理员，想知道你新增了一个用户后去跟踪系统的活动状态是很有帮助的（万一那新用户不老实，一上来就乱搞，你很快就知道了  ^_^）：

	# find -newer /etc/passwd

**例7：查找文件访问时间在某一文件的修改时间之后的文件：**

	# find -newer /etc/passwd

下面的例子显示所有在/etc/hosts文件被修改后被访问到的文件。如果你新增了一个主机/端口记录在/etc/hosts文件中，你很可能很想知道在那之后有什么文件被访问到了，下面是这个命令：

	# find -anewer /etc/hosts

**例8：查找状态改变时间在某个文件修改时间之后的文件：**

	语法： find -cnewer FILE

下面的例子显示在修改文件/etc/fstab之后所有文件状态改变过的文件。如果你在/etc/fstab新增了一个挂载点，你很可能想知道之后哪些文件的状态发生了改变，这时候你可以使用如下命令：

	# find -cnewer /etc/fstab

在查找到的文件列表结果上直接执行命令：

这之前你已经看到了如果通过find命令去查找各种条件的文件列表。如果你对这些find命令还不熟悉，我建议你看完上面的第一部分

接下来这部分我们向你介绍如果在find命令上执行各种不同的命令，也就是说如何去操作find命令查找出来的文件列表。

我们能在find命令查找出来的文件名列表上指定任意的操作：

	# find <CONDITION to Find files> -exec <OPERATION> \;

其中的OPERATION可以是任意的命令，下面列举一下比较常用的：

*  rm 命令，用于删除find查找出来的文件
*  mv 命令，用于重命名查找出的文件
*  ls -l 命令，显示查找出的文件的详细信息
*  md5sum， 对查找出的文件进行md5sum运算，可以获得一个字符串，用于检测文件内容的合法性
*  wc 命令，用于统计计算文件的单词数量，文件大小等待
*  执行任何Unix的shell命令
*  执行你自己写的shell脚本，参数就是每个查找出来的文件名

**例9：在find命令输出上使用 ls -l， 列举出1小时内被编辑过的文件的详细信息**

	# find -mmin -60
	./cron
	./secure
	
	# find -mmin -60 -exec ls -l {} \;
	-rw-------  1 root root 1028 Jun 21 15:01 ./cron
	-rw-------  1 root root 831752 Jun 21 15:42 ./secure

**例10：仅仅在当前文件系统中搜索**

系统管理员有时候仅仅想在/挂载的文件系统分区上搜索，而不想去搜索其他的挂载分区，比如/home/挂载分区。如果你有多个分区被挂载了，你想在/下搜索，一般可以按下面的这样做

下面这个命令会搜索根目录/及其子目录下所有.log结尾的文件名。如果你有多个分区在/下面，那么这个搜索会去搜索所有的被挂载的分区：

	# find / -name "*.log"

如果我们使用-xdev选项，那么仅仅会在在当前文件系统中搜索，下面是在xdev的man page上面找到的一段-xdev的定义：

* -xdev Don’t descend directories on other filesystems.

下面的命令会在/目录及其子目录下搜索当前文件系统(也就是/挂载的文件系统)中所有以.log结尾的文件，也就是说如果你有多个分区挂载在/下面，下面的搜索不会去搜索其他的分区的（比如/home/）

	# find / -xdev -name "*.log"

**例11： 在同一个命令中使用多个{}**

linux手册说命令中只能使用一个{}，不过你可以像下面这样在同一个命令中使用多个{}

	# find -name "*.txt" cp {} {}.bkup \;

注意，在同一个命令中使用这个{}是可以的，但是在不同的命令里就不行了，也就是说，如果你想象下面这样重命名文件是行不通的

	find -name "*.txt" -exec mv {} `basename {} .htm`.html \;

**例12： 使用多个{}实例**

你可以像下面这样写一个shell脚本去模拟上面那个重命名的例子

	# mv "$1" "`basename "$1" .htm`.html"

上面的双引号是为了防止文件名中出现的空格，不加的话会有问题。然后你把这个shell脚本保存为mv.sh，你可以像下面这样使用find命令了

	find -name "*.html" -exec ./mv.sh '{}' \;

所以，任何情况下你在find命令执行中想使用同一个文件名多次的话，先写一个脚本，然后在find中通过-exec执行这个脚本，把文件名参数传递进去就行，这是最简单的办法

**例13： 将错误重定向到/dev/nul**

重定向错误输出一般不是什么好的想法。一个有经验的程序员懂得在终端显示错误并及时修正它是很重要的。

尤其是在find命令中重定向错误不是个好的实践。 但是如果你确实不想看到那些烦人的错误，想把错误都重定向到null设备中（也就是linux上的黑洞装置，任何丢进去的东西消失的无影无踪了）。你可以像下面这样做

	find -name "*.txt" 2>>/dev/null

有时候这是很有用的。比如，如果你想通过你自己的账号在/目录下查找所有的*.conf文件，你会得到很多很多的"Permission denied"的错误消息， 就像下面这样：

	$ find / -name "*.conf"
	/sbin/generate-modprobe.conf
	find: /tmp/orbit-root: Permission denied
	find: /tmp/ssh-gccBMp5019: Permission denied
	find: /tmp/keyring-5iqiGo: Permission denied
	find: /var/log/httpd: Permission denied
	find: /var/log/ppp: Permission denied
	/boot/grub/grub.conf
	find: /var/log/audit: Permission denied
	find: /var/log/squid: Permission denied
	find: /var/log/samba: Permission denied
	find: /var/cache/alchemist/printconf.rpm/wm: Permission denied
	[Note: There are two valid *.conf files burned in the "Permission denied" messages]

你说烦人不？所以，如果你只想看到find命令真实的查找结果而不是这些"Permission denied"错误消息，你可以将这些错误消息重定向到/dev/null中去

	$ find / -name "*.conf" 2>>/dev/null
	/sbin/generate-modprobe.conf
	/boot/grub/grub.conf
	[Note: All the "Permission denied" messages are not displayed]

**例14： 将文件名中的空格换成下划线**

你从网上下载下来的音频文件的文件名很多都带有空格。但是带有空格的文件名在linux(类Unix)系统里面是很不好的。你可以使用find然后后面加上rename命令的替换功能去重命名这些文件，将空格转换成下划线

下面显示怎样将所有mp3文件的文件名中的空格换成_

	$ find . -type f -iname “*.mp3″ -exec rename “s/ /_/g” {} \;

**例15： 在find结果中同时执行两条命令**

在find的man page页面中，下面是一次文件查找遍历中使用两条命令的语法举例

下面的find命令的例子，遍历文件系统一次，列出拥有setuid属性的文件和目录，写入/root/suid.txt文件， 如果文件大小超过100M，将其记录到/root/big.txt中

	# find / \( -perm -4000 -fprintf /root/suid.txt '%#m %u %p\n' \) , \
	\( -size +100M -fprintf /root/big.txt '%-10s %p\n' \)
