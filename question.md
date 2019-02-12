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