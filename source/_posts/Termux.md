title: Termux高级终端
author: Jie
tags: 
  - Android
categories: 
  - Termux
date: 2022-09-29 21:20:00
---
![](https://image.3001.net/images/20180501/15251875958364.png)
Termux 是一个 Android 终端仿真应用程序，用于在 Android 手机上搭建一个完整的 Linux 环境。 不需要 root 权限 Termux 就可以正常运行。Termux 基本实现 Linux 下的许多基本操作。可以使用 Termux 安装 python，并实现 python 编程，可以用手机架设 Server，同样可以用于渗透测试等等。
<!-- more -->
# 开发环境
Termux 支持的开发环境很强，可以完美的运行 C、Python、Java、PHP、Ruby 等开发环境，建议朋友们选择自己需要的开发环境折腾。

## 编辑器
写代码前总得折腾一下编辑器，毕竟磨刀不误砍柴工嘛。Termux 支持多种编辑器，完全可以满足日常使用需求。

### Vim
Vim 被称为编辑器之神，基本上 Linux 发行版都会自带 Vim，这个在前文基本工具已经安装了，如果你没有安装的话，可以使用如下命令安装：
```
pkg install vim
```
并且官方也已经封装了 vim-python，对 Python 相关的优化。
```
pkg install vim-python
```
### 解决汉字乱码
如果你的 Vim 打开汉字出现乱码的话，那么在家目录 (~) 下，新建.vimrc 文件
```
vim .vimrc
```
添加内容如下：
```
set fileencodings=utf-8,gb2312,gb18030,gbk,ucs-bom,cp936,latin1
set enc=utf8
set fencs=utf8,gbk,gb2312,gb18030
```
然后 source 下变量：
```
source .vimrc
```

## C
Termux 官方封装了 Clang，他是一个 C、C++、Objective-C 和 Objective-C++ 编程语言的编译器前端。

安装 clang
```
pkg install clang
```
编译测试
clang 在编译这一块很强大，感兴趣的朋友可以去网上查看详细的教程，国光这里只演示基本的 Hello World 使用。写一个 Hello World 的 C 程序，如下 hello.c：
```c
#include <stdio.h>

int main(){
  printf("Hello World")
  return 0;
}
```
编辑完成后，使用 clang 来编译生成 hello 的可执行文件：
```
clang hello.c -o hello
```

## Java
Termux 早期原生编译 JAVA 只能使用 ecj (Eclipse Compiler for Java) 和 dx 了，然后使用 Android 自带的 dalvikvm 运行。后面 Termux 官方也封装了 openjdk-17 这样安装起来就更方便了。

还有如果想要完整体验 JAVA 环境的话，另一个方法就是 Termux 里面安装一个完整的 Linux 系统，然后在 Linux 里面运行 Java，安装系统部分下面文章会详细介绍，这种思路也是可以的。

Openjdk-17
```
pkg update
pkg install openjdk-17
```
当然这个包可能不太稳定，遇到相关问题可以去 Termux 官方的项目下提交 issue：

https://github.com/termux/termux-packages/issues?q=openjdk
https://github.com/termux/termux-packages/issues?q=java

ECJ
安装编译工具
```
pkg install ecj dx -y
```
这里只演示基本的 Hello World 使用。写一个 Hello World 的 JAVA 程序，如下 HelloWorld.java:
```java
public class HelloWorld {
    public static void main(String[] args){
        System.out.println("Hello Termux");
    }
}
```
编译生成 class 文件
```
ecj HelloWorld.java
```
编译生成 dex 文件
```
dx --dex --output=hello.dex HelloWorld.class
```
使用 dalvikvm 运行
格式规范如下：dalvikvm -cp dex文件名 类名
```
dalvikvm -cp hello.dex HelloWorld
```

## Python
Python 是近几年非常流行的语言，Python 相关的书籍和资料也如雨后春笋一般不断涌现，带来了活跃了 Python 学习氛围。

安装 Python2
Python2 版本要淘汰了，大家简单了解一下就好：
```
pkg install python2 -y
```
安装完成后，使用 python2 命令启动 Python2.7 的环境。

安装 Python3
Termux 安装 Python 默认版本是 Python3 的版本，与此同时也顺便安装了 clang
```
pkg install python -y
```
安装完成后，查看下 clang 和 Python 的版本：
```
python -V
```

### 升级 pip2 
```
python2 -m pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

### 升级 pip3
```
python -m pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
这两条命令分别升级了 pip2 和 pip3 到最新版。升级完成后你会惊讶的发现你的 pip3 命令不见了？？？
不要慌 问题不大，我们可以手动查看当前有哪些可执行的 pip 文件，使用如下命令：
```
ls /data/data/com.termux/files/usr/bin|grep pip
```
原来我们的pip3变成了pip3.8了啊

接下来分别查看对应 pip 可执行文件的版本：
pip -V
现在全都是最新版的 pip 了哦


## Jupyter Notebook
官方建议安装的完整的命令：
```
pkg update
pkg install proot
termux-chroot
apt install git clang
apt install pkg-config
apt install python python3-dev 
apt install libclang libclang-dev
apt install libzmq libzmq-dev
apt install ipython 
pip install jupyter
```
```
# -i 手动指定国内中清华 pip 源 提高下载速度
# 更新是个好习惯
pkg update

# 切换模拟的 root 环境
termux-chroot

# 安装相关依赖
pkg install libclang

# 安装 jupyter
pip3 install jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple some-package

# 安装完成退出 chroot
exit

# 安装 jupyterlab
pip3 install jupyterlab -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
安装好之后查看一下版本信息：
```
jupyter --version
```
所有插件均安装完成

Jupyter Notebook 就安装好了，这个比较强大更详细的教程大家可以自行去谷歌或者百度一下，这里只演示基本的功能。

先启动 notebook
```
jupyter notebook
```
然后会看到运行的日志，我们复制出 提示的 URL：

复制出的这个 URL 地址 在浏览器中打开：

可以看到成功运行了，那我们按照图片提示走个形式，输出个 Hello World 就跑路：

OK 运行成功，那么回到 Termux 里面使用组合键 Ctrl + C -> 中止当前的 Jupyter 进程。

## Code-Server
下载code-server的最新版本  
[下载链接](https://github.com/cdr/code-server/releases)
下载对应服务器版本的文件。

使用部署脚本一键安装
```
curl -fsSL https://code-server.dev/install.sh | sh
```

运行code-server  
直接输入 `code-server` ，服务就自行开启了，默认是8080端口，127.0.0.1的访问ip。

如果想要自行更换ip或者是端口的话可以在后面添加–bind-addr参数。  
这里如果想让code-server被任意访问的话则需要把IP地址设置为0.0.0.0。  
bind-addr参数仅限code-server版本3.2.0及以上才会有，如果想知道具体参数可以输入`code-server -h`查看参数详情。
```
code-server --bind-addr 0.0.0.0:[端口号]
```

## 安装完整Linux环境
Termux 官网已经有官方版本的纯种Linux了，官方提供了最新的安装纯种Linux的方法：[PRoot - Termux Wiki](https://wiki.termux.com/wiki/PRoot#Installing_Linux_distributions)

首先最好是换个国内的Termux源，我用的清华源，换源方法看这里：[Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/termux/)

然后就是安装基础件proot-distro了：
```
pkg install proot-distro
```
或者
```
apt install proot-distro
```
查看proot-distro的使用帮助为：
```
proot-distro help
```
查看可安装的系统列表
```
proot-distro list
```
安装Ubuntu
```
proot-distro install ubuntu
```
登录
```
proot-distro login ubuntu
```
这样就进入了真正的linux环境了，以上就是官方版的纯种Linux安装全过程。