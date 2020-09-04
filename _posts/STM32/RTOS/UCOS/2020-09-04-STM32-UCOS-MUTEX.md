---
title: 互斥量UCOS篇6
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS互斥量
---
# 互斥量常用函数
- 创建互斥量`void OSMutexCreate(OS_MUTEX *p_mutex,CPU_CHAR *p_name,OS_ERR *p_err)`
- 删除互斥量`OS_OBJ_QTY OSMutexDel(OS_MUTEX *p_mutex,OS_OPT opt,OS_ERR *p_err)`
- 获取互斥量`void OSMutexPend(OS_MUTEX *p_mutex,OS_TICK timeout,OS_OPT opt,CPU_TS *p_ts,OS_ERR *p_err)`
- 中止获取互斥量`OS_OBJ_QTY OSMutexPendAbort(OS_MUTEX *p_mutex,OS_OPT opt,OS_ERR *p_err)`
- 释放互斥量`void OSMutexPost(OS_MUTEX *p_mutex,OS_OPT opt,OS_ERR *p_err)`

# 模拟优先级翻转
## 程序
- 修改`os_cfg_app.h`中TICK的优先级

```c
#define  OS_CFG_TICK_TASK_PRIO            1u               /* Priority  */
```
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
static  OS_TCB   AppTaskLed3TCB;

/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskLed1Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed2Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed3Stk [ APP_TASK_STK_SIZE ];
/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskLed1  ( void * p_arg );
static  void  AppTaskLed2  ( void * p_arg );
static  void  AppTaskLed3  ( void * p_arg );

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
                (OS_SEM_CTR   )1,             //表示现有资源数目，表示事件还没有发生
                (OS_ERR      *)&err);         //错误类型
    /* 创建 Led1 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed1TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led1",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed1,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+2),                          //任务的优先级
                 (CPU_STK    *)&AppTaskLed1Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
                 
    /* 创建 Led2 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed2TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led2",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed2,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+1),                          //任务的优先级
                 (CPU_STK    *)&AppTaskLed2Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
    /* 创建 Led3 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed3TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led3",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed3,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                              //任务的优先级
                 (CPU_STK    *)&AppTaskLed3Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
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
    static uint32_t i;
    CPU_TS         ts_sem_post;
    (void)p_arg;
    while (DEF_TRUE) {                                            //任务体
        printf("AppTaskLed1 Running!\n");
        //获取二值信号量 xSemaphore,没获取到则一直等待
        OSSemPend ((OS_SEM   *)&SemOfKey,             //等待该信号量被发布
                   (OS_TICK   )0,                     //无期限等待
                   (OS_OPT    )OS_OPT_PEND_BLOCKING,  //如果没有信号量可用就等待
                   (CPU_TS   *)&ts_sem_post,          //获取信号量最后一次被发布的时间戳
                   (OS_ERR   *)&err);                 //返回错误类型
        printf("AppTaskLed1 Pend!\n");
        for(i=0; i<600000; i++)    //模拟低优先级任务占用信号量
        {
            OSSched();//发起任务调度
        }
        printf("AppTaskLed1 Post!\n");
        OSSemPost((OS_SEM  *)&SemOfKey,                                        //发布SemOfKey
                  (OS_OPT   )OS_OPT_POST_1,                                   //发布给所有等待任务
                  (OS_ERR  *)&err);
        LED1_TOGGLE;
        OSTimeDlyHMSM (0,0,1,0,OS_OPT_TIME_PERIODIC,&err);
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
    while (DEF_TRUE) {                                       //任务体
        printf("AppTaskLed2 Running\n");

        OSTimeDlyHMSM (0,0,0,200,OS_OPT_TIME_PERIODIC,&err);
    }
}
/**
 * @brief  LED3任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed3 ( void * p_arg )
{
    OS_ERR      err;
	  CPU_TS         ts_sem_post;
   (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */
			
      printf("AppTaskLed3 Running!\n");	
      //获取二值信号量 xSemaphore,没获取到则一直等待
			OSSemPend ((OS_SEM   *)&SemOfKey,             //等待该信号量被发布
								 (OS_TICK   )0,                     //无期限等待
								 (OS_OPT    )OS_OPT_PEND_BLOCKING,  //如果没有信号量可用就等待
								 (CPU_TS   *)&ts_sem_post,          //获取信号量最后一次被发布的时间戳
								 (OS_ERR   *)&err);                 //返回错误类型
			printf("AppTaskLed3 Pend!\n");	
      LED2_TOGGLE;   
      printf("AppTaskLed3 Post!\n");
      //给出二值信号量
		  OSSemPost((OS_SEM  *)&SemOfKey,                                        //发布SemOfKey
							 (OS_OPT   )OS_OPT_POST_1,                                 
							 (OS_ERR  *)&err);           
			OSTimeDlyHMSM (0,0,1,0,OS_OPT_TIME_PERIODIC,&err);      
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
- 可以看到高优先级任务在等待低优先级任务运行完毕才能得到信号量继续运行，中优先级一直在打断低优先级任务

# 互斥量
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

OS_MUTEX TestMutex;
/*
*********************************************************************************************************
*                                                 TCB
*********************************************************************************************************
*/
static  OS_TCB   AppTaskStartTCB;

static  OS_TCB   AppTaskLed1TCB;
static  OS_TCB   AppTaskLed2TCB;
static  OS_TCB   AppTaskLed3TCB;

/*
*********************************************************************************************************
*                                                STACKS
*********************************************************************************************************
*/
static  CPU_STK  AppTaskStartStk[APP_TASK_START_STK_SIZE];

static  CPU_STK  AppTaskLed1Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed2Stk [ APP_TASK_STK_SIZE ];
static  CPU_STK  AppTaskLed3Stk [ APP_TASK_STK_SIZE ];
/*
*********************************************************************************************************
*                                         FUNCTION PROTOTYPES
*********************************************************************************************************
*/
static  void  AppTaskStart(void *p_arg);

static  void  AppTaskLed1  ( void * p_arg );
static  void  AppTaskLed2  ( void * p_arg );
static  void  AppTaskLed3  ( void * p_arg );

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
    /* 创建互斥信号量 mutex */
    OSMutexCreate ((OS_MUTEX  *)&TestMutex,           //指向信号量变量的指针
                   (CPU_CHAR  *)"Mutex For Test", //信号量的名字
                   (OS_ERR    *)&err);            //错误类型
    /* 创建 Led1 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed1TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led1",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed1,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+2),                          //任务的优先级
                 (CPU_STK    *)&AppTaskLed1Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 Led2 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed2TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led2",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed2,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+1),                          //任务的优先级
                 (CPU_STK    *)&AppTaskLed2Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型
    /* 创建 Led3 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskLed3TCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Led3",                             //任务名称
                 (OS_TASK_PTR ) AppTaskLed3,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) APP_TASK_PRIO,                              //任务的优先级
                 (CPU_STK    *)&AppTaskLed3Stk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
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
    static uint32_t i;
    (void)p_arg;
    while (DEF_TRUE) {                                            //任务体
        printf("AppTaskLed1 Running!\n");
        //获取 互斥量 ,没获取到则一直等待
        OSMutexPend ((OS_MUTEX  *)&TestMutex,                  //申请互斥信号量 mutex
                     (OS_TICK    )0,                       //无期限等待
                     (OS_OPT     )OS_OPT_PEND_BLOCKING,    //如果不能申请到信号量就堵塞任务
                     (CPU_TS    *)0,                       //不想获得时间戳
                     (OS_ERR    *)&err);                   //返回错误类型
        //返回错误类型
        printf("AppTaskLed1 Pend!\n");
        for(i=0; i<600000; i++)    //模拟低优先级任务占用信号量
        {
            OSSched();//发起任务调度
        }
        printf("AppTaskLed1 Post!\n");
        OSMutexPost ((OS_MUTEX  *)&TestMutex,                  //释放互斥信号量 mutex
                     (OS_OPT     )OS_OPT_POST_NONE,        //进行任务调度
                     (OS_ERR    *)&err);                   //返回错误类型
        LED1_TOGGLE;
        OSTimeDlyHMSM (0,0,1,0,OS_OPT_TIME_PERIODIC,&err);
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
    while (DEF_TRUE) {                                       //任务体
        printf("AppTaskLed2 Running\n");

        OSTimeDlyHMSM (0,0,0,200,OS_OPT_TIME_PERIODIC,&err);
    }
}
/**
 * @brief  LED3任务主体
 * @param  p_arg 参数
 * @retval 无
 */
static  void  AppTaskLed3 ( void * p_arg )
{
    OS_ERR      err;
    (void)p_arg;
    while (DEF_TRUE) {                                          /* Task body, always written as an infinite loop.       */

        printf("AppTaskLed3 Running!\n");
        //获取互斥量,没获取到则一直等待
        OSMutexPend ((OS_MUTEX  *)&TestMutex,                  //申请互斥信号量 mutex
                     (OS_TICK    )0,                       //无期限等待
                     (OS_OPT     )OS_OPT_PEND_BLOCKING,    //如果不能申请到信号量就堵塞任务
                     (CPU_TS    *)0,                       //不想获得时间戳
                     (OS_ERR    *)&err);                   //返回错误类型
        printf("AppTaskLed3 Pend!\n");
        LED2_TOGGLE;
        printf("AppTaskLed3 Post!\n");
        OSMutexPost ((OS_MUTEX  *)&TestMutex,                  //释放互斥信号量 mutex
                     (OS_OPT     )OS_OPT_POST_NONE,        //进行任务调度
                     (OS_ERR    *)&err);                   //返回错误类型
        OSTimeDlyHMSM (0,0,1,0,OS_OPT_TIME_PERIODIC,&err);
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
- 可以看到在低优先级任务运行的时候，中优先级任务无法抢占低优先级的任务