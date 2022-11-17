# WSL安装教程

本文源于[Microsoft官方文档](https://docs.microsoft.com/en-us/windows/wsl/install)

## 前置条件

Windows10 2004（build 19041）及更高，或者Windows11

## 安装

### 快速安装

于powershell或cmd中输入（注意用管理员权限打开）

```powershell
wsl --install
```

默认安装ubuntu，如需安装其他发行版，请参考[安装其他发行版](#distro)

```
PS C:\Users\郁蒸十四> wsl --install
正在安装: 虚拟机平台
已安装 虚拟机平台。
正在安装: 适用于 Linux 的 Windows 子系统
已安装 适用于 Linux 的 Windows 子系统。
正在下载: WSL 内核
正在安装: WSL 内核
已安装 WSL 内核。
正在下载: GUI 应用支持
正在安装: GUI 应用支持
已安装 GUI 应用支持。
正在下载: Ubuntu
请求的操作成功。直到重新启动系统前更改将不会生效。
```

安装好后需重启电脑

### 分部安装

请参考[Manual installation steps for older versions of WSL | Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

### 安装其他发行版<span id="distro"></span>

输入指令

```powershell
wsl --list --online
```

或

```powershell
wsl -l -o
```

```
PS C:\Users\郁蒸十四> wsl --list --online
以下是可安装的有效分发的列表。
使默认分发用 “*” 表示。
使用 'wsl --install -d <Distro>' 安装。

  NAME            FRIENDLY NAME
* Ubuntu          Ubuntu
  Debian          Debian GNU/Linux
  kali-linux      Kali Linux Rolling
  openSUSE-42     openSUSE Leap 42
  SLES-12         SUSE Linux Enterprise Server v12
  Ubuntu-16.04    Ubuntu 16.04 LTS
  Ubuntu-18.04    Ubuntu 18.04 LTS
  Ubuntu-20.04    Ubuntu 20.04 LTS
```

然后输入

```powershell
wsl --install -d <Distribution Name>
```

即可安装特定发行版

## 设置Linux账户信息

此部分源于[Set up a WSL development environment | Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/setup/environment#set-up-your-linux-username-and-password)

安装好后重启电脑，即可自动弹出设置页面

![image-20220212222345226](wsl%E5%AE%89%E8%A3%85.assets/image-20220212222345226.png)

依据提示输入用户名和密码即可（此账户为管理员账户）

![image-20220212222451599](wsl%E5%AE%89%E8%A3%85.assets/image-20220212222451599.png)

成功

## 端口转发<span id="portproxy"></span>

目前微软只允许宿主通过 `localhost` 来访问 `wsl`，无法通过局域网其他设备进行访问，如果局域网其他设备想要访问只能进行端口转发

原文章[如何在局域网的其他主机上中访问本机的WSL2 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/425312804)

### 开放 `windows` 端口

打开 `Windows Defender FireWal with Advanced security`

![image-20221112103928450](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-defender%20firewall.png)

找到入栈规则，点击新建规则，按照图示开放自己需要开放的端口

![img](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-%E5%85%A5%E7%AB%99%E8%A7%84%E5%88%99-%E8%A7%84%E5%88%99%E7%B1%BB%E5%9E%8B.webp)

![img](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-%E5%85%A5%E7%AB%99%E8%A7%84%E5%88%99-%E5%8D%8F%E8%AE%AE%E5%92%8C%E7%AB%AF%E5%8F%A3.webp)

![img](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-%E5%85%A5%E7%AB%99%E8%A7%84%E5%88%99-%E6%93%8D%E4%BD%9C.webp)

![img](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-%E5%85%A5%E7%AB%99%E8%A7%84%E5%88%99-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.webp)

![img](wsl%E5%AE%89%E8%A3%85.assets/端口转发-入站规则-名称.webp)

### 转发端口

在 `wsl` 内部输入 `ifconfig` 指令获取到 `wsl` 的 ip 地址

![QQ截图20221112110738](wsl%E5%AE%89%E8%A3%85.assets/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-ip%E5%9C%B0%E5%9D%80.png)

输入以下指令进行端口转发

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=<windows port> connectport=<wsl port> connectaddress=<wsl ip>
```

```
listenport, 表示要监听的 Windows 端口
listenaddress, 表示监听地址, 0.0.0.0 表示匹配所有地址, 比如Windows 既有Wifi网卡, 又有有线网卡, 那么访问任意两个网卡, 都会被监听到,当然也可以指定其中之一的IP的地址
connectaddress ,要转发的地址
connectport, 要转发到的端口
```

查看所有的端口转发

```powershell
netsh interface portproxy show all
```

删除转发

```powershell
netsh interface portproxy delete v4tov4 listenport=<windows port> listenaddress=0.0.0.0
```

