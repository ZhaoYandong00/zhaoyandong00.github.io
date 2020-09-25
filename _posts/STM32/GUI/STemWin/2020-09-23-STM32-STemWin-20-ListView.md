---
title: 表格控件STemWin篇20
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 表格控件
---
# 表格控件API
- 添加一列数据`int LISTVIEW_AddColumn(LISTVIEW_Handle hObj, int Width, const char * s, int Align)`
- 添加一行数据`int LISTVIEW_AddRow(LISTVIEW_Handle hObj, const GUI_ConstString * ppText)`
- 比较功能，用于比较2 个整数值`int LISTVIEW_CompareDec(const void * p0, const void * p1)`
- 比较功能，用于比较2 个字符串`int LISTVIEW_CompareText(const void * p0, const void * p1)`
- 附加表格控件到窗口`LISTVIEW_Handle LISTVIEW_CreateAttached(WM_HWIN hParent, int Id, int SpecialFlags)`
- 创建一个表格控件`LISTVIEW_Handle LISTVIEW_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id)`
- 从对话框资源表中创建表格控件`LISTVIEW_Handle LISTVIEW_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外的字节作为用户数据创建表格`LISTVIEW_Handle LISTVIEW_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumExtraBytes)`
- 减量选择`void LISTVIEW_DecSel(LISTVIEW_Handle hObj)`
- 删除给定的列`void LISTVIEW_DeleteColumn(LISTVIEW_Handle hObj, unsigned Index)`
- 删除给定的行`void LISTVIEW_DeleteRow(LISTVIEW_Handle hObj, unsigned Index)`
- 将给定行的状态设置为禁用`void LISTVIEW_DisableRow(LISTVIEW_Handle hObj, unsigned Row)`
- 禁用控件的排序`void LISTVIEW_DisableSort(LISTVIEW_Handle hObj)`
- 启用/禁用给定控件的单元格选择模式`void LISTVIEW_EnableCellSelect(LISTVIEW_Handle hObj, unsigned OnOff)`
- 将给定行的状态设置为启用`void LISTVIEW_EnableRow(LISTVIEW_Handle hObj, unsigned Row)`
- 启用控件的排序`void LISTVIEW_EnableSort(LISTVIEW_Handle hObj)`
- 返回当前背景颜色`GUI_COLOR LISTVIEW_GetBkColor(LISTVIEW_Handle hObj, unsigned Index)`
- 返回当前字体`const GUI_FONT *  LISTVIEW_GetFont(LISTVIEW_Handle hObj)`
- 返回附加的HEADER 控件的句柄`HEADER_Handle LISTVIEW_GetHeader(LISTVIEW_Handle hObj)`
- 复制指定项目的矩形坐标`void LISTVIEW_GetItemRect(LISTVIEW_Handle hObj, U32 Col, U32 Row, GUI_RECT * pRect)`
- 将指定项目的文本复制到给定的缓冲区`void LISTVIEW_GetItemText(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, char * pBuffer, unsigned MaxSize)`
- 返回给定单元格的字符长度`unsigned LISTVIEW_GetItemTextLen(LISTVIEW_Handle hObj, unsigned Column, unsigned Row)`
- 将指定项目的文本复制到给定的缓冲区`void LISTVIEW_GetItemTextSorted(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, char * pBuffer, unsigned MaxSize)`
- 返回列数`unsigned LISTVIEW_GetNumColumns(LISTVIEW_Handle hObj)`
- 返回行数`unsigned LISTVIEW_GetNumRows(LISTVIEW_Handle hObj)`
- 返回所选行的索引`int LISTVIEW_GetSel(LISTVIEW_Handle hObj)`
- 返回所选列的索引`int LISTVIEW_GetSelCol(LISTVIEW_Handle hObj)`
- 返回未排序状态的选定行的索引`int LISTVIEW_GetSelUnsorted(LISTVIEW_Handle hObj)`
- 返回文本颜色`GUI_COLOR LISTVIEW_GetTextColor(LISTVIEW_Handle hObj, unsigned Index)`
- 检索用户数据集`int LISTVIEW_GetUserData(LISTVIEW_Handle hObj, void * pDest, int NumBytes)`
- 返回给定行的用户数据`U32 LISTVIEW_GetUserDataRow(LISTVIEW_Handle hObj, unsigned Row)`
- 返回当前用于换行的模式`GUI_WRAPMODE LISTVIEW_GetWrapMode(LISTVIEW_Handle hObj)`
- 增量选择`void LISTVIEW_IncSel(LISTVIEW_Handle hObj)`
- 在给定位置插入新行`int LISTVIEW_InsertRow(LISTVIEW_Handle hObj, unsigned Index, const GUI_ConstString * ppText)`
- 用于绘制控件单元格的默认函数`int LISTVIEW_OwnerDraw(const WIDGET_ITEM_DRAW_INFO * pDrawItemInfo)`
- 启用自动使用水平滚动条`void LISTVIEW_SetAutoScrollH(LISTVIEW_Handle hObj, int OnOff)`
- 启用自动使用垂直滚动条`void LISTVIEW_SetAutoScrollV(LISTVIEW_Handle hObj, int OnOff)`
- 设置背景色`void LISTVIEW_SetBkColor(LISTVIEW_Handle hObj, unsigned int Index, GUI_COLOR Color)`
- 设置列宽`void LISTVIEW_SetColumnWidth(LISTVIEW_Handle hObj, unsigned int Index, int Width)`
- 设置给定列的比较功能`void LISTVIEW_SetCompareFunc(LISTVIEW_Handle hObj, unsigned Column, int (* fpCompare)(const void * p0, const void * p1))`
- 设置HEADER 控件的默认背景颜色`GUI_COLOR LISTVIEW_SetDefaultBkColor(unsigned  Index, GUI_COLOR Color)`
- 设置HEADER 控件的默认字体`const GUI_FONT * LISTVIEW_SetDefaultFont(const GUI_FONT * pFont)`
- 设置HEADER 控件的默认颜色`GUI_COLOR LISTVIEW_SetDefaultGridColor(GUI_COLOR Color)`
- 设置HEADER 控件的网格线默认颜色`GUI_COLOR LISTVIEW_SetDefaultTextColor(unsigned  Index, GUI_COLOR Color)`
- 固定给定的列数`unsigned LISTVIEW_SetFixed(LISTVIEW_Handle hObj, unsigned Fixed)`
- 设置字体`void LISTVIEW_SetFont(LISTVIEW_Handle hObj, const GUI_FONT * pFont)`
- 设置网格线的可见性标志`int LISTVIEW_SetGridVis(LISTVIEW_Handle hObj, int Show)`
- 设置页眉的高度`void LISTVIEW_SetHeaderHeight(LISTVIEW_Handle hObj, unsigned HeaderHeight)`
- 将位图设置为控件单元格的背景`void LISTVIEW_SetItemBitmap(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, int xOff, int yOff, const GUI_BITMAP * pBitmap)`
- 设置单元格的背景色`void LISTVIEW_SetItemBkColor(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, unsigned int Index, GUI_COLOR Color)`
- 设置单元格文本`void LISTVIEW_SetItemText(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, const char * s)`
- 设置单元格文本颜色`void LISTVIEW_SetItemTextColor(LISTVIEW_Handle hObj, unsigned Column, unsigned Row, unsigned int Index, GUI_COLOR Color)`
- 设置用于左边框的像素数`void LISTVIEW_SetLBorder(LISTVIEW_Handle hObj, unsigned BorderSize)`
- 设置用于绘制单元格的自定义函数`void LISTVIEW_SetOwnerDraw(LISTVIEW_Handle hObj, WIDGET_DRAW_ITEM_FUNC * pfDrawItem)`
- 设置用于右边框的像素数`void LISTVIEW_SetRBorder(LISTVIEW_Handle hObj, unsigned BorderSize)`
- 设置控件行高`unsigned LISTVIEW_SetRowHeight(LISTVIEW_Handle hObj, unsigned RowHeight)`
- 设置当前选中的行`void LISTVIEW_SetSel(LISTVIEW_Handle hObj, int Sel)`
- 设置当前选定的列`void LISTVIEW_SetSelCol(LISTVIEW_Handle hObj, int NewCol)`
- 将当前选择设置为未排序状态`void LISTVIEW_SetSelUnsorted(LISTVIEW_Handle hObj, int Sel)`
- 设置要排序的列和排序顺序`unsigned LISTVIEW_SetSort(LISTVIEW_Handle hObj, unsigned Column, unsigned Reverse)`
- 设置列的文本对齐方式`void LISTVIEW_SetTextAlign(LISTVIEW_Handle hObj, unsigned int Index, int Align)`
- 设置文字颜色`void LISTVIEW_SetTextColor(LISTVIEW_Handle hObj, unsigned int Index, GUI_COLOR Color)`
- 设置额外用户数据集`int LISTVIEW_SetUserData(LISTVIEW_Handle hObj, const void * pSrc, int NumBytes)`
- 设置给定行的用户数据`void LISTVIEW_SetUserDataRow(LISTVIEW_Handle hObj, unsigned Row, U32 UserData)`
- 设置给定控件的换行模式`void LISTVIEW_SetWrapMode(LISTVIEW_Handle hObj, GUI_WRAPMODE WrapMode)`

# 表格控件实验
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
#define ID_FRAMEWIN_0 (GUI_ID_USER + 0x00)
#define ID_LISTVIEW_0 (GUI_ID_USER + 0x01)
#define ID_BUTTON_0 (GUI_ID_USER + 0x02)
#define ID_BUTTON_1 (GUI_ID_USER + 0x03)
#define ID_BUTTON_2 (GUI_ID_USER + 0x04)
#define ID_BUTTON_3 (GUI_ID_USER + 0x05)
#define ID_TEXT_0 (GUI_ID_USER + 0x06)
#define ID_TEXT_1 (GUI_ID_USER + 0x07)
#define ID_TEXT_2 (GUI_ID_USER + 0x08)
#define ID_TEXT_3 (GUI_ID_USER + 0x09)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
static const char * _Table[][4] = {
    { "A00", "Item AAA", "123-A", "378" },
    { "A01", "Item BBB", "123-B", "308" },
    { "A02", "Item CCC", "123-C", "344" },
    { "A03", "Item DDD", "123-D", "451" },
    { "A04", "Item EEE", "123-E", "364" },
    { "A05", "Item FFF", "123-F", "194" },
    { "A06", "Item GGG", "123-G", "774" },
    { "A07", "Item HHH", "123-H", "339" }
};

typedef struct
{
    char Col0[10];
    char Col1[10];
    char Col2[10];
    char Col3[10];
} _ListviewItem;
_ListviewItem ListviewItem;

static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { LISTVIEW_CreateIndirect, "Listview", ID_LISTVIEW_0, 5, 15, 220, 150, 0, 0x0, 0 },
    { BUTTON_CreateIndirect, "Button0", ID_BUTTON_0, 3, 175, 55, 20, 0, 0x0, 0 },
    { BUTTON_CreateIndirect, "Button1", ID_BUTTON_1, 60, 175, 55, 20, 0, 0x0, 0 },
    { BUTTON_CreateIndirect, "Button2", ID_BUTTON_2, 117, 175, 55, 20, 0, 0x0, 0 },
    { BUTTON_CreateIndirect, "Button3", ID_BUTTON_3, 174, 175, 55, 20, 0, 0x0, 0 },
    { TEXT_CreateIndirect, "Text0", ID_TEXT_0, 10, 200, 400, 16, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text1", ID_TEXT_1, 10, 220, 400, 16, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text2", ID_TEXT_2, 10, 240, 400, 16, 0, 0x64, 0 },
    { TEXT_CreateIndirect, "Text3", ID_TEXT_3, 10, 260, 400, 16, 0, 0x64, 0 },
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
    char    buf[20];
    static  U8 listview_RowIndex = 1, listview_ColumnIndex = 1;
    U8      i;
    int     Listview_RowNum;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetTitleHeight(hItem, 32);
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化Listview控件 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
        LISTVIEW_SetHeaderHeight(hItem, 20);
        LISTVIEW_AddColumn(hItem, 70, "Col 0", GUI_TA_HCENTER | GUI_TA_VCENTER);
        LISTVIEW_AddColumn(hItem, 70, "Col 1", GUI_TA_HCENTER | GUI_TA_VCENTER);
        LISTVIEW_AddColumn(hItem, 70, "Col 2", GUI_TA_HCENTER | GUI_TA_VCENTER);
        LISTVIEW_AddColumn(hItem, 70, "Col 3", GUI_TA_HCENTER | GUI_TA_VCENTER);
        for(i = 0; i < GUI_COUNTOF(_Table); i++)
        {
            LISTVIEW_AddRow(hItem, _Table[i]);
        }
        LISTVIEW_SetGridVis(hItem, 1);
        LISTVIEW_SetFont(hItem, GUI_FONT_16_ASCII);
        LISTVIEW_SetAutoScrollH(hItem, 1);
        LISTVIEW_SetAutoScrollV(hItem, 1);

        /* 初始化Button0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_BUTTON_0);
        BUTTON_SetText(hItem, "Add Row");
        BUTTON_SetFont(hItem, GUI_FONT_10_ASCII);
        /* 初始化Button1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_BUTTON_1);
        BUTTON_SetText(hItem, "Del Row");
        BUTTON_SetFont(hItem, GUI_FONT_10_ASCII);
        /* 初始化Button2 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_BUTTON_2);
        BUTTON_SetText(hItem, "Add Col");
        BUTTON_SetFont(hItem, GUI_FONT_10_ASCII);
        /* 初始化Button3 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_BUTTON_3);
        BUTTON_SetText(hItem, "Del Col");
        BUTTON_SetFont(hItem, GUI_FONT_10_ASCII);
        /* 初始化Text0 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Col 0: ");
        TEXT_SetFont(hItem, GUI_FONT_16_ASCII);
        /* 初始化Text1 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_1);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Col 1: ");
        TEXT_SetFont(hItem, GUI_FONT_16_ASCII);
        /* 初始化Text2 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_2);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Col 2: ");
        TEXT_SetFont(hItem, GUI_FONT_16_ASCII);
        /* 初始化Text3 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_3);
        TEXT_SetTextAlign(hItem, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetText(hItem, "Col 3: ");
        TEXT_SetFont(hItem, GUI_FONT_16_ASCII);
        break;
    case WM_NOTIFY_PARENT:
        Id = WM_GetId(pMsg->hWinSrc);
        NCode = pMsg->Data.v;
        switch (Id) {
        case ID_LISTVIEW_0: // Notifications sent by 'Listview'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
                Listview_RowNum = LISTVIEW_GetSel(hItem);
                LISTVIEW_GetItemText(hItem, 0, Listview_RowNum, ListviewItem.Col0, 10);
                LISTVIEW_GetItemText(hItem, 1, Listview_RowNum, ListviewItem.Col1, 10);
                LISTVIEW_GetItemText(hItem, 2, Listview_RowNum, ListviewItem.Col2, 10);
                LISTVIEW_GetItemText(hItem, 3, Listview_RowNum, ListviewItem.Col3, 10);
                break;
            case WM_NOTIFICATION_RELEASED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
                sprintf(buf, "Col 0: %s", ListviewItem.Col0);
                TEXT_SetText(hItem, buf);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_1);
                sprintf(buf, "Col 1: %s", ListviewItem.Col1);
                TEXT_SetText(hItem, buf);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_2);
                sprintf(buf, "Col 2: %s", ListviewItem.Col2);
                TEXT_SetText(hItem, buf);
                hItem = WM_GetDialogItem(pMsg->hWin, ID_TEXT_3);
                sprintf(buf, "Col 3: %s", ListviewItem.Col3);
                TEXT_SetText(hItem, buf);
                break;
            case WM_NOTIFICATION_SEL_CHANGED:
                break;
            }
            break;
        case ID_BUTTON_0: // Notifications sent by 'Add Row'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
                LISTVIEW_AddRow(hItem, _Table[7]);
                break;
            }
            break;
        case ID_BUTTON_1: // Notifications sent by 'Del Row'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
                listview_RowIndex = LISTVIEW_GetNumRows(hItem);
                if(listview_RowIndex == 1)
                {
                    break;
                }
                listview_RowIndex = listview_RowIndex - 1;
                LISTVIEW_DeleteRow(hItem, listview_RowIndex);
                break;
            }
            break;
        case ID_BUTTON_2: // Notifications sent by 'Add column'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
                listview_ColumnIndex = LISTVIEW_GetNumColumns(hItem);
                sprintf(buf, "Col %d", listview_ColumnIndex);
                LISTVIEW_AddColumn(hItem, 40, buf, GUI_TA_HCENTER | GUI_TA_VCENTER);
                break;
            }
            break;
        case ID_BUTTON_3: // Notifications sent by 'Del column'
            switch (NCode) {
            case WM_NOTIFICATION_CLICKED:
                break;
            case WM_NOTIFICATION_RELEASED:
                hItem = WM_GetDialogItem(pMsg->hWin, ID_LISTVIEW_0);
                listview_ColumnIndex = LISTVIEW_GetNumColumns(hItem);
                if(listview_ColumnIndex == 1) {
                    break;
                }
                listview_ColumnIndex = listview_ColumnIndex - 1;
                LISTVIEW_DeleteColumn(hItem, listview_ColumnIndex);
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
- 可以看到LCD显示表格
- 点击“Add Row”按钮可以向表格中增加新的一行数据，“Del Row”按钮可以删除表格的最后一行，“Add Column”按钮可以增加新的一列，“Del Column”按钮可以删除表格的最后一列