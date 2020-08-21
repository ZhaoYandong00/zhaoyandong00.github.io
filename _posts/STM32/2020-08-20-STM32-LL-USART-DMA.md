---
title: STM32底层LL库库串口通信使用DMA发送/接收不定长数据
categories: STM32 LL USART DMA
tags: STM32 CUBE KL USART DMA
description: LL库串口通信之DMA
---
# 配置DMA
- 和HAL库配置一样
- 在高级设置里为**DMA**选择`LL`库
- 生成代码

#  DMA函数
## DMA结构体分析

```c
/** @defgroup DMA_LL_ES_INIT DMA Exported Init structure
  * @{
  */
typedef struct
{
  uint32_t PeriphOrM2MSrcAddress;  /*!< Specifies the peripheral base address for DMA transfer
                                        or as Source base address in case of memory to memory transfer direction.

                                        This parameter must be a value between Min_Data = 0 and Max_Data = 0xFFFFFFFF. */

  uint32_t MemoryOrM2MDstAddress;  /*!< Specifies the memory base address for DMA transfer
                                        or as Destination base address in case of memory to memory transfer direction.

                                        This parameter must be a value between Min_Data = 0 and Max_Data = 0xFFFFFFFF. */

  uint32_t Direction;              /*!< Specifies if the data will be transferred from memory to peripheral,
                                        from memory to memory or from peripheral to memory.
                                        This parameter can be a value of @ref DMA_LL_EC_DIRECTION

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetDataTransferDirection(). */

  uint32_t Mode;                   /*!< Specifies the normal or circular operation mode.
                                        This parameter can be a value of @ref DMA_LL_EC_MODE
                                        @note: The circular buffer mode cannot be used if the memory to memory
                                               data transfer direction is configured on the selected Channel

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetMode(). */

  uint32_t PeriphOrM2MSrcIncMode;  /*!< Specifies whether the Peripheral address or Source address in case of memory to memory transfer direction
                                        is incremented or not.
                                        This parameter can be a value of @ref DMA_LL_EC_PERIPH

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetPeriphIncMode(). */

  uint32_t MemoryOrM2MDstIncMode;  /*!< Specifies whether the Memory address or Destination address in case of memory to memory transfer direction
                                        is incremented or not.
                                        This parameter can be a value of @ref DMA_LL_EC_MEMORY

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetMemoryIncMode(). */

  uint32_t PeriphOrM2MSrcDataSize; /*!< Specifies the Peripheral data size alignment or Source data size alignment (byte, half word, word)
                                        in case of memory to memory transfer direction.
                                        This parameter can be a value of @ref DMA_LL_EC_PDATAALIGN

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetPeriphSize(). */

  uint32_t MemoryOrM2MDstDataSize; /*!< Specifies the Memory data size alignment or Destination data size alignment (byte, half word, word)
                                        in case of memory to memory transfer direction.
                                        This parameter can be a value of @ref DMA_LL_EC_MDATAALIGN

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetMemorySize(). */

  uint32_t NbData;                 /*!< Specifies the number of data to transfer, in data unit.
                                        The data unit is equal to the source buffer configuration set in PeripheralSize
                                        or MemorySize parameters depending in the transfer direction.
                                        This parameter must be a value between Min_Data = 0 and Max_Data = 0x0000FFFF

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetDataLength(). */

  uint32_t Priority;               /*!< Specifies the channel priority level.
                                        This parameter can be a value of @ref DMA_LL_EC_PRIORITY

                                        This feature can be modified afterwards using unitary function @ref LL_DMA_SetChannelPriorityLevel(). */

} LL_DMA_InitTypeDef;
```
- 外设地址`PeriphOrM2MSrcAddress`
- 存储器地址`MemoryOrM2MDstAddress`
- 方向`Direction`

```c
/** @defgroup DMA_LL_EC_DIRECTION Transfer Direction
  * @{
  */
#define LL_DMA_DIRECTION_PERIPH_TO_MEMORY 0x00000000U             /*!< Peripheral to memory direction */
#define LL_DMA_DIRECTION_MEMORY_TO_PERIPH DMA_CCR_DIR             /*!< Memory to peripheral direction */
#define LL_DMA_DIRECTION_MEMORY_TO_MEMORY DMA_CCR_MEM2MEM         /*!< Memory to memory direction     */
```
- 模式`Mode`

```c
/** @defgroup DMA_LL_EC_MODE Transfer mode
  * @{
  */
#define LL_DMA_MODE_NORMAL                0x00000000U             /*!< Normal Mode                  */
#define LL_DMA_MODE_CIRCULAR              DMA_CCR_CIRC            /*!< Circular Mode                */
```
- 外设地址自增`PeriphOrM2MSrcIncMode`

```c
/** @defgroup DMA_LL_EC_PERIPH Peripheral increment mode
  * @{
  */
#define LL_DMA_PERIPH_INCREMENT           DMA_CCR_PINC            /*!< Peripheral increment mode Enable */
#define LL_DMA_PERIPH_NOINCREMENT         0x00000000U             /*!< Peripheral increment mode Disable */
```
- 存储器地址自增`MemoryOrM2MDstIncMode`

```c
/** @defgroup DMA_LL_EC_MEMORY Memory increment mode
  * @{
  */
#define LL_DMA_MEMORY_INCREMENT           DMA_CCR_MINC            /*!< Memory increment mode Enable  */
#define LL_DMA_MEMORY_NOINCREMENT         0x00000000U             /*!< Memory increment mode Disable */
```
- 外设数据长度`PeriphOrM2MSrcDataSize`

```c
/** @defgroup DMA_LL_EC_PDATAALIGN Peripheral data alignment
  * @{
  */
#define LL_DMA_PDATAALIGN_BYTE            0x00000000U             /*!< Peripheral data alignment : Byte     */
#define LL_DMA_PDATAALIGN_HALFWORD        DMA_CCR_PSIZE_0         /*!< Peripheral data alignment : HalfWord */
#define LL_DMA_PDATAALIGN_WORD            DMA_CCR_PSIZE_1         /*!< Peripheral data alignment : Word     */
```
- 存储器数据长度`MemoryOrM2MDstDataSize`

```c
/** @defgroup DMA_LL_EC_MDATAALIGN Memory data alignment
  * @{
  */
#define LL_DMA_MDATAALIGN_BYTE            0x00000000U             /*!< Memory data alignment : Byte     */
#define LL_DMA_MDATAALIGN_HALFWORD        DMA_CCR_MSIZE_0         /*!< Memory data alignment : HalfWord */
#define LL_DMA_MDATAALIGN_WORD            DMA_CCR_MSIZE_1         /*!< Memory data alignment : Word     */
```
- 数据长度`NbData`
- 优先级`Priority`

```c
/** @defgroup DMA_LL_EC_PRIORITY Transfer Priority level
  * @{
  */
#define LL_DMA_PRIORITY_LOW               0x00000000U             /*!< Priority level : Low       */
#define LL_DMA_PRIORITY_MEDIUM            DMA_CCR_PL_0            /*!< Priority level : Medium    */
#define LL_DMA_PRIORITY_HIGH              DMA_CCR_PL_1            /*!< Priority level : High      */
#define LL_DMA_PRIORITY_VERYHIGH          DMA_CCR_PL              /*!< Priority level : Very_High */
```

## 常用函数

- 使能通道`void LL_DMA_EnableChannel(DMA_TypeDef *DMAx, uint32_t Channel)`
- 关闭通道`void LL_DMA_DisableChannel(DMA_TypeDef *DMAx, uint32_t Channel)`
- 设置数据长度`void LL_DMA_SetDataLength(DMA_TypeDef *DMAx, uint32_t Channel, uint32_t NbData)`
- 获取剩余数据长度`uint32_t LL_DMA_GetDataLength(DMA_TypeDef *DMAx, uint32_t Channel)`
- 设置存储器地址`void LL_DMA_SetMemoryAddress(DMA_TypeDef *DMAx, uint32_t Channel, uint32_t MemoryAddress)`
- 设置外设地址`void LL_DMA_SetPeriphAddress(DMA_TypeDef *DMAx, uint32_t Channel, uint32_t PeriphAddress)`
- 读取TC1标志位`uint32_t LL_DMA_IsActiveFlag_TC1(DMA_TypeDef *DMAx)`
- 读取TC2标志位`uint32_t LL_DMA_IsActiveFlag_TC2(DMA_TypeDef *DMAx)`
- 读取TC3标志位`uint32_t LL_DMA_IsActiveFlag_TC3(DMA_TypeDef *DMAx)`
- 读取TC4标志位`uint32_t LL_DMA_IsActiveFlag_TC4(DMA_TypeDef *DMAx)`
- 读取TC5标志位`uint32_t LL_DMA_IsActiveFlag_TC5(DMA_TypeDef *DMAx)`
- 读取TC6标志位`uint32_t LL_DMA_IsActiveFlag_TC6(DMA_TypeDef *DMAx)`
- 读取TC7标志位`uint32_t LL_DMA_IsActiveFlag_TC7(DMA_TypeDef *DMAx)`
- 清TC1标志位`void LL_DMA_ClearFlag_TC1(DMA_TypeDef *DMAx)`
- 清TC2标志位`void LL_DMA_ClearFlag_TC2(DMA_TypeDef *DMAx)`
- 清TC3标志位`void LL_DMA_ClearFlag_TC3(DMA_TypeDef *DMAx)`
- 清TC4标志位`void LL_DMA_ClearFlag_TC4(DMA_TypeDef *DMAx)`
- 清TC5标志位`void LL_DMA_ClearFlag_TC5(DMA_TypeDef *DMAx)`
- 清TC6标志位`void LL_DMA_ClearFlag_TC6(DMA_TypeDef *DMAx)`
- 清TC7标志位`void LL_DMA_ClearFlag_TC7(DMA_TypeDef *DMAx)`
- 使能TC中断`void LL_DMA_EnableIT_TC(DMA_TypeDef *DMAx, uint32_t Channel)`

# 代码移植
## 修改发送函数
- 在`my_process_data.c`中修改定义

```c
/**
  * @brief  串口数据发送
  * @param  huart
  * @retval 无
  */
void Send_Usart_Data(USART_TypeDef * pUSARTx)
{
    QUEUE_DATA_TYPE *tx_data;
    if(pUSARTx==USART1)
    {
        /*从缓冲区读取数据，进行处理，*/
        tx_data = cbRead(&tx_queue);
        if (tx_data != NULL) //缓冲队列非空
        {
            if((LL_USART_IsActiveFlag_TXE(USART1)!=RESET) && (LL_USART_IsActiveFlag_TC(USART1)!=RESET))
            {
                LL_DMA_DisableChannel(DMA1,LL_DMA_CHANNEL_4);
                LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_4,(uint32_t)tx_data->head);
                LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_4,tx_data->len);
                LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_4);
                //使用完数据必须调用cbReadFinish更新读指针
                cbReadFinish(&tx_queue);
            }
        }
    }
}
```
- 在`my_process_data.h`中修改声明

```c
void Send_Usart_Data(USART_TypeDef *pUSARTx);
```
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
    /* USER CODE BEGIN USART1_IRQn 1 */
    //数据帧接收完毕
    if ( LL_USART_IsActiveFlag_IDLE(USART1 ) == SET )
    {
        LL_DMA_DisableChannel(DMA1,LL_DMA_CHANNEL_5);

        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            data_p->len=QUEUE_NODE_DATA_LEN-LL_DMA_GetDataLength(DMA1,LL_DMA_CHANNEL_5);
            if(data_p->len>0)
            {
                /*写入缓冲区完毕*/
                cbWriteFinish(&rx_queue);
            }
        }
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_5,(uint32_t)data_p->head);
            LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_5,QUEUE_NODE_DATA_LEN);
            LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_5);
        }       
        /*写入缓冲区完成*/
        LL_USART_ClearFlag_IDLE(USART1);
    }
    /* USER CODE END USART1_IRQn 1 */
}
```

## 修改DMA中断函数

```c
/**
  * @brief This function handles DMA1 channel5 global interrupt.
  */
void DMA1_Channel5_IRQHandler(void)
{
    /* USER CODE BEGIN DMA1_Channel5_IRQn 0 */
    QUEUE_DATA_TYPE *data_p;
    if(LL_DMA_IsActiveFlag_TC5(DMA1)!=RESET)
    {
        LL_DMA_ClearFlag_TC5(DMA1);
        LL_DMA_DisableChannel(DMA1,LL_DMA_CHANNEL_5);
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            data_p->len=QUEUE_NODE_DATA_LEN;
            /*写入缓冲区完毕*/
            cbWriteFinish(&rx_queue);
        }
        data_p = cbWrite(&rx_queue);
        if (data_p != NULL) //若缓冲队列未满，开始传输
        {
            LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_5,(uint32_t)data_p->head);
            LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_5,QUEUE_NODE_DATA_LEN);
            LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_5);
        }
    }
    /* USER CODE END DMA1_Channel5_IRQn 0 */

    /* USER CODE BEGIN DMA1_Channel5_IRQn 1 */

    /* USER CODE END DMA1_Channel5_IRQn 1 */
}
```

## 使能空闲中断和DMA中断

```c
    LL_USART_EnableIT_IDLE(USART1);
    QUEUE_DATA_TYPE *data_p;
    data_p = cbWrite(&rx_queue);
    LL_DMA_SetPeriphAddress(DMA1,LL_DMA_CHANNEL_4,LL_USART_DMA_GetRegAddr(USART1));
    LL_DMA_SetPeriphAddress(DMA1,LL_DMA_CHANNEL_5,LL_USART_DMA_GetRegAddr(USART1));
    LL_DMA_SetMemoryAddress(DMA1,LL_DMA_CHANNEL_5,(uint32_t)data_p->head);
    LL_DMA_SetDataLength(DMA1,LL_DMA_CHANNEL_5,QUEUE_NODE_DATA_LEN);
    LL_USART_EnableDMAReq_RX(USART1);
    LL_USART_EnableDMAReq_TX(USART1);
    LL_DMA_EnableChannel(DMA1,LL_DMA_CHANNEL_5);
    LL_DMA_EnableIT_TC(DMA1,LL_DMA_CHANNEL_5);
```
## 添加发送函数到主循环

```c
        Process_Usart_Data(ch);
        Send_Usart_Data(USART1);
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
            Send_Channel_Data(CHANNEL_A ,ADDR_SWITCH,ch->state);//返回开关状态
        }
        if(ch[1].state==1&&ch[1].start_up==0)
        {
            ch[1].stop_up=0;
            ch[1].start_up=1;
            LED2(ON);
            Send_Channel_Data(CHANNEL_B ,ADDR_SWITCH,ch->state);//返回开关状态
        }
        if(ch[1].state==2&&ch[1].stop_up==0)
        {
            ch[1].stop_up=1;
            ch[1].start_up=0;
            LED2(OFF);
            Send_Channel_Data(CHANNEL_B ,ADDR_SWITCH,ch->state);//返回开关状态
        }
```

## 调试
- 编译下载
- 验证
- 发送数据 `5A A5 06 83 10 10 01 00 01 5A A5 06 83 11 10 01 00 01`会看到LED1亮LED2亮，串口会回传`5A A5 05 82 10 10 00 01 5A A5 05 82 11 10 00 01`，证明发送接收正常
- 和HAL库一致
