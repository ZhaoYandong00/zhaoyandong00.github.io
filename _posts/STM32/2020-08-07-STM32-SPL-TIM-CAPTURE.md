---
title: STM32标准外设库TIM输入捕获
categories: STM32 TIM CAPTURE
tags: STM32 SPL TIM CAPTURE
description: SPL库TIM输入捕获
---
# TIM初始化
## TIM宏定义
- `my_capture.h`

```c
#ifndef __MY_CAPTURE_H
#define __MY_CAPTURE_H
#include "stm32f10x.h"
#define            GENERAL_CAPTURE_TIM                   TIM5
#define            GENERAL_CAPTURE_TIM_APBxClock_FUN     RCC_APB1PeriphClockCmd
#define            GENERAL_CAPTURE_TIM_CLK               RCC_APB1Periph_TIM5
#define            GENERAL_CAPTURE_TIM_PERIOD            0XFFFF
#define            GENERAL_CAPTURE_TIM_PSC               (72-1)

// TIM 输入捕获通道GPIO相关宏定义
#define            GENERAL_CAPTURE_TIM_CH1_GPIO_CLK      RCC_APB2Periph_GPIOA
#define            GENERAL_CAPTURE_TIM_CH1_PORT          GPIOA
#define            GENERAL_CAPTURE_TIM_CH1_PIN           GPIO_Pin_0
#define            GENERAL_CAPTURE_TIM_CHANNEL_x         TIM_Channel_1

// 中断相关宏定义
#define            GENERAL_CAPTURE_TIM_IT_CCx            TIM_IT_CC1
#define            GENERAL_CAPTURE_TIM_IRQ               TIM5_IRQn
#define            GENERAL_CAPTURE_TIM_INT_FUN           TIM5_IRQHandler

// 获取捕获寄存器值函数宏定义
#define            GENERAL_CAPTURE_TIM_GetCapturex_FUN                 TIM_GetCapture1
// 捕获信号极性函数宏定义
#define            GENERAL_CAPTURE_TIM_OCxPolarityConfig_FUN           TIM_OC1PolarityConfig

// 测量的起始边沿
#define            GENERAL_CAPTURE_TIM_STRAT_ICPolarity                TIM_ICPolarity_Rising
// 测量的结束边沿
#define            GENERAL_CAPTURE_TIM_END_ICPolarity                  TIM_ICPolarity_Falling


// 定时器输入捕获用户自定义变量结构体声明
typedef struct
{   
	uint8_t   Capture_FinishFlag;   // 捕获结束标志位
	uint8_t   Capture_StartFlag;    // 捕获开始标志位
	uint16_t  Capture_CcrValue;     // 捕获寄存器的值
	uint16_t  Capture_Period;       // 自动重装载寄存器更新标志 
}TIM_ICUserValueTypeDef;

extern TIM_ICUserValueTypeDef TIM_ICUserValueStructure;

/**************************函数声明********************************/
void CAPTURE_TIM_Init(void);
#endif

```
## 输入捕获管脚设置和工作模式配置
- `my_capture.c`

```c
#include "my_capture.h"
// 定时器输入捕获用户自定义变量结构体定义
TIM_ICUserValueTypeDef TIM_ICUserValueStructure = {0,0,0,0};

/**
 * @brief  NVIC配置
 * @param  无
 * @retval 无
 */
static void GENERAL_TIM_NVIC_Config(void)
{
    NVIC_InitTypeDef NVIC_InitStructure; 
    // 设置中断组为0
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_0);		
		// 设置中断来源
    NVIC_InitStructure.NVIC_IRQChannel = GENERAL_CAPTURE_TIM_IRQ ;	
		// 设置主优先级为 0
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;	 
	  // 设置抢占优先级为3
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 3;	
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);
}
/**
 * @brief  GPIO配置
 * @param  无
 * @retval 无
 */
static void GENERAL_TIM_GPIO_Config(void) 
{
  GPIO_InitTypeDef GPIO_InitStructure;

  // 输入捕获通道 GPIO 初始化
	RCC_APB2PeriphClockCmd(GENERAL_CAPTURE_TIM_CH1_GPIO_CLK, ENABLE);
  GPIO_InitStructure.GPIO_Pin =  GENERAL_CAPTURE_TIM_CH1_PIN;
  GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
  GPIO_Init(GENERAL_CAPTURE_TIM_CH1_PORT, &GPIO_InitStructure);	
}
/**
 * @brief  TIM配置
 * @param  无
 * @retval 无
 */
static void GENERAL_TIM_Mode_Config(void)
{
  // 开启定时器时钟,即内部时钟CK_INT=72M
	GENERAL_CAPTURE_TIM_APBxClock_FUN(GENERAL_CAPTURE_TIM_CLK,ENABLE);

/*--------------------时基结构体初始化-------------------------*/	
  TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure;
	// 自动重装载寄存器的值，累计TIM_Period+1个频率后产生一个更新或者中断
	TIM_TimeBaseStructure.TIM_Period=GENERAL_CAPTURE_TIM_PERIOD;	
	// 驱动CNT计数器的时钟 = Fck_int/(psc+1)
	TIM_TimeBaseStructure.TIM_Prescaler= GENERAL_CAPTURE_TIM_PSC;	
	// 时钟分频因子 ，配置死区时间时需要用到
	TIM_TimeBaseStructure.TIM_ClockDivision=TIM_CKD_DIV1;		
	// 计数器计数模式，设置为向上计数
	TIM_TimeBaseStructure.TIM_CounterMode=TIM_CounterMode_Up;		
	// 重复计数器的值，没用到不用管
	TIM_TimeBaseStructure.TIM_RepetitionCounter=0;	
	// 初始化定时器
	TIM_TimeBaseInit(GENERAL_CAPTURE_TIM, &TIM_TimeBaseStructure);

	/*--------------------输入捕获结构体初始化-------------------*/	
	TIM_ICInitTypeDef TIM_ICInitStructure;
	// 配置输入捕获的通道，需要根据具体的GPIO来配置
	TIM_ICInitStructure.TIM_Channel = GENERAL_CAPTURE_TIM_CHANNEL_x;
	// 输入捕获信号的极性配置
	TIM_ICInitStructure.TIM_ICPolarity = GENERAL_CAPTURE_TIM_STRAT_ICPolarity;
	// 输入通道和捕获通道的映射关系，有直连和非直连两种
	TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;
	// 输入的需要被捕获的信号的分频系数
	TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	// 输入的需要被捕获的信号的滤波系数
	TIM_ICInitStructure.TIM_ICFilter = 0;
	// 定时器输入捕获初始化
	TIM_ICInit(GENERAL_CAPTURE_TIM, &TIM_ICInitStructure);
	
	// 清除更新和捕获中断标志位
    TIM_ClearFlag(GENERAL_CAPTURE_TIM, TIM_FLAG_Update|GENERAL_CAPTURE_TIM_IT_CCx);
    // 开启更新和捕获中断
    TIM_ITConfig (GENERAL_CAPTURE_TIM, TIM_IT_Update | GENERAL_CAPTURE_TIM_IT_CCx, ENABLE );

    // 使能计数器
    TIM_Cmd(GENERAL_CAPTURE_TIM, ENABLE);
}

void CAPTURE_TIM_Init(void)
{
	GENERAL_TIM_GPIO_Config();
	GENERAL_TIM_NVIC_Config();
	GENERAL_TIM_Mode_Config();		
}

```

### 输入捕获结构体分析

```c
/** 
  * @brief  TIM Input Capture Init structure definition  
  */

typedef struct
{

  uint16_t TIM_Channel;      /*!< Specifies the TIM channel.
                                  This parameter can be a value of @ref TIM_Channel */

  uint16_t TIM_ICPolarity;   /*!< Specifies the active edge of the input signal.
                                  This parameter can be a value of @ref TIM_Input_Capture_Polarity */

  uint16_t TIM_ICSelection;  /*!< Specifies the input.
                                  This parameter can be a value of @ref TIM_Input_Capture_Selection */

  uint16_t TIM_ICPrescaler;  /*!< Specifies the Input Capture Prescaler.
                                  This parameter can be a value of @ref TIM_Input_Capture_Prescaler */

  uint16_t TIM_ICFilter;     /*!< Specifies the input capture filter.
                                  This parameter can be a number between 0x0 and 0xF */
} TIM_ICInitTypeDef;
```
- 输入通道选择`TIM_Channel`

```c
/** @defgroup TIM_Channel 
  * @{
  */

#define TIM_Channel_1                      ((uint16_t)0x0000)
#define TIM_Channel_2                      ((uint16_t)0x0004)
#define TIM_Channel_3                      ((uint16_t)0x0008)
#define TIM_Channel_4                      ((uint16_t)0x000C)
```
- 输入捕获触发选择`TIM_ICPolarity`

```c
/** @defgroup TIM_Input_Capture_Polarity 
  * @{
  */

#define  TIM_ICPolarity_Rising             ((uint16_t)0x0000) //上升沿触发
#define  TIM_ICPolarity_Falling            ((uint16_t)0x0002) //下降沿触发
#define  TIM_ICPolarity_BothEdge           ((uint16_t)0x000A) //边沿跳变触发
```
- 输入捕获选择`TIM_ICSelection`

```c
/** @defgroup TIM_Input_Capture_Selection 
  * @{
  */

#define TIM_ICSelection_DirectTI           ((uint16_t)0x0001) /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to IC1, IC2, IC3 or IC4, respectively */ 
#define TIM_ICSelection_IndirectTI         ((uint16_t)0x0002) /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to IC2, IC1, IC4 or IC3, respectively. */
#define TIM_ICSelection_TRC                ((uint16_t)0x0003) /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to TRC. */
```
- 输入捕获预分频器`TIM_ICPrescaler`

```c
/** @defgroup TIM_Input_Capture_Prescaler 
  * @{
  */

#define TIM_ICPSC_DIV1                     ((uint16_t)0x0000) /*!< Capture performed each time an edge is detected on the capture input. *///无预分频器，捕获输入口上检测到的每一个边沿都触发一次捕获
#define TIM_ICPSC_DIV2                     ((uint16_t)0x0004) /*!< Capture performed once every 2 events. *///每2个事件触发一次捕获
#define TIM_ICPSC_DIV4                     ((uint16_t)0x0008) /*!< Capture performed once every 4 events. *///每4个事件触发一次捕获
#define TIM_ICPSC_DIV8                     ((uint16_t)0x000C) /*!< Capture performed once every 8 events. *///每8个事件触发一次捕获
```

- 输入捕获滤波器`TIM_ICFilter`

范围0-15

|ICxF[3:0]|采样频率|数字滤波器长度|
|:----:|:----:|:----:|
|0000|f<sub>DTS</sub>|无|
|0001|f<sub>ck_INT</sub>|2|
|0010|f<sub>ck_INT</sub>|4|
|0011|f<sub>ck_INT</sub>|8|
|0100|f<sub>DTS</sub>/2|6|
|0101|f<sub>DTS</sub>/2|8|
|0110|f<sub>DTS</sub>/4|6|
|0111|f<sub>DTS</sub>/4|8|
|1000|f<sub>DTS</sub>/8|6|
|1001|f<sub>DTS</sub>/8|8|
|1010|f<sub>DTS</sub>/16|5|
|1011|f<sub>DTS</sub>/16|6|
|1100|f<sub>DTS</sub>/16|8|
|1101|f<sub>DTS</sub>/32|5|
|1110|f<sub>DTS</sub>/32|6|
|1111|f<sub>DTS</sub>/32|8|

# 添加中断函数到`stm32f10x_it.c`
## 添加TIM头文件

`#include "my_capture.h"`
## 添加中断函数

```c
/**
  * @brief  This function handles TIM interrupt request.
  * @param  None
  * @retval None
  */
void GENERAL_CAPTURE_TIM_INT_FUN(void)
{
	// 当要被捕获的信号的周期大于定时器的最长定时时，定时器就会溢出，产生更新中断
    // 这个时候我们需要把这个最长的定时周期加到捕获信号的时间里面去
    if ( TIM_GetITStatus ( GENERAL_CAPTURE_TIM, TIM_IT_Update) != RESET )
    {
        TIM_ICUserValueStructure.Capture_Period ++;
        TIM_ClearITPendingBit ( GENERAL_CAPTURE_TIM, TIM_FLAG_Update );
    }

    // 上升沿捕获中断
    if ( TIM_GetITStatus (GENERAL_CAPTURE_TIM, GENERAL_CAPTURE_TIM_IT_CCx ) != RESET)
    {
        // 第一次捕获
        if ( TIM_ICUserValueStructure.Capture_StartFlag == 0 )
        {
            // 计数器清0
            TIM_SetCounter ( GENERAL_CAPTURE_TIM, 0 );
            // 自动重装载寄存器更新标志清0
            TIM_ICUserValueStructure.Capture_Period = 0;
            // 存捕获比较寄存器的值的变量的值清0
            TIM_ICUserValueStructure.Capture_CcrValue = 0;

            // 当第一次捕获到上升沿之后，就把捕获边沿配置为下降沿
            GENERAL_CAPTURE_TIM_OCxPolarityConfig_FUN(GENERAL_CAPTURE_TIM, GENERAL_CAPTURE_TIM_END_ICPolarity);
            // 开始捕获标准置1
            TIM_ICUserValueStructure.Capture_StartFlag = 1;
        }
        // 下降沿捕获中断
        else // 第二次捕获
        {
            // 获取捕获比较寄存器的值，这个值就是捕获到的高电平的时间的值
            TIM_ICUserValueStructure.Capture_CcrValue =
                GENERAL_CAPTURE_TIM_GetCapturex_FUN (GENERAL_CAPTURE_TIM);

            // 当第二次捕获到下降沿之后，就把捕获边沿配置为上升沿，好开启新的一轮捕获
            GENERAL_CAPTURE_TIM_OCxPolarityConfig_FUN(GENERAL_CAPTURE_TIM, GENERAL_CAPTURE_TIM_STRAT_ICPolarity );
            // 开始捕获标志清0
            TIM_ICUserValueStructure.Capture_StartFlag = 0;
            // 捕获完成标志置1
            TIM_ICUserValueStructure.Capture_FinishFlag = 1;
        }
        TIM_ClearITPendingBit (GENERAL_CAPTURE_TIM,GENERAL_CAPTURE_TIM_IT_CCx);
    }
}
```
## 捕获函数

|通道|捕获计数函数|更改捕获边沿函数|
|:----:|:----:|:----:|
|Channel|TIM_GetCapture1|TIM_OC1PolarityConfig|
|Channe2|TIM_GetCapture2|TIM_OC2PolarityConfig|
|Channe3|TIM_GetCapture3|TIM_OC3PolarityConfig|
|Channe4|TIM_GetCapture4|TIM_OC4PolarityConfig|

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
    freq=10000;
    duty1=50;
    duty2=40;
    duty3=30;
    duty4=20;
    printf( "\r\n Print current Temperature  \r\n");

    while (1)
    {
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
- 连接开发板串口
- 打开串口助手
- 按下按键1，可以看到串口发来的按键时间
