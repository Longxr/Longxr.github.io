---
title: Qt自定义Dialog
categories:
  - Qt
tags:
  - Qt
  - QDialog
date: 2017-07-29 16:21:02
---

<Excerpt in index | 摘要> 
自定义Qt对话框Demo<!-- more -->
<The rest of contents | 余下全文>



## 默认对话框

Qt默认的对话框是系统自带的样式，不同版本的windows也有些许不同，如果希望风格统一的话，这时候就需要自定义一个属于自己的对话框了

![系统自带](http://upload-images.jianshu.io/upload_images/2756183-c4ea50fde2fe7f0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![自定义样式](http://upload-images.jianshu.io/upload_images/2756183-8f3daac0e7b9ac0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 新建Qt设计师类
虽然是自定义界面，但是有一个.ui文件放置按钮还是要方便些，如果是直接新建一个C++类文件就需要在代码中添加相关的控件了

![新建设计师类](http://upload-images.jianshu.io/upload_images/2756183-e63504076098c1d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 去除默认的样式
在构造函数中加上：
```
setAttribute(Qt::WA_TranslucentBackground, true);
        setWindowFlags(Qt::Window | Qt::FramelessWindowHint
                       | Qt::WindowSystemMenuHint | Qt::WindowMinimizeButtonHint
                       | Qt::WindowMaximizeButtonHint);
```
就把默认的标题栏、背景等给去掉了。现在运行的话背景是透明的，除了自己加的控件外，其他啥也没有。

## 重写paintEvent
自己对于Dialog的界面有什么需求，可以在paintEvent函数中添加。例如，我想设置边框为圆角，在文本输入框上下各有一条分割线：
```
void LoginDialog::paintEvent(QPaintEvent *event)
{
    QPainterPath path;
    path.setFillRule(Qt::WindingFill);
    QRect rect = QRect(0, 0, this->width(), this->height());
    path.addRoundRect(rect,10,10);

    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing, true);
    painter.fillPath(path, QBrush(Qt::white));
    painter.setPen(Qt::gray);
    painter.drawPath(path);

    painter.drawLine(rect.left()+40, 60, rect.right()-40, 60);
    painter.drawLine(rect.left()+40, rect.bottom()-70, rect.right()-40, rect.bottom()-70);
}
```

# 重写鼠标事件
由于默认的情况被我们去掉了，此时对话框是不能移动的，需要重写鼠标事件，函数声明：
```
void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
void mouseReleaseEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
```

具体实现：
```
void LoginDialog::paintEvent(QPaintEvent *event)
{
    QPainterPath path;
    path.setFillRule(Qt::WindingFill);
    QRect rect = QRect(0, 0, this->width(), this->height());
    path.addRoundRect(rect,10,10);

    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing, true);
    painter.fillPath(path, QBrush(Qt::white));
    painter.setPen(Qt::gray);
    painter.drawPath(path);

    painter.drawLine(rect.left()+40, 60, rect.right()-40, 60);
    painter.drawLine(rect.left()+40, rect.bottom()-70, rect.right()-40, rect.bottom()-70);
}

void LoginDialog::mousePressEvent(QMouseEvent *event)
{

    if( event->button() == Qt::LeftButton){
        m_Press = event->globalPos();
        leftBtnClk = true;
    }
    event->ignore();//表示继续向下传递事件，其他的控件还可以去获取
}

void LoginDialog::mouseReleaseEvent(QMouseEvent *event)
{

    if( event->button() == Qt::LeftButton ){
        leftBtnClk = false;
    }
    event->ignore();
}

void LoginDialog::mouseMoveEvent(QMouseEvent *event)
{
    if( leftBtnClk ){
        m_Move = event->globalPos();
        this->move( this->pos() + m_Move - m_Press );
        m_Press = m_Move;
    }
    event->ignore();
}
```

## 添加自己的控件
之后就像是在用默认的Dialog一样，在.ui中添加需要的控件，以及需要的信号/槽函数这些了。

![界面设计](http://upload-images.jianshu.io/upload_images/2756183-3e06f1424003c671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了重绘的背景之外，其他的控件都可以在ui文件里添加。附上[Github链接地址](https://github.com/Longxr/QDialogDemo)。