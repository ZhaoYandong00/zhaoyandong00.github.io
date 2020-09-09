---
title: STM32硬件层HAL库串口通信之队列HAL篇5
categories: STM32 CUBE HAL USART QUEUE
tags: STM32 CUBE HAL USART QUEUE
description: HAL库串口通信之队列
---
# 环形队列文件
- 复制SPL库时写的就行
- `my_queue.c`放到**Core**文件价的**src**文件夹内
- `my_queue.h`放到**Core**文件价的**inc**文件夹内
- 修改`my_queue.h`包含头文件,把`stm32f10x.h`改为`#include "main.h"`

# 串口数据处理文件
- 复制SPL库时写的就行
- `my_process_data.c`放到**Core**文件价的**src**文件夹内
- `my_process_data.h`放到**Core**文件价的**inc**文件夹内
- 修改`my_process_data.h`包含头文件,把`stm32f10x.h`改为`#include "main.h"`，删除`#include "my_usart.h"`

# 修改串口中断函数
- 在`stm32f1xx_it.c`添加头文件包含

```c
/* USER CODE BEGIN Includes */
#include "my_data_queue.h"
/* USER CODE END Includes *
```
- 修改外部变量定义

```c
/* USER CODE BEGIN EV */
extern uint8_t rx_data;
/* USER CODE END EV */
```

- 修改串口空闲中断处理

```c
/**
  * @brief This function handles USART1 global interrupt.
  */
void USART1_IRQHandler(void)
{
    /* USER CODE BEGIN USART1_IRQn 0 */
    QUEUE_DATA_TYPE *data_p;
    /* USER CODE END USART1_IRQn 0 */
    HAL_UART_IRQHandler(&huart1);
    /* USER CODE BEGIN USART1_IRQn 1 */
    if((__HAL_UART_GET_FLAG(&huart1,UART_FLAG_IDLE)!=RESET) &&(__HAL_UART_GET_IT_SOURCE(&huart1,UART_IT_IDLE)!=RESET))
    {
        if(rx_queue.write!=rx_queue.write_using)
        {
            rx_queue.elems[(rx_queue.write_using)&(rx_queue.size - 1)]->len=QUEUE_NODE_DATA_LEN-huart1.RxXferCount;
            cbWriteFinish(&rx_queue);
        }
        __HAL_UART_CLEAR_IDLEFLAG(&huart1);
        HAL_UART_AbortReceive(&huart1);
        data_p = cbWrite(&rx_queue);
        if(data_p!=NULL)
        {
            HAL_UART_Receive_IT(&huart1,data_p->head,QUEUE_NODE_DATA_LEN);
        }
        else
        {
            //防止环形队列已满，串口将不进行接收问题，环形队列满了后，会继续读数据并覆盖直到队列可以进行操作
            HAL_UART_Receive_IT(&huart1,rx_data,1);
        }
    }
    /* USER CODE END USART1_IRQn 1 */
}
```
# 修改串口接收完成回调函数
- 在`main.c`文件添加头文件引用

```c
#include "my_process_data.h"
```
- 定义临时变量

```c
/* USER CODE BEGIN PV */
uint8_t rx_data;
Channel ch[2];
/* USER CODE END PV */
```
- 定义数据变量

```c
/* USER CODE BEGIN 1 */
QUEUE_DATA_TYPE *data_p;
/* USER CODE END 1 */
```
- 初始化数据队列

```c
    /* USER CODE BEGIN 2 */
    /*初始化接收数据队列*/
    RX_Queue_Init();
    data_p = cbWrite(&rx_queue);
    HAL_UART_Receive_IT(&huart1,data_p->head,QUEUE_NODE_DATA_LEN);
    /* USER CODE END 2 */
```
- 主循环添加数据处理调用

```c
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
```
- 修改串口回调

```c
/**
  * @brief  Rx Transfer completed callbacks.
  * @param  huart  Pointer to a UART_HandleTypeDef structure that contains
  *                the configuration information for the specified UART module.
  * @retval None
  */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    QUEUE_DATA_TYPE *data_p;
    if(huart->Instance==USART1)
    {
        if(rx_queue.write!=rx_queue.write_using)
        {
            rx_queue.elems[(rx_queue.write_using)&(rx_queue.size - 1)]->len=QUEUE_NODE_DATA_LEN;
            cbWriteFinish(&rx_queue);
        }
			  data_p = cbWrite(&rx_queue);
        if(data_p!=NULL)
        {
            HAL_UART_Receive_IT(huart,data_p->head,QUEUE_NODE_DATA_LEN);
        }
        else
        {
            HAL_UART_Receive_IT(huart,&rx_data,1);
        }
    }
}
```
# 调试
- 编译下载
- 验证
- 发送数据 `5A A5 06 83 10 10 01 00 01`会看到LED1亮
- 发送数据 `5A A5 06 83 10 10 01 00 02`会看到LED1灭
- 发送数据 `5A A5 06 83 11 10 01 00 01`会看到LED2亮
- 发送数据 `5A A5 06 83 11 10 01 00 02`会看到LED2灭
- 发送数据 `5A A5 06 83 10 10 01 00 01 5A A5 06 83 11 10 01 00 01`会看到LED1亮LED2亮，这个验证了如果上位机两帧发送过快，系统仍然可以正常处理数据，而不是出现个别帧不处理
- 结果和SPL库一致
