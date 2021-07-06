## 基本概念（锁/锁属性）

```
保护临界区对象的方法
限制多个线程同时执行某段代码来保护资源
使用数据结构pthread_mutex_t表示互斥体对象
在设置互斥体对象属性时需要创建pthread_mutexattr_t类型对象

锁的功能是保护一段执行区域的执行是连续的。将一系列操作变成原子操作
```

## 两种初始化方法（锁/锁属性）

```
1.#include <pthread.h> 
pthread_mutex_t mymutex =PTHREAD_MUTEX_INITIALIZER;
2.对于动态分配的互斥体或者需要设置相关属性的时候，用如下方法
int pthread_mutex_init(pthread_mutex_t* restrict mutex, 
const pthread mutexattr t* restrict attr);
//参数 mutex 即我们需要初始化的 mutex的指针
//参数 ttr 是需要设置的互斥体属性

int pthread_mutexattr_init(pthread_mutexattr_t* attr);

```

### 这是和获取锁的属性类型

```
int pthread_mutexattr_settype(pthread_mutexattr_t* attr, int type); 
int pthread_mutexattr_gettype(const pthread_mutexattr_t* restrict attr, int* restrict type);
```

### 锁的属性

```
1.PTHREAD MUTEX NORMAL (普通锁)默认属性
//一个线程对普通锁加锁以后，其他线程会阻塞在 pthread_mutexlock 调用处，直到对互斥体加锁的线程释放了锁
//pthread mutex trylock 函数如果拿不到锁， 则也不会阻 ，而是会立即返回 EBUSY 错误码
2.PTHREAD MUTEX ERRORCHECK (检错锁)
//如果某一个线程重复调用pthread_mutex _lock会直接返回EDEADLOCK
//其他线程如果对这个互斥体再次调用 pthread_mutex_1ock ，则会阻塞在该函数的调用处
3.PTHREAD_MUTEX_RECURSIVE (可重入锁)
//允许同一个线程对持有的互斥体重复加速，
//每成功调用 pthread_ mutex _lock一次，该互斥体对象的锁引用计数就会增 ，相反，每成功调用 pthread_ mutex _ unlock一次，锁引用计数就会减1,当锁引用计数值为0时，允许其他线程获得该锁，
```



## 互斥体的销毁

```
int pthread mutex destroy(pthread mutex t* mutex);
//不能销毁正在枷锁或正在被条件变量使用的互斥体对象

int pthread_mutexattr_destroy(pthread_mutexattr_t* attr);
```

## 加锁和解锁操作

```
int pthread_mutex_lock(pthread_mutex_t* mutex); 
//当对普通锁加锁后，其他线程会阻塞在该方法处
int pthread_mutex_trylock(pthread_mutex_t* mutex); 
int pthread_mutex_unlock(pthread_mutex_t* mutex);
```

使用规范

```
如创建互斥体对象后再对其加锁
加锁后才对其进行解锁操作
解锁后才进行销毁操作
```

