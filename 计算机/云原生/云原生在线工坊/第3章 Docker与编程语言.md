# 第3章 Docker与编程语言

## Docker生命周期与Dockfile

### Docker生命周期

尽管在Ubuntu容器里创建或修改了文件、或安装了软件，或者软件在运行过程中生成了数据或文件，但是当这个容器重启或者退出容器的交互模式之后，容器里面的这些数据、文件统统都会被清空，这个就是容器的生命周期。

> 若创建容器时没有`--rm`参数则退出后不会清空

使用传统的开发方式时，很多人也会直接在Linux上先安装编程语言的开发环境，再安装依赖包，尽管项目可以运行起来，但是这些命令却是**不可追溯的**。这里就需要使用到`Dockerfile`了。

### Dockerfile与命令行

按文件树格式创建文件

```
project-name
├── Dockerfile
├── sources.list 
```

打开`sources.list`文件，然后将前面的镜像源地址复制粘贴到这个文件里

打开`Dockerfile`这个文件，然后输入以下代码：

```dockerfile
#获取版本号为latest的ubuntu镜像
FROM ubuntu:latest

#修改apt的镜像源地址
COPY ./sources.list /etc/apt/

#更新本地缓存包
RUN apt-get update && apt-get install -y python3 
```

在终端中输入以下命令将`Dockerfile`打包成镜像

```shell
docker build -t project-name .
```

打包完成后，输入`docker images`就可以看到该镜像了

```shell
docker run -it --rm --name tke-lesson3 tke-lesson3
```

在容器的交互模式的终端里输入`python3`就能进入到python的交互模式了，说明只需要一个`Dockerfile`文件，Docker就会自动先下载ubuntu的镜像，更换apt的镜像地址，然后会更新本地缓存包并下载Python3。

## 编程环境的镜像选择

### 容器与编程环境

在传统开发方式下，要学习一门编程语言，需先要在操作系统上安装这门编程语言的运行环境、类库、编译工具等。使用Docker，我们也可以先拉取操作系统镜像，然后再安装相应的编程语言的环境。就如前面在Ubuntu操作系统里安装Python环境一样，其他几乎所有编程语言也可以同样处理。

不过在实际开发时，却并不会使用Ubuntu，因为运行一个编程语言的项目，我们只需要包含运行该应用进程所需的文件即可，Ubuntu包含的各种软件太多，导致用Ubuntu生成的镜像也太大。去了解一下上一节生成的镜像包有多大？160多MB。还记得Alpine操作系统多大么？

一般来说，应该尽量降低操作系统基础镜像的大小，这让镜像占用的磁盘空间更小，同时让`docker pull`命令执行的速度更快，因此在实际项目中，通常会使用精简小巧的Alpine Linux操作系统而不是Ubuntu、CentOS。

### 官方编程语言镜像

alpine操作系统不到6M，由于它足够“轻量”，一般来说，更建议选择在alpine基础上构建编程语言的运行环境。但是由于alpine太精简了，我们要运行编程语言的项目，需要安装很多零散的依赖，这是非常琐碎而复杂的工作，如果应用缺少了某个依赖就很容易出现bug，因此精简也并非绝对。不过不用担心，官方提供了几乎所有编程语言的镜像，哪个编程语言更建议使用哪个操作系统，以及基于这个操作系统需要安装哪些依赖，都为你搞定了。

>- JavaScript的Node：[node官方镜像](https://hub.docker.com/_/node)
>- Java的Open JDK：[openjdk官方镜像](https://hub.docker.com/_/openjdk)
>- Python：[python官方镜像](https://hub.docker.com/_/python)
>- Go：[golang官方镜像](https://hub.docker.com/_/golang)
>- PHP：[PHP官方镜像](https://hub.docker.com/_/php)
>- .Net ：[.net官方镜像](https://hub.docker.com/_/microsoft-dotnet)

>直接使用`docker pull python`、`docker pull node`这些方式拉取的是tag为`latest`的镜像，需要注意的是`latest`的tag并不都是指向最新的版本，具体指向哪个版本，需要去文档上查找。生成实际中，建议还是使用固定的tag，而不是`latest`，比如python的`latest`目前指向的是`3.10.0-bullseye`。

### 官方镜像的使用

对于任何来自第三方的镜像，更加建议使用Dockerfile来管理

如python的官方镜像

使用VSCode新建一个文件夹，比如`tke-python`，然后再在该文件夹下新建一个`Dockerfile`，并在Dockerfile里输入以下代码：

```dockerfile
FROM python:3.10.0-bullseye
```

然后输入`docker build -t tke-python .`就能构建一个名称为`tke-python`的镜像：

## 使用Docker学习编程语言

### python项目

接下来了解如何将项目的源代码打包进容器，并能用容器运行起来，还是以人人都应该学一下的Python为例。

新建一个名称为`python-start`文件夹（用来存储和管理Python项目），创建`index.py`文件和一个Dockerfile文件，其文件夹结构如下

```shell
python-start
├── Dockerfile
└── index.py
```

在`index.py`文件里随便输入点代码

在`Dockerfile`里输入以下代码，`Dockerfile`主要是来定义一个镜像是如何构建的，以及基于这个镜像创建的容器运行时会进行什么处理

```dockerfile
FROM python:3.10.0-bullseye  # 用于拉取镜像环境，并设置基映像
WORKDIR /usr/src/app  # 为下面的RUN，CMD，ENTRYPOINT，COPY和ADD指令设定工作目录，如果该文件夹不存在，则创建该文件，即使指令未用到该目录。可以多次使用。
COPY . .  # 将电脑的文件或目录复制到容器，用法COPY [--chown=<user>:<group>] <src>... <dest>
CMD [ "python", "./index.py" ]  #  容器run阶段执行指令，作为容器启动的默认值，推荐用法CMD ["executable","param1","param2"]
```

通过Dockerfile，就将本地的Python代码和镜像联系起来了（这里只有一个`index.py`，更多代码原理也是一样的）以及也设定了容器启动后的主进程就执行Python文件。

```shell
docker build -t python-start:1.0.1 .  
```

### PHP项目

大体同python项目，且不会php，略

## 包管理器与Docker

>包管理是编程里十分重要的机制，任何命令式的包管理工具，不仅可以使用命令行来安装各种包，还支持用通过文件来批量管理包，比如docker我们可以使用Dockerfile，Ubuntu和Debian可以使用`.deb`文件，CentOS可以使用`.rpm`文件，Nodejs使用的`package.json`，Python使用的`requirements.txt`，Java的Maven使用的`pom.xml`，PHP的包管理工具Composer使用的是`composer.json`，在日常维护时用包管理文件取代命令行既方便也有利于溯源。

在电脑上新建一个名称为`node-start`文件夹（用来存储和管理PHP项目），然后在文件夹下创建`index.js`文件、`package.json`文件、Dockerfile文件、`.dockerignore`文件，其文件夹结构如下：

```shell
php-start
├── Dockerfile
└── package.json
└── index.js
└── .dockerignore
```

使用打开`package.json`，然后输入以下代码。我们将要安装的express以及它的版本都用`package.json`给管理起来

```json
{
  "name": "TKE express基础案例",
  "description": "使用Docker来部署后端应用",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

然后再在`index.js`里输入以下代码，这段代码会启动一个网页服务器并监听3000端口，当访问`/`路径也就是网站首页时会返回一段字符串

```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('你好啊，你打开的是网站的首页')
})

app.listen(port, () => {
  console.log(`你现在打开的端口是： http://localhost:${port}`)
})
```

然后再在Dockerfile里输入以下代码，首先拉取node环境，然后设置工作目录，再将`package.json`复制到工作目录，再使用`RUN`指令来执行`npm install --only=production`，这样`package.json`里的依赖包就下载好了，再将本地的代码复制到工作目录，最后使用`CMD`设置容器启动后的主进程

```dockerfile
FROM node:17-slim
WORKDIR /usr/src/app
COPY package.json ./
RUN npm install --only=production
COPY . ./
CMD [ "npm", "start"]
```

输入以下命令就可以了

```shell
docker build -t node-start:1.0.1 .
docker run -dp 3005:3000 --name node-start node-start:1.0.1
```

以通过命令来查看日志

```shell
docker logs node-start
```

`.dockerignore`是在Dockerfile执行`COPY`指令时，我们不想把一些文件或文件夹添加到镜像里，比如`node_modules`，因为容器在使用`npm install`安装包时也会生成`node_modules`，或者一些配置文件等等，我们可以把这些需要排除在外的文件或文件夹写进`.dockerignore`里，比如在`.dockerignore`里输入以下代码

```
node_modules
.env
```

