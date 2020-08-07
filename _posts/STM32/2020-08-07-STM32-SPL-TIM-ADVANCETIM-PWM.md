---
title: STM32标准外设库高级定时器PWM输出
categories: STM32 TIM PWM
tags: STM32 SPL TIM PWM
description: SPL库高级定时器PWM带互补输出
---
# 定时器通道引脚分布

<table>
<thead>
<tr>
<th style="text-align: center" rowspan="2">通道</th>
<th style="text-align: center" colspan="2">高级定时器</th>
<th style="text-align: center" colspan="4">通用定时器</th>
</tr>
<tr>
<th style="text-align: center">TIM1</th>
<th style="text-align: center">TIM8</th>
<th style="text-align: center">TIM2</th>
<th style="text-align: center">TIM3</th>
<th style="text-align: center">TIM4</th>
<th style="text-align: center">TIM5</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center">CH1</th>
<td style="text-align: center">PA8</td>
<td style="text-align: center">PC6</td>
<td style="text-align: center">PA0/PA15</td>
<td style="text-align: center">PA0</td>
<td style="text-align: center">PA6/PC6/PB4</td>
<td style="text-align: center">PB6/PD12</td>
</tr>
<tr>
<th style="text-align: center">CH1N</th>
<td style="text-align: center">PB13/PA7</td>
<td style="text-align: center">PA7</td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
</tr>
<tr>
<th style="text-align: center">CH2</th>
<td style="text-align: center">PA9</td>
<td style="text-align: center">PC7</td>
<td style="text-align: center">PA1/PB3</td>
<td style="text-align: center">PA1</td>
<td style="text-align: center">PA7/PC7/PB5</td>
<td style="text-align: center">PB7/PD13</td>
</tr>
<tr>
<th style="text-align: center">CH2N</th>
<td style="text-align: center">PB14/PB0</td>
<td style="text-align: center">PB0</td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
</tr>
<tr>
<th style="text-align: center">CH3</th>
<td style="text-align: center">PA10</td>
<td style="text-align: center">PC8</td>
<td style="text-align: center">PA2/PB10</td>
<td style="text-align: center">PA2</td>
<td style="text-align: center">PB0/PC8</td>
<td style="text-align: center">PB8/PD14</td>
</tr>
<tr>
<th style="text-align: center">CH3N</th>
<td style="text-align: center">PB15/PB1</td>
<td style="text-align: center">PB1</td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
</tr>
<tr>
<th style="text-align: center">CH4</th>
<td style="text-align: center">PA11</td>
<td style="text-align: center">PC9</td>
<td style="text-align: center">PA3/PB11</td>
<td style="text-align: center">PA3</td>
<td style="text-align: center">PB1/PC9</td>
<td style="text-align: center">PB9/PD15</td>
</tr>
<tr>
<th style="text-align: center">ETR</th>
<td style="text-align: center">PA12</td>
<td style="text-align: center">PA0</td>
<td style="text-align: center">PA0/PA15</td>
<td style="text-align: center"></td>
<td style="text-align: center">PD2</td>
<td style="text-align: center">PE0</td>
</tr>
<tr>
<th style="text-align: center">BKIN</th>
<td style="text-align: center">PB12/PA6</td>
<td style="text-align: center">PA6</td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
</tr>
</tbody>
</table>


# TIM初始化
## TIM宏定义

- `my_advance.pwm.h`

```c
#ifndef __ADVANCE_PWM_H
#define __ADVANCE_PWM_H
#include "stm32f10x.h"


/************高级定时器TIM参数定义，只限TIM1和TIM8************/
// 当使用不同的定时器的时候，对应的GPIO是不一样的，这点要注意
// 这里我们使用高级控制定时器TIM1

#define            ADVANCE_TIM                   TIM1
#define            ADVANCE_TIM_APBxClock_FUN     RCC_APB2PeriphClockCmd
#define            ADVANCE_TIM_CLK               RCC_APB2Periph_TIM1
// PWM 信号的频率 F = TIM_CLK/{(ARR+1)*(PSC+1)}
#define            ADVANCE_TIM_PERIOD            (8-1)
#define            ADVANCE_TIM_PSC               (9-1)
#define            ADVANCE_TIM_PULSE             4

// TIM1 输出比较通道
#define            ADVANCE_TIM_CH1_GPIO_CLK      RCC_APB2Periph_GPIOA
#define            ADVANCE_TIM_CH1_PORT          GPIOA
#define            ADVANCE_TIM_CH1_PIN           GPIO_Pin_8

// TIM1 输出比较通道的互补通道
#define            ADVANCE_TIM_CH1N_GPIO_CLK      RCC_APB2Periph_GPIOB
#define            ADVANCE_TIM_CH1N_PORT          GPIOB
#define            ADVANCE_TIM_CH1N_PIN           GPIO_Pin_13

// TIM1 输出比较通道的刹车通道
#define            ADVANCE_TIM_BKIN_GPIO_CLK      RCC_APB2Periph_GPIOB
#define            ADVANCE_TIM_BKIN_PORT          GPIOB
#define            ADVANCE_TIM_BKIN_PIN           GPIO_Pin_12

/**************************函数声明********************************/

void ADVANCE_PWM_Init(void);
#endif

```
## TIM配置和PWM设置
- `my_advance_pwm.c`

```c
#include "my_advance_pwm.h"
/**
 * @brief  PWM GPIO配置
 * @param  无
 * @retval 无
 */
static void ADVANCE_TIM_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    // 输出比较通道 GPIO 初始化
    RCC_APB2PeriphClockCmd(ADVANCE_TIM_CH1_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  ADVANCE_TIM_CH1_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(ADVANCE_TIM_CH1_PORT, &GPIO_InitStructure);

    // 输出比较通道互补通道 GPIO 初始化
    RCC_APB2PeriphClockCmd(ADVANCE_TIM_CH1N_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  ADVANCE_TIM_CH1N_PIN;
    GPIO_Init(ADVANCE_TIM_CH1N_PORT, &GPIO_InitStructure);

    // 输出比较通道刹车通道 GPIO 初始化
    RCC_APB2PeriphClockCmd(ADVANCE_TIM_BKIN_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  ADVANCE_TIM_BKIN_PIN;
    GPIO_Init(ADVANCE_TIM_BKIN_PORT, &GPIO_InitStructure);
    // BKIN引脚默认先输出低电平
    GPIO_ResetBits(ADVANCE_TIM_BKIN_PORT,ADVANCE_TIM_BKIN_PIN);
}


/* ----------------   PWM信号 周期和占空比的计算--------------- */
// ARR ：TIM_Period
// CLK_cnt：计数器的时钟，等于 Fck_int / (TIM_Prescaler+1) = 72M/(TIM_Prescaler+1)
// PWM 信号的周期 T = (TIM_Period+1) * (1/CLK_cnt) = (TIM_Period+1)*(TIM_Prescaler+1) / 72M
// 占空比P=TIM_Pulse/(TIM_Period+1)
/**
 * @brief  PWM 定时器配置
 * @param  无
 * @retval 无
 */
static void ADVANCE_TIM_Mode_Config(void)
{
    // 开启定时器时钟,即内部时钟CK_INT=72M
    ADVANCE_TIM_APBxClock_FUN(ADVANCE_TIM_CLK,ENABLE);

    /*--------------------时基结构体初始化-------------------------*/
    TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure;
    // 自动重装载寄存器的值，累计TIM_Period+1个频率后产生一个更新或者中断
    TIM_TimeBaseStructure.TIM_Period=ADVANCE_TIM_PERIOD;
    // 驱动CNT计数器的时钟 = Fck_int/(psc+1)
    TIM_TimeBaseStructure.TIM_Prescaler= ADVANCE_TIM_PSC;
    // 时钟分频因子 ，配置死区时间时需要用到
    TIM_TimeBaseStructure.TIM_ClockDivision=TIM_CKD_DIV1;
    // 计数器计数模式，设置为向上计数
    TIM_TimeBaseStructure.TIM_CounterMode=TIM_CounterMode_Up;
    // 重复计数器的值，没用到不用管
    TIM_TimeBaseStructure.TIM_RepetitionCounter=0;
    // 初始化定时器
    TIM_TimeBaseInit(ADVANCE_TIM, &TIM_TimeBaseStructure);

    /*--------------------输出比较结构体初始化-------------------*/
    TIM_OCInitTypeDef  TIM_OCInitStructure;
    // 配置为PWM模式1
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
    // 输出使能
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
    // 互补输出使能
    TIM_OCInitStructure.TIM_OutputNState = TIM_OutputNState_Enable;
    // 设置占空比大小
    TIM_OCInitStructure.TIM_Pulse = ADVANCE_TIM_PULSE;
    // 输出通道电平极性配置
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
    // 互补输出通道电平极性配置
    TIM_OCInitStructure.TIM_OCNPolarity = TIM_OCNPolarity_High;
    // 输出通道空闲电平极性配置
    TIM_OCInitStructure.TIM_OCIdleState = TIM_OCIdleState_Set;
    // 互补输出通道空闲电平极性配置
    TIM_OCInitStructure.TIM_OCNIdleState = TIM_OCNIdleState_Reset;
    TIM_OC1Init(ADVANCE_TIM, &TIM_OCInitStructure);
    TIM_OC1PreloadConfig(ADVANCE_TIM, TIM_OCPreload_Enable);

    /*-------------------刹车和死区结构体初始化-------------------*/
    // 有关刹车和死区结构体的成员具体可参考BDTR寄存器的描述
    TIM_BDTRInitTypeDef TIM_BDTRInitStructure;
    TIM_BDTRInitStructure.TIM_OSSRState = TIM_OSSRState_Enable;
    TIM_BDTRInitStructure.TIM_OSSIState = TIM_OSSIState_Enable;
    TIM_BDTRInitStructure.TIM_LOCKLevel = TIM_LOCKLevel_1;
    // 输出比较信号死区时间配置，具体如何计算可参考 BDTR:UTG[7:0]的描述
    // 这里配置的死区时间为152ns
    TIM_BDTRInitStructure.TIM_DeadTime = 11;
    TIM_BDTRInitStructure.TIM_Break = TIM_Break_Enable;
    // 当BKIN引脚检测到高电平的时候，输出比较信号被禁止，就好像是刹车一样
    TIM_BDTRInitStructure.TIM_BreakPolarity = TIM_BreakPolarity_High;
    TIM_BDTRInitStructure.TIM_AutomaticOutput = TIM_AutomaticOutput_Enable;
    TIM_BDTRConfig(ADVANCE_TIM, &TIM_BDTRInitStructure);

    // 使能计数器
    TIM_Cmd(ADVANCE_TIM, ENABLE);
    // 主输出使能，当使用的是通用定时器时，这句不需要
    TIM_CtrlPWMOutputs(ADVANCE_TIM, ENABLE);
}

void ADVANCE_PWM_Init(void)
{
    ADVANCE_TIM_GPIO_Config();
    ADVANCE_TIM_Mode_Config();
}

```
### 输出比较结构体分析

```c
/** 
  * @brief  TIM Output Compare Init structure definition  
  */

typedef struct
{
  uint16_t TIM_OCMode;        /*!< Specifies the TIM mode.
                                   This parameter can be a value of @ref TIM_Output_Compare_and_PWM_modes */

  uint16_t TIM_OutputState;   /*!< Specifies the TIM Output Compare state.
                                   This parameter can be a value of @ref TIM_Output_Compare_state */

  uint16_t TIM_OutputNState;  /*!< Specifies the TIM complementary Output Compare state.
                                   This parameter can be a value of @ref TIM_Output_Compare_N_state
                                   @note This parameter is valid only for TIM1 and TIM8. */

  uint16_t TIM_Pulse;         /*!< Specifies the pulse value to be loaded into the Capture Compare Register. 
                                   This parameter can be a number between 0x0000 and 0xFFFF */

  uint16_t TIM_OCPolarity;    /*!< Specifies the output polarity.
                                   This parameter can be a value of @ref TIM_Output_Compare_Polarity */

  uint16_t TIM_OCNPolarity;   /*!< Specifies the complementary output polarity.
                                   This parameter can be a value of @ref TIM_Output_Compare_N_Polarity
                                   @note This parameter is valid only for TIM1 and TIM8. */

  uint16_t TIM_OCIdleState;   /*!< Specifies the TIM Output Compare pin state during Idle state.
                                   This parameter can be a value of @ref TIM_Output_Compare_Idle_State
                                   @note This parameter is valid only for TIM1 and TIM8. */

  uint16_t TIM_OCNIdleState;  /*!< Specifies the TIM Output Compare pin state during Idle state.
                                   This parameter can be a value of @ref TIM_Output_Compare_N_Idle_State
                                   @note This parameter is valid only for TIM1 and TIM8. */
} TIM_OCInitTypeDef;
```
- 比较输出模式`TIM_OCMode`

```c
/** @defgroup TIM_Output_Compare_and_PWM_modes 
  * @{
  */

#define TIM_OCMode_Timing                  ((uint16_t)0x0000) //冻结
#define TIM_OCMode_Active                  ((uint16_t)0x0010) //匹配时设置通道1为有效电平
#define TIM_OCMode_Inactive                ((uint16_t)0x0020) //匹配时设置通道1为无效电平
#define TIM_OCMode_Toggle                  ((uint16_t)0x0030) //翻转
#define TIM_OCMode_PWM1                    ((uint16_t)0x0060) //PWM模式1
#define TIM_OCMode_PWM2                    ((uint16_t)0x0070) //PWM模式2
```
- 比较输出使能`TIM_OutputState`

```c
/** @defgroup TIM_Output_Compare_state 
  * @{
  */

#define TIM_OutputState_Disable            ((uint16_t)0x0000) //禁止
#define TIM_OutputState_Enable             ((uint16_t)0x0001) //使能
```
- 比较互补输出使能`TIM_OutputNState`

```c
/** @defgroup TIM_Output_Compare_N_state 
  * @{
  */

#define TIM_OutputNState_Disable           ((uint16_t)0x0000) //禁止
#define TIM_OutputNState_Enable            ((uint16_t)0x0004) //使能
```
- 脉冲宽度`TIM_Pulse`

设定比较寄存器CCR 的值，决定脉冲宽度
- 输出极性`TIM_OCPolarity`

```c
/** @defgroup TIM_Output_Compare_Polarity 
  * @{
  */

#define TIM_OCPolarity_High                ((uint16_t)0x0000) //高
#define TIM_OCPolarity_Low                 ((uint16_t)0x0002) //低
```
- 互补输出极性`TIM_OCNPolarity`

```c
/** @defgroup TIM_Output_Compare_N_Polarity 
  * @{
  */
  
#define TIM_OCNPolarity_High               ((uint16_t)0x0000) //高
#define TIM_OCNPolarity_Low                ((uint16_t)0x0008) //低
```
- 空闲状态下比较输出状态`TIM_OCIdleState`

```c
/** @defgroup TIM_Output_Compare_Idle_State 
  * @{
  */

#define TIM_OCIdleState_Set                ((uint16_t)0x0100) //置位
#define TIM_OCIdleState_Reset              ((uint16_t)0x0000) //复位
```
- 空闲状态下比较互补输出状态`TIM_OCNIdleState`

```c
/** @defgroup TIM_Output_Compare_N_Idle_State 
  * @{
  */

#define TIM_OCNIdleState_Set               ((uint16_t)0x0200) //置位
#define TIM_OCNIdleState_Reset             ((uint16_t)0x0000) //复位
```
### 输出比较结构体初始化通道选择

|通道|初始化函数|CCR使能预加载函数|快速使能函数|强制输出函数|输出比较清0使能函数|
|:----:|:----:|:----:|:----:|:----:|:----:|
|Channel_1|TIM_OC1Init|TIM_OC1PreloadConfig|TIM_OC1FastConfig|TIM_ForcedOC1Config|TIM_ClearOC1Ref|
|Channel_2|TIM_OC2Init|TIM_OC2PreloadConfig|TIM_OC2FastConfig|TIM_ForcedOC2Config|TIM_ClearOC2Ref|
|Channel_3|TIM_OC3Init|TIM_OC3PreloadConfig|TIM_OC3FastConfig|TIM_ForcedOC3Config|TIM_ClearOC3Ref|
|Channel_4|TIM_OC4Init|TIM_OC4PreloadConfig|TIM_OC4FastConfig|TIM_ForcedOC4Config|TIM_ClearOC4Ref|

- CCR使能预加载

```c
/** @defgroup TIM_Output_Compare_Preload_State 
  * @{
  */

#define TIM_OCPreload_Enable               ((uint16_t)0x0008) //开启TIMx_CCR1寄存器的预装载功能，读写操作仅对预装载寄存器操作，TIMx_CCR1的预装载值在更新事件到来时被加载至当前寄存器中
#define TIM_OCPreload_Disable              ((uint16_t)0x0000) //禁止TIMx_CCR1寄存器的预装载功能，可随时写入TIMx_CCR1寄存器，并且新写入的数值立即起作用。
```
注1：一旦LOCK级别设为3(TIMx_BDTR寄存器中的LOCK位)并且CC1S=00(该通道配置成输出)则该位不能被修改。<br/> 
注2：仅在单脉冲模式下(TIMx_CR1寄存器的OPM=1)，可以在未确认预装载寄存器情况下使用PWM模式，否则其动作不确定。

- 快速使能函数

```c
/** @defgroup TIM_Output_Compare_Fast_State 
  * @{
  */

#define TIM_OCFast_Enable                  ((uint16_t)0x0004) //输入到触发器的有效沿的作用就象发生了一次比较匹配。因此，OC被设置为比较电平而与比较结果无关。采样触发器的有效沿和CC输出间的延时被缩短为3个时钟周期
#define TIM_OCFast_Disable                 ((uint16_t)0x0000) //根据计数器与CCR的值，CC正常操作，即使触发器是打开的。当触发器的输入有一个有效沿时，激活CC输出的最小延时为5个时钟周期
```
注：OCFE只在通道被配置成PWM1或PWM2模式时起作用

- 强制输出函数

```c
/** @defgroup TIM_Forced_Action 
  * @{
  */

#define TIM_ForcedAction_Active            ((uint16_t)0x0050) //：强制为有效电平。强制OC1REF为高
#define TIM_ForcedAction_InActive          ((uint16_t)0x0040) //强制为无效电平。强制OC1REF为低
```

- 输出比较清0使能函数

```c
/** @defgroup TIM_Output_Compare_Clear_State 
  * @{
  */

#define TIM_OCClear_Enable                 ((uint16_t)0x0080) //一旦检测到ETRF输入高电平，清除OCREF=0。
#define TIM_OCClear_Disable                ((uint16_t)0x0000) //OCREF 不受ETRF输入的影响；
```

### 断路和死区结构体分析

```c
/** 
  * @brief  BDTR structure definition 
  * @note   This structure is used only with TIM1 and TIM8.    
  */

typedef struct
{

  uint16_t TIM_OSSRState;        /*!< Specifies the Off-State selection used in Run mode.
                                      This parameter can be a value of @ref OSSR_Off_State_Selection_for_Run_mode_state */

  uint16_t TIM_OSSIState;        /*!< Specifies the Off-State used in Idle state.
                                      This parameter can be a value of @ref OSSI_Off_State_Selection_for_Idle_mode_state */

  uint16_t TIM_LOCKLevel;        /*!< Specifies the LOCK level parameters.
                                      This parameter can be a value of @ref Lock_level */ 

  uint16_t TIM_DeadTime;         /*!< Specifies the delay time between the switching-off and the
                                      switching-on of the outputs.
                                      This parameter can be a number between 0x00 and 0xFF  */

  uint16_t TIM_Break;            /*!< Specifies whether the TIM Break input is enabled or not. 
                                      This parameter can be a value of @ref Break_Input_enable_disable */

  uint16_t TIM_BreakPolarity;    /*!< Specifies the TIM Break Input pin polarity.
                                      This parameter can be a value of @ref Break_Polarity */

  uint16_t TIM_AutomaticOutput;  /*!< Specifies whether the TIM Automatic Output feature is enabled or not. 
                                      This parameter can be a value of @ref TIM_AOE_Bit_Set_Reset */
} TIM_BDTRInitTypeDef;
```
- 运行模式下的关闭状态选择`TIM_OSSRState`

```c
/** @defgroup OSSR_Off_State_Selection_for_Run_mode_state 
  * @{
  */

#define TIM_OSSRState_Enable               ((uint16_t)0x0800) //当定时器不工作时，一旦CCxE=1或CCxNE=1，首先开启OC/OCN并输出无效电平，然后置OC/OCN使能输出信号=1
#define TIM_OSSRState_Disable              ((uint16_t)0x0000) //当定时器不工作时，禁止OC/OCN输出(OC/OCN使能输出信号=0)
```
注：一旦LOCK级别(TIMx_BDTR寄存器中的LOCK位)设为2，则该位不能被修改。

- 空闲模式下的关闭状态选择`TIM_OSSIState`

```c
/** @defgroup OSSI_Off_State_Selection_for_Idle_mode_state 
  * @{
  */

#define TIM_OSSIState_Enable               ((uint16_t)0x0400) //当定时器不工作时，一旦CCxE=1或CCxNE=1，OC/OCN首先输出其空闲电平，然后OC/OCN使能输出信号=1
#define TIM_OSSIState_Disable              ((uint16_t)0x0000) //当定时器不工作时，禁止OC/OCN输出(OC/OCN使能输出信号=0)；
```
注：一旦LOCK级别(TIMx_BDTR寄存器中的LOCK位)设为2，则该位不能被修改。

- 锁定配置`TIM_LOCKLevel`

```c
/** @defgroup Lock_level 
  * @{
  */

#define TIM_LOCKLevel_OFF                  ((uint16_t)0x0000) //锁定关闭，寄存器无写保护
#define TIM_LOCKLevel_1                    ((uint16_t)0x0100) //锁定级别1，不能写入TIMx_BDTR寄存器的DTG、BKE、BKP、AOE位和TIMx_CR2寄存器的OISx/OISxN位
#define TIM_LOCKLevel_2                    ((uint16_t)0x0200) //锁定级别2，不能写入锁定级别1中的各位，也不能写入CC极性位(一旦相关通道通过CCxS位设为输出，CC极性位是TIMx_CCER寄存器的CCxP/CCNxP位)以及OSSR/OSSI位
#define TIM_LOCKLevel_3                    ((uint16_t)0x0300) //锁定级别3，不能写入锁定级别2中的各位，也不能写入CC控制位(一旦相关通道通过CCxS位设为输出，CC控制位是TIMx_CCMRx寄存器的OCxM/OCxPE位)
```
注：在系统复位后，只能写一次LOCK位，一旦写入TIMx_BDTR寄存器，则其内容冻结直至复位。

- 死区时间`TIM_DeadTime`

可设置范围为0 至255<br/>
DT表示其持续时间:<br/>
DTG[7:5]=0xx => DT=DTG[7:0] × T<sub>dtg</sub>，T<sub>dtg</sub> = T<sub>DTS</sub>；<br/> 
DTG[7:5]=10x => DT=(64+DTG[5:0]) × T<sub>dtg</sub>，T<sub>dtg</sub>= 2 × T<sub>DTS</sub>；<br/> 
DTG[7:5]=110 => DT=(32+DTG[4:0]) × T<sub>dtg</sub>，T<sub>dtg</sub> = 8 × T<sub>DTS</sub>；<br/>  
DTG[7:5]=111 => DT=(32+DTG[4:0])× T<sub>dtg</sub>，T<sub>dtg</sub> = 16 × T<sub>DTS</sub>；<br/> 
本例：因`TIM_TimeBaseStructure.TIM_ClockDivision=TIM_CKD_DIV1`，故T<sub>DTS</sub>=1/72M，DT=11/72M=152ns
- 断路输入使能控制`TIM_Break`

```c
/** @defgroup Break_Input_enable_disable 
  * @{
  */

#define TIM_Break_Enable                   ((uint16_t)0x1000) //使能
#define TIM_Break_Disable                  ((uint16_t)0x0000) //禁止
```
- 断路输入极性`TIM_BreakPolarity`

```c
/** @defgroup Break_Polarity 
  * @{
  */

#define TIM_BreakPolarity_Low              ((uint16_t)0x0000) //低电平有效
#define TIM_BreakPolarity_High             ((uint16_t)0x2000) //高电平有效
```
- 自动输出使能`TIM_AutomaticOutput`

```c
/** @defgroup TIM_AOE_Bit_Set_Reset 
  * @{
  */

#define TIM_AutomaticOutput_Enable         ((uint16_t)0x4000) //使能
#define TIM_AutomaticOutput_Disable        ((uint16_t)0x0000) //禁止
```
# 调试
## 添加PWM头文件`main.c`和调用初始化PWM
- `main.c`

```c
#include "stm32f10x.h"
#include "my_gpio.h"
#include "my_usart.h"
#include "my_data_queue.h"
#include "my_process_data.h"
#include "my_adc.h"
#include "my_tim.h"
#include "my_advance_pwm.h"
extern __IO uint16_t ADC_ConvertedValue[NOFCHANEL];
Channel ch[2];
float VREF,VSENSE;
float temp;
float adcValue;
__IO uint32_t time = 0; // ms 计时变量
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
    /*初始化TIM 1ms*/
    MY_TIM_Init();
    /*初始化PWM  1MHz 50%*/
    ADVANCE_PWM_Init();
    printf( "\r\n Print current Temperature  \r\n");
    while (1)
    {
        if ( time >= 1000 ) /* 1000 * 1 ms = 1s 时间到 */
        {
            time = 0;
            /* LED1 取反 */
            LED1_TOGGLE;
            VREF=1.2f*4096/ADC_ConvertedValue[2];
            VSENSE=VREF*ADC_ConvertedValue[1]/4096;
            temp=(1.43f-VSENSE)/4.3+25;
            adcValue=VREF*ADC_ConvertedValue[0]/4096;
            printf( "\r\n The IC current tem= %.2fC\r\n", temp);
            printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
            printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
        }
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
- 使用示波器查看输出波形,波形为1M,50%占空比方波
- 两路互补带死区
- PB12接高电平，PWM输出被禁止