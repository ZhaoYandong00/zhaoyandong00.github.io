---
title: STM32标准外设库FatFs使用外部FLASH字符转换 SPL篇28
categories: STM32 SPL
tags: STM32 SPL
description: SPL库FatFs使用外部FLASH字符转换
---
# 工程设置
- 把FatFs-SDTOFLASH工程复制一份

# 制作`UNIGBK.BIN`
- 因为FatFs字符转换占用空间太大，为了减少对单片机资源的占用，使用外部FLASH存储这些文件
- 我们提前准备好`UNIGBK.BIN`文件，并烧录的FLASH中
- `UNIGBK`使用C2B软件制作，把`uni2oem936[]``oem2uni936[]`两个数组放到一个txt里,使用C2B软件转换
- [C2B](http://www.openedv.com/thread-284821-1-1.html)

# FatFs使用外部FLASH
- 修改`ffunicode.c`
- 使用外部编码，不编译原来的编码 

```c
//使用外部字符
#define EXTERN_FLASH_CODE_PAGE 
//使用外部的编码
#ifndef EXTERN_FLASH_CODE_PAGE
#if FF_CODE_PAGE == 936 || FF_CODE_PAGE == 0	/* Simplified Chinese */
... 
#endif
#endif
```
- 使用外部编码转换 

```c
#if ((defined EXTERN_FLASH_CODE_PAGE ) && ( FF_CODE_PAGE == 936))
#include "my_file.h"

#define UGBKNAME "UNIGBK.BIN"
#define UGBKSIZE 174344

static WCHAR Extern_FLASH_convert (	/* Converted code, 0 means conversion error */
    WCHAR	src,	/* Character code to be converted */
    UINT	dir		/* 0: Unicode to OEMCP, 1: OEMCP to Unicode */
)
{
    WCHAR c;
    UINT i = 0, n, li, hi;
    /* 资源表在flash中的地址 */
    static int32_t ugbk_addr = -1;
    uint32_t uni2oem_offset=0;
    WCHAR t[2];
    /* 只加载一次 */
    if(ugbk_addr<0)
    {
        ugbk_addr =GetResOffset(UGBKNAME);
        if(ugbk_addr < 0)
        {
            return 0;
        }
        ugbk_addr+=RESOURCE_BASE_ADDR;
    }
    if(dir)	//GBK 2 UNICODE
    {
        uni2oem_offset=UGBKSIZE/2;
    } else	//UNICODE 2 GBK
    {
        uni2oem_offset=0;
    }
    hi=UGBKSIZE/8-1;
    li = 0;
    for (n = 16; n; n--) {
        i = li + (hi - li) / 2;
        W25Q_BufferRead((uint8_t *)&t,uni2oem_offset+i*4+ugbk_addr,4);
        if (src == t[0]) break;
        if (src > t[0]) {
            li = i;
        } else {
            hi = i;
        }
    }
    if (n != 0) c = t[1];
    return c;
}
#endif

#if FF_CODE_PAGE >= 900
WCHAR ff_uni2oem (	/* Returns OEM code character, zero on error */
    DWORD	uni,	/* UTF-16 encoded character to be converted */
    WORD	cp		/* Code page for the conversion */
)
{
#if ((defined EXTERN_FLASH_CODE_PAGE ) && ( FF_CODE_PAGE == 936))
    WCHAR c = 0, uc;
#else
    const WCHAR *p;
    WCHAR c = 0, uc;
    UINT i = 0, n, li, hi;
#endif
    if (uni < 0x80) {	/* ASCII? */
        c = (WCHAR)uni;

    } else {			/* Non-ASCII */
        if (uni < 0x10000 && cp == FF_CODE_PAGE) {	/* Is it in BMP and valid code page? */
            uc = (WCHAR)uni;
#if ((defined EXTERN_FLASH_CODE_PAGE ) && ( FF_CODE_PAGE == 936))
            c=Extern_FLASH_convert(uc,0);
#else
            p = CVTBL(uni2oem, FF_CODE_PAGE);
            hi = sizeof CVTBL(uni2oem, FF_CODE_PAGE) / 4 - 1;
            li = 0;
            for (n = 16; n; n--) {
                i = li + (hi - li) / 2;
                if (uc == p[i * 2]) break;
                if (uc > p[i * 2]) {
                    li = i;
                } else {
                    hi = i;
                }
            }
            if (n != 0) c = p[i * 2 + 1];
#endif

        }
    }

    return c;
}


WCHAR ff_oem2uni (	/* Returns Unicode character in UTF-16, zero on error */
    WCHAR	oem,	/* OEM code to be converted */
    WORD	cp		/* Code page for the conversion */
)
{
#if ((defined EXTERN_FLASH_CODE_PAGE ) && ( FF_CODE_PAGE == 936))
    WCHAR c = 0;
#else
    const WCHAR *p;
    WCHAR c = 0;
    UINT i = 0, n, li, hi;
#endif

    if (oem < 0x80) {	/* ASCII? */
        c = oem;

    } else {			/* Extended char */
        if (cp == FF_CODE_PAGE) {	/* Is it valid code page? */
#if ((defined EXTERN_FLASH_CODE_PAGE ) && ( FF_CODE_PAGE == 936))
            c=Extern_FLASH_convert(oem,1);
#else
            p = CVTBL(oem2uni, FF_CODE_PAGE);
            hi = sizeof CVTBL(oem2uni, FF_CODE_PAGE) / 4 - 1;
            li = 0;
            for (n = 16; n; n--) {
                i = li + (hi - li) / 2;
                if (oem == p[i * 2]) break;
                if (oem > p[i * 2]) {
                    li = i;
                } else {
                    hi = i;
                }
            }
            if (n != 0) c = p[i * 2 + 1];
#endif
        }
    }

    return c;
}
#endif

```
# FatFs使用外部FLASH编码实验
- `main.c`

```c
#include "my_gpio.h"
#include "my_usart.h"
#include "my_SD.h"
#include "ff.h"
#include "my_file.h"

FATFS fs;													/* FatFs文件系统对象 */
FRESULT res_sd;                /* 文件操作结果 */
FIL fnew;
FILINFO fno;
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
    BSP_Init();
    printf("\n*************** SD-FatFs **************\r\n");
    res_sd = f_mount(&fs,SD_ROOT,1);
    if(res_sd!=FR_OK)
    {
        LED1(ON);
        printf("fail res_sd =%d\r\n",res_sd);
        printf("may be:SD init fail\r\n");
        while(1);
    }
    printf("SD init success!\r\n");
    printf("\r\n****** 创建文件... ******\r\n");
    res_sd = f_open(&fnew, "1:中文.txt",FA_CREATE_ALWAYS |  FA_WRITE );
    if ( res_sd == FR_OK )
    {
        /* 不再读写，关闭文件 */
        f_close(&fnew);
    }
    else
    {
        LED1(ON);
        printf("open/creat fail\r\n");
    }
    char path[256];
    strcpy(path,SD_ROOT);
    scan_files(path);
    /*------------------- 文件系统测试：读测试 ------------------------------------*/
    printf("****** 读取文件信息... ******\r\n");

    res_sd = f_stat("1:中文.txt",&fno);
    if(res_sd == FR_OK)
    {
        LED2(ON);
        printf("open success\r\n");
        printf("name:%s ", fno.fname);

    }
    else
    {
        LED1(ON);
        printf("open fail\r\n");
    }
    /* 不再读写，关闭文件 */
    f_close(&fnew);
    /* 不再使用文件系统，取消挂载文件系统 */
    f_mount(NULL,SD_ROOT,1);
    while(1);
}

```
# 调试
- 编译之后下载到开发板
- 打开串口助手,可以看到创建中文名称的文件成功
