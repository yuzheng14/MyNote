# wsl 如何使用系统代理

> 环境：
>
> Ubuntu 20.04
>
> Clash For Windows

在 Clash 的 `General` 中找到 `Port`

![image-20221011193331888](wsl%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%90%86.assets/image-20221011193331888.png)

wsl1 直接在 wsl 环境中将以下两行加入到 `~/.bashrc` 的末行

```bash
export http_proxy=127.0.0.1:<your port>
export https_proxy=127.0.0.1:<your port>
```

wsl2 由于使用 Hyper-V 技术，与宿主系统不共享端口，所以需要获取到宿主系统的 IP

加入到 `~/.bashrc` 末行的内容改为

```bash
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890";
export http_proxy="http://${hostip}:7890";
```

并打开 clash 的Allow LAN 设置，由于此设置选项为一次性，故打开配置文件进行更改