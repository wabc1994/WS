# 如何使用OpenSSL实现https

实质是在应用层和TCP传输层之间增加,就是在普通的socket 建立好TCP连接后，再用SSL——connect 建立ssl层的连接，然后使用SSL_read/ SSL_write 代替read 和send 进行首发数据， 并在close socket 的前后释放ssl 层的资源即可



OpenSSL连接的两个过程

1. 握手协议，生成一个对话秘钥
2. 传输协议，就是利用这个对话秘钥是使用对称加密进行传输的

# 具体过程如下
[基于OpenSSL的HTTPS通信C++实现](https://segmentfault.com/a/1190000016855991)



2.1. 初始化OpenSSL 

2.2. 创建CTX,CTX是SSL会话环境，

2.3 创建OpenSSL 套接字， openssl 套接字的建立必须建立在普通TCP已经建立好连接之后，也就是服务器和客户端进行正常的connect()和accept 后，SSL_connect() 这个也就是上面锁说的握手协议，SSL_do_handshake() 函数等过程

