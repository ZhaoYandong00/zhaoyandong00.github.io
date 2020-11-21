---
title: JPEG、PNG图片显示STemWin篇27
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin JPEG、PNG图片显示
---
# JPEG
## JPEG 图片
- JPEG是一种标准的全色和灰度图像压缩方法
- emWin 的JPEG 库在解码图像时会固定占用33KB 的RAM空间，这部分与图像大小无关，而总的RAM占用可通过下面的公式得到： 总RAM占用 = 图像X 方向大小 * 80 字节 + 33KB 
- 由于开发板内存较小，不做实验

## JPEG 显示相关API
- 绘制已加载到内存中的JPEG 文件`int GUI_JPEG_Draw(const void * pFileData, int DataSize, int x0, int y0)`
- 绘制不需要加载到内存中的JPEG 文件`int GUI_JPEG_DrawEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0)`
- 绘制具有缩放比例的JPEG 文件，该文件已加载到内存中`int GUI_JPEG_DrawScaled(const void * pFileData, int DataSize, int x0, int y0, int Num, int Denom)`
- 从已加载到内存中的JPEG 文件填充GUI_JPEG_INFO 结构`int GUI_JPEG_GetInfo(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0, int Num, int Denom)`
- 从不需要加载到内存中的JPEG 文件填充GUI_JPEG_INFO 结构`int GUI_JPEG_GetInfoEx(GUI_GET_DATA_FUNC * pfGetData, void * p, GUI_JPEG_INFO * pInfo)`

# PNG
## PNG 图片
- PNG（Portable Network Graphics）格式，是一种光栅图形文件格式，支持无损数据压缩和透明度，支持基于调色板的图像、灰度图像以及非调色板的全色RGB 图像
- PNG 库在做解码操作的时候会固定占用21KB 的RAM 空间，这部分与图像大小无关，而总的RAM占用可通过下面的公式得到：总RAM占用 = （图像xSize + 1）* 图像ySize*4 + 21KB
- 由于开发板内存较小，不做实验

## PNG 显示相关API
- 绘制已加载到内存中的PNG 文件`int GUI_PNG_Draw(const void * pFileData, int DataSize, int x0, int y0)`
- 绘制不需要加载到内存中的PNG 文件`int GUI_PNG_DrawEx(GUI_GET_DATA_FUNC * pfGetData, void * p, int x0, int y0)`
- 返回加载到内存中的位图的X 大小`int GUI_PNG_GetXSize(const void * pFileData, int FileSize)`
- 返回不需要加载到内存中的位图的X 大小`int GUI_PNG_GetXSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`
- 返回加载到内存中的位图的Y 大小`int GUI_PNG_GetYSize(const void * pFileData, int FileSize)`
- 返回不需要加载到内存中的位图的Y 大小`int GUI_PNG_GetYSizeEx(GUI_GET_DATA_FUNC * pfGetData, void * p)`

## PNG库
- 下载并解压emWin_png 压缩包。PNG 解码库可以从emWin 官网下载到，链接如下：[emWin-PNG](https://www.segger.com/downloads/emwin/emWin_PNG)
- 添加PNG 库到工程文件夹。将PNG 库整个文件夹复制到工程
- 添加PNG 库文件到工程
- 添加PNG 库头文件路径