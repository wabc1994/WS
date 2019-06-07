
[toc]

# spring-boot-seckill项目总结

项目总体是怎么搞的？


1. 流量肖峰 排队方式
- 消息队列，同步请求转换为异步请求，起到缓冲的作用
- 利用
- 
2. 分层过滤，漏桶，对不符合请求的进行过滤
3. 答题系统，减缓请求的发出，利用图片进行一些干扰活动，防止各种秒杀器的基础使用情况

## com模块

### AOP 
主要用于在页面限流模块


## 限流模块
限流模块主要可以在两个地方来做

1. 可以在网关层面做api， 比如spring cloud nignx,
2. 也可以在应用层的AOP实现


## 服务降级
在限流之后，用户的请求是得不到响应的，是没有立即返回结果内，因此为了提升用户体验，我们是针对没有抢购成功的请求，进行一个服务降级的处理， 直接推推送到客户端的,



## 排队
将用户的请求变成有序的情况，顺便大量的请求搞成有序的一个队列，实现方式主要有下面几种情况，方便程序进行一个有序的处理。

1. 利用线程池加锁等待也是一种常用的排队方式,阻塞队列LinkedBlockingQueue，100设置固定
2. 先进入先出，先进后出等常用的内存排队队列算法的实现方式，（内存队列）
3. 把请求序列化到文件中，然后在顺序读文件（比如基于MySQL binlog的同步机制）来做回复请求
4. 分布式消息队列，分布在不同的机器上面，分布式消息队列与内存队列是有区别的情况，




### Disrupto模块应用
为何要使用Disruptor
>基于内存的无锁队列，可以高效地实现生产者和消费者的情况，

1. ringbuffer设置为1024个大小的情况
2. 生产者是前端的用户请求
3. 消费者是后面的


## nginx限流模块开发

用户前端请求先经过CDN加速，然后再进入nginx限流模块， nginx限流主要是通过在controller层来实现, nginx主要是API接口进行一个限流，因为后端部署了两台机器，这里面也可以成为分布式限流的功能了，

nginx 

```java
#统一在http域中进行配置
#限制请求
limit_req_zone $binary_remote_addr $uri zone=api_read:20m rate=50r/s;
#按 ip配置一个连接 zone
limit_conn_zone $binary_remote_addr zone=perip_conn:10m;
#按server配置一个连接 zone
limit_conn_zone $server_name zone=perserver_conn:100m;
server {
        listen       80;
        server_name  seckill.52itstyle.com;
        index index.jsp;
        location / {
              #请求限流排队通过 burst默认是0
              limit_req zone=api_read burst=5;
              #连接数限制,每个IP并发请求为2
              limit_conn perip_conn 2;
              #服务所限制的连接数(即限制了该server并发连接数量)
              limit_conn perserver_conn 1000;
              #连接限速
              limit_rate 100k;
              proxy_pass      http://seckill;
        }
}
# 负载集群配置， nginx 作为负载均衡模块，
upstream seckill {
        fair;
        server  172.16.1.120:8080 weight=1  max_fails=2 fail_timeout=30s;
        server  172.16.1.130:8080 weight=1  max_fails=2 fail_timeout=30s;
 }
```

后端主要是搞了两台服务器做一个负载均衡模块，


### 限流

在这里面的限流情况都是根据ip地址来报的，如果请求数目超过一定的量，就会返回一个http503的错误码的情况 503代表service unavailable，代表服务不可用
- limit_req_zone: 是限流声明

针对nginx，单个ip地址，每秒50个读次，写10次


zone=one\_ :10m 代表生成一个大小为10M,名字为one 的内存区域，用来存储访问的频次消息，

burst = 5 设置为大小为5的缓冲队列，在缓冲队列中的请求会慢慢等待处理，

### 负载均衡模块

1. max\_fail 

这个是nginx在负载均衡功能中，用于判断后端节点状态，所用到的两个参数， 30m秒进行探测

Nginx基于连接探测，如果发现后端异常，在单位周期为fail\_timeout设置的时间，中达到max\_fails次数，这个周期次数内，如果后端同一个节点不可用，那么接将把节点标记为不可用，并等待下一个周期（同样时常为fail\_timeout）再一次去请求，判断是否连接是否成功。

nginx只有当有访问时后，才发起对后端节点探测, 如果当前节点挂了的话，会转发给其他节点的情况，

## 分布式锁模块

要明白为何要使用分布式锁的基础功能，

1. 比如之前说的超卖，下单减库存等操作要保证原子性，既然在这种情况 比如，i++ 类似于这种操作，并不是原子操作，在单台机器上我们的操作可能就是利用synchronized或者锁这种机制确保代码的原子性， 
2.  因此我们需要进行一个操作确保原子性，并且因为代码是部署在不同的机器上面，因此需要在该基础上增加一个分布式锁


### 基于redission的分布式锁机制

锁的实现我是基于redisson组件提供的RLock, 底层还是基于setnx 命令操作

- redission是基于redis 进行改进的
- 获取分布式锁工具类 RedissLockUtil 

```java
private static RedissonClient redissonClient; //代表一个客户端
然后尝试获取锁的机制 

获取锁后进行一些系列判断，判断是否获取锁成功，然后可以进行
public void testReentrantLock(RedissonClient redisson) {
		RLock lock = redisson.getLock("anyLock");
		try {
			// 1. 最常见的使用方法
			// lock.lock();
			// 2. 支持过期解锁功能,10秒钟以后自动解锁, 无需调用unlock方法手动解锁
			// lock.lock(10, TimeUnit.SECONDS);
			// 3. 尝试加锁，最多等待3秒，上锁以后10秒自动解锁
			boolean res = lock.tryLock(3, 10, TimeUnit.SECONDS);
			if (res) {
				// 成功
				// do your business
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}

```

如何解决死锁以及超时等待的问题？

> 尝试获取锁，最多等待3秒，上锁以后10秒后自动解锁


在这里面我们有一个问题，就是使用了redission 实现了分布式锁， 是已经封装好的了，面试过程当中肯定会如果由你来实现，你会怎么实现这个问题


- [redisson/RedissonLock.java 分布式锁底层实现原理](https://github.com/redisson/redisson/blob/f01fd0823b86a609ed7c6f09620f9996a71ef827/redisson/src/main/java/org/redisson/RedissonLock.java)
- [redisson实现分布式锁原理](https://www.cnblogs.com/ASPNET2008/p/6385249.html)

## 原理实现级别

1. 加锁
 最简单的方法是使用setnx命令。key是锁的唯一标识，按业务来决定命名。比如想要给一种商品的秒杀活动加锁，可以给key命名为 “lock_sale_商品ID” 。而value设置成什么呢？我们可以姑且设置成
 
1. 加锁的伪代码如下：
 
```java
setnx（key，1）
```

当一个线程执行setnx返回1，说明key原本不存在，该线程成功得到了锁；当一个线程执行setnx返回0，说明key已经存在，该线程抢锁失败。



2. 超时解锁
  
  setnx的key必须设置一个超时时间，以保证即使没有被显式释放，这把锁也要在一定时间后自动释放。setnx不支持超时参数，所以需要额外的指令，
  
```java

expire(key,30)

```

综合起来，我们分布式锁实现的第一版伪代码如下：

```java
if（setnx（key，1） == 1）{
expire（key，30）
try {
do something ......
} finally {
del（key）
}


```


# 分布式session以及单点登录的问题

抢购项目当中，一般来讲都是服务器都是部署在多个机器上面，客户端的请求有时候请求映射到A服务器，如果过下一次的请求映射到B服务器，不可能又要用户重新登录两次，因此我们需要做一个单点登录的问题， 
在一个服务不同模块之间共用同一个session,


1. session复制
2. session集群第三方缓存
3. 基于cookies的session机制情况，客户端向服务器发送请求的时候将sessionid放在cookies id里面的基本情况
4. 粘性session，粘性Session是指将用户锁定到某一个服务器上
5. 非粘性session异同点
6. 基于数据的session，持久化session机制


解决方案：将用户登录模块抽取出来，作为一个基础模块， 然后将session存储在独立redis缓存服务器中，不同模块判断客户是否登录都是到redis服务器当中去读取即可，如果session存在redis，就代表用户已经登录了，就不需要再次登录了，


[集群/分布式环境下5种session处理策略](https://blog.csdn.net/woaigaolaoshi/article/details/50902010)


# dao 层设计 repository
```java
public interface SeckillRepository extends JpaRepository<Seckill, Long> {
    // 只有最基本的CRUD功能情况, 所有也就没有多少代码可以写了
}
```


# others
1. @service注解是放在实现类前面
2. @Autowired使用在接口前面的勤快过，既可以操作普通类型，也可以操作bean


接口就是真正在使用的


## websocket推送到前台的情况

用户抢单的消息被压缩进入消息队列，同步变成了异步的方式，因此客户端如何知道自己的抢购是否成功还是失败了，所有就有了下面的websocket， 

抢单成功之后将利用websocket推送到前端，实现服务器向用户浏览器的消息推送环节,

websocket主要是在服务器和客户端浏览器之间进行一个通信，实现任务的推胸

## 公共模块

- 账号服务模块设计
- 


# 疑问
就是有个一个问题，如何是消息队列，谁是消息的生产者，谁是消费者的问题，情况情况


## 如何解决库存超卖问题

**问题背景**
一开始一共100个商品，现在成功被抢购了99件，库存只有1个了，但是此时要是有很多了客户流量请求到系统，如果同时读取到库存为1，就代表可以购买成功， 这样就会出现超卖的问题？

一个抢购成功包括两个步骤
-
1. 减库存
2. 创建秒杀订单成功,也就是要在数据库当中创建一个章userid, goodid成功，然后持久化到磁盘当中去，也就是要落地

要保证上述两个步骤是一个原子操作， 

解决之道：声明式事务情况,情况

## 基础组件如何在代码中使用

**基础组件代码都是放在另一个包， 搞成一个个bean 实现了之后我们可以service层代码使用这些东西即可， 如果是工具类其实可以直接搞成静态方法类，直接使用类名字就可以使用了**


所以的基础组件代码都是为了service服务的，或者将


```java
public class SeckillDistributedController {
	private final static Logger LOGGER = LoggerFactory.getLogger(SeckillDistributedController.class);
	
	private static int corePoolSize = Runtime.getRuntime().availableProcessors();
	//调整队列数 拒绝服务
	private static ThreadPoolExecutor executor  = new ThreadPoolExecutor(corePoolSize, corePoolSize+1, 10l, TimeUnit.SECONDS,
			new LinkedBlockingQueue<Runnable>(10000));
	
	@Autowired
	private ISeckillService seckillService;
	@Autowired
	private ISeckillDistributedService seckillDistributedService;
	@Autowired
	private RedisSender redisSender;
	@Autowired
	private KafkaSender kafkaSender;
	@Autowired
	private ActiveMQSender activeMQSender;

	@Autowired
	private RedisUtil redisUtil;
	
	@ApiOperation(va
```


 

## 消息队列
基础的消费者就是有对个里面的数据做一些处理， 比之类，方法参数就是从broker当中取出的数据，

然后生产者的话可以直接使用kakfa提供的接口之类

##  resource 包下面的东西如何java 文件进行交互的


# dao
一个mapper接口，对应资源文件下面的mapper文件情况