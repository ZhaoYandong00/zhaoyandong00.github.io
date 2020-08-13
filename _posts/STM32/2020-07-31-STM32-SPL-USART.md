---
title: STM32标准外设库串口通信
categories: STM32 USART UART 
tags: STM32 SPL USART UART
description: SPL库串口通信
---

# 添加库函数

* 打开 `Manage Run-Time Environment`
* 选择 `Device->StdPeriph Drivers->USART`
# 串口初始化

## 串口宏定义

* `my_usart.h`

``` c
#ifndef __USART_H
#define	__USART_H

#include "stm32f10x.h"
#include <stdio.h>

/**

  + 串口宏定义，不同的串口挂载的总线和IO不一样，移植时需要修改这几个宏
  + 1-修改总线时钟的宏，uart1挂载到apb2总线，其他uart挂载到apb1总线
  + 2-修改GPIO的宏

  */

// 串口1-USART1
#define  DEBUG_USARTx                   USART1
#define  DEBUG_USART_CLK                RCC_APB2Periph_USART1
#define  DEBUG_USART_APBxClkCmd         RCC_APB2PeriphClockCmd
#define  DEBUG_USART_BAUDRATE           9600

// USART GPIO 引脚宏定义
#define  DEBUG_USART_GPIO_CLK           (RCC_APB2Periph_GPIOA)
#define  DEBUG_USART_GPIO_APBxClkCmd    RCC_APB2PeriphClockCmd

#define  DEBUG_USART_TX_GPIO_PORT       GPIOA
#define  DEBUG_USART_TX_GPIO_PIN        GPIO_Pin_9
#define  DEBUG_USART_RX_GPIO_PORT       GPIOA
#define  DEBUG_USART_RX_GPIO_PIN        GPIO_Pin_10

#define  DEBUG_USART_IRQ                USART1_IRQn
#define  DEBUG_USART_IRQHandler         USART1_IRQHandler

void USART_Config(void);
void Usart_SendByte( USART_TypeDef * pUSARTx, uint8_t ch);
void Usart_SendString( USART_TypeDef * pUSARTx, char *str);
void Usart_SendHalfWord( USART_TypeDef * pUSARTx, uint16_t ch);
/****************** 发送8位的数组 ************************/
void Usart_SendArray( USART_TypeDef * pUSARTx, uint8_t *array, uint16_t num);

#endif /* __USART_H */

```

## 串口中断配置和初始化设置

* `my_usart.c`

``` c
#include "my_usart.h"

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

###  使能外设时钟和IO初始化

``` c
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
```

###  中断结构体分析

* 嵌套向量中断控制器组选择

``` c
/**

  + @brief  Configures the priority grouping: pre-emption priority and subpriority.
  + @param  NVIC_PriorityGroup: specifies the priority grouping bits length. 
  +   This parameter can be one of the following values:
  +     @arg NVIC_PriorityGroup_0: 0 bits for pre-emption priority
  +                                4 bits for subpriority
  +     @arg NVIC_PriorityGroup_1: 1 bits for pre-emption priority
  +                                3 bits for subpriority
  +     @arg NVIC_PriorityGroup_2: 2 bits for pre-emption priority
  +                                2 bits for subpriority
  +     @arg NVIC_PriorityGroup_3: 3 bits for pre-emption priority
  +                                1 bits for subpriority
  +     @arg NVIC_PriorityGroup_4: 4 bits for pre-emption priority
  +                                0 bits for subpriority
  + @retval None

  */
void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)
```

* 中断结构体

``` c
/** 

  + @brief  NVIC Init Structure definition  

  */

typedef struct
{
  uint8_t NVIC_IRQChannel;                    /*!< Specifies the IRQ channel to be enabled or disabled.
                                                   This parameter can be a value of @ref IRQn_Type 
                                                   (For the complete STM32 Devices IRQ Channels list, please
                                                    refer to stm32f10x.h file) */

  uint8_t NVIC_IRQChannelPreemptionPriority;  /*!< Specifies the pre-emption priority for the IRQ channel
                                                   specified in NVIC_IRQChannel. This parameter can be a value
                                                   between 0 and 15 as described in the table @ref NVIC_Priority_Table */

  uint8_t NVIC_IRQChannelSubPriority;         /*!< Specifies the subpriority level for the IRQ channel specified
                                                   in NVIC_IRQChannel. This parameter can be a value
                                                   between 0 and 15 as described in the table @ref NVIC_Priority_Table */

  FunctionalState NVIC_IRQChannelCmd;         /*!< Specifies whether the IRQ channel defined in NVIC_IRQChannel
                                                   will be enabled or disabled. 
                                                   This parameter can be set either to ENABLE or DISABLE */   
} NVIC_InitTypeDef;
```

* 中断源 `NVIC_IRQChannel`
宏定义如下:

``` c
/**
 * @brief STM32F10x Interrupt Number Definition, according to the selected device 
 *        in @ref Library_configuration_section 
 */
typedef enum IRQn
{
/******  Cortex-M3 Processor Exceptions Numbers ***************************************************/
  NonMaskableInt_IRQn         = -14,    /*!< 2 Non Maskable Interrupt                             */
  MemoryManagement_IRQn       = -12,    /*!< 4 Cortex-M3 Memory Management Interrupt              */
  BusFault_IRQn               = -11,    /*!< 5 Cortex-M3 Bus Fault Interrupt                      */
  UsageFault_IRQn             = -10,    /*!< 6 Cortex-M3 Usage Fault Interrupt                    */
  SVCall_IRQn                 = -5,     /*!< 11 Cortex-M3 SV Call Interrupt                       */
  DebugMonitor_IRQn           = -4,     /*!< 12 Cortex-M3 Debug Monitor Interrupt                 */
  PendSV_IRQn                 = -2,     /*!< 14 Cortex-M3 Pend SV Interrupt                       */
  SysTick_IRQn                = -1,     /*!< 15 Cortex-M3 System Tick Interrupt                   */

/******  STM32 specific Interrupt Numbers *********************************************************/
  WWDG_IRQn                   = 0,      /*!< Window WatchDog Interrupt                            */
  PVD_IRQn                    = 1,      /*!< PVD through EXTI Line detection Interrupt            */
  TAMPER_IRQn                 = 2,      /*!< Tamper Interrupt                                     */
  RTC_IRQn                    = 3,      /*!< RTC global Interrupt                                 */
  FLASH_IRQn                  = 4,      /*!< FLASH global Interrupt                               */
  RCC_IRQn                    = 5,      /*!< RCC global Interrupt                                 */
  EXTI0_IRQn                  = 6,      /*!< EXTI Line0 Interrupt                                 */
  EXTI1_IRQn                  = 7,      /*!< EXTI Line1 Interrupt                                 */
  EXTI2_IRQn                  = 8,      /*!< EXTI Line2 Interrupt                                 */
  EXTI3_IRQn                  = 9,      /*!< EXTI Line3 Interrupt                                 */
  EXTI4_IRQn                  = 10,     /*!< EXTI Line4 Interrupt                                 */
  DMA1_Channel1_IRQn          = 11,     /*!< DMA1 Channel 1 global Interrupt                      */
  DMA1_Channel2_IRQn          = 12,     /*!< DMA1 Channel 2 global Interrupt                      */
  DMA1_Channel3_IRQn          = 13,     /*!< DMA1 Channel 3 global Interrupt                      */
  DMA1_Channel4_IRQn          = 14,     /*!< DMA1 Channel 4 global Interrupt                      */
  DMA1_Channel5_IRQn          = 15,     /*!< DMA1 Channel 5 global Interrupt                      */
  DMA1_Channel6_IRQn          = 16,     /*!< DMA1 Channel 6 global Interrupt                      */
  DMA1_Channel7_IRQn          = 17,     /*!< DMA1 Channel 7 global Interrupt                      */

#ifdef STM32F10X_LD
  ADC1_2_IRQn                 = 18,     /*!< ADC1 and ADC2 global Interrupt                       */
  USB_HP_CAN1_TX_IRQn         = 19,     /*!< USB Device High Priority or CAN1 TX Interrupts       */
  USB_LP_CAN1_RX0_IRQn        = 20,     /*!< USB Device Low Priority or CAN1 RX0 Interrupts       */
  CAN1_RX1_IRQn               = 21,     /*!< CAN1 RX1 Interrupt                                   */
  CAN1_SCE_IRQn               = 22,     /*!< CAN1 SCE Interrupt                                   */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_IRQn               = 24,     /*!< TIM1 Break Interrupt                                 */
  TIM1_UP_IRQn                = 25,     /*!< TIM1 Update Interrupt                                */
  TIM1_TRG_COM_IRQn           = 26,     /*!< TIM1 Trigger and Commutation Interrupt               */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  USBWakeUp_IRQn              = 42      /*!< USB Device WakeUp from suspend through EXTI Line Interrupt */    
#endif /* STM32F10X_LD */  

#ifdef STM32F10X_LD_VL
  ADC1_IRQn                   = 18,     /*!< ADC1 global Interrupt                                */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_TIM15_IRQn         = 24,     /*!< TIM1 Break and TIM15 Interrupts                      */
  TIM1_UP_TIM16_IRQn          = 25,     /*!< TIM1 Update and TIM16 Interrupts                     */
  TIM1_TRG_COM_TIM17_IRQn     = 26,     /*!< TIM1 Trigger and Commutation and TIM17 Interrupt     */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  CEC_IRQn                    = 42,     /*!< HDMI-CEC Interrupt                                   */
  TIM6_DAC_IRQn               = 54,     /*!< TIM6 and DAC underrun Interrupt                      */
  TIM7_IRQn                   = 55      /*!< TIM7 Interrupt                                       */       
#endif /* STM32F10X_LD_VL */

#ifdef STM32F10X_MD
  ADC1_2_IRQn                 = 18,     /*!< ADC1 and ADC2 global Interrupt                       */
  USB_HP_CAN1_TX_IRQn         = 19,     /*!< USB Device High Priority or CAN1 TX Interrupts       */
  USB_LP_CAN1_RX0_IRQn        = 20,     /*!< USB Device Low Priority or CAN1 RX0 Interrupts       */
  CAN1_RX1_IRQn               = 21,     /*!< CAN1 RX1 Interrupt                                   */
  CAN1_SCE_IRQn               = 22,     /*!< CAN1 SCE Interrupt                                   */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_IRQn               = 24,     /*!< TIM1 Break Interrupt                                 */
  TIM1_UP_IRQn                = 25,     /*!< TIM1 Update Interrupt                                */
  TIM1_TRG_COM_IRQn           = 26,     /*!< TIM1 Trigger and Commutation Interrupt               */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  USBWakeUp_IRQn              = 42      /*!< USB Device WakeUp from suspend through EXTI Line Interrupt */  
#endif /* STM32F10X_MD */  

#ifdef STM32F10X_MD_VL
  ADC1_IRQn                   = 18,     /*!< ADC1 global Interrupt                                */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_TIM15_IRQn         = 24,     /*!< TIM1 Break and TIM15 Interrupts                      */
  TIM1_UP_TIM16_IRQn          = 25,     /*!< TIM1 Update and TIM16 Interrupts                     */
  TIM1_TRG_COM_TIM17_IRQn     = 26,     /*!< TIM1 Trigger and Commutation and TIM17 Interrupt     */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  CEC_IRQn                    = 42,     /*!< HDMI-CEC Interrupt                                   */
  TIM6_DAC_IRQn               = 54,     /*!< TIM6 and DAC underrun Interrupt                      */
  TIM7_IRQn                   = 55      /*!< TIM7 Interrupt                                       */       
#endif /* STM32F10X_MD_VL */

#ifdef STM32F10X_HD
  ADC1_2_IRQn                 = 18,     /*!< ADC1 and ADC2 global Interrupt                       */
  USB_HP_CAN1_TX_IRQn         = 19,     /*!< USB Device High Priority or CAN1 TX Interrupts       */
  USB_LP_CAN1_RX0_IRQn        = 20,     /*!< USB Device Low Priority or CAN1 RX0 Interrupts       */
  CAN1_RX1_IRQn               = 21,     /*!< CAN1 RX1 Interrupt                                   */
  CAN1_SCE_IRQn               = 22,     /*!< CAN1 SCE Interrupt                                   */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_IRQn               = 24,     /*!< TIM1 Break Interrupt                                 */
  TIM1_UP_IRQn                = 25,     /*!< TIM1 Update Interrupt                                */
  TIM1_TRG_COM_IRQn           = 26,     /*!< TIM1 Trigger and Commutation Interrupt               */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  USBWakeUp_IRQn              = 42,     /*!< USB Device WakeUp from suspend through EXTI Line Interrupt */
  TIM8_BRK_IRQn               = 43,     /*!< TIM8 Break Interrupt                                 */
  TIM8_UP_IRQn                = 44,     /*!< TIM8 Update Interrupt                                */
  TIM8_TRG_COM_IRQn           = 45,     /*!< TIM8 Trigger and Commutation Interrupt               */
  TIM8_CC_IRQn                = 46,     /*!< TIM8 Capture Compare Interrupt                       */
  ADC3_IRQn                   = 47,     /*!< ADC3 global Interrupt                                */
  FSMC_IRQn                   = 48,     /*!< FSMC global Interrupt                                */
  SDIO_IRQn                   = 49,     /*!< SDIO global Interrupt                                */
  TIM5_IRQn                   = 50,     /*!< TIM5 global Interrupt                                */
  SPI3_IRQn                   = 51,     /*!< SPI3 global Interrupt                                */
  UART4_IRQn                  = 52,     /*!< UART4 global Interrupt                               */
  UART5_IRQn                  = 53,     /*!< UART5 global Interrupt                               */
  TIM6_IRQn                   = 54,     /*!< TIM6 global Interrupt                                */
  TIM7_IRQn                   = 55,     /*!< TIM7 global Interrupt                                */
  DMA2_Channel1_IRQn          = 56,     /*!< DMA2 Channel 1 global Interrupt                      */
  DMA2_Channel2_IRQn          = 57,     /*!< DMA2 Channel 2 global Interrupt                      */
  DMA2_Channel3_IRQn          = 58,     /*!< DMA2 Channel 3 global Interrupt                      */
  DMA2_Channel4_5_IRQn        = 59      /*!< DMA2 Channel 4 and Channel 5 global Interrupt        */
#endif /* STM32F10X_HD */  

#ifdef STM32F10X_HD_VL
  ADC1_IRQn                   = 18,     /*!< ADC1 global Interrupt                                */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_TIM15_IRQn         = 24,     /*!< TIM1 Break and TIM15 Interrupts                      */
  TIM1_UP_TIM16_IRQn          = 25,     /*!< TIM1 Update and TIM16 Interrupts                     */
  TIM1_TRG_COM_TIM17_IRQn     = 26,     /*!< TIM1 Trigger and Commutation and TIM17 Interrupt     */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  CEC_IRQn                    = 42,     /*!< HDMI-CEC Interrupt                                   */
  TIM12_IRQn                  = 43,     /*!< TIM12 global Interrupt                               */
  TIM13_IRQn                  = 44,     /*!< TIM13 global Interrupt                               */
  TIM14_IRQn                  = 45,     /*!< TIM14 global Interrupt                               */
  TIM5_IRQn                   = 50,     /*!< TIM5 global Interrupt                                */
  SPI3_IRQn                   = 51,     /*!< SPI3 global Interrupt                                */
  UART4_IRQn                  = 52,     /*!< UART4 global Interrupt                               */
  UART5_IRQn                  = 53,     /*!< UART5 global Interrupt                               */  
  TIM6_DAC_IRQn               = 54,     /*!< TIM6 and DAC underrun Interrupt                      */
  TIM7_IRQn                   = 55,     /*!< TIM7 Interrupt                                       */  
  DMA2_Channel1_IRQn          = 56,     /*!< DMA2 Channel 1 global Interrupt                      */
  DMA2_Channel2_IRQn          = 57,     /*!< DMA2 Channel 2 global Interrupt                      */
  DMA2_Channel3_IRQn          = 58,     /*!< DMA2 Channel 3 global Interrupt                      */
  DMA2_Channel4_5_IRQn        = 59,     /*!< DMA2 Channel 4 and Channel 5 global Interrupt        */
  DMA2_Channel5_IRQn          = 60      /*!< DMA2 Channel 5 global Interrupt (DMA2 Channel 5 is 
                                             mapped at position 60 only if the MISC_REMAP bit in 
                                             the AFIO_MAPR2 register is set)                      */       
#endif /* STM32F10X_HD_VL */

#ifdef STM32F10X_XL
  ADC1_2_IRQn                 = 18,     /*!< ADC1 and ADC2 global Interrupt                       */
  USB_HP_CAN1_TX_IRQn         = 19,     /*!< USB Device High Priority or CAN1 TX Interrupts       */
  USB_LP_CAN1_RX0_IRQn        = 20,     /*!< USB Device Low Priority or CAN1 RX0 Interrupts       */
  CAN1_RX1_IRQn               = 21,     /*!< CAN1 RX1 Interrupt                                   */
  CAN1_SCE_IRQn               = 22,     /*!< CAN1 SCE Interrupt                                   */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_TIM9_IRQn          = 24,     /*!< TIM1 Break Interrupt and TIM9 global Interrupt       */
  TIM1_UP_TIM10_IRQn          = 25,     /*!< TIM1 Update Interrupt and TIM10 global Interrupt     */
  TIM1_TRG_COM_TIM11_IRQn     = 26,     /*!< TIM1 Trigger and Commutation Interrupt and TIM11 global interrupt */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  USBWakeUp_IRQn              = 42,     /*!< USB Device WakeUp from suspend through EXTI Line Interrupt */
  TIM8_BRK_TIM12_IRQn         = 43,     /*!< TIM8 Break Interrupt and TIM12 global Interrupt      */
  TIM8_UP_TIM13_IRQn          = 44,     /*!< TIM8 Update Interrupt and TIM13 global Interrupt     */
  TIM8_TRG_COM_TIM14_IRQn     = 45,     /*!< TIM8 Trigger and Commutation Interrupt and TIM14 global interrupt */
  TIM8_CC_IRQn                = 46,     /*!< TIM8 Capture Compare Interrupt                       */
  ADC3_IRQn                   = 47,     /*!< ADC3 global Interrupt                                */
  FSMC_IRQn                   = 48,     /*!< FSMC global Interrupt                                */
  SDIO_IRQn                   = 49,     /*!< SDIO global Interrupt                                */
  TIM5_IRQn                   = 50,     /*!< TIM5 global Interrupt                                */
  SPI3_IRQn                   = 51,     /*!< SPI3 global Interrupt                                */
  UART4_IRQn                  = 52,     /*!< UART4 global Interrupt                               */
  UART5_IRQn                  = 53,     /*!< UART5 global Interrupt                               */
  TIM6_IRQn                   = 54,     /*!< TIM6 global Interrupt                                */
  TIM7_IRQn                   = 55,     /*!< TIM7 global Interrupt                                */
  DMA2_Channel1_IRQn          = 56,     /*!< DMA2 Channel 1 global Interrupt                      */
  DMA2_Channel2_IRQn          = 57,     /*!< DMA2 Channel 2 global Interrupt                      */
  DMA2_Channel3_IRQn          = 58,     /*!< DMA2 Channel 3 global Interrupt                      */
  DMA2_Channel4_5_IRQn        = 59      /*!< DMA2 Channel 4 and Channel 5 global Interrupt        */
#endif /* STM32F10X_XL */  

#ifdef STM32F10X_CL
  ADC1_2_IRQn                 = 18,     /*!< ADC1 and ADC2 global Interrupt                       */
  CAN1_TX_IRQn                = 19,     /*!< USB Device High Priority or CAN1 TX Interrupts       */
  CAN1_RX0_IRQn               = 20,     /*!< USB Device Low Priority or CAN1 RX0 Interrupts       */
  CAN1_RX1_IRQn               = 21,     /*!< CAN1 RX1 Interrupt                                   */
  CAN1_SCE_IRQn               = 22,     /*!< CAN1 SCE Interrupt                                   */
  EXTI9_5_IRQn                = 23,     /*!< External Line[9:5] Interrupts                        */
  TIM1_BRK_IRQn               = 24,     /*!< TIM1 Break Interrupt                                 */
  TIM1_UP_IRQn                = 25,     /*!< TIM1 Update Interrupt                                */
  TIM1_TRG_COM_IRQn           = 26,     /*!< TIM1 Trigger and Commutation Interrupt               */
  TIM1_CC_IRQn                = 27,     /*!< TIM1 Capture Compare Interrupt                       */
  TIM2_IRQn                   = 28,     /*!< TIM2 global Interrupt                                */
  TIM3_IRQn                   = 29,     /*!< TIM3 global Interrupt                                */
  TIM4_IRQn                   = 30,     /*!< TIM4 global Interrupt                                */
  I2C1_EV_IRQn                = 31,     /*!< I2C1 Event Interrupt                                 */
  I2C1_ER_IRQn                = 32,     /*!< I2C1 Error Interrupt                                 */
  I2C2_EV_IRQn                = 33,     /*!< I2C2 Event Interrupt                                 */
  I2C2_ER_IRQn                = 34,     /*!< I2C2 Error Interrupt                                 */
  SPI1_IRQn                   = 35,     /*!< SPI1 global Interrupt                                */
  SPI2_IRQn                   = 36,     /*!< SPI2 global Interrupt                                */
  USART1_IRQn                 = 37,     /*!< USART1 global Interrupt                              */
  USART2_IRQn                 = 38,     /*!< USART2 global Interrupt                              */
  USART3_IRQn                 = 39,     /*!< USART3 global Interrupt                              */
  EXTI15_10_IRQn              = 40,     /*!< External Line[15:10] Interrupts                      */
  RTCAlarm_IRQn               = 41,     /*!< RTC Alarm through EXTI Line Interrupt                */
  OTG_FS_WKUP_IRQn            = 42,     /*!< USB OTG FS WakeUp from suspend through EXTI Line Interrupt */
  TIM5_IRQn                   = 50,     /*!< TIM5 global Interrupt                                */
  SPI3_IRQn                   = 51,     /*!< SPI3 global Interrupt                                */
  UART4_IRQn                  = 52,     /*!< UART4 global Interrupt                               */
  UART5_IRQn                  = 53,     /*!< UART5 global Interrupt                               */
  TIM6_IRQn                   = 54,     /*!< TIM6 global Interrupt                                */
  TIM7_IRQn                   = 55,     /*!< TIM7 global Interrupt                                */
  DMA2_Channel1_IRQn          = 56,     /*!< DMA2 Channel 1 global Interrupt                      */
  DMA2_Channel2_IRQn          = 57,     /*!< DMA2 Channel 2 global Interrupt                      */
  DMA2_Channel3_IRQn          = 58,     /*!< DMA2 Channel 3 global Interrupt                      */
  DMA2_Channel4_IRQn          = 59,     /*!< DMA2 Channel 4 global Interrupt                      */
  DMA2_Channel5_IRQn          = 60,     /*!< DMA2 Channel 5 global Interrupt                      */
  ETH_IRQn                    = 61,     /*!< Ethernet global Interrupt                            */
  ETH_WKUP_IRQn               = 62,     /*!< Ethernet Wakeup through EXTI line Interrupt          */
  CAN2_TX_IRQn                = 63,     /*!< CAN2 TX Interrupt                                    */
  CAN2_RX0_IRQn               = 64,     /*!< CAN2 RX0 Interrupt                                   */
  CAN2_RX1_IRQn               = 65,     /*!< CAN2 RX1 Interrupt                                   */
  CAN2_SCE_IRQn               = 66,     /*!< CAN2 SCE Interrupt                                   */
  OTG_FS_IRQn                 = 67      /*!< USB OTG FS global Interrupt                          */
#endif /* STM32F10X_CL */     
} IRQn_Type;
```

* 抢占优先级 `NVIC_IRQChannelPreemptionPriority` 和子优先级 `NVIC_IRQChannelSubPriority`

``` c
/**
@code  
 The table below gives the allowed values of the pre-emption priority and subpriority according
 to the Priority Grouping configuration performed by NVIC_PriorityGroupConfig function
  ============================================================================================================================
    NVIC_PriorityGroup   | NVIC_IRQChannelPreemptionPriority | NVIC_IRQChannelSubPriority  | Description
  ============================================================================================================================
   NVIC_PriorityGroup_0  |                0                  |            0-15             |   0 bits for pre-emption priority
                         |                                   |                             |   4 bits for subpriority
  ----------------------------------------------------------------------------------------------------------------------------
   NVIC_PriorityGroup_1  |                0-1                |            0-7              |   1 bits for pre-emption priority
                         |                                   |                             |   3 bits for subpriority
  ----------------------------------------------------------------------------------------------------------------------------    
   NVIC_PriorityGroup_2  |                0-3                |            0-3              |   2 bits for pre-emption priority
                         |                                   |                             |   2 bits for subpriority
  ----------------------------------------------------------------------------------------------------------------------------    
   NVIC_PriorityGroup_3  |                0-7                |            0-1              |   3 bits for pre-emption priority
                         |                                   |                             |   1 bits for subpriority
  ----------------------------------------------------------------------------------------------------------------------------    
   NVIC_PriorityGroup_4  |                0-15               |            0                |   4 bits for pre-emption priority
                         |                                   |                             |   0 bits for subpriority                       
  ============================================================================================================================
@endcode
*/
```

* 中断使能 `NVIC_IRQChannelCmd`

``` c
typedef enum {DISABLE = 0, ENABLE = !DISABLE} FunctionalState;
```

### 串口结构体分析

``` c
/** 

  + @brief  USART Init Structure definition  

  */ 
  
typedef struct
{
  uint32_t USART_BaudRate;            /*!< This member configures the USART communication baud rate.
                                           The baud rate is computed using the following formula:

                                            - IntegerDivider = ((PCLKx) / (16 * (USART_InitStruct->USART_BaudRate)))
                                            - FractionalDivider = ((IntegerDivider - ((u32) IntegerDivider)) * 16) + 0.5 */

  uint16_t USART_WordLength;          /*!< Specifies the number of data bits transmitted or received in a frame.
                                           This parameter can be a value of @ref USART_Word_Length */

  uint16_t USART_StopBits;            /*!< Specifies the number of stop bits transmitted.
                                           This parameter can be a value of @ref USART_Stop_Bits */

  uint16_t USART_Parity;              /*!< Specifies the parity mode.
                                           This parameter can be a value of @ref USART_Parity
                                           @note When parity is enabled, the computed parity is inserted
                                                 at the MSB position of the transmitted data (9th bit when
                                                 the word length is set to 9 data bits; 8th bit when the
                                                 word length is set to 8 data bits). */
 
  uint16_t USART_Mode;                /*!< Specifies wether the Receive or Transmit mode is enabled or disabled.
                                           This parameter can be a value of @ref USART_Mode */

  uint16_t USART_HardwareFlowControl; /*!< Specifies wether the hardware flow control mode is enabled
                                           or disabled.
                                           This parameter can be a value of @ref USART_Hardware_Flow_Control */
} USART_InitTypeDef;
```

* 波特率 `USART_BaudRate`

$ IntegerDivider = \frac{PCLKx}{16 \times USART\\_ InitStruct \to USART\\_ BaudRate}$

$ FractionalDivider = ((IntegerDivider - ((u32) IntegerDivider)) \times 16) + 0.5$

如：时钟频率为72M, 波特率为9600

则Divider=72000000/16/9600=468.75

IntegerDivider=468=0x1D4

FractionalDivider=0.75*16+0.5=12.5=0x0C

USART_BRR 0x1D4C

* 字长 `USART_WordLength`
宏定义如下：

``` c
/** @defgroup USART_Word_Length 

  + @{

  */ 
  
#define USART_WordLength_8b                  ((uint16_t)0x0000)//8位
#define USART_WordLength_9b                  ((uint16_t)0x1000)//9位
```

* 停止位 `USART_StopBits`
宏定义如下：

``` c
/** @defgroup USART_Stop_Bits 

  + @{

  */ 
  
#define USART_StopBits_1                     ((uint16_t)0x0000)//1个停止位
#define USART_StopBits_0_5                   ((uint16_t)0x1000)//0.5个停止位
#define USART_StopBits_2                     ((uint16_t)0x2000)//2个停止位
#define USART_StopBits_1_5                   ((uint16_t)0x3000)//1.5个停止位
```

* 校验位 `USART_Parity`
宏定义如下：

``` c
/** @defgroup USART_Parity 

  + @{

  */ 
  
#define USART_Parity_No                      ((uint16_t)0x0000) //无校验
#define USART_Parity_Even                    ((uint16_t)0x0400) //偶校验
#define USART_Parity_Odd                     ((uint16_t)0x0600) //奇校验
```

* 模式选择 `USART_Mode`
宏定义如下：

``` c
/** @defgroup USART_Mode 

  + @{

  */ 
  
#define USART_Mode_Rx                        ((uint16_t)0x0004) //接收模式
#define USART_Mode_Tx                        ((uint16_t)0x0008) //发送模式
```

* 硬件流控制选择 `USART_HardwareFlowControl`
宏定义如下：

``` c
/** @defgroup USART_Hardware_Flow_Control 

  + @{

  */ 
#define USART_HardwareFlowControl_None       ((uint16_t)0x0000) //不使能硬件流
#define USART_HardwareFlowControl_RTS        ((uint16_t)0x0100) //使能RTS
#define USART_HardwareFlowControl_CTS        ((uint16_t)0x0200) //使能CTS
#define USART_HardwareFlowControl_RTS_CTS    ((uint16_t)0x0300) //同时使能RTS 和CTS
```

### 使能中断和串口

``` c
// 使能串口接收中断
USART_ITConfig(DEBUG_USARTx, USART_IT_RXNE, ENABLE);
//使能串口总线空闲中断
USART_ITConfig ( DEBUG_USARTx, USART_IT_IDLE, ENABLE );
// 使能串口
USART_Cmd(DEBUG_USARTx, ENABLE);
```

# 中断函数

## 添加 `stm32f10x_it.c` 和 `stm32f10x_it.h` 文件

* 右键项目文件夹，选择 `Add New Item to Group1 ...` 选项
* 选择 `User Code Template`
* 选择 `Device->StdPeriph Drivers:Framework`
* 添加

![it1][it1]
![it2][it2]

## 添加串口头文件和变量到 `stm32f10x_it.c`

``` c
#include "my_usart.h"
uint8_t rx_data[50];
uint8_t rx_len=0;
```

## 添加串口中断函数 `stm32f10x_it.c`

``` c
/**

  + @brief  串口中断服务函数
  + @param  无
  + @retval 无

  */
void DEBUG_USART_IRQHandler(void)
{
    uint8_t ucCh;
    //数据接收完成
    if(USART_GetITStatus(DEBUG_USARTx,USART_IT_RXNE)!=RESET)
    {
        ucCh  = USART_ReceiveData( DEBUG_USARTx );
        rx_data[rx_len++]=ucCh;
        if(rx_len>=50)
        {
            rx_len=0;
        }
    }
    //数据帧接收完毕
    if ( USART_GetITStatus( DEBUG_USARTx, USART_IT_IDLE ) == SET )
    {
        Usart_SendByte(DEBUG_USARTx,rx_len);
        Usart_SendArray(DEBUG_USARTx,rx_data,rx_len);
        ucCh = USART_ReceiveData( DEBUG_USARTx );
        rx_len=0;
    }
}
```

# 调试

## 添加串口头文件 `main.c` 和调用初始化串口

* `main.c`

``` c
#include "stm32f10x.h"
#include "my_gpio.h"
#include "my_usart.h"

int main(void) {
    /* LED 端口初始化 */
    LED_GPIO_Config();
    Key_GPIO_Config();
    /* 串口初始化 */
	USART_Config();
    while(1)
    {

        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON  )
        {
            /*LED1反转*/
            LED1_TOGGLE;
        }

        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON  )
        {
            LED2_TOGGLE;
        }
    }
}

```

## 下载调试

* 编译之后下载到开发板
* 连接开发板串口
* 打开串口助手
* 向开发板发送数据，将会看到开发板返回数据长度和完整数据

![usart][usart]

* 可以看到我们发了10个数据，返回了11个数据
* 0A 代表总共接收到了10个数据

### 开启空闲中断的好处

* 关闭空闲中断

``` c
//使能串口总线空闲中断
//USART_ITConfig ( DEBUG_USARTx, USART_IT_IDLE, ENABLE );
```

* 中断函数更改

``` c
void DEBUG_USART_IRQHandler(void)
{
    uint8_t ucCh;
    if(USART_GetITStatus(DEBUG_USARTx,USART_IT_RXNE)!=RESET)
    {
      ucCh  = USART_ReceiveData( DEBUG_USARTx );
      if(rx_len>=50)
      {
        rx_len=0;
      }
      rx_data[rx_len++]=ucCh;
      Usart_SendByte(DEBUG_USARTx,rx_len);
      Usart_SendArray(DEBUG_USARTx,rx_data,rx_len);
    }
    //数据帧接收完毕
    if ( USART_GetITStatus( DEBUG_USARTx, USART_IT_IDLE ) == SET )
    {
      Usart_SendByte(DEBUG_USARTx,rx_len);
      Usart_SendArray(DEBUG_USARTx,rx_data,rx_len);
      rx_len=0;
      ucCh = USART_ReceiveData( DEBUG_USARTx );
    }
}

```

* 重新编译下载

会发现串口传回的数不全，数据会每接到一个字节回传一次。就算加上长度判断，也只能选择定长。对于不定长的帧写起来会很复杂

[usart]:data:image/jpeg; base64, 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

[it1]:data:image/jpeg; base64, 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DaoGvWqFgZHjjetgKX/gupGAq+akEE6UYNym9lCwb1Eho7y+RciO1NgoN1fIuJHai0NnCAtOk6avvvj92BiSCahh+VP6rx/8pKQO4vav6ftX9P2r+n7V/T9q/p+1f0/av6ftX9P2r+n7V/T9q/p+1f0/aoSb3H2oipEZNa/te1f2vav7XtX9r2r+17V/a9q/te1f2vav7XtX9r2r+17VCxCW/tX9/wBq/p+1f0/av6ftX9P2r+n7V/T9q/p+1f0/aj4AxI+1Y/8Aty8MNeY3PYhkyDJPL5rX6MUl4cPuroyrM6raPihihCNGQTN4vbhXntqM/WGFS5xDuNqM5bUIQbE7hUre1lanqHO1Y/8Aty8MNeY3PZSanPgwOgVCYllVPIVSfGViX9Fee2oz9cW6kEkm/wB6U9yomESqNFtxqXs3bbBmXG3esf8A25eGGvMbnsIkAJV0pcKsUYdKHujAiHrQJBCRNa89tRmv1GBvG3s//9oADAMBAAIAAwAAABDYIIMMMMMMMMNJlLl6w80Z5/8A/wBf/wAfnj/f337777762tJIIYL28444Lxz3MQgpzzzzzzz+gAADRjzzzzgzzywuZhLDDDDDDDOgAABhzzzzzM3zzxXNt1OTxrTDaYkYgAQDDDDlczzy1kE1dcvBBBHAABCBGBCABAGDz5L53XmDDHzLjvDXnTnr7/vP3nHHzzyyimhTqmPeAhIYHXtJSmwjLIzzzzwv2tToCFHaJIAIAMABbMAADzzzzyNL5Spl2Cqh/wAABqQglNoAA88888jtqUoAAAAAAAAAAAAApAAA88888sOpU44fTZoAAAAAAAAUAAA88888o9qVoDLEBFMPAAAAAABAAA888884xokrwEAQwgQAAAAAASAEA888884/+ZDTAkiAAAAAAAAAAATD88888j/6TgEMEEEAEAAAAAAAAAA88888r0ukrAzAwgDDAAAAAAAAAA88888MN4FEIMJKBMIAIIAAAAAAA8888888oFoAgxwBwtcsssMMAAAAw88888888888888/oAAAkc8888g/88888888888888/gAAA8A888888//8QAKBEAAQIDBgcBAQAAAAAAAAAAAQARITHwQVFhcZGhECAwgbHB0eFA/9oACAEDAQE/EGEEwrM+mREYJimKYpimKYpimKYoBGBDJQthL2XaISTkk2xA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

[it2]:data:image/jpeg; base64, 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jkuOS45LjkuOS45LjkuOX9ClsTIPzBZ6EgSILJ9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J9I6J0sgGhxPX9elsTIEmXBBoYknGilIwl4g6EVRG8O9IeGEli7IEPIWJSo7GBpzCN2wRxaCx/DDI4iQSsBdMiMDJwtXejAoc9VVj8FvrBFV8wLymIIWDFAtKL7GxBB5VvoEcljLxdv1ahjURGZPFAFiGTmTIAINV/CZ5fBnwCb3YhEFAArLTXoWGVGKrGLHFPgJlNabaKCqENAIagThoRgQi7wMaIrEDo1ghricu28oagB5EBun9+3h+vS2JkCUSU7VkO6EXBMIxBAAYOJDQcVyclfi8UKtsmRACpFMUeiaEGDB4HD8JPhzI0l5RiLwbAGNxBwBQAjCm3wDQUAABPeGNyLMATkggCDehlm0Aw5BAAYAE2LapjksHNqA1VIBimaCzMkyXrUKyjkIiTOFKiEACAaArei2FcszCDSRkCQrepKL4OT9+3h+vS2JkH5gAAYAG5cByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcByXAclwHJcuX69LYmQIeBjvAsT2O1wBnaIlhWm/wDEQCqYEWGUpsRszmG2biULYA4wQcKhFQDVwkObAmQiMEHaZ2TVrHtFWei5gCgMa04IQSXeAAtMhqA4ICBh8AbjEFDjohI4M9CpxdAgNwc0HFEDQexMgsTB4IZeS9yz34IsAVp2otvRhiKj2YWctQODJuRZK5CrVyxFBEqgTFmFlq/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/utX91q/uiHsJDJZdRdlqLsg/dQIWi5UtiZAgCZbDAs1aAtCnMTMEIFq/A4S8IDVaQdM3LUYMJVGo3IpdRFBaunA2r21QEIuUwNZAn3hBxb8AXFggieyKAcaE5Kyk0etrp97WQCXJ4tWqWOQEint8HAYCcwQUIjCjRaKdxQLCKliKJJivYzvBDG2qo4SUCVCINDeeKMWGka7DojFOs20cBnkmkQcGQZtGaFV5PCzDEjlli4Eg8m9uSl3J0BpvAVO20KHClUqiLFnsBqWRwIgVmc5HyBcICALYDXkuBbNJ/rV8797cXjGYUCjDok7wMYzWqxCpbEyD8wU0CUEWklbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYW7mFu5hbuYRyZ07gDey+q8L6rwjBxUhLhUtiZB/yylsTIP4LiovyUX5KL8lF+Si/JRfkovyUX5It5vOmzR2ReovyUX5KL8lF+Si/JRfkovyUX5JzhgHifx+vxUX5KL8lF+Si/JRfkovyUX5KL8kW83nTZoqlsTIP4K3htIywHJVXFwg3QAAOYlNSJYDKwBAENy1rUKbTmejZadoKxs2BMWJDKDeKJhGbggdrA3EolNCGZ97sxrho6IG54Nlt0TH9e3htOm/O3QJ7IICaQmMAkwMSgCAgVdcHqQVwNhaNpzPQqWxMg/greG17jaghzBOKzRuAk7uMG+xEYUfRc7kixVf/g5no2WnaQXc2g5cgOpwNU3iQqeCGYvAhDFOhAUUzgyZL2MtwQclcW26Jj+vbw20AOsash3T7CzoGQFHE3kDDh5dBYMI542B67TmehUtiZB/BO5qxL3Wn+Vp/laf5Wn+Vp/laf5Wn+Vp/lNtG8WvAb9hIWpI2a0/ytP8rT/K0/ytP8rT/K0/ytP8oAYYIsn8/ru5qxL3Wn+Vp/laf5Wn+Vp/laf5Wn+Vp/lNtG8WvAb0ewgAJuDOqwDPuIWFoYrC0MUWKaGKGMa/FA9dXihpfqtPd1adNirDpsUdHdUH9fzVpGhisDQxVj02K1N3Wj+6033Wq+6133Wq+6tOnxR1P1VsGpijcaGKsWmxR1X1Wu+60v3Vr02KsOnxTGdJitH91Y9firTo8Vr/ALoa76rT/da37qHTc0XdHzQ0/wBULbTYo6O6rX/dTa/mg3pua0P3Wu+60/3Wj+60/wB1rvutd91qvurRp8UdT9VFp+am1fNC012KL02awtDFWDSYrQ/daf7rTfda77rVfdWjT4o6n6q2DUxWFoYrTNmrVpcUdH9UbHR4rXfdDX/VWjXYoXGhirINTFDU/VWHT4rTfdBzV81pbugyRBfnpg9StAhZv/B/TH8UfiGw/wChsP8ArL7P/9oADAMBAAIAAwAAABBBzD2YJEbXeDD44wxRzNrLLLLLLLLLLI7pl6H1bI3Wc2bf3XHKnQSoRWyN6knUuA00000000000012yLnZxVsHY3WkggAAAAAAAAAQQAAQQwwgAQgAAAAAAAAAAAAAAAAAABelKrzzzzzzzzyjADCLCJAAJKAAAAAAAAAAAAAAAAAAADeEGzzzzzzzzzyhDr3r7zzzzxH/AG8888888888888888gfrHJ888888888oA6qACMMMMMkcuMMMMMMMMMMMMMMMMgeCGY888888888oE9cdu88888Qwwwwwwwwwwwwwwwwwwg4AGs888888888oU8El0W8888pR8FFwSAZW888888888oTDTk888888888oU8YldoYlLp5oKpeenxGMMMMMMMMMMI+JUP888888888oU88MMc2OMMEMcMMMMMMMMMMMMMMMMIKpVf888888888oU88888uM88U888888888888888888o2pRI888888888oU8888885V8U888888888888888888oWpDM888888888oU8888888a1U888888888888888888oWpPwU88888888oR8888888/QU888888888888888888oWpid+888888wwJk88888888ef888888888888888888oWpERsAAIAEMPiUU8888888882888888888888888888oWBmngAQgAAgAAUU888888888U888888888888888888o6BU88888888884U888888888U888888888888888888oCBgAAAAAAAAAAAQwwwwwwwwwQwwwwwwwwwwww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SOLRbBjnQmuhuC4NXbgUFiOKlqeoUtWuALAm4JVhAlGYPAUTOhx/WltMK0FmlGyBpcXygH1oadwWngIFXQu00cINloIdMgIrxl1C0DJZqjhkG6Nu7kAyCmi7+NEnK9UoJ297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vedve87e95297zt73nb3vO3vf6AOr7PDp22EwAKq4CIVtKxtPkYlBGcoo1faUWzV5vT7N1Pd9CHX9nh07bMhJrT1yWhC72lVlPLf8ARLFyoIBU0IsEsU4f1kWzIKOQwII0apk14QjeaOsOAlXu7dLLuLFRTKlypQXZZx0MWBtqY0wTw1L1RS0uoKREK5C2N0FRVmANTboHPkziR5uBHAC6J3HieVgNqFN1II3mrcExGIhABWBTkAu2F1OZOZOZOZOZOZOZOZOZOZB3nx9Cah6Vij5QafJvYmUylW6DuqFRWniKgLhTvrwU0ujqc6bBclwgowFsNQsKzzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzJzIqS999AHV9nh07bLRIsBOgCVloeekAl+gYAKqGrPDDphYLys/FUAANOg3ry/hMScgkWImETN+NJduFqU2Ningjx/iRbEWKDyQLhUxclAKGXXeOoajlYgFDlk3d4AAON6ws7LI6tu8QlFbKUNMv5WDLkR6iIClEG+RCwuXSACEyAKvY2lpZl5ZbgVlg0y3gyvjQdFqQ2nZ07OnZ07OnZ07OnZ07OnZ07Ohbuzg2I8a0yZWtBtVb1mHgADSgoMLwYmuyaJbV0aDRg2NoVjKCBjBTYabTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6dnTs6IgLiHm8NbwA9PweHTts1ximjzihhwJw8rpTE3BQoFa9LDxxcSybo0ua61nMu0PQMejn1GqyGfFaDZUAZSITp462JRTMRVNm4IAAnl2JQs0Al64XALv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
