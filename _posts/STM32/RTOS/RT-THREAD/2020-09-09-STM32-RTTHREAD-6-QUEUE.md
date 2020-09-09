---
title: 消息队列RT-Thread篇6
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread消息队列
---
# 消息队列常用函数
- 创建消息队列`rt_mq_t rt_mq_create(const char *name,rt_size_t msg_size,rt_size_t max_msgs,rt_uint8_t flag)`
- 删除消息队列`rt_err_t rt_mq_delete(rt_mq_t mq)`
- 消息队列发送消息`rt_err_t rt_mq_send(rt_mq_t mq,void *buffer,rt_size_t size)`
- 消息队列发送紧急消息`rt_err_t rt_mq_urgent(rt_mq_t mq,void *buffer,rt_size_t size)`
- 消息队列接收消息`rt_err_t rt_mq_recv(rt_mq_t mq,void *buffer,rt_size_t size,rt_int32_t timeout)`

# 消息队列
- 在`rtconfig.h`中打开消息队列

```c
// </c>
// <c1>Using Message Queue
//  <i>Using Message Queue
#define RT_USING_MESSAGEQUEUE
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
/* 定义消息队列控制块 */
static rt_mq_t test_mq = RT_NULL;
/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void receive_thread_entry(void* parameter);
static void send_thread_entry(void* parameter);

int main(void)
{
    test_mq = rt_mq_create("test_mq",/* 消息队列名字 */
                           4,     /* 消息的最大长度 */
                           20,    /* 消息队列的最大容量 */
                           RT_IPC_FLAG_FIFO);/* 队列模式 FIFO(0x00)*/
    if (test_mq != RT_NULL)
        rt_kprintf("消息队列创建成功！\n\n");
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
    uint32_t r_queue;
    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        /* 队列读取（接收），等待时间为一直等待 */
        uwRet = rt_mq_recv(test_mq,	/* 读取（接收）队列的ID(句柄) */
                           &r_queue,			/* 读取（接收）的数据保存位置 */
                           sizeof(r_queue),		/* 读取（接收）的数据的长度 */
                           RT_WAITING_FOREVER); 	/* 等待时间：一直等 */
        if(RT_EOK == uwRet)
        {
            rt_kprintf("receive data:%d\n",r_queue);
        }
        else
        {
            rt_kprintf("receive err:0x%lx\n",uwRet);
        }
        rt_thread_delay(200);
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
    uint32_t send_data1 = 1;
    uint32_t send_data2 = 2;
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )/* K1 被按下 */
        {
            /* 将数据写入（发送）到队列中，等待时间为 0  */
            uwRet = rt_mq_send(	test_mq,	/* 写入（发送）队列的ID(句柄) */
                                &send_data1,			/* 写入（发送）的数据 */
                                sizeof(send_data1));			/* 数据的长度 */
            if(RT_EOK != uwRet)
            {
                rt_kprintf("err:%lx\n",uwRet);
            }
        }
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )/* K1 被按下 */
        {
            /* 将数据写入（发送）到队列中，等待时间为 0  */
            uwRet = rt_mq_send(	test_mq,	/* 写入（发送）队列的ID(句柄) */
                                &send_data2,			/* 写入（发送）的数据 */
                                sizeof(send_data2));			/* 数据的长度 */
            if(RT_EOK != uwRet)
            {
                rt_kprintf("err:%lx\n",uwRet);
            }
        }
        rt_thread_delay(20);
    }
}

```
# 调试
- 编译下载到开发板
- 按下开发版的KEY1 按键发送消息1,按下KEY2按键发送消息2
- 按下KEY1,在串口助手中可以看到接收到消息1,按下KEY2,在串口助手中可以看到接收到消息2