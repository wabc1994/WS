各个部分关系的描述
1. fd ,httpdata, channel

为何要讲fd 封装成为一个httpdata和channle 

1. channel 包含回调函数的设置，fd ,处理该fd的eventloop 反应器，导致epoll层操作的都


>我是这么理解的，情况就像原生的epoll




# 基本流程
一个客户端连接进来，socket 转换成为fd,然后被主反应器eventloopThread 封装成为一个channel，然后为每个channel 构造一个timeout, 在添加定时器的时候，往定时堆中添加时，

poller 类的函数

每个eventloop 都有一个定时堆，一个poller类对象，然后eventpoll对象


```java
void Epoll::add_timer(SP_Channel request_data, int timeout)
{
    // 进一步封装成为 HttpData,
    shared_ptr<HttpData> t = request_data->getHolder();
    if (t)
        timerManager_.addTimer(t, timeout);
    else
        LOG << "timer add fail";
}
```