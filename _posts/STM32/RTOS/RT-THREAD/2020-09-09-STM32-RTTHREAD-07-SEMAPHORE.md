---
title: 消息量RT-Thread篇7
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread消息量
---
# 信号量常用函数
- 创建信号量`rt_sem_t rt_sem_create(const char *name,rt_uint32_t value,rt_uint8_t flag)`
- 删除信号量`rt_err_t rt_sem_delete(rt_sem_t sem)`
- 获取信号量`rt_err_t rt_sem_take(rt_sem_t sem, rt_int32_t time)`
- 尝试获取信号量`rt_err_t rt_sem_trytake(rt_sem_t sem)`
- 释放信号量`rt_err_t rt_sem_release(rt_sem_t sem)`

# 二值信号量
## 程序
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
static rt_thread_t receive_thread = RT_NULL;
static rt_thread_t send_thread = RT_NULL;
/* 定义消息队列控制块 */
static rt_sem_t test_sem = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void receive_thread_entry(void* parameter);
static void send_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/
uint8_t ucValue [ 2 ] = { 0x00, 0x00 };


int main(void)
{
    /* 创建一个信号量 */
    test_sem = rt_sem_create("test_sem",/* 信号量名字 */
                             1,     /* 信号量初始值，默认有一个信号量 */
                             RT_IPC_FLAG_FIFO); /* 信号量模式 FIFO(0x00)*/
    if (test_sem != RT_NULL)
        rt_kprintf("信号量创建成功！\n\n");
    receive_thread =                          /* 线程控制块指针 */
        rt_thread_create( "receive",              /* 线程名字 */
                          receive_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (receive_thread != RT_NULL)
        rt_thread_startup(receive_thread);
    else
        return -1;
    send_thread =                          /* 线程控制块指针 */
        rt_thread_create( "send",              /* 线程名字 */
                          send_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          2,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (send_thread != RT_NULL)
        rt_thread_startup(send_thread);
    else
        return -1;
}
/**
 * @brief  receive_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void receive_thread_entry(void* parameter)
{
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        rt_sem_take(test_sem,	/* 获取信号量 */
                    RT_WAITING_FOREVER); 	/* 等待时间：一直等 */
        if ( ucValue [ 0 ] == ucValue [ 1 ] )
        {
            rt_kprintf ( "Successful\n" );
        }
        else
        {
            rt_kprintf ( "Fail\n" );
        }
        rt_sem_release(	test_sem	);   //释放二值信号量

        rt_thread_delay ( 1000 );  					      //每1s读一次
    }
}
/**
 * @brief  send_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void send_thread_entry(void* parameter)
{
    while (1)
    {
        rt_sem_take(test_sem,	/* 获取信号量 */
                    RT_WAITING_FOREVER); /* 等待时间：一直等 */
        ucValue [ 0 ] ++;
        rt_thread_delay ( 100 );        	 /* 延时100ms */
        ucValue [ 1 ] ++;
        rt_sem_release(	test_sem	);	//释放二值信号量
        rt_thread_yield();  			//放弃剩余时间片，进行一次任务切换
    }
}

```
## 调试
- 编译下载到开发板
- 在串口助手中可以看到接收到消息

# 计数信号量
## 程序
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
static rt_thread_t receive_thread = RT_NULL;
static rt_thread_t send_thread = RT_NULL;
/* 定义消息队列控制块 */
static rt_sem_t test_sem = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void receive_thread_entry(void* parameter);
static void send_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/


int main(void)
{
    /* 创建一个信号量 */
    test_sem = rt_sem_create("test_sem",/* 信号量名字 */
                             5,     /* 信号量初始值，默认有5个信号量 */
                             RT_IPC_FLAG_FIFO); /* 信号量模式 FIFO(0x00)*/
    if (test_sem != RT_NULL)
        rt_kprintf("信号量创建成功！\n\n");
    receive_thread =                          /* 线程控制块指针 */
        rt_thread_create( "receive",              /* 线程名字 */
                          receive_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (receive_thread != RT_NULL)
        rt_thread_startup(receive_thread);
    else
        return -1;
    send_thread =                          /* 线程控制块指针 */
        rt_thread_create( "send",              /* 线程名字 */
                          send_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          2,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (send_thread != RT_NULL)
        rt_thread_startup(send_thread);
    else
        return -1;
}
/**
 * @brief  receive_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void receive_thread_entry(void* parameter)
{
    rt_err_t uwRet = RT_EOK;
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
        {
            /* 获取一个计数信号量 */
            uwRet = rt_sem_trytake(test_sem); 	/* 等待时间：0 */
            if ( RT_EOK == uwRet )
                printf( "KEY1 success!\n" );
            else
                printf( "KEY1 fail!\n" );
        }
        rt_thread_delay(20);     //每20ms扫描一次
    }
}
/**
 * @brief  send_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void send_thread_entry(void* parameter)
{
    rt_err_t uwRet = RT_EOK;
    while (1)
    {
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
        {
            /* 释放一个计数信号量 */
            uwRet = rt_sem_release(test_sem);
            if ( RT_EOK == uwRet )
                rt_kprintf ( "KEY2 success\r\n" );
            else
                rt_kprintf ( "KEY2 fail\r\n" );
        }
        rt_thread_delay(20);     //每20ms扫描一次
    }
}

```
## 调试
- 编译下载到开发板
- 串口助手可看到当按下5次KEY1,再按就会出现错误，这是先按下KEY2,再按KEY1，成功