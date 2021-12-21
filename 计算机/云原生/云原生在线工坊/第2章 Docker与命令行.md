# 第2章 Docker与命令行

## Docker与操作系统

### Linux发行版

使用`docker pull`将这四个常用的发行版给下载下来

```shell
docker pull ubuntu
docker pull alpine
docker pull centos
docker pull debian
```

### 容器的交互模式

有两种方式进入到Ubuntu的命令行，一种是使用`docker run`来通过镜像启动一个容器之后并分配一个终端，另一种是使用`docker exec`命令在正在运行的容器内运行bash

```shell
docker run -it --rm --name tke-ubuntu ubuntu 
# 或者docker run -it --rm --name tke-ubuntu ubuntu /bin/bash
# 或者docker run -it --rm --name tke-ubuntu ubuntu /bin/sh
```

```shell
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
  -i, --interactive                    Keep STDIN open even if not attached #交互模式
  -t, --tty                            Allocate a pseudo-TTY #分配伪输入终端
      --rm                             Automatically remove the container when it exits #容器退出时自动删除
```

在交互模式下输入`exit`并按Enter执行就可以退出交互模式

> 注意，--rm模式下退出容器会将该容器删除

重新再在终端里输入`docker run -it --rm --name tke-ubuntu ubuntu /bin/bash`创建并启动`tke-ubuntu`容器，再新建一个终端窗口，并输入以下命令

```shell
docker exec --help
docker exec -it tke-ubuntu /bin/bash
```

使用`docker run -it`和`docker exec`都可以，只是前者包含启动容器的过程，而后者是在正在运行的容器中执行命令。

## Linux常用命令

太简单了大部分直接过

### 最基础的命令

### 文件夹的操作

```shell
rmdir tke2 tke3    #删除空文件夹
```

### 文件的操作

要给文件添加内容，可以使用`cat`命令，输入该命令后，可以在终端输入一些文字

```shell
cat >filename
```

输入完成后，就可以按`Ctrl+D`（EOF）快捷键保存并退出

## Linux基础信息

### 可执行文件

使用`which`指令会在环境变量`$PATH`设置的目录里查找符合条件的命令

```shell
echo $PATH                   #了解环境变量设置的目录，Linux是用冒号:隔开，Windows是用分号
which ls                     #了解ls命令所在的路径
which pwd rmdir mkdir cp cd  #了解其他命令所在的路径
which docker                 #返回为空
ls /bin                      #查看有哪些可执行文件,主要放置系统必备执行和应用程序的执行文件
ls /sbin                     #查看有哪些可执行文件，主要放置系统管理和网络管理的执行文件
```

### Linux基础信息命令

在`bin`和`sbin`文件夹里还有一些非常实用的可执行文件

```shell
uptime   #获取主机运行时间和查询linux系统负载等信息
top      #持续查看当前系统正在运行的进程状态，有点类似于windows的任务管理器，要退出查看窗口，可以按“Ctrl+D”
ps       #查看当前系统正在运行的进程，常使用 ps -ef 的组合
free     #用于显示内存状态
uname -a #了解操作系统的版本信息
whoami   #显示当前用户名称
env      #显示系统环境变量
```

## Linux应用程序管理

>Ubuntu的包管理器是`apt`，alpine的包管理器是`apk`，而centos的包管理器是`yum`

### 包管理器apt

复制粘贴以下内容到终端，然后按`Ctrl+D`保存并退出

```shell
cat >>/etc/apt/sources.list
```

```
deb http://mirrors.tencent.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.tencent.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.tencent.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.tencent.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.tencent.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.tencent.com/ubuntu/ bionic-updates main restricted universe multiverse
```

> 注意备份原文件，后边安装git可能镜像无所需依赖，需要官方依赖
>
> deb http://cn.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
> deb http://cn.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
> deb http://cn.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
> deb http://cn.archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
> ##測試版源
> deb http://cn.archive.ubuntu.com/ubuntu/ focal-proposed main restricted universe multiverse
> #源碼
> deb-src http://cn.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
> deb-src http://cn.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
> deb-src http://cn.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
> deb-src http://cn.archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
> ##測試版源
> deb-src http://cn.archive.ubuntu.com/ubuntu/ focal-proposed main restricted universe multiverse

这个命令的意思是从软件源下载最新的软件包列表文件，update更新本地软件包缓存信息，可以保证下载的软件包是最新的

```shell
apt update   #也可以使用apt-get update
```

>`apt update`：只检查，不更新（已安装的软件包是否有可用的更新，给出汇总报告）
>
>`apt upgrade`：更新已安装的软件包

```shell
apt --help      #了解apt包管理器还有哪些flag
apt list        #了解apt有哪些包
```

### 使用apt安装软件

```shell
apt install softwarename
```

