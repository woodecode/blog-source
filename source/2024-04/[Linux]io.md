
```json
{
    "front-matter": {
        "title": "标准IO",
        "date": "2024-04-02",
        "author": "我是谁",
        "tags": ["Example1", "Example1"],
        "categories": ["Cate1", "Cate1"],
        "description": "文章描述",
        "cover": "封面图片链接",
        "featured": false, 
        "draft": true 
	}
}
```

# 标准IO

## 文件操作

### fopen()

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    FILE *fp;
    fp = fopen();
    if(fp == NULL)
    {
		printf("fopen() failed, errno = %d\n", errno);
     //  
        exit(1);
    }
    printf("OK\n");
    
    exit(0);
}
```

- `fclose()` 

  - >a
    >
    >

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

