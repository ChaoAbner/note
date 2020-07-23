这么多年来，数据压缩对我们来说是非常有用的。无论是在邮件中发送的图片用的zip文件还是在服务器压缩数据文件，我们都可以让下载更容易或者有效 的节约磁盘空间。某些压缩格式有时允许我们以60％的比率（甚至更高）压缩文件。下面我将会给大家演示如何用这些命令在Linux下面去压缩文件或者目 录。我们将学习zip, tar, tar.gz和tar.bz2等压缩格式的基本用法。这几个是在Linux里面常用的压缩格式。

 

在我们探究这些用法之前，我想先跟大家分享一下使用不同压缩格式的经验。当然，我这里讲到的只是其中的一些用法，除我讲到的之外，他们还有更多的地 方值得我们探讨。我已经意识到我需要了解两到三种压缩格式，才能更好的使用他们。zip格式是第一个需要了解的格式。因为它实际上已成为压缩文件的标准选 择，而且它在windows上也能使用。我经常用zip格式压缩那些需要共享给windows用户的文件。如果只是共享给linux用户或者Mac用户， 那我偏向于选择tar.gz格式。

 

ZIP

zip可能是目前使用得最多的文档压缩格式。它最大的优点就是在不同的操作系统平台，比如Linux， Windows以及Mac OS，上使用。缺点就是支持的压缩率不是很高，而tar.gz和tar.gz2在压缩率方面做得非常好。闲话少说，我们步入正题吧：

我们可以使用下列的命令压缩一个目录：

# zip -r archive_name.zip directory_to_compress

 

下面是如果解压一个zip文档：

# unzip archive_name.zip

 

TAR

Tar是在Linux中使用得非常广泛的文档打包格式。它的好处就是它只消耗非常少的CPU以及时间去打包文件，他仅仅只是一个打包工具，并不负责压缩。下面是如何打包一个目录：

# tar -cvf archive_name.tar directory_to_compress

 

如何解包：

# tar -xvf archive_name.tar.gz

 

上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：

# tar -xvf archive_name.tar -C /tmp/extract_here/

 

TAR.GZ

这种格式是我使用得最多的压缩格式。它在压缩时不会占用太多CPU的，而且可以得到一个非常理想的压缩率。使用下面这种格式去压缩一个目录：

# tar -zcvf archive_name.tar.gz directory_to_compress

 

解压缩：

# tar -zxvf archive_name.tar.gz

 

上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：

# tar -zxvf archive_name.tar.gz -C /tmp/extract_here/

 

TAR.BZ2

这种压缩格式是我们提到的所有方式中压缩率最好的。当然，这也就意味着，它比前面的方式要占用更多的CPU与时间。这个就是你如何使用tar.bz2进行压缩。

# tar -jcvf archive_name.tar.bz2 directory_to_compress

 

上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：

# tar -jxvf archive_name.tar.bz2 -C /tmp/extract_here/

 

数据压缩是非常有用的，尤其是对于备份来说。所以，你现在应该考虑在你的备份脚本中使用你在这里学到的压缩方式备份你基本的规则文件以减小你备份文件的大小。

 

过段时间之后，你就会意识到，在压缩率与CPU占用时间上会有一个平衡，你也要学会如何去权衡什么时候你需要一个快但是压缩率低，什么时候需要一个压缩率高但是CPU点用高的压缩方式，然后你才能避免无谓的空间与时间。

 

来源：http://www.simplehelp.net/2008/12/15/how-to-create-and-extract-zip-tar-targz-and-tarbz2-files-in-linux/

 

 

如果tar不支持j这个参数就先用 
bzip2 -d xxx.tar.bz2 
把它解压成.tar文件，然后再用 
tar xvf xxx.tar 
拆包。
压缩解压 
linux下怎么解后缀名是gzip的文件？ 
1.以.a为扩展名的文件: 
#tar xv file.a 
2.以.z为扩展名的文件: 
#uncompress file.Z 
3.以.gz为扩展名的文件: 
#gunzip file.gz 
4.以.bz2为扩展名的文件: 
#bunzip2 file.bz2 
5.以.tar.Z为扩展名的文件: 
#tar xvZf file.tar.Z  
或 #compress -dc file.tar.Z | tar xvf - 
6.以.tar.gz/.tgz为扩展名的文件: 
#tar xvzf file.tar.gz  
或 gzip -dc file.tar.gz | tar xvf - 
7.以.tar.bz2为扩展名的文件: 
#tar xvIf file.tar.bz2  
或 bzip2 -dc file.tar.bz2 | xvf - 
8.以.cpio.gz/.cgz为扩展名的文件: 
#gzip -dc file.cgz | cpio -div 
9.以.cpio/cpio为扩展名的文件: 
#cpio -div file.cpio  
或cpio -divc file.cpio 
10.以.rpm为扩展名的文件安装: 
#rpm -i file.rpm 
11.以.rpm为扩展名的文件解压缩： 
#rpm2cpio file.rpm | cpio -div 
12.以.deb为扩展名的文件安装： 
#dpkg -i file.deb 
13.以.deb为扩展名的文件解压缩: 
#dpkg-deb --fsys-tarfile file.deb | tar xvf - ar p  
file.deb data.tar.gz | tar xvzf - 
14.以.zip为扩展名的文件: 
#unzip file.zip 
在linux下解压Winzip格式的文件 
　　要是装了jdk的话，可以用jar命令；还可以使用unzip命令。 
直接解压.tar.gz文件 
　　xxxx.tar.gz文件使用tar带zxvf参数，可以一次解压开。XXXX为文件名。 例如： 
$tar zxvf xxxx.tar.gz 各种压缩文件的解压（安装方法）

文件扩展名 解压（安装方法）

.a ar xv file.a 
.Z uncompress file.Z 
.gz gunzip file.gz 
.bz2 bunzip2 file.bz2 
.tar.Z tar xvZf file.tar.Z 
compress -dc file.tar.Z | tar xvf - 
.tar.gz/.tgz tar xvzf file.tar.gz 
gzip -dc file.tar.gz | tar xvf - 
.tar.bz2 tar xvIf file.tar.bz2 
bzip2 -dc file.tar.bz2 | xvf - 
.cpio.gz/.cgz gzip -dc file.cgz | cpio -div 
.cpio/cpio cpio -div file.cpio 
cpio -divc file.cpio 
.rpm/install rpm -i file.rpm 
.rpm/extract rpm2cpio file.rpm | cpio -div 
.deb/install dpkg -i file.deb 
.deb/exrtact dpkg-deb --fsys-tarfile file.deb | tar xvf - 
ar p file.deb data.tar.gz | tar xvzf - 
.zip unzip file.zip 

bzip2 -d myfile.tar.bz2 | tar xvf

tar xvfz myfile.tar.bz2

x 是解压 
v 是复杂输出 
f 是指定文件 
z gz格式

gzip 
gzip[选项]要压缩（或解压缩）的文件名 
-c将输出写到标准输出上，并保留原有文件。 
-d将压缩文件压缩。 
-l对每个压缩文件，显示下列字段：压缩文件的大小，未压缩文件的大小、压缩比、未压缩文件的名字 
-r递归式地查找指定目录并压缩或压缩其中的所有文件。 
-t测试压缩文件是正完整。 
-v对每一个压缩和解压缩的文件，显示其文件名和压缩比。 
-num-用指定的数字调整压缩的速度。 
举例： 
把/usr目录并包括它的子目录在内的全部文件做一备份，备份文件名为usr.tar 
tar cvf usr.tar /home 
把/usr 目录并包括它的子目录在内的全部文件做一备份并进行压缩，备份文件名是usr.tar.gz 
tar czvf usr.tar.gz /usr 
压缩一组文件，文件的后缀为tar.gz 
#tar cvf back.tar /back/ 
#gzip -q back.tar 
or 
#tar cvfz back.tar.gz /back/ 
释放一个后缀为tar.gz的文件。 
#tar zxvf back.tar.gz 
#gzip back.tar.gz 
#tar xvf back.tar

tar的使用方法：

1：压缩一组文件为tar.gz后缀 
tar cvf backup.tar /etc 
或gzip -q backup.tar.gz

2:释放一个后缀为tar.gz的文件 
gunzip backup.tar.gz 
或tar xvf backup.tar

3:用一个命令完成压缩 
tar cvf -/etc | gzip -qc >; backup.tar.gz

4:用一个命令完成释放 
gunzip -c backup.tar.gz | tar xvf -

5:如何解开ta.Z的文件 
tar xvfz backup.tar.Z 
或uncompress backup.tar.Z 
tar xvf backup.tar

6:如何解开.tgz文件 
gunzip backup.tgz

7:如何压缩和解压缩.bz2的包 
bzip2 /etc/smb.conf 这将压缩文件smb.conf成smb.conf.bz2 
bunzip2 /etc/smb.conf.bz2 在当前目录下还原smb.conf.bz2为smb.conf
