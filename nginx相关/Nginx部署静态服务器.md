Nginx

1.将webserver文件夹下的html都移动到nginx的html文件夹下

```
 mv /home/nowcoder/WebServer-master/resources /usr/local/nginx/html/
```

配置nginx

![image-20210626232205747](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626232205747.png)

出错

```
resources一定要配置在和html同级别的目录下，否则不能找到
```

![image-20210626231959269](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626231959269.png)

通过修改主页ip斜杠后的内容可以访问resouces内的任意html文件

![image-20210626232705428](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626232705428.png)

![image-20210626232723906](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210626232723906.png)

