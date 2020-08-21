---
title: STM32底层LL库高级定时器PWM输出
categories: STM32 CUBE LL TIM PWM
tags: STM32 CUBE LL TIM PWM
description: LL库高级定时器PWM带互补输出
---
# 配置TIM
- 和HAL库配置一样
- 在高级设置里为**TIM1**选择`LL`库
- 生成代码

# TIM函数
## TIM比较输出结构体分析

```c
/**
  * @brief  TIM Output Compare configuration structure definition.
  */
typedef struct
{
  uint32_t OCMode;        /*!< Specifies the output mode.
                               This parameter can be a value of @ref TIM_LL_EC_OCMODE.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetMode().*/

  uint32_t OCState;       /*!< Specifies the TIM Output Compare state.
                               This parameter can be a value of @ref TIM_LL_EC_OCSTATE.

                               This feature can be modified afterwards using unitary functions @ref LL_TIM_CC_EnableChannel() or @ref LL_TIM_CC_DisableChannel().*/

  uint32_t OCNState;      /*!< Specifies the TIM complementary Output Compare state.
                               This parameter can be a value of @ref TIM_LL_EC_OCSTATE.

                               This feature can be modified afterwards using unitary functions @ref LL_TIM_CC_EnableChannel() or @ref LL_TIM_CC_DisableChannel().*/

  uint32_t CompareValue;  /*!< Specifies the Compare value to be loaded into the Capture Compare Register.
                               This parameter can be a number between Min_Data=0x0000 and Max_Data=0xFFFF.

                               This feature can be modified afterwards using unitary function LL_TIM_OC_SetCompareCHx (x=1..6).*/

  uint32_t OCPolarity;    /*!< Specifies the output polarity.
                               This parameter can be a value of @ref TIM_LL_EC_OCPOLARITY.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetPolarity().*/

  uint32_t OCNPolarity;   /*!< Specifies the complementary output polarity.
                               This parameter can be a value of @ref TIM_LL_EC_OCPOLARITY.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetPolarity().*/


  uint32_t OCIdleState;   /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_LL_EC_OCIDLESTATE.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetIdleState().*/

  uint32_t OCNIdleState;  /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_LL_EC_OCIDLESTATE.

                               This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetIdleState().*/
} LL_TIM_OC_InitTypeDef;
```

- 比较输出模式`OCMode`

```c
/** @defgroup TIM_LL_EC_OCMODE Output Configuration Mode
  * @{
  */
#define LL_TIM_OCMODE_FROZEN                   0x00000000U                                              /*!<The comparison between the output compare register TIMx_CCRy and the counter TIMx_CNT has no effect on the output channel level */
#define LL_TIM_OCMODE_ACTIVE                   TIM_CCMR1_OC1M_0                                         /*!<OCyREF is forced high on compare match*/
#define LL_TIM_OCMODE_INACTIVE                 TIM_CCMR1_OC1M_1                                         /*!<OCyREF is forced low on compare match*/
#define LL_TIM_OCMODE_TOGGLE                   (TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0)                    /*!<OCyREF toggles on compare match*/
#define LL_TIM_OCMODE_FORCED_INACTIVE          TIM_CCMR1_OC1M_2                                         /*!<OCyREF is forced low*/
#define LL_TIM_OCMODE_FORCED_ACTIVE            (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_0)                    /*!<OCyREF is forced high*/
#define LL_TIM_OCMODE_PWM1                     (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1)                    /*!<In upcounting, channel y is active as long as TIMx_CNT<TIMx_CCRy else inactive.  In downcounting, channel y is inactive as long as TIMx_CNT>TIMx_CCRy else active.*/
#define LL_TIM_OCMODE_PWM2                     (TIM_CCMR1_OC1M_2 | TIM_CCMR1_OC1M_1 | TIM_CCMR1_OC1M_0) /*!<In upcounting, channel y is inactive as long as TIMx_CNT<TIMx_CCRy else active.  In downcounting, channel y is active as long as TIMx_CNT>TIMx_CCRy else inactive*/
```
- 输出使能`OCState`
- 比较输出使能`OCNState`

```c
/** @defgroup TIM_LL_EC_OCSTATE Output Configuration State
  * @{
  */
#define LL_TIM_OCSTATE_DISABLE                 0x00000000U             /*!< OCx is not active */
#define LL_TIM_OCSTATE_ENABLE                  TIM_CCER_CC1E           /*!< OCx signal is output on the corresponding output pin */
```
- 脉冲宽度`CompareValue`

- 输出极性`OCPolarity`
- 互补输出极性`OCNPolarity`
```c
/** @defgroup TIM_LL_EC_OCPOLARITY Output Configuration Polarity
  * @{
  */
#define LL_TIM_OCPOLARITY_HIGH                 0x00000000U                 /*!< OCxactive high*/
#define LL_TIM_OCPOLARITY_LOW                  TIM_CCER_CC1P               /*!< OCxactive low*/
```
- 空闲状态下比较输出状态`OCIdleState`
- 空闲状态下比较互补输出状态`OCNIdleState`

```c
/** @defgroup TIM_LL_EC_OCIDLESTATE Output Configuration Idle State
  * @{
  */
#define LL_TIM_OCIDLESTATE_LOW                 0x00000000U             /*!<OCx=0 (after a dead-time if OC is implemented) when MOE=0*/
#define LL_TIM_OCIDLESTATE_HIGH                TIM_CR2_OIS1            /*!<OCx=1 (after a dead-time if OC is implemented) when MOE=0*/
```

## 断路和死区结构体分析

```c
/**
  * @brief  BDTR (Break and Dead Time) structure definition
  */
typedef struct
{
  uint32_t OSSRState;            /*!< Specifies the Off-State selection used in Run mode.
                                      This parameter can be a value of @ref TIM_LL_EC_OSSR

                                      This feature can be modified afterwards using unitary function @ref LL_TIM_SetOffStates()

                                      @note This bit-field cannot be modified as long as LOCK level 2 has been programmed. */

  uint32_t OSSIState;            /*!< Specifies the Off-State used in Idle state.
                                      This parameter can be a value of @ref TIM_LL_EC_OSSI

                                      This feature can be modified afterwards using unitary function @ref LL_TIM_SetOffStates()

                                      @note This bit-field cannot be modified as long as LOCK level 2 has been programmed. */

  uint32_t LockLevel;            /*!< Specifies the LOCK level parameters.
                                      This parameter can be a value of @ref TIM_LL_EC_LOCKLEVEL

                                      @note The LOCK bits can be written only once after the reset. Once the TIMx_BDTR register
                                            has been written, their content is frozen until the next reset.*/

  uint8_t DeadTime;              /*!< Specifies the delay time between the switching-off and the
                                      switching-on of the outputs.
                                      This parameter can be a number between Min_Data = 0x00 and Max_Data = 0xFF.

                                      This feature can be modified afterwards using unitary function @ref LL_TIM_OC_SetDeadTime()

                                      @note This bit-field can not be modified as long as LOCK level 1, 2 or 3 has been programmed. */

  uint16_t BreakState;           /*!< Specifies whether the TIM Break input is enabled or not.
                                      This parameter can be a value of @ref TIM_LL_EC_BREAK_ENABLE

                                      This feature can be modified afterwards using unitary functions @ref LL_TIM_EnableBRK() or @ref LL_TIM_DisableBRK()

                                      @note This bit-field can not be modified as long as LOCK level 1 has been programmed. */

  uint32_t BreakPolarity;        /*!< Specifies the TIM Break Input pin polarity.
                                      This parameter can be a value of @ref TIM_LL_EC_BREAK_POLARITY

                                      This feature can be modified afterwards using unitary function @ref LL_TIM_ConfigBRK()

                                      @note This bit-field can not be modified as long as LOCK level 1 has been programmed. */

  uint32_t AutomaticOutput;      /*!< Specifies whether the TIM Automatic Output feature is enabled or not.
                                      This parameter can be a value of @ref TIM_LL_EC_AUTOMATICOUTPUT_ENABLE

                                      This feature can be modified afterwards using unitary functions @ref LL_TIM_EnableAutomaticOutput() or @ref LL_TIM_DisableAutomaticOutput()

                                      @note This bit-field can not be modified as long as LOCK level 1 has been programmed. */
} LL_TIM_BDTR_InitTypeDef;
```

- 运行模式下的关闭状态选择`OSSRState`

```c
/** @defgroup TIM_LL_EC_OSSR OSSR
  * @{
  */
#define LL_TIM_OSSR_DISABLE                    0x00000000U             /*!< When inactive, OCx/OCxN outputs are disabled */
#define LL_TIM_OSSR_ENABLE                     TIM_BDTR_OSSR           /*!< When inactive, OC/OCN outputs are enabled with their inactive level as soon as CCxE=1 or CCxNE=1 */
```
- 空闲模式下的关闭状态选择`OSSIState`

```c
/** @defgroup TIM_LL_EC_OSSI OSSI
  * @{
  */
#define LL_TIM_OSSI_DISABLE                    0x00000000U             /*!< When inactive, OCx/OCxN outputs are disabled */
#define LL_TIM_OSSI_ENABLE                     TIM_BDTR_OSSI           /*!< When inactive, OxC/OCxN outputs are first forced with their inactive level then forced to their idle level after the deadtime */
```
- 锁定配置`LockLevel`

```c
/** @defgroup TIM_LL_EC_LOCKLEVEL Lock Level
  * @{
  */
#define LL_TIM_LOCKLEVEL_OFF                   0x00000000U          /*!< LOCK OFF - No bit is write protected */
#define LL_TIM_LOCKLEVEL_1                     TIM_BDTR_LOCK_0      /*!< LOCK Level 1 */
#define LL_TIM_LOCKLEVEL_2                     TIM_BDTR_LOCK_1      /*!< LOCK Level 2 */
#define LL_TIM_LOCKLEVEL_3                     TIM_BDTR_LOCK        /*!< LOCK Level 3 */
```
- 死区时间`DeadTime`

- 断路输入使能控制`BreakState`

```c
#if defined(USE_FULL_LL_DRIVER)
/** @defgroup TIM_LL_EC_BREAK_ENABLE Break Enable
  * @{
  */
#define LL_TIM_BREAK_DISABLE            0x00000000U             /*!< Break function disabled */
#define LL_TIM_BREAK_ENABLE             TIM_BDTR_BKE            /*!< Break function enabled */
```
- 断路输入极性`BreakPolarity`

```c
/** @defgroup TIM_LL_EC_BREAK_POLARITY break polarity
  * @{
  */
#define LL_TIM_BREAK_POLARITY_LOW              0x00000000U               /*!< Break input BRK is active low */
#define LL_TIM_BREAK_POLARITY_HIGH             TIM_BDTR_BKP              /*!< Break input BRK is active high */

```
- 自动输出使能`AutomaticOutput`

```c
/** @defgroup TIM_LL_EC_AUTOMATICOUTPUT_ENABLE Automatic output enable
  * @{
  */
#define LL_TIM_AUTOMATICOUTPUT_DISABLE         0x00000000U             /*!< MOE can be set only by software */
#define LL_TIM_AUTOMATICOUTPUT_ENABLE          TIM_BDTR_AOE            /*!< MOE can be set by software or automatically at the next update event */
```

# 代码移植
## 定时器应用
- 在主函数初始化添加TIM启动

```c
    LL_TIM_EnableCounter(TIM1);//使能定时器
    LL_TIM_CC_EnableChannel(TIM1,LL_TIM_CHANNEL_CH1|LL_TIM_CHANNEL_CH1N);//使能通道
    LL_TIM_EnableAllOutputs(TIM1);//主输出使能
```

## 下载调试
- 编译之后下载到开发板
- 由于我们SPL库对PB12使用的推挽输出，先把输出置低了，HAL,LL库使用的是浮空输入，需要把PB12接地才能输出
- 使用示波器查看输出波形,波形为1M,50%占空比方波
- 两路互补带死区
- PB12接高电平，PWM输出被禁止
- 和HAL库结果一致
