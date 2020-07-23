# Hash算法，MD5，SHA

让我们先来了解一些基本知识，了解hash。

Hash，一般翻译做“散列”，也有直接音译为”哈希”的，就是把任意长度的输入（又叫做预映射，pre-image），通过散列算法，变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，而不可能从散列值来唯一的确定输入值。

简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。



HASH主要用于信息安全领域中加密算法，他把一些不同长度的信息转化成杂乱的128位的编码里,叫做HASH值.也可以说，hash就是找到一种数据内容和数据存放地址之间的映射关系

了解了hash基本定义，就不能不提到一些著名的hash算法，MD5和SHA1可以说是目前应用最广泛的Hash算法，而它们都是以MD4为基础设计的。那么他们都是什么意思呢？



这里简单说一下：

**1)MD4**

MD4(RFC1320)是MIT的RonaldL.Rivest在1990年设计的，MD是MessageDigest的缩写。它适用在32位字长的处理器上用高速软件实现--它是基于32位操作数的位操作来实现的。

**2)MD5**

MD5(RFC1321)是Rivest于1991年对MD4的改进版本。它对输入仍以512位分组，其输出是4个32位字的级联，与MD4相同。MD5比MD4来得复杂，并且速度较之要慢一点，但更安全，在抗分析和抗差分方面表现更好。

**MD5是一种不可逆的加密算法，目前是最牢靠的加密算法之一，尚没有能够逆运算的程序被开发出来，它对应任何字符串都可以加密成一段唯一的固定长度的代码**。



那么它有什么用呢？很简单，通过它可以判断原始值是否正确（是否被更改过）。一般用于密码的加密。而我们所提供的MD5校验码就是针对安装程序的唯一对应的一段代码。你可以使用任何MD5运算器对下载的文件进行运算，运算出来的结果如果完全符合我们提供的MD5校验码，那么说明你下载的这个程序没有被中途修改过。

这个特征码有如下特性，首先它不可逆，例如我有一段秘密的文字如：”MySecretWords”，经算法变换后得到[MD5码](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3DMD5%E7%A0%81%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)(b9944e9367d2e40dd1f0c4040d4daaf7)，把这个码告诉其他人，他们根据这个[MD5码](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3DMD5%E7%A0%81%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)是没有系统的方法可以知道你原来的文字是什么的。

其次，这个码具有高度的离散性，也就是说，原信息的一点点变化就会导致MD5的巨大变化，例如”ABC”MD5(902fbdd2b1df0c4f70b4a5d23525e932)和”ABC”（多了一空格）MD5(12c774468f981a9487c30773d8093561)差别非常大，而且之间没有任何关系，也就是说产生的[MD5码](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3DMD5%E7%A0%81%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)是不可预测的。

最后由于这个码有128位那么长，所以任意信息之间具有相同MD5码的可能性非常之低，通常被认为是不可能的。

所以一般认为MD5码可以唯一地代表原信息的特征，通常用于密码的加密存储，数字签名，文件完整性验证等。



**3)SHA1及其他**

SHA1是由NISTNSA设计为同DSA一起使用的，它对长度小于264的输入，产生长度为160bit的散列值，因此抗穷举(brute-force)性更好。[SHA-1](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3DSHA-1%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)设计时基于和MD4相同原理,并且模仿了该算法。SHA-1是由美国标准技术局（NIST）颁布的国家标准，是一种应用最为广泛的[hash函数](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3Dhash%E5%87%BD%E6%95%B0%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)算法，也是目前最先进的加密技术，被政府部门和[私营业主](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%E7%A7%81%E8%90%A5%E4%B8%9A%E4%B8%BB%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)用来处理敏感的信息。而SHA-1基于MD5，MD5又基于MD4。

论坛里提供的[系统镜像文件](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%E7%B3%BB%E7%BB%9F%E9%95%9C%E5%83%8F%E6%96%87%E4%BB%B6%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)的hash也就是微软官方提供的SHA-1值，下载后和此值对应，就说明你下载过程中文件没有被更改，属于原版。

**什么是CRC**

CRC的全称为CyclicRedundancyCheck，中文名称为[循环冗余校验](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%E5%BE%AA%E7%8E%AF%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)。它是一类重要的[线性分组码](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%E7%BA%BF%E6%80%A7%E5%88%86%E7%BB%84%E7%A0%81%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)，编码和解码方法简单，检错和纠错能力强，在通信领域广泛地用于实现差错控制。实际上，除数据通信外，CRC在其它很多领域也是大有用武之地的。例如我们读软盘上的文件，以及解压一个[ZIP文件](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3DZIP%E6%96%87%E4%BB%B6%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)时，偶尔会碰到“BadCRC”错误，由此它在[数据存储](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%26tn%3DSE_PcZhidaonwhc_ngpagmjz%26rsv_dl%3Dgh_pc_zhidao)方面的应用可略见一斑。



那么这些Hash算法到底有什么用呢？

Hash算法在信息安全方面的应用主要体现在以下的3个方面：

**1)文件校验**

我们比较熟悉的校验算法有奇偶校验和CRC校验，这2种校验并没有抗数据篡改的能力，它们一定程度上能检测并纠正数据传输中的信道误码，但却不能防止对数据的恶意破坏。

MD5Hash算法的”数字指纹”特性，使它成为目前应用最广泛的一种文件完整性校验和(Checksum)算法，不少Unix系统有提供计算md5checksum的命令。

**2)数字签名**

Hash算法也是现代密码体系中的一个重要组成部分。由于非对称算法的运算速度较慢，所以在数字签名协议中，单向散列函数扮演了一个重要的角色。对Hash值，又称”数字摘要”进行数字签名，在统计上可以认为与对文件本身进行数字签名是等效的。而且这样的协议还有其他的优点。



**3)鉴权协议**

如下的鉴权协议又被称作”挑战--认证模式：在传输信道是可被侦听，但不可被篡改的情况下，这是一种简单而安全的方法。