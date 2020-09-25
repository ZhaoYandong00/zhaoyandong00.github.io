---
title: GIF图片显示STemWin篇28
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin GIF图片显示
---
# GIF 图片
- GIF 标准支持交错、透明度、应用程序定义的数据，动画和原始文本的渲染
- GIF 文件使用LZW （Lempel-Zif-Welch）文件压缩方法来压缩图像数据
- emWin 在做GIF 解压的时候需要大约16KB 左右的RAM 空间

# GIF 显示相关API
- 绘制已加载到内存中的GIF 文件的第一张图像`int GUI_GIF_Draw(const void * pGIF, U32 NumBytes, int x0, int y0)`
- 绘制不需要加载到内存中的GIF 文件的第一张图像`int GUI_GIF_DrawEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0)`
- 绘制已加载到内存中的GIF 文件的给定子图像`int GUI_GIF_DrawSub(const void * pGIF, U32 NumBytes, int x0, int y0, int Index)`
- 绘制不需要加载到内存中的GIF 文件的给定子图像`int GUI_GIF_DrawSubEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0, int Index)`
- 使用缩放比例绘制已加载到内存中的GIF 文件的给定子图像`int GUI_GIF_DrawSubScaled(const void * pGIF, U32 NumBytes, int x0, int y0, int Index, int Num, int Denom)`
- 使用缩放比例绘制GIF 文件的给定子图像，无需缩放即可将其加载到内存中`int GUI_GIF_DrawSubScaledEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0, int Index, int Num, int Denom)`
- 返回已加载到内存中的GIF 文件的给定注释`int GUI_GIF_GetComment(const void * pGIF, U32 NumBytes, U8 * pBuffer, int MaxSize, int Index)`
- 返回不需要加载到内存中的GIF 文件的给定注释`int GUI_GIF_GetCommentEx(GUI_GET_DATA_FUNC * pfGetData, void * p, U8 * pBuffer, int MaxSize, int Index)`
- 返回有关已加载到内存中的GIF 文件的给定子图像的信息`int GUI_GIF_GetImageInfo(const void * pGIF, U32 NumBytes, GUI_GIF_IMAGE_INFO * pInfo, int Index)`
- 返回有关GIF 文件的给定子图像的信息，该信息无需加载到内存中`int GUI_GIF_GetImageInfoEx(GUI_GET_DATA_FUNC * pfGetData, void * p, GUI_GIF_IMAGE_INFO * pInfo, int Index)`
- 返回有关已加载到内存中的GIF 文件的信息`int GUI_GIF_GetInfo(const void * pGIF, U32 NumBytes, GUI_GIF_INFO * pInfo)`
- 返回有关不需要加载到内存中的GIF 文件的信息`int GUI_GIF_GetInfoEx(GUI_GET_DATA_FUNC * pfGetData, void * p, GUI_GIF_INFO * pInfo)`
- 返回加载到内存中的位图的X 大小`int GUI_GIF_GetXSize(const void * pGIF)`
- 返回不需要加载到内存中的位图的X 大小`int GUI_GIF_GetXSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`
- 返回加载到内存中的位图的Y 大小`int GUI_GIF_GetYSize(const void * pGIF)`
- 返回不需要加载到内存中的位图的Y 大小`int GUI_GIF_GetYSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`

# GIF 图片显示实验
## 前期准备
- 把准备好的图片拷贝到内存卡里

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
static char _acBuffer[1024 * 2];

extern FRESULT result;
FIL     file;							/* file objects */

static const char gifName[]="1:/image/dolphin.gif";


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
  * @brief 直接从存储器中绘制GIF图片数据
  * @note 无
  * @param sFilename：需要加载的图片名
  * @retval 无
  */
static void ShowGIFEx(const char *sFilename)
{
    GUI_GIF_INFO Gifinfo = {0};
    GUI_GIF_IMAGE_INFO Imageinfo = {0};
    int i = 0;

    /* 进入临界段 */
    taskENTER_CRITICAL();
    /* 打开图片 */
    result = f_open(&file, sFilename, FA_READ);
    if ((result != FR_OK))
    {
        printf("文件打开失败！\r\n");
        _acBuffer[0]='\0';
    }
    /* 退出临界段 */
    taskEXIT_CRITICAL();

    /* 获取GIF文件信息 */
    GUI_GIF_GetInfoEx(_GetData, &file, &Gifinfo);
    /* 循环显示所有的GIF帧 */
    for(i = 0; i < Gifinfo.NumImages; i++)
    {
        /* 获取GIF子图象信息 */
        GUI_GIF_GetImageInfoEx(_GetData, &file, &Imageinfo, i);
        /* 绘制GIF子图象 */
        GUI_GIF_DrawSubEx(_GetData, &file,
                          (LCD_GetXSize() - Gifinfo.xSize) / 2,
                          (LCD_GetYSize() - Gifinfo.ySize) / 2, i);
        /* 帧延时 */
        GUI_Delay(Imageinfo.Delay);
    }
    /* 读取完毕关闭文件 */
    f_close(&file);
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
    /* 设置背景色 */
    GUI_SetBkColor(GUI_WHITE);
    GUI_Clear();
    /* 设置字体 */
    GUI_SetFont(GUI_FONT_16B_ASCII);
    GUI_SetTextMode(GUI_TM_XOR);
    char  buffer[50];
    t0 = GUI_GetTime();
    /* 直接从外部存储器绘制GIF图片 */
    ShowGIFEx(gifName);
    t1 = GUI_GetTime();
    sprintf(buffer,"ShowGIFEx:%dms",t1-t0);
    GUI_DispStringAt(buffer, 0, 0);  
    printf("%dms\r\n",t1-t0);
    while(1)
    {
        GUI_Delay(1000);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示图片
- 绘制完成后，图片上方显示GIF总时间