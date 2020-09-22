---
title: 窗口管理器STemWin篇9
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 窗口管理器
---
# 窗口管理器解析
- 窗口在形状上是矩形的，由它们的原点(左上角的X 和y 坐标)以及它们的X 和y 大小(分别为宽度和高度)来定义

## 窗口特性
- 是矩形的
- 有一个Z 位置
- 可能隐藏或显示
- 可能包含有效和/或无效区域
- 可能有也可能没有透明度
- 可能有也可能没有回调函数

## 窗口管理器术语
- 活动窗口：当前用于绘图操作的窗口称为活动窗口。它不一定就是最上层的窗口。
- 回调函数：回调函数由用户程序定义，指示图形系统在发生特定事件时调用特定函数。通常，当窗口的内容发生更改时，它们用于自动重绘窗口。
- 子窗口/父窗口，同属窗口：子窗口是相对于其他窗口（称为父窗口）定义的。只要父窗口移动，其子窗口就会相应移动。子窗口始终完全包含在其父窗口中，并在必要时会被裁剪。具有相同父窗口的多个子窗口被视为同属窗口。
- 客户区：客户区就是窗口的的可用区域。如果一个窗口包含一个框架或标题栏，那么客户端区域就是矩形的内部区域。如果没有这样的框架，则客户端区域的坐标与窗口本身的坐标相
同。
- 裁剪，裁剪区域：裁剪是将输出限制为窗口或窗口的一部分的过程。窗口的剪辑区域是其可见区域，是窗口区域减去被更高 Z 轴阶层的同属窗口遮挡的区域，然后减去没有放入父窗口可见区域
的任何部分。
- 坐标：坐标通常是二维坐标，以像素为单位表示。一个坐标由两个值组成。第一个值指定水平分量(也称为x 坐标)，第二个值指定垂直分量(也称为y 坐标)。
- 桌面坐标：桌面坐标是桌面窗口的坐标。屏幕的左上角位置(原点)是(0,0)。
- 桌面窗口：也叫背景窗口，是由窗口管理器自动创建的，并且总是覆盖整个显示区域。在多图层情况下，每一个图层都有自己的桌面窗口。桌面始终是最底层的窗口，当没有定义其他窗口时，它是默认的(活动的)窗口。所有窗口都是当前选定层的桌面窗口的后代(子窗口、孙窗口等)。
- 前期裁剪/后期裁剪：前期裁剪是默认的裁剪模式。在此模式下，裁剪动作在窗口接收绘制事件之前执行。如果需要裁剪当前窗口，它将在单个绘图过程中接收多个WM_PAINT 消息。在后期裁剪模式下，窗口始终只接收一条WM_PAINT 消息，此时裁剪动作在绘图操作中执行。
- 句柄：创建新窗口时，窗口管理器会为其分配一个名为句柄的唯一标识符。句柄用于在该特定窗口上执行的任何进一步操作。
- 隐藏/显示窗口：一个隐藏的窗口是不可见的，尽管它仍然存在(有一个句柄)。创建窗口时，如果没有指定创建标志，则默认情况下它是隐藏的。显示窗口使其可见，隐藏窗口则使其不可见。
- 父坐标：父坐标是相对于父窗口的窗口坐标。窗口的左上角位置（原点）是（0,0）。
- 窗口坐标：窗口坐标是窗口的坐标。 窗口的左上位置（原点）是（0,0）。
- 透明度：具有透明度的窗口包含不随窗口其余部分重新绘制的区域。这些区域的运作方式就像“透过”它们背后的窗口一样。在这种情况下，重要的是要在窗口之前以透明的方式重新绘制后面的窗口。窗口管理器自动按照正确的顺序处理重绘。
- 有效化/无效化：一个有效的窗口是一个完全更新的窗口，它不需要重新绘制。无效窗口尚未反映所有更新，因此需要全部或部分重新绘制。当发生影响特定窗口的更改时，窗口管理器将该窗口标记为无效。下一次重新绘制窗口(手动或通过回调例程)时，将验证它。
- Z 轴位置，底部/顶部：虽然窗口以X 和Y 的形式显示在二维屏幕上，但窗口管理器还可管理Z-位置(深度坐标)即虚拟三维中的一个位置，它决定了窗口从背景到前景的位置。因此，窗口可以出现在彼此的底部或顶部。将一个窗口设置为底部将把它“放在”它所有的同属窗口(如果有的话)下面；将它设置为顶部将会将它“置于”它的同属窗口之上。创建窗口时，如果没有指定创建标志，则默认将其设置为顶部。

## 消息结构体

```c
struct WM_MESSAGE {
  int MsgId;            /* 消息类型 */
  WM_HWIN hWin;         /* 目标窗口 */
  WM_HWIN hWinSrc;      /* 源窗口  */
  union {
    const void * p;     /* 消息特定数据指针 */
    int v;
    GUI_COLOR Color;
    void (* pFunc)(void);
  } Data;
};
```

## 消息类型

<table>
<thead>
<tr>
<th style="text-align: center" bgcolor="#80FF00" >消息ID(MsgId)</th>
<th style="text-align: center" bgcolor="#80FF00" > 描述(MsgId)</th>
</tr>
</thead>
<tbody>
<tr>
<th style="text-align: center" bgcolor="#80FFFF" colspan="2">系统定义消息</th>
</tr>
<tr>
<td style="text-align: center" >WM_CREATE</td>
<td style="text-align: center">创建窗口后立即发送，使窗口有机会初始化和创建任何子窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_DELETE</td>
<td style="text-align: center">在窗口被删除之前发送，告诉窗口释放它的数据结构(如果有的话)</td>
</tr>
<tr>
<td style="text-align: center">WM_GET_ACCEPT_FOCUS</td>
<td style="text-align: center">发送到窗口，以确定窗口是否能够接收输入焦点</td>
</tr>
<tr>
<td style="text-align: center">WM_GET_ID</td>
<td style="text-align: center">发送到窗口以请求窗口的ID</td>
</tr>
<tr>
<td style="text-align: center">WM_INIT_DIALOG</td>
<td style="text-align: center">在对话框创建后立即发送到对话框窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_KEY</td>
<td style="text-align: center">如果按下一个键，则发送到当前包含焦点的窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_MOVE</td>
<td style="text-align: center">被移动后立即发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFY_PARENT</td>
<td style="text-align: center">通知父窗口，其中一个子窗口中发生了某些事件</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFY_VIS_CHANGED</td>
<td style="text-align: center">如果窗口的可见性已更改，则发送到该窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_PAINT</td>
<td style="text-align: center">在窗口无效后发送到窗口，应执行重绘</td>
</tr>
<tr>
<td style="text-align: center">WM_POST_PAINT</td>
<td style="text-align: center">在最后一个WM_PAINT 消息被处理后发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_PRE_PAINT</td>
<td style="text-align: center">在发送第一个WM_PAINT 消息之前发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_SET_FOCUS</td>
<td style="text-align: center">如果窗口获得或失去输入焦点，则将其发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_SET_ID</td> 
<td style="text-align: center">发送到窗口以更改窗口 ID</td>
</tr>
<tr>
<td style="text-align: center">WM_SIZE</td> 
<td style="text-align: center">在窗口大小更改后发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_TIMER</td> 
<td style="text-align: center">定时器到期后发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_USER_DATA</td>
<td style="text-align: center">设置用户数据后发送到窗口</td>
</tr>
<tr>
<th style="text-align: center" colspan="2" bgcolor="#80FFFF">指针输入设备(PID)消息</th>
</tr>
<tr>
<td style="text-align: center">WM_MOTION</td>
<td style="text-align: center">发送到窗口以实现高级运动支持</td>
</tr>
<tr>
<td style="text-align: center">WM_MOUSEOVER</td>
<td style="text-align: center">如果指针输入设备接触窗口的边缘，则发送到窗口。只有在启用鼠标支持时才发送</td>
</tr>
<tr>
<td style="text-align: center">WM_MOUSEOVER_END</td>
<td style="text-align: center">如果指针输入设备已移出窗口边缘，则发送到窗口。 仅在启用鼠标支持时才发送</td>
</tr>
<tr>
<td style="text-align: center">WM_PID_STATE_CHANGED</td>
<td style="text-align: center">当按下状态被更改时，发送到指针输入设备所指向的窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_TOUCH</td>
<td style="text-align: center">一旦指针输入设备在其区域上按下、按下并移动或释放，将发送到窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_TOUCH_CHILD</td>
<td style="text-align: center">如果指针输入设备触摸了子窗口，则将其发送到父窗口</td>
</tr>
<tr>
<th style="text-align: center" colspan="2" bgcolor="#80FFFF">通知代码</th>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_CHILD_DELETED</td>
<td style="text-align: center">此通知消息将在窗口删除之前从窗口发送到其父窗口</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_CLICKED</td>
<td style="text-align: center">单击窗口时将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_GOT_FOCUS</td>
<td style="text-align: center">当窗口接收并接受焦点时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_LOST_FOCUS</td>
<td style="text-align: center">当窗口失去焦点时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_MOVED_OUT</td>
<td style="text-align: center">当指针在单击并移出窗口时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_RELEASED</td>
<td style="text-align: center">当被单击的控件被释放时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_SCROLL_CHANGED</td>
<td style="text-align: center">当滚动条控件的滚动位置发生更改时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_SCROLLBAR_ADDED</td>
<td style="text-align: center">当将滚动条控件添加到窗口时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_SEL_CHANGED</td>
<td style="text-align: center">当窗口控件的选择发生更改时，将发送此通知消息</td>
</tr>
<tr>
<td style="text-align: center">WM_NOTIFICATION_VALUE_CHANGED</td>
<td style="text-align: center">当窗口控件的特定值发生更改时，将发送此通知消息</td>
</tr>
<tr>
<th style="text-align: center" colspan="2" bgcolor="#80FFFF">用户自定义消息</th>
</tr>
<tr>
<td style="text-align: center">WM_USER</td>
<td style="text-align: center">应用程序可以使用WM_USER 常量定义私有消息，通常形式是(WM_USER + X)，其中X 是一个整数值</td>
</tr>
</tbody>
</table>

## 窗口管理器常用函数
### 基本函数
- 将给定的消息发送到所有现有窗口`int WM_BroadcastMessage(WM_MESSAGE * pMsg)`
- 将指定窗口放置于同属窗口的底部`void WM_BringToBottom(WM_HWIN hWin)`
- 将指定窗口放置于同属窗口的定部`void WM_BringToTop(WM_HWIN hWin)`
- 创建一个窗口`WM_HWIN WM_CreateWindow(int x0, int y0, int xSize, int ySize, U32 Style, WM_CALLBACK * cb, int NumExtraBytes)`
- 创建一个子窗口`WM_HWIN WM_CreateWindowAsChild(int x0, int y0, int xSize, int ySize, WM_HWIN hWinParent, U32 Style, WM_CALLBACK* cb, int NumExtraBytes)`
- 处理消息的默认函数`void WM_DefaultProc(WM_MESSAGE * pMsg)`
- 删除指定窗口`void WM_DeleteWindow(WM_HWIN hWin)`
- 将窗口状态设置为启用(默认)`void WM_EnableWindow(WM_HWIN hWin)`
- 通过执行回调重绘所有无效窗口`int WM_Exec(void)`
- 返回活动窗口的句柄`WM_HWIN WM_GetActiveWindow(void)`
- 返回指向窗口的回调函数指针`WM_CALLBACK * GetCallback(WM_HWIN hWin)`
- 返回活动窗口的大小`void WM_GetClientRect(GUI_RECT * pRect)`
- 返回桌面窗口的窗口句柄`WM_HWIN WM_GetDesktopWindow(void)`
- 返回对话框或窗口控件的窗口句柄`WM_HWIN WM_GetDialogItem(WM_HWIN hWin, int Id)`
- 返回具有输入焦点的窗口句柄`WM_HWIN WM_GetFocusedWindow(void)`
- 返回活动窗口的X 原点`int WM_GetOrgX(void)`
- 返回活动窗口的Y 原点`int WM_GetOrgY(void)`
- 返回父窗口句柄`WM_HWIN WM_GetParent(WM_HWIN hWin)`
- 返回活动窗口的屏幕坐标`void WM_GetWindowRect(GUI_RECT * pRect)`
- 返回窗口的水平大小(宽度)`int WM_GetWindowSizeX(WM_HWIN hWin)`
- 返回窗口的垂直大小(高度)`int WM_GetWindowSizeY(WM_HWIN hWin)`
- 隐藏指定窗口`void WM_HideWindow(WM_HWIN hWin)`
- 使整个窗口无效`void WM_InvalidateWindow(WM_HWIN hWin)`
- 在桌面坐标中设置窗口的位置`void WM_MoveTo(WM_HWIN hWin, int x, int y)`
- 将窗口移动到另一个位置`void WM_MoveWindow(WM_HWIN hWin, int dx, int dy)`
- 立即绘制或重绘窗口`void WM_Paint(WM_HWIN hObj)`
- 选择用于绘图操作的窗口`WM_HWIN WM_SelectWindow(WM_HWIN  hWin)`
- 将消息发送到窗口`void WM_SendMessage(WM_HWIN hWin, WM_MESSAGE * p)`
- 设置窗口的回调函数`WM_CALLBACK * WM_SetCallback(WM_HWIN hWin, WM_CALLBACK * cb)`
- 设置创建新窗口时默认使用的标志`U32 WM_SetCreateFlags(U32 Flags)`
- 设置桌面窗口颜色`GUI_COLOR WM_SetDesktopColor(GUI_COLOR Color)`
- 将输入焦点设置为指定的窗口`int WM_SetFocus(WM_HWIN hWin)`
- 设置窗口的新大小`void WM_SetSize(WM_HWIN hWin, int XSize, int YSize)`
- 暂时减少裁剪区域`const GUI_RECT * WM_SetUserClipRect(const GUI_RECT * pRect)`
- 使窗口可见`void WM_ShowWindow(WM_HWIN hWin)`

### 多帧缓冲支持
- 启用或禁用窗口管理器自动使用多个缓冲区`int WM_MULTIBUF_Enable(int OnOff)`

### 内存设备支持
- 禁用内存设备进行重绘`void WM_DisableMemdev(WM_HWIN hWin)`
- 使用内存设备进行重绘`void WM_EnableMemdev(WM_HWIN hWin)`

### 定时器相关
- 创建一个计时器，它向窗口发送一个WM_TIMER 消息`WM_HMEM WM_CreateTimer(WM_HWIN hWin, int UserID, int Period, int Mode)`
- 删除一个定时器`void WM_DeleteTimer(WM_HMEM hTimer)`
- 获取给定计时器的ID`int WM_GetTimerId(WM_HTIMER hTimer)`
- 重启定时器`void WM_RestartTimer(WM_HMEM hTimer, int Period)`

### 控件相关函数
- 返回客户区窗口的句柄`WM_HWIN WM_GetClientWindow(WM_HWIN hObj)`
- 返回控件ID`int WM_GetId(WM_HWIN hWin)`
- 返回活动窗口减去边框后的大小`void WM_GetInsideRect(GUI_RECT * pRect)`

# 窗口重绘实验
## 重绘程序
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"

/*******************************************************************************
 * 全局变量
 ******************************************************************************/

/*******************************************************************************
 *      static code
 ******************************************************************************/
/**
  * @brief 背景窗口回调函数
  * @note pMsg：消息指针
  * @param 无
  * @retval 无
  */
static void _cbBkWindow(WM_MESSAGE* pMsg)
{
    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        GUI_ClearRect(0, 50, 319, 239);
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口回调函数
  * @note pMsg：消息指针
  * @param 无
  * @retval 无
  */
static void _cbWindow(WM_MESSAGE* pMsg)
{
    GUI_RECT Rect;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        /* 返回窗口客户区坐标 */
        WM_GetInsideRect(&Rect);
        /* 设置窗口背景颜色 */
        GUI_SetBkColor(GUI_RED);
        /* 设置前景颜色 */
        GUI_SetColor(GUI_YELLOW);
        /* 绘制窗口 */
        GUI_ClearRectEx(&Rect);
        GUI_DrawRectEx(&Rect);
        /* 设置文本颜色 */
        GUI_SetColor(GUI_BLACK);
        /* 设置文本格式 */
        GUI_SetFont(&GUI_Font8x16);
        /* 显示提示信息 */
        GUI_DispStringHCenterAt("Foreground window", 75, 40);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口移动函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _MoveWindow(const char* pText)
{
    WM_HWIN hWnd;
    int     i;

    /* 创建前景窗口 */
    hWnd = WM_CreateWindow(10, 50, 150, 100, WM_CF_SHOW, _cbWindow, 0);
    GUI_Delay(500);
    /* 移动前景窗口 */
    for (i = 0; i < 40; i++)
    {
        WM_MoveWindow(hWnd, 2, 2);
        GUI_Delay(10);
    }
    /* 移动结束后显示提示文字 */
    if (pText)
    {
        GUI_DispStringAt(pText, 5, 50);
        GUI_Delay(2500);
    }
    /* 删除前景窗口 */
    WM_DeleteWindow(hWnd);
    WM_Invalidate(WM_HBKWIN);
    GUI_Exec();
}

/**
  * @brief 窗口重绘DEMO
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoRedraw(void)
{
    WM_CALLBACK * _cbOldBk;

    GUI_SetBkColor(GUI_BLACK);
    GUI_Clear();
    GUI_SetColor(GUI_WHITE);
    GUI_SetFont(&GUI_Font20_ASCII);
    GUI_DispStringHCenterAt("WM_Redraw - Sample", 120, 5);
    GUI_SetFont(&GUI_Font8x16);
    while(1)
    {
        /* 在背景上移动窗口 */
        _MoveWindow("Background has not been redrawn");
        /* 清除背景 */
        GUI_ClearRect(0, 50, 319, 239);
        GUI_Delay(1000);
        /* 重定向背景窗口的回调函数 */
        _cbOldBk = WM_SetCallback(WM_HBKWIN, _cbBkWindow);
        /* 在背景上移动窗口 */
        _MoveWindow("Background has been redrawn");
        /* 还原背景窗口的回调函数 */
        WM_SetCallback(WM_HBKWIN, _cbOldBk);
    }
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
    _DemoRedraw();
}

```

## 编译调试
- 编译并下载到开发板
- 可以看到不执行桌面窗口重绘的时候，窗口移动会产生重影，而使用了桌面窗口重绘则没有重影

# 窗口管理器
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include <string.h>
/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
/* 自定义消息ID */
#define MSG_CHANGE_TEXT (WM_USER + 0)
/* 移动速度 */
#define SPEED           1500

/*******************************************************************
*
*     Static variables
*
********************************************************************/
/* 文本缓冲去 */
static char _acInfoText[40];

/* 颜色 */
static GUI_COLOR _WindowColor1 = GUI_GREEN;
static GUI_COLOR _FrameColor1  = GUI_BLUE;
static GUI_COLOR _WindowColor2 = GUI_RED;
static GUI_COLOR _FrameColor2  = GUI_YELLOW;
static GUI_COLOR _ChildColor   = GUI_YELLOW;
static GUI_COLOR _ChildFrame   = GUI_BLACK;

/* 回调函数 */
static WM_CALLBACK * _cbBkWindowOld;

/* 句柄*/
static WM_HWIN _hWindow1;
static WM_HWIN _hWindow2;
static WM_HWIN _hChild;
/*******************************************************************************
 *      static code
 ******************************************************************************/
/**
  * @brief 将自定义消息发送到后台窗口并使其无效，因此后台窗口的回调显示新文本。
  * @note 无
  * @param 无
  * @retval 无
  */
static void _ChangeInfoText(char * pStr)
{
    WM_MESSAGE Message;

    Message.MsgId  = MSG_CHANGE_TEXT;
    Message.Data.p = pStr;
    WM_SendMessage(WM_HBKWIN, &Message);
    WM_InvalidateWindow(WM_HBKWIN);
}

/**
  * @brief 直接在显示屏上绘制信息文本
  * @note 此功能适用于未设置回调函数的时候
  * @param 无
  * @retval 无
  */
static void _DrawInfoText(char * pStr)
{
    GUI_SetColor(GUI_WHITE);
    GUI_SetFont(&GUI_Font16_ASCII);
    GUI_DispStringHCenterAt("WindowManager - Sample", 120, 5);
    GUI_SetFont(&GUI_Font8x16);
    GUI_DispStringAtCEOL(pStr, 5, 40);
}
/*******************************************************************
*
*       _LiftUp
*/
static void _LiftUp(int dy)
{
    int i;
    int tm;

    for (i = 0; i < (dy/4); i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1, 0, -4);
        WM_MoveWindow(_hWindow2, 0, -4);
        while ((GUI_GetTime() - tm) < 20)
        {
            WM_Exec();
        }
    }
}

/*******************************************************************
*
*       _LiftDown
*/
static void _LiftDown(int dy)
{
    int i;
    int tm;

    for (i = 0; i < (dy/4); i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1, 0, 4);
        WM_MoveWindow(_hWindow2, 0, 4);
        while ((GUI_GetTime() - tm) < 20)
        {
            WM_Exec();
        }
    }
}
/*******************************************************************
*
*       Static code, callbacks for windows
*
********************************************************************
*/
/**
  * @brief 背景窗口回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbBkWindow(WM_MESSAGE * pMsg)
{
    switch (pMsg->MsgId)
    {
    case MSG_CHANGE_TEXT:
        strcpy(_acInfoText, (char const *)pMsg->Data.p);
    case WM_PAINT:
        GUI_SetBkColor(GUI_BLACK);
        GUI_Clear();
        GUI_SetColor(GUI_WHITE);
        GUI_SetFont(&GUI_Font20_ASCII);
        GUI_DispStringHCenterAt("WindowManager - Sample", 120, 5);
        GUI_SetFont(&GUI_Font8x16);
        GUI_DispStringAt(_acInfoText, 5, 40);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口1回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbWindow1(WM_MESSAGE * pMsg)
{
    GUI_RECT Rect;
    int      x;
    int      y;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        WM_GetInsideRect(&Rect);
        GUI_SetBkColor(_WindowColor1);
        GUI_SetColor(_FrameColor1);
        GUI_ClearRectEx(&Rect);
        GUI_DrawRectEx(&Rect);
        GUI_SetColor(GUI_WHITE);
        GUI_SetFont(&GUI_Font24_ASCII);
        x = WM_GetWindowSizeX(pMsg->hWin);
        y = WM_GetWindowSizeY(pMsg->hWin);
        GUI_DispStringHCenterAt("Window 1", x / 2, (y / 2) - 12);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口2回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbWindow2(WM_MESSAGE * pMsg)
{
    GUI_RECT Rect;
    int      x;
    int      y;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        WM_GetInsideRect(&Rect);
        GUI_SetBkColor(_WindowColor2);
        GUI_SetColor(_FrameColor2);
        GUI_ClearRectEx(&Rect);
        GUI_DrawRectEx(&Rect);
        GUI_SetColor(GUI_WHITE);
        GUI_SetFont(&GUI_Font24_ASCII);
        x = WM_GetWindowSizeX(pMsg->hWin);
        y = WM_GetWindowSizeY(pMsg->hWin);
        GUI_DispStringHCenterAt("Window 2", x / 2, (y / 4) - 12);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 子窗口回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbChild(WM_MESSAGE * pMsg)
{
    GUI_RECT Rect;
    int      x;
    int      y;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        WM_GetInsideRect(&Rect);
        GUI_SetBkColor(_ChildColor);
        GUI_SetColor(_ChildFrame);
        GUI_ClearRectEx(&Rect);
        GUI_DrawRectEx(&Rect);
        GUI_SetColor(GUI_RED);
        GUI_SetFont(&GUI_Font24_ASCII);
        x = WM_GetWindowSizeX(pMsg->hWin);
        y = WM_GetWindowSizeY(pMsg->hWin);
        GUI_DispStringHCenterAt("Child window", x / 2, (y / 2) - 12);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口1的另一个回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbDemoCallback1(WM_MESSAGE * pMsg)
{
    int x;
    int y;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        GUI_SetBkColor(GUI_GREEN);
        GUI_Clear();
        GUI_SetColor(GUI_RED);
        GUI_SetFont(&GUI_FontComic18B_1);
        x = WM_GetWindowSizeX(pMsg->hWin);
        y = WM_GetWindowSizeY(pMsg->hWin);
        GUI_DispStringHCenterAt("Window 1\nanother Callback", x / 2, (y / 2) - 18);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/**
  * @brief 窗口2的另一个回调函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _cbDemoCallback2(WM_MESSAGE * pMsg)
{
    int x;
    int y;

    switch (pMsg->MsgId)
    {
    case WM_PAINT:
        GUI_SetBkColor(GUI_MAGENTA);
        GUI_Clear();
        GUI_SetColor(GUI_YELLOW);
        GUI_SetFont(&GUI_FontComic18B_1);
        x = WM_GetWindowSizeX(pMsg->hWin);
        y = WM_GetWindowSizeY(pMsg->hWin);
        GUI_DispStringHCenterAt("Window 2\nanother Callback", x / 2, (y / 4) - 18);
        break;
    default:
        WM_DefaultProc(pMsg);
    }
}

/*******************************************************************
*
*       Static code, functions for demo
*
********************************************************************
*/
/**
  * @brief 演示WM_SetDesktopColor的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoSetDesktopColor(void)
{
    GUI_SetBkColor(GUI_BLUE);
    GUI_Clear();
    _DrawInfoText("WM_SetDesktopColor()");
    GUI_Delay(SPEED*3/2);
    WM_SetDesktopColor(GUI_BLACK);
    GUI_Delay(SPEED/2);
    /* 设置背景颜色并使桌面颜色无效，这是后续重绘演示所必需的 */
    GUI_SetBkColor(GUI_BLACK);
    WM_SetDesktopColor(GUI_INVALID_COLOR);
}

/**
  * @brief 演示WM_CreateWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoCreateWindow(void)
{
    /* 重定向背景窗口回调函数 */
    _cbBkWindowOld = WM_SetCallback(WM_HBKWIN, _cbBkWindow);
    /* 创建窗口 */
    _ChangeInfoText("WM_CreateWindow()");
    GUI_Delay(SPEED);
    _hWindow1 = WM_CreateWindow( 50,  70, 120, 70, WM_CF_SHOW | WM_CF_MEMDEV, _cbWindow1, 0);
    GUI_Delay(SPEED/3);
    _hWindow2 = WM_CreateWindow(105, 125, 120, 70, WM_CF_SHOW | WM_CF_MEMDEV, _cbWindow2, 0);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_CreateWindowAsChild的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoCreateWindowAsChild(void)
{
    /* 创建子窗口 */
    _ChangeInfoText("WM_CreateWindowAsChild()");
    GUI_Delay(SPEED);
    _hChild = WM_CreateWindowAsChild(10, 30, 100, 40, _hWindow2, WM_CF_SHOW | WM_CF_MEMDEV, _cbChild, 0);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_InvalidateWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoInvalidateWindow(void)
{
    _ChangeInfoText("WM_InvalidateWindow()");
    _WindowColor1 = GUI_BLUE;
    _FrameColor1  = GUI_GREEN;
    GUI_Delay(SPEED);
    WM_InvalidateWindow(_hWindow1);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_BringToTop的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoBringToTop(void)
{
    _ChangeInfoText("WM_BringToTop()");
    GUI_Delay(SPEED);
    WM_BringToTop(_hWindow1);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_MoveTo的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoMoveTo(void)
{
    int i;
    int tm;
    int tDiff;

    _ChangeInfoText("WM_MoveTo()");
    GUI_Delay(SPEED);
    for (i = 1; i < 56; i++)
    {
        tm = GUI_GetTime();
        WM_MoveTo(_hWindow1,  50 + i,  70 + i);
        WM_MoveTo(_hWindow2, 105 - i, 125 - i);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 1; i < 56; i++)
    {
        tm = GUI_GetTime();
        WM_MoveTo(_hWindow1, 105 - i, 125 - i);
        WM_MoveTo(_hWindow2,  50 + i,  70 + i);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_BringToBottom的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoBringToBottom(void)
{
    _ChangeInfoText("WM_BringToBottom()");
    GUI_Delay(SPEED);
    WM_BringToBottom(_hWindow1);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_MoveWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoMoveWindow(void)
{
    int i;
    int tm;
    int tDiff;

    _ChangeInfoText("WM_MoveWindow()");
    GUI_Delay(SPEED);
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1,  1,  1);
        WM_MoveWindow(_hWindow2, -1, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1, -1, -1);
        WM_MoveWindow(_hWindow2,  1,  1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    GUI_Delay(SPEED);
}

/**
  * @brief 演示在父窗口中WM_HideWindow和WM_ShowWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoHideShowParent(void)
{
    _ChangeInfoText("WM_HideWindow(Parent)");
    GUI_Delay(SPEED);
    WM_HideWindow(_hWindow2);
    GUI_Delay(SPEED/3);
    WM_HideWindow(_hWindow1);
    GUI_Delay(SPEED);
    _ChangeInfoText("WM_ShowWindow(Parent)");
    GUI_Delay(SPEED);
    WM_ShowWindow(_hWindow1);
    GUI_Delay(SPEED/3);
    WM_ShowWindow(_hWindow2);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示在子窗口中WM_HideWindow和WM_ShowWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoHideShowChild(void)
{
    _ChangeInfoText("WM_HideWindow(Child)");
    GUI_Delay(SPEED);
    WM_HideWindow(_hChild);
    GUI_Delay(SPEED);
    _ChangeInfoText("WM_ShowWindow(Child)");
    GUI_Delay(SPEED);
    WM_ShowWindow(_hChild);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示父边界的裁剪
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoClipping(void)
{
    int i;
    int tm;
    int tDiff;

    _ChangeInfoText("Demonstrating clipping of child");
    GUI_Delay(SPEED);
    for (i = 0; i < 25; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hChild,  1,  0);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 25; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hChild,  0,  1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 50; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hChild, -1,  0);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 25; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hChild,  0, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 25; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hChild,  1,  0);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    GUI_Delay(SPEED);
}

/**
  * @brief 演示回调函数的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoRedrawing(void)
{
    int i;
    int tm;
    int tDiff;

    _ChangeInfoText("Demonstrating redrawing");
    GUI_Delay(SPEED);
    _LiftUp(40);
    GUI_Delay(SPEED/3);
    _ChangeInfoText("Using a callback for redrawing");
    GUI_Delay(SPEED/3);
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1,  1,  1);
        WM_MoveWindow(_hWindow2, -1, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1, -1, -1);
        WM_MoveWindow(_hWindow2,  1,  1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    GUI_Delay(SPEED/4);
    _LiftDown(30);
    GUI_Delay(SPEED/2);
    _ChangeInfoText("Without redrawing");
    GUI_Delay(SPEED);
    _LiftUp(30);
    GUI_Delay(SPEED/4);
    WM_SetCallback(WM_HBKWIN, _cbBkWindowOld);
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1,  1,  1);
        WM_MoveWindow(_hWindow2, -1, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 55; i++)
    {
        tm = GUI_GetTime();
        WM_MoveWindow(_hWindow1, -1, -1);
        WM_MoveWindow(_hWindow2,  1,  1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    GUI_Delay(SPEED/3);
    WM_SetCallback(WM_HBKWIN, _cbBkWindow);
    _LiftDown(40);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_ResizeWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoResizeWindow(void)
{
    int i;
    int tm;
    int tDiff;

    _ChangeInfoText("WM_ResizeWindow()");
    GUI_Delay(SPEED);
    _LiftUp(30);
    for (i = 0; i < 20; i++)
    {
        tm = GUI_GetTime();
        WM_ResizeWindow(_hWindow1,  1,  1);
        WM_ResizeWindow(_hWindow2, -1, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 40; i++)
    {
        tm = GUI_GetTime();
        WM_ResizeWindow(_hWindow1, -1, -1);
        WM_ResizeWindow(_hWindow2,  1,  1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    for (i = 0; i < 20; i++)
    {
        tm = GUI_GetTime();
        WM_ResizeWindow(_hWindow1,  1,  1);
        WM_ResizeWindow(_hWindow2, -1, -1);
        tDiff = 15 - (GUI_GetTime() - tm);
        GUI_Delay(tDiff);
    }
    _LiftDown(30);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_SetCallback的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoSetCallback(void)
{
    _ChangeInfoText("WM_SetCallback()");
    GUI_Delay(SPEED);
    WM_SetCallback(_hWindow1, _cbDemoCallback1);
    WM_InvalidateWindow(_hWindow1);
    GUI_Delay(SPEED/2);
    WM_SetCallback(_hWindow2, _cbDemoCallback2);
    WM_InvalidateWindow(_hWindow2);
    GUI_Delay(SPEED*3);
    WM_SetCallback(_hWindow1, _cbWindow1);
    WM_InvalidateWindow(_hWindow1);
    GUI_Delay(SPEED/2);
    WM_SetCallback(_hWindow2, _cbWindow2);
    WM_InvalidateWindow(_hWindow2);
    GUI_Delay(SPEED);
}

/**
  * @brief 演示WM_DeleteWindow的使用
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoDeleteWindow(void)
{
    _ChangeInfoText("WM_DeleteWindow()");
    GUI_Delay(SPEED);
    WM_DeleteWindow(_hWindow2);
    GUI_Delay(SPEED/3);
    WM_DeleteWindow(_hWindow1);
    GUI_Delay(SPEED);
    _ChangeInfoText("");
    GUI_Delay(SPEED);
    /* 还原背景窗口回调函数和颜色 */
    WM_SetCallback(WM_HBKWIN, _cbBkWindowOld);
    _WindowColor1 = GUI_GREEN;
    _WindowColor2 = GUI_RED;
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
    GUI_SetBkColor(GUI_BLACK);
    WM_SetCreateFlags(WM_CF_MEMDEV);
    WM_EnableMemdev(WM_HBKWIN);
    while (1)
    {
        _DemoSetDesktopColor();
        _DemoCreateWindow();
        _DemoCreateWindowAsChild();
        _DemoInvalidateWindow();
        _DemoBringToTop();
        _DemoMoveTo();
        _DemoBringToBottom();
        _DemoMoveWindow();
        _DemoHideShowParent();
        _DemoHideShowChild();
        _DemoClipping();
        _DemoRedrawing();
        _DemoResizeWindow();
        _DemoSetCallback();
        _DemoDeleteWindow();
    }
}

```
## 编译调试
- 编译并下载到开发板
- LCD显示运行结果