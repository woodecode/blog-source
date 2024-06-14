---
title: 使用sdl2在windows环境下渲染帧
date: 2024-02-16
author: 3oR
categories:
  - NES
  - GiM
tags:
  - ARM
# cover: /images/cover.jpg # 文章封面图片路径
# thumbnail: /images/thumbnail.jpg # 缩略图路径
description: 文章描述，这里可以是一段简短的介绍
# keywords: 关键字1, 关键字2 # SEO 关键字
top: false # 置顶文章（可选）
comments: true # 是否开启评论
---
# 使用sdl2在windows环境下渲染帧

## 本机环境

windows 11

mingw64-posix

> gcc -v
>
> gcc version 8.1.0 (x86_64-posix-sjlj-rev0, Built by MinGW-W64 project)

vscode

## 下载SDL2

1. 访问链接 https://github.com/libsdl-org/SDL/releases/latest

   

2. 下载 SDL2-devel-x.xx.x-mingw.tar.gz 或 SDL2-devel-x.xx.x-mingw.zip 

   (其中 x.xx.x 为具体版本号，笔者版本号为2.30.0)

## 复制库文件

1. 解压压缩包到文件夹` SDL2-2.30.0`。

2. 在工程下新建`libs/SDL2`文件夹，将文件夹` SDL2-2.30.0/x86_64-w64-mingw32`目录下的`include`和`lib`文件夹复制到新建的`libs/SDL2`文件夹中。

3. 将文件夹` SDL2-2.30.0/x86_64-w64-mingw32/bin`目录下的`SDL2.dll`文件复制到工程目录下。



## 工程结构

```
.
├── main.c
├── Makefile
├── SDL2.dll
├── libs
│   └── SDL2
│       ├── include
│       │   └── SDL2
│       │       ├── SDL.h
│       │       └── SDL_xxx.h
│       └── lib
│           ├── libSDL2xxx.a
│           └── libSDL2xxx.la
└── nes
```



## 编写Makefile

```makefile
# ################################################
#                      工程
# ################################################
TARGET = GiM

# 构建目录
BUILD_DIR = build

GCC_PATH := C:/mingw64-posix/bin/

CC := $(GCC_PATH)/gcc
LD := $(GCC_PATH)/gcc

# ################################################
#                    源文件
# ################################################

INCLUDE_DIRS = \
	-Ilibs/SDL2/include

LIBS = \
	-L./libs/SDL2/lib \
	-lSDL2 \
	-lmingw32

SOURCES = \
	main.c

# ################################################
#                  构建可执行文件
# ################################################


CFLAGS  = $(INCLUDE_DIRS) -Wall -Wextra -g 

LDFLAGS = $(LIBS)

vpath %.c $(sort $(dir $(SOURCES)))

# list of objects
OBJECTS  =  $(addprefix $(BUILD_DIR)/,$(notdir $(SOURCES:.c=.o)))


$(BUILD_DIR)/%.o: %.c   Makefile | $(BUILD_DIR) 
	$(CC) -c $< -o $@ $(CFLAGS) 


$(TARGET): $(OBJECTS) Makefile
	$(LD) $(OBJECTS) $(LDFLAGS) -o $@

$(BUILD_DIR):
	mkdir $@	

#######################################
# 命令
#######################################

all: $(TARGET)

.PHONY : clean
clean:
	del /Q /F $(BUILD_DIR)
	del /Q /F $(TARGET).exe


# *** EOF ***
```

