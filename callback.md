# callback

**要理解下面这个这个问题**

1. 如何理解Callback
2. Channel 负责callback的哪一部分
3. HttpData 这部分负责callback的哪部分
4. 真实的handlerread 里面是 如何在socket描述符上面进行正常的读写操作，handlerRead 操作缓存区和fd,

函数的关系， 其中


在channel 当中设计几个下面几个变量 回调变量

channel.h

```java

    CallBack readHandler_;   // callback
    CallBack writeHandler_;
    CallBack errorHandler_;
    CallBack connHandler_;  // 处理连接
    
```
    
    
一个请求当中绑定的变量

channel.h


```java

 void setReadHandler(CallBack &&readHandler)
    {
        readHandler_ = readHandler;
    }
    
```
当中的参数Callback readHandler 是httpdata 当中定义的

httpdata.h   定义正式的处理函数

```java

void HttpData::handleWrite()
{
    if (!error_ && connectionState_ != H_DISCONNECTED)
    {
        __uint32_t &events_ = channel_->getEvents();
        if (writen(fd_, outBuffer_) < 0)
        {
            perror("writen");
            events_ = 0;
            error_ = true;
        }
        if (outBuffer_.size() > 0)
            events_ |= EPOLLOUT;
    }
}
```

回调函数的设置和调用位置主要是在Channel当中
>在然后我们通过set***Callback来事件发生时的回调(事件回调主要是socket缓冲区准备好读写后的处理可读可写)