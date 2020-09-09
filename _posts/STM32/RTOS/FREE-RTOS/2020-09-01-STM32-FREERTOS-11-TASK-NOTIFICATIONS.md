---
title: 任务通知FREERTOS篇11
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS任务通知
---
# 任务通知常用函数
- 发送任务通知,并更新对方的任务通知值`BaseType_t xTaskNotifyGive(TaskHandle_t xTaskToNotify)`
- 发送任务通知带指定值`BaseType_t xTaskNotify(TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction)`
- 发送任务通知并返回对象任务的上一个通知值`BaseType_t xTaskNotifyAndQuery(TaskHandle_t xTaskToNotify,uint32_t ulValue,eNotifyAction eAction,uint32_t * pulPreviousNotifyValue)`
- 中断中发送任务通知,带指定值`void vTaskNotifyGiveFromISR(TaskHandle_t xTaskHandle, BaseType_t *pxHigherPriorityTaskWoken)`
- 中断中发送任务通知,并更新对方的任务通知值`BaseType_t xTaskNotifyFromISR(TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction, BaseType_t *pxHigherPriorityTaskWoken)`
- 中断中发送任务通知并返回对象任务的上一个通知值`xTaskNotifyAndQueryFromISR(TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction, uint32_t *pulPreviousNotificationValue, BaseType_t *pxHigherPriorityTaskWoken)`
- 等待任务通知`BaseType_t xTaskNotifyWait(uint32_t ulBitsToClearOnEntry, uint32_t ulBitsToClearOnExit, uint32_t *pulNotificationValue, TickType_t xTicksToWait)`
- 获取任务通知`uint32_t ulTaskNotifyTake(BaseType_t xClearCountOnExit, TickType_t xTicksToWait)`
- 清除通知状态`BaseType_t xTaskNotifyStateClear(TaskHandle_t xTask)`
- 清除通知值`uint32_t ulTaskNotifyValueClear(TaskHandle_t xTask, uint32_t ulBitsToClear)`

# 任务通知代替消息队列
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t Receive1_Task_Handle = NULL;/* Receive1_Task任务句柄 */
static TaskHandle_t Receive2_Task_Handle = NULL;/* Receive2_Task任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* Send_Task任务句柄 */



static void AppTaskCreate(void);/* 用于创建任务 */


static void Receive1_Task(void* pvParameters);/* Receive1_Task任务实现 */
static void Receive2_Task(void* pvParameters);/* Receive2_Task任务实现 */

static void Send_Task(void* pvParameters);/* Send_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-NOTIFICATIONS!\r\n");
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
    xReturn = xTaskCreate((TaskFunction_t )Receive1_Task, /* 任务入口函数 */
                          (const char*    )"Receive1_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&Receive1_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Receive1_Task  SUCCESS!\r\n");
    /* 创建Receive2_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Receive2_Task, /* 任务入口函数 */
                          (const char*    )"Receive2_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )3,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&Receive2_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Receive2_Task  SUCCESS!\r\n");

    /* 创建Send_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Send_Task,  /* 任务入口函数 */
                          (const char*    )"Send_Task",/* 任务名字 */
                          (uint16_t       )512,  /* 任务栈大小 */
                          (void*          )NULL,/* 任务入口函数参数 */
                          (UBaseType_t    )4, /* 任务的优先级 */
                          (TaskHandle_t*  )&Send_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Send_Task  SUCCESS!\n\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Receive1_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive1_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    uint32_t r_num;
    while (1)
    {
        xReturn=xTaskNotifyWait(0x0,			//进入函数的时候不清除任务bit
                                0xffffffff,	  //退出函数的时候清除所有的bit
                                &r_num,		  //保存任务通知值
                                portMAX_DELAY);	//阻塞时间
        if( pdTRUE == xReturn )
        {
            printf("Receive1_Task notification: %d \n",r_num);
            LED1_TOGGLE;
        }
    }
}
/**
 * @brief  Receive2_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive2_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    uint32_t r_num;
    while (1)
    {
        xReturn=xTaskNotifyWait(0x0,			//进入函数的时候不清除任务bit
                                0xffffffff,	  //退出函数的时候清除所有的bit
                                &r_num,		  //保存任务通知值
                                portMAX_DELAY);	//阻塞时间
        if( pdTRUE == xReturn )
        {
            printf("Receive2_Task notification: %d \n",r_num);
            LED2_TOGGLE;
        }
    }
}
/**
 * @brief  Send_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Send_Task(void* parameter)
{
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    uint32_t send1 = 1;
    uint32_t send2 = 2;
    while (1)
    {
        /* KEY1 被按下 */
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
            xReturn = xTaskNotify( Receive1_Task_Handle, /*任务句柄*/
                                   send1, /* 发送的数据，最大为4字节 */
                                   eSetValueWithOverwrite );/*覆盖当前通知*/
            if( xReturn == pdPASS )
                printf("Receive1_Task_Handle SEND SUCCESS!\r\n");
        }
        /* KEY2 被按下 */
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {
            xReturn = xTaskNotify( Receive2_Task_Handle, /*任务句柄*/
                                   send2, /* 发送的数据，最大为4字节 */
                                   eSetValueWithOverwrite );/*覆盖当前通知*/
            if( xReturn == pdPASS )
                printf("Receive2_Task_Handle SEND SUCCESS!\r\n");
        }
        vTaskDelay(20);
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
- 按下KEY1，串口调试助手中可以看到接收到消息1
- 按下KEY2，串口调试助手中可以看到接收到消息2

# 任务通知代替二值信号量
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t Receive1_Task_Handle = NULL;/* Receive1_Task任务句柄 */
static TaskHandle_t Receive2_Task_Handle = NULL;/* Receive2_Task任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* Send_Task任务句柄 */



static void AppTaskCreate(void);/* 用于创建任务 */


static void Receive1_Task(void* pvParameters);/* Receive1_Task任务实现 */
static void Receive2_Task(void* pvParameters);/* Receive2_Task任务实现 */

static void Send_Task(void* pvParameters);/* Send_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-NOTIFICATIONS!\r\n");
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
    xReturn = xTaskCreate((TaskFunction_t )Receive1_Task, /* 任务入口函数 */
                          (const char*    )"Receive1_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&Receive1_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Receive1_Task  SUCCESS!\r\n");
    /* 创建Receive2_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Receive2_Task, /* 任务入口函数 */
                          (const char*    )"Receive2_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )3,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&Receive2_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Receive2_Task  SUCCESS!\r\n");

    /* 创建Send_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Send_Task,  /* 任务入口函数 */
                          (const char*    )"Send_Task",/* 任务名字 */
                          (uint16_t       )512,  /* 任务栈大小 */
                          (void*          )NULL,/* 任务入口函数参数 */
                          (UBaseType_t    )4, /* 任务的优先级 */
                          (TaskHandle_t*  )&Send_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Send_Task  SUCCESS!\n\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Receive1_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive1_Task(void* parameter)
{
    while (1)
    {
        ulTaskNotifyTake(pdTRUE,portMAX_DELAY);
        printf("Receive1_Task notification\n");
        LED1_TOGGLE;
    }
}
/**
 * @brief  Receive2_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive2_Task(void* parameter)
{
    while (1)
    {
        ulTaskNotifyTake(pdTRUE,portMAX_DELAY);
        printf("Receive2_Task notification\n");
        LED2_TOGGLE;
    }
}
/**
 * @brief  Send_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Send_Task(void* parameter)
{
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        /* KEY1 被按下 */
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
           xReturn = xTaskNotifyGive(Receive1_Task_Handle);
            if( xReturn == pdPASS )
                printf("Receive1_Task_Handle SEND SUCCESS!\r\n");
        }
        /* KEY2 被按下 */
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {
            xReturn = xTaskNotifyGive(Receive2_Task_Handle);
            if( xReturn == pdPASS )
                printf("Receive2_Task_Handle SEND SUCCESS!\r\n");
        }
        vTaskDelay(20);
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

# 任务通知代替计数信号量
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t Take_Task_Handle = NULL;/* Take_Task任务句柄 */
static TaskHandle_t Give_Task_Handle = NULL;/* Give_Task任务句柄 */


static void AppTaskCreate(void);/* 用于创建任务 */


static void Take_Task(void* pvParameters);/* Take_Task任务实现 */
static void Give_Task(void* pvParameters);/* Give_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-NOTIFICATIONS!\r\n");
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
    /* 创建Take_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Take_Task, /* 任务入口函数 */
                          (const char*    )"Take_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&Take_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Take_Task  SUCCESS!\r\n");


    /* 创建Give_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )Give_Task,  /* 任务入口函数 */
                          (const char*    )"Give_Task",/* 任务名字 */
                          (uint16_t       )512,  /* 任务栈大小 */
                          (void*          )NULL,/* 任务入口函数参数 */
                          (UBaseType_t    )3, /* 任务的优先级 */
                          (TaskHandle_t*  )&Give_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("Give_Task  SUCCESS!\n\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Take_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Take_Task(void* parameter)
{
    uint32_t take_num = pdTRUE;
    while(1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
            take_num=ulTaskNotifyTake(pdFALSE,0);
            if(take_num > 0)
            {
                printf( "KEY1 success,%d\n",take_num-1);
            }
            else {
                printf( "KEY1 fail\n" );
            }
        }
        vTaskDelay(20);
    }
}
/**
 * @brief  Give_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void Give_Task(void* parameter)
{
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    while (1)
    {
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {
            xReturn= xTaskNotifyGive(Take_Task_Handle);
            if ( pdPASS == xReturn )
                printf( "KEY2 success\n" );
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
## 调试
- 编译下载到开发板
- 按下开发版的KEY1获取信号量，按下KEY2 按键释放信号量
- 在串口助手中可以看到接收到消息

# 任务通知代替事件组
## 程序
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t LED_Task_Handle = NULL;/* LED_Task任务句柄 */
static TaskHandle_t KEY_Task_Handle = NULL;/* KEY_Task任务句柄 */

#define KEY1_EVENT  (0x01 << 0)//设置事件掩码的位0
#define KEY2_EVENT  (0x01 << 1)//设置事件掩码的位1

static void AppTaskCreate(void);/* 用于创建任务 */


static void LED_Task(void* pvParameters);/* LED_Task 任务实现 */
static void KEY_Task(void* pvParameters);/* KEY_Task 任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-NOTIFICATIONS!\r\n");
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
    /* 创建Take_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                          (const char*    )"Take_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("LED_Task  SUCCESS!\r\n");


    /* 创建Give_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )KEY_Task,  /* 任务入口函数 */
                          (const char*    )"KEY_Task",/* 任务名字 */
                          (uint16_t       )512,  /* 任务栈大小 */
                          (void*          )NULL,/* 任务入口函数参数 */
                          (UBaseType_t    )3, /* 任务的优先级 */
                          (TaskHandle_t*  )&KEY_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("KEY_Task  SUCCESS!\n\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LED_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void LED_Task(void* parameter)
{
    uint32_t r_event = 0;  /* 定义一个事件接收变量 */
    uint32_t last_event = 0;/* 定义一个保存事件的变量 */
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    while(1)
    {
        xReturn = xTaskNotifyWait(0x0,			//进入函数的时候不清除任务bit
                                  0xFFFFFFFF,	  //退出函数的时候清除所有的bitR
                                  &r_event,		  //保存任务通知值
                                  portMAX_DELAY);	//阻塞时间
        if( pdTRUE == xReturn )
        {
            last_event |= r_event;
            /* 如果接收完成并且正确 */
            if(last_event == (KEY1_EVENT|KEY2_EVENT))
            {
                last_event = 0;     /* 上一次的事件清零 */
                printf ( "Key1 Key2 ALL\n");
                LED1_TOGGLE;       //LED1	反转
            }
            else  /* 否则就更新事件 */
                last_event = r_event;   /* 更新上一次触发的事件 */
        }
    }
}
/**
 * @brief  KEY_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void KEY_Task(void* parameter)
{
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
            printf ( "KEY1 press\n" );
            xTaskNotify((TaskHandle_t	)LED_Task_Handle,//接收任务通知的任务句柄
                        (uint32_t		)KEY1_EVENT,			//要触发的事件
                        (eNotifyAction)eSetBits);			//设置任务通知值中的位
        }
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {
            printf ( "KEY2 press\n" );
            xTaskNotify((TaskHandle_t	)LED_Task_Handle,//接收任务通知的任务句柄
                        (uint32_t		)KEY2_EVENT,			//要触发的事件
                        (eNotifyAction)eSetBits);			//设置任务通知值中的位
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
## 调试
- 编译下载到开发板
- 按下KEY1 按键发送事件1,按下KEY2,按键发送事件2
- 我们按下KEY1与KEY2试试,在串口调试助手中可以看到运行结果,并且当事件1与事件2都发生的时候,开发板的LED会进行翻转