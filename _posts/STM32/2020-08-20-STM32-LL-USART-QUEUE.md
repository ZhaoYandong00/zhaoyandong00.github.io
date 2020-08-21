---
title: STM32底层LL库串口通信之队列
categories: STM32 CUBE LL USART QUEUE
tags: STM32 CUBE LL USART QUEUE
description: LL库串口通信之队列
---
# 环形队列文件
- 复制HAL库时的就行
- `my_queue.c`放到**Core**文件价的**src**文件夹内
- `my_queue.h`放到**Core**文件价的**inc**文件夹内

# 串口数据处理文件
- 复制HAL库时写的就行
- `my_process_data.c`放到**Core**文件价的**src**文件夹内
- `my_process_data.h`放到**Core**文件价的**inc**文件夹内
- 注释掉串口数据发送函数的定义和声明，因为是HAL库写的

```c
/**
  * @brief  串口数据发送
  * @param  huart
  * @retval 无
  */
//void Send_Usart_Data(UART_HandleTypeDef *huart)
//{
//    QUEUE_DATA_TYPE *tx_data;

//    /*从缓冲区读取数据，进行处理，*/
//    tx_data = cbRead(&tx_queue);
//    if (tx_data != NULL) //缓冲队列非空
//    {
//        if(HAL_UART_Transmit_DMA(huart,tx_data->head,tx_data->len)==HAL_BUSY)
//        {
//            return;
//        }
//        //使用完数据必须调用cbReadFinish更新读指针
//        cbReadFinish(&tx_queue);
//    }
//}
```

```c
//void Send_Usart_Data(UART_HandleTypeDef *huart);
```

# 修改串口中断函数
- 在`stm32f1xx_it.c`添加头文件包含

```c
#include "my_data_queue.h"
```

- 注释或删除接收数据变量

```c
/* USER CODE BEGIN PV */
//uint8_t rx_data[50];
//uint8_t rx_len=0;
/* USER CODE END PV */
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
    uint8_t ucCh;
    //数据接收完成
    if(LL_USART_IsActiveFlag_RXNE(USART1)!=RESET)
    {
        ucCh  = LL_USART_ReceiveData8( USART1);
        /*获取写缓冲区指针，准备写入新数据*/
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            //往缓冲区写入数据，如使用串口接收、dma写入等方式
            *(data_p->head + data_p->len) = ucCh;
            if (++data_p->len >= QUEUE_NODE_DATA_LEN)
            {
                cbWriteFinish(&rx_queue);
            }
        }
        else
          return;
    }
    /* USER CODE END USART1_IRQn 0 */
    /* USER CODE BEGIN USART1_IRQn 1 */
    //数据帧接收完毕
    if ( LL_USART_IsActiveFlag_IDLE(USART1 ) == SET )
    {
        /*写入缓冲区完毕*/
        cbWriteFinish(&rx_queue);
        LL_USART_ClearFlag_IDLE(USART1);
    }
    /* USER CODE END USART1_IRQn 1 */
}
```
# 修改主函数
- 在`main.c`文件添加头文件引用

```c
/* USER CODE BEGIN Includes */
#include "my_process_data.h"
/* USER CODE END Includes */
```
- 定义变量

```c
/* USER CODE BEGIN PV */
Channel ch[2];
/* USER CODE END PV */
```

- 初始化数据队列

```c
     /*初始化接收数据队列*/
    RX_Queue_Init();
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
# 调试
- 编译下载
- 验证
- 发送数据 `5A A5 06 83 10 10 01 00 01`会看到LED1亮
- 发送数据 `5A A5 06 83 10 10 01 00 02`会看到LED1灭
- 发送数据 `5A A5 06 83 11 10 01 00 01`会看到LED2亮
- 发送数据 `5A A5 06 83 11 10 01 00 02`会看到LED2灭
- 发送数据 `5A A5 06 83 10 10 01 00 01 5A A5 06 83 11 10 01 00 01`会看到LED1亮LED2亮，这个验证了如果上位机两帧发送过快，系统仍然可以正常处理数据，而不是出现个别帧不处理
- 结果和HAL库一致