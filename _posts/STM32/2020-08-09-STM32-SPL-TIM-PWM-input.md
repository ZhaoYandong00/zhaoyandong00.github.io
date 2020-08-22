---
title: STM32标准外设库TIMPWM输入测量SPL篇13
categories: STM32 TIM PWM
tags: STM32 SPL TIM PWM DUTY
description: SPL库TIMPWM输入测量
---
# TIM初始化
## TIM宏定义
- `my_PWM_capture.h`

```c
#ifndef __MY_PWM_CAPTURE_H
#define __MY_PWM_CAPTURE_H
#include "stm32f10x.h"
#define            ADVANCE_CAPTURE_TIM                   TIM8
#define            ADVANCE_CAPTURE_TIM_APBxClock_FUN     RCC_APB2PeriphClockCmd
#define            ADVANCE_CAPTURE_TIM_CLK               RCC_APB2Periph_TIM8

// 输入捕获能捕获到的最小的频率为 72M/{ (ARR+1)*(PSC+1) }
#define            ADVANCE_CAPTURE_TIM_PERIOD            (1000-1)
#define            ADVANCE_CAPTURE_TIM_PSC               (72-1)

// 中断相关宏定义
#define            ADVANCE_CAPTURE_TIM_IRQ               TIM8_CC_IRQn
#define            ADVANCE_CAPTURE_TIM_IRQHandler        TIM8_CC_IRQHandler

// TIM1 输入捕获通道1
#define            ADVANCE_CAPTURE_TIM_CH1_GPIO_CLK      RCC_APB2Periph_GPIOC
#define            ADVANCE_CAPTURE_TIM_CH1_PORT          GPIOC
#define            ADVANCE_CAPTURE_TIM_CH1_PIN           GPIO_Pin_6

#define            ADVANCE_CAPTURE_TIM_IC1PWM_CHANNEL    TIM_Channel_1
#define            ADVANCE_CAPTURE_TIM_IC2PWM_CHANNEL    TIM_Channel_2

/**************************函数声明********************************/

void ADVANCE_CAPTURE_TIM_Init(void);
#endif

```
## 输入捕获管脚设置和工作模式配置
- `my_PWM_capture.c`

```c
#include "my_PWM_capture.h"
/**
 * @brief  高级控制定时器 TIMx,x[1,8]中断优先级配置
 * @param  无
 * @retval 无
 */
static void ADVANCE_CAPTURE_TIM_NVIC_Config(void)
{
    NVIC_InitTypeDef NVIC_InitStructure;
    // 设置中断组为0
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_0);
    // 设置中断来源
    NVIC_InitStructure.NVIC_IRQChannel = ADVANCE_CAPTURE_TIM_IRQ;
    // 设置抢占优先级
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
    // 设置子优先级
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 3;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);
}
/**
  * @brief  高级定时器PWM输入用到的GPIO初始化
  * @param  无
  * @retval 无
  */
static void ADVANCE_CAPTURE_TIM_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    RCC_APB2PeriphClockCmd(ADVANCE_CAPTURE_TIM_CH1_GPIO_CLK, ENABLE);
    GPIO_InitStructure.GPIO_Pin =  ADVANCE_CAPTURE_TIM_CH1_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(ADVANCE_CAPTURE_TIM_CH1_PORT, &GPIO_InitStructure);
}

/**
  * @brief  高级定时器PWM输入初始化和用到的GPIO初始化
  * @param  无
  * @retval 无
  */
static void ADVANCE_CAPTURE_TIM_Mode_Config(void)
{
    // 开启定时器时钟,即内部时钟CK_INT=72M
    ADVANCE_CAPTURE_TIM_APBxClock_FUN(ADVANCE_CAPTURE_TIM_CLK,ENABLE);

    /*--------------------时基结构体初始化-------------------------*/
    TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure;
    // 自动重装载寄存器的值，累计TIM_Period+1个频率后产生一个更新或者中断
    TIM_TimeBaseStructure.TIM_Period=ADVANCE_CAPTURE_TIM_PERIOD;
    // 驱动CNT计数器的时钟 = Fck_int/(psc+1)
    TIM_TimeBaseStructure.TIM_Prescaler= ADVANCE_CAPTURE_TIM_PSC;
    // 时钟分频因子 ，配置死区时间时需要用到
    TIM_TimeBaseStructure.TIM_ClockDivision=TIM_CKD_DIV1;
    // 计数器计数模式，设置为向上计数
    TIM_TimeBaseStructure.TIM_CounterMode=TIM_CounterMode_Up;
    // 重复计数器的值，没用到不用管
    TIM_TimeBaseStructure.TIM_RepetitionCounter=0;
    // 初始化定时器
    TIM_TimeBaseInit(ADVANCE_CAPTURE_TIM, &TIM_TimeBaseStructure);

    /*--------------------输入捕获结构体初始化-------------------*/
    // 使用PWM输入模式时，需要占用两个捕获寄存器，一个测周期，另外一个测占空比

    TIM_ICInitTypeDef  TIM_ICInitStructure;
    // 捕获通道IC1配置
    // 选择捕获通道
    TIM_ICInitStructure.TIM_Channel = ADVANCE_CAPTURE_TIM_IC1PWM_CHANNEL;
    // 设置捕获的边沿
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;
    // 设置捕获通道的信号来自于哪个输入通道，有直连和非直连两种
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;
    // 1分频，即捕获信号的每个有效边沿都捕获
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
    // 不滤波
    TIM_ICInitStructure.TIM_ICFilter = 0;
    // 初始化PWM输入模式
    TIM_PWMIConfig(ADVANCE_CAPTURE_TIM, &TIM_ICInitStructure);

    // 当工作做PWM输入模式时,只需要设置触发信号的那一路即可（用于测量周期）
    // 另外一路（用于测量占空比）会由硬件自带设置，不需要再配置

    // 捕获通道IC2配置
//	TIM_ICInitStructure.TIM_Channel = ADVANCE_TIM_IC1PWM_CHANNEL;
//  TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Falling;
//  TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_IndirectTI;
//  TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
//  TIM_ICInitStructure.TIM_ICFilter = 0x0;
//  TIM_PWMIConfig(ADVANCE_TIM, &TIM_ICInitStructure);

    // 选择输入捕获的触发信号
    TIM_SelectInputTrigger(ADVANCE_CAPTURE_TIM, TIM_TS_TI1FP1);

    // 选择从模式: 复位模式
    // PWM输入模式时,从模式必须工作在复位模式，当捕获开始时,计数器CNT会被复位
    TIM_SelectSlaveMode(ADVANCE_CAPTURE_TIM, TIM_SlaveMode_Reset);
    TIM_SelectMasterSlaveMode(ADVANCE_CAPTURE_TIM,TIM_MasterSlaveMode_Enable);

    // 使能捕获中断,这个中断针对的是主捕获通道（测量周期那个）
    TIM_ITConfig(ADVANCE_CAPTURE_TIM, TIM_IT_CC1, ENABLE);
    // 清除中断标志位
    TIM_ClearITPendingBit(ADVANCE_CAPTURE_TIM, TIM_IT_CC1);

    // 使能高级控制定时器，计数器开始计数
    TIM_Cmd(ADVANCE_CAPTURE_TIM, ENABLE);
}
/**
  * @brief  高级定时器PWM输入初始化和用到的GPIO初始化
  * @param  无
  * @retval 无
  */
void ADVANCE_CAPTURE_TIM_Init(void)
{
    ADVANCE_CAPTURE_TIM_GPIO_Config();
    ADVANCE_CAPTURE_TIM_NVIC_Config();
    ADVANCE_CAPTURE_TIM_Mode_Config();
}

```
## PWM捕获输入配置

```c
    // 选择输入捕获的触发信号
    TIM_SelectInputTrigger(ADVANCE_CAPTURE_TIM, TIM_TS_TI1FP1);

    // 选择从模式: 复位模式
    // PWM输入模式时,从模式必须工作在复位模式，当捕获开始时,计数器CNT会被复位
    TIM_SelectSlaveMode(ADVANCE_CAPTURE_TIM, TIM_SlaveMode_Reset);
    TIM_SelectMasterSlaveMode(ADVANCE_CAPTURE_TIM,TIM_MasterSlaveMode_Enable);

    // 使能捕获中断,这个中断针对的是主捕获通道（测量周期那个）
    TIM_ITConfig(ADVANCE_CAPTURE_TIM, TIM_IT_CC1, ENABLE);
    // 清除中断标志位
    TIM_ClearITPendingBit(ADVANCE_CAPTURE_TIM, TIM_IT_CC1);
```
# 添加中断函数到`stm32f10x_it.c`
## 添加TIM头文件

`#include "my_PWM_capture.h"`
## 添加中断函数

```c
__IO uint16_t IC2Value = 0;
__IO uint16_t IC1Value = 0;
__IO float DutyCycle = 0;
__IO float Frequency = 0;
/**
  * @brief  This function handles TIM interrupt request.
  * @param  None
  * @retval None
  */
void ADVANCE_CAPTURE_TIM_IRQHandler(void)
{
    if(TIM_GetITStatus ( ADVANCE_CAPTURE_TIM, TIM_IT_CC1) != RESET)
    {   /* 清除中断标志位 */
        TIM_ClearITPendingBit(ADVANCE_CAPTURE_TIM, TIM_IT_CC1);

        /* 获取输入捕获值 */
        IC1Value = TIM_GetCapture1(ADVANCE_CAPTURE_TIM);
        IC2Value = TIM_GetCapture2(ADVANCE_CAPTURE_TIM);

        // 注意：捕获寄存器CCR1和CCR2的值在计算占空比和频率的时候必须加1
        if (IC1Value != 0)
        {
            /* 占空比计算 */
            DutyCycle = (float)((IC2Value+1) * 100) / (IC1Value+1);

            /* 频率计算 */
            Frequency = (72000000/(ADVANCE_CAPTURE_TIM_PSC+1))/(float)(IC1Value+1);
        }
        else
        {
            DutyCycle = 0;
            Frequency = 0;
        }
    }
}
```
# 调试
## 添加捕获TIM头文件`main.c`和调用初始化

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
#include "my_capture.h"
#include "my_PWM_capture.h"
extern __IO uint16_t ADC_ConvertedValue[NOFCHANEL];
Channel ch[2];
float VREF,VSENSE;
float temp;
float adcValue;
__IO uint32_t time = 0; // ms 计时变量
uint32_t freq;
uint8_t duty1,duty2,duty3,duty4;
extern __IO uint16_t IC1Value;
extern __IO float DutyCycle;
extern __IO float Frequency;
int main(void)
{
    uint32_t capture_time;
    uint32_t TIM_PscCLK = 72000000 / (GENERAL_CAPTURE_TIM_PSC+1);
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
    /*初始化输入捕获*/
    CAPTURE_TIM_Init();
    /*初始化PWM捕获*/
    ADVANCE_CAPTURE_TIM_Init();
    freq=10000;
    duty1=50;
    duty2=40;
    duty3=30;
    duty4=20;
    printf( "\r\n Print current Temperature  \r\n");

    while (1)
    {
        if(IC1Value!=0)
        {
            printf("DUTY:%0.2f%%   FREQ:%0.2fHz\n",DutyCycle,Frequency);
        }
        if(TIM_ICUserValueStructure.Capture_FinishFlag == 1)
        {
            // 计算高电平时间的计数器的值
            capture_time = TIM_ICUserValueStructure.Capture_Period * (GENERAL_CAPTURE_TIM_PERIOD+1) +
                           (TIM_ICUserValueStructure.Capture_CcrValue+1);

            // 打印高电平脉宽时间
            printf ( "\r\n duty time:%d.%d s\r\n",capture_time/TIM_PscCLK,capture_time%TIM_PscCLK );

            TIM_ICUserValueStructure.Capture_FinishFlag = 0;
        }
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
- 连接PC6到PA6
- 连接开发板串口
- 打开串口助手
- 可以看到频率和占空比和示波器看到的一样