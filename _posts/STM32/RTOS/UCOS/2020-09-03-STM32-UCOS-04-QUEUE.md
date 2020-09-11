---
title: 消息队列UCOS篇4
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS消息队列
---
# 消息队列常用函数
- 创建消息队列`void OSQCreate(OS_Q *p_q,CPU_CHAR *p_name,OS_MSG_QTY max_qty,OS_ERR *p_err)`
- 删除消息队列`OS_OBJ_QTY OSQDel(OS_Q *p_q,OS_OPT opt,OS_ERR *p_err)`
- 刷新消息队列`OS_MSG_QTY OSQFlush(OS_Q *p_q,OS_ERR *p_err)`
- 获取消息队列`void *OSQPend(OS_Q *p_q,OS_TICK timeout,OS_OPT opt,OS_MSG_SIZE *p_msg_size,CPU_TS *p_ts,OS_ERR *p_err)`
- 中止获取消息队列`OS_OBJ_QTY OSQPendAbort(OS_Q *p_q,OS_OPT opt,OS_ERR *p_err)`
- 发送消息队列`void OSQPost(OS_Q *p_q,void *p_void,OS_MSG_SIZE msg_size,OS_OPT opt,OS_ERR *p_err)`

# 消息队列
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

OS_Q queue;                             //声明消息队列
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
    /* 创建消息队列 queue */
    OSQCreate ((OS_Q         *)&queue,            //指向消息队列的指针
               (CPU_CHAR     *)"Queue For Test",  //队列的名字
               (OS_MSG_QTY    )20,                //最多可存放消息的数目
               (OS_ERR       *)&err);             //返回错误类型
    /* 创建 AppTaskPost 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskPostTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Post",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPost,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                              //任务的优先级
                 (CPU_STK    *)&AppTaskPostStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 AppTaskPend 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskPendTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Pend",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPend,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                             //任务的优先级
                 (CPU_STK    *)&AppTaskPendStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
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
    while (DEF_TRUE) {                                            //任务体
        /* 发布消息到消息队列 queue */
        OSQPost ((OS_Q        *)&queue,                             //消息变量指针
                 (void        *)"uC/OS-III",                //要发送的数据的指针，将内存块首地址通过队列“发送出去”
                 (OS_MSG_SIZE  )sizeof ( "uC/OS-III" ),     //数据字节大小
                 (OS_OPT       )OS_OPT_POST_FIFO | OS_OPT_POST_ALL, //先进先出和发布给全部任务的形式
                 (OS_ERR      *)&err);	                            //返回错误类型

        OSTimeDlyHMSM ( 0, 0, 0, 500, OS_OPT_TIME_DLY, & err );     //每隔500ms发送一次
    }
}
/**
 * @brief  Pend任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskPend ( void * p_arg )
{
    OS_ERR      err;
    OS_MSG_SIZE msg_size;
    CPU_SR_ALLOC(); //使用到临界段（在关/开中断时）时必需该宏，该宏声明和
    //定义一个局部变量，用于保存关中断前的 CPU 状态寄存器
    // SR（临界段关中断只需保存SR），开中断时将该值还原。
    char * pMsg;
    (void)p_arg;
    while (DEF_TRUE) {                                       //任务体
        /* 请求消息队列 queue 的消息 */
        pMsg = OSQPend ((OS_Q         *)&queue,                //消息变量指针
                        (OS_TICK       )0,                     //等待时长为无限
                        (OS_OPT        )OS_OPT_PEND_BLOCKING,  //如果没有获取到信号量就等待
                        (OS_MSG_SIZE  *)&msg_size,             //获取消息的字节大小
                        (CPU_TS       *)0,                     //获取任务发送时的时间戳
                        (OS_ERR       *)&err);                 //返回错误

        if ( err == OS_ERR_NONE )                              //如果接收成功
        {
            OS_CRITICAL_ENTER();                                 //进入临界段
            printf ( "\r\nlen:%d bytes,data:%s\r\n", msg_size, pMsg );
            OS_CRITICAL_EXIT();
        }
    }
}

```
# 调试
- 编译下载到开发板
- 打开串口助手，可以看到每500ms接收到一次数据，证明消息队列运行正常