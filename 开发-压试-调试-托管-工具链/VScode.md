## 1.VScode中配置C/C++环境

详细可参考https://blog.csdn.net/bat67/article/details/81268581

1.1下载codeblocks最新版

https://www.codeblocks.org/downloads/binaries/

我下载的是codeblocks-20.03mingw-setup

1.2配置环境变量

![image-20210625220028209](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210625220028209.png)

1.3下载安装VScode

1.4安装VScode cpp相关的插件

![img](https://img-blog.csdn.net/20180729110904750?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhdDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1.5配置.vscode文件

![image-20210625220341863](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210625220341863.png)

此时会生成launch.json启动配置文件

```
{
    "version": "2.0.0",
    "configurations": [
        {
            "name": "(gdb) Launch",	// 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", 		// 配置类型，这里只能为cppdbg
            "request": "launch",	// 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${workspaceRoot}/${fileBasenameNoExtension}.exe",// 将要进行调试的程序的路径
            "args": [],				// 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, 	// 设为true时程序将暂停在程序入口处，一般设置为false
            "cwd": "${workspaceRoot}",// 调试程序时的工作目录，一般为${workspaceRoot}即代码所在目录
            "environment": [],
            "externalConsole": true,// 调试时是否显示控制台窗口，一般设置为true显示控制台
            "MIMode": "gdb",
            "miDebuggerPath": "C:/Program Files (x86)/CodeBlocks/MinGW/bin/gdb32.exe",// miDebugger的路径，注意这里要与MinGw的路径对应
            "preLaunchTask": "g++",	// 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
#注意miDebuggerPath要和安装路径gdb所在的安装路径保持一直
```

再新建tasks.json文件

```
{
    "version": "2.0.0",
    "command": "g++",
    "args": ["-g","${file}","-o","${fileBasenameNoExtension}.exe"], // 编译命令参数
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": ["relative", "${workspaceRoot}"],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    }
}

```

1.5调试即可

![image-20210625222408312](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210625222408312.png)

1.6注意事项

1.6.2在进行调试之前，应该选中到要调试的源文件上（如下），否则会报错

![image-20210625222525023](C:\Users\Echo\AppData\Roaming\Typora\typora-user-images\image-20210625222525023.png)

1.6.2VScode只能打开文件，不能打开

1.6.3vscode不支持万能头文件<bits/stdc++.h>。所以可在同文件夹下新建一个bits/stdc++.h用来包含所有可能用大的头文件。

1.6.4vscode的输入也是在

## 2.用户界面

## 常用命令和使用方法

0.配置文件

```
打开配置文件输入
settings.json
```

1.控制面板

ctrl+shift+p打开

设计初衷就是解放鼠标，使所有的操作都能通过键盘进行

```
ctrl+p文件跳转
ctrl+shift+tab在所有打开的文件中跳转
ctrl+shift+p打开命令面板
ctrl+shift+o跳转到文件中的符号
ctrl+g跳转到文件中的某一行
```

在命令行输入？   就能列出所有可用的命令

2.并排编辑

```
Ctrl+\
或者  查看-编辑器布局-拆分
```

3.缩略图控制

```

```

4.禅模式

```
查看-外观-禅模式
或  ctrl+K+Z 
退出   双击esc
```

5.多光标

```
相当于对同一个源文件内的同类的变量进行同时的修改
Ctrl+d 会选择当前光标处的单词，再按下会选择下下个相同位置处的单词
```

6.自动保存

```
配置文件内设置
files.auto Save
注：发现在不同的配置块内又不同的配置指令生效
```

7.热退出

```
功能：记住未保存的文件
配置指令：files.hotExit
```

8.搜索

```
局部搜索
支持：区分大小写/全字匹配/正则表达式
Ctrl+F
全局搜索（跨文件搜索）
即左侧的搜索功能
```

9.代码提示（IntelliSense）

```
ctrl+space触发智能提示
```

10.代码折叠

```
shift+Click
```

11.缩进

```
配置项editor.insertSpace
在主界面的最下方有制表符长度选项
```


$$

$$