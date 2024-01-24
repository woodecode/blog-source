```json
{
    "front-matter": {
        "title": "第一个程序",
        "date": "2024-01-17",
        "author": "3oR",
        "tags": ["ARM", "Embedded"],
        "categories": ["ARM"],
        "description": "文章描述",
        "cover": "封面图片链接",
        "featured": false, 
        "draft": true 
	}
}
```



# 第一个程序-电亮LED灯(以STM32F103为例)

## GPIO操作逻辑

本质：读/写寄存器的数据

GPIO以一组为单位。当操作单个引脚时，修改寄存器`GPIO_REG`的数据，需要三步。

1 读取寄存器`GPIO_REG`数据

2 修改读取的数据

3 写入修改的数据



使用额外的寄存器表示对寄存器特定位进行SET/RST操作，可以简化上面略显繁琐的步骤。使用`SET_REG` 和 `RST_REG`

将`SET_REG`中的某些位设置为1，即为对`GPIO_REG`中对应的位 置为1；将`RST_REG`中的某些位设置为1，即为对`GPIO_REG`中对应的位 置为0；对`GPIO_REG`实际数据的操作由硬件电路实现，这样就不必关心`GPIO_REG`中不需要操作的位的数据了。



## GPIO操作的步骤实现

[STM32F103 Datasheet Link](https://www.st.com/resource/en/datasheet/stm32f103ze.pdf)



1 时钟RCC，使能GPIOx



2 设置GPIOx



## GPIO操作的具体实现



```c
/*

*/
#include <stdint.h>

#define PERIPH_BASE 	(0x40000000UL)
#define AHBPERIPH_BASE  (PERIPH_BASE + 0x00020000UL)
#define APB2PERIPH_BASE	(PERIPH_BASE + 0x00010000UL)

#define RCC_BASE (AHBPERIPH_BASE + 0x00001000UL)
#define RCC_APB2ENR_offset (0x18)

#define GPIOB_BASE (APB2PERIPH_BASE + 0x00000C00UL)
#define GPIO_CRL_offset (0x00)
#define GPIO_CRH_offset (0x04)
#define GPIO_ODR_offset (0x0C)

void delay(uint32_t n)
{
	while(n--);
}

int main()
{
    uint32_t *p;
    // 使能GPIOB
    p = (uint32_t *)(RCC_BASE + RCC_APB2ENR_offset);
    *p |= (1 << 3);
    
    // 设置GPIOB0为输出引脚
    p = (uint32_t *)(GPIOB_BASE + GPIO_CRH_offset);
    *p |= (1 << 0);
    
    p = (uint32_t *)(GPIOB_BASE + GPIO_ODR_offset);
    
    while(1) {
        // 设置GPIOB0输出为 1
    	*p |=  (1 << 0);
        delay(100000);
        // 设置GPIOB0输出为 0
        *p &= !(1 << 0);
        delay(100000);
    }
    return 0;
}

void delay(uint32_t n)
{
    while(n--);
}
```

启动代码`startup.s`。

```assembly
				PRESERVES
				THUMB
	
				AREA RESET, DATA, READONLY
				
				EXPORT __Vectors
__Vectors	 	DCD	0
				DCD Reset_Handler



				AREA |.text|, CODE, READONLY
				
Reset_Handler	PROC
				EXPORT Reset_Handler		[WEAK]
				IMPORT main
				
				;设置栈(从最高地址向下增长)
				LDR SP, = (0x20000000 + 0x10000)
				;调用main函数
				BL main
				ENDP
				
				END
				
```



