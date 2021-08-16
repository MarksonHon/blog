---
title: 在 Windows 上配置 VSCode 搭配 Clang 使用
date: 2021-08-16T07:16:13.000Z
draft: false
---

# 下载 winlibs

> 下载地址：<https://winlibs.com/>

需要下载包含了 Clang 的版本，同时建议下载 7z 的版本，该版本具有很高的压缩率，适合节省带宽。

下载后用 7zip 将压缩包解压，然后将解压目录添加到当前用户的 path 里面。

# 配置 VSCode

安装两个扩展：`clangd` 与 `codelldb`。两个扩展的具体作用可以查看其详情页面。

## 配置任务

打开一个 C++ 任务（以 Hello World 为例子），点击"终端"、"配置任务"，然后选择"Build with Clang"，然后编辑器会自动打开 `tasks.json`，里面的内容可以按需修改：

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build with Clang", 
            "type": "shell",
            "command": "clang++",
            "args": [
                "${fileBasenameNoExtension}.cpp",
                "-o",
                "${fileBasenameNoExtension}.exe",
                "--debug"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

再点击"运行"、"打开配置"，然后编辑器会自动打开 `launch.json`，配置如下：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch",
      "type": "lldb",
      "request": "launch",
      "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe",
      "preLaunchTask": "Build with Clang"
    }
  ]
}
```

`Hello-World.cpp` 如下：

```cpp
#include <stdio.h>

int main()
{
    printf("Hello World!");
}
```

启动调试，VSCode 下方的终端会自动弹出，你可以在终端看到 `Hello World!`，同时可执行文件已经生成。