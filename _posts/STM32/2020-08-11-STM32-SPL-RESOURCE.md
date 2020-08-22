---
title: STM32标准外设库资源总结SPL篇17
categories: STM32 SPL
tags: STM32 SPL
description: SPL库资源总结
---
# 启动模式

<table >
<thead>
<tr>
<th style="text-align: center" colspan="2">启动模式选择引脚</th>
<th style="text-align: center" rowspan="2">启动模式</th>
<th style="text-align: center" rowspan="2">说明</th>
</tr>
<tr>
<th style="text-align: center">BOOT1</th>
<th style="text-align: center">BOOT0</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center">X</td>
<td style="text-align: center">0</td>
<td style="text-align: center">主闪存存储器</td>
<td style="text-align: center">主闪存存储器被选为启动区域</td>
</tr>
<tr>
<td style="text-align: center">0</td>
<td style="text-align: center">1</td>
<td style="text-align: center">系统存储器</td>
<td style="text-align: center">系统存储器被选为启动区域</td>
</tr>
<tr>
<td style="text-align: center">1X</td>
<td style="text-align: center">1</td>
<td style="text-align: center">内置SRAM</td>
<td style="text-align: center">内置SRAM被选为启动区域</td>
</tr>
</tbody>
</table>

# 低功耗模式

<table >
<thead>
<tr>
<th style="text-align: center" >模式</th>
<th style="text-align: center" >进入</th>
<th style="text-align: center" >唤醒</th>
<th style="text-align: center" >对1.8V区域时钟的影响</th>
<th style="text-align: center" >对VDD区域时钟的影响</th>
<th style="text-align: center" >电压调节器</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">睡眠(SLEEP-NOW或SLEEP-ON-EXIT)</th>
<td style="text-align: center">WFI</td>
<td style="text-align: center">任一中断</td>
<td style="text-align: center" rowspan="2">CPU时钟关，对其他时钟和ADC时钟无影响</td>
<td style="text-align: center" rowspan="2">无</td>
<td style="text-align: center" rowspan="2">开</td>
</tr>
<tr>
<td style="text-align: center">WFI</td>
<td style="text-align: center">唤醒事件</td>
</tr>
<tr>
<th style="text-align: center">停机</th>
<td style="text-align: center">PDDS和LPDS位+SLEEPDEEP位+WFI或WFE</td>
<td style="text-align: center">任一外部中断(在外部中断寄存器中设置)</td>
<td style="text-align: center" rowspan="2">关闭所有1.8V区域的时钟</td>
<td style="text-align: center" rowspan="2">HSI 和HSE的振荡器关闭</td>
<td style="text-align: center" >开启或处于低功耗模式(依据电源控制寄存器(PWR_CR)的设定)</td>
</tr>
<tr>
<th style="text-align: center">待机</th>
<td style="text-align: center">PDDS位 +SLEEPDEEP位+WFI或WFE</td>
<td style="text-align: center">WKUP引脚的上升沿、RTC闹钟事件、NRST引脚上的外部复位、IWDG复位</td>
<td style="text-align: center">关</td>
</tr>
</tbody>
</table>

# RCC

<table >
<thead>
<tr>
<th style="text-align: center" rowspan="2">时钟</th>
<th style="text-align: center" rowspan="2">使能函数</th>
<th style="text-align: center" colspan="2">小容量，中容量和大容量</th>
<th style="text-align: center" colspan="2" >互联型</th>
</tr>
<tr>
<th style="text-align: center" >外设</th>
<th style="text-align: center" >参数</th>
<th style="text-align: center" >外设</th>
<th style="text-align: center" >参数</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="9">AHB</th>
<td style="text-align: center" rowspan="9">RCC_AHBPeriphClockCmd</td>
<td style="text-align: center">DMA1</td>
<td style="text-align: center">RCC_AHBPeriph_DMA1</td>
<td style="text-align: center">DMA1</td>
<td style="text-align: center">RCC_AHBPeriph_DMA1</td>
</tr>
<tr>
<td style="text-align: center">DMA2</td>
<td style="text-align: center">RCC_AHBPeriph_DMA2</td>
<td style="text-align: center">DMA2</td>
<td style="text-align: center">RCC_AHBPeriph_DMA2</td>
</tr>
<tr>
<td style="text-align: center">SRAM</td>
<td style="text-align: center">RCC_AHBPeriph_SRAM</td>
<td style="text-align: center">SRAM</td>
<td style="text-align: center">RCC_AHBPeriph_SRAM</td>
</tr>
<tr>
<td style="text-align: center">FLITF</td>
<td style="text-align: center">RCC_AHBPeriph_FLITF</td>
<td style="text-align: center">FLITF</td>
<td style="text-align: center">RCC_AHBPeriph_FLITF</td>
</tr>
<tr>
<td style="text-align: center">CRC</td>
<td style="text-align: center">RCC_AHBPeriph_CRC</td>
<td style="text-align: center">CRC</td>
<td style="text-align: center">RCC_AHBPeriph_CRC</td>
</tr>
<tr>
<td style="text-align: center">FSMC</td>
<td style="text-align: center">RCC_AHBPeriph_FSMC</td>
<td style="text-align: center">OTG_FS</td>
<td style="text-align: center">RCC_AHBPeriph_OTG_FS</td>
</tr>
<tr>
<td style="text-align: center">SDIO</td>
<td style="text-align: center">RCC_AHBPeriph_SDIO</td>
<td style="text-align: center">ETH_MAC</td>
<td style="text-align: center">RCC_AHBPeriph_ETH_MAC</td>
</tr>
<tr>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center">ETH_MAC_Tx</td>
<td style="text-align: center">RCC_AHBPeriph_ETH_MAC_Tx</td>
</tr>
<tr>
<td style="text-align: center"></td>
<td style="text-align: center"></td>
<td style="text-align: center">ETH_MAC_Rx</td>
<td style="text-align: center">RCC_AHBPeriph_ETH_MAC_Rx</td>
</tr>
<tr>
<th style="text-align: center" rowspan="24">APB1</th>
<td style="text-align: center" rowspan="24">RCC_APB1PeriphClockCmd</td>
<td style="text-align: center">TIM2</td>
<td style="text-align: center">RCC_APB1Periph_TIM2</td>
</tr>
<tr>
<td style="text-align: center">TIM3</td>
<td style="text-align: center">RCC_APB1Periph_TIM3</td>
</tr>
<tr>
<td style="text-align: center">TIM4</td>
<td style="text-align: center">RCC_APB1Periph_TIM4</td>
</tr>
<tr>
<td style="text-align: center">TIM5</td>
<td style="text-align: center">RCC_APB1Periph_TIM5</td>
</tr>
<tr>
<td style="text-align: center">TIM6</td>
<td style="text-align: center">RCC_APB1Periph_TIM6</td>
</tr>
<tr>
<td style="text-align: center">TIM7</td>
<td style="text-align: center">RCC_APB1Periph_TIM7</td>
</tr>
<tr>
<td style="text-align: center">WWDG</td>
<td style="text-align: center">RCC_APB1Periph_WWDG</td>
</tr>
<tr>
<td style="text-align: center">SPI2</td>
<td style="text-align: center">RCC_APB1Periph_SPI2</td>
</tr>
<tr>
<td style="text-align: center">SPI3</td>
<td style="text-align: center">RCC_APB1Periph_SPI3</td>
</tr>
<tr>
<td style="text-align: center">USART2</td>
<td style="text-align: center">RCC_APB1Periph_USART2</td>
</tr>
<tr>
<td style="text-align: center">USART3</td>
<td style="text-align: center">RCC_APB1Periph_USART3</td>
</tr>
<tr>
<td style="text-align: center">USART4</td>
<td style="text-align: center">RCC_APB1Periph_USART4</td>
</tr>
<tr>
<td style="text-align: center">USART5</td>
<td style="text-align: center">RCC_APB1Periph_USART5</td>
</tr>
<tr>
<td style="text-align: center">I2C1</td>
<td style="text-align: center">RCC_APB1Periph_I2C1</td>
</tr>
<tr>
<td style="text-align: center">I2C2</td>
<td style="text-align: center">RCC_APB1Periph_I2C2</td>
</tr>
<tr>
<td style="text-align: center">USB</td>
<td style="text-align: center">RCC_APB1Periph_USB</td>
</tr>
<tr>
<td style="text-align: center">CAN1</td>
<td style="text-align: center">RCC_APB1Periph_CAN1</td>
</tr>
<tr>
<td style="text-align: center">BKP</td>
<td style="text-align: center">RCC_APB1Periph_BKP</td>
</tr>
<tr>
<td style="text-align: center">PWR</td>
<td style="text-align: center">RCC_APB1Periph_PWR</td>
</tr>
<tr>
<td style="text-align: center">DAC</td>
<td style="text-align: center">RCC_APB1Periph_DAC</td>
</tr>
<tr>
<td style="text-align: center">CEC</td>
<td style="text-align: center">RCC_APB1Periph_CEC</td>
</tr>
<tr>
<td style="text-align: center">TIM12</td>
<td style="text-align: center">RCC_APB1Periph_TIM12</td>
</tr>
<tr>
<td style="text-align: center">TIM13</td>
<td style="text-align: center">RCC_APB1Periph_TIM13</td>
</tr>
<tr>
<td style="text-align: center">TIM14</td>
<td style="text-align: center">RCC_APB1Periph_TIM14</td>
</tr>
<tr>
<th style="text-align: center" rowspan="21">APB2</th>
<td style="text-align: center" rowspan="21">RCC_APB2PeriphClockCmd</td>
<td style="text-align: center">AFIO</td>
<td style="text-align: center">RCC_APB2Periph_AFIO</td>
</tr>
<tr>
<td style="text-align: center">GPIOA</td>
<td style="text-align: center">RCC_APB2Periph_GPIOA</td>
</tr>
<tr>
<td style="text-align: center">GPIOB</td>
<td style="text-align: center">RCC_APB2Periph_GPIOB</td>
</tr>
<tr>
<td style="text-align: center">GPIOC</td>
<td style="text-align: center">RCC_APB2Periph_GPIOC</td>
</tr>
<tr>
<td style="text-align: center">GPIOD</td>
<td style="text-align: center">RCC_APB2Periph_GPIOD</td>
</tr>
<tr>
<td style="text-align: center">GPIOE</td>
<td style="text-align: center">RCC_APB2Periph_GPIOE</td>
</tr>
<tr>
<td style="text-align: center">GPIOF</td>
<td style="text-align: center">RCC_APB2Periph_GPIOF</td>
</tr>
<tr>
<td style="text-align: center">GPIOG</td>
<td style="text-align: center">RCC_APB2Periph_GPIOG</td>
</tr>
<tr>
<td style="text-align: center">ADC1</td>
<td style="text-align: center">RCC_APB2Periph_ADC1</td>
</tr>
<tr>
<td style="text-align: center">ADC2</td>
<td style="text-align: center">RCC_APB2Periph_ADC2</td>
</tr>
<tr>
<td style="text-align: center">TIM1</td>
<td style="text-align: center">RCC_APB2Periph_TIM1</td>
</tr>
<tr>
<td style="text-align: center">SPI1</td>
<td style="text-align: center">RCC_APB2Periph_SPI1</td>
</tr>
<tr>
<td style="text-align: center">TIM8</td>
<td style="text-align: center">RCC_APB2Periph_TIM8</td>
</tr>
<tr>
<td style="text-align: center">USART1</td>
<td style="text-align: center">RCC_APB2Periph_USART1</td>
</tr>
<tr>
<td style="text-align: center">ADC3</td>
<td style="text-align: center">RCC_APB2Periph_ADC3</td>
</tr>
<tr>
<td style="text-align: center">TIM15</td>
<td style="text-align: center">RCC_APB2Periph_TIM15</td>
</tr>
<tr>
<td style="text-align: center">TIM16</td>
<td style="text-align: center">RCC_APB2Periph_TIM16</td>
</tr>
<tr>
<td style="text-align: center">TIM17</td>
<td style="text-align: center">RCC_APB2Periph_TIM17</td>
</tr>
<tr>
<td style="text-align: center">TIM9</td>
<td style="text-align: center">RCC_APB2Periph_TIM9</td>
</tr>
<tr>
<td style="text-align: center">TIM10</td>
<td style="text-align: center">RCC_APB2Periph_TIM10</td>
</tr>
<tr>
<td style="text-align: center">TIM11</td>
<td style="text-align: center">RCC_APB2Periph_TIM11</td>
</tr>
</tbody>
</table>

# GPIO
## 端口位配置

<table >
<thead>
<tr>
<th style="text-align: center" colspan="2">配置模式</th>
<th style="text-align: center" >CNF1</th>
<th style="text-align: center" >CNF0</th>
<th style="text-align: center" >MODE1</th>
<th style="text-align: center" >MODE0</th>
<th style="text-align: center" >PxODR</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" rowspan="2">通用输出</td>
<td style="text-align: center">推挽(Push-Pull)</td>
<td style="text-align: center" rowspan="2">0</td>
<td style="text-align: center" >0</td>
<td style="text-align: center" colspan="2" rowspan="4">01<br/>10<br/>11</td>
<td style="text-align: center" >0或1</td>
</tr>
<tr>
<td style="text-align: center">推挽(Push-Pull)</td>
<td style="text-align: center" >1</td>
<td style="text-align: center" >0或1</td>
</tr>
<tr>
<td style="text-align: center" rowspan="2">复用功能输出</td>
<td style="text-align: center">推挽(Push-Pull)</td>
<td style="text-align: center" rowspan="2">1</td>
<td style="text-align: center" >0</td>
<td style="text-align: center" >不使用</td>
</tr>
<tr>
<td style="text-align: center">开漏(Open-Drain)</td>
<td style="text-align: center" >1</td>
<td style="text-align: center" >不使用</td>
</tr>
<tr>
<td style="text-align: center" rowspan="4">输入</td>
<td style="text-align: center">模拟输入</td>
<td style="text-align: center" rowspan="2">0</td>
<td style="text-align: center" >0</td>
<td style="text-align: center" colspan="2" rowspan="4">00</td>
<td style="text-align: center" >不使用</td>
</tr>
<tr>
<td style="text-align: center">浮空输入</td>
<td style="text-align: center" >1</td>
<td style="text-align: center" >不使用</td>
</tr>
<tr>
<td style="text-align: center">下拉输入</td>
<td style="text-align: center" rowspan="2">1</td>
<td style="text-align: center" rowspan="2">0</td>
<td style="text-align: center" >0</td>
</tr>
<tr>
<td style="text-align: center">上拉输入</td>
<td style="text-align: center" >1</td>
</tr>
</tbody>
</table>

## 输出模式位

|MODE[1:0] |意义|
|:----:|:----:|
|00 |保留 |
|01 |最大输出速度为10MHz|
|10| 最大输出速度为2MHz|
|11 |最大输出速度为50MHz|

## 外设的GPIO配置

### 高级定时器TIM1/TIM8

<table >
<thead>
<tr>
<th style="text-align: center" >TIM1/TIM8引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">TIM1/8_CHx</th>
<td style="text-align: center" >输入捕获通道x</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<td style="text-align: center" >输出比较通道x</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >TIM1/8_CHxN</th>
<td style="text-align: center" >互补输出通道x</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >TIM1/8_BKIN</th>
<td style="text-align: center" >刹车输入</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" >TIM1/8_ETR</th>
<td style="text-align: center" >外部触发时钟输入</td>
<td style="text-align: center" >浮空输入</td>
</tr>
</tbody>
</table>

### 通用定时器TIM2/3/4/5

<table>
<thead>
<tr>
<th style="text-align: center" >TIM2/3/4/5引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">TIM2/3/4/5_CHx</th>
<td style="text-align: center" >输入捕获通道x</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<td style="text-align: center" >输出比较通道x</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >TIM2/3/4/5_ETR</th>
<td style="text-align: center" >外部触发时钟输入</td>
<td style="text-align: center" >浮空输入</td>
</tr>
</tbody>
</table>

### USART

<table>
<thead>
<tr>
<th style="text-align: center" >USART引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">USARTx_TX</th>
<td style="text-align: center" >全双工模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >半双工同步模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" rowspan="2">USARTx_RX</th>
<td style="text-align: center" >全双工模式</td>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
<tr>
<td style="text-align: center" >半双工同步模式</td>
<td style="text-align: center" >未用，可作为通用I/O</td>
</tr>
<tr>
<th style="text-align: center" >USARTx_CK</th>
<td style="text-align: center" >同步模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >USARTx_RTS</th>
<td style="text-align: center" >硬件流量控制</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >USARTx_CTS</th>
<td style="text-align: center" >硬件流量控制</td>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
</tbody>
</table>

### SPI

<table >
<thead>
<tr>
<th style="text-align: center" >SPI引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">SPIx_SCK</th>
<td style="text-align: center" >主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >从模式</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="4">SPIx_MOSI</th>
<td style="text-align: center" >全双工模式/主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >全双工模式/从模式</td>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
<tr>
<td style="text-align: center" >简单的双向数据线/主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >简单的双向数据线/从模式</td>
<td style="text-align: center" >未用，可作为通用I/O</td>
</tr>
<tr>
<th style="text-align: center" rowspan="4">SPIx_MISO</th>
<td style="text-align: center" >全双工模式/主模式</td>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
<tr>
<td style="text-align: center" >全双工模式/从模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >简单的双向数据线/主模式</td>
<td style="text-align: center" >未用，可作为通用I/O</td>
</tr>
<tr>
<td style="text-align: center" >简单的双向数据线/从模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" rowspan="3">SPIx_NSS</th>
<td style="text-align: center" >硬件主/从模式</td>
<td style="text-align: center" >浮空输入或带上拉输入或带下拉输入</td>
</tr>
<tr>
<td style="text-align: center" >硬件主模式/NSS输出使能</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >软件模式</td>
<td style="text-align: center" >未用，可作为通用I/O</td>
</tr>
</tbody>
</table>

### I2S

<table >
<thead>
<tr>
<th style="text-align: center" >I2S引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">I2Sx_WS</th>
<td style="text-align: center" >主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >从模式</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="2">I2Sx_CK</th>
<td style="text-align: center" >主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >从模式</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="2">I2Sx_SD</th>
<td style="text-align: center" >发送器</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >接收器</td>
<td style="text-align: center" >浮空输入或带上拉输入或带下拉输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="2">I2Sx_MCK</th>
<td style="text-align: center" >主模式</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center" >从模式</td>
<td style="text-align: center" >未用，可作为通用I/O</td>
</tr>
</tbody>
</table>

### I2C接口

<table >
<thead>
<tr>
<th style="text-align: center" >I2C引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >I2Cx_SCL</th>
<td style="text-align: center" >I2C时钟</td>
<td style="text-align: center" >开漏复用输出</td>
</tr>
<tr>
<th style="text-align: center" >I2Cx_SDA</th>
<td style="text-align: center" >I2C数据</td>
<td style="text-align: center" >开漏复用输出</td>
</tr>
</tbody>
</table>

### BxCAN

<table >
<thead>
<tr>
<th style="text-align: center" >BxCAN引脚</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >CAN_TX</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >CAN_RX</th>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
</tbody>
</table>

### USB

<table >
<thead>
<tr>
<th style="text-align: center" >USB引脚</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >USB_DM / USB_DP</th>
<td style="text-align: center" >一旦使能了USB模块，这些引脚会自动连接到内部USB收发器</td>
</tr>
</tbody>
</table>

注：本表内容只适用于小容量、中容量和大容量产品

### 全速USB OTG引脚配置

<table >
<thead>
<tr>
<th style="text-align: center" >OTG_FS引脚</th>
<th style="text-align: center" >配置</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="3">OTG_FS_SOF</th>
<td style="text-align: center">主机</td>
<td style="text-align: center" >如果使用此引脚，则为推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center">设备</td>
<td style="text-align: center" >如果使用此引脚，则为推挽复用输出</td>
</tr>
<tr>
<td style="text-align: center">OTG</td>
<td style="text-align: center" >如果使用此引脚，则为推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" rowspan="3">OTG_FS_VBUS</th>
<td style="text-align: center">主机</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<td style="text-align: center">设备</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<td style="text-align: center">OTG</td>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="3">OTG_FS_ID</th>
<td style="text-align: center">主机</td>
<td style="text-align: center" >如果软件选择了强置主机模式(OTG_FS_GUSBCFG寄存器的FHMOD位)，则不需要此引脚</td>
</tr>
<tr>
<td style="text-align: center">设备</td>
<td style="text-align: center" >如果软件选择了强置设备模式(OTG_FS_GUSBCFG寄存器的FHMOD位)，则不需要此引脚</td>
</tr>
<tr>
<td style="text-align: center">OTG</td>
<td style="text-align: center" >上拉输入</td>
</tr>
<tr>
<th style="text-align: center" rowspan="3">OTG_FS_DM</th>
<td style="text-align: center">主机</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
<tr>
<td style="text-align: center">设备</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
<tr>
<td style="text-align: center">OTG</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
<tr>
<th style="text-align: center" rowspan="3">OTG_FS_DP</th>
<td style="text-align: center">主机</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
<tr>
<td style="text-align: center">设备</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
<tr>
<td style="text-align: center">OTG</td>
<td style="text-align: center" >由USB断电自动控制</td>
</tr>
</tbody>
</table>

注：
- 本表内容只适用于互联型产品
- 如果另一个共享的外设要使用OTG_FS_VBUS引脚(PA9)或把它作为通用I/O口，必须激活PHY的断电模式(清除OTG_FS_GCCFG寄存器的位16)

### SDIO

<table >
<thead>
<tr>
<th style="text-align: center" >SDIO引脚</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >SDIO_CK</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >SDIO_CMD</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >SDIO[D7:D0]</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
</tbody>
</table>

### ADC

<table >
<thead>
<tr>
<th style="text-align: center" >ADC/DAC引脚</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >ADC/DAC</th>
<td style="text-align: center" >模拟输入</td>
</tr>
</tbody>
</table>

### FSMC

<table >
<thead>
<tr>
<th style="text-align: center" >FSMC引脚</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >FSMC_A[25:0]<br/>FSMC_D[15:0]</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_CK</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NOE<br/>FSMC_NWE</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NE[4:1]<br/>FSMC_NCE[3:2]<br/>FSMC_NCE4_1<br/>FSMC_NCE4_2</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NWAIT<br/>FSMC_CD</th>
<td style="text-align: center" >浮空输入或带上拉输入</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NIOS16<br/>FSMC_INTR<br/>FSMC_INT[3:2]</th>
<td style="text-align: center" >浮空输入</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NL<br/>FSMC_NBL[1:0]</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >FSMC_NIORD<br/>FSMC_NIOWR<br/>FSMC_NREG</th>
<td style="text-align: center" >推挽复用输出</td>
</tr>
</tbody>
</table>

### 其它I/O功能

<table >
<thead>
<tr>
<th style="text-align: center" >引脚</th>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >GPIO配置</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" rowspan="2">TAMPER-RTC</th>
<td style="text-align: center">RTC输出</td>
<td style="text-align: center" rowspan="2">当配置BKP_CR和BKP_RTCCR寄存器时，由硬件强制设置</td>
</tr>
<tr>
<td style="text-align: center">侵入事件输入</td>
</tr>
<tr>
<th style="text-align: center" >MCO</th>
<td style="text-align: center">时钟输出</td>
<td style="text-align: center" >推挽复用输出</td>
</tr>
<tr>
<th style="text-align: center" >EXTI输入线</th>
<td style="text-align: center">外部中断输入</td>
<td style="text-align: center" >浮空输入或带上拉输入或带下拉输入</td>
</tr>
</tbody>
</table>

## 复用功能I/O和调试配置(AFIO)
### CAN1复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能<sup>(1)</sup></th>
<th style="text-align: center" >CAN_REMAP[1:0]=”00”</th>
<th style="text-align: center" >CAN_REMAP[1:0]=”10”<sup>(2)</sup></th>
<th style="text-align: center" >CAN_REMAP[1:0]=”11”<sup>(3)</sup></th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >CAN1_RX 或 AN_RX</th>
<td style="text-align: center">PA11</td>
<td style="text-align: center" >PB8</td>
<td style="text-align: center" >PD0</td>
</tr>
<tr>
<th style="text-align: center">CAN1_TX 或 AN_TX</th>
<td style="text-align: center">PA12</td>
<td style="text-align: center" >PB9</td>
<td style="text-align: center" >PD1</td>
</tr>
</tbody>
</table>

1. 在互联型产品中是CAN1_RX和CAN1_TX；在其它带有单个CAN接口的产品中是CAN_RX和CAN_TX。 
2. 重映射不适用于36脚的封装 
3. 当PD0和PD1没有被重映射到OSC_IN和OSC_OUT时，重映射功能只适用于100脚和144脚的封装

### CAN2复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >CAN2_REMAP=”0”</th>
<th style="text-align: center" >CAN2_REMAP=”1”</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >CAN2_RX</th>
<td style="text-align: center">PB12</td>
<td style="text-align: center" >PB5</td>
</tr>
<tr>
<th style="text-align: center" >CAN2_TX</th>
<td style="text-align: center" >PB13</td>
<td style="text-align: center" >PB6</td>
</tr>
</tbody>
</table>

### JTAG/SWD复用功能重映射
#### 调试接口信号

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >GPIO端口</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >JTMS/SWDIO</th>
<td style="text-align: center">PA13</td>
</tr>
<tr>
<th style="text-align: center" >JTCK/SWCLK</th>
<td style="text-align: center">PA14</td>
</tr>
<tr>
<th style="text-align: center" >JTDI</th>
<td style="text-align: center">PA15</td>
</tr>
<tr>
<th style="text-align: center" >JTDO/TRACESWO</th>
<td style="text-align: center">PB3</td>
</tr>
<tr>
<th style="text-align: center" >JNTRST</th>
<td style="text-align: center">PB4</td>
</tr>
<tr>
<th style="text-align: center" >TRACECK</th>
<td style="text-align: center">PE2</td>
</tr>
<tr>
<th style="text-align: center" >TRACED0</th>
<td style="text-align: center">PE3</td>
</tr>
<tr>
<th style="text-align: center" >TRACED1</th>
<td style="text-align: center">PE4</td>
</tr>
<tr>
<th style="text-align: center" >TRACED2</th>
<td style="text-align: center">PE5</td>
</tr>
<tr>
<th style="text-align: center" >TRACED3</th>
<td style="text-align: center">PE6</td>
</tr>
</tbody>
</table>

#### 调试端口映像

<table >
<thead>
<tr>
<th style="text-align: center" rowspan="2">SWJ_CFG[2:0]</th>
<th style="text-align: center" rowspan="2">可能的调试端口</th>
<th style="text-align: center" colspan="5">SWJ I/O引脚分配</th>
</tr>
<tr>
<th style="text-align: center" >PA13<br/>JTMS<br/>SWDIO</th>
<th style="text-align: center" >PA14<br/>JTCK<br/>SWCLK</th>
<th style="text-align: center" >PA15<br/>JTDI</th>
<th style="text-align: center" >PB3<br/>JTDO<br/>TRACESWO</th>
<th style="text-align: center" >PB4<br/>NJTRST</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >000</th>
<td style="text-align: center">完全SWJ(JTAG-DP + SW-DP)(复位状态)</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
</tr>
<tr>
<th style="text-align: center" >001</th>
<td style="text-align: center">完全SWJ(JTAG-DP + SW-DP)但没有JNTRST</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O可用</td>
</tr>
<tr>
<th style="text-align: center" >010</th>
<td style="text-align: center">关闭JTAG-DP，启用SW-DP</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O不可用</td>
<td style="text-align: center">I/O可用</td>
<td style="text-align: center">I/O可用<sup>(1)</sup></td>
<td style="text-align: center">I/O可用</td>
</tr>
<tr>
<th style="text-align: center" >100</th>
<td style="text-align: center">关闭JTAG-DP，关闭SW-DP</td>
<td style="text-align: center">I/O可用</td>
<td style="text-align: center">I/O可用</td>
<td style="text-align: center">I/O可用</td>
<td style="text-align: center">I/O可用</td>
<td style="text-align: center">I/O可用</td>
</tr>
<tr>
<th style="text-align: center" >其它</th>
<td style="text-align: center">禁用</td>
</tr>
</tbody>
</table>

1. I/O口只可在不使用异步跟踪时使用

### ADC复用功能重映射
- 重映射仅存在于大容量产品

#### ADC1外部触发注入转换复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >ADC1_ETRGINJ_REMAP = 0</th>
<th style="text-align: center" >ADC1_ETRGINJ_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >ADC1外部触发注入转换</td>
<td style="text-align: center">ADC1外部触发注入转换与EXTI15相连</td>
<td style="text-align: center">ADC1外部触发注入转换与TIM8_CH4相连</td>
</tr>
</tbody>
</table>

#### ADC1外部触发规则转换复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >ADC1_ETRGREG_REMAP = 0</th>
<th style="text-align: center" >ADC1_ETRGREG_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >ADC1外部触发规则转换</td>
<td style="text-align: center">ADC1外部触发规则转换与EXTI11相连</td>
<td style="text-align: center">ADC1外部触发规则转换与TIM8_TRGO相连</td>
</tr>
</tbody>
</table>

#### ADC2外部触发注入转换复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >ADC2_ETRGINJ_REMAP = 0</th>
<th style="text-align: center" >ADC2_ETRGINJ_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >ADC2外部触发注入转换</td>
<td style="text-align: center">ADC2外部触发注入转换与EXTI15相连</td>
<td style="text-align: center">ADC2外部触发注入转换与TIM8_CH4相连</td>
</tr>
</tbody>
</table>

#### ADC2外部触发规则转换复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >ADC2_ETRGREG_REMAP = 0</th>
<th style="text-align: center" >ADC2_ETRGREG_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >ADC2外部触发规则转换</td>
<td style="text-align: center">ADC2外部触发规则转换与EXTI11相连</td>
<td style="text-align: center">ADC2外部触发规则转换与TIM8_TRGO相连</td>
</tr>
</tbody>
</table>

### 定时器复用功能重映射
#### TIM5复用功能重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >TIM5CH4_IREMAP = 0</th>
<th style="text-align: center" >TIM5CH4_IREMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM5_CH4</th>
<td style="text-align: center">TIM5的通道4连至PA3</td>
<td style="text-align: center">LSI内部时钟连至TIM5_CH4的输入作为校准使用</td>
</tr>
</tbody>
</table>

- 重映像只适用于大容量产品和互联型产品

#### TIM4复用功能重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >TIM4_REMAP = 0</th>
<th style="text-align: center" >TIM4_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM4_CH1</th>
<td style="text-align: center">PB6</td>
<td style="text-align: center">PD12</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_CH2</th>
<td style="text-align: center">PB7</td>
<td style="text-align: center">PD13</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_CH3</th>
<td style="text-align: center">PB8</td>
<td style="text-align: center">PD14</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_CH4</th>
<td style="text-align: center">PB9</td>
<td style="text-align: center">PD15</td>
</tr>
</tbody>
</table>

- 重映像只适用于100和144脚的封装

#### TIM3复用功能重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >TIM3_REMAP[1:0] = 00<br/>(没有重映像)</th>
<th style="text-align: center" >TIM3_REMAP[1:0] = 10<br/>(部分重映像)</th>
<th style="text-align: center" >TIM3_REMAP[1:0] = 11<br/>(完全重映像)<sup>(1)</sup></th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM3_CH1</th>
<td style="text-align: center">PA6</td>
<td style="text-align: center">PB4</td>
<td style="text-align: center">PC6</td>
</tr>
<tr>
<th style="text-align: center" >TIM3_CH2</th>
<td style="text-align: center">PA7</td>
<td style="text-align: center">PB13</td>
<td style="text-align: center">PC7</td>
</tr>
<tr>
<th style="text-align: center" >TIM3_CH3</th>
<td style="text-align: center" colspan="2">PB0</td>
<td style="text-align: center">PC8</td>
</tr>
<tr>
<th style="text-align: center" >TIM3_CH4</th>
<td style="text-align: center" colspan="2">PB1</td>
<td style="text-align: center">PC9</td>
</tr>
</tbody>
</table>

1. 重映像只适用于64、100和144脚的封装

#### TIM2复用功能重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >TIM2_REMAP[1:0] = 00<br/>(没有重映像)</th>
<th style="text-align: center" >TIM2_REMAP[1:0] = 01<br/>(部分重映像)</th>
<th style="text-align: center" >TIM2_REMAP[1:0] = 10<br/>(部分重映像)<sup>(1)</sup></th>
<th style="text-align: center" >TIM2_REMAP[1:0] = 11<br/>(完全重映像)<sup>(1)</sup></th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM2_CH1_ETR<sup>(2)</sup></th>
<td style="text-align: center">PA0</td>
<td style="text-align: center">PA15</td>
<td style="text-align: center">PA0</td>
<td style="text-align: center">PA15</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CH2</th>
<td style="text-align: center">PA1</td>
<td style="text-align: center">PB3</td>
<td style="text-align: center">PA1</td>
<td style="text-align: center">PB3</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CH3</th>
<td style="text-align: center" colspan="2">PA2</td>
<td style="text-align: center" colspan="2">PB10</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CH4</th>
<td style="text-align: center" colspan="2">PA3</td>
<td style="text-align: center" colspan="2">PB11</td>
</tr>
</tbody>
</table>

1. 重映像不适用于36脚的封装
2. TIM2_CH1和TIM2_ETR共用一个引脚，但不能同时使用(因此在此使用这样的标记：TIM2_CH1_ETR)

#### TIM1复用功能重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >TIM1_REMAP[1:0] = 00<br/>(没有重映像)</th>
<th style="text-align: center" >TIM1_REMAP[1:0] = 01<br/>(部分重映像)</th>
<th style="text-align: center" >TIM1_REMAP[1:0] = 11<br/>(完全重映像)<sup>(1)</sup></th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM1_ETR</th>
<td style="text-align: center" colspan="2">PA12</td>
<td style="text-align: center">PE7</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH1</th>
<td style="text-align: center" colspan="2">PA8</td>
<td style="text-align: center">PE9</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH2</th>
<td style="text-align: center" colspan="2">PA9</td>
<td style="text-align: center">PE11</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH3</th>
<td style="text-align: center" colspan="2">PA10</td>
<td style="text-align: center" >PE13</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH4</th>
<td style="text-align: center" colspan="2">PA11</td>
<td style="text-align: center" >PE14</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_BKIN</th>
<td style="text-align: center" >PB12<sup>(2)</sup></td>
<td style="text-align: center" >PA6</td>
<td style="text-align: center" >PE15</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH1N</th>
<td style="text-align: center" >PB13<sup>(2)</sup></td>
<td style="text-align: center" >PA7</td>
<td style="text-align: center" >PE8</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH2N</th>
<td style="text-align: center" >PB14<sup>(2)</sup></td>
<td style="text-align: center" >PB0</td>
<td style="text-align: center" >PE10</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CH3N</th>
<td style="text-align: center" >PB15<sup>(2)</sup></td>
<td style="text-align: center" >PB1</td>
<td style="text-align: center" >PE12</td>
</tr>
</tbody>
</table>

1. 重映像只适用于100和144脚的封装
2. 重映像不适用于36脚的封装

### USART复用功能重映射
#### USART3重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >USART3_REMAP[1:0] = 00<br/>(没有重映像)</th>
<th style="text-align: center" >USART3_REMAP[1:0] = 01<br/>(部分重映像)<sup>(1)</sup></th>
<th style="text-align: center" >USART3_REMAP[1:0] = 11<br/>(完全重映像)<sup>(2)</sup></th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >USART3_TX</th>
<td style="text-align: center" >PB10</td>
<td style="text-align: center">PC10</td>
<td style="text-align: center">PD8</td>
</tr>
<tr>
<th style="text-align: center" >USART3_RX</th>
<td style="text-align: center" >PB11</td>
<td style="text-align: center">PC11</td>
<td style="text-align: center">PD9</td>
</tr>
<tr>
<th style="text-align: center" >USART3_CK</th>
<td style="text-align: center" >PB12</td>
<td style="text-align: center">PC12</td>
<td style="text-align: center">PD10</td>
</tr>
<tr>
<th style="text-align: center" >USART3_CTS</th>
<td style="text-align: center" colspan="2">PB13</td>
<td style="text-align: center">PD11</td>
</tr>
<tr>
<th style="text-align: center" >USART3_RTS</th>
<td style="text-align: center" colspan="2">PB14</td>
<td style="text-align: center">PD12</td>
</tr>
</tbody>
</table>

1. 重映像只适用于64、100和144脚的封装
2. 重映像只适用于100和144脚的封装

#### USART2重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >USART2_REMAP = 0</th>
<th style="text-align: center" >USART2_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >USART2_CTS</th>
<td style="text-align: center" >PA0</td>
<td style="text-align: center">PD3</td>
</tr>
<tr>
<th style="text-align: center" >USART2_RTS</th>
<td style="text-align: center" >PA1</td>
<td style="text-align: center">PD4</td>
</tr>
<tr>
<th style="text-align: center" >USART2_TX</th>
<td style="text-align: center" >PA2</td>
<td style="text-align: center">PD5</td>
</tr>
<tr>
<th style="text-align: center" >USART2_RX</th>
<td style="text-align: center" >PA3</td>
<td style="text-align: center">PD6</td>
</tr>
<tr>
<th style="text-align: center" >USART2_CK</th>
<td style="text-align: center" >PA4</td>
<td style="text-align: center">PD7</td>
</tr>
</tbody>
</table>

- 重映像只适用于100和144脚的封装

#### USART1重映像

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >USART1_REMAP = 0</th>
<th style="text-align: center" >USART1_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >USART1_TX</th>
<td style="text-align: center" >PA9</td>
<td style="text-align: center">PB6</td>
</tr>
<tr>
<th style="text-align: center" >USART1_RX</th>
<td style="text-align: center" >PA10</td>
<td style="text-align: center">PB7</td>
</tr>
</tbody>
</table>

### I<sup>2</sup>C1复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >I2C1_REMAP = 0</th>
<th style="text-align: center" >I2C1_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >I2C1_SCL</th>
<td style="text-align: center" >PB6</td>
<td style="text-align: center">PB8</td>
</tr>
<tr>
<th style="text-align: center" >I2C1_SDK</th>
<td style="text-align: center" >PB7</td>
<td style="text-align: center">PB9</td>
</tr>
</tbody>
</table>

- 重映像不适用于36脚封装

### SPI1复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >SPI1_REMAP = 0</th>
<th style="text-align: center" >SPI1_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >SPI1_NSS</th>
<td style="text-align: center" >PA4</td>
<td style="text-align: center">PA15</td>
</tr>
<tr>
<th style="text-align: center" >SPI1_SCK</th>
<td style="text-align: center" >PA5</td>
<td style="text-align: center">PB3</td>
</tr>
<tr>
<th style="text-align: center" >SPI1_MISO</th>
<td style="text-align: center" >PA6</td>
<td style="text-align: center">PB4</td>
</tr>
<tr>
<th style="text-align: center" >SPI1_MOSI</th>
<td style="text-align: center" >PA7</td>
<td style="text-align: center">PB5</td>
</tr>
</tbody>
</table>

### SPI3复用功能重映射

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >SPI3_REMAP = 0</th>
<th style="text-align: center" >SPI3_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >SPI3_NSS</th>
<td style="text-align: center">PA15</td>
<td style="text-align: center" >PA4</td>
</tr>
<tr>
<th style="text-align: center" >SPI3_SCK</th>
<td style="text-align: center">PB3</td>
<td style="text-align: center" >PC10</td>
</tr>
<tr>
<th style="text-align: center" >SPI3_MISO</th>
<td style="text-align: center">PB4</td>
<td style="text-align: center" >PC11</td>
</tr>
<tr>
<th style="text-align: center" >SPI3_MOSI</th>
<td style="text-align: center">PB5</td>
<td style="text-align: center" >PC12</td>
</tr>
</tbody>
</table>

### 以太网复用功能重映射
- 以太网只出现在互联型产品

<table >
<thead>
<tr>
<th style="text-align: center" >复用功能</th>
<th style="text-align: center" >ETH_REMAP = 0</th>
<th style="text-align: center" >ETH_REMAP = 1</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >RX_DV-CRS_DV</th>
<td style="text-align: center">PA7</td>
<td style="text-align: center" >PD8</td>
</tr>
<tr>
<th style="text-align: center" >RXD0</th>
<td style="text-align: center">PC4</td>
<td style="text-align: center" >PD9</td>
</tr>
<tr>
<th style="text-align: center" >RXD1</th>
<td style="text-align: center">PC5</td>
<td style="text-align: center" >PD10</td>
</tr>
<tr>
<th style="text-align: center" >RXD2</th>
<td style="text-align: center">PB0</td>
<td style="text-align: center" >PD11</td>
</tr>
<tr>
<th style="text-align: center" >RXD3</th>
<td style="text-align: center">PB1</td>
<td style="text-align: center" >PD12</td>
</tr>
</tbody>
</table>

# 中断

|NVIC_PriorityGroup<br/>中断向量组|NVIC_IRQChannelPreemptionPriority<br/>抢占优先级|NVIC_IRQChannelSubPriority<br/>子优先级|Description<br/>描述|
|:----:|:----:|:----:|:----:|:----:|
|NVIC_PriorityGroup_0|0|0-15|0 bits for pre-emption priority<br/>4 bits for subpriority|
|NVIC_PriorityGroup_1|0-1|0-7|1 bits for pre-emption priority<br/>3 bits for subpriority|
|NVIC_PriorityGroup_2|0-3|0-3|2 bits for pre-emption priority<br/>2 bits for subpriority|
|NVIC_PriorityGroup_3|0-7|0-1|3 bits for pre-emption priority<br/>1 bits for subpriority|
|NVIC_PriorityGroup_4|0-15|0|4 bits for pre-emption priority<br/>0 bits for subpriority|

# DMA
## 各个通道的DMA1请求一览

|外设|DMA1_Channel1|DMA1_Channel2|DMA1_Channel3|DMA1_Channel4|DMA1_Channel5|DMA1_Channel6|DMA1_Channel7|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|ADC1|ADC1|
|SPI/I<sup>2</sup>S||SPI1_RX|SPI1_TX|SPI/I<sup>2</sup>S2_RX|SPI/I<sup>2</sup>S_TX|
|USART||USART3_TX|USART3_RX|USART1_TX|USART1_RX|USART2_RX|USART2_TX|
|I<sup>2</sup>C||||I<sup>2</sup>C2_TX|I<sup>2</sup>C2_RX|I<sup>2</sup>C1_TX|I<sup>2</sup>C1_RX|
|TIM1||TIM1_CH1|TIM1_CH2|TIM1_CH4<BR/>TIM1_TRIG<BR/>TIM1_COM|TIM1_UP|TIM1_CH3|
|TIM2|TIM2_CH3|TIM2_UP|||TIM2_CH1||TIM2_CH2<BR/>TIM2_CH4|
|TIM3||TIM3_CH3|TIM3_CH4<BR/>TIM3_UP|||TIM3_CH1<BR/>TIM3_TRIG|
|TIM4|TIM4_CH1|||TIM4_CH2|TIM4_CH3||TIM4_UP|

## 各个通道的DMA2请求一览

|外设|DMA2_Channel1|DMA2_Channel2|DMA2_Channel3|DMA2_Channel4|DMA2_Channel5|
|:----:|:----:|:----:|:----:|:----:|:----:|
|ADC3|||||ADC3|
|SPI/I<sup>2</sup>S3|SPI/I<sup>2</sup>S3_RX|SPI/I<sup>2</sup>S3_TX|
|UART4|||UART4_RX||UART4_TX|
|SDIO||||SDIO|
|TIM5|TIM5_CH4<BR/>TIM5_TRIG|TIM5_CH3<BR/>TIM5_UP||TIM5_CH2|TIM5_CH1|
|TIM6/DAC_Channel_1|||TIM6_UP/DAC_Channel_1|
|TIM7/DAC_Channel_2||||TIM7_UP/DAC_Channel_2|
|TIM8|TIM8_CH3<BR/>TIM8_UP|TIM8_CH4<BR/>TIM8_TRIG<BR/>TIM8_COM|TIM8_CH1||TIM8_CH2|

- ADC3、SDIO和TIM8的DMA请求只在大容量的产品中存在

# ADC
## ADC1和ADC2用于规则通道的外部触发

<table >
<thead>
<tr>
<th style="text-align: center" >触发源</th>
<th style="text-align: center" >连接类型</th>
<th style="text-align: center" >EXTSEL[2:0]</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM1_CC1事件</th>
<td style="text-align: center" rowspan="6">来自片上定时器的内部信号</td>
<td style="text-align: center" >000</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CC2事件</th>
<td style="text-align: center" >001</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CC3事件</th>
<td style="text-align: center" >010</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CC2事件</th>
<td style="text-align: center" >011</td>
</tr>
<tr>
<th style="text-align: center" >TIM3_TRGO事件</th>
<td style="text-align: center" >100</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_CC4事件</th>
<td style="text-align: center" >101</td>
</tr>
<tr>
<th style="text-align: center" >EXTI线11/TIM8_TRGO事件</th>
<td style="text-align: center" >外部引脚/来自片上定时器的内部信号</td>
<td style="text-align: center" >110</td>
</tr>
<tr>
<th style="text-align: center" >SWSTART</th>
<td style="text-align: center" >软件控制位</td>
<td style="text-align: center" >111</td>
</tr>
</tbody>
</table>

- TIM8_TRGO事件只存在于大容量产品 
- 对于规则通道，选中EXTI线路11或TIM8_TRGO作为外部触发事件，可以分别通过设置ADC1和ADC2的ADC1_ETRGREG_REMAP位和ADC2_ETRGREG_REMAP位实现

## ADC1和ADC2用于注入通道的外部触发

<table >
<thead>
<tr>
<th style="text-align: center" >触发源</th>
<th style="text-align: center" >连接类型</th>
<th style="text-align: center" >JEXTSEL[2:0]</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM1_TRGO事件</th>
<td style="text-align: center" rowspan="6">来自片上定时器的内部信号</td>
<td style="text-align: center" >000</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CC4事件</th>
<td style="text-align: center" >001</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_TRGO事件</th>
<td style="text-align: center" >010</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CC1事件</th>
<td style="text-align: center" >011</td>
</tr>
<tr>
<th style="text-align: center" >TIM3_CC4事件</th>
<td style="text-align: center" >100</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_TRGO事件</th>
<td style="text-align: center" >101</td>
</tr>
<tr>
<th style="text-align: center" >EXTI线15/TIM8_CC4事件</th>
<td style="text-align: center" >外部引脚/来自片上定时器的内部信号</td>
<td style="text-align: center" >110</td>
</tr>
<tr>
<th style="text-align: center" >JSWSTART</th>
<td style="text-align: center" >软件控制位</td>
<td style="text-align: center" >111</td>
</tr>
</tbody>
</table>

- TIM8_CC4事件只存在于大容量产品 
- 对于注入通道，选中EXTI线路15和TIM8_CC4作为外部触发事件，可以分别通过设置ADC1和ADC2的ADC1_ETRGINJ_REMAP位和ADC2_ ETRGINJ_REMAP位实现

## ADC3用于规则通道的外部触发

<table >
<thead>
<tr>
<th style="text-align: center" >触发源</th>
<th style="text-align: center" >连接类型</th>
<th style="text-align: center" >EXTSEL[2:0]</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM3_CC1事件</th>
<td style="text-align: center" rowspan="7">来自片上定时器的内部信号</td>
<td style="text-align: center" >000</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_CC3事件</th>
<td style="text-align: center" >001</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CC3事件</th>
<td style="text-align: center" >010</td>
</tr>
<tr>
<th style="text-align: center" >TIM8_CC1事件</th>
<td style="text-align: center" >011</td>
</tr>
<tr>
<th style="text-align: center" >TIM8_TRGO事件</th>
<td style="text-align: center" >100</td>
</tr>
<tr>
<th style="text-align: center" >TIM5_CC1事件</th>
<td style="text-align: center" >101</td>
</tr>
<tr>
<th style="text-align: center" >TIM5_CC3事件</th>
<td style="text-align: center" >110</td>
</tr>
<tr>
<th style="text-align: center" >SWSTART</th>
<td style="text-align: center" >软件控制位</td>
<td style="text-align: center" >111</td>
</tr>
</tbody>
</table>

## ADC3用于注入通道的外部触发

<table >
<thead>
<tr>
<th style="text-align: center" >触发源</th>
<th style="text-align: center" >连接类型</th>
<th style="text-align: center" >JEXTSEL[2:0]</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM1_TRGO事件</th>
<td style="text-align: center" rowspan="7">来自片上定时器的内部信号</td>
<td style="text-align: center" >000</td>
</tr>
<tr>
<th style="text-align: center" >TIM1_CC4事件</th>
<td style="text-align: center" >001</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_CC3事件</th>
<td style="text-align: center" >010</td>
</tr>
<tr>
<th style="text-align: center" >TIM8_CC2事件</th>
<td style="text-align: center" >011</td>
</tr>
<tr>
<th style="text-align: center" >TIM8_CC4事件</th>
<td style="text-align: center" >100</td>
</tr>
<tr>
<th style="text-align: center" >TIM5_TRGO事件</th>
<td style="text-align: center" >101</td>
</tr>
<tr>
<th style="text-align: center" >TIM5_CC4事件</th>
<td style="text-align: center" >110</td>
</tr>
<tr>
<th style="text-align: center" >JSWSTART</th>
<td style="text-align: center" >软件控制位</td>
<td style="text-align: center" >111</td>
</tr>
</tbody>
</table>

## ADC通道

|通道|ADC1-IO|ADC2-IO|ADC3-IO|
|:----:|:----:|:----:|:----:|
|ADC_Channel_0 |PA0|PA0|PA0|
|ADC_Channel_1 |PA1|PA1|PA1|
|ADC_Channel_2 |PA2|PA2|PA2|
|ADC_Channel_3 |PA3|PA3|PA3|
|ADC_Channel_4 |PA4|PA4|N|
|ADC_Channel_5 |PA5|PA5|N|
|ADC_Channel_6 |PA6|PA6|N|
|ADC_Channel_7 |PA7|PA7|N|
|ADC_Channel_8 |PB0|PB0|N|
|ADC_Channel_9 |PB1|PB1|VSS|
|ADC_Channel_10 |PC0|PC0|PC0|
|ADC_Channel_11 |PC1|PC1|PC1|
|ADC_Channel_12 |PC2|PC2|PC2|
|ADC_Channel_13 |PC3|PC3|PC3|
|ADC_Channel_14 |PC4|PC4|VSS|
|ADC_Channel_15 |PC5|PC5|VSS|
|ADC_Channel_16 |TempSensor|VSS|VSS|
|ADC_Channel_17 |Vrefint|VSS|VSS|

# DAC
## DAC触发

<table >
<thead>
<tr>
<th style="text-align: center" >触发源</th>
<th style="text-align: center" >连接类型</th>
<th style="text-align: center" >TSELx[2:0]</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >TIM6_TRGO事件</th>
<td style="text-align: center" rowspan="6">来自片上定时器的内部信号</td>
<td style="text-align: center" >000</td>
</tr>
<tr>
<th style="text-align: center" >互联型产品为TIM3_TRGO事件<br/>大容量产品为TIM8_TRGO事件</th>
<td style="text-align: center" >001</td>
</tr>
<tr>
<th style="text-align: center" >TIM7_TRGO事件</th>
<td style="text-align: center" >010</td>
</tr>
<tr>
<th style="text-align: center" >TIM5_TRGO事件</th>
<td style="text-align: center" >011</td>
</tr>
<tr>
<th style="text-align: center" >TIM2_TRGO事件</th>
<td style="text-align: center" >100</td>
</tr>
<tr>
<th style="text-align: center" >TIM4_TRGO事件</th>
<td style="text-align: center" >101</td>
</tr>
<tr>
<th style="text-align: center" >EXTI线路9</th>
<td style="text-align: center" >外部引脚</td>
<td style="text-align: center" >110</td>
</tr>
<tr>
<th style="text-align: center" >SWTRIG(软件触发)</th>
<td style="text-align: center" >软件控制位</td>
<td style="text-align: center" >111</td>
</tr>
</tbody>
</table>

## 通道

|通道|IO|
|:----:|:----:|
|Channel_0|PA4|
|Channel_1|PA5|

# TIM
## 定时器分类

|分类|定时器|计数器分辨率|计数器类型|捕获/比较通道|互补输出|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|基本定时器|TIM6|16位|向上|0|无|
|基本定时器|TIM7|16位|向上|0|无|
|通用定时器|TIM2|16位|向上/向下|4|无|
|通用定时器|TIM3|16位|向上/向下|4|无|
|通用定时器|TIM4|16位|向上/向下|4|无|
|通用定时器|TIM5|16位|向上/向下|4|无|
|高级定时器|TIM1|16位|向上/向下|4|有|
|高级定时器|TIM8|16位|向上/向下|4|有|

## 定时时间计算

$ TIME=\frac{(TIM\\_Period+1)\times (TIM\\_Prescaler+1)}{CLK} $

# USART
## 波特率计算

$ USART\\_BaudRate=\frac{PCLKx}{16 \times USARTDIV} $

$ IntegerDivider = \frac{PCLKx}{16 \times USART\\_InitStruct \to USART\\_BaudRate}$

$ FractionalDivider = ((IntegerDivider - ((u32) IntegerDivider)) \times 16) + 0.5$

## 波特率误差

<table >
<thead>
<tr>
<th style="text-align: center" >波特率</th>
<th style="text-align: center" colspan="3">f<sub>PCLK</sub> = 36MHz</th>
<th style="text-align: center" colspan="3">f<sub>PCLK</sub> = 72MHz</th>
</tr>
<tr>
<th style="text-align: center" >Kbps</th>
<th style="text-align: center" >实际</th>
<th style="text-align: center" >置于波特率寄存器中的值</th>
<th style="text-align: center" >误差%</th>
<th style="text-align: center" >实际</th>
<th style="text-align: center" >置于波特率寄存器中的值</th>
<th style="text-align: center" >误差</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" >2.4</th>
<td style="text-align: center" >2.400</td>
<td style="text-align: center" >937.5</td>
<td style="text-align: center" >0%</td>
<td style="text-align: center" >2.400</td>
<td style="text-align: center" >1875</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >9.6</th>
<td style="text-align: center" >9.600</td>
<td style="text-align: center" >234.375</td>
<td style="text-align: center" >0%</td>
<td style="text-align: center" >9.600</td>
<td style="text-align: center" >468.75</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >19.2</th>
<td style="text-align: center" >19.2</td>
<td style="text-align: center" >117.1875</td>
<td style="text-align: center" >0%</td>
<td style="text-align: center" >19.2</td>
<td style="text-align: center" >234.375</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >57.6</th>
<td style="text-align: center" >57.6</td>
<td style="text-align: center" >39.0625</td>
<td style="text-align: center" >0%</td>
<td style="text-align: center" >57.6</td>
<td style="text-align: center" >78.125</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >115.2</th>
<td style="text-align: center" >115.384</td>
<td style="text-align: center" >19.5</td>
<td style="text-align: center" >0.15%</td>
<td style="text-align: center" >115.2</td>
<td style="text-align: center" >139.0625</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >230.4</th>
<td style="text-align: center" >230.769</td>
<td style="text-align: center" >9.75</td>
<td style="text-align: center" >0.16%</td>
<td style="text-align: center" >230.769</td>
<td style="text-align: center" >19.5</td>
<td style="text-align: center" >0.16%</td>
</tr>
<tr>
<th style="text-align: center" >460.8</th>
<td style="text-align: center" >461.538</td>
<td style="text-align: center" >4.875</td>
<td style="text-align: center" >0.16%</td>
<td style="text-align: center" >461.538</td>
<td style="text-align: center" >9.75</td>
<td style="text-align: center" >0.16%</td>
</tr>
<tr>
<th style="text-align: center" >921.6</th>
<td style="text-align: center" >923.076</td>
<td style="text-align: center" >2.4375</td>
<td style="text-align: center" >0.16%</td>
<td style="text-align: center" >923.076</td>
<td style="text-align: center" >4.875</td>
<td style="text-align: center" >0.16%</td>
</tr>
<tr>
<th style="text-align: center" >2250</th>
<td style="text-align: center" >2250</td>
<td style="text-align: center" >1</td>
<td style="text-align: center" >0%</td>
<td style="text-align: center" >2250</td>
<td style="text-align: center" >2</td>
<td style="text-align: center" >0%</td>
</tr>
<tr>
<th style="text-align: center" >4500</th>
<td style="text-align: center" >不可能</td>
<td style="text-align: center" >不可能</td>
<td style="text-align: center" >不可能</td>
<td style="text-align: center" >4500</td>
<td style="text-align: center" >1</td>
<td style="text-align: center" >0%</td>
</tr>
</tbody>
</table>

- CPU的时钟频率越低，则某一特定波特率的误差也越低。可以达到的波特率上限可以由这组数据得到。
- 只有USART1使用PCLK2(最高72MHz)。其它USART使用PCLK1(最高36MHz)

## USART中断请求

<table >
<thead>
<tr>
<th style="text-align: center" >中断事件</th>
<th style="text-align: center" >事件标志</th>
<th style="text-align: center" >使能位</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >发送数据寄存器空</td>
<td style="text-align: center" >TXE</td>
<td style="text-align: center" >TXEIE</td>
</tr>
<tr>
<td style="text-align: center" >CTS标志</td>
<td style="text-align: center" >CTS</td>
<td style="text-align: center" >CTSIE</td>
</tr>
<tr>
<td style="text-align: center" >发送完成</td>
<td style="text-align: center" >TC</td>
<td style="text-align: center" >TCIE</td>
</tr>
<tr>
<td style="text-align: center" >接收数据就绪可读</td>
<td style="text-align: center" >RXNE</td>
<td style="text-align: center" rowspan="2">RXNEIE</td>
</tr>
<tr>
<td style="text-align: center" >检测到数据溢出</td>
<td style="text-align: center" >ORE</td>
</tr>
<tr>
<td style="text-align: center" >检测到空闲线路</td>
<td style="text-align: center" >IDLE</td>
<td style="text-align: center" >IDLEIE</td>
</tr>
<tr>
<td style="text-align: center" >奇偶检验错</td>
<td style="text-align: center" >PE</td>
<td style="text-align: center" >PEIE</td>
</tr>
<tr>
<td style="text-align: center" >断开标志</td>
<td style="text-align: center" >LBD</td>
<td style="text-align: center" >LBDIE</td>
</tr>
<tr>
<td style="text-align: center" >噪声标志，多缓冲通信中的溢出错误和帧错误</td>
<td style="text-align: center" >NE或ORT或FE</td>
<td style="text-align: center" >EIE<sup>(1)</sup></td>
</tr>
</tbody>
</table>

1. 仅当使用DMA接收数据时，才使用这个标志位

USART的各种中断事件被连接到同一个中断向量，有以下各种中断事件：
- 发送期间：发送完成、清除发送、发送数据寄存器空。
- 接收期间：空闲总线检测、溢出错误、接收数据寄存器非空、校验错误、LIN断开符号检测、噪音标志(仅在多缓冲器通信)和帧错误(仅在多缓冲器通信)

## USART模式配置

|USART模式|USART1|USART2|USART3|USART4|USART5|
|:----:|:----:|:----:|:----:|:----:|:----:|
|异步模式|支持|支持|支持|支持|支持|
|硬件流控制|支持|支持|支持|不支持|不支持|
|多缓存通讯(DMA)|支持|支持|支持|支持|不支持|
|多处理器通讯|支持|支持|支持|支持|支持|
|同步|支持|支持|支持|不支持|不支持|
|智能卡|支持|支持|支持|不支持|不支持|
|半双工(单线模式)|支持|支持|支持|支持|支持|
|IrDA|支持|支持|支持|支持|支持|
|LIN|支持|支持|支持|支持|支持|

# SPI

<table >
<thead>
<tr>
<th style="text-align: center" >中断事件</th>
<th style="text-align: center" >事件标志</th>
<th style="text-align: center" >使能位</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >发送缓冲器空标志</td>
<td style="text-align: center" >TXE</td>
<td style="text-align: center" >TXEIE</td>
</tr>
<tr>
<td style="text-align: center" >接收缓冲器非空标志</td>
<td style="text-align: center" >RXNE</td>
<td style="text-align: center" >RXNEIE</td>
</tr>
<tr>
<td style="text-align: center" >主模式失效事件</td>
<td style="text-align: center" >MODF</td>
<td style="text-align: center"  rowspan="3">ERRIE</td>
</tr>
<tr>
<td style="text-align: center" >溢出错误</td>
<td style="text-align: center" >OVR</td>
</tr>
<tr>
<td style="text-align: center" >CRC错误标志</td>
<td style="text-align: center" >CRCERR</td>
</tr>
</tbody>
</table>