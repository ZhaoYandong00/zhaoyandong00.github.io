---
title: 控件STemWin篇10
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 控件
---
# 支持的控件类型

<table>
<thead>
<tr>
<th style="text-align: center" bgcolor="#80FF00" >控件名称</th>
<th style="text-align: center" bgcolor="#80FF00" > 描述</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center" >BUTTON</td>
<td style="text-align: center" >按钮，可按下。文本或位图可以显示在按钮上</td>
</tr>
<tr>
<td style="text-align: center" >CHECKBOX</td>
<td style="text-align: center" >复选框可以选中或取消选中</td>
</tr>
<tr>
<td style="text-align: center" >DROPDOWN</td>
<td style="text-align: center" >下拉列表框，按下后打开列表框</td>
</tr>
<tr>
<td style="text-align: center" >EDIT</td>
<td style="text-align: center" >单行编辑字段，提示用户键入数字或文本</td>
</tr>
<tr>
<td style="text-align: center" >FRAMEWIN</td>
<td style="text-align: center" >框架窗口， 创建典型的GUI 外观</td>
</tr>
<tr>
<td style="text-align: center" >GRAPH</td>
<td style="text-align: center" >图形控件，用于显示曲线或测量值</td>
</tr>
<tr>
<td style="text-align: center" >HEADER</td>
<td style="text-align: center" >标头控件，用于管理列</td>
</tr>
<tr>
<td style="text-align: center" >ICONVIEW</td>
<td style="text-align: center" >图标视图控件，适用于常见手持设备中的基于图标的平台</td>
</tr>
<tr>
<td style="text-align: center" >IMAGE</td>
<td style="text-align: center" >图像控件，自动显示多种图像格式</td>
</tr>
<tr>
<td style="text-align: center" >KNOB</td>
<td style="text-align: center" >旋钮控件，可用于调整不可数的值</td>
</tr>
<tr>
<td style="text-align: center" >LISTBOX</td>
<td style="text-align: center" >列表框，其中突出显示用户选择的项</td>
</tr>
<tr>
<td style="text-align: center" >LISTVIEW</td>
<td style="text-align: center" >列表视图控件用于创建表</td>
</tr>
<tr>
<td style="text-align: center" >LISTWHEEL</td>
<td style="text-align: center" >列表轮控件，可以通过指针输入设备移动和加速数据</td>
</tr>
<tr>
<td style="text-align: center" >MENU</td>
<td style="text-align: center" >菜单控件用于创建水平和垂直菜单</td>
</tr>
<tr>
<td style="text-align: center" >MULTIEDIT</td>
<td style="text-align: center" >此控件用于编辑多行文本</td>
</tr>
<tr>
<td style="text-align: center" >MULTIPAGE</td>
<td style="text-align: center" >多页控件用于创建具有多个页面的对话框</td>
</tr>
<tr>
<td style="text-align: center" >PROGBAR</td>
<td style="text-align: center" >用于可视化的进度条</td>
</tr>
<tr>
<td style="text-align: center" >RADIO</td>
<td style="text-align: center" >单选按钮可以被选择，一次只能选择一个按钮</td>
</tr>
<tr>
<td style="text-align: center" >SCROLLBAR</td>
<td style="text-align: center" >滚动条控件可以是水平或垂直的</td>
</tr>
<tr>
<td style="text-align: center" >SLIDER</td>
<td style="text-align: center" >滑块控件用于更改值</td>
</tr>
<tr>
<td style="text-align: center" >SPINBOX</td>
<td style="text-align: center" >旋转框控件显示和调整特定值</td>
</tr>
<tr>
<td style="text-align: center" >SWIPELIST</td>
<td style="text-align: center" >滑动列表控件用于创建可滑动的列表，可通过在触摸屏上滑动手指（或任何其他PID 设备）来移动滑动列表</td>
</tr>
<tr>
<td style="text-align: center" >TEXT</td>
<td style="text-align: center" >通常在对话框中使用的静态文本控件</td>
</tr>
<tr>
<td style="text-align: center" >TREEVIEW</td>
<td style="text-align: center" >用于管理分层列表的列表树控件</td>
</tr>
</tbody>
</table>


# 控件资源表结构体

```c
struct GUI_WIDGET_CREATE_INFO_struct {
  GUI_WIDGET_CREATE_FUNC * pfCreateIndirect; //指向控件创建函数的指针
  const char             * pName;            // 控件名称
  I16                      Id;               // 控件ID
  I16                      x0;               // 控件的最左侧坐标
  I16                      y0;               // 控件的最顶部坐标
  I16                      xSize;            // 控件的横向尺寸
  I16                      ySize;            // 控件的纵向尺寸
  U16                      Flags;            // 控件的创建标志，默认为0
  I32                      Para;             // 控件的参数，默认为0
  U32                      NumExtraBytes;    // 控件的额外字节
};
```

# 控件常用函数
- 是否支持所有流式位图格式`void WIDGET_EnableStreamAuto(void)`
- 返回控件的默认效果`const WIDGET_EFFECT* WIDGET_GetDefaultEffect(void)`
- 设置控件的默认效果`const WIDGET_EFFECT* WIDGET_SetDefaultEffect(const WIDGET_EFFECT* pEffect)`
- 设置给定控件的效果`void WIDGET_SetEffect(WM_HWIN hObj, const WIDGET_EFFECT* pEffect)`
- 设置接收输入焦点的功能`void WIDGET_SetFocusable(WM_HWIN hObj, int State)`
- 默认的回调函数`void WINDOW_Callback(WM_MESSAGE * pMsg)`
- 用于在对话框中自动创建控件`WM_HWIN WINDOW_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外字节作为用户数据创建控件`WM_HWIN WINDOW_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, WM_CALLBACK * cb, int NumExtraBytes);`
- 检索上个函数创建的额外用户数据`int WINDOW_GetUserData(WM_HWIN hObj, void * pDest, int NumBytes)`
- 设置控件的额外数据`int WINDOW_SetUserData(WM_HWIN hObj, const void * pSrc, int NumBytes)`
