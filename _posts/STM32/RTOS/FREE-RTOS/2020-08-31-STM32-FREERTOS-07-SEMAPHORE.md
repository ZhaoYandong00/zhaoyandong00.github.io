---
title: 信号量FREERTOS篇7
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS信号量
---
# 常用函数
- 创建二值信号量`SemaphoreHandle_t xSemaphoreCreateBinary(void)`
- 创建计数信号量`SemaphoreHandle_t xSemaphoreCreateCounting(UBaseType_t uxMaxCount,UBaseType_t uxInitialCount)`
- 信号量删除`void vSemaphoreDelete(SemaphoreHandle_t xSemaphore)`
- 信号量释放`BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore)`
- 中断中信号量释放`BaseType_t xSemaphoreGiveFromISR(SemaphoreHandle_t xSemaphore,BaseType_t * pxHigherPriorityTaskWoken)`
- 信号量获取`BaseType_t xSemaphoreTake(SemaphoreHandle_t xSemaphore,TickType_t xBlockTime)`
- 中断中信号量获取`BaseType_t xSemaphoreTakeFromISR(SemaphoreHandle_t xSemaphore,BaseType_t *pxHigherPriorityTaskWoken)`

# 二值信号量
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
static TaskHandle_t Receive_Task_Handle = NULL;/* 接收任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* 发送任务句柄 */
/* 消息队列句柄 */
SemaphoreHandle_t BinarySem_Handle =NULL;

static void AppTaskCreate(void);/* 用于创建任务 */

static void Receive_Task(void* pvParameters);/* Receive_Task任务实现 */
static void Send_Task(void* pvParameters);/* Send_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-BinarySem!\r\n");
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

    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Receive_Task,		//任务函数
                          (const char* 	)"Receive_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&Receive_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Receive_Task SUCCESS!\n");

    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Send_Task,		//任务函数
                          (const char* 	)"Send_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&Send_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Send_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Receive_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    while (1)
    {
        xReturn = xSemaphoreTake(BinarySem_Handle,/* 二值信号量句柄 */
                                 portMAX_DELAY); /* 等待时间 */
        if(pdTRUE == xReturn)
        {
            printf("receive  success");
            LED1_TOGGLE;
        }

    }
}
/**
 * @brief  Send_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Send_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {   /* K1 被按下 */
            xReturn = xSemaphoreGive( BinarySem_Handle );
            if(pdPASS == xReturn)
                printf("BinarySem_Handle success!\n\n");
            else
                printf("BinarySem_Handle fail!\n\n");
        }
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {   /* K2 被按下 */
            xReturn = xSemaphoreGive( BinarySem_Handle );
            if(pdPASS == xReturn)
                printf("BinarySem_Handle success!\n\n");
            else
                printf("BinarySem_Handle fail!\n\n");
        }
        vTaskDelay(20);/* 延时20个tick */
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
- 按下开发版的KEY1 KEY2释放二值信息
- 在串口助手中可以看到接收到消息

# 计数信号量
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
static TaskHandle_t Receive_Task_Handle = NULL;/* 接收任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* 发送任务句柄 */
/* 消息队列句柄 */
SemaphoreHandle_t CountSem_Handle =NULL;


static void AppTaskCreate(void);/* 用于创建任务 */

static void Receive_Task(void* pvParameters);/* Receive_Task任务实现 */
static void Send_Task(void* pvParameters);/* Send_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-CountSem!\r\n");
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

    CountSem_Handle = xSemaphoreCreateCounting(5,5);
    if(NULL != CountSem_Handle)
        printf("BinarySem SUCCESS!\n");

    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Receive_Task,		//任务函数
                          (const char* 	)"Receive_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&Receive_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Receive_Task SUCCESS!\n");

    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Send_Task,		//任务函数
                          (const char* 	)"Send_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&Send_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Send_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Receive_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    while (1)
    {
        //如果KEY1被单击
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
            /* 获取一个计数信号量 */
            xReturn = xSemaphoreTake(CountSem_Handle,	/* 计数信号量句柄 */
                                     0); 	/* 等待时间：0 */
            if ( pdTRUE == xReturn )
                printf( "success!\n" );
            else
                printf( "fail!\n" );
        }
        vTaskDelay(20);     //每20ms扫描一次
    }
}
/**
 * @brief  Send_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Send_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    while (1)
    {
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {   /* K2 被按下 */
            xReturn = xSemaphoreGive(CountSem_Handle);
            if(pdPASS == xReturn)
                printf("key2 success!\n\n");
            else
                printf("key2 fail!\n\n");
        }
        vTaskDelay(20);/* 延时20个tick */
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
- 按下开发版的KEY1获取，按下KEY2释放
- 串口助手可看到当按下5次KEY1,再按就会出现错误，这是先按下KEY2,再按KEY1，成功