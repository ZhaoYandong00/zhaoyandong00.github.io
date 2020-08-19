---
title: STM32硬件层HAL库ADC使用DMA多通道采集
categories: STM32 CUBE HAL ADC DMA
tags: STM32 CUBE HAL ADC DMA
description: HAL库ADC使用DMA多通道采集
---
# 配置ADC
- 打开CUBE工程

## 在**`Analog`**->**`ADC1`**配置
- 选择通道`IN11`,`Temperature Sensor Channel`,`Vrefint Channel`
- **`Parameter Settings`**配置
- 设置模式——`Independent mode`
- 数据对齐——`Right alignment`,扫描模式——`Enabled`,连续转换——`Enabled`,间断转换模式——`Disable`
- 规则转换——`Enable`,转换数量——`3`,外部触发——`Regular Conversion lanuched by software`
- Rank1 通道——`Channel 11`,采样时间——`55.5 Cycles`
- Rank2 通道——`Temperature Sensor Channel`,采样时间——`239.5 Cycles`
- Rank3 通道——`Vrefint Channel`,采样时间——`239.5 Cycles`
- **`DMA_Settings`**
- 配置ADC1 优先级——`High` 模式——`Circle` 存储器地址自增 外设不自增 数据半字——`Half Word`

## 时钟配置**`Clock Configuration`**
- ADC分频因子`ADC Prescaler`设为`8`,即ADC时钟为9M

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# ADC函数
## ADC结构体分析

```c
/**
  * @brief  ADC handle Structure definition  
  */ 
typedef struct __ADC_HandleTypeDef
{
  ADC_TypeDef                   *Instance;              /*!< Register base address */

  ADC_InitTypeDef               Init;                   /*!< ADC required parameters */

  DMA_HandleTypeDef             *DMA_Handle;            /*!< Pointer DMA Handler */

  HAL_LockTypeDef               Lock;                   /*!< ADC locking object */
  
  __IO uint32_t                 State;                  /*!< ADC communication state (bitmap of ADC states) */

  __IO uint32_t                 ErrorCode;              /*!< ADC Error code */

#if (USE_HAL_ADC_REGISTER_CALLBACKS == 1)
  void (* ConvCpltCallback)(struct __ADC_HandleTypeDef *hadc);              /*!< ADC conversion complete callback */
  void (* ConvHalfCpltCallback)(struct __ADC_HandleTypeDef *hadc);          /*!< ADC conversion DMA half-transfer callback */
  void (* LevelOutOfWindowCallback)(struct __ADC_HandleTypeDef *hadc);      /*!< ADC analog watchdog 1 callback */
  void (* ErrorCallback)(struct __ADC_HandleTypeDef *hadc);                 /*!< ADC error callback */
  void (* InjectedConvCpltCallback)(struct __ADC_HandleTypeDef *hadc);      /*!< ADC group injected conversion complete callback */       /*!< ADC end of sampling callback */
  void (* MspInitCallback)(struct __ADC_HandleTypeDef *hadc);               /*!< ADC Msp Init callback */
  void (* MspDeInitCallback)(struct __ADC_HandleTypeDef *hadc);             /*!< ADC Msp DeInit callback */
#endif /* USE_HAL_ADC_REGISTER_CALLBACKS */
}ADC_HandleTypeDef;
```
- ADC寄存器基本地址`Instance`

可选参数：**`ADC1`** **`ADC2`** **`ADC3`**
- ADC初始化结构体`Init`
- DMA指针`DMA_Handle`
- 锁`Lock`
- ADC状态`State`

```c
/** 
  * @brief  HAL ADC state machine: ADC states definition (bitfields)
  */ 
/* States of ADC global scope */
#define HAL_ADC_STATE_RESET             0x00000000U    /*!< ADC not yet initialized or disabled *///ADC没有初始化
#define HAL_ADC_STATE_READY             0x00000001U    /*!< ADC peripheral ready for use *///ADC初始化完成和准备好
#define HAL_ADC_STATE_BUSY_INTERNAL     0x00000002U    /*!< ADC is busy to internal process (initialization, calibration) *///ADC在忙(初始化,校准)
#define HAL_ADC_STATE_TIMEOUT           0x00000004U    /*!< TimeOut occurrence *///超时

/* States of ADC errors */
#define HAL_ADC_STATE_ERROR_INTERNAL    0x00000010U    /*!< Internal error occurrence *///内部错误发生
#define HAL_ADC_STATE_ERROR_CONFIG      0x00000020U    /*!< Configuration error occurrence *///配置错误发生
#define HAL_ADC_STATE_ERROR_DMA         0x00000040U    /*!< DMA error occurrence *///DMA错误发生

/* States of ADC group regular */
#define HAL_ADC_STATE_REG_BUSY          0x00000100U    /*!< A conversion on group regular is ongoing or can occur (either by continuous mode,external trigger, low power auto power-on, multimode ADC master control) *///规则转换正在进行
#define HAL_ADC_STATE_REG_EOC           0x00000200U    /*!< Conversion data available on group regular *///规则转换结束
#define HAL_ADC_STATE_REG_OVR           0x00000400U    /*!< Not available on STM32F1 device: Overrun occurrence *///溢出(STM32F1系列无)
#define HAL_ADC_STATE_REG_EOSMP         0x00000800U    /*!< Not available on STM32F1 device: End Of Sampling flag raised  *///取样结束标志升起(STM32F1系列无)

/* States of ADC group injected */
#define HAL_ADC_STATE_INJ_BUSY          0x00001000U    /*!< A conversion on group injected is ongoing or can occur (either by auto-injection mode,external trigger, low power auto power-on, multimode ADC master control) *///注入转换正在进行
#define HAL_ADC_STATE_INJ_EOC           0x00002000U    /*!< Conversion data available on group injected *///注入转换结束
#define HAL_ADC_STATE_INJ_JQOVF         0x00004000U    /*!< Not available on STM32F1 device: Injected queue overflow occurrence *///溢出(STM32F1系列无)

/* States of ADC analog watchdogs */
#define HAL_ADC_STATE_AWD1              0x00010000U    /*!< Out-of-window occurrence of analog watchdog 1 *///模拟看门狗1事件
#define HAL_ADC_STATE_AWD2              0x00020000U    /*!< Not available on STM32F1 device: Out-of-window occurrence of analog watchdog 2 *///模拟看门狗2事件(STM32F1系列无)
#define HAL_ADC_STATE_AWD3              0x00040000U    /*!< Not available on STM32F1 device: Out-of-window occurrence of analog watchdog 3 *///模拟看门狗3事件(STM32F1系列无)

/* States of ADC multi-mode */
#define HAL_ADC_STATE_MULTIMODE_SLAVE   0x00100000U    /*!< ADC in multimode slave state, controlled by another ADC master ( *///ADC处于多模式从属状态，由另一个ADC主机控制
```

- 错误码`ErrorCode`

```c
/** @defgroup ADC_Error_Code ADC Error Code
  * @{
  */
#define HAL_ADC_ERROR_NONE                0x00U   /*!< No error*///无错误
#define HAL_ADC_ERROR_INTERNAL            0x01U   /*!< ADC IP internal error: if problem of clocking, enable/disable, erroneous state *///内部错误,如时钟错误，启用/禁用,错误状态
#define HAL_ADC_ERROR_OVR                 0x02U   /*!< Overrun error *///溢出错误
#define HAL_ADC_ERROR_DMA                 0x04U   /*!< DMA transfer error*///DMA传输错误

#if (USE_HAL_ADC_REGISTER_CALLBACKS == 1)
#define HAL_ADC_ERROR_INVALID_CALLBACK  (0x10U)   /*!< Invalid Callback error *///无效的回调
#endif /* USE_HAL_ADC_REGISTER_CALLBACKS */
```
- 转换完成回调`ConvCpltCallback`
- 转换一半完成回调`ConvHalfCpltCallback`
- 看门狗事件回调`LevelOutOfWindowCallback`
- 错误回调`ErrorCallback`
- 注入转换完成回调`InjectedConvCpltCallback`
- 初始化完成回调`MspInitCallback`
- 反初始化完成回调`MspDeInitCallback`

## ADC初始化结构体

```c
/** 
  * @brief  Structure definition of ADC and regular group initialization 
  * @note   Parameters of this structure are shared within 2 scopes:
  *          - Scope entire ADC (affects regular and injected groups): DataAlign, ScanConvMode.
  *          - Scope regular group: ContinuousConvMode, NbrOfConversion, DiscontinuousConvMode, NbrOfDiscConversion, ExternalTrigConvEdge, ExternalTrigConv.
  * @note   The setting of these parameters with function HAL_ADC_Init() is conditioned to ADC state.
  *         ADC can be either disabled or enabled without conversion on going on regular group.
  */
typedef struct
{
  uint32_t DataAlign;                        /*!< Specifies ADC data alignment to right (MSB on register bit 11 and LSB on register bit 0) (default setting)
                                                  or to left (if regular group: MSB on register bit 15 and LSB on register bit 4, if injected group (MSB kept as signed value due to potential negative value after offset application): MSB on register bit 14 and LSB on register bit 3).
                                                  This parameter can be a value of @ref ADC_Data_align */
  uint32_t ScanConvMode;                     /*!< Configures the sequencer of regular and injected groups.
                                                  This parameter can be associated to parameter 'DiscontinuousConvMode' to have main sequence subdivided in successive parts.
                                                  If disabled: Conversion is performed in single mode (one channel converted, the one defined in rank 1).
                                                               Parameters 'NbrOfConversion' and 'InjectedNbrOfConversion' are discarded (equivalent to set to 1).
                                                  If enabled:  Conversions are performed in sequence mode (multiple ranks defined by 'NbrOfConversion'/'InjectedNbrOfConversion' and each channel rank).
                                                               Scan direction is upward: from rank1 to rank 'n'.
                                                  This parameter can be a value of @ref ADC_Scan_mode
                                                  Note: For regular group, this parameter should be enabled in conversion either by polling (HAL_ADC_Start with Discontinuous mode and NbrOfDiscConversion=1)
                                                        or by DMA (HAL_ADC_Start_DMA), but not by interruption (HAL_ADC_Start_IT): in scan mode, interruption is triggered only on the
                                                        the last conversion of the sequence. All previous conversions would be overwritten by the last one.
                                                        Injected group used with scan mode has not this constraint: each rank has its own result register, no data is overwritten. */
  FunctionalState ContinuousConvMode;         /*!< Specifies whether the conversion is performed in single mode (one conversion) or continuous mode for regular group,
                                                  after the selected trigger occurred (software start or external trigger).
                                                  This parameter can be set to ENABLE or DISABLE. */
  uint32_t NbrOfConversion;                  /*!< Specifies the number of ranks that will be converted within the regular group sequencer.
                                                  To use regular group sequencer and convert several ranks, parameter 'ScanConvMode' must be enabled.
                                                  This parameter must be a number between Min_Data = 1 and Max_Data = 16. */
  FunctionalState  DiscontinuousConvMode;    /*!< Specifies whether the conversions sequence of regular group is performed in Complete-sequence/Discontinuous-sequence (main sequence subdivided in successive parts).
                                                  Discontinuous mode is used only if sequencer is enabled (parameter 'ScanConvMode'). If sequencer is disabled, this parameter is discarded.
                                                  Discontinuous mode can be enabled only if continuous mode is disabled. If continuous mode is enabled, this parameter setting is discarded.
                                                  This parameter can be set to ENABLE or DISABLE. */
  uint32_t NbrOfDiscConversion;              /*!< Specifies the number of discontinuous conversions in which the  main sequence of regular group (parameter NbrOfConversion) will be subdivided.
                                                  If parameter 'DiscontinuousConvMode' is disabled, this parameter is discarded.
                                                  This parameter must be a number between Min_Data = 1 and Max_Data = 8. */
  uint32_t ExternalTrigConv;                 /*!< Selects the external event used to trigger the conversion start of regular group.
                                                  If set to ADC_SOFTWARE_START, external triggers are disabled.
                                                  If set to external trigger source, triggering is on event rising edge.
                                                  This parameter can be a value of @ref ADC_External_trigger_source_Regular */
}ADC_InitTypeDef;
```
- 数据对齐`DataAlign`

```c
/** @defgroup ADC_Data_align ADC data alignment
  * @{
  */
#define ADC_DATAALIGN_RIGHT      0x00000000U //右对齐
#define ADC_DATAALIGN_LEFT       ((uint32_t)ADC_CR2_ALIGN)//左对齐
```

- 扫描模式`ScanConvMode`

```c
/** @defgroup ADC_Scan_mode ADC scan mode
  * @{
  */
/* Note: Scan mode values are not among binary choices ENABLE/DISABLE for     */
/*       compatibility with other STM32 devices having a sequencer with       */
/*       additional options.                                                  */
#define ADC_SCAN_DISABLE         0x00000000U //禁止
#define ADC_SCAN_ENABLE          ((uint32_t)ADC_CR1_SCAN) //使能
```
- 连续模式`ContinuousConvMode`
- 转换通道数`NbrOfConversion`
- 非连续模式`DiscontinuousConvMode`
- 不连续转换数`NbrOfDiscConversion`

范围：1~8
- 外部触发`ExternalTrigConv`

```c
/** @defgroup ADC_External_trigger_source_Regular ADC External trigger selection for regular group
  * @{
  */
/*!< List of external triggers with generic trigger name, independently of    */
/* ADC target, sorted by trigger name:                                        */

/*!< External triggers of regular group for ADC1&ADC2 only */
#define ADC_EXTERNALTRIGCONV_T1_CC1         ADC1_2_EXTERNALTRIG_T1_CC1 //T1_CC1
#define ADC_EXTERNALTRIGCONV_T1_CC2         ADC1_2_EXTERNALTRIG_T1_CC2 //T1_CC2
#define ADC_EXTERNALTRIGCONV_T2_CC2         ADC1_2_EXTERNALTRIG_T2_CC2 //T2_CC2
#define ADC_EXTERNALTRIGCONV_T3_TRGO        ADC1_2_EXTERNALTRIG_T3_TRGO //T3_TRGO
#define ADC_EXTERNALTRIGCONV_T4_CC4         ADC1_2_EXTERNALTRIG_T4_CC4 //T4_CC4
#define ADC_EXTERNALTRIGCONV_EXT_IT11       ADC1_2_EXTERNALTRIG_EXT_IT11 //EXT_IT11

#if defined (STM32F103xE) || defined (STM32F103xG)
/*!< External triggers of regular group for ADC3 only */
#define ADC_EXTERNALTRIGCONV_T2_CC3         ADC3_EXTERNALTRIG_T2_CC3 //T2_CC3
#define ADC_EXTERNALTRIGCONV_T3_CC1         ADC3_EXTERNALTRIG_T3_CC1 //T3_CC1
#define ADC_EXTERNALTRIGCONV_T5_CC1         ADC3_EXTERNALTRIG_T5_CC1 //T5_CC1
#define ADC_EXTERNALTRIGCONV_T5_CC3         ADC3_EXTERNALTRIG_T5_CC3 //T5_CC3
#define ADC_EXTERNALTRIGCONV_T8_CC1         ADC3_EXTERNALTRIG_T8_CC1 //T8_CC1
#endif /* STM32F103xE || defined STM32F103xG */

/*!< External triggers of regular group for all ADC instances */
#define ADC_EXTERNALTRIGCONV_T1_CC3         ADC1_2_3_EXTERNALTRIG_T1_CC3 //T1_CC3

#if defined (STM32F101xE) || defined (STM32F103xE) || defined (STM32F103xG) || defined (STM32F105xC) || defined (STM32F107xC)
/*!< Note: TIM8_TRGO is available on ADC1 and ADC2 only in high-density and   */
/*         XL-density devices.                                                */
/*         To use it on ADC or ADC2, a remap of trigger must be done from     */
/*         EXTI line 11 to TIM8_TRGO with macro:                              */
/*           __HAL_AFIO_REMAP_ADC1_ETRGREG_ENABLE()                           */
/*           __HAL_AFIO_REMAP_ADC2_ETRGREG_ENABLE()                           */

/* Note for internal constant value management: If TIM8_TRGO is available,    */
/* its definition is set to value for ADC1&ADC2 by default and changed to     */
/* value for ADC3 by HAL ADC driver if ADC3 is selected.                      */
#define ADC_EXTERNALTRIGCONV_T8_TRGO        ADC1_2_EXTERNALTRIG_T8_TRGO  //T8_TRGO
#endif /* STM32F101xE || STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */

#define ADC_SOFTWARE_START                  ADC1_2_3_SWSTART //软件触发
```
## ADC通道结构体分析

```c
/** 
  * @brief  Structure definition of ADC channel for regular group   
  * @note   The setting of these parameters with function HAL_ADC_ConfigChannel() is conditioned to ADC state.
  *         ADC can be either disabled or enabled without conversion on going on regular group.
  */ 
typedef struct 
{
  uint32_t Channel;                /*!< Specifies the channel to configure into ADC regular group.
                                        This parameter can be a value of @ref ADC_channels
                                        Note: Depending on devices, some channels may not be available on package pins. Refer to device datasheet for channels availability.
                                        Note: On STM32F1 devices with several ADC: Only ADC1 can access internal measurement channels (VrefInt/TempSensor) 
                                        Note: On STM32F10xx8 and STM32F10xxB devices: A low-amplitude voltage glitch may be generated (on ADC input 0) on the PA0 pin, when the ADC is converting with injection trigger.
                                              It is advised to distribute the analog channels so that Channel 0 is configured as an injected channel.
                                              Refer to errata sheet of these devices for more details. */
  uint32_t Rank;                   /*!< Specifies the rank in the regular group sequencer 
                                        This parameter can be a value of @ref ADC_regular_rank
                                        Note: In case of need to disable a channel or change order of conversion sequencer, rank containing a previous channel setting can be overwritten by the new channel setting (or parameter number of conversions can be adjusted) */
  uint32_t SamplingTime;           /*!< Sampling time value to be set for the selected channel.
                                        Unit: ADC clock cycles
                                        Conversion time is the addition of sampling time and processing time (12.5 ADC clock cycles at ADC resolution 12 bits).
                                        This parameter can be a value of @ref ADC_sampling_times
                                        Caution: This parameter updates the parameter property of the channel, that can be used into regular and/or injected groups.
                                                 If this same channel has been previously configured in the other group (regular/injected), it will be updated to last setting.
                                        Note: In case of usage of internal measurement channels (VrefInt/TempSensor),
                                              sampling time constraints must be respected (sampling time can be adjusted in function of ADC clock frequency and sampling time setting)
                                              Refer to device datasheet for timings values, parameters TS_vrefint, TS_temp (values rough order: 5us to 17.1us min). */
}ADC_ChannelConfTypeDef;
```
- 通道`Channel`

```c
/** @defgroup ADC_channels ADC channels
  * @{
  */
/* Note: Depending on devices, some channels may not be available on package  */
/*       pins. Refer to device datasheet for channels availability.           */
#define ADC_CHANNEL_0                       0x00000000U
#define ADC_CHANNEL_1           ((uint32_t)(ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_2           ((uint32_t)(ADC_SQR3_SQ1_1))
#define ADC_CHANNEL_3           ((uint32_t)(ADC_SQR3_SQ1_1 | ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_4           ((uint32_t)(ADC_SQR3_SQ1_2))
#define ADC_CHANNEL_5           ((uint32_t)(ADC_SQR3_SQ1_2| ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_6           ((uint32_t)(ADC_SQR3_SQ1_2 | ADC_SQR3_SQ1_1))
#define ADC_CHANNEL_7           ((uint32_t)(ADC_SQR3_SQ1_2 | ADC_SQR3_SQ1_1 | ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_8           ((uint32_t)(ADC_SQR3_SQ1_3))
#define ADC_CHANNEL_9           ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_10          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_1 ))
#define ADC_CHANNEL_11          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_1 | ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_12          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_2 ))
#define ADC_CHANNEL_13          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_2| ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_14          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_2 | ADC_SQR3_SQ1_1))
#define ADC_CHANNEL_15          ((uint32_t)(ADC_SQR3_SQ1_3 | ADC_SQR3_SQ1_2 | ADC_SQR3_SQ1_1 | ADC_SQR3_SQ1_0))
#define ADC_CHANNEL_16          ((uint32_t)(ADC_SQR3_SQ1_4 ))
#define ADC_CHANNEL_17          ((uint32_t)(ADC_SQR3_SQ1_4 | ADC_SQR3_SQ1_0))

#define ADC_CHANNEL_TEMPSENSOR  ADC_CHANNEL_16  /* ADC internal channel (no connection on device pin) */
#define ADC_CHANNEL_VREFINT     ADC_CHANNEL_17  /* ADC internal channel (no connection on device pin) */
```

- 序列`Rank`

```c
/** @defgroup ADC_regular_rank ADC rank into regular group
  * @{
  */
#define ADC_REGULAR_RANK_1                 0x00000001U
#define ADC_REGULAR_RANK_2                 0x00000002U
#define ADC_REGULAR_RANK_3                 0x00000003U
#define ADC_REGULAR_RANK_4                 0x00000004U
#define ADC_REGULAR_RANK_5                 0x00000005U
#define ADC_REGULAR_RANK_6                 0x00000006U
#define ADC_REGULAR_RANK_7                 0x00000007U
#define ADC_REGULAR_RANK_8                 0x00000008U
#define ADC_REGULAR_RANK_9                 0x00000009U
#define ADC_REGULAR_RANK_10                0x0000000AU
#define ADC_REGULAR_RANK_11                0x0000000BU
#define ADC_REGULAR_RANK_12                0x0000000CU
#define ADC_REGULAR_RANK_13                0x0000000DU
#define ADC_REGULAR_RANK_14                0x0000000EU
#define ADC_REGULAR_RANK_15                0x0000000FU
#define ADC_REGULAR_RANK_16                0x00000010U
```

- 采样时间`SamplingTime`

```c
/** @defgroup ADC_sampling_times ADC sampling times
  * @{
  */
#define ADC_SAMPLETIME_1CYCLE_5                   0x00000000U        /*!< Sampling time 1.5 ADC clock cycle */
#define ADC_SAMPLETIME_7CYCLES_5      ((uint32_t)(ADC_SMPR2_SMP0_0)) /*!< Sampling time 7.5 ADC clock cycles */
#define ADC_SAMPLETIME_13CYCLES_5     ((uint32_t)(ADC_SMPR2_SMP0_1)) /*!< Sampling time 13.5 ADC clock cycles */
#define ADC_SAMPLETIME_28CYCLES_5     ((uint32_t)(ADC_SMPR2_SMP0_1 | ADC_SMPR2_SMP0_0)) /*!< Sampling time 28.5 ADC clock cycles */
#define ADC_SAMPLETIME_41CYCLES_5     ((uint32_t)(ADC_SMPR2_SMP0_2)) /*!< Sampling time 41.5 ADC clock cycles */
#define ADC_SAMPLETIME_55CYCLES_5     ((uint32_t)(ADC_SMPR2_SMP0_2| ADC_SMPR2_SMP0_0)) /*!< Sampling time 55.5 ADC clock cycles */
#define ADC_SAMPLETIME_71CYCLES_5     ((uint32_t)(ADC_SMPR2_SMP0_2 | ADC_SMPR2_SMP0_1)) /*!< Sampling time 71.5 ADC clock cycles */
#define ADC_SAMPLETIME_239CYCLES_5    ((uint32_t)(ADC_SMPR2_SMP0_2 | ADC_SMPR2_SMP0_1 | ADC_SMPR2_SMP0_0)) /*!< Sampling time 239.5 ADC clock cycles */
```

## RCC扩展时钟结构分析

```c
/**
  * @brief  RCC extended clocks structure definition
  */
typedef struct
{
  uint32_t PeriphClockSelection;      /*!< The Extended Clock to be configured.
                                       This parameter can be a value of @ref RCCEx_Periph_Clock_Selection */

  uint32_t RTCClockSelection;         /*!< specifies the RTC clock source.
                                       This parameter can be a value of @ref RCC_RTC_Clock_Source */

  uint32_t AdcClockSelection;         /*!< ADC clock source
                                       This parameter can be a value of @ref RCCEx_ADC_Prescaler */

#if defined(STM32F103xE) || defined(STM32F103xG) || defined(STM32F105xC)\
 || defined(STM32F107xC)
  uint32_t I2s2ClockSelection;         /*!< I2S2 clock source
                                       This parameter can be a value of @ref RCCEx_I2S2_Clock_Source */

  uint32_t I2s3ClockSelection;         /*!< I2S3 clock source
                                       This parameter can be a value of @ref RCCEx_I2S3_Clock_Source */

#if defined(STM32F105xC) || defined(STM32F107xC)
  RCC_PLLI2SInitTypeDef PLLI2S;  /*!< PLL I2S structure parameters
                                      This parameter will be used only when PLLI2S is selected as Clock Source I2S2 or I2S3 */

#endif /* STM32F105xC || STM32F107xC */
#endif /* STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */

#if defined(STM32F102x6) || defined(STM32F102xB) || defined(STM32F103x6)\
 || defined(STM32F103xB) || defined(STM32F103xE) || defined(STM32F103xG)\
 || defined(STM32F105xC) || defined(STM32F107xC)
  uint32_t UsbClockSelection;         /*!< USB clock source
                                       This parameter can be a value of @ref RCCEx_USB_Prescaler */

#endif /* STM32F102x6 || STM32F102xB || STM32F103x6 || STM32F103xB || STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */
} RCC_PeriphCLKInitTypeDef;
```
- 要配置的时钟`PeriphClockSelection`

```c
/** @defgroup RCCEx_Periph_Clock_Selection Periph Clock Selection
  * @{
  */
#define RCC_PERIPHCLK_RTC           0x00000001U //RTC时钟
#define RCC_PERIPHCLK_ADC           0x00000002U // ADC时钟
#if defined(STM32F103xE) || defined(STM32F103xG) || defined(STM32F105xC)\
 || defined(STM32F107xC)
#define RCC_PERIPHCLK_I2S2          0x00000004U //I2S2时钟
#define RCC_PERIPHCLK_I2S3          0x00000008U //I2S3时钟
#endif /* STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */
#if defined(STM32F102x6) || defined(STM32F102xB) || defined(STM32F103x6)\
 || defined(STM32F103xB) || defined(STM32F103xE) || defined(STM32F103xG)\
 || defined(STM32F105xC) || defined(STM32F107xC)
#define RCC_PERIPHCLK_USB          0x00000010U // USB时钟
#endif /* STM32F102x6 || STM32F102xB || STM32F103x6 || STM32F103xB || STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */
```
- RTC时钟源`RTCClockSelection`

```c
/** @defgroup RCC_RTC_Clock_Source RTC Clock Source
  * @{
  */
#define RCC_RTCCLKSOURCE_NO_CLK          0x00000000U                 /*!< No clock *///无
#define RCC_RTCCLKSOURCE_LSE             RCC_BDCR_RTCSEL_LSE                  /*!< LSE oscillator clock used as RTC clock *///LSE
#define RCC_RTCCLKSOURCE_LSI             RCC_BDCR_RTCSEL_LSI                  /*!< LSI oscillator clock used as RTC clock *///LSI
#define RCC_RTCCLKSOURCE_HSE_DIV128      RCC_BDCR_RTCSEL_HSE                    /*!< HSE oscillator clock divided by 128 used as RTC clock *///HSE
```
- ADC时钟分频器`AdcClockSelection`

```c
/** @defgroup RCCEx_ADC_Prescaler ADC Prescaler
  * @{
  */
#define RCC_ADCPCLK2_DIV2              RCC_CFGR_ADCPRE_DIV2 //2分频
#define RCC_ADCPCLK2_DIV4              RCC_CFGR_ADCPRE_DIV4 //4分频
#define RCC_ADCPCLK2_DIV6              RCC_CFGR_ADCPRE_DIV6 //6分频
#define RCC_ADCPCLK2_DIV8              RCC_CFGR_ADCPRE_DIV8 //8分频
```

- `I2s2ClockSelection`

```c
/** @defgroup RCCEx_I2S2_Clock_Source I2S2 Clock Source
  * @{
  */
#define RCC_I2S2CLKSOURCE_SYSCLK              0x00000000U //系统时钟
#if defined(STM32F105xC) || defined(STM32F107xC)
#define RCC_I2S2CLKSOURCE_PLLI2S_VCO          RCC_CFGR2_I2S2SRC //I2S2输入时钟源
#endif /* STM32F105xC || STM32F107xC */
```
- `I2s3ClockSelection`

```c
/** @defgroup RCCEx_I2S3_Clock_Source I2S3 Clock Source
  * @{
  */
#define RCC_I2S3CLKSOURCE_SYSCLK              0x00000000U //系统时钟
#if defined(STM32F105xC) || defined(STM32F107xC)
#define RCC_I2S3CLKSOURCE_PLLI2S_VCO          RCC_CFGR2_I2S3SRC //I2S3输入时钟源
#endif /* STM32F105xC || STM32F107xC */
```
- PLL I2S结构体`PLLI2S`

注:仅在`STM32F105xC`和`STM32F107xC`系列可用
- USB时钟分频器`UsbClockSelection`

```c
#if defined(STM32F102x6) || defined(STM32F102xB) || defined(STM32F103x6)\
 || defined(STM32F103xB) || defined(STM32F103xE) || defined(STM32F103xG)

/** @defgroup RCCEx_USB_Prescaler USB Prescaler
  * @{
  */
#define RCC_USBCLKSOURCE_PLL              RCC_CFGR_USBPRE //PLL时钟
#define RCC_USBCLKSOURCE_PLL_DIV1_5       0x00000000U    //1.5分频

/**
  * @}
  */

#endif /* STM32F102x6 || STM32F102xB || STM32F103x6 || STM32F103xB || STM32F103xE || STM32F103xG */


#if defined(STM32F105xC) || defined(STM32F107xC)
/** @defgroup RCCEx_USB_Prescaler USB Prescaler
  * @{
  */
#define RCC_USBCLKSOURCE_PLL_DIV2              RCC_CFGR_OTGFSPRE //2分频
#define RCC_USBCLKSOURCE_PLL_DIV3              0x00000000U //3分频

/**
  * @}
  */
  #endif /* STM32F105xC || STM32F107xC */
```
## PLL I2S结构体分析

```c
#if defined(STM32F105xC) || defined(STM32F107xC)
/**
  * @brief  RCC PLLI2S configuration structure definition
  */
typedef struct
{
  uint32_t PLLI2SMUL;         /*!< PLLI2SMUL: Multiplication factor for PLLI2S VCO input clock
                              This parameter must be a value of @ref RCCEx_PLLI2S_Multiplication_Factor*/

#if defined(STM32F105xC) || defined(STM32F107xC)
  uint32_t HSEPrediv2Value;       /*!<  The Prediv2 factor value.
                                       This parameter can be a value of @ref RCCEx_Prediv2_Factor */

#endif /* STM32F105xC || STM32F107xC */
} RCC_PLLI2SInitTypeDef;
#endif /* STM32F105xC || STM32F107xC */
```
- PLLI2S倍频器`PLLI2SMUL`

```c
/** @defgroup RCCEx_PLLI2S_Multiplication_Factor PLLI2S Multiplication Factor
  * @{
  */

#define RCC_PLLI2S_MUL8                   RCC_CFGR2_PLL3MUL8   /*!< PLLI2S input clock * 8 *///8倍频
#define RCC_PLLI2S_MUL9                   RCC_CFGR2_PLL3MUL9   /*!< PLLI2S input clock * 9 *///9倍频
#define RCC_PLLI2S_MUL10                  RCC_CFGR2_PLL3MUL10  /*!< PLLI2S input clock * 10 *///10倍频
#define RCC_PLLI2S_MUL11                  RCC_CFGR2_PLL3MUL11  /*!< PLLI2S input clock * 11 *///11倍频
#define RCC_PLLI2S_MUL12                  RCC_CFGR2_PLL3MUL12  /*!< PLLI2S input clock * 12 *///12倍频
#define RCC_PLLI2S_MUL13                  RCC_CFGR2_PLL3MUL13  /*!< PLLI2S input clock * 13 *///13倍频
#define RCC_PLLI2S_MUL14                  RCC_CFGR2_PLL3MUL14  /*!< PLLI2S input clock * 14 *///14倍频
#define RCC_PLLI2S_MUL16                  RCC_CFGR2_PLL3MUL16  /*!< PLLI2S input clock * 16 *///16倍频
#define RCC_PLLI2S_MUL20                  RCC_CFGR2_PLL3MUL20  /*!< PLLI2S input clock * 20 *///20倍频

```

- HSE预分频器2`HSEPrediv2Value`

注：和PLL2结构体`RCC_PLL2InitTypeDef`的`HSEPrediv2Value`是同一个参数，只有在PLL2未启用的情况下，才能写入，如果已启用，必须和PLL2的保持一致

## 常用库函数解析

- 获取标志位`__HAL_ADC_GET_FLAG(__HANDLE__, __FLAG__)`
- 清除标志位`__HAL_ADC_CLEAR_FLAG(__HANDLE__, __FLAG__)`

可选标志位：
**`ADC_FLAG_STRT`**——ADC规则转换开始
**`ADC_FLAG_JSTRT`**——ADC注入转换开始
**`ADC_FLAG_EOC`**——ADC规则转换完成
**`ADC_FLAG_JEOC`**——ADC注入转换完成
**`ADC_FLAG_AWD`**——ADC看门狗事件


- 阻塞模式启动ADC`HAL_StatusTypeDef       HAL_ADC_Start(ADC_HandleTypeDef* hadc)`

程序会等待启用完成，才继续下一步
- 停止ADC`HAL_StatusTypeDef       HAL_ADC_Stop(ADC_HandleTypeDef* hadc)`

程序会等待停止完成，才继续下一步
- 轮询转换完成`HAL_StatusTypeDef       HAL_ADC_PollForConversion(ADC_HandleTypeDef* hadc, uint32_t Timeout)`

程序会等待转换完成，才继续下一步

- 轮询转换事件`HAL_StatusTypeDef       HAL_ADC_PollForEvent(ADC_HandleTypeDef* hadc, uint32_t EventType, uint32_t Timeout)`

程序会等待转换事件发生，才继续下一步

- 中断模式启动ADC`HAL_StatusTypeDef       HAL_ADC_Start_IT(ADC_HandleTypeDef* hadc)`

转换完成会调用转换完成回调函数,下次转换一定要等回调函数调用后再开启，否则会无法开启

- 停止ADC失效ADC中断`HAL_StatusTypeDef       HAL_ADC_Stop_IT(ADC_HandleTypeDef* hadc)`

- DMA启动`HAL_StatusTypeDef       HAL_ADC_Start_DMA(ADC_HandleTypeDef* hadc, uint32_t* pData, uint32_t Length)`

转换完成会调用转换完成回调函数

- DMA停止`HAL_StatusTypeDef       HAL_ADC_Stop_DMA(ADC_HandleTypeDef* hadc)`

- 获取ADC当前值`uint32_t          HAL_ADC_GetValue(ADC_HandleTypeDef* hadc)`

- 校准ADC`HAL_StatusTypeDef       HAL_ADCEx_Calibration_Start(ADC_HandleTypeDef* hadc)`
 
校准ADC一定要在ADC开始之前或者停止之后

# 代码移植
## 添加ADC校准
- 在`HAL_ADC_MspInit`函数中添加ADC校准

```c
/* USER CODE BEGIN ADC1_MspInit 1 */
HAL_ADCEx_Calibration_Start(adcHandle);
/* USER CODE END ADC1_MspInit 1 */
```

## 开始ADC转换
- 在`main.c`文件添加变量

```c
__IO uint16_t ADC_ConvertedValue[3] = {0};
float VREF,VSENSE;
float temp;
float adcValue;
```
- 在主函数初始化添加DMA开始转换

```c
HAL_ADC_Start_DMA(&hadc1,(uint32_t*)&ADC_ConvertedValue,3);
```
- 主循环添加数据发送

```c
Delay(0xffffee);      // 延时
printf( "\r\n The IC current tem= %.2fC\r\n", temp);
printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
```
- 添加回调函数

```c
/**
  * @brief  Conversion complete callback in non blocking mode
  * @param  hadc: ADC handle
  * @retval None
  */
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc)
{
    if(hadc->Instance==ADC1)
    {
        VREF=1.2f*4096/ADC_ConvertedValue[2];
        VSENSE=VREF*ADC_ConvertedValue[1]/4096;
        temp=(1.43f-VSENSE)/4.3+25;
        adcValue=VREF*ADC_ConvertedValue[0]/4096;
    }
}
```

## 增加重定向c库函数printf到串口
- Keil设置**`Options`**->**`Target`**->**`Use Micro LIB`**
- 代码

```c
int fputc(int ch, FILE *f)
{
    uint8_t temp[1]={ch};
    HAL_UART_Transmit(&huart1,temp,1,2);      
    return (ch);
}
```
# 下载调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 会看到串口电源电压，温度，PC1电压值，用万用表测量比对
- 和SPL库结果一致