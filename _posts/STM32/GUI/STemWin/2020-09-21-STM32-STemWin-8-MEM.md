---
title: 内存设备STemWin篇8
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 内存设备
---
# 内存要求
## 不支持透明度的内存设备

|内存设备的色彩深度|系统色彩深度<br/>(LCD_BITSPERPIXEL)|内存使用情况|
|:----:|:----:|:----:|
|1 bpp|1 bpp|1 字节/8 像素：<br/>(XSIZE + 7) / 8 * YSIZE|
|8 bpp|2bpp、4bpp、8 bpp|XSIZE * YSIZE|
|16 bpp|12bpp、16bpp|2 字节/像素：<br/>XSIZE * YSIZE * 2|
|32 bpp|18bpp、24bpp、32bpp|4 字节/像素：<br/>XSIZE * YSIZE * 4|

## 支持透明度的内存设备

|内存设备的色彩深度|系统色彩深度<br/>(LCD_BITSPERPIXEL)|内存使用情况|
|:----:|:----:|:----:|
|1 bpp|1 bpp|2 字节/8 像素：<br/>(XSIZE + 7) / 8 * YSIZE * 2|
|8 bpp|2bpp、4bpp、8 bpp|2 字节/像素 + 1 字节/8 像素：<br/>(XSIZE + (XSIZE + 7) / 8) *YSIZE|
|16 bpp|12bpp、16bpp|2 字节/像素 + 1 字节/8 像素：<br/>(XSIZE * 2 + (XSIZE + 7) / 8)* YSIZE|
|32 bpp|18bpp、24bpp、32bpp|4 字节/像素：<br/>XSIZE * YSIZE * 4|

# 内存常用函数
- 将内存设备内容标记为未更改`void GUI_MEMDEV_Clear(GUI_MEMDEV_Handle hMem)`
- 清除给定内存设备中的所有alpha 值`int GUI_MEMDEV_ClearAlpha(GUI_MEMDEV_Handle hMemData, GUI_MEMDEV_Handle hMemMask)`
- 读取屏幕内容并将其存储在指定内存设备中`void GUI_MEMDEV_CopyFromLCD(GUI_MEMDEV_Handle hMem)`
- 将内存设备的内容复制到LCD`void GUI_MEMDEV_CopyToLCD(GUI_MEMDEV_Handle hMem)`
- 复制抗锯齿内存设备的内容到LCD`void GUI_MEMDEV_CopyToLCDAA(GUI_MEMDEV_Handle hMem)`
- 将内存设备的内容复制到LCD 的指定位置`void GUI_MEMDEV_CopyToLCDAt(GUI_MEMDEV_Handle hMem, int x, int y)`
- 创建内存设备`GUI_MEMDEV_Handle GUI_MEMDEV_Create(int x0, int y0, int xSize, int ySize)`
- 使用附加的创建标志创建内存设备`GUI_MEMDEV_Handle GUI_MEMDEV_CreateEx(int x0, int y0, int xSize, int ySize, int Flags)`
- 创建具有给定色彩深度的内存设备`GUI_MEMDEV_Handle GUI_MEMDEV_CreateFixed(int x0, int y0, int xSize, int ySize, int Flags,const GUI_DEVICE_API * pDeviceAPI,const LCD_API_COLOR_CONV * pColorConvAPI)`
- 创建一个32bpp 的内存设备`GUI_MEMDEV_Handle GUI_MEMDEV_CreateFixed32(int x0, int y0, int xSize, int ySize)`
- 删除内存设备并释放使用的内存`int GUI_MEMDEV_Delete(GUI_MEMDEV_Handle hMem)`
- 返回指向数据区域的指针，用于直接操作`void* GUI_MEMDEV_GetDataPtr(GUI_MEMDEV_Handle hMem)`
- 返回内存设备的X-size(宽度)`int GUI_MEMDEV_GetXSize(GUI_MEMDEV_Handle hMem)`
- 返回内存设备的y 大小(高度)`int GUI_MEMDEV_GetYSize(GUI_MEMDEV_Handle hMem)`
- 将矩形区域标记为脏区域`void GUI_MEMDEV_MarkDirty(GUI_MEMDEV_Handle hMem, int x0, int y0, int x1, int y1)`
- 从存储设备中冲出给定的形状`int GUI_MEMDEV_PunchOutDevice(GUI_MEMDEV_Handle hMemData, GUI_MEMDEV_Handle hMemMask)`
- 减少内存设备的y 尺寸`void GUI_MEMDEV_ReduceYSize(GUI_MEMDEV_Handle hMem, int YSize)`
- 选择一个内存设备作为绘图操作的目标`GUI_MEMDEV GUI_MEMDEV_Select(GUI_MEMDEV_Handle hMem)`
- 从给定的内存设备创建BMP 文件`void GUI_MEMDEV_SerializeBMP(GUI_MEMDEV_Handle hDev, GUI_CALLBACK_VOID_U8_P * pfSerialize, void * p)`
- 改变LCD 上存储设备的原点`void GUI_MEMDEV_SetOrg(GUI_MEMDEV_Handle hMem, int x0, int y0)`
- 将内存设备的内容写入当前选定设备，带alpha 通道`void GUI_MEMDEV_Write(GUI_MEMDEV_Handle hMem)`
- 将内存设备的内容写入当前选定的设备，带alpha 通道和附加alpha 值`void GUI_MEMDEV_WriteAlpha(GUI_MEMDEV_Handle hMem, int Alpha)`
- 将内存设备的内容写入当前选定设备的指定位置，带alpha 通道和附加alpha 值`void GUI_MEMDEV_WriteAlphaAt(GUI_MEMDEV_Handle hMem, int Alpha, int x, int y)`
- 将内存设备的内容写入当前选定设备的指定位置，带alpha 通道`void GUI_MEMDEV_WriteAt(GUI_MEMDEV_Handle hMem, int x, int y)`
- 将内存设备的内容写入当前选择的设备，带附加alpha值和缩放`void GUI_MEMDEV_WriteEx(GUI_MEMDEV_Handle hMem, int xMag, int yMag, int Alpha)`
- 将内存设备的内容写入当前选择设备的指定位置，带附加alpha 值和缩放`void GUI_MEMDEV_WriteExAt(GUI_MEMDEV_Handle hMem, int x, int y, int xMag, int yMag, int Alpha)`
- 将内存设备的内容写入当前选定的设备，不带alpha 通道`void GUI_MEMDEV_WriteOpaque(GUI_MEMDEV_Handle hMem)`
- 将内存设备的内容写入当前选定设备的指定位置，不带alpha 通道`void GUI_MEMDEV_WriteOpaqueAt(GUI_MEMDEV_Handle hMem, int x, int y)`
- 选择LCD 作为绘图操作的目标`void GUI_SelectLCD(void)`

# 内存设备
- 修改`MainTask.c`

```c
#include "GUI.h"
#include "stdio.h"
/*******************************************************************************
 * 全局变量
 ******************************************************************************/

/*******************************************************************************
 * 函数
 ******************************************************************************/
/**
  * @brief 绘图函数
  * @note 无
  * @param 
  * @retval 无
  */
static void _Draw(int x0, int y0, int x1, int y1, int i)
{
  char buf[] = {0};

  /* 绘制矩形背景 */
  GUI_SetColor(GUI_BLUE);
	GUI_FillRect(x0, y0, x1, y1);
  
  /* 绘制文本 */
	GUI_SetFont(GUI_FONT_D64);
  GUI_SetTextMode(GUI_TEXTMODE_XOR);
  sprintf(buf, "%d", i);
	GUI_DispStringHCenterAt(buf, x0 + (x1 - x0)/2, (y0 + (y1 - y0)/2) - 32);
}

/**
  * @brief 内存设备演示函数
  * @note 无
  * @param 无
  * @retval 无
  */
static void _DemoMemDev(void)
{
  GUI_MEMDEV_Handle hMem = 0;
	int i = 0;
  
	/* 设置背景色 */
  GUI_SetBkColor(GUI_BLACK);
  GUI_Clear();
  
	/* 显示提示文字 */
  GUI_SetColor(GUI_WHITE);
  GUI_SetFont(GUI_FONT_20_ASCII);
  GUI_DispStringHCenterAt("MEMDEV_MemDev - Sample", 120, 5);
  GUI_SetFont(GUI_FONT_16_ASCII);
  GUI_DispStringHCenterAt("Shows the advantage of using a\nmemorydevice", 120, 45);
  GUI_SetFont(GUI_FONT_13_ASCII);
  GUI_DispStringHCenterAt("Draws the number\nwithout a\nmemory device", 65, 220);
  GUI_DispStringHCenterAt("Draws the number\nusing a\nmemory device", 175, 220);
  
  /* 创建内存设备 */
  hMem = GUI_MEMDEV_Create(125, 100, 100, 100);
  
	while (1)
	{
    /* 直接绘制 */
    _Draw(15, 100, 115, 200, i);
    
    /* 激活内存设备 */
    GUI_MEMDEV_Select(hMem);
    /* 向内存设备中绘制图形 */
    _Draw(125, 100, 225, 200, i);
    /* 选择LCD */
    GUI_MEMDEV_Select(0);
    /* 将内存设备中的内容复制到LCD */
    GUI_MEMDEV_CopyToLCDAt(hMem, 125, 100);

		GUI_Delay(40);
    i++;
		if (i > 99)
		{
			i = 0;
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
    _DemoMemDev();

}

```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示
- 左侧不使用内存设备直接绘制的数字有明显的闪烁现象，而右侧使用内存设备绘制的数字稳定无闪烁