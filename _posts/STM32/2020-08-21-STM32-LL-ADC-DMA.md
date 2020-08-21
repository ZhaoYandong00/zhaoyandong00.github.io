---
title: STM32底层LL库ADC使用DMA多通道采集
categories: STM32 CUBE LL ADC DMA
tags: STM32 CUBE LL ADC DMA
description: LL库ADC使用DMA多通道采集
---
# 配置ADC
- 和HAL库配置一样
- 在高级设置里为**ADC1**选择`LL`库
- 生成代码

# ADC函数
## ADC结构体分析

```c
/**
  * @brief  Structure definition of some features of ADC instance.
  * @note   These parameters have an impact on ADC scope: ADC instance.
  *         Affects both group regular and group injected (availability
  *         of ADC group injected depends on STM32 families).
  *         Refer to corresponding unitary functions into
  *         @ref ADC_LL_EF_Configuration_ADC_Instance .
  * @note   The setting of these parameters by function @ref LL_ADC_Init()
  *         is conditioned to ADC state:
  *         ADC instance must be disabled.
  *         This condition is applied to all ADC features, for efficiency
  *         and compatibility over all STM32 families. However, the different
  *         features can be set under different ADC state conditions
  *         (setting possible with ADC enabled without conversion on going,
  *         ADC enabled with conversion on going, ...)
  *         Each feature can be updated afterwards with a unitary function
  *         and potentially with ADC in a different state than disabled,
  *         refer to description of each function for setting
  *         conditioned to ADC state.
  */
typedef struct
{
  uint32_t DataAlignment;               /*!< Set ADC conversion data alignment.
                                             This parameter can be a value of @ref ADC_LL_EC_DATA_ALIGN
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_SetDataAlignment(). */

  uint32_t SequencersScanMode;          /*!< Set ADC scan selection.
                                             This parameter can be a value of @ref ADC_LL_EC_SCAN_SELECTION
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_SetSequencersScanMode(). */

} LL_ADC_InitTypeDef;
```

- 数据对齐`DataAlignment`

```c
/** @defgroup ADC_LL_EC_DATA_ALIGN  ADC instance - Data alignment
  * @{
  */
#define LL_ADC_DATA_ALIGN_RIGHT            0x00000000U            /*!< ADC conversion data alignment: right aligned (alignment on data register LSB bit 0)*/
#define LL_ADC_DATA_ALIGN_LEFT             (ADC_CR2_ALIGN)        /*!< ADC conversion data alignment: left aligned (aligment on data register MSB bit 15)*/
```
- 扫描模式`SequencersScanMode`

```c
/** @defgroup ADC_LL_EC_SCAN_SELECTION ADC instance - Scan selection
  * @{
  */
#define LL_ADC_SEQ_SCAN_DISABLE            0x00000000U    /*!< ADC conversion is performed in unitary conversion mode (one channel converted, that defined in rank 1). Configuration of both groups regular and injected sequencers (sequence length, ...) is discarded: equivalent to length of 1 rank.*/
#define LL_ADC_SEQ_SCAN_ENABLE             (ADC_CR1_SCAN) /*!< ADC conversions are performed in sequence conversions mode, according to configuration of both groups regular and injected sequencers (sequence length, ...). */
```

## 模式结构体分析
```c
/**
  * @brief  Structure definition of some features of ADC common parameters
  *         and multimode
  *         (all ADC instances belonging to the same ADC common instance).
  * @note   The setting of these parameters by function @ref LL_ADC_CommonInit()
  *         is conditioned to ADC instances state (all ADC instances
  *         sharing the same ADC common instance):
  *         All ADC instances sharing the same ADC common instance must be
  *         disabled.
  */
typedef struct
{
  uint32_t Multimode;                   /*!< Set ADC multimode configuration to operate in independent mode or multimode (for devices with several ADC instances).
                                             This parameter can be a value of @ref ADC_LL_EC_MULTI_MODE
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_SetMultimode(). */
} LL_ADC_CommonInitTypeDef;
```
- 工作模式`Multimode`

```c
#if !defined(ADC_MULTIMODE_SUPPORT)
/** @defgroup ADC_LL_EC_MULTI_MODE Multimode - Mode
  * @{
  */
#define LL_ADC_MULTI_INDEPENDENT              0x00000000U                                                         /*!< ADC dual mode disabled (ADC independent mode) */
/**
  * @}
  */
#endif
#if defined(ADC_MULTIMODE_SUPPORT)
/** @defgroup ADC_LL_EC_MULTI_MODE  Multimode - Mode
  * @{
  */
#define LL_ADC_MULTI_INDEPENDENT              0x00000000U                                                                     /*!< ADC dual mode disabled (ADC independent mode) */
#define LL_ADC_MULTI_DUAL_REG_SIMULT          (                    ADC_CR1_DUALMOD_2 | ADC_CR1_DUALMOD_1                    ) /*!< ADC dual mode enabled: group regular simultaneous */
#define LL_ADC_MULTI_DUAL_REG_INTERL_FAST     (                    ADC_CR1_DUALMOD_2 | ADC_CR1_DUALMOD_1 | ADC_CR1_DUALMOD_0) /*!< ADC dual mode enabled: Combined group regular interleaved fast (delay between ADC sampling phases: 7 ADC clock cycles) (equivalent to multimode sampling delay set to "LL_ADC_MULTI_TWOSMP_DELAY_7CYCLES" on other STM32 devices)) */
#define LL_ADC_MULTI_DUAL_REG_INTERL_SLOW     (ADC_CR1_DUALMOD_3                                                            ) /*!< ADC dual mode enabled: Combined group regular interleaved slow (delay between ADC sampling phases: 14 ADC clock cycles) (equivalent to multimode sampling delay set to "LL_ADC_MULTI_TWOSMP_DELAY_14CYCLES" on other STM32 devices)) */
#define LL_ADC_MULTI_DUAL_INJ_SIMULT          (                    ADC_CR1_DUALMOD_2                     | ADC_CR1_DUALMOD_0) /*!< ADC dual mode enabled: group injected simultaneous slow (delay between ADC sampling phases: 14 ADC clock cycles) (equivalent to multimode sampling delay set to "LL_ADC_MULTI_TWOSMP_DELAY_14CYCLES" on other STM32 devices)) */
#define LL_ADC_MULTI_DUAL_INJ_ALTERN          (ADC_CR1_DUALMOD_3                                         | ADC_CR1_DUALMOD_0) /*!< ADC dual mode enabled: group injected alternate trigger. Works only with external triggers (not internal SW start) */
#define LL_ADC_MULTI_DUAL_REG_SIM_INJ_SIM     (                                                            ADC_CR1_DUALMOD_0) /*!< ADC dual mode enabled: Combined group regular simultaneous + group injected simultaneous */
#define LL_ADC_MULTI_DUAL_REG_SIM_INJ_ALT     (                                        ADC_CR1_DUALMOD_1                    ) /*!< ADC dual mode enabled: Combined group regular simultaneous + group injected alternate trigger */
#define LL_ADC_MULTI_DUAL_REG_INTFAST_INJ_SIM (                                        ADC_CR1_DUALMOD_1 | ADC_CR1_DUALMOD_0) /*!< ADC dual mode enabled: Combined group regular interleaved fast (delay between ADC sampling phases: 7 ADC clock cycles) + group injected simultaneous */
#define LL_ADC_MULTI_DUAL_REG_INTSLOW_INJ_SIM (                    ADC_CR1_DUALMOD_2                                        ) /*!< ADC dual mode enabled: Combined group regular interleaved slow (delay between ADC sampling phases: 14 ADC clock cycles) + group injected simultaneous */
```
## 规则组结构体分析

```c
/**
  * @brief  Structure definition of some features of ADC group regular.
  * @note   These parameters have an impact on ADC scope: ADC group regular.
  *         Refer to corresponding unitary functions into
  *         @ref ADC_LL_EF_Configuration_ADC_Group_Regular
  *         (functions with prefix "REG").
  * @note   The setting of these parameters by function @ref LL_ADC_REG_Init()
  *         is conditioned to ADC state:
  *         ADC instance must be disabled.
  *         This condition is applied to all ADC features, for efficiency
  *         and compatibility over all STM32 families. However, the different
  *         features can be set under different ADC state conditions
  *         (setting possible with ADC enabled without conversion on going,
  *         ADC enabled with conversion on going, ...)
  *         Each feature can be updated afterwards with a unitary function
  *         and potentially with ADC in a different state than disabled,
  *         refer to description of each function for setting
  *         conditioned to ADC state.
  */
typedef struct
{
  uint32_t TriggerSource;               /*!< Set ADC group regular conversion trigger source: internal (SW start) or from external IP (timer event, external interrupt line).
                                             This parameter can be a value of @ref ADC_LL_EC_REG_TRIGGER_SOURCE
                                             @note On this STM32 serie, external trigger is set with trigger polarity: rising edge
                                                   (only trigger polarity available on this STM32 serie).
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_REG_SetTriggerSource(). */

  uint32_t SequencerLength;             /*!< Set ADC group regular sequencer length.
                                             This parameter can be a value of @ref ADC_LL_EC_REG_SEQ_SCAN_LENGTH
                                             @note This parameter is discarded if scan mode is disabled (refer to parameter 'ADC_SequencersScanMode').
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_REG_SetSequencerLength(). */

  uint32_t SequencerDiscont;            /*!< Set ADC group regular sequencer discontinuous mode: sequence subdivided and scan conversions interrupted every selected number of ranks.
                                             This parameter can be a value of @ref ADC_LL_EC_REG_SEQ_DISCONT_MODE
                                             @note This parameter has an effect only if group regular sequencer is enabled
                                                   (scan length of 2 ranks or more).
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_REG_SetSequencerDiscont(). */

  uint32_t ContinuousMode;              /*!< Set ADC continuous conversion mode on ADC group regular, whether ADC conversions are performed in single mode (one conversion per trigger) or in continuous mode (after the first trigger, following conversions launched successively automatically).
                                             This parameter can be a value of @ref ADC_LL_EC_REG_CONTINUOUS_MODE
                                             Note: It is not possible to enable both ADC group regular continuous mode and discontinuous mode.
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_REG_SetContinuousMode(). */

  uint32_t DMATransfer;                 /*!< Set ADC group regular conversion data transfer: no transfer or transfer by DMA, and DMA requests mode.
                                             This parameter can be a value of @ref ADC_LL_EC_REG_DMA_TRANSFER
                                             
                                             This feature can be modified afterwards using unitary function @ref LL_ADC_REG_SetDMATransfer(). */

} LL_ADC_REG_InitTypeDef;
```

- 触发源`TriggerSource`

```c
/** @defgroup ADC_LL_EC_REG_TRIGGER_SOURCE  ADC group regular - Trigger source
  * @{
  */
/* ADC group regular external triggers for ADC instances: ADC1, ADC2, ADC3 (for ADC instances ADCx available on the selected device) */
#define LL_ADC_REG_TRIG_SOFTWARE           (ADC_CR2_EXTSEL_2 | ADC_CR2_EXTSEL_1 | ADC_CR2_EXTSEL_0) /*!< ADC group regular conversion trigger internal: SW start. */
#define LL_ADC_REG_TRIG_EXT_TIM1_CH3       (ADC_CR2_EXTSEL_1)                                       /*!< ADC group regular conversion trigger from external IP: TIM1 channel 3 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
/* ADC group regular external triggers for ADC instances: ADC1, ADC2 (for ADC instances ADCx available on the selected device) */
#define LL_ADC_REG_TRIG_EXT_TIM1_CH1       0x00000000U                                              /*!< ADC group regular conversion trigger from external IP: TIM1 channel 1 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM1_CH2       (ADC_CR2_EXTSEL_0)                                       /*!< ADC group regular conversion trigger from external IP: TIM1 channel 2 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM2_CH2       (ADC_CR2_EXTSEL_1 | ADC_CR2_EXTSEL_0)                    /*!< ADC group regular conversion trigger from external IP: TIM2 channel 2 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM3_TRGO      (ADC_CR2_EXTSEL_2)                                       /*!< ADC group regular conversion trigger from external IP: TIM3 TRGO. Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM4_CH4       (ADC_CR2_EXTSEL_2 | ADC_CR2_EXTSEL_0)                    /*!< ADC group regular conversion trigger from external IP: TIM4 channel 4 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_EXTI_LINE11    (ADC_CR2_EXTSEL_2 | ADC_CR2_EXTSEL_1)                    /*!< ADC group regular conversion trigger from external IP: external interrupt line 11. Trigger edge set to rising edge (default setting). */
#if defined (STM32F101xE) || defined (STM32F103xE) || defined (STM32F103xG) || defined (STM32F105xC) || defined (STM32F107xC)
/* Note: TIM8_TRGO is available on ADC1 and ADC2 only in high-density and     */
/*       XL-density devices.                                                  */
/* Note: To use TIM8_TRGO on ADC1 or ADC2, a remap of trigger must be done    */
/*       A remap of trigger must be done at top level (refer to               */
/*       AFIO peripheral).                                                    */
#define LL_ADC_REG_TRIG_EXT_TIM8_TRGO      (LL_ADC_REG_TRIG_EXT_EXTI_LINE11)                        /*!< ADC group regular conversion trigger from external IP: TIM8 TRGO. Trigger edge set to rising edge (default setting). Available only on high-density and XL-density devices. A remap of trigger must be done at top level (refer to AFIO peripheral).*/
#endif /* STM32F101xE || STM32F103xE || STM32F103xG || STM32F105xC || STM32F107xC */
#if defined (STM32F103xE) || defined (STM32F103xG)
/* ADC group regular external triggers for ADC instances: ADC3 (for ADC instances ADCx available on the selected device) */
#define LL_ADC_REG_TRIG_EXT_TIM3_CH1       (LL_ADC_REG_TRIG_EXT_TIM1_CH1)                           /*!< ADC group regular conversion trigger from external IP: TIM3 channel 1 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM2_CH3       (LL_ADC_REG_TRIG_EXT_TIM1_CH2)                           /*!< ADC group regular conversion trigger from external IP: TIM2 channel 3 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM8_CH1       (LL_ADC_REG_TRIG_EXT_TIM2_CH2)                           /*!< ADC group regular conversion trigger from external IP: TIM8 channel 1 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM8_TRGO_ADC3 (LL_ADC_REG_TRIG_EXT_TIM3_TRGO)                          /*!< ADC group regular conversion trigger from external IP: TIM8 TRGO. Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM5_CH1       (LL_ADC_REG_TRIG_EXT_TIM4_CH4)                           /*!< ADC group regular conversion trigger from external IP: TIM5 channel 1 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#define LL_ADC_REG_TRIG_EXT_TIM5_CH3       (LL_ADC_REG_TRIG_EXT_EXTI_LINE11)                        /*!< ADC group regular conversion trigger from external IP: TIM5 channel 3 event (capture compare: input capture or output capture). Trigger edge set to rising edge (default setting). */
#endif
```
- 通道数`SequencerLength`

```c
/** @defgroup ADC_LL_EC_REG_SEQ_SCAN_LENGTH  ADC group regular - Sequencer scan length
  * @{
  */
#define LL_ADC_REG_SEQ_SCAN_DISABLE        0x00000000U                                                 /*!< ADC group regular sequencer disable (equivalent to sequencer of 1 rank: ADC conversion on only 1 channel) */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_2RANKS  (                                             ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 2 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_3RANKS  (                              ADC_SQR1_L_1               ) /*!< ADC group regular sequencer enable with 3 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_4RANKS  (                              ADC_SQR1_L_1 | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 4 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_5RANKS  (               ADC_SQR1_L_2                              ) /*!< ADC group regular sequencer enable with 5 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_6RANKS  (               ADC_SQR1_L_2                | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 6 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_7RANKS  (               ADC_SQR1_L_2 | ADC_SQR1_L_1               ) /*!< ADC group regular sequencer enable with 7 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_8RANKS  (               ADC_SQR1_L_2 | ADC_SQR1_L_1 | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 8 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_9RANKS  (ADC_SQR1_L_3                                             ) /*!< ADC group regular sequencer enable with 9 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_10RANKS (ADC_SQR1_L_3                               | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 10 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_11RANKS (ADC_SQR1_L_3                | ADC_SQR1_L_1               ) /*!< ADC group regular sequencer enable with 11 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_12RANKS (ADC_SQR1_L_3                | ADC_SQR1_L_1 | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 12 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_13RANKS (ADC_SQR1_L_3 | ADC_SQR1_L_2                              ) /*!< ADC group regular sequencer enable with 13 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_14RANKS (ADC_SQR1_L_3 | ADC_SQR1_L_2                | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 14 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_15RANKS (ADC_SQR1_L_3 | ADC_SQR1_L_2 | ADC_SQR1_L_1               ) /*!< ADC group regular sequencer enable with 15 ranks in the sequence */
#define LL_ADC_REG_SEQ_SCAN_ENABLE_16RANKS (ADC_SQR1_L_3 | ADC_SQR1_L_2 | ADC_SQR1_L_1 | ADC_SQR1_L_0) /*!< ADC group regular sequencer enable with 16 ranks in the sequence */
```
- 不连续转换数`SequencerDiscont`

```c
/** @defgroup ADC_LL_EC_REG_SEQ_DISCONT_MODE  ADC group regular - Sequencer discontinuous mode
  * @{
  */
#define LL_ADC_REG_SEQ_DISCONT_DISABLE     0x00000000U                                                                  /*!< ADC group regular sequencer discontinuous mode disable */
#define LL_ADC_REG_SEQ_DISCONT_1RANK       (                                                            ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every rank */
#define LL_ADC_REG_SEQ_DISCONT_2RANKS      (                                        ADC_CR1_DISCNUM_0 | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enabled with sequence interruption every 2 ranks */
#define LL_ADC_REG_SEQ_DISCONT_3RANKS      (                    ADC_CR1_DISCNUM_1                     | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 3 ranks */
#define LL_ADC_REG_SEQ_DISCONT_4RANKS      (                    ADC_CR1_DISCNUM_1 | ADC_CR1_DISCNUM_0 | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 4 ranks */
#define LL_ADC_REG_SEQ_DISCONT_5RANKS      (ADC_CR1_DISCNUM_2                                         | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 5 ranks */
#define LL_ADC_REG_SEQ_DISCONT_6RANKS      (ADC_CR1_DISCNUM_2                     | ADC_CR1_DISCNUM_0 | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 6 ranks */
#define LL_ADC_REG_SEQ_DISCONT_7RANKS      (ADC_CR1_DISCNUM_2 | ADC_CR1_DISCNUM_1                     | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 7 ranks */
#define LL_ADC_REG_SEQ_DISCONT_8RANKS      (ADC_CR1_DISCNUM_2 | ADC_CR1_DISCNUM_1 | ADC_CR1_DISCNUM_0 | ADC_CR1_DISCEN) /*!< ADC group regular sequencer discontinuous mode enable with sequence interruption every 8 ranks */
```
- 转换模式`ContinuousMode`

```c
/** @defgroup ADC_LL_EC_REG_CONTINUOUS_MODE  ADC group regular - Continuous mode
* @{
*/
#define LL_ADC_REG_CONV_SINGLE             0x00000000U             /*!< ADC conversions are performed in single mode: one conversion per trigger */
#define LL_ADC_REG_CONV_CONTINUOUS         (ADC_CR2_CONT)          /*!< ADC conversions are performed in continuous mode: after the first trigger, following conversions launched successively automatically */
```
- DMA请求`DMATransfer`

```c
/** @defgroup ADC_LL_EC_REG_DMA_TRANSFER  ADC group regular - DMA transfer of ADC conversion data
  * @{
  */
#define LL_ADC_REG_DMA_TRANSFER_NONE       0x00000000U              /*!< ADC conversions are not transferred by DMA */
#define LL_ADC_REG_DMA_TRANSFER_UNLIMITED  (ADC_CR2_DMA)                        /*!< ADC conversion data are transferred by DMA, in unlimited mode: DMA transfer requests are unlimited, whatever number of DMA data transferred (number of ADC conversions). This ADC mode is intended to be used with DMA mode circular. */
```

## 常用函数
- 计算电压值`__LL_ADC_CALC_DATA_TO_VOLTAGE(__VREFANALOG_VOLTAGE__,__ADC_DATA__,__ADC_RESOLUTION__)`
- 计算内部温度值`__LL_ADC_CALC_TEMPERATURE_TYP_PARAMS(__TEMPSENSOR_TYP_AVGSLOPE__,__TEMPSENSOR_TYP_CALX_V__,__TEMPSENSOR_CALX_TEMP__, __VREFANALOG_VOLTAGE__,__TEMPSENSOR_ADC_DATA__,__ADC_RESOLUTION__) `
- 获取ADC地址`uint32_t LL_ADC_DMA_GetRegAddr(ADC_TypeDef *ADCx, uint32_t Register)`
- 使能ADC`void LL_ADC_Enable(ADC_TypeDef *ADCx)`
- 开始校准`void LL_ADC_StartCalibration(ADC_TypeDef *ADCx)`
- 正在校准`uint32_t LL_ADC_IsCalibrationOnGoing(ADC_TypeDef *ADCx)`
- 软件触发开始转换`void LL_ADC_REG_StartConversionSWStart(ADC_TypeDef *ADCx)`
- 外部触发开始转换`void LL_ADC_REG_StartConversionExtTrig(ADC_TypeDef *ADCx, uint32_t ExternalTriggerEdge)`
- 读取转换值带过采样`uint32_t LL_ADC_REG_ReadConversionData32(ADC_TypeDef *ADCx)`
- 读取转换值`uint16_t LL_ADC_REG_ReadConversionData12(ADC_TypeDef *ADCx)`
- 读取转换完成标志位`uint32_t LL_ADC_IsActiveFlag_EOS(ADC_TypeDef *ADCx)`
- 使能转换完成中断`void LL_ADC_EnableIT_EOS(ADC_TypeDef *ADCx)`

# 代码移植
## 修改DMA中断
- 在`stm32f10x_it.c`添加外部变量

```c
/* USER CODE BEGIN EV */
extern __IO uint16_t ADC_ConvertedValue[3];
extern float VREF;
extern float temp;
extern float adcValue;
/* USER CODE END EV */
```
- 修改中断

```c
/**
  * @brief This function handles DMA1 channel1 global interrupt.
  */
void DMA1_Channel1_IRQHandler(void)
{
    /* USER CODE BEGIN DMA1_Channel1_IRQn 0 */
    if(LL_DMA_IsActiveFlag_TC1(DMA1)!=RESET)
    {
        LL_DMA_ClearFlag_TC1(DMA1);
        VREF=1.2f*4095/ADC_ConvertedValue[2];
        temp=__LL_ADC_CALC_TEMPERATURE_TYP_PARAMS(4300,1430,25,VREF*1000,ADC_ConvertedValue[1],LL_ADC_RESOLUTION_12B);//返回的是一个整数，如果到小数位，可以参考原来的
        adcValue=__LL_ADC_CALC_DATA_TO_VOLTAGE(VREF*1000,ADC_ConvertedValue[0],LL_ADC_RESOLUTION_12B)/1000;
    }
    /* USER CODE END DMA1_Channel1_IRQn 0 */

    /* USER CODE BEGIN DMA1_Channel1_IRQn 1 */

    /* USER CODE END DMA1_Channel1_IRQn 1 */
}
```

## 开始ADC转换
- 在`main.c`文件添加变量

```c
__IO uint16_t ADC_ConvertedValue[3] = {0};
float VREF;
float temp;
float adcValue;
```
- 在主函数初始化添加DMA开始转换

```c
    LL_DMA_SetPeriphAddress(DMA1,LL_DMA_CHANNEL_1,LL_ADC_DMA_GetRegAddr(ADC1,LL_ADC_DMA_REG_REGULAR_DATA)); //设置外设地址
    LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_1,(uint32_t)ADC_ConvertedValue);//设置存储器地址
    LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_1,3);//设置长度
    LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_1);//使能DMA通道
    LL_DMA_EnableIT_TC(DMA1,LL_DMA_CHANNEL_1);//使能DMA完成中断
    LL_ADC_Enable(ADC1);//使能ADC
    LL_ADC_StartCalibration(ADC1);//开始校准
    while(LL_ADC_IsCalibrationOnGoing(ADC1));//等待校准完成
    LL_ADC_REG_StartConversionSWStart(ADC1);//软件触发启动
```
- 主循环添加数据发送

```c
        Delay(0xffffee);      // 延时
        printf( "\r\n The IC current tem= %.2fC\r\n", temp);
        printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
        printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
```
# 下载调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 会看到串口电源电压，温度，PC1电压值，用万用表测量比对
- 和HAL库结果一致