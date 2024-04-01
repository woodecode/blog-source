
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

## vListInitialise函数和反汇编代码阅读

.

```c
 typedef struct xLIST
 {
     volatile UBaseType_t uxNumberOfItems;
     ListItem_t * configLIST_VOLATILE pxIndex;
     MiniListItem_t xListEnd;
 } List_t;
 
 struct xLIST_ITEM
 {
     configLIST_VOLATILE TickType_t xItemValue;
     struct xLIST_ITEM * configLIST_VOLATILE pxNext;
     struct xLIST_ITEM * configLIST_VOLATILE pxPrevious;
     void * pvOwner;
     struct xLIST * configLIST_VOLATILE pxContainer;
 };
 
 typedef struct xLIST_ITEM ListItem_t;
 typedef struct xLIST_ITEM MiniListItem_t;
 
 
 void vListInitialise( List_t * const pxList )
 {
     /* The list structure contains a list item which is used to mark the
      * end of the list.  To initialise the list the list end is inserted
      * as the only list entry. */
     pxList->pxIndex = ( ListItem_t * ) &( pxList->xListEnd ); /*lint !e826 !e740 !e9087 The mini list structure is used as the list end to save RAM.  This is checked and valid. */
 
     /* The list end value is the highest possible value in the list to
      * ensure it remains at the end of the list. */
     pxList->xListEnd.xItemValue = portMAX_DELAY;
 
     /* The list end next and previous pointers point to itself so we know
      * when the list is empty. */
     pxList->xListEnd.pxNext = ( ListItem_t * ) &( pxList->xListEnd );
     pxList->xListEnd.pxPrevious = ( ListItem_t * ) &( pxList->xListEnd );
 
     pxList->uxNumberOfItems = ( UBaseType_t ) 0U;
 }
```

.

| 字节       | [24:21]     | [23:20] | [19:16]    | [15:12] | [11:8]     | [7:4]   | [3:0]           |
| ---------- | ----------- | ------- | ---------- | ------- | ---------- | ------- | --------------- |
| 基地址偏移 |             |         |            | + C     | + 8        | + 4     | + 0             |
| 变量名     | pxContainer | pvOwner | pxPrevious | pxNext  | xItemValue | pxIndex | uxNumberOfItems |

.

```assembly
 ; 无优化 -O0
 ; void __fastcall vListInitialise(List_t *const pxList)
     SUB      sp,sp,#4           ;栈减1，存放局部变量temp1
     STR      r0,[sp,#0]         ;将R0值存放到temp1(即 pxList)
     ; 
     LDR      r1,[sp,#0]         ;令R1 = temp1
     ADD      r0,r1,#8           ;令R0 = R1 + 8
     STR      r0,[r1,#4]         ;将R0值保存到(R1 + 4)的地址中
     LDR      r1,[sp,#0]         ;
     
     ; 
     ; pxList->xListEnd.xItemValue = portMAX_DELAY;
     MOV      r0,#0xffffffff
     STR      r0,[r1,#8]
     
     LDR      r1,[sp,#0]
     ADD      r0,r1,#8
     STR      r0,[r1,#0xc]
     LDR      r1,[sp,#0]
     ADD      r0,r1,#8
     STR      r0,[r1,#0x10]
     LDR      r1,[sp,#0]
     ; pxList->uxNumberOfItems = ( UBaseType_t ) 0U;
     MOVS     r0,#0
     STR      r0,[r1,#0]
     ; 
     ADD      sp,sp,#4
     BX       lr
     MOVS     r0,r0
     
 ; 最大优化 -Ofast
 ; void __fastcall vListInitialise(List_t *const pxList)
 
     MOV      r1,#0xffffffff     ;令R0 = 0xffffffff
     MOV      r2,r0              ;将R2赋值pxList地址
     
     STR      r1,[r2,#8]!        ;将R1的值保存到(R2 + 8)的地址中
     MOVS     r1,#0              ;令R1为0
     STR      r2,[r0,#4]         ;将R2保存到(R0+4)地址中
     
     STRD     r2,r2,[r0,#0xc]    ;将R2分别保存到地址(R0 + 0xC)和(R0 + 0xC + 4)中
     STR      r1,[r0,#0]         ;将R1保存到(R0)地址中
     
     BX       lr                 ;返回
     MOVS     r0,r0              ;
```

.

