---
title: 美化 Powershell
date: 2021-08-16T06:37:41.000Z
draft: false
tags: powershell
---

# 安装

首先要下载 PowerShell，而不是用系统自带的 Windows PowerShell。PowerShell 是开源的且拥有更多的特性，oh-my-posh 也是适配 PowerShell 的。一般来说，安装 stable 版本的 PowerShell 即可。

>下载地址：<https://github.com/powershell/powershell>

打开安装好的 PowerShell，运行：

```ps1
Install-Module oh-my-posh -Scope CurrentUser
Install-Module posh-git -Scope CurrentUser
```

如果有提示则选择 A，代表全部接受。然后从 Microsoft Store 安装 Windows Terminal。

# 配置

编辑 PowerShell 的配置：

```ps1
notepad $PROFILE
```

内容大致如下：

```ps1
Import-Module posh-git
Import-Module oh-my-posh ##导入模块
Set-PoshPrompt -Theme gmay  ##设置主题，主题可以使用 Get-PoshThemes 查看
Set-PSReadLineOption -PredictionSource History  ##预测源来源于历史操作记录
Set-PSReadlineKeyHandler -Key Tab -Function Complete  ## Tab 键用于补全
Set-PSReadLineKeyHandler -Key "Ctrl+d" -Function MenuComplete
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
```

保存，然后安装 [Delugia Mono](https://github.com/adam7/delugia-code) 字体，建议下载“delugia-mono-complete.zip”这个包然后安装。

然后在 Windows Terminal 里面修改 PowerShell 的配置文件，把字体修改为 `delugia mono` 即可。