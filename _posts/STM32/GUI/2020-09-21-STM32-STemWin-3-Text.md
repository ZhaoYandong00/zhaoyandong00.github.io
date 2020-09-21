---
title: 文本显示StemWin篇3
categories: STM32 emWin StemWin
tags: STM32 emWin StemWin
description: StemWin文本显示
---
# 文本显示常用函数
## 字符显示函数
- 清除当前行从当前位置到结束位置`void GUI_DispCEOL(void)`
- 在当前位置显示单个字符`void GUI_DispChar(U16 c)`
- 在指定位置显示单个字符`void GUI_DispCharAt(U16 c, I16P x, I16P y)`
- 指定次数的重复显示单个字符`void GUI_DispChars(U16 c, int Cnt)`
- 在当前位置显示一个字符串`void GUI_DispString(const char * s)`
- 在指定位置显示一个字符串`void GUI_DispStringAt(const char * s, int x, int y)`
- 在指定位置显示一个字符串，并清除当前行到行尾`void GUI_DispStringAtCEOL(const char * s, int x, int y)`
- 在指定位置显示水平居中的字符串`void GUI_DispStringHCenterAt(const char * s, int x, int y)`
- 在指定的矩形中显示字符串`void GUI_DispStringInRect(const char * s, GUI_RECT * pRect, int TextAlign)`
- 在指定矩形中显示包含在矩形中的字符串，带旋转`void GUI_DispStringInRectEx(const char * s, GUI_RECT * pRect, int TextAlign, int MaxLen, const GUI_ROTATION * pLCD_Api)`
- 在指定矩形中显示包含在矩形中的字符串，带换行模式`void GUI_DispStringInRectWrap(const char * s, GUI_RECT * pRect, int TextAlign, GUI_WRAPMODE WrapMode)`
- 在指定矩形中显示包含在矩形中的字符串，带换行和旋转`void GUI_DispStringinRectWrapEx(const char * s, GUI_RECT * pRect, int TextAlign, GUI_WRAPMODE WrapMode, const GUI_ROTATION * pLCD_Api)` 
- 在当前位置显示具有指定字符数的字符串`void GUI_DispStringLen(const char * s, int Len)`
- 返回在指定换行模式下字符串所需的行数`int GUI_WrapGetNumLines(const char * pText, int xSize, GUI_WRAPMODE WrapMode)`

## 绘制模式
- 返回当前的文本绘制模式`int GUI_GetTextMode(void)`
- 设置文本绘制模式`int GUI_SetTextMode(int Mode)`
- 设置文本显示样式`char GUI_SetTextStyle(char Style)`

## 对齐方式
- 返回当前的文本对齐方式`int GUI_GetTextAlign(void)`
- 设置换行后的左边框大小`int GUI_SetLBorder(int x)`
- 设置文本对齐方式`int GUI_SetTextAlign(int Align)`

## 坐标设置
- 将光标移动到下一行的开头`void GUI_DispNextLine(void)` 
- 设置X 坐标`char GUI_GotoX(int x)` 
- 设置X 和Y 坐标`char GUI_GotoXY(int x, int y)` 
- 设置Y 坐标`char GUI_GotoY(int y)` 
- 返回当前X 坐标`int GUI_GetDispPosX(void)` 
- 返回当前Y 坐标`int GUI_GetDispPosY(void)`

# 文本显示
- 修改`MainTask.c`

```c
#include "GUI.h"

/*******************************************************************************
 * 全局变量
 ******************************************************************************/
char acText[] = "This example demostrates text wrapping";
GUI_RECT rect = {10, 150, 75, 270};
GUI_WRAPMODE aWm[] = {GUI_WRAPMODE_NONE, GUI_WRAPMODE_CHAR, GUI_WRAPMODE_WORD};
/*******************************************************************************
 * 函数
 ******************************************************************************/
/**
  * @brief GUI主任务
  * @param 无
  * @retval 无
  */
void MainTask(void)
{
  U8 i;
	
	/* 设置背景色 */
	GUI_SetBkColor(GUI_BLUE);	
	GUI_Clear();
	
	/* 设置字体大小 */
	GUI_SetFont(GUI_FONT_16_1);
	GUI_DispStringAt("STemWIN STM32F103", 10, 10);
	
	/* 画线 */
	GUI_SetPenSize(8);
	GUI_SetColor(GUI_RED);
	GUI_DrawLine(10, 40, 230, 120);
	GUI_DrawLine(10, 120, 230, 40);
	
	/* 绘制文本 */
	GUI_SetBkColor(GUI_BLACK);
	GUI_SetColor(GUI_WHITE);
	GUI_SetFont(GUI_FONT_16B_ASCII);
	/* 正常模式 */
	GUI_SetTextMode(GUI_TM_NORMAL);
	GUI_DispStringHCenterAt("GUI_TM_NORMAL" , 120, 40);
	/* 反转显示 */
	GUI_SetTextMode(GUI_TM_REV);
	GUI_DispStringHCenterAt("GUI_TM_REV" , 120, 40 + 16);
	/* 透明文本 */
	GUI_SetTextMode(GUI_TM_TRANS);
	GUI_DispStringHCenterAt("GUI_TM_TRANS" , 120, 40 + 16 * 2);
	/* 异或文本 */
	GUI_SetTextMode(GUI_TM_XOR);
	GUI_DispStringHCenterAt("GUI_TM_XOR" , 120, 40 + 16 * 3);
	/* 透明反转文本 */
	GUI_SetTextMode(GUI_TM_TRANS | GUI_TM_REV);
	GUI_DispStringHCenterAt("GUI_TM_TRANS | GUI_TM_REV", 120, 40 + 16 * 4);
	
	/* 在矩形区域内显示文本 */
	GUI_SetFont(GUI_FONT_16B_ASCII);
	GUI_SetTextMode(GUI_TM_TRANS);
	for(i = 0;i < 3;i++)
	{
		GUI_SetColor(GUI_WHITE);
		GUI_FillRectEx(&rect);
		GUI_SetColor(GUI_RED);
		GUI_DispStringInRectWrap(acText, &rect, GUI_TA_LEFT, aWm[i]);
		rect.x0 += 75;
		rect.x1 += 75;
	}
  while(1)
  {
    GUI_Delay(100);
  }
}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示

