
# RAII
资源获取即初始化，resource acquisition is initialization, 


在系统中，资源是有限的，一旦用完必须归还给系统，否则可能会造成资源耗尽或其他问题。例如，动态分配的内存如果用完不释放会造成内存泄漏。

这里说的资源不仅仅是指内存，还包括其他，例如文件描述符、网络连接、数据库连接、互斥锁等。

RAII当中的资源是指多个种类的情况

**为何要使用RAII方式**

RAII 的核心简单来说，就是为了防止内存泄漏，资源泄漏等情况产生的一种编程技巧， 我们平常在编程的过程当中，经常碰到申请堆内存。 在这些情况下面我们需要手动释放我们之前申请的资源(有借有还)。 但是由于程序出现异常获取提前return，导致程序没有运行到释放资源的代码块，导致内存泄漏等情况


**如何确保资源能够一定释放**

我们知道类当中有构造函数和析构函数， 这两个东西如果我们没有定义的话，编译器会调用默认的， 所以我们可以把要申请、 使用和释放的资源分别放在类的的构造函数和析构函数当中， 这两个函数反正一定会使用，这样就确保了我们申请的资源最后都一定会得到释放，

>RAII能让我们优雅的做完释放资源这件事。C++标准保证任何情况下，已构造的对象最终会销毁，即它的析构函数最终一定会被调用。我们前面说过，“借”了，一定要“还”。注意到那两个红色的“一定”了吗？一个是C++中一定会调用的函数，一个是我们一定要做的事。那我们把一定要做的事情放到一定会调用的函数中可以吗？这样我们就保证了一定要做的事最终一定会发生！

   没错，RAII就是利用析构函数一定会被调用的特性(不管是在return还是在异常退出的情况下)，将释放资源的代码写到类的析构函数中。在类对象初始化的时候，将资源传入该类对象中“托管”，并利用类对象最终调用析构函数将托管的资源释放掉。

 这样，资源的生命周期就等同于类对象的生命周期。
 
 
 
# 实现步骤
1. 定义一个属性变量(资源)
2. 构造函数(将资源传入对象初始化资源，实现托管)
3. 析构函数（当类对象生命周期结束（或异常退出时，在进入catch语句前，会自动调用对象析构函数），调用其析构函数，托管的资源在析构函数中同时也释放掉）


 



# 应用在项目当中
1. 智能指针shared_ptr 和weak_ptr
2. 自定义RAII C++实现范围互斥锁，方便线程安全上锁和释放锁

# 为何需要安全释放锁
1. 如果程序员忘了释放锁
2. 如何程序出现提前return 或者终止导致不能正常执行到pthread_mutex_unlock()代码块的话
3. 这避免了在临界区中因为抛出异常或return等操作导致没有解锁就退出的问题。极大地简化了程序员编写mutex相关的异常处理代码



# 互斥锁的使用

互斥器的使用

- 用 RAII 手法封装 mutex 的创建、销毁、加锁、解锁这四个操作。保证锁的生效期间等于一个作用域（scope）。
- 只用非递归的 mutex（即不可重入的 mutex）。
- 不手工调用 lock() 和 unlock() 函数，一切交给栈上的 Guard 对象的构造和析构函数负责（Scoped Locking）。
- 在每次构造 Guard 对象的时候，思考一路上（调用栈上）已经持有的锁，防止因加锁顺序不同而导致死锁。


RAII技术，很有意思，与其说是一个技术，不如说是一个编程上的窍门。我们平常编程时也可能会用到，在stl模板库里也有体现，但可能我们并不知道它的名字。


# MutexLock 
- pthread_mutex_t mutex
- MutexLock() 构造函数
	- pthread_mutex_init(& mutex)
- ~MutexLock() 析构函数
 	- phthead_mutex_destroy(mutex)

 	

# MutexLockGuard


# 源码
```java
// @Author Lin Ya
// @Email xxbbb@vip.qq.com
#pragma once
#include "noncopyable.h"
#include <pthread.h>
#include <cstdio>


// 自定义RAII C++实现范围互斥锁 , 主要有两个地方使用了RAII方法， 对pthread_mutex_t mutex进行一个封装成为mutex 和一个条件变量
// 其中构造函数当中调用pthread_mutex_init,在析构函数当中调用pthread_mutex_destroy()
// 在析构函数当中


// 这样定义定义之后，

//https://blog.csdn.net/liuxuejiang158blog/article/details/10953305
class MutexLock: noncopyable
{
public:
    MutexLock()
    {
        pthread_mutex_init(&mutex, NULL);
    }
    ~MutexLock()
    {
        pthread_mutex_destroy(&mutex);
    }

    //
    void lock()
    {
        pthread_mutex_lock(&mutex);
    }
    void unlock()
    {
        pthread_mutex_unlock(&mutex);
    }
    pthread_mutex_t *get()
    {
        return &mutex;
    }
private:
    pthread_mutex_t mutex;

// 友元类不受访问权限影响
private:
    friend class Condition;
};


// MutexLockGuard 包括一个mutex变量然后对 MutexLock 进行封装, MutexLock实际上是对 pthrea_mutex_t 变量的封装


//下面是RAII类的标准设计方法
class MutexLockGuard: noncopyable   //RAII class
{
public:
    explicit MutexLockGuard(MutexLock &_mutex):
    mutex(_mutex)
    {
        mutex.lock();   //构造时加锁

    }
    ~MutexLockGuard()
    {
        mutex.unlock(); //析构时解锁
    }
private:
    MutexLock &mutex;
};

// 这也是避免死锁的方式，
// 禁止复制

// 使用的时候，是先定义一个mutex 对象，然后 Mutex mutex， MutexLockGurad lock(mutex)
```

# 使用

对pthread_mutex_t 利用RAII技巧进行封装后，我们使用锁的方式就可以很方便了，

没有封装之前

```java
      pthread_mutex_lock(mutex)
   ······
   操作临界区互斥变量
   
   ·····
     最后一定要手动释放锁，
     pthread_mutex_unlock(mutex)
     
     
      pthread_mutex_lock(&lock);
        // 在这里面服务器本身存放的文件，html, avi,bmp .c .doc ,,    http当中的content/type
        if (mime.size() == 0)
        {
            mime[".html"] = "text/html";
            mime[".avi"] = "video/x-msvideo";
            mime[".bmp"] = "image/bmp";
            mime[".c"] = "text/plain";
            mime[".doc"] = "application/msword";
            mime[".gif"] = "image/gif";
            mime[".gz"] = "application/x-gzip";
            mime[".htm"] = "text/html";
            mime[".ico"] = "application/x-ico";
            mime[".jpg"] = "image/jpeg";
            mime[".png"] = "image/png";
            mime[".txt"] = "text/plain";
            mime[".mp3"] = "audio/mp3";
            mime["default"] = "text/html";
        }
        pthread_mutex_unlock(&lock);
```


**封装后的使用情况MutexLockGurad**

```java      
      MutexLockGuard lock(mutex_);
            // 如果buffer为空，那么表示没有数据需要写入文件，那么就等待指定的时间（注意这里没有用倒数计数器）
            if (buffers_.empty())  // unusual usage!
            {
                // 条件变量进行等待  // 如果buffers_为空，那么表示没有数据需要写入文件，那么就等待指定的时间。
                cond_.waitForSeconds(flushInterval_);
            }

            buffers_.push_back(currentBuffer_);
            // 重新值
            currentBuffer_.reset();

            currentBuffer_ = std::move(newBuffer1);
            // buffersToWrite 是后台日志部分使用的，而buffers_前台程序使用的基本情况
            buffersToWrite.swap(buffers_);
            if (!nextBuffer_)
            {
                nextBuffer_ = std::move(newBuffer2);
            }
   // 没有释放代码块片段的情况
   
   // 离开这段代码块自动释放
            
```

在临界区代码前面直接定义一个MutexLockGurad lock(mutex_)即可
当离开临界区代码块，这个自动调用析构函数，析构函数里面封装释放锁操作，就可以释放了这个锁


# c++标准当中的Lock_guard

- lock_guard和mutex配套使用, mutex是c++11 语言带有的#include<thread.h>C++11标准库定义了4个互斥类：
- 我们自己在项目当中使用的是 pthread_mutex_t是Linux平台下就已经有的， 需要包含# include<pthread.h>

# 智能指针
1.什么情况下使用unique_ptr呢， 不需要共享变量的时候， 整个程序当中只有一份，比如
>  std::unique_ptr\<EventLoopThreadPool>eventLoopThreadPool_;


2. 需要在不同线程之间进行共享的一个变量，比如HttpData 还有Channel 之类的，则使用share_ptr
3. 放入容器中的动态对象，使用shared_ptr包装，比unique_ptr更合适 ,比如线程池底层的存储vector
> std::vector\<std::shared\_ptr\<EventLoopThread>> threads_;   // EventLoop线程池里面的线程
> 
4. 如果你没有打算在多个线程之间来共享资源的话，那么就请使用unique_ptr。


# 什么情况下使用weak_ptr
要解决环形引用的问题，没有特别好的办法，一般都是在可能出现环形引用的地方使用weak_ptr来代替shared_ptr。说到了weak_ptr，那下面就接着总结weak_ptr吧。

# 参考链接
[一种优雅的资源管理技术——RAII ](https://blog.csdn.net/u013378438/article/details/30336333)

[RAII封装mutex ](https://blog.csdn.net/liuxuejiang158blog/article/details/10953305)
