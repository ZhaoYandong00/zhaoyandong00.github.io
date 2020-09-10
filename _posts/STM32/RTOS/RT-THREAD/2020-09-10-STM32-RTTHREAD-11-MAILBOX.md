---
title: 邮箱RT-Thread篇11
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread邮箱
---
# 邮箱常用函数
- 创建邮箱`rt_mailbox_t rt_mb_create(const char *name,rt_size_t size,rt_uint8_t flag)`
- 删除邮箱`rt_err_t rt_mb_delete(rt_mailbox_t mb)`
- 直接发送`rt_err_t rt_mb_send(rt_mailbox_t mb, rt_uint32_t value)`
- 阻塞发送`rt_err_t rt_mb_send_wait(rt_mailbox_t mb,rt_uint32_t value,rt_int32_t timeout)`
- 接收`rt_err_t rt_mb_recv(rt_mailbox_t mb,rt_uint32_t *value,rt_int32_t timeout)`

# 邮箱
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
/* 定义邮箱控制块 */
static rt_mailbox_t test_mail = RT_NULL;

/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void receive_thread_entry(void* parameter);
static void send_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/
char test_str1[] = "this is a mail test 1";/* 邮箱消息test1 */
char test_str2[] = "this is a mail test 2";/* 邮箱消息test2 */

int main(void)
{
    /* 创建一个邮箱 */
    test_mail = rt_mb_create("test_mail", /* 邮箱名字 */
                             10,
                             RT_IPC_FLAG_FIFO);/* 信号量模式 FIFO(0x00)*/
    if (test_mail != RT_NULL)
        rt_kprintf("mail box!\n\n");

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
    char *r_str;
    /* 任务都是一个无限循环，不能返回 */
    while(1)
    {
        /* 等待接邮箱消息 */
        uwRet = rt_mb_recv(test_mail, /* 邮箱对象句柄 */
                           (rt_uint32_t*)&r_str, /* 接收邮箱消息 */
                           RT_WAITING_FOREVER);/* 指定超时事件,一直等 */

        if(RT_EOK == uwRet) /* 如果接收完成并且正确 */
        {
            rt_kprintf ( "mail box receive:%s\n\n",r_str);
            LED1_TOGGLE;       //LED1	反转
        }
        else
            rt_kprintf ( "mail box,err:0x%x\n",uwRet);
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
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {   //如果KEY1被单击
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {
            rt_kprintf ( "KEY1 press\n" );
            /* 发送一个邮箱消息1 */
            uwRet = rt_mb_send(test_mail,(rt_uint32_t)&test_str1);
            if(RT_EOK == uwRet)
                rt_kprintf ( "mail box send success\n" );
            else
                rt_kprintf ( "mail box send fail\n" );
        }
        //如果KEY2被单击
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {
            rt_kprintf ( "KEY2 press\n" );
            /* 发送一个邮箱2 */
            uwRet = rt_mb_send(test_mail,(rt_uint32_t)&test_str2);
            if(RT_EOK == uwRet)
                rt_kprintf ( "mail box send success\n" );
            else
                rt_kprintf ( "mail box send fail\n" );
        }
        rt_thread_delay(20);     //每20ms扫描一次
    }
}

```
# 调试
- 编译下载到开发板
- 按下开发版的KEY1发送邮件1，按下KEY2 按键发送邮件2
- 在串口助手中可以看到接收到消息