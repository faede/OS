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





