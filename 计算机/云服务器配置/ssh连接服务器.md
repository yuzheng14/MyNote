# ssh链接服务器

## 连接服务器

cmd或powershell中键入

```powershell
ssh 账户名@IP地址或域名
```

## 免密登录

以windows远程登陆linux服务器为例

参考配置ssh到github生成ssh密钥

#### 将密钥上传到服务器

```shell
scp ~/.ssh/id_rsa.pub user@url:/download/id_rsa.pub
```

#### 登录服务器添加密钥

```shell
cat /download/id_rsa.pub >> ~/.ssh/authorized_keys
```

即可

## 设置别名

修改`~/.ssh/config`，添加如下记录

```shell
Host 别名
HostName url
User username
Port 22
IdentityFile ~/.ssh/id_rsa 
```

