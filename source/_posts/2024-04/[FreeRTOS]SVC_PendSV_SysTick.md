
```json
{
    "front-matter": {
        "title": "SVC，PendSV和SysTick",
        "date": "2024-04-01",
        "author": "我是谁",
        "tags": ["FreeRTOS", "Example1"],
        "categories": ["FreeRTOS", "Cate1"],
        "description": "文章描述",
        "cover": "封面图片链接",
        "featured": false, 
        "draft": true 
	}
}
```

# SVC，PendSV和SysTick

## SVC_Handler

.

```c
typedef struct tskTaskControlBlock 
{
    volatile StackType_t * pxTopOfStack;
    ListItem_t xStateListItem;
    ListItem_t xEventListItem;
    UBaseType_t uxPriority;
    StackType_t * pxStack; 
    char pcTaskName[ configMAX_TASK_NAME_LEN ]; 
} tskTCB;
```

.

> **Stack Pointer **
>
> The Stack Pointer (SP) is register R13. In Thread mode, bit[1] of the CONTROL register  indicates the stack pointer to use: 
>
> - 0 = Main Stack Pointer (MSP). This is the reset value. 
>
> - 1 = Process Stack Pointer (PSP). 
>
> On reset, the processor loads the MSP with the value from address 0x00000000. 
>
> **Link Register** 
>
> The Link Register (LR) is register R14. It stores the return information for subroutines, function  calls, and exceptions. 
>
> On reset, the processor sets the LR value to 0xFFFFFFFF.

.

```assembly
SVC_Handler
        ; 加载pxCurrentTCB地址到R3
        LDR      r3,[pc,#28] ; LDR R3, =pxCurrentTCB
        
        ; 加载当前任务的堆栈地址到R0
        LDR      r1,[r3,#0]
        LDR      r0,[r1,#0]
        
        ; 从R0指向的堆栈中恢复R4到R11的值，并更新堆栈指针R0
        LDM      r0!,{r4-r11}
        
        ; 将当前线程的堆栈指针更改为R0中的值。
        MSR      PSP,r0
        
        ; 指令同步屏障，确保之前的所有指令完成执行
        ISB
        
        ; BASEPRI 寄存器中的值存储的是最低的允许运行中断号
        ; MSR 指令将通用寄存器的内容移动到指定的特殊寄存器中
        ; 清除基础优先级寄存器BASEPRI，允许所有优先级的中断
        MOV      r0,#0
        MSR      BASEPRI,r0
        
        ; 通过设置LR[3:0]的值为0X0D，设置堆栈指针为PSP
        ORR      lr,lr,#0xd
        
        BX       lr
```

.

```assembly
ORR lr,lr,#0xd
```

| 原始LR尾部 | 执行指令后 |
| ---------- | ---------- |
| 0b0000     | 0b1101     |
| 0b0100     | 0b1101     |
| 0b1000     | 0b1101     |
| 0b1100     | 0b1101     |

.

## SysTick_Handler

```c
void xPortSysTickHandler( void )
{
    portDISABLE_INTERRUPTS();
    {
        /* Increment the RTOS tick. */
        if( xTaskIncrementTick() != pdFALSE )
        {
            /* A context switch is required.*/
            portNVIC_INT_CTRL_REG = portNVIC_PENDSVSET_BIT;
        }
    }
    portENABLE_INTERRUPTS();
}
```

.

```assembly
SysTick_Handler
		
        PUSH     {r7,lr}
        
        ; portDISABLE_INTERRUPTS();
        MOV      r0,#0x50
        MSR      BASEPRI,r0
        ISB      
        DSB      
        
        ; if( xTaskIncrementTick() != pdFALSE )
        BL       xTaskIncrementTick ; 0x80039ac
        CBZ      r0,loc_800207a
        MOV      r0,#0xed04
        MOVT     r0,#0xe000
        MOV      r1,#0x10000000
        STR      r1,[r0,#0]
loc_800207a:
        ; portENABLE_INTERRUPTS();
        MOVS     r0,#0
        MSR      BASEPRI,r0
        
        POP      {r7,pc}
        MOVS     r0,r0

```

.

## PendSV_Handler

.

.

```assembly
PendSV_Handler
		; MSR 指令将特殊寄存器的内容移动到指定的通用寄存器中
        MRS      r0,PSP
        ; 指令同步屏障，确保之前的所有指令完成执行
        ISB      
        
        LDR      r3,[pc,#52] ; [0x8002020] = 0x2000024c
        LDR      r2,[r3,#0]
        
        STMDB    r0!,{r4-r11}
        STR      r0,[r2,#0]
        
        PUSH.W   {r3,lr}
        
        MOV      r0,#0x50
        MSR      BASEPRI,r0
        
        BL       vTaskSwitchContext
        MOV      r0,#0
        MSR      BASEPRI,r0
        POP      {r3,lr}
        
        LDR      r1,[r3,#0]
        LDR      r0,[r1,#0]
        LDM      r0!,{r4-r11}
        MSR      PSP,r0
        
        ; 指令同步屏障，确保之前的所有指令完成执行
        ISB      
        BX       lr

```

.

```c
#define listGET_OWNER_OF_NEXT_ENTRY( pxTCB, pxList ) \
{  \
    List_t * const pxConstList = ( pxList ); \
    ( pxConstList )->pxIndex = ( pxConstList )->pxIndex->pxNext; \
    if( ( void * ) ( pxConstList )->pxIndex == \
        ( void * ) &( ( pxConstList )->xListEnd ) ) \
    { \
        ( pxConstList )->pxIndex = \
            ( pxConstList )->pxIndex->pxNext; \
    } \
    ( pxTCB ) = ( pxConstList )->pxIndex->pvOwner; \
}

#define taskSELECT_HIGHEST_PRIORITY_TASK() \
{ \
    UBaseType_t uxTopPriority = uxTopReadyPriority; \
    while( listLIST_IS_EMPTY( &( pxReadyTasksLists[ uxTopPriority ] ) ) ) \
    { \
        configASSERT( uxTopPriority ); \
        --uxTopPriority; \
    } \
    listGET_OWNER_OF_NEXT_ENTRY( pxCurrentTCB, &( pxReadyTasksLists[ uxTopPriority ] ) ); \
    uxTopReadyPriority = uxTopPriority;                                                   \
} /* taskSELECT_HIGHEST_PRIORITY_TASK */

void vTaskSwitchContext( void )
{
    if( uxSchedulerSuspended != ( UBaseType_t ) pdFALSE )
    {
        /* The scheduler is currently suspended - do not allow a context
         * switch. */
        xYieldPending = pdTRUE;
    }
    else
    {
        xYieldPending = pdFALSE;
        /* Select a new task to run using either the generic C or port
         * optimised asm code. */
        taskSELECT_HIGHEST_PRIORITY_TASK();
    }
}
```

.
