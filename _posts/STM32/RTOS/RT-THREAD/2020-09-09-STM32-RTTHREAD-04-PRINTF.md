---
title: 重映射串口到rt_kprintf函数RT-Thread篇4
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread重映射串口到rt_kprintf函数
---
# 实现输出函数
- 要想使用`rt_kprintf`,需要实现`rt_hw_console_output`函数

```c
/**
  * @brief  重映射串口DEBUG_USARTx到rt_kprintf()函数
  * @param  str  要输出到串口的字符串
  * @retval 无
  */
void rt_hw_console_output(const char *str)
{
    /* 进入临界段 */
    rt_enter_critical();

    /* 直到字符串结束 */
    while (*str!='\0')
    {
        /* 换行 */
        if (*str=='\n')
        {
            USART_SendData(DEBUG_USARTx, '\r');
            while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
        }

        USART_SendData(DEBUG_USARTx, *str++);
        while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
    }

    /* 退出临界段 */
    rt_exit_critical();
}
```
#  修改程序框架
- 把硬件初始化放到`bord.c`的硬件初始化函数中
- 如果硬件初始化放到`main.c`函数，会造成系统在串口没有设置时，调用`rt_kprintf()`函数，从而调用串口，导致硬件错误

```c
/**
 * This function will initial your board.
 */
void rt_hw_board_init()
{
    /* System Clock Update */
    SystemCoreClockUpdate();

    /* System Tick Configuration */
    _SysTick_Config(SystemCoreClock / RT_TICK_PER_SECOND);
    /*硬件初始化*/
    BSP_Init();
    /* Call components board initial (use INIT_BOARD_EXPORT()) */
#ifdef RT_USING_COMPONENTS_INIT
    rt_components_board_init();
#endif

#if defined(RT_USING_USER_MAIN) && defined(RT_USING_HEAP)
    rt_system_heap_init(rt_heap_begin_get(), rt_heap_end_get());
#endif
}
```
# 测试rt_kprintf函数
- `main.h`

```c
#ifndef __MAIN_H
#define	__MAIN_H

#include <rtthread.h>
#include "my_gpio.h"
#include "my_usart.h"

#endif

```
- `bord.c`

```c
/*
 * Copyright (c) 2006-2019, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2017-07-24     Tanek        the first version
 * 2018-11-12     Ernest Chen  modify copyright
 */

#include <stdint.h>
#include <rthw.h>
#include <rtthread.h>
#include "main.h"


#define _SCB_BASE       (0xE000E010UL)
#define _SYSTICK_CTRL   (*(rt_uint32_t *)(_SCB_BASE + 0x0))
#define _SYSTICK_LOAD   (*(rt_uint32_t *)(_SCB_BASE + 0x4))
#define _SYSTICK_VAL    (*(rt_uint32_t *)(_SCB_BASE + 0x8))
#define _SYSTICK_CALIB  (*(rt_uint32_t *)(_SCB_BASE + 0xC))
#define _SYSTICK_PRI    (*(rt_uint8_t  *)(0xE000ED23UL))

// Updates the variable SystemCoreClock and must be called
// whenever the core clock is changed during program execution.
extern void SystemCoreClockUpdate(void);

// Holds the system core clock, which is the system clock
// frequency supplied to the SysTick timer and the processor
// core clock.
extern uint32_t SystemCoreClock;

static uint32_t _SysTick_Config(rt_uint32_t ticks)
{
    if ((ticks - 1) > 0xFFFFFF)
    {
        return 1;
    }

    _SYSTICK_LOAD = ticks - 1;
    _SYSTICK_PRI = 0xFF;
    _SYSTICK_VAL  = 0;
    _SYSTICK_CTRL = 0x07;

    return 0;
}

#if defined(RT_USING_USER_MAIN) && defined(RT_USING_HEAP)
#define RT_HEAP_SIZE 1024
static uint32_t rt_heap[RT_HEAP_SIZE];     // heap default size: 4K(1024 * 4)
RT_WEAK void *rt_heap_begin_get(void)
{
    return rt_heap;
}

RT_WEAK void *rt_heap_end_get(void)
{
    return rt_heap + RT_HEAP_SIZE;
}
#endif
static void BSP_Init(void);/* 用于初始化板载相关资源 */
/**
 * This function will initial your board.
 */
void rt_hw_board_init()
{
    /* System Clock Update */
    SystemCoreClockUpdate();

    /* System Tick Configuration */
    _SysTick_Config(SystemCoreClock / RT_TICK_PER_SECOND);
    BSP_Init();
    /* Call components board initial (use INIT_BOARD_EXPORT()) */
#ifdef RT_USING_COMPONENTS_INIT
    rt_components_board_init();
#endif

#if defined(RT_USING_USER_MAIN) && defined(RT_USING_HEAP)
    rt_system_heap_init(rt_heap_begin_get(), rt_heap_end_get());
#endif
}

void SysTick_Handler(void)
{
    /* enter interrupt */
    rt_interrupt_enter();

    rt_tick_increase();

    /* leave interrupt */
    rt_interrupt_leave();
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
  * @brief  重映射串口DEBUG_USARTx到rt_kprintf()函数
  * @param  str  要输出到串口的字符串
  * @retval 无
  *
  * @attention
  *
  */
void rt_hw_console_output(const char *str)
{
    /* 进入临界段 */
    rt_enter_critical();

    /* 直到字符串结束 */
    while (*str!='\0')
    {
        /* 换行 */
        if (*str=='\n')
        {
            USART_SendData(DEBUG_USARTx, '\r');
            while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
        }

        USART_SendData(DEBUG_USARTx, *str++);
        while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
    }

    /* 退出临界段 */
    rt_exit_critical();
}

```

- `main.c`

```c
#include "rtthread.h"
#include "main.h"

/*
*************************************************************************
*                               变量
*************************************************************************
*/
/* 定义线程控制块 */
static rt_thread_t led1_thread = RT_NULL;
static rt_thread_t led2_thread = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void led1_thread_entry(void* parameter);
static void led2_thread_entry(void* parameter);

int main(void)
{
    led1_thread =                          /* 线程控制块指针 */
        rt_thread_create( "led1",              /* 线程名字 */
                          led1_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (led1_thread != RT_NULL)
        rt_thread_startup(led1_thread);
    else
        return -1;
    led2_thread =                          /* 线程控制块指针 */
        rt_thread_create( "led2",              /* 线程名字 */
                          led2_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (led2_thread != RT_NULL)
        rt_thread_startup(led2_thread);
    else
        return -1;
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
        rt_kprintf("led1_thread running,LED1_ON\r\n");
        LED1(OFF);
        rt_thread_delay(500);   /* 延时500个tick */
        rt_kprintf("led1_thread running,LED1_OFF\r\n");
    }
}
/**
 * @brief  led2_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void led2_thread_entry(void* parameter)
{
    while (1)
    {
        LED2(ON);
        rt_thread_delay(300);   /* 延时300个tick */
        rt_kprintf("led2_thread running,LED2_ON\r\n");
        LED2(OFF);
        rt_thread_delay(300);   /* 延时300个tick */
        rt_kprintf("led2_thread running,LED2_OFF\r\n");
    }
}

```
# 调试
- 编译下载到开发板
- 可以看到LED1,LED2闪烁
- 打开串口助手，可以看到串口发送的数据