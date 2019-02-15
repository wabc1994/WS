# 为何要引入线程池
1. 进程fork的开销
2. 频繁创建消耗切换线程的开销要比进程小
3. 线程池和进程的overhead进行比较
4. 多进程还是线程
5. 由于共享地址空间，线程通信vfork比进程通信高效得多

就应该为每个IO请求都创建一个线程(), 一个I/0就是read()或者write()



T1 创建线程时间，T2 在线程中执行任务的时间，T3 销毁线程时间。
如果:T1+T3远远大于>T2，那么我们则可以采用线程池

一个线程池的基本设计主要包括三部分

- 线程池管理器（ThreadPool），用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
- 工作线程（PoolWorker）： 线程池中工作线程，在没有任务的时候处于等待转台，可以循环执行任务
- 任务接口（Task）:每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定任务的接口
- 任务队列(taskQueue): 用于存放没有处理的任务，提供一种缓冲机制，

# 线程池实现

[【源码剖析】threadpool —— 基于 pthread 实现的简单线程池](https://blog.csdn.net/jcjc918/article/details/50395528)



## 编码实现

```java
const int MAX_THREADS = 1024;  // 最多的线程数目 是1024个
const int MAX_QUEUE = 65535;      // 任务队列数目是 65535
```
1. 任务队列，可以接收的任务多大，采取有界还是无界的基本问题 ,任务队列是 include<vector> 或者队列都可以
2. 线程池数据结构  **同任务队列一样**
2. 锁机制 pthread_ mutex_ t
3. condition 条件机制 
4.

```java

std::vector<pthread_t> ThreadPool::threads;
std::vector<ThreadPoolTask> ThreadPool::queue;

```

对外接口 线程

1. threadpool_t *threadpool_create(int thread_count, int queue_size, int flags); 创建线程池，用 thread_count 指定派生线程数，queue_size 指定任务队列长度，flags 为保留参数，未使用。

2. int threadpool_add(threadpool_t *pool, void (*routine)(void *),void *arg, int flags); 添加需要执行的任务。第二个参数为对应函数指针，第三个为对应函数参数。flags 未使用。 往任务队列里边添加任务并发出线程同步信号
3. int threadpool_destroy(threadpool_t *pool, int flags); 销毁存在的线程池。flags 可以指定是立刻结束还是平和结束。立刻结束指不管任务队列是否为空，立刻结束。平和结束指等待任务队列的任务全部执行完后再结束，在这个过程中不可以添加新的任务。


**为何要加锁**

线程同步和通信

**condition什么时候使用**

线程同步和通信


## 特点

1. 在创建一个线程池的就创建好所有的线程(starts all threads on creation of the thread pool)
2. 额外创建一个线程通知线程是否满了(reverse one task for signaling the queue is full)
3. (stop and joins all worker thread on destory)

# 改进
1. 懒惰创建模式，当有任务来临的时候再创建一个，
2. 线程池销毁时等待所有任务执行完成，通常是没有必要的
3. 工作线程内部存在冗余逻辑，在有任务存在时没有必要检查线程池是否停止工作

# 使用哪个模式

是采用ET还是LT 

1. 系统开销LT要多些，因为要每次都触发通知机制，
2. 

**问题**

>如果将epoll_wait的返回结果fd数组放入生产者-消费者队列中后，拿到这个任务的工作线程还没有读完这个fd，因为没读完数据，所以这个fd可读，那么下一次事件循环又返回这个fd，又分给别的线程，怎么处理？

**换句话说就是同一个套接字描述符如何避免被线程池当中两个不同的线程同时处理**



# 线程究竟处理的是什么？

线程处理的就是准备就绪的I/O操作，比如read()或者write() 操作

request一个请求是如何跟一个线程关联起来的

```java
handle_events(线程池){
 // 将一个放进入线程池
 int rc = threadpool_add(tp, myHandler, events[i].data.ptr, 0);
 
 将一个任务(I/O操作添加到任务队列当中)
 //tp  
}
```

# 线程池的底层实现
基于vector的循环队列， 为何要使用循环队列，而不是直接使用数组操作即可
 
对头head,
对尾巴tail

**出队列**


**进入队列**



# 线程间同步
等待一个消息或者某种条件

1. 条件变量pthrea_cond_t  
2. CountDownLatch
3. 互斥锁
4. eventfd 线程间通信
其中锁作为一个基础组件，配合1和2使用 

## CountDownLatch

- **为何要使用CountDownLatch**

CountDownLatch 非常适合于对任务进行拆分，使其并行执行，比如某个任务执行2s，其对数据的请求可与分为五部分，那么就可以将这个任务拆分为5个子任务，执行完成之后再由主线程进行汇总，此时，总的执行时间将决定于执行最慢的任务，平均来看，还是大大减少了总的执行时间

- **Java当中的CountDownLatch**

CountDownLatch是基于AbstractQueuedSynchronizer实现的，在AbstractQueuedSynchronizer中维护了一个volatile类型的整数state，volatile可以保证多线程环境下该变量的修改对每个线程都可见，并且由于该属性为整型，因而对该变量的修改也是原子的。创建一个CountDownLatch对象时，所传入的整数n就会赋值给state属性，



- 参考链接


	- [CountDownLatch 使用常见参考链接](https://www.jianshu.com/p/128476015902)



**编码实现**

主要有两个方法

- void wait();在主线程中使用，阻塞主线程
     
- void countDown();  让计数器减小一个1,**java当中的CountDwonLatch**

## 条件变量


```java
 cond 条件变量，互斥锁
 void wait()
    {
        pthread_cond_wait(&cond, mutex.get());
    }
    
    
    void notify()
    {
        pthread_cond_signal(&cond);
    }
    void notifyAll()
    {
        pthread_cond_broadcast(&cond);
    }
    bool waitForSeconds(int seconds)
    {
        struct timespec abstime;
        clock_gettime(CLOCK_REALTIME, &abstime);
        abstime.tv_sec += static_cast<time_t>(seconds);
        return ETIMEDOUT == pthread_cond_timedwait(&cond, mutex.get(), &abstime);
    }
```

## 条件变量condition_variable
条件变量的工作机制如下: 

- 至少有一个线程在等待某个条件成立，等待的线程必须先持有锁，然后锁被传递给wait()方法，这会让线程释放持有的锁，进而阻塞该线程，wait() 释放锁(**同Java一样的思路**)， 知道条件变量收到其他线程的通知信号，线程被唤醒，重新持有锁

- 至少有一个线程在发送条件成立的通知信号，信号的发送可以使用notify_one(), 通知任意一个在等待信号的线程，也可


# 优化
1. 采用内存池的方式，避免频繁地创建内存，在这里面我仅仅是采用malloc来获取或者free 释放内存
2. 采用 std::function<void()> 代替函数指正，代表一个线程要执行的任务。

# 每个线程究竟在做什么？

```java
void *startThread(void* obj)
{
    ThreadData* data = static_cast<ThreadData*>(obj);
    //每个线程做的事情
    data->runInThread();
    delete data;
    return NULL;
}
```
# I/O线程

任何一个线程，只要创建并运行了EventLoop(时间循环)，都称之为IO线程。

EventLoopThread创建了一个线程
在线程函数中创建了一个EvenLoop对象并调用EventLoop::loop

# 参考链接
[【源码剖析】threadpool](https://blog.csdn.net/jcjc918/article/details/50395528)