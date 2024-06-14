---
title: gdb调试vscode配置
date: 2024-02-03
author: 3oR
categories:
  - VSCODE
tags:
  - gdb
  - WSL
# cover: /images/cover.jpg # 文章封面图片路径
# thumbnail: /images/thumbnail.jpg # 缩略图路径
description: 文章描述，这里可以是一段简短的介绍
# keywords: 关键字1, 关键字2 # SEO 关键字
top: false # 置顶文章（可选）
comments: true # 是否开启评论
---
# c/cpp project debug on wsl with vscode



### launch.json

```json
{
    "configurations": [
        {
            "name": "GDB",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/benos.elf",
            "stopAtEntry": true,
            "cwd": "${fileDirname}",
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb-multiarch",
            "miDebuggerServerAddress": "localhost:1234"
        }
    ]
}
```

### task.json

```
{
	"" : ""
}
```

