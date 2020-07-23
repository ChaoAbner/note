### 1. ls — List

ls会列举出当前工作目录的内容（文件或文件夹）。

![img](https:////upload-images.jianshu.io/upload_images/1712440-d16f09f5d9f399c1.png?imageMogr2/auto-orient/strip|imageView2/2/w/689/format/webp)

ls命令演示

### 2.mkdir — Make Directory

mkdir 用于新建一个新目录

![img](https:////upload-images.jianshu.io/upload_images/1712440-71ff701f90ca847c.png?imageMogr2/auto-orient/strip|imageView2/2/w/84/format/webp)

执行mkdir命令创建相应的文件夹

### 3.pwd — Print Working Directory

显示当前工作目录

![img](https:////upload-images.jianshu.io/upload_images/1712440-95038b266cb1e3ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/302/format/webp)

显示当前工作目录

### 4.cd — Change Directory

切换文件路径，cd 将给定的文件夹（或目录）设置成当前工作目录。

![img](https:////upload-images.jianshu.io/upload_images/1712440-b1d50b5bcbbac513.png?imageMogr2/auto-orient/strip|imageView2/2/w/262/format/webp)

切换路径到桌面

### 5.rmdir— Remove Directory

删除给定的目录。

![img](https:////upload-images.jianshu.io/upload_images/1712440-cf76967c2555a9f9.png?imageMogr2/auto-orient/strip|imageView2/2/w/359/format/webp)

创建的文件夹会被删除

### 6. rm— Remove

rm 会删除给定的文件

![img](https:////upload-images.jianshu.io/upload_images/1712440-acc84ffe7939f46b.png?imageMogr2/auto-orient/strip|imageView2/2/w/389/format/webp)

删除某个文件

### 7. cp— Copy

cp 命令对文件进行复制

![img](https:////upload-images.jianshu.io/upload_images/1712440-74dca6a2c9215a75.png?imageMogr2/auto-orient/strip|imageView2/2/w/404/format/webp)

文件的复制

### 8. mv— Move

mv 命令对文件或文件夹进行移动，如果文件或文件夹存在于当前工作目录，还可以对文件或文件夹进行重命名。

![img](https:////upload-images.jianshu.io/upload_images/1712440-794d4345347f6b68.png?imageMogr2/auto-orient/strip|imageView2/2/w/460/format/webp)

移动文件

### 9. cat— concatenate and print files

cat 用于在标准输出（监控器或屏幕）上查看文件内容

![img](https:////upload-images.jianshu.io/upload_images/1712440-b4db478db25d6643.png?imageMogr2/auto-orient/strip|imageView2/2/w/872/format/webp)

查看某个文件内容

### 10. tail — print TAIL(from last)

ail 默认在标准输出上显示给定文件的最后10行内容，可以使用tail -n N 指定在标准输出上显示文件的最后N行内容。

![img](https:////upload-images.jianshu.io/upload_images/1712440-22be8f6ffe6bca0c.png?imageMogr2/auto-orient/strip|imageView2/2/w/873/format/webp)

显示文件内容

### 11.less — print LESS

less 按页或按窗口打印文件内容。在查看包含大量文本数据的大文件时是非常有用和高效的。你可以使用Ctrl+F向前翻页，Ctrl+B向后翻页。

![img](https:////upload-images.jianshu.io/upload_images/1712440-eb3dc1cb618b3c3d.png?imageMogr2/auto-orient/strip|imageView2/2/w/888/format/webp)

打印文件内容

### 12.grep

grep 在给定的文件中搜寻指定的字符串。grep -i “” 在搜寻时会忽略字符串的大小写，而grep -r “” 则会在当前工作目录的文件中递归搜寻指定的字符串。

![img](https:////upload-images.jianshu.io/upload_images/1712440-8f2d382da7ec99e0.png?imageMogr2/auto-orient/strip|imageView2/2/w/534/format/webp)

查找相关内容

### 13.find

这个命令会在给定位置搜寻与条件匹配的文件。你可以使用find -name 的-name选项来进行区分大小写的搜寻，find -iname 来进行不区分大小写的搜寻。

![img](https:////upload-images.jianshu.io/upload_images/1712440-7668c5809f876d83.png?imageMogr2/auto-orient/strip|imageView2/2/w/580/format/webp)

三种使用示例

### 14.tar

tar命令能创建、查看和提取tar压缩文件。tar -cvf 是创建对应压缩文件，tar -tvf 来查看对应压缩文件，tar -xvf 来提取对应压缩文件。

![img](https:////upload-images.jianshu.io/upload_images/1712440-5bad1e453ac32975.png?imageMogr2/auto-orient/strip|imageView2/2/w/620/format/webp)

创建压缩文件

![img](https:////upload-images.jianshu.io/upload_images/1712440-771921ded8b195bf.png?imageMogr2/auto-orient/strip|imageView2/2/w/759/format/webp)

查看压缩文件

### 15. gzip

gzip 命令创建和提取gzip压缩文件，还可以用gzip -d 来提取压缩文件。

![img](https:////upload-images.jianshu.io/upload_images/1712440-1eb08f8ab6d421de.png?imageMogr2/auto-orient/strip|imageView2/2/w/563/format/webp)

生成gzip文件

### 16. unzip

unzip 对gzip文档进行解压。在解压之前，可以使用unzip -l 命令查看文件内容。

![img](https:////upload-images.jianshu.io/upload_images/1712440-a9faf5a2c8d75127.png?imageMogr2/auto-orient/strip|imageView2/2/w/794/format/webp)

解压缩文件

### 17.help

help会在终端列出所有可用的命令,可以使用任何命令的-h或-help选项来查看该命令的具体用法。图就省略啦，会有详细列表显示出来的。

### 18.whatis — What is this command

whatis 会用单行来描述给定的命令，就是解释当前命令。

![img](https:////upload-images.jianshu.io/upload_images/1712440-787cb8a6826556fe.png?imageMogr2/auto-orient/strip|imageView2/2/w/889/format/webp)

cd的描述

### 19.exit

exit用于结束当前的终端会话。

![img](https:////upload-images.jianshu.io/upload_images/1712440-1f17065446c1fd63.png?imageMogr2/auto-orient/strip|imageView2/2/w/671/format/webp)

结束当前终端会话

### 20.ping

ping 通过发送数据包ping远程主机(服务器)，常用与检测网络连接和服务器状态。

![img](https:////upload-images.jianshu.io/upload_images/1712440-37638b8ce2acfff3.png?imageMogr2/auto-orient/strip|imageView2/2/w/770/format/webp)

ping百度演示

### 21.who — Who Is logged in

who能列出当前登录的用户名。

![img](https:////upload-images.jianshu.io/upload_images/1712440-b92247235b23e608.png?imageMogr2/auto-orient/strip|imageView2/2/w/456/format/webp)

当前用户列表

### 22.su — Switch User

su 用于切换不同的用户。即使没有使用密码，超级用户也能切换到其它用户。

![img](https:////upload-images.jianshu.io/upload_images/1712440-4fa4eadc3d79cc19.png?imageMogr2/auto-orient/strip|imageView2/2/w/383/format/webp)

切换当前用户

### 23.uname

uname会显示出关于系统的重要信息，如内核名称、主机名、内核版本、处理机类型等等，使用uname -a可以查看所有信息。

![img](https:////upload-images.jianshu.io/upload_images/1712440-c6a2d4392cc0f801.png?imageMogr2/auto-orient/strip|imageView2/2/w/883/format/webp)

系统信息

### 24.df — Disk space Free

df查看文件系统中磁盘的使用情况–硬盘已用和可用的存储空间以及其它存储设备。你可以使用df -h将结果以人类可读的方式显示。

![img](https:////upload-images.jianshu.io/upload_images/1712440-52e99951ebb56d50.png?imageMogr2/auto-orient/strip|imageView2/2/w/885/format/webp)

磁盘使用情况

### 25.ps — ProcesseS

ps显示系统的运行进程。

![img](https:////upload-images.jianshu.io/upload_images/1712440-e2bbe9821ceb0459.png?imageMogr2/auto-orient/strip|imageView2/2/w/430/format/webp)

系统进程

### 26.top — Top processes

top命令会默认按照CPU的占用情况，显示占用量较大的进程,可以使用top -u 查看某个用户的CPU使用排名情况。

![img](https:////upload-images.jianshu.io/upload_images/1712440-753bac6c23e50395.png?imageMogr2/auto-orient/strip|imageView2/2/w/939/format/webp)

cpu占用情况

### 27. shutdown

shutdown用于关闭计算机，而shutdown -r用于重启计算机。这个我就不试了......

