Nginx搭建反向代理服务器

```
背景意义：上游要处理复杂的业务逻辑，及强调开发效率，性能较差
反向代理功能可以使nginx将请求按照负载均衡算法代理给多台上游服务器。
实现了水平扩展，提升了性能
```

```
上游服务器通常对公网是不提供访问的
```

```
首先进入conf文件
修改server内的listen为   
127.0.0.1:8080         #表示本机的进程才能访问8080端口
之前的用1.14版本  反向代理用openresty
```

修改上游服务器的conf文件

```
1.添加upstream配置块--块内是上游服务器的ip地址
xxx是这一批上游服务器的命名  假设为local
upstream xxx{

}
2.修改location块的配置指令
proxy_pass  http://local    #将请求代理到local上游服务集群
```

```
更多的反向代理proxy特性可在
官网http://nginx.org/--documentation--Modeles reference -找到proxy模块
```

