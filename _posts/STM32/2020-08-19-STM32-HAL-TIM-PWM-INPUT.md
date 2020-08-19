---
title: STM32硬件层HAL库TIMPWM输入测量
categories: STM32 CUBE HAL TIM PWM
tags: STM32 CUBE HAL TIM PWM DUTY
description: HAL库TIMPWM输入测量
---
# 配置TIM
- 打开CUBE工程

## 在**`Timers`**->**`TIM8`**配置
- 选择**Slave Mode**——`Reset Mode`
- 选择**Trigger Source**——`TI1FP1`
- 选择**Clock Source**——`Internal Clock`
- 选择**Channel1**——`Input Capture direct mode`
- 选择**Channel2**——`Input Capture indirect mode`
- **`Parameter Settings`**配置
- 配置预分频器72分频——`71`，自动重载数值——`999`，计数模式——`UP`，自动重载预装载——`Disable` **CKD**-`No Division`
- 配置**Channel 1**:极性选择——`Rising Edge`, IC选择——`Direct`,其余默认
- 配置**Channel 2**:极性选择——`Falling Edge`, IC选择——`Indirect`,其余默认
- **`NVIC Settings`**使能`TIM8 capture compare interrupt`中断

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# TIM函数
## 从模式结构体分析

```c
/**
  * @brief  TIM Slave configuration Structure definition
  */
typedef struct
{
  uint32_t  SlaveMode;         /*!< Slave mode selection
                                    This parameter can be a value of @ref TIM_Slave_Mode */
  uint32_t  InputTrigger;      /*!< Input Trigger source
                                    This parameter can be a value of @ref TIM_Trigger_Selection */
  uint32_t  TriggerPolarity;   /*!< Input Trigger polarity
                                    This parameter can be a value of @ref TIM_Trigger_Polarity */
  uint32_t  TriggerPrescaler;  /*!< Input trigger prescaler
                                    This parameter can be a value of @ref TIM_Trigger_Prescaler */
  uint32_t  TriggerFilter;     /*!< Input trigger filter
                                    This parameter can be a number between Min_Data = 0x0 and Max_Data = 0xF  */

} TIM_SlaveConfigTypeDef;
```

- 从模式选择`SlaveMode`

```c
/** @defgroup TIM_Slave_Mode TIM Slave mode
  * @{
  */
#define TIM_SLAVEMODE_DISABLE                0x00000000U                                        /*!< Slave mode disabled           *///从模式关闭
#define TIM_SLAVEMODE_RESET                  TIM_SMCR_SMS_2                                     /*!< Reset Mode                    *///复位模式
#define TIM_SLAVEMODE_GATED                  (TIM_SMCR_SMS_2 | TIM_SMCR_SMS_0)                  /*!< Gated Mode                    *///门控模式
#define TIM_SLAVEMODE_TRIGGER                (TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1)                  /*!< Trigger Mode                  *///触发模式
#define TIM_SLAVEMODE_EXTERNAL1              (TIM_SMCR_SMS_2 | TIM_SMCR_SMS_1 | TIM_SMCR_SMS_0) /*!< External Clock Mode 1         *///外部时钟模式
```
- 触发源选择`InputTrigger`

```c
/** @defgroup TIM_Trigger_Selection TIM Trigger Selection
  * @{
  */
#define TIM_TS_ITR0          0x00000000U                                                       /*!< Internal Trigger 0 (ITR0)              *///内部触发0(ITR0)
#define TIM_TS_ITR1          TIM_SMCR_TS_0                                                     /*!< Internal Trigger 1 (ITR1)              *///内部触发1(ITR1)
#define TIM_TS_ITR2          TIM_SMCR_TS_1                                                     /*!< Internal Trigger 2 (ITR2)              *///内部触发2(ITR2)
#define TIM_TS_ITR3          (TIM_SMCR_TS_0 | TIM_SMCR_TS_1)                                   /*!< Internal Trigger 3 (ITR3)              *///内部触发3(ITR3)
#define TIM_TS_TI1F_ED       TIM_SMCR_TS_2                                                     /*!< TI1 Edge Detector (TI1F_ED)            *///TI1的边沿检测器(TI1F_ED)
#define TIM_TS_TI1FP1        (TIM_SMCR_TS_0 | TIM_SMCR_TS_2)                                   /*!< Filtered Timer Input 1 (TI1FP1)        *///滤波后的定时器输入1(TI1FP1)
#define TIM_TS_TI2FP2        (TIM_SMCR_TS_1 | TIM_SMCR_TS_2)                                   /*!< Filtered Timer Input 2 (TI2FP2)        *///滤波后的定时器输入2(TI2FP2)
#define TIM_TS_ETRF          (TIM_SMCR_TS_0 | TIM_SMCR_TS_1 | TIM_SMCR_TS_2)                   /*!< Filtered External Trigger input (ETRF) *///外部触发输入(ETRF)
#define TIM_TS_NONE          0x0000FFFFU                                                       /*!< No trigger selected                    *///没有选择触发
```
- 触发极性`TriggerPolarity`

```c
/** @defgroup TIM_Trigger_Polarity TIM Trigger Polarity
  * @{
  */
#define TIM_TRIGGERPOLARITY_INVERTED           TIM_ETRPOLARITY_INVERTED               /*!< Polarity for ETRx trigger sources             *///ETR反相，低电平或下降沿有效
#define TIM_TRIGGERPOLARITY_NONINVERTED        TIM_ETRPOLARITY_NONINVERTED            /*!< Polarity for ETRx trigger sources             *///ETR不反相，高电平或上升沿有效
#define TIM_TRIGGERPOLARITY_RISING             TIM_INPUTCHANNELPOLARITY_RISING        /*!< Polarity for TIxFPx or TI1_ED trigger sources *///上升沿
#define TIM_TRIGGERPOLARITY_FALLING            TIM_INPUTCHANNELPOLARITY_FALLING       /*!< Polarity for TIxFPx or TI1_ED trigger sources *///下降沿
#define TIM_TRIGGERPOLARITY_BOTHEDGE           TIM_INPUTCHANNELPOLARITY_BOTHEDGE      /*!< Polarity for TIxFPx or TI1_ED trigger sources *///边沿
```

- 触发分频器`TriggerPrescaler`

```c
/** @defgroup TIM_Trigger_Prescaler TIM Trigger Prescaler
  * @{
  */
#define TIM_TRIGGERPRESCALER_DIV1             TIM_ETRPRESCALER_DIV1             /*!< No prescaler is used                                                       *///不分频
#define TIM_TRIGGERPRESCALER_DIV2             TIM_ETRPRESCALER_DIV2             /*!< Prescaler for External ETR Trigger: Capture performed once every 2 events. *///2分频
#define TIM_TRIGGERPRESCALER_DIV4             TIM_ETRPRESCALER_DIV4             /*!< Prescaler for External ETR Trigger: Capture performed once every 4 events. *///4分频
#define TIM_TRIGGERPRESCALER_DIV8             TIM_ETRPRESCALER_DIV8             /*!< Prescaler for External ETR Trigger: Capture performed once every 8 events. *///8分频
```
- 触发过滤器`TriggerFilter`

范围0-15

# 代码移植
## 开始捕获
- 在`main.c`文件添加变量

```c
__IO uint8_t IC_State;
__IO float DutyCycle;
__IO float Frequency;
```

- 在主函数初始化添加TIM启动

```c
TIM_CCxChannelCmd(TIM8,TIM_CHANNEL_2,TIM_CCx_ENABLE);
HAL_TIM_IC_Start_IT(&htim8,TIM_CHANNEL_1);
```

- 主循环添加捕获处理

```c
        if(IC_State)
        {
            IC_State=0;
            printf("DUTY:%0.2f%%   FREQ:%0.2fHz\n",DutyCycle,Frequency);
        }
```
- 在捕获完成回调函数添加

```c
    if(htim->Instance==TIM8)
    {
        /* 获取输入捕获值 */
        uint16_t IC1Value =  HAL_TIM_ReadCapturedValue(htim,TIM_CHANNEL_1);
        uint16_t IC2Value =  HAL_TIM_ReadCapturedValue(htim,TIM_CHANNEL_2);

        // 注意：捕获寄存器CCR1和CCR2的值在计算占空比和频率的时候必须加1
        if (IC1Value != 0)
        {
            /* 占空比计算 */
            DutyCycle = (float)((IC2Value+1) * 100) / (IC1Value+1);

            /* 频率计算 */
            Frequency = (72000000/(71+1))/(float)(IC1Value+1);
            IC_State=1;
        }
        else
        {
            DutyCycle = 0;
            Frequency = 0;
        }
    }
```
## 下载调试
- 编译之后下载到开发板
- 连接PC6到PA6
- 连接开发板串口
- 打开串口助手
- 可以看到频率和占空比和示波器看到的一样
- 和SPL库结果一致

