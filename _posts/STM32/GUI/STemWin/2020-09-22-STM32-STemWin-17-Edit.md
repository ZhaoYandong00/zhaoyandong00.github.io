---
title: 编辑框控件STemWin篇17
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 编辑框控件
---
# 编辑框控件API
- 添加按键输入`void EDIT_AddKey(EDIT_Handle hObj, int Key)`
- 创建编辑框`EDIT_Handle EDIT_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int MaxLen)`
- 从对话框资源表中创建编辑框控件`EDIT_Handle EDIT_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建编辑框`EDIT_Handle EDIT_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int MaxLen, int NumExtraBytes)`
- 启用/禁用光标闪烁`void EDIT_EnableBlink(EDIT_Handle hObj, int Period, int OnOff)`
- 启用/禁用反向光标（默认为反向）`int EDIT_EnableInversion(EDIT_Handle hObj, int OnOff)`
- 返回编辑框背景颜色`GUI_COLOR EDIT_GetBkColor(EDIT_Handle hObj, unsigned int Index)`
- 返回光标位置的字符数`int EDIT_GetCursorCharPos(EDIT_Handle hObj)`
- 返回光标的像素位置`void EDIT_GetCursorPixelPos(EDIT_Handle hObj, int * pxPos, int * pyPos)`
- 返回默认背景颜色`GUI_COLOR EDIT_GetDefaultBkColor(unsigned int Index)`
- 返回默认字体`const GUI_FONT * EDIT_GetDefaultFont(void)`
- 返回默认文本对齐方式`int EDIT_GetDefaultTextAlign(void)`
- 返回默认文本颜色`GUI_COLOR  EDIT_GetDefaultTextColor(unsigned int Index)`
- 以浮点值的形式返回当前值`float EDIT_GetFloatValue(EDIT_Handle hObj)`
- 返回当前字体`const GUI_FONT * EDIT_GetFont(EDIT_Handle hObj)`
- 返回给定编辑框的字符数`int EDIT_GetNumChars(EDIT_Handle hObj)`
- 返回用户输入的字符`void EDIT_GetText(EDIT_Handle hObj, char * sDest, int MaxLen)`
- 返回当前文本对齐方式`int EDIT_GetTextAlign(EDIT_Handle hObj)`
- 放回当前文本颜色`GUI_COLOR EDIT_GetTextColor(EDIT_Handle hObj, unsigned int Index)`
- 检索用户数据集`int EDIT_GetUserData(EDIT_Handle hObj, void * pDest, int NumBytes)`
- 返回当前的数值`I32 EDIT_GetValue(EDIT_Handle hObj)`
- 设置编辑框为二进制编辑模式`void EDIT_SetBinMode(EDIT_Handle hEdit, U32 Value, U32 Min, U32 Max)`
- 设置当前背景颜色`GUI_COLOR EDIT_SetBkColor(EDIT_Handle hObj, unsigned int Index, GUI_COLOR color)`
- 将编辑框的光标设置到指定字符位置`void EDIT_SetCursorAtChar(EDIT_Handle hObj, int Pos)`
- 将编辑框的光标设置到指定像素位置`void EDIT_SetCursorAtPixel(EDIT_Handle hObj, int xPos)`
- 设置编辑框为十进制编辑模式`void EDIT_SetDecMode(EDIT_Handle hEdit, I32 Value, I32 Min, I32 Max, int Shift, U8 Flags)`
- 设置默认背景颜色`void EDIT_SetDefaultBkColor(unsigned int Index, GUI_COLOR Color)`
- 设置默认字体`void EDIT_SetDefaultFont(const GUI_FONT * pFont)`
- 设置默认文本对齐方式`void EDIT_SetDefaultTextAlign(int Align)`
- 设置默认文本颜色`void EDIT_SetDefaultTextColor(unsigned int Index, GUI_COLOR Color)`
- 设置编辑框为浮点数编辑模式`void EDIT_SetFloatMode(EDIT_Handle hEdit, float Value, float Min, float Max, int Shift, U8 Flags)`
- 如果使用浮点编辑模式，则设置浮点值`void EDIT_SetFloatValue(EDIT_Handle hObj, float Value)`
- 设置编辑框的可聚焦性`void EDIT_SetFocusable(WM_HWIN hObj, int State)`
- 设置当前字体`void EDIT_SetFont(EDIT_Handle hObj, const GUI_FONT * pFont)`
- 设置编辑框为十六进制数编辑模式`void EDIT_SetHexMode(EDIT_Handle hEdit, U32 Value, U32 Min, U32 Max)`
- 启用或禁用插入模式`int EDIT_SetInsertMode(EDIT_Handle hObj, int OnOff)`
- 设置编辑框的最大字符数`void EDIT_SetMaxLen(EDIT_Handle hObj, int MaxLen)`
- 向编辑框中添加字符`void EDIT_SetpfAddKeyEx(EDIT_Handle hObj, tEDIT_AddKeyEx * pfAddKeyEx)`
- 设置当前选项`void EDIT_SetSel(EDIT_Handle hObj, int FirstChar, int LastChar)`
- 设置当前文本`void EDIT_SetText(EDIT_Handle hObj, const char * s)`
- 设置文本对齐方式`void EDIT_SetTextAlign(EDIT_Handle hObj, int Align)`
- 设置当前文本颜色`void EDIT_SetTextColor(EDIT_Handle hObj, unsigned int Index, GUI_COLOR Color)`
- 设置编辑框为文本编辑模式`void EDIT_SetTextMode(EDIT_Handle hEdit)`
- 设置当前数值`void EDIT_SetValue(EDIT_Handle hObj, I32 Value)`
- 设置编辑框为无符号长十进制编辑模式`void EDIT_SetUlongMode(EDIT_Handle hEdit, U32 Value, U32 Min, U32 Max)`
- 设置额外用户数据集`int EDIT_SetUserData(EDIT_Handle hObj, const void * pSrc, int NumBytes)`
- 在当前光标位置编辑二进制值`U32 GUI_EditBin(U32 Value, U32 Min, U32 Max, int Len, int xSize)`
- 在当前光标位置编辑十进制值`I32 GUI_EditDec(I32 Value, I32 Min, I32 Max, int Len, int xSize, int Shift, U8 Flags)`
- 在当前光标位置编辑浮点值`float GUI_EditFloat(float Value, float Min, float Max, int Len, int xSize, int Shift, U8 Flags)`
- 在当前光标位置编辑十六进制值`U32 GUI_EditHex(U32 Value, U32 Min, U32 Max, int Len, int xSize)`
- 在当前光标位置编辑字符串`void GUI_EditString(char * pString, int Len, int xSize)`

# 编辑框控件实验
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
#define ID_FRAMEWIN_0 (GUI_ID_USER + 0x00)
#define ID_EDIT_0     (GUI_ID_USER + 0x01)
#define ID_EDIT_1     (GUI_ID_USER + 0x02)
#define ID_EDIT_2     (GUI_ID_USER + 0x03)
#define ID_EDIT_3     (GUI_ID_USER + 0x04)
#define ID_TEXT_0     (GUI_ID_USER + 0x05)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { EDIT_CreateIndirect, "Edit0", ID_EDIT_0, 5, 40, 220, 50, 0, 0x64, 0 },
    { EDIT_CreateIndirect, "Edit1", ID_EDIT_1, 5, 110, 100, 50, 0, 0x64, 0 },
    { EDIT_CreateIndirect, "Edit2", ID_EDIT_2, 125, 110, 100, 50, 0, 0x64, 0 },
    { EDIT_CreateIndirect, "Edit3", ID_EDIT_3, 5, 180, 220, 50, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "", ID_TEXT_0, 5, 240, 220, 50, 0, 0x64, 0 },
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
    int     NCode;
    int     Id;
    char    EditBuff[30] = {0};

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* Edit0初始化 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_0);
        EDIT_SetText(hItem, "STemWIN@ STM32F103");
        EDIT_SetFont(hItem, GUI_FONT_16_ASCII);
        EDIT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        EDIT_EnableBlink(hItem, 500, 1);
        /* Edit1初始化 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_1);
        EDIT_SetFloatMode(hItem, 3.1415926, 0.0, 10.0, 7, GUI_EDIT_NORMAL);
        EDIT_SetFont(hItem, GUI_FONT_16_ASCII);
        EDIT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        EDIT_EnableBlink(hItem, 500, 1);
        /* Edit2初始化 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_2);
        EDIT_SetMaxLen(hItem, 8);
        EDIT_SetHexMode(hItem, 232425, 0, 4294967295);
        EDIT_SetFont(hItem, GUI_FONT_16_ASCII);
        EDIT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        EDIT_EnableBlink(hItem, 500, 1);
        /* Edit3初始化 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_3);
        EDIT_SetMaxLen(hItem, 28);
        EDIT_SetBinMode(hItem, 123456789, 0, 268435455);
        EDIT_SetFont(hItem, GUI_FONT_16_ASCII);
        EDIT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        EDIT_EnableBlink(hItem, 500, 1);
        /* 初始化Text0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetFont(hItem, GUI_FONT_16B_ASCII);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        break;
    case WM_NOTIFY_PARENT:
        Id    = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch(Id) {
        case ID_EDIT_0: // Notifications sent by 'Edit0'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_0);
                EDIT_GetText(hItem, EditBuff, 40);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, EditBuff);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        case ID_EDIT_1: // Notifications sent by 'Edit1'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_1);
                EDIT_GetText(hItem, EditBuff, 40);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, EditBuff);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        case ID_EDIT_2: // Notifications sent by 'Edit2'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_2);
                EDIT_GetText(hItem, EditBuff, 40);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, EditBuff);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        case ID_EDIT_3: // Notifications sent by 'Edit3'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_EDIT_3);
                EDIT_GetText(hItem, EditBuff, 40);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, EditBuff);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_VALUE_CHANGED:
                break;
            }
            break;
        }
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
- 点击编辑框文本显示出现变化