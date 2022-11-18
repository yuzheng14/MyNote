# Ubuntu 开荒

记录一下全新的 Ubuntu 系统如何变成 yuzheng14 的样子（

## 包管理 apt

### 更换镜像源

先更改 `sources.list` 文件的权限

```bash
cd /etc/apt
sudo chmod 777 sources.list
```

备份原有文件

```bash
sudo mv sources.list sources.list.bak
```

写入镜像源

```bash
cat >/etc/apt/sources.list
```

将一下内容输入后按 <kbd>Ctrl+D</kbd> 保存并退出

ubuntu 20.04(focal)

```
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```

ubuntu 18.04(bionic)

```
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

### 更新软件包列表文件

```bash
sudo apt update   #也可以使用apt-get update
```

> 这个命令的意思是从软件源下载最新的软件包列表文件，update更新本地软件包缓存信息，可以保证下载的软件包是最新的

## 配置中文

首先输入`locale -a`查看是否安装有中文语言包，即`zh_CN.utf8`

如果没有，则需要安装中文语言包，输入以下指令

```bash
sudo apt install language-pack-zh-hans
```

添加中文支持

```bash
sudo locale-gen zh_CN.UTF-8
```

## sudo 免密（可选）

输入

```bash
sudo visudo
```

进入编辑器

找到

```
%sudo   ALL=(ALL:ALL) ALL
```

在下一行输入

```
%username ALL = (ALL:ALL) NOPASSWD: ALL
```

将username替换为自己的用户名

依次按下<kbd>Ctrl</kbd>+<kbd>X</kbd>, <kbd>Y</kbd>，<kbd>Enter</kbd> 保存退出

## 配置 git

生成 ssh 密钥

```bash
ssh-keygen -t rsa -C "你的邮箱"
```

直接全部回车

然后将生成的 id_rsa.pub 公钥复制到github上

输入以下指令测试能否连接到 github

```bash
ssh -T git@github.com
```

如果出现

```
ssh: connect to host github.com port 22: Connection refused
```

则在 ./ssh 目录下新建文件 config，输入

```
Host github.com  
User <your email>
Hostname ssh.github.com  
PreferredAuthentications publickey  
IdentityFile ~/.ssh/id_rsa  
Port 443
```

配置用户

```bash
git config --global user.name "Your Name"（注意前边是“- -global”，有两个横线）
git config --global user.email "email@example.com"
```

配置 alias

```bash
git config --global alias.cam "commit -a -m"
git config --global alias.cm "commit -m"
git config --global alias.pure "pull --rebase"
git config --global alias.lg "log --graph --decorate"
git config --global alias.lg1 "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative"
```

更改 .bashrc 中关于 PS1 变量的内容以显示当前分支 [如何在Linux下显示当前git分支 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/133291342)

```shell
git_branch()
{
   branch=`git rev-parse --abbrev-ref HEAD 2>/dev/null`
   if [ "${branch}" != "" ]
   then
       if [ "${branch}" = "(no branch)" ]
       then
           branch="(`git rev-parse --short HEAD`...)"
       fi
       echo "($branch)"
   fi
}

if [ "$color_prompt" = yes ]; then
        PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\] \[\033[01;32m\]$(git_branch)\[\033[00m\]\$ '
else
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\W $(git_branch)\$ '
fi
```

## 安装 node

首先安装 nvm，访问 [nvm库的安装说明](https://github.com/nvm-sh/nvm#installing-and-updating)，按照说明进行安装

安装完成后输入以下指令测试是否正确安装

```bash
nvm --version
```

如无反应则刷新 .bashrc 文件

```bash
source ~/.bashrc
```

查看可安装的版本

```bash
nvm ls-remote
```

通过 nvm 安装 node.js

```bash
nvm install <version>
```

## 配置 npm

更换国内镜像源（淘宝）

```bash
npm config set registry https://registry.npmmirror.com/
```

## 配置 java 开发环境

### 安装 open-jdk

```bash
sudo apt update
# 可先通过 apt list | grep openjdk 查询版本，建议使用 LTS
sudo apt install openjdk-<version>-jdk 
```

### 配置 JAVA_HOME 环境变量（可选）

在一些 Java 应用中，环境变量`JAVA_HOME`被用来表示 Java 安装位置。

首先找到 java 的安装路径

```bash
sudo update-alternatives --config java
```

使用以 `/usr/lib/jvm` 开头的路径，输入到 `/etc/enviroment` 文件末行

```bash
sudo cat JAVA_HOME=<your path> >> /etc/environment
```

刷新变量

```bash
source /etc/environment
```

若输入 `echo $JAVA_HOME` 输出设置的路径，则为正确设置。

### 安装 maven

自动安装

```bash
sudo apt install maven
```

手动安装最新版本（由于个人使用 Java17，apt 安装的 maven 不支持，故手动安装）

去 maven 官网获取到最新版的下载地址，下载解压

```bash
sudo mkdir /usr/local/maven && cd /usr/local/maven
sudo wget <download url>
sudo tar zxvf <downloaded file>
```

设置环境变量

```bash
sudo vim /etc/profile.d/maven.sh
```

```
export M2_HOME=<your maven path>
export MAVEN_HOME=<your maven path>
export PATH=${M2_HOME}/bin:${PATH}
```

更改执行权限，刷新变量

```bash
sudo chmod +x /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
```

maven 本地仓库初始化，在用户根目录下会生成一个 .m2 的文件夹

```bash
mvn help:system
cd ~/.m2
cp ${MAVEN_HOME}/conf/settings.xml .
vim settings.xml
```

```bash
<mirror>
	<id>alimaven</id>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	<mirrorOf>central</mirrorOf>
</mirror>
```

## 配置 C 开发环境（WSL）

```bash
sudo apt update
sudo apt install -y build-essential
```

这个命令将会安装一系列软件包，包括`gcc`,`g++`,和`make`。

