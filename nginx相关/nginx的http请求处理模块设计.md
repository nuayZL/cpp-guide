```
核心模块更像是框架，
其余的模块更像是一个函数。一个模块实现一个功能。
```

http模块是如何介入

```
worker进程会在一个for循环反复调用事件模块检测网络事件
```

![image-20210630225027316](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630225027316.png)

```
对上图的总结:
1.事件模块检测网络事件，建立连接后会根据配置文件交给HTTP框架处理
2.框架负责接受完成的请求头/请求体。再根据请求的URL等分配到location
3.在处理请求结束时，会依次调用过滤模块，(决定是否压缩等等)
```

```
一个location相当于对应一个文件夹(源目录)，可以匹配文件夹内的任意的html
```

两种调用方式

```
1.如果想使得某一模块（功能)对任一请求都生效，如access_moduel..其在ACCESS_phase阶段工作，对任意模块都生效。
通过module_ctx里的postconfigutation来注册处理函数。想content阶段添加handler。。即向http_core_main_conf_t结构里的phases数组添加元素。
```

![image-20210701092138100](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701092138100.png)

![image-20210701092740123](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701092740123.png)

```
2.而自己设计的大部分模块都只需要对匹配了某一location的请求生效即可，即content handler方式（即内容处理）注册处理函数。。。设置http_core_loc_conf_t结构的handler成员。。
下方的模块2就是使用的HttpCodeModule.handle方式
```

![image-20210701092547877](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701092547877.png)

![image-20210701092758437](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701092758437.png)

模块开发综述

```
1.模块的名字(类比函数名)
2.配置指令和参数  相当于传入参数-----------cmd
3.业务逻辑开发  (主要的功能实现)
4.框架注册   将业务逻辑加入到请求的处理
5.模块集成   实现module_t中的cmds和ctx结构里的回调函数
6.编译       将我们给定的源代码和config编译进Nginx(3.3.2两种方法)
```

```
ctx是不同类型模块的公共接口。使得每个模块都有自己的特性。实际为不同的接口函数
http模块有八个接口函数
```

```
http模块基本都需要用到请求的信息，所有请求信息都可以在http_request_t结构体中
```

模块开发1--hello world

```
配置指令 mytest 
当出现配置指令时就调用配置解析函数 http_mytest{
1.获取mytest配置项所属的配置块。的配置结构体
2.将函数指针加入到handler
}
handler{      //处理函数如下
将响应包的内容设置为hello world
构造ngx_buf_t结构体准备发送包体
将内容拷贝到ngx_buf_t指向的内存
构造发送时的chain_t结构体}
```

![image-20210630234959714](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630234959714.png)

![image-20210630235701187](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210630235701187.png)

```
clcf表示 core_loc_conf_t
ngx_conf_t结构保存了在解析配置文件时的重要数据据结构，表示当前配置指令的运行环境数据。。最重要的时handler
```

模块开发2--向客户端输出指定的字符串（8.10）

```
定义模块的配置数据结构{
ngx_str_t  msg   //存储配置文件立的字符串    即配置指令后的字符串
}
NgxEchoHandler{
获取配置数据里的字符串，准备响应数据
设置响应体的长度，状态码  send函数发送配置字符串
}
set_echo{ //指令解析函数
 内容处理的注册函数
}
```

```
总结：模块开发总共分为三个类
1.ndgxxxconf  配置类  是对类配置文件的封装和ctx里的create_conf的封装
2.ndgxxxhandler  实现主要的业务逻辑
3.ndgxxxInit  指令函数cmds&功能函数（ctx）&&指令解析函数
```

filter模块概述

```
过滤模块处理一些附加的功能。
gzip可以对静态文件的处理。image_filter将静态文件制成略缩图 。效果是可以叠加的

何时被调用
特点：在ngx_http_send_filter发送包体或者ngx_http_output_filter发送包体时，
才会由这两个方法依次调用过滤模块处理请求。。
响应分为头部或者包体，所以过滤模块可以选择性的处理头部或者包体

调用顺序
过滤链表：链表的每一个元素都是独立的c源代码文件，源代码文件会通过两个static静态指针指向下一个文件中的过滤方法。
处理响应头/体的方法原型
http_output_header_filter_pt
http_output_body_filter_pt
过滤模块都需要实现这两个方法
框架的入口是http_top_header/body_filter

？？单链表中的元素都是怎么用next指针连接起来的
(以下加入的顺序和处理的顺序是正好相反的)
```

![image-20210701115153893](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210701115153893.png)

```
过滤链表的顺序
在编译过程决定模块的顺序
configure命令生成的module.c文件里模块的顺序就是过滤模块的顺序
可以通过在configure之后，make编译前  修改数组中国成员的顺序

```

模块开发-filter模块-1

```
0.模块名/配置指令cmd/模块ctx/定义过滤模块/等与http模块类似
1.实现处理头部/包体的方法
2.实现初始化方法，把本模块的方法插入到过滤模块的首部
既处理响应头也处理响应体
通过postconfiguration  将处理函数插入到过滤链表的正确位置
FooterHandler的init函数是作为http模块的ctx的postconfiguration函数指针

```

