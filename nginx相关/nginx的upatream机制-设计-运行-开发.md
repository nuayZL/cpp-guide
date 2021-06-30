总结

```
nginx的upstream是怎么运行的。
有什么功能？
怎么才能在web服务器上实现？upstream是如何介入运行机制的？
怎么判断是上游服务器发来的消息呢。
（个人认为是通过设置回调函数到读事件，当有可读事件时直接就调用回调）
怎么设计和开发一个upstream模块
```

```
upstream首要的功能是透传（将第三方服务器的内容原封不动的返回给用户）。
当访问第三方是为了获取信息，再根据这些信息构造响应时，通常用subrequest方式。
并发访问数只会受制于物理内存的大小，完全可以轻松达到几十万的并发TCP连接

功能时使得Nginx的能力不局限在但服务器范围、提供全异步，高性能的请求转发
upstream框架时反向代理的基础，使Nginx成为网络里的核心节点，建立大型的服务网络

不仅支持HTTP协议，还支持FastCGi memcached Redis等任意协议

*****upstream模块是如何介入操作的
怎么将普通请求设置为请求转发
将ngx_http_upstream_init作为http_read_client_request_body的回调函数
(这样在接受请求体数据之后就会回调ngx_http_upstream_init()启动upstream框架)
或者类似memcached中的一样在配置指令的解析函数中将ngx_http_upstream_init()注册到loc中
```

![image-20210630100043533](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630100043533.png)

```
是一类特殊的http模块，相当于content handler模块，
工作在NGX_HTTP_CONTENT_PHASE   在内容产生流程的关键节点上定义若干回调函数，
实现了大部分的底层网络收发逻辑和Nginx处理流程，并在关键节点定义了回调函数
```

请求转发机制的基本工作流程

![image-20210630100908144](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630100908144.png)

upstream执行的一般流程

![image-20210630092204530](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630092204530.png)

配置参数

```
ngx_http_upstream_t的conf成员，用于设置模块处理请求时的参数
包括连接/发送/接受的超时时间/
connect_timeout
send_timeout
read_timeout
必须设置，否则无法与上游服务器建立TCP连接
```

启动机制

```
ngx_http_upstream_init启动upstream
```

回调方法的执行场景

![image-20210630094101301](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630094101301.png)

重要结构体

```
ngx_http_upstream_t请求转发的关键结构体{//分为三部分：基本参数--处理响应的回调-发送/接受请求的回调
1--基本参数
count   请求的引用计数
peer    连接结构体
conf   连接参数配置
resolved  上有服务器地址
request_bufs   发送的请求数据
out_bufs  从上游接收到的数据
subrequest_in_memory   当为0的时候就不经任何处理就发送给客户端

2--处理上游响应数据的回调
```

![image-20210630101649535](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630101649535.png)

```
3--发送接受请求的回调
```

![image-20210630101737580](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630101737580.png)

常用函数（回调方法）的执行场景（可实现九个回调函数，但以下四个必须由upstream模块提供）

```
***具体的回调函数根据协议的不同会有所不同，见memcached模块.。前四个函数必须实现。
create_request   //生成发送到上游的请求数据
reinit_request  //在第一次试图建立连接失败时，会再次重连上游服务器
finalize_request在请求被销毁时会调用
process_header用于解析接受上游服务器返回的TCP响应头部，会被多次调用（和正常的接受请求行请求头时一样，因为大小不确定，所以会被多次调用）

rewrite_redirect重定向

input_filter_init分配内存用于存放解析的中间状态
input_filter实际处理包体的方法
（如果不进行input_filter和input_filter_init就是透传操作）

全部九个回调函数
```

![image-20210630102109470](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630102109470.png)

初始化

```
要使用请求转发机制 必须要调用ngx_http_upstream_create() c创建对象
即创建http_upstream_t结构体

然后根据http_upstream_conf_t的结构体设置upstream_t结构体里的参数

```

启动连接

```
在设置好回调函数和连接参数后，调用ngx_http_upstream_init

将ngx_http_upstream_init作为http_read_client_request_body的回调函数
(这样在接受请求体数据之后就会回调ngx_http_upstream_init()启动upstream框架)
```

http_memcached_module源码分析

```
配置数据结构
http_memcached_loc_conf_t{
}
配置指令{
memcached_pass   启动upstream框架
..其余都是设置各种连接参数
}
指令解析函数（注册处理函数）{
解析url
向当前的loc注册处理函数ngx_http_memcached_hanler
}
ngx_http_memcached_hanler{   //三部分---参数初始化--设置回调函数---启动
设置连接参数/增加引用计数/
设置在回调函数。（与协议相关）
启动upstream（ngx_http_upstream_init） 
}
```

开发upstream模块--1--Echo

```
配置信息类{
设置超时时间和缓冲区大小四个基本参数
}
业务逻辑类{
UpstreamCallback类   实现回调函数
UpstreamHandler      组装回调函数并启动框架
}
模块集成类{
}
在指令解析函数注册处理函数{

}
总结；开发步骤繁琐，需要在配置解析阶段做的事情比较多，要这是各种参数和回调
```

开发upstream模块--2--

```
访问mytest模块的URL参数作为搜索引擎的关键字，用upstream访问google。查询参数
请求上下文{
保存解析的状态，是能正确回调process_header方法
}
create_request{
主要的业务处理逻辑都在里边
}
process_header方法解析包头{
响应行和响应头都使用状态机
}
finalize_request方法释放资源
ngx_http_mytest{    配置指令mytest的指令解析函数
向匹配的loc注册http_mytest_hander函数
}
***http_mytest_handler方法{---主要的三个步骤都在里边执行
设置配置参数
设置回调方法
启动upstream
}
```

概述

```
是事件驱动框架和HTTP框架的综合
无阻塞的实现与上游服务器的交互有很好解决了一个请求，多个TCP连接，多个读写事件的复杂关系。
不仅可以与上游交互，还可以转发上游协议的响应到下游客户端。
高效能/高效率/高度灵活性
访问上游服务器的六个步骤{
启动upstream机制
连接上游服务器
发送请求
接受上游响应头
接受上游响应包体
接受请求
}

上游服务器可以是动态服务器--Apache/Tomcat
           或者键值存储系统memcached
           或者关系型数据库 Mysql
对于客户端可以派生出很多子请求   子请求可以访问上游服务器
每个request_t可以访问一个上游服务器

```

