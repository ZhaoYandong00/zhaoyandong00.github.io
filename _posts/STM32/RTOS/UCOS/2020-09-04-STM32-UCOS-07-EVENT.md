---
title: 事件UCOS篇7
categories: STM32 RTOS UCOS
tags: STM32 RTOS UCOS
description: UCOS事件
---
# 事件常用函数
- 创建事件`void OSFlagCreate(OS_FLAG_GRP *p_grp,CPU_CHAR *p_name,OS_FLAGS flags,OS_ERR *p_err)`
- 删除事件`OS_OBJ_QTY OSFlagDel(OS_FLAG_GRP *p_grp,OS_OPT opt,OS_ERR *p_err)`
- 读取事件`OS_FLAGS OSFlagPend(OS_FLAG_GRP *p_grp,OS_FLAGS flags,OS_TICK timeout,OS_OPT opt,CPU_TS *p_ts,OS_ERR *p_err)`
- 中止读取事件`OS_OBJ_QTY OSFlagPendAbort(OS_FLAG_GRP *p_grp,OS_OPT opt,OS_ERR *p_err)`
- 读取事件准备运行标志`OS_FLAGS OSFlagPendGetFlagsRdy(OS_ERR *p_err)`
- 设置事件`OS_FLAGS OSFlagPost(OS_FLAG_GRP *p_grp,OS_FLAGS flags,OS_OPT opt,OS_ERR *p_err)`

# 事件
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

OS_FLAG_GRP flag_grp;                   //声明事件标志组

#define KEY1_EVENT  (0x01 << 0)//设置事件掩码的位0
#define KEY2_EVENT  (0x01 << 1)//设置事件掩码的位1
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
    /* 创建事件标志组 flag_grp */
    OSFlagCreate ((OS_FLAG_GRP  *)&flag_grp,        //指向事件标志组的指针
                  (CPU_CHAR     *)"FLAG For Test",  //事件标志组的名字
                  (OS_FLAGS      )0,                //事件标志组的初始值
                  (OS_ERR       *)&err);					  //返回错误类型
    /* 创建 Led1 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskPostTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Post",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPost,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+2),                          //任务的优先级
                 (CPU_STK    *)&AppTaskPostStk[0],                          //任务堆栈的基地址
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE / 10,                     //任务堆栈空间剩下1/10时限制其增长
                 (CPU_STK_SIZE) APP_TASK_STK_SIZE,                          //任务堆栈空间（单位：sizeof(CPU_STK)）
                 (OS_MSG_QTY  ) 5u,                                         //任务可接收的最大消息数
                 (OS_TICK     ) 0u,                                         //任务的时间片节拍数（0表默认值OSCfg_TickRate_Hz/10）
                 (void       *) 0,                                          //任务扩展（0表不扩展）
                 (OS_OPT      )(OS_OPT_TASK_STK_CHK | OS_OPT_TASK_STK_CLR), //任务选项
                 (OS_ERR     *)&err);                                       //返回错误类型

    /* 创建 Led2 任务 */
    OSTaskCreate((OS_TCB     *)&AppTaskPendTCB,                             //任务控制块地址
                 (CPU_CHAR   *)"App Task Pend",                             //任务名称
                 (OS_TASK_PTR ) AppTaskPend,                                //任务函数
                 (void       *) 0,                                          //传递给任务函数（形参p_arg）的实参
                 (OS_PRIO     ) (APP_TASK_PRIO+1),                          //任务的优先级
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
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON ) //如果KEY1被按下
        {   //点亮LED1
            printf("KEY1 Press\n");
            OSFlagPost ((OS_FLAG_GRP  *)&flag_grp,                             //将标志组的BIT0置1
                        (OS_FLAGS      )KEY1_EVENT,
                        (OS_OPT        )OS_OPT_POST_FLAG_SET,
                        (OS_ERR       *)&err);

        }

        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON ) //如果KEY2被按下
        {   //点亮LED2
            printf("KEY2 Press\n");
            OSFlagPost ((OS_FLAG_GRP  *)&flag_grp,                             //将标志组的BIT1置1
                        (OS_FLAGS      )KEY2_EVENT,
                        (OS_OPT        )OS_OPT_POST_FLAG_SET,
                        (OS_ERR       *)&err);

        }

        OSTimeDlyHMSM ( 0, 0, 0, 20, OS_OPT_TIME_DLY, & err );  //每20ms扫描一次
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
    OS_FLAGS      flags_rdy;
    (void)p_arg;
    while (DEF_TRUE) {                                       //任务体
        //等待标志组的的BIT0和BIT1均被置1
        flags_rdy =   OSFlagPend ((OS_FLAG_GRP *)&flag_grp,
                                  (OS_FLAGS     )( KEY1_EVENT | KEY2_EVENT ),
                                  (OS_TICK      )0,
                                  (OS_OPT       )OS_OPT_PEND_FLAG_SET_ALL |
                                  OS_OPT_PEND_BLOCKING |
                                  OS_OPT_PEND_FLAG_CONSUME,
                                  (CPU_TS      *)0,
                                  (OS_ERR      *)&err);
        if((flags_rdy & (KEY1_EVENT|KEY2_EVENT)) == (KEY1_EVENT|KEY2_EVENT))
        {
            /* 如果接收完成并且正确 */
            printf ( "KEY1 KEY2 All\n");
            LED1_TOGGLE;       //LED1	反转
        }
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
- 按下KEY1 按键发送事件1,按下KEY2,按键发送事件2
- 我们按下KEY1与KEY2试试,在串口调试助手中可以看到运行结果,并且当事件1与事件2都发生的时候,开发板的LED会进行翻转