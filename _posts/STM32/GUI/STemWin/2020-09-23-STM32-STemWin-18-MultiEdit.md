---
title: 多行文本控件STemWin篇18
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 多行文本控件
---
# 多行文本控件API
- 按键输入函数`int MULTIEDIT_AddKey(MULTIEDIT_HANDLE hObj, U16 Key)`
- 在当前光标位置添加文本`int MULTIEDIT_AddText(MULTIEDIT_HANDLE hObj, const char * s)`
- 创建一个多行文本`MULTIEDIT_HANDLE MULTIEDIT_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int BufferSize, const char * pTextMULTIEDIT_HANDLE )`
- 从对话框资源表中创建多行文本控件`MULTIEDIT_HANDLE MULTIEDIT_CreateIndirect(const GUI_WIDGET_CREATE_INFO* pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建多行文本`MULTIEDIT_HANDLE MULTIEDIT_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int BufferSize, const char * pText, int NumExtraBytes)`
- 启用/禁用光标闪烁`void MULTIEDIT_EnableBlink(MULTIEDIT_HANDLE hObj, int Period, int OnOff)`
- 放回当前背景颜色`GUI_COLOR MULTIEDIT_GetBkColor(MULTIEDIT_HANDLE hObj, unsigned Index)`
- 返回光标所在的字符位置`int MULTIEDIT_GetCursorCharPos(MULTIEDIT_HANDLE hObj)`
- 返回光标所在的像素位置`void MULTIEDIT_GetCursorPixelPos(MULTIEDIT_HANDLE hObj, int * pxPos, int * pyPos)`
- 返回当前字体`const GUI_FONT * MULTIEDIT_GetFont(MULTIEDIT_HANDLE hObj)`
- 返回提示符文本`void MULTIEDIT_GetPrompt(MULTIEDIT_HANDLE hObj, char* sDest, int MaxNumChars)`
- 返回当前文本`void MULTIEDIT_GetText(MULTIEDIT_HANDLE hObj, char* sDest, int MaxNumChars)`
- 返回当前文本颜色`GUI_COLOR MULTIEDIT_GetTextColor(MULTIEDIT_HANDLE hObj, unsigned Index)`
- 返回当前文本使用的缓冲区大小`int MULTIEDIT_GetTextSize(MULTIEDIT_HANDLE hObj)`
- 检索用户数据集`int MULTIEDIT_GetUserData(MULTIEDIT_HANDLE hObj, void * pDest, int NumBytes)`
- 激活自动使用水平滚动条`void MULTIEDIT_SetAutoScrollH(MULTIEDIT_HANDLE hObj, int OnOff)`
- 激活自动使用垂直滚动条`void MULTIEDIT_SetAutoScrollV(MULTIEDIT_HANDLE hObj, int OnOff)`
- 设置当前背景颜色`void MULTIEDIT_SetBkColor(MULTIEDIT_HANDLE hObj, unsigned Index, GUI_COLOR color)`
- 设置用于文本和提示符的缓冲区大小`void MULTIEDIT_SetBufferSize(MULTIEDIT_HANDLE hObj, int BufferSize)`
- 将光标设置到给定的字符位置`void MULTIEDIT_SetCursorCharPos(MULTIEDIT_HANDLE hObj, int x, int y)`
- 将光标设置为给定字符`void MULTIEDIT_SetCursorOffset(MULTIEDIT_HANDLE hObj, int Offset)`
- 将光标设置为给定的像素位置`void MULTIEDIT_SetCursorPixelPos(MULTIEDIT_HANDLE hObj, int x, int y)`
- 设置指定控件的可聚焦性`void MULTIEDIT_SetFocusable(MULTIEDIT_HANDLE hObj, int State)`
- 设置当前字体`void MULTIEDIT_SetFont(MULTIEDIT_HANDLE hObj, const GUI_FONT * pFont)`
- 启用/禁用插入模式`void MULTIEDIT_SetInsertMode(MULTIEDIT_HANDLE hObj, int OnOff)`
- 设置包括提示符在内的最大字符数`void MULTIEDIT_SetMaxNumChars(MULTIEDIT_HANDLE hObj, unsigned MaxNumChars)`
- 启用/禁用密码模式`void MULTIEDIT_SetPasswordMode(MULTIEDIT_HANDLE hObj, int OnOff)`
- 设置提示符文本`void MULTIEDIT_SetPrompt(MULTIEDIT_HANDLE hObj, const char* sPrompt)`
- 启用/禁用只读模式`void MULTIEDIT_SetReadOnly(MULTIEDIT_HANDLE hObj, int OnOff)`
- 设置当前文本`void MULTIEDIT_SetText(MULTIEDIT_HANDLE hObj, const char* s)`
- 设置文本对齐方式`void MULTIEDIT_SetTextAlign(MULTIEDIT_HANDLE hObj, int Align)`
- 设置文本颜色`void MULTIEDIT_SetTextColor(MULTIEDIT_HANDLE hObj, unsigned Index, GUI_COLOR color)`
- 设置额外用户数据集`int MULTIEDIT_SetUserData(MULTIEDIT_HANDLE hObj, const void * pSrc, int NumBytes)`
- 启用/禁用自动换行`void MULTIEDIT_SetWrapWord(MULTIEDIT_HANDLE hObj)`
- 启用/禁用非换行模式`void MULTIEDIT_SetWrapNone(MULTIEDIT_HANDLE hObj)`
- 启用/禁用光标`int MULTIEDIT_ShowCursor(MULTIEDIT_HANDLE hObj, unsigned OnOff)`

# 多行文本控件实验
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
#define ID_MULTIEDIT_0   (GUI_ID_USER + 0x01)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { MULTIEDIT_CreateIndirect, "Multiedit0", ID_MULTIEDIT_0, 10, 40, 200, 100, 0, 0x0, 0 },
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
    int     NCode;
    int     Id;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN@ STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化MULTIEDIT控件 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_MULTIEDIT_0);
        MULTIEDIT_SetReadOnly(hItem, 1);
        MULTIEDIT_ShowCursor(hItem, 0);
        MULTIEDIT_SetBufferSize(hItem, 500);
        MULTIEDIT_SetAutoScrollV(hItem, 1);
        MULTIEDIT_SetFont(hItem, GUI_FONT_COMIC18B_ASCII);
        MULTIEDIT_SetBkColor(hItem, MULTIEDIT_CI_READONLY, GUI_BLACK);
        MULTIEDIT_SetTextColor(hItem, MULTIEDIT_CI_READONLY, GUI_GREEN);
        MULTIEDIT_SetTextAlign(hItem, GUI_TA_LEFT);
        /* 显示内容 */
        MULTIEDIT_AddText(hItem, "\\*****************\\\r\n");
        MULTIEDIT_AddText(hItem, "STemWIN\r\nVersion: ");
        MULTIEDIT_AddText(hItem, GUI_GetVersionString());
        MULTIEDIT_AddText(hItem, "\r\n\\*****************\\\r\n");
        break;
    case WM_NOTIFY_PARENT:
        Id = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch (Id) {
        case ID_MULTIEDIT_0: // Notifications sent by 'Multiedit'
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
- 可以看到LCD显示多行文本