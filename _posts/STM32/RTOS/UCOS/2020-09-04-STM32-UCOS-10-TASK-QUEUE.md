---
title: 任务消息队列UCOS篇10
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS任务消息队列
---
# 任务消息队列
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


/*
*********************************************************************************************************
*                                                 TCB
*********************************************************************************************************
*/
static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskPostTCB;
static  OS_TCB   AppTaskPendTCB;

/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskPostStk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskPendStk [ APP_TASK_STK_SIZE ];


/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskPost   ( void * p_arg );
static  void  AppTaskPend   ( void * p_arg );

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
    /* 创建 AppTaskPost 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskPostTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Post",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPost,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                         //任务的优先级
                 (CPU_STK    *)&AppTaskPostStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型                              //返回错误类型
    OSTaskCreate((OS_TCB     *)&AppTaskPendTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Pend",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPend,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                         //任务的优先级
                 (CPU_STK    *)&AppTaskPendStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                     //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);
    OSTaskDel ( & AppTaskStartTCB, & err );
}

/**
 * @brief  Post任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskPost ( void * p_arg )
{
    OS_ERR      err;
    (void)p_arg;
    while (DEF_TRUE) {                                                         //任务体
        /* 发送消息到任务 AppTaskPend */
        OSTaskQPost ((OS_TCB      *)&AppTaskPendTCB,                 //目标任务的控制块
                     (void        *)"uC/OS-III",             //消息内容
                     (OS_MSG_SIZE  )sizeof ( "uC/OS-III" ),  //消息长度
                     (OS_OPT       )OS_OPT_POST_FIFO,                //发送到任务消息队列的入口端
                     (OS_ERR      *)&err);                           //返回错误类型

        OSTimeDlyHMSM ( 0, 0, 1, 0, OS_OPT_TIME_DLY, & err );        //每20ms发送一次                 //每20ms扫描一次
    }
}
/**
 * @brief  Pend任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskPend ( void * p_arg )
{
    OS_ERR         err;
    CPU_SR_ALLOC();
    OS_MSG_SIZE    msg_size;
    CPU_TS         ts;
    char * pMsg;
    (void)p_arg;
    while (DEF_TRUE) {                                  //任务体

        /* 阻塞任务，等待任务消息 */
        pMsg = OSTaskQPend ((OS_TICK        )0,                    //无期限等待
                            (OS_OPT         )OS_OPT_PEND_BLOCKING, //没有消息就阻塞任务
                            (OS_MSG_SIZE   *)&msg_size,            //返回消息长度
                            (CPU_TS        *)&ts,                  //返回消息被发送的时间戳
                            (OS_ERR        *)&err);                //返回错误类型

        ts = OS_TS_GET() - ts;                            //计算消息从发送到被接收的时间差

        LED1_TOGGLE;                                //切换LED1的亮灭状态

        OS_CRITICAL_ENTER();                              //进入临界段，避免串口打印被打断


        printf( "msg:%s,len:%d\n", pMsg, msg_size );
        printf ( "\r\ntime:%dus\r\n",ts / ( SystemCoreClock / 1000000 ) );

        OS_CRITICAL_EXIT();                               //退出临界段
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
- 打开串口助手，可以看到每1s接收到一次数据，证明消息队列运行正常