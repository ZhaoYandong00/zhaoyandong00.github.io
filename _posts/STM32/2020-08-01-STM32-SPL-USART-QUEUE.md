---
title: STM32标准外设库串口通信之队列SPL篇5
categories: STM32 USART QUEUE
tags: STM32 SPL USART UART QUEUE
description: SPL库串口通信之队列
---
# 添加环形队列文件
- `my_data_queue.h`

```c
#ifndef __ESP_DATA_QUEUE_H_
#define __ESP_DATA_QUEUE_H_
#include "stm32f10x.h"
#include <string.h>
#include <stdio.h>
//缓冲队列的个数需要为2的幂
#define QUEUE_NODE_NUM        (2)            //缓冲队列的大小（有多少个缓冲区）
#define QUEUE_NODE_DATA_LEN   (2*1024 )       //单个接收缓冲区大小

//数据主体
typedef struct
{
    uint8_t  *head; 	//缓冲区头指针
    uint16_t len; //接收到的数据长度

} QUEUE_DATA_TYPE ;

//队列结构
typedef struct {
    int         size;  /* 缓冲区大小          */
    int         read; /* 读指针              */
    int         write;   /* 写指针  */
    int read_using;	/*正在读取的缓冲区指针*/
    int write_using;		/*正在写入的缓冲区指针*/
    QUEUE_DATA_TYPE    *elems[QUEUE_NODE_NUM];  /* 缓冲区地址                   */
} QueueBuffer;

extern QueueBuffer rx_queue;

QUEUE_DATA_TYPE* cbWrite(QueueBuffer *cb);
QUEUE_DATA_TYPE* cbRead(QueueBuffer *cb);
void cbReadFinish(QueueBuffer *cb);
void cbWriteFinish(QueueBuffer *cb);
int cbIsFull(QueueBuffer *cb) ;
int cbIsEmpty(QueueBuffer *cb) ;
void RX_Queue_Init(void);
#endif

```
- `my_data_queue.c`

```c
#include "my_data_queue.h"

//实例化节点数据类型
QUEUE_DATA_TYPE  node_data[QUEUE_NODE_NUM];
//实例化队列类型
QueueBuffer rx_queue;

//队列缓冲区的内存池
__align(4) uint8_t node_buff[QUEUE_NODE_NUM][QUEUE_NODE_DATA_LEN] ;



/*环形缓冲队列*/

/**
  * @brief  初始化缓冲队列
  * @param  cb:缓冲队列结构体
  * @param  size: 缓冲队列的元素个数
  * @note 	初始化时还需要给cb->elems指针赋值
  */
void cbInit(QueueBuffer *cb, int size)
{
    cb->size  = size;	/* maximum number of elements           */
    cb->read = 0; 		/* index of oldest element              */
    cb->write   = 0; 	 	/* index at which to write new element  */
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
    return (p + 1)&(2*cb->size-1); /* read and write pointers incrementation is done modulo 2*size */
}

/**
  * @brief  获取可写入的缓冲区指针
  * @param  cb:缓冲队列结构体
  * @return  可进行写入的缓冲区指针
  * @note  得到指针后可进入写入操作，但写指针不会立即加1，
           写完数据时，应调用cbWriteFinish对写指针加1
  */
QUEUE_DATA_TYPE* cbWrite(QueueBuffer *cb)
{
    if (cbIsFull(cb)) /* full, overwrite moves read pointer */
    {
        return NULL;
    }
    else
    {
        //当wriet和write_using相等时，表示上一个缓冲区已写入完毕，需要对写指针加1
        if(cb->write == cb->write_using)
        {
            cb->write_using = cbIncr(cb, cb->write); //未满，则增加1
        }
    }

    return  cb->elems[cb->write_using&(cb->size-1)];
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
QUEUE_DATA_TYPE* cbRead(QueueBuffer *cb)
{
    if(cbIsEmpty(cb))
        return NULL;

    //当read和read_using相等时，表示上一个缓冲区已读取完毕(即已调用cbReadFinish)，
    //需要对写指针加1
    if(cb->read == cb->read_using)
        cb->read_using = cbIncr(cb, cb->read);

    return cb->elems[cb->read_using&(cb->size-1)];
}


/**
  * @brief 数据读取完毕，更新读指针到缓冲结构体
  * @param  cb:缓冲队列结构体
  */
void cbReadFinish(QueueBuffer *cb)
{
    //重置当前读完的数据节点的长度
    cb->elems[cb->read_using&(cb->size-1)]->len = 0;
    cb->read = cb->read_using;
}



//队列的指针指向的缓冲区全部销毁
void camera_queue_free(void)
{
    uint32_t i = 0;

    for(i = 0; i < QUEUE_NODE_NUM; i ++)
    {
        if(node_data[i].head != NULL)
        {
            //若是动态申请的空间才要free
//            free(node_data[i].head);
            node_data[i].head = NULL;
        }
    }

    return;
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

    /*初始化缓冲队列*/
    cbInit(&rx_queue,QUEUE_NODE_NUM);

    for(i = 0; i < QUEUE_NODE_NUM; i ++)
    {
        node_data[i].head = node_buff[i];

        /*初始化队列缓冲指针，指向实际的内存*/
        rx_queue.elems[i] = &node_data[i];
        memset(node_data[i].head, 0, QUEUE_NODE_DATA_LEN);
    }
}

```
# 更改串口中断函数

```c
/**
  * @brief  串口中断服务函数
  * @param  无
  * @retval 无
  */
void DEBUG_USART_IRQHandler(void)
{
  uint8_t ucCh;
  QUEUE_DATA_TYPE *data_p;

  if (USART_GetITStatus(DEBUG_USARTx, USART_IT_RXNE) != RESET)
  {
    ucCh = USART_ReceiveData(DEBUG_USARTx);
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
  //数据帧接收完毕
  if (USART_GetITStatus(DEBUG_USARTx, USART_IT_IDLE) == SET)
  {
    /*写入缓冲区完毕*/
    cbWriteFinish(&rx_queue);
    //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
    ucCh = USART_ReceiveData(DEBUG_USARTx);
  }
}
```
# 增加串口数据处理文件
- `my_process_data.h`

```c
#ifndef __PROCESS_DATA_H
#define	__PROCESS_DATA_H
#include "stm32f10x.h"
#include "my_data_queue.h"
#include "my_usart.h"
/*********************数据****************************/
#define HEAD_BYTE0    0x5A //接收头低字节
#define HEAD_BYTE1    0xA5 //接收头高字节
#define CMD_READ      0x83 //数据下发命令
typedef struct {
    uint8_t state;//状态
    uint8_t start_up;//状态
    uint8_t stop_up;//状态
    uint16_t strength;//强度
    uint8_t mod;//模式
} Channel;

#define CHANNEL_A           0X10             //通道1
#define CHANNEL_B           0X11             //通道2
#define ADDR_SWITCH         0x10             //开关
#define ADDR_STRENGTH       0x11             //强度
#define ADDR_MOD            0x12             //模式

#define BUTTON_LEN          0x01             //按键自动下发参数长度
#define PARAMETER_LEN       0x03             //参数检测数据长度
/**
  * @brief  串口数据处理
  * @param  通道
  * @retval 无
  */
void Process_Usart_Data(Channel *ch);
#endif

```

- `my_process_data.c`

```c
/**
  ******************************************************************************
  * @file    my_process_data.c
  * @author  
  * @version V1.0
  * @date    
  * @brief   串口数据处理程序
  ******************************************************************************
  * @attention
  *
  * 定义      帧头   数据长度                     指令    数据
  * 数据长度  5A A5  该项右侧全部数据的长度加和    82或83    N
  * 
  * 举例：
  * 82指令：变量地址（word)+ 写入的变量数据(word)
  * 向地址(0x1000)写数据1:5A A5 05 82 10 00 00 01
  * 向地址(0x1000 0x1001)写数据1,2:5A A5 07 82 10 00 00 01 00 02
  * 83指令：变量地址（word)+ 地址数量+变量数据(word)
  * 地址(0x1000)发来的数据1:5A A5 06 83 10 00 01 00 01
  * 地址(0x1000 0x1001)发来的数据1,2:5A A5 08 83 10 00 02 00 01 00 02
  * 
  *
  ******************************************************************************
  */
#include "my_process_data.h"
/**
  * @brief  通道设置
  * @param  ch 通道   cmd 串口数据
  * @retval 无
  */
static void Set_Channel(Channel *ch, uint8_t *cmd)
{
    switch (cmd[5])
    {
    case ADDR_SWITCH:
        switch (cmd[6])
        {
        case BUTTON_LEN:
            ch->state = cmd[8];
            break;
        case PARAMETER_LEN:
            ch->state = cmd[8];
            ch->strength = (((cmd[9] << 8) & 0xFF00) | (cmd[10] & 0xFF));
            ch->mod = cmd[12]; //模式
            break;
        default:
            break;
        }
        break;
    case ADDR_MOD:
        ch->mod = cmd[8];
        break;
    case ADDR_STRENGTH:
        ch->strength = (((cmd[7] << 8) & 0xFF00) | (cmd[8] & 0xFF));
        break;
    default:
        break;
    }
}
/**
  * @brief  通道分配
  * @param  ch 通道   cmd 串口数据
  * @retval 无
  */
static void Sel_Channel(Channel *ch, uint8_t *cmd)
{
    if (cmd[4] == CHANNEL_A)
    {
        Set_Channel(&ch[0], cmd);
    }
    else if (cmd[4] == CHANNEL_B)
    {
        Set_Channel(&ch[1], cmd);
    }
}
/**
  * @brief  串口数据处理
  * @param  通道
  * @retval 无
  */
static void Process_Data(Channel *ch, uint8_t *array, uint16_t data_len)
{
    uint16_t i = 0;
    uint16_t len = 0;
    if (data_len > 8)
    {
        if ((array[0] == HEAD_BYTE0) && (array[1] == HEAD_BYTE1) && (array[3] == CMD_READ) && ((array[6] * 2 + 4) == array[2]) && (data_len >= (array[2] + 3)))
        {
            len = array[2] + 3;
            for (i = 7; i < (data_len - 1); i++)
            {
                if ((array[i] == HEAD_BYTE0) && (array[i + 1] == HEAD_BYTE1))
                {
                    len = i;
                    break;
                }
            }
            if (len >= (array[2] + 3))
            {
                Sel_Channel(ch, array);
            }
            if (data_len > len)
            {
                memmove(array,array+len,data_len - len);
                Process_Data(ch, array, data_len - len);
            }
        }
        else
        {
            if (data_len > 9)
            {
                for (i = 1; i < (data_len - 1); i++)
                {
                    if ((array[i] == HEAD_BYTE0) && (array[i + 1] == HEAD_BYTE1))
                    {
                        len = i;
                        break;
                    }
                }
                if (len > 0)
                {
                    memmove(array,array+len,data_len - len);
                    Process_Data(ch, array, data_len - len);
                }
            }
        }
    }
}
/**
  * @brief  串口数据处理
  * @param  通道
  * @retval 无
  */
void Process_Usart_Data(Channel *ch)
{
    QUEUE_DATA_TYPE *rx_data;
    uint8_t *array;

    /*从缓冲区读取数据，进行处理，*/
    rx_data = cbRead(&rx_queue);
    if (rx_data != NULL) //缓冲队列非空
    {
        array = (uint8_t *)rx_data->head;
        if (rx_data->len > 8)
        {
            if ((array[0] == HEAD_BYTE0) && (array[1] == HEAD_BYTE1) && (array[3] == CMD_READ) && ((array[6] * 2 + 4) == array[2]))
            {
                Process_Data(ch, array, rx_data->len);
            }
        }
        //使用完数据必须调用cbReadFinish更新读指针
        cbReadFinish(&rx_queue);
    }
}

```
# 主函数修改
- `main.c`

```c
#include "stm32f10x.h"
#include "my_gpio.h"
#include "my_usart.h"
#include "my_data_queue.h"
#include "my_process_data.h"
Channel ch[2];
int main(void)
{
    /* LED 端口初始化 */
    LED_GPIO_Config();
    Key_GPIO_Config();
    /* 初始化USART 配置模式为 9600 8-N-1 */
    USART_Config();
    /*初始化接收数据队列*/
    RX_Queue_Init();
    while (1)
    {
        Process_Usart_Data(ch);
        if (Key_Scan(KEY1_GPIO_PORT, KEY1_GPIO_PIN) == KEY_ON)
        {
            /*LED1反转*/
            LED1_TOGGLE;
        }

        if (Key_Scan(KEY2_GPIO_PORT, KEY2_GPIO_PIN) == KEY_ON)
        {
            LED2_TOGGLE;
        }
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