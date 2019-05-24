想林亚学长要提的问题

# 1. 定时器采用优先级队列 ，里面为何又多了一个队列，是二叉树优先级堆当中每个节点元素都是一个双向队列吗


 std::priority_queue<SPTimerNode, std::deque<SPTimerNode>, TimerCmp> timerNodeQueue;
 
 
# 2. 线程池当中的任务队列 为何要采用基于vector的循环队列
>生活中有很多队列的影子，比如打饭排队，买火车票排队问题等，可以说与时间相关的问题，一般都会涉及到队列问题；从生活中，可以抽象出队列的概念，队列就是一个能够实现“先进先出”的存储结构。队列分为链式队列和静态队列；静态队列一般用数组来实现，但此时的队列必须是循环队列，否则会造成巨大的内存浪费；
>
>
>定义的线程池，最后是直接没使用了
>
>
>

线程池的实现还是基于简单的vector向量


1. 从时间堆当中弹出元素，堆顶过期元素，我们是否还还需要进一步从相应的任务队列当中删除 该任务当中的请求
2. 二叉堆pop(）出元素，为何是直接pop() 处理，先判断是否过期， 如果没有过期，我们可以不用弹出元素，而代码当中是只要队列

什么情况下讲一个请求设置为isDeleted()

# 1. Http 管线化实现的代码在哪里

# 利用RAII来实现安全上锁类MutexLockGurd

为什么要进行两次的封装，因为RAII类一般来说,从简单实现的原则出发我们都是直接设置只有两个方法，那就是构造函数和析构函数，而且我们在实际使用当中都不是直接调用这两个方法(根本不用显示调用)， 但是实现一个锁 的基本功能，有很多个过程， 以及涉及到的方法等情况， 所以在这种情况下我们要进行多层包装，tong'guo


关于innodb当中数据是存储在哪里的


![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)



数据库的物理存储会分成许多个页（Page），记录都存储在页中，数据每一次IO，操作的最小单位是一个页。数据库运行时，会在内存中维护一个缓冲区（Buffer Pool）。缓冲区缓存部分的页（因为内存有限，所有不能把所有数据都缓存进来）。每次对数据的操作，都需要把磁盘中的页读入缓冲区，然后再进行操作；如果缓冲区中有对应的页，就不需要读磁盘。如果数据库要读入一个页，而此时需要缓冲区已经被用完了，这个时候就要替换出一个页面，腾出一块内存给新读入的页面。如果被换出的页被改过（在内存中每个页会有个Dirty标记，标识当前页面是否被修改过），就要把内存页写入到磁盘中


![](https://ww1.sinaimg.cn/large/006tKfTcgy1fckb93qmy5j30g00fh0vq.jpg)



![](https://imgchr.com/i/A2eYJf)

![A2eYJf.png](https://s2.ax1x.com/2019/04/04/A2eYJf.png)

![](https://github.com/wabc1994/InterviewRecord/blob/master/JVM/picture/reference.png)


![A2Kl1f.png](https://s2.ax1x.com/2019/04/04/A2Kl1f.png)