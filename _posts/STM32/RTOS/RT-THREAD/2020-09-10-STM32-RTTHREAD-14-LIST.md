---
title: 双向链表RT-Thread篇14
categories: STM32 RTOS RT-Thread
tags: STM32 RTOS RT-Thread
description: RT-Thread双向链表
---
# 双向链表常用函数
- 链表初始化`void rt_list_init(rt_list_t *l)`
- 向链表指定节点后面插入节点`void rt_list_insert_after(rt_list_t *l,rt_list_t *n)`
- 向链表指定节点前面插入节点`void rt_list_insert_before(rt_list_t *l,rt_list_t *n)`
- 从链表删除节点函数`void rt_list_remove(rt_list_t *n)`
- 链表是否为空`int rt_list_isempty(const rt_list_t *l)`

# 双向链表
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
static rt_thread_t test1_thread = RT_NULL;
static rt_thread_t test2_thread = RT_NULL;;


/*
*************************************************************************
*                             函数声明
*************************************************************************
*/
static void test1_thread_entry(void* parameter);
static void test2_thread_entry(void* parameter);

/************************* 全局变量声明 ****************************/


int main(void)
{

    test1_thread =                          /* 线程控制块指针 */
        rt_thread_create( "test1",              /* 线程名字 */
                          test1_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          2,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (test1_thread != RT_NULL)
        rt_thread_startup(test1_thread);
    else
        return -1;

    test2_thread =                          /* 线程控制块指针 */
        rt_thread_create( "test2",              /* 线程名字 */
                          test2_thread_entry,   /* 线程入口函数 */
                          RT_NULL,             /* 线程入口函数参数 */
                          512,                 /* 线程栈大小 */
                          3,                   /* 线程的优先级 */
                          20);                 /* 线程时间片 */

    /* 启动线程，开启调度 */
    if (test2_thread != RT_NULL)
        rt_thread_startup(test2_thread);
    else
        return -1;
}
/**
 * @brief  test1_thread线程主体
 * @param  parameter 参数
 * @retval 无
 */
static void test1_thread_entry(void* parameter)
{
    rt_list_t *head;  /* 定义一个双向链表的头节点 */
    rt_list_t *node1;  /* 定义一个双向链表的头节点 */
    rt_list_t *node2;  /* 定义一个双向链表的头节点 */

    head = rt_malloc(sizeof(rt_list_t));/* 申请动态内存 */
    if(RT_NULL == head)/* 没有申请成功 */
        rt_kprintf("malloc fail\n");
    else
      rt_kprintf("malloc suceess,addr:%d!\n",head);

    rt_kprintf("\nlist......\n");
    rt_list_init(head);
    if(rt_list_isempty(head))
        rt_kprintf("list success!\n\n");

    /* 插入节点：顺序插入与从末尾插入 */
    rt_kprintf("list insert......\n");
    /* 动态申请第一个结点的内存 */
    node1 = rt_malloc(sizeof(rt_list_t));
    /* 动态申请第二个结点的内存 */
    node2 = rt_malloc(sizeof(rt_list_t));

    rt_kprintf("insert.....\n");
    /* 因为这是在某个节点后面添加一个节点函数
       为后面的rt_list_insert_before（某个节点之前）
       添加节点做铺垫,两个函数添加完之后的顺序是
       head -> node1 -> node2 */
    rt_list_insert_after(head,node2);
    rt_list_insert_before(node2,node1);
    if ((node1->prev == head) && (node2->prev == node1))
        rt_kprintf("insert success!\n\n");
    else
        rt_kprintf("insert fail!\n\n");

    rt_kprintf("delete......\n");	/* 删除已有节点 */
    rt_list_remove(node1);
    rt_free(node1);/* 释放第一个节点的内存 */
    if (node2->prev == head)
        rt_kprintf("delete success\n\n");

    /* 任务都是一个无限循环，不能返回 */
    while(1)
    {
        LED1_TOGGLE;
        rt_thread_delay(500);     //每500ms扫描一次
    }
}

static void test2_thread_entry(void* parameter)
{

    /* 任务都是一个无限循环，不能返回 */
    while (1)
    {
        rt_kprintf("running\n");
        LED2_TOGGLE;
        rt_thread_delay(1000);     //每1000ms扫描一次
    }
}

```
# 调试
- 编译下载到开发板
- 在串口助手中可以看到接收到消息