### IO多路复用及其工作模式

```
select的缺点：
同时处理的文件描述符是由上限的，
每次都需要重新设置fd集合
1.每次调用都需要把全部的fd集合从用户态拷贝到内核态
2.拷贝到用户态之后会遍历所有的fd

poll
优：是对select的改进，文件描述符数量无上限，
输入输出参数进行了分离
缺:
1.每次调用都需要把全部的fd集合从用户态拷贝到内核态
2.拷贝到用户态之后会遍历所有的fd

epoll的优点:
文件描述符数量无上限。使用红黑树进行监管。
事件就绪通知方式，类似callback的回调机制，fd就绪就迅速激活该文件描述符
维护就绪队列，避免无效遍历
```

```
epoll将原先的select分成了epoll_create、epoll_ctl、epoll_wait三部分
```

### select详解

```
int select(int nfds, 
           fd_set *readfds,
           fd_set *writefds,
           fd_set *exceptfds,
           struct timeval *timeout)
参数说明：

参数 nfds， Linux 下 socket 也称 fd，这个参数的值设置成所有需要使用 select 函数检测事件的 fd 中的最大 fd 值加 1。
参数 readfds，需要监听可读事件的 fd 集合。
参数 writefds，需要监听可写事件的 fd 集合。
参数 exceptfds，需要监听异常事件 fd 集合。
readfds、writefds 和 exceptfds 类型都是 fd_set，这是一个结构体信息，其定义位于 /usr/include/sys/select.h 中：

函数是阻塞的。。可以监听的数量是有限的 1024个
fd集合不能重复使用  每次都要进行重置
函数对文件描述符检测的操作是由内核完成的
会告诉进程有多少描述符进行IO操作
```

### poll详解

```
struct pollfd {
int fd;              //* 待检测事件的fd 
short events;       / 关心的事件组合
short revents;     //实际发生的事件组合   **比select高效的地方，将检测的和发生的事件进行分离 
};

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
- 参数：
- fds : 是一个struct pollfd 结构体数组，这是一个需要检测的文件描述符的集合 - nfds : 这个是第一个参数数组中最后一个有效元素的下标 + 1 
- timeout : 阻塞时长 0 : 不阻塞 -1 : 阻塞，当检测到需要检测的文件描述符有变化，解除阻塞 >0 : 阻塞的时长
- 返回值： -1 : 失败 >0（n） : 成功,n表示检测到集合中有n个文件描述符发生变化
```

### epoll详解

```
int epoll_create(int size); 
- 参数：size : 目前没有意义了。随便写一个数，必须大于0 
- 返回值： -1 : 失败 > 0 : 文件描述符，
操作epoll实例的
typedef union epoll_data {
void *ptr; 
int fd; 
uint32_t u32; 
uint64_t u64; 
} epoll_data_t;

struct epoll_event {
uint32_t events; //表示epoll的检测事件，常见的有EPOLLIN/EPOLLOUT/EPOLLERR
epoll_data_t data; /* User data variable */
};

//events可以取以下值
   EPOLLIN：表示对应的文件描述符可以读；
   EPOLLOUT：表示对应的文件描述符可以写；
   EPOLLPRI：表示对应的文件描述符有紧急的数可读；

   EPOLLERR：表示对应的文件描述符发生错误；
    EPOLLHUP：表示对应的文件描述符被挂断；
   EPOLLET：    ET的epoll工作模式；
   
 
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); -  - epfd : epoll实例对应的文件描述符 
- op : 要进行什么操作 EPOLL_CTL_ADD: 添加 EPOLL_CTL_MOD: 修改 EPOLL_CTL_DEL: 删除
- fd : 要检测的文件描述符
- event : 检测文件描述符什么事情

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout); - 参数：- epfd : epoll实例对应的文件描述符 
- events : 传出参数，保存了发送了变化的文件描述符的信息
- maxevents : 第二个参数结构体数组的大小 
- timeout : 阻塞时间 
   - 0 : 不阻塞 - 
   -1 : 阻塞，直到检测到fd数据发生变化，解除阻塞 - 
   > 0 : 阻塞的时长（毫秒） 
- 返回值：
   -成功，返回发送变化的文件描述符的个数 > 0 
   -失败 -1
```

### epoll的工作模式-及程序实现

```
LT 模式 （水平触发）
假设委托内核检测读事件 -> 检测fd的读缓冲区
读缓冲区有数据 - > epoll检测到了会给用户通知
a.用户不读数据，数据一直在缓冲区，epoll 会一直通知
b.用户只读了一部分数据，epoll会通知
c.缓冲区的数据读完了，不通知
LT（level - triggered）是缺省的工作方式，并且同时支持 block 和 no-block socket。在这
种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操
作。如果你不作任何操作，内核还是会继续通知你的。

ET 模式（边沿触发）    更高效  如果用户只读了一部分数据，epoll不通知，省去了重复通知
假设委托内核检测读事件 -> 检测fd的读缓冲区
读缓冲区有数据 - > epoll检测到了会给用户通知
a.用户不读数据，数据一致在缓冲区中，epoll下次检测的时候就不通知了
b.用户只读了一部分数据，epoll不通知。不会多次重复通知***所以效率比较高
c.缓冲区的数据读完了，不通知

因为ET模式下要求是一次性读取全部数据，所有要求是非阻塞的。。
以避免一个文件句柄的阻塞读/写把处理多个文件描述符的任务饿死。。
```

listenEvent和connectEvent的ET和LT的区别

```
1.当出现新连接的时候，因为ET只通知一次，而有可能一次同时进入多个连接
所以应该一次读取多个连接
```

![image-20210708205634264](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708205634264.png)

```
2.connEvent的ET模式
```

![image-20210708205723738](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708205723738.png)

![image-20210708205814383](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708205814383.png)

#### ET下的读事件

```
当读到的长度不为空时，一直读取
```

![image-20210708210045076](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708210045076.png)

#### ET下的写事件

```
ssize_t HttpConn::write(int* saveErrno) {
    ssize_t len = -1;
    do {
 ......
         } while(isET || ToWriteBytes() > 10240);
    return len;
    
//如果为ET模式，也一次写完
```

### 触发模式的管理

```
在不同的场景下有不同的需求
为了方便管理，可以设置一个传入参数，根据参数的值，选择不同的触发模式

方便了不同模式下的切换
```

![image-20210708210739265](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708210739265.png)

![image-20210708210848146](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210708210848146.png)

### Reactor模式的程序实现

#### 0.初始化设置  InitSocket_

```
创建监听套接字
设置IO多路复用
绑定服务器本机地址
监听
```

#### 1.事件的收集--start函数

```
类似muduo的eventloop

        int eventCnt = epoller_->Wait(timeMS);
        for(int i = 0; i < eventCnt; i++) {
            int fd = epoller_->GetEventFd(i);           // 获取事件对应的fd
            uint32_t events = epoller_->GetEvents(i);   // 获取事件的类型  //检测到可写
            if(fd == listenFd_) {
                DealListen_();   // 处理--监听--的操作，接受客户端连接
            }      
            else if(events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
                CloseConn_(&users_[fd]);    // 关闭连接
            }
            else if(events & EPOLLIN) {       
                DealRead_(&users_[fd]);    
            }
            else if(events & EPOLLOUT) {   
                DealWrite_(&users_[fd]);      
            } else {
                LOG_ERROR("Unexpected event");
            }
        }
    }
```

#### 2.事件的分发

##### 1.新连接事件----DealListen_

```
直接在主线程处理
```

##### 2.可读事件---DealRead_

```
加入到线程池处理
threadpool_->AddTask(std::bind(&WebServer::OnRead_, this, client));
```

##### 3.可写事件---DealWrite_

```
加入到线程池处理
threadpool_->AddTask(std::bind(&WebServer::OnWrite_, this, client));
```

