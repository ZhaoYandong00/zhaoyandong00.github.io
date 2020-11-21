---
title: 列表轮控件STemWin篇21
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 列表轮控件
---
# 列表轮控件API
- 添加一个新的字符串`void LISTWHEEL_AddString(LISTWHEEL_Handle hObj, const char * s)`
- 创建一个列表轮控件`LISTWHEEL_Handle LISTWHEEL_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const GUI_ConstString * ppText)`
- 从对话框资源表中创建列表轮控件`LISTWHEEL_Handle LISTWHEEL_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建列表轮`LISTWHEEL_Handle LISTWHEEL_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, const GUI_ConstString * ppText, int NumExtraBytes)`
- 获取当前背景颜色`GUI_COLOR LISTWHEEL_GetBkColor(LISTWHEEL_Handle hObj, unsigned int Index)`
- 获取当前字体`const GUI_FONT * LISTWHEEL_GetFont(LISTWHEEL_Handle hObj)`
- 返回与给定位置匹配的项目的索引`int LISTWHEEL_GetItemFromPos(LISTWHEEL_Handle hObj, int yPos)`
- 返回请求项的文本`void LISTWHEEL_GetItemText(LISTWHEEL_Handle hObj, unsigned Index, char * pBuffer, int MaxSize)`
- 返回左边框的大小，以像素为单位`int LISTWHEEL_GetLBorder(LISTWHEEL_Handle hObj)`
- 返回一项的高度`unsigned LISTWHEEL_GetLineHeight(LISTWHEEL_Handle hObj)`
- 返回数据项数`int LISTWHEEL_GetNumItems(LISTWHEEL_Handle hObj)`
- 返回当前参与项目的项目索引`int LISTWHEEL_GetPos(LISTWHEEL_Handle hObj)`
- 返回右边框的大小，以像素为单位`int LISTWHEEL_GetRBorder(LISTWHEEL_Handle hObj)`
- 返回当前选择的项目`int LISTWHEEL_GetSel(LISTWHEEL_Handle hObj)`
- 返回捕捉位置，以像素为单位`int LISTWHEEL_GetSnapPosition(LISTWHEEL_Handle hObj)`
- 返回用于绘制数据项的文本对齐方式`int LISTWHEEL_GetTextAlign(LISTWHEEL_Handle hObj)`
- 返回文本颜色`GUI_COLOR LISTWHEEL_GetTextColor(LISTWHEEL_Handle hObj, unsigned int Index)`
- 检索用户数据`int LISTWHEEL_GetUserData(LISTWHEEL_Handle hObj, void * pDest, int NumBytes)`
- 返回控件是否在移动`int LISTWHEEL_IsMoving(LISTWHEEL_Handle hObj)`
- 将列表轮移动到指定位置`void LISTWHEEL_MoveToPos(LISTWHEEL_Handle hObj, unsigned int Index)`
- 绘制控件件的默认函数`int LISTWHEEL_OwnerDraw(const WIDGET_ITEM_DRAW_INFO * pDrawItemInfo)`
- 设置当前背景颜色`void LISTWHEEL_SetBkColor(LISTWHEEL_Handle hObj, unsigned int Index, GUI_COLOR Color)`
- 设置轮的减速度`void LISTWHEEL_SetDeceleration(LISTWHEEL_Handle hObj, unsigned Deceleration)`
- 设置当前字体`void LISTWHEEL_SetFont(LISTWHEEL_Handle hObj, const GUI_FONT * pFont)`
- 设置左边框的大小，以像素为单位`void LISTWHEEL_SetLBorder(LISTWHEEL_Handle hObj, unsigned BorderSize)`
- 设置一个数据项的高度`void LISTWHEEL_SetLineHeight(LISTWHEEL_Handle hObj, unsigned LineHeight)`
- 设置用于绘制控件的所有者绘图函数`void LISTWHEEL_SetOwnerDraw(LISTWHEEL_Handle hObj, WIDGET_DRAW_ITEM_FUNC * pfOwnerDraw)`
- 将列表轮设置为给定位置`void LISTWHEEL_SetPos(LISTWHEEL_Handle hObj, unsigned int Index)`
- 设置右边框的大小，以像素为单位`void LISTWHEEL_SetRBorder(LISTWHEEL_Handle hObj, unsigned BorderSize)`
- 设置当前选定的项`void LISTWHEEL_SetSel(LISTWHEEL_Handle hObj, int Sel)`
- 从控件的顶部设置对齐位置，以像素为单位`void LISTWHEEL_SetSnapPosition(LISTWHEEL_Handle hObj, int SnapPosition)`
- 设置控件的文本`void LISTWHEEL_SetText(LISTWHEEL_Handle hObj, const GUI_ConstString * ppText)`
- 设置文本对齐方式`void LISTWHEEL_SetTextAlign(LISTWHEEL_Handle hObj, int Align)`
- 设置文本颜色`void LISTWHEEL_SetTextColor(LISTWHEEL_Handle hObj, unsigned int Index, GUI_COLOR Color)`
- 设置定时器周期`void LISTWHEEL_SetTimerPeriod(LISTWHEEL_Handle hObj, GUI_TIMER_TIME TimerPeriod)`
- 设置额外用户数据`int LISTWHEEL_SetUserData(LISTWHEEL_Handle hObj, const void * pSrc, int NumBytes)`
- 设置轮环的移动速度`void LISTWHEEL_SetVelocity(LISTWHEEL_Handle hObj, int Velocity)`

# 列表轮实验
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
#define ID_FRAMEWIN_0  (GUI_ID_USER + 0x00)
#define ID_LISTWHEEL_0 (GUI_ID_USER + 0x01)
#define ID_LISTWHEEL_1 (GUI_ID_USER + 0x02)
#define ID_LISTWHEEL_2 (GUI_ID_USER + 0x03)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static char *_apYear[] = {
    "1990","1991", "1992", "1993", "1994", "1995", "1996",
    "1997", "1998", "1999", "2000", "2001", "2002", "2003",
    "2004", "2005", "2006", "2007", "2008", "2009", "2010",
    "2011", "2012", "2013", "2014", "2015", "2016", "2017",
    "2018", "2019", "2020",
};

static char *_apMonth[] = {
    "January",
    "February",
    "March",
    "April",
    "May",
    "June",
    "July",
    "August",
    "September",
    "October",
    "November",
    "December",
};

static char *_apDay[] = {
    "01", "02", "03", "04",
    "05", "06", "07", "08",
    "09", "10", "11", "12",
    "13", "14", "15", "16",
    "17", "18", "19", "20",
    "21", "22", "23", "24",
    "25", "26", "27", "28",
    "29", "30", "31",
};
static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { LISTWHEEL_CreateIndirect, "", ID_LISTWHEEL_0, 10, 50, 60, 178, WM_CF_MEMDEV, 0x0, 0 },
    { LISTWHEEL_CreateIndirect, "", ID_LISTWHEEL_1, 80, 50, 100, 178, WM_CF_MEMDEV, 0x0, 0 },
    { LISTWHEEL_CreateIndirect, "", ID_LISTWHEEL_2, 180, 50, 50, 178, WM_CF_MEMDEV, 0x0, 0 },
};
/*******************************************************************************
 *      static code
 ******************************************************************************/
/**
  * @brief 用户绘制函数
  * @note 无
  * @param pDrawItemInfo：指向WIDGET_ITEM_DRAW_INFO结构的指针
  * @retval 默认绘制函数 或 0
  */
static int _OwnerDraw(const WIDGET_ITEM_DRAW_INFO * pDrawItemInfo)
{
    GUI_RECT aRect;

    switch (pDrawItemInfo->Cmd) {
    case WIDGET_ITEM_DRAW_OVERLAY:
        /* 获取控件坐标 */
        aRect.x0 = pDrawItemInfo->x0;
        aRect.x1 = pDrawItemInfo->x1;
        aRect.y1 = pDrawItemInfo->y1;
        /* 画分割线 */
        GUI_SetColor(GUI_GRAY_E7);
        GUI_DrawLine(aRect.x0, (aRect.y1-19-16)/2, aRect.x1, (aRect.y1-19-16)/2);
        GUI_DrawLine(aRect.x0, (aRect.y1+19+16)/2, aRect.x1, (aRect.y1+19+16)/2);
        break;
    default:
        return LISTWHEEL_OwnerDraw(pDrawItemInfo);
    }
    return 0;
}
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
        FRAMEWIN_SetFont(hItem, &GUI_Font16B_ASCII);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");

        /* 初始化LISTWHEEL */
        for(int i=0; i < 3; i++)
        {
            hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_0 + i);
            LISTWHEEL_SetLineHeight(hItem, 34);
            LISTWHEEL_SetSnapPosition(hItem, (178-34)/2);
            LISTWHEEL_SetFont(hItem, GUI_FONT_20B_ASCII);
            LISTWHEEL_SetTextAlign(hItem, GUI_TA_HCENTER | GUI_TA_VCENTER);
            LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_UNSEL, 0x191919);
            LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x007dfe);
            LISTWHEEL_SetDeceleration(hItem, 35);
            LISTWHEEL_SetOwnerDraw(hItem, _OwnerDraw);
            LISTWHEEL_SetSel(hItem, 0);
        }
        /* 添加LISTWHEEL0文本项 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_0);
        for(int i=0; i<GUI_COUNTOF(_apYear); i++)
        {
            LISTWHEEL_AddString(hItem, *(_apYear+i));
        }
        /* 添加LISTWHEEL1文本项 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_1);
        for(int i=0; i<GUI_COUNTOF(_apMonth); i++)
        {
            LISTWHEEL_AddString(hItem, *(_apMonth+i));
        }
        /* 添加LISTWHEEL2文本项 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_2);
        for(int i=0; i<GUI_COUNTOF(_apDay); i++)
        {
            LISTWHEEL_AddString(hItem, *(_apDay+i));
        }
        break;
    case WM_NOTIFY_PARENT:
        /* 获取控件ID */
        Id = WM_GetId(pMsg->hWinSrc);
        /* 获取通知代码 */
        NCode = pMsg->Data.v;
        switch(Id)
        {
        case ID_LISTWHEEL_0: // Notifications sent by 'Button'
            switch(NCode)
            {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_0);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x191919);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_0);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x007dfe);
                U8 index = LISTWHEEL_GetPos(hItem);
                LISTWHEEL_SetSel(hItem, index);
                break;
            }
            break;
        case ID_LISTWHEEL_1: // Notifications sent by 'Button'
            switch(NCode)
            {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_1);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x191919);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_1);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x007dfe);
                U8 index = LISTWHEEL_GetPos(hItem);
                LISTWHEEL_SetSel(hItem, index);
                break;
            }
            break;
        case ID_LISTWHEEL_2: // Notifications sent by 'Button'
            switch(NCode)
            {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_2);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x191919);
                break;
            case WM_NOTIFICATION_RELEASED:
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTWHEEL_2);
                LISTWHEEL_SetTextColor(hItem, LISTWHEEL_CI_SEL, 0x007dfe);
                U8 index = LISTWHEEL_GetPos(hItem);
                LISTWHEEL_SetSel(hItem, index);
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
- 可以看到LCD显示列表轮
- 分别上下滑动年月日3 个控件可以选择不同的选项，每个控件的选项都是循环显示的