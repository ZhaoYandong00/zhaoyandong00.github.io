---
title: 创建静态任务RTOS篇2
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS创建静态任务
---
# 静态任务常用函数
## 创建静态任务

```c
xTaskCreateStatic(	TaskFunction_t pxTaskCode, 
                    const char * const pcName,	/*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                    const uint32_t ulStackDepth,
                    void * const pvParameters,
                    UBaseType_t uxPriority,
                    StackType_t * const puxStackBuffer,
                    StaticTask_t * const pxTaskBuffer )
```
- 任务函数`pxTaskCode`
- 任务名字`pcName`
- 任务堆栈大小`ulStackDepth`,单位为字
- 传递给任务的函数`pvParameters`
- 任务优先级`uxPriority`
- 任务堆栈起始地址`puxStackBuffer`
- 任务控制块指针`pxTaskBuffer`

## 启动任务调度

`void vTaskStartScheduler( void )`

## 删除任务

`void vTaskDelete( TaskHandle_t xTaskToDelete )`

# 硬件初始化配置
- 复制SPL时的gpio和usart文件到工程目录
- `my_gpio.c`

```c
#include "my_gpio.h"

/**
 * @brief  初始化控制LED的IO
 * @param  无
 * @retval 无
 */
void LED_GPIO_Config(void)
{
    /*定义一个GPIO_InitTypeDef类型的结构体*/
    GPIO_InitTypeDef GPIO_InitStructure;

    /*开启LED相关的GPIO外设时钟*/
    RCC_APB2PeriphClockCmd(LED1_GPIO_CLK | LED2_GPIO_CLK, ENABLE);
    /*选择要控制的GPIO引脚*/
    GPIO_InitStructure.GPIO_Pin = LED1_GPIO_PIN;

    /*设置引脚模式为通用推挽输出*/
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;

    /*设置引脚速率为50MHz */
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;

    /*调用库函数，初始化GPIO*/
    GPIO_Init(LED1_GPIO_PORT, &GPIO_InitStructure);

    /*选择要控制的GPIO引脚*/
    GPIO_InitStructure.GPIO_Pin = LED2_GPIO_PIN;

    /*调用库函数，初始化GPIO*/
    GPIO_Init(LED2_GPIO_PORT, &GPIO_InitStructure);

    /* 关闭所有led灯	*/
    GPIO_SetBits(LED1_GPIO_PORT, LED1_GPIO_PIN);

    /* 关闭所有led灯	*/
    GPIO_SetBits(LED2_GPIO_PORT, LED2_GPIO_PIN);
}

```

- `my_usart.c`

```c
#include "my_usart.h"

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
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    /* 子优先级 */
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    /* 使能中断 */
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    /* 初始化配置NVIC */
    NVIC_Init(&NVIC_InitStructure);
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

    // 使能串口接收中断
    USART_ITConfig(DEBUG_USARTx, USART_IT_RXNE, ENABLE);
    //使能串口总线空闲中断
    USART_ITConfig ( DEBUG_USARTx, USART_IT_IDLE, ENABLE );
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

# 创建静态任务
- 在`FreeRTOSConfig.h`文件中修改静态内存分配改为1
- `main.c`文件

```c
#include "FreeRTOS.h"
#include "task.h"
#include "my_gpio.h"
#include "my_usart.h"
/* 创建任务句柄 */
static TaskHandle_t AppTaskCreate_Handle;
/* LED任务句柄 */
static TaskHandle_t LED_Task_Handle;

/* AppTaskCreate任务任务堆栈 */
static StackType_t AppTaskCreate_Stack[128];
/* LED任务堆栈 */
static StackType_t LED_Task_Stack[128];

/* AppTaskCreate 任务控制块 */
static StaticTask_t AppTaskCreate_TCB;
/* AppTaskCreate 任务控制块 */
static StaticTask_t LED_Task_TCB;

/* 空闲任务任务堆栈 */
static StackType_t Idle_Task_Stack[configMINIMAL_STACK_SIZE];
/* 定时器任务堆栈 */
static StackType_t Timer_Task_Stack[configTIMER_TASK_STACK_DEPTH];

/* 空闲任务控制块 */
static StaticTask_t Idle_Task_TCB;
/* 定时器任务控制块 */
static StaticTask_t Timer_Task_TCB;

static void AppTaskCreate(void);/* 用于创建任务 */
static void BSP_Init(void);/* 用于初始化板载相关资源 */
static void LED_Task(void* pvParameters);/* LED_Task任务实现 */

void vApplicationGetTimerTaskMemory(StaticTask_t **ppxTimerTaskTCBBuffer,StackType_t **ppxTimerTaskStackBuffer,uint32_t *pulTimerTaskStackSize);

void vApplicationGetIdleTaskMemory(StaticTask_t **ppxIdleTaskTCBBuffer,StackType_t **ppxIdleTaskStackBuffer,uint32_t *pulIdleTaskStackSize);

int main(void)
{
    BSP_Init();
    printf("FreeRTOS-STATIC!\r\n");
    /* 创建 AppTaskCreate 任务 */
    AppTaskCreate_Handle = xTaskCreateStatic((TaskFunction_t	)AppTaskCreate,		//任务函数
                           (const char* 	)"AppTaskCreate",		//任务名称
                           (uint32_t 		)128,	//任务堆栈大小
                           (void* 		  	)NULL,				//传递给任务函数的参数
                           (UBaseType_t 	)3, 	//任务优先级
                           (StackType_t*   )AppTaskCreate_Stack,	//任务堆栈
                           (StaticTask_t*  )&AppTaskCreate_TCB);	//任务控制块

    if(NULL != AppTaskCreate_Handle)/* 创建成功 */
        vTaskStartScheduler();   /* 启动任务，开启调度 */

    while(1);   /* 正常不会执行到这里 */
}
/**********************************************************************
  * @ 函数名  ： LED_Task
  * @ 功能说明： LED_Task任务主体
  * @ 参数    ：
  * @ 返回值  ： 无
  ********************************************************************/
static void LED_Task(void* parameter)
{
    while (1)
    {
        LED1(ON);
        vTaskDelay(500);   /* 延时500个tick */
        printf("LED_Task Running,LED1_ON\r\n");

        LED1(OFF);
        vTaskDelay(500);   /* 延时500个tick */
        printf("LED_Task Running,LED1_OFF\r\n");
    }
}
/**
 * @brief  为了方便管理，所有的任务创建函数都放在这个函数里面
 * @param  无
 * @retval 无
 */
static void AppTaskCreate(void)
{
    taskENTER_CRITICAL();           //进入临界区
    /* 创建LED_Task任务 */
    LED_Task_Handle = xTaskCreateStatic((TaskFunction_t	)LED_Task,		//任务函数
                                        (const char* 	)"LED_Task",		//任务名称
                                        (uint32_t 		)128,					//任务堆栈大小
                                        (void* 		  	)NULL,				//传递给任务函数的参数
                                        (UBaseType_t 	)4, 				//任务优先级
                                        (StackType_t*   )LED_Task_Stack,	//任务堆栈
                                        (StaticTask_t*  )&LED_Task_TCB);	//任务控制块

    if(NULL != LED_Task_Handle)/* 创建成功 */
        printf("LED_Task SUCCESS!\n");
    else
        printf("LED_Task FAIL!\n");

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
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
}
/**
  * @brief  获取空闲任务的任务堆栈和任务控制块内存
	*	@param	ppxTimerTaskTCBBuffer	:		任务控制块内存
	*	@param	ppxTimerTaskStackBuffer	:	任务堆栈内存
	*	@param	pulTimerTaskStackSize	:		任务堆栈大小
  */
void vApplicationGetIdleTaskMemory(StaticTask_t **ppxIdleTaskTCBBuffer,
                                   StackType_t **ppxIdleTaskStackBuffer,
                                   uint32_t *pulIdleTaskStackSize)
{
    *ppxIdleTaskTCBBuffer=&Idle_Task_TCB;/* 任务控制块内存 */
    *ppxIdleTaskStackBuffer=Idle_Task_Stack;/* 任务堆栈内存 */
    *pulIdleTaskStackSize=configMINIMAL_STACK_SIZE;/* 任务堆栈大小 */
}

/**
  * @brief  获取定时器任务的任务堆栈和任务控制块内存
	*	@param	ppxTimerTaskTCBBuffer	:		任务控制块内存
	*	@param	ppxTimerTaskStackBuffer	:	任务堆栈内存
	*	@param	pulTimerTaskStackSize	:		任务堆栈大小
  */
void vApplicationGetTimerTaskMemory(StaticTask_t **ppxTimerTaskTCBBuffer,
                                    StackType_t **ppxTimerTaskStackBuffer,
                                    uint32_t *pulTimerTaskStackSize)
{
    *ppxTimerTaskTCBBuffer=&Timer_Task_TCB;/* 任务控制块内存 */
    *ppxTimerTaskStackBuffer=Timer_Task_Stack;/* 任务堆栈内存 */
    *pulTimerTaskStackSize=configTIMER_TASK_STACK_DEPTH;/* 任务堆栈大小 */
}

```
# 调试
- 编译下载到开发板
- 可以看到LED闪烁