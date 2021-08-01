### ***socket地址

```
是一个结构体，封装了端口号和IP等信息，
相关的api中需要使用到该结构体
```



```
struct sockaddr {
sa_family_t sa_family;   //地址族类型的变量，与协议族相对应
char sa_data[14];        //最大14字节
};
```

#### 功能

```
在bind、connect、accept中都有用到。
```

![](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704164508697.png)

![image-20210704164546349](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704164546349.png)

现如今都是使用结构体sockaddr_in

![image-20210704172057146](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704172057146.png)

```
sockaddr_in将原先的sockaddr的ip和端口号分成了三部分，对IP和端口进行了区分，使用起来更加方便
```



```
sa_data的最大容量是14字节，无法容纳多数协议族的地址值。
于是linux定义了下面的新的socket地址结构体 --足够大的空间&内存对齐
struct sockaddr_storage { 
sa_family_t       sa_family;
unsigned long     int __ss_align;
char __ss_padding[ 128 - sizeof(__ss_align) ];
};
```



### IP地址转换

```
通信过程中需要用字符串标识IP地址，比如用点分十进制标识IPv4地址
但是需要转化为整数进行使用
记录日志时使用字符串
下列函数可用于IPv4的十进制字符串和网络字节序整数转换
```

```
#include <arpa/inet.h> 
// p:点分十进制的IP字符串，n:表示network，网络字节序的整数 
int inet_pton(int af, const char *src, void *dst);
af:地址族： AF_INET AF_INET6 
src:需要转换的点分十进制的IP字符串 
dst:转换后的结果保存在这个里面 

// 将网络字节序的整数，转换成点分十进制的IP地址字符串
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size); 
af:地址族： AF_INET AF_INET6 
src: 要转换的ip的整数的地址 
dst: 转换成IP地址字符串保存的地方 
size：第三个参数的大小（数组的大小） 
返回值：返回转换后的数据的地址（字符串）,和dst是一样的
```

网络通信常用API

![image-20210704165700776](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704165700776.png)

服务器/客户端程序开发流程

```
服务器端
1.socket 函数创建 socket（侦听socket）
2. bind 函数 将 socket绑定到某个ip和端口的二元组上
3. listen 函数 开启侦听
4. 当有客户端请求连接上来后，调用accept函数接受连接，产生一个新的 socket（客户端 socket）
***accept函数产生对应客户端socket
5. 基于新产生的 socket 调用 send 或 recv 函数开始与客户端进行数据交流
6. 通信结束后，调用 close 函数关闭侦听 socket

客户端
1. socket函数创建客户端 socket
2. connect 函数尝试连接服务器
3. 连接成功以后调用 send 或 recv 函数开始与服务器进行数据交流
4. 通信结束后，调用 close 函数关闭侦听socket

总结：客户端相比于服务器端少了bind和listen两步
且在三次握手的建立过程中，客户端的connect函数在第二次握手后就建立了连接
而服务器端的accept是在第三次握手后才建立了连接
```

### ***创建socket函数

```
//通过协议族、协议类型、具体的协议创建套接字
int socket(int domain, int type, int protocol); 
- 功能：创建一个套接字 
- 参数：
  - domain: 协议族 
     AF_INET : ipv4 
     AF_INET6 : ipv6 
     AF_UNIX, AF_LOCAL : 本地套接字通信（进程间通信） 
  - type: 通信过程中使用的协议类型 S
      OCK_STREAM : 流式协议 
      SOCK_DGRAM : 报式协议 
      **SOCK_NONBLOCK  ：标识创建非阻塞socket  默认创建的都是阻塞的
  - protocol : 具体的一个协议。一般写0
     - SOCK_STREAM : 流式协议默认使用 TCP 
     - SOCK_DGRAM : 报式协议默认使用 UDP 
  - 返回值： - 成功：返回文件描述符，操作的就是内核缓冲区。 - 失败：-1
  
 socket的阻塞会影响send或者recv函数的行为
```

### 设置set函数

```
int setsockopt(int sockfd, int level, int optname,const void *optval, socklen_t optlen);
sockfd:要设置的套接口的描述字。
level:选项定义的层次;支持SOL_SOCKET、IPPROTO_TCP、IPPROTO_IP和IPPROTO_IPV6。
optname:需设置的选项。
      SO_REUSEADDR  //端口复用
      SO_REUSEPORT  //地址复用
optval:指针，指向存放选项待设置的新值的缓冲区。
     1  可以复用
     0  不可以复用
optlen:optval缓冲区长度

```

### 绑定socketfd和本地的IP&端口

```
//

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen); 
- 参数： 
   - sockfd : 通过socket函数得到的文件描述符 
   - addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息 
   - addrlen : 第二个参数结构体占的内存大小
```

```
具体的代码实现
struct sockaddr_in bindaddr;
bindaddr.sin_family = AF_INET;                    //协议族
bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);     //ip  主机转
bindaddr.sin_port = htons(80);                  //端口号
if (bind(listenfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) == -1)
{
    std::cout << "bind listen socket error." << std::endl;
    return -1;
}

关于绑定的地址
注：INADDR_ANY相当于地址0.0.0.0    
如果你指向本机上可以访问，那么你 bind 函数中的地址就可以使用127.0.0.1;
如果你的服务只想被局域网内部机器访问，bind 函数的地址可以使用192.168.1.104；
如果希望这个服务可以被公网访问，你就可以使用地址0.0.0.0 或 INADDR_ANY。
也可以通过#define SERVER_ADDRESS "127.0.0.1"类似形式进行修改

关于端口(不仅是服务器端，客户端也可以使用bind函数)
1.如果服务器将端口号设成0，操作系统会随机给程序分配一个可用的侦听端口。
2.如果将客户端端口设为0，系统会分配给其端口号。。效果相当于不做修改
3.在特殊的应用中需要客户端以指定的端口号去连接服务器时，就可以在客户端程序bind绑定端口
客户端端口号是连接发起时操作系统随机分配的。。
```

### 启动侦听listen

```
int listen(int sockfd, int backlog);
- 参数：
     - sockfd : 通过socket()函数得到的文件描述符 
     - backlog : 未连接的和已经连接的和的最大值
- 返回值
    当返回-1时  标识侦听失败
```

```
不会阻塞
知识将套接字和套接字对应的而连接队列的长度告诉linux内核
```

```
**第二个参数是告诉内核连接队列的长度。
内核为任一个套接字接口维护两个队列。
1.未完成连接队列。(SYN-RCVD状态)
2.已完成连接队列。(estanlished状态)

队列满了之后不会直接拒绝，而是会延时建立连接。最大限制是128

backlog为两个队列的数量之和。。应该将其增大
```



### 主动发起连接connect

```
**//connect和accept的参数都是一样的

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen); 
- 参数： 
    - sockfd : 用于通信的文件描述符 
    - addr : 客户端要连接的服务器的地址信息 
    - addrlen : 第二个参数的内存大小 
**返回值：成功 0， 失败 -1


1.客户端也是可以用epoll来connect的。比如客户端请求资源会拆分成不同的请求。也需要很强的并发处理能力。。

2.connect会阻塞吗，如果设置connect不阻塞，能通过epoll来进行IO多路复用吗？

```

### 被动接受连接，阻塞等待连接accept

```
accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
 - 参数： 
     - sockfd : 用于监听的文件描述符 
     - addr : 传出参数，记录了连接成功后客户端的地址信息（ip，port） 
     - addrlen : 指定第二个参数的对应的内存大小
- 返回值：- 成功 ：用于通信的文件描述符
         - 失败 ：返回-1

//会得到用于通信的客户端的文件描述符

int accept4(int sockfd, struct sockaddr *addr, socklen_t *addrlen, int flags);
//扩展函数可以将返回的socket设置成非阻塞的    flags 设置成 SOCK_NONBLOCK 即可

**具体的实现：根据监听fd描述符找到socket 新建一个fd 将这个链接到新创建的fd上
```

底层具体有哪些操作

```
从处于estanlished状态的连接队列头部取出一个已完成的连接。.

```



### send和recv函数的阻塞和非阻塞模式下的行为

```
本质上是将应用层缓冲区的数据和内核缓冲区的数据间的拷贝。
```

![image-20210704183344128](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704183344128.png)

```
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
sockfd  目的sockfd
buf     存放要发送的数据
len     要发送的缓存的大小
flags   设置为0，表示阻塞发送的，
        发送一般不阻塞
返回值   大于0  表示成功发送n个字节
        等于0  对端关闭连接
        小于0  出错（信号中断/TCP窗口太小/）
```

```
recv函数
非阻塞下非立即返回-1，
int recv（SOCKET s，char FAR *buf，int len，int flags）;
s  表示接收端的套接字描述符
buf  表示需要接受的数据所在的缓冲区
len   变送hi缓冲区的最大尺寸
flags  一般都置0

以下为常见的错误码
```

![image-20210704193729267](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704193729267.png)

### 域名解析

```
socket地址的两个要素：IP地址和端口号
```

![image-20210704210951261](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210704210951261.png)

```
以上两条telnet命令有完全相同的作用。。
通过某些网络API实现主机名到IP地址的转换。。
struct hostent* gethostbyname(const char* name);  //可以由域名得到ip地址
struct hostent 
{
    char*  h_name;            //地址的正式名称
    char** h_aliases;         //地址的预备名称指针
    int    h_addrtype;        //
    int    h_length;          /* length of address */
    char** h_addr_list;       /* list of addresses */
}

gethostbyaddr   //根据ip地址获得主机完整信息

getservbyname  //根据名称获得某个服务的完整信息
getservbypory  //根据端口号获得服务的完整信息
```

### readv/writev  散布读/聚集写

```
若成功则返回已读、写的字节数，若出错则返回-1。readv和writev函数用于在一次函数调用中读、写多个非连续缓冲区。有时也将这两个函数成为散布读和聚集写。

readv则将读入的数据按上述同样顺序散布读到缓冲区中。readv总是先填满一个缓冲区，然后再填写下一个。readv返回读到的总字节数。如果遇到文件结尾，已无数据可读，则返回0。
```

### 返回值总结

```
socket()   //返回创建成功的文件描述符

bind()     //0标识成功了，-1表示不成功
listen()   //成功0，-1失败
connect()  //成功0，失败-1

accept()   //成功返回文件描述符，失败返回-1
send()     //返回发送的大小，失败返回-1
recv()     //返回接受的数据大小，失败返回-1
```

### 怎么判断是否是新连接

```
如果文件描述符等于监听的socket的文件描述符
则说明是有了新的连接。
否则说明是触发了监听的可读或可写事件。
```

