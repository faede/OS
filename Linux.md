ls 

第一个字母 目录(d) 、文件(-)、字符型文件(c)、块设备(b)



touch test_one

创建新的空文件,更改文件的修改时间（不需要改变内容）



链接文件

符号链接，是一个文件，指向存放在虚拟目录结构中某个地方的另一个文件。两个通过符号链接在一起的文件，彼此内容并不相同。

ln -s data_file sl_data_file

sl指向data

inode不同。

硬链接

创建独立的虚拟文件，其中包含了原始文件的信息和位置，本质上是同一个文件，引用硬链接文件等于引用了源文件。

inode相同。

只能对处于同一存储媒体的文件创建硬链接，否则只能用符号链接。



mv

移动或重命名



rm 

建议加-i

rm -i fall

-f强制删除



mkdir

创建目录

mkdir -p new_dir/sub_dir/under_dir

-p参数会自动创建缺失的目录。



rmdir

默认只接受空目录

或者 rm -ri My_Dir



tree

展示目录树



### 3.8查看文件内容

file My_file

查看文件类型，编码，符号链接，二进制文件需要何种库



cat test1

显示

-n 加上行号

-b 只给有文本的地方加行号

-T 不显示制表符(^I )替换所有制表符



more

显示一页之后就停下来



less

加强more （less is more）

可以上下键翻页等



tail

展示尾部文件，默认10行

tail -n 2 log_file

只展示最后两行



head

前十行

head -5 log_file



## 更多的bash shell命令

### 4.1监测程序

ps

默认显示运行在当前控制台下的属于当前用户的进程

Unix风格的参数，前面加单破折号。

BSD风格的参数，前面不加破折号。

GNU风格的长参数前面加双破折线。



ps -ef

查看系统运行的所有进程

-f

UID 启动进程的用户，PID进程的进程ID，PPID父进程的进程号，C进程生命周期中CPU利用率，STIME进程启动时系统时间，TTY进程启动时的中断设备，TIME运行进程需要的累计CPU时间，CMD启动的程序名称。

ps -l

F内个分配给进程的系统标记，S进程的转台,PRI进程的优先级（和数字大小成反比），NI谦让度值用来决定优先级，ADDR进程的内存地址，SZ加入进程被换出，所需交换空间的大致大小，WCHAN进程休眠的内核函数的地址。

ps --forest

显示进程的层级信息。



top

ps只能显示某个特定时间点的信息，动态使用top



Linux中进程之间通过信号来通信。

信号 名称 描述

1	HUP	挂起

2	INT	中断

3	QUIT	结束进程

9	KILL	无条件错误

11	SEGV	段错误	

15	TERM	 尽可能终止

17	STOP	无条件停止运行，但不终止

18	TSTP	停止或暂停，但继续在后台运行

19	CONT	在STOP或TSTP之后继续执行

kill

kill 3940

通过PID给进程发信号

kill -s HUP 3940

如果要强制终止，-s参数支持其他信号。

### 4.2 监测磁盘空间

Linux文件系统将所有的磁盘都并入一个虚拟目录下。在使用新的存储媒体之前需要把它放入虚拟目录下，这项工作成为挂载。



mount

输出当前系统上挂载的设备列表

四部分：

媒体的设备文件名	媒体挂载到虚拟目录下的挂载点	文件系统类型	已挂载媒体的访问状态

mount -t type device directory

type参数指定了磁盘被格式化的系统文件类型。



umount [directory| device]

卸载设备，正在使用时不允许卸载



df

查看所有已挂载磁盘的使用情况



du

显示某个特定目录（默认当前）的磁盘使用情况

左边占用的磁盘块数，列表从目录层级的最底部开始，按文件、子目录、目录逐级向上。

-c 显示所有已列出文件的总大小

-h 按用户易读的格式输出大小



### 4.3处理数据

sort

排序数据，默认被当做字符来排序

sort -n file2

排序数字

sort -M file3

按日期排序

sort 	-t 	':' 	-k	3 	-n /etc/passwd

-t参数指定字段分隔符，-k参数指定排序的字段。

du -sh | sort -nr

-r反向



grep [options] pattern [file]

grep在输入或指定的文件中查找包含匹配指定模式的字符的行

-v 

反向搜索，输出不匹配该模式的行

-n 显示行号

grep -e t -e f

-e指定多个匹配模式



压缩数据

工具				文件拓展名			描述

bzip2 			 .bz2			采用Burrows-Wheeler块排序文本压缩算法和霍夫曼编码

compress		.Z				最初的Unix文件压缩工具（几乎不使用）

gzip				.gz				GNU压缩工具，采用Lepel-Ziv编码

zip					,zip			windows上的PKZIP工具的Unix实现

gzip	压缩文件

gzcat	哟你查看压缩过的文件

gunzip	用来解压文件



归档数据

tar命令用来将文件写入到磁带设备上归档，也能把输出写出到文件里

tar function [options]   object1   object2

function

功能	长名称	描述

-A	 --concatenate	将一个已有tar归档文件追加到另一个已有tar归档文件

-c	--cteate	 创建一个新的tar归档文件

-d 	--diff	检查归档文件和文件系统的不同之处

​		--delete	 从已有tar归档文件中删除

-r	--apend	追加文件到已有tar归档文件末尾

-t	 --list	追出已有tar文档文件文件的内容

-u 	--update	将比tar归档文件中已有的同名文件新的文件追加到该tar归档文件中

-x --extract从已有tar归档文件中提取文件

options

选项				描述

-c dir			切换到指定目录

-f file			输出结果到文件或设备file

-j					将输出重定向给bzip2命令来压缩内容

-p				保留所用文件权限

-v				在处理文件时显示文件

-z				将输出重定向给gzip命令来压缩内容

tar -cvf test.tar	test/	test2/

创建一个归档文件test.tat包含test和test2目录内容

tar -tf test.tar

列出文件test.tar的内容（但并不提取）

tar -xvf	test.tar

提取test.tar的内容

.tgz是gzip压缩过的tar文件，可以用命令tar -zxvf filename.tgz 来解压



## 5理解shell

bash

 在当前bash下创建子bash

-c string 

从string 中读取命令并进行处理

-i

启动一个能够接收用户输入的交互bash

-l

以登陆shell的形式启动

-r

启动一个受限shell，用户会被限制在默认目录

-s

从标准输入中读取命令



一行依次运行多个命令，用; 分开

要想组成进程列表，需要在外边加上（），尽管加括号后看起来没什么不同，但是加括号，让命令列表变成了进程列表，生成了一个子shell来执行对应的命令。

另一种{ comand;},不创建子shell

echo  $BASH_SUBSHELL,如果返回1或者更大的数字，就说明有子shell





sleep 10

让进程等待（休眠），10秒，然后返回CLI提示符

sleep 10&

把sleep命令置于后台模式。输出后台作业号和后台作业的进程ID



jobs

显示出当前运行在后台模式中的所有用户进程（作业）



(sleep 10;echo 2;sleep 10)&; 

后台作业



协程

coproc sleep 10

在后台生成一个shell，并在这个shell中执行命令。

coproc My_Job  { sleep 10; }

通过拓展语法，重命名协程名字为 My_Job，但是{（}）前（后），要加空格，并且命令要以；结尾





### 5.3理解shell的内建命令

外部命令，也成为文件系统命令。是存在于bash shell 之外的程序。它们并不是shell程序的一部分。外部命令通常位于/bin、/usr/bin、/sbin或/usr/sbin中。

ps就是一个外部命令，可以用which，type命令找到。



当外部命令执行时，会创建一个子进程，这种操作称为衍生（forking）。

输入ps -f，父进程发出外部命令：ps -f，衍生子进程，子进程执行外部命令ps  -f



内建命令

不需要子进程来执行，效率更高，执行速度更快。

type -a echo

如果有两种实现方式，则显示每个命令的两种实现。注意which只显示出来外部命令文件，对于多种实现命令，如果需要使用外部命令，可以直接输入对应文件 比如/bin/pwd



history

内建命令，显示最近用过的命令列表。默认保存1000条，可以通过修改名为HISTSIZE的环境变量修改。



!!

输入!!唤起并重用历史列表中最近使用的命令，然后执行。

命令历史记录放在隐藏文件.bash_history中，它位于用户的主目录中，并且bash的历史记录先是存放在内存当中，当shell退出时才被写入到历史文件中。

history -a

强制写入历史文件。

history -n

使用history -a多个终端时在一个终端中强制写入其他不会更新，要想更新使用



命令别名

alias -p

查看系统中命令的别名





alias li='ls -li'

创建自己的别名，但是只在它所被定义的shell进程中才有效，要想都有效之后介绍。



## 6使用Linux环境变量

bash shell用一个叫做环境变量的特性来存储有关shell会话和工作环境的信息。

- 全局环境变量
- 局部环境变量



全局环境变量

对于shell会话和所有生成的子shell都可见，局部变量只对创建它们的shell可见。

printenv 或者env

查看全局变量



printenv    HOME

查看个别变量，env不可以使用，或者

echo $HOME

还能让变量作为命令行参数。



局部环境变量

set

显示全局变量，局部变量以及用户定义变量，还会按照字母顺序对结果排序，env和printenv不会排序，也不会输出局部变量和用户定义变量，但是env又一个printenv没有的功能，这使它要更有用一些。



设置局部用户定义变量

echo $my_variable

my_variable=Hello

echo $my_variable

赋值含有空格的字符串时，要加""

窍门：所有环境变量都使用大写字母，这是bash shell的标准惯例，如果是自己创建的局部变脸或者是shell脚本变量请使用小写字母，变量名区分大小写，涉及到哟哦难怪乎局部变量时坚持使用小写字母，这能够有效的避免重新定义系统环境变量可能带来的灾难。

my_variable = "Hello word"

错误，变量名、等号和值之间没有空格，如果加上bash shell会把值当作一个单独的命令





设置全局环境变量

在设定全局环境变量的进程所创建的子进程中，该变量都是可见的。创建全局环境变量的方法是：先创建一个局部环境变量，然后再把它导出到全局环境中。

my_variable="Hello word"

export my_variable

在定义并导出环境变量后，子shell能正确显示出变量my_variable的值。

**修改子shell中全局变量并不会影响到父shell中该变量的值。**



删除环境变量

unset my_variable

不加$



但是处理全局变量时，子进程中删除一个全局环境变量，只对子进程有效，该全局环境变量在父进程中依然可用，和修改变量一样，在子进程中删除全局变量后无法将效果映射到父shell中。



涉及到环境变量名时什么时候加$,什么时候不加，关键点：如果要用到变量，使用$, 如果要操作变量，不使用$,这条规则的一个例外就是使用printenv显示某个变量的值。



默认的shell环境变量

略



设置PATH环境变量

当在shell命令行界面输入一个外部命令时，shell必须搜索系统来找到对应的程序，PATH环境变量定义了用于进行命令和程序查找的目录 。



echo $PATH

冒号分开，如果命令或程序不在PATH变量中，那么如果不使用绝对路径，shell是没办法找到的。

PATH中各个目录是由冒号分隔。

添加环境变量

PATH=$PATH:/home/christine/Scripts

如果希望子shell也能找到程序的位置，一定要将修改后的PATH环境变量导出



程序员常见的办法是将单点符也加入PATH环境变量中。

PATH=$PATH:.



定位系统环境变量

当登入Linux启动一个bash shell时，默认情况下会在几个文件中查找命令，这些文件叫做启动文件或环境文件。bash检查的启动文件取决于你启动bash shell的方式。启动bash shell有三种方式

**登录时作为默认登录shell**

**作为非登录shell的交互式shell**

**作为运行脚本的非交互式shell**







