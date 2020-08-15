---
title: STM32硬件层HAL库串口通信
categories: STM32 CUBE HAL USART
tags: STM32 CUBE HAL USART
description: HAL库串口通信
---
# 配置串口
- 打开CUBE工程

## 在**`Connectivity`**->**`USART1`**配置
- 模式选项：
`Disable`——关闭 `Asynchronous`——异步模式 `Synchronous`——同步模式 `Single Wire(Half-Duplex)`——单线模式(半双工)
`Multiprocessor Communication`——多处理器通讯 `IrDA`——红外数据通信模式 `LIN`——(局域互联网)模式 `SmartCard`——智能卡模式
- 我们选择异步模式`Asynchronous`
- `Hardware Flow Control`选择关闭,即硬件流控关闭
- **`Parameter Settings`**配置为波特率9600，长度8位，无校验位，停止位1，数据方向：接收和发送
- **`NVIC Settings`**使能中断

## 在**`System Core`**->**`NVIC`**配置
- 设置**`USART1 global interrupt`**子优先级为1

## 生成代码
- 点击**`GENERATE CODE`**自动生成代码
- 等待生成完成

# 串口函数
## 串口结构体分析

```c
/**
  * @brief  UART handle Structure definition
  */
typedef struct __UART_HandleTypeDef
{
  USART_TypeDef                 *Instance;        /*!< UART registers base address        */

  UART_InitTypeDef              Init;             /*!< UART communication parameters      */

  uint8_t                       *pTxBuffPtr;      /*!< Pointer to UART Tx transfer Buffer */

  uint16_t                      TxXferSize;       /*!< UART Tx Transfer size              */

  __IO uint16_t                 TxXferCount;      /*!< UART Tx Transfer Counter           */

  uint8_t                       *pRxBuffPtr;      /*!< Pointer to UART Rx transfer Buffer */

  uint16_t                      RxXferSize;       /*!< UART Rx Transfer size              */

  __IO uint16_t                 RxXferCount;      /*!< UART Rx Transfer Counter           */

  DMA_HandleTypeDef             *hdmatx;          /*!< UART Tx DMA Handle parameters      */

  DMA_HandleTypeDef             *hdmarx;          /*!< UART Rx DMA Handle parameters      */

  HAL_LockTypeDef               Lock;             /*!< Locking object                     */

  __IO HAL_UART_StateTypeDef    gState;           /*!< UART state information related to global Handle management and also related to Tx operations.This parameter can be a value of @ref HAL_UART_StateTypeDef */

  __IO HAL_UART_StateTypeDef    RxState;          /*!< UART state information related to Rx operations.This parameter can be a value of @ref HAL_UART_StateTypeDef */

  __IO uint32_t                 ErrorCode;        /*!< UART Error code                    */

#if (USE_HAL_UART_REGISTER_CALLBACKS == 1)
  void (* TxHalfCpltCallback)(struct __UART_HandleTypeDef *huart);        /*!< UART Tx Half Complete Callback        */
  void (* TxCpltCallback)(struct __UART_HandleTypeDef *huart);            /*!< UART Tx Complete Callback             */
  void (* RxHalfCpltCallback)(struct __UART_HandleTypeDef *huart);        /*!< UART Rx Half Complete Callback        */
  void (* RxCpltCallback)(struct __UART_HandleTypeDef *huart);            /*!< UART Rx Complete Callback             */
  void (* ErrorCallback)(struct __UART_HandleTypeDef *huart);             /*!< UART Error Callback                   */
  void (* AbortCpltCallback)(struct __UART_HandleTypeDef *huart);         /*!< UART Abort Complete Callback          */
  void (* AbortTransmitCpltCallback)(struct __UART_HandleTypeDef *huart); /*!< UART Abort Transmit Complete Callback */
  void (* AbortReceiveCpltCallback)(struct __UART_HandleTypeDef *huart);  /*!< UART Abort Receive Complete Callback  */
  void (* WakeupCallback)(struct __UART_HandleTypeDef *huart);            /*!< UART Wakeup Callback                  */

  void (* MspInitCallback)(struct __UART_HandleTypeDef *huart);           /*!< UART Msp Init callback                */
  void (* MspDeInitCallback)(struct __UART_HandleTypeDef *huart);         /*!< UART Msp DeInit callback              */
#endif  /* USE_HAL_UART_REGISTER_CALLBACKS */

} UART_HandleTypeDef;
```
- 串口寄存器基本地址`Instance`

可选参数：**`USART1`** **`USART2`** **`USART3`** **`UART4`** **`UART5`**

- 串口初始化结构体`Init`
- 发送缓冲指针`pTxBuffPtr`
- 发送数据长度`TxXferSize`
- 发送数据计数`TxXferCount`
- 接收缓冲指针`pRxBuffPtr `
- 接收数据长度`RxXferSize`
- 接收数据计数`RxXferCount`
- 发送DMA`hdmatx`
- 接收DMA`hdmarx`
- 锁`Lock`

```c
/**
  * @brief  HAL Lock structures definition
  */
typedef enum
{
  HAL_UNLOCKED = 0x00U,//未锁
  HAL_LOCKED   = 0x01U //锁定
} HAL_LockTypeDef;
```
- 串口全局状态`gState`
- 串口接收状态`RxState`

```c
/**
  * @brief HAL UART State structures definition
  * @note  HAL UART State value is a combination of 2 different substates: gState and RxState.
  *        - gState contains UART state information related to global Handle management
  *          and also information related to Tx operations.
  *          gState value coding follow below described bitmap :
  *          b7-b6  Error information
  *             00 : No Error
  *             01 : (Not Used)
  *             10 : Timeout
  *             11 : Error
  *          b5     Peripheral initialization status
  *             0  : Reset (Peripheral not initialized)
  *             1  : Init done (Peripheral not initialized. HAL UART Init function already called)
  *          b4-b3  (not used)
  *             xx : Should be set to 00
  *          b2     Intrinsic process state
  *             0  : Ready
  *             1  : Busy (Peripheral busy with some configuration or internal operations)
  *          b1     (not used)
  *             x  : Should be set to 0
  *          b0     Tx state
  *             0  : Ready (no Tx operation ongoing)
  *             1  : Busy (Tx operation ongoing)
  *        - RxState contains information related to Rx operations.
  *          RxState value coding follow below described bitmap :
  *          b7-b6  (not used)
  *             xx : Should be set to 00
  *          b5     Peripheral initialization status
  *             0  : Reset (Peripheral not initialized)
  *             1  : Init done (Peripheral not initialized)
  *          b4-b2  (not used)
  *            xxx : Should be set to 000
  *          b1     Rx state
  *             0  : Ready (no Rx operation ongoing)
  *             1  : Busy (Rx operation ongoing)
  *          b0     (not used)
  *             x  : Should be set to 0.
  */
typedef enum
{
  HAL_UART_STATE_RESET             = 0x00U,    /*!< Peripheral is not yet Initialized Value is allowed for gState and RxState *///外设尚未初始化
  HAL_UART_STATE_READY             = 0x20U,    /*!< Peripheral Initialized and ready for use Value is allowed for gState and RxState *///外设初始化完成和准备好
  HAL_UART_STATE_BUSY              = 0x24U,    /*!< an internal process is ongoing Value is allowed for gState only *///正在忙
  HAL_UART_STATE_BUSY_TX           = 0x21U,    /*!< Data Transmission process is ongoing Value is allowed for gState only *///发送忙
  HAL_UART_STATE_BUSY_RX           = 0x22U,    /*!< Data Reception process is ongoing Value is allowed for RxState only *///s接收忙
  HAL_UART_STATE_BUSY_TX_RX        = 0x23U,    /*!< Data Transmission and Reception process is ongoing Not to be used for neither gState nor RxState.Value is result of combination (Or) between gState and RxState values *///发送和接收忙
  HAL_UART_STATE_TIMEOUT           = 0xA0U,    /*!< Timeout state Value is allowed for gState only *///超时
  HAL_UART_STATE_ERROR             = 0xE0U     /*!< Error Value is allowed for gState only *///错误
} HAL_UART_StateTypeDef;
```
- 错误码`ErrorCode`

```c
/** @defgroup UART_Error_Code UART Error Code
  * @{
  */
#define HAL_UART_ERROR_NONE              0x00000000U   /*!< No error            *///无错误
#define HAL_UART_ERROR_PE                0x00000001U   /*!< Parity error        *///检验错误
#define HAL_UART_ERROR_NE                0x00000002U   /*!< Noise error         *///噪声错误
#define HAL_UART_ERROR_FE                0x00000004U   /*!< Frame error         *///帧错误
#define HAL_UART_ERROR_ORE               0x00000008U   /*!< Overrun error       *///溢出错误
#define HAL_UART_ERROR_DMA               0x00000010U   /*!< DMA transfer error  *///DMA传输错误
#if (USE_HAL_UART_REGISTER_CALLBACKS == 1)
#define  HAL_UART_ERROR_INVALID_CALLBACK 0x00000020U   /*!< Invalid Callback error  *///无效的回调
#endif /* USE_HAL_UART_REGISTER_CALLBACKS */
```
- 发送完成一半回调函数`TxHalfCpltCallback`
- 发送完成回调函数`TxCpltCallback`
- 接收完成一半回调函数`RxHalfCpltCallback`
- 发送完成回调函数`RxCpltCallback`
- 错误回调函数`ErrorCallback`
- 中止完成回调函数`AbortCpltCallback`
- 中止发送完成回调函数`AbortTransmitCpltCallback`
- 中止接收完成回调函数`AbortReceiveCpltCallback`
- 唤醒回调函数`WakeupCallback`
- 初始化回调函数`MspInitCallback`
- 反初始化回调函数`MspDeInitCallback`

## 串口初始化结构体分析

```c
/**
  * @brief UART Init Structure definition
  */
typedef struct
{
  uint32_t BaudRate;                  /*!< This member configures the UART communication baud rate. The baud rate is computed using the following formula:
  - IntegerDivider = ((PCLKx) / (16 * (huart->Init.BaudRate)))
  - FractionalDivider = ((IntegerDivider - ((uint32_t) IntegerDivider)) * 16) + 0.5 */

  uint32_t WordLength;                /*!< Specifies the number of data bits transmitted or received in a frame.This parameter can be a value of @ref UART_Word_Length */

  uint32_t StopBits;                  /*!< Specifies the number of stop bits transmitted. This parameter can be a value of @ref UART_Stop_Bits */

  uint32_t Parity;                    /*!< Specifies the parity mode.This parameter can be a value of @ref UART_Parity @note When parity is enabled, the computed parity is inserted at the MSB position of the transmitted data (9th bit when the word length is set to 9 data bits; 8th bit when the word length is set to 8 data bits). */

  uint32_t Mode;                      /*!< Specifies whether the Receive or Transmit mode is enabled or disabled.This parameter can be a value of @ref UART_Mode */

  uint32_t HwFlowCtl;                 /*!< Specifies whether the hardware flow control mode is enabled or disabled.This parameter can be a value of @ref UART_Hardware_Flow_Control */

  uint32_t OverSampling;              /*!< Specifies whether the Over sampling 8 is enabled or disabled, to achieve higher speed (up to fPCLK/8). This parameter can be a value of @ref UART_Over_Sampling. This feature is only available on STM32F100xx family, so OverSampling parameter should always be set to 16. */
} UART_InitTypeDef;
```

- 波特率 `BaudRate`

$ IntegerDivider = \frac{PCLKx}{16 \times huart \to Init.BaudRate}$

$ FractionalDivider = ((IntegerDivider - ((uint32_t) IntegerDivider)) \times 16) + 0.5$

如：时钟频率为72M, 波特率为9600

则Divider=72000000/16/9600=468.75

IntegerDivider=468=0x1D4

FractionalDivider=0.75*16+0.5=12.5=0x0C

USART_BRR 0x1D4C

- 字长 `WordLength`

``` c
/** @defgroup UART_Word_Length UART Word Length
  * @{
  */
#define UART_WORDLENGTH_8B                  0x00000000U //8位
#define UART_WORDLENGTH_9B                  ((uint32_t)USART_CR1_M) //9位
```

- 停止位 `StopBits`

``` c
/** @defgroup UART_Stop_Bits UART Number of Stop Bits
  * @{
  */
#define UART_STOPBITS_1                     0x00000000U //1个停止位
#define UART_STOPBITS_2                     ((uint32_t)USART_CR2_STOP_1) //2个停止位
```

- 校验位 `Parity`
宏定义如下：

``` c
/** @defgroup UART_Parity UART Parity
  * @{
  */
#define UART_PARITY_NONE                    0x00000000U  //无校验
#define UART_PARITY_EVEN                    ((uint32_t)USART_CR1_PCE) //偶校验
#define UART_PARITY_ODD                     ((uint32_t)(USART_CR1_PCE | USART_CR1_PS)) //奇校验
```

- 模式选择 `Mode`

``` c
/** @defgroup UART_Mode UART Transfer Mode
  * @{
  */
#define UART_MODE_RX                        ((uint32_t)USART_CR1_RE) //接收模式
#define UART_MODE_TX                        ((uint32_t)USART_CR1_TE) //发送模式
#define UART_MODE_TX_RX                     ((uint32_t)(USART_CR1_TE | USART_CR1_RE)) //接收和发送模式
```

- 硬件流控制选择 `HwFlowCtl`

``` c
/** @defgroup UART_Hardware_Flow_Control UART Hardware Flow Control
  * @{
  */
#define UART_HWCONTROL_NONE                  0x00000000U //不使能硬件流
#define UART_HWCONTROL_RTS                   ((uint32_t)USART_CR3_RTSE) //使能RTS
#define UART_HWCONTROL_CTS                   ((uint32_t)USART_CR3_CTSE) //使能CTS
#define UART_HWCONTROL_RTS_CTS               ((uint32_t)(USART_CR3_RTSE | USART_CR3_CTSE)) //同时使能RTS 和CTS
```
- 串口过采样`OverSampling`

```c
/** @defgroup UART_Over_Sampling UART Over Sampling
  * @{
  */
#define UART_OVERSAMPLING_16                    0x00000000U //过采样参数16
#if defined(USART_CR1_OVER8)
#define UART_OVERSAMPLING_8                     ((uint32_t)USART_CR1_OVER8) //过采样参数8
#endif /* USART_CR1_OVER8 */
```

## DMA结构体分析
 
```c
/** 
  * @brief  DMA handle Structure definition
  */
typedef struct __DMA_HandleTypeDef
{
  DMA_Channel_TypeDef   *Instance;                       /*!< Register base address                  */
  
  DMA_InitTypeDef       Init;                            /*!< DMA communication parameters           */ 
  
  HAL_LockTypeDef       Lock;                            /*!< DMA locking object                     */  
  
  HAL_DMA_StateTypeDef  State;                           /*!< DMA transfer state                     */
  
  void                  *Parent;                                                      /*!< Parent object state                    */  
  
  void                  (* XferCpltCallback)( struct __DMA_HandleTypeDef * hdma);     /*!< DMA transfer complete callback         */
  
  void                  (* XferHalfCpltCallback)( struct __DMA_HandleTypeDef * hdma); /*!< DMA Half transfer complete callback    */
  
  void                  (* XferErrorCallback)( struct __DMA_HandleTypeDef * hdma);    /*!< DMA transfer error callback            */

  void                  (* XferAbortCallback)( struct __DMA_HandleTypeDef * hdma);    /*!< DMA transfer abort callback            */  
  
  __IO uint32_t         ErrorCode;                                                    /*!< DMA Error code                         */

  DMA_TypeDef            *DmaBaseAddress;                                             /*!< DMA Channel Base Address               */
  
  uint32_t               ChannelIndex;                                                /*!< DMA Channel Index                      */  

} DMA_HandleTypeDef; 
```
- DMA通道基地址`Instance`

可选参数：**`DMA1_Channel1`** **`DMA1_Channel2`** **`DMA1_Channel3`** **`DMA1_Channel4`** **`DMA1_Channel5`** **`DMA1_Channel6`** **`DMA1_Channel7`** **`DMA2_Channel1`** **`DMA2_Channel2`** **`DMA2_Channel3`** **`DMA2_Channel4`** **`DMA2_Channel5`**
- DMA初始化结构体`Init`
- 锁`Lock`
- DMA状态`State`

```c
/**
  * @brief  HAL DMA State structures definition
  */
typedef enum
{
  HAL_DMA_STATE_RESET             = 0x00U,  /*!< DMA not yet initialized or disabled    *///未初始化
  HAL_DMA_STATE_READY             = 0x01U,  /*!< DMA initialized and ready for use      *///初始化完成和准备好
  HAL_DMA_STATE_BUSY              = 0x02U,  /*!< DMA process is ongoing                 *///忙
  HAL_DMA_STATE_TIMEOUT           = 0x03U   /*!< DMA timeout state                      *///超时
}HAL_DMA_StateTypeDef;
```
- 父对象即DMA处理的外设`Parent`
- 发送完成回调函数`XferCpltCallback`
- 发送完成一半回调函数`XferHalfCpltCallback`
- 错误回调函数`XferErrorCallback`
- 中止回调函数`XferAbortCallback`
- 错误码`ErrorCode`

```c
/** @defgroup DMA_Error_Code DMA Error Code
  * @{
  */
#define HAL_DMA_ERROR_NONE                     0x00000000U    /*!< No error             *///无错误
#define HAL_DMA_ERROR_TE                       0x00000001U    /*!< Transfer error       *///传输错误
#define HAL_DMA_ERROR_NO_XFER                  0x00000004U    /*!< no ongoing transfer  *///无持续传输
#define HAL_DMA_ERROR_TIMEOUT                  0x00000020U    /*!< Timeout error        *///超时
#define HAL_DMA_ERROR_NOT_SUPPORTED            0x00000100U    /*!< Not supported mode *///不支持的模式
```
- DMA基地址`DmaBaseAddress`
- DMA通道号`ChannelIndex`

## DMA初始化结构体分析

```c
/**
  * @brief  DMA Configuration Structure definition
  */
typedef struct
{
  uint32_t Direction;                 /*!< Specifies if the data will be transferred from memory to peripheral, from memory to memory or from peripheral to memory.This parameter can be a value of @ref DMA_Data_transfer_direction */

  uint32_t PeriphInc;                 /*!< Specifies whether the Peripheral address register should be incremented or not.This parameter can be a value of @ref DMA_Peripheral_incremented_mode */

  uint32_t MemInc;                    /*!< Specifies whether the memory address register should be incremented or not.This parameter can be a value of @ref DMA_Memory_incremented_mode */

  uint32_t PeriphDataAlignment;       /*!< Specifies the Peripheral data width.This parameter can be a value of @ref DMA_Peripheral_data_size */

  uint32_t MemDataAlignment;          /*!< Specifies the Memory data width. This parameter can be a value of @ref DMA_Memory_data_size */

  uint32_t Mode;                      /*!< Specifies the operation mode of the DMAy Channelx.This parameter can be a value of @ref DMA_mode @note The circular buffer mode cannot be used if the memory-to-memory data transfer is configured on the selected Channel */

  uint32_t Priority;                  /*!< Specifies the software priority for the DMAy Channelx.This parameter can be a value of @ref DMA_Priority_level */
} DMA_InitTypeDef;
```
- 方向`Direction`

```c
/** @defgroup DMA_Data_transfer_direction DMA Data transfer direction
  * @{
  */
#define DMA_PERIPH_TO_MEMORY         0x00000000U                 /*!< Peripheral to memory direction *///外设到存储器
#define DMA_MEMORY_TO_PERIPH         ((uint32_t)DMA_CCR_DIR)     /*!< Memory to peripheral direction *//存储器到外设
#define DMA_MEMORY_TO_MEMORY         ((uint32_t)DMA_CCR_MEM2MEM) /*!< Memory to memory direction     *///存储器到存储器
```
- 外设地址自增`PeriphInc`

```c
/** @defgroup DMA_Peripheral_incremented_mode DMA Peripheral incremented mode
  * @{
  */
#define DMA_PINC_ENABLE        ((uint32_t)DMA_CCR_PINC)  /*!< Peripheral increment mode Enable *///使能
#define DMA_PINC_DISABLE       0x00000000U               /*!< Peripheral increment mode Disable *///禁止
```
- 存储器地址自增`MemInc`

```c
/** @defgroup DMA_Memory_incremented_mode DMA Memory incremented mode
  * @{
  */
#define DMA_MINC_ENABLE         ((uint32_t)DMA_CCR_MINC)  /*!< Memory increment mode Enable  *///使能
#define DMA_MINC_DISABLE        0x00000000U               /*!< Memory increment mode Disable *///禁止
```
- 外设数据长度`PeriphDataAlignment`

```c
/** @defgroup DMA_Peripheral_data_size DMA Peripheral data size
  * @{
  */
#define DMA_PDATAALIGN_BYTE          0x00000000U                  /*!< Peripheral data alignment: Byte     *///字节8位
#define DMA_PDATAALIGN_HALFWORD      ((uint32_t)DMA_CCR_PSIZE_0)  /*!< Peripheral data alignment: HalfWord *///半字16位
#define DMA_PDATAALIGN_WORD          ((uint32_t)DMA_CCR_PSIZE_1)  /*!< Peripheral data alignment: Word     *///字32位
```
- 存储器数据长度`MemDataAlignment`

```c
/** @defgroup DMA_Memory_data_size DMA Memory data size
  * @{
  */
#define DMA_MDATAALIGN_BYTE          0x00000000U                  /*!< Memory data alignment: Byte     *///字节8位
#define DMA_MDATAALIGN_HALFWORD      ((uint32_t)DMA_CCR_MSIZE_0)  /*!< Memory data alignment: HalfWord *///半字16位
#define DMA_MDATAALIGN_WORD          ((uint32_t)DMA_CCR_MSIZE_1)  /*!< Memory data alignment: Word     *///字32位
```
- 模式`Mode`

```c
/** @defgroup DMA_mode DMA mode
  * @{
  */
#define DMA_NORMAL         0x00000000U                  /*!< Normal mode *///单次
#define DMA_CIRCULAR       ((uint32_t)DMA_CCR_CIRC)     /*!< Circular mode *///循环
```
- 优先级`Priority`

```c
/** @defgroup DMA_Priority_level DMA Priority level
  * @{
  */
#define DMA_PRIORITY_LOW             0x00000000U               /*!< Priority level : Low       *///低
#define DMA_PRIORITY_MEDIUM          ((uint32_t)DMA_CCR_PL_0)  /*!< Priority level : Medium    *///中
#define DMA_PRIORITY_HIGH            ((uint32_t)DMA_CCR_PL_1)  /*!< Priority level : High      *///高
#define DMA_PRIORITY_VERY_HIGH       ((uint32_t)DMA_CCR_PL)    /*!< Priority level : Very_High *///非常高
```
## 常用库函数解析

- 获取标志位`__HAL_UART_GET_FLAG(__HANDLE__, __FLAG__)`

可获取标志位：

**`UART_FLAG_CTS`**——CTS变化标志 
**`UART_FLAG_LBD`**——LIN断开检测标志 
**`UART_FLAG_TXE`**——发送数据寄存器空 
**`UART_FLAG_TC`**——发送完成 
**`UART_FLAG_RXNE`**——读数据寄存器非空 
**`UART_FLAG_IDLE`**——监测到总线空闲 
**`UART_FLAG_ORE`**——过载错误 
**`UART_FLAG_NE`**——噪声错误标志 
**`UART_FLAG_FE`**——帧错误 
**`UART_FLAG_PE`**——校验错误 

- 清除标志位`__HAL_UART_CLEAR_FLAG(__HANDLE__, __FLAG__)`

可清除标志位:

**`UART_FLAG_CTS`**——CTS变化标志
**`UART_FLAG_LBD`**——LIN断开检测标志
**`UART_FLAG_TC`**——发送完成
**`UART_FLAG_RXNE`**——读数据寄存器非空

剩余标志位是由软件序列清除，需要先读SR,再读DR清除

- 清除总线空闲标志`__HAL_UART_CLEAR_IDLEFLAG(__HANDLE__)`

内部执行的是先读SR,再读DR
- 使能中断`__HAL_UART_ENABLE_IT(__HANDLE__, __INTERRUPT__)`
- 获取中断标志`__HAL_UART_GET_IT_SOURCE(__HANDLE__, __IT__)`

可选中断：
**`UART_IT_CTS`**——CTS变化中断
**`UART_IT_LBD`**——LIN断开检测中断
**`UART_IT_TXE`**——发送数据寄存器空中断
**`UART_IT_TC`**——发送完成中断
**`UART_IT_RXNE`**——读数据寄存器非空中断
**`UART_IT_IDLE`**——总线空闲中断
**`UART_IT_PE`**——校验错误中断
**`UART_IT_ERR`**——错误中断(帧错误,噪声错误标,过载错误)

- 阻塞模式发送函数`HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)`

程序会一直发，直到发送完毕或超时，才会进行下一步程序

- 阻塞模式接收函数`HAL_StatusTypeDef HAL_UART_Receive(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)`

程序会一直收，直到接收到指定的数量的数据或超时，才会进行下一步程序

- 中断模式发送函数`HAL_StatusTypeDef HAL_UART_Transmit_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)`

发送完成会调用发送完成回调函数,下次发送一定要等回调函数调用后再发，否则会无法发送

- 中断模式接收函数`HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)`

接收到指定数量的数据后会调用接收完成回调函数，但是一定小心，如果接收的数超过了指定数量可能会产生过载错误，需要在错误回调函数里处理，否则会出现无法再进入中断的问题

- DMA发送函数`HAL_StatusTypeDef HAL_UART_Transmit_DMA(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)`

单次模式：完成后使能发送完成中断`TCIE`，中断里调用发送完成回调函数
循环模式：完成直接调用发送完成回调函数

- DMA接收函数`HAL_StatusTypeDef HAL_UART_Receive_DMA(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size)`

单次模式：完成后先清除RXNE, PE ,ERR中断位,取消DMA接收请求，接收状态，再调用接收完成回调函数
循环模式：完成直接调用接收完成回调函数

- 阻塞模式停止串口发送和接收`HAL_StatusTypeDef HAL_UART_Abort(UART_HandleTypeDef *huart)`

- 阻塞模式停止串口发送`HAL_StatusTypeDef HAL_UART_AbortTransmit(UART_HandleTypeDef *huart)`

- 阻塞模式停止串口接收`HAL_StatusTypeDef HAL_UART_AbortTransmit(UART_HandleTypeDef *huart)`

- 中断模式停止串口发送和接收`HAL_StatusTypeDef HAL_UART_Abort_IT(UART_HandleTypeDef *huart)`

完成后调用停止完成回调函数

- 中断模式停止串口发送`HAL_StatusTypeDef HAL_UART_AbortTransmit_IT(UART_HandleTypeDef *huart)`

完成后调用停止发送完成回调函数

- 中断模式停止串口接收`HAL_StatusTypeDef HAL_UART_AbortReceive_IT(UART_HandleTypeDef *huart)`

完成后调用停止接收完成回调函数

# 代码移植
## 使能空闲中断

- 在`HAL_UART_MspInit`函数中添加使能空闲
```c
__HAL_UART_ENABLE_IT(uartHandle, UART_IT_IDLE);
```

## 中断函数添加空闲中断判断
- 在`stm32f1xx_it.c`文件添加外部变量

```c
/* USER CODE BEGIN EV */
extern uint8_t rx_data[50];
extern uint8_t rx_len;
extern uint8_t rx_data1[50];
extern uint8_t rx_n;
/* USER CODE END EV */
```
- 在`stm32f1xx_it.c`文件`USART1_IRQHandler`函数里添加

```c
    /* USER CODE BEGIN USART1_IRQn 1 */
    if((__HAL_UART_GET_FLAG(&huart1,UART_FLAG_IDLE)!=RESET) &&(__HAL_UART_GET_IT_SOURCE(&huart1,UART_IT_IDLE)!=RESET))
    {
        rx_len=huart1.RxXferSize-huart1.RxXferCount;
        if(rx_n==1)
        {
            HAL_UART_Transmit(&huart1,rx_data1,50,0xffff);
        }
        if(rx_len>0)
        {
            HAL_UART_Transmit(&huart1,rx_data,rx_len,0xffff);
        }
        rx_len+=rx_n*50;
        HAL_UART_Transmit(&huart1,&rx_len,1,0xffff);
        rx_n=0;
        __HAL_UART_CLEAR_IDLEFLAG(&huart1);
        HAL_UART_AbortReceive(&huart1);//如果不是接收完成，需要手动中止上一次接收，才能开始下一次接收，不然无法开启新的接收
        HAL_UART_Receive_IT(&huart1,rx_data,50);
    }
    /* USER CODE END USART1_IRQn 1 */
```

## 开始接收数据
- 在`main.c`文件添加头文件

```c
/* USER CODE BEGIN Includes */
#include "string.h"
/* USER CODE END Includes */
```
- 在`main.c`文件添加变量

```c
/* USER CODE BEGIN PV */
uint8_t rx_data[50];
uint8_t rx_data1[50];
uint8_t rx_n=0;
uint8_t rx_len=0;
/* USER CODE END PV */
```
- 在主函数初始化添加使能接收

```c
    /* USER CODE BEGIN 2 */
    /* 使能接收，进入中断回调函数 */
    HAL_UART_Receive_IT(&huart1,rx_data,50);
    /* USER CODE END 2 */
```
- 添加串口回调函数

```c
/**
  * @brief  串口接收完成回调函数
  * @param  huart  串口结构体指针
  * @retval None
  */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance==USART1)
    {
			memcpy(rx_data1,rx_data,50); //接收到的数据有50个，先把数据存起来
			rx_n=1; //接收到50个数据，
      HAL_UART_Receive_IT(huart,rx_data,50);//继续接收
    }
}
```
# 调试
- 编译之后下载到开发板
- 连接开发板串口
- 打开串口助手
- 向开发板发送数据，将会看到开发板返回完整数据和长度
- 因为我们开了空闲中断，里边进行了中断的清除，即先读SR,再读DR,会把溢出错误也清除掉，我们就不用写溢出错误回调函数去消除溢出错误了
