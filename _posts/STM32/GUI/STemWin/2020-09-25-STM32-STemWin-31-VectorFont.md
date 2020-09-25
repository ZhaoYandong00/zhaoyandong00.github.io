---
title: 矢量字体显示STemWin篇31
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin 矢量字体显示
---
# 矢量字体
- 矢量字体(Vector font)中每一个字形是通过数学曲线来描述的，它包含了字形边界上的关键点，连线的导数信息等，字体的渲染引擎通过读取这些数学矢量，然后进行一定的数学运算来进行渲染。这类字体的点是字体实际尺寸可以任意缩放而不变形、变色。目前较为流行的矢量字体主要包括 Type1 、 TrueType、OpenType 等几类
- 在矢量字体文件中，每个字符都是以一系列的点存放的，其中有一些点是控制点，利用这些控制点来实现绘制抛物线的功能
- P1 是控制点，P0、P2 是曲线的端点，因此曲线可以用下面的公式进行描述，这描述方式就是贝塞尔曲线

    $ B(t) = (1-t)^2P_0 + 2t(1-t)P_1+t^2P_2,t\in[0,1] $

- 和SIF 格式一样，使用TTF格式字库的时候需要将字库全部加载到RAM 中，然后使用TTF 字体渲染引擎显示字体
- emWin 可以使用一款叫FreeType 的TTF 字体渲染引擎，
- 此引擎占用的RAM空间在很大程度上取决于所用的字体，但最少会占用80~300KB，其中200KB 用于字体的光栅化位图缓存
- 此引擎的ROM需求250K.具体大小取决于CPU、编译器和编译器的优化级别
- 由于开发板内存较小，不做实验

# 移植TTF 渲染引擎
- 下载并解压TTF 引擎。FreeType 字体渲染引擎可以从emWin 官网下载到，链接如下：[FreeType](https://www.segger.com/downloads/emwin/emWin_FreeType)
- 添加TTF 引擎到工程。选择好版本之后，将TTF 引擎库整个文件夹复制到工程
- 添加PNG 库头文件路径
- 修改启动文件。由于TTF 引擎需要用到malloc、free 和relloc 等C 库函数，而这些函数又使用的是系统堆内存空间，所以需要增加堆内存空间大小

# TTF 字体显示相关API
- 从TTF 字体文件创建GUI 字体`int GUI_TTF_CreateFont(GUI_FONT * pFont, GUI_TTF_CS * pCS)`
- 使用抗锯齿从TTF 字体文件创建GUI 字体`int GUI_TTF_CreateFontAA(GUI_FONT * pFont, GUI_TTF_CS * pCS)`
- 销毁TTF 引擎的缓存`void GUI_TTF_DestroyCache(void)`
- 释放TTF 引擎的所有动态分配的内存`void GUI_TTF_Done(void)`
- 返回字体的家族名称`int GUI_TTF_GetFamilyName(GUI_FONT * pFont, char * pBuffer, int NumBytes)`
- 返回字体的样式名称`int GUI_TTF_GetStyleName(GUI_FONT * pFont, char * pBuffer, int NumBytes)`
- 可用于设置TTF 缓存的默认大小`void GUI_TTF_SetCacheSize(unsigned MaxFaces, unsigned MaxSizes, U32 MaxBytes)`

