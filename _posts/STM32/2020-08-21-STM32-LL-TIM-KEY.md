---
title: STM32底层LL库TIM做按键消抖
categories: STM32 CUBE LL TIM GPIO
tags: STM32 CUBE LL TIM GPIO
description: LL库TIM做按键消抖
---
# 修改按键检测函数

```c
/* USER CODE BEGIN 2 */
uint8_t key_state1=0,key_state2=0;
uint32_t key_time1=0,key_time2=0;
/**
  * @brief  KEY_SACN
  * @param  GPIOx
  *
  * @retval KEY
  */
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint32_t GPIO_Pin)
{
  if(GPIOx==KEY1_GPIO_Port && GPIO_Pin==KEY1_Pin)
    {
        /*检测是否有按键按下 */
        if(key_state1==0)
        {
            if (LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_ON)
            {
                key_state1=1;//开始消抖
            }
        }
        else if(key_state1==2)//消抖完成
        {
            if (LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_OFF)
            {
                key_state1=0;
                key_time1=0;
                return KEY_ON;
            }
        }
        else if(LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_OFF)
        {
            key_state1=0;
            key_time1=0;
        }
        return KEY_OFF;
    } else if(GPIOx==KEY2_GPIO_Port && GPIO_Pin==KEY2_Pin)
    {
        /*检测是否有按键按下 */
        if(key_state2==0)
        {
            if (LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_ON)
            {
                key_state2=1;//开始消抖
            }
        }
        else if(key_state2==2)//消抖完成
        {
            if (LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_OFF)
            {
                key_state2=0;
                key_time2=0;
                return KEY_ON;
            }
        }
        else if(LL_GPIO_IsInputPinSet(GPIOx, GPIO_Pin) == KEY_OFF)
        {
            key_state2=0;
            key_time2=0;
        }
        return KEY_OFF;
    }
    return KEY_OFF;   
}
/* USER CODE END 2 */
```

# 修改定时器中断函数
- 在`stm32f1xx_it.c`添加外部变量

```c
extern uint32_t key_time1,key_time2;
extern uint8_t key_state1,key_state2;
#define KEY_BUFFETING_TIME 20
```
- 修改中断函数

```c
/**
  * @brief This function handles TIM6 global interrupt.
  */
void TIM6_IRQHandler(void)
{
    /* USER CODE BEGIN TIM6_IRQn 0 */
    if(LL_TIM_IsActiveFlag_UPDATE(TIM6))
    {
        LL_TIM_ClearFlag_UPDATE(TIM6);
        time++;
        if(key_state1==1)
        {
            if(++key_time1>=KEY_BUFFETING_TIME)
            {
                key_state1=2;
            }
        }
        if(key_state2==1)
        {
            if(++key_time2>=KEY_BUFFETING_TIME)
            {
                key_state2=2;
            }
        }
    }
/* USER CODE END TIM6_IRQn 0 */
/* USER CODE BEGIN TIM6_IRQn 1 */

/* USER CODE END TIM6_IRQn 1 */
}
```
# 调试
- 编译之后下载到开发板
- 点击按键1，LED1进行反转输出
- 点击按键2，LED2进行反转输出
- 和HAL库结果一致