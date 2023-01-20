# WSL 如何安装 WebStorm

安装在 Windows 下的 WebStorm 不知道为什么总是不识别 `pnpm`，一气之下打算将 WebStorm 装在 WSL 里面。

## 安装 WebStorm

首先去官网找到下载连接

[Download WebStorm: The Smartest JavaScript IDE by JetBrains](https://www.jetbrains.com/webstorm/download/#section=linux)

下载下来压缩包并解压，创建软连接

```shell
# 更改目录
cd /tmp

# 下载 webstorm
wget <下载链接>

# 解压
tar -zxvf <下载下来的压缩包名>

# 创建软连接到 /usr/local/bin
ln -s <解压路径> /usr/local/bin/webstorm
```

启动过程中如果报下列错误

```
libXtst.so.6: cannot open shared object file: No such file or directory
```

则是缺少几个动态库

```shell
sudo apt install -y libxtst6 libxi6 libxp6 libxrender1
```

做好之后使用 `webstorm` 指令就可以了

## 安装 Edge 浏览器

如果你是使用 JB account 来进行认证的话，需要打开浏览器，这就需要在 WSL 内安装一个浏览器

首先去官网找到下载链接

[Microsoft Edge](https://www.microsoft.com/zh-cn/edge/home?form=MA13FJ)

然后下载安装

```shell
# 更改目录
cd /tmp

# 下载 webstorm
wget <下载链接>

# 获取当前稳定版本
sudo dpkg -i <下载包名>

# 修复包
sudo apt install --fix-broken -y

# 配置包
sudo dpkg -i <下载包名>
```

