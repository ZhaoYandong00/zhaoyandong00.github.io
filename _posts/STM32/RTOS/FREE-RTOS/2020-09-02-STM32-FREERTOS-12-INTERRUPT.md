---
title:  中断管理FREERTOS篇12
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS中断管理
---
# 中断管理
- `configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY`,它是用于配置内核中的basepri 寄存器的,当basepri 设置为某个值的时候,NVIC 不会响应比该优先级低的中断,而优先级比之更高的中断则不受影响
- 就是说当这个宏定义配置为5 的时候,中断优先级数值在0、1、2、3、4 的这些中断是不受FreeRTOS 屏蔽的，也就是说即使在系统进入临界段的时候，这些中断也能被触发而不是等到退出临界段的时候才被触发，当然，这些中断服务函数中也不能调用FreeRTOS 提供的API 函数接口，而中断优先级在5 到15 的这些中断是可以被屏蔽的，也能安全调用FreeRTOS 提供的API 函数接口

# 程序
## 串口
- 串口修改为DMA接收,中断抢断优先必须在5-15之间
- `my_usart.c`

```c
#include "my_usart.h"

uint8_t ReceiveBuff[USART_RX_BUFF_SIZE];

/**
 * @brief  配置嵌套向量中断控制器NVIC
 * @param  无
 * @retval 无
 */
static void NVIC_Configuration(void)
{
    NVIC_InitTypeDef NVIC_InitStructure;

    /* 配置USART为中断源 */
    NVIC_InitStructure.NVIC_IRQChannel = DEBUG_USART_IRQ;
    /* 抢断优先级*/
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 7;
    /* 子优先级 */
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
    /* 使能中断 */
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    /* 初始化配置NVIC */
    NVIC_Init(&NVIC_InitStructure);

}

/**
 * @brief  串口接收DMA配置
 * @param  无
 * @retval 无
 */
static void USARTx_DMA_RX_TX_Config(void)
{
    DMA_InitTypeDef DMA_InitStructure;
    // 开启DMA时钟
    RCC_AHBPeriphClockCmd(USART_DMA_CLK, ENABLE);
    // 设置DMA源地址：串口数据寄存器地址*/
    DMA_InitStructure.DMA_PeripheralBaseAddr = USART_DR_ADDRESS;
    // 外设地址不增
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
    // 内存地址自增
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
    // 外设数据单位
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;
    // 内存数据单位
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;
    // 优先级：高
    DMA_InitStructure.DMA_Priority = DMA_Priority_VeryHigh;
    // 禁止内存到内存的传输
    DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;


    // 内存地址(要传输的变量的指针)
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)ReceiveBuff;
    // 方向：从外设到内存
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
    // 传输大小
    DMA_InitStructure.DMA_BufferSize = USART_RX_BUFF_SIZE;
    // DMA模式,循环模式
    DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;
    // 配置DMA通道
    DMA_Init(USART_RX_DMA_CHANNEL, &DMA_InitStructure);
    // 使能传输完成中断
    //DMA_ITConfig(USART_RX_DMA_CHANNEL, DMA_IT_TC, ENABLE);
    // 使能DMA
    DMA_Cmd (USART_RX_DMA_CHANNEL,ENABLE);

}

/**
 * @brief  USART GPIO 配置,工作参数配置
 * @param  无
 * @retval 无
 */
void USART_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    USART_InitTypeDef USART_InitStructure;

    // 打开串口GPIO的时钟
    DEBUG_USART_GPIO_APBxClkCmd(DEBUG_USART_GPIO_CLK, ENABLE);

    // 打开串口外设的时钟
    DEBUG_USART_APBxClkCmd(DEBUG_USART_CLK, ENABLE);

    // 将USART Tx的GPIO配置为推挽复用模式
    GPIO_InitStructure.GPIO_Pin = DEBUG_USART_TX_GPIO_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(DEBUG_USART_TX_GPIO_PORT, &GPIO_InitStructure);

    // 将USART Rx的GPIO配置为浮空输入模式
    GPIO_InitStructure.GPIO_Pin = DEBUG_USART_RX_GPIO_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(DEBUG_USART_RX_GPIO_PORT, &GPIO_InitStructure);

    // 配置串口的工作参数
    // 配置波特率
    USART_InitStructure.USART_BaudRate = DEBUG_USART_BAUDRATE;
    // 配置 针数据字长
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    // 配置停止位
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    // 配置校验位
    USART_InitStructure.USART_Parity = USART_Parity_No ;
    // 配置硬件流控制
    USART_InitStructure.USART_HardwareFlowControl =
        USART_HardwareFlowControl_None;
    // 配置工作模式，收发一起
    USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;
    // 完成串口的初始化配置
    USART_Init(DEBUG_USARTx, &USART_InitStructure);

    // 串口中断优先级配置
    NVIC_Configuration();


    // 配置串口接收DMA
    USARTx_DMA_RX_TX_Config();
    // 开启 串口空闲IDEL 中断
    USART_ITConfig(DEBUG_USARTx, USART_IT_IDLE, ENABLE);
    // 开启串口DMA接收
    USART_DMACmd(DEBUG_USARTx, USART_DMAReq_Rx, ENABLE);


    // 使能串口
    USART_Cmd(DEBUG_USARTx, ENABLE);
}

/*****************  发送一个字节 **********************/
void Usart_SendByte( USART_TypeDef * pUSARTx, uint8_t ch)
{
    /* 发送一个字节数据到USART */
    USART_SendData(pUSARTx,ch);

    /* 等待发送数据寄存器为空 */
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);
}

/****************** 发送8位的数组 ************************/
void Usart_SendArray( USART_TypeDef * pUSARTx, uint8_t *array, uint16_t num)
{
    uint8_t i;

    for(i=0; i<num; i++)
    {
        /* 发送一个字节数据到USART */
        Usart_SendByte(pUSARTx,array[i]);

    }
    /* 等待发送完成 */
    while(USART_GetFlagStatus(pUSARTx,USART_FLAG_TC)==RESET);
}

/*****************  发送字符串 **********************/
void Usart_SendString( USART_TypeDef * pUSARTx, char *str)
{
    unsigned int k=0;
    do
    {
        Usart_SendByte( pUSARTx, *(str + k) );
        k++;
    } while(*(str + k)!='\0');

    /* 等待发送完成 */
    while(USART_GetFlagStatus(pUSARTx,USART_FLAG_TC)==RESET)
    {}
}

/*****************  发送一个16位数 **********************/
void Usart_SendHalfWord( USART_TypeDef * pUSARTx, uint16_t ch)
{
    uint8_t temp_h, temp_l;

    /* 取出高八位 */
    temp_h = (ch&0XFF00)>>8;
    /* 取出低八位 */
    temp_l = ch&0XFF;

    /* 发送高八位 */
    USART_SendData(pUSARTx,temp_h);
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);

    /* 发送低八位 */
    USART_SendData(pUSARTx,temp_l);
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET);
}

///重定向c库函数printf到串口，重定向后可使用printf函数
int fputc(int ch, FILE *f)
{
    /* 发送一个字节数据到串口 */
    USART_SendData(DEBUG_USARTx, (uint8_t) ch);

    /* 等待发送完毕 */
    while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);

    return (ch);
}

///重定向c库函数scanf到串口，重写向后可使用scanf、getchar等函数
int fgetc(FILE *f)
{
    /* 等待串口输入数据 */
    while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_RXNE) == RESET);
    return (int)USART_ReceiveData(DEBUG_USARTx);
}

```

## 中断
- `stm32f10x_it.c`

```c
#include "stm32f10x_it.h"
/* FreeRTOS头文件 */
#include "FreeRTOS.h"
#include "task.h"
#include "semphr.h"

#include "my_usart.h"

extern uint8_t ReceiveBuff[USART_RX_BUFF_SIZE];
extern QueueHandle_t Test_Queue;
/**
  * @brief  串口中断服务函数
  * @param  无
  * @retval 无
  */
void DEBUG_USART_IRQHandler(void)
{
    uint32_t ulReturn;
    BaseType_t pxHigherPriorityTaskWoken;
    /* 进入临界段，临界段可以嵌套 */
    ulReturn = taskENTER_CRITICAL_FROM_ISR();

    //数据帧接收完毕
    if ( USART_GetITStatus( DEBUG_USARTx, USART_IT_IDLE ) == SET )
    {
        DMA_Cmd(USART_RX_DMA_CHANNEL,DISABLE);
        DMA_SetCurrDataCounter(USART_RX_DMA_CHANNEL,USART_RX_BUFF_SIZE);
        DMA_Cmd(USART_RX_DMA_CHANNEL,ENABLE);
        xQueueSendFromISR(Test_Queue, /* 消息队列的句柄 */
                          ReceiveBuff,/* 发送的消息内容 */
                          &pxHigherPriorityTaskWoken);
        //如果需要的话进行一次任务切换
        portYIELD_FROM_ISR(pxHigherPriorityTaskWoken);
        memset(ReceiveBuff,0,USART_RX_BUFF_SIZE);/* 清零 */
        //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
        USART_ReceiveData(DEBUG_USARTx);
    }
    /* 退出临界段 */
    taskEXIT_CRITICAL_FROM_ISR( ulReturn );
}

```

## 主程序
- `main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "semphr.h"

#include "my_gpio.h"
#include "my_usart.h"

/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TaskHandle_t LED_Task_Handle = NULL;/* LED_Task任务句柄 */

QueueHandle_t Test_Queue =NULL;

static void AppTaskCreate(void);/* 用于创建任务 */


static void LED_Task(void* pvParameters);/* LED_Task 任务实现 */

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-INTERRUPT!\r\n");
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

    /* 创建 BinarySem */
    Test_Queue = xQueueCreate((UBaseType_t ) 4,/* 消息队列的长度 */
                              (UBaseType_t ) USART_RX_BUFF_SIZE);/* 消息的大小 */

    if(NULL != Test_Queue)
        printf("BinarySem SUCCESS!\n");
    /* 创建Take_Task任务 */
    xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                          (const char*    )"LED_Task",/* 任务名字 */
                          (uint16_t       )512,   /* 任务栈大小 */
                          (void*          )NULL,	/* 任务入口函数参数 */
                          (UBaseType_t    )2,	    /* 任务的优先级 */
                          (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
    if(pdPASS == xReturn)
        printf("LED_Task  SUCCESS!\r\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  LED_Task主体
 * @param  parameter 参数
 * @retval 无
 */
static void LED_Task(void* parameter)
{
    BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
    uint8_t r_queue[USART_RX_BUFF_SIZE];
    while(1)
    {
        xReturn = xQueueReceive( Test_Queue,    /* 消息队列的句柄 */
                                 r_queue,      /* 发送的消息内容 */
                                 portMAX_DELAY); /* 等待时间 一直等 */

        if(pdPASS == xReturn)
        {
            printf("USART_RECEIVE: %s!\n",r_queue);
            memset(r_queue,0,USART_RX_BUFF_SIZE);/* 清零 */
            LED1_TOGGLE;
        }
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
- 使用串口助手向开发板发送数据，会看到回传数据和LED翻转