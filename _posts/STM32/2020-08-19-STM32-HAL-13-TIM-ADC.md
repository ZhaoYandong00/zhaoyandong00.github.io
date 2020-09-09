---
title: STM32硬件层HAL库TIM触发ADCHAL篇13
categories: STM32 CUBE HAL ADC DMA TIM
tags: STM32 CUBE HAL ADC DMA TIM
description: HAL库TIM触发ADC
---
# 配置ADC
- 打开CUBE工程

## 在**`Analog`**->**`ADC1`**配置
- 修改外部触发为`Timer 4 Capture Compare 4 event`
- 修改**Countinuons Conversion Mode**——`Disable`

## 在**`Timers`**->**`TIM4`**配置
- 选择`Internal Clock`
- 选择**Channel4**——`PWM Compare No Output`
- **`Parameter Settings`**配置
- 配置预分频器720分频——`719`，自动重载数值——`999`，计数模式——`UP`，自动重载预装载——`Disable` **CKD**——`No Division`
- 配置**Channel 4**:**Mode**——`PWM Mode 1`, **Pulse**——`500`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`Low`

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# 代码移植
## 修改接收数组

```c
//__IO uint16_t ADC_ConvertedValue[3] = {0};
#define ADC_BUFF_SIZE 100
__IO uint16_t ADC_ConvertedValue[ADC_BUFF_SIZE][3];
```

## 修改主程序初始化函数的ADC启动

```c
//HAL_ADC_Start_DMA(&hadc1,(uint32_t*)&ADC_ConvertedValue,3);
HAL_ADC_Start_DMA(&hadc1,(uint32_t*)&ADC_ConvertedValue,3*ADC_BUFF_SIZE);
HAL_TIM_PWM_Start(&htim4,TIM_CHANNEL_4);
```
## 修改ADC完成回调函数

```c
/**
  * @brief  Conversion complete callback in non blocking mode
  * @param  hadc: ADC handle
  * @retval None
  */
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc)
{
    uint8_t i=0;
    uint32_t vref_adc=0,vsense_adc=0,adc_adc=0;
    if(hadc->Instance==ADC1)
    {
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
- 和SPL库结果一致
