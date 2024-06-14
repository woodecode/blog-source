---
title: 第一个程序解析
date: 2024-01-22
author: 3oR
categories:
  - ARM
tags:
  - ARM
# cover: /images/cover.jpg # 文章封面图片路径
# thumbnail: /images/thumbnail.jpg # 缩略图路径
description: 文章描述，这里可以是一段简短的介绍
# keywords: 关键字1, 关键字2 # SEO 关键字
top: false # 置顶文章（可选）
comments: true # 是否开启评论
---
# 第一个程序解析

ここからノートが始まる。

## 1 字节序和位操作

### 字节序

小端

```c
uint32_t *p = (uint32_t *)0x20000UL;
*p = 0x12345678;
```

`p`指向地址的存储结果如下👇

| 0x2000C | 0x20008 | 0x20004 | 0x20000 |
| ------- | ------- | ------- | ------- |
| 0x12    | 0x34    | 0x56    | 0x78    |



### 位操作

移位

```c
uint8_t a = 0x3C;	//a = 0b0011_1100
// 左移2位
uint8_t b = a << 2;	//b = 0b1111_0000
// 右移3位
uint8_t c = a >> 3;	//c = 0b0000_0111
```

取反

```c
uint8_t a = 0x3C;	//a = 0b0011_1100
uint8_t b = ~a;		//b = 0b1100_0011
```

位与

```c
uint8_t a = 0x3C;	//a = 0b0011_1100
uint8_t b = 0x68;	//b = 0b0110_1000
uint8_t c = a & b;	//c = 0b0010_1000
```

位或

```c
uint8_t a = 0x3C;	//a = 0b0011_1100
uint8_t b = 0x68;	//b = 0b0110_1000
uint8_t c = a | b;	//c = 0b0111_1100
```

置位和清零

```c
```



## 2 汇编与反汇编

程序处理的步骤

预处理->



## 3 Cの解析



## 4 汇编点灯

下面仿照C指针实现电灯的逻辑，手写汇编代码实现LED灯闪烁的效果。

下面这些是所需要用到的地址的说明

```c
#define PERIPH_BASE 	(0x40000000UL)
#define APB2PERIPH_BASE (PERIPH_BASE + 0x00010000UL)
#define AHBPERIPH_BASE  (PERIPH_BASE + 0x00020000UL)

#define RCC_BASE 	(AHBPERIPH_BASE  + 0x00001000UL)
#define RCC_APB2ENR_offset (0x18)

#define GPIOA_BASE 	(APB2PERIPH_BASE + 0x00000800UL)
#define GPIO_CRL_offset (0x00)
#define GPIO_CRH_offset (0x04)
#define GPIO_ODR_offset (0x0C)
```

在C语言中，实现LED灯闪烁的效果如下。下面是[👉第一节]([ARM]A-1-the-first-program.md)中就实现的LED闪烁程序。

```c
int main(void)
{	
	uint32_t *p;
	// 使能GPIOA
	p = (uint32_t *)(RCC_BASE + RCC_APB2ENR_offset);
	*p |= (1 << 2);
	
	// 设置GPIOA8为输出引脚
	p = (uint32_t *)(GPIOA_BASE + GPIO_CRH_offset);
	*p |= (1 << 0);
	
	p = (uint32_t *)(GPIOA_BASE + GPIO_ODR_offset);
	
	while (1) 
	{
		// 设置GPIOA8输出为 1
		*p |= (uint32_t)(1 << 8);
		delay(100000);
		// 设置GPIOA8输出为 0
		*p &= ~(uint32_t)(1 << 8);
		delay(100000);
	}
}
```

有了简单的汇编基础，我们就可以用汇编取代C实现上面这些逻辑。

先看一下原来使用C语言实现时，`startup.s`里面的代码。

```assembly
                PRESERVE8
                THUMB

                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors
__Vectors       DCD     0 
                DCD     Reset_Handler  ; Reset Handler

                AREA    |.text|, CODE, READONLY
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  main
                ; set stack pointer
                LDR SP, = (0x20000000 + 0xC000)
                BL      main
                ENDP
    
                END
```

在调用C函数`main()`之前，我们在`startup.s`里面设置栈(L:16)，然后再跳转到`main()`函数中执行C程序(L:17)。

现在我们使用汇编实现，也不会用到栈，因此这两行也就不需要了。我们在`Reset_Handler`函数中实现我们需要的程序。

```assembly
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
				
				; do something ...
				
                ENDP
```

### 详细过程

#### 1 使能 GPIOA

```c
// 使能GPIOA
p = (uint32_t *)(RCC_BASE + RCC_APB2ENR_offset);
*p |= (1 << 2);
```

改写成汇编

```assembly
; RCC_BASE = 0x40021000
; RCC_APB2ENR_offset = 0x18
LDR R0, =(0x40021000 + 0x18)

; 取出地址中的内容, 放到R1寄存器中
LDR R1, [R0]

; R1中的值和0x04相或，所得结果再存回R1中
ORR R1, R1, #0x04 == (1 << 2)

; 将R1中的值存储到地址R0存储的地址中去
STR R1, [R0]
```

.

#### 2 配置引脚 PA8

```c
// 设置GPIOA8为输出引脚
p = (uint32_t *)(GPIOA_BASE + GPIO_CRH_offset);
*p |= (1 << 0);
```

改写成汇编

```assembly
; PA8引脚 上拉输出/速度10MHz

; GPIOA_BASE = 0x40010800
; GPIO_CRH_offset = 0x04
LDR R0, =(0x40010800 + 0x04)

; 取出地址中的内容, 放到R1寄存器中
LDR R1, [R0]

; R1中的值和0x01相或，所得结果再存回R1中
ORR R1, R1, #0x01 == (1 << 0)

; 将R1中的值存储到地址R0存储的地址中去
STR R1, [R0]
```

.

#### 3 while循环

```c
p = (uint32_t *)(GPIOA_BASE + GPIO_ODR_offset);

while (1) 
{
    // 设置GPIOA8输出为 1
    *p |= (uint32_t)(1 << 8);
    delay(100000);
    // 设置GPIOA8输出为 0
    *p &= ~(uint32_t)(1 << 8);
    delay(100000);
}
```

在C程序中，使用`while (1) `实现无条件循环。在汇编中，可以这样做。

```assembly
loop
		; do something ...
        B loop
```

最终该段的汇编如下

```assembly
; GPIOA_BASE = 0x40010800
; GPIO_ODR_offset = 0x0C
LDR R2, =(0x40010800 + 0x0C)

loop
        ; *p |= (uint32_t)(1 << 8);
        LDR R1, [R2]
        ORR R1, R1, #0x100 
        STR R1, [R2]
        ; call delay() function
        LDR R0, =(500000)
        BL delay

        ; *p &= ~(uint32_t)(1 << 8);
        LDR R1, [R2]
        ; BIC指令用于将寄存器中的一个或多个特定位 清0。
        BIC R1, R1, #0x100 
        STR R1, [R2]
        ; call delay() function
        LDR R0, =(500000)
        BL delay

        B loop
```

#### 4 delay函数

最后还缺一个`delay`函数

```c
void delay(uint32_t n)
{
	while(n--);
}
```

改写成汇编如下

```assembly
delay	PROC

        SUBS R0, R0, #1
        BNE delay
        BX LR
        NOP

        ENDP
```

.

### 最终结果

最终，使用汇编实现LED闪烁程序的完整代码如下。

```assembly
                PRESERVE8
                THUMB

                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0 
                DCD     Reset_Handler  ; Reset Handler

                AREA    |.text|, CODE, READONLY
					
Reset_Handler   PROC
                EXPORT  Reset_Handler  [WEAK]
			
		;;; enable GPIOA clock
				
				; p = (uint32_t *)(RCC_BASE + RCC_APB2ENR_offset);
				LDR R0, =(0x40021000 + 0x18)
				LDR R1, [R0]
				; *p |= (1 << 2);
				ORR R1, R1, #0x04 
				STR R1, [R0]
				
				
		;;; set PA8 as output/pull-up/speed_10MHz pin
				
				; p = (uint32_t *)(GPIOA_BASE + GPIO_CRH_offset);
				LDR R0, =(0x40010800 + 0x04)
				LDR R1, [R0]
				; *p |= (1 << 0);
				ORR R1, R1, #0x01
				STR R1, [R0]
				
				; p = (uint32_t *)(GPIOA_BASE + GPIO_ODR_offset);
				LDR R2, =(0x40010800 + 0x0C)
				
		;;; set PA8 high/low level in while loop
loop
				; *p |= (uint32_t)(1 << 8);
				LDR R1, [R2]
				ORR R1, R1, #0x100 
				STR R1, [R2]
				; call delay() function
				LDR R0, =(500000)
				BL delay
				
				; *p &= ~(uint32_t)(1 << 8);
				LDR R1, [R2]
				BIC R1, R1, #0x100 
				STR R1, [R2]
				; call delay() function
				LDR R0, =(500000)
				BL delay
				B loop
				
                ENDP

delay			PROC
				SUBS R0, R0, #1
				BNE delay
				BX LR
				NOP
				ENDP

				END
```

完。

