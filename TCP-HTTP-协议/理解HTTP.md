HTTP的具体格式

![image-20210707212027404](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707212027404.png)

```
请求行：请求方法/URL/协议版本
结束的时候以回车符，换行符结束
请求头：请求头名称/请求头内容/
请求体(包体)：POST有请求头

请求行+请求体=包头
包头和包体
```

```
HTTP 在大多数情况下都是文本形式的明文格式
也就是Hypertext Transfer Protocol text (文本)的名称含义
包头的每一行以\r\n结束
包头以\r\n\r\n结束

注意和HTML里边的<head>和<body>标签继续区分，
HTML文档中的内容仅仅式包体的一部分
```

GET 和POST 方法

```
常用请求有GET POST HEA PUT DELETE等
 
 GET时的HTTP方法
 应用场景
 当：为了查找资源，请求结果无持续的副作用
 参数一般放在URL后面，用？分割，参数与参数间用&分割
 浏览器对URL的最大值有限制。
 请求的内容（资源文件）直接跟在get请求命令后边
 
 以下为请求http: //www.hootina.org/index_2013.php?param1 =value1&param2=value2&param3=value3时发送的内容
```

![image-20210707221625772](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707221625772.png)

```
POST请求
应用场景：创建资源  /在数据库加入新数据行
典型的post请求 
:当登录12306网站输入用户名。密码后 会发送出一个post请求
post方法的请求数据在HTTP包体中，

怎么使对端知道包体的长度？
1.采用HTTP chunk
2.在包头中设置Content-length字段
```

post和get的安全性比较

```
上述两种请求的破解难度是相同的，
都需要对一些关键性信息进行混淆和加密

对安全性较高的应用，需要SSL与HTTP结合，即HTTPS
SSL(Secure Socket Layer 安全套接字层) 即在TCP和HTTP层再加入一个SSL层，
使用443接口和加密/身份验证层用于对互联网上安全敏感的通信
```

HTTP chunk编码

```
如果传输过程中包体过大，传输方无法知道传输的内容，就使用HTTP chunk编码技术
原理是将HTTP包体分成多个小块  每一块有字段说明自身长度

通过再HTTP包头中设置了Transfer-Encoding:chunked字段告诉对端这个数据是分块传输的

分块传输的编码格式如下
```

![image-20210707222432459](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707222432459.png)

```
每个chunk都为 [chunkSize] [\E\n] [chunkData] [\r\n]
第一部分是长度  第二部分是长度的内容
最后一个长度为0的chunk的表示结束

chunksize以十六进制的ASCII码表示每字节
如果为35 36（ascii中的第35/36个）
分别对应ascii的阿拉伯数字5和6
即对应十六进制的56   
对应十进制的86
```

HTTP客户端编码实现

```
以请求'' http: //www.hootina.org/index_2013.php''这个URL为例
先取出域名部分 hootina.org
通过api gethostbyname函数得到域名对应的     IP地址
没有显式只当请求端口号，所以使用http的    默认端口号 80.
有了地址和端口号就使用connect函数连接服务器
然后组包
然后send函数将协议包发出去
```

HTTP服务器端的实现（实现一个支持HTTP格式的注册请求为例）

```
检查是否以\r\n\r\n 结束，如果不是，则说明包头不完整
```

![image-20210707225115212](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707225115212.png)

```
以 \r\n 分割每一行
```

![image-20210707225204544](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707225204544.png)

```
对于请求行，以空格分割请求方法/URL/版本号
```

![image-20210707225802127](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707225802127.png)

```
通过"?"分割前后两端，前面是URL，后面是参数
```

![image-20210707225839068](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210707225839068.png)

