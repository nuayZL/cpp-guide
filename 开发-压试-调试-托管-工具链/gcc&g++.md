### GCC简介

```
GCC 原名为 GNU C语言编译器（GNU C Compiler）

GCC（GNU Compiler Collection，GNU编译器套件）是由 GNU 开发的编程语言译器。GNU 编译

器套件包括 C、C++、Objective-C、Java、Ada 和 Go 语言前端，也包括了这些语言的库（如libstdc++，libgcj等）

GCC 不仅支持 C 的许多“方言”，也可以区别不同的 C 语言标准；可以使用命令行选项来控制编译器

在翻译源代码时应该遵循哪个 C 标准。例如，当使用命令行参数 -std=c99 启动 GCC 时，编译器

支持 C99 标准。

安装命令 sudo apt install build-essential

查看版本 gcc/g++ -v/--version
```

### GCC工作流程

![image-20210705202245504](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210705202245504.png)

![image-20210705202255378](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210705202255378.png)

### GCC常用参数选项

![image-20210705202508859](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210705202508859.png)

### 可以使gcc在4个阶段中的任何一个阶段停止下来

```
-E
预编译后停下来，生成后缀为 .i 的预编译文件。
-c
编译后停下来，生成后缀为 .o 的目标文件。
-S
汇编后停下来，生成后缀为 .s 的汇编源文件。

第一步：进行预编译，使用 -E 参数
gcc -E test.c -o test.i
查看 test.i 文件中的内容，会发现 stdio.h 的内容确实都插到文件里去了，而其他应当被预处理的宏定义也都做了相应的处理。
第二步：将 test.i 编译为目标代码，使用 -c 参数
gcc -c test.c -o test.o
第三步：生成汇编源文件
gcc -S test.c -o test.s
第四步：将生成的目标文件链接成可执行文件
gcc test.o - o test
```

### gcc和g++编译C文件

```
编译C文件都是可执行的
gcc和g++所做的事情确实是一样的，g++在编译C文件时调用了gcc
```

### gcc和g++编译C++文件

```
g++没有问题
gcc会报错
```

```
g++在内部做了处理、默认编译C++程序
如果遇到C程序、会直接调用gcc编译
```

