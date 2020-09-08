---
title: 动态内存创建任务RT-Thread篇3
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread动态内存创建线程
---
# 线程常用函数
- 创建线程

```c
rt_thread_t rt_thread_create(const char *name,
                             void (*entry)(void *parameter),
                             void       *parameter,
                             rt_uint32_t stack_size,
                             rt_uint8_t  priority,
                             rt_uint32_t tick)
```
- 启动线程`rt_err_t rt_thread_startup(rt_thread_t thread)`
- 删除线程`rt_err_t rt_thread_delete(rt_thread_t thread)`
- 线程调度`rt_err_t rt_thread_yield(void)`
- 延时`rt_err_t rt_thread_delay(rt_tick_t tick)`
- ms延时`rt_err_t rt_thread_mdelay(rt_int32_t ms)`
- 线程挂起`rt_err_t rt_thread_suspend(rt_thread_t thread)`
- 线程恢复`rt_err_t rt_thread_resume(rt_thread_t thread)`
- 等待调用`void rt_thread_timeout(void *parameter)`

# 动态内存创建线程
## 配置
- 修改`rtconfig.h`,打开动态内存

```c
#define RT_USING_HEAP
```

## 创建单任务
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
static rt_thread_t led1_thread = RT_NULL;

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

##　创建多任务
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
static rt_thread_t led1_thread = RT_NULL;
static rt_thread_t led2_thread = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void led1_thread_entry(void* parameter);
static void led2_thread_entry(void* parameter);

static void BSP_Init(void);/* 用于初始化板载相关资源 */

int main(void)
{
    BSP_Init();
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
                      4,                   /* 线程的优先级 */
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

        LED1(OFF);
        rt_thread_delay(500);   /* 延时500个tick */

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
        
        LED2(OFF);     
        rt_thread_delay(300);   /* 延时300个tick */		 		

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
- 可以看到LED1,LED2闪烁