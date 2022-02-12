# WSL安装教程

本文源于[Microsoft官方文档]([Install WSL | Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/install))

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

