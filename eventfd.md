# eventfd进程间或线程间通信
实现了线程之间的notify()和wait()机制， 类似于pthread_cond_t 通信机制，是在内核2.6.22 之后进行支持的情况，轻量级的进程通信方式

int eventfd(unsigned int initval,int flags)



>这个函数会创建一个事件对象（eventfd object），用来实现进程（线程）之间的wait/notify机制。
要通信时，一个线程往这个eventfd object中写8个字节，相当于发送消息，然后另一个线程读到了这8个字节，相当于收到这个消息，然后就会进行相应的处理。这样就实现了wait/notify。

# eventfd 线程通信从优势
eventfd类似于管道的概念，可以实现线程间的事件通知，类似于pipe。而eventfd 是一个比 pipe 更高效的线程间事件通知机制，一方面它比 pipe 少用一个 file descriper，节省了资源；另一方面，eventfd 的缓冲区管理也简单得多，全部“buffer”一共只有8字节，不像pipe那样可能有不定长的真正buffer。

> pipe需要两个文件描述符，两个进程各自都有一个

eventfd的缓冲区大小是sizeof(uint64_t)也就是8字节，它是一个64位的计数器，写入递增计数器，读取将得到计数器的值，并且清零。


1. 这个函数会创建一个事件对象 (eventfd object), 用来实现进程(线程)间的等待/通知(wait/notify) 机制. 内核会为这个对象维护一个64位的计数器(uint64_t)。
2. 返回一个文件描述符eventfd() returns a new file descriptor that can be used to refer to the eventfd object. 

# 工作原理

**调用eventfd，内核创建一个64位的计数器，这个也就是内核维护的event object**

多个进程可以向这个计数器(**event object**)进行读写操作，这样就达到了线程之间的通信原则。

**为何说实现了notify()和wait()机制**

eventfd通过一个进程间共享的64位计数器完成进程间通信，这个计数器由在linux内核空间维护，用户可以通过调用write方法向内核空间写入一个64位的值，另一进程也可以调用read方法读取这个值。

# eventfd与epoll搭配使用

>As its return value, eventfd() returns a new file descriptor that can  be used to refer to the eventfd object.  The following operations can be performed on the file descriptor:

本质是一个文件描述符，所以说也可以在这个文件描述符上面进行一些系列的读写操作等情况

1. read
2. write 
3. epoll_add
 