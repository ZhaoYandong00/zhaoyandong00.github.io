---
title: BMP图片显示STemWin篇26
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin BMP图片显示
---
# BMP 图片格式
## BMP文件头

```c
typedef struct tagBITMAPFILEHEADER
{
//attention: sizeof(DWORD)=4 sizeof(WORD)=2
    DWORD bfSize; //文件大小
    WORD bfReserved1; //保留字，不考虑
    WORD bfReserved2; //保留字，同上
    DWORD bfOffBits; //实际位图数据的偏移字节数，即前三个部分长度之和
} BITMAPFILEHEADER,tagBITMAPFILEHEADER;
```

|变量名|地址|作用|值|参数说明|
|:----:|:----:|:----:|:----:|:----:|
|bfType|00~01h|文件类型|42 4D|如果是位图文件类型,必须分别为0x42 和0x4D ，0x424D=’BM’|
|bfSize|02~05h|文件大小|36 30 0F 00|000F3036h = 995382B 约等于 972k|
|bfReserved1|06~07h|保留字|00 00|不考虑|
|bfReserved2|08~09h|保留字|00 00|同上|
|bfOffBits|0A~0Dh|实际位图数据的偏移字节数,即BMP文件头、位图信息头与调色板之和|36 00 00 00|00000036h = 54,刚刚好等于我们文件头部信息（BMP 文件头和位图信息头）,因为这张图片是24 位位图,所以调色板的大小为0|


## 位图信息头

```c
typedef struct tagBITMAPINFOHEADER
{
//attention: sizeof(DWORD)=4 sizeof(WORD)=2
    DWORD biSize; //指定此结构体的长度，为40
    LONG biWidth; //位图宽，说明本图的宽度，以像素为单位
    LONG biHeight; //位图高，指明本图的高度，像素为单位
    WORD biPlanes; //平面数，为1
    WORD biBitCount; //采用颜色位数，可以是1，2，4，8，16，24 新的可以是32
    DWORD biCompression; //压缩方式，可以是0，1，2，其中0 表示不压缩
    DWORD biSizeImage; //实际位图数据占用的字节数
    LONG biXPelsPerMeter; //X 方向分辨率
    LONG biYPelsPerMeter; //Y 方向分辨率
    DWORD biClrUsed; //使用的颜色数，如果为0，则表示默认值(2^颜色位数)
    DWORD biClrImportant; //重要颜色数，如果为0，则表示所有颜色都是重要的
} BITMAPINFOHEADER,tagBITMAPINFOHEADER;
```

|变量名|地址|作用|值|参数说明|
|:----:|:----:|:----:|:----:|:----:|
|biSize|0E~11h|指定结构体BITMAPINFOHEADER的长度，为40|28 00 00 00|00000028h = 40,就是说这个位图信息头的大小为40 个字节|
|biWidth|12~15h|位图宽，以像素为单位|00 03 00 00|0000300h = 768像素|
|biHeight|16~19h|位图高，以像素为单位|B0 01 00 00|000001B0h = 432像素|
|biPlanes|1A~1Bh|平面数，该值总为1|00 01|平面数：1|
|biBitCount|1C~1Dh|图像色彩深度，可以是1，2，4，8，16，24或 32|18 00|0018h=24 位色彩深度|
|biCompression|1E~21h|压缩方式，可以是0，1，2，其中0 表示不压缩|00 00 00|不压缩|
|biSizeImage|22~25h|实际位图数据占用的字节数|00 00 00 00|图像不压缩时，可以设置为0|
|biXPelsPerMeter|26~29h|X方向分辨率|C4 0E 00 00|00000EC4h=3780像素/米|
|biYPelsPerMeter|2A~2Dh|Y方向分辨率|C4 0E 00 00|00000EC4h=3780像素/米|
|biClrUsed|2E~31h|使用的颜色数，如果为0，则表示默认值(2^颜色位数)|00 00 00 00|默认值|
|biClrImportant|32~35h|重要颜色数，如果为0，则表示所有颜色都是重要的|00 00 00 00|所有颜色都是重要的|

## 调色板
- 图像是单色、16色和256色，则紧跟着调色板的是位图数据，位图数据是指向调色板的索引序号
- 位图是16位、24位和32位色，则图像文件中不保留调色板，即不存在调色板，图像的颜色直接在位图数据中给出
- 调色板的大小主要决定与0x2e-0x31的颜色索引数N
- 调色板大小 = N*4 (因为每种颜色包含blue，green，red，alpha 4个字节的分量)

```c
typedef struct tagRGBQUAD {
BYTE    rgbBlue; //蓝色值
BYTE    rgbGreen;//绿色值
BYTE    rgbRed; //红色值
BYTE    rgbReserved;//保留，总为0
} RGBQUAD;
```
## 图像像素数据
- 由于位图信息头中的位图高度是正的，所以位图数据在文件中的排列顺序是从左下角到右上角，以行为主序排列的。也就是第一个数据是图像最后一行第一列像素色彩数据，第二个数据是图像最后一行第二列像素色彩数据
- 如果RGB 24位位图则使用3个bytes存储一个像素，按照BGR顺序存储。如果是32位ARGB数据则按照BGRA的顺序存储
- 如果存在调色板，则这些值代表的是调色板的索引序号

# BMP 显示相关API
- 绘制已加载到内存中的BMP 文件`int GUI_BMP_Draw(const void * pFileData, int x0, int y0)`
- 绘制不加载到内存中的BMP 文件`int GUI_BMP_DrawEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0)`
- 绘制已加载到内存中的带缩放的BMP 文件`int GUI_BMP_DrawScaled(const void * pFileData, int x0, int y0, int Num, int Denom)`
- 绘制不加载到内存中的带缩放的BMP 文件`int GUI_BMP_DrawScaledEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0, int Num, int Denom)`
- 为32bpp V3 BMP 文件启用alpha 通道`void GUI_BMP_EnableAlpha(void)`
- 返回加载到内存中的BMP 文件的X 大小`int GUI_BMP_GetXSize(const void * pFileData)`
- 返回不加载到内存中的BMP 文件的X 大小`int GUI_BMP_GetXSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`
- 返回加载到内存中的BMP 文件的Y 大小`int GUI_BMP_GetYSize(const void * pFileData)`
- 返回不加载到内存中的BMP 文件的Y 大小`int GUI_BMP_GetYSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`
- 创建一张BMP 文件`void GUI_BMP_Serialize(GUI_CALLBACK_VOID_U8_P * pfSerialize, void * p)`
- 从给定的矩形创建一个BMP 文件`void GUI_BMP_SerializeEx(GUI_CALLBACK_VOID_U8_P * pfSerialize, int x0, int y0, int xSize, int ySize, void * p)`
- 使用指定的颜色深度从给定的矩形创建BMP 文件`void GUI_BMP_SerializeExBpp(GUI_CALLBACK_VOID_U8_P * pfSerialize, int x0, int y0, int xSize, int ySize, void * p, int BitsPerPixel)`

# BMP 图片显示实验
## 前期准备
- 把准备好的图片拷贝到内存卡里

## 挂载文件系统
- 添加头文件包含到`main.c`

```c
/* FatFs头文件 */
#include "ff.h"

/**************************** 文件系统 ********************************/
FATFS   fs;			/* FatFs文件系统对象 */
FRESULT result;
```
- 在板级外设初始化里挂载文件系统

```c
/**
 * @brief  板级外设初始化，所有板子上的初始化均可放在这个函数里面
 * @param  无
 * @retval 无
 */
static void BSP_Init(void)
{
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_CRC, ENABLE);
    NVIC_PriorityGroupConfig( NVIC_PriorityGroup_4 );
    /* LED 初始化 */
    LED_GPIO_Config();
    /* 串口初始化	*/
    USART_Config();
    /* 触摸屏初始化 */
    XPT2046_Init();
    /* 挂载文件系统，挂载时会对SD卡初始化 */
    result = f_mount(&fs,"1:",1);
    if(result != FR_OK)
    {
        printf("SD init fail!\n");
        while(1);
    }
}
```

## 图片显示程序
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <stdio.h>
#include "ff.h"
#include "FreeRTOS.h"
#include "task.h"
/*********************************************************************
*
*     Defines
*
**********************************************************************
*/

/*********************************************************************
*
*     Types
*
**********************************************************************
*/


/*******************************************************************
*
*     Static variables
*
********************************************************************/

int t0, t1;
static char _acBuffer[1024 * 4];

extern FRESULT result;
FIL     file;							/* file objects */
//BMP图片存储路径
const char bmpName[]="1:/image/ngc7635.bmp";


/*******************************************************************************
 *      static code
 ******************************************************************************/

/**
  * @brief 从存储器中读取数据
  * @note 无
  * @param
  * @retval NumBytesRead：读到的字节数
  */
int _GetData(void * p, const U8 ** ppData, unsigned NumBytesReq, U32 Off)
{
    static int FileAddress = 0;
    UINT NumBytesRead;
    FIL *Picfile;

    Picfile = (FIL *)p;

    /* 检查缓冲区大小 */
    if(NumBytesReq > sizeof(_acBuffer))
    {
        NumBytesReq = sizeof(_acBuffer);
    }

    /* 读取偏移量 */
    if(Off == 1) FileAddress = 0;
    else FileAddress = Off;
    result = f_lseek(Picfile, FileAddress);

    /* 进入临界段 */
    taskENTER_CRITICAL();
    /* 读取图片数据 */
    result = f_read(Picfile, _acBuffer, NumBytesReq, &NumBytesRead);
    /* 退出临界段 */
    taskEXIT_CRITICAL();

    *ppData = (const U8 *)_acBuffer;

    /* 返回以读到的字节数 */
    return NumBytesRead;
}

/**
  * @brief 直接从外部存储器中绘制BMP图片
  * @note 无
  * @param sFilename：需要加载的图片名
  *        x0：图片左上角在屏幕上的横坐标
  *        y0：图片左上角在屏幕上的纵坐标
  * @retval 无
  */
static void ShowBMPEx(const char *sFilename, int x0, int y0)
{
    /* 进入临界段 */
    taskENTER_CRITICAL();
    /* 打开图片 */
    result = f_open(&file, sFilename, FA_READ);
    if ((result != FR_OK))
    {
        printf("文件打开失败！\r\n");
    }
    /* 退出临界段 */
    taskEXIT_CRITICAL();

    /* 绘制图片 */
    GUI_BMP_DrawEx(_GetData, &file, x0, y0);
}



/*********************************************************************
*
*     Public code
*
**********************************************************************
*/
/**
  * @brief GUI主任务
  * @note 无
  * @param 无
  * @retval 无
  */
void MainTask(void)
{
    GUI_SetFont(GUI_FONT_16B_ASCII);
    GUI_SetTextMode(GUI_TM_XOR);
    char  buffer[50];
    t0 = GUI_GetTime();
    /* 直接从外部存储器绘制BMP图片 */
    ShowBMPEx(bmpName,0,0);
    t1 = GUI_GetTime();
    sprintf(buffer,"GUI_BMP_DrawEx():%dms",t1-t0);
    GUI_DispStringAt(buffer, 0, 0);
    printf("\r\n直接从外部存储器绘制BMP：%dms\r\n",t1 - t0);
    while(1)
    {
        GUI_Delay(2000);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示图片
- 图片上方显示BMP绘制时间
