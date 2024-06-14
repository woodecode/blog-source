---
title: 标准IO
date: 2024-04-02
author: 3oR
categories:
  - Linux
tags:
  - Linux
# cover: /images/cover.jpg # 文章封面图片路径
# thumbnail: /images/thumbnail.jpg # 缩略图路径
description: 文章描述，这里可以是一段简短的介绍
# keywords: 关键字1, 关键字2 # SEO 关键字
top: false # 置顶文章（可选）
comments: true # 是否开启评论
---

# 标准IO

## 文件操作

### fopen()/fclose() 

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h> // strerror

int main()
{
    FILE *fp;
    fp = fopen("temp", "r");
    if(fp == NULL)
    {
		fprintf(stderr, "fopen() failed, errno = %d\n", errno);
        perror("fopen():");
        fprintf(stderr, "fopen():%s\n", strerror(error));
        exit(1);
    }
    printf("OK\n");
    
    fclose(fp);
    
    exit(0);
}
```

.

## 字符操作/二进制文件操作

  - `fgetc()` 
  - `fputc() `
  - `fgets()` 
  - `fputs()` 
  - `fread()` 
  - `fwrite()` 

## 格式化输出
  - `printf()` 
  - `scanf()` 

## 文件位置操作
  - `fseek()` 
  - `ftell()` 
  - `rewind()` 

## 刷新缓冲区
  - `fflush()` 

