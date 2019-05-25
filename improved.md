
[toc]

# 自己提供的优化

1. 添加了HTTP1.1 持久化机制
2. HTTPS支持, SSL\_cache\_tickets
SSL\_cache\_timeout   
3. 内存池优化,采用类似于nginx的方式大块内存和小块内存的分配方式
4. epoll+ timer实现定时器
5. 在前面部署一个最前端的 CDN内容分发网络，在客户端进行输入域名进行DNS服务的时候进行叠加一层进行服务，特别是对静态资源，相当于一个缓存，加速访问速度
6.磁盘io优化，sendfile机制， gzip 对内存记性压缩优化


第一版的服务器已经相对较完整了，该有的功能都已经具备了



# 主要是参考nginx如何进行优化

优化的思路主要集中下面几个点去，情况

1. 应用层优化
	- 代码设计层面优化
	- https协议层的优化, 要明白https带来的开销以及如何解决这些开销的问题

[HTTPS 要比 HTTP 多用多少服务器资源](https://www.zhihu.com/question/21518760)
	
   - 负载均衡
	- 方向代理
	- 容灾服务
2. tcp传输层
3. 网络层进行优化ip
4. 外层进行优化，比如
5. 服务器系统级别进行的优化
6. 最外层进行的基础优化方案





## 1. 应用层进行优化

**https的优化总体思路**

[![VitSOS.md.png](https://s2.ax1x.com/2019/05/24/VitSOS.md.png)](https://imgchr.com/i/VitSOS)

先声明这里面的keep-alive机制与tcp的keep-alive机制是两个不同的东西，这里面的是http连接的时间

### 1.1 tcp keep-alive机制

先搞懂两个东西, 

一个https连接是包括两个部分的, ，

1. 类似于http的基础tcp三次握手建立连接 
2. 在1的基础之上建立ssl 连接

所以我们在回答面试问题的时候要搞清楚这两个点的所在之处, 是不同的


**另一个区别**
还有一个东西就是https ssl里面的session id 和session tickets 跟http当中的session 和cookies也不是同一个东西
 

### 1.2 https session cache 缓存的使用

```c
ssl_session_cache  on 机制

1m能够实现的缓冲的session 具有4000多个


ssl_session_tickets
ssl_session_timeout 机制  默认的

```

**我们在编写代码的时候就是在OpenSSL当中就是**

```c
SSL_CTX_set_session_cache_mode(SSL_CTX ctx, long mode)模式
```



**为何要对Https进行一个优化操作**

- 建立连接的数据包问题

>HTTPS Server Optimization, SSL需要消耗额外的资源, 

**具体开销**
http 建立连接的时间大概是114ms, hhts建立的时间大概是436ms, 也就是说建立SSL连接的时间花费了大概322ms,http建立连接是三次握手， https 在三次握手的基础上还有9次握手， 一种就是12个tcp 包，加密需要消费更多的内存和资源


**建立连接带来的网络带宽开销**
[![ViALwD.md.jpg](https://s2.ax1x.com/2019/05/24/ViALwD.md.jpg)](https://imgchr.com/i/ViALwD)


**加密和解密的内存资源和cpu**

### 1.3 session  Resumption 机制

针对 ssl 建立连接带来的巨大开销，我们选择了进行一个session 复用的机制， 

- session id  
- session tickets 做的事情是同样的，只不过在该基础上还进行了进一步的处理



![Vi7odO.png](https://s2.ax1x.com/2019/05/24/Vi7odO.png)
 


## 2. 内存池
- 智能指针share_ptr， unique_ptr，
- nginx 大块内存和小块内存优化精细化管理
- 使用google的 gperftools 当中的tcmalloc替代gli当中的malloc, 特别是在高并发的情况下，降低系统的负载
- 内存池的设计（重点在于如何设计一个内存的思路）



# 2. 如何进行系统高可用搭建?

   这个问题主要的关注点主要集中，高性能的服务器可用或热备解决方案，keepalived可用用来防止服务器单点故障的发生，通过配合nginx可用实现web前端服务的高可用。
  
  
## 2.1 keepalived机制
   keepavlived 提供了很好的高可用性保障服务，它可以检查服务器的状态，如果服务器出现问题， keepalived会将其从系统移除，当这台服务器可以正常工作后，Keepalived再将其放入服务器群中，这个过程是Keepalived自动完成的，不需要人工干涉，我们只需要修复出现问题的服务器。
   
   


   
 
 keepalived解决的问题主要有下面三个
 1. 容灾，多机备份
 2. 单点故障问题，双机热备，主要是指当提供服务的一台出现故障时候，另外一台马上可以主动接管并且对外提供服务， 并且切换的时间非常对
 3. 主备切换，
 
   
### VRRP
底层的原理就是采用Virtual Router Redundancy Protocol,即虚假路由冗余协议， keepalived 服务器集群当中**主机A和备机B之间 是通过VRRP协议来实现监听和选举功能的实现**



**VRRP(Virtual RouterRedundancy Protocol)**协议是用于实现路由器冗余的协议， VRRP协议将两台或者多态路由器设备虚拟化为一台设备，然后这台设备对外有一个虚拟ip地址。 

然后当master正 常工作的时候，利用对外提供的虚拟地址对外提供服务即可，提供各种网络功能，比如ARP协议，ICMP, 数据转发， 而backup 对外是不拥有对外的虚拟ip地址， 也不提供对外网络功能， 仅仅接受MASTER 的VRRP 状态通知信息， 这些路由器信息被统称为备份路由器， 当MASTER路由器失效时，处于BACKUP角色的备份路由器将重新进行选举，产生一个新的路由器进入MASTER 继续对外提供对外服务， 整个切换过程对用户来说是完全透明的情况 


[![VkgQSK.md.png](https://s2.ax1x.com/2019/05/25/VkgQSK.md.png)](https://imgchr.com/i/VkgQSK)


### 工作的层次问题
它还能实现对集群中服务器运行状态的监控及故障隔离。 

1. ip 层 网络层， 主要运行icmp , ARP, RARP协议等功能，
2. 传输层 tcp/udp层面的
3. 应用层面



### 负载均衡

四层负载均衡 + 七层的负载均衡

四层负载均衡：在第传输层做负载均衡，也就是ip + 端口号进行处理，软件在第四层做的负载均衡也就是  采用NAT模式对ip 地址进行一个转换即可，

比如现在的软负载 -- LVS就是利用NAT协议搞的，


## slb服务器负载均衡



server load balance 问题，  slb 关注的是如何对部署在云服务器ECS上面的应用进行负载均衡个， 因为我们的java应用最后都是部署在 云武器上面的

是阿里云开源的产品

主要包括三个方面的内容，四层负载均衡， 七层负载均衡和控制系统

- 四层负载均衡，主要是采用 LVS(linux, virtual server)，并根据云计算需求对其进行了定制化开发，四层负载均
- 七层负载均衡， 采用来本身开源的Tengine，七层负载均衡就是采用nignx来处理的
- 控制系统， 用于配置