---
title: STM32底层LL库TIMPWM输入测量LL篇11
categories: STM32 CUBE LL TIM PWM
tags: STM32 CUBE LL TIM PWM DUTY
description: LL库TIMPWM输入测量
---
# 配置TIM
- 和HAL库配置一样
- 在高级设置里为**TIM8**选择`LL`库
- 生成代码

# 代码移植
## 修改定时器中断
- 在`stm32f10x_it.c`文件添加外部变量

```c
extern __IO uint8_t IC_State;
extern __IO float DutyCycle;
extern __IO float Frequency;
```
- 修改中断

```c
/**
  * @brief This function handles TIM8 capture compare interrupt.
  */
void TIM8_CC_IRQHandler(void)
{
    /* USER CODE BEGIN TIM8_CC_IRQn 0 */
    if(LL_TIM_IsActiveFlag_CC1(TIM8))
    {
        LL_TIM_ClearFlag_CC1(TIM8);
        /* 获取输入捕获值 */
        uint16_t IC1Value =  LL_TIM_IC_GetCaptureCH1(TIM8);
        uint16_t IC2Value =  LL_TIM_IC_GetCaptureCH2(TIM8);
        // 注意：捕获寄存器CCR1和CCR2的值在计算占空比和频率的时候必须加1
        if (IC1Value != 0)
        {
            /* 占空比计算 */
            DutyCycle = (float)((IC2Value+1) * 100) / (IC1Value+1);

            /* 频率计算 */
            Frequency = (72000000/(71+1))/(float)(IC1Value+1);
            IC_State=1;
        }
        else
        {
            DutyCycle = 0;
            Frequency = 0;
        }

    }
    /* USER CODE END TIM8_CC_IRQn 0 */
    /* USER CODE BEGIN TIM8_CC_IRQn 1 */

    /* USER CODE END TIM8_CC_IRQn 1 */
}
```
## 开始捕获
- 在`main.c`文件添加变量

```c
__IO uint8_t IC_State;
__IO float DutyCycle;
__IO float Frequency;
```

- 在主函数初始化添加TIM启动

```c
    LL_TIM_EnableCounter(TIM8);//使能定时器
    LL_TIM_CC_EnableChannel(TIM8,LL_TIM_CHANNEL_CH1|LL_TIM_CHANNEL_CH2);//使能通道1和通道2
    LL_TIM_EnableIT_CC1(TIM8);//使能捕获中断
```
- 主循环添加捕获处理

```c
        if(IC_State)
        {
            IC_State=0;
            printf("DUTY:%0.2f%%   FREQ:%0.2fHz\n",DutyCycle,Frequency);
        }
```
## 下载调试
- 编译之后下载到开发板
- 连接PC6到PA6
- 连接开发板串口
- 打开串口助手
- 可以看到频率和占空比和示波器看到的一样
- 和HAL库结果一致
