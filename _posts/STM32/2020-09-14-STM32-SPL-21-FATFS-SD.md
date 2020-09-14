---
title: STM32标准外设库FatFs SD卡 SPL篇21
categories: STM32 SPL
tags: STM32 SPL
description: SPL库FatFs SD卡
---
# SD卡FatFs测试
## 工程设置
- 把SD卡工程复制一份
- 把SPI-FLASH FatFs的`source`复制到工程
- 把`source`文件夹下的文件添加到工程

## 实现移植所需函数
- `diskio.c`

```c
#define SD_BLOCKSIZE     512
#include "my_SD.h"
```
```c
DSTATUS disk_status (
    BYTE pdrv		/* Physical drive nmuber to identify the drive */
)
{
    DSTATUS stat;

    switch (pdrv) {

    case DEV_MMC :
        stat=0;

        return stat;
    }
    return STA_NOINIT;
}
```
```c
DSTATUS disk_initialize (
    BYTE pdrv				/* Physical drive nmuber to identify the drive */
)
{
    DSTATUS stat;
    switch (pdrv) {
    case DEV_MMC :
        if(SD_Init()==SD_RESPONSE_NO_ERROR)
        {
            stat=0;
        } else
        {
            stat=STA_NOINIT;
        }
        return stat;
    }
    return STA_NOINIT;
}
```
```c
DRESULT disk_read (
    BYTE pdrv,		/* Physical drive nmuber to identify the drive */
    BYTE *buff,		/* Data buffer to store read data */
    LBA_t sector,	/* Start sector in LBA */
    UINT count		/* Number of sectors to read */
)
{
    DRESULT res;
//    int result;

    switch (pdrv) {
    case DEV_MMC :
        if(SD_ReadMultiBlocks(buff,(uint64_t)sector*SD_BLOCKSIZE,SD_BLOCKSIZE,count)==SD_RESPONSE_NO_ERROR)
            res=RES_OK;
        else
            res=RES_PARERR;
        return res;
    }

    return RES_PARERR;
}
```
```c
DRESULT disk_write (
    BYTE pdrv,			/* Physical drive nmuber to identify the drive */
    const BYTE *buff,	/* Data to be written */
    LBA_t sector,		/* Start sector in LBA */
    UINT count			/* Number of sectors to write */
)
{
    DRESULT res;
    switch (pdrv) {
    case DEV_MMC :
        if(SD_WriteMultiBlocks((uint8_t *)buff,(uint64_t)sector*SD_BLOCKSIZE,SD_BLOCKSIZE,count)==SD_RESPONSE_NO_ERROR)
            res=RES_OK;
        else
            res=RES_PARERR;
        return res;

    }

    return RES_PARERR;
}
```
```c
DRESULT disk_ioctl (
    BYTE pdrv,		/* Physical drive nmuber (0..) */
    BYTE cmd,		/* Control code */
    void *buff		/* Buffer to send/receive control data */
)
{
    DRESULT res;

    switch (pdrv) {

    case DEV_MMC :

        // Process of the command for the MMC/SD card
        switch (cmd)
        {
            // Get R/W sector size (WORD)
        case GET_SECTOR_SIZE :
            *(WORD * )buff = SD_BLOCKSIZE;
            break;
            // Get erase block size in unit of sector (DWORD)
        case GET_BLOCK_SIZE :
            *(DWORD * )buff = 1;
            break;

        case GET_SECTOR_COUNT:
            *(DWORD * )buff = SDCardInfo.CardCapacity/SDCardInfo.CardBlockSize;
            break;
        case CTRL_SYNC :
            break;
        }
        res= RES_OK;
        return res;

    }
    return RES_PARERR;
}
```
## 程序
- `main.c`

```c
#include "my_gpio.h"
#include "my_usart.h"
#include "my_SD.h"
#include "ff.h"
FATFS fs;													/* FatFs文件系统对象 */
FIL fnew;													/* 文件对象 */
FRESULT res_sd;                /* 文件操作结果 */
UINT fnum;            					  /* 文件成功读写数量 */
BYTE ReadBuffer[1024];
BYTE WriteBuffer[] = "SD WRITE & READ\r\n";              /* 写缓冲区*/

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
}

int main(void)
{
    BYTE work[512];
    BSP_Init();
    printf("\n*************** SD-FatFs **************\r\n");
    res_sd = f_mount(&fs,"1:",1);
    if(res_sd==FR_NO_FILESYSTEM)
    {
        printf("sd no file system,format...\r\n");
        res_sd=f_mkfs("1:",0,work,sizeof work);
        if(res_sd == FR_OK)
        {
            printf("format success\r\n");
            /* 格式化后，先取消挂载 */
            res_sd = f_mount(NULL,"0:",1);
            /* 重新挂载	*/
            res_sd = f_mount(&fs,"0:",1);
        }
        else
        {
            LED1(ON);
            printf("format fail,res_sd =%d\r\n",res_sd);
            while(1);
        }
    }
    else if(res_sd!=FR_OK)
    {
        LED1(ON);
        printf("fail res_sd =%d\r\n",res_sd);
        printf("may be:SD init fail\r\n");
        while(1);
    }
    printf("SD init success!\r\n");

    /*----------------------- 文件系统测试：写测试 -----------------------------*/
    /* 打开文件，如果文件不存在则创建它 */
    printf("\r\n****** file check ... ******\r\n");
    res_sd = f_open(&fnew, "1:FatFstest.txt",FA_CREATE_ALWAYS | FA_WRITE );
    if ( res_sd == FR_OK )
    {
        printf("open/creat success\r\n");
        /* 将指定存储区内容写入到文件内 */
        res_sd=f_write(&fnew,WriteBuffer,sizeof WriteBuffer,&fnum);
        if(res_sd==FR_OK)
        {
            printf("write success,len:%d\n",fnum);
            printf("data:\r\n%s\r\n",WriteBuffer);
        }
        else
        {
            printf("write fail:(%d)\n",res_sd);
        }
        /* 不再读写，关闭文件 */
        f_close(&fnew);
    }
    else
    {
        LED1(ON);
        printf("open/creat fail\r\n");
    }

    /*------------------- 文件系统测试：读测试 ------------------------------------*/
    printf("****** read check... ******\r\n");
    res_sd = f_open(&fnew, "1:FatFstest.txt", FA_OPEN_EXISTING | FA_READ);
    if(res_sd == FR_OK)
    {
        LED2(ON);
        printf("open success\r\n");
        res_sd = f_read(&fnew, ReadBuffer, sizeof ReadBuffer, &fnum);
        if(res_sd==FR_OK)
        {
            printf("read success,len:%d\r\n",fnum);
            printf("data:\r\n%s \r\n", ReadBuffer);
        }
        else
        {
            printf("read fail:(%d)\n",res_sd);
        }
    }
    else
    {
        LED1(ON);
        printf("open fail\r\n");
    }
    /* 不再读写，关闭文件 */
    f_close(&fnew);

    /* 不再使用文件系统，取消挂载文件系统 */
    f_mount(NULL,"1:",1);
    while(1);
}

```
## 调试
- 编译之后下载到开发板
- 串口调试助手可看到FatFs测试的调试信息