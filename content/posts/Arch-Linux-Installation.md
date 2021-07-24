---
title: "Arch Linux 安装教程"
date: 2021-07-07T10:27:12+08:00
draft: false
---

# 一、准备安装媒介

## 1.1 下载镜像

>官方镜像地址：
>[https://www.archlinux.org/download/](https://www.archlinux.org/download/)

各大镜像源一般也会提供 Arch Linux 的安装镜像下载，下载之后需要验证下载完整性。

## 1.2 制作 U 盘

可以通过以下两款工具来制作镜像 U 盘：

>Rufus：<https://rufus.ie/downloads/>   
>Etcher: <https://www.balena.io/etcher/>   

>注意：Windows 10 ARM / ARM64 用户只能使用 Rufus 的 arm / arm64 版本来制作。

# 二、给Live环境联网

Arch Linux 的安装文件需要从互联网下载，因此安装之时需要联网。

## 2.1 有线网络

如果你使用有线网络上网，那么你需要接好网线，后台将自动进行 DHCP。

## 2.2 无线网络

如果你使用无线网卡，那么首先输入以下命令搜索 WiFi：

```sh
iwctl
```
>iwctl 使用帮助：<https://wiki.archlinux.org/index.php/Iwd>

进入 iwd 的操作命令行里面后，可以先用 ``help`` 来查看帮助信息。

```sh
# 查看启动了的网络界面（一般是一个 wlan0）
station list 
# 扫描并连接 WiFi（支持 Tab 补全 WiFi 名称）
station wlan0 scan
station wlan0 get-networks
station wlan0 connect WiFi-SSID
# 退出
quit
```
运行这个命令来查看IP地址：

```sh
ip addr
```

如果除了lo以外的设备获取到了IP地址，说明你的网络设置完成了。你可以随意```ping```一个网站试试网络是否正常。

# 三、安装

## 3.1 准备分区

建议使用 cfdisk 来编辑分区。编辑分区之前，你需要了解 Linux 对磁盘与分区的标记逻辑，比如，你需要知道 `nvme0n1p1` 与 `sda1` 是什么。

>Arch Linux 官方分区 Wiki：<https://wiki.archlinux.org/index.php/Partitioning>

```sh
cfdisk /dev/sdX
```


格式化新分区：

```sh
mkfs.ext4 /dev/sdXN
```

把刚刚格式化的分区作为主分区进行挂载：

```sh
mount /dev/sdXN /mnt
 ```

格式化 EFI 分区（该步骤非必须操作，一般只在新建 ESP 的时候才运行）：

```sh
mkfs.vfat /dev/sdX1
```

挂载EFI分区到 ```/boot/efi``` 目录（仅 UEFI 启动需要）:

```sh
mkdir -p /mnt/boot/efi
mount /dev/sdX1 /mnt/boot/efi
 ```

## 3.2 准备软件源

Arch Linux 的安装镜像会自动根据速度来自动建立一个软件源列表，因而一般无需手动编辑软件源。如果你需要修改软件源以选择你感觉最快的服务器，使用 nano 或者 vim 打开软件源配置文件：

```sh
nano /etc/pacman.d/mirrorlist
```

在文件开头加上至少一个中国的软件源，不过建议多添加几个：

```conf
## 中国的软件源
## 腾讯
Server = https://mirrors.cloud.tencent.com/archlinux/$repo/os/$arch
## 华为
Server = https://mirrors.huaweicloud.com/archlinux/$repo/os/$arch
```

## 3.3 安装系统

安装基本包

```sh
pacstrap /mnt base base-devel linux-lts linux-lts-headers linux-firmware nano
```
>注意：如果你的硬件较新，建议使用 `linux` 替换 `linux-lts`，`linux-headers` 替换 `linux-lts-headers`。

生成 fstab 文件（必须步骤）

```sh
genfstab -U /mnt > /mnt/etc/fstab
```

校验文件是否生成：

```sh
cat /mnt/etc/fstab
```

其内容一般包含你设置的 Linux 系统的所有的分区。

# 四、配置

## 4.1 基本配置

使用 arch-chroot 进入到新系统

```sh
arch-chroot /mnt
```

设置时区

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

修改 root 密码

```sh
passwd root
```

设置 locale，使用 nano 编辑 ```/etc/locale.gen``` ，一般取消 ```zh_CN```开头的注释即可，然后运行以下命令以配置语言：

```sh
locale-gen
```

新建或者编辑 ```/etc/locale.conf``` 文件，配置全局语言。

```sh
echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```
>如果不使用图形界面则需要把本地设置改为 ```LANG=en_US.UTF-8``` ,这是为了 TTY 始终以英文显示（在 TTY 下，中文会显示成一个个方块或者方框）。

新建 ```/etc/hostname``` 文件，用于保存主机名，同时，编辑 ```/etc/hosts``` 文件，设置```localhost```本地回环 IP 与你的主机 IP（替换下面的 hostname 为你自己设置的主机名）：

```conf
127.0.0.1 localhost
::1 localhost
127.0.1.1 hostname.localdomain hostname
```

安装 ucode

```sh
pacman -S intel-ucode # 或者安装 amd-ucode
```

## 4.2 安装启动管理器

>Arch Linux 官方 GRUB Wiki：<https://wiki.archlinux.org/index.php/GRUB>

安装基本程序：

```sh
pacman -S os-prober grub efibootmgr
```

安装 Grub 启动管理器：

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
```

如果是新建立的 ESP，那么建议再执行以下步骤，否则 Arch Linux 将可能只能启动一次：

```sh
mkdir /boot/efi/BOOT && cp /boot/efi/Arch/grubx64.efi /boot/efi/BOOT/BOOTx64.efi
```

## 4.3 添加非特权用户

新建用户

```sh
useradd -m -G wheel username
```

给新用户设置密码：

```sh
passwd username
```

你可以更改 sudo 设置，使得 wheel 组或者单个用户可以通过 sudo 命令临时调用 root 权限。

```sh
nano /etc/sudoers
```

## 4.4 nVidia 显卡闭源驱动

*此步骤为非必须步骤，除非你电脑只有独显。即使如此，你也不一定要安装闭源驱动。*

```sh
pacman -S mesa nvidia-lts nvidia-settings
``` 

## 4.5 网络服务

以下服务二选一，不可以同时启用。

### 4.5.1 Network Manager 服务（与大部分桌面集成很好）

```sh
pacman -S networkmanager
systemctl enable NetworkManager
``` 
### 4.5.2 systemd-networkd + iwd （通用的命令行界面）

```sh
pacman -S iwd
systemctl enable iwd && systemctl enable systemd-networkd && systemctl enable systemd-resolved
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

启用 DHCP（可选）

无线配置：

```conf
# /etc/systemd/network/wireless.network
[Match]
Name=wl*

[Network]
DHCP=yes
```

有线配置：

```conf
# /etc/systemd/network/wired.network
[Match]
Name=en*

[Network]
DHCP=yes
```

## 4.6 图形界面

图形界面可以安装多个，但是显示管理器只能启用一个。

>建议参考：<https://wiki.archlinux.org/index.php/Desktop_environment_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>

### Xorg 驱动

>官方文档：<https://wiki.archlinux.org/index.php/Xorg>

### Gnome

```sh
pacman -S gnome gnome-extra
systemctl enable gdm
```

### KDE Plasma

```sh
pacman -S plasma kde-system kde-utilities kde-graphics sddm xdg-user-dirs
systemctl enable sddm
```
## 4.7 蓝牙

安装蓝牙管理的相关包

```sh
pacman -S bluez-utils bluez
```

开启服务：

```sh
systemctl enable bluetooth
```

## 4.8 重启

退出 chroot 然后重启

```sh
exit
```

# 5、字体与输入法

## 5.1 字体

### 基本显示字体

```sh
sudo pacman -S noto-fonts noto-fonts-extra noto-fonts-emoji
```

### 中日韩统一汉字字体

__思源字体（推荐）__

```sh
sudo pacman -S adobe-source-han-mono-otc-fonts adobe-source-han-sans-otc-fonts adobe-source-han-serif-otc-fonts
```
>`adobe-source-han-mono-otc-fonts` 只在 AUR 或 Arch Linux CN 源提供，建议添加 Arch Linux CN 源来安装。

__Noto Sans CJK__
```sh
sudo pacman -S noto-fonts-cjk
```

__更纱黑体__
```sh
sudo pacman -S ttf-sarasa-gothic
```
>`ttf-sarasa-gothic` 只在 AUR 或 Arch Linux CN 源提供，建议添加 Arch Linux CN 源来安装。

### 汉字大字符集字体

__花园明朝__
```sh
sudo pacman -S ttf-hanazono
```

### 字体优先级

中日韩统一汉字字体同时包含了多个版本的汉字，而不同版本的汉字使用者的需求不一样，以下是以中国大陆版本的思源字体版本为例子：

新建```/etc/fonts/conf.avail/64-language-selector-prefer.conf```文件

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Source Han Sans SC</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif</family>
      <family>Source Han Serif SC</family>
      </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Noto Sans Mono</family>
      <family>Source Han Mono SC</family>
    </prefer>
  </alias>
</fontconfig>
```

启用该配置：

```sh
sudo ln -s /etc/fonts/conf.avail/64-language-selector-prefer.conf /etc/fonts/conf.d/64-language-selector-prefer.conf
```

## 5.2 输入法

>Arch Linux 官方文档  
>iBus：<https://wiki.archlinux.org/index.php/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>  
>fcitx4：<https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>  
>fcitx5：<https://wiki.archlinux.org/index.php/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>  

# 六、第三方软件源

>官方文档：<https://wiki.archlinux.org/index.php/Unofficial_user_repositories_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>

可以按需添加自己需要的源，比如 Arch Linux CN 源。

# 七. 硬件解码加速

>官方文档：<https://wiki.archlinux.org/index.php/Hardware_video_acceleration>
