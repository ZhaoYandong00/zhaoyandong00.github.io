---
title: STM32标准外设库从SD卡烧录FLASh SPL篇27
categories: STM32 SPL
tags: STM32 SPL
description: SPL库从SD卡烧录FLASh 
---
# 工程设置
- 把FatFs-INFO工程复制一份
- 把FLASH工程的W25Q64驱动复制过来

# 创建资源目录
- `my_file.h`

```c
#ifndef __MY_FILE_H
#define __MY_FILE_H
#include "ff.h"
#include "stm32f10x.h"
#include "my_W25Qx.h"
#define SD_ROOT "1:"
#define RESOURCE_DIR          SD_ROOT"/srcdata"
#define BURN_INFO_NAME        "burn_info.txt"
#define BURN_INFO_NAME_FULL   (RESOURCE_DIR "/" BURN_INFO_NAME)
#define RESOURCE_BASE_ADDR    (4096)
/* 存储在FLASH中的资源目录大小 */
#define CATALOG_SIZE           (8*1024)
enum{EXTERN_CALL,INTERN_CALL};

/* 目录信息类型 */
typedef struct 
{
	char 	      name[40];  /* 资源的名字 */
	uint32_t  	size;      /* 资源的大小 */ 
	uint32_t 	  offset;    /* 资源相对于基地址的偏移 */
}CatalogTypeDef;

FRESULT scan_files (char* path);
FRESULT Make_Catalog (char* path,uint8_t clear);
void Burn_Catalog(void);
FRESULT Burn_Content(void);
FRESULT Check_Resource(void);
int GetResOffset(const char *res_name);
#endif

```
- `my_file.c`

```c
#include "my_file.h"

#include "stdio.h"
#include "string.h"
#include "stdlib.h"
static char full_file_name[512];
static char line_temp[512];
static FIL file_temp;

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
/**
  * @brief  Make_Catalog 创建资源目录文件，这是个递归函数
  * @param  path[in]:资源路径，必须输入一个空间足够的数组，
                     函数执行时会对它进行路径赋值操作
  * @param  clear: 0 表示重新生成目录文件，1表示在原文件下追加内容
  * @note   在外部调用本函数时，clear必须设置为0，
  *         函数内部递归调用才设置为1.
  * @retval result:文件系统的返回值
  */
FRESULT Make_Catalog (char* path,uint8_t clear)
{
    FILINFO fno;
    FRESULT res;
    DIR dir;
    UINT i;
    static uint32_t resource_addr = CATALOG_SIZE ;
    if(clear == 0)
    {
        f_unlink(BURN_INFO_NAME_FULL);
    }
    res=f_opendir(&dir,path);
    if(res==FR_OK)
    {
        for (;;) {
            res = f_readdir(&dir, &fno);                   /* Read a directory item */
            if (res != FR_OK || fno.fname[0] == 0) break;  /* Break on error or end of dir */
            if (fno.fattrib & AM_DIR) {                    /* It is a directory */
                i = strlen(path);
                sprintf(&path[i], "/%s", fno.fname);
                res = Make_Catalog(path,INTERN_CALL);                    /* Enter the directory */
                if (res != FR_OK) break;
                path[i] = 0;
            } else {                                       /* It is a file. */
                if(strcasecmp(fno.fname,BURN_INFO_NAME)==0)
                    continue;
                sprintf(full_file_name, "%s/%s",path, fno.fname);
                f_open(&file_temp, BURN_INFO_NAME_FULL, FA_OPEN_ALWAYS |FA_WRITE | FA_READ);
                f_lseek(&file_temp,f_size(&file_temp));
                /* 每个文件记录占5行 */
                f_printf(&file_temp, "%s\n", full_file_name);			//完整文件名
                f_printf(&file_temp, "%s\n", fno.fname);					//文件名
                f_printf(&file_temp, "%d\n", fno.fsize);				 //文件大小
                f_printf(&file_temp, "%d\n\n", resource_addr);	 //文件名要存储到的资源目录(未加上基地址)

                f_close(&file_temp);
                resource_addr+=fno.fsize;
            }
        }
        f_closedir(&dir);
    }
    return res;
}

/**
  * @brief  读取目录文件的信息到
  * @param  catalog_name[in] 存储了目录信息的文件名
  * @param  file_index 第file_index条记录
  * @param  dir[out] 要烧录到FLASH的目录信息
  * @param  full_file_name[out] 第file_index条记录中的文件的全路径
  * @retval 1表示到达了文件尾，0为正常
  */
uint8_t Read_CatalogInfo(uint32_t file_index,CatalogTypeDef *dir,char *full_name)
{
    uint32_t i;

    f_open(&file_temp, BURN_INFO_NAME_FULL, FA_OPEN_EXISTING | FA_READ);

    /* 跳过前N行,每个文件记录5行 */
    for(i=0; i<file_index*5; i++)
    {
        f_gets(line_temp, sizeof(line_temp), &file_temp);
    }
    /* 到达文件尾*/
    if(f_eof(&file_temp) != 0 )
    {
        f_close(&file_temp);
        return 1;
    }

    /* 文件全路径 */
    f_gets(line_temp, sizeof(line_temp), &file_temp);
    memcpy(full_name, line_temp, strlen(line_temp)+1);
    /* 替换掉回车 */
    full_name[strlen(full_name)-1] = '\0';

    /* 文件名 */
    f_gets(line_temp, sizeof(line_temp), &file_temp);
    memcpy(dir->name, line_temp, strlen(line_temp)> (sizeof(CatalogTypeDef)-8) ? (sizeof(CatalogTypeDef)-8):strlen(line_temp)+1 );
    dir->name[strlen(dir->name)-1] = '\0';

    /* 文件大小 */
    f_gets(line_temp, sizeof(line_temp), &file_temp);
    dir->size = atoi(line_temp);

    /* 文件要烧录的位置 */
    f_gets(line_temp, sizeof(line_temp), &file_temp);
    dir->offset = atoi(line_temp);

    f_close(&file_temp);

    return 0;
}
/**
  * @brief  烧录目录到FLASH中
  * @param  无
  * @retval 无
  */
void Burn_Catalog(void)
{
    CatalogTypeDef dir;
    uint8_t i;

    /* 遍历目录文件 */
    for(i=0; 1; i++)
    {
        if( Read_CatalogInfo(i,&dir,full_file_name)!=0)
            break;
        /* 把dir信息烧录到FLASH中 */
        W25Q_BufferWrite((uint8_t*)&dir,RESOURCE_BASE_ADDR + sizeof(CatalogTypeDef)*i,sizeof(CatalogTypeDef));
    }
}
/**
  * @brief  根据目录文件信息烧录内容
  * @param  catalog_name 记录了烧录信息的目录文件名
  * @note  使用本函数前需要确认flash已为擦除状态
  * @retval 文件系统操作返回值
  */
FRESULT Burn_Content(void)
{
    CatalogTypeDef dir;
    uint8_t i;
    FRESULT result;
    UINT  bw;            					    /* File R/W count */
    uint32_t write_addr=0;
    uint8_t tempbuf[SPI_FLASH_PageSize];
    /* 遍历目录文件 */
    for(i=0; 1; i++)
    {
        if( Read_CatalogInfo(i,&dir,full_file_name)!=0)
            break;
        result = f_open(&file_temp,full_file_name,FA_OPEN_EXISTING | FA_READ);
        if(result!=FR_OK)
        {
            return result;
        }
        write_addr = dir.offset + RESOURCE_BASE_ADDR;
        while(result == FR_OK)
        {
            result = f_read( &file_temp, tempbuf, SPI_FLASH_PageSize, &bw);//读取数据
            if(result==FR_OK)
            {
                if(bw == 0)break;//为0时不进行读写，跳出
                W25Q_BufferWrite(tempbuf,write_addr,bw);  //拷贝数据到外部flash上
                write_addr+=bw;
                if(bw !=256)break;
            } else
            {
                f_close(&file_temp);
                return result;
            }
        }
        f_close(&file_temp);
    }
    return FR_OK;
}
/**
  * @brief  从FLASH中的目录查找相应的资源位置
  * @param  res_base 目录在FLASH中的基地址
  * @param  res_name[in] 要查找的资源名字
  * @retval -1表示找不到，其余值表示资源在FLASH中的基地址
  */
int GetResOffset(const char *res_name)
{

    int i,len;
    CatalogTypeDef dir;

    len =strlen(res_name);
    for(i=0; i<CATALOG_SIZE; i+=sizeof(CatalogTypeDef))
    {
        W25Q_BufferRead((uint8_t*)&dir,RESOURCE_BASE_ADDR+i,sizeof(CatalogTypeDef));

        if(strncasecmp(dir.name,res_name,len)==0)
        {
            return dir.offset;
        }
    }

    return -1;
}
/**
  * @brief  校验写入的内容
  * @param  catalog_name 记录了烧录信息的目录文件名
  * @retval 文件系统操作返回值
*/
FRESULT Check_Resource(void)
{
    CatalogTypeDef dir;
    uint8_t i;
    int offset;

    FRESULT result;
    UINT  bw;
    uint32_t read_addr=0,j=0;
    uint8_t tempbuf[256],flash_buf[256];
    /* 遍历目录文件 */
    for(i=0; 1; i++)
    {
        if(Read_CatalogInfo(i,&dir,full_file_name)!=0)
            break;

        /* 在FLASH的目录里查找对应的offset */
        offset = GetResOffset(dir.name);
        if(offset == -1)
        {
            return FR_NO_FILE;
        }
        dir.offset = offset + RESOURCE_BASE_ADDR;


        result = f_open(&file_temp,full_file_name,FA_OPEN_EXISTING | FA_READ);
        if(result != FR_OK)
        {

            return result;
        }

        //校验数据
        read_addr = dir.offset;

        f_lseek(&file_temp,0);
        while(result == FR_OK)
        {
            result = f_read( &file_temp, tempbuf, 256, &bw);//读取数据
            if(result!=FR_OK)			 //执行错误
            {

                return result;
            }

            if(bw == 0)break;//为0时不进行读写，跳出

            W25Q_BufferRead(flash_buf,read_addr,bw);  //从FLASH中读取数据

            read_addr+=bw;

            for(j=0; j<bw; j++)
            {
                if(tempbuf[j] != flash_buf[j])
                {
                    f_close(&file_temp);
                    return FR_INT_ERR;
                }
            }

            if(bw !=256)break;//到了文件尾
        }

        f_close(&file_temp);
    }

    return FR_OK;
}

```
# 烧写Flash
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
    printf("\r\n 按一次KEY1开始烧写字库并复制文件到FLASH\r\n");
    while(Key_Scan(KEY1_GPIO_PORT, KEY1_GPIO_PIN) == KEY_OFF);
    printf("\r\n 正在进行整片擦除请耐心等候\r\n");
    W25Q_BulkErase();
    /* 生成烧录目录信息文件 */
    res_sd=Make_Catalog(RESOURCE_DIR,EXTERN_CALL);
    if(res_sd!=FR_OK)
    {
        LED1(ON);
        printf("fail res_sd =%d\r\n",res_sd);
        printf("创建目录失败\r\n");
        while(1);
    }
    printf("\r\n 创建目录完成\r\n");
    /* 烧录 目录信息至FLASH*/
    Burn_Catalog();
    printf("\r\n 烧录目录信息完成\r\n");
    /* 根据 目录 烧录内容至FLASH*/
    Burn_Content();
    if(res_sd!=FR_OK)
    {
        LED1(ON);
        printf("fail res_sd =%d\r\n",res_sd);
        printf("烧录失败\r\n");
        while(1);
    }
    printf("\r\n 烧录完成\r\n");
    /* 校验烧录的内容 */
    Check_Resource();
    if(res_sd!=FR_OK)
    {
        LED1(ON);
        printf("fail res_sd =%d\r\n",res_sd);
        printf("校验烧录失败\r\n");
        while(1);
    }
    printf("\r\n 校验完成\r\n");
    /* 不再使用文件系统，取消挂载文件系统 */
    f_mount(NULL,SD_ROOT,1);
    while(1);
}

```
# 调试
- 编译之后下载到开发板
- 打开串口助手
- 按下KEY1按键，开始FLASH写入