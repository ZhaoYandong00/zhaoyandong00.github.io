---
title: STM32硬件库HAL库串口通信使用DMA发送/接收不定长数据
categories: STM32 HAL USART DMA
tags: STM32 CUBE HAL USART DMA
description: HAL库串口通信之DMA
---
# 配置DMA
- 打开CUBE工程

## 在**`Connectivity`**->**`USART1`**配置
- **`DMA_Settings`**
- 配置USART1_RX 优先级——`High` 模式——`Normal` 存储器地址自增 外设不自增 数据字节——`Byte`
- 配置USART1_TX 优先级——`High` 模式——`Normal` 存储器地址自增 外设不自增 数据字节——`Byte`

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# 修改代码
- 把接收`HAL_UART_Receive_IT`都改为`HAL_UART_Receive_DMA`

## 修改串口中断函数

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
            rx_queue.elems[(rx_queue.write_using)&(rx_queue.size - 1)]->len=QUEUE_NODE_DATA_LEN-__HAL_DMA_GET_COUNTER(&hdma_usart1_rx);
            cbWriteFinish(&rx_queue);
        }
        __HAL_UART_CLEAR_IDLEFLAG(&huart1);
        HAL_UART_AbortReceive(&huart1);
        data_p = cbWrite(&rx_queue);
        if(data_p!=NULL)
        {
            HAL_UART_Receive_DMA(&huart1,data_p->head,QUEUE_NODE_DATA_LEN);
        }
        else
        {
            HAL_UART_Receive_DMA(&huart1,&rx_data,1);
        }
    }
    /* USER CODE END USART1_IRQn 1 */
}
```
## 修改传口接收完成回调函数
- 修改初始化读取

```c
data_p = cbWrite(&rx_queue);
HAL_UART_Receive_DMA(&huart1,data_p->head,QUEUE_NODE_DATA_LEN);
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
            HAL_UART_Receive_DMA(huart,data_p->head,QUEUE_NODE_DATA_LEN);
        }
        else
        {
            HAL_UART_Receive_DMA(huart,&rx_data,1);
        }
    }
}
```
# 调试
- 编译下载
- 验证
- 和原来的串口通信效果一样

# 发送
- 原来SPL库时我们全是用的阻塞模式发送，DMA也是用的阻塞，这就会造成发送数据如果过长，程序无法正常执行
- HAL库默认DMA使用中断模式，如果发送太多，回导致后续发送失败
- 我们在环形对列队基础上，添加发送环形队列定义

## 修改环形队列
- `my_data_queue.h`

```c
#ifndef __ESP_DATA_QUEUE_H_
#define __ESP_DATA_QUEUE_H_
#include "main.h"
#include <string.h>
#include <stdio.h>
//缓冲队列的个数需要为2的幂
#define QUEUE_NODE_NUM (4)             //缓冲队列的大小（有多少个缓冲区）
#define QUEUE_NODE_DATA_LEN (128) //单个接收缓冲区大小

#define TX_QUEUE_NODE_DATA_LEN (20) //单个发送缓冲区大小
//数据主体
typedef struct
{
    uint8_t *head;   //缓冲区头指针
    uint16_t len; //接收到的数据长度

} QUEUE_DATA_TYPE;

//队列结构
typedef struct
{
    int size;                               /* 缓冲区大小          */
    int read;                               /* 读指针              */
    int write;                              /* 写指针  */
    int read_using;                         /*正在读取的缓冲区指针*/
    int write_using;                        /*正在写入的缓冲区指针*/
    QUEUE_DATA_TYPE *elems[QUEUE_NODE_NUM]; /* 缓冲区地址                   */
} QueueBuffer;

extern QueueBuffer rx_queue;
extern QueueBuffer tx_queue;//添加发送队列

QUEUE_DATA_TYPE *cbWrite(QueueBuffer *cb);
QUEUE_DATA_TYPE *cbRead(QueueBuffer *cb);
void cbReadFinish(QueueBuffer *cb);
void cbWriteFinish(QueueBuffer *cb);
int cbIsFull(QueueBuffer *cb);
int cbIsEmpty(QueueBuffer *cb);
void RX_Queue_Init(void);
#endif

```

- `my_data_queue.c`

```c
#include "my_data_queue.h"

//实例化节点数据类型
QUEUE_DATA_TYPE node_data[QUEUE_NODE_NUM];
//实例化队列类型
QueueBuffer rx_queue;

//队列缓冲区的内存池
__align(4) uint8_t node_buff[QUEUE_NODE_NUM][QUEUE_NODE_DATA_LEN];

//添加发送队列
//实例化节点数据类型
QUEUE_DATA_TYPE tx_node_data[QUEUE_NODE_NUM];
//实例化队列类型
QueueBuffer tx_queue;

//队列缓冲区的内存池
__align(4) uint8_t tx_node_buff[QUEUE_NODE_NUM][TX_QUEUE_NODE_DATA_LEN];
/*环形缓冲队列*/

/**
  * @brief  初始化缓冲队列
  * @param  cb:缓冲队列结构体
  * @param  size: 缓冲队列的元素个数
  * @note 	初始化时还需要给cb->elems指针赋值
  */
void cbInit(QueueBuffer *cb, int size)
{
    cb->size = size; /* maximum number of elements           */
    cb->read = 0;    /* index of oldest element              */
    cb->write = 0;   /* index at which to write new element  */
    //    cb->elems = (uint8_t *)calloc(cb->size, sizeof(uint8_t));  //elems 要额外初始化
}

/**
  * @brief  判断缓冲队列是(1)否(0)已满
  * @param  cb:缓冲队列结构体
  */
int cbIsFull(QueueBuffer *cb)
{
    return cb->write == (cb->read ^ cb->size); /* This inverts the most significant bit of read before comparison */
}

/**
  * @brief  判断缓冲队列是(1)否(0)全空
  * @param  cb:缓冲队列结构体
  */
int cbIsEmpty(QueueBuffer *cb)
{
    return cb->write == cb->read;
}

/**
  * @brief  对缓冲队列的指针加1
  * @param  cb:缓冲队列结构体
  * @param  p：要加1的指针
  * @return  返回加1的结果
  */
int cbIncr(QueueBuffer *cb, int p)
{
    return (p + 1) & (2 * cb->size - 1); /* read and write pointers incrementation is done modulo 2*size */
}

/**
  * @brief  获取可写入的缓冲区指针
  * @param  cb:缓冲队列结构体
  * @return  可进行写入的缓冲区指针
  * @note  得到指针后可进入写入操作，但写指针不会立即加1，
           写完数据时，应调用cbWriteFinish对写指针加1
  */
QUEUE_DATA_TYPE *cbWrite(QueueBuffer *cb)
{
    if (cbIsFull(cb)) /* full, overwrite moves read pointer */
    {
        return NULL;
    }
    else
    {
        //当wriet和write_using相等时，表示上一个缓冲区已写入完毕，需要对写指针加1
        if (cb->write == cb->write_using)
        {
            cb->write_using = cbIncr(cb, cb->write); //未满，则增加1
        }
    }

    return cb->elems[cb->write_using & (cb->size - 1)];
}

/**
  * @brief 数据写入完毕，更新写指针到缓冲结构体
  * @param  cb:缓冲队列结构体
  */
void cbWriteFinish(QueueBuffer *cb)
{
    cb->write = cb->write_using;
}

/**
  * @brief  获取可读取的缓冲区指针
  * @param  cb:缓冲队列结构体
  * @return  可进行读取的缓冲区指针
  * @note  得到指针后可进入读取操作，但读指针不会立即加1，
					 读取完数据时，应调用cbReadFinish对读指针加1
  */
QUEUE_DATA_TYPE *cbRead(QueueBuffer *cb)
{
    if (cbIsEmpty(cb))
        return NULL;

    //当read和read_using相等时，表示上一个缓冲区已读取完毕(即已调用cbReadFinish)，
    //需要对写指针加1
    if (cb->read == cb->read_using)
        cb->read_using = cbIncr(cb, cb->read);

    return cb->elems[cb->read_using & (cb->size - 1)];
}

/**
  * @brief 数据读取完毕，更新读指针到缓冲结构体
  * @param  cb:缓冲队列结构体
  */
void cbReadFinish(QueueBuffer *cb)
{
    //重置当前读完的数据节点的长度
    cb->elems[cb->read_using & (cb->size - 1)]->len = 0;
    cb->read = cb->read_using;
}

/**
  * @brief  缓冲队列初始化，分配内存,使用缓冲队列时，
  * @param  无
  * @retval 无
  */
void RX_Queue_Init(void)
{
    uint32_t i = 0;

    memset(node_data, 0, sizeof(node_data));
    memset(tx_node_data, 0, sizeof(tx_node_data));
    /*初始化缓冲队列*/
    cbInit(&rx_queue, QUEUE_NODE_NUM);
    /*初始化缓冲队列*/
    cbInit(&tx_queue, QUEUE_NODE_NUM);
    for (i = 0; i < QUEUE_NODE_NUM; i++)
    {
        node_data[i].head = node_buff[i];
        /*初始化队列缓冲指针，指向实际的内存*/
        rx_queue.elems[i] = &node_data[i];
        memset(node_data[i].head, 0, QUEUE_NODE_DATA_LEN);

        tx_node_data[i].head = tx_node_buff[i];
        /*初始化队列缓冲指针，指向实际的内存*/
        tx_queue.elems[i] = &tx_node_data[i];
        memset(tx_node_data[i].head, 0, TX_QUEUE_NODE_DATA_LEN);
    }
}

```
## 修改数据处理
- 发送数据数组

```c
uint8_t send_data[TX_QUEUE_NODE_DATA_LEN]= {0x5A,0xA5,0x05,0x82,0x10,0x10,0x00,0x01};
```
- 添加串口发送函数
- 在`my_process_data.c`中添加定义

```c
/**
  * @brief  串口数据发送
  * @param  huart
  * @retval 无
  */
void Send_Usart_Data(UART_HandleTypeDef *huart)
{
    QUEUE_DATA_TYPE *tx_data;

    /*从缓冲区读取数据，进行处理，*/
    tx_data = cbRead(&tx_queue);
    if (tx_data != NULL) //缓冲队列非空
    {
        //如正在忙，下次发送
        if(HAL_UART_Transmit_DMA(huart,tx_data->head,tx_data->len)==HAL_BUSY)
        {
            return;
        }
        //使用完数据必须调用cbReadFinish更新读指针
        cbReadFinish(&tx_queue);
    }
}
/**
  * @brief  串口数据发送
  * @param  channel 通道
  * @param  addr 地址
  * @param  data 数据
  * @retval 无
  */
void Send_Channel_Data(uint8_t channel,uint8_t addr,uint16_t data)
{
    QUEUE_DATA_TYPE *tx_data;
    send_data[4]=channel;
    send_data[5]=addr;
    send_data[6]=(data&0xFF00)>>8;
    send_data[7]=data&0xFF;
    tx_data=cbWrite(&tx_queue);
    if(tx_data!=NULL)
    {
        tx_data->len=8;
        memcpy(tx_data->head,send_data,8);
        cbWriteFinish(&tx_queue);
    }
}
```
- 在`my_process_data.h`中添加声明

```c
void Send_Usart_Data(UART_HandleTypeDef *huart);
void Send_Channel_Data(uint8_t channel,uint8_t addr,uint16_t data);
```
## 修改主循环
```c
    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        if( Key_Scan(KEY1_GPIO_Port,KEY1_Pin) == KEY_ON  )
        {
            /*LED1反转*/
            LED1_TOGGLE;
        }

        if( Key_Scan(KEY2_GPIO_Port,KEY2_Pin) == KEY_ON  )
        {
            LED2_TOGGLE;
        }
        Process_Usart_Data(ch);
        Send_Usart_Data(&huart1);//添加发送函数
        if(ch[0].state==1&&ch[0].start_up==0)
        {
            ch[0].stop_up=0;
            ch[0].start_up=1;
            LED1(ON);
            Send_Channel_Data(CHANNEL_A ,ADDR_SWITCH,ch->state);//返回开关状态
        }
        if(ch[0].state==2&&ch[0].stop_up==0)
        {
            ch[0].stop_up=1;
            ch[0].start_up=0;
            LED1(OFF);
            Send_Channel_Data(CHANNEL_A ,ADDR_SWITCH,ch->state);
        }
        if(ch[1].state==1&&ch[1].start_up==0)
        {
            ch[1].stop_up=0;
            ch[1].start_up=1;
            LED2(ON);
            Send_Channel_Data(CHANNEL_B ,ADDR_SWITCH,ch->state);
        }
        if(ch[1].state==2&&ch[1].stop_up==0)
        {
            ch[1].stop_up=1;
            ch[1].start_up=0;
            LED2(OFF);
            Send_Channel_Data(CHANNEL_B ,ADDR_SWITCH,ch->state);
        }
        /* USER CODE END WHILE */
```

## 调试
- 编译下载
- 验证
- 发送数据 `5A A5 06 83 10 10 01 00 01 5A A5 06 83 11 10 01 00 01`会看到LED1亮LED2亮，串口会回传`5A A5 05 82 10 10 00 01 5A A5 05 82 11 10 00 01`，证明发送正常，如果不用队列直接发送，如果上一次没发送完成，出现正在发送，这时要么一直等待发送完成，要么取消这次发送
