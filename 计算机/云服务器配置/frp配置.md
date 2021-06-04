# Frp配置

## 下载S/C客户端

注意服务端和客户端要用相同的版本

## 解压

```bash
tar -zxvf frp_0.30.0_linux_amd64.tar.gz
```

## 配置frps.ini文件

```bash
vim frps.ini
```

```
[common]
bind_port = 7000
vhost_https_port = 7001		#当代理出来的是web服务时，在外网访问http://vps的IP:7001
#dashboard_port状态以及代理统计信息展示,网址:7500可查看详情
dashboard_port = 7500
#dashboard_user访问用户dashboard_pwd访问密码
dashboard_user = admin
dashboard_pwd = password
#log_file日志文件log_level记录的日志级别log_max_days日志留存3天authentication_timeout超时时间
log_file = ./frps.log
log_level = info
log_max_days = 3
authentication_timeout = 0
#max_pool_count最大链接池,每个代理预先与后端服务器建立起指定数量的最大链接数
max_pool_count = 50
```

## 启动frps

```bash
./frps -c frps.ini
```

启动后可以在浏览器中访问`https://ip:7500`来观察dashboard是否启动

因为配置了`log_file=./frps.log`所以没有日志输出

