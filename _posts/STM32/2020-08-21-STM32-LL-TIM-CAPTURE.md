---
title: STM32底层LL库TIM输入捕获
categories: STM32 CUBE LL TIM CAPTURE
tags: STM32 CUBE LL TIM CAPTURE
description: LL库TIM输入捕获
---
# 配置TIM
- 和HAL库配置一样
- 在高级设置里为**TIM5**选择`LL`库
- 生成代码

#  TIM函数
## 输入捕获结构体分析

```c
/**
  * @brief  TIM Input Capture configuration structure definition.
  */

typedef struct
{

  uint32_t ICPolarity;    /*!< Specifies the active edge of the input signal.
                               This parameter can be a value of @ref TIM_LL_EC_IC_POLARITY.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_IC_SetPolarity().*/

  uint32_t ICActiveInput; /*!< Specifies the input.
                               This parameter can be a value of @ref TIM_LL_EC_ACTIVEINPUT.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_IC_SetActiveInput().*/

  uint32_t ICPrescaler;   /*!< Specifies the Input Capture Prescaler.
                               This parameter can be a value of @ref TIM_LL_EC_ICPSC.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_IC_SetPrescaler().*/

  uint32_t ICFilter;      /*!< Specifies the input capture filter.
                               This parameter can be a value of @ref TIM_LL_EC_IC_FILTER.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_IC_SetFilter().*/
} LL_TIM_IC_InitTypeDef;
```

- 输入捕获触发选择`ICPolarity` 

```c
/** @defgroup TIM_LL_EC_IC_POLARITY Input Configuration Polarity
  * @{
  */
#define LL_TIM_IC_POLARITY_RISING              0x00000000U                      /*!< The circuit is sensitive to TIxFP1 rising edge, TIxFP1 is not inverted */
#define LL_TIM_IC_POLARITY_FALLING             TIM_CCER_CC1P                    /*!< The circuit is sensitive to TIxFP1 falling edge, TIxFP1 is inverted */
```
- 输入捕获选择`ICActiveInput`

```c
/** @defgroup TIM_LL_EC_ACTIVEINPUT Active Input Selection
  * @{
  */
#define LL_TIM_ACTIVEINPUT_DIRECTTI            (TIM_CCMR1_CC1S_0 << 16U) /*!< ICx is mapped on TIx */
#define LL_TIM_ACTIVEINPUT_INDIRECTTI          (TIM_CCMR1_CC1S_1 << 16U) /*!< ICx is mapped on TIy */
#define LL_TIM_ACTIVEINPUT_TRC                 (TIM_CCMR1_CC1S << 16U)   /*!< ICx is mapped on TRC */
```
- 输入捕获预分频器`ICPrescaler`

```c
/** @defgroup TIM_LL_EC_ICPSC Input Configuration Prescaler
  * @{
  */
#define LL_TIM_ICPSC_DIV1                      0x00000000U                    /*!< No prescaler, capture is done each time an edge is detected on the capture input */
#define LL_TIM_ICPSC_DIV2                      (TIM_CCMR1_IC1PSC_0 << 16U)    /*!< Capture is done once every 2 events */
#define LL_TIM_ICPSC_DIV4                      (TIM_CCMR1_IC1PSC_1 << 16U)    /*!< Capture is done once every 4 events */
#define LL_TIM_ICPSC_DIV8                      (TIM_CCMR1_IC1PSC << 16U)      /*!< Capture is done once every 8 events */
```
- 输入捕获滤波器`ICFilter`

```c
/** @defgroup TIM_LL_EC_IC_FILTER Input Configuration Filter
  * @{
  */
#define LL_TIM_IC_FILTER_FDIV1                 0x00000000U                                                        /*!< No filter, sampling is done at fDTS */
#define LL_TIM_IC_FILTER_FDIV1_N2              (TIM_CCMR1_IC1F_0 << 16U)                                          /*!< fSAMPLING=fCK_INT, N=2 */
#define LL_TIM_IC_FILTER_FDIV1_N4              (TIM_CCMR1_IC1F_1 << 16U)                                          /*!< fSAMPLING=fCK_INT, N=4 */
#define LL_TIM_IC_FILTER_FDIV1_N8              ((TIM_CCMR1_IC1F_1 | TIM_CCMR1_IC1F_0) << 16U)                     /*!< fSAMPLING=fCK_INT, N=8 */
#define LL_TIM_IC_FILTER_FDIV2_N6              (TIM_CCMR1_IC1F_2 << 16U)                                          /*!< fSAMPLING=fDTS/2, N=6 */
#define LL_TIM_IC_FILTER_FDIV2_N8              ((TIM_CCMR1_IC1F_2 | TIM_CCMR1_IC1F_0) << 16U)                     /*!< fSAMPLING=fDTS/2, N=8 */
#define LL_TIM_IC_FILTER_FDIV4_N6              ((TIM_CCMR1_IC1F_2 | TIM_CCMR1_IC1F_1) << 16U)                     /*!< fSAMPLING=fDTS/4, N=6 */
#define LL_TIM_IC_FILTER_FDIV4_N8              ((TIM_CCMR1_IC1F_2 | TIM_CCMR1_IC1F_1 | TIM_CCMR1_IC1F_0) << 16U)  /*!< fSAMPLING=fDTS/4, N=8 */
#define LL_TIM_IC_FILTER_FDIV8_N6              (TIM_CCMR1_IC1F_3 << 16U)                                          /*!< fSAMPLING=fDTS/8, N=6 */
#define LL_TIM_IC_FILTER_FDIV8_N8              ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_0) << 16U)                     /*!< fSAMPLING=fDTS/8, N=8 */
#define LL_TIM_IC_FILTER_FDIV16_N5             ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_1) << 16U)                     /*!< fSAMPLING=fDTS/16, N=5 */
#define LL_TIM_IC_FILTER_FDIV16_N6             ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_1 | TIM_CCMR1_IC1F_0) << 16U)  /*!< fSAMPLING=fDTS/16, N=6 */
#define LL_TIM_IC_FILTER_FDIV16_N8             ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_2) << 16U)                     /*!< fSAMPLING=fDTS/16, N=8 */
#define LL_TIM_IC_FILTER_FDIV32_N5             ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_2 | TIM_CCMR1_IC1F_0) << 16U)  /*!< fSAMPLING=fDTS/32, N=5 */
#define LL_TIM_IC_FILTER_FDIV32_N6             ((TIM_CCMR1_IC1F_3 | TIM_CCMR1_IC1F_2 | TIM_CCMR1_IC1F_1) << 16U)  /*!< fSAMPLING=fDTS/32, N=6 */
#define LL_TIM_IC_FILTER_FDIV32_N8             (TIM_CCMR1_IC1F << 16U)                                            /*!< fSAMPLING=fDTS/32, N=8 */
```

# 代码移植
## 定义变量
- 在`tim.h`文件添加结构体定义

```c
/* USER CODE BEGIN Private defines */
// 定时器输入捕获用户自定义变量结构体声明
typedef struct
{
    uint8_t   Capture_FinishFlag;   // 捕获结束标志位
    uint8_t   Capture_StartFlag;    // 捕获开始标志位
    uint16_t  Capture_CcrValue;     // 捕获寄存器的值
    uint16_t  Capture_Period;       // 自动重装载寄存器更新标志
} TIM_ICUserValueTypeDef;
extern TIM_ICUserValueTypeDef TIM_ICUserValueStructure;
/* USER CODE END Private defines */
```
## 修改定时器中断
- 在`stm32f10x_it.c`文件添加头文件包含

```c
#include "tim.h"
```
- 修改中断

```c
/**
  * @brief This function handles TIM5 global interrupt.
  */
void TIM5_IRQHandler(void)
{
    /* USER CODE BEGIN TIM5_IRQn 0 */
    // 当要被捕获的信号的周期大于定时器的最长定时时，定时器就会溢出，产生更新中断
    // 这个时候我们需要把这个最长的定时周期加到捕获信号的时间里面去
    if(LL_TIM_IsActiveFlag_UPDATE(TIM5))
    {
        LL_TIM_ClearFlag_UPDATE(TIM5);
        TIM_ICUserValueStructure.Capture_Period ++;
    }
    /* USER CODE END TIM5_IRQn 0 */
    /* USER CODE BEGIN TIM5_IRQn 1 */
    if(LL_TIM_IsActiveFlag_CC1(TIM5))
    {
        LL_TIM_ClearFlag_CC1(TIM5);
        // 第一次捕获
        if ( TIM_ICUserValueStructure.Capture_StartFlag == 0 )
        {
            // 计数器清0
            LL_TIM_SetCounter( TIM5, 0 );
            // 自动重装载寄存器更新标志清0
            TIM_ICUserValueStructure.Capture_Period = 0;
            // 存捕获比较寄存器的值的变量的值清0
            TIM_ICUserValueStructure.Capture_CcrValue = 0;

            // 当第一次捕获到上升沿之后，就把捕获边沿配置为下降沿
            LL_TIM_IC_SetPolarity(TIM5,LL_TIM_CHANNEL_CH1,LL_TIM_IC_POLARITY_FALLING);
            // 开始捕获标准置1
            TIM_ICUserValueStructure.Capture_StartFlag = 1;
        }
        // 下降沿捕获中断
        else // 第二次捕获
        {
            // 获取捕获比较寄存器的值，这个值就是捕获到的高电平的时间的值
            TIM_ICUserValueStructure.Capture_CcrValue =LL_TIM_IC_GetCaptureCH1(TIM5);

            // 当第二次捕获到下降沿之后，就把捕获边沿配置为上升沿，好开启新的一轮捕获
            LL_TIM_IC_SetPolarity(TIM5,LL_TIM_CHANNEL_CH1,LL_TIM_IC_POLARITY_RISING);
            // 开始捕获标志清0
            TIM_ICUserValueStructure.Capture_StartFlag = 0;
            // 捕获完成标志置1
            TIM_ICUserValueStructure.Capture_FinishFlag = 1;
        }
    }
    /* USER CODE END TIM5_IRQn 1 */
}
```
## 开始捕获
- 在`main.c`文件添加变量

```c
// 定时器输入捕获用户自定义变量结构体定义
TIM_ICUserValueTypeDef TIM_ICUserValueStructure = {0,0,0,0};
```

- 在主函数初始化添加TIM启动

```c
    LL_TIM_EnableCounter(TIM5);//使能定时器
    LL_TIM_CC_EnableChannel(TIM5,LL_TIM_CHANNEL_CH1);//使能通道
    LL_TIM_EnableIT_UPDATE(TIM5);//使能更新中断
    LL_TIM_EnableIT_CC1(TIM5);//使能捕捉中断
    uint32_t capture_time;
    uint32_t TIM_PscCLK = 72000000 / (71+1);
```
- 主循环添加捕获处理

```c
        if(TIM_ICUserValueStructure.Capture_FinishFlag == 1)
        {
            // 计算高电平时间的计数器的值
            capture_time = TIM_ICUserValueStructure.Capture_Period * (65535+1) +
                           (TIM_ICUserValueStructure.Capture_CcrValue+1);

            // 打印高电平脉宽时间
            printf ( "\r\n duty time:%d.%d s\r\n",capture_time/TIM_PscCLK,capture_time%TIM_PscCLK );

            TIM_ICUserValueStructure.Capture_FinishFlag = 0;
        }
```
## 下载调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 按下按键1，可以看到串口发来的按键时间
- 和HAL库结果一致