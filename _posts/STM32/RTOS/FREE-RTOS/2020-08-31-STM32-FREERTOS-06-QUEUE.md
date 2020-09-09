---
title: 消息队列FREERTOS篇6
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS消息队列
---
# 消息队列解析
## 消息队列结构体分析

```c
/*
 * Definition of the queue used by the scheduler.
 * Items are queued by copy, not reference.  See the following link for the
 * rationale: https://www.freertos.org/Embedded-RTOS-Queues.html
 */
typedef struct QueueDefinition 		/* The old naming convention is used to prevent breaking kernel aware debuggers. */
{
	int8_t *pcHead;					/*< Points to the beginning of the queue storage area. */
	int8_t *pcWriteTo;				/*< Points to the free next place in the storage area. */

	union
	{
		QueuePointers_t xQueue;		/*< Data required exclusively when this structure is used as a queue. */
		SemaphoreData_t xSemaphore; /*< Data required exclusively when this structure is used as a semaphore. */
	} u;

	List_t xTasksWaitingToSend;		/*< List of tasks that are blocked waiting to post onto this queue.  Stored in priority order. */
	List_t xTasksWaitingToReceive;	/*< List of tasks that are blocked waiting to read from this queue.  Stored in priority order. */

	volatile UBaseType_t uxMessagesWaiting;/*< The number of items currently in the queue. */
	UBaseType_t uxLength;			/*< The length of the queue defined as the number of items it will hold, not the number of bytes. */
	UBaseType_t uxItemSize;			/*< The size of each items that the queue will hold. */

	volatile int8_t cRxLock;		/*< Stores the number of items received from the queue (removed from the queue) while the queue was locked.  Set to queueUNLOCKED when the queue is not locked. */
	volatile int8_t cTxLock;		/*< Stores the number of items transmitted to the queue (added to the queue) while the queue was locked.  Set to queueUNLOCKED when the queue is not locked. */

	#if( ( configSUPPORT_STATIC_ALLOCATION == 1 ) && ( configSUPPORT_DYNAMIC_ALLOCATION == 1 ) )
		uint8_t ucStaticallyAllocated;	/*< Set to pdTRUE if the memory used by the queue was statically allocated to ensure no attempt is made to free the memory. */
	#endif

	#if ( configUSE_QUEUE_SETS == 1 )
		struct QueueDefinition *pxQueueSetContainer;
	#endif

	#if ( configUSE_TRACE_FACILITY == 1 )
		UBaseType_t uxQueueNumber;
		uint8_t ucQueueType;
	#endif

} xQUEUE;
```
- 队列消息存储区起始位置`pcHead`
- 队列消息存储区结束位置地址`pcTail`
- 队列消息存储区下一个可用消息空间`pcWriteTo`
- 仅在队列中可用`xQueue`
- 仅在信号量中可用`xSemaphore`
- 发送消息阻塞列表`xTasksWaitingToSend`
- 获取消息阻塞列表`xTasksWaitingToReceive`
- 记录当前消息队列的消息个数`uxMessagesWaiting`
- 队列的长度`uxLength`
- 单个消息的大小`uxItemSize`
- 队列接收锁`cRxLock`
- 队列发送锁`cTxLock`

## 常用函数
- 创建消息队列`QueueHandle_t xQueueCreate(UBaseType_t uxQueueLength,UBaseType_t uxItemSize)`
- 发送操作`BaseType_t xQueueGenericSend(QueueHandle_t xQueue, const void * const pvItemToQueue, TickType_t xTicksToWait, const BaseType_t xCopyPosition)`
- 中断中送`BaseType_t xQueueGenericSendFromISR(QueueHandle_t xQueue, const void * const pvItemToQueue, BaseType_t * const pxHigherPriorityTaskWoken, const BaseType_t xCopyPosition)`
- 向队尾送`BaseType_t xQueueSend(QueueHandle_t xQueue,const void * pvItemToQueue,TickType_t xTicksToWait)`
- 中断中向队尾送`BaseType_t xQueueSendFromISR(QueueHandle_t xQueue,const void * pvItemToQueue,BaseType_t * pxHigherPriorityTaskWoken)`
- 向队首送`BaseType_t xQueueSendToFront(QueueHandle_t xQueue,const void * pvItemToQueue,TickType_t xTicksToWait);`
- 中断中向队首送`BaseType_t xQueueSendToFrontFromISR(QueueHandle_t xQueue,const void *pvItemToQueue,BaseType_t *pxHigherPriorityTaskWoken)`
- 读队列操作删除消息`BaseType_t xQueueReceive(QueueHandle_t xQueue,void *pvBuffer,TickType_t xTicksToWait)`
- 读队列不删除消息`BaseType_t xQueuePeek(QueueHandle_t xQueue, void * const pvBuffer, TickType_t xTicksToWait)`
- 中断中读队列操作删除消息`BaseType_t xQueueReceiveFromISR(QueueHandle_t xQueue, void * const pvBuffer, BaseType_t * const pxHigherPriorityTaskWoken)`
- 中断中读队列不删除消息`BaseType_t xQueuePeekFromISR(QueueHandle_t xQueue, void * const pvBuffer)`
- 删除队列`void vQueueDelete(QueueHandle_t xQueue)`

# 消息队列
- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "my_gpio.h"
#include "my_usart.h"


/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */
static TaskHandle_t Receive_Task_Handle = NULL;/* 接收任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* 发送任务句柄 */
/* 消息队列句柄 */
QueueHandle_t Test_Queue =NULL;

#define  QUEUE_LEN    4   /* 队列的长度，最大可包含多少个消息 */
#define  QUEUE_SIZE   4   /* 队列中每个消息大小（字节） */

static void AppTaskCreate(void);/* 用于创建任务 */

static void Receive_Task(void* pvParameters);/* Receive_Task任务实现 */
static void Send_Task(void* pvParameters);/* Send_Task任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-QUEUE!\r\n");
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    /* 创建 AppTaskCreate 任务 */
    xReturn = xTaskCreate((TaskFunction_t	)AppTaskCreate,		//任务函数
                          (const char* 	)"AppTaskCreate",		//任务名称
                          (uint32_t 		)128,	//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)1, 	//任务优先级
                          (TaskHandle_t*  )&AppTaskCreate_Handle);	//任务控制块

    if (pdPASS == xReturn)/* 创建成功 */
        vTaskStartScheduler();   /* 启动任务，开启调度 */
    else
        return -1;
    while(1);   /* 正常不会执行到这里 */
}

/**
 * @brief  为了方便管理，所有的任务创建函数都放在这个函数里面
 * @param  无
 * @retval 无
 */
static void AppTaskCreate(void)
{
    taskENTER_CRITICAL();           //进入临界区
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */

    Test_Queue = xQueueCreate((UBaseType_t ) QUEUE_LEN,/* 消息队列的长度 */
                              (UBaseType_t ) QUEUE_SIZE);/* 消息的大小 */
    if(NULL != Test_Queue)
        printf("QUEUE SUCCESS!\n");
    
    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Receive_Task,		//任务函数
                          (const char* 	)"Receive_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)2, 				//任务优先级
                          (TaskHandle_t* )&Receive_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Receive_Task SUCCESS!\n");

    /* 创建LED_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t	)Send_Task,		//任务函数
                          (const char* 	)"Send_Task",		//任务名称
                          (uint32_t 		)128,					//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)3, 				//任务优先级
                          (TaskHandle_t* )&Send_Task_Handle);/* 任务控制块指针 */

    if (pdPASS == xReturn)/* 创建成功 */
        printf("Send_Task SUCCESS!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Receive_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Receive_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    uint32_t r_queue;	/* 定义一个接收消息的变量 */
    while (1)
    {
        xReturn = xQueueReceive( Test_Queue,    /* 消息队列的句柄 */
                                 &r_queue,      /* 发送的消息内容 */
                                 portMAX_DELAY); /* 等待时间 一直等 */
        if(pdTRUE == xReturn)
            printf("receive data is:%d\n\n",r_queue);
        else
            printf("receive data err,err code:0x%lx\n",xReturn);
    }
}
/**
 * @brief  Send_Task任务主体
 * @param  parameter 参数
 * @retval 无
 */
static void Send_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
    uint32_t send_data1 = 1;
    uint32_t send_data2 = 2;
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
        {   /* K1 被按下 */
            printf("send_data1!\n");
            xReturn = xQueueSend( Test_Queue, /* 消息队列的句柄 */
                                  &send_data1,/* 发送的消息内容 */
                                  0 );        /* 等待时间 0 */
            if(pdPASS == xReturn)
                printf("send_data1 success!\n\n");
        }
        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
        {   /* K2 被按下 */
            printf("send_data2!\n");
            xReturn = xQueueSend( Test_Queue, /* 消息队列的句柄 */
                                  &send_data2,/* 发送的消息内容 */
                                  0 );        /* 等待时间 0 */
            if(pdPASS == xReturn)
                printf("send_data2 success!\n\n");
        }
        vTaskDelay(20);/* 延时20个tick */
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
# 调试
- 编译下载到开发板
- 按下开发版的KEY1 按键发送消息1,按下KEY2按键发送消息2
- 按下KEY1,在串口助手中可以看到接收到消息1,按下KEY2,在串口助手中可以看到接收到消息2