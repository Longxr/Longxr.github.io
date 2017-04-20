---
title: AE蛋疼问题插件记录
categories:
  - 影视后期
tags:
  - AE
date: 2017-04-20 11:33:36
---

<Excerpt in index | 摘要> 
重装插件总是一件蛋疼的事情，下次做个快照好了...<!-- more -->
<The rest of contents | 余下全文>
## AE插件的安装路径
> 提示什么特效副本副本重复的时候三个路径找找删掉不要的版本

- 免安装插件：X:\Adobe\Adobe After Effects CC\Support Files\Plug-ins(有时候在前一个目录)
- 需安装插件：C:\Program Files\Adobe\Common\Plug-ins\7.0\MediaCore
- C:\Program Files\Adobe\Adobe After Effects CC\Support Files\Plug-ins

## 问题：缺少XXX.dll文件
解决：下载[DirectX修复工具](http://download.csdn.net/detail/vbcom/8651973)修复下


## 问题：安装了Trapcode Suite插件套装后，3D stroke预设点了什么反应都没有
解决：找到3D stroke的安装路径D:\Adobe\Adobe After Effects CC\Support Files\Plug-ins\Trapecode Suite\Trapcode Suite 64bit，在当前目录新建一个Trapcode文件夹，把SVG文件夹放到里面，预设就能用了_(:з」∠)_

## Twitch插件问题
把插件放到D:\Adobe\Adobe After Effects CC\Support Files目录下(不要放到Plug-ins下)，预设扔到Presets里即可

## E3D插件还有模型安装
安装的第一个路径选择是插件路径，第二个路径是模型路径。插件路径一般是：D:\Adobe\Adobe After Effects CC\Support Files\Plug-ins\，模型路径随意
E3D的2.2.0是2.0.7的升级版，首先要先安装2.0.0才行。如果装过1.6.X的安装前要将之前的版本软件本体以及注册信息(在C:\ProgramData\VideoCopilot)全部删掉

##E3D插件 AE报错：could not be loaded (126)：无法加载增效工具（48：46）
解决：显卡问题，更新显卡驱动

## 中英文切换
修改中文配置文件名：
CC2015:    Adobe After Effects CC 2015\Support Files\zdictionaries\after_effects_zh-Hans.dat
CC2017:    Adobe After Effects CC 2017\Support Files\zdictionaries\after_effects_zh_CN.dat

## 中文版表达式切换回英文
修改zdictionaries中的aftre_effects_zh_Hans.dat
"$$$/AE/Effect/Name/CheckboxControl=Checkbox Control"
"$$$/AE/Effect/Name/ColorControl=Color Control"
"$$$/AE/Effect/Name/Fill=Fill"
"$$$/AE/Fill/LStr/0002=Color"
"$$$/AE/Motion_Tile/LStr/0001=Tile Center"
"$$$/AE/Slider_Control/LStr/0001=Slider"
"$$$/AE/Slider_Control/LStr/0002=Angle"
"$$$/AE/Slider_Control/LStr/0003=Checkbox"
"$$$/AE/Slider_Control/LStr/0004=Color"
"$$$/AE/Slider_Control/LStr/0005=Point"
"$$$/AE/Slider_Control/LStr/0006=Layer"

