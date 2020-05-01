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



## 5.理解shell

