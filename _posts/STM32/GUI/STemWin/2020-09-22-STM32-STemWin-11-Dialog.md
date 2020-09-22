---
title: 对话框STemWin篇11
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 对话框
---
# 对话框API
- 创建一个非阻塞式对话框`WM_HWIN GUI_CreateDialogBox(const GUI_WIDGET_CREATE_INFO * paWidget, int NumWidgets, WM_CALLBACK * cb, WM_HWIN hParent, int x0, int y0)`
- 执行已创建的对话框`int GUI_ExecCreatedDialog(WM_HWIN hDialog)`
- 创建一个阻塞式对话框`int GUI_ExecDialogBox(const GUI_WIDGET_CREATE_INFO * paWidget, int NumWidgets, WM_CALLBACK * cb, WM_HWIN hParent, int x0, int y0)`
- 结束一个对话框`void GUI_EndDialog(WM_HWIN hWin, int r)`

# 对话框
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
GUI_ConstString _apListBox[]={"ENGLISH","CHINESE","FRANCAIS","A"};

/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Dialog", 0, 10, 10, 180, 230, 0, 0 },
    { BUTTON_CreateIndirect, "OK", GUI_ID_OK, 100, 5, 60, 20, 0, 0 },
    { BUTTON_CreateIndirect, "Cancel", GUI_ID_CANCEL, 100, 30, 60, 20, 0, 0 },
    { TEXT_CreateIndirect, "LText", 0, 10, 55, 48, 15, TEXT_CF_LEFT, 0 },
    { TEXT_CreateIndirect, "RText", 0, 10, 80, 48, 15, TEXT_CF_RIGHT, 0 },
    { EDIT_CreateIndirect, NULL, GUI_ID_EDIT0, 60, 55, 100, 15, 0, 50 },
    { EDIT_CreateIndirect, NULL, GUI_ID_EDIT1, 60, 80, 100, 15, 0, 50 },
    { TEXT_CreateIndirect, "Hex", 0, 10, 100, 48, 15, TEXT_CF_RIGHT, 0 },
    { EDIT_CreateIndirect, NULL, GUI_ID_EDIT2, 60, 100, 100, 15, 0, 6 },
    { TEXT_CreateIndirect, "Bin", 0, 10, 120, 48, 15, TEXT_CF_RIGHT, 0 },
    { EDIT_CreateIndirect, NULL, GUI_ID_EDIT3, 60, 120, 100, 15, 0, 0 },
    { LISTBOX_CreateIndirect, NULL, GUI_ID_LISTBOX0, 10, 10, 48, 40, 0, 0 },
    { CHECKBOX_CreateIndirect, NULL, GUI_ID_CHECK0, 10, 140, 0, 0, 0, 0 },
    { CHECKBOX_CreateIndirect, NULL, GUI_ID_CHECK1, 30, 140, 0, 0, 0, 0 },
    { SLIDER_CreateIndirect, NULL, GUI_ID_SLIDER0, 60, 140, 100, 20, 0, 0 },
    { SLIDER_CreateIndirect, NULL, GUI_ID_SLIDER1, 10, 170, 150, 30, 0, 0 }
};
/* 对话框回调函数 */
static void _cbCallback(WM_MESSAGE * pMsg)
{
    WM_HWIN hItem;
    WM_HWIN hWin;
    hWin = pMsg->hWin;
    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        hItem = WM_GetDialogItem(hWin, GUI_ID_EDIT0);
        EDIT_SetText(hItem, "EDIT widget 0");
        hItem = WM_GetDialogItem(hWin, GUI_ID_EDIT1);
        EDIT_SetText(hItem, "EDIT widget 1");
        EDIT_SetTextAlign(hItem, GUI_TA_LEFT);
        hItem = WM_GetDialogItem(hWin, GUI_ID_EDIT2);
        EDIT_SetHexMode(hItem, 0x1234, 0, 0xffff);
        hItem = WM_GetDialogItem(hWin, GUI_ID_EDIT3);
        EDIT_SetBinMode(hItem, 0x1234, 0, 0xffff);
        hItem = WM_GetDialogItem(hWin, GUI_ID_CHECK0);
        CHECKBOX_Check(WM_GetDialogItem(hWin, GUI_ID_CHECK0));
        hItem = WM_GetDialogItem(hWin, GUI_ID_CHECK1);
        WM_DisableWindow(WM_GetDialogItem(hWin, GUI_ID_CHECK1));
        CHECKBOX_Check(WM_GetDialogItem(hWin, GUI_ID_CHECK1));
        hItem = WM_GetDialogItem(hWin, GUI_ID_SLIDER0);
        SLIDER_SetWidth(WM_GetDialogItem(hWin, GUI_ID_SLIDER0), 5);
        hItem = WM_GetDialogItem(hWin, GUI_ID_SLIDER1);
        SLIDER_SetValue(WM_GetDialogItem(hWin, GUI_ID_SLIDER1), 50);
        hItem = WM_GetDialogItem(hWin, GUI_ID_LISTBOX0);
        LISTBOX_SetText(hItem, _apListBox);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}
/*******************************************************************************
 *      static code
 ******************************************************************************/

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
    GUI_ExecDialogBox(_aDialogCreate, GUI_COUNTOF(_aDialogCreate),_cbCallback, 0, 0, 0);
    while (1)
    {

    }
}

```

# 编译调试
- 编译并下载到开发板
- LCD显示运行结果