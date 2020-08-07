---
title: STM32标准外设库通用定时器PWM输出
categories: STM32 TIM PWM
tags: STM32 SPL TIM PWM
description: SPL库通用定时器PWM输出
---
# TIM初始化
## TIM宏定义

- `my_general.pwm.h`

```c
#ifndef __GENERAL_PWM_H
#define __GENERAL_PWM_H
#include "stm32f10x.h"
/************通用定时器TIM参数定义，只限TIM2、3、4、5************/
// 当使用不同的定时器的时候，对应的GPIO是不一样的，这点要注意
// 我们这里使用TIM3

#define            GENERAL_TIM                   TIM3
#define            GENERAL_TIM_APBxClock_FUN     RCC_APB1PeriphClockCmd
#define            GENERAL_TIM_CLK               RCC_APB1Periph_TIM3
#define            GENERAL_TIM_Period            100-1
#define            GENERAL_TIM_Prescaler         72-1
// TIM3 输出比较通道1
#define            GENERAL_TIM_CH1_GPIO_CLK      RCC_APB2Periph_GPIOA
#define            GENERAL_TIM_CH1_PORT          GPIOA
#define            GENERAL_TIM_CH1_PIN           GPIO_Pin_6

// TIM3 输出比较通道2
#define            GENERAL_TIM_CH2_GPIO_CLK      RCC_APB2Periph_GPIOA
#define            GENERAL_TIM_CH2_PORT          GPIOA
#define            GENERAL_TIM_CH2_PIN           GPIO_Pin_7

// TIM3 输出比较通道3
#define            GENERAL_TIM_CH3_GPIO_CLK      RCC_APB2Periph_GPIOB
#define            GENERAL_TIM_CH3_PORT          GPIOB
#define            GENERAL_TIM_CH3_PIN           GPIO_Pin_0

// TIM3 输出比较通道4
#define            GENERAL_TIM_CH4_GPIO_CLK      RCC_APB2Periph_GPIOB
#define            GENERAL_TIM_CH4_PORT          GPIOB
#define            GENERAL_TIM_CH4_PIN           GPIO_Pin_1



void GENERAL_PWM_Init(void);
void Set_General_PWM_FREQ(uint16_t freq,uint8_t duty1,uint8_t duty2,uint8_t duty3,uint8_t duty4);
#endif

```

## TIM配置和PWM设置
- `my_general_pwm.c`

```c
#include "my_general_pwm.h"
/**
 * @brief  PWM GPIO配置
 * @param  无
 * @retval 无
 */
static void GENERAL_TIM_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    // 输出比较通道1 GPIO 初始化
    RCC_APB2PeriphClockCmd(GENERAL_TIM_CH1_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  GENERAL_TIM_CH1_PIN;
    GPIO_Init(GENERAL_TIM_CH1_PORT, &GPIO_InitStructure);

    // 输出比较通道2 GPIO 初始化
    RCC_APB2PeriphClockCmd(GENERAL_TIM_CH2_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  GENERAL_TIM_CH2_PIN;
    GPIO_Init(GENERAL_TIM_CH2_PORT, &GPIO_InitStructure);

    // 输出比较通道3 GPIO 初始化
    RCC_APB2PeriphClockCmd(GENERAL_TIM_CH3_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  GENERAL_TIM_CH3_PIN;
    GPIO_Init(GENERAL_TIM_CH3_PORT, &GPIO_InitStructure);

    // 输出比较通道4 GPIO 初始化
    RCC_APB2PeriphClockCmd(GENERAL_TIM_CH3_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  GENERAL_TIM_CH3_PIN;
    GPIO_Init(GENERAL_TIM_CH3_PORT, &GPIO_InitStructure);
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

static void GENERAL_TIM_Mode_Config(void)
{
    // 开启定时器时钟,即内部时钟CK_INT=72M
    GENERAL_TIM_APBxClock_FUN(GENERAL_TIM_CLK,ENABLE);

    /*--------------------时基结构体初始化-------------------------*/
    // 配置周期，这里配置为10K

    TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure;
    // 自动重装载寄存器的值，累计TIM_Period+1个频率后产生一个更新或者中断
    TIM_TimeBaseStructure.TIM_Period=GENERAL_TIM_Period;
    // 驱动CNT计数器的时钟 = Fck_int/(psc+1)
    TIM_TimeBaseStructure.TIM_Prescaler= GENERAL_TIM_Prescaler;
    // 时钟分频因子 ，配置死区时间时需要用到
    TIM_TimeBaseStructure.TIM_ClockDivision=TIM_CKD_DIV1;
    // 计数器计数模式，设置为向上计数
    TIM_TimeBaseStructure.TIM_CounterMode=TIM_CounterMode_Up;
    // 重复计数器的值，没用到不用管
    TIM_TimeBaseStructure.TIM_RepetitionCounter=0;
    // 初始化定时器
    TIM_TimeBaseInit(GENERAL_TIM, &TIM_TimeBaseStructure);

    /*--------------------输出比较结构体初始化-------------------*/
    // 占空比配置
    uint16_t CCR1_Val = 49;
    uint16_t CCR2_Val = 39;
    uint16_t CCR3_Val = 29;
    uint16_t CCR4_Val = 19;

    TIM_OCInitTypeDef  TIM_OCInitStructure;
    // 配置为PWM模式1
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
    // 输出使能
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
    // 输出通道电平极性配置
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;

    // 输出比较通道 1
    TIM_OCInitStructure.TIM_Pulse = CCR1_Val;
    TIM_OC1Init(GENERAL_TIM, &TIM_OCInitStructure);
    TIM_OC1PreloadConfig(GENERAL_TIM, TIM_OCPreload_Enable);

    // 输出比较通道 2
    TIM_OCInitStructure.TIM_Pulse = CCR2_Val;
    TIM_OC2Init(GENERAL_TIM, &TIM_OCInitStructure);
    TIM_OC2PreloadConfig(GENERAL_TIM, TIM_OCPreload_Enable);

    // 输出比较通道 3
    TIM_OCInitStructure.TIM_Pulse = CCR3_Val;
    TIM_OC3Init(GENERAL_TIM, &TIM_OCInitStructure);
    TIM_OC3PreloadConfig(GENERAL_TIM, TIM_OCPreload_Enable);

    // 输出比较通道 4
    TIM_OCInitStructure.TIM_Pulse = CCR4_Val;
    TIM_OC4Init(GENERAL_TIM, &TIM_OCInitStructure);
    TIM_OC4PreloadConfig(GENERAL_TIM, TIM_OCPreload_Enable);

    // 使能计数器
    TIM_Cmd(GENERAL_TIM, ENABLE);
    //使能重载寄存器ARR
    TIM_ARRPreloadConfig(GENERAL_TIM, ENABLE);
}

void GENERAL_PWM_Init(void)
{
    GENERAL_TIM_GPIO_Config();
    GENERAL_TIM_Mode_Config();
}
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
    TIM_SetAutoreload(GENERAL_TIM, count-1);
    TIM_SetCompare1(GENERAL_TIM,count*duty1/100-1);
    TIM_SetCompare2(GENERAL_TIM,count*duty2/100-1);
    TIM_SetCompare3(GENERAL_TIM,count*duty3/100-1);
    TIM_SetCompare4(GENERAL_TIM,count*duty4/100-1);
}

```

# 动态更改输出频率和占空比
## 添加使能

```c
//使能重载寄存器ARR
TIM_ARRPreloadConfig(GENERAL_TIM, ENABLE);
```
## 更改频率和占空比

```c
uint16_t count=1000000/freq; //计算ARR值
TIM_SetAutoreload(GENERAL_TIM, count-1); //设置ARR
TIM_SetCompare1(GENERAL_TIM,count*duty1/100-1);//1通道占空比
TIM_SetCompare2(GENERAL_TIM,count*duty2/100-1);//2通道占空比
TIM_SetCompare3(GENERAL_TIM,count*duty3/100-1);//3通道占空比
TIM_SetCompare4(GENERAL_TIM,count*duty4/100-1);//4通道占空比
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
#include "my_general_pwm.h"
extern __IO uint16_t ADC_ConvertedValue[NOFCHANEL];
Channel ch[2];
float VREF,VSENSE;
float temp;
float adcValue;
__IO uint32_t time = 0; // ms 计时变量
uint32_t freq;
uint8_t duty1,duty2,duty3,duty4;
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
    /*初始化PWM  10KHz 50% 40% 30% 20%*/
    GENERAL_PWM_Init();
    freq=10000;
    duty1=50;
    duty2=40;
    duty3=30;
    duty4=20;
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
- 使用示波器查看输出波形
- 可以看到波形频率每1s减少1K,从10K-1K变化，
- PA6波形占空比从10-90变化,步进10
- PA7波形占空比从100-10变化,步进10
- PB0波形占空比从5-95变化,步进5
- PB1波形占空比从95-5变化,步进5
