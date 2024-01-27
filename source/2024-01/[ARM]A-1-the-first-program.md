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



### 使用指针实现

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
				LDR SP, = (0x20000000 + 0xC000)
				;调用main函数
				BL main
				ENDP
				
				END
				
```

.

### 使用结构体实现

.

```C
/* Peripheral Memory map */
#define PERIPH_BASE 	(0x40000000UL)
#define APB1PERIPH_BASE (PERIPH_BASE + 0X00000000UL)
#define APB2PERIPH_BASE (PERIPH_BASE + 0x00010000UL)
#define AHBPERIPH_BASE  (PERIPH_BASE + 0x00020000UL)

#define RCC_BASE	(AHBPERIPH_BASE  + 0x00001000UL)
#define GPIOA_BASE	(APB2PERIPH_BASE + 0x00000800UL)
#define GPIOD_BASE  (APB2PERIPH_BASE + 0x00001400UL)
```

#### 结构体

.

```c
#define __IO volatile

typedef struct {
	__IO uint32_t CRL;
	__IO uint32_t CRH;
	__IO uint32_t IDR;
	__IO uint32_t ODR;
	__IO uint32_t BSRR;
	__IO uint32_t BRR;
	__IO uint32_t LCKR;
} GPIO_TypeDef;

#define GPIOA 	((GPIO_TypeDef *)(GPIOA_BASE))
#define GPIOD 	((GPIO_TypeDef *)(GPIOD_BASE))
```

.

|               | + offset 0xc  | + offset 0x8  | + offset 0x4 | 0x40010800 |
| ------------- | ------------- | ------------- | ------------ | ---------- |
| …             | ODR           | IDR           | CRH          | CRL        |
| + offset 0x18 | + offset 0x14 | + offset 0x10 | …            |            |
| LCKR          | BRR           | BSRR          | …            | …          |

.

```c
#define __IO volatile

typedef struct {
	__IO uint32_t CR;
	__IO uint32_t CFGR;
	__IO uint32_t CIR;
	__IO uint32_t APB2RSTR;
	__IO uint32_t APB1RSTR;
	__IO uint32_t AHBENR;
	__IO uint32_t APB2ENR;
	__IO uint32_t APB1ENR;
	__IO uint32_t BDCR;
	__IO uint32_t CSR;
} RCC_TypeDef;

#define RCC 	((RCC_TypeDef *)RCC_BASE)
```

.

|               | + offset 0x10 | + offset 0xc  | + offset 0x8  | + offset 0x4  | 0x40021000 |
| ------------- | ------------- | ------------- | ------------- | ------------- | ---------- |
| …             | APB1RSTR      | APB2RSTR      | CIR           | CFGR          | CR         |
| + offset 0x24 | + offset 0x20 | + offset 0x1c | + offset 0x18 | + offset 0x14 |            |
| CSR           | BDCR          | APB1ENR       | APB2ENR       | AHBENR        | …          |

.

#### 宏

```c
#define GPIO_Pin_2  (1 << 2)
#define GPIO_Pin_8  (1 << 8)
#define GPIO_Pin_15 (1 << 15)

#define GPIO_PIN_SET (1)
#define GPIO_PIN_RST (0)

#define __RCC_GPIOA_CLK_ENABLE()  	RCC->APB2ENR |= (1 << 2)
#define __RCC_GPIOD_CLK_ENABLE()  	RCC->APB2ENR |= (1 << 5)

#define GPIO_WritePin(GPIOx, GPIO_Pin, PinState) \
	do{ \
		if (PinState) \
			GPIOx->BSRR = (uint32_t)GPIO_Pin; \
		else \
			GPIOx->BRR  = (uint32_t)GPIO_Pin; \
	}while(0)

#define GPIO_ReadPin(GPIOx, GPIO_Pin) (GPIOx->IDR & GPIO_Pin)
```

.

#### 最终实现

```c
void GPIO_Init()
{
	// 使能GPIOA GPIOD
	__RCC_GPIOA_CLK_ENABLE();
	__RCC_GPIOD_CLK_ENABLE();
	// 设置GPIOA8  通用输出/上拉/速度10MHz
	GPIOA->CRH &= 0xFFFFFFF0;
	GPIOA->CRH |= 0x00000001;
	// 设置GPIOD2  通用输出/上拉/速度10MHz
	GPIOD->CRL &= 0xFFFFF0FF;
	GPIOD->CRL |= 0x00000100;
	// 设置GPIOA15 浮动输入
	GPIOA->CRH &= 0x0FFFFFFF;
	GPIOA->CRH |= 0x80000000;
}

int main(void)
{
	GPIO_Init();
	
	while (1) {

		if(GPIO_ReadPin(GPIOA, GPIO_Pin_15) == 0) {
			// 设置PA8输出为 0; 设置PD2输出为 1
			GPIO_WritePin(GPIOA, GPIO_Pin_8, GPIO_PIN_RST);
			GPIO_WritePin(GPIOD, GPIO_Pin_2, GPIO_PIN_SET);
		}else {
			// 设置PA8输出为 1; 设置PD2输出为 0
			GPIO_WritePin(GPIOA, GPIO_Pin_8, GPIO_PIN_SET);
			GPIO_WritePin(GPIOD, GPIO_Pin_2, GPIO_PIN_RST);
		}
	}
}

```





