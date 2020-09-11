---
title: 事件RT-Thread篇9
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread事件
---
# 事件常用函数
- 创建事件`rt_event_t rt_event_create(const char *name,rt_uint8_t flag)`
- 删除事件`rt_err_t rt_event_delete(rt_event_t event)`
- 事件发送`rt_err_t rt_event_send(rt_event_t event,rt_uint32_t set)`
- 事件接收`rt_err_t rt_event_recv(rt_event_t  event,rt_uint32_t set,rt_uint8_t  opt,rt_int32_t  timeout,rt_uint32_t *recved)`

# 事件
- 在`rtconfig.h`中打开事件

```c
// <c1>Using Event
//  <i>Using Event
#define RT_USING_EVENT
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
static rt_thread_t receive_thread = RT_NULL;
static rt_thread_t send_thread = RT_NULL;
/* 定义事件控制块 */
static rt_event_t test_event = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void receive_thread_entry(void* parameter);
static void send_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/
#define KEY1_EVENT  (0x01 << 0)//设置事件掩码的位0
#define KEY2_EVENT  (0x01 << 1)//设置事件掩码的位1

int main(void)
{
    /* 创建一个事件 */
    test_event = rt_event_create("test_event",/* 事件标志组名字 */
                                 RT_IPC_FLAG_PRIO); /* 事件模式 FIFO(0x00)*/
    if (test_event != RT_NULL)
        rt_kprintf("Event Success\n\n");
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
    rt_uint32_t recved;
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        rt_event_recv(test_event,  /* 事件对象句柄 */
                      KEY1_EVENT|KEY2_EVENT,/* 接收线程感兴趣的事件 */
                      RT_EVENT_FLAG_AND|RT_EVENT_FLAG_CLEAR,/* 接收选项 */
                      RT_WAITING_FOREVER,/* 指定超时事件,一直等 */
                      &recved);    /* 指向接收到的事件 */
        if(recved == (KEY1_EVENT|KEY2_EVENT)) /* 如果接收完成并且正确 */
        {
            rt_kprintf ( "Key1 Key2 Press\n");
            LED1_TOGGLE;       //LED1	反转
        }
        else
            rt_kprintf ( "ERR\n");
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
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
        {
            rt_kprintf ( "KEY1 Press\n" );
            /* 发送一个事件1 */
            rt_event_send(test_event,KEY1_EVENT);
        }

        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
        {
            rt_kprintf ( "KEY2 Press\n" );
            /* 发送一个事件2 */
            rt_event_send(test_event,KEY2_EVENT);
        }
        rt_thread_delay(20);     //每20ms扫描一次
    }
}


```
# 调试
- 编译下载到开发板
- 按下KEY1 按键发送事件1,按下KEY2,按键发送事件2
- 我们按下KEY1与KEY2试试,在串口调试助手中可以看到运行结果,并且当事件1与事件2都发生的时候,开发板的LED会进行翻转