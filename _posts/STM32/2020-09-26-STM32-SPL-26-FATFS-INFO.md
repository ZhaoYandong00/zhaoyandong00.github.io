---
title: STM32标准外设库遍历SD卡内文件 SPL篇26
categories: STM32 SPL
tags: STM32 SPL
description: SPL库遍历SD卡内文件
---
# 工程设置
- 把FatFs-SD卡工程复制一份


# FatFs-RTC支持
- 库添加选择`Device->StdPeriph Drivers->RTC`
- 添加`my_RTC.c``my_RTC.h`
- `my_RTC.h`

```c
#ifndef __RTC_H
#define __RTC_H
#include "stm32f10x.h"
typedef struct  {
    int tm_sec;
    int tm_min;
    int tm_hour;
    int tm_mday;
    int tm_mon;
    int tm_year;
} rtc_time;
void getRTC(rtc_time *tm);
#endif

```
- `my_RTC.c`

```c
#include "my_RTC.h"
#include <stdio.h>
static  uint8_t month_days[12] = {	31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
//北京时间的时区秒数差
#define TIME_ZOOM           (8*60*60)
/*  一天有多少s */
#define SECDAY              86400L
#define	STARTOFTIME         1970
#define FEBRUARY            2
#define leapyear(year)		((year % 4 == 0)&&(((year %100)!=0)||((year % 400)==0)))
#define	days_in_year(a) 	(leapyear(a) ? 366 : 365)
#define	days_in_month(a) 	(month_days[(a) - 1])
/**
  * @brief  移植于Linux的时间计算函数
  * @param  tim 时间戳
  * @param  tm 时间
  * @retval 无
  */
void to_tm(uint32_t tim, rtc_time * tm)
{
    register uint32_t    i;
    register long   hms, day;

    day = tim / SECDAY;			/* 有多少天 */
    hms = tim % SECDAY;			/* 今天的时间，单位s */

    /* Hours, minutes, seconds are easy */
    tm->tm_hour = hms / 3600;
    tm->tm_min = (hms % 3600) / 60;
    tm->tm_sec = (hms % 3600) % 60;

    /* Number of years in days */ /*算出当前年份，起始的计数年份为1970年*/
    for (i = STARTOFTIME; day >= days_in_year(i); i++) {
        day -= days_in_year(i);
    }
    tm->tm_year = i;

    /* Number of months in days left */ /*计算当前的月份*/
    if (leapyear(tm->tm_year)) {
        days_in_month(FEBRUARY) = 29;
    }
    for (i = 1; day >= days_in_month(i); i++) {
        day -= days_in_month(i);
    }
    days_in_month(FEBRUARY) = 28;
    tm->tm_mon = i;

    /* Days are what is left over (+1) from all that. *//*计算当前日期*/
    tm->tm_mday = day + 1;
}
void getRTC(rtc_time *tm)
{
    uint32_t BJ_TimeVar;
    /*  把标准时间转换为北京时间*/
    BJ_TimeVar =RTC_GetCounter() + TIME_ZOOM;
    /*把定时器的值转换为北京时间*/
    to_tm(BJ_TimeVar, tm);
}

```
# 遍历文件
- 参照FatFs官方例程
- [官方例程](http://elm-chan.org/fsw/ff/doc/readdir.html)
- `my_file.c`

```c
#include "my_file.h"
#include "stdio.h"
#include "string.h"
/**
 * @brief  扫描指定目录下的所有文件
 * @param  path: 资源路径
 * @retval 文件系统的返回值
 */
FRESULT scan_files (char* path)
{
    static FILINFO fno;
    FRESULT res;
    DIR dir;
    UINT i = 0;
    res=f_opendir(&dir,path);
    if(res==FR_OK)
    {
        for (;;) {
            res = f_readdir(&dir, &fno);                   /* Read a directory item */
            if (res != FR_OK || fno.fname[0] == 0) break;  /* Break on error or end of dir */
            if (fno.fattrib & AM_DIR) {                    /* It is a directory */
                i = strlen(path);
                sprintf(&path[i], "/%s", fno.fname);
                res = scan_files(path);                    /* Enter the directory */
                if (res != FR_OK) break;
                path[i] = 0;
            } else {                                       /* It is a file. */
                printf("%s/%s ", path,fno.fname);
                printf("%dBytes ", fno.fsize);
                printf("%c%c%c%c%c ",
                       (fno.fattrib & AM_DIR) ? 'D' : '-',
                       (fno.fattrib & AM_RDO) ? 'R' : '-',
                       (fno.fattrib & AM_HID) ? 'H' : '-',
                       (fno.fattrib & AM_SYS) ? 'S' : '-',
                       (fno.fattrib & AM_ARC) ? 'A' : '-');
                printf("%.4d/%.2d/%.2d %.2d:%.2d:%.2d\r\n",
                       ((fno.fdate&0xFE00)>>9)+1980,
                       (fno.fdate&0x1E0)>>5,
                       fno.fdate&0x1F,
                       (fno.ftime&0xF800)>>11,
                       (fno.ftime&0x7E0)>>5,
                       (fno.ftime&0x1F)<<1);
            }
        }
        f_closedir(&dir);
    }
    return res;
}
```

- `main.c`

```c
#include "my_gpio.h"
#include "my_usart.h"
#include "my_SD.h"
#include "ff.h"
#include "my_file.h"
#define SD_ROOT "1:"
FATFS fs;                      /* FatFs文件系统对象 */
FRESULT res_sd;                /* 文件操作结果 */
FIL fnew;


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
    res_sd = f_open(&fnew, "1:FatFstest.txt",FA_CREATE_ALWAYS | FA_WRITE );
    if ( res_sd == FR_OK )
    {
       /* 不再读写，关闭文件 */
        f_close(&fnew);
    }
    char path[256];
    strcpy(path,SD_ROOT);
    scan_files(path);
    /* 不再使用文件系统，取消挂载文件系统 */
    f_mount(NULL,SD_ROOT,1);
    while(1);
}

```
# 调试
- 编译之后下载到开发板
- 打开串口助手,可以看到SD卡内文件的信息