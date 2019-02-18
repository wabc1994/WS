# EventLoop 

I/O线程：包含一个EventLoop对象的线程

EventLoop有许多变量，几个bool变量，looping_：是否正在执行loop循环；quit_:是否已经调用quit()函数退出loop循环；eventHandling是否正在处理event事件；callingPendingFunctors是否正在调用pendingFunctors_的函数对象。

一个eventLoop 可以对应多个channel

## event 系列函数
- loop()
- isRunLoop()
- queueInLoop()
- doPendingFunctors

下面的关键 就是 loop()  如何进行进行处理I/0事件,以及超时时间


# Channel 



Channel 是Reactor 结构当中的”事件", 也就是reactor基于事件模型当中的事件源问题，说白了我们可以把channel 简单得理解为文件描述符，也就是我们关系的事件源，对channel 的remove update 等的操作其实就是调用Poller 类当中封装的epoll_mod 和epoll_del_ 等操作

**通道的功能主要总结为如下三种**
>1. 首先我们给定Channel所属的loop以及其要处理的fd
2. 接着我们开始注册fd上需要监听的事件，如果是常用事件(读写等)的话，我们可以直接调用接口enable***来注册对应fd上的事件，与之对应的是disable*用来销毁特定的事件
3. 在然后我们通过set***Callback来事件发生时的回调


## channel 使用

不单独使用，它常常包含在其他类中（Acceptor、Connector、EventLoop、TimerQueue、TcpConnection）使用。


1. 首先我们给定Channel所属的loop以及其要处理的fd

2. 接着我们开始注册fd上需要监听的事件，如果是常用事件(读写等)的话，我们可以直接调用接口enable***来注册对应fd上的事件，与之对应的是disable*用来销毁特定的事件

3.在然后我们通过set***Callback来事件发生时的回调

一个httpdata 关联一个channel,channel会为每个客户端request 绑定一个要处理的事件

在httpdata.c 源码当中我们要实现绑定操作

```
channel_->setReadHandler(bind(&HttpData::handleRead, this));

channel_->setWriteHandler(bind(&HttpData::handleWrite, this));

channel_->setConnHandler(bind(&HttpData::handleConn, this));

```

## 包含
1. 一个文件描述符或者socket描述符 ，实际不拥有，只是建立一种映射关系, I/O 复用类当中还有一个 map<int,channel> 
2. 在该描述符上要进行的回调函数，也就是处理函数 hander_event 
3. 还有在该fd 上面的感兴趣事件，event 等

## channel与epoll_event的关系
struct epoll_event event基本关系
event.events = channel->events();
  event.data.ptr = channel;

主要
```
  ReadEventCallback readCallback_;
  EventCallback writeCallback_;
  EventCallback closeCallback_;
  EventCallback errorCallback_;
  EventCallback 定时器处理函数
```

## EventLoop 与channel的关系
channel 的很多函数都是要通过EventLoop 来进行调用的，然后EventLoop 再调用poller 来实现

EventLoop::loop()调用Poller::poll()获得当前活动事件的Channel列表，然后依次调用每个Channel的handleEvent()函数
# 参考链接
[muduo::EventLoop分析](https://blog.csdn.net/KangRoger/article/details/47266785)

[线程间使用eventfd通信和EventLoop::runInLoop系列函数](https://blog.csdn.net/NK_test/article/details/51138359)

[muduo当中EventLoop, Channel、poller 类 ](https://blog.csdn.net/jnu_simba/article/details/14486661)