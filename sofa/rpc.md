[toc]

# rpc学习



## 四种调用过程
1. 同步调用
2. 异步调用 futere调用系列


3.  callback调用系列

callback 需要向rpc 注册一个callback 对象，我们需要实现一个callback类， 
1. onResponese() 成功之后的处理方法
2. onException() 异常处理方法


callback 对象

4. one way 调用

# proxy代理模块
代理类生成存根stub，解决客户端如何保存本次调用的参数，作为客户端与rpc 交互的中间层


# router模块

路由寻址模块，找到ip地址， 以及端口号

# 负载均衡

# cluster 后端服务集群模式




# 调用链filter工作机制

filter chain机制,


# remote模块
通过ip地址和端口之后与服务器端建立连接， 服务器端在80号端口上进行一个监听，

就是底层的通信模块

比如netty等模块的作用



## 序列化机制 

# spi 扩展机制
extendtionloader 扩展机制