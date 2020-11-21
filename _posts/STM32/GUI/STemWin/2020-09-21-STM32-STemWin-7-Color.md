---
title: 颜色STemWin篇7
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 颜色
---
# 颜色
- 色彩深度(color depth)，也称为位深度(bit depth)，表示图像中存储一个像素的色彩信息所占用的位数，单位是位/像素(bits per pixel)或bpp。通常说的什么8 位、24
位图像，这个多少位指的就是色彩深度。色彩深度越大，则单个像素包含的色彩信息越多，图像整体的颜色就越丰富。常见的色彩深度有1bpp、4bpp、8bpp、16bpp、24bpp 和32bpp。
- 像素格式(pixel format)，它表示为一个像素的颜色信息以什么样的方式和顺序进行存储。例如RGB565 表示用16 位的色彩深度存储单个像素的颜色信息，从高位到低位依次存放红绿蓝三色，其中红色和蓝色占5 位，绿色占6 位。同一种色彩深度可以对应不同的像素格式。还是刚才的RGB565 像素格式，这次我交换红色和蓝色的存放顺序，就变成了另一种像素格式BGR565，但色彩深度还是16 位。

## 颜色常用函数
### 基本函数
- 返回当前背景颜色`GUI_COLOR GUI_GetBkColor(void)`
- 返回当前背景颜色的索引`int GUI_GetBkColorIndex(void)`
- 返回当前前景色`GUI_COLOR GUI_GetColor(void)`
- 返回当前前景色的索引`int GUI_GetColorIndex(void)`
- 返回默认的前景色`GUI_COLOR GUI_GetDefaultColor(void)`
- 返回默认背景颜色`GUI_COLOR GUI_GetDefaultBkColor(void)`
- 设置当前背景颜色`void GUI_SetBkColor(GUI_COLOR)`
- 设置当前背景颜色的索引`void GUI_SetBkColorIndex(int Index)`
- 设置当前前景色`void GUI_SetColor(GUI_COLOR)`
- 设置当前前景色的索引`void GUI_SetColorIndex(int Index)`
- 设置默认前景色`void GUI_SetDefaultColor(GUI_COLOR)`
- 设置默认背景颜色`void GUI_SetDefaultBkColor(GUI_COLOR)`

### 颜色转换函数
- 返回2 种颜色之间的差异`U32 GUI_CalcColorDist(GUI_COLOR Color0, GUI_COLOR  Color1)`
- 返回与下一个可用颜色的差异`U32 GUI_CalcVisColorError(GUI_COLOR color)`
- 将颜色转换为颜色索引`int GUI_Color2Index(GUI_COLOR color)`
- 返回最近的可用颜色`GUI_COLOR GUI_Color2VisColor(GUI_COLOR color)`
- 检查是否有可用的颜色`char GUI_ColorIsAvailable(GUI_COLOR color)`
- 将颜色索引转换为颜色`GUI_COLOR GUI_Index2Color(int Index)`

# 颜色显示
- 修改`MainTask.c`

```c
#include "GUI.h"

/*******************************************************************************
 * 全局变量
 ******************************************************************************/
/* 起始坐标 */
#define X_START 60
#define Y_START 5
typedef struct {
    int NumBars;

    GUI_COLOR Color;
    const char * s;
} BAR_DATA;

static const BAR_DATA _aBarData[] = {
    { 2, GUI_RED    , "Red" },
    { 2, GUI_GREEN  , "Green" },
    { 2, GUI_BLUE   , "Blue" },
    { 1, GUI_WHITE  , "Grey" },
    { 2, GUI_YELLOW , "Yellow" },
    { 2, GUI_CYAN   , "Cyan" },
    { 2, GUI_MAGENTA, "Magenta" },
};

static const GUI_COLOR _aColorStart[] = { GUI_BLACK, GUI_WHITE };
/*******************************************************************************
 * 函数
 ******************************************************************************/
/**
  * @brief 色条显示函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoShowColorBar(void)
{
    GUI_RECT Rect;
    int      yStep;
    int      i;
    int      j;
    int      xSize;
    int      ySize;
    int      NumBars;
    int      NumColors;

    xSize = LCD_GetXSize();
    ySize = LCD_GetYSize();

    /* 可以显示的色条数 */
    NumColors = GUI_COUNTOF(_aBarData);
    for (i = NumBars = 0, NumBars = 0; i < NumColors; i++)
    {
        NumBars += _aBarData[i].NumBars;
    }
    yStep = (ySize - Y_START) / NumBars;

    /* 显示文本 */
    Rect.x0 = 0;
    Rect.x1 = X_START - 1;
    Rect.y0 = Y_START;
    GUI_SetFont(&GUI_Font16B_ASCII);
    for (i = 0; i < NumColors; i++)
    {
        Rect.y1 = Rect.y0 + yStep * _aBarData[i].NumBars - 1;
        GUI_DispStringInRect(_aBarData[i].s, &Rect, GUI_TA_LEFT | GUI_TA_VCENTER);
        Rect.y0 = Rect.y1 + 1;
    }

    /* 绘制色条 */
    Rect.x0 = X_START;
    Rect.x1 = xSize - 1;
    Rect.y0 = Y_START;
    for (i = 0; i < NumColors; i++)
    {
        for (j = 0; j < _aBarData[i].NumBars; j++)
        {
            Rect.y1 = Rect.y0 + yStep - 1;
            GUI_DrawGradientH(Rect.x0, Rect.y0, Rect.x1, Rect.y1, _aColorStart[j], _aBarData[i].Color);
            Rect.y0 = Rect.y1 + 1;
        }
    }
}

/**
  * @brief GUI主任务
  * @param 无
  * @retval 无
  */
void MainTask(void)
{
    /* 设置背景色 */
    GUI_SetBkColor(GUI_BLACK);
    GUI_Clear();

    /* 设置前景色、字体大小 */
    GUI_SetColor(GUI_RED);
    GUI_SetFont(&GUI_Font24B_ASCII);

    /* 显示色条 */
    _DemoShowColorBar();
    while(1)
    {
        GUI_Delay(100);
    }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示