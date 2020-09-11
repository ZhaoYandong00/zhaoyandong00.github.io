---
title: 软件定时器RT-Thread篇10
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread软件定时器
---
# 软件定时器常用函数
- 创建软件计时器`rt_timer_t rt_timer_create(const char *name,void (*timeout)(void *parameter),void *parameter,rt_tick_t time,rt_uint8_t flag)`
- 删除软件计时器`rt_err_t rt_timer_delete(rt_timer_t timer)`
- 开始软件计时器`rt_err_t rt_timer_start(rt_timer_t timer)`
- 停止软件计时器`rt_err_t rt_timer_stop(rt_timer_t timer)`


# 软件定时器
- 在`rtconfig.h`中打开软件定时器

```c
// <e>Software timers Configuration
// <i> Enables user timers
#define RT_USING_TIMER_SOFT         1
```
- 修改`main.c`

```c
#include "rtthread.h"
#include "main.h"

/*
*************************************************************************
*                               变量
*************************************************************************
*/
/* 定义软件定时器控制块 */
static rt_timer_t swtmr1 = RT_NULL;
static rt_timer_t swtmr2 = RT_NULL;

/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void swtmr1_callback(void* parameter);
static void swtmr2_callback(void* parameter);

/************************* 全局变量声明 ****************************/
static uint32_t TmrCb_Count1 = 0;
static uint32_t TmrCb_Count2 = 0;

int main(void)
{
    /* 创建一个软件定时器 */
    swtmr1 = rt_timer_create("swtmr1_callback", /* 软件定时器的名称 */
                             swtmr1_callback,/* 软件定时器的回调函数 */
                             0,			/* 定时器超时函数的入口参数 */
                             5000,   /* 软件定时器的超时时间(周期回调时间) */
                             RT_TIMER_FLAG_ONE_SHOT | RT_TIMER_FLAG_SOFT_TIMER);
    /* 软件定时器模式 一次模式 */
    /* 启动定时器 */
    if (swtmr1 != RT_NULL)
        rt_timer_start(swtmr1);

    /* 创建一个软件定时器 */
    swtmr2 = rt_timer_create("swtmr2_callback", /* 软件定时器的名称 */
                             swtmr2_callback,/* 软件定时器的回调函数 */
                             0,			/* 定时器超时函数的入口参数 */
                             1000,   /* 软件定时器的超时时间(周期回调时间) */
                             RT_TIMER_FLAG_PERIODIC | RT_TIMER_FLAG_SOFT_TIMER);
    /* 软件定时器模式 周期模式 */
    /* 启动定时器 */
    if (swtmr2 != RT_NULL)
        rt_timer_start(swtmr2);
}
/**
 * @brief  swtmr1_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void swtmr1_callback(void* parameter)
{		
    uint32_t tick_num1;

    TmrCb_Count1++;						/* 每回调一次加一 */

    tick_num1 = (uint32_t)rt_tick_get();	/* 获取滴答定时器的计数值 */
	
    rt_kprintf("swtmr1_callback %d \n", TmrCb_Count1);
    rt_kprintf("TICK=%d\n", tick_num1);
}
/**
 * @brief  swtmr2_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void swtmr2_callback(void* parameter)
{	
    uint32_t tick_num2;
	
    TmrCb_Count2++;				/* 每回调一次加一 */
	
    tick_num2 = (uint32_t)rt_tick_get();	/* 获取滴答定时器的计数值 */
	
    rt_kprintf("swtmr2_callback %d \n", TmrCb_Count2);
	
    rt_kprintf("TICK=%d\n", tick_num2);
}

```
# 调试
- 编译下载到开发板
- 每1000 个tick 时候软件定时器就会触发一次回调函数
- 当5000 个tick 到来的时候,触发软件定时器单次模式的回调函数,之后便不会再次调用了
