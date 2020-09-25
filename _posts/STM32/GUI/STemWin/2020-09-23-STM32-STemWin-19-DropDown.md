---
title: 下拉框控件STemWin篇19
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 下拉框控件
---
# 下拉框控件API
- 向下拉框控件添加一个新元素`void DROPDOWN_AddString(DROPDOWN_Handle hObj, const char* s)`
- 关闭下拉框`void DROPDOWN_Collapse(DROPDOWN_Handle hObj)`
- 创建一个下拉框`DROPDOWN_Handle DROPDOWN_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int IdDROPDOWN_Handle )`
- 从对话框资源表中创建下拉框控件`DROPDOWN_Handle DROPDOWN_CreateIndirect(const GUI_WIDGET_CREATE_INFO* pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK* cb)`
- 使用额外的字节作为用户数据创建下拉框`DROPDOWN_Handle DROPDOWN_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumExtraBytes)`
- 递减选择前一个元素`void DROPDOWN_DecSel(DROPDOWN_Handle hObj)`
- 在展开的下拉框中递减选择前一个元素void `DROPDOWN_DecSelExp(DROPDOWN_Handle hObj)`
- 删除下拉框中的一个元素`void DROPDOWN_DeleteItem(DROPDOWN_Handle hObj, unsigned int Index)`
- 展看对话框`void DROPDOWN_Expand(DROPDOWN_Handle hObj)`
- 返回背景颜色`GUI_COLOR DROPDOWN_GetBkColor(DROPDOWN_Handle hObj, unsigned int Index)`
- 返回按钮或箭头使用的颜色`GUI_COLOR DROPDOWN_GetColor(DROPDOWN_Handle hObj, unsigned int Index)`
- 返回默认字体`const GUI_FONT * DROPDOWN_GetDefaultFont(void)`
- 返回当前字体`const GUI_FONT * DROPDOWN_GetFont(DROPDOWN_Handle hObj)`
- 返回给定项目的状态`unsigned DROPDOWN_GetItemDisabled(DROPDOWN_Handle hObj, unsigned Index)`
- 返回项目文本`int DROPDOWN_GetItemText(DROPDOWN_Handle hObj, unsigned Index, char * pBuffer, int MaxSize)`
- 以展开状态返回附加下拉框的句柄`LISTBOX_Handle DROPDOWN_GetListbox(DROPDOWN_Handle hObj)`
- 返回下拉框中的项数`int DROPDOWN_GetNumItems(DROPDOWN_Handle hObj)`
- 返回当前选定元素的编号`int DROPDOWN_GetSel(DROPDOWN_Handle hObj)`
- 返回处于展开状态的当前选定元素的编号`int DROPDOWN_GetSelExp(DROPDOWN_Handle hObj)`
- 返回文本颜色`GUI_COLOR DROPDWON_GetTextColor(DROPDOWN_Handle hObj, unsigned int Index)`
- 检索用户数据集`int DROPDOWN_GetUserData(DROPDOWN_Handle hObj, void * pDest, int NumBytes)`
- 递增选择后一个元素`void DROPDOWN_IncSel(DROPDOWN_Handle hObj)`
- 在展开的下拉框中递增选择后一个元素`void DROPDOWN_IncSelExp(DROPDOWN_Handle hObj)`
- 插入字符串到下拉框`void DROPDOWN_InsertString(DROPDOWN_Handle hObj, const char* s, unsigned int Index)`
- 启用自动滚动条`void DROPDOWN_SetAutoScroll(DROPDOWN_Handle hObj, int OnOff)`
- 设置背景颜色`void DROPDOWN_SetBkColor(DROPDOWN_Handle hObj, unsigned int Index, GUI_COLOR color)`
- 按钮或箭头使用的颜色`void DROPDOWN_SetColor(DROPDOWN_Handle hObj, unsigned int Index, GUI_COLOR color)`
- 设置默认按钮或箭头使用的颜色`GUI_COLOR DROPDOWN_SetDefaultColor(int Index, GUI_COLOR Color)`
- 设置默认字体`void DROPDOWN_SetDefaultFont(const GUI_FONT * pFont)`
- 设置下拉框滚动条的默认颜色`GUI_COLOR DROPDOWN_SetDefaultScrollbarColor(int Index, GUI_COLOR Color)`
- 设置文本字体`void DROPDOWN_SetFont(DROPDOWN_Handle hObj, const GUI_FONT * pfont)`
- 设置给定项目的状态`void DROPDOWN_SetItemDisabled(DROPDOWN_Handle hObj, unsigned Index, int OnOff)`
- 设置下拉框的元素间距`void DROPDOWN_SetItemSpacing(DROPDOWN_Handle hObj, unsigned Value)`
- 设置下拉框展开的高度，以像素为单位`int DROPDOWN_SetListHeight(DROPDOWN_Handle hObj, unsigned Height)`
- 设置下拉框滚动条颜色`void DROPDOWN_SetScrollbarColor(DROPDOWN_Handle hObj, unsigned Index, GUI_COLOR Color)`
- 设置下拉框滚动条宽度`void DROPDOWN_SetScrollbarWidth(DROPDOWN_Handle hObj, unsigned Width)`
- 设置当前选择的元素`void DROPDOWN_SetSel(DROPDOWN_Handle hObj, int Sel)`
- 设置展开状态选择的元素`void DROPDOWN_SetSelExp(DROPDOWN_Handle hObj, int Sel)`
- 设置文本对齐方式`void DROPDOWN_SetTextAlign(DROPDOWN_Handle hObj, int Align)`
- 设置文本颜色`void DROPDOWN_SetTextColor(DROPDOWN_Handle hObj, unsigned int index, GUI_COLOR color)`
- 设置在下拉框关闭状态的文本高度`void DROPDOWN_SetTextHeight(DROPDOWN_Handle hObj, unsigned TextHeight)`
- 设置下拉框为向上展看模式`int DROPDOWN_SetUpMode(DROPDOWN_Handle hObj, int OnOff)`
- 设置额外用户数据集`int DROPDOWN_SetUserData(DROPDOWN_Handle hObj, const void * pSrc, int NumBytes)`

# 下拉框控件实验
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
#define ID_DROPDOWN_0 (GUI_ID_USER + 0x01)
#define ID_DROPDOWN_1 (GUI_ID_USER + 0x02)
#define ID_TEXT_0 (GUI_ID_USER + 0x03)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { DROPDOWN_CreateIndirect, "Dropdown1", ID_DROPDOWN_0, 5, 140, 75, 25, 0, 0x0, 0 },
    { DROPDOWN_CreateIndirect, "Dropdown2", ID_DROPDOWN_1, 155, 140, 75, 25, 0, 0x0, 0 },
    { TEXT_CreateIndirect, "", ID_TEXT_0, 82, 138, 75, 25, 0, 0x0, 0 },
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
    int     value;
    char    Dropdown_buf[8] = {0};

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        /* 初始化DROPDOWN0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_DROPDOWN_0);
        DROPDOWN_SetListHeight(hItem, 100);
        DROPDOWN_SetFont(hItem, GUI_FONT_16B_1);
        DROPDOWN_AddString(hItem, "Item1-0");
        DROPDOWN_AddString(hItem, "Item1-1");
        DROPDOWN_AddString(hItem, "Item1-2");
        DROPDOWN_AddString(hItem, "Item1-3");
        DROPDOWN_AddString(hItem, "Item1-4");
        DROPDOWN_AddString(hItem, "Item1-5");
        DROPDOWN_AddString(hItem, "Item1-6");
        DROPDOWN_AddString(hItem, "Item1-7");
        DROPDOWN_SetAutoScroll(hItem, 1);
        DROPDOWN_SetScrollbarWidth(hItem, 20);
        /* 初始化DROPDOWN1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_DROPDOWN_1);
        DROPDOWN_SetListHeight(hItem, 100);
        DROPDOWN_SetFont(hItem, GUI_FONT_16B_1);
        DROPDOWN_AddString(hItem, "Item2-0");
        DROPDOWN_AddString(hItem, "Item2-1");
        DROPDOWN_AddString(hItem, "Item2-2");
        DROPDOWN_AddString(hItem, "Item2-3");
        DROPDOWN_AddString(hItem, "Item2-4");
        DROPDOWN_AddString(hItem, "Item2-5");
        DROPDOWN_AddString(hItem, "Item2-6");
        DROPDOWN_AddString(hItem, "Item2-7");
        DROPDOWN_SetAutoScroll(hItem, 1);
        DROPDOWN_SetScrollbarWidth(hItem, 20);
        DROPDOWN_SetUpMode(hItem, 1);
        /* 初始化TEXT */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetTextAlign(hItem, GUI_TA_HCENTER | GUI_TA_VCENTER);
        TEXT_SetFont(hItem, GUI_FONT_COMIC18B_ASCII);
        break;
    case WM_NOTIFY_PARENT:
        Id    = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch(Id) {
        case ID_DROPDOWN_0: // Notifications sent by 'Dropdown0'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_DROPDOWN_0);
                value = DROPDOWN_GetSel(hItem);
                DROPDOWN_GetItemText(hItem, value, Dropdown_buf, GUI_COUNTOF(Dropdown_buf));
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, Dropdown_buf);
                break;
            }
            break;
        case ID_DROPDOWN_1: // Notifications sent by 'Dropdown1'
            switch(NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_DROPDOWN_1);
                value = DROPDOWN_GetSel(hItem);
                DROPDOWN_GetItemText(hItem, value, Dropdown_buf, GUI_COUNTOF(Dropdown_buf));
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                TEXT_SetText(hItem, Dropdown_buf);
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
- 可以看到LCD显示下拉框
- 选择下拉框，可以看到显示选择项