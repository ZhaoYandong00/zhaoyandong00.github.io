---
title: 互斥量RT-Thread篇8
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread互斥量
---
# 互斥量常用函数
- 创建互斥量`rt_mutex_t rt_mutex_create(const char *name,rt_uint8_t flag)`
- 删除互斥量`rt_err_t rt_mutex_delete(rt_mutex_t mutex)`
- 获取互斥量`rt_err_t rt_mutex_take(rt_mutex_t mutex,rt_int32_t time)`
- 释放互斥量`rt_err_t rt_mutex_release(rt_mutex_t mutex)`

# 互斥量
- 在`rtconfig.h`中打开互斥量

```c
// </c>
// <c1>Using Mutex
//  <i>Using Mutex
#define RT_USING_MUTEX
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
/* 定义线程控制块 */
static rt_thread_t LowPriority_thread = RT_NULL;
static rt_thread_t MidPriority_thread = RT_NULL;
static rt_thread_t HighPriority_thread = RT_NULL;
/* 定义互斥量控制块 */
static rt_mutex_t test_mux = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void LowPriority_thread_entry(void* parameter);
static void MidPriority_thread_entry(void* parameter);
static void HighPriority_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/
uint8_t ucValue [ 2 ] = { 0x00, 0x00 };

int main(void)
{
    /* 创建一个互斥量 */
    test_mux = rt_mutex_create("test_mux",RT_IPC_FLAG_PRIO);
    if (test_mux != RT_NULL)
        rt_kprintf("mux \n\n");
    LowPriority_thread =                          /* 线程控制块指针 */
        rt_thread_create( "LowPriority",              /* 线程名字 */
                          LowPriority_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          5,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (LowPriority_thread != RT_NULL)
        rt_thread_startup(LowPriority_thread);
    else
        return -1;
    MidPriority_thread =                          /* 线程控制块指针 */
        rt_thread_create( "MidPriority",              /* 线程名字 */
                          MidPriority_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          4,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (MidPriority_thread != RT_NULL)
        rt_thread_startup(MidPriority_thread);
    else
        return -1;
    HighPriority_thread =                          /* 线程控制块指针 */
        rt_thread_create( "HighPriority",              /* 线程名字 */
                          HighPriority_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (HighPriority_thread != RT_NULL)
        rt_thread_startup(HighPriority_thread);
    else
        return -1;
}
/**
 * @brief  LowPriority_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void LowPriority_thread_entry(void* parameter)
{
    static uint32_t i;
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        rt_kprintf("LowPriority  take\n\n");
        rt_mutex_take(test_mux,	          /* 获取互斥量 */
                      RT_WAITING_FOREVER); 	/* 等待时间：一直等 */
        for(i=0; i<2000000; i++) //模拟低优先级任务占用信号量
        {
            rt_thread_yield();
        }
        rt_kprintf("LowPriority  release\n\n");
        rt_mutex_release(	test_mux	);    //释放互斥量

        rt_thread_delay ( 500 );
    }
}
/**
 * @brief  MidPriority_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void MidPriority_thread_entry(void* parameter)
{
    while (1)
    {
        rt_kprintf("MidPriority  runing\n\n");
        rt_thread_delay ( 500 );
    }
}
/**
 * @brief  HighPriority_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void HighPriority_thread_entry(void* parameter)
{
    while (1)
    {
        rt_kprintf("HighPriority  take\n\n");
        rt_mutex_take(test_mux,	          /* 获取互斥量 */
                      RT_WAITING_FOREVER); 	/* 等待时间：一直等 */
        rt_kprintf("HighPriority  give\n\n");
        rt_mutex_release(	test_mux	);    //释放互斥量
        rt_thread_delay ( 500 );
    }
}

```
# 调试
- 编译下载到开发板
- 可以看到在低优先级任务运行的时候，中优先级任务无法抢占低优先级的任务