概述

```
HTTP模块的数量远超其他的四类模块
模块高度的可定制性
HTTP框架=核心模块ngx_http_module+Http模块core_module/upstream_module

HTTP模块会做哪些工作
获取已经解析完毕的HTTP请求（ngx_http_request_t结构体）
获取自己感兴趣的不同的配置块内的配置项
调用HTTPP框架提供的方法发送响应
将请求分为顺序性的多个处理阶段
异步节后请求中的包体 将网络数据曹村到磁盘上
异步访问第三方服务
分解多个子请求构造处理复杂业务能力
```

HTTP框架概述

```
通过nginx.conf配置范例来说明框架的行为
框架的首要任务是管理所有模块的配置项
server虚拟主机会以散列表的数据结构组织起来，高效查询
location表达式会以静态的二叉查找树的组织起来，高效查询

ngx_http_module_t接口{    //完全围绕配置项进行
pre/postconfigure  在解析配置项前和配置项后回调
create/init_main_conf  创建全局配置项结构体/解析完main配置项后回调
}
```

管理HTTP模块的配置项

```
有三种不同级别的配置项
有几个配置块就需要几个用于存储配置结构体指针的数组
？？在create_main时换回调用create_srv和create_loc创建结构体
目的是为了相同配置项的合并
create_srv的时候会创建create_loc

管理main级别下的配置项
http_core_module模块完成了HTTP框架的大部分功能
框架配置项结构体 ngx_http_conf_ctx_t
ngx_http_core_module模块是第一个HTTP模块  序号是0
http_cycle_get_module_main_conf宏获取配置项

管理server级别的配置项
发现server配置块时就回调ngx_http_core_server方法
建立ngx_http_conf_ctx_t结构体
解析块内配置项的流程{
调用所有srv/loc级别方法
将属于当前server的配置项添加到server动态数组
解析server块下的全部srv级别配置项
}

管理loc级别下的配置项
location配置块就回调ngx_http_core_location方法
先建立ngx_http_conf_ctx_t结构体

不同级别配置项的合并
merge_srv_conf  合并main/srv级别的配置项
merge_loc_conf  合并main/srv/loc级别的配置项
存在的价值：举例；当一个配置项即出现在main又出现在srv时就执行merge_srv_conf
```

监听端口的管理

```
属于server配置块----由listen配置项决定
每监听一个端口，就可以使用http_conf_port_t来表示{
in_port_t port  //监听多个端口
ngx_array_t addrs  //该端口下所有的地址（存储http_conf_addr_t）
}
addrs是该端口对应的监听地址http_conf_addr_t
http_conf_addr_t中有对应的默认server{}虚拟主机（即ngx_http_core_srv_conf_t）
每一个监听地址还有ngx_listening_t与其对应
ngx_lisening_t的handler回调方法是ngx_http_init_connection

**一个端口可监听多个不同地址的
有几个要监听的端口就在http_core_main_conf中有几个http_conf_port_t结构体

```

server的快速检索

```
在配置文件中  监听端口是在http块内的  和server{}配置块是同级别
所以一个端口往往对应多个server_name
一个虚拟主机配置块都由一个ngx_http_core_srv_conf_t结构体表示
为了防止conf文件有多个server块时查找困难的问题， 实现散列表存放虚拟主机
键时server name 值时ngx_http_core_srv_conf_t结构体指针

```

location块的快速检索

```
location是在server块下的
一个server对应多个location
  每一个location块通过双向链表说父配置块关联
因为location是在nginx.conf中读取的，静态不变的，静态的完全二叉树的查找效率更高。
```

HTTP请求的11个处理阶段

```
模块化使得每个模块专注于独立/简单的功能。
这种设计使得nginx有简单/可测试/可扩展的优点

包含checker检查方法和handler方法

HTTP块解析完会根据nginx.conf产生ngx_http_phase_handler_t组成的数组
即ngx_http_phase_engine_t结构体包含所有http_phase_handler_t组成的数组

在http_core_main_conf中
关于HTTTP阶段有两个成员：phase_engine和phases
phase_engine控制运行过程中请求所要经过的http处理阶段
phases按照11个阶段的概念初始化phase_engine的handlers数组
**想在哪个阶段生效就在那个阶段的phases[xxx]中添加http_handler_pt方法

POST_READ_PHASE阶段
在接受完完整的请求头后执行该方法
realip模块在该阶段生效  

SERVER_REWRITE_PHASE阶段
checker方法是core_rewrite_phase

...//以上九个阶段专注于基础性工作：rewrite重写URI 找到location配置快 判断权限 获取静态资源
(已经书用于绝大多数请求)

NGX_HTTP_CONTENT_PHASE   核心阶段  大多数模块在此阶段定义服务器的行为
**提供了两种介入该阶段的方式
1.向全局的ngx_http_core_main_conf_t的phase数组中添加http_handler_pt处理方法（其余阶段的处理方式）
（通过postconfiguration方法添加ngx_http_handler_pt处理方法）
2.把希望处理请求http_handler_pt设置到loaction相关的core_loc_conf_t结构体的handler指针中。
（优点是不应用与所有的HTTP请求，仅应用于URI匹配location时）
checker方法是ngx_http_core_content_phase

```

***HTTP框架的初始化流程

```
当初始化过程在ngx_http_mudule模块
当出现了http{}配置快时就回调ngx_http_block方法
流程
1.初始化http模块的ctx_index序号
2.分配存放http模块结构体指针的3个数组
3.调用create_main/ser/loc_conf方法
4.所有模块的preconfiguration
5.解析main级别配置
6.调用所有模块的init_main_conf方法
7.合并配置项
8.构造location的静态二叉平衡查找树
9.处理postfiguration使添加方法到7个处理阶段
10.根据各个模块的方法构造phase_engine_handlers数组
11.设置server虚拟主机构成的支持通配符的散列表
12.构造监听端口与server间的关联关系
```

