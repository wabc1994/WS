# 读写相关异常
优雅关闭连接
int shutdown(int sock, int howto);  //Linux

sock 代表要关闭的套接字描述符， howto代表具体要怎么样的方式


[优雅的断开连接-shutdown()](http://c.biancheng.net/cpp/html/3044.html)


在TCP层，有个FLAGS字段，这个字段有以下几个标识：SYN, FIN, ACK, PSH, RST, URG. SYN表示建立连接，  FIN表示关闭连接，    ACK表示响应， PSH表示有 DATA数据传输，RST表示连接重置。只要TCP栈的读缓冲里还有未读取（read）数据，则调用close时会直接向对端发送RST。



**RST报文有什么作用**


其中复位标志RST的作用就是“复位相应的TCP连接”。发送rst段关闭连接时，不必等缓冲区的数据都发送出去，直接丢弃缓冲区中的数据。而接收端收到rst段后，也不必发送ack来确认。


rst 是谁向谁发这个得弄情况，首先一般而言都是关闭方向请求方发送，比如服务器端已经close了，但是服务端recv buf还有数据的要读的话怎么办，tcp程序向客户端发送一个rst位


TCP处理程序会在自己认为的异常时刻发送RST包。例如，A向B发起连接，但B之上并未监听相应的端口，这时B操作系统上的TCP处理程序会发RST包。


>例如，客户端发了两个请求，服务器只从buffer 读取第一个请求处理完就关闭连接，tcp层认为数据没有正确提交到应用，使用rst发送给客户端，rst是

[Linux-TCP RST 的几种情况](https://www.cnblogs.com/JohnABC/p/6323046.html)


## EPOLLIN触发但是read()返回0的情况

这种情况通常有两个原因:
> * 对端已经关闭了连接close()，这时再写该fd会出错，此时应该关闭连接,这是服务器这段也要关闭连接
> * 对端只是shutdown()了写端，告诉server我已经写完了，但是还可以接收信息。server应该在写完所有的信息后再关闭连接。更优雅的做法是透明的传递这个行为，即server顺着关闭读端SHUT_RD，然后发完数据后关闭。

>这个问题在思考测试，询问同事之后，找到了一个方法，可以做到这一点。


## select 返回可读但是读取不到数据
 当使用 select()函数测试一个socket是否可读时，如果select()函数返回值为1，且使用recv()函数读取的数据长度为0 时，就说明该socket已经断开。
 为了更好的判定socket是否断开，我判断当read()返回值小于等于0时，socket连接断开。但是还需要判断 errno是否等于 EINTR 。如果errno == EINTR 则说明read函数是由于程序接收到信号后返回的，socket连接还是正常的，不应close掉socket连接。



# accept 
int accept(int sockfd, struct sockaddr* addr, socklen_t* len)
返回：非负描述字——成功， -1——失败

addr 和len 都是这是一个结果参数，，代表一个已经建立了连接的客户端，这个函数就是有一个疑点，为何会返回两个socket


#connect accept函数

connect是第二次握手返回

accept 是在第三次握手建立后才调用的

# readn,writen两个函数的设计
1. readn


就我目前了解阻塞与非阻塞read返回值没有区分，都是 <0：出错，=0：连接关闭，>0接收到数据大小，特别：返回值 <0时并且(errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况下认为连接是正常的，继续接收。只是阻塞模式下read会阻塞着接收数据，非阻塞模式下如果没有数据会返回，不会阻塞着读，因此需要 循环读取

# 错误代码

**EAGIN** 

在非阻塞socket 套接字意思是当前不可读写，只要继续重试就好，结束本次调用，重新再进行一次系统调用即可 ，直接return 本次系统调用，

**EINTR**

遇见EINRT信号的问题，我是借鉴 《unp 》，采用continue或者goto again循环解决的。但是感觉这个还是很有必要记录一下。网络上查找到的信息很多

如果进程再一个慢系统调用(slow system call)中阻塞时，当捕获到某个信号并

**问题**

# 捉包工具分析

**wireshark工具介绍**


# tcp和udp的区别

tcp 和 udp 两者为何发一个是可靠一个是不可靠，反映在socket 里面，可以查看下面的两个链接

[(7条消息)TCP/UDP的接收缓冲区和发送缓冲区 - Swallow_he的博客 - CSDN博客](https://blog.csdn.net/Swallow_he/article/details/84392285)