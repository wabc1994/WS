# Reactor模式
是一种基于事件驱动的设计模式(EventLoop) I/O复用是Reactor模式当中的一部分, 事件驱动代表着具体的事情，状态变化，比如 read for write, read for write，new incoming connection ，并且一行

# reactor定义
Reactor释义"反应堆"，是一种事件驱动机制。和普通函数调用的不同之处在于：应用程序不是主动的调用某个API完成处理，而是恰恰相反，Reactor逆置了事件处理流程，应用程序需要提供相应的接口并注册到Reactor上，如果相应的事件发生，Reactor将主动调用应用程序注册的接口，这些接口又称为"回调函数"。

> Reactor、应用程序； 应用程序向Reactor注册事件handler 以及事件处理函数接口event_handler

# event_driven architecture
事件驱动体系结构是目前比较广泛使用的一种。这种方式会定义一系列的事件处理器来响应事件的发生，并且将服务端接受连接与对事件的处理分离。其中，事件是一种状态的改变。比如，tcp中socket的new incoming connection、ready for read、ready for write。

> 将服务端对接受连接和对事件的的处理分离开来

# reactor的线程模型
## reactor单线程模式
单点问题，不能充分利用多核CPU,高负载情况下，一个线程处理的话负担
![](https://github.com/wabc1994/WS/blob/master/pic/%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%BC%8F.png)

## reactor多线程模型
只有一个反应器(Reactor)线程处理客户端连接，多个线程处理I/0事件，如果

![](https://github.com/wabc1994/WS/blob/master/pic/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%BC%8F.png)

Reactor多线程模型就是将Handler中的IO操作和非IO操作分开，操作IO的线程称为IO线程，非IO操作的线程称为工作线程!这样的话，客户端的请求会直接被丢到线程池中，客户端发送请求就不会堵塞！
## reactor 主从模型 

![](https://github.com/wabc1994/WS/blob/master/pic/%E4%B8%BB%E4%BB%8E%E6%A8%A1%E5%BC%8F.png)

作法：把Reactor拆成两个角色Main Reactor及Sub Reactor，以提升效能与资源利用率​​。

1. Main Reactor: 只负责监听客户端的连接请求，并分派给Acceptor 处理，Acceptor接受连接后,采用round-Robin 轮询调度算法交给不同的孩子Reactor, 其中一半来讲孩子Reactor的数目跟当前处理器可用核数一样， subReactor 负责read, write, 线程池负责endcode,decode, compute等计算任务情况，

**为何要选择主从模式**

1. 每个连接一个线程，线程创建和销毁的资源


**单Reactor模式 + 线程池的模式**

例如并发百万客户端连接，或者服务端需要对客户端握手进行安全认证，但是认证本身非常损耗性能。既要处理客户端连接请求，又要处理read,write等I/O操作，虽然encode,decode, compute等计算任务都是放在线程池处理， 一个Reactor也是忙不过来的

比如对Web Server而言，decode通常是HTTP请求的解析， 

但是当用户进一步增加的时候，Reactor会出现瓶颈！因为Reactor既要处理IO操作请求，又要响应连接请求(acceptor来处理)！为了分担Reactor的负担，所以引入了主从Reactor模型!





#  为何选择reactor主从线程模式
1. 与传统的阻塞I/0 处理相比
2. 每一个请求一个线程，多线程模式a thread-per-connection model 
3. 线程池模式
4. reactor eventloop模式+ 线程池模式
5. reactor 单线程和多线程模型存在的问题
6. 原生的I/0 复用编程复杂性比较高

thread vs event

[高性能IO之Reactor模式](https://www.cnblogs.com/doit8791/p/7461479.html)

# 什么叫做eventLoop?
什么情况下叫做事件循环,其中reactor我们可以理解为事件分发器，
>In the reactor pattern, the initiation dispatcher is the most crucial component. Often this is also known as the ‘Reactor’. the event


# 组件

1. handler 代表一个套接字描述符(在网络编程当中）系统处理程序(Handles)：操作系统中的句柄，是对资源在操作系统层面上的一种抽象，它可以是打开的文件、一个连接(Socket)、Timer(定时器)等。由于Reactor模式一般使用在网络编程中，因而这里一般指Socket Handle，即一个网络连接（Connection，网络连接）。这个Channel(或者fd)注册到Synchronous Event Demultiplexer中，以监听Handle中发生的事件，对ServerSocketChannnel可以是CONNECT事件，对SocketChannel可以是READ、WRITE、CLOSE事件等。或者timeout或者超时时间 

**handler 就是事件，event_handler 其实就是事件处理器，也就是函数**

2. event_handler(handler) 这个处理，据具体处理read(),write()或者处理超时任务. read/write事件处理器(Event Handler)，accept或者timeout事件处理

3. 主reactor 负责event_handler的register，handle_events(),
4. acceptor 受请求连接，然后分发给不同event_handler(回调函数)和register_handler()注册
5. reactor反应器(dispatcher分离器的作用)是什么作用，在内部维护一些表，将不同的事件类型与事件处理器关联起来;在用户已登记的某个时间发生时，反应器发出对处理器当中的相应方法的回调(其实就是处理的意思)


使用一个reactor模型，必备的几个组件：事件源(handle), Reactor 框架，多路复用机制(select()、poll()、 epoll())和事件处理程序(event_handler),  

4. I/O复用负责部分，同步事件分离器


# epoll 和reactor的关系

有了epoll等I/0复用了，为何还要reactor模型，？

一般来说通过 I/O 复用，epoll 模式已经可以使服务器并发几十万连接的同时，维持极高 TPS，为什么还需要 Reactor 模式？原因是原生的 I/O 复用编程复杂性比较高。

一个个网络请求可能涉及到多个 I/O 请求，相比传统的单线程完整处理请求生命期的方法，I/O 复用在人的大脑思维中并不自然，因为，程序员编程中，处理请求 A 的时候，假定 A 请求必须经过多个 I/O 操作 A1-An（两次 IO 间可能间隔很长时间），每经过一次 I/O 操作，再调用 I/O 复用时，I/O 复用的调用返回里，非常可能不再有 A，而是返回了请求 B。即请求 A 会经常被请求 B 打断，处理请求 B 时，又被 C 打断。这种思维下，编程容易出错。


# Acceptor与Reactor

Acceptor是处理客户端的连接， 然后想reactor 

Acceptor:Acceptor接受client连接，建立对应client的Handler，并向Reactor注册此Handler。


# dispatcher与Demultiplexer
在整个reactor模型当中的，这个东西是不一样的，其中Dispatcher 是指reactor，而Demultiplexer是分离器**(其实就是I/O复用)**，其中

1. dispatcher 是调度员
2. demultiplexer是分离器

一般地,I/O多路复用机制都依赖于一个事件多路分离器(Event Demultiplexer)。分离器对象可将来自事件源的I/O事件分离出来，并分发到对应的read/write事件处理器(Event Handler)。

# 借鉴muduo
最后成型的模式是reactor 主从模式，每个循环一个线程，也即是per loop per thread 线程模式

muduo：one loop per thread，主线程(MainReactor)注册listen事件，通过某种负载均衡机制（round robin）将连接的事件注册到子线程的subReactor上， MainReactor 和SubReactor 其实就是一个线程

# 优点
1. 可扩展性，可以方便地通过增加Reactor实例个数来充分利用CPU资源
这也是我们为何要设计多个reactor的基本原因，设计多个main Reactor 和subReactor 模式两者的区别。



# 参考链接
[反应器(Reactor)(写得非常好)](https://blog.csdn.net/mozun1/article/details/56665489)

[Reactor基本处理流程](https://www.jianshu.com/p/eef7ebe28673)