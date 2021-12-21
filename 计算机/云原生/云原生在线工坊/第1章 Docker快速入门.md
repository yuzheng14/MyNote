# 第一章 Docker快速入门

## 安装Docker

### 卸载旧版本

```shell
yum remove docker docker-common docker-selinux docker-engine
```

### 安装

#### 安装依赖包

**安装需要的软件包， yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖**

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### 设置yum源

官方源

```shell
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo
```

阿里云

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学

```
yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

##### 报错解决

```shell
[root@VM-8-12-centos ~ ]# yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
  File "/usr/bin/yum-config-manager", line 135
    except yum.Errors.RepoError, e:
                               ^
SyntaxError: invalid syntax
```

如果出现这种情况，则为python软链指向非python2

```shell
vim /usr/bin/yum-config-manager
```

将第一行的`#!/usr/bin/python -tt`改为`#!/usr/bin/python2 -tt`

或者将软链改为指向python2（不推荐）

#### 安装Docker

```shell
yum install -y docker-ce docker-ce-cli containerd.io
```

##### 安装特定版本

如需安装特定版本，输入

```shell
yum list docker-ce --showduplicates | sort -r
```

根据版本进行安装

#### 启动及开机自启

启动docker

```shell
systemctl start docker
```

通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 

```shell
docker run hello-world
```

```shell
[root@VM-8-12-centos ~ ]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

设置Docker开机自启

```shell
systemctl enable docker
```

### 卸载

删除安装包

```shell
yum remove docker-ce
```

删除镜像、容器、配置文件等内容

```shell
rm -rf /var/lib/docker
```

## Docker的基础配置

### 镜像仓库与镜像加速

#### 设置

首先执行下列命令

```shell
systemctl cat docker | grep '\-\-registry\-mirror'
```

如果该命令有输出，那么请执行 `systemctl cat docker` 查看 `ExecStart=` 出现的位置，修改对应的文件内容去掉 `--registry-mirror` 参数及其值，并按接下来的步骤进行配置。

如果以上命令没有任何输出，那么打开配置文件`/etc/docker/daemon.json`，添加`registry-mirrors`的配置

```
{
  "registry-mirrors": [
  	"https://mirror.ccs.tencentyun.com/"
  ]
}
```

未避免镜像服务器宕机，多添加几条镜像源

```
{
        "registry-mirrors": [
                "https://mirror.ccs.tencentyun.com/",
                "http://hub-mirror.c.163.com",
                "https://mirror.baidubce.com"
        ]
}
```

之后重新启动服务

```shell
systemctl daemon-reload
systemctl restart docker
```

#### 检验

运行命令 `docker info`, 在输出中查看`Registry Mirrors`下是否有设置的内容，有的话就说明成功了

```shell
[root@VM-8-12-centos ~ ]# docker info|grep "Registry Mirrors"
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
 Registry Mirrors:
```

### Docker CLI

以此输入以下指令，如有输出，则环境正常

```shell
docker version
docker help #或者直接输入docker或docker --help
```

[docker cli官方文档](https://docs.docker.com/engine/reference/commandline/cli/)

## Docker快速入门

### 使用Docker运行一个网站

输入以下命令

```shell
docker run -d -p 80:80 docker/getting-started
```

### 可视化与命令行

输入命令可看到已安装的镜像

```shell
docker image ls #或者docker images
```

输入命令管理容器

```shell
docker container ls #列出容器列表及其信息，也可以使用docker ps
docker container
```

## 库、镜像和容器

### 镜像版本与拉取镜像

使用`docker search`命令来搜索镜像，如

```shell
docker search nginx
```

说拉取镜像会使用`docker pull`，如

```shell
docker pull nginx #如果是要下载指定版本的nginx，比如可以使用docker pull nginx:1.21.3
```

### 创建并启动容器

可以使用`docker create`或`docker container create`命令基于一个镜像来创建容器，如

```shell
docker create -p 8080:80 nginx 
```

**基于一个相同的镜像，可以创建很多个容器**

`docker ps`或`docker container ls`只显示运行中的容器，要查看所有运行中的容器需输入

```shell
docker ps -a #或者输入docker container ls -a
```

在创建容器时如果没有给容器指定name名称，那么它会随机生成一个名字。给容器命名的方式就是增加name的flag

```shell
docker create -p 8000:80 --name tke_nginx nginx #docker run给容器命名也是一样的格式
```

要启动创建好了的容器，可以在命令行里输入`docker start 容器的ID或名称`来启动

```shell
docker start tke_nginx 
```

同样的也可以使用`stop`来终止容器的运行，用`restart`来实现重启

```shell
docker restart tke_nginx
docker stop tke_nginx
```

也可以通过`kill`来杀死容器

可以使用`docker rm`逐个删除或`docker prune`批量删除容器

## 项目的源代码与镜像

首先`git clone`源代码

### 传统方式部署

使用传统的方式在本地电脑将网站运行起来，在下载了项目源代码之后

- 首先需要安装Python开发环境，因为这个网站是基于Python的静态网站生成器MkDocs来开发的；
- 然后再使用Python的pip包管理器安装`requirements.txt`的依赖，这时就会安装MkDocs程序、MkDocs的模板和插件；
- 然后再来执行`MkDocs build`将docs文件夹里的Markdown文件生成静态html文件；
- 然后再安装nginx并运行；
- 最后再将MkDocs生成的静态文件放置到nginx目录，这样就能访问了

### Docker部署

命令行输入

```shell
docker build -t tke-start . #最后的小点.是当前目录的意思，不要漏了
```

`docker build`就是将项目的源代码打包成镜像，而`-t`是则是给打包的镜像命名以及打标签（标签内容是版本号），打包完成后，就可以使用docker image ls`看到打包的镜像

