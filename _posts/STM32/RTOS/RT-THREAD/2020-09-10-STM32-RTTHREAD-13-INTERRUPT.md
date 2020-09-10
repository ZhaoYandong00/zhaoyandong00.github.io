---
title: 中断管理RT-Thread篇13
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread中断管理
---
# 中断
- `stm32f10x_it.c`

```c
#include "stm32f10x_it.h"
#include "rtthread.h"
#include "main.h"
/* 外部定义消息队列控制块 */
extern rt_mq_t test_mq;
extern uint8_t ReceiveBuff[USART_RX_BUFF_SIZE];

// 串口中断服务函数
void DEBUG_USART_IRQHandler(void)
{
    uint16_t len;
    /* 进入中断 */
    rt_interrupt_enter();

    if(USART_GetITStatus(DEBUG_USARTx,USART_IT_IDLE)!=RESET)
    {
        DMA_Cmd(USART_RX_DMA_CHANNEL,DISABLE);
        len=USART_RX_BUFF_SIZE-DMA_GetCurrDataCounter(USART_RX_DMA_CHANNEL);
        DMA_SetCurrDataCounter(USART_RX_DMA_CHANNEL,USART_RX_BUFF_SIZE);
        DMA_Cmd(USART_RX_DMA_CHANNEL,ENABLE);
        //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
        USART_ReceiveData(DEBUG_USARTx);
        rt_mq_send(	test_mq,	/* 写入（发送）队列的ID(句柄) */
                    &ReceiveBuff,			/* 写入（发送）的数据 */
                    len);			/* 数据的长度 */
        memset(ReceiveBuff,0,USART_RX_BUFF_SIZE);
    }
    /* 离开中断 */
    rt_interrupt_leave();
}

```
## 主程序
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
static rt_thread_t usart_thread = RT_NULL;
/* 定义消息队列控制块 */
rt_mq_t test_mq = RT_NULL;


/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void usart_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/


int main(void)
{
    /* 创建一个消息队列 */
    test_mq = rt_mq_create("test_mq",/* 消息队列名字 */
                           USART_RX_BUFF_SIZE,     /* 消息的最大长度 */
                           2,    /* 消息队列的最大容量 */
                           RT_IPC_FLAG_FIFO);/* 队列模式 FIFO(0x00)*/
    if (test_mq != RT_NULL)
        rt_kprintf("queue success\n\n");
    usart_thread =                          /* 线程控制块指针 */
        rt_thread_create( "usart",              /* 线程名字 */
                          usart_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          2,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (usart_thread != RT_NULL)
        rt_thread_startup(usart_thread);
    else
        return -1;
}
/**
 * @brief  alloc_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void usart_thread_entry(void* parameter)
{
    rt_err_t uwRet = RT_EOK;
    uint8_t r_queue[USART_RX_BUFF_SIZE];
    while (1)
    {        
        uwRet = rt_mq_recv(test_mq,	/* 读取（接收）队列的ID(句柄) */
                           &r_queue,			/* 读取（接收）的数据保存位置 */
                           USART_RX_BUFF_SIZE,		/* 读取（接收）的数据的长度 */
                           RT_WAITING_FOREVER); 	/* 等待时间：一直等 */

        if(RT_EOK == uwRet)
        {
            rt_kprintf("data:%s\n",r_queue);
            memset(r_queue,0,USART_RX_BUFF_SIZE);
            LED1_TOGGLE;
        }
    }
}

```
# 调试
- 编译下载到开发板
- 使用串口助手向开发板发送数据，会看到回传数据和LED翻转