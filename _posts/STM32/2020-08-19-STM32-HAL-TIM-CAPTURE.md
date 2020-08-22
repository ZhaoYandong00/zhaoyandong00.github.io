---
title: STM32硬件层HAL库TIM输入捕获HAL篇11
categories: STM32 CUBE HAL TIM CAPTURE
tags: STM32 CUBE HAL TIM CAPTURE
description: HAL库TIM输入捕获
---
# 配置TIM
- 打开CUBE工程

## 在**`Pinout View`**配置
- 设置**PA0**为`Reset_State`,以保证TIM5的通道1可设置输入捕获

## 在**`Timers`**->**`TIM5`**配置
- 选择`Internal Clock`
- 选择**Channel1**——`Input Capture direct mode`
- **`Parameter Settings`**配置
- 配置预分频器72分频——`71`，自动重载数值——`65535`，计数模式——`UP`，自动重载预装载——`Disable` **CKD**-`No Division`
- 配置**Channel 1**:极性选择——`Rising Edge`, IC选择——`Direct`,其余默认
- **`NVIC Settings`**使能中断


## 在**`System Core`**->**`GPIO`**配置
- **`TIM`**下**PA0**标号设置**KEY1**

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

#  TIM函数
## 输入捕获结构体分析

```c
/**
  * @brief  TIM Input Capture Configuration Structure definition
  */
typedef struct
{
  uint32_t  ICPolarity;  /*!< Specifies the active edge of the input signal.
                              This parameter can be a value of @ref TIM_Input_Capture_Polarity */

  uint32_t ICSelection;  /*!< Specifies the input.
                              This parameter can be a value of @ref TIM_Input_Capture_Selection */

  uint32_t ICPrescaler;  /*!< Specifies the Input Capture Prescaler.
                              This parameter can be a value of @ref TIM_Input_Capture_Prescaler */

  uint32_t ICFilter;     /*!< Specifies the input capture filter.
                              This parameter can be a number between Min_Data = 0x0 and Max_Data = 0xF */
} TIM_IC_InitTypeDef;
```
- 输入捕获触发选择`ICPolarity`

```c
/** @defgroup TIM_Input_Capture_Polarity TIM Input Capture Polarity
  * @{
  */
#define  TIM_ICPOLARITY_RISING             TIM_INPUTCHANNELPOLARITY_RISING      /*!< Capture triggered by rising edge on timer input                  *///上升沿
#define  TIM_ICPOLARITY_FALLING            TIM_INPUTCHANNELPOLARITY_FALLING     /*!< Capture triggered by falling edge on timer input                 *///下降沿
#define  TIM_ICPOLARITY_BOTHEDGE           TIM_INPUTCHANNELPOLARITY_BOTHEDGE    /*!< Capture triggered by both rising and falling edges on timer input*///边沿
```
- 输入捕获选择`ICSelection`

```c
/** @defgroup TIM_Input_Capture_Selection TIM Input Capture Selection
  * @{
  */
#define TIM_ICSELECTION_DIRECTTI           TIM_CCMR1_CC1S_0                     /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to IC1, IC2, IC3 or IC4, respectively *///CCx通道被配置为输入，ICx映射在TIx上
#define TIM_ICSELECTION_INDIRECTTI         TIM_CCMR1_CC1S_1                     /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to IC2, IC1, IC4 or IC3, respectively *///CCx通道被配置为输入，IC1,IC2,IC3,IC4映射在TI2,TI1,TI4,TI3上
#define TIM_ICSELECTION_TRC                TIM_CCMR1_CC1S                       /*!< TIM Input 1, 2, 3 or 4 is selected to be connected to TRC *///：CCx通道被配置为输入，ICx映射在TRC上。此模式仅工作在内部触发器输入被选中时(由TIMx_SMCR寄存器的TS位选择)
```
- 输入捕获预分频器`ICPrescaler`

```c
/** @defgroup TIM_Input_Capture_Prescaler TIM Input Capture Prescaler
  * @{
  */
#define TIM_ICPSC_DIV1                     0x00000000U                          /*!< Capture performed each time an edge is detected on the capture input *///无预分频器，捕获输入口上检测到的每一个边沿都触发一次捕获
#define TIM_ICPSC_DIV2                     TIM_CCMR1_IC1PSC_0                   /*!< Capture performed once every 2 events                                *///每2个事件触发一次捕获
#define TIM_ICPSC_DIV4                     TIM_CCMR1_IC1PSC_1                   /*!< Capture performed once every 4 events                                *///每4个事件触发一次捕获
#define TIM_ICPSC_DIV8                     TIM_CCMR1_IC1PSC                     /*!< Capture performed once every 8 events                                *///每8个事件触发一次捕获
```
- 输入捕获滤波器`ICFilter`

范围0-15

## 常用函数

- 设置当前计数值`__HAL_TIM_SET_COUNTER(__HANDLE__, __COUNTER__)`
- 得到当前计数值`__HAL_TIM_GET_COUNTER(__HANDLE__)`
- 设置通道输入捕获极性`__HAL_TIM_SET_CAPTUREPOLARITY(__HANDLE__, __CHANNEL__, __POLARITY__)`

注：HAL库使用这个函数会报错，是因为调用的`TIM_RESET_CAPTUREPOLARITY`在第一行多了一个`)`,改成下边这样

```c
#define TIM_RESET_CAPTUREPOLARITY(__HANDLE__, __CHANNEL__) \
  (((__CHANNEL__) == TIM_CHANNEL_1) ? ((__HANDLE__)->Instance->CCER &= ~(TIM_CCER_CC1P | TIM_CCER_CC1NP)) :\
   ((__CHANNEL__) == TIM_CHANNEL_2) ? ((__HANDLE__)->Instance->CCER &= ~(TIM_CCER_CC2P | TIM_CCER_CC2NP)) :\
   ((__CHANNEL__) == TIM_CHANNEL_3) ? ((__HANDLE__)->Instance->CCER &= ~(TIM_CCER_CC3P)) :\
   ((__HANDLE__)->Instance->CCER &= ~(TIM_CCER_CC4P)))
```

如想永久解决此问题，就在CUBE库文件里进行修改，这样的以后生成的都是修正过得

默认仓库地址：

`C:\Users\Administrator\STM32Cube\Repository\STM32Cube_FW_F1_V1.8.0`

在`Drivers\STM32F1xx_HAL_Driver\Inc`文件夹找到`stm32f1xx_hal_tim.h`

该函数在的1744-1749行，去掉1745行的最后一个`)`


- 获取通道捕获值`uint32_t HAL_TIM_ReadCapturedValue(TIM_HandleTypeDef *htim, uint32_t Channel)`

- 开启输入捕获`HAL_StatusTypeDef HAL_TIM_IC_Start(TIM_HandleTypeDef *htim, uint32_t Channel)`
- 关闭输入捕获`HAL_StatusTypeDef HAL_TIM_IC_Stop(TIM_HandleTypeDef *htim, uint32_t Channel)`

- 启动输入捕获带中断`HAL_StatusTypeDef HAL_TIM_IC_Start_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`

捕获到会调用中断，调用捕获完成回调函数`HAL_TIM_IC_CaptureCallback`
- 关闭输入捕获`HAL_StatusTypeDef HAL_TIM_IC_Stop_IT(TIM_HandleTypeDef *htim, uint32_t Channel)`

- DMA启动输入捕获`HAL_StatusTypeDef HAL_TIM_IC_Start_DMA(TIM_HandleTypeDef *htim, uint32_t Channel, uint32_t *pData, uint16_t Length)`

DMA设置必须设置成外设到存储器,捕获后会调用捕获完成回调函数`HAL_TIM_IC_CaptureCallback`
- 关闭输入捕获`HAL_StatusTypeDef HAL_TIM_IC_Stop_DMA(TIM_HandleTypeDef *htim, uint32_t Channel)`

# 代码移植
## 使能计数器中断
- 在`HAL_TIM_Base_MspInit`函数添加

```c
  /* USER CODE BEGIN TIM5_MspInit 1 */
   __HAL_TIM_ENABLE_IT(tim_baseHandle,TIM_IT_UPDATE);
  /* USER CODE END TIM5_MspInit 1 */
```
- 在`tim.h`文件添加结构体定义

```c
// 定时器输入捕获用户自定义变量结构体声明
typedef struct
{
    uint8_t   Capture_FinishFlag;   // 捕获结束标志位
    uint8_t   Capture_StartFlag;    // 捕获开始标志位
    uint16_t  Capture_CcrValue;     // 捕获寄存器的值
    uint16_t  Capture_Period;       // 自动重装载寄存器更新标志
} TIM_ICUserValueTypeDef;
```
## 开始捕获
- 在`main.c`文件添加变量

```c
// 定时器输入捕获用户自定义变量结构体定义
TIM_ICUserValueTypeDef TIM_ICUserValueStructure = {0,0,0,0};
```

- 在主函数初始化添加TIM启动

```c
HAL_TIM_IC_Start_IT(&htim5,TIM_CHANNEL_1);
uint32_t capture_time;
uint32_t TIM_PscCLK = 72000000 / (71+1);
```

- 主循环添加捕获处理

```c
        if(TIM_ICUserValueStructure.Capture_FinishFlag == 1)
        {
            // 计算高电平时间的计数器的值
            capture_time = TIM_ICUserValueStructure.Capture_Period * (65535+1) +
                           (TIM_ICUserValueStructure.Capture_CcrValue+1);

            // 打印高电平脉宽时间
            printf ( "\r\n duty time:%d.%d s\r\n",capture_time/TIM_PscCLK,capture_time%TIM_PscCLK );

            TIM_ICUserValueStructure.Capture_FinishFlag = 0;
        }
```
- 添加捕获完成回调函数

```c
/**
  * @brief  Input Capture callback in non-blocking mode
  * @param  htim TIM IC handle
  * @retval None
  */
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance==TIM5)
    {
        // 第一次捕获
        if ( TIM_ICUserValueStructure.Capture_StartFlag == 0 )
        {
            // 计数器清0
            __HAL_TIM_SET_COUNTER(htim,0);
            // 自动重装载寄存器更新标志清0
            TIM_ICUserValueStructure.Capture_Period = 0;
            // 存捕获比较寄存器的值的变量的值清0
            TIM_ICUserValueStructure.Capture_CcrValue = 0;
            // 当第一次捕获到上升沿之后，就把捕获边沿配置为下降沿
            __HAL_TIM_SET_CAPTUREPOLARITY(htim,TIM_CHANNEL_1,TIM_INPUTCHANNELPOLARITY_FALLING);
            // 开始捕获标准置1
            TIM_ICUserValueStructure.Capture_StartFlag = 1;
        }
        // 下降沿捕获中断
        else // 第二次捕获
        {
            // 获取捕获比较寄存器的值，这个值就是捕获到的高电平的时间的值
            TIM_ICUserValueStructure.Capture_CcrValue =  HAL_TIM_ReadCapturedValue(htim,TIM_CHANNEL_1);

            // 当第二次捕获到下降沿之后，就把捕获边沿配置为上升沿，好开启新的一轮捕获
            __HAL_TIM_SET_CAPTUREPOLARITY(htim,TIM_CHANNEL_1,TIM_INPUTCHANNELPOLARITY_RISING);
            // 开始捕获标志清0
            TIM_ICUserValueStructure.Capture_StartFlag = 0;
            // 捕获完成标志置1
            TIM_ICUserValueStructure.Capture_FinishFlag = 1;
        }
    }
}
```

- 修改延时完成回调函数

```c
/**
  * @brief  Period elapsed callback in non-blocking mode
  * @param  htim TIM handle
  * @retval None
  */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance==TIM6)
    {
        time++;
    }
    if(htim->Instance==TIM5)
    {
        if(TIM_ICUserValueStructure.Capture_StartFlag==1)
        {
            TIM_ICUserValueStructure.Capture_Period ++;
        }
    }
}
```

## 下载调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 按下按键1，可以看到串口发来的按键时间
- 和SPL库结果一致