---
title: 框架窗口STemWin篇12
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 框架窗口
---
# 框架窗口API
- 在标题栏中添加一个按钮`WM_HWIN FRAMEWIN_AddButton(FRAMEWIN_Handle hObj, int Flags, int Off, int Id)`
- 在标题栏中添加一个关闭按钮`WM_HWIN FRAMEWIN_AddCloseButton(FRAMEWIN_Handle hObj, int Flags, int Off)`
- 直接方式创建一个框架窗口`FRAMEWIN_Handle FRAMEWIN_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const char * pTitle, WM_CALLBACK * cb)`
- 对话框方式创建一个框架窗口`FRAMEWIN_Handle FRAMEWIN_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外字节作为用户数据创建框架窗口`FRAMEWIN_Handle FRAMEWIN_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const char * pTitle, WM_CALLBACK * cb, int NumExtraBytes)`
- 返回框架窗口的活动状态`int FRAMEWIN_GetActive(FRAMEWIN_Handle hObj)`
- 检索用户数据集`int FRAMEWIN_GetUserData(FRAMEWIN_Handle hObj, void * pDest, int NumBytes)`
- 返回框架窗口是否最小化`int FRAMEWIN_IsMinimized(FRAMEWIN_Handle hObj)`
- 返回框架窗口是否最大化`int FRAMEWIN_IsMaximized(FRAMEWIN_Handle hObj)`
- 将框架窗口放大到其父窗口的大小`void FRAMEWIN_Maximize(FRAMEWIN_Handle hObj)`
- 隐藏框架窗口的客户区域`void FRAMEWIN_Minimize(FRAMEWIN_Handle hObj)`
- 绘制标题栏的默认函数`int FRAMEWIN_OwnerDraw(const WIDGET_ITEM_DRAW_INFO * pDrawItemInfo)`
- 还原已被最小化或最大化的框架窗口`void FRAMEWIN_Restore(FRAMEWIN_Handle hObj)`
- 设置标题栏颜色`void FRAMEWIN_SetBarColor(FRAMEWIN_Handle hObj, unsigned Index, GUI_COLOR Color)`
- 设置边框尺寸`void FRAMEWIN_SetBorderSize(FRAMEWIN_Handle hObj, unsigned Size)`
- 设置客户区颜色`void FRAMEWIN_SetClientColor(FRAMEWIN_Handle hObj, GUI_COLOR Color)`
- 设置本文字体`void FRAMEWIN_SetFont(FRAMEWIN_Handle hObj, const GUI_FONT * pFont)`
- 设置框架窗口是否可移动`void FRAMEWIN_SetMoveable(FRAMEWIN_Handle hObj, int State)`
- 绘制自定义标题栏`void FRAMEWIN_SetOwnerDraw(FRAMEWIN_Handle hObj, WIDGET_DRAW_ITEM_FUNC * pfDrawItem)`
- 设置标题栏文本`void FRAMEWIN_SetText(FRAMEWIN_Handle hObj, const char* s)`
- 设置文本对齐方式`void FRAMEWIN_SetTextAlign(FRAMEWIN_Handle hObj, int Align)`
- 设置文本颜色`void FRAMEWIN_SetTextColor(FRAMEWIN_Handle hObj, GUI_COLOR Color)`
- 设置标题栏高度`int FRAMEWIN_SetTitleHeight(FRAMEWIN_Handle hObj, int Height)`
- 设置标题栏是否可见`void FRAMEWIN_SetTitleVis(FRAMEWIN_Handle hObj, int Show)`

# 框架窗口实验
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
#define ID_BUTTON_0   (GUI_ID_USER + 0x01)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
/* 资源表 */
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320,FRAMEWIN_CF_MOVEABLE, 0x0, 0 },
    { BUTTON_CreateIndirect, "Button0", ID_BUTTON_0, 10, 30, 160, 48, 0, 0x0, 0 },
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
        /* 初始化框架窗口控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetFont(hItem, GUI_FONT_16_1);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        /* 初始化Button0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_BUTTON_0);
        BUTTON_SetFont(hItem, GUI_FONT_24B_ASCII);
        break;
    case WM_NOTIFY_PARENT:
        /* 获取控件ID */
        Id = WM_GetId(pMsg->hWinSrc);
        /* 获取消息内容 */
        NCode = pMsg->Data.v;
        switch(Id)
        {
        case ID_BUTTON_0: // Notifications sent by 'Button'
            switch(NCode)
            {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
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
    /* 设置桌面窗口颜色 */
    WM_SetDesktopColor(GUI_BLUE);

    /* 创建对话框 */
    CreateFramewin();
    /* 开启光标 */
    GUI_CURSOR_Show();
    while (1)
    {
        GUI_Delay(500);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示

