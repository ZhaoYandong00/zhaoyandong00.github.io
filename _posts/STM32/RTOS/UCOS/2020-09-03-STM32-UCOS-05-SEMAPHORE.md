---
title: 信号量UCOS篇5
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS信号量
---
# 信号量常用函数
- 创建信号量`void OSSemCreate(OS_SEM *p_sem,CPU_CHAR *p_name,OS_SEM_CTR cnt,OS_ERR *p_err)`
- 删除信号量`OS_OBJ_QTY OSSemDel(OS_SEM *p_sem,OS_OPT opt,OS_ERR *p_err)`
- 获取信号量`OS_SEM_CTR OSSemPend(OS_SEM *p_sem,OS_TICK timeout,OS_OPT opt,CPU_TS *p_ts,OS_ERR *p_err)`
- 中止获取信号量`OS_OBJ_QTY OSSemPendAbort(OS_SEM *p_sem,OS_OPT opt,OS_ERR *p_err)`
- 发送信号量`OS_SEM_CTR OSSemPost(OS_SEM *p_sem,OS_OPT opt,OS_ERR *p_err)`
- 设置信号量计数值`void OSSemSet(OS_SEM *p_sem,OS_SEM_CTR cnt,OS_ERR *p_err)`

# 二值信号量
## 修改配置
- 修改`os_cfg.h`打开时间戳

```c
#define OS_CFG_TS_EN                    1u   /* Enable (1) or Disable (0) time stamping                               */
```
- 修改`cpu_cfg.h`

```c
#define  CPU_CFG_TS_32_EN                       DEF_ENABLED
```
- 实现`CPU_TS_TmrInit()`和`CPU_TS_TmrRd()`函数

```c
#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
void  CPU_TS_TmrInit (void)
{
    CoreDebug->DEMCR|=CoreDebug_DEMCR_TRCENA_Msk;//使能DWT
    DWT->CYCCNT=0; //DWT计数归0
    DWT->CTRL|=DWT_CTRL_CYCCNTENA_Msk; //开始计数
    CPU_TS_TmrFreqSet(SystemCoreClock); //设置时钟
}
CPU_TS_TMR  CPU_TS_TmrRd (void)
{
   return DWT->CYCCNT;
}
#endif
```
## 程序
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

static  OS_TCB   AppTaskKeyTCB;
static  OS_TCB   AppTaskLed1TCB;
/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskKeyStk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed1Stk [ APP_TASK_STK_SIZE ];
/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskKey  ( void * p_arg );
static  void  AppTaskLed1 ( void * p_arg );

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
                (OS_SEM_CTR   )0,             //信号量这里是指示事件发生，所以赋值为0，表示事件还没有发生
                (OS_ERR      *)&err);         //错误类型
    /* 创建 AppTaskPost 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskKeyTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Post",                             //任务名称
                 (OS_TASK_PTR ) AppTaskKey,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                              //任务的优先级
                 (CPU_STK    *)&AppTaskKeyStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 AppTaskPend 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed1TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Pend",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed1,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                             //任务的优先级
                 (CPU_STK    *)&AppTaskLed1Stk[0],                          //任务堆栈的基地址
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
 * @brief  Key任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskKey ( void * p_arg )
{
    OS_ERR      err;
    (void)p_arg;
    while (DEF_TRUE) {                                            //任务体
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON ) //如果KEY1被单击
            OSSemPost((OS_SEM  *)&SemOfKey,                                        //发布SemOfKey
                      (OS_OPT   )OS_OPT_POST_ALL,                                   //发布给所有等待任务
                      (OS_ERR  *)&err);                                             //返回错误类型

        OSTimeDlyHMSM ( 0, 0, 0, 20, OS_OPT_TIME_DLY, & err );                   //每20ms扫描一次
    }
}
/**
 * @brief  Key任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed1 ( void * p_arg )
{
    OS_ERR      err;
    CPU_TS         ts_sem_post, ts_sem_get;
    CPU_SR_ALLOC(); //使用到临界段（在关/开中断时）时必需该宏，该宏声明和
    //定义一个局部变量，用于保存关中断前的 CPU 状态寄存器
    // SR（临界段关中断只需保存SR），开中断时将该值还原。
    (void)p_arg;
    while (DEF_TRUE) {                                       //任务体
        OSSemPend ((OS_SEM   *)&SemOfKey,             //等待该信号量被发布
                   (OS_TICK   )0,                     //无期限等待
                   (OS_OPT    )OS_OPT_PEND_BLOCKING,  //如果没有信号量可用就等待
                   (CPU_TS   *)&ts_sem_post,          //获取信号量最后一次被发布的时间戳
                   (OS_ERR   *)&err);                 //返回错误类型
        ts_sem_get = OS_TS_GET();                     //获取解除等待时的时间戳

        LED1_TOGGLE;                            //切换LED1的亮灭状态

        OS_CRITICAL_ENTER();                          //进入临界段，不希望下面串口打印遭到中断

        printf ( "\r\npost time:%d", ts_sem_post );
        printf ( "\r\nget time:%d", ts_sem_get );
        printf ( "\r\ntime: %dus\r\n",
                 ( ts_sem_get - ts_sem_post ) / ( SystemCoreClock / 1000000 ) );

        OS_CRITICAL_EXIT();
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
## 调试
- 编译下载到开发板
- 按下开发板的按键，串口打印任务运行的信息，表明两个任务同步成功

# 计数信号量
## 程序
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

static  OS_TCB   AppTaskKey1TCB;
static  OS_TCB   AppTaskKey2TCB;
/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskKey1Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskKey2Stk [ APP_TASK_STK_SIZE ];
/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskKey1  ( void * p_arg );
static  void  AppTaskKey2 ( void * p_arg );

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
                (OS_SEM_CTR   )5,             //表示现有资源数目，表示事件还没有发生
                (OS_ERR      *)&err);         //错误类型
    /* 创建 AppTaskPost 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskKey1TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Post",                             //任务名称
                 (OS_TASK_PTR ) AppTaskKey1,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                              //任务的优先级
                 (CPU_STK    *)&AppTaskKey1Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 AppTaskPend 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskKey2TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Pend",                             //任务名称
                 (OS_TASK_PTR ) AppTaskKey2,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                             //任务的优先级
                 (CPU_STK    *)&AppTaskKey2Stk[0],                          //任务堆栈的基地址
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
 * @brief  Key任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskKey1 ( void * p_arg )
{
    OS_ERR      err;
    OS_SEM_CTR  ctr;
    CPU_SR_ALLOC();//使用到临界段（在关/开中断时）时必需该宏
    (void)p_arg;
    while (DEF_TRUE) {                                            //任务体
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON ) //如果KEY1被按下
        {
            ctr = OSSemPend ((OS_SEM   *)&SemOfKey,               //等待该信号量 SemOfKey
                             (OS_TICK   )0,                       //下面选择不等待，该参无效
                             (OS_OPT    )OS_OPT_PEND_NON_BLOCKING,//如果没信号量可用不等待
                             (CPU_TS   *)0,                       //不获取时间戳
                             (OS_ERR   *)&err);                   //返回错误类型

            OS_CRITICAL_ENTER();
            //进入临界段                                           //返回错误类型
            if ( err == OS_ERR_NONE )
                printf ( "\r\nKEY1 Press, %d\r\n", ctr );
            else if ( err == OS_ERR_PEND_WOULD_BLOCK )
                printf ( "\r\nKEY1 Press,fail\r\n" );

            OS_CRITICAL_EXIT();
        }

        OSTimeDlyHMSM ( 0, 0, 0, 20, OS_OPT_TIME_DLY, & err );                   //每20ms扫描一次
    }
}
/**
 * @brief  Key任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskKey2 ( void * p_arg )
{
    OS_ERR      err;
    OS_SEM_CTR  ctr;
    CPU_SR_ALLOC(); //使用到临界段（在关/开中断时）时必需该宏，该宏声明和
    //定义一个局部变量，用于保存关中断前的 CPU 状态寄存器
    // SR（临界段关中断只需保存SR），开中断时将该值还原。
    (void)p_arg;
    while (DEF_TRUE) {                                       //任务体
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON ) //如果KEY2被按下
        {
            ctr = OSSemPost((OS_SEM  *)&SemOfKey,                                  //发布SemOfKey
                            (OS_OPT   )OS_OPT_POST_ALL,                            //发布给所有等待任务
                            (OS_ERR  *)&err);
            OS_CRITICAL_ENTER();                                                   //进入临界段

            printf ( "\r\nKEY2 Press:%d\r\n", ctr );

            OS_CRITICAL_EXIT();
        }
        OSTimeDlyHMSM ( 0, 0, 0, 20, OS_OPT_TIME_DLY, & err );                    //每20ms扫描一次
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
## 调试
- 编译下载到开发板
- 串口助手可看到当按下5次KEY1,再按就会出现错误，这是先按下KEY2,再按KEY1，成功