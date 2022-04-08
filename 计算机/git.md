# git 

## 配置ssh到github

```shell
ssh-keygen -t rsa -C "你的邮箱"
```

全部回车即可

## git bash初始化

设置用户

```shell
git config --global user.name "Your Name"（注意前边是“- -global”，有两个横线）
git config --global user.email "email@example.com"
```

## git拒绝连接

问题

```
ssh: connect to host github.com port 22: Connection refused
```

在`./ssh`目录下新建文件`config`，输入

```
Host github.com  
User xxxx@xx.com
Hostname ssh.github.com  
PreferredAuthentications publickey  
IdentityFile ~/.ssh/id_rsa  
Port 443
```

即可

## git 不显示中文

```bash
git config --global core.quotepath false
```

## git 分离子库

采用 `git subtree` 工具

首先进入原仓目录，注意需在工作区顶级目录，否则会如下提醒

```bash
您需要在工作区的顶级目录中运行这个命令。
```

运行指令

```bash
git subtree split -P <要独立出去的子仓库文件夹路径> -b <分支名称（随便起）>
```

运行后，git 会遍历仓库中所有的提交记录，挑选出和指定路径有关的提交记录，存入新分支中。

到仓库外新建一个仓库

```bash
cd ..
mkdir <新仓库名>
cd <新仓库名>
git init
```

把新建的分支拉取到这个仓库

```bash
git pull <原仓库路径> <新分支名称>
```

新仓库已包含要独立出去的子仓库文件夹中的所有文件和提交记录。可以放心清理原仓库了

```bash
cd <原仓库路径>
git rm -rf <要独立出去的子仓库文件夹路径>
git commit -a -m "${删除说明}"
git branch -D <新分支名称>
```

将新子仓库推送到 GitHub，操作省略

使用下列指令将子仓库添加到原仓库中

```bash
git subtree add --prefix=<子仓库路径前缀> <github 地址> [--squash]
```

`--squash` 表示不拉取历史记录，只生成一条提交记录信息

执行 `git status` 可以看到有多条提交记录

执行 `git push` 将修改推送到远程仓库

## git subtree

git subtree 主要指令

```bash
git subtree add   --prefix=<prefix> <commit>
git subtree add   --prefix=<prefix> <repository> <ref>
git subtree pull  --prefix=<prefix> <repository> <ref>
git subtree push  --prefix=<prefix> <repository> <ref>
git subtree merge --prefix=<prefix> <commit>
git subtree split --prefix=<prefix> [OPTIONS] [<commit>]
```

通过添加 remote 仓库的方法简化仓库地址

修改拉去子仓库代码均需使用 `subtree` 的相关指令，并在原仓库提交推送

## 统计代码行数

```bash	
git ls-files | xargs wc -l
```

