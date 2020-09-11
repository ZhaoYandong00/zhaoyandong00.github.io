---
title: 动态内存创建多任务FREERTOS篇4
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS动态内存创建多任务
---
# 任务常用函数
- 创建任务

```c
BaseType_t xTaskCreate(TaskFunction_t pxTaskCode,
                       const char * const pcName,		/*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                       const configSTACK_DEPTH_TYPE usStackDepth,
                       void * const pvParameters,
                       UBaseType_t uxPriority,
                       TaskHandle_t * const pxCreatedTask )
```
- 开启调度器 `void vTaskStartScheduler(void)`
- 任务挂起`void vTaskSuspend(TaskHandle_t xTaskToSuspend)`
- 任务全部挂起`void vTaskSuspendAll(void)`
- 任务恢复`void vTaskResume(TaskHandle_t xTaskToResume)`
- 任务从中断中恢复`xTaskResumeFromISR()`
- 任务全部恢复`BaseType_t xTaskResumeAll(void)`
- 任务删除`void vTaskDelete(TaskHandle_t xTaskToDelete)`
- 相对延时`void vTaskDelay(const TickType_t xTicksToDelay)`
- 绝对延时`void vTaskDelayUntil(TickType_t * const pxPreviousWakeTime, const TickType_t xTimeIncrement)`

# 创建多任务
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"
/* 创建任务句柄 */
static TaskHandle_t AppTaskCreate_Handle;
/* LED任务句柄 */
static TaskHandle_t LED1_Task_Handle;
static TaskHandle_t LED2_Task_Handle;


static void AppTaskCreate(void);/* 用于创建任务 */
static void BSP_Init(void);/* 用于初始化板载相关资源 */
static void LED1_Task(void* pvParameters);/* LED_Task任务实现 */
static void LED2_Task(void* pvParameters);/* LED_Task任务实现 */

int main(void)
{
    BSP_Init();
    printf("FreeRTOS-DYNAMIC!\r\n");
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
/**********************************************************************
  * @ 函数名  ： LED_Task
  * @ 功能说明： LED_Task任务主体
  * @ 参数    ：
  * @ 返回值  ： 无
  ********************************************************************/
static void LED1_Task(void* parameter)
{
    while (1)
    {
        LED1(ON);
        vTaskDelay(500);   /* 延时500个tick */
        printf("LED_Task Running,LED1_ON\r\n");

        LED1(OFF);
        vTaskDelay(500);   /* 延时500个tick */
        printf("LED_Task Running,LED1_OFF\r\n");
    }
}
/**********************************************************************
  * @ 函数名  ： LED_Task
  * @ 功能说明： LED_Task任务主体
  * @ 参数    ：
  * @ 返回值  ： 无
  ********************************************************************/
static void LED2_Task(void* parameter)
{
    while (1)
    {
        LED2(ON);
        vTaskDelay(1000);   /* 延时1000个tick */
        printf("LED_Task Running,LED2_ON\r\n");

        LED2(OFF);
        vTaskDelay(1000);   /* 延时500个tick */
        printf("LED_Task Running,LED2_OFF\r\n");
    }
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
    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)LED1_Task,		//任务函数
                          (const char* 	)"LED1_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&LED1_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("LED_Task SUCCESS!\n");
    else
        printf("LED_Task FAIL!\n");
        /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)LED2_Task,		//任务函数
                          (const char* 	)"LED2_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&LED2_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("LED_Task SUCCESS!\n");
    else
        printf("LED_Task FAIL!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
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
}

```
# 调试
- 编译下载到开发板
- 可以看到LED1和LED2以不同频率闪烁