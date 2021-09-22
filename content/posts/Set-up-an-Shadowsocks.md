---
title: "部署一个 Shadowsocks 服务器"
date: 2021-08-12T15:41:14+08:00
draft: false
---

# 前言

## 服务端配置

* 一个 RAM 为 128MB 的、或者拥有更多 RAM 的 VPS，且此 VPS 拥有公网 IP 地址，或者是一个做好了端口映射的 NAT VPS。  
* 一个 Linux 系统，比如 Alpine Linux，该发行版适合在低 RAM 的 VPS 上运行。也可以使用 Debian 系统，但是本文会以 Alpine Linux 作为服务器系统来继续。  

## 客户端的选择

* 跨平台的命令行客户端，比如 Shadowsocks-Rust 中的 `sslocal` 命令。
* 针对于各个平台的客户端，比如 Shadowsocks-Windows 与 ShadowsocksX-Ng。
* 其它的通用客户端，比如 Clash for Windows、Qv2ray、v2rayN、SagerNet 或 ShadowRocket。

# 准备服务端

## 连接到 VPS

[通过 netboot.xyz 安装 Alpine Linux](Use-Netboot-to-Install-Any-OS)，记得开启 OpenSSH 或 Dropbear，然后通过 PuTTY、`ssh` 命令等工具连上你的 VPS。

## 下载与安装 Shadowsocks-Rust

>下载地址：  
><https://github.com/shadowsocks/shadowsocks-rust/releases>

**安装需要的工具**

```bash
apk add wget p7zip nano
```

**下载**

```bash
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.11.2/shadowsocks-v1.11.2.x86_64-unknown-linux-musl.tar.xz
```

由于 Alpine Linux 使用的 C 库是 musl，所以这里下载 musl 的版本，Debian 系统使用的是 glibc 库，所以要下载用 gnu 字样的版本。

**解压**

```bash
7z e ./shadowsocks-v1.11.2.x86_64-unknown-linux-musl.tar.xz -so |  7z x -si -ttar
mv ss* /usr/local/bin
```

**给予二进制文件可执行权限**

```bash
chmod +x /usr/local/bin/ss*
```

## 运行服务端

**配置 OpenRC 服务**

新建一个名为 `shadowsocks-rust` 的文件，将其保存到 `/etc/init.d/`。

```bash
nano /etc/init.d/shadowsocks-rust
```

输入内容如下：

```ini
#!/sbin/openrc-run

name="Shadowsocks Rust Port Server"
description="A port of shadowsocks in Rust"

command="/usr/local/bin/ssserver"
command_args="--config /usr/local/etc/shadowsocks-rust/config.json"
command_user="nobody"
pidfile="/run/shadowsocks-rust.pid"
command_background="yes"

depend() {
        need net
}
```

保存文件，然后给予它可执行权限：

```bash
chmod +x /etc/init.d/shadowsocks-rust
```

**配置 Shadowsocks**

新建 Shadowsocks-Rust 的配置：

```bash
mkdir -p /usr/local/etc/shadowsocks-rust/
nano /usr/local/etc/shadowsocks-rust/config.json
```

输入内容如下，然后保存：

```json
{
    "server": "::",
    "server_port": 10086,
    "password": "Your_Password",
    "method": "aes-128-gcm"
}
```

建议使用高强度的密码，比如一个 UUID，[UUID Generator](https://www.uuidgenerator.net/) 这个网站可以为你随机生成 UUID。

加密方式（method）建议使用 `aes-xxx-gcm` 系列与 `chacha-ietf-poly1305`，即“AEAD 加密”。

**运行 Shadowsocks**

使用 OpenRC 的服务管理工具运行服务：

```bash
rc-service shadowsocks-rust start
```

开机启动：

```bash
rc-update add shadowsocks-rust
```

# 客户端

大多数客户端都有图形界面以供配置服务器，这里只提供 Shadowsocks-Rust 中 `sslocal` 命令所需要的配置：

```json
{
    "server": "Your_Server_IP",
    "server_port": 10086,
    "password": "Your_Password",
    "method": "aes-128-gcm",
    "local_address": "127.0.0.1",
    "local_port": 1080
}
```

`sslocal` 默认使用 socks5 入站，如果需要 HTTP 入站，你需要搭配 [Privoxy](https://www.privoxy.org/) 或者类似的软件来使用。

详见：<https://github.com/shadowsocks/shadowsocks-rust>
