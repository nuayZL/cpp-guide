# **1.****概念**

## **简介**

与进程（process）类似，线程（thread）是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中

包括初始化数据段、未初始化数据段，以及堆内存段。（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）

进程是 CPU 分配资源的最小单位，线程是操作系统调度执行的最小单位。

线程是轻量级的进程（LWP：Light Weight Process），在 Linux 环境下线程的本质仍是进程。

查看指定进程的 LWP 号：ps –Lf pid

## **进程线程区别**

进程间的信息难以共享。由于除去只读代码段外，父子进程并未共享内存，因此必须采用一些进程间通

信方式，在进程间进行信息交换。

调用 fork() 来创建进程的代价相对较高，即便利用写时复制技术，仍然需要复制诸如内存页表和文件描

述符表之类的多种进程属性，这意味着 fork() 调用在时间上的开销依然不菲。

线程之间能够方便、快速地共享信息。只需将数据复制到共享（全局或堆）变量中即可。

创建线程比创建进程通常要快 10 倍甚至更多。线程间是共享虚拟地址空间的，无需采用写时复制来复

制内存，也无需复制页表。

![image-20210706094950696](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210706094950696.png)

## **线程之间共享和非共享资源**

共享资源

进程 ID 和父进程 ID

进程组 ID 和会话 ID

用户 ID 和 用户组 ID

文件描述符表

信号处置

文件系统的相关信息：文件权限掩码（umask）、当前工作目录

虚拟地址空间（除栈、.text）

非共享资源

线程 ID

信号掩码

线程特有数据

error 变量

实时调度策略和优先级

栈，本地变量和函数的调用链接信息

**NPTL**

当 Linux 最初开发时，在内核中并不能真正支持线程。但是它的确可以通过 clone() 系统调用将进程作为可调度的实体。这个调用创建了调用进程（calling process）的一个拷贝，这个拷贝与调用进程共享相同的地址空间。

LinuxThreads 项目使用这个调用来完成在用户空间模拟对线程的支持。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步等方面都存在问题。另外，这个线程模型也不符合 POSIX 的要求。

要改进 LinuxThreads，需要内核的支持，并且重写线程库。有两个相互竞争的项目开始来满足这些要求。一个包括 IBM 的开发人员的团队开展了 NGPT（Next-Generation POSIX Threads）项目。同时，Red Hat 的一些开发人员开展了 NPTL 项目。NGPT 在 2003 年中期被放弃了，把这个领域完全留给了NPTL。

NPTL，或称为 Native POSIX Thread Library，是 Linux 线程的一个新实现，它克服了 LinuxThreads的缺点，同时也符合 POSIX 的需求。与 LinuxThreads 相比，它在性能和稳定性方面都提供了重大的改进。

查看当前 pthread 库版本：getconf GNU_LIBPTHREAD_VERSION

# **2.** **线程操作函数**

## Linux线程库

### 创建线程

```
int pthread_create(pthread_t *thread,
                 const pthread_attr_t *attr,
                 void * (*start_routine) (void *),
                 void *arg);
//参数 thread 个输出参数，如果线程创建成功，则通过这个参数可以得到创建成功的线程ID 
参数 attr 指定了该线程的属性，般被设置为 NULL 表示使用默认的属性参数 
参数start routine 定了线程函数,函数的调用方式必须是cdecl，(即C/C++ 中定义全局函数时默认的调用方式)
参数arg表示要传入线程函数的参数
pthread_t pthread_self(void); 
//在当前线程中获取线程ID
int pthread_equal(pthread_t t1, pthread_t t2);
void pthread_exit(void *retval);
int pthread_join(pthread_t thread, void **retval); 
//等待某线程退出并接受其返回值
int pthread_detach(pthread_t thread); 
int pthread_cancel(pthread_t thread);
```

### 线程属性相关：

```
线程属性类型
pthread_attr_t int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate); 
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
```

### 线程ID的获取方法

```
1.
pthread_t tid; 
pthread_create(&tid, NULL, thread-proc , NULL);
2.
pthread t tid=pthread self();
3.
int tid=syscall(SYS gettid);

方法1&2得到的结果是一样的，都是pthread_t类型，输出内存空间地址。
转换成16进程时和pstack命令看到的线程ID是一样的
方法3的得到的是轻量级进程的ID
```

### 等待线程结束--join--汇接操作

```
使用场景--一个线程等待另一个线程执行完任务并退出后在继续执行。
即两个线程执行上有先后顺序并由信息交换的时候
```

```
int pthread_join(pthread_t thread, void** retval);  
//参数thread 需要等待的线程 ID; 
//参数retval 是输出参数，用于接收等待退出的线程的退出码 (Exit Code)

在等待目标线程期间会挂起当前线程。
```

## C++11提供的std::thread类

```
为了解决线程api使用必须使用规定格式的问题,C++11引入了新的类std::thread（在头文件thread）
可将任意签名形式的函数作为线程函数

//同一了Linux和Windows的线程创建函数
```

### 线程对象和线程函数

std::thread对象在函数运行期间必须是有效的

```
void threadproc(){
while (true)  printf( " I am New Thread!\n" ) ; 
}
void func (){
std: :thread t(threadproc); 
}
int main () 
{
    func () ; 
    11 权宜之计，让主线程不要退出
    while (true)
    return 0;
}

以上程序会出现问题：
func函数调用结束后，func中的局部变 t(线程对象)被销毁，而此时线程函数仍在运行 
所以在使用 std: :thread类时，必须保证在线程函数运行期间其线程对象有效 
std: :thread 对象提供了 detach方法，通过这个方法可以让线程对象与线程函数脱离关系，这样即使线程对象被销毁，也不影响线程函数的运行 
但是实际中不推荐这么做  应该尽可能用线程对象去控制和管理线程的声明周期
```

### 获取当前线程ID的方法

```
使用类静态方法 thread类的get_id获取
得到的是thread::id包装类型,不能被直接强制转换成整形
可以直接用cout<<输出流输出，或者先转换为ostringstream再转为字符串类，再转为我们需要的整形
```

### 等待线程结束

```
#include <pthread .h> 
void pthread exit(void* value ptr);
```

## **将C++类对象实例指针作为线程函数的参数

```
线程函数不能是类的实例方法，必须是类的静态方法
因为C++编译器在翻译时会将类的实例对象地址（也就是this指针操作第一个参数）
vo d* threadFunc(Thread* this， void* arg);
以上就不符合线程函数的签名要求

使用C++11的std::thread就没有这个限制
但必须显式地将线程函数所属的类对象实例指针(在类的内部就是this指针)作为构造函数参数传递给 std::thread。

在实际开发中往往会在创建线程时将当前对象的地址( this 指针)
传递给线程函数，然后在线程函数中将该指针转换为原来的类实例，再通过这个实例就可以访问类的所有方法了

在线程函数中创建线程(调用 CreateThread pthread_ create 方法)时，将当前对象this 指针作为线程函数的唯一参数传入，这样在线程函数中就可以通过线程函数的参数得到对象的指针了，通过这个指针可以自由访问类的实例方法

C++11的语法还允许使用std::bind工具给函数绑定this指针
```



# **3.**线程同步

## 概述

```
线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的：必须确保多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。
临界区是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应终端该片段的执行。
线程同步：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。

互斥量
为避免线程更新共享变量时出现问题，可以使用互斥量（mutex 是 mutual exclusion的缩写）来确保同
时仅有一个线程可以访问某项共享资源。可以使用互斥量来保证对任意共享资源的原子访问。
互斥量有两种状态：已锁定（locked）和未锁定（unlocked）。任何时候，至多只有一个线程可以锁定
该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报错失败，具体取决于加锁时
使用的方法。
一旦线程锁定互斥量，随即成为该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每
一共享资源（可能由多个相关变量组成）会使用不同的互斥量，每一线程在访问同一资源时将采用如下
协议：
针对共享资源锁定互斥量
访问共享资源

对互斥量解锁
如果多个线程试图执行这一块代码（一个临界区），事实上只有一个线程能够持有该互斥量（其他线程
将遭到阻塞），即同时只有一个线程能够进入这段代码区域，如下图所示：
```

![image-20210706094515024](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210706094515024.png)

## 互斥量

```
互斥量相关操作函数：
互斥量的类型 pthread_mutex_t int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr); int pthread_mutex_destroy(pthread_mutex_t *mutex); int pthread_mutex_lock(pthread_mutex_t *mutex); int pthread_mutex_trylock(pthread_mutex_t *mutex); int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

```
死锁
有时，一个线程需要同时访问两个或更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超
过一个线程加锁同一组互斥量时，就有可能发生死锁。
两个或两个以上的进程在执行过程中，因争夺共享资源而造成的一种互相等待的现象，若无外力作用，
它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁。
死锁的几种场景：
忘记释放锁
重复加锁
多线程多锁，抢占锁资源
```

![](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210706094600153.png)

## 读写锁

```
当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，
当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由
于互斥锁的排它性，所有其它线程都无法获取锁，也就无法读访问共享资源了，但是实际上多个线程同
时读访问共享资源并不会导致问题。
在对数据的读写操作中，更多的是读操作，写操作较少，例如对数据库数据的读写应用。为了满足当前
能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现。
读写锁的特点：
如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。
如果有其它线程写数据，则其它线程都不允许读、写操作。
写是独占的，写的优先级高。
```

```
读写锁相关函数：
读写锁的类型 pthread_rwlock_t int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock); 
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock); 
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock); 
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

## **生产者消费者模型**

![image-20210706094709160](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210706094709160.png)

## **条件变量**

```
条件变量的类型 pthread_cond_t int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr); int pthread_cond_destroy(pthread_cond_t *cond); int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex); int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime); int pthread_cond_signal(pthread_cond_t *cond); int pthread_cond_broadcast(pthread_cond_t *cond);
```

## 信号量

```
信号量的类型 
sem_t int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_destroy(sem_t *sem); 
int sem_wait(sem_t *sem); 
int sem_trywait(sem_t *sem); 
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout); int sem_post(sem_t *sem); 
int sem_getvalue(sem_t *sem, int *sval);
```

# 相关问题

## 某个线程崩溃，是否会导致进程的退出

```
一般来说，每个线程都是独立执行的单位，都有自己的上下文堆枝， 个线程崩溃不会但是在通常情况下，一个线程崩溃也会导致整个进程退出 例如Linux 操作系统中可能会产生 Segment Fault 错误，这个错误会产生 个信号，操作系统对这个信号的默认处理就是结束进程，这样整个进程都被销毁，在这个进程中存在的其他线程自然也就不存在了
```

## Linux命令

### pstack

```
pstack pid    //pid指示当前的进程ID
可以通过pstack命令查看一个进程的线程数量和该进程下每个线程的调用堆栈情况
```

pstack和top

```
排查和定位一个进程CPU占用率过高的问题
使用流程：
1.使用top 查看高占用率的进程
2.top -H  显示每个进程各个线程的运行状态(线程模式)
3.使用pstack ID（进程id） 可以查看该进程所有线程此时的堆栈（线程正在做什么）

```

