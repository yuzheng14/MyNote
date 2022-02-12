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

