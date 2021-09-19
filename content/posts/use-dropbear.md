---
title: "Debian 使用 Dropbear 替换 OpenSSH"
date: 2021-09-2T06:37:41.000Z

description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories:
comment: true
draft: true
---

Dropbear 是一个精简的 SSH 版本，相对于 OpenSSH 而言，Dropbear 取消了一些向后的兼容性功能，这使得它的体积相对较小，且更容易运行于路由器、交换机等小内存设备。

## 安装 Dropbear

```bash
sudo apt install dropbear
```

## 修改配置

Dropbear 的配置文件位于 `/etc/default/dropbear`，使用文本编辑器打开它，然后修改以下几项：

```ini
# change to NO_START=0 to enable Dropbear
NO_START=0
# the TCP port that Dropbear listens on
DROPBEAR_PORT=2022
DROPBEAR_EXTRA_ARGS=-w
```

额外选项 `-w` 是为了禁用 root 账户的登录。作为替代，你需要添加一个新的用户，并把这个用户添加到 `sudo` 组。

```bash
sudo useradd -m -G sudo username
```

添加用户后**一定要记得修改密码**。

## 启用 Dropbear

```bash
sudo systemctl enable dropbear --now
```

## 卸载 OpenSSH

通过 2022 端口重新登录到你的服务器，登录之后就可以卸载 OpenSSH 了。

```bash
sudo apt remove openssh-server --autoremove
```
