---
title: "Alpine Linux 上搭建 Halo 博客"
date: 2021-07-08T13:46:37+08:00
draft: false
tags: Linux
---

# 安装 Nginx 与 Certbot

我的配置是：

**操作系统：** Alpine Linux 3.13   
**VPS：** 随便买的

编辑 `/etc/apk/repositories`，打开 `community` 源，然后运行：

```bash
apk add nginx certbot certbot-nginx
```

# Nginx 配置 与 SSL 证书

创建一个配置文件：

```bash
nano /etc/nginx/conf.d/your.domain.site.conf
```

在里面填写以下内容：

```conf
server {
#   root /var/www/html;
    server_name your.domain.site;

  location /  ## 代理 Halo 博客后端的配置
        {
      proxy_pass http://127.0.0.1:8090;
      proxy_redirect off;
      proxy_http_version 1.1;
      proxy_set_header HOST $host;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      client_max_body_size 1000m;
      sendfile on;
      tcp_nopush on;
      tcp_nodelay on;
      keepalive_requests 25600;
      keepalive_timeout 300 300;
      proxy_buffering off;
      proxy_buffer_size 8k;
    }
}
```

保存文件，然后运行：

```bash
certbot -d your.domain.site -m your@email.site
```

`-d`后面是你的域名，`-m`后面是你的邮箱。

命令正常完成后再次打开上面的配置文件，可以看到配置文件已经添加了证书与80端口跳转HTTPS的配置。

如果想要使用 HTTP/2 进行数据传输，可以在 443 端口配置后 `ssl` 字样后面加上 `http2` 这个配置。如果想要使用 ZeroSSL 替代Certbot 默认的 Let's Encrypt 证书，你可以参考 [`ZeroSSL Bot`](https://github.com/zerossl/zerossl-bot) 。

# 安装 Halo 博客系统

## Halo 配置

[Halo 博客系统](https://github.com/halo-dev/halo) 需要使用 JRE 环境来运行，因此你应该先安装 JRE：

```bash
apk add openjdk11-jre
```

新建 `/etc/init.d/halo` 文件，添加内容如下：

```sh
#!/sbin/openrc-run

name="Halo blog"
description="Halo is a modern personal independent blog system"

command="/usr/bin/java"
command_args="-jar /opt/halo/halo.jar"
pidfile="/run/halo.pid"
command_background="yes"

depend() {
	need net
}
```

给此文件以可执行权限：

```bash
chmod +x /etc/init.d/halo
```

## 下载 Halo

下载地址：<https://github.com/halo-dev/halo/releases>

```bash
mkdir /opt/halo; ## 如果目录存在则不用新建
curl -L https://github.com/halo-dev/halo/releases/download/v1.4.8/halo-1.4.8.jar --output /opt/halo/halo.jar
```
## 运行

```bash
rc-service nginx start; rc-service halo start
```

开机自启：

```bash
rc-update add nginx; rc-update add halo
```