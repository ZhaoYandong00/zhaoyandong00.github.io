---
title: 任务管理UCOS篇3
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS任务管理
---
# 程序
- `os_cfg.h`打开时间片轮转调度

```c
#define OS_CFG_SCHED_ROUND_ROBIN_EN     1u   /* Include code for Round-Robin scheduling */ 
```

- 修改`app_cfg.h`

```c
#ifndef  __APP_CFG_H__
#define  __APP_CFG_H__
/*
*********************************************************************************************************
*                                            TASK PRIORITIES
*********************************************************************************************************
*/
#define  APP_TASK_START_PRIO                        2

#define  APP_TASK_LED1_PRIO                         3
#define  APP_TASK_LED2_PRIO                         3
#define  APP_TASK_USART_PRIO                        3
/*
*********************************************************************************************************
*                                            TASK STACK SIZES
*                             Size of the task stacks (# of OS_STK entries)
*********************************************************************************************************
*/
#define  APP_TASK_START_STK_SIZE                    128

#define  APP_TASK_LED1_STK_SIZE                     512
#define  APP_TASK_LED2_STK_SIZE                     512
#define  APP_TASK_USART_STK_SIZE                    512 
#endif

```
- 修改`main.c`

```c
#include "os.h"
#include  "lib_ascii.h"
#include  "lib_math.h"
#include  "lib_mem.h"
#include  "lib_str.h"
#include "app_cfg.h"
#include "my_gpio.h"
#include "my_usart.h"
/*
*********************************************************************************************************
*                                                 TCB
*********************************************************************************************************
*/
static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskLed1TCB;
static  OS_TCB   AppTaskLed2TCB;
static  OS_TCB   AppTaskUsartTCB;
/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskLed1Stk [ APP_TASK_LED1_STK_SIZE ];
static  CPU_STK  AppTaskLed2Stk [ APP_TASK_LED2_STK_SIZE ];
static  CPU_STK  AppTaskUsartStk [ APP_TASK_USART_STK_SIZE ];
/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskLed1  ( void * p_arg );
static  void  AppTaskLed2  ( void * p_arg );
static  void  AppTaskUsart  ( void * p_arg );

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    OS_ERR  err;

    OSInit(&err);

    OSTaskCreate((OS_TCB     *)&AppTaskStartTCB,                /* Create the start task                                */
                 (CPU_CHAR   *)"App Task Start",
                 (OS_TASK_PTR ) AppTaskStart,
                 (void       *) 0,
                 (OS_PRIO     ) APP_TASK_START_PRIO,
                 (CPU_STK    *)&AppTaskStartStk[0],
                 (CPU_STK_SIZE) APP_TASK_START_STK_SIZE / 10,
                 (CPU_STK_SIZE) APP_TASK_START_STK_SIZE,
                 (OS_MSG_QTY  ) 5u,
                 (OS_TICK     ) 0u,
                 (void       *) 0,
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
                 (OS_ERR     *)&err);

    OSStart(&err);

    while(1);
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
/**
 * @brief  初始化和任务创建
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskStart(void *p_arg)
{
    OS_ERR      err;
    BSP_Init();
    CPU_Init();
    OS_CPU_SysTickInit(SystemCoreClock/OSCfg_TickRate_Hz);
    Mem_Init();
#if OS_CFG_STAT_TASK_EN > 0u
    OSStatTaskCPUUsageInit(&err);                               /* Compute CPU capacity with no task running            */
#endif
#ifdef CPU_CFG_INT_DIS_MEAS_EN
    CPU_IntDisMeasMaxCurReset();
#endif
    /* 配置时间片轮转调度 */
    OSSchedRoundRobinCfg((CPU_BOOLEAN   )DEF_ENABLED,          //使能时间片轮转调度
                         (OS_TICK       )0,                    //把 OSCfg_TickRate_Hz / 10 设为默认时间片值
                         (OS_ERR       *)&err );               //返回错误类型
    OSTaskCreate((OS_TCB     *)&AppTaskLed1TCB,                /* Create the Led1 task                                */
                 (CPU_CHAR   *)"App Task Led1",
                 (OS_TASK_PTR ) AppTaskLed1,
                 (void       *) 0,
                 (OS_PRIO     ) APP_TASK_LED1_PRIO,
                 (CPU_STK    *)&AppTaskLed1Stk[0],
                 (CPU_STK_SIZE) APP_TASK_LED1_STK_SIZE / 10,
                 (CPU_STK_SIZE) APP_TASK_LED1_STK_SIZE,
                 (OS_MSG_QTY  ) 5u,
                 (OS_TICK     ) 0u,
                 (void       *) 0,
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
                 (OS_ERR     *)&err);

    OSTaskCreate((OS_TCB     *)&AppTaskLed2TCB,                /* Create the Led2 task                                */
                 (CPU_CHAR   *)"App Task Led2",
                 (OS_TASK_PTR ) AppTaskLed2,
                 (void       *) 0,
                 (OS_PRIO     ) APP_TASK_LED2_PRIO,
                 (CPU_STK    *)&AppTaskLed2Stk[0],
                 (CPU_STK_SIZE) APP_TASK_LED2_STK_SIZE / 10,
                 (CPU_STK_SIZE) APP_TASK_LED2_STK_SIZE,
                 (OS_MSG_QTY  ) 5u,
                 (OS_TICK     ) 0u,
                 (void       *) 0,
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
                 (OS_ERR     *)&err);
    /* 创建 Usart 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskUsartTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Usart",                             //任务名称
                 (OS_TASK_PTR ) AppTaskUsart,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_USART_PRIO,                         //任务的优先级
                 (CPU_STK    *)&AppTaskUsartStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_USART_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_USART_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    OSTaskDel ( & AppTaskStartTCB, & err );
}
/**
 * @brief  LED1任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed1 ( void * p_arg )
{
    OS_ERR      err;
    OS_REG      value;
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        LED1_TOGGLE;                                 //切换 LED1 的亮灭状态
        printf("AppTaskLED1 Running\n");
        value = OSTaskRegGet ( 0, 0, & err );              //获取自身任务寄存器值

        if ( value < 10 )                                  //如果任务寄存器值<10
        {
            OSTaskRegSet ( 0, 0, ++ value, & err );          //继续累加任务寄存器值
        }
        else                                               //如果累加到10
        {
            OSTaskRegSet ( 0, 0, 0, & err );                 //将任务寄存器值归0

            OSTaskResume ( & AppTaskLed2TCB, & err );        //恢复 LED2 任务
            printf("Resume LED!\n");

            OSTaskResume ( & AppTaskUsartTCB, & err );        //恢复 LED3 任务
            printf("Resume usart!\n");
        }

        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err );        //相对性延时1000个时钟节拍（1s）
    }
}
/**
 * @brief  LED2任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed2 ( void * p_arg )
{
    OS_ERR      err;
    OS_REG      value;
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        LED2_TOGGLE;                               //切换 LED2 的亮灭状态
        printf("AppTaskLED2 Running\n");
        value = OSTaskRegGet ( 0, 0, & err );            //获取自身任务寄存器值

        if ( value < 5 )                                 //如果任务寄存器值<5
        {
            OSTaskRegSet ( 0, 0, ++ value, & err );        //继续累加任务寄存器值
        }
        else                                             //如果累加到5
        {
            OSTaskRegSet ( 0, 0, 0, & err );               //将任务寄存器值归0
            printf("Suspend LED!\n");
            OSTaskSuspend ( 0, & err );                    //挂起自身
        }

        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err );      //相对性延时1000个时钟节拍（1s）
    }
}
/**
 * @brief  Usart任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskUsart ( void * p_arg )
{
    OS_ERR      err;
    OS_REG      value;
    (void)p_arg;
    while (DEF_TRUE) {                                 //任务体，通常写成一个死循环
        printf("AppTaskUsart Running\n");
        value = OSTaskRegGet ( 0, 0, & err );            //获取自身任务寄存器值

        if ( value < 5 )                                 //如果任务寄存器值<5
        {
            OSTaskRegSet ( 0, 0, ++ value, & err );        //继续累加任务寄存器值
        }
        else                                             //如果累加到5
        {
            OSTaskRegSet ( 0, 0, 0, & err );               //将任务寄存器值归零
            printf("Suspend Usart!\n");
            OSTaskSuspend ( 0, & err );                    //挂起自身

        }

        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err );      //相对性延时1000个时钟节拍（1s）

    }
}

```
# 调试
- 编译下载到开发板
- LED 在闪烁，同时在串口调试助手也输出了相应的信息，说明任务已经被挂起与恢复
- 串口这时看到的数据有被其他任务穿插的，代表了任务时间片轮询调度在起作用
- 如果我们把时间片轮转调度程序注释掉，串口数据就是正常的，或者把三个任务优先级设为不同的