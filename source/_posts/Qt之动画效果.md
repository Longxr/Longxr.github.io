---
title: Qt之动画效果
categories:
  - Qt
tags:
  - Qt
  - QPropertyAnimation
date: 2017-08-16 22:20:49
---

<Excerpt in index | 摘要> Qt自带的动画框架用来做简单的控件动画效果是比较简单易用的，像是对控件的位移、缩放、不透明度这些来做动画效果。<!-- more -->
<The rest of contents | 余下全文>

## 动画框架结构

![动画框架结构](http://upload-images.jianshu.io/upload_images/2756183-7aadfddf8ab4e1cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

动画框架由基类QAbstractAnimation以及它的两个子类QVariantAnimation、QAnimationGroup组成。基础动画由QVariantAnimation的子类QPropertyAnimation来设置，再通过将多个QPropertyAnimation和QPauseAnimation组合成为动画组（QParallelAnimationGroup、QSequentialAnimationGroup），完成一个连续的动画。

## QPropertyAnimation
QPropertyAnimation类能够修改Qt的属性值，如pos、geometry等属性。设置好动画的初值和末值，以及持续的时间后，一个属性动画就基本完成了。

### 缩放
通过修改控件的geometry属性可以实现缩放效果，也可以实现位移的动画，该属性的前两个值确定了控件左上角的位置，后两个值确定了控件的大小。
```
//scale
    QPropertyAnimation *pScaleAnimation1 = new QPropertyAnimation(ui->scaleButton, "geometry");
    pScaleAnimation1->setDuration(1000);
    pScaleAnimation1->setStartValue(QRect(190, 230, 0, 0));
    pScaleAnimation1->setEndValue(QRect(120, 160, 140, 140));
```
### 位移
如果只是需要位移动画的话，修改控件的pos属性即可。pos属性就是控件的左上角所在的位置。
```
//pos
    QPropertyAnimation *pPosAnimation1 = new QPropertyAnimation(ui->posButton, "pos");
    pPosAnimation1->setDuration(1000);
    pPosAnimation1->setStartValue(QPoint(360, 160));
    pPosAnimation1->setEndValue(QPoint(360, 350));
    pPosAnimation1->setEasingCurve(QEasingCurve::InOutQuad);
```

### 不透明度
Qt的控件没有单独的透明度属性，要修改控件的透明度可以通过QGraphicsOpacityEffect类来实现。
```
    //opacity
    QGraphicsOpacityEffect *pButtonOpacity = new QGraphicsOpacityEffect(this);
    pButtonOpacity->setOpacity(1);
    ui->opasityButton->setGraphicsEffect(pButtonOpacity);

    QPropertyAnimation *pOpacityAnimation1 = new QPropertyAnimation(pButtonOpacity, "opacity");
    pOpacityAnimation1->setDuration(1000);
    pOpacityAnimation1->setStartValue(1);
    pOpacityAnimation1->setEndValue(0);
```

### 动画曲线
动画还可以设置时间的插值曲线，默认是linear，即线性运动，通过设置QEasingCurve即可。Qt提供了40种已经定义好的曲线（如果有需要也可以自定义曲线）：

![动画曲线](http://upload-images.jianshu.io/upload_images/2756183-93a4e973c0096e6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
pScaleAnimation1->setEasingCurve(QEasingCurve::InOutQuad);
```

## QSequentialAnimationGroup

### 串行动画分组
通过将QPropertyAnimation或者QPauseAnimation加入，构成一个按加入顺序依次播放的动画组，动画组的总时长是各个加入动画的总和。

```
    QSequentialAnimationGroup *pPosGroup = new QSequentialAnimationGroup(this);
    pPosGroup->addPause(500);
    pPosGroup->addAnimation(pPosAnimation1);
```

### 往返运动
Qt的动画可以设置循环次数，默认的循环是从头再播放一遍，往返运动可以在一个串行动画组中加入初值末值相反的一组动画来实现。

## QParallelAnimationGroup

### 并行动画组
加入并行动画组的动画会同时播放，动画组的总时长是最长的动画所需的时间。
```
    m_group = new QParallelAnimationGroup(this);
    m_group->addAnimation(pScaleGroup);
    m_group->addAnimation(pPosGroup);
    m_group->addAnimation(pOpacityGroup);
```
### 延时播放
在串行动画组的开始先加入一个QPauseAnimation，再将Pause不同的串行动画组加入并行动画组就可以实现延时效果了。

### 动画方向
默认动画是从开始到结束这个方向播放的， 可以设置为从结束到开始播放。
```
m_group->setDirection(QAbstractAnimation::Backward);
```
## 动画效果
综合几个小栗子：

![动画效果](http://upload-images.jianshu.io/upload_images/2756183-0756901906a0faa2.gif?imageMogr2/auto-orient/strip)

举个栗子:->[Github链接地址](https://github.com/Longxr/QtAnimationDemo)