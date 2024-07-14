---
title: FreeRTOS下的uart封装
date: 2024-04-03
author: 3oR
categories:
  - FreeRTOS
tags:
  - FreeRTOS
# cover: /images/cover.jpg # 文章封面图片路径
# thumbnail: /images/thumbnail.jpg # 缩略图路径
description: 文章描述，这里可以是一段简短的介绍
# keywords: 关键字1, 关键字2 # SEO 关键字
top: false # 置顶文章（可选）
comments: true # 是否开启评论
---
# FreeRTOS下的uart封装

## 最初级的使用

​		这段代码是一个简单的C语言程序，主要功能是通过UART（通用异步收发传输）接口接收字符，对接收到的字符进行加一操作，然后再通过UART接口发送回去。

```c
int main()
{
    // 初始化过程，此处略去
    
    char c;
    
	while (1)
	{
		while(HAL_UART_Receive(&huart1, &c, 1, 100) != HAL_OK);
		c += 1;
		HAL_UART_Transmit(&huart1, &c, 1, 100);
		HAL_Delay(10);
	}
}
```

.



## 增加FreeRTOS

下面这段代码创建了一个名为`vTask1`的FreeRTOS任务，在该任务中实现了一个字符接收、加一、发送的循环过程。然后通过`vTaskStartScheduler()`函数启动了FreeRTOS调度器。

```c

__NO_RETURN void vTask1(void *argument)
{
	(void)argument;
    
    char c;
    
    while(1)
    {
		while(HAL_UART_Receive(&huart1, &c, 1, 100) != HAL_OK);
		c += 1;
		HAL_UART_Transmit(&huart1, &c, 1, 100);
        
        vTaskDelay(10);
    }
}

void user_main(void)
{
	// 初始化过程，此处略去
    
    xTaskCreate(vTask1, "vTask1", 128, NULL, 1, NULL);

    vTaskStartScheduler();
    
	Error_Handler();
}
```

​		我们使用的函数`HAL_UART_Receive()`和`HAL_UART_Transmit()`以及句柄`&huart1`都是依赖于STM32Cube HAL库的实现。当底层硬件平台发生变化时，我们不得不修改代码以适应新的硬件接口。为了解决这个问题，我们希望引入一个统一的接口层，这样当更换底层硬件平台时，我们只需要修改这个接口层的实现，而无需修改应用层的代码。这样可以大大简化代码维护和移植的工作。

## FreeRTOS 中断

```c
/************UART*************/

struct UART_Data
{
	void *uart_handle;
	SemaphoreHandle_t *xTxSem;
	QueueHandle_t *xRxQueue;
	uint8_t rxdata;
};


struct UART_Device 
{
	const char *name;
	struct UART_Data *prvdata;
	
	int (*Init)(	struct UART_Device *pDev, 
					int baud, 
					char parity, 
					int stop);
	
    int (*Send)(	struct UART_Device *pDev, 
					uint8_t *data, 
					int len, 
					int timeout_ms);
	
	int (*Recv)(	struct UART_Device *pDev, 
					uint8_t *data, 
					int len, 
					int timeout_ms);

};


static int stm32_uart_init(struct UART_Device *pDev, int baud, char parity, int stop)
{
	pDev->prvdata->xTxSem = xSemaphoreCreateBinary();
	pDev->prvdata->xRxQueue = xQueueCreate(100, 1);
	// 启动第一次数据接收
	HAL_UART_Receive_IT(pDev->prvdata->uart_handle, &pDev->prvdata->rxdata, 1);
}

static int stm32_uart_send(struct UART_Device *pDev, uint8_t *data, int len, int timeout_ms)
{
	// 触发中断发送数据
	HAL_UART_Transmit_IT(pDev->prvdata, data, len);
	// 等待信号量
	if(pdTRUE != xSemaphoreTake(pDev->prvdata->xTxSem, timeout_ms)){
		return -1;
	}
	return 0;
}

static int stm32_uart_recv(struct UART_Device *pDev, uint8_t *data, int len, int timeout_ms)
{
	if (pdPASS != xQueueReceive(pDev->prvdata->xRxQueue, data, timeout_ms)) {
		return -1;
	}
	return 0;
}

/*****************************/

static struct UART_Data stm32_uart1_data = {0};

static struct UART_Device stm32_uart1 = {
	.name = "stm32_uart1",
	.prvdata = &stm32_uart1_data,
	.Init = stm32_uart_init,
	.Send = stm32_uart_send,
	.Recv = stm32_uart_recv
};

/**
 * @brief 发送完成回调函数
*/
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart == &huart1)
	{
		// 释放信号量
		xSemaphoreGiveFromISR(stm32_uart1.prvdata->xTxSem, NULL);
	}
}

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	struct UART_Data *uart_data;

	if (huart == &huart1)
	{
		uart_data = stm32_uart1.prvdata;

		// 写入队列
		xQueueSendFromISR(uart_data->xRxQueue, &uart_data->rxdata, 1);
		// 再次启动数据接收
		HAL_UART_Receive_IT(&huart1, uart_data->rxdata, 1);
	}
}



```

.向外部暴露的接口

```c

struct UART_Device *stm32_uart_devs[] = {&stm32_uart1, };

struct UART_Device *GetUARTDevie(const char *name)
{
	int stm32_uart_devs_len = sizeof(stm32_uart_devs)/sizeof(stm32_uart_devs[0]);

	for (int i = 0; i < stm32_uart_devs_len; i++)
	{
		if(strcmp(stm32_uart_devs[i]->name, name) == 0)
			return stm32_uart_devs[i];
	}

	return NULL;
}
```

.主程序

```c
void vTask1(void *argument);

void user_main(void)
{

    xTaskCreate(vTask1, "vTask1", 128, NULL, 1, NULL);

    vTaskStartScheduler();
    
	Error_Handler();
}

void vTask1(void *argument)
{
	(void)argument;

	struct UART_Device *pUARTDev = GetUARTDevie("stm32_uart1");

	char c;
	
    while(1)
	{
		
		pUARTDev->Send(pUARTDev, "test ok\n", 8, 100);

		while (0 != pUARTDev->Recv(pUARTDev, &c, 1, 100));
		c += 1;

		pUARTDev->Send(pUARTDev, &c, 1, 1);

    }
}
```



.

## FreeRTOS DMA

.
