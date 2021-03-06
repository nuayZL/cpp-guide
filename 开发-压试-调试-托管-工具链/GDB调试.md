[TOC]



### GDB命令总结

![image-20210705200453896](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210705200453896.png)

![image-20210705200516800](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210705200516800.png)

### 调试常用命令详解

### 设置断点

| 格式                  | 说明                                             |
| --------------------- | ------------------------------------------------ |
| break <函数名>        | 对当前正在执行的文件中的指定函数设置断点         |
| break <行号>          | 对当前正在执行的文件中的特定行设置断点           |
| break <文件名:行号>   | 对指定文件的指定行设置断点，最常用的设置断点方式 |
| break <文件名:函数名> | 对指定文件的指定函数设置断点                     |
| break <+/-偏移量>     | 当前指令行+/-偏移量出设置断点                    |
| break <*地址>         | 指定地址处设置断点                               |

```
tb  设置临时断点
可以通过info break  查看断点信息
```

```
delete<断点编号>   删除断点和监视点
disable<断点编号>   禁用断点
enable<断点编号>    启动断点
```

```
break_if  设置条件断点
```

### 程序运行和查看

``` 
run(r)   程序运行
continue（c）  使运行的程序继续执行
list     展开源码
next（n）  运行到下一行
util      运行到某一行停止
jump    程序跳转到指定行
```

### 函数调试

```
step  进入调用函数
finish  返回调用函数
return  结束当前调用函数并返回指定值给上一层函数调用

backtrace（bt）  展开堆栈  查看函数的调用关系
f x  进行到指定的堆栈层
```

### 变量信息查看

```
printf 命令查看值的历史
 show value 命令可以显示历史中的最后 10 个值
ptype  显示变量的类型

***display  跟踪一个变量  每次停下来的偶显示它的值
watch	使你能监视一个变量的值而不管它何时被改变
rwatch	指定一个变量，如果这个变量被读，则暂停程序运行，在调试器中显示信息，并等待下一个调试命令。
awatch	指定一个变量，如果这个变量被读或者被写，则暂停程序运行，在调试器中显示信息，并等待下一个调试命令。参考rwatch和watch命令
```

### 进程调试

```
attach  pid   调试已经启动的进程
detach  pid   和进程分离
 
info inferiors   //显示所有进程
inferior <infer number>   切换调试的进程
```

### 多线程调试

```
info threads  //该命令会显示所有可以调试的线程
thread ID    切换为指定ID的线程
set scheduler-locking on/off  设置只运行当前线程或所有线程并发执行
thread apply ID1 ID2 command	让一个或者多个线程执行GDB命令command
thread apply all command	让所有被调试线程执行GDB命令command
```

### 退出

```
quit  退出gdb
kill [filename]	终止正在调试的程序
```

### GDB总结

```
通过对服务器开发星球专栏的理解学习，在专栏《Redis 6.0源码解析专题》。大致可以清楚GDB调试的使用。例如一个大型的程序通常都包含很多的功能。层层嵌套，相对也很复杂。如果相对某一个功能进行认识和学习。比如说想要对加收客户端的连接进行学习的话。。我们可以先用VS code对于要研究的函数进行搜索（对于小型程序通常是在行处打断点。对于大型程序通常是在函数处打断点。）。。然后找到相应的函数（因为底层的函数都是相同的）。。找到之后就可以Ctlr+C退出正在运行的程序。然后打想要研究的函数处打断点。。然后再重新运行。。就会在断点处的函数停止。。。通过bt命令就可以研究其的堆栈的关系。。然后通过frame等GDB命令就可以在某一层堆栈处展开。。然后可以看到相关的代码。。

总结：找底层函数--重新启动--研究堆栈--展开某层次

用GDB调试可以研究其整体的框架和研究的思路。。
```

```
查看一个函数被调用的层次关系，，

比如在webserver服务器中，知道很多函数都层层封装了bind函数，这时候就可以通过对函数进行断点，然后运行，就可以知道函数的被调用的层次关系，，来确定经过了哪些类和哪些函数。。

比如查看parse函数什么时候被调用和执行可以通过对parse函数断点来查看

总结：nginx架构分析里边介绍的很多流程应该就是按照GDB调试得到的其实行的顺序
```

