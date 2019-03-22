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

主机host,就是www.mywesite.com这个名字， 采用域名来表示，并且后面浏览器向DNS域名服务器请求解析该域名对应的ip地址，

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
4. 请求数据 (GET 不适用，POST 使用) request body

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
1. 状态行 status line
2. 消息报文 headers
3. blank line
3. 响应正文 response body

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


一个重要的概念就是GET和POST的区别
1. get 请求报文把请求数据放在url 之后，所以请求报文里面没有请求数据这东西，而post有，只不过位置不一样了，这也导致了数据的安全性问题

![](https://github.com/wabc1994/WS/blob/master/pic/Screen%20Shot%202019-02-18%20at%201.30.57%20PM.png)


# GET与POST 区别
解释的时候就要使用字面意思解释和

从字面意思很容易来解释就是从get 是获取服务器上面的某种资源，而POST 就

1. get把请求的数据放在url上，即HTTP协议头上，其格式为： 
以?分割URL和传输数据，参数之间以&相连。 
数据如果是英文字母/数字，原样发送， 
如果是空格，转换为+， 
如果是中文/其他字符，则直接把字符串用BASE64加密，及“%”加上“字符串的16进制ASCII码”。 
post把数据放在HTTP的包体内（requrest body）。
2. GET的URL会有长度上的限制，则POST的数据则可以非常大
3. POST比GET安全，因为数据在地址栏上不可见

# cookies 和session区别

## cookies 的类型

- 可以按照过期时间分为两类：会话cookie和持久cookie。
- 会话cookie是一种临时cookie，用户退出浏览器，会话Cookie就会被删除了。
- 持久cookie则会储存在硬盘里，保留时间更长，关闭浏览器，重启电脑，它依然存在，通常是持久性的cookie会维护某一个用户周期性访问服务器的配置文件或者登录信息。


## cookies的
cookies的处理步骤
1. 服务器端向客户端发送cookies
2. 浏览器保存cookies
3. 之后每次http请求浏览器都会向服务器发送cookies 发送给服务器端

## 服务端对产生cookies
- set-cookies 是服务器发送给客户端的，用在客户端第一次想服务器端请求一个连接，设置一些东西，比如

Set-Cookies:id=" ", domain = "服务器的域名"（放在响应头里面），  Expires=Wed, 21 Oct 2019 07:28:00 GMT;或者cookies的过期时间

1. cookies的域 说白了就是服务器的域名，只有当客户端访问对应的服务器域名，客户单端才向相应的服务器发送相应的cookies
2. cookies的路径Path
3. cookies的Secure 


服务器端可以设置的cookies字段
```java
Set-Cookie: <cookie-name>=<cookie-value> 
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
Set-Cookie: <cookie-name>=<cookie-value>; Secure
Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly

Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax

// Multiple directives are also possible, for example:
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```
## 客户端的cookies 
cookies分为两种，一种是回话cookies和持久化cookies

cookies 字段name=value是客户端在后面的请求后都放在请求头当中，header 当中就具有的情况


# 跟网络连接的一些问题
[服务器如何判断读取完一个http请求了](https://blog.csdn.net/QHH_QHH/article/details/51206832)

- content-length
- 结束标识符/\n


# HTTPS的加密算法
HTTP协议由于是明文传送，所以存在三大风险：

1. 被窃听的风险：第三方可以截获并查看你的内容

2. 被篡改的危险：第三方可以截获并修改你的内容

3. 被冒充的风险：第三方可以伪装成通信方与你通信


## 面试回答要点

回答这个问题主要从这个方面来回答即可该种情况
1. 异同点https和http, 明文和密文等情况
2. 非对称加密算的过程
3. 具体服务器通信过程
4. 具体编码过程openssl来实现

**密钥**

1. 对称加密，对加密和解密使用相同密钥的加密算法，DES,AES,
>加密速度快，秘钥传递困哪

**对称加密算法的缺点**

保存和传递密钥，就成了最头疼的问题。如何保证安全性，



2. 非对称加密 其中RSA是非对称加密，因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法,RSA,DSA,DH，公钥是可以公开的，完全就没有影响，反正私钥是比较难以计算的不怕，


**什么叫做对称的，什么叫做非对称**

对称：加密和解密使用同一套规则

非对称加密：加密和解密可以使用不同的规则，只要这两种规则存在某种对应关系即可，这样就避免了直接传递秘钥；**公钥加密，私钥解密；私有加密，公钥解密；


**毫不夸张地说，只要有计算机网络的地方，就有RSA算法**

## 其他方法
1. 对称加密算法， DES,3DES,AES，最快速、最简单、效率很高
2. 非对称加密算法 RSA,DSA，安全性高，但加密与解密速度慢。
3. 散列算法 SHA-1,MD5算法

## RSA非对称加密算法
1977年由美国3个麻省理工三个数学家做出来的，在1976年之前都是使用对称加密算法，为何选择RSA算法，主要是在openssl提供的非对称加密算法当中，包括DH算法，


**DSA一般用于数字签名，RSA算法既可以用于密钥交换，也可以用于数字签名。所以选择这个**


**加密和数字签名不是一个概念；RSA不仅能够加密，也能够用于数字签名**

1. 互质
2. 欧拉函数，如果求解小于等于N的所有数的互质关系数
3. 欧拉定理
4. 因素分解方法


**面试回答核心要点问题**
利用单项函数正向求解很简单，反向求解很复杂的特性；


涉及到六个参数，
1. 质数1p,
2. 质数q2, 
3. 摸值n, = p*q
4. 公钥指数e(随机选择的),
5. 私钥指数d;
6. 摸值n;

>这六个数字之中，公钥用到了两个（n和e），其余四个数字都是不公开的。其中最关键的是d，因为n和d组成了私钥，一旦d泄漏，就等于私钥泄漏。



组成： 
- 模值n和e 组成为公钥
- 模值n和d 组成私钥,

知道公钥的情况 n和e的情况下，如果要求出d

>结论：如果n可以被因数分解，d就可以算出，也就意味着私钥被破解。

**结论：如果n可以被因数分解，d就可以算出，也就意味着私钥被破解。**
>大整数的因数分解，是一件非常困难的事情。目前，除了暴力破解
这里面的大正数都是指很多位的数组，比如 1024位或者2048位重要场合的情况下面

在该种情况下面我们该如何



>两个数相乘非常容易实现，但要将一个特别特别特别大的数分解为两个很大的质数相乘，这个难度，

具体是利用了：
1. 两个质数相乘很容易，而将其合数分解很难的这个特点进行的加密算法，n= p*q,知道p和q求出n简单，已知n 反向求出p,q却很困那的问题


## https握手过程
我们知道对称加密和非对称加密各有都有优点，效率、速度，成本等，所以在https数据传输过程我们是采用两者结合的一个思想，使用非对称加密将客户端和服务器端的浏览器对称秘钥加密，d

1. 非对称加密算法用于在握手过程中加密生成对称秘钥
 
2. 对称加密算法用于对真正传输的数据进行加密 
3. 而HASH算法用于验证数据的完整性

**浏览器是持有服务器的公钥，然后服务器端保持有私钥，服务器将自己的公钥发送给浏览器**

1. 协商加密算法
2. 服务器身份验证，包含RSA的数字证书
3. 会话秘钥计算，使用随机数，使用服务器端的公钥加密后发送给服务器，
4. 然后就是安全数据传输

1. 浏览器将自己支持的一套加密规则发送给网站，比如RSA加密算法，DES对称加密算法，SHA1摘要算法
2. 服务器从中选择一种加密算法，并告知浏览器，

**上述过程也称为协商加密算法**

主要分为

(1). 协商加密算法。浏览器A向服务器B发送浏览器的SSL版本号和一些可选的加密算法，B从中选择自己所支持的加密算法，并告知A

[https的加密方式 介绍 + 常见的加密技术](https://blog.csdn.net/guizaijianchic/article/details/77961418)

## https编码实现

就是普通的openssl实现了
1. OpenSSL 初始化 SSL_CTX对象
2. 普通socket 建立连接
3. SSL层建立连接
4. 收发数据SSL_read(),SSL_write(）等情况
5. 连接关闭以及资源清理工作

# https中间人攻击


即使中间人截获了服务器发送给客户端的公钥， 然后伪造一个假的公钥发送浏览器，然后在这，，

解决https中间人攻击:公证处，CA 权威机构，权威结构会对来自服务器端发送过来的公钥进行一个验证，





[深入HTTPS系列四（中间人攻击）](https://www.jianshu.com/p/9c52693a09dc)


1. 引申到平常的生活当中如何防范中间人攻击
	- 不要接入为验证的wifi
	- 你比如12306证书等内容，我们尽量使用正规渠道下载的证书情况
[HTTPS原理以及HTTPS中间人攻击](https://ifunbox.top/security-https-and-MITM)

## RSA参考链接


[关于RSA非对称加密相关概念整理](https://blog.csdn.net/luochoudan/article/details/70551798)

[RSA算法原理（二）](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)
# 参考链接
[关于HTTP协议](https://www.jianshu.com/p/80e25cb1d81a)

[HTTP协议详解](https://www.cnblogs.com/TankXiao/archive/2012/02/13/2342672.html)

[HTTP长连接和短连接](https://www.cnblogs.com/0201zcr/p/4694945.html)

[Cookie看这一片就OK了基本上 ](https://juejin.im/entry/5a29fffa51882531ba10da1c)