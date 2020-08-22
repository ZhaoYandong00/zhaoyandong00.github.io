---
title: STM32硬件层HAL库通用定时器PWM输出HAL篇10
categories: STM32 CUBE HAL TIM PWM
tags: STM32 CUBE HAL TIM PWM
description: HAL库通用定时器PWM输出
---
# 配置TIM
- 打开CUBE工程

## 在**`Timers`**->**`TIM3`**配置
- 选择**Clock Source**——`Internal Clock`
- 选择**Channel1**——`PWM Generation CH1`
- 选择**Channel2**——`PWM Generation CH2`
- 选择**Channel3**——`PWM Generation CH3`
- 选择**Channel4**——`PWM Generation CH4`
- **`Parameter Settings`**配置
- 配置预分频器72分频——`71`，自动重载数值——`99`，计数模式——`UP`，自动重载预装载——`Enable` **CKD**-`No Division`
- 配置**Channel 1**:**Mode**——`PWM Mode 1`, **Pulse**——`50`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`High`
- 配置**Channel 2**:**Mode**——`PWM Mode 1`, **Pulse**——`40`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`High`
- 配置**Channel 3**:**Mode**——`PWM Mode 1`, **Pulse**——`30`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`High`
- 配置**Channel 4**:**Mode**——`PWM Mode 1`, **Pulse**——`20`,**Output compare preload**——`Enable`,**Fast Mode**——`Disable`,**CH Polarity**——`High`

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# 库函数解析

- 使能通道`void TIM_CCxChannelCmd(TIM_TypeDef *TIMx, uint32_t Channel, uint32_t ChannelState)`


# 代码移植
## 添加更改频率和占空比函数
- 在`tim.c`文件添加函数

```c
/* USER CODE BEGIN 1 */
/**
  * @brief  控制PWM频率
  * @param  freq  频率
  * @param  duty1 通道1占空比
  * @param  duty2 通道2占空比
  * @param  duty3 通道3占空比
  * @param  duty4 通道4占空比
  * @retval 无
  */
void Set_General_PWM_FREQ(uint16_t freq,uint8_t duty1,uint8_t duty2,uint8_t duty3,uint8_t duty4)
{
    uint16_t count=1000000/freq;
    __HAL_TIM_SET_AUTORELOAD(&htim3, count-1);
    __HAL_TIM_SET_COMPARE(&htim3,TIM_CHANNEL_1,count*duty1/100);
    __HAL_TIM_SET_COMPARE(&htim3,TIM_CHANNEL_2,count*duty2/100);
    __HAL_TIM_SET_COMPARE(&htim3,TIM_CHANNEL_3,count*duty3/100);
    __HAL_TIM_SET_COMPARE(&htim3,TIM_CHANNEL_4,count*duty4/100);
}
/* USER CODE END 1 */
```
- 在`tim.h`文件添加函数定义

```c
/* USER CODE BEGIN Prototypes */
void Set_General_PWM_FREQ(uint16_t freq,uint8_t duty1,uint8_t duty2,uint8_t duty3,uint8_t duty4);
/* USER CODE END Prototypes */
```
## 定时器应用
- 在`main.c`文件添加变量

```c
uint32_t freq;
uint8_t duty1,duty2,duty3,duty4;
```
- 在主函数初始化添加TIM启动

```c
TIM_CCxChannelCmd(TIM3,TIM_CHANNEL_1,TIM_CCx_ENABLE);//使能通道1
TIM_CCxChannelCmd(TIM3,TIM_CHANNEL_2,TIM_CCx_ENABLE);//使能通道2
TIM_CCxChannelCmd(TIM3,TIM_CHANNEL_3,TIM_CCx_ENABLE);//使能通道3
HAL_TIM_PWM_Start(&htim3,TIM_CHANNEL_4);//使能通道4，并开启定时器
freq=10000;
duty1=50;
duty2=40;
duty3=30;
duty4=20;
```
- 主循环TIM6计时里添加PWM占空比处理

```c
            freq-=1000;
            if(freq<=1000)
            {
                freq=10000;
            }
            duty1+=10;
            duty2-=10;
            duty3+=5;
            duty4-=5;
            if(duty1>=100)
            {
                duty1=10;
            }
            if(duty2<=10)
            {
                duty2=100;
            }
            if(duty3>=100)
            {
                duty3=5;
            }
            if(duty4<=5)
            {
                duty4=100;
            }
            Set_General_PWM_FREQ(freq,duty1,duty2,duty3,duty4);
```
## 下载调试
- 编译之后下载到开发板
- 使用示波器查看输出波形
- 可以看到波形频率每1s减少1K,从10K-1K变化，
- PA6波形占空比从10-90变化,步进10
- PA7波形占空比从100-10变化,步进10
- PB0波形占空比从5-95变化,步进5
- PB1波形占空比从95-5变化,步进5
- 和SPL库结果一致