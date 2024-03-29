## 1、前言


随着互联网的发展，面对海量用户高并发业务，传统的阻塞式的服务端架构模式已经无能为力。本文（和下篇《[高性能网络编程(六)：一文读懂高性能网络编程中的线程模型](http://www.52im.net/thread-1939-1-1.html)》）旨在为大家提供有用的高性能网络编程的I/O模型概览以及网络服务进程模型的比较，以揭开设计和实现高性能网络架构的神秘面纱。

限于篇幅原因，请将本文与《[高性能网络编程(六)：一文读懂高性能网络编程中的线程模型](http://www.52im.net/thread-1939-1-1.html)》连起来读，这样会让知识更连贯。

另外，作者的其它文章《[新手入门：目前为止最透彻的的Netty高性能原理和框架架构解析](http://www.52im.net/thread-2043-1-1.html)》、《[IM开发基础知识补课(六)：数据库用NoSQL还是SQL？读这篇就够了！](http://www.52im.net/thread-2759-1-1.html)》，也值得一读，推荐一并阅读之。

## 2、关于作者


**陈彩华（caison）：**主要从事服务端开发、需求分析、系统设计、优化重构工作，主要开发语言是 Java，现任广州贝聊服务端研发工程师。

**关于广州贝聊：**

![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_1.jpg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/211612obrg1kft1zgh9ooc.jpg) 



广州市贝聊信息科技有限公司成立于2013年8月21日，是一家专注于搭建幼儿园家园共育平台的信息科技公司。

公司产品“贝聊”是中国幼儿园家长工作平台，致力于通过互联网产品及定制化解决方案，帮助幼儿园解决展示、通知、沟通等家长工作中的痛点，促进家园关系和谐。贝聊是威创股份（A股幼教第一股）、清华启迪、网易联手投资的唯一品牌。

截止目前，“贝聊”已覆盖全国31省份的5万所幼儿园及机构，注册用户超过1000万，用户次月留存率高达74%，复合增长率为18.94%，领跑全行业。



## 3、C10K问题系列文章


**本文是C10K问题系列文章中的第5篇，总目录如下：**



- 《[高性能网络编程(一)：单台服务器并发TCP连接数到底可以有多少](http://www.52im.net/thread-561-1-1.html)》
- 《[高性能网络编程(二)：上一个10年，著名的C10K并发连接问题](http://www.52im.net/thread-566-1-1.html)》
- 《[高性能网络编程(三)：下一个10年，是时候考虑C10M并发问题了](http://www.52im.net/thread-568-1-1.html)》
- 《[高性能网络编程(四)：从C10K到C10M高性能网络应用的理论探索](http://www.52im.net/thread-578-1-1.html)》
- 《[高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型](http://www.52im.net/thread-1935-1-1.html)》（本文）
- 《[高性能网络编程(六)：一文读懂高性能网络编程中的线程模型](http://www.52im.net/thread-1939-1-1.html)》
- 《[高性能网络编程(七)：到底什么是高并发？一文即懂！](http://www.52im.net/thread-3120-1-1.html)》
- 《[高性能网络编程经典：《The C10K problem(英文)》[附件下载\]](http://www.52im.net/thread-560-1-1.html)》



## 4、互联网服务端处理网络请求的原理


**首先看看一个典型互联网服务端处理网络请求的典型过程：**

![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_1.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/211858pgsyanbk1yffennv.jpeg)



**由上图可以看到，主要处理步骤包括：** 



- 1）获取请求数据，客户端与服务器建立连接发出请求，服务器接受请求（1-3）；
- 2）构建响应，当服务器接收完请求，并在用户空间处理客户端的请求，直到构建响应完成（4）；
- 3）返回数据，服务器将已构建好的响应再通过内核空间的网络 I/O 发还给客户端（5-7）。


**设计服务端并发模型时，主要有如下两个关键点：** 



- 1）服务器如何管理连接，获取输入数据；
- 2）服务器如何处理请求。


以上两个关键点最终都与操作系统的 I/O 模型以及线程(进程)模型相关，这也是本文和下篇《高性能网络编程(六)：一文读懂高性能网络编程中的线程模型》将要介绍的内容。下面先详细介绍这I/O模型。

## 5、“I/O 模型”的基本认识


**介绍操作系统的 I/O 模型之前，先了解一下几个概念：** 



- 1）阻塞调用与非阻塞调用；
- 2）阻塞调用是指调用结果返回之前，当前线程会被挂起，调用线程只有在得到结果之后才会返回；
- 3）非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。


两者的最大区别在于被调用方在收到请求到返回结果之前的这段时间内，调用方是否一直在等待。

**阻塞**是指调用方一直在等待而且别的事情什么都不做；**非阻塞**是指调用方先去忙别的事情。

**同步处理与异步处理：**同步处理是指被调用方得到最终结果之后才返回给调用方；异步处理是指被调用方先返回应答，然后再计算调用结果，计算完最终结果后再通知并返回给调用方。

**阻塞、非阻塞和同步、异步的区别（**阻塞、非阻塞和同步、异步其实针对的对象是不一样的）**：**



- 1）阻塞、非阻塞的讨论对象是调用者；
- 2）同步、异步的讨论对象是被调用者。


**recvfrom 函数：**
recvfrom 函数(经 Socket 接收数据)，这里把它视为系统调用。

**一个输入操作通常包括两个不同的阶段：**



- 1）等待数据准备好；
- 2）从内核向进程复制数据。


对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待分组到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

实际应用程序在系统调用完成上面的 2 步操作时，调用方式的阻塞、非阻塞，操作系统在处理应用程序请求时，处理方式的同步、异步处理的不同，可以分为 5 种 I/O 模型（下面的章节将逐个展开介绍）。（参考《[UNIX网络编程卷1](http://www.52im.net/thread-1015-1-1.html)》）

## 6、I/O模型1：阻塞式 I/O 模型(blocking I/O）



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_2.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/212717yp8iwt5z8j1niw8a.jpeg) 



在阻塞式 I/O 模型中，应用程序在从调用 recvfrom 开始到它返回有数据报准备好这段时间是阻塞的，recvfrom 返回成功后，应用进程开始处理数据报。

**比喻：**一个人在钓鱼，当没鱼上钩时，就坐在岸边一直等。
**优点：**程序简单，在阻塞等待数据期间进程/线程挂起，基本不会占用 CPU 资源。
**缺点：**每个连接需要独立的进程/线程单独处理，当并发请求量大时为了维护程序，内存、线程切换开销较大，这种模型在实际生产中很少使用。

## 7、I/O模型2：非阻塞式 I/O 模型(non-blocking I/O）



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_3.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/212910wn44nrr6zp5siiuo.jpeg) 



在非阻塞式 I/O 模型中，应用程序把一个套接口设置为非阻塞，就是告诉内核，当所请求的 I/O 操作无法完成时，不要将进程睡眠。

而是返回一个错误，应用程序基于 I/O 操作函数将不断的轮询数据是否已经准备好，如果没有准备好，继续轮询，直到数据准备好为止。

**比喻：**边钓鱼边玩手机，隔会再看看有没有鱼上钩，有的话就迅速拉杆。
**优点：**不会阻塞在内核的等待数据过程，每次发起的 I/O 请求可以立即返回，不用阻塞等待，实时性较好。
**缺点：**轮询将会不断地询问内核，这将占用大量的 CPU 时间，系统资源利用率较低，所以一般 Web 服务器不使用这种 I/O 模型。

## 8、I/O模型3：I/O 复用模型(I/O multiplexing）



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_4.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/213041mtejdsoeojfjy7dd.jpeg) 



在 I/O 复用模型中，会用到 Select 或 Poll 函数或 Epoll 函数(Linux 2.6 以后的内核开始支持)，这两个函数也会使进程阻塞，但是和阻塞 I/O 有所不同。

这两个函数可以同时阻塞多个 I/O 操作，而且可以同时对多个读操作，多个写操作的 I/O 函数进行检测，直到有数据可读或可写时，才真正调用 I/O 操作函数。

**比喻：**放了一堆鱼竿，在岸边一直守着这堆鱼竿，没鱼上钩就玩手机。
**优点：**可以基于一个阻塞对象，同时在多个描述符上等待就绪，而不是使用多个线程(每个文件描述符一个线程)，这样可以大大节省系统资源。
**缺点：**当连接数较少时效率相比多线程+阻塞 I/O 模型效率较低，可能延迟更大，因为单个连接处理需要 2 次系统调用，占用时间会有增加。
众所周之，Nginx这样的高性能互联网反向代理服务器大获成功的关键就是得益于Epoll。

## 9、I/O模型4：信号驱动式 I/O 模型（signal-driven I/O)



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_5.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/213143a7n3mnxb38ybgxy3.jpeg) 



在信号驱动式 I/O 模型中，应用程序使用套接口进行信号驱动 I/O，并安装一个信号处理函数，进程继续运行并不阻塞。

当数据准备好时，进程会收到一个 SIGIO 信号，可以在信号处理函数中调用 I/O 操作函数处理数据。

**比喻：**鱼竿上系了个铃铛，当铃铛响，就知道鱼上钩，然后可以专心玩手机。
**优点：**线程并没有在等待数据时被阻塞，可以提高资源的利用率。
**缺点：**信号 I/O 在大量 IO 操作时可能会因为信号队列溢出导致没法通知。

信号驱动 I/O 尽管对于处理 UDP 套接字来说有用，即这种信号通知意味着到达一个数据报，或者返回一个异步错误。

但是，对于 TCP 而言，信号驱动的 I/O 方式近乎无用，因为导致这种通知的条件为数众多，每一个来进行判别会消耗很大资源，与前几种方式相比优势尽失。

## 10、I/O模型5：异步 I/O 模型（即AIO，全称asynchronous I/O）



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_6.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/213218wbeovsvt6g7s4zsj.jpeg)



由 POSIX 规范定义，应用程序告知内核启动某个操作，并让内核在整个操作（包括将数据从内核拷贝到应用程序的缓冲区）完成后通知应用程序。

这种模型与信号驱动模型的主要区别在于：信号驱动 I/O 是由内核通知应用程序何时启动一个 I/O 操作，而异步 I/O 模型是由内核通知应用程序 I/O 操作何时完成。

**优点：**异步 I/O 能够充分利用 DMA 特性，让 I/O 操作与计算重叠。
**缺点：**要实现真正的异步 I/O，操作系统需要做大量的工作。目前 Windows 下通过 IOCP 实现了真正的异步 I/O。

而在 Linux 系统下，Linux 2.6才引入，目前 AIO 并不完善，因此在 Linux 下实现高并发网络编程时都是以 IO 复用模型模式为主。

关于AOI的介绍，请见：《[Java新一代网络编程模型AIO原理及Linux系统AIO介绍](http://www.52im.net/thread-306-1-1.html)》。

## 11、5 种 I/O 模型总结



![高性能网络编程(五)：一文读懂高性能网络编程中的I/O模型_7.jpeg](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%E9%AB%98%E6%80%A7%E8%83%BD%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E4%B8%AD%E7%9A%84IO%E6%A8%A1%E5%9E%8B.assets/213459mmmhohhgwom24uoj.jpeg)



从上图中我们可以看出，越往后，阻塞越少，理论上效率也是最优。

这五种 I/O 模型中，前四种属于同步 I/O，因为其中真正的 I/O 操作(recvfrom)将阻塞进程/线程，只有异步 I/O 模型才与 POSIX 定义的异步 I/O 相匹配。



