# 日志


1. 一个日志库大题可分为前端和后端， 前端是提供应用程序应用使用的接口， 并生产日志信息； 后端则是负责将日志信息写入到目的地。 

2. 每个线程都有自己的前端，而整个程序共用一个后端。  业务进程， 日志线程(背景线程)

本项目当中实现是异步双缓冲区的？

同步日志和异步日志的区别

![](https://github.com/wabc1994/HTTPServer/blob/master/datum/ASLOG.png)

# 异步日志

**基本概念以及流程**

   用一个背景线程负责收集日志消息并写入日志文件，其它业务线程只管往这个“日志线程”发送日志消息，这称为"异步日志"。异步日志也可以叫做非阻塞日志**类似于生产者和消费者**   
 
   业务线程不是直接向磁盘中写入， 而是先写入buffer, 然后日志线程再从buffer 区当中读取


# 同步日志的问题
**为何要异步日志**

 [一个高效的异步日志 ](https://blog.csdn.net/Shreck66/article/details/50413696)
 
 回答点：
 
原始做法的缺陷：首先将要写入日志文件的内容转化为字符串，然后调用write系统调用将其写入文件，write 是阻塞式的，

 1. 其每条日志内容都会调用write,我们都知道write为系统调用，每一条就调一次它，势必系统开销会很大
 2. 当我们在关键的地方调用write会不会导致关键部分的代码不能即使的执行
 3. 应用程序业务应该尽量避免，直接磁盘I/O速度是很慢的，

# 异步日志如何解决同步日志的问题

 **如何接上述问题**
 
 1. 既然每条日志调用一次write 会导致系统开销很大，那么我们就设计一个buffer 将日志内容保存在buffer当中，待buffer满之后再一次性调用write 将其写入即可,这样也可以降低系统的write的效用次数
 2. 由于write 可能会阻塞在当前的位置，导致紧随其后的关键代码可能不能马上执行，那么我们单独开一个线程来专门执行write系统调用不久可以咯

 
 低延迟，高吞吐量

**网络I/0与磁盘I/0不是同一个东西**

磁盘I/O 是想计算机系统存储系统当中写入，而网络I/O 是写入套接字缓冲区的，这是不一样的，

假设在网络I/O线程或者业务线程当中写日志(直接向磁盘写), 写操作偶尔可能阻塞一回儿， 可能会造成响应请求会发生超时。因此，在业务线程到


# 基本流程

## AsyncLogging 前台程序append 追加到缓冲区
可以看到前台线程所做的工作比较简单，如果currentBuffer_够用，就把日志内容写入到currentBuffer_中，如果不够用(就认为其满了)，就把currentBuffer_放到已满buffer数组中，等待消费者线程（即后台线程）来取。并且把currentBuffer_指向nextBuffer_的buffer。

1. 当前缓冲区
2. 备用缓冲区
3. 缓冲区向量

AsyncLogging::append(） 主要是这个函数

# 日志线程threadfuc write进行磁盘I/O


LogFile output

从代码可以看到后台线程的主要如下：

- 先用线程同步中的条件变量进行睡眠，睡眠时间为日志库的flush时间。所以，当条件变量的条件满足（即前台线程把一个已经满了的buffer放到了buffers_中），或者超时。
      
- 无论是哪种情况，都还会有一个currentBuffer_前台buffer正在使用。将这个currentBuffer_放到已满buffers_数组中。这样buffers_就有了待进行IO的buffer了。
        
- 将bufferToWrite和buffers_进行swap。这就完成了将写了日志记录的buffer从前台线程到后台线程的转变。后台线程慢慢进行IO即可！

# 设计
从需求、现有的硬件等限制，学习如何设计并实现一个合格的日志库？

- 需求分为功能需求、性能需求。
  
  功能需求主要考虑: 接口设计是否合理、易用？需要支持哪些功能？muduo日志库的设计原则是：尽量提供最精简的日志设施，不必要的就不提供。封闭的接口尽量便于程序员阅读、查错。
- 性能需求考虑单位时间内可写入的最大日志量，且要求它不会阻塞正常的业务处理流程。muduo的设计原则：性能只需要“足够好”，即能达到现代硬盘的最大写入带宽即可。且在实现时，要考虑减少多线程的锁争用，尽量不阻塞正常的业务处理逻辑。
 

- 如何做异常处理、性能调优？


- 日志库的实现逻辑基本大同小异，逻辑基本是：在各个业务线程中拼装日志串，然后将日志串存入一个Buffer（访问时需要加锁），另外有一个日志线程不停地从Buffer中取数据，然后将它输出到日志文件中。实现的时候会考虑：
         
         
- **Buffer如何设计？什么时候唤醒日志线程从Buffer中取数据？**
- **如何减少 业务线程、日志线程 访问Buffer时的锁竞争？**
>用双缓冲技术来减少线程之间的锁竞争，分别是操作两个互斥变量，
- **日志串如何组装，才能使它组装速度足够快、且要兼顾接口设计的易用性？**
- **要考虑线程间的竞争、写入速度等各种情况，保证不会丢失每条日志串。**
- **什么时候切换写到另一个日志文件？什么时候flush到日志文件？**
- **若日志串写入过多，日志线程来不及消费，怎么办？**


# 代码实现

1. 对底层FileUtil文件定义AppendFile类封装了append()，然后append 调用write(),这个类负责把日志写入到文件 
2. LogStream 封装FixedBuffer定长缓存区**(本质上是一个字符数组)**, 该对象有基本的函数，笔记获取缓存冲区的大小，可用空间大小， 写到哪个位置处，已经空间大小， 重载<< ,将日志信息追加到字符缓冲数组当中(buffer.add)
3. LogFile类负责日志的滚动，例如日大小达到指定大小、到达某一个时间点都会新建一个日志。日志的名字为：日志名+日期+时间+主机名+线程ID+.log，封装了FileUtil 当中的Appendfile ,LogFile当中的各种操作就是append flush 其实就是使用Appendfile当中系统write 操作
4. AsyncLogging 类封装LogStream当中的Buffer,  BufferVector，存储当前的已经满的缓冲区，最大为25，这是异步日志前后台实现的关键，分别定义了消费者append()和生产者的功能threadfuc, 其中append 是追加到两个缓存冲， 如果期间发生产生大量的日志，就会把满的缓冲区放到bufferVector,日志线程真正然后将BufferVector向量当中存储的日志文件正在写到磁盘当中

  output.append(buffersToWrite[i]->data(), buffersToWrite[i]->length());
  
  output是Logfile 类
5. Logging是最顶层的封装Logger类，AsyncLogging是其中一个对象，其中最主要是完成基本的函数操作声明最终的名字，


## 为何要采用双缓存区的设计？

**双缓冲区的基本流程**

muduo日志库是用双缓冲技术。基本思路是准备两块buffer：A和B, 前端负责往buffer A填数据(日志消息), 后端负责将buffer B的数据写入文件；当buffer A写满之后, 交换A和B, 让后端将buffer A的数据写入文件, 而前端则往buffer B填入新的日志消息, 如此往复。
！

**为啥**

前端不是将一条条日志消息分别送给后端，而是将多条日志消息拼接成一个大的buffer传送给后端，相当于批处理，减少了线程唤醒的开销。

**生产者和消费者速度不匹配的问题**

# 双缓冲区要考虑的问题
如何减少磁盘I/O操作

1. 什么时候切换写到另一个日志文件？前一个buffer已经写满了，则交换两个buffer（写满的buffer置空）。

2. 日志串写入过多，日志线程来不及消费，怎么办？直接丢掉多余的日志buffer，腾出内存，防止引起程序故障。

3. 什么时候唤醒日志线程从Buffer中取数据？其一是超时，其二是前端写满了一个或者多个buffer
4. 多线程操作缓存块之间的线程安全性问题，如何减少锁的使用？
>当当前缓冲区满了，前端业务线程通过条件变量通知后台线程


**如何较少日志线程的睡眠唤醒**
A, B两个缓存区， 前台线程操作A， 日志线程操作B, 什么情况下A,B缓冲区交换， 那就是当A写满，就把A和B进行交换，这样 就确保了日志线程操作的缓冲区都是有数据的


# 改进提升
## 标准同步操作
1. 减少业务线程和日志线程之间对于临界区资源的竞争问题。使用锁和条件变量

**// Producer，前端程序 **

```java
write_log(log_statement item)
       lock(buffer)
            buffer.append(item)
       consumer_event.notify()
   
```

**// 后台消费者**

```java
read()
   while true
        consumer_event.wait()
        lock(buffer)
            data = read(buffer)
            log_file.write(data)
```
**解释**

>It uses a shared buffer, a vector or list, and a mutex to serialize access. An event notifies the consumer that new data is available in the buffer.

**该种模式存在的问题**
对于大部分的应用来说，该种应用的性能问题是可以接受的

[Wait-free queueing and ultra-low latency logging](https://mortoray.com/2014/05/29/wait-free-queueing-and-ultra-low-latency-logging/)



2. ring buffer 情况

# 参考
[多缓冲提高日志系统性能 ](https://blog.csdn.net/wwh578867817/article/details/47413977)

[双缓冲区 - CSDN博客](https://blog.csdn.net/yzhang6_10/article/details/52337726)

[双缓冲区](https://blog.csdn.net/yzhang6_10/article/details/52337726)

[g2log: An efficient asynchronous logger using C++11 ](https://www.codeproject.com/Articles/288827/g-log-An-efficient-asynchronous-logger-using-Cplus)
