# eventfd

本质：内核维和的一个无符号的64位整数，

轻量级进程或者线程之间的高效事件通知机制

eventfd是Linux系统后来才引入的另一种轻量级的IPC方式，不同进程可以通过eventfd机制建立起一个共享的计数器，这个计数器由内核负责维护，充当了信息的角色，与它关联的进程可以对其进行读写，从而起到进程间通讯的目的。



**额外回到的点**
 
很重要的一个点就是eventfd可以 I/0复用机制等联系起来等情况，比如eventfd经常会和epoll一起出现，在Android消息处理机制中，监听是否有消息传入用的机制就是eventfd + epoll。