# MacOS 给 homebrew 安装的 NGINX 加载 njs 模块

在 MacOS 上使用 `homebrew` 装软件是个司空见惯的行为，他会自动配置好依赖和各种环境，非常方便便捷。在安装 `NGINX` 的时候只需要执行 `brew install nginx` 指令就可以了。同时通过 `homebrew` 的 `brew services` 指令管理各种服务的启停也很方便。

但是最近在搞 `nginScript` （以下简称 `njs` ）的时候却遇到了很大的问题：无法编译出来 `njs` 的二进制模块，且没有任何参考文档，似乎除了完全从二进制编译 `NGINX` 以外别无他法，想要用 `njs` 就只能放弃使用 `homebrew` 进行安装管理。同时官网上有些步骤使用的指令也有些过时了。

经过个人解读源码及一定的大胆尝试后发现了另外一条路，本文只谈操作，背后的原理和心路历程不再赘述，感兴趣的小伙伴多的话也可以另开一篇文章写一写。

## 下载源码

首先需要下载下来 `NGINX` 和 `njs` 模块的源码。

```shell
# 官方文档中使用 hg 下载源码
hg clone http://hg.nginx.org/nginx
hg clone http://hg.nginx.org/njs

# 个人探索发现 NGINX 组织在 GitHub 有个镜像
git clone git@github.com:nginx/nginx.git
git clone git@github.com:nginx/njs.git

# 如果没有配置 GitHub ssh key 的话就使用下面的命令 clone
git clone https://github.com/nginx/nginx.git
git clone https://github.com/nginx/njs.git
```

下载下来源码后进入 `NGINX` 的源码目录

```shell
cd nginx
```

切换 tag 到 `homebrew` 安装的版本

```shell
git checkout $( git tag | grep $( nginx -v 2>&1 | awk '{print substr($3, 7, length($3) - 6)}' ))
```

> 这个指令可以一键切换到 `homebrew` 安装的脚本，你也可以用 `nginx -v` 的结果切换 tag

## 编译二进制文件

编译 `NGINX` 需要很多参数，我们可以用 `nginx -V` 查看配置参数

```shell
➜  nginx git:(master) ✗ nginx -V
nginx version: nginx/1.25.4
built by clang 15.0.0 (clang-1500.1.0.2.5)
built with OpenSSL 3.2.1 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/opt/homebrew/Cellar/nginx/1.25.4 --sbin-path=/opt/homebrew/Cellar/nginx/1.25.4/bin/nginx --with-cc-opt='-I/opt/homebrew/opt/pcre2/include -I/opt/homebrew/opt/openssl@3/include' --with-ld-opt='-L/opt/homebrew/opt/pcre2/lib -L/opt/homebrew/opt/openssl@3/lib' --conf-path=/opt/homebrew/etc/nginx/nginx.conf --pid-path=/opt/homebrew/var/run/nginx.pid --lock-path=/opt/homebrew/var/run/nginx.lock --http-client-body-temp-path=/opt/homebrew/var/run/nginx/client_body_temp --http-proxy-temp-path=/opt/homebrew/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/opt/homebrew/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/opt/homebrew/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/opt/homebrew/var/run/nginx/scgi_temp --http-log-path=/opt/homebrew/var/log/nginx/access.log --error-log-path=/opt/homebrew/var/log/nginx/error.log --with-compat --with-debug --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-ipv6 --with-mail --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module
```

现在我们就可以使用参数配置 `NGINX` 源码了，直接在 `NGINX` 的源码根目录内执行如下指令

```shell
eval ./auto/configure --add-dynamic-module=../njs/nginx $( nginx -V 2>&1 | tail -n 1 | awk '{$1=$2=""; print $0}' )
```

执行完成之后根目录下将生成一个 `objs` 文件，结构如下

```
objs
├── addon
│   ├── external
│   └── nginx
├── src
│   ├── core
│   ├── event
│   │   ├── modules
│   │   └── quic
│   ├── http
│   │   ├── modules
│   │   │   └── perl
│   │   ├── v2
│   │   └── v3
│   ├── mail
│   ├── misc
│   ├── os
│   │   ├── unix
│   │   └── win32
│   └── stream
├── Makefile
├── autoconf.err
├── ngx_auto_config.h
├── ngx_auto_headers.h
├── ngx_http_js_module_modules.c
├── ngx_modules.c
└── ngx_stream_js_module_modules.c

20 directories, 7 files
```

接着执行 `make modules` 指令，`objs` 目录下将生成 `objs/ngx_http_js_module.so` `objs/ngx_stream_js_module.so` 这两个文件，把两个文件复制到 `homebrew` 安装的 `NGINX` 路径下即可，执行下面的指令就可以直接把这两个文件复制过去了

```shell
mkdir "$( realpath $( where nginx | head -n 1)/../.. )/modules" && cp objs/*.so $( realpath $( where nginx | head -n 1)/../../modules )
```

## 配置 NGINX 加载 njs 模块

现在我们就可以在 `NGINX` 的配置文件中加载 `njs` 模块了

```nginx
# nginx 的配置文件

load_module modules/ngx_http_js_module.so;
```

执行 `nginx -t` 后发现出现类似如下输出就可以了

```
➜  nginx git:(stable) nginx -t
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```

现在就可以在 `nginx` 里面加载 `nginScript` 了！