```json
{
    "front-matter": {
        "title": "gdb调试vscode配置",
        "date": "2024-02-03",
        "tags": ["VSCODE", "wsl", "gdb"],
        "categories": ["RISC-V"],
        "description": "NULL",
        "cover": "封面图片链接",
        "featured": false, 
        "draft": true 
	}
}
```

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

