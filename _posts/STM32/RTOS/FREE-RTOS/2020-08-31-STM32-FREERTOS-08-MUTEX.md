---
title: 互斥量FREERTOS篇8
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS互斥
---
# 常用函数
- 创建互斥量`SemaphoreHandle_t xSemaphoreCreateMutex(void)`
- 创建递归互斥量`SemaphoreHandle_t xSemaphoreCreateRecursiveMutex(void)`
- 递归互斥量获取`BaseType_t xSemaphoreTakeRecursive(SemaphoreHandle_t xMutex,TickType_t xBlockTime)`
- 递归互斥量释放`BaseType_t xSemaphoreGiveRecursive(SemaphoreHandle_t xMutex)`

# 模拟优先级翻转
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "semphr.h"
#include "my_gpio.h"
#include "my_usart.h"


/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */
static TaskHandle_t LowPriority_Task_Handle = NULL;/* LowPriority_Task任务句柄 */
static TaskHandle_t MidPriority_Task_Handle = NULL;/* MidPriority_Task任务句柄 */
static TaskHandle_t HighPriority_Task_Handle = NULL;/* HighPriority_Task任务句柄 */
/* 消息队列句柄 */
SemaphoreHandle_t BinarySem_Handle =NULL;


static void AppTaskCreate(void);/* 用于创建任务 */

static void LowPriority_Task(void* pvParameters);/* LowPriority_Task任务实现 */
static void MidPriority_Task(void* pvParameters);/* MidPriority_Task任务实现 */
static void HighPriority_Task(void* pvParameters);/* MidPriority_Task任务实现 */

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

    BinarySem_Handle = xSemaphoreCreateBinary();
    if(NULL != BinarySem_Handle)
        printf("BinarySem SUCCESS!\n");
    xReturn = xSemaphoreGive( BinarySem_Handle );
    /* 创建LowPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)LowPriority_Task,		//任务函数
                          (const char* 	)"LowPriority_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&LowPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("LowPriority_Task SUCCESS!\n");

    /* 创建MidPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)MidPriority_Task,		//任务函数
                          (const char* 	)"Send_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&MidPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("MidPriority_Task SUCCESS!\n");

    /* 创建HighPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)HighPriority_Task,		//任务函数
                          (const char* 	)"HighPriority_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)4, 				//任务优先级
                          (TaskHandle_t* )&HighPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("HighPriority_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LowPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void LowPriority_Task(void* parameter)
{
    static uint32_t i;
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        printf("LowPriority_Task take\r\n");
        xReturn = xSemaphoreTake(BinarySem_Handle,/* 二值信号量句柄 */
                                 portMAX_DELAY); /* 等待时间 */
        if( xReturn == pdTRUE )
            printf("LowPriority_Task Runing\n\n");
        for(i=0; i<2000000; i++) //模拟低优先级任务占用信号量
        {
            taskYIELD();//发起任务调度
        }
        printf("LowPriority_Task give\r\n");
        xReturn = xSemaphoreGive(BinarySem_Handle);
        LED1_TOGGLE;

        vTaskDelay(500);
    }
}
/**
 * @brief  MidPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void MidPriority_Task(void* parameter)
{
    while (1)
    {
        printf("MidPriority_Task Runing\n");
        vTaskDelay(500);
    }
}
/**
 * @brief  HighPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void HighPriority_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        printf("HighPriority_Task take\n");
        //获取二值信号量 xSemaphore,没获取到则一直等待
        xReturn = xSemaphoreTake(BinarySem_Handle,/* 二值信号量句柄 */
                                 portMAX_DELAY); /* 等待时间 */
        if(pdTRUE == xReturn)
            printf("HighPriority_Task Runing\n");
        LED1_TOGGLE;
        xReturn = xSemaphoreGive( BinarySem_Handle );//释放二值信号量

        vTaskDelay(500);
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
## 调试
- 编译下载到开发板
- 可以看到高优先级任务在等待低优先级任务运行完毕才能得到信号量继续运行

# 互斥量
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "semphr.h"
#include "my_gpio.h"
#include "my_usart.h"


/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */
static TaskHandle_t LowPriority_Task_Handle = NULL;/* LowPriority_Task任务句柄 */
static TaskHandle_t MidPriority_Task_Handle = NULL;/* MidPriority_Task任务句柄 */
static TaskHandle_t HighPriority_Task_Handle = NULL;/* HighPriority_Task任务句柄 */
/* 消息队列句柄 */
SemaphoreHandle_t MuxSem_Handle =NULL;


static void AppTaskCreate(void);/* 用于创建任务 */

static void LowPriority_Task(void* pvParameters);/* LowPriority_Task任务实现 */
static void MidPriority_Task(void* pvParameters);/* MidPriority_Task任务实现 */
static void HighPriority_Task(void* pvParameters);/* MidPriority_Task任务实现 */

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

    MuxSem_Handle = xSemaphoreCreateMutex();
    if(NULL != MuxSem_Handle)
        printf("MuxSem SUCCESS!\n");
    xReturn = xSemaphoreGive( MuxSem_Handle );
    /* 创建LowPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)LowPriority_Task,		//任务函数
                          (const char* 	)"LowPriority_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&LowPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("LowPriority_Task SUCCESS!\n");

    /* 创建MidPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)MidPriority_Task,		//任务函数
                          (const char* 	)"Send_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&MidPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("MidPriority_Task SUCCESS!\n");

    /* 创建HighPriority_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)HighPriority_Task,		//任务函数
                          (const char* 	)"HighPriority_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)4, 				//任务优先级
                          (TaskHandle_t* )&HighPriority_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("HighPriority_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LowPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void LowPriority_Task(void* parameter)
{
    static uint32_t i;
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        printf("LowPriority_Task take\r\n");
        xReturn = xSemaphoreTake(MuxSem_Handle,/* 二值信号量句柄 */
                                 portMAX_DELAY); /* 等待时间 */
        if( xReturn == pdTRUE )
            printf("LowPriority_Task Runing\n\n");
        for(i=0; i<2000000; i++) //模拟低优先级任务占用信号量
        {
            taskYIELD();//发起任务调度
        }
        printf("LowPriority_Task give\r\n");
        xReturn = xSemaphoreGive(MuxSem_Handle);
        LED1_TOGGLE;

        vTaskDelay(500);
    }
}
/**
 * @brief  MidPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void MidPriority_Task(void* parameter)
{
    while (1)
    {
        printf("MidPriority_Task Runing\n");
        vTaskDelay(500);
    }
}
/**
 * @brief  HighPriority_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void HighPriority_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        printf("HighPriority_Task take\n");
        //获取二值信号量 xSemaphore,没获取到则一直等待
        xReturn = xSemaphoreTake(MuxSem_Handle,/* 二值信号量句柄 */
                                 portMAX_DELAY); /* 等待时间 */
        if(pdTRUE == xReturn)
            printf("HighPriority_Task Runing\n");
        LED1_TOGGLE;
        xReturn = xSemaphoreGive( MuxSem_Handle );//释放二值信号量

        vTaskDelay(500);
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
## 调试
- 编译下载到开发板
- 可以看到在低优先级任务运行的时候，中优先级任务无法抢占低优先级的任务
