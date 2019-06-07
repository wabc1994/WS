# 高可用
1. 资源隔离，在你的系统，某个一个东西，在故障的情况下，不会耗尽所有的资源，比如线程资源，某个代码块出了故障， 陷入死循环，然后不断创建线程，，我们可以进行资源隔离，比如限定创建线程的数目，比如最多只能创建10个
2. 限流
3. 熔断
4. 运维监控，监控 + 报警 + 优化，各种异常的情况，有问题就即使，优化代码和配置，系统代码层面的，


# hystrix 
类似于一个spring的高可用架构，包括了上面的各个环节


资源隔离主要有下面几种方式
1. 线程池隔离
2. 信号量隔离 java semaphore

资源隔离之后要做的事情可能就是java

两个机制 HystrixCommand机制 和HystrixObservableCommand

## 调用失败
调用失败时候的fallback 机制， 不同服务调用失败时候的fallback 机制，说白了就是回退机制，降级处理之类的


fallback(降级) 回退

主要有以下几种情况会被fallback所捕获，
1. FAILURE：执行失败，抛出异常，比如rpc 调用过程失败之类的情况，
2. TIMEOUT: 执行超时
3. SHORT\_CIRCUITED : 断路器打开
4. THREAD\_POOL\_REJECTED:  线程池拒绝
5. SEMAPHORE\_REJECTED : 信号量拒绝情况
6. 

getcallback 降级重试之类的情况，


# rpc 学习

