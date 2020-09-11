---
title: STM32标准外设库SPI读写FLASH SPL篇18
categories: STM32 SPL
tags: STM32 SPL
description: SPL库SPI读写FLASH
---
# 参考文档
- [W25Q64](https://www.winbond.com/resource-files/w25q64jv%20revj%2003272018%20plus.pdf)

# 新建工程
## 添加库函数
- 打开 `Manage Run-Time Environment`
- 选择`Device->StdPeriph Drivers->SPI`
- 选择`Device->StdPeriph Drivers->GPIO`
- 选择`Device->StdPeriph Drivers->USART`

##　添加工程文件
- 复制SPL库时的`my_spi.c`、`my_spi.h`、`my_gpio.c`、`my_gpio.h`、`my_usart.c`、`my_usart.h`到工程
- 以上文件添加到工程

## 编写Flash芯片接口函数
- 开发板上使用的flash芯片为W25Q64
- 新建`my_W25Qx.c`,`my_W25Qx.h`
- `my_W25Qx.h

```c
#ifndef __W25Qx_H
#define __W25Qx_H
#include "my_spi.h"

#define  sFLASH_ID              0XEF4017    //W25Q64
#define SPI_FLASH_PageSize              256
#define SPI_FLASH_PerWritePageSize      256
/*命令定义-开头*******************************/
#define W25X_WriteEnable		      0x06 
#define W25X_WriteDisable		      0x04 
#define W25X_ReadStatusReg		    0x05 
#define W25X_WriteStatusReg		    0x01 
#define W25X_ReadData			        0x03 
#define W25X_FastReadData		      0x0B 
#define W25X_FastReadDual		      0x3B 
#define W25X_PageProgram		      0x02 
#define W25X_BlockErase			      0xD8 
#define W25X_SectorErase		      0x20 
#define W25X_ChipErase			      0xC7 
#define W25X_PowerDown			      0xB9 
#define W25X_ReleasePowerDown	    0xAB 
#define W25X_DeviceID			        0xAB 
#define W25X_ManufactDeviceID   	0x90 
#define W25X_JedecDeviceID		    0x9F

/* WIP(busy)标志，FLASH内部正在写入 */
#define WIP_Flag                  0x01
#define Dummy_Byte                0xFF
/*命令定义-结尾*******************************/

void W25Q_SectorErase(u32 SectorAddr);
void W25Q_BulkErase(void);
void W25Q_PageWrite(u8* pBuffer, u32 WriteAddr, u16 NumByteToWrite);
void W25Q_BufferWrite(u8* pBuffer, u32 WriteAddr, u16 NumByteToWrite);
void W25Q_BufferRead(u8* pBuffer, u32 ReadAddr, u16 NumByteToRead);
uint32_t W25Q_ReadID(void);
uint32_t W25Q_ReadDeviceID(void);

void SPI_FLASH_WriteEnable(void);
void SPI_FLASH_WaitForWriteEnd(void);
void W25Q_PowerDown(void);
void W25Q_WAKEUP(void);

#endif

```
- `my_W25Qx.c`

```c
#include "my_W25Qx.h"

//进入掉电模式
void W25Q_PowerDown(void)
{
    /* 通讯开始：CS低 */
    SPI_FLASH_CS_LOW();

    /* 发送 掉电 命令 */
    SPI_FLASH_SendByte(W25X_PowerDown);

    /*通讯结束：CS高 */
    SPI_FLASH_CS_HIGH();
}

//唤醒
void W25Q_WAKEUP(void)
{
    /*选择 FLASH: CS 低 */
    SPI_FLASH_CS_LOW();

    /* 发送 上电 命令 */
    SPI_FLASH_SendByte(W25X_ReleasePowerDown);

    /* 停止信号 FLASH: CS 高 */
    SPI_FLASH_CS_HIGH();
}
/**
 * @brief  向FLASH发送 写使能 命令
 * @param  none
 * @retval none
 */
void W25Q_WriteEnable(void)
{
    /* 通讯开始：CS低 */
    SPI_FLASH_CS_LOW();

    /* 发送写使能命令*/
    SPI_FLASH_SendByte(W25X_WriteEnable);

    /*通讯结束：CS高 */
    SPI_FLASH_CS_HIGH();
}
/**
 * @brief  等待WIP(BUSY)标志被置0，即等待到FLASH内部数据写入完毕
 * @param  none
 * @retval none
 */
void W25Q_WaitForWriteEnd(void)
{
    uint8_t FLASH_Status = 0;

    /* 选择 FLASH: CS 低 */
    SPI_FLASH_CS_LOW();

    /* 发送 读状态寄存器 命令 */
    SPI_FLASH_SendByte(W25X_ReadStatusReg);

    /* 若FLASH忙碌，则等待 */
    do
    {
        /* 读取FLASH芯片的状态寄存器 */
        FLASH_Status = SPI_FLASH_SendByte(Dummy_Byte);
    }
    while ((FLASH_Status & WIP_Flag) == SET);  /* 正在写入标志 */

    /* 停止信号  FLASH: CS 高 */
    SPI_FLASH_CS_HIGH();
}

/**
 * @brief  擦除FLASH扇区
 * @param  SectorAddr：要擦除的扇区地址
 * @retval 无
 */
void W25Q_SectorErase(uint32_t SectorAddr)
{
    /* 发送写使能命令 */
    W25Q_WriteEnable();
    W25Q_WaitForWriteEnd();
    /* 擦除扇区 */
    /* 选择FLASH: CS低电平 */
    SPI_FLASH_CS_LOW();
    /* 发送扇区擦除指令*/
    SPI_FLASH_SendByte(W25X_SectorErase);
    /*发送擦除扇区地址的高位*/
    SPI_FLASH_SendByte((SectorAddr & 0xFF0000) >> 16);
    /* 发送擦除扇区地址的中位 */
    SPI_FLASH_SendByte((SectorAddr & 0xFF00) >> 8);
    /* 发送擦除扇区地址的低位 */
    SPI_FLASH_SendByte(SectorAddr & 0xFF);
    /* 停止信号 FLASH: CS 高电平 */
    SPI_FLASH_CS_HIGH();
    /* 等待擦除完毕*/
    W25Q_WaitForWriteEnd();
}
/**
 * @brief  擦除FLASH扇区，整片擦除
 * @param  无
 * @retval 无
 */
void SPI_FLASH_BulkErase(void)
{
    /* 发送FLASH写使能命令 */
    W25Q_WriteEnable();

    /* 整块 Erase */
    /* 选择FLASH: CS低电平 */
    SPI_FLASH_CS_LOW();
    /* 发送整块擦除指令*/
    SPI_FLASH_SendByte(W25X_ChipErase);
    /* 停止信号 FLASH: CS 高电平 */
    SPI_FLASH_CS_HIGH();

    /* 等待擦除完毕*/
    W25Q_WaitForWriteEnd();
}
/**
  * @brief  对FLASH按页写入数据，调用本函数写入数据前需要先擦除扇区
  * @param	pBuffer，要写入数据的指针
  * @param WriteAddr，写入地址
  * @param  NumByteToWrite，写入数据长度，必须小于等于SPI_FLASH_PerWritePageSize
  * @retval 无
  */
void W25Q_PageWrite(uint8_t* pBuffer, uint32_t WriteAddr, uint16_t NumByteToWrite)
{
    /* 发送FLASH写使能命令 */
    W25Q_WriteEnable();

    /* 选择FLASH: CS低电平 */
    SPI_FLASH_CS_LOW();
    /* 写页写指令*/
    SPI_FLASH_SendByte(W25X_PageProgram);
    /*发送写地址的高位*/
    SPI_FLASH_SendByte((WriteAddr & 0xFF0000) >> 16);
    /*发送写地址的中位*/
    SPI_FLASH_SendByte((WriteAddr & 0xFF00) >> 8);
    /*发送写地址的低位*/
    SPI_FLASH_SendByte(WriteAddr & 0xFF);

    if(NumByteToWrite > SPI_FLASH_PerWritePageSize)
    {
        NumByteToWrite = SPI_FLASH_PerWritePageSize;
    }

    /* 写入数据*/
    while (NumByteToWrite--)
    {
        /* 发送当前要写入的字节数据 */
        SPI_FLASH_SendByte(*pBuffer);
        /* 指向下一字节数据 */
        pBuffer++;
    }

    /* 停止信号 FLASH: CS 高电平 */
    SPI_FLASH_CS_HIGH();

    /* 等待写入完毕*/
    W25Q_WaitForWriteEnd();
}
/**
 * @brief  对FLASH写入数据，调用本函数写入数据前需要先擦除扇区
 * @param	pBuffer，要写入数据的指针
 * @param  WriteAddr，写入地址
 * @param  NumByteToWrite，写入数据长度
 * @retval 无
 */
void W25Q_BufferWrite(uint8_t* pBuffer, uint32_t WriteAddr, uint16_t NumByteToWrite)
{
    uint8_t NumOfPage = 0, NumOfSingle = 0, Addr = 0, count = 0, temp = 0;

    /*mod运算求余，若writeAddr是SPI_FLASH_PageSize整数倍，运算结果Addr值为0*/
    Addr = WriteAddr % SPI_FLASH_PageSize;

    /*差count个数据值，刚好可以对齐到页地址*/
    count = SPI_FLASH_PageSize - Addr;
    /*计算出要写多少整数页*/
    NumOfPage =  NumByteToWrite / SPI_FLASH_PageSize;
    /*mod运算求余，计算出剩余不满一页的字节数*/
    NumOfSingle = NumByteToWrite % SPI_FLASH_PageSize;

    /* Addr=0,则WriteAddr 刚好按页对齐 aligned  */
    if (Addr == 0)
    {
        /* NumByteToWrite < SPI_FLASH_PageSize */
        if (NumOfPage == 0)
        {
            W25Q_PageWrite(pBuffer, WriteAddr, NumByteToWrite);
        }
        else /* NumByteToWrite > SPI_FLASH_PageSize */
        {
            /*先把整数页都写了*/
            while (NumOfPage--)
            {
                W25Q_PageWrite(pBuffer, WriteAddr, SPI_FLASH_PageSize);
                WriteAddr +=  SPI_FLASH_PageSize;
                pBuffer += SPI_FLASH_PageSize;
            }
            /*若有多余的不满一页的数据，把它写完*/
            W25Q_PageWrite(pBuffer, WriteAddr, NumOfSingle);
        }
    }
    /* 若地址与 SPI_FLASH_PageSize 不对齐  */
    else
    {
        /* NumByteToWrite < SPI_FLASH_PageSize */
        if (NumOfPage == 0)
        {
            /*当前页剩余的count个位置比NumOfSingle小，一页写不完*/
            if (NumOfSingle > count)
            {
                temp = NumOfSingle - count;
                /*先写满当前页*/
                W25Q_PageWrite(pBuffer, WriteAddr, count);

                WriteAddr +=  count;
                pBuffer += count;
                /*再写剩余的数据*/
                W25Q_PageWrite(pBuffer, WriteAddr, temp);
            }
            else /*当前页剩余的count个位置能写完NumOfSingle个数据*/
            {
                W25Q_PageWrite(pBuffer, WriteAddr, NumByteToWrite);
            }
        }
        else /* NumByteToWrite > SPI_FLASH_PageSize */
        {
            /*地址不对齐多出的count分开处理，不加入这个运算*/
            NumByteToWrite -= count;
            NumOfPage =  NumByteToWrite / SPI_FLASH_PageSize;
            NumOfSingle = NumByteToWrite % SPI_FLASH_PageSize;

            /* 先写完count个数据，为的是让下一次要写的地址对齐 */
            W25Q_PageWrite(pBuffer, WriteAddr, count);

            /* 接下来就重复地址对齐的情况 */
            WriteAddr +=  count;
            pBuffer += count;
            /*把整数页都写了*/
            while (NumOfPage--)
            {
                W25Q_PageWrite(pBuffer, WriteAddr, SPI_FLASH_PageSize);
                WriteAddr +=  SPI_FLASH_PageSize;
                pBuffer += SPI_FLASH_PageSize;
            }
            /*若有多余的不满一页的数据，把它写完*/
            if (NumOfSingle != 0)
            {
                W25Q_PageWrite(pBuffer, WriteAddr, NumOfSingle);
            }
        }
    }
}
/**
 * @brief  读取FLASH数据
 * @param 	pBuffer，存储读出数据的指针
 * @param   ReadAddr，读取地址
 * @param   NumByteToRead，读取数据长度
 * @retval 无
 */
void W25Q_BufferRead(uint8_t* pBuffer, uint32_t ReadAddr, uint16_t NumByteToRead)
{
    /* 选择FLASH: CS低电平 */
    SPI_FLASH_CS_LOW();

    /* 发送 读 指令 */
    SPI_FLASH_SendByte(W25X_ReadData);

    /* 发送 读 地址高位 */
    SPI_FLASH_SendByte((ReadAddr & 0xFF0000) >> 16);
    /* 发送 读 地址中位 */
    SPI_FLASH_SendByte((ReadAddr& 0xFF00) >> 8);
    /* 发送 读 地址低位 */
    SPI_FLASH_SendByte(ReadAddr & 0xFF);

    /* 读取数据 */
    while (NumByteToRead--) /* while there is data to be read */
    {
        /* 读取一个字节*/
        *pBuffer = SPI_FLASH_SendByte(Dummy_Byte);
        /* 指向下一个字节缓冲区 */
        pBuffer++;
    }

    /* 停止信号 FLASH: CS 高电平 */
    SPI_FLASH_CS_HIGH();
}
/**
  * @brief  读取FLASH ID
  * @param 	无
  * @retval FLASH ID
  */
uint32_t W25Q_ReadID(void)
{
    uint32_t Temp = 0, Temp0 = 0, Temp1 = 0, Temp2 = 0;

    /* 开始通讯：CS低电平 */
    SPI_FLASH_CS_LOW();

    /* 发送JEDEC指令，读取ID */
    SPI_FLASH_SendByte(W25X_JedecDeviceID);

    /* 读取一个字节数据 */
    Temp0 = SPI_FLASH_SendByte(Dummy_Byte);

    /* 读取一个字节数据 */
    Temp1 = SPI_FLASH_SendByte(Dummy_Byte);

    /* 读取一个字节数据 */
    Temp2 = SPI_FLASH_SendByte(Dummy_Byte);

    /* 停止通讯：CS高电平 */
    SPI_FLASH_CS_HIGH();

    /*把数据组合起来，作为函数的返回值*/
    Temp = (Temp0 << 16) | (Temp1 << 8) | Temp2;

    return Temp;
}
/**
  * @brief  读取FLASH Device ID
  * @param 	无
  * @retval FLASH Device ID
  */
uint32_t W25Q_ReadDeviceID(void)
{
    uint32_t Temp = 0;

    /* Select the FLASH: Chip Select low */
    SPI_FLASH_CS_LOW();

    /* Send "RDID " instruction */
    SPI_FLASH_SendByte(W25X_DeviceID);
    SPI_FLASH_SendByte(Dummy_Byte);
    SPI_FLASH_SendByte(Dummy_Byte);
    SPI_FLASH_SendByte(Dummy_Byte);

    /* Read a byte from the FLASH */
    Temp = SPI_FLASH_SendByte(Dummy_Byte);

    /* Deselect the FLASH: Chip Select high */
    SPI_FLASH_CS_HIGH();

    return Temp;
}

```
# FLASH读写测试
- `main.c`

```c
#include "my_gpio.h"
#include "my_usart.h"
#include "my_spi.h"
#include "my_W25Qx.h"

//读取的ID存储位置
__IO uint32_t DeviceID = 0;
__IO uint32_t FlashID = 0;
// 函数原型声明
void Delay(__IO uint32_t nCount);
/**
 * @brief  板级外设初始化，所有板子上的初始化均可放在这个函数里面
 * @param  无
 * @retval 无
 */
static void BSP_Init(void)
{
    NVIC_PriorityGroupConfig( NVIC_PriorityGroup_4 );
    /* LED 初始化 */
    LED_GPIO_Config();
    /* 串口初始化	*/
    USART_Config();
    /*SPI初始化 */
    SPI_FLASH_Init();
}
int main(void)
{
    uint8_t cal_flag = 0;
    uint8_t k;

    /*存储小数和整数的数组，各7个*/
    long double double_buffer[7] = {0};
    int int_bufffer[7] = {0};
    BSP_Init();
    /* 获取 Flash Device ID */
    DeviceID = W25Q_ReadDeviceID();
    Delay( 200 );

    /* 获取 SPI Flash ID */
    FlashID = W25Q_ReadID();
    printf("\r\nFlashID is 0x%X,  Manufacturer Device ID is 0x%X\r\n", FlashID, DeviceID);
    /* 检验 SPI Flash ID */
    if (FlashID == sFLASH_ID)
    {
        LED2(ON);

        printf("\r\nSPI FLASH W25Q64!\r\n");

        /*读取数据标志位*/
        W25Q_BufferRead(&cal_flag, SPI_FLASH_PageSize*0, 1);

        if( cal_flag == 0xCD )	/*若标志等于0xcd，表示之前已有写入数据*/
        {
            printf("\r\ncal flag\r\n");

            /*读取小数数据*/
            W25Q_BufferRead((void*)double_buffer, SPI_FLASH_PageSize*1, sizeof(double_buffer));

            /*读取整数数据*/
            W25Q_BufferRead((void*)int_bufffer, SPI_FLASH_PageSize*2, sizeof(int_bufffer));


          printf("\r\ndata:\r\n");
            for( k=0; k<7; k++ )
            {
                printf("double rx = %LF \r\n",double_buffer[k]);
                printf("int rx = %d \r\n",int_bufffer[k]);
            }
        }
        else
        {
            printf("\r\nno cal\r\n");
            cal_flag =0xCD;
            /*擦除扇区*/
            W25Q_SectorErase(0);

            /*写入标志到第0页*/
            W25Q_BufferWrite(&cal_flag, SPI_FLASH_PageSize*0, 1);

            /*生成要写入的数据*/
            for( k=0; k<7; k++ )
            {
                double_buffer[k] = k +0.1;
                int_bufffer[k]=k*500+1 ;
            }

            /*写入小数数据到第一页*/
            W25Q_BufferWrite((void*)double_buffer, SPI_FLASH_PageSize*1, sizeof(double_buffer));
            /*写入整数数据到第二页*/
            W25Q_BufferWrite((void*)int_bufffer, SPI_FLASH_PageSize*2, sizeof(int_bufffer));

            printf("write data:");
            /*打印到串口*/
            for( k=0; k<7; k++ )
            {
                printf("double tx = %LF\r\n",double_buffer[k]);
                printf("int tx = %d\r\n",int_bufffer[k]);
            }

            printf("\r\nplese reset!\r\n");
        }

    }// if (FlashID == sFLASH_ID)
    else
    {
        LED1(ON);
        printf("\r\ncannot find W25Q64 ID!\n\r");
    }

    W25Q_PowerDown();
    while(1);
}
void Delay(__IO uint32_t nCount)
{
    for(; nCount != 0; nCount--);
}

```

# 调试
- 编译之后下载到开发板
- 串口调试助手可看到FLASH 测试的调试信息