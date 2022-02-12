# Linux

## 修改中文

### ubuntu

首先输入`locale -a`查看是否安装有中文语言包，即`zh_CN.utf8`

如果没有，则需要安装中文语言包，输入以下指令

```bash
sudo apt-get install language-pack-zh-hans
```

添加中文支持

```bash
locale-gen zh_CN.UTF-8
```



## sudo免密码

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

依次按下<kbd>Ctrl</kbd>+<kbd>O</kbd>，<kbd>Enter</kbd>，<kbd>Ctrl</kbd>+<kbd>X</kbd>

即可

