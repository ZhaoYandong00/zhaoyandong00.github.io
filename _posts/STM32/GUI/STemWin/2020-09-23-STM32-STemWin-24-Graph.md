---
title: 曲线图控件STemWin篇24
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 曲线图控件
---
# 曲线图控件API
## 通用函数
- 附加数据对象到控件`void GRAPH_AttachData(GRAPH_Handle hObj, GRAPH_DATA_Handle hData)`
- 附加刻度对象到控件`void GRAPH_AttachScale(GRAPH_Handle hObj, GRAPH_SCALE_Handle hScale)`
- 创建一个曲线图控件`GRAPH_Handle GRAPH_CreateEx(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id)`
- 从对话框资源表中创建曲线图控件`GRAPH_Handle GRAPH_CreateIndirect(const GUI_WIDGET_CREATE_INFO * pCreateInfo, WM_HWIN hWinParent, int x0, int y0, WM_CALLBACK * cb)`
- 使用额外字节作为用户数据创建曲线图控件`GRAPH_Handle GRAPH_CreateUser(int x0, int y0, int xSize, int ySize, WM_HWIN hParent, int WinFlags, int ExFlags, int Id, int NumExtraBytes)`
- 从控件中分离数据对象`void GRAPH_DetachData(GRAPH_Handle hObj, GRAPH_DATA_Handle hData)`
- 从控件中分离刻度对象`void GRAPH_DetachScale(GRAPH_Handle hObj, GRAPH_SCALE_Handle hScale)`
- 返回控件一种结构的颜色`GUI_COLOR GRAPH_GetColor(GRAPH_Handle hObj, unsigned Index)`
- 返回指定滚动条的当前值`I32 GRAPH_GetScrollValue(GRAPH_Handle hObj, U8 Coord)`
- 检索用户数据集`int GRAPH_GetUserData(GRAPH_Handle hObj, void * pDest, int NumBytes)`
- 使能自动使用滚动条`void GRAPH_SetAutoScrollbar(GRAPH_Handle hObj, U8 Coord, U8 OnOff)`
- 设置上下左右各个边框的大小`void GRAPH_SetBorder(GRAPH_Handle hObj, unsigned BorderL, unsigned BorderT, unsigned BorderR, unsigned BorderB)`
- 设置控件一种结构的颜色`GUI_COLOR GRAPH_SetColor(GRAPH_Handle hObj, GUI_COLOR Color, unsigned Index)`
- 设置水平网格间距`unsigned GRAPH_SetGridDistX(GRAPH_Handle hObj, unsigned Value)`
- 设置垂直网格间距`unsigned GRAPH_SetGridDistY(GRAPH_Handle hObj, unsigned Value)`
- 固定水平网格`unsigned GRAPH_SetGridFixedX(GRAPH_Handle hObj, unsigned OnOff)`
- 添加垂直网格线偏移量`unsigned GRAPH_SetGridOffY(GRAPH_Handle hObj, unsigned Value)`
- 启用网格绘制`unsigned GRAPH_SetGridVis(GRAPH_Handle hObj, unsigned OnOff)`
- 设置水平网格线的线型`U8 GRAPH_SetLineStyleH(GRAPH_Handle hObj, U8 Value)`
- 设置垂直网格线的线型`U8 GRAPH_SetLineStyleV(GRAPH_Handle hObj, U8 Value)`
- 设置指定滚动条的滚动值`void GRAPH_SetScrollValue(GRAPH_Handle hObj, U8 Coord, U32 Value)`
- 设置额外用户数据集`int void GRAPH_SetUserData(GRAPH_Handle hObj, const void * pSrc, int NumBytes)`
- 设置用户回调函数`void GRAPH_SetUserDraw(GRAPH_Handle hObj, void (* pOwnerDraw)(WM_HWIN, int))`
- 设置控件的水平范围`unsigned GRAPH_SetVSizeX(GRAPH_Handle hObj, unsigned Value)`
- 设置控件的垂直范围`unsigned GRAPH_SetVSizeY(GRAPH_Handle hObj, unsigned Value)`

## GRAPH_DATA_YT 相关函数
- 添加一个数据项到YT 数据对象`void GRAPH_DATA_YT_AddValue(GRAPH_DATA_Handle hDataObj, I16 Value)`
- 清除YT 数据对象的所有数据项`void GRAPH_DATA_YT_Clear(GRAPH_DATA_Handle hDataObj)`
- 创建一个YT 数据对象`GRAPH_DATA_Handle GRAPH_DATA_YT_Create(GUI_COLOR Color, unsigned MaxNumItems, const I16 * pData, unsigned NumItems)`
- 删除YT 数据对象`void GRAPH_DATA_YT_Delete(GRAPH_DATA_Handle hDataObj)`
- 返回给定索引处的数据值`int GRAPH_DATA_YT_GetValue(GRAPH_DATA_Handle hDataObj, I16 * pValue, U32 Index)`
- 镜像x 轴`void GRAPH_DATA_YT_MirrorX(GRAPH_DATA_Handle hDataObj, int OnOff)`
- 设置指定YT 数据对象的对齐方式`void GRAPH_DATA_YT_SetAlign(GRAPH_DATA_Handle hDataObj, int Align)`
- 设置用于绘制数据的垂直偏移`void GRAPH_DATA_YT_SetOffY(GRAPH_DATA_Handle hDataObj, int Off)`

## 刻度对象相关函数
- 创建一个刻度对象`GRAPH_SCALE_Handle GRAPH_SCALE_Create(int Pos, int TextAlign, unsigned Flags, unsigned TickDist)`
- 删除刻度对象`void GRAPH_SCALE_Delete(GRAPH_SCALE_Handle hScaleObj)`
- 设置用于从像素到所需单位的计算系数`float GRAPH_SCALE_SetFactor(GRAPH_SCALE_Handle hScaleObj, float Factor)`
- 设置用于绘制数字的字体`const GUI_FONT * GRAPH_SCALE_SetFont(GRAPH_SCALE_Handle hScaleObj, const GUI_FONT * pFont)`
- 设置小数部分的位数`int GRAPH_SCALE_SetNumDecs(GRAPH_SCALE_Handle hScaleObj, int NumDecs)`
- 设置添加到数字的可选偏移量`int GRAPH_SCALE_SetOff(GRAPH_SCALE_Handle hScaleObj, int Off)`

# 曲线图控件实验
## 添加库函数
- 打开 `Manage Run-Time Environment`
- 选择`Device->StdPeriph Drivers->ADC`

## 初始化ADC
- 复制SPL时ADC程序的`my_adc.c``my_adc.h`到工程
- 添加头文件包含到`main.c`

```c
#include "my_adc.h"
```
- 在板级外设初始化里添加ADC初始化

```c
/**
 * @brief  板级外设初始化，所有板子上的初始化均可放在这个函数里面
 * @param  无
 * @retval 无
 */
static void BSP_Init(void)
{
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_CRC, ENABLE);
    NVIC_PriorityGroupConfig( NVIC_PriorityGroup_4 );
    /* LED 初始化 */
    LED_GPIO_Config();
    /* 串口初始化	*/
    USART_Config();
    /* 触摸屏初始化 */
    XPT2046_Init();
    /* ADC初始化 */
    ADCx_Init();
}
```

## 曲线图控件程序
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "WM.h"
#include "DIALOG.h"
#include <stdio.h>
#include "my_adc.h"

/*********************************************************************
*
*     Defines
*
**********************************************************************
*/
#define ID_FRAMEWIN_0 (GUI_ID_USER + 0x00)
#define ID_GRAPH_0 (GUI_ID_USER + 0x01)
#define ID_TEXT_0 (GUI_ID_USER + 0x02)
/*******************************************************************
*
*     Static variables
*
********************************************************************/
GRAPH_DATA_Handle Graphdata;
TEXT_Handle text;
extern __IO uint16_t ADC_ConvertedValue[NOFCHANEL];

/*
 * 曲线图控件总共270减去上下边框剩下250像素，
 * 我们想显示0-3.3V,每刻度0.5V,最少7个刻度
 * 250/7≈35.7
 * 我们刻度间距需要是小于35.7的整数
 * 为了好计算像素和单位的比例，刻度间距*像素和单位的比例=0.5
 * 为了避免计算误差，我们选择刻度间距为32，像素和单位的比例即为0.5/32=0.015625
 * 为了刻度和网格间隙对齐，网格间隙可选择32的约数，我们这里选择32
 */
//X方向网格间隙(像素)
static const U8 gridDistX=32;
//Y方向网格间隙(像素)
static const U8 gridDistY=32;
//刻度间距(像素)，
static const U8 tickDist=32;
//设置像素和单位的比例，
static const float factor=0.015625;

static const GUI_WIDGET_CREATE_INFO _aDialogCreate[] = {
    { FRAMEWIN_CreateIndirect, "Framewin", ID_FRAMEWIN_0, 0, 0, 240, 320, 0, 0x0, 0 },
    { GRAPH_CreateIndirect, "Graph", ID_GRAPH_0, 10, 10, 220, 270, 0, 0x0, 0 },
    { TEXT_CreateIndirect, "Text0", ID_TEXT_0, 10, 10, 40, 16, 0, 0, 0 },
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
    GRAPH_SCALE_Handle hScaleV;

    switch (pMsg->MsgId) {
    case WM_INIT_DIALOG:
        /* 初始化Framewin控件 */
        hItem = pMsg->hWin;
        FRAMEWIN_SetText(hItem, "STemWIN STM32F103");
        FRAMEWIN_SetFont(hItem, GUI_FONT_16B_ASCII);
        /* 初始化Text控件 */
        text=WM_GetDialogItem(pMsg->hWin, ID_TEXT_0);
        TEXT_SetTextAlign(text, GUI_TA_LEFT | GUI_TA_VCENTER);
        TEXT_SetTextColor(text,GUI_BLUE);
        /* 初始化Graph控件 */
        hItem = WM_GetDialogItem(pMsg->hWin, ID_GRAPH_0);
        GRAPH_SetColor(hItem, GUI_WHITE, GRAPH_CI_BK);
        GRAPH_SetColor(hItem, GUI_BLACK, GRAPH_CI_GRID);
        GRAPH_SetBorder(hItem, 30, 10, 10, 10);
        GRAPH_SetGridDistX(hItem, gridDistX);
        GRAPH_SetGridDistY(hItem, gridDistY);
        GRAPH_SetLineStyleH(hItem, GUI_LS_DOT);
        GRAPH_SetLineStyleV(hItem, GUI_LS_DOT);
        GRAPH_SetGridVis(hItem, 1);
        /* 创建垂直刻度对象 */
        hScaleV = GRAPH_SCALE_Create(15, GUI_TA_HCENTER | GUI_TA_LEFT,
                                     GRAPH_SCALE_CF_VERTICAL, tickDist);
        GRAPH_AttachScale(hItem, hScaleV);
        GRAPH_SCALE_SetFactor(hScaleV, factor);
        GRAPH_SCALE_SetNumDecs(hScaleV, 2);
        /* 创建数据对象 */
        Graphdata = GRAPH_DATA_YT_Create(GUI_RED, 300, 0, 0);
        GRAPH_AttachData(hItem, Graphdata);
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
    float VREF,mul;
    float vlot;
    /* 创建窗口 */
    CreateFramewin();
    while (1)
    {
        //获取电压参考值
        VREF=1.2f*4095/ADC_ConvertedValue[2];
        //计算像素比例
        mul=VREF/factor;
        vlot=((float)ADC_ConvertedValue[0]*VREF/4095);
        /* 向GRAPH数据对象添加数据 */
        GRAPH_DATA_YT_AddValue(Graphdata, ((float)ADC_ConvertedValue[0]*mul/4095));
        TEXT_SetDec(text,vlot*100,0,2,0,0);
        GUI_Delay(10);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示电压值曲线
- 控件中的红色曲线随着电位器的转动而出现变化