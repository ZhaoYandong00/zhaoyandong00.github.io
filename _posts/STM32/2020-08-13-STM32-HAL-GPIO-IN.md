---
title: STM32硬件层HAL库GPIO使用输入篇
categories: STM32 CUBE HAL GPIO
tags: STM32 CUBE HAL GPIO
description: HAL库GPIO输入
---
# 配置IO
- 打开CUBE工程
- 在**`Pinout View`**选择开发板上LED灯对应管脚**PA0**,**PC13**
- 设置为`GPIO_input`
- 在**`System Core`**->**`GPIO`**配置
- **PA0**标号设置**KEY1**,**PC13**标号设置**KEY2**
- 代码生成

# 按键函数
## 添加按键宏和声明函数

```c
#define KEY_ON	1
#define KEY_OFF	0
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin);
```
## 定义函数

```c
/*
* 函数名：Key_Scan
* 描述  ：检测是否有按键按下
* 输入  ：GPIOx：x  A，B，C，D，E
*		     GPIO_Pin：待读取的端口位
* 输出  ：KEY_OFF(没按下)、KEY_ON（按下）
*/
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin)
{
    /*是否有按键按下 */
    if(HAL_GPIO_ReadPin(GPIOx,GPIO_Pin) == KEY_ON )
    {
        /*等待按键释放 */
        while(HAL_GPIO_ReadPin(GPIOx,GPIO_Pin) == KEY_ON);
        return 	KEY_ON;
    }
    else
        return KEY_OFF;
}
```

# 调试
## 主程序修改

```c
if( Key_Scan(KEY1_GPIO_Port,KEY1_Pin) == KEY_ON  )
{
    /*LED1反转*/
    LED1_TOGGLE;
}

if( Key_Scan(KEY2_GPIO_Port,KEY2_Pin) == KEY_ON  )
{
    LED2_TOGGLE;
}
```
## 调试
- 编译并下载到开发板
- 点击按键1，LED1进行反转输出
- 点击按键2，LED2进行反转输出
- 结果和SPL库写的程序输出效果一致