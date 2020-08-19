---
title: STM32标准外设库计时器做按键消抖
categories: STM32 SPL TIM GPIO
tags: STM32 SPL TIM GPIO
description: SPL库计时器做按键消抖
---
# 修改按键检测函数

```c
uint8_t key_state1=0,key_state2=0;
uint32_t key_time1=0,key_time2=0;
/*
* 函数名：Key_Scan
* 描述  ：检测是否有按键按下
* 输入  ：GPIOx：x 可以是 A，B，C，D或者 E
*		     GPIO_Pin：待读取的端口位
* 输出  ：KEY_OFF(没按下按键)、KEY_ON（按下按键）
*/
uint8_t Key_Scan(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)
{
    if(GPIOx==KEY1_GPIO_PORT && GPIO_Pin==KEY1_GPIO_PIN)
    {
        /*检测是否有按键按下 */
        if(key_state1==0)
        {
            if (GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_ON)
            {
                key_state1=1;//开始消抖
            }
        }
        else if(key_state1==2)//消抖完成
        {
            if (GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_OFF)
            {
                key_state1=0;
                key_time1=0;
                return KEY_ON;
            }
        }
        else if(GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_OFF)
        {
            key_state1=0;
            key_time1=0;
        }
        return KEY_OFF;
    } else if(GPIOx==KEY2_GPIO_PORT && GPIO_Pin==KEY2_GPIO_PIN)
    {
        /*检测是否有按键按下 */
        if(key_state2==0)
        {
            if (GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_ON)
            {
                key_state2=1;//开始消抖
            }
        }
        else if(key_state2==2)//消抖完成
        {
            if (GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_OFF)
            {
                key_state2=0;
                key_time2=0;
                return KEY_ON;
            }
        }
        else if(GPIO_ReadInputDataBit(GPIOx, GPIO_Pin) == KEY_OFF)
        {
            key_state2=0;
            key_time2=0;
        }
        return KEY_OFF;
    }
    return KEY_OFF;
}
```

# 修改定时器中断函数

```c
extern uint32_t key_time1,key_time2;
extern uint8_t key_state1,key_state2;
#define KEY_BUFFETING_TIME 20
/**
  * @brief  This function handles TIM interrupt request.
  * @param  None
  * @retval None
  */
void  MY_TIM_IRQHandler (void)
{
    if ( TIM_GetITStatus( MY_TIM, TIM_IT_Update) != RESET )
    {
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
        TIM_ClearITPendingBit(MY_TIM , TIM_FLAG_Update);
    }
}
```

# 修改主函数

- 还是GPIO输入时用的代码，放到`main`函数的`while`循环里
```c
        if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON  )
        {
            /*LED1反转*/
            LED1_TOGGLE;
        }

        if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON  )
        {
            /*LED2反转*/
            LED2_TOGGLE;
        }
```

# 调试
- 编译之后下载到开发板
- 点击按键1，LED1进行反转输出
- 点击按键2，LED2进行反转输出