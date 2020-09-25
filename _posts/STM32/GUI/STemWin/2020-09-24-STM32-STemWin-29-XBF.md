---
title: XBF 格式字体显示STemWin篇29
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin XBF 格式字体显示
---
# emWin支持的字体类型
- 等宽位图字体:等宽位图字体的每个字符都具有相同的大小。在比例字体中，每个字符都有自己的宽度，而在等宽字体中，宽度只定义一次。像素信息用1bpp 即1 位色彩深度保存，覆盖整个字符区域
- 比例位图字体:比例位图字体的每个字符的高度相同，宽度不同。像素信息用1bpp 即1 位色彩深度保存，覆盖整个字符区域
- 2bpp 抗锯齿字体:每个2bpp 抗锯齿字体的每个字符的高度相同，宽度不同。 像素信息用2bpp 抗锯齿信息保存，并覆盖整个字符区域
- 4bpp 抗锯齿字体:每个4bpp 抗锯齿字体的每个字符的高度相同，宽度不同。像素信息用4bpp 抗锯齿信息保存，并覆盖整个字符区域
- 扩展比例位图字体:扩展比例位图字体的每个字符都有自己的高度和宽度。像素信息用1bpp 保存，只覆盖字形位图的区域
- 带2bpp抗锯齿的扩展比例位图字体:每个2bpp 抗锯齿扩展比例字体字符都有自己的高度和宽度。像素信息用2bpp 抗锯齿信息保存，只覆盖字形位图的区域
- 带4bpp抗锯齿的扩展比例位图字体:每个4bpp 抗锯齿扩展比例字体字符都有自己的高度和宽度。像素信息用4bpp 抗锯齿信息保存，只覆盖字形位图的区域
- 带边框的扩展比例位图字体:无论当前设置如何，边框字体总是以透明模式绘制。字符像素是用当前选择的前景色绘制的，框架是用背景色绘制的。前景和背景颜色的良好对比确保文本可以在任何背景下阅读

# emWin支持的字体格式
- C文件格式:在使用C 文件形式的字体时，我们建议编译所有可用的字体并将它们链接为库模块，或者将所有字体对象文件放在一个库中，以便与应用程序链接
- SIF格式:SIF 格式又叫系统独立字体格式，是包含字体信息的二进制数据块
- XBF格式:外部位图字体（XBF）格式，和SIF 字体一样，XBF 字体是包含字体信息的二进制数据块
- TTF格式:TrueType 格式是苹果公司开发的一种轮廓字体标准。它为字体开发人员提供了高度的控制，可以控制字体在各种字体高度上的显示方式。与基于每个字符的位图的位图字体相反，TrueType 字体基于矢量图形。 矢量表示的优点是无损可伸缩性。


# 字体转换器`FontCvtST`
- 字体转换器是SEGGER 公司为emWin 开发的一种主要用于将安装在Windows 系统中的字体转换为emWin 支持的字体的软件，仅支持Windows 系统环境
- FontCvtST在`STemWin\Software`内

# 生成XBF 字库
- 选择需要生成的字体类型。双击FontCvtST.exe 打开字体转换器，在弹出来的对话框中选择Extended, antialised, 2bpp，也就是带2bpp 抗锯齿的扩展比例位图字体，然后编码格式选择16Bit UNICODE
- 点击OK 后，选择字体，字形，大小，尺寸单位选择Pixels
- 保存字库文件。选择找字体类型和大小之后，点击字体转换器左上角的File，然后选择Save As
- 然后选择保存路径，设置文件名，保存文件格式为.xbf
- 等待字体转换器转换完成。完成后会在窗口左下角显示“Ready”

|FontCvtST选项|字体类型|描述|
|:----:|:----:|:----:|
|Standard|GUI_XBF_TYPE_PROP|比例位图字体|
|Extended|GUI_XBF_TYPE_PROP_EXT|扩展比例位图字体|
|Extended,fremed|GUI_XBF_TYPE_PROP_FRM|带边框的扩展比例位图字体|
|Extended,antialiased,2bpp|GUI_XBF_TYPE_PROP_AA2_EXT|带2bpp抗锯齿的扩展比例位图字体|
|Extended,antialiased,4bpp|GUI_XBF_TYPE_PROP_AA4_EXT|带4bpp抗锯齿的扩展比例位图字体|

# XBF 字体显示相关API
- 创建和选择XBF 字体`int GUI_XBF_CreateFont(GUI_FONT * pFont, GUI_XBF_DATA * pXBF, const GUI_XBF_TYPE * pFontType, GUI_XBF_GET_DATA_FUNC * pfGetData, void * pVoid)`
- 删除已创建的字体`void GUI_XBF_DeleteFont(GUI_FONT * pFont)`

# XBF 格式字体显示实验
## 前期准备
- 把准备好的XBF字体放到SD卡里
- 由于keil5 的文本编辑器目前暂不支持Unicode 编码的中文字符，仅支持UTF-8 编码的中文字符。创建并转换待显示文本,使用外部编辑器，文件以**UTF-8-BOM**编码
- `text.c`

```c
const char Framewin_text[] = {"中文显示"};
const char text[] = {"STemWIn学习\r\n本例程使用XBF格式字库\r\n支持中文显示"};
const char MULTIEDIT_text[] = {"硬件平台：\r\n野火STM32F103开发板\r\n软件平台：\r\nFREERTOS\r\n"};

```
- 由于FatFs的CODE_PAGE使用简体中文（936）占用空间较多，故本例程修改为英语（437）

```c
#define FF_CODE_PAGE	437
```

## 读取XBF字库
- `GUIFont_Create.h`

```c
#ifndef __GUIFONT_CREATE_H
#define	__GUIFONT_CREATE_H

void Create_XBF_Font(void);

#endif /* __GUIFONT_CREATE_H */

```
- `GUIFont_Create.c`

```c
#include "GUIFont_Create.h"
#include "GUI.h"
#include "stm32f10x.h"
#include "ff.h"
#include "stdio.h"

GUI_FONT     	FONT_WEIRUANYAHEI_17;
/* 字库结构体 */
GUI_XBF_DATA 	FONT_WEIRUANYAHEI_17_Data;

GUI_XBF_DATA 	XBF_XINSONGTI_18_Data;
GUI_FONT     	FONT_XINSONGTI_18;

static const char FONT_WEIRUANYAHEI_17_ADDR[] = "1:/Font/yahei17.xbf";
static const char FONT_XINSONGTI_18_ADDR[] = 	 "1:/Font/新宋体18.xbf";
static FIL fnew;									/* file objects */
static FRESULT res;
static UINT br;            				/* File R/W count */

/**
  * @brief  获取字体数据的回调函数
  * @param  Offset：要读取的内容在XBF文件中的偏移位置
  * @param  NumBytes：要读取的字节数
	* @param  pVoid：自定义数据的指针
  * @param  pBuffer：存储读取内容的指针
  * @retval 0 成功, 1 失败
  */
static int _cb_FONT_XBF_GetData(U32 Offset, U16 NumBytes, void * pVoid, void * pBuffer)
{
    /* 从pVoid中获取字库的存储地址(pvoid的值在GUI_XBF_CreateFont中传入) */
    res = f_open(&fnew , (char *)pVoid, FA_OPEN_EXISTING | FA_READ);

    if (res == FR_OK)
    {
        f_lseek (&fnew, Offset);/* 指针偏移 */
        /* 读取内容 */
        res = f_read( &fnew, pBuffer, NumBytes, &br );
        f_close(&fnew);
        return 0;
    }
    else
    {
        printf("%s",(char *)pVoid);
        return 1;
    }

}
/**
  * @brief  创建XBF字体
  * @param  无
  * @retval 无
  */
void Create_XBF_Font(void)
{
    GUI_XBF_CreateFont(&FONT_WEIRUANYAHEI_17,              /* GUI_FONT 字体结构体指针 */
                       &FONT_WEIRUANYAHEI_17_Data,          /* GUI_XBF_DATA 结构体指针 */
                       GUI_XBF_TYPE_PROP,           /* 字体类型 */
                       _cb_FONT_XBF_GetData,            /* 获取字体数据的回调函数 */
                       (void *)&FONT_WEIRUANYAHEI_17_ADDR);/* 要传输给回调函数的自定义数据指针，此处传输字库的地址 */
    /* 新宋体18 */
    GUI_XBF_CreateFont(&FONT_XINSONGTI_18,              /* GUI_FONT 字体结构体指针 */
                       &XBF_XINSONGTI_18_Data,          /* GUI_XBF_DATA 结构体指针 */
                       GUI_XBF_TYPE_PROP_EXT,           /* 字体类型 */
                       _cb_FONT_XBF_GetData,            /* 获取字体数据的回调函数 */
                       (void *)&FONT_XINSONGTI_18_ADDR);/* 要传输给回调函数的自定义数据指针，此处传输字库的地址 */
}

```
## 界面程序
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <stdio.h>

#include "GUIFont_Create.h"
/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
#define ID_FRAMEWIN_0   (GUI_ID_USER + 0x00)
#define ID_TEXT_0   (GUI_ID_USER + 0x01)
#define ID_TEXT_1   (GUI_ID_USER + 0x02)
#define ID_MULTIEDIT_0   (GUI_ID_USER + 0x03)
/*********************************************************************
*
*     Types
*
**********************************************************************
*/

/*******************************************************************
*
*     Extern variables
*
********************************************************************/
extern const char Framewin_text[];
extern const char text[];
extern const char MULTIEDIT_text[];
/*支持的字库类型*/
extern GUI_FONT     FONT_WEIRUANYAHEI_17;
extern GUI_FONT     FONT_XINSONGTI_18;
/*******************************************************************
*
*     Static variables
*
********************************************************************/

static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { TEXT_CreateIndirect, "Text", ID_TEXT_0, 0, 0, 230, 55, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text", ID_TEXT_1, 0, 55, 230, 55, 0, 0x64, 0 },
    { MULTIEDIT_CreateIndirect, "Multiedit", ID_MULTIEDIT_0, 5, 120, 230, 130, 0, 0x0, 0 },
};


/*******************************************************************************
 *      static code
 ******************************************************************************/

/**
  * @brief 对话框回调函数
  * @note 无
  * @param pMsg：消息指针
  * @retval 无
  */
static void _cbDialog(WM_MESSAGE * pMsg)
{
    WM_HWIN hItem;
    int     NCode;
    int     Id;

    switch (pMsg->MsgId)
    {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 34);
        FRAMEWIN_SetText(hItem, Framewin_text);
        FRAMEWIN_SetFont(hItem, &FONT_WEIRUANYAHEI_17);
        /* 初始化TEXT0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetText(hItem, text);
        TEXT_SetFont(hItem, &FONT_XINSONGTI_18);
        /* 初始化TEXT1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_1);
        TEXT_SetText(hItem, text);
        TEXT_SetFont(hItem, &FONT_WEIRUANYAHEI_17);
        /* 初始化MULTIEDIT0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_MULTIEDIT_0);
        MULTIEDIT_SetReadOnly(hItem, 1);
        MULTIEDIT_SetBufferSize(hItem, 200);
        MULTIEDIT_SetWrapWord(hItem);
        MULTIEDIT_SetText(hItem, MULTIEDIT_text);
        MULTIEDIT_SetFont(hItem, &FONT_WEIRUANYAHEI_17);
        MULTIEDIT_SetTextColor(hItem, MULTIEDIT_CI_READONLY, GUI_GREEN);
        MULTIEDIT_SetBkColor(hItem, MULTIEDIT_CI_READONLY, GUI_BLACK);
        MULTIEDIT_ShowCursor(hItem, 0);
        break;
    case WM_NOTIFY_PARENT:
        Id    = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch(Id)
        {
        case ID_MULTIEDIT_0: // Notifications sent by 'Multiedit'
            switch(NCode)
            {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        }
        break;
    default:
        WM_DefaultProc(pMsg);
        break;
    }
}
/**
  * @brief 以对话框方式间接创建控件
  * @note 无
  * @param 无
  * @retval hWin：资源表中第一个控件的句柄
  */
WM_HWIN CreateFramewin(void)
{
    WM_HWIN hWin;

    hWin = GUI_CreateDialogBox(_aDialogCreate, GUI_COUNTOF(_aDialogCreate), _cbDialog, WM_HBKWIN, 0, 0);
    return hWin;
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
    /* 启用UTF-8编码 */
    GUI_UC_SetEncodeUTF8();
    /* 创建字体 */
    Create_XBF_Font();
    /* 创建窗口 */
    CreateFramewin();
    while(1)
    {
        GUI_Delay(100);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示不同字体的中文