---
title: 软件定时器UCOS篇8
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS软件定时器
---
# 软件定时器常用函数
- 创建软件定时器`void OSTmrCreate(OS_TMR *p_tmr,CPU_CHAR *p_name,OS_TICK dly,OS_TICK period,OS_OPT opt,OS_TMR_CALLBACK_PTR p_callback,void *p_callback_arg,OS_ERR *p_err)`
- 删除软件定时器`CPU_BOOLEAN OSTmrDel(OS_TMR *p_tmr,OS_ERR *p_err)`
- 设置软件定时器`void OSTmrSet(OS_TMR *p_tmr,OS_TICK dly,OS_TICK period,OS_TMR_CALLBACK_PTR p_callback,void *p_callback_arg,OS_ERR *p_err)`
- 获取剩余时间`OS_TICK OSTmrRemainGet(OS_TMR *p_tmr,OS_ERR *p_err)`
- 启动软件定时器`CPU_BOOLEAN OSTmrStart(OS_TMR *p_tmr,OS_ERR *p_err)`
- 获取定时器状态`OS_STATE OSTmrStateGet(OS_TMR *p_tmr,OS_ERR *p_err)`
- 停止软件定时器`CPU_BOOLEAN OSTmrStop(OS_TMR *p_tmr,OS_OPT opt,void *p_callback_arg,OS_ERR *p_err)`

# 软件定时器
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

CPU_TS             ts_start;       //时间戳变量
CPU_TS             ts_end;

/*
*********************************************************************************************************
*                                                 TCB
*********************************************************************************************************
*/
static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskTmrTCB;

/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskTmrStk [ APP_TASK_STK_SIZE ];

/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskTmr  ( void * p_arg );

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
    /* 创建 AppTaskTmr 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskTmrTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Tmr",                             //任务名称
                 (OS_TASK_PTR ) AppTaskTmr,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                         //任务的优先级
                 (CPU_STK    *)&AppTaskTmrStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型                              //返回错误类型

    OSTaskDel ( & AppTaskStartTCB, & err );
}
void TmrCallback (OS_TMR *p_tmr, void *p_arg) //软件定时器MyTmr的回调函数
{
    CPU_SR_ALLOC();      //使用到临界段（在关/开中断时）时必需该宏，该宏声明和定义一个局部变
    //量，用于保存关中断前的 CPU 状态寄存器 SR（临界段关中断只需保存SR）
    //，开中断时将该值还原。
    printf ( "%s", ( char * ) p_arg );

    LED1_TOGGLE;

    ts_end = OS_TS_GET() - ts_start;     //获取定时后的时间戳（以CPU时钟进行计数的一个计数值）
    //，并计算定时时间。
    OS_CRITICAL_ENTER();                 //进入临界段，不希望下面串口打印遭到中断

    printf("\r\nTIMER 1s,TEST DWT %07d us, %04d ms.\r\n",
           ts_end / ( SystemCoreClock / 1000000 ),     //将定时时间折算成 us
           ts_end / ( SystemCoreClock / 1000 ) );      //将定时时间折算成 ms

    OS_CRITICAL_EXIT();

    ts_start = OS_TS_GET();                            //获取定时前时间戳
}
/**
 * @brief  Tmr任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskTmr ( void * p_arg )
{
    OS_ERR      err;
    OS_TMR      my_tmr;   //声明软件定时器对象
    (void)p_arg;
    /* 创建软件定时器 */
    OSTmrCreate ((OS_TMR              *)&my_tmr,             //软件定时器对象
                 (CPU_CHAR            *)"MySoftTimer",       //命名软件定时器
                 (OS_TICK              )10,                  //定时器初始值，依10Hz时基计算，即为1s
                 (OS_TICK              )10,                  //定时器周期重载值，依10Hz时基计算，即为1s
                 (OS_OPT               )OS_OPT_TMR_PERIODIC, //周期性定时
                 (OS_TMR_CALLBACK_PTR  )TmrCallback,         //回调函数
                 (void                *)"Timer Over!",       //传递实参给回调函数
                 (OS_ERR              *)err);                //返回错误类型

    /* 启动软件定时器 */
    OSTmrStart ((OS_TMR   *)&my_tmr, //软件定时器对象
                (OS_ERR   *)err);    //返回错误类型

    ts_start = OS_TS_GET();                       //获取定时前时间戳
    while (DEF_TRUE) {                            //任务体，通常写成一个死循环
        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err ); //不断阻塞该任务
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
- 打开串口助手，每1S 时间到的时候，软件定时器就会触发一次回调函数