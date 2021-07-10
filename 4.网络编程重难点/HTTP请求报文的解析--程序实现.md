## 请求报文类设计--httpRequest

```
一个请求类应该有
0.构造函数、析构函数
1.解析请求函数
    进一步细分可以分为解析请求行、请求头、请求体  
2.有数据成员（存储请求的内容）和对应的获取其的成员函数
    路径、版本、方法、请求体、
```

## 请求解析方法parse

### 有限状态机

```
状态自动机器拥有有限数量的状态，每个状态可以迁移到零个或多个状态，输入子串决定执行哪个状态的迁移。
```

```
有限状态机的要素
状态：REQUEST_LINE、HEADERS、 BODY
动作：ParsePath_、ParseHeader_、ParseBody_
状态与动作是一一对应的
```

![image-20210710213408207](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210710213408207.png)

### 正则表达式

#### 规则

```
^	匹配一行开头
$	匹配一行结尾
*	匹配0次或多次
+	匹配1次或多次
？	0次或1次
{n}	n次
{n,}	n次以上
{n,m}	匹配n-m次
[xyz]	匹配集合中的人一个字符
[^xyz]	不匹配集合中的任何一个字符
```

#### 使用实例

```
处理请求行
POST /hello/index.jsp HTTP/1.1
分成了三部分  每部分以空格结束
^([^ ]*) ([^ ]*) HTTP/([^ ]*)$
//以上语法分析  
1.最开始的^和最后的$表示匹配的开始和结束
2.([^ ]*) 表示匹配任意个非空格的单词

以上为匹配规则regex patten

smatch subMatch
regex_match表示正则表达式要匹配整个字符串
```

```
 regex patten("^([^ ]*) ([^ ]*) HTTP/([^ ]*)$");    //（）表示一组 []表示满足里边的字符  *表示前边的内容可以有一个或者多个
    smatch subMatch;
    if(regex_match(line, subMatch, patten)) { //参数1：要匹配的内容  参数2 保存到的位置 参数3 正则规则
        method_ = subMatch[1];
        path_ = subMatch[2];
        version_ = subMatch[3];
        state_ = HEADERS;                       
        return true;
    }
```

