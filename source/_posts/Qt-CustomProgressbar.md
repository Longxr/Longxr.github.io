---
title: Qt自定义进度条
categories:
  - Qt
tags:
  - Qt
  - QProgressbar
date: 2017-11-16 21:47:45
---

<Excerpt in index | 摘要> Qt真的是一个绘图软件emmmmm<!-- more -->
<The rest of contents | 余下全文>

##  瞎扯

> 转眼已经折腾Qt（画图软件23333）快5个月了，C++的东西没学到多少，各种软件绘图技巧倒是折腾了不少。在刚开始工作的时候看到设计稿的软件界面还是一脸懵逼的，现在发现没有什么是Qt的paint()和[PS](https://baike.baidu.com/item/ADOBE%20PHOTOSHOP/2297297?fromtitle=photoshop&fromid=133866)解决不了的，如果有，再加个[Ae](https://baike.baidu.com/item/Adobe%20After%20Effects/5452364?fromtitle=ae&fromid=1368576)~ o(*￣▽￣*)ブ



##  简介

在需要按比例显示数据的时候，进度条是必不可少的。Qt中有一个进度条相关的类QProgressBar，这个类提供了简单的水平或垂直的进度条，可以通过修改qss简单的改变下样式:
![自带进度条](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-CustomProgressbar_01.png)


简单使用的话自带的样式就够看了，但是也有时候设计非要搞些酷炫的效果，没办法只能折腾下了......比如，设计稿非要整个两端椭圆的进度条，qss就没辙了。这篇文章主要总结下Qt在绘制控件方面的经验，画都能画出来了，其他骚操作就随便搞了，下面是折腾出来的栗子：
![各种进度条](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Qt-CustomProgressbar_02.gif)


##  各种进度条

### 重绘paint之两端圆角进度条(继承QProgressbar)
图中第一个，其实也可以直接继承QWidget的，就少写个设置百分比和最小最大值的函数，主要还是重写paint函数，画出进度条底图和根据当前value值画出蓝色进度条：
```
void RadiusProgressBar::paintEvent(QPaintEvent *)
{
    QPainter p(this);
    QRect rect = QRect(0, 0, width(), height()/2);
    QRect textRect = QRect(0, height()/2, width(), height()/2);

    const double k = (double)(value() - minimum()) / (maximum()-minimum());
    int x = (int)(rect.width() * k);
    QRect fillRect = rect.adjusted(0, 0, x-rect.width(), 0);

    QString valueStr = QString("%1%").arg(QString::number(value()));
    QPixmap buttomMap = QPixmap(":/resource/radius_back.png");
    QPixmap fillMap = QPixmap(":/resource/radius_front.png");

    //画进度条
    p.drawPixmap(rect, buttomMap);
    p.drawPixmap(fillRect, fillMap, fillRect);

    //画文字
    QFont f = QFont("Microsoft YaHei", 15, QFont::Bold);
    p.setFont(f);
    p.setPen(QColor("#555555"));
    p.drawText(textRect, Qt::AlignCenter, valueStr);
}
```

### 重绘paint之纯色圆环进度条(继承QWidget)
图中第二个，这是单纯用程序画出来的，没有贴图，在画出底图的圆以及上层的圆弧后，在上层再画一个透明的圆，相当于遮罩：
```
void RingsProgressbar::paintEvent(QPaintEvent *)
{
    QPainter p(this);
    p.setRenderHint(QPainter::Antialiasing);

    m_rotateAngle = 360*m_persent/100;

    int side = qMin(width(), height());
    QRectF outRect(0, 0, side, side);
    QRectF inRect(20, 20, side-40, side-40);
    QString valueStr = QString("%1%").arg(QString::number(m_persent));

    //画外圆
    p.setPen(Qt::NoPen);
    p.setBrush(QBrush(QColor(97, 117, 118)));
    p.drawEllipse(outRect);
    p.setBrush(QBrush(QColor(255, 107, 107)));
    p.drawPie(outRect, (90-m_rotateAngle)*16, m_rotateAngle*16);
    //画遮罩
    p.setBrush(palette().window().color());
    p.drawEllipse(inRect);
    //画文字
    QFont f = QFont("Microsoft YaHei", 15, QFont::Bold);
    p.setFont(f);
    p.setFont(f);
    p.setPen(QColor("#555555"));
    p.drawText(inRect, Qt::AlignCenter, valueStr);
}
```


### 重绘paint之贴图圆环进度条(继承QWidget)
图中第三个，在玩第二种进度条的时候，意外发现drawPie()设置笔刷后可以直接画圆弧，这种骚操作 不试试还真不知道：
```
void RingsMapProgressbar::paintEvent(QPaintEvent *)
{
    QPainter p(this);
    p.setRenderHint(QPainter::Antialiasing);

    m_rotateAngle = 360*m_persent/100;

    int side = qMin(width(), height());
    QRectF outRect(0, 0, side, side);
    QRectF inRect(20, 20, side-40, side-40);
    QString valueStr = QString("%1%").arg(QString::number(m_persent));

    //画底圆
    p.setPen(Qt::NoPen);
    QPixmap backMap = QPixmap(":/resource/progress_back.png");
    p.drawPixmap(outRect, backMap, outRect);
    //画内弧
    QPixmap frontMap = QPixmap(":/resource/progress_front.png");
    p.setBrush(QBrush(frontMap));
    p.drawPie(outRect, (90-m_rotateAngle)*16, m_rotateAngle*16);
    //画文字
    QFont f = QFont("Microsoft YaHei", 15, QFont::Bold);
    p.setFont(f);
    p.setPen(QColor("#DDDDDD"));
    p.drawText(inRect, Qt::AlignCenter, valueStr);
}
```

### 重绘paint之属性动画进度条(继承QWidget)
图中第4个，上面3个都是用程序控制显示进度的，最后一个是直接将进度条动画导出成序列帧图片，播放动画的形式来画进度条。这种方式主要是在某些等待操作以及需要进度条动画有曲线的时候用的，前提是你要会做动画和导出序列帧哦(⊙o⊙)：
```
AnimationProgressbar::AnimationProgressbar(QWidget *parent) : QWidget(parent),
    m_animaindex(0),
    m_animaTotal(30),
    m_persent(0)
{
    QPixmap aniMap(":/resource/sequence_animate.png");

    for(int i=0; i<m_animaTotal; i++){
        m_animalist << aniMap.copy(i*(aniMap.width()/m_animaTotal), 0, aniMap.width()/m_animaTotal, aniMap.height());
    }

    m_animation=new QPropertyAnimation(this,"");
    m_animation->setStartValue(0);
    m_animation->setEndValue(m_animaTotal - 1);
    m_animation->setDuration(2000);
    m_animation->setLoopCount(-1);
    connect(m_animation,SIGNAL(valueChanged(QVariant)),this,SLOT(slot_valuechange(QVariant)));
}

void AnimationProgressbar::startAnimation()
{
    update();

    m_animation->start();
}

void AnimationProgressbar::paintEvent(QPaintEvent *)
{
    QPainter p(this);
    p.setRenderHint(QPainter::SmoothPixmapTransform);

    int side = qMin(width(), height());
    QRect outRect(0, 0, side, side);
    QRect inRect(20, 20, side-40, side-40);
    QString valueStr = QString("%1%").arg(QString::number(m_persent));

    //画圆环
    p.drawPixmap(outRect, m_animalist.at(m_animaindex));

    //画文字
    QFont f = QFont("Microsoft YaHei", 15, QFont::Bold);
    p.setFont(f);
    p.setPen(QColor("#555555"));
    p.drawText(inRect, Qt::AlignCenter, valueStr);
}

void AnimationProgressbar::slot_valuechange(QVariant var)
{
    m_animaindex = var.toInt();
    m_persent = m_animaindex*100 / m_animaTotal;

    update();
}
```


##  进度条栗子

代码仓库在这里->[Github链接地址](https://github.com/Longxr/QtCustomProgressbar)，喜欢的话点下star哦~
