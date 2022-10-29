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

## 多 Java 版本管理

如果你在你的 Ubuntu 系统上安装了多个 Java 版本，你可以输入下面的命令，检测哪个版本被设置成了默认值：

```bash
java -version
```

复制

想要修改默认的版本，使用`update-alternatives`命令：

```bash
sudo update-alternatives --config java
```

复制

输出像下面这样：

```
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

所有已经安装的 Java 版本将会列出来。输入你想要设置为默认值的序号，并且按"Enter”。
