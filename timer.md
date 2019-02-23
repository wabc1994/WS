# 为何要定时器
最开始其实是在tcp当中的
APP实时消息通道系统，对每个用户会维护一个APP到服务器的TCP连接，用来实时收发消息，对这个TCP连接，有这样一个需求：“如果连续30s没有请求包（例如登录，消息，keepalive包），服务端就要将这个用户的状态置为离线”。


**为何要设置超时设置**

客户端如果关闭了请求连接，长时间没有通知服务器，但是在这个过程当中服务器端依然在进行这个请求的后续操作，严重浪费服务器的资源，所以我们要设置一个超时时间，所以我们可以为每一个连接设置一个超时时间，如果发生超时服务器端直接丢弃该任务即可

其中每个客户端连接都需要管理它的 timeout 时间。写一个网络服务程序时需要管理大量客户端连接的

# 定时器类型比较

1. 基于升序链表实现的定时器  
2. 基于小根堆 
3. 基于时间轮的定时器，这种定时器也叫做hash 时间轮，在netty当中处理超时I/O当中使用得到的请
4. 基于hashl8un


各种定时器优缺点，我们都需要清除和知道的， 特别是时间轮和二叉堆之间的比较， 基于升序链表的定时器**（需要遍历链表，时间效率比较低）**，插入时间复杂度为O(N), 一般来讲我们不考虑该种结构实现的定时器

**如何做出选择**

定时器的结构有多种，比如链表式，最小堆，时间轮的 ，在不同应用场景下使用哪种需要考虑效率和复杂度

# 定时器的设计
要首先回答这几个方面的问题

1. 为何要设计定时器?
>主要用来处理断开连接超时或者不活跃的连接，


2. 定时器采取的数据结构是怎样的？
3. 定时器如何跟一个客户端连接请求关联起来


# 基本比较
## 基于升序链表的定时器

在升序链表定时器里，可以看到在添加一个定时器的时候，复杂度是O(n)
因为要保持有序性，所以的遍历链表插入到合适的位置。假设系统有大量的定时器(10W个)
使用升序链表型的就会有性能问题。这时时间轮定时器就会比较适合。

>有基于升序的定时器时间链表，但是这种链表存在效率的不足，就是当插入定时器的时候时间复杂度是O(n).今天

 **涉及的数据结构 **
 
1. head 节点
2. tail 节点

**涉及的api**

1.  void add_timer(TimerNode* timerNode )  将目标定时器插入到链表当中, 时间复杂度为O(N)， 因为要查找插入位置， 


```
```

2. void  del_timer() 删除定时器，删除到期的定时器，时间复杂度为0(1)， 链表头结点
3. void update(TimerNode * timerNode) 更新
4. tick() 查找出超时定时器， **这个是关键的地方**

```c
void tick()
{
    time_t cur = time(NULL);  /*获取当前时间*/
    util_timer* temp = head;
    while( temp )
    {
        if( cur < temp-> expire )  /*超过当前时间则为超时*/
        {
            break;
        }
        temp -> cb_func( tmp -> user_data );
         /*...删除处理完的定时器，处理头结点...*/
    }
}

```

# 代码
在C语言中可以使用函数gettimeofday()函数来得到时间。它的精度可以达到微妙

使用结构体


```
struct  timeval{
       long  tv_sec;/*秒*/
       long  tv_usec;/*微妙*/
}；
```

expiredTime_ = (((now.tv_sec % 10000) * 1000) + (now.tv_usec / 1000)) + timeout;

经常使用绝对时间来做超时判断等 我们得到的最终是绝对时间， 

**缺点**

基于升序链表的定时器用起来很方便，但一旦定时事件多起来，效率就会很低，查找时间复杂度为O(N)

[基于有序链表的定时器](https://blog.csdn.net/ythunder/article/details/52048035)

# 基于二叉堆的定时器

因为处理定时器事件的时候都是超时时间最小的，所以可以采用最小堆的方式来打理这群定时器，即每个节点的值都小于或等于他的子节点的值的二叉树

因此本实现方案使用最小堆来维护多个定时器，插入O(logn)、删除O(1)、查找O(1)的效率较高。


# 时间堆编码实现
1.  调整
二叉堆的底层还是采用数组来表示

```
time_heap:: percolate_down(int hole){
	heap_timer * temp = array[hole];
	int child = 0;
	for()
}
```

2. 定时器类
3. 定时器管理器类

当堆的容量不满足的时候，扩容为两倍

# reactor反应堆如何跟定时器关联起来

定时堆当中的存储的节点 是<用户请求requestdata, 过期时间>

用户请求request 跟一个fd, channel, eventloop对象关联起来

```

 EventLoop *loop_;
 std::shared_ptr<Channel> channel_;
 
 ```


![reac](https://github.com/wabc1994/WS/blob/master/rm.png)

[比较](http://oserror.com/distributed/implement-network-framework-using-C/#%E4%BB%8B%E7%BB%8D)

[使用epoll+时间堆实现高性能定时器](https://blog.csdn.net/gbjj123/article/details/25155501)
复杂度方面的比较

# epoll + timer定时器设置
主要是出现在epoll_wait() 当中的第三个参数


# 参考链接
[升序定时器的时间链表的完全实现](https://blog.csdn.net/HELPLEE601276804/article/details/36445053)

[Reactor_Implemention/timeheap.h at master · song0071000/Reactor_Implemention](https://github.com/song0071000/Reactor_Implemention/blob/master/timeheap.h)

[定时器的设计](http://oserror.com/distributed/network-framework-timers/#%E5%BC%95%E8%A8%80)