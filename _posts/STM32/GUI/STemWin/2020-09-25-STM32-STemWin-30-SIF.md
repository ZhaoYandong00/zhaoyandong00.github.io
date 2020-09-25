---
title: SIF 格式字体显示STemWin篇30
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin SIF 格式字体显示
---
# SIF 字库
- XBF 字体所有数据都需要从外部存储器加载，所以在中文字符内容变化或窗口切换时影响系统流畅度
- SIF 格式的字体在emWin 运行时会全部加载在例如RAM 中，使用时直接从RAM 读取
- SIF 格式字体缺点，就是需要消耗大量RAM空间，在RAM较小的系统上无法使用此格式的大规模字库

# 生成SIF 字库
- 全字库的生成步骤与生成XBF 格式全字库的步骤完全一致
- 由于空间原因我们只生成我们需要的部分
- 选择需要生成的字体类型。首先还是跟XBF 格式一样，双击FontCvtST.exe 打开字体转换器，在弹出来的对话框中选择Extended, antialised, 4bpp，也就是带4bpp抗锯齿的扩展比例位图字体，然后编码格式选择16Bit UNICODE
- 点击OK 后，选择字体，字形，大小，尺寸单位选择Pixels
- 失能所有字符。为了让字体转换器只生成我们想要的部分，需要失能所以字符，字体转换器左上角菜单栏左键点击Edit-> Disable all characters
- 制作模式文件。接着新建一个txt 文件，把想要生成的字符输入txt 文件并保存为UTF-16 LE 编码格式，如果没有这个格式则保存为Unicode

```
中文显示
STemWIn学习
本例程使用SIF格式字库\r\n
支持中文显示
硬件平台：
野火STM32F103开发板
软件平台：
FREERTOS
```
- 导入模式文件。将制作好的模式文件导入字体转换器，在字体转换器左上角菜单栏左键单击Edit->Read parttern file
- 保存字库。导入模式文件之后，点击字体转换器左上角的File，然后选择Save As，设置文件名，保存文件格式为.sif
- 最后，等待字体转换器转换完成。完成后会在窗口左下角显示“Ready”

|FontCvtST选项|字体类型|描述|
|:----:|:----:|:----:|
|Standard|GUI_SIF_TYPE_PROP|比例位图字体|
|Extended|GUI_SIF_TYPE_PROP_EXT|扩展比例位图字体|
|Extended,fremed|GUI_SIF_TYPE_PROP_FRM|带边框的扩展比例位图字体|
|Antialiased,2bpp|GUI_SIF_TYPE_PROP_AA2|2bpp 抗锯齿字体|
|Antialiased,4bpp|GUI_SIF_TYPE_PROP_AA4|4bpp 抗锯齿字体|
|Extended,antialiased,2bpp|GUI_SIF_TYPE_PROP_AA2_EXT|带2bpp抗锯齿的扩展比例位图字体|
|Extended,antialiased,4bpp|GUI_SIF_TYPE_PROP_AA4_EXT|带4bpp抗锯齿的扩展比例位图字体|

# SIF 字体相关API
- 通过将指针传递给系统独立字体数据来创建和选择字体`void GUI_SIF_CreateFont(const void * pFontData, GUI_FONT * pFont, const GUI_SIF_TYPE * pFontType)`
- 删除由GUI_SIF_CreateFont()创建的字体`void GUI_SIF_DeleteFont(GUI_FONT * pFont)`

# SIF 格式字体显示实验
## 前期准备
- 把准备好的SIF字体放到SD卡里
- 同样创建并转换待显示文本,使用外部编辑器，文件以**UTF-8-BOM**编码

```c
const char Framewin_text[] = {"中文显示"};
const char text[] = {"STemWIn学习\r\n本例程使用SIF格式字库\r\n支持中文显示"};
const char MULTIEDIT_text[] = {"硬件平台：\r\n野火STM32F103开发板\r\n软件平台：\r\nFREERTOS\r\n"};

```
## 读取SIF字库
- `GUIFont_Create.h`

```c
#ifndef __GUIFONT_CREATE_H
#define	__GUIFONT_CREATE_H

void Create_SIF_Font(void);

#endif /* __GUIFONT_CREATE_H */

```
- `GUIFont_Create.c`

```c
#include "GUIFont_Create.h"
#include "GUI.h"
#include "stm32f10x.h"
#include "ff.h"
#include "stdio.h"

GUI_FONT FONT_WEIRUANYAHEI_16_4BPP;
/* 字库结构体 */
GUI_FONT FONT_XINSONGTI_16_4BPP;

uint8_t *WEIRUANYAHEI_SIFbuffer16;
uint8_t *XINSONGTI_SIFbuffer16;

static const char FONT_WEIRUANYAHEI_16_ADDR[] = "1:/Font/font16.sif";
static const char FONT_XINSONGTI_16_ADDR[] = 	 "1:/Font/16_4bpp.sif";
static FIL fnew;									/* file objects */
static FRESULT res;
static UINT br;            				/* File R/W count */

/**
  * @brief  加载字体数据到SDRAM
  * @param  res_name：要加载的字库文件名
  * @retval Fontbuffer：已加载好的字库数据
  */
void *FONT_SIF_GetData(const char *res_name)
{
    uint8_t *Fontbuffer;
    GUI_HMEM hFontMem;
    /* 从pVoid中获取字库的存储地址(pvoid的值在GUI_XBF_CreateFont中传入) */
    res = f_open(&fnew , res_name, FA_OPEN_EXISTING | FA_READ);

    if (res == FR_OK)
    {
        hFontMem = GUI_ALLOC_AllocZero(fnew.obj.objsize);
        Fontbuffer = GUI_ALLOC_h2p(hFontMem);
        /* 读取内容 */
        res = f_read( &fnew, Fontbuffer, fnew.obj.objsize, &br );
        f_close(&fnew);
        return Fontbuffer;
    }
    else
    {
        printf("%s",res_name);
        while(1);
    }
}
/**
 * @brief 创建SIF 字体
 * @param 无
 * @retval 无
 */
void Create_SIF_Font(void)
{
    /* 获取字体数据 */
    XINSONGTI_SIFbuffer16 = FONT_SIF_GetData(FONT_XINSONGTI_16_ADDR);
    WEIRUANYAHEI_SIFbuffer16=FONT_SIF_GetData(FONT_WEIRUANYAHEI_16_ADDR);

    GUI_SIF_CreateFont( XINSONGTI_SIFbuffer16,/* 已加载到内存中的字体数据 */
                        &FONT_XINSONGTI_16_4BPP,/* GUI_FONT 字体结构体指针 */
                        GUI_SIF_TYPE_PROP_AA4_EXT);/* 字体类型 */

    GUI_SIF_CreateFont(WEIRUANYAHEI_SIFbuffer16,/* 已加载到内存中的字体数据 */
                       &FONT_WEIRUANYAHEI_16_4BPP,/* GUI_FONT 字体结构体指针 */
                       GUI_SIF_TYPE_PROP_AA4_EXT);/* 字体类型 */
}

```
## 界面程序
- 修改`MainTask.c`,简单修改下XBF的即可

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
extern GUI_FONT     FONT_WEIRUANYAHEI_16_4BPP;
extern GUI_FONT     FONT_XINSONGTI_16_4BPP;
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
        FRAMEWIN_SetFont(hItem, &FONT_WEIRUANYAHEI_16_4BPP);
        /* 初始化TEXT0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetText(hItem, text);
        TEXT_SetFont(hItem, &FONT_XINSONGTI_16_4BPP);
        /* 初始化TEXT1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_1);
        TEXT_SetText(hItem, text);
        TEXT_SetFont(hItem, &FONT_WEIRUANYAHEI_16_4BPP);
        /* 初始化MULTIEDIT0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_MULTIEDIT_0);
        MULTIEDIT_SetReadOnly(hItem, 1);
        MULTIEDIT_SetBufferSize(hItem, 200);
        MULTIEDIT_SetWrapWord(hItem);
        MULTIEDIT_SetText(hItem, MULTIEDIT_text);
        MULTIEDIT_SetFont(hItem, &FONT_XINSONGTI_16_4BPP);
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
    Create_SIF_Font();
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
- 显示速度比XBF要快些
