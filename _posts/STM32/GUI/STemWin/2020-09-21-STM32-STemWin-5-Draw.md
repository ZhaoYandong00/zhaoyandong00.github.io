---
title: 绘图STemWin篇5
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin绘图
---
# 绘图常用函数
## 绘制相关功能
- 调整矩形大小`void GUI_AddRect(GUI_RECT * pDest, const GUI_RECT * pRect, int Dist)`
- 返回当前可用的绘图区域`void GUI_GetClientRect(GUI_RECT * pRect)`
- 返回当前的绘图模式`GUI_DRAWMODE GUI_GetDrawMode(void)`
- 返回当前的画笔大小`U8 GUI_GetPenSize(void)`
- 返回指定位置的颜色索引`unsigned GUI_GetPixelIndex(int x, int y)`
- 设置绘图模式`GUI_DRAWMODE GUI_SetDrawMode(GUI_DRAWMODE dm)`
- 设置画笔大小`U8 GUI_SetPenSize(U8 Size)`

## 基本绘图函数
- 为一个矩形区域填充背景颜色`void GUI_ClearRect(int x0, int y0, int x1, int y1)`
- 复制屏幕中的一个矩形区域`void GUI_CopyRect(int x0, int y0, int x1, int y1, int dx, int dy)`
- 绘制使用水平渐变颜色填充的矩形`void GUI_DrawGradientH(int x0, int y0, int x1, int y1, GUI_COLOR Color0, GUI_COLOR Color1)`
- 绘制使用垂直渐变颜色填充的矩形`void GUI_DrawGradientV(int x0, int y0, int x1, int y1, GUI_COLOR Color0, GUI_COLOR Color1)`
- 绘制使用水平渐变颜色填充的圆角矩形`void GUI_DrawGradientRoundedH(int x0, int y0, int x1, int y1, int rd, GUI_COLOR Color0, GUI_COLOR Color1)`
- 绘制使用垂直渐变颜色填充的圆角矩形`void GUI_DrawGradientRoundedV(int x0, int y0, int x1, int y1, int rd, GUI_COLOR Color0, GUI_COLOR Color1)`
- 绘制像素点`void GUI_DrawPixel(int x, int y)`
- 绘制矩形`void GUI_DrawRect(int x0, int y0, int x1, int y1)`
- 绘制矩形`void GUI_DrawRectEx(const GUI_RECT * pRect)`
- 绘制圆角矩形框`void GUI_DrawRoundedFrame(int x0, int y0, int x1, int y1, int r, int w)`
- 绘制圆角矩形`void GUI_DrawRoundedRect(int x0, int y0, int x1, int y1, int r)`
- 绘制填充颜色的矩形`void GUI_FillRect(int x0, int y0, int x1, int y1)`
- 绘制填充颜色的矩形`void GUI_FillRectEx(const GUI_RECT * pRect)`
- 绘制填充颜色的圆角矩形`void GUI_FillRoundedRect(int x0, int y0, int x1, int y1, int r)`

## Alpha 混合
- 开启/禁止自动Alpha 混合`void GUI_EnableAlpha(void)`
- 在绘制完成后保留Alpha 通道`unsigned GUI_PreserveTrans(unsigned OnOff)`
- 恢复Alpha 混合到之前的状态`U32 GUI_RestoreUserAlpha(GUI_ALPHA_STATE * pAlphaState)`
- 设置当前的Alpha 混合值`unsigned GUI_SetAlpha(U8 Alpha)`
- 设置一个附加值，用于计算实际的Alpha 混合值`U32 GUI_SetUserAlpha(GUI_ALPHA_STATE * pAlphaState, U32 UserAlpha)`

## 绘制线
- 绘制水平线`void GUI_DrawHLine(int y0, int x0, int x1)`
- 绘制从起始坐标到终点坐标的线（绝对坐标）`void GUI_DrawLine(int x0, int y0, int x1, int y1)`
- 绘制从当前位置到由X 和Y 距离指定终点的直线（相对坐标）`void GUI_DrawLineRel(int dx, int dy)`
- 绘制从当前位置到指定终点的直线`void GUI_DrawLineTo(int x, int y)`
- 绘制折线`void GUI_DrawPolyLine(const GUI_POINT * pPoints, int NumPoints, int x0, int y0)`
- 绘制垂直线`void GUI_DrawVLine(int x0, int y0, int y1)`
- 返回当前直线绘制样式`U8 GUI_GetLineStyle(void)`
- 以当前位置为参考移动直线`void GUI_MoveRel(int dx, int dy)`
- 将直线指针移动到指定位置`void GUI_MoveTo(int x, int y)`
- 设置直线绘制样式`U8 GUI_SetLineStyle(U8 Style)`

## 绘制多边形
- 绘制多边形的轮廓`void GUI_DrawPolygon(const GUI_POINT * pPoints, int NumPoints, int x0, int y0)`
- 放大多边形`void GUI_EnlargePolygon(GUI_POINT * pDest, const GUI_POINT * pSrc, int NumPoints, int Len)`
- 绘制具有颜色填充的多边形`void GUI_FillPolygon(const GUI_POINT * pPoints, int NumPoints, int x0, int y0)`
- 放大多边形`void GUI_MagnifyPolygon(GUI_POINT * pDest, const GUI_POINT * pSrc, int NumPoints, int Mag)`
- 按照指定角度旋转多边形`void GUI_RotatePolygon(GUI_POINT * pDest, const GUI_POINT * pSrc, int NumPoints, float Angle)`

## 绘制圆
- 绘制圆的轮廓`void GUI_DrawCircle(int x0, int y0, int r)`
- 绘制具有颜色填充的圆`void GUI_FillCircle(int x0, int y0, int r)`

## 绘制椭圆
- 绘制椭圆的轮廓`void GUI_DrawEllipse(int x0, int y0, int rx, int ry)`
- 绘制具有颜色填充的椭圆`void GUI_FillEllipse(int x0, int y0, int rx, int ry)`

## 绘制圆弧
- 绘制指定角度的圆弧`void GUI_DrawArc(int x0, int y0, int rx, int ry, int a0, int a1)`

## 绘制线图
- 绘制线图`void GUI_DrawGraph(I16 * pay, int NumPoints, int x0, int y0)`

## 绘制二维码
- 创建二维码位图`GUI_HMEM GUI_QR_Create(const char * pText, int PixelSize, int EccLevel, int Version)`
- 删除二维码位图`void GUI_QR_Delete(GUI_HMEM hQR)`
- 绘制二维码位图`void GUI_QR_Draw(GUI_HMEM hQR, int xPos, int yPos)`
- 返回包含有关二维码信息的结构体`void GUI_QR_GetInfo(GUI_HMEM hQR, GUI_QR_INFO * pInfo)`

## 绘制饼图
- 绘制一个扇形`void GUI_DrawPie(int x0, int y0, int r, int a0, int a1, int Type)`

# 绘图
- 修改`MainTask.c`

```c
#include "GUI.h"

#include <stdlib.h>
/*******************************************************************************
 * 全局变量
 ******************************************************************************/
GUI_RECT BasicRect = {10, 10, 50, 50};
static const unsigned aValues[] = {100, 135, 190, 240, 340, 360};
static const GUI_COLOR aColor[] = {GUI_BLUE, GUI_GREEN, GUI_RED,
                                   GUI_CYAN, GUI_MAGENTA, GUI_YELLOW};
static const char QR_TEXT[] = "http://www.st.com";
static const GUI_POINT _aPointArrow[] = {
  {  0,   0 },
  {-20, -15 },
  {-5 , -10 },
  {-5 , -35 },
  { 5 , -35 },
  { 5 , -10 },
  { 20, -15 },
};
static const GUI_POINT DashCube_BackPoint[] = {
		{ 76 /2, 104/2 },
		{ 176/2, 104/2 },
		{ 176/2,   4  /2},
		{  76/2,   4  /2}
};
static const GUI_POINT DashCube_LeftPoint[] = {
		{ 40/2, 140 /2},
		{ 76/2, 104 /2},
		{ 76/2,   4 /2},
		{ 40/2,  40 /2}
};
static const GUI_POINT DashCube_BottonPoint[] = {
		{  40/2, 140 /2},
		{ 140/2, 140 /2},
		{ 176/2, 104 /2},
		{  76/2, 104 /2}
};
static const GUI_POINT DashCube_TopPoint[] = {
		{  40/2, 40 /2},
		{ 140/2, 40 /2},
		{ 176/2,  4 /2},
		{  76/2,  4 /2},
};
static const GUI_POINT DashCube_RightPoint[] = {
		{ 140/2, 140/2 },
		{ 176/2, 104/2 },
		{ 176/2,   4/2 },
		{ 140/2,  40/2 },
};
static const 	GUI_POINT DashCube_FrontPoint[] = {
		{  40/2, 140/2},
		{ 140/2, 140/2},
		{ 140/2,  40/2},
		{  40/2,  40/2},
};
/*******************************************************************************
 * 函数
 ******************************************************************************/
/**
  * @brief 饼图绘图函数
  * @note 无
  * @param x0：饼图圆心的x坐标
  *        y0：饼图圆心的y坐标
  *        r：饼图半径
  * @retval 无
  */
static void Pie_Chart_Drawing(int x0, int y0, int r)
{
	int i, a0 = 0, a1 = 0;
	
	for(i = 0; i < GUI_COUNTOF(aValues); i++)
	{
		if(i == 0) a0 = 0;
		else a0 = aValues[i - 1];
		a1 = aValues[i];	
		GUI_SetColor(aColor[i]);
		GUI_DrawPie(x0, y0, r, a0, a1, 0);
	}
}

/**
  * @brief 二维码生成
  * @note 无
  * @param pText：二维码内容
  *        PixelSize：二维码数据色块的大小，单位：像素
  *        EccLevel：纠错编码级别
  *        x0：二维码图像在LCD的坐标x
  *        y0：二维码图像在LCD的坐标y
  * @retval 无
  */
static void QR_Code_Drawing(const char *pText, int PixelSize, int EccLevel, int x0, int y0)
{
	GUI_HMEM hQR;
	
	/* 创建二维码对象 */
	hQR = GUI_QR_Create(pText, PixelSize, EccLevel, 0);
	/* 绘制二维码到LCD */
	GUI_QR_Draw(hQR, x0, y0);
	/* 删除二维码对象 */
	GUI_QR_Delete(hQR);
}

/**
  * @brief 2D绘图函数
  * @note 无
  * @param 无
  * @retval 无
  */
/* 用于存放多边形旋转后的点列表 */ 
GUI_POINT aArrowRotatedPoints[GUI_COUNTOF(_aPointArrow)];
static void _2D_Graph_Drawing(void)
{
	I16 aY[90] = {0};
	int i;
  float pi = 3.1415926L;
  float angle = 0.0f;
	
	/* 绘制各种矩形 */
	GUI_SetColor(GUI_GREEN);
	GUI_DrawRectEx(&BasicRect);
	BasicRect.x0 += 45;
	BasicRect.x1 += 45;
	GUI_FillRectEx(&BasicRect);
  GUI_SetColor(GUI_RED);
	GUI_DrawRoundedRect(100, 10, 140, 50, 5);
	GUI_DrawRoundedFrame(145, 10, 185, 50, 5, 5);
	GUI_FillRoundedRect(190, 10, 230, 50, 5);
	GUI_DrawGradientRoundedH(10, 55, 50, 95, 5, GUI_LIGHTMAGENTA, GUI_LIGHTCYAN);
	GUI_DrawGradientRoundedV(55, 55, 95, 95, 5, GUI_LIGHTMAGENTA, GUI_LIGHTCYAN);
	
	/* 绘制线条 */
	GUI_SetPenSize(5);
  GUI_SetColor(GUI_YELLOW);
	GUI_DrawLine(100, 55, 180, 95);
	
	/* 绘制多边形 */
	GUI_SetColor(GUI_RED);
	GUI_FillPolygon(_aPointArrow, 7, 210, 100);
  /* 旋转多边形 */
	angle = pi / 2;
	GUI_RotatePolygon(aArrowRotatedPoints,
	                  _aPointArrow, 
                    (sizeof(_aPointArrow) / sizeof(_aPointArrow[0])),
										angle);
	GUI_FillPolygon(&aArrowRotatedPoints[0], 7, 230, 120);
  
  /* 绘制线框正方体 */
  GUI_SetPenSize(1);
	GUI_SetColor(0x4a51cc);
	GUI_SetLineStyle(GUI_LS_DOT);
	GUI_DrawPolygon(DashCube_BackPoint, 4, 0, 105);
  GUI_DrawPolygon(DashCube_LeftPoint, 4, 0, 105);
  GUI_DrawPolygon(DashCube_BottonPoint, 4, 0, 105);
  GUI_SetPenSize(2);
  GUI_SetLineStyle(GUI_LS_SOLID);
  GUI_DrawPolygon(DashCube_TopPoint, 4, 0, 105);
  GUI_DrawPolygon(DashCube_RightPoint, 4, 0, 105);
  GUI_DrawPolygon(DashCube_FrontPoint, 4, 0, 105);
                    
	/* 绘制圆 */
	GUI_SetColor(GUI_LIGHTMAGENTA);
	for(i = 10; i <= 40; i += 10)
	{
		GUI_DrawCircle(140, 135, i);
	}
	GUI_SetColor(GUI_LIGHTCYAN);
	GUI_FillCircle(210, 155, 20);
	
	/* 绘制椭圆 */
	GUI_SetColor(GUI_BLUE);
	GUI_FillEllipse(30, 210, 20, 30);
	GUI_SetPenSize(2);
	GUI_SetColor(GUI_WHITE);
	GUI_DrawEllipse(30, 210, 20, 10);
	
	/* 绘制圆弧 */
	GUI_SetPenSize(4);
	GUI_SetColor(GUI_GRAY_3F);
	GUI_DrawArc(100, 215, 30, 30, -30, 210);
	
	/* 绘制折线图 */
	for(i = 0; i< GUI_COUNTOF(aY); i++)
	{
		aY[i] = rand() % 50;
	}
	GUI_SetColor(GUI_BLACK);
	GUI_DrawGraph(aY, GUI_COUNTOF(aY), 140, 180);
	
	/* 绘制饼图 */
	Pie_Chart_Drawing(60, 280, 35);
	
	/* 绘制二维码 */
	QR_Code_Drawing(QR_TEXT, 3, GUI_QR_ECLEVEL_L, 140, 240);
}

/**
  * @brief Alpha混合
  * @note 无
  * @param 无
  * @retval 无
  */
static void Alpha_Blending(void)
{
  /* 显示字符 */
	GUI_SetColor(GUI_BLACK);
	GUI_SetTextMode(GUI_TM_TRANS);
	GUI_SetFont(GUI_FONT_13B_ASCII);
	GUI_DispStringHCenterAt("Alpha blending", 55, 275);
	GUI_DispStringHCenterAt("Not Used", 185, 275);
  /* 开启自动Alpha混合 */
  GUI_EnableAlpha(1);
	/* 将Alpha数值添加到颜色中并显示 */
	GUI_SetColor((0xC0uL << 24) | 0xFF0000);
	GUI_FillRect(10, 10, 100, 90);
	GUI_SetColor((0x80uL << 24) | 0x00FF00);
	GUI_FillRect(10, 100, 100, 180);
	GUI_SetColor((0x40uL << 24) | 0x0000FF);
	GUI_FillRect(10, 190, 100, 270);
  /* 关闭自动Alpha混合 */
  GUI_EnableAlpha(0);
	GUI_SetColor((0xC0uL << 24) | 0xFF0000);
	GUI_FillRect(140, 10, 230, 90);
	GUI_SetColor((0x80uL << 24) | 0x00FF00);
	GUI_FillRect(140, 100, 230, 180);
	GUI_SetColor((0x40uL << 24) | 0x0000FF);
	GUI_FillRect(140, 190, 230, 270);
}

/**
  * @brief GUI主任务
  * @param 无
  * @retval 无
  */
void MainTask(void)
{
  /* 设置背景色 */
	GUI_SetBkColor(GUI_BLUE);
	GUI_Clear();
	
/* 2D绘图 */
	_2D_Graph_Drawing();
	
	GUI_Delay(5000);
	GUI_Clear();

	/* Alpha混合 */
	Alpha_Blending();
  while(1)
  {
    GUI_Delay(100);
  }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示