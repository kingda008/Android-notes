# 网卡清空缓存命令_清除linux缓存命令

命令

#sync

#echo 3 > /proc/sys/vm/drop_caches

查看内存情况：

# more /proc/meminfo

Kernels 2.6.16 and newer provide a mechanism to have the kernel drop the page cache and/or inode and dentry caches on command, which can help free up a lot of memory. Now you can throw away that script that allocated a ton of memory just to get rid of the cache...

To use /proc/sys/vm/drop_caches, just echo a number to it.

To free pagecache:

# echo 1 > /proc/sys/vm/drop_caches

To free dentries and inodes:

# echo 2 > /proc/sys/vm/drop_caches

To free pagecache, dentries and inodes:

echo 3 > /proc/sys/vm/drop_caches

This is a non-destructive operation and will only free things that are completely unused. Dirty objects will continue to be in use until written out to disk and are not freeable. If you run "sync" first to flush them out to disk, these drop operations will tend to free more memory.
————————————————
版权声明：本文为CSDN博主「weixin_39524147」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_39524147/article/details/111815095



setSoLinger是当socket在关闭时对当前未发送给对方数据的一种控制方式。

1、setSoLinger(true, 0)，表示不管当前有没有还未发送给对方的数据就直接关闭链接。

2、setSoLinger(true, 1)，表示等待1秒如果还有未发送给对方的数据也直接关闭链接。

3、setSoLinger(false, -1)，表示无论如何都等未发送给对方的数据发送完毕才按照4次挥手的过程正常关闭链接。



由此得出以下结论：

在客户端或者服务端通过socket.shutdownOutput()都是单向关闭的，即关闭客户端的输出流并不会关闭服务端的输出流，所以是一种单方向的关闭流；
通过socket.shutdownOutput()关闭输出流，但socket仍然是连接状态，连接并未关闭
如果直接关闭输入或者输出流，即：in.close()或者out.close()，会直接关闭socket
————————————————
版权声明：本文为CSDN博主「zhuoni2013」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zhuoni2013/article/details/56494681





Blog：http://blog.csdn.net/c359719435/

Copyright 2013 阿冬哥 http://blog.csdn.net/c359719435/

使用以及转载请注明出处

1 设置socket tcp缓冲区大小的疑惑

疑惑1：通过setsockopt设置SO_SNDBUF、SO_RCVBUF这连个默认缓冲区的值，再用getsockopt获取设置的值，发现返回值是设置值的两倍。为什么？

通过网上查找，看到linux的内核代码/usr/src/linux-2.6.13.2/net/core/sock.c，找到sock_setsockopt这个函数的这段代码：

case SO_SNDBUF:

/* Don't error on this BSD doesn't and if you think

about it this is right. Otherwise apps have to

play 'guess the biggest size' games. RCVBUF/SNDBUF

are treated in BSD as hints */

if (val > sysctl_wmem_max)//val是我们想设置的缓冲区大小的值

val = sysctl_wmem_max;//大于最大值，则val值设置成最大值

sk->sk_userlocks |= SOCK_SNDBUF_LOCK;

if ((val * ) < SOCK_MIN_SNDBUF)//val的两倍小于最小值，则设置成最小值

sk->sk_sndbuf = SOCK_MIN_SNDBUF;

else

sk->sk_sndbuf = val * ;//val的两倍大于最小值，则设置成val值的两倍

/*

\* Wake up sending tasks if we

\* upped the value.

*/

sk->sk_write_space(sk);

break;

case SO_RCVBUF:

/* Don't error on this BSD doesn't and if you think

about it this is right. Otherwise apps have to

play 'guess the biggest size' games. RCVBUF/SNDBUF

are treated in BSD as hints */

if (val > sysctl_rmem_max)

val = sysctl_rmem_max;

sk->sk_userlocks |= SOCK_RCVBUF_LOCK;

/* FIXME: is this lower bound the right one? */

if ((val * ) < SOCK_MIN_RCVBUF)

sk->sk_rcvbuf = SOCK_MIN_RCVBUF;

else

sk->sk_rcvbuf = val * ;

break;

从上述代码可以看出：(1)当设置的值val > 最大值sysctl_wmem_max，则设置为最大值的2倍：2*sysctl_wmem_max；

(2)当设置的值的两倍val*2 > 最小值，则设置成最小值：SOCK_MIN_SNDBUF；

(3)当设置的值val < 最大值sysctl_wmem_max，且 val*2 > SOCK_MIN_SNDBUF， 则设置成2*val。

查看linux 手册：

SO_RCVBUF：

Sets or gets the maximum socket receive buffer in bytes.

The kernel doubles this value (to allow space for bookkeeping overhead) when it is set using setsockopt(),

and this doubled value is returned by getsockopt().

The default value is set by the /proc/sys/net/core/rmem_default file,

and the maximum allowed value is set by the /proc/sys/net/core/rmem_max file.

The minimum (doubled) value for this option is .

查看我的主机Linux 2.6.6 ：/proc/sys/net/core/rmem_max：

4194304 //4M

查看/proc/sys/net/core/wmem_max：

8388608  //8M

所以，能设置的接收缓冲区的最大值是8M，发送缓冲区的最大值是16M。

疑惑2：为什么要有2倍这样的一个内核设置呢？我的理解是，用户在设置这个值的时候，可能只考虑到数据的大小，没有考虑数据封包的字节开销。所以将这个值设置成两倍。

注：overhead，在计算机网络的帧结构中，除了有用数据以外，还有很多控制信息，这些控制信息用来保证通信的完成。这些控制信息被称作系统开销。

2 tcp缓冲区大小的默认值

建立一个socket，通过getsockopt获取缓冲区的值如下：

发送缓冲区大小：SNDBufSize = 16384

接收缓冲区大小：RCVBufSize = 87380

疑惑3：linux手册中，接收缓冲区的默认值保存在/proc/sys/net/core/rmem_default，发送缓冲区保存在/proc/sys/net/core/wmem_default。

[root@cfs_netstorage core]# cat /proc/sys/net/core/rmem_default

1048576

[root@cfs_netstorage core]# cat /proc/sys/net/core/wmem_default

512488

可知，接收缓冲区的默认值是：1048576，1M。发送缓冲区的默认值是：512488，512K。为什么建立一个socket时得到的默认值是87380、16384？？？

进一步查阅资料发现， linux下socket缓冲区大小的默认值在/proc虚拟文件系统中有配置。分别在一下两个文件中：

/proc/sys/net/ipv4/tcp_wmem

[root@cfs_netstorage core]# cat /proc/sys/net/ipv4/tcp_wmem

4096   16384  131072  //第一个表示最小值，第二个表示默认值，第三个表示最大值。

/proc/sys/net/ipv4/tcp_rmem

[root@cfs_netstorage core]# cat /proc/sys/net/ipv4/tcp_rmem

4096   87380  174760

由此可见，新建socket，选取的默认值都是从这两个文件中读取的。可以通过更改这两个文件中的值进行调优，但是最可靠的方法还是在程序中调用setsockopt进行设置。通过setsockopt的设置，能设置的接收缓冲区的最大值是8M，发送缓冲区的最大值是16M(Linux 2.6.6中)。

另一文章中简单介绍：http://www.linuxidc.com/Linux/2012-08/68874.htm

\1. tcp 收发缓冲区默认值

[root@ www.linuxidc.com]# cat /proc/sys/net/ipv4/tcp_rmem

4096   87380  4161536

87380  ：tcp接收缓冲区的默认值

[root@ www.linuxidc.com]# cat /proc/sys/net/ipv4/tcp_wmem

4096   16384  4161536

16384  ： tcp 发送缓冲区的默认值

\2. tcp 或udp收发缓冲区最大值

[root@ www.linuxidc.com]# cat /proc/sys/net/core/rmem_max

131071

131071：tcp 或 udp 接收缓冲区最大可设置值的一半。

也就是说调用 setsockopt(s, SOL_SOCKET, SO_RCVBUF, &rcv_size, &optlen);  时rcv_size 如果超过 131071，那么

getsockopt(s, SOL_SOCKET, SO_RCVBUF, &rcv_size, &optlen); 去到的值就等于 131071 * 2 = 262142

[root@ www.linuxidc.com]# cat /proc/sys/net/core/wmem_max

131071

131071：tcp 或 udp 发送缓冲区最大可设置值得一半。

跟上面同一个道理

\3. udp收发缓冲区默认值

[root@ www.linuxidc.com]# cat /proc/sys/net/core/rmem_default

111616：udp接收缓冲区的默认值

[root@ www.linuxidc.com]# cat /proc/sys/net/core/wmem_default

111616

111616：udp发送缓冲区的默认值

\4. tcp 或udp收发缓冲区最小值

tcp 或udp接收缓冲区的最小值为 256 bytes，由内核的宏决定；

tcp 或udp发送缓冲区的最小值为 2048 bytes，由内核的宏决定

socket tcp缓冲区大小的默认值、最大值

Author:阿冬哥 Created:2013-4-17 Blog:http://blog.csdn.net/c359719435/ Copyright 2013 阿冬哥 http://blog.cs ...

TCP缓冲区大小及限制

这个问题在前面有的部分已经涉及,这里在重新总结下.主要参考UNIX网络编程. (1)数据报大小IPv4的数据报最大大小是65535字节,包括IPv4首部.因为首部中说明大小的字段为16位.IPv6的数 ...

TCP&sol;IP学习(四)TCP缓冲区大小及限制(转)

链接来自:http://blog.csdn.net/ysu108/article/details/7764461 这个问题在前面有的部分已经涉及,这里在重新总结下.主要参考UNIX网络编程. (1)数 ...

&lbrack;转&rsqb;linux tcp&sol;ip调优

LINUX tcp/ip性能调优 On 2011年03月15日, in linux, tips, by netoearth 在TCP/IP协议中,TCP协议提供可靠的连接服务,采用三次握手建立一个连接 ...

Linux网卡调优篇-禁用ipv6与优化socket缓冲区大小

Linux网卡调优篇-禁用ipv6与优化socket缓冲区大小 作者:尹正杰 版权声明:原创作品,谢绝转载!否则将追究法律责任.  一般在内网环境中,我们几乎是用不到IPV6,因此我们没有必要把多不 ...

udp之关于linux udp收发包缓冲区大小

1.修订单个socket的缓冲区大小:通过setsockopt使用SO_RCVBUF来设置接收缓冲区,该参数在设置的时候不会与rmem_max进行对比校验,但是如果设置的大小超过rmem_max的话, ...

&lbrack;cmd&rsqb;如何设置 Windows 默认命令行窗口大小和缓冲区大小

Windows 命令行 cmd 窗口系统默认的大小(80*40)对于现在的屏幕配置已经跟不上时代了,我们总是要把它改大些,而且缓冲区大小也想改得大大的.单纯的为当前的 Windows 命令行窗口修改显 ...

如何设置 Windows 默认命令行窗口大小和缓冲区大小

关键字: 命令行不能全屏 命令行最大化只有一半屏幕 命令行 字体 背景 颜色 解决方案:http://unmi.cc/save-windows-command-size/ 简要说明: win+r,输入 ...

Redis 的 maxmemory 和 dbnum 默认值都是多少？对于最大值会有限制吗？

一.Redis 的默认配置 了解 Redis 的都知道,Redis 服务器状态有很多可配置的默认值. 例如:数据库数量,最大可用内存,AOF 持久化相关配置和 RDB 持久化相关配置等等.我相信,关于 ...

随机推荐

Python学习教程&lpar;learning Python&rpar;--1&period;3 Python数据输入

多数应用程序都有数据输入语句,用于读入数据,和用户进行交互,在Python语言里,可以通过raw_input函数实现数据的从键盘读入数据操作. 基本语法结构:raw_input(prompt) 通常p ...

RHEL6中ulimit的nproc限制

http://blog.csdn.net/kumu_linux/article/details/8301760 当前shell下更改用户可打开进程数 修改limits.conf配置文件生效 [root ...

ORACLE ORDER BY用法总结

order by后面的形式却比较新颖(对于我来说哦),以前从来没看过这种用法,就想记下来,正好总结一下ORDER BY的知识. 1.ORDER BY 中关于NULL的处理 缺省处理,Oracle在Or ...

如何选择版本控制系统 ---为什么选择Git版本控制系统

版本控制系统 "代码"作为软件研发的核心产物,在整个开发周期都在递增,不断合入新需求以及解决bug的新patch,这就需要有一款系统,能够存储.追踪文件的修改历史,记录多个版本的开 ...

primary库新增数据文件后，standby库无法创建文件并终止数据同步

主库是RAC环境,使用asm存放数据文件,备库是操作系统本地文件系统存放数据文件.在主库执行以下操作: SQL> alter tablespace ysdv add datafile '+dat ...

audio video 控制播放和停止

...

Java学习笔记11(this，super)

this在构造方法间的使用, public class Person { private String name; private int age; public Person() { //this( ...

08-03 java 继承

继承格式,优缺点,概述: /* 继承概述: 把多个类中相同的内容给提取出来定义到一个类中. 如何实现继承呢? Java提供了关键字:extends 格式: class 子类名 extends 父类名 ...

验证组件——FluentValidation

FluentValidation FluentValidation是与ASP.NET DataAnnotataion Attribute验证实体不同的数据验证组件,提供了将实体与验证分离开 ...

【MySQL笔记】用户管理

1.账户管理 1.1登录和退出MySQL服务器 MySQL –hhostname|hostIP –P port –u username –p[password] databaseName –e &qu ...

相关资源：[*linux**tcp*keepalive存活代码*设置*_*tcp*keepalive-C代码类资源](https://download.csdn.net/download/glen30/11292916?spm=1001.2101.3001.5697)









