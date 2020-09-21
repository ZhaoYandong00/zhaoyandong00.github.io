---
title: GUIBuidler的使用STemWin篇2
categories: STM32 emWin STemWin
tags: STM32 emWin STemWin
description: STemWin GUIBuidler的使用
---
# GUIBuilder介绍
- GUIBuilder 是Segger 公司为emWin 开发的一款界面编辑软件工具，用于显示界面的前期设计，或在不了解 C 语言的情况下设计界面
- emWin 的控件在GUIBuilder 可以直接通过拖放来放置和调整大小，而不必编写源代码
- 可以按上下文菜单添加其他属性，可以通过编辑小部件的属性来微调
- 设计好的界面可以保存为 C 文件，直接添加进工程中使用
- 利用好GUIBuilder 可以更快的完成emWin 界面的初步设计
- GUIBuilder在`STemWin\Software`内

# 使用
- 创建框架窗口,首先点击GUIBuilder 控件选择栏中的FrameWin 控件，创建一个框架窗口
- 修改框架窗口尺寸,在控件属性框中，修改FrameWin 控件的尺寸
- 设置标题栏高度,右键FrameWin 控件，选择Set title height，在控件属性框中新增选项中设置标题栏高度
- 设置标题内容,右键FrameWin 控件，选择Set title text，输入
- 设置标题字体,同样右键FrameWin 控件，选择Set font，在弹出的选择框中选择
- 同样的方法添加其他组件
- 保存并生成C文件
- 把C文件添加到工程
- 文件中添加MainTask 函数(如原工程中存在MainTask函数，请删除)

```c
// USER START (Optionally insert additional public code)
/**
  * @brief GUI主任务
  * @param 无
  * @retval 无
  */
void MainTask(void)
{
  CreateWindow();
  
  while(1)
  {
    GUI_Delay(1000);
  }
}
// USER END
```
# 编译调试
- 编译并下载到开发板
- 可以看到LCD显示
