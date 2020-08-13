---
title: STM32标准外设库ADC使用DMA多通道采集
categories: STM32 ADC DMA
tags: STM32 SPL ADC DMA
description: SPL库ADC采集
---

# 添加库函数
- 打开 `Manage Run-Time Environment`
- 选择`Device->StdPeriph Drivers->ADC`

# ADC初始化
## ADC宏定义

- `my_adc.h`

```c
#ifndef __MY_ADC_H
#define __MY_ADC_H
#include "stm32f10x.h"
// 注意：用作ADC采集的IO必须没有复用，否则采集电压会有影响
/********************ADC1输入通道（引脚）配置**************************/
#define    ADC_APBxClock_FUN             RCC_APB2PeriphClockCmd
#define    ADC_CLK                       RCC_APB2Periph_ADC1

#define    ADC_GPIO_APBxClock_FUN        RCC_APB2PeriphClockCmd
#define    ADC_GPIO_CLK                  RCC_APB2Periph_GPIOC
#define    ADC_PORT                      GPIOC
// 转换通道个数
#define    NOFCHANEL		3
#define    ADC_PIN                       GPIO_Pin_1
#define    ADC_CHANNEL1                  ADC_Channel_11 //管脚GPIOC_1
#define    ADC_CHANNEL2                  ADC_Channel_16 //芯片温度
#define    ADC_CHANNEL3                  ADC_Channel_17 //内部参考电压

// ADC1 对应 DMA1通道1
#define    ADC_x                         ADC1
#define    ADC_DMA_CHANNEL               DMA1_Channel1
#define    ADC_DMA_CLK                   RCC_AHBPeriph_DMA1
#define    ADC_DR_Address                (ADC1_BASE+0x4c)

/**************************函数声明********************************/
void ADCx_Init(void);
#endif

```

## ADC管脚设置和工作模式配置
- `my_adc.c`

```c
#include "my_adc.h"
__IO uint16_t ADC_ConvertedValue[NOFCHANEL] = {0};


/**
  * @brief  ADC GPIO 初始化
  * @param  无
  * @retval 无
  */
static void ADCx_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    // ADC GPIO 初始化
    ADC_GPIO_APBxClock_FUN ( ADC_GPIO_CLK, ENABLE );
    GPIO_InitStructure.GPIO_Pin = ADC_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
    GPIO_Init(ADC_PORT, &GPIO_InitStructure);
}
/**
  * @brief  配置ADC工作模式
  * @param  无
  * @retval 无
  */
static void ADCx_Mode_Config(void)
{
    DMA_InitTypeDef DMA_InitStructure;
    ADC_InitTypeDef ADC_InitStructure;
    // 打开DMA时钟
    RCC_AHBPeriphClockCmd(ADC_DMA_CLK, ENABLE);
    // 打开ADC时钟
    ADC_APBxClock_FUN( ADC_CLK, ENABLE);
    // ADC 模式配置
    // 只使用一个ADC，属于单模式
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    // 扫描模式
    ADC_InitStructure.ADC_ScanConvMode = ENABLE ;
    // 连续转换模式
    ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
    // 不用外部触发转换，软件开启即可
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    // 转换结果右对齐
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    // 转换通道个数
    ADC_InitStructure.ADC_NbrOfChannel = NOFCHANEL;
    // 初始化ADC
    ADC_Init(ADC_x, &ADC_InitStructure);
    // 配置ADC时钟RCC_PCLK2的8分频，即9MHz
    RCC_ADCCLKConfig(RCC_PCLK2_Div8);
    // 配置ADC 通道的转换顺序和采样时间
    ADC_RegularChannelConfig(ADC_x, ADC_CHANNEL1, 1, ADC_SampleTime_55Cycles5);
    ADC_RegularChannelConfig(ADC_x, ADC_CHANNEL2, 2, ADC_SampleTime_239Cycles5);
    ADC_RegularChannelConfig(ADC_x, ADC_CHANNEL3, 3, ADC_SampleTime_239Cycles5);
    //使能温度传感器和内部参考电压
    ADC_TempSensorVrefintCmd(ENABLE);
    // 使能ADC DMA 请求
    ADC_DMACmd(ADC_x, ENABLE);
    // 开启ADC ，并开始转换
    ADC_Cmd(ADC_x, ENABLE);
    // 初始化ADC 校准寄存器
    ADC_ResetCalibration(ADC_x);
    // 等待校准寄存器初始化完成
    while(ADC_GetResetCalibrationStatus(ADC_x));
    // ADC开始校准
    ADC_StartCalibration(ADC_x);
    // 等待校准完成
    while(ADC_GetCalibrationStatus(ADC_x));
    // 由于没有采用外部触发，所以使用软件触发ADC转换
    ADC_SoftwareStartConvCmd(ADC_x, ENABLE);

    // 复位DMA控制器
    DMA_DeInit(ADC_DMA_CHANNEL);
    // 配置 DMA 初始化结构体
    // 外设基址为：ADC 数据寄存器地址
    DMA_InitStructure.DMA_PeripheralBaseAddr = ADC_DR_Address;
    // 存储器地址
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)ADC_ConvertedValue;
    // 数据源来自外设
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
    // 缓冲区大小，应该等于数据目的地的大小
    DMA_InitStructure.DMA_BufferSize = NOFCHANEL;
    // 外设寄存器只有一个，地址不用递增
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
    // 存储器地址递增
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
    // 外设数据大小为半字，即两个字节
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
    // 内存数据大小也为半字，跟外设数据大小相同
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
    // 循环传输模式
    DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;
    // DMA 传输通道优先级为高，当使用一个DMA通道时，优先级设置不影响
    DMA_InitStructure.DMA_Priority = DMA_Priority_High;
    // 禁止存储器到存储器模式，因为是从外设到存储器
    DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;
    // 初始化DMA
    DMA_Init(ADC_DMA_CHANNEL, &DMA_InitStructure);
    // 使能 DMA 通道
    DMA_Cmd(ADC_DMA_CHANNEL , ENABLE);
}
/**
  * @brief  ADC初始化
  * @param  无
  * @retval 无
  */
void ADCx_Init(void)
{
    ADCx_GPIO_Config();
    ADCx_Mode_Config();
}

```

### ADC管脚初始化

```c
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//浮空输入
```

### ADC结构体分析

```c
/** 
  * @brief  ADC Init structure definition  
  */

typedef struct
{
  uint32_t ADC_Mode;                      /*!< Configures the ADC to operate in independent or
                                               dual mode. 
                                               This parameter can be a value of @ref ADC_mode */

  FunctionalState ADC_ScanConvMode;       /*!< Specifies whether the conversion is performed in
                                               Scan (multichannels) or Single (one channel) mode.
                                               This parameter can be set to ENABLE or DISABLE */

  FunctionalState ADC_ContinuousConvMode; /*!< Specifies whether the conversion is performed in
                                               Continuous or Single mode.
                                               This parameter can be set to ENABLE or DISABLE. */

  uint32_t ADC_ExternalTrigConv;          /*!< Defines the external trigger used to start the analog
                                               to digital conversion of regular channels. This parameter
                                               can be a value of @ref ADC_external_trigger_sources_for_regular_channels_conversion */

  uint32_t ADC_DataAlign;                 /*!< Specifies whether the ADC data alignment is left or right.
                                               This parameter can be a value of @ref ADC_data_align */

  uint8_t ADC_NbrOfChannel;               /*!< Specifies the number of ADC channels that will be converted
                                               using the sequencer for regular channel group.
                                               This parameter must range from 1 to 16. */
}ADC_InitTypeDef;
```

- 工作模式选择`ADC_Mode`

```c
/** @defgroup ADC_mode 
  * @{
  */

#define ADC_Mode_Independent                       ((uint32_t)0x00000000) //独立模式
#define ADC_Mode_RegInjecSimult                    ((uint32_t)0x00010000) //混合的同步规则+注入同步模式
#define ADC_Mode_RegSimult_AlterTrig               ((uint32_t)0x00020000) //混合的同步规则+交替触发模式
#define ADC_Mode_InjecSimult_FastInterl            ((uint32_t)0x00030000) //混合同步注入+快速交叉模式
#define ADC_Mode_InjecSimult_SlowInterl            ((uint32_t)0x00040000) //混合同步注入+慢速交叉模式
#define ADC_Mode_InjecSimult                       ((uint32_t)0x00050000) //注入同步模式
#define ADC_Mode_RegSimult                         ((uint32_t)0x00060000) //规则同步模式
#define ADC_Mode_FastInterl                        ((uint32_t)0x00070000) //快速交叉模式
#define ADC_Mode_SlowInterl                        ((uint32_t)0x00080000) //慢速交叉模式
#define ADC_Mode_AlterTrig                         ((uint32_t)0x00090000) //交替触发模式
```

- 扫描模式`ADC_ScanConvMode`

单通道AD 转换使用`DISABLE`，如果是多通道AD 转换使用`ENABLE`。

- 转换模式`ADC_ContinuousConvMode`

使用`ENABLE`配置为使能自动连续转换;使用`DISABLE`配置为单次转换,转换一次后停止需要手动控制才重新启动转换。

- 转换触发信号选择`ADC_ExternalTrigConv`

```c
/** @defgroup ADC_external_trigger_sources_for_regular_channels_conversion 
  * @{
  */

#define ADC_ExternalTrigConv_T1_CC1                ((uint32_t)0x00000000) /*!< For ADC1 and ADC2 */
#define ADC_ExternalTrigConv_T1_CC2                ((uint32_t)0x00020000) /*!< For ADC1 and ADC2 */
#define ADC_ExternalTrigConv_T2_CC2                ((uint32_t)0x00060000) /*!< For ADC1 and ADC2 */
#define ADC_ExternalTrigConv_T3_TRGO               ((uint32_t)0x00080000) /*!< For ADC1 and ADC2 */
#define ADC_ExternalTrigConv_T4_CC4                ((uint32_t)0x000A0000) /*!< For ADC1 and ADC2 */
#define ADC_ExternalTrigConv_Ext_IT11_TIM8_TRGO    ((uint32_t)0x000C0000) /*!< For ADC1 and ADC2 */

#define ADC_ExternalTrigConv_T1_CC3                ((uint32_t)0x00040000) /*!< For ADC1, ADC2 and ADC3 */
#define ADC_ExternalTrigConv_None                  ((uint32_t)0x000E0000) /*!< For ADC1, ADC2 and ADC3 */

#define ADC_ExternalTrigConv_T3_CC1                ((uint32_t)0x00000000) /*!< For ADC3 only */
#define ADC_ExternalTrigConv_T2_CC3                ((uint32_t)0x00020000) /*!< For ADC3 only */
#define ADC_ExternalTrigConv_T8_CC1                ((uint32_t)0x00060000) /*!< For ADC3 only */
#define ADC_ExternalTrigConv_T8_TRGO               ((uint32_t)0x00080000) /*!< For ADC3 only */
#define ADC_ExternalTrigConv_T5_CC1                ((uint32_t)0x000A0000) /*!< For ADC3 only */
#define ADC_ExternalTrigConv_T5_CC3                ((uint32_t)0x000C0000) /*!< For ADC3 only */
```

- 数据寄存器对齐格式`ADC_DataAlign`

```c
/** @defgroup ADC_data_align 
  * @{
  */

#define ADC_DataAlign_Right                        ((uint32_t)0x00000000) //右对齐
#define ADC_DataAlign_Left                         ((uint32_t)0x00000800) //左对齐
```

- 采集通道数`ADC_NbrOfChannel`

### 采样时钟分频和采样时间

- ADC最大时钟为14M
- 温度和内部电压值需要采样时间为17.1 μs
- 我们选择ADC时钟为9M,采样周期为ADC_SampleTime_239Cycles5

- ADC分频因子

```c
/** @defgroup ADC_clock_source 
  * @{
  */

#define RCC_PCLK2_Div2                   ((uint32_t)0x00000000) //2分频
#define RCC_PCLK2_Div4                   ((uint32_t)0x00004000) //4分频
#define RCC_PCLK2_Div6                   ((uint32_t)0x00008000) //6分频
#define RCC_PCLK2_Div8                   ((uint32_t)0x0000C000) //8分频
```

- 采样周期

```c
/** @defgroup ADC_sampling_time 
  * @{
  */

#define ADC_SampleTime_1Cycles5                    ((uint8_t)0x00)
#define ADC_SampleTime_7Cycles5                    ((uint8_t)0x01)
#define ADC_SampleTime_13Cycles5                   ((uint8_t)0x02)
#define ADC_SampleTime_28Cycles5                   ((uint8_t)0x03)
#define ADC_SampleTime_41Cycles5                   ((uint8_t)0x04)
#define ADC_SampleTime_55Cycles5                   ((uint8_t)0x05)
#define ADC_SampleTime_71Cycles5                   ((uint8_t)0x06)
#define ADC_SampleTime_239Cycles5                  ((uint8_t)0x07)
```

- ADC通道

```c
/** @defgroup ADC_channels 
  * @{
  */

#define ADC_Channel_0                               ((uint8_t)0x00)
#define ADC_Channel_1                               ((uint8_t)0x01)
#define ADC_Channel_2                               ((uint8_t)0x02)
#define ADC_Channel_3                               ((uint8_t)0x03)
#define ADC_Channel_4                               ((uint8_t)0x04)
#define ADC_Channel_5                               ((uint8_t)0x05)
#define ADC_Channel_6                               ((uint8_t)0x06)
#define ADC_Channel_7                               ((uint8_t)0x07)
#define ADC_Channel_8                               ((uint8_t)0x08)
#define ADC_Channel_9                               ((uint8_t)0x09)
#define ADC_Channel_10                              ((uint8_t)0x0A)
#define ADC_Channel_11                              ((uint8_t)0x0B)
#define ADC_Channel_12                              ((uint8_t)0x0C)
#define ADC_Channel_13                              ((uint8_t)0x0D)
#define ADC_Channel_14                              ((uint8_t)0x0E)
#define ADC_Channel_15                              ((uint8_t)0x0F)
#define ADC_Channel_16                              ((uint8_t)0x10)
#define ADC_Channel_17                              ((uint8_t)0x11)

#define ADC_Channel_TempSensor                      ((uint8_t)ADC_Channel_16)
#define ADC_Channel_Vrefint                         ((uint8_t)ADC_Channel_17)
```

|通道|ADC1-IO|ADC2-IO|ADC3-IO|
|:----:|:----:|:----:|:----:|
|ADC_Channel_0 |PA0|PA0|PA0|
|ADC_Channel_1 |PA1|PA1|PA1|
|ADC_Channel_2 |PA2|PA2|PA2|
|ADC_Channel_3 |PA3|PA3|PA3|
|ADC_Channel_4 |PA4|PA4|N|
|ADC_Channel_5 |PA5|PA5|N|
|ADC_Channel_6 |PA6|PA6|N|
|ADC_Channel_7 |PA7|PA7|N|
|ADC_Channel_8 |PB0|PB0|N|
|ADC_Channel_9 |PB1|PB1|VSS|
|ADC_Channel_10 |PC0|PC0|PC0|
|ADC_Channel_11 |PC1|PC1|PC1|
|ADC_Channel_12 |PC2|PC2|PC2|
|ADC_Channel_13 |PC3|PC3|PC3|
|ADC_Channel_14 |PC4|PC4|VSS|
|ADC_Channel_15 |PC5|PC5|VSS|
|ADC_Channel_16 |TempSensor|VSS|VSS|
|ADC_Channel_17 |Vrefint|VSS|VSS|

### 使能与开启ADC转换
- 使能温度传感器和内部参考电压
- 使能ADC DMA 请求
- 开启ADC ，并开始转换
- 校准ADC
- 开启触发

# 调试
## 添加头文件和调用ADC初始化函数
- `main.c`

```c
#include "stm32f10x.h"
#include "my_gpio.h"
#include "my_usart.h"
#include "my_data_queue.h"
#include "my_process_data.h"
#include "my_adc.h"
extern __IO uint16_t ADC_ConvertedValue[NOFCHANEL];
Channel ch[2];
float VREF,VSENSE;
float temp;
float adcValue;
/* 软件延时	*/
void Delay(__IO uint32_t nCount)
{
    for(; nCount != 0; nCount--);
}
int main(void)
{
    /* LED 端口初始化 */
    LED_GPIO_Config();
    Key_GPIO_Config();
    /* 初始化USART 配置模式为 9600 8-N-1 */
    USART_Config();
    /*初始化接收数据队列*/
    RX_Queue_Init();
    /*初始化ADC*/
    ADCx_Init();
	  printf( "\r\n Print current Temperature  \r\n");
    while (1)
    {
        Delay(0xffffee);      // 延时
        VREF=1.2f*4096/ADC_ConvertedValue[2];
        VSENSE=VREF*ADC_ConvertedValue[1]/4096;
        temp=(1.43f-VSENSE)/4.3+25;
        adcValue=VREF*ADC_ConvertedValue[0]/4096;
			  printf( "\r\n The IC current tem= %.2fC\r\n", temp);
			  printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
			  printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
        Process_Usart_Data(ch);
        if (Key_Scan(KEY1_GPIO_PORT, KEY1_GPIO_PIN) == KEY_ON)
        {
            /*LED1反转*/
            LED1_TOGGLE;
        }

        if (Key_Scan(KEY2_GPIO_PORT, KEY2_GPIO_PIN) == KEY_ON)
        {
            LED2_TOGGLE;
        }
        if(ch[0].state==1&&ch[0].start_up==0)
        {
            ch[0].stop_up=0;
            ch[0].start_up=1;
            LED1(ON);
        }
        if(ch[0].state==2&&ch[0].stop_up==0)
        {
            ch[0].stop_up=1;
            ch[0].start_up=0;
            LED1(OFF);
        }
        if(ch[1].state==1&&ch[1].start_up==0)
        {
            ch[1].stop_up=0;
            ch[1].start_up=1;
            LED2(ON);
        }
        if(ch[1].state==2&&ch[1].stop_up==0)
        {
            ch[1].stop_up=1;
            ch[1].start_up=0;
            LED2(OFF);
        }
    }
}

```
## 下载调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 会看到串口电源电压，温度，PC1电压值，用万用表测量比对