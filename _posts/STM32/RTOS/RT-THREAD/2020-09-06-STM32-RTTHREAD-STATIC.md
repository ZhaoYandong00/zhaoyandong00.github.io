---
title: 静态内存创建线程RT-Thread篇2
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread静态内存创建线程
---
# 静态内存创建线程
## 程序
- 把UCOS写的`my_gpio.c`,`my_gpio.h`,`my_uasrt.c`,`my_usart.h`复制到工程
- 修改`main.c`

```c
#include "rtthread.h"
#include "my_gpio.h"
#include "my_usart.h"

/*
*************************************************************************
*                               变量
*************************************************************************
*/
/* 定义线程控制块 */
static struct rt_thread led1_thread;

/* 定义线程控栈时要求RT_ALIGN_SIZE个字节对齐 */
ALIGN(RT_ALIGN_SIZE)
/* 定义线程栈 */
static rt_uint8_t rt_led1_thread_stack[1024];
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void led1_thread_entry(void* parameter);


static void BSP_Init(void);/* 用于初始化板载相关资源 */

int main(void)
{
    BSP_Init();
    rt_thread_init(&led1_thread,                 /* 线程控制块 */
                   "led1",                       /* 线程名字 */
                   led1_thread_entry,            /* 线程入口函数 */
                   RT_NULL,                      /* 线程入口函数参数 */
                   &rt_led1_thread_stack[0],     /* 线程栈起始地址 */
                   sizeof(rt_led1_thread_stack), /* 线程栈大小 */
                   3,                            /* 线程的优先级 */
                   20);                          /* 线程时间片 */
    rt_thread_startup(&led1_thread);             /* 启动线程，开启调度 */
}
/**
 * @brief  led1_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void led1_thread_entry(void* parameter)
{
    while (1)
    {
        LED1(ON);
        rt_thread_delay(500);   /* 延时500个tick */

        LED1(OFF);
        rt_thread_delay(500);   /* 延时500个tick */

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
}

```

## 调试
- 编译下载到开发板
- 可以看到LED1闪烁