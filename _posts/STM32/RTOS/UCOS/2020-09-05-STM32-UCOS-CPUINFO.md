---
title: CPU使用率统计UCOS篇13
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS CPU使用率统计
---
# CPU使用率统计常用函数
- CPU使用率统计初始化`void OSStatTaskCPUUsageInit(OS_ERR *p_err)`

# CPU使用率统计
- 修改`main.c`

```c
#include "os.h"
#include "lib_ascii.h"
#include "lib_math.h"
#include "lib_mem.h"
#include "lib_str.h"
#include "app_cfg.h"
#include "my_gpio.h"
#include "my_usart.h"

OS_SEM SemOfKey;          //标志KEY1是否被单击的多值信号量
/*
*********************************************************************************************************
*                                                 TCB
*********************************************************************************************************
*/
static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskLed1TCB;
static  OS_TCB   AppTaskLed2TCB;
static  OS_TCB   AppTaskUsartTCB;
static  OS_TCB   AppTaskStatusTCB;
/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskLed1Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed2Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskUsartStk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskStatusStk [ APP_TASK_STK_SIZE ];

/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskLed1  ( void * p_arg );
static  void  AppTaskLed2  ( void * p_arg );
static  void  AppTaskUsart  ( void * p_arg );
static  void  AppTaskStatus  ( void * p_arg );


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

    /* 创建多值信号量 SemOfKey */
    OSSemCreate((OS_SEM      *)&SemOfKey,    //指向信号量变量的指针
                (CPU_CHAR    *)"SemOfKey",    //信号量的名字
                (OS_SEM_CTR   )1,             //信号量这里是指示事件发生，所以赋值为0，表示事件还没有发生
                (OS_ERR      *)&err);         //错误类型

    OSTaskCreate((OS_TCB     *)&AppTaskLed1TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led1",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed1,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO+3,                         //任务的优先级
                 (CPU_STK    *)&AppTaskLed1Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
    OSTaskCreate((OS_TCB     *)&AppTaskLed2TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led2",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed2,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO+2,                         //任务的优先级
                 (CPU_STK    *)&AppTaskLed2Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 AppTaskUsart 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskUsartTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Usart",                             //任务名称
                 (OS_TASK_PTR ) AppTaskUsart,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO+1,                         //任务的优先级
                 (CPU_STK    *)&AppTaskUsartStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
    OSTaskCreate((OS_TCB     *)&AppTaskStatusTCB,
                 (CPU_CHAR   *)"App Task Status",
                 (OS_TASK_PTR ) AppTaskStatus,
                 (void       *) 0,
                 (OS_PRIO     ) APP_TASK_PRIO,
                 (CPU_STK    *)&AppTaskStatusStk[0],
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,
                 (OS_MSG_QTY  ) 5u,
                 (OS_TICK     ) 0u,
                 (void       *) 0,
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR),
                 (OS_ERR     *)&err);
    OSTaskDel ( & AppTaskStartTCB, & err );
}
/**
 * @brief  Led1任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed1 ( void * p_arg )
{
    OS_ERR      err;
    uint32_t    i;
    (void)p_arg;
    while (DEF_TRUE) {

        printf("AppTaskLed1 Running\n");

        for(i=0; i<10000; i++)    //模拟任务占用cpu
        {
            ;
        }
        LED1_TOGGLE;
        OSTimeDlyHMSM (0,0,0,500,OS_OPT_TIME_PERIODIC,&err);
    }
}
/**
 * @brief  Led2任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed2 ( void * p_arg )
{
    OS_ERR      err;
    uint32_t    i;
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        printf("AppTaskLed2 Running\n");

        for(i=0; i<100000; i++)    //模拟任务占用cpu
        {
            ;
        }
        LED2_TOGGLE;
        OSTimeDlyHMSM (0,0,0,500,OS_OPT_TIME_PERIODIC,&err);
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
    uint32_t    i;
    (void)p_arg;
    while (DEF_TRUE) {

        for(i=0; i<500000; i++)    //模拟任务占用cpu
        {
            ;
        }
        printf("AppTaskUsart Running\n");
        OSTimeDlyHMSM (0,0,0,500,OS_OPT_TIME_PERIODIC,&err);
    }
}

static  void  AppTaskStatus  ( void * p_arg )
{ 
    OS_ERR      err;

    CPU_SR_ALLOC();

    (void)p_arg;

    while (DEF_TRUE) {
      
      OS_CRITICAL_ENTER();                              //进入临界段，避免串口打印被打断
      printf("------------------------------------------------------------\n");
      printf ( "CPU Utilization:%d.%d%%\r\n",
               OSStatTaskCPUUsage / 100, OSStatTaskCPUUsage % 100 );  
      printf ( "CPU Max Utilization:%d.%d%%\r\n", 
               OSStatTaskCPUUsageMax / 100, OSStatTaskCPUUsageMax % 100 );         
      printf ( "LED1 Task Utilization:%d.%d%%\r\n", 
               AppTaskLed1TCB.CPUUsageMax / 100, AppTaskLed1TCB.CPUUsageMax % 100 ); 
      printf ( "LED2 Task Utilization:%d.%d%%\r\n", 
               AppTaskLed2TCB.CPUUsageMax / 100, AppTaskLed2TCB.CPUUsageMax % 100 ); 
      printf ( "Usart Task Utilization:%d.%d%%\r\n", 
               AppTaskUsartTCB.CPUUsageMax / 100, AppTaskUsartTCB.CPUUsageMax % 100 ); 
      printf ( "Status Task Utilization:%d.%d%%\r\n", 
               AppTaskStatusTCB.CPUUsageMax / 100, AppTaskStatusTCB.CPUUsageMax % 100 ); 
      printf ( "LED1 Use:%d,Free:%d\r\n", 
               AppTaskLed1TCB.StkUsed, AppTaskLed1TCB.StkFree ); 
      printf ( "LED2 Use:%d,Free:%d\r\n", 
               AppTaskLed2TCB.StkUsed, AppTaskLed2TCB.StkFree ); 
      printf ( "Usart Use:%d,Free:%d\r\n", 
               AppTaskUsartTCB.StkUsed, AppTaskUsartTCB.StkFree );     
      printf ( "Status Use:%d,Free:%d\r\n", 
               AppTaskStatusTCB.StkUsed, AppTaskStatusTCB.StkFree );               
      printf("------------------------------------------------------------\n");         
      OS_CRITICAL_EXIT();                               //退出临界段
      OSTimeDlyHMSM (0,0,0,500,OS_OPT_TIME_PERIODIC,&err);       
    }
}
#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
void  CPU_TS_TmrInit (void)
{
    CoreDebug->DEMCR|=CoreDebug_DEMCR_TRCENA_Msk;
    DWT->CYCCNT=0;
    DWT->CTRL|=DWT_CTRL_CYCCNTENA_Msk;
    CPU_TS_TmrFreqSet(SystemCoreClock);
}
CPU_TS_TMR  CPU_TS_TmrRd (void)
{
    return DWT->CYCCNT;
}
#endif

```
# 调试
- 编译下载到开发板
- 打开串口助手，可以看到CPU信息