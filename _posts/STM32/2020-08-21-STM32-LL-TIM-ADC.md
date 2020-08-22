---
title: STM32底层LL库TIM触发ADCLL篇12
categories: STM32 CUBE LL ADC DMA TIM
tags: STM32 CUBE LL ADC DMA TIM
description: LL库TIM触发ADC
---
# 配置TIM
- 和HAL库配置一样
- 在高级设置里为**TIM4**选择`LL`库
- 生成代码

# 代码移植
## 修改接收数组
- 在`main.h`添加宏

```c
#define ADC_BUFF_SIZE 100
```
- 修改`main.c`文件的数组为

```c
__IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][3];
```

- 修改`stm32f10x_it.c`文件的外部变量数组为

```c
extern __IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][3];
```
## 修改中断函数

```c
/**
  * @brief This function handles DMA1 channel1 global interrupt.
  */
void DMA1_Channel1_IRQHandler(void)
{
  /* USER CODE BEGIN DMA1_Channel1_IRQn 0 */
    uint8_t i=0;
    uint32_t vref_adc=0,vsense_adc=0,adc_adc=0;
    if(LL_DMA_IsActiveFlag_TC1(DMA1)!=RESET)
    {
        LL_DMA_ClearFlag_TC1(DMA1);
        for(i=0; i<ADC_BUFF_SIZE; i++)
        {
            vref_adc+=ADC_ConvertedValue[i][2];
            vsense_adc+=ADC_ConvertedValue[i][1];
            adc_adc+=ADC_ConvertedValue[i][0];
        }
        VREF=1.2f*4095*100/vref_adc;
        temp=__LL_ADC_CALC_TEMPERATURE_TYP_PARAMS(4300,1430,25,VREF*1000,vsense_adc/100,LL_ADC_RESOLUTION_12B);
        adcValue=__LL_ADC_CALC_DATA_TO_VOLTAGE(VREF*1000,adc_adc/100,LL_ADC_RESOLUTION_12B)/1000;
        printf( "\r\n The IC current tem= %.2fC\r\n", temp);
        printf( "\r\n The IC current VDDA= %.2fV\r\n", VREF);
        printf( "\r\n The IC current ADC= %.2fV\r\n", adcValue);
    }
  /* USER CODE END DMA1_Channel1_IRQn 0 */

  /* USER CODE BEGIN DMA1_Channel1_IRQn 1 */

  /* USER CODE END DMA1_Channel1_IRQn 1 */
}
```
## 时间应用
- 在主函数初始化添加定时器使能，修改ADC外部触发启动

```c
    LL_DMA_SetPeriphAddress(DMA1,LL_DMA_CHANNEL_1,LL_ADC_DMA_GetRegAddr(ADC1,LL_ADC_DMA_REG_REGULAR_DATA));
    LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_1,(uint32_t)ADC_ConvertedValue);
    //LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_1,3);
    LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_1,3*ADC_BUFF_SIZE); //修改缓冲区大小
    LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_1);
    LL_DMA_EnableIT_TC(DMA1,LL_DMA_CHANNEL_1);
    LL_ADC_Enable(ADC1);
    LL_ADC_StartCalibration(ADC1);
    while(LL_ADC_IsCalibrationOnGoing(ADC1));
    //LL_ADC_REG_StartConversionSWStart(ADC1);
    LL_ADC_REG_StartConversionExtTrig(ADC1,LL_ADC_REG_TRIG_EXT_RISING);//修改为外部触发
    LL_TIM_EnableCounter(TIM4);//使能定时器
    LL_TIM_CC_EnableChannel(TIM4,LL_TIM_CHANNEL_CH4);//使能通道
```

- 删除主循环里的发送

# 调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 可以看到串口每1秒接收一次计算后的数值
- 和HAL库结果一致