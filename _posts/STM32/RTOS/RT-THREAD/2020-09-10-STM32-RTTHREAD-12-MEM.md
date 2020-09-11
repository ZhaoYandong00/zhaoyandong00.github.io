---
title: 内存管理RT-Thread篇12
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread内存管理
---
# 内存管理常用函数
- 申请系统堆内存`void *rt_malloc(rt_size_t nbytes)`
- 释放系统堆内存`void rt_free(void *ptr)`
- 更改分配内存`void *rt_realloc(void *ptr,rt_size_t nbytes)`
- 申请多个内存`void *rt_calloc(rt_size_t count,rt_size_t size)`
- 申请内存带字节对齐`void *rt_malloc_align(rt_size_t size,rt_size_t align)`
- 释放内存带字节对齐`void rt_free_align(void *ptr)`

# 内存管理
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
static rt_thread_t alloc_thread = RT_NULL;
static rt_thread_t free_thread = RT_NULL;
/* 定义申请内存的指针 */
static rt_uint32_t *p_test = RT_NULL;

/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void alloc_thread_entry(void* parameter);
static void free_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/
#define  TEST_SIZE   100	  //内存大小

int main(void)
{
    alloc_thread =                          /* 线程控制块指针 */
        rt_thread_create( "alloc",              /* 线程名字 */
                          alloc_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          1,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (alloc_thread != RT_NULL)
        rt_thread_startup(alloc_thread);
    else
        return -1;

    free_thread =                          /* 线程控制块指针 */
        rt_thread_create( "free",              /* 线程名字 */
                          free_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          2,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (free_thread != RT_NULL)
        rt_thread_startup(free_thread);
    else
        return -1;
}
/**
 * @brief  alloc_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void alloc_thread_entry(void* parameter)
{
    rt_kprintf("DYNAMIC ...........\n");
    p_test = rt_malloc(TEST_SIZE);    /* 申请动态内存 */
    if(RT_NULL == p_test)/* 没有申请成功 */
        rt_kprintf("malloc fail\n");
    else
      rt_kprintf("malloc success,addr:%d!\n\n",p_test);

    rt_kprintf("writing p_test...........\n");
    *p_test = 1234;
    rt_kprintf("writed p_test\n");
    rt_kprintf("*p_test = %.4d,addr:%d \n\n", *p_test,p_test);

    /* 任务都是一个无限循环，不能返回 */
    while(1)
    {
        LED2_TOGGLE;
        rt_thread_delay(1000);     //每1000ms扫描一次
    }
}
/**
 * @brief  free_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void free_thread_entry(void* parameter)
{
  rt_kprintf("free...........\n");
  rt_free(p_test);
  rt_kprintf("free success\n\n");

    /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    LED1_TOGGLE;
		rt_thread_delay(500);     //每500ms扫描一次		
  }
}

```
# 调试
- 编译下载到开发板
- 在调试助手中看到串口的打印信息与运行结果