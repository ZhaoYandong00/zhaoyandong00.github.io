---
title: STM32标准外设库TIM触发ADCSPL篇14
categories: STM32 ADC DMA TIM
tags: STM32 SPL ADC DMA TIM
description: SPL库定时采集ADC
---
# 修改ADC初始化
## 添加TIM相关宏
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
#define    ADC_CHANNEL1                  ADC_Channel_11 //管脚
#define    ADC_CHANNEL2                  ADC_Channel_16 //芯片温度
#define    ADC_CHANNEL3                  ADC_Channel_17 //内部参考电压

// ADC1 对应 DMA1通道1
#define    ADC_x                         ADC1
#define    ADC_DMA_CHANNEL               DMA1_Channel1
#define    ADC_DMA_CLK                   RCC_AHBPeriph_DMA1
#define    ADC_DR_Address                (ADC1_BASE+0x4c)

//ADC 定时器4触发 10ms触发一次
#define    ADC_TIM                       TIM4
#define    ADC_TIM_APBxClock_FUN         RCC_APB1PeriphClockCmd
#define    ADC_TIM_CLK                   RCC_APB1Periph_TIM4
#define    ADC_TIM_OCInit                TIM_OC4Init
#define    ADC_TIM_Period                (1000-1)
#define    ADC_TIM_Prescaler             (720-1)
#define    ADC_TIM_Conv                  ADC_ExternalTrigConv_T4_CC4
#define    ADC_DMA_IRQ                   DMA1_Channel1_IRQn
#define    ADC_DMA_IRQHandler            DMA1_Channel1_IRQHandler


#define    ADC_BUFF_SIZE                 100

/**************************函数声明********************************/
void ADCx_Init(void);
#endif

```

## 修改初始化程序
- `my_adc.c`

```c
#include "my_adc.h"
__IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][NOFCHANEL] = {0};

/**
  * @brief 中断配置
  * @param 无
  * @retval 无
  */
static void ADC_NVIC_Config(void)
{
    NVIC_InitTypeDef NVIC_InitStructure;
    // 设置中断组为0
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_0);
    // 设置中断来源
    NVIC_InitStructure.NVIC_IRQChannel = ADC_DMA_IRQ;
    // 设置主优先级为 0
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
    // 设置抢占优先级为0
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);
}

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
  * @brief 时间初始化
  * @param 无
  * @retval 无
  */

static void ADC_Time_Config(void)
{
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_OCInitTypeDef TIM_OCInitStructure;

    ADC_TIM_APBxClock_FUN(ADC_TIM_CLK, ENABLE); 		//时钟使能

    //定时器TIM4初始化
    /*--------------------时基结构体初始化-------------------------*/
    TIM_TimeBaseStructure.TIM_Period = ADC_TIM_Period; 		       //设置在下一个更新事件装入活动的自动重装载寄存器周期的值
    TIM_TimeBaseStructure.TIM_Prescaler =ADC_TIM_Prescaler; 	   //设置用来作为TIMx时钟频率除数的预分频值
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1; 		 //设置时钟分割:TDTS = Tck_tim
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;  //TIM向上计数模式
    TIM_TimeBaseInit(ADC_TIM, &TIM_TimeBaseStructure);			     //根据指定的参数初始化TIMx的时间基数单位
    /*--------------------输出比较结构体初始化-------------------*/
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;		           //选择定时器模式:TIM脉冲宽度调制模式1
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;		//比较输出使能
    TIM_OCInitStructure.TIM_Pulse = (ADC_TIM_Period+1)/2;
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_Low;		   //输出极性:TIM输出比较极性低
    ADC_TIM_OCInit(ADC_TIM, & TIM_OCInitStructure);		               //初始化外设TIM4 C4
    //使能TIMx
    TIM_Cmd(ADC_TIM,ENABLE);
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
    // 修改为不连续转换
    // ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
    // 不用外部触发转换，软件开启即可
    // 修改为定时触发
    //ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    ADC_InitStructure.ADC_ExternalTrigConv =ADC_TIM_Conv ;
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
    // 修改为外部触发
    //ADC_SoftwareStartConvCmd(ADC_x, ENABLE);
    ADC_ExternalTrigConvCmd(ADC_x,ENABLE);

    // 复位DMA控制器
    DMA_DeInit(ADC_DMA_CHANNEL);
    // 配置 DMA 初始化结构体
    // 外设基址为：ADC 数据寄存器地址
    DMA_InitStructure.DMA_PeripheralBaseAddr = ADC_DR_Address;
    // 存储器地址
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)&ADC_ConvertedValue;
    // 数据源来自外设
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
    // 缓冲区大小，应该等于数据目的地的大小
    //DMA_InitStructure.DMA_BufferSize = NOFCHANEL;
    // 修改缓冲区大小
    DMA_InitStructure.DMA_BufferSize = ADC_BUFF_SIZE*NOFCHANEL;
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
    //使能传输完成中断
    DMA_ITConfig(ADC_DMA_CHANNEL,DMA_IT_TC, ENABLE);
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
    ADC_NVIC_Config();
    ADCx_GPIO_Config();
    ADC_Time_Config();
    ADCx_Mode_Config();
}

```

## 定时器设置
- ADC转换时间：T<sub>conv</sub>=采样时间+12.5周期
- 本例内部参考电压转换时间：(239.5+12.5)/9M=28us
- 内部温度28us
- 外部ADC电压时间(55.5+12.5)/9M=7.56us
- 总转换时间63.56us
- 故定时器周期不能小于64us
- 我们根据需求设置10ms，低电耗时可能需要更长的延时
- DMA传输中断时间：1s

$ TC\\_TIME=\frac{缓冲区大小(100*3)\times每个缓冲区大小(16位)\times采集间隔(10ms)}{传输数据大小(HalfWord)\times转换通道个数(3)}$

TC_TIME=100\*3\*16\*10/16/3=1s

## ADC设置
- 把连续转换方式修改为单次

```c
// 连续转换模式
// 修改为不连续转换
// ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
```
- 更改触发条件为定时器触发

```c
// 不用外部触发转换，软件开启即可
// 修改为定时触发
//ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
ADC_InitStructure.ADC_ExternalTrigConv =ADC_TIM_Conv ;
```
- 打开外部处发使能

```c
// 由于没有采用外部触发，所以使用软件触发ADC转换 
//ADC_SoftwareStartConvCmd(ADC_x, ENABLE);
// 修改为外部触发
ADC_ExternalTrigConvCmd(ADC_x,ENABLE);
```

## DMA设置
- 为了更好取得ADC精度，我们取100次，求平均值
- 修改ADC转换数组

```c
//修改为二维数组，取数方便
//__IO uint16_t ADC_ConvertedValue[NOFCHANEL] = {0};
__IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][NOFCHANEL] = {0};
```
- 修改DMA缓冲区大小

```c
// 缓冲区大小，应该等于数据目的地的大小
//DMA_InitStructure.DMA_BufferSize = NOFCHANEL;
// 修改缓冲区大小
DMA_InitStructure.DMA_BufferSize = ADC_BUFF_SIZE*NOFCHANEL;
```
- 打开传输完成中断

```c
//使能传输完成中断
DMA_ITConfig(ADC_DMA_CHANNEL,DMA_IT_TC, ENABLE);
```
# 添加DMA中断
## 添加ADC转换数组外部定义和ADC头文件到`stm32f10x_it.c`

```c
#include "my_adc.h"
extern __IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][NOFCHANEL];
```
## 添加DMA中断函数

```c
float VREF,VSENSE;
float temp;
float adcValue;
/**
  * @brief  DMA传输中断
  * @param  无
  * @retval 无
  */
void  ADC_DMA_IRQHandler(void)
{
    uint8_t i=0;
	  uint32_t vref_adc=0,vsense_adc=0,adc_adc=0;
    if(DMA_GetITStatus(DMA1_IT_TC1)!=RESET)
    {
        DMA_ClearITPendingBit(DMA1_IT_TC1);
        for(i=0; i<ADC_BUFF_SIZE; i++)
        {
					vref_adc+=ADC_ConvertedValue[i][2];
					vsense_adc+=ADC_ConvertedValue[i][1];
					adc_adc+=ADC_ConvertedValue[i][0];
        }
        VREF=1.2f*4095*ADC_BUFF_SIZE/vref_adc;
        VSENSE=VREF*vsense_adc/ADC_BUFF_SIZE/4095;
        temp=(1.43f-VSENSE)*1000/4.3+25;
        adcValue=VREF*adc_adc/ADC_BUFF_SIZE/4095;
        printf( "\r\n The IC current tem= %.2fC\r\n", temp);
        printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
        printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
    }
}
```
# 调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 可以看到串口每1秒接收一次计算后的数值