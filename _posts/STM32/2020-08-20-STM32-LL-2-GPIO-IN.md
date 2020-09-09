---
title: STM32底层LL库GPIO使用输入LL篇2
categories: STM32 CUBE LL GPIO
tags: STM32 CUBE LL GPIO
description: LL库GPIO输入
---
# 配置工程
- 与HAL库操作方式一样
- 生成代码

# 按键函数
## 添加按键宏和声明函数
- 有一点要注意`GPIO_Pin`在**LL**库里是 **`32`** 位的,而在**HAL**库和**SPL**库均是 **`16`** 位的

```c
#define KEY_ON	1
#define KEY_OFF	0
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint32_t GPIO_Pin);
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
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint32_t GPIO_Pin)
{
    /*是否有按键按下 */
    if(LL_GPIO_IsInputPinSet(GPIOx,GPIO_Pin) == KEY_ON )
    {
        /*等待按键释放 */
        while(LL_GPIO_IsInputPinSet(GPIOx,GPIO_Pin) == KEY_ON);
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
## 下载调试
- 编译并下载到开发板
- 点击按键1，LED1进行反转输出
- 点击按键2，LED2进行反转输出
- 结果和HAL库写的程序输出效果一致