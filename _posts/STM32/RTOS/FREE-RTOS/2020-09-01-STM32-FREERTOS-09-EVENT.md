---
title: 事件FREERTOS篇9
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS事件
---
# 事件解析
## 事件结构体分析

```c
typedef struct EventGroupDef_t
{
	EventBits_t uxEventBits;
	List_t xTasksWaitingForBits;		/*< List of tasks waiting for a bit to be set. */

	#if( configUSE_TRACE_FACILITY == 1 )
		UBaseType_t uxEventGroupNumber;
	#endif

	#if( ( configSUPPORT_STATIC_ALLOCATION == 1 ) && ( configSUPPORT_DYNAMIC_ALLOCATION == 1 ) )
		uint8_t ucStaticallyAllocated; /*< Set to pdTRUE if the event group is statically allocated to ensure no attempt is made to free the memory. */
	#endif
} EventGroup_t;
```
- 事件标志组`uxEventBits`
- 记录等待事件的任务链表`xTasksWaitingForBits`

## 常用函数
- 创建事件`EventGroupHandle_t xEventGroupCreate(void)`
- 删除事件`void vEventGroupDelete(EventGroupHandle_t xEventGroup)`
- 事件组置位`EventBits_t xEventGroupSetBits(EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToSet)`
- 中断中事件组置位`BaseType_t xEventGroupSetBitsFromISR(EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToSet,BaseType_t *pxHigherPriorityTaskWoken)`
- 等待事件函数`EventBits_t xEventGroupWaitBits(const EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToWaitFor,const BaseType_t xClearOnExit,const BaseType_t xWaitForAllBits,TickType_t xTicksToWait)`
- 清除事件组指定的位`EventBits_t xEventGroupClearBits(EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToClear)`
- 中断中清除事件组指定的位`BaseType_t xEventGroupClearBitsFromISR(EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToClear)`

# 添加库函数

- 打开 `Manage Run-Time Environment`
- 选择 `RTOS->Event Groups`

# 事件
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "event_groups.h"
#include "my_gpio.h"
#include "my_usart.h"


/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */
static TaskHandle_t LED_Task_Handle = NULL;/* LED_Task任务句柄 */
static TaskHandle_t KEY_Task_Handle = NULL;/* KEY_Task任务句柄 */
/* 消息队列句柄 */
static EventGroupHandle_t Event_Handle =NULL;

#define KEY1_EVENT  (0x01 << 0)//设置事件掩码的位0
#define KEY2_EVENT  (0x01 << 1)//设置事件掩码的位1

static void AppTaskCreate(void);/* 用于创建任务 */

static void LED_Task(void* pvParameters);/* LED_Task 任务实现 */
static void KEY_Task(void* pvParameters);/* KEY_Task 任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-MUTEX!\r\n");
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    /* 创建 AppTaskCreate 任务 */
    xReturn = xTaskCreate((TaskFunction_t	)AppTaskCreate,		//任务函数
                          (const char* 	)"AppTaskCreate",		//任务名称
                          (uint32_t 		)128,	//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)1, 	//任务优先级
                          (TaskHandle_t*  )&AppTaskCreate_Handle);	//任务控制块

    if (pdPASS == xReturn)/* 创建成功 */
        vTaskStartScheduler();   /* 启动任务，开启调度 */
    else
        return -1;
    while(1);   /* 正常不会执行到这里 */
}

/**
 * @brief  为了方便管理，所有的任务创建函数都放在这个函数里面
 * @param  无
 * @retval 无
 */
static void AppTaskCreate(void)
{
    taskENTER_CRITICAL();           //进入临界区
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */

    Event_Handle = xEventGroupCreate();
    if(NULL != Event_Handle)
        printf("Event SUCCESS!\n");

    xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                          (const char*    )"LED_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("LED_Task SUCCESS!\n");

    xReturn = xTaskCreate((TaskFunction_t )KEY_Task, /* 任务入口函数 */
                          (const char*    )"KEY_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&KEY_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("KEY_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LED_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void LED_Task(void* parameter)
{
    EventBits_t r_event;
    while (1)
    {
        r_event = xEventGroupWaitBits(Event_Handle,  /* 事件对象句柄 */
                                      KEY1_EVENT|KEY2_EVENT,/* 接收线程感兴趣的事件 */
                                      pdTRUE,   /* 退出时清除事件位 */
                                      pdTRUE,   /* 满足感兴趣的所有事件 */
                                      portMAX_DELAY);/* 指定超时事件,一直等 */

        if((r_event & (KEY1_EVENT|KEY2_EVENT)) == (KEY1_EVENT|KEY2_EVENT))
        {
            /* 如果接收完成并且正确 */
            printf ( "KEY1 KEY2 ALL\n");
            LED1_TOGGLE;       //LED1	反转
        }
        else
            printf ( "ERR\n");
    }
}
/**
 * @brief  KEY_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void KEY_Task(void* parameter)
{	 
    /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
		{
      printf ( "KEY1 press\n" );
			/* 触发一个事件1 */
			xEventGroupSetBits(Event_Handle,KEY1_EVENT);  					
		}
    
		if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
		{
      printf ( "KEY2  press\n" );	
			/* 触发一个事件2 */
			xEventGroupSetBits(Event_Handle,KEY2_EVENT); 				
		}
		vTaskDelay(20);     //每20ms扫描一次		
  }
}
/**
 * @brief  板级外设初始化，所有板子上的初始化均可放在这个函数里面
 * @param  无
 * @retval 无
 */
static void BSP_Init(void)
{
    /*
     * STM32中断优先级分组为4，即4bit都用来表示抢占优先级，范围为：0~15
     * 优先级分组只需要分组一次即可，以后如果有其他的任务需要用到中断，
     * 都统一用这个优先级分组，千万不要再分组，切忌。
     */
    NVIC_PriorityGroupConfig( NVIC_PriorityGroup_4 );

    /* LED 初始化 */
    LED_GPIO_Config();

    /* 串口初始化	*/
    USART_Config();
    /* 按键初始化 */
    Key_GPIO_Config();
}

```
# 调试
- 编译下载到开发板
- 按下KEY1 按键发送事件1,按下KEY2,按键发送事件2
- 我们按下KEY1与KEY2试试,在串口调试助手中可以看到运行结果,并且当事件1与事件2都发生的时候,开发板的LED会进行翻转