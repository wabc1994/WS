# HTTP 协议
**与TCP/IP**


TCP/IP负责通信，定位是哪台主机,

HTTP 是应用层协议，负责处理web server 服务器上面的各种资源 
doc, html 等，这些资源都是放在另一台电脑上面的，如何对这些资源进行解析，然后返回给客户端的电脑，这是一个问题， 浏览器就是一个电脑端的应用。


HTTP协议是放在URL 后面的，找到对应的主机后server后，server端部署的代码对客户端的HTTP请求进行一个解析


然后这个服务器端的代码数据就要讲本地数据写入socket套接字，因为在客户端通信之前，他们之间建立的TCP连接，然后TCP 连接利用socket 进行通信，接下来的数据传输就类似于传统的socket编程长不多了，所以在服务器端的代码中，要设计两个部分的基础内容

1. socket tcp 实现通信原则
2. 解析客户端的http请求request，各个字段
		- 请求行
		- 请求头
		- 空
		- 请求数据

# url request, http 的关系

在浏览器当中输入一个url，浏览器就向服务器发送一个request, request就以http 的方式发送给服务器，

1. 一个url 代表客户端的一个请求

2. 一个url  request :http://www.mywebsite.com/sj/test/test.aspx?name=sviergn&x=true#stuff

	- http: 指明协议而已，具体协议长什么样（url没有给出），所有这里面就涉及到http的结构问题

	- host : www.mywebsite.com

	- path: sj/test/test.aspx 

	- path就是requestData 当中的位置




# http请求报文结构
1. request line method  uri 协议版本  
2. request header
3. 空行
4. 请求数据

 **常见的请求头**
 
 包含的常见字段
 
 请求头有很多个key-value 类型的，  
 
 排版方式 
 
 - 头部字段名称 空格 值   
 
	- Host 主机：ip地址
 	- Cookie: sessionid
 	- Agent- : 浏览器代理
 	- Connection: keep alive or close 长连接还是短连接
 
 ![http结构](https://github.com/wabc1994/WS/blob/master/pic/http%20struct.png)
 
 

- mime 多媒体类型

## http响应报文
1. 状态行
2. 消息报文
3. 响应正文

**HEAD就像GET，只不过服务端接受到HEAD请求后只返回响应头，而不会发送响应内容。当我们只需要查看某个页面的状态的时候，使用HEAD是非常高效的，因为在传输的过程中省去了页面内容。此方法经常被用来测试超文本链接的有效性，可访问性，和最近的改变。**

## 不同请求方法请求报文和响应报文的区别
1. 请求数据不在GET 方法中使用，而是在POST 方法中使用，与请求数据相关的请求头为Content-Type 和Content-Length

# HTTP 协议特点
get和post方法， get 方法请求的参数是同过url显示的，而post 不是

1. 无连接，每一次请求都是要重新建立一个连接
2. 无状态 ，也就是不记客户端，即使同一个客户端接连发送连个http请求，服务器也不知道这两个请求是来自于同一个客户端，为了解决这个问题，引入了cookies机制和session机制，



# HTTP的五种常见状态码

分为五种，1，2，3，4，5 开头

[服务器返回的14种常见HTTP状态码](https://blog.csdn.net/q1056843325/article/details/53147180)


# HTTP 1.0 版本和1.1版本区别
1. http1.0 默认使用短连接
2. http1.1 默认使用长连接

# HTTP 常连接和短连接

**Connection:keep-alive**

通常情况下，一条http连接是发送一个资源的，参数的主要作用是延长一条HTTP连接，可以做到在一条连接使用多次，处理多个http请求。

1. HTTP服务器一般会提供Keepalive Timeout参数，用来决定连接保持多久，什么时候关闭连接。

2. 当连接使用了Keepalive功能时，对于客户端发送过来的一个请求，服务器端会发送一个响应，然后开始计时，

3. 如果经过Timeout时间后，客户端没有再发送请求过来，服务器端就把连接关了，不再保持连接了。

相当于延长了一个HTTP连接，这也是HTTP1.0 长连接的由来吧


## 长和短
**何为长连接和断连接**

短连接的操作步骤是：

建立连接——数据传输——关闭连接...建立连接——数据传输——关闭连接

长连接的操作步骤是：

建立连接——数据传输...（保持连接）...数据传输——关闭连接

**关键点**

其实HTTP的长连接就是TCP的长连接，因为HTTP是应用成协议，底层依靠的还是TCP做为传输层

# HTTP和HTTPs的简单区别


# 本项目实现

主要是实现这三种方式，
1. get
2. put
3. head


# 参考链接
[关于HTTP协议](https://www.jianshu.com/p/80e25cb1d81a)

[HTTP协议详解](https://www.cnblogs.com/TankXiao/archive/2012/02/13/2342672.html)

[HTTP长连接和短连接](https://www.cnblogs.com/0201zcr/p/4694945.html)