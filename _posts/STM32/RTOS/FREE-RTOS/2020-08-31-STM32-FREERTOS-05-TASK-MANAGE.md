---
title: 任务管理FREERTOS篇5
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS任务管理
---
# 硬件初始化配置
- 和SPL库按键输入一样
- 在`my_gpio.c`添加按键处理

```c
/**
  * @brief  配置按键用到的I/O口
  * @param  无
  * @retval 无
  */
void Key_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    /*开启按键端口的时钟*/
    RCC_APB2PeriphClockCmd(KEY1_GPIO_CLK | KEY2_GPIO_CLK, ENABLE);

    //选择按键的引脚
    GPIO_InitStructure.GPIO_Pin = KEY1_GPIO_PIN;
    // 设置按键的引脚为浮空输入
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    //使用结构体初始化按键
    GPIO_Init(KEY1_GPIO_PORT, &GPIO_InitStructure);

    //选择按键的引脚
    GPIO_InitStructure.GPIO_Pin = KEY2_GPIO_PIN;
    //设置按键的引脚为浮空输入
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    //使用结构体初始化按键
    GPIO_Init(KEY2_GPIO_PORT, &GPIO_InitStructure);
}
/*
* 函数名：Key_Scan
* 描述  ：检测是否有按键按下
* 输入  ：GPIOx：x 可以是 A，B，C，D或者 E
*		     GPIO_Pin：待读取的端口位
* 输出  ：KEY_OFF(没按下按键)、KEY_ON（按下按键）
*/
uint8_t Key_Scan(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)
{
    /*检测是否有按键按下 */
    if(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON )
    {
        /*等待按键释放 */
        while(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON);
        return 	KEY_ON;
    }
    else
        return KEY_OFF;
}

```
# 任务管理
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
static TaskHandle_t KEY_Task_Handle;

static void AppTaskCreate(void);/* 用于创建任务 */
static void BSP_Init(void);/* 用于初始化板载相关资源 */
static void LED1_Task(void* pvParameters);/* LED_Task任务实现 */
static void LED2_Task(void* pvParameters);/* LED_Task任务实现 */
static void KEY_Task(void* pvParameters);/* LED_Task任务实现 */

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
/**********************************************************************
  * @ 函数名  ： KEY_Task
  * @ 功能说明： KEY_Task任务主体
  * @ 参数    ：
  * @ 返回值  ： 无
  ********************************************************************/
static void KEY_Task(void* parameter)
{
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {   /* K1 被按下 */
            printf("挂起LED任务！\n");
            vTaskSuspend(LED1_Task_Handle);/* 挂起LED任务 */
            printf("挂起LED任务成功！\n");
        }
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {   /* K2 被按下 */
            printf("恢复LED任务！\n");
            vTaskResume(LED1_Task_Handle);/* 恢复LED任务！ */
            printf("恢复LED任务成功！\n");
        }
        vTaskDelay(20);/* 延时20个tick */
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
    /* 创建KEY_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)KEY_Task,		//任务函数
                          (const char* 	)"KEY_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)4, 				//任务优先级
                          (TaskHandle_t* )&KEY_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("KEY_Task SUCCESS!\n");
    else
        printf("KEY_Task FAIL!\n");

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
    /* 按键初始化 */
    Key_GPIO_Config();
}

```
# 调试
- 编译下载到开发板
- 可以看到LED1和LED2以不同频率闪烁
- 按下KEY1，LED1不闪烁了，按下KEY2,LED1继续闪烁