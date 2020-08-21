---
title: STM32底层LL库通用定时器PWM输出
categories: STM32 CUBE LL TIM PWM
tags: STM32 CUBE LL TIM PWM
description: LL库通用定时器PWM输出
---
---
# 配置TIM
- 和HAL库配置一样
- 在高级设置里为**TIM3**选择`LL`库
- 生成代码

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
    uint16_t count=__LL_TIM_CALC_ARR(72000000,LL_TIM_GetPrescaler(TIM3),freq);
    LL_TIM_SetAutoReload(TIM3,count);
    LL_TIM_OC_SetCompareCH1(TIM3,(count+1)*duty1/100);
    LL_TIM_OC_SetCompareCH2(TIM3,(count+1)*duty2/100);
    LL_TIM_OC_SetCompareCH3(TIM3,(count+1)*duty3/100);
    LL_TIM_OC_SetCompareCH4(TIM3,(count+1)*duty4/100);
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
- 和HAL库结果一致