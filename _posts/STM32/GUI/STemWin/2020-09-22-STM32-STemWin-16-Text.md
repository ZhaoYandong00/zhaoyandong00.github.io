---
title: 文本控件STemWin篇16
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 文本控件
---
# 文本控件API
- 创建文本控件`TEXT_Handle TEXT_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const char * pText)`
- 从对话框资源表中创建文本控件`TEXT_Handle TEXT_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建文本控件`TEXT_Handle TEXT_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const char * pText, int NumExtraBytes)`
- 返回当前背景色`GUI_COLOR TEXT_GetBkColor(TEXT_Handle hObj)`
- 返回默认字体`const GUI_FONT * TEXT_GetDefaultFont(void)`
- 返回默认文本颜色`GUI_COLOR TEXT_GetDefaultTextColor(void)`
- 返回默认文本换行模式`GUI_WRAPMODE TEXT_GetDefaultWrapMode(void)`
- 返回当前字体`const GUI_FONT * TEXT_GetFont(TEXT_Handle hObj)`
- 返回文本控件中当前显示的行数`int TEXT_GetNumLines(TEXT_Handle hObj)`
- 返回当前文本内容`int TEXT_GetText(TEXT_Handle hObj, char * pDest, U32 BufferSize)`
- 返回当前文本对齐方式`int TEXT_GetTextAlign(TEXT_Handle hObj)`
- 返回当前文本颜色`GUI_COLOR TEXT_GetTextColor(TEXT_Handle hObj)`
- 检索用户数据集`int TEXT_GetUserData(TEXT_Handle hObj, void * pDest, int NumBytes)`
- 返回当前文本换行模式`GUI_WRAPMODE TEXT_GetWrapMode(TEXT_Handle hObj)`
- 返回当前文本背景颜色`void TEXT_SetBkColor(TEXT_Handle hObj, GUI_COLOR Color)`
- 设置要显示的数值`int TEXT_SetDec(TEXT_Handle hObj, I32 v, U8 Len, U8 Shift, U8 Signed, U8 Space)`
- 设置默认字体`void TEXT_SetDefaultFont(const GUI_FONT * pFont)`
- 设置默认文本颜色`void TEXT_SetDefaultTextColor(GUI_COLOR Color)`
- 设置默认文本换行模式`GUI_WRAPMODE TEXT_SetDefaultWrapMode(GUI_WRAPMODE WrapMode)`
- 设置文本字体`void TEXT_SetFont(TEXT_Handle hObj, const GUI_FONT * pFont)`
- 设置想要显示的文本`int TEXT_SetText(TEXT_Handle hObj, const char * s)`
- 设置文本对齐方式`void TEXT_SetTextAlign(TEXT_Handle hObj, int Align)`
- 设置当前文本颜色`void TEXT_SetTextColor(TEXT_Handle hObj, GUI_COLOR Color)`
- 设置额外用户数据集`int TEXT_SetUserData(TEXT_Handle hObj, const void * pSrc, int NumBytes)`
- 设置文本换行模式`void TEXT_SetWrapMode(TEXT_Handle hObj, GUI_WRAPMODE WrapMode)`

# 文本控件实验
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <string.h>

/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
#define ID_FRAMEWIN_0   (GUI_ID_USER + 0x00)
#define ID_TEXT_0   (GUI_ID_USER + 0x01)
#define ID_TEXT_1   (GUI_ID_USER + 0x02)
#define ID_TEXT_2   (GUI_ID_USER + 0x03)
#define ID_TEXT_3   (GUI_ID_USER + 0x04)
#define ID_TEXT_4   (GUI_ID_USER + 0x05)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { TEXT_CreateIndirect, "Text0", ID_TEXT_0, 30, 20, 200, 40, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text1", ID_TEXT_1, 30, 80, 100, 20, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text2", ID_TEXT_2, 30, 110, 100, 30, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text3", ID_TEXT_3, 30, 150, 100, 40, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text4", ID_TEXT_4, 10, 185, 220, 130, 0, 0x64, 0 },
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
static void _cbDialog(WM_MESSAGE* pMsg) {
    WM_HWIN hItem;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Text0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetFont(hItem, GUI_FONT_COMIC18B_ASCII);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "STemWIN \r\nSTM32F103");
        /* 初始化Text1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_1);
        TEXT_SetFont(hItem, GUI_FONT_8X16X1X2);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Text");
        /* 初始化Text2 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_2);
        TEXT_SetFont(hItem, GUI_FONT_8X16X2X2);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Text");
        /* 初始化Text3 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_3);
        TEXT_SetFont(hItem, GUI_FONT_8X16X3X3);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Text");
        /* 初始化Text4 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_4);
        TEXT_SetFont(hItem, GUI_FONT_D48X64);
        TEXT_SetTextAlign(hItem, GUI_TA_HCENTER | GUI_TA_VCENTER);
        TEXT_SetDec(hItem, 0, 4, 0, 0, 0);
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
WM_HWIN CreateFramewin(void) {
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
    /* 创建对话框 */
    CreateFramewin();
    while (1)
    {
        GUI_Delay(200);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示