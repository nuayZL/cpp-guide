Nginx使用过程中的问题

先安装pcre表达式库（rewrite模块需要）

```
#下载
wget    https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz
#解压安装包:
tar -zxvf pcre-8.40.tar.gz
#进入安装包目录
cd pcre-8.40
#编译安装  
./configure
make && make install
#查看pcre版本
pcre-config --version
```

安装过程权限不够  先su命令 获得root权限

```
(下载不同的版本可能会有所区别，所以还是按照书上的版本来安装)
tar xvfz nginx-1.20.1.tar.gz
```

安装gcc编译器

```
（因为nginx是纯c写的）
sudo apt install gcc
```

快速安装

```
cd nginx-1.20.1
./configure
make
sudo make install
-------------------------以下为注意事项-----------------------
（nginx默认会安装到/usr/local/nginx文件）
（如果想指定nginx安装到哪个目录下可以使用）
./configure --prefix=home/geek/nginx

对配置文件的修改  在安装目录下的conf修改才是有效的

./configure后会生成一些中间文件到objs文件夹下
其中最重要的是ngx_modules.c文件（决定了又哪些文件会被编译进nginx）

C语言编译时生成的所有文件都会放在src目录（.o文件）

在安装目录最主要的二进制文件在sbin文件下
```

安装目录下功能介绍

```
conf文件决定nginx文件功能
sbin文件存放主要的二进制文件
```

在编译时将警告当成错误的解决方案

```
https://blog.csdn.net/qq_44689970/article/details/116891874
```

打开配置文件查看相关的配置文件

```
cd conf
vim nginx.conf
```

运行

```
启动和停止都需要root权限
输入以下命令将使用默认的配置文件启动nginx
/usr/local/nginx/sbin/nginx     
```

验证安装

```
默认配置文件开启了localhost：80服务
使用curl工具进行验证
curl - vo /dev/null http://localhost/index . html
```

![image-20210626191927309](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626191927309.png)

得到以下回应说明已经开启nginx

![image-20210626192015053](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626192015053.png)

通过ps  aux|grep nginx命令查看是否有nginx相关的命令

如下可以看到当前有master进程和work进程

![image-20210626192337750](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626192337750.png)

即然已经知道了进程号，尝试通过gdb调试下

```
sudo gdb
attach 23271
b ngx_http_output_filter    //在该函数处打个断点
c                           //继续执行   发现无反应
此时开启另一个终端     重新访问下页面
curl - vo /dev/null http://localhost/index . html
```

```
断点触发（16层堆栈）
```

![image-20210626193030899](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626193030899.png)

重复运行可能出现如下问题

![image-20210626193810041](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626193810041.png)

```
修改端口号和主机地址有没有作用
只能通过修改
netstat -tpl   查看相关的进程
killall -9 nginx   //删除nginx相关的进程
在此尝试启动即可
```

尝试从网页登录

```
server_name是localhost
服务器部署在虚拟机上，虚拟机的ip才是要访问的localhost
直接输入  192.168.231.137  
如果访问不成功有可能是被防火墙拦截了，
需要使用firewall-cmd --zone=public --add-port=80/tcp --permanent 把80这个端口打开
在firewall-cmd --reload来刷新一下
```

nginx拒接访问的解决访问

```
重载配置文件
cd /usr/local/nginx/conf
rm nginx.conf
cp nginx.conf.default nginx.conf
```

