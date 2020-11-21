---
title: 进度条控件STemWin篇23
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 进度条控件
---
# 进度条控件API
- 创建一个进度条控件`PROGBAR_Handle PROGBAR_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id)`
- 从对话框资源表中创建进度条控件`PROGBAR_Handle PROGBAR_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外字节作为用户数据创建进度条`PROGBAR_Handle PROGBAR_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumExtraBytes)`
- 返回进度条颜色`GUI_COLOR PROGBAR_GetBarColor(PROGBAR_Handle hObj, unsigned int Index)`
- 返回文本字体`const GUI_FONT * PROGBAR_GetFont(PROGBAR_Handle hObj)`
- 返回最大最小值`void PROGBAR_GetMinMax(PROGBAR_Handle hObj, int * pMin, int * pMax)`
- 返回文本颜色`GUI_COLOR PROGBAR_GetTextColor(PROGBAR_Handle hObj, unsigned int Index)`
- 检索用户数据集`int PROGBAR_GetUserData(PROGBAR_Handle hObj, void * pDest, int NumBytes)`
- 返回当前进度值`int PROGBAR_GetValue(PROGBAR_Handle hObj)`
- 设置进度条颜色`void PROGBAR_SetBarColor(PROGBAR_Handle hObj, unsigned int index, GUI_COLOR color)`
- 设置文本字体`void PROGBAR_SetFont(PROGBAR_Handle hObj, const GUI_FONT * pfont)`
- 设置最大最小值`void PROGBAR_SetMinMax(PROGBAR_Handle hObj, int Min, int Max)`
- 设置文本`void PROGBAR_SetText(PROGBAR_Handle hObj, const char* s)`
- 设置文本对齐方式`void PROGBAR_SetTextAlign(PROGBAR_Handle hObj, int Align)`
- 设置文本颜色`void PROGBAR_SetTextColor(PROGBAR_Handle hObj, unsigned int index, GUI_COLOR color)`
- 设置文本位置`void PROGBAR_SetTextPos(PROGBAR_Handle hObj, int XOff, int YOff)`
- 设置额外用户数据集`int PROGBAR_SetUserData(PROGBAR_Handle hObj, const void * pSrc, int NumBytes)`
- 设置进度值(如果没有文本，则设置百分比)`void PROGBAR_SetValue(PROGBAR_Handle hObj, int v)`

# 进度条控件实验
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
#define ID_FRAMEWIN_0     (GUI_ID_USER + 0x00)
#define ID_PROGBAR_0     (GUI_ID_USER + 0x01)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { PROGBAR_CreateIndirect, "Progbar", ID_PROGBAR_0, 10, 90, 210, 40, 0, 0x0, 0 },
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
static void _cbDialog(WM_MESSAGE * pMsg) {
    WM_HWIN hItem;
    static U16 progbar_value = 0;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化Progbar0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_PROGBAR_0);
        PROGBAR_SetFont(hItem, GUI_FONT_COMIC24B_ASCII);
        PROGBAR_SetMinMax(hItem, 0, 100);
        break;
    case WM_PAINT:
        hItem = WM_GetDialogItem(pMsg->hWin, ID_PROGBAR_0);
        progbar_value = PROGBAR_GetValue(hItem);
        PROGBAR_SetValue(hItem, progbar_value+1);
        if(progbar_value == 100)
            PROGBAR_SetValue(hItem, 0);
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
    WM_HWIN hWin;

    hWin = CreateFramewin();
    while (1)
    {
        GUI_Delay(10);
        WM_InvalidateWindow(hWin);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示进度条
- 进度条从左侧0%一直增加到右侧100%，到100%后又从0%开始