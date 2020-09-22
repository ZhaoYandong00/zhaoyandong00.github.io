---
title: 单选按钮STemWin篇15
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 单选按钮
---
# 单选按钮控件API
- 创建一个单选按钮控件`RADIO_Handle RADIO_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumItems, int Spacing)`
- 从对话框资源表中创建单选按钮控件`RADIO_Handle RADIO_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外字节作为用户数据创建单选按钮`RADIO_Handle RADIO_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumItems, int Spacing, int NumExtraBytes)`
- 将按钮选择值减少1`void RADIO_Dec(RADIO_Handle hObj)`
- 返回单选按钮背景色`GUI_COLOR RADIO_GetBkColor(RADIO_Handle hObj)`
- 返回单选按钮默认字体`const GUI_FONT * pFont RADIO_GetDefaultFont(void)`
- 返回单选按钮默认文本颜色`GUI_COLOR RADIO_GetDefaultTextColor(void)`
- 返回单选按钮默认聚焦颜色`GUI_COLOR RADIO_GetFocusColor(RADIO_Handle hObj)`
- 返回单选按钮当前字体`const GUI_FONT * RADIO_GetFont(RADIO_Handle hObj)`
- 返回单选按钮当前文本`int RADIO_GetText(RADIO_Handle hObj, unsigned Index, char * pBuffer, int MaxLen)`
- 返回单选按钮当前文本颜色`GUI_COLOR RADIO_GetTextColor(RADIO_Handle hObj)`
- 检索用户数据集`int RADIO_GetUserData(RADIO_Handle hObj, void * pDest, int NumBytes)`
- 返回当前选择的按钮`int RADIO_GetValue(RADIO_Handle hObj)`
- 将按钮选择值增加1`void RADIO_Inc(RADIO_Handle hObj)`
- 设置单选按钮背景颜色`void RADIO_SetBkColor(RADIO_Handle hObj, GUI_COLOR Color)`
- 设置单选按钮默认聚焦颜色`GUI_COLOR RADIO_SetDefaultFocusColor(RADIO_Handle hObj, GUI_COLOR Color)`
- 设置单选按钮默认字体`void RADIO_SetDefaultFont(const GUI_FONT * pFont)`
- 设置要用于新单选按钮的图像`void RADIO_SetDefaultImage(const GUI_BITMAP * pBitmap, unsigned int Index)`
- 设置单选按钮默认文本颜色`void RADIO_SetDefaultTextColor(GUI_COLOR TextColor)`
- 设置单选按钮聚焦颜色`GUI_COLOR RADIO_SetFocusColor(RADIO_Handle hObj, GUI_COLOR Color)`
- 设置单选按钮字体`void RADIO_SetFont(RADIO_Handle hObj, const GUI_FONT * pFont)`
- 设置给定单选按钮的组ID`void RADIO_SetGroupId(RADIO_Handle hObj, U8 GroupId)`
- 设置用于显示单选按钮的图像`void RADIO_SetImage(RADIO_Handle hObj, const GUI_BITMAP * pBitmap, unsigned int Index)`
- 设置单选按钮文本`void RADIO_SetText(RADIO_Handle hObj, const char* pText, unsigned Index)`
- 设置单选按钮文本颜色`void RADIO_SetTextColor(RADIO_Handle hObj, GUI_COLOR Color)`
- 设置单选按钮的额外数据`int RADIO_SetUserData(RADIO_Handle hObj, const void * pSrc, int NumBytes)`
- 设置按钮选项值`void RADIO_SetValue(RADIO_Handle hObj, int v)`

# 单选按钮实验
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <string.h>
#include "my_gpio.h"
/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
#define ID_FRAMEWIN_0   (GUI_ID_USER + 0x00)
#define ID_RADIO_0      (GUI_ID_USER + 0x01)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { RADIO_CreateIndirect, "Radio0", ID_RADIO_0, 60, 5, 100, 320, 0, 0x2807, 0 },
};

static const GUI_COLOR aColor[] = {GUI_WHITE, GUI_RED, GUI_GREEN,
                                   GUI_BLUE, GUI_CYAN, GUI_MAGENTA,
                                   GUI_ORANGE
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
int value = 0;
static void _cbDialog(WM_MESSAGE* pMsg) {
    WM_HWIN hItem;
    int     NCode;
    int     Id;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN@ STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化Radio控件0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_RADIO_0);
        RADIO_SetFont(hItem, GUI_FONT_24_ASCII);
        RADIO_SetText(hItem, "WHITE",   0);
        RADIO_SetText(hItem, "RED",     1);
        RADIO_SetText(hItem, "GREEN",   2);
        RADIO_SetText(hItem, "BLUE",    3);
        RADIO_SetText(hItem, "CYAN",    4);
        RADIO_SetText(hItem, "MAGENTA", 5);
        RADIO_SetText(hItem, "ORANGE",  6);
        break;
    case WM_NOTIFY_PARENT:
        Id = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch (Id) {
        case ID_RADIO_0: // Notifications sent by 'Radio0'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_RADIO_0);
                value = RADIO_GetValue(hItem);
                WM_InvalidateWindow(pMsg->hWin);
                break;
            }
            break;
        }
        break;
    case WM_PAINT:
        GUI_SetBkColor(aColor[value]);
        GUI_Clear();
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
- 点击屏幕上单选按钮可以看到效果