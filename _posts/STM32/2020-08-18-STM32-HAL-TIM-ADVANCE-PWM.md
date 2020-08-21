---
title: STM32硬件层HAL库高级定时器PWM输出
categories: STM32 CUBE HAL TIM PWM
tags: STM32 CUBE HAL TIM PWM
description: HAL库高级定时器PWM带互补输出
---
# 配置TIM
- 打开CUBE工程

## 在**`Timers`**->**`TIM1`**配置
- 选择**Clock Source**——`Internal Clock`
- 选择**Channel1**——`PWM Generation CH1 CH1N`
- 选择`Activate-Break-Input`
- **`Parameter Settings`**配置
- 配置预分频器9分频——`8`，自动重载数值——`7`，计数模式——`UP`，自动重载预装载——`Disable` **CKD**-`No Division`
- 配置**BRK State**——`Enable`,**BRK Polarity**——`High`
- 配置**Automatic Output State**——`Enable`,**Off State Selection for Run Mode(OSSR)**——`Enable`,**Off State Selection for Idle Mode(OSSR)**——`Enable`,**Lock Configuaration**——`Lock Level 1`,**Dead Time**——`11`
- 配置**Mode**——`PWM Mode 1`, **Pulse**——`4`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`High`,**CHN Polarity**——`High`,**CH Idle State**——`Set`,**CHN Idle State**——`Reset`

## 在**`Pinout View`**配置
- 设置**PB13**为`TIM1_CH1N`,**PB12**为`TIM1_BKIN`

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

#  TIME函数
## 时钟配置结构体分析

```c
/**
  * @brief  Clock Configuration Handle Structure definition
  */
typedef struct
{
  uint32_t ClockSource;     /*!< TIM clock sources
                                 This parameter can be a value of @ref TIM_Clock_Source */
  uint32_t ClockPolarity;   /*!< TIM clock polarity
                                 This parameter can be a value of @ref TIM_Clock_Polarity */
  uint32_t ClockPrescaler;  /*!< TIM clock prescaler
                                 This parameter can be a value of @ref TIM_Clock_Prescaler */
  uint32_t ClockFilter;     /*!< TIM clock filter
                                 This parameter can be a number between Min_Data = 0x0 and Max_Data = 0xF */
} TIM_ClockConfigTypeDef;
```
- 时钟源`ClockSource`

```c
/** @defgroup TIM_Clock_Source TIM Clock Source
  * @{
  */
#define TIM_CLOCKSOURCE_ETRMODE2    TIM_SMCR_ETPS_1      /*!< External clock source mode 2                          *///外部输入
#define TIM_CLOCKSOURCE_INTERNAL    TIM_SMCR_ETPS_0      /*!< Internal clock source                                 *///内部时钟
#define TIM_CLOCKSOURCE_ITR0        TIM_TS_ITR0          /*!< External clock source mode 1 (ITR0)                   *///IRT0
#define TIM_CLOCKSOURCE_ITR1        TIM_TS_ITR1          /*!< External clock source mode 1 (ITR1)                   *///IRT1
#define TIM_CLOCKSOURCE_ITR2        TIM_TS_ITR2          /*!< External clock source mode 1 (ITR2)                   *///IRT2
#define TIM_CLOCKSOURCE_ITR3        TIM_TS_ITR3          /*!< External clock source mode 1 (ITR3)                   *///IRT3
#define TIM_CLOCKSOURCE_TI1ED       TIM_TS_TI1F_ED       /*!< External clock source mode 1 (TTI1FP1 + edge detect.) *///TI1FP1_ED信号
#define TIM_CLOCKSOURCE_TI1         TIM_TS_TI1FP1        /*!< External clock source mode 1 (TTI1FP1)                *///TI1FP1信号
#define TIM_CLOCKSOURCE_TI2         TIM_TS_TI2FP2        /*!< External clock source mode 1 (TTI2FP2)                *///TI2FP2信号
#define TIM_CLOCKSOURCE_ETRMODE1    TIM_TS_ETRF          /*!< External clock source mode 1 (ETRF)                   *///外部输入
```

|从定时器|ITR0|ITR1|ITR2|ITR3|
|:----:|:----:|:----:|:----:|:----:|
|TIM1|TIM5|TIM2|TIM3|TIM4|
|TIM8|TIM1|TIM2|TIM4|TIM5|
|TIM2|TIM1|TIM8|TIM3|TIM4|
|TIM3|TIM1|TIM2|TIM5|TIM4|
|TIM4|TIM1|TIM2|TIM3|TIM8|
|TIM5|TIM2|TIM3|TIM4|TIM8|

- 时钟极性`ClockPolarity`

```c
/** @defgroup TIM_Clock_Polarity TIM Clock Polarity
  * @{
  */
#define TIM_CLOCKPOLARITY_INVERTED           TIM_ETRPOLARITY_INVERTED           /*!< Polarity for ETRx clock sources *///倒置
#define TIM_CLOCKPOLARITY_NONINVERTED        TIM_ETRPOLARITY_NONINVERTED        /*!< Polarity for ETRx clock sources *///不倒置
#define TIM_CLOCKPOLARITY_RISING             TIM_INPUTCHANNELPOLARITY_RISING    /*!< Polarity for TIx clock sources *///上升沿
#define TIM_CLOCKPOLARITY_FALLING            TIM_INPUTCHANNELPOLARITY_FALLING   /*!< Polarity for TIx clock sources *///下降沿
#define TIM_CLOCKPOLARITY_BOTHEDGE           TIM_INPUTCHANNELPOLARITY_BOTHEDGE  /*!< Polarity for TIx clock sources *///边沿
```
- 时钟预分频`ClockPrescaler`

```c
/** @defgroup TIM_Clock_Prescaler TIM Clock Prescaler
  * @{
  */
#define TIM_CLOCKPRESCALER_DIV1                 TIM_ETRPRESCALER_DIV1           /*!< No prescaler is used                                                     */
#define TIM_CLOCKPRESCALER_DIV2                 TIM_ETRPRESCALER_DIV2           /*!< Prescaler for External ETR Clock: Capture performed once every 2 events. */
#define TIM_CLOCKPRESCALER_DIV4                 TIM_ETRPRESCALER_DIV4           /*!< Prescaler for External ETR Clock: Capture performed once every 4 events. */
#define TIM_CLOCKPRESCALER_DIV8                 TIM_ETRPRESCALER_DIV8           /*!< Prescaler for External ETR Clock: Capture performed once every 8 events. */
```
- 时钟滤波器`ClockFilter`

范围0-15

## 输出比较结构体分析

```c
/**
  * @brief  TIM Output Compare Configuration Structure definition
  */
typedef struct
{
  uint32_t OCMode;        /*!< Specifies the TIM mode.
                               This parameter can be a value of @ref TIM_Output_Compare_and_PWM_modes */

  uint32_t Pulse;         /*!< Specifies the pulse value to be loaded into the Capture Compare Register.
                               This parameter can be a number between Min_Data = 0x0000 and Max_Data = 0xFFFF */

  uint32_t OCPolarity;    /*!< Specifies the output polarity.
                               This parameter can be a value of @ref TIM_Output_Compare_Polarity */

  uint32_t OCNPolarity;   /*!< Specifies the complementary output polarity.
                               This parameter can be a value of @ref TIM_Output_Compare_N_Polarity
                               @note This parameter is valid only for timer instances supporting break feature. */

  uint32_t OCFastMode;    /*!< Specifies the Fast mode state.
                               This parameter can be a value of @ref TIM_Output_Fast_State
                               @note This parameter is valid only in PWM1 and PWM2 mode. */


  uint32_t OCIdleState;   /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_Output_Compare_Idle_State
                               @note This parameter is valid only for timer instances supporting break feature. */

  uint32_t OCNIdleState;  /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_Output_Compare_N_Idle_State
                               @note This parameter is valid only for timer instances supporting break feature. */
} TIM_OC_InitTypeDef;
```

- 比较输出模式`OCMode` 

```c
/** @defgroup TIM_Output_Compare_and_PWM_modes TIM Output Compare and PWM Modes
  * @{
  */
#define TIM_OCMODE_TIMING                   0x00000000U                                              /*!< Frozen                                 *///冻结
#define TIM_OCMODE_ACTIVE                   TIM_CCMR1_OC1M_0                                         /*!< Set channel to active level on match   *///匹配时设置通道1为有效电平
#define TIM_OCMODE_INACTIVE                 TIM_CCMR1_OC1M_1                                         /*!< Set channel to inactive level on match *///匹配时设置通道1为无效电平
#define TIM_OCMODE_TOGGLE                   (TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0)                    /*!< Toggle                                 *///翻转
#define TIM_OCMODE_PWM1                     (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1)                    /*!< PWM mode 1                             *///PWM模式1
#define TIM_OCMODE_PWM2                     (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0) /*!< PWM mode 2                             *///PWM模式2
#define TIM_OCMODE_FORCED_ACTIVE            (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_0)                    /*!< Force active level                     *///强制有效电平
#define TIM_OCMODE_FORCED_INACTIVE          TIM_CCMR1_OC1M_2                                         /*!< Force inactive level                   *///强制无效电平
```
- 脉冲宽度`Pulse` 

设定比较寄存器CCR 的值，决定脉冲宽度
- 输出极性`OCPolarity` 

```c
/** @defgroup TIM_Output_Compare_Polarity TIM Output Compare Polarity
  * @{
  */
#define TIM_OCPOLARITY_HIGH                0x00000000U                          /*!< Capture/Compare output polarity  *///高
#define TIM_OCPOLARITY_LOW                 TIM_CCER_CC1P                        /*!< Capture/Compare output polarity  *///低
```
- 互补输出极性`OCNPolarity` 

```c
/** @defgroup TIM_Output_Compare_N_Polarity TIM Complementary Output Compare Polarity
  * @{
  */
#define TIM_OCNPOLARITY_HIGH               0x00000000U                          /*!< Capture/Compare complementary output polarity *///高
#define TIM_OCNPOLARITY_LOW                TIM_CCER_CC1NP                       /*!< Capture/Compare complementary output polarity *///低
```
- 快速模式状态`OCFastMode` 

```c
/** @defgroup TIM_Output_Fast_State TIM Output Fast State
  * @{
  */
#define TIM_OCFAST_DISABLE                 0x00000000U                          /*!< Output Compare fast disable */
#define TIM_OCFAST_ENABLE                  TIM_CCMR1_OC1FE                      /*!< Output Compare fast enable  */
```
- 空闲状态下比较输出状态`OCIdleState`

```c
/** @defgroup TIM_Output_Compare_Idle_State TIM Output Compare Idle State
  * @{
  */
#define TIM_OCIDLESTATE_SET                TIM_CR2_OIS1                         /*!< Output Idle state: OCx=1 when MOE=0 */
#define TIM_OCIDLESTATE_RESET              0x00000000U                          /*!< Output Idle state: OCx=0 when MOE=0 */
```
- 空闲状态下比较互补输出状态`OCNIdleState`

```c
/** @defgroup TIM_Output_Compare_N_Idle_State TIM Complementary Output Compare Idle State
  * @{
  */
#define TIM_OCNIDLESTATE_SET               TIM_CR2_OIS1N                        /*!< Complementary output Idle state: OCxN=1 when MOE=0 */
#define TIM_OCNIDLESTATE_RESET             0x00000000U                          /*!< Complementary output Idle state: OCxN=0 when MOE=0 */
```

## 断路和死区结构体分析

```c
/**
  * @brief  TIM Break input(s) and Dead time configuration Structure definition
  * @note   2 break inputs can be configured (BKIN and BKIN2) with configurable
  *        filter and polarity.
  */
typedef struct
{
  uint32_t OffStateRunMode;      /*!< TIM off state in run mode
                                      This parameter can be a value of @ref TIM_OSSR_Off_State_Selection_for_Run_mode_state */
  uint32_t OffStateIDLEMode;     /*!< TIM off state in IDLE mode
                                      This parameter can be a value of @ref TIM_OSSI_Off_State_Selection_for_Idle_mode_state */
  uint32_t LockLevel;            /*!< TIM Lock level
                                      This parameter can be a value of @ref TIM_Lock_level */
  uint32_t DeadTime;             /*!< TIM dead Time
                                      This parameter can be a number between Min_Data = 0x00 and Max_Data = 0xFF */
  uint32_t BreakState;           /*!< TIM Break State
                                      This parameter can be a value of @ref TIM_Break_Input_enable_disable */
  uint32_t BreakPolarity;        /*!< TIM Break input polarity
                                      This parameter can be a value of @ref TIM_Break_Polarity */
  uint32_t BreakFilter;          /*!< Specifies the break input filter.
                                      This parameter can be a number between Min_Data = 0x0 and Max_Data = 0xF */
  uint32_t AutomaticOutput;      /*!< TIM Automatic Output Enable state
                                      This parameter can be a value of @ref TIM_AOE_Bit_Set_Reset */
} TIM_BreakDeadTimeConfigTypeDef;
```
- 运行模式下的关闭状态选择`OffStateRunMode` 

```c
** @defgroup TIM_OSSR_Off_State_Selection_for_Run_mode_state TIM OSSR OffState Selection for Run mode state
  * @{
  */
#define TIM_OSSR_ENABLE                          TIM_BDTR_OSSR                  /*!< When inactive, OC/OCN outputs are enabled (still controlled by the timer)           */
#define TIM_OSSR_DISABLE                         0x00000000U                    /*!< When inactive, OC/OCN outputs are disabled (not controlled any longer by the timer) */

```
- 空闲模式下的关闭状态选择`OffStateIDLEMode` 

```c
/** @defgroup TIM_OSSI_Off_State_Selection_for_Idle_mode_state TIM OSSI OffState Selection for Idle mode state
  * @{
  */
#define TIM_OSSI_ENABLE                          TIM_BDTR_OSSI                  /*!< When inactive, OC/OCN outputs are enabled (still controlled by the timer)           */
#define TIM_OSSI_DISABLE                         0x00000000U                    /*!< When inactive, OC/OCN outputs are disabled (not controlled any longer by the timer) */
```
- 锁定配置`LockLevel`

```c
/** @defgroup TIM_Lock_level  TIM Lock level
  * @{
  */
#define TIM_LOCKLEVEL_OFF                  0x00000000U                          /*!< LOCK OFF     */
#define TIM_LOCKLEVEL_1                    TIM_BDTR_LOCK_0                      /*!< LOCK Level 1 */
#define TIM_LOCKLEVEL_2                    TIM_BDTR_LOCK_1                      /*!< LOCK Level 2 */
#define TIM_LOCKLEVEL_3                    TIM_BDTR_LOCK                        /*!< LOCK Level 3 */
```
- 死区时间`DeadTime`
- 断路输入使能控制`BreakState`

```c
/** @defgroup TIM_Break_Input_enable_disable TIM Break Input Enable
  * @{
  */
#define TIM_BREAK_ENABLE                   TIM_BDTR_BKE                         /*!< Break input BRK is enabled  */
#define TIM_BREAK_DISABLE                  0x00000000U                          /*!< Break input BRK is disabled */
```
- 断路输入极性`BreakPolarity`

```c
/** @defgroup TIM_Break_Polarity TIM Break Input Polarity
  * @{
  */
#define TIM_BREAKPOLARITY_LOW              0x00000000U                          /*!< Break input BRK is active low  */
#define TIM_BREAKPOLARITY_HIGH             TIM_BDTR_BKP                         /*!< Break input BRK is active high */
```
- 断路滤波器`BreakFilter`

范围0-15
- 自动输出使能`AutomaticOutput`

```c
/** @defgroup TIM_AOE_Bit_Set_Reset TIM Automatic Output Enable
  * @{
  */
#define TIM_AUTOMATICOUTPUT_DISABLE        0x00000000U                          /*!< MOE can be set only by software */
#define TIM_AUTOMATICOUTPUT_ENABLE         TIM_BDTR_AOE                         /*!< MOE can be set by software or automatically at the next update event (if none of the break inputs BRK and BRK2 is active) */
```
## 常用函数

- 启动PWM`HAL_StatusTypeDef HAL_TIM_PWM_Start(TIM_HandleTypeDef *htim, uint32_t Channel)`
- 停止PWM`HAL_StatusTypeDef HAL_TIM_PWM_Stop(TIM_HandleTypeDef *htim, uint32_t Channel)`

- 启动PWM带中断`HAL_StatusTypeDef HAL_TIM_PWM_Start_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`

会调用`HAL_TIM_OC_DelayElapsedCallback`和`HAL_TIM_PWM_PulseFinishedCallback`
- 停止PWM`HAL_StatusTypeDef HAL_TIM_PWM_Stop_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`

- DMA启动PWM`HAL_StatusTypeDef HAL_TIM_PWM_Start_DMA(TIM_HandleTypeDef *htim, uint32_t Channel, uint32_t *pData, uint16_t Length)`

DMA设置必须设置成存储器到外设，每更新一次DMA向CCR1传输一次,改变输出占空比，会调用`HAL_TIM_PWM_PulseFinishedCallback`
- 停止`HAL_StatusTypeDef HAL_TIM_PWM_Stop_DMA(TIM_HandleTypeDef *htim, uint32_t Channel)`

- 以下几个函数和PWM函数一模一样可通用,只是PWM模式在初始化的时候会判断输出快速使能
- 启动比较输出`HAL_StatusTypeDef HAL_TIM_OC_Start(TIM_HandleTypeDef *htim, uint32_t Channel)`
- 停止比较输出`HAL_StatusTypeDef HAL_TIM_OC_Stop(TIM_HandleTypeDef *htim, uint32_t Channel)`

- 启动比较输出带中断`HAL_StatusTypeDef HAL_TIM_OC_Start_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`
- 停止比较输出`HAL_StatusTypeDef HAL_TIM_OC_Stop_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`

- DMA启动比较输出`HAL_StatusTypeDef HAL_TIM_OC_Start_DMA(TIM_HandleTypeDef *htim, uint32_t Channel, uint32_t *pData, uint16_t Length)`
- 停止比较输出`HAL_StatusTypeDef HAL_TIM_OC_Stop_DMA(TIM_HandleTypeDef *htim, uint32_t Channel)`



# 代码移植
## 定时器应用
- 在主函数初始化添加TIM启动

```c
HAL_TIM_PWM_Start(&htim1,TIM_CHANNEL_1);
```

## 下载调试
- 编译之后下载到开发板
- 由于我们SPL库对PB12使用的推挽输出，先把输出置低了，HAL库使用的是浮空输入，需要把PB12接地才能输出
- 使用示波器查看输出波形,波形为1M,50%占空比方波
- 两路互补带死区
- PB12接高电平，PWM输出被禁止
- 和SPL库结果一致
