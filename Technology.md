在春招距离秋招我自己的一个成长，以及我关注的一个技术方向


# 关注的技术
1. 分布式存储系统, 比如微信的 
2. 系统架构设计方面
3. 新一代网络通信服务，sevice mesher,
4. 高可用， 微服务当中的高可用阶段，主要集中hystirx, 微服务系统当中的高可用解决方案，过载保护等机制，


## 分布式存储

总体特点就是如何在多台服务器之间就某个数据达成一个共认，不可更改，在分布式系统如何就某个值达成一致，一种基于消息传递模型的一致性算法


### PaxosStore
1. 微信的PaxosStore存储，基于paxos算法

	- [首发 | 微信成功挑战10亿人聊天记录的](https://cloud.tencent.com/developer/article/1111436)
	- [微信 PaxosStore:理论基础与创新设计](https://cloud.tencent.com/developer/article/1005265)



PaxosStore 主要分为三个模块，将一致性协议作为一个中间件来使用，上面是编程模型，基础的数据结构，比如<key,value>，set,table 等等， 底层是真正的存储系统级别,存储引擎，（bitcask）以及LSM等级别的系统，比如rockdb, 或者leveldb,
bitcask是一篇论文，BeansDB 是其开源实现版本


支持的特性主要下面几个
1. 同步复制，多主多写系统
2. 支持键值/列表等数据结构
3. 支持表格， 单表亿行，这样就不用分库分表
4.   


### SOFAJRAFT

2. 蚂蚁的SOFAJRAFT 开源的基于c++ 百度braft，基于raft算法, 

### raft 与paxos的异同点



## 微信 
1. libco支持千万级别的协程库，基于共享stack模式

主要分为三个模块，第一个是事件驱动， 网络Hook, 协程机制,

2. paxosStore 数据一致性存储模块， 