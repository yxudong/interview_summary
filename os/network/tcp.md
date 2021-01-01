## 三次握手

<img src="https://img.caiyifan.cn/typora_picgo/P10101-224755.jpg" alt="P10101-224755" style="zoom: 15%;" /> 

### 为什么是三次握手而不是二次握手或四次握手

考虑一种异常case: client 发出的第一个连接请求报文段并未丢失，而是在某个网络结点长时间滞留了，以致**延误到连接释放以后**的某个时间才到达 server。本来这是一个早已失效的报文段，但server 收到此失效的连接请求报文段后，就误认为是 client 再次发出的一个新的连接请求，于是就向client发出确认报文段，同意建立连接。假设不采用 “三次握手” ，那么只要 server 发出确认，新的连接就建立了。由于现在 client 并没有发出建立连接的请求，因此不会理睬 server 的确认，也不会向server发送数据。但 server 却以为新的连接已经建立，并一直等待 client 发来数据。这样，server 的很多资源就白白浪费掉了。采用 “三次握手” 可以防止上述现象发生。例如刚才那种情况，client 不会向 server 的确认发出确认。server 由于收不到确认，就知道client并没有要求建立连接

第四次握手完全没有必要，三次握手已经能够满足设计需求了，四次握手是冗余的。

### Linux kernel TCP握手和队列

<img src="https://img.caiyifan.cn/typora_picgo/image-20210101231438511.png" alt="image-20210101231438511" style="zoom:80%;" />  

- tcb (tcp 控制块)是做什么的？

  用于表示一个 tcp 连接，生命周期伴随整个连接的握手和挥手，socket fd 唯一对应一个 tcb ，主要存放了以下信息：

  - 五元组（sip，sport，dip，dport，proto）
  - tcp 状态

- listen(fd, backlog) 中的 backlog 表示的是什么

  kernel 2.2 之前 sync queue 和 accept queue 共用一个队列，当时 backlog 表示这个队列大小

  kerenl 2.2 之后 sync queue 和 accept queue 分离后用两个队列表示，此时 backlog 表示 sync queue 大小

- accept 系统调用流程
    1. accept queue 为空，则阻塞；不为空，则取出一个节点(tcb)
    2. 分配一个 fd，与节点(tcb)一一对应

- ddos 攻击原理，如何防御 ddos 攻击

  1. 针对半连接队列(sync queue)的，攻击方不停地建连接，但是建连接地时候只做第一步，第二步中攻击方收到 server 的 syn + ack 后故意扔掉什么都不做，导致 server 上这个队列满其他正常请求无法进来
  2. 在服务器前端加上工作于 tcp 层之下的 ip 层的防火墙，采用 sync cookie

  

  