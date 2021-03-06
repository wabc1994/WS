# nginx如何解决惊群问题 
nginx 解决的是epoll_wait里面的惊群问题

1. epoll exclusive epoll排他模式情况, 

基本现在都使用epoll exclusive 替代了 accept\_mutex模式了,

要理解明白epoll exclusive 工作模式， 

就要明白epoll exclusive就要明白，
1. epoll文件描述epfd，与fd文件描述符的区别，epoll\_create 函数通过fd文件描述符创建epoll文件描述符 epfd,  
 

一个epfd 代表一个epoll实例,  将epoll 实例设置为独占唤醒epollexclusive模式的，

>nginx 的accept\_mutex 主要是为了解决 epoll\_wait的惊群问题，

如何不是独占模式的话，就是下面该种情况了，多个进程共用一个epfd, 那么就会阻塞在epoll\_wait处，多个进程等待同一个epfd, 这就是问题，所以设置为epoll exclusive 后，每个进程以独占的方式exclusive 占用一个epfd, 这样就避免了多个进程在同一个epfd 处等待的情况



# 另一种解释

还有另一种解释的就是，同一个fd可能放入多个epfd，导致惊群问题，多个epfd句柄当中监听同一个socket所导致的的惊群问题，设置为独占模式之后就变成了 一个fd 只能放入一个epfd,




#注意
epollexclusive只能与epoll_clt_add 方法使用，并且是放在第二个参数





[nginx源码阅读(十四).惊群问题的解决](https://blog.csdn.net/move_now/article/details/78509211)

 

# Accept()惊群问题

在Linux 2.6当中， 已经解决了accept上面的惊群问题， 

是怎么解决的呢？

情况 对accept() 进行了处理， 从等待队列当中取出第一个等待的线程或进程，

