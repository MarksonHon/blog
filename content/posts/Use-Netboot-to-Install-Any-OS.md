---
title: "通过网络启动给 VPS 安装系统"
date: 2021-07-17T10:27:12+08:00
draft: false
---


# 1、什么是网络启动？

网络启动是指从网络加载操作系统并启动计算机的一个过程，它常用于网吧的无盘系统、以前很流行的 Ghost 网络克隆，以及微软的 Windows 部署服务，等等。

[netboot.xyz](https://netboot.xyz/) 提供了一组在线的工具与对应的网站，使我们可以在基于全虚拟化的 VPS 上安装各种系统。

这篇文章主要讲解如何在 grub 中添加启动项，然后启动网络安装。不过，只有全虚拟化技术（比如 KVM、Hyper-V）的 VPS 才能如此进行操作。另外，不提供 VNC 访问的 VPS 同样无法进行系统安装。

# 2、安装启动项

远程连接到你的 VPS，切换到 root 用户。然后，切换到 `/boot` 目录。从<https://boot.netboot.xyz/>下载 `netboot.xyz.lkrn` 这个文件并保存到当前目录，这个是用于从 grub 启动的核心文件。

```sh
wget https://boot.netboot.xyz/ipxe/netboot.xyz.lkrn
```

编写 `netboot.xyz-initrd` 文件，保存到当前目录。注意，IP 地址、地址掩码、网关需要修改成你自己的。DNS 可以随意设置一个，比如著名的四个一或四个八。

```conf
#!ipxe
#/boot/netboot.xyz-initrd
imgfree
set net0/ip <instance public ip>
set net0/netmask <instance public netmask>
set net0/gateway <instance public gateway>
set dns <instance dns address>
ifopen net0
chain --autofree https://boot.netboot.xyz
```

修改 ` /etc/grub.d/40_custom` 文件，加上以下内容：

```conf
menuentry 'netboot.xyz' {
    set root='hd0,msdos1'
    linux16 /boot/netboot.xyz.lkrn
    initrd16 /boot/netboot.xyz-initrd
}
```

更新 grub 配置：

```sh
update-grub
```

或

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

# 3、重启

重启 VPS，然后通过 VNC 访问，在启动菜单中选择 `netboot.xyz` 来进行启动。

![启动页面](https://netboot.xyz/images/netboot.xyz.gif)

如此就可以安装自己需要的系统了。不过，RAM 小于 256M 的 VPS 将只能安装 Alpine Linux。