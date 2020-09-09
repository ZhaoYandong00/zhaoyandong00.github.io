---
title:  CPU使用率统计FREERTOS篇13
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS CPU使用率统计
---
# 修改配置文件
- 修改`FreeRTOSConfig.h`

```c
#define configUSE_TRACE_FACILITY              1 //
#define configGENERATE_RUN_TIME_STATS         1 //运行时间宏
#define configUSE_STATS_FORMATTING_FUNCTIONS  1 //状态格式函数宏
extern volatile uint32_t CPU_RunTime;
#define portCONFIGURE_TIMER_FOR_RUN_TIME_STATS() (CPU_RunTime = 0ul)
#define portGET_RUN_TIME_COUNTER_VALUE() CPU_RunTime
```
# 添加定时器
- 复制SPL的`my_tim.c``my_tim.h`即可

# 中断
- `stm32f10x_it.c`

```c
#include "stm32f10x_it.h"
/* FreeRTOS头文件 */
#include "FreeRTOS.h"
#include "task.h"
#include "my_tim.h"
/* 用于统计运行时间 */
volatile uint32_t CPU_RunTime = 0UL;

void  MY_TIM_IRQHandler (void)
{
    if ( TIM_GetITStatus( MY_TIM, TIM_IT_Update) != RESET )
    {
        CPU_RunTime++;
        TIM_ClearITPendingBit(MY_TIM , TIM_IT_Update);
    }
}

```
# 主程序
- `main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"
#include "my_tim.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t LED1_Task_Handle = NULL;
static TaskHandle_t LED2_Task_Handle = NULL;
static TaskHandle_t CPU_Task_Handle = NULL;



static void AppTaskCreate(void);/* 用于创建任务 */

static void LED1_Task(void* pvParameters);/* LED1_Task任务实现 */
static void LED2_Task(void* pvParameters);/* LED2_Task任务实现 */
static void CPU_Task(void* pvParameters);/* CPU_Task任务实现 */


static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-CPUINFO!\r\n");
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    /* 创建 AppTaskCreate 任务 */
    xReturn = xTaskCreate((TaskFunction_t	)AppTaskCreate,		//任务函数
                          (const char* 	)"AppTaskCreate",		//任务名称
                          (uint32_t 		)512,	//任务堆栈大小
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

  /* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LED1_Task, /* 任务入口函数 */
                        (const char*    )"LED1_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,	/* 任务入口函数参数 */
                        (UBaseType_t    )2,	    /* 任务的优先级 */
                        (TaskHandle_t*  )&LED1_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
        printf("LED1_Task  SUCCESS!\r\n");
   /* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LED2_Task, /* 任务入口函数 */
                        (const char*    )"LED2_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,	/* 任务入口函数参数 */
                        (UBaseType_t    )3,	    /* 任务的优先级 */
                        (TaskHandle_t*  )&LED2_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
        printf("LED2_Task  SUCCESS!\r\n");
   /* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )CPU_Task, /* 任务入口函数 */
                        (const char*    )"CPU_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,	/* 任务入口函数参数 */
                        (UBaseType_t    )4,	    /* 任务的优先级 */
                        (TaskHandle_t*  )&CPU_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
        printf("CPU_Task  SUCCESS!\r\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LED1_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void LED1_Task(void* parameter)
{	
  while (1)
  {
    LED1(ON);
    vTaskDelay(500);   /* 延时500个tick */
    printf("LED1_Task Running,LED1_ON\r\n");
    LED1(OFF);     
    vTaskDelay(500);   /* 延时500个tick */		 		
    printf("LED1_Task Running,LED1_OFF\r\n");
  }
}
/**
 * @brief  LED2_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void LED2_Task(void* parameter)
{	
  while (1)
  {
    LED2(ON);
    vTaskDelay(300);   /* 延时500个tick */
    printf("LED2_Task Running,LED1_ON\r\n");
    
    LED2(OFF);     
    vTaskDelay(300);   /* 延时500个tick */		 		
    printf("LED2_Task Running,LED1_OFF\r\n");
  }
}
/**
 * @brief  CPU_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void CPU_Task(void* parameter)
{	
  uint8_t CPU_RunInfo[400];		//保存任务运行时间信息
  
  while (1)
  {
    memset(CPU_RunInfo,0,400);				//信息缓冲区清零
    
    vTaskList((char *)&CPU_RunInfo);  //获取任务运行时间信息
    
    printf("---------------------------------------------\r\n");
    printf("Name        State Priority Free_Strack ID\r\n");
    printf("%s", CPU_RunInfo);
    printf("---------------------------------------------\r\n");
    
    memset(CPU_RunInfo,0,400);				//信息缓冲区清零
    
    vTaskGetRunTimeStats((char *)&CPU_RunInfo);
    
    printf("Name          Count         Utilization\r\n");
    printf("%s", CPU_RunInfo);
    printf("---------------------------------------------\r\n\n");
    vTaskDelay(1000);   /* 延时500个tick */		
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
    /*初始化TIM 1ms*/
    MY_TIM_Init();
}

```
# 调试
- 编译下载到开发板
- 打开串口助手，可以看到CPU信息