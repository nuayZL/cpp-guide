

## function和bind

```
C++11通过function将lambda、函数指针、仿函数统一起来了。
bind用来将可调用对象与其参数一起进行绑定。绑定后的结果可以使用function来进行保存。

bind用来将可调用对象与其参数一起进行绑定。绑定后的结果可以使用function来进行保存
```

```
function<void()>是一个模板类对象，它可以用一个函数签名为void()的可调用对象来进行初始化；
```

![image-20210711161443691](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711161443691.png)

![image-20210711161558433](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711161558433.png)

![image-20210711161629359](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711161629359.png)

### 静态成员的初始化

```
一般都是类内声明，类外初始化，因为静态成员属于整个类，而不属于某个对象，如果在类内初始化，会导致每个对象都包含该静态成员。

以下为连接类的资源路径和连接总数的初始化
```

![image-20210711154806860](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711154806860.png)

# forward和move

```
1、move：实现移动语义，将左值变为右值。使用语法：move(左值)。
2、forward：可以保持数据的类型不变，可用于实现完美传参和完美转发。
```



## 原子量

```
因为采用了多线程机制
而httpcoon是属于静态成员
所以同一时间有可能有多个线程对其进行操作。
所以应该将其设置成原子量

static std::atomic<int> userCount;    
```

## const常量

```
对于错误码和对应的html界面等可以用常量const修饰
```

![image-20210711155659283](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210711155659283.png)

### 注意事项

```
1.常量必须初始化，且不能复制。
2.把常量定义为大写字母形式，是一个很好的编程习惯。便于与变量区分
3.常量一旦被定义 不能被删除 不能被修改 
4.常量的值必须为标量数据类型 （标量包含:整形 浮点 布尔 字符串）
```

