---
title: "Shell简介及自己动手实现一个Shell"
date: 2023-01-09T10:18:37+08:00
draft: true
tags: [OS,Shell,DoItYourself,Go]
categories: [OS]
---


## 引入

一个运行在操作系统之上的应用程序必须通过操作系统提供的统一接口，而这部分接口大多使用C/C++定义，应用程序通过固定编码调用操作系统统一封装的接口实现对内核、系统库或三方库的调用。在日常使用中编码势必带来诸多不便，因此Shell运势而生。
Shell：文本命令解析器，通过交互式解析用户输入的字符串命令或则文件提供的命令集，实现对应用程序、系统程序或内核的调用。

![进程与OS的关系.png](https://cdn.nlark.com/yuque/0/2022/png/22267852/1668737755114-b472a04b-96dd-4f5c-b480-894b7187da59.png)

## 一、常见Shell

| **操作系统** | **常用Shell** |
| --- | --- |
| Windows | Cmd、PowerShell |
| Mac/Linux | bash、zsh、sh等 |

## 二、bash

> 本文以Linux下最为常用的bash为例，对bash的一些特性做简单讲解

### 2.1 bash特性

| 快捷操作 | TAB：TAB两次显示所有相关命令，TAB单次补全命令
Ctrl+C：终止当前命令运行 |
| --- | --- |
| 历史记录 | 运行命令自动记录，可通过history查看历史 |
| 别名 | 通过alias和unalias命令设置和取消别名 |

> 一些shell还针对命令有特殊解析，比如zsh将无法识别的命令作为路径进行cd

### 2.2 bash命令优先级

| **优先级** | **解释** | **举例** |
| --- | --- | --- |
| alias | 设置的别名 | alias ll='ls -al' |
| built_in | bash内建定义的命令解析 | echo、export、alias、unalias |
| hash | 使用过的命令将其路径缓存 |  |
| PATH | 外部环境变量命令，在硬盘中有路径存储的命令 | $PATH |

> 其中alias、内建命令和hash是bash提供的高级功能，PATH是系统提供的变量

常见bash内建命令

| 命令 | 解释 | 举例 |
| --- | --- | --- |
| alias | 为命令设置别名 | alias ll='ls -al' |
| unalias | 取消设置别名 | unalias ll |
| export | 设置环境变量 | export name=zhangsan |
| echo | 输出打印，支持环境变量等 | echo $name |
| help | 命令的帮助文档 | help echo |
| type | 查看命令的类型 | type echo |

## 三、自己动手写Shell

### 3.1 效果演示

通过简单实现一个命令解析器goshell，能交互式解析和解析命令集合，并实现一个自定义的打印命令。
> 源码地址：[https://github.com/ns-cn/ttools/tree/main/goshell](https://github.com/ns-cn/ttools/tree/main/goshell)

```shell
➜  ~ goshell
/Users/tangyujun >dayin 你好世界
你好世界
/Users/tangyujun >ll
exec: "ll": executable file not found in $PATH
/Users/tangyujun >cd go
/Users/tangyujun/go >exit
➜  ~
```

```shell
#!/Users/tangyujun/go/bin/goshell
# 注：已将goshell安装到go的可执行环境中
ls
dayin 你好世界
```

```shell
➜  release git:(main) ./myshell.sh 
welcome to use goshell!
myshell.sh
你好世界
```

### 3.2 goshell的逻辑伪代码

```shell
if 有文件参数{
   打开文件
   逐行执行
}else{
   for{
       读取用户输入
       执行用户输入
    }
}
```

> 注：这种实现方式在实现逻辑上就缺少bash中alias、hash的支持，也不如bash实现的内建命令多，但基本能满足一个命令解释器的该有的功能。

## 四、一些有用的命令

### 4.1 命令系统调用跟踪-strace

通过使用命令`strace`命令查看跟踪命令执行过程中所有的系统调用

```shell
tangyujun@tangyujun-Parallels-Virtual-Platform:~$ strace -T echo hello
execve("/usr/bin/echo", ["echo", "hello"], 0x7ffc9d667cf0 /* 53 vars */) = 0 <0.000373>
brk(NULL)                               = 0x557a2c1c2000 <0.000074>
-- 删减 --
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0 <0.000110>
write(1, "hello\n", 6hello)                  = 6 <0.000110>
close(1)                                = 0 <0.000109>
close(2)                                = 0 <0.000069>
exit_group(0)                           = ?
+++ exited with 0 +++
```

### 4.2 命令CPU执行时间跟踪-time

```shell
tangyujun@tangyujun-Parallels-Virtual-Platform:~$ time sleep 1

real 0m1.005s
user 0m0.002s
sys 0m0.000s
```

> 响应结果：
>
> - real：完整执行时长
> - user：用户CPU占用时长
> - sys：系统CPU占用时长

### 4.3 java

java命令用于解释class文件、并在java虚拟机上执行，因java使用的广泛性，现针对java多版本管理工具做简单介绍。
**sdkman**
sdkman是类Unix上的开发工具sdk管理工具（仅支持Linux、MacOS），可以方便的管理开发工具sdk的安装、卸载、版本切换等。支持包括gradle、maven项目管理工具、java、kotlin编程语言、springboot、vertx框架等。

```shell
# 安装sdkman
curl -s "https://get.sdkman.io" | bash

# 查看所有java版本
sdk list java
# 安装zulu的JDK8
sdk install java 8.0.352.fx-zulu
# 安装amzn的JDK17
sdk install java 17.0.5-amzn
# 全局设置java的版本为JDK8
sdk default java 8.0.352.fx-zulu
# 查看java的当前使用版本
sdk current java
# 临时使用JDK17，生效日期仅当前会话
sdk use java 17.0.5-amzn
```

**jEnv**
jEnv是一款 Java 多版本管理工具，它为安装，切换，删除和列出候选提供了一个方便的命令行界面。支持Linux、Mac和Windows（参考[FelixSelter/JEnv-for-Windows](https://github.com/FelixSelter/JEnv-for-Windows)）

```shell
# 通过brew安装jenv
brew install jenv
# 查看所有已受jenv管理的JDK
jenv versions
# 添加本地JDK受jenv管理
jenv add /usr/local/Cellar/openjdk@8/1.8.0+352/libexec/openjdk.jdk/Contents/Home/
# 查看当前全局使用的版本
jenv version
# 指定全局使用版本
jenv global 1.8.0.352
# 指定当前会话使用版本
jenv shell 17.0.5
```
