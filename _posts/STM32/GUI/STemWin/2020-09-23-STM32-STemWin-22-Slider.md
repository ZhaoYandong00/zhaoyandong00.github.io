---
title: 滑块控件STemWin篇22
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 滑块控件
---
# 滑块控件API
- 创建一个滑块控件`SLIDER_Handle SLIDER_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id)`
- 从对话框资源表中创建滑块控件`SLIDER_Handle SLIDER_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建滑块`SLIDER_Handle SLIDER_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumExtraBytes)`
- 减少滑块的值`void SLIDER_Dec(SLIDER_Handle hObj)`
- 返回当前背景颜色`GUI_COLOR SLIDER_GetBkColor(SLIDER_Handle hObj)`
- 返回当前聚焦颜色`GUI_COLOR SLIDER_GetFocusColor(SLIDER_Handle hObj)`
- 返回指定滑块的范围`void SLIDER_GetRange(SLIDER_Handle hObj, int * pMin, int * pMax)`
- 检索用户数据集`int SLIDER_GetUserData(SLIDER_Handle hObj, void * pDest, int NumBytes)`
- 返回滑块当前的值`int SLIDER_GetValue(SLIDER_Handle hObj)`
- 增加滑块的值`void SLIDER_Inc(SLIDER_Handle hObj)`
- 设置当前背景颜色`void SLIDER_SetBkColor(SLIDER_Handle hObj, GUI_COLOR Color)`
- 设置默认聚焦颜色`GUI_COLOR SLIDER_SetDefaultFocusColor(GUI_COLOR Color)`
- 设置聚焦颜色`GUI_COLOR SLIDER_SetFocusColor(SLIDER_Handle hObj, GUI_COLOR Color)`
- 设置滑块的刻度线数量`void SLIDER_SetNumTicks(SLIDER_Handle hObj, int NumTicks)`
- 设置滑块的范围`void SLIDER_SetRange(SLIDER_Handle hObj, int Min, int Max)`
- 设置额外用户数据集`int SLIDER_SetUserData(SLIDER_Handle hObj, const void * pSrc, int NumBytes)`
- 设置滑块当前的值`void SLIDER_SetValue(SLIDER_Handle hObj, int v)`
- 设置滑块的宽度`void SLIDER_SetWidth(SLIDER_Handle hObj, int Width)`

# 滑块控件实验
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <stdio.h>

/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
#define ID_FRAMEWIN_0   (GUI_ID_USER + 0x00)
#define ID_SLIDER_0     (GUI_ID_USER + 0x01)
#define ID_SLIDER_1     (GUI_ID_USER + 0x02)
#define ID_SLIDER_2     (GUI_ID_USER + 0x03)
#define ID_EDIT_0       (GUI_ID_USER + 0x04)
#define ID_EDIT_1       (GUI_ID_USER + 0x05)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { SLIDER_CreateIndirect, "Slider", ID_SLIDER_0, 60, 10, 170, 40, 0, 0x0, 0 },
    { SLIDER_CreateIndirect, "Slider", ID_SLIDER_1, 70, 115, 40, 140, 8, 0x0, 0 },
    { SLIDER_CreateIndirect, "Slider", ID_SLIDER_2, 10, 65, 220, 40, 0, 0x0, 0 },
    { EDIT_CreateIndirect, "Edit", ID_EDIT_0, 10, 10, 50, 40, 0, 0x4, 0 },
    { EDIT_CreateIndirect, "Edit", ID_EDIT_1, 10, 115, 50, 40, 0, 0x3, 0 },
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
    WM_HWIN hSlider;
    WM_HWIN hEdit;
    int     NCode;
    int     Id;
    int     value;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化Slider0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_SLIDER_0);
        SLIDER_SetRange(hItem, 0, 1000);
        /* 初始化Slider1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_SLIDER_1);
        SLIDER_SetRange(hItem, 0, 100);
        SLIDER_SetWidth(hItem, 20);
        /* 初始化Slider2 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_SLIDER_2);
        SLIDER_SetSkinClassic(hItem);
        SLIDER_SetWidth(hItem, 30);
        /* 初始化Edit0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_0);
        EDIT_SetText(hItem, "0000");
        EDIT_SetTextAlign(hItem, GUI_TA_HCENTER | GUI_TA_VCENTER);
        EDIT_SetFont(hItem, GUI_FONT_COMIC18B_ASCII);
        EDIT_SetDecMode(hItem, 0, 0, 1000, 0, GUI_EDIT_NORMAL);
        /* 初始化Edit1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_1);
        EDIT_SetText(hItem, "000");
        EDIT_SetTextAlign(hItem, GUI_TA_HCENTER | GUI_TA_VCENTER);
        EDIT_SetFont(hItem, GUI_FONT_COMIC18B_ASCII);
        EDIT_SetDecMode(hItem, 0, 0, 100, 0, GUI_EDIT_NORMAL);
        break;
    case WM_NOTIFY_PARENT:
        Id = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch (Id) {
        case ID_SLIDER_0: // Notifications sent by 'Slider0'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                /* 滑块的值被改变，将改变后的值更新到EDIT控件中 */
                hSlider = WM_GetDialogItem(pMsg->hWin, ID_SLIDER_0);
                hEdit = WM_GetDialogItem(pMsg->hWin, ID_EDIT_0);
                value = SLIDER_GetValue(hSlider);
                EDIT_SetValue(hEdit, value);
                break;
            }
            break;
        case ID_SLIDER_1: // Notifications sent by 'Slider1'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                /* 滑块的值被改变，将改变后的值更新到EDIT控件中 */
                hSlider = WM_GetDialogItem(pMsg->hWin, ID_SLIDER_1);
                hEdit = WM_GetDialogItem(pMsg->hWin, ID_EDIT_1);
                value = SLIDER_GetValue(hSlider);
                EDIT_SetValue(hEdit, value);
                break;
            }
            break;
        case ID_EDIT_0: // Notifications sent by 'EDIT0'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        case ID_EDIT_1: // Notifications sent by 'EDIT1'
            switch (NCode) {
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
- 可以看到LCD显示滑块
- 滑动滑块后编辑框内的数值会跟着相应改变