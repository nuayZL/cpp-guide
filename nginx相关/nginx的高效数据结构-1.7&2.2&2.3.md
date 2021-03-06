```
为了做到跨平台，Nginx定义，封装了一些基本的数据结构。
为了节省内存，nginx数据结构都尽可能的少占用内存。
```

nginx的高效数据结构

```
任何复杂的程序都需要用到数据/链表/树等数据结构，这些容器可以使用户忽略底层细节，快速开发处高级数据结构，实现复杂的业务。

定时器通过红黑树实现

server虚拟主机会以散列表的数据结构组织起来，高效查询
键时server name 值时ngx_http_core_srv_conf_t结构体指针
location表达式会以静态的二叉查找树的组织起来，高效查询

每一个location块通过双向链表说父配置块关联

request数据结构中所有的子请求都是通过单链表连接起来的
```

内存池Ngxpool3.4

```
对内存分配比较吝啬，只有保持低内存消耗，才能实现十万甚至百万级别的同时并发连接数据。nginx的并发连接仅受限于内存的大小，
是对象池模式的具体应用，一次性向系统申请大块的内存，内部自行切割分配。
优点；减少系统调用，避免内存泄漏，避免内存碎片

负责内存分配与释放的函数ngx_palloc()/pnalloc()/pcalloc()
```

![image-20210701093850527](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701093850527.png)

字符串

```
因为HTTP协议都是基于纯文本的协议，所有nginx必须正确，高效的处理字符串
nginx的字符串有len 和data两个成员  data执行字符串的首地址。拷贝和修改很高效 
其实际为字符串引用，并不持有，非常轻量级、
应尽量以只读的方式使用。
```

