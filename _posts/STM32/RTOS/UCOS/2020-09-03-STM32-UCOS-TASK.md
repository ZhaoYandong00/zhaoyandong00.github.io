---
title: 创建任务UCOS篇2
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS创建任务
---
# 任务常用函数
- OS初始化`void OSInit(OS_ERR *p_err)`
- OS开始`void OSStart(OS_ERR *p_err)`
- CPU初始化`void CPU_Init(void)`
- 内存初始化`void Mem_Init(void)`
- 初始化系统时钟`void OS_CPU_SysTickInit(CPU_INT32U cnts)`
- CPU使用率初始化`void OSStatTaskCPUUsageInit(OS_ERR *p_err)`
- 重置最大关中断时间`CPU_TS_TMR CPU_IntDisMeasMaxCurReset(void)`
- 延时`void OSTimeDly(OS_TICK ly,OS_OPT opt,OS_ERR *p_err)`
- 创建任务

```c
void OSTaskCreate(OS_TCB      *p_tcb,
                  CPU_CHAR    *p_name,
                  OS_TASK_PTR  p_task, 
                  void        *p_arg,
                  OS_PRIO      prio,
                  CPU_STK     *p_stk_base,
                  CPU_STK_SIZE stk_limit,
                  CPU_STK_SIZE stk_size,
                  OS_MSG_QTY   q_size,
                  OS_TICK      time_quanta,
                  void        *p_ext,
                  OS_OPT       opt,
                  OS_ERR       *p_err)
```
- 更改任务优先级`void OSTaskChangePrio (OS_TCB *p_tcb,OS_PRIO prio_new,OS_ERR *p_err)`
- 删除任务`void OSTaskDel(OS_TCB *p_tcb,OS_ERR *p_err)`
- 更新任务内部信息`OS_MSG_QTY OSTaskQFlush(OS_TCB *p_tcb,OS_ERR *p_err)`
- 等待消息`void *OSTaskQPend(OS_TICK timeout,OS_OPT opt,OS_MSG_SIZE *p_msg_size,CPU_TS *p_ts,OS_ERR p_err)`
- 中止等待消息`CPU_BOOLEAN OSTaskQPendAbort(OS_TCB *p_tcb,OS_OPT opt,OS_ERR *p_err)`
- 向任务发送消息`void OSTaskQPost(OS_TCB *p_tcb,void *p_void,OS_MSG_SIZE msg_size,OS_OPT opt,OS_ERR *p_err)`
- 获取任务寄存器值`OS_REG OSTaskRegGet(OS_TCB *p_tcb,OS_REG_ID id,OS_ERR *p_err)`
- 获取任务ID`OS_REG_ID OSTaskRegGetID(OS_ERR *p_err)`
- 设置任务寄存器值`void OSTaskRegSet(OS_TCB *p_tcb,OS_REG_ID id,OS_REG value,OS_ERR *p_err)`
- 恢复任务`void OSTaskResume(OS_TCB *p_tcb,OS_ERR *p_err)`
- 挂起任务`void OSTaskSuspend(OS_TCB *p_tcb,OS_ERR *p_err)`
- 接收信号`OS_SEM_CTR OSTaskSemPend(OS_TICK timeout,OS_OPT opt,CPU_TS *p_ts,OS_ERR *p_err)`
- 中止接收信号`CPU_BOOLEAN OSTaskSemPendAbort(OS_TCB *p_tcb,OS_OPT opt,OS_ERR *p_err)`
- 发送信号`OS_SEM_CTR OSTaskSemPost(OS_TCB *p_tcb,OS_OPT opt,OS_ERR *p_err)`
- 清除信号计数器`OS_SEM_CTR OSTaskSemSet (OS_TCB *p_tcb,OS_SEM_CTR cnt,OS_ERR *p_err)`
- 查询任务堆栈`void OSTaskStkChk(OS_TCB *p_tcb,CPU_STK_SIZE *p_free,CPU_STK_SIZE *p_used,OS_ERR *p_err)`

# 修改配置
- `os_cfg.h`打开任务删除

```c
define OS_CFG_TASK_DEL_EN              1u   /* Include code for OSTaskDel()    
```
- `cpu_cfg.h`中打开前导零函数

```c
#if 1                                                           /* Configure CPU count leading  zeros bits ...          */
#define  CPU_CFG_LEAD_ZEROS_ASM_PRESENT                         /* ... assembly-version (see Note #1a).                 */
#endif
```
- `os_cfg_app.h`修改堆栈大小

```c

                                                            /* --------------------- MISCELLANEOUS ------------------ */
#define  OS_CFG_MSG_POOL_SIZE            100u               /* Maximum number of messages                             */

#define  OS_CFG_ISR_STK_SIZE             128u               /* Stack size of ISR stack (number of CPU_STK elements)   */

#define  OS_CFG_TASK_STK_LIMIT_PCT_EMPTY  10u               /* Stack limit position in percentage to empty            */


                                                            /* ---------------------- IDLE TASK --------------------- */
#define  OS_CFG_IDLE_TASK_STK_SIZE       128u               /* Stack size (number of CPU_STK elements)                */


                                                            /* ------------------ ISR HANDLER TASK ------------------ */
#define  OS_CFG_INT_Q_SIZE                10u               /* Size of ISR handler task queue                         */
#define  OS_CFG_INT_Q_TASK_STK_SIZE      128u               /* Stack size (number of CPU_STK elements)                */


                                                            /* ------------------- STATISTIC TASK ------------------- */
#define  OS_CFG_STAT_TASK_PRIO            11u               /* Priority                                               */
#define  OS_CFG_STAT_TASK_RATE_HZ         10u               /* Rate of execution (1 to 10 Hz)                         */
#define  OS_CFG_STAT_TASK_STK_SIZE       128u               /* Stack size (number of CPU_STK elements)                */


                                                            /* ------------------------ TICKS ----------------------- */
#define  OS_CFG_TICK_RATE_HZ            1000u               /* Tick rate in Hertz (10 to 1000 Hz)                     */
#define  OS_CFG_TICK_TASK_PRIO            10u               /* Priority                                               */
#define  OS_CFG_TICK_TASK_STK_SIZE       512u               /* Stack size (number of CPU_STK elements)                */


                                                            /* ----------------------- TIMERS ----------------------- */
#define  OS_CFG_TMR_TASK_PRIO             11u               /* Priority of 'Timer Task'                               */
#define  OS_CFG_TMR_TASK_RATE_HZ          10u               /* Rate for timers (10 Hz Typ.)                           */
#define  OS_CFG_TMR_TASK_STK_SIZE        128u               /* Stack size (number of CPU_STK elements)                */
 
```

- `app_cfg.h`添加配置

```c
#ifndef  __APP_CFG_H__
#define  __APP_CFG_H__
#define  APP_TASK_START_STK_SIZE   128
#define  APP_TASK_START_PRIO       2

#endif

```
- `startup_stm32f10x_hd.s`修改以下两处

```nasm
;                DCD     OS_CPU_PendSVHandler             ; PendSV Handler
;                DCD     OS_CPU_SysTickHandler           ; SysTick Handler
                DCD     OS_CPU_PendSVHandler             ; PendSV Handler
                DCD     OS_CPU_SysTickHandler           ; SysTick Handler
```

```nasm
;PendSV_Handler  PROC
;                EXPORT  PendSV_Handler             [WEAK]
;                B       .
;                ENDP
;SysTick_Handler PROC
;                EXPORT  SysTick_Handler            [WEAK]
;                B       .
;                ENDP
OS_CPU_PendSVHandler  PROC
                EXPORT  OS_CPU_PendSVHandler             [WEAK]
                B       .
                ENDP
OS_CPU_SysTickHandler PROC
                EXPORT  OS_CPU_SysTickHandler            [WEAK]
                B       .
                ENDP
```

# 单任务
## 程序
- 把FREERTOS写的`my_gpio.c`,`my_gpio.h`,`my_uasrt.c`,`my_usart.h`复制到工程
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

static  OS_TCB   AppTaskStartTCB;

static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  void  AppTaskStart(void *p_arg);

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
    //USART_Config();
    /* 按键初始化 */
    Key_GPIO_Config();
}
/**
 * @brief  任务
 * @param  无
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
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        LED1_TOGGLE;
        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err );
    }
}

```
## 调试
- 编译下载到开发板
- 可以看到LED1闪烁

# 多任务
## 程序
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
/*
*********************************************************************************************************
*                                            TASK STACK SIZES
*                             Size of the task stacks (# of OS_STK entries)
*********************************************************************************************************
*/
#define  APP_TASK_START_STK_SIZE                    128

#define  APP_TASK_LED1_STK_SIZE                     512
#define  APP_TASK_LED2_STK_SIZE                     512
#endif

```
- 修改`main.c

```c
#include "os.h"
#include  "lib_ascii.h"
#include  "lib_math.h"
#include  "lib_mem.h"
#include  "lib_str.h"
#include "app_cfg.h"
#include "my_gpio.h"
#include "my_usart.h"

static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskLed1TCB;
static  OS_TCB   AppTaskLed2TCB;

static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskLed1Stk [ APP_TASK_LED1_STK_SIZE ];
static  CPU_STK  AppTaskLed2Stk [ APP_TASK_LED2_STK_SIZE ];

static  void  AppTaskStart(void *p_arg);

static  void  AppTaskLed1  ( void * p_arg );
static  void  AppTaskLed2  ( void * p_arg );

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
    //USART_Config();
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
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        LED1_TOGGLE;
        OSTimeDly ( 1000, OS_OPT_TIME_DLY, & err );
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
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
        LED2_TOGGLE;
        OSTimeDly ( 5000, OS_OPT_TIME_DLY, & err );
    }
}

```
## 调试
- 编译下载到开发板
- 可以看到LED1以1秒闪烁，LED2以5秒闪烁