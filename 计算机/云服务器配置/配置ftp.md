# 配置ftp

## 安装

### 安装命令

```bash
yum install vsftpd -y
```

![image-20210518223129917](.\配置ftp.assets\image-20210518223129917.png)

显示complete即完成

### 服务相关

#### 关闭

```bash
systemctl stop vsftpd.service
```

#### 启动

```bash
systemctl start vsftpd.service
```

#### 查看状态

```bash
system status vsftpd.service
```

### 检查端口

```bash
netstat -anp|grep 21
```

![image-20210518230942052](.\配置ftp.assets\image-20210518230942052.png)

可以看到端口21已经被vsftpd监听了

## 用户<span id="user"></span> 

### 概念

要连接上 vsftpd 服务器需要为Linux创建专门的用户。

### 用户目录

在Linux中，不同用户是有不同目录访问权限的，所以首先创建一个目录，作为这个ftp用户所拥有的目录。

```bash
mkdir -p 用户目录
```

### 创建用户

```bash
useradd -d 用户目录 -g ftp -s /sbin/nologin 用户名
```

-g ftp 表示该用户属于ftp分组 (ftp分组是内置的，本来就存在，不需要自己创建)
-s /sbin/nologin 表示这个用户不能用来登录secureCRT这样的客户端。 这种不能登陆的用户又叫做虚拟用户
创建过程给出的警告信息是正常的，不用理会

![image-20210518231608088](.\配置ftp.assets\image-20210518231608088.png)

### 删除用户

```bash
userdel -r 用户名
```

### 设置目录权限

```bash
chown -R 用户名 用户目录
chmod -R 775 用户目录
```

设置用户目录的拥有者

使该用户拥有这个目录的读写权限

### 设置密码

```bash
passwd 用户名
```

![image-20210518232435425](.\配置ftp.assets\image-20210518232435425.png)

## 用户配置

### 去除匿名登录

默认情况下vsftpd服务器是允许匿名登陆的，这样非常不安全，所以要把这个选项关闭掉。

首先通过vim命令打开ftp服务器配置文件：

```bash
vim /etc/vsftpd/vsftpd.conf
```

然后把本来的

> anonymous_enable=YES

修改为

> anonymous_enable=NO

### 限制用户访问

接下来是限制用户访问，什么叫做限制用户访问呢？ [用户](#user) 教程中创建的用户所拥有的目录,如果不做限制，那么使用ftptest登陆之后可以切换到其他敏感目录去，比如切换到/usr目录去，这样就存在巨大的安全隐患。

为了规避这个隐患，需要限制用户只能通过ftp访问到其拥有的目录以及子目录。

配置办法:

首先通过vim命令打开ftp服务器配置文件：

```bash
vim /etc/vsftpd/vsftpd.conf
```

找到：

>\#chroot_list_enable=YES  
>\# (default follows)  
>\#chroot_list_file=/etc/vsftpd.chroot_list

并修改为：

>chroot_list_enable=YES  
>\# (default follows)  
>chroot_list_file=/etc/vsftpd/chroot_list

### 用户清单

接着上一个步骤，在chroot_list中添加ftptest用户

首先通过vim命令打开chroot_list文件(此文件本来是空的)：

```bash
vim /etc/vsftpd/chroot_list
```

然后增加一行: 用户名

修改完成之后，保存退出。

![image-20210518233513312](.\配置ftp.assets\image-20210518233513312.png)

### 允许写权限

vsftpd服务器是这样的，一旦某个用户被限制访问了，那么默认情况下，该用户的写权限也被剥夺了。 这就导致ftp客户端连接上服务器之后无法上传文件。

这个时候，就需要打开此用户的写权限，请按照如下办法操作：

首先通过vim命令打开ftp服务器配置文件：

```bash
vim /etc/vsftpd/vsftpd.conf
```

在最后面新加一行：

> allow_writeable_chroot=YES

## 配置端口

### 两种端口

vsftpd有两种端口，一个是21端口，用来监听客户端连接请求的。 这个一般说来是固定的，就一直使用21端口。

另一种是，一旦获取到请求之后，再专门用户服务端和客户端传输数据的端口。

本知识点就是用于指定第二种端口的获取范围

### 配置端口

打开配置文件

```bash
vim /etc/vsftpd/vsftpd.conf
```

在最后添加

> pasv_enable=YES  
> pasv_min_port=30000  
> pasv_Max_port=30010

这表示使用被动模式，用于传输数据的端口分配从30000到30010之间

在后续的Linux开放端口中也会做相应的配合工作

## 用户鉴权

因为用户 ftptest 是 nologin的，所以存在鉴权的问题。 如果鉴权问题不解决，就是永不停息的 530错误。。。搞死宝宝了
解决办法有如下两种:

方式一：pam.d/vsftpd文件

方式二：shells文件

### 方式一

```bash
vim /etc/pam.d/vsftpd
```

注释掉/etc/pam.d/vsftpd文件里这一行：

> \#auth required pam_shells.so

这样不去鉴权，从而允许nologin用户登录 ftp 服务器.

### 方式二

```bash
vim /etc/shells
```

在`/etc/shells`文件里增加一行

> /sbin/nologin

这样允许不能登录系统的用户通过鉴权

## 重启ftp服务器

```bash
systemctl restart vsftpd.service
```

## 开放端口

不同服务器提供商方法不同

开放21和30000-30010端口

## FTP客户端

个人使用`FileZilla`

