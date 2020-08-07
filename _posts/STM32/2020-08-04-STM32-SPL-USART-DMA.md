---
title: STM32标准外设库串口使用DMA发送/接收不定长数据
categories: STM32 USART DMA
tags: STM32 SPL USART UART QUEUE DMA
description: SPL库串口通信之DMA发送/接收不定长数据
---
# 添加库函数
- 打开 `Manage Run-Time Environment`
- 选择`Device->StdPeriph Drivers->DMA`

# 修改串口初始化
## 添加DMA相关宏
- `my_usart.h`

```c
#ifndef __USART_H
#define __USART_H

#include "stm32f10x.h"
#include <stdio.h>
#include <string.h>
/**
  * 串口宏定义，不同的串口挂载的总线和IO不一样，移植时需要修改这几个宏
	* 1-修改总线时钟的宏，uart1挂载到apb2总线，其他uart挂载到apb1总线
	* 2-修改GPIO的宏
  */

// 串口1-USART1
#define DEBUG_USARTx USART1
#define DEBUG_USART_CLK RCC_APB2Periph_USART1
#define DEBUG_USART_APBxClkCmd RCC_APB2PeriphClockCmd
#define DEBUG_USART_BAUDRATE 9600

// USART GPIO 引脚宏定义
#define DEBUG_USART_GPIO_CLK (RCC_APB2Periph_GPIOA)
#define DEBUG_USART_GPIO_APBxClkCmd RCC_APB2PeriphClockCmd

#define DEBUG_USART_TX_GPIO_PORT GPIOA
#define DEBUG_USART_TX_GPIO_PIN GPIO_Pin_9
#define DEBUG_USART_RX_GPIO_PORT GPIOA
#define DEBUG_USART_RX_GPIO_PIN GPIO_Pin_10

#define DEBUG_USART_IRQ USART1_IRQn
#define DEBUG_USART_IRQHandler USART1_IRQHandler

//DMA相关宏定义
#define USE_USART_DMA_RX  1
#define USE_USART_DMA_TX  1
#if USE_USART_DMA_RX
#define USART_RX_BUFF_SIZE 128
#define USART_RX_DMA_CHANNEL DMA1_Channel5
#define USART_RX_DMA_IRQ DMA1_Channel5_IRQn
#define USART_RX_DMA_IRQHandler DMA1_Channel5_IRQHandler
#define USART_RX_DMA_IT_TC DMA1_IT_TC5
#endif
#if USE_USART_DMA_TX
#define USART_TX_BUFF_SIZE 50
#define USART_TX_DMA_CHANNEL DMA1_Channel4
#define USART_TX_DMA_FLAG_TC DMA1_FLAG_TC4
#endif
#if USE_USART_DMA_RX  || USE_USART_DMA_TX
#define USART_DR_ADDRESS (USART1_BASE+0x04)
#define USART_DMA_CLK   RCC_AHBPeriph_DMA1
#endif


void USART_Config(void);
void Usart_SendByte(USART_TypeDef *pUSARTx, uint8_t ch);
/****************** 发送8位的数组 ************************/
void Usart_SendArray(USART_TypeDef *pUSARTx, uint8_t *array, uint16_t num);

#endif /* __USART_H */

```

## 修改初始化程序

- `my_usart.c`

```c
#include "my_usart.h"
#if USE_USART_DMA_TX
uint8_t SendBuff[USART_TX_BUFF_SIZE];
#endif
#if USE_USART_DMA_RX
uint8_t ReceiveBuff[USART_RX_BUFF_SIZE];
#endif
/**
 * @brief  配置嵌套向量中断控制器NVIC
 * @param  无
 * @retval 无
 */
static void NVIC_Configuration(void)
{
    NVIC_InitTypeDef NVIC_InitStructure;

    /* 嵌套向量中断控制器组选择 */
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);

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
#if USE_USART_DMA_RX
    /* 配置DMA中断源 */
    NVIC_InitStructure.NVIC_IRQChannel =  USART_RX_DMA_IRQ;
    /* 初始化配置NVIC */
    NVIC_Init(&NVIC_InitStructure);
#endif
}
#if USE_USART_DMA_RX || USE_USART_DMA_TX
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

#if USE_USART_DMA_RX
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
    DMA_ITConfig(USART_RX_DMA_CHANNEL, DMA_IT_TC, ENABLE);
    // 使能DMA
    DMA_Cmd (USART_RX_DMA_CHANNEL,ENABLE);
#endif

#if USE_USART_DMA_TX
    // 内存地址(要传输的变量的指针)
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)SendBuff;
    // 方向：从内存到外设
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralDST;
    // 传输大小
    DMA_InitStructure.DMA_BufferSize = USART_TX_BUFF_SIZE;
    // DMA模式,单次模式
    DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;
    // 配置DMA通道
    DMA_Init(USART_TX_DMA_CHANNEL, &DMA_InitStructure);
#endif
}
#endif

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
    USART_InitStructure.USART_Parity = USART_Parity_No;
    // 配置硬件流控制
    USART_InitStructure.USART_HardwareFlowControl =USART_HardwareFlowControl_None;
    // 配置工作模式，收发一起
    USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;
    // 完成串口的初始化配置
    USART_Init(DEBUG_USARTx, &USART_InitStructure);

    // 串口中断优先级配置
    NVIC_Configuration();
#if USE_USART_DMA_RX
    // 配置串口接收DMA
    USARTx_DMA_RX_TX_Config();
    // 开启 串口空闲IDEL 中断
    USART_ITConfig(DEBUG_USARTx, USART_IT_IDLE, ENABLE);
    // 开启串口DMA接收
    USART_DMACmd(DEBUG_USARTx, USART_DMAReq_Rx, ENABLE);

#else
    // 使能串口接收中断
    USART_ITConfig(DEBUG_USARTx, USART_IT_RXNE, ENABLE);
    //使能串口总线空闲中断
    USART_ITConfig(DEBUG_USARTx, USART_IT_IDLE, ENABLE);
#endif

#if USE_USART_DMA_TX
    // 配置串口发送DMA
    USARTx_DMA_RX_TX_Config();
    // 开启串口DMA发送
    USART_DMACmd(DEBUG_USARTx, USART_DMAReq_Tx, ENABLE);
#endif

    // 使能串口
    USART_Cmd(DEBUG_USARTx, ENABLE);
}
#if USE_USART_DMA_TX
/**
 * @brief  USART GPIO 配置,工作参数配置
 * @param  无
 * @retval 无
 */
static void Usart_DMA_SendArray(DMA_Channel_TypeDef *Channel,uint8_t *array, uint16_t num)
{
    DMA_Cmd(Channel, DISABLE);                      //关闭DMA传输
    DMA_SetCurrDataCounter(Channel,num);
    memcpy(SendBuff,array,num);
    DMA_Cmd(Channel, ENABLE);
}
#endif


/*****************  发送一个字节 **********************/
void Usart_SendByte(USART_TypeDef *pUSARTx, uint8_t ch)
{
    /* 发送一个字节数据到USART */
    USART_SendData(pUSARTx, ch);

    /* 等待发送数据寄存器为空 */
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TXE) == RESET)
        ;
}

/****************** 发送8位的数组 ************************/
void Usart_SendArray(USART_TypeDef *pUSARTx, uint8_t *array, uint16_t num)
{
#if USE_USART_DMA_TX
    if(pUSARTx==DEBUG_USARTx)
    {
        Usart_DMA_SendArray(USART_TX_DMA_CHANNEL,array,num);
        while(DMA_GetFlagStatus(USART_TX_DMA_FLAG_TC)==RESET)
        {}
    }
#else
    uint8_t i;
    for (i = 0; i < num; i++)
    {
        /* 发送一个字节数据到USART */
        Usart_SendByte(pUSARTx, array[i]);
    }
    /* 等待发送完成 */
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TC) == RESET)
        ;
#endif
}
///重定向c库函数printf到串口，重定向后可使用printf函数
int fputc(int ch, FILE *f)
{
    /* 发送一个字节数据到串口 */
		USART_SendData(DEBUG_USARTx, (uint8_t) ch);
		
		/* 等待发送完毕 */
		while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET)
		{}
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
### DMA结构体分析

```c
/** 
  * @brief  DMA Init structure definition
  */

typedef struct
{
  uint32_t DMA_PeripheralBaseAddr; /*!< Specifies the peripheral base address for DMAy Channelx. */

  uint32_t DMA_MemoryBaseAddr;     /*!< Specifies the memory base address for DMAy Channelx. */

  uint32_t DMA_DIR;                /*!< Specifies if the peripheral is the source or destination.
                                        This parameter can be a value of @ref DMA_data_transfer_direction */

  uint32_t DMA_BufferSize;         /*!< Specifies the buffer size, in data unit, of the specified Channel. 
                                        The data unit is equal to the configuration set in DMA_PeripheralDataSize
                                        or DMA_MemoryDataSize members depending in the transfer direction. */

  uint32_t DMA_PeripheralInc;      /*!< Specifies whether the Peripheral address register is incremented or not.
                                        This parameter can be a value of @ref DMA_peripheral_incremented_mode */

  uint32_t DMA_MemoryInc;          /*!< Specifies whether the memory address register is incremented or not.
                                        This parameter can be a value of @ref DMA_memory_incremented_mode */

  uint32_t DMA_PeripheralDataSize; /*!< Specifies the Peripheral data width.
                                        This parameter can be a value of @ref DMA_peripheral_data_size */

  uint32_t DMA_MemoryDataSize;     /*!< Specifies the Memory data width.
                                        This parameter can be a value of @ref DMA_memory_data_size */

  uint32_t DMA_Mode;               /*!< Specifies the operation mode of the DMAy Channelx.
                                        This parameter can be a value of @ref DMA_circular_normal_mode.
                                        @note: The circular buffer mode cannot be used if the memory-to-memory
                                              data transfer is configured on the selected Channel */

  uint32_t DMA_Priority;           /*!< Specifies the software priority for the DMAy Channelx.
                                        This parameter can be a value of @ref DMA_priority_level */

  uint32_t DMA_M2M;                /*!< Specifies if the DMAy Channelx will be used in memory-to-memory transfer.
                                        This parameter can be a value of @ref DMA_memory_to_memory */
}DMA_InitTypeDef;
```

- 外设地址`DMA_PeripheralBaseAddr`
- 存储器地址`DMA_MemoryBaseAddr`
- 传输方向`DMA_DIR`

```c
/** @defgroup DMA_data_transfer_direction 
  * @{
  */

#define DMA_DIR_PeripheralDST              ((uint32_t)0x00000010) //存储器到外设
#define DMA_DIR_PeripheralSRC              ((uint32_t)0x00000000) //外设到存储器
```

- 传输数目
- 外设地址增量模式

```c
/** @defgroup DMA_peripheral_incremented_mode 
  * @{
  */

#define DMA_PeripheralInc_Enable           ((uint32_t)0x00000040)//外设地址自增使能
#define DMA_PeripheralInc_Disable          ((uint32_t)0x00000000)//禁止外设地址自增
```

- 存储器地址增量模式

```c
/** @defgroup DMA_memory_incremented_mode 
  * @{
  */

#define DMA_MemoryInc_Enable               ((uint32_t)0x00000080) //存储器地址自增使能
#define DMA_MemoryInc_Disable              ((uint32_t)0x00000000)//禁止存储器地址自增
```

- 外设数据宽度

```c
/** @defgroup DMA_peripheral_data_size 
  * @{
  */

#define DMA_PeripheralDataSize_Byte        ((uint32_t)0x00000000) //字节(8位)
#define DMA_PeripheralDataSize_HalfWord    ((uint32_t)0x00000100) //半字(16位)
#define DMA_PeripheralDataSize_Word        ((uint32_t)0x00000200) //字(32位)
```

- 存储器数据宽度

```c
/** @defgroup DMA_memory_data_size 
  * @{
  */

#define DMA_MemoryDataSize_Byte            ((uint32_t)0x00000000) //字节(8位)
#define DMA_MemoryDataSize_HalfWord        ((uint32_t)0x00000400) //半字(16位)
#define DMA_MemoryDataSize_Word            ((uint32_t)0x00000800) //字(32位)
```

- 模式选择

```c
/** @defgroup DMA_circular_normal_mode 
  * @{
  */

#define DMA_Mode_Circular                  ((uint32_t)0x00000020) //循环模式
#define DMA_Mode_Normal                    ((uint32_t)0x00000000) //单次模式
```

- 通道优先级

```c
/** @defgroup DMA_priority_level 
  * @{
  */

#define DMA_Priority_VeryHigh              ((uint32_t)0x00003000) //非常高
#define DMA_Priority_High                  ((uint32_t)0x00002000) //高
#define DMA_Priority_Medium                ((uint32_t)0x00001000) //中
#define DMA_Priority_Low                   ((uint32_t)0x00000000) //低
```

- 存储器到存储器模式

```c
/** @defgroup DMA_memory_to_memory 
  * @{
  */

#define DMA_M2M_Enable                     ((uint32_t)0x00004000) //使能存储器到存储器
#define DMA_M2M_Disable                    ((uint32_t)0x00000000) //禁止存储器到存储器
```
### DMA通道选择

|外设|DMA1_Channel1|DMA1_Channel2|DMA1_Channel3|DMA1_Channel4|DMA1_Channel5|DMA1_Channel6|DMA1_Channel7|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|ADC1|ADC1|
|SPI/I<sup>2</sup>S||SPI1_RX|SPI1_TX|SPI/I<sup>2</sup>S2_RX|SPI/I<sup>2</sup>S_TX|
|USART||USART3_TX|USART3_RX|USART1_TX|USART1_RX|USART2_RX|USART2_TX|
|I<sup>2</sup>C||||I<sup>2</sup>C2_TX|I<sup>2</sup>C2_RX|I<sup>2</sup>C1_TX|I<sup>2</sup>C1_RX|
|TIM1||TIM1_CH1|TIM1_CH2|TIM1_CH4<BR/>TIM1_TRIG<BR/>TIM1_COM|TIM1_UP|TIM1_CH3|
|TIM2|TIM2_CH3|TIM2_UP|||TIM2_CH1||TIM2_CH2<BR/>TIM2_CH4|
|TIM3||TIM3_CH3|TIM3_CH4<BR/>TIM3_UP|||TIM3_CH1<BR/>TIM3_TRIG|
|TIM4|TIM4_CH1|||TIM4_CH2|TIM4_CH3||TIM4_UP|


|外设|DMA2_Channel1|DMA2_Channel2|DMA2_Channel3|DMA2_Channel4|DMA2_Channel5|
|:----:|:----:|:----:|:----:|:----:|:----:|
|ADC3|||||ADC3|
|SPI/I<sup>2</sup>S3|SPI/I<sup>2</sup>S3_RX|SPI/I<sup>2</sup>S3_TX|
|UART4|||UART4_RX||UART4_TX|
|SDIO||||SDIO|
|TIM5|TIM5_CH4<BR/>TIM5_TRIG|TIM5_CH3<BR/>TIM5_UP||TIM5_CH2|TIM5_CH1|
|TIM6/DAC_Channel_1|||TIM6_UP/DAC_Channel_1|
|TIM7/DAC_Channel_2||||TIM7_UP/DAC_Channel_2|
|TIM8|TIM8_CH3<BR/>TIM8_UP|TIM8_CH4<BR/>TIM8_TRIG<BR/>TIM8_COM|TIM8_CH1||TIM8_CH2|



# 发送函数修改

```c
#if USE_USART_DMA_TX
/**
 * @brief  发送8位的数组
 * @param  无
 * @retval 无
 */
static void Usart_DMA_SendArray(DMA_Channel_TypeDef *Channel,uint8_t *array, uint16_t num)
{
    DMA_Cmd(Channel, DISABLE);                      //关闭DMA传输
    DMA_SetCurrDataCounter(Channel,num);            //必须关闭DMA才可更改
    memcpy(SendBuff,array,num);                     //发送缓存
    DMA_Cmd(Channel, ENABLE);                       //开启DMA
}
#endif
/****************** 发送8位的数组 ************************/
void Usart_SendArray(USART_TypeDef *pUSARTx, uint8_t *array, uint16_t num)
{
#if USE_USART_DMA_TX
    if(pUSARTx==DEBUG_USARTx)
    {
        Usart_DMA_SendArray(USART_TX_DMA_CHANNEL,array,num);
        while(DMA_GetFlagStatus(USART_TX_DMA_FLAG_TC)==RESET) //等待发送完成
        {}
    }
#else
    uint8_t i;
    for (i = 0; i < num; i++)
    {
        /* 发送一个字节数据到USART */
        Usart_SendByte(pUSARTx, array[i]);
    }
    /* 等待发送完成 */
    while (USART_GetFlagStatus(pUSARTx, USART_FLAG_TC) == RESET)
        ;
#endif
}

```

# 修改串口中断函数
## 添加接收缓冲区外部定义

```c
#if USE_USART_DMA_RX
extern uint8_t ReceiveBuff[USART_RX_BUFF_SIZE];
#endif

```

## 添加DMA完成中断

```c
#if USE_USART_DMA_RX
/**
  * @brief  串口DMA接收中断
  * @param  无
  * @retval 无
  */
void USART_RX_DMA_IRQHandler(void)
{
    QUEUE_DATA_TYPE *data_p;
    if(DMA_GetITStatus(USART_RX_DMA_IT_TC)!=RESET)
    {
        DMA_ClearITPendingBit(USART_RX_DMA_IT_TC);//清除中断标志
        DMA_Cmd(USART_RX_DMA_CHANNEL,DISABLE);    //关闭DMA
        /*获取写缓冲区指针，准备写入新数据*/
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            data_p->len=USART_RX_BUFF_SIZE-DMA_GetCurrDataCounter(USART_RX_DMA_CHANNEL);
            memcpy(data_p->head,ReceiveBuff,data_p->len);
            /*写入缓冲区完毕*/
            cbWriteFinish(&rx_queue);
        }
        DMA_SetCurrDataCounter(USART_RX_DMA_CHANNEL,USART_RX_BUFF_SIZE);//重新改变DMA接收计数
        DMA_Cmd(USART_RX_DMA_CHANNEL,ENABLE);//开启DMA
    }
}
#endif
```

## 更改串口中断函数

```c
/**
  * @brief  串口中断服务函数
  * @param  无
  * @retval 无
  */
void DEBUG_USART_IRQHandler(void)
{
    QUEUE_DATA_TYPE *data_p;
#if !USE_USART_DMA_RX
    uint8_t ucCh;
    if (USART_GetITStatus(DEBUG_USARTx, USART_IT_RXNE) != RESET)
    {
        ucCh = USART_ReceiveData(DEBUG_USARTx);
        /*获取写缓冲区指针，准备写入新数据*/
        data_p = cbWrite(&rx_queue);

        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            //往缓冲区写入数据，如使用串口接收、dma写入等方式
            *(data_p->head + data_p->len) = ucCh;
            if (++data_p->len >= QUEUE_NODE_DATA_LEN)
            {
                cbWriteFinish(&rx_queue);
            }
        }
        else
            return;
    }
#endif
    //数据帧接收完毕
    if (USART_GetITStatus(DEBUG_USARTx, USART_IT_IDLE) == SET)
    {
#if USE_USART_DMA_RX
        DMA_Cmd(USART_RX_DMA_CHANNEL,DISABLE);
        /*获取写缓冲区指针，准备写入新数据*/
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            data_p->len=USART_RX_BUFF_SIZE-DMA_GetCurrDataCounter(USART_RX_DMA_CHANNEL);
            if(data_p->len>0)
            {
                memcpy(data_p->head,ReceiveBuff,data_p->len);
                /*写入缓冲区完毕*/
                cbWriteFinish(&rx_queue);
            }
        }
        DMA_SetCurrDataCounter(USART_RX_DMA_CHANNEL,USART_RX_BUFF_SIZE);
        DMA_Cmd(USART_RX_DMA_CHANNEL,ENABLE);
#else
        /*写入缓冲区完毕*/
        cbWriteFinish(&rx_queue);
#endif
        //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
        USART_ReceiveData(DEBUG_USARTx);
    }
}
```
# 调试
- 编译下载
- 验证
- 和原来的串口通信效果一样