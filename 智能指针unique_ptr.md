### 概述

```
unique_ptr 是 C++ 11 提供的用于防止内存泄漏的智能指针中的一种实现，独享被管理对象指针所有权的智能指针。

unique_ptr对象包装一个原始指针，并负责其生命周期。当该对象被销毁时，会在其析构函数中删除关联的原始指针。
```

```
unique_ptr对象始终是关联的原始指针的唯一所有者。我们无法复制unique_ptr对象，它只能移动。

由于每个unique_ptr对象都是原始指针的唯一所有者，因此在其析构函数中它直接删除关联的指针，不需要任何参考计数。
```

### 为什么使用指针

```

因为对于自己定义的类往往要在堆上创建对象，
所以要得到指向其的指针，
比如以下
    std::unique_ptr<HeapTimer> timer_;          // 定时器（类）
    std::unique_ptr<ThreadPool> threadpool_;    // 线程池(类)
    std::unique_ptr<Epoller> epoller_;          // epoll对象
其都会通过new运算符得到指向一块动态内存的指针

？为什么连接类没有动态指针
因为连接类比较占内存的就是缓冲区  而缓冲区是用vector实现的，其是在堆中创建的。
```

![image-20210711193211906](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711193211906.png)

### 应用场景：

```
1 为动态申请的资源提供异常安全保证
2 返回函数内动态申请资源的所有权
3 在容器中保存指针
4 管理动态数组
```



### 实例：

```
以下为服务器类的线程池的创建和初始化
先创建了一个空的智能指针
以下可以说threadpool_类包装了线程池，负责其生命周期。该类销毁时，在析构函数中删除关联的原始指针。
```

![](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711173538316.png)

![image-20210711173632353](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711173632353.png)

```
另外服务器类的几个主要的成员类也都是使用unique_ptr管理
```

![image-20210711174924327](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711174924327.png)

### 特点：

```
unique_ptr 独享所有权

1 unique_ptr独占管理对象，只有移动语义。
2 unique_ptr可以不占用对象，即为空。可以通过reset()或者赋值nullptr释放管理对象。
3 标准库早期版本中定了auto_ptr，它具有unique_ptr的部分特征，但不是全部，例如不能在容器中保存auto_ptr,不能从函数中返回auto_ptr等等，这也是unique_ptr主要的使用场景。

unique_ptr对象始终是关联的原始指针的唯一所有者。我们无法复制unique_ptr对象，它只能移动。
由于每个unique_ptr对象都是原始指针的唯一所有者，因此在其析构函数中它直接删除关联的指针，不需要任何参考计数。
```

### 创建

#### 创建一个空的 unique_ptr 对象

创建一个空的`unique_ptr<int>`对象，因为没有与之关联的原始指针，所以它是空的。

```
std::unique_ptr<int> ptr1;
```

#### 使用原始指针创建 unique_ptr 对象

要创建非空的 unique_ptr 对象，需要在创建对象时在其构造函数中传递原始指针，即：

```
std::unique_ptr<Task> taskPtr(new Task(22));
```

#### 使用 std::make_unique 创建 unique_ptr 对象 / C++14

```
std::unique_ptr<Task> taskPtr = std::make_unique<Task>(34);
```

#### unique_ptr 对象不可复制、只能移动

```
通过移动，转移所有权
std::unique_ptr<Task> taskPtr2(new Task(55));
std::unique_ptr<Task> taskPtr4 = std::move(taskPtr2);
```

