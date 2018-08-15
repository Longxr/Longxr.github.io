---
title: Qt-AutoLayout
categories:
  - Qt
tags:
  - Qt
  - Layout
date: 2018-02-28 16:12:10
---

<Excerpt in index | 摘要> 
Qt之自动布局<!-- more -->
<The rest of contents | 余下全文>

> 每月一更差点忘了emmmmmm，新年第一更

## Qt的布局类

比较常用的有QHBoxLayout、QVBoxLayout、QGridLayout、QButtonGroup等。

|    类   | 说明 |
| ---------- | --- |
| QBoxLayout | 水平或垂直排列控件 |
| QButtonGroup | 组织按钮的容器 |
| QFormLayout | 管理输入控件和其相关的标签 |
| QGraphicsAnchor | 表示在QGraphicsAnchorLayout中两个项目之间的锚 |
| QGraphicsAnchorLayout | 在图形视图中可以将锚连接到一起 |
| QGridLayout | 网格布局（多行多列） |
| QGroupBox | 带标题的分组框 |
| QHBoxLayout | 水平排列控件 |
| QVBoxLayout | 垂直排列控件 |
| QLayout | 几何管理器的基类 |
| QLayoutItem | 抽象的操作布局Item |
| QSizePolicy | 描述水平和垂直大小调整的策略 |
| QSpacerItem | 布局中的空间隔 |
| QStackedLayout | 切换控件，同一时间只有一个控件可见 |
| QStackedWidget | 切换控件，同一时间只有一个控件可见 |
| QWidgetItem | 表示一个控件的布局项 |

## 参数介绍
### 工作原理
> 1. 所有的widgets将根据他们的QWidget::sizePolicy()和QWidget::sizeHint()被初始化一定数量的空间。
> 2. 如果一个widget有stretch factors(值大于0的)要设置，那么就会为他们分配这些空间。
> 3. 如果其他widgets都不在需要空间了，那么就会把剩余的空间都给那些stretch factors为0的widget。那些设置了QSizePolicy::Expanding的优先分配。
> 4. 那些设置了最小大小却还没有被分配到他们的最小值的widgets会被分配空间直到他们的最小值。
> 5. 那些设置了最大大小却分配空间多于了这个最大值的widgets会被分配空间直到他们的最大值。

### SizePolicy属性
控件的sizePolicy说明控件在布局管理中的缩放方式。Qt提供的控件都有一个合理的缺省sizePolicy，但是这个缺省值有时不能适合所有的布局，开发人员经常需要改变窗体上的某些控件的sizePolicy。一个QSizePolicy的所有变量对水平方向和垂直方向都适用。下面列举了一些最长用的值：

|    属性值   | 说明 |
| ---------- | --- |
| Fixed | 控件不能放大或者缩小，控件的大小就是它的sizeHint |
| Minimum | 控件的sizeHint为控件的最小尺寸。控件不能小于这个sizeHint，但是可以放大 |
| Maximum | 控件的sizeHint为控件的最大尺寸，控件不能放大，但是可以缩小到它的最小的允许尺寸 |
| Preferred | 控件的sizeHint是它的sizeHint，但是可以放大或者缩小 |
| Expandint | 控件可以自行增大或者缩小 |

注：sizeHint（布局管理中的控件默认尺寸，如果控件不在布局管理中就为无效的值）

### layoutStretch属性
布局中的layoutStretch表明了在该布局中的控件所占比例的分配，默认是0。当设置了大于0的数字后，其余为0的控件默认只占最小的大小，剩下的空间则按照比例分配给数字大于0的控件。举个栗子，在一个水平布局中有3个控件，控件的SizePolicy属性都是*Preferred*，布局的layoutStretch属性为：*0，0，1*，那么前2个控件占有最小大小，最后一个控件占满剩下的空间。
![布局栗子](https://upload-images.jianshu.io/upload_images/2756183-7984f8a4c469af3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 代码中使用自动布局
### 将控件加入布局
在代码中将一个控件加入布局主要用:
`void QBoxLayout::addWidget([QWidget](qwidget.html) *widget, int  stretch = 0, [Qt::Alignment](../qtcore/qt.html#AlignmentFlag-enum)  alignment = Qt::Alignment())`

QGridLayout要多2个参数,定义控件所占单元格大小：
`void QGridLayout::addWidget([QWidget](qwidget.html) *widget, int  fromRow, int  fromColumn, int  rowSpan, int  columnSpan, [Qt::Alignment](../qtcore/qt.html#AlignmentFlag-enum)  alignment = Qt::Alignment())`

### 将布局加入布局
布局也可以嵌套，将布局加入到另一个布局中。
`void QBoxLayout::addLayout([QLayout](qlayout.html#QLayout) *layout, int  stretch = 0)`

```
    m_pGridLayout = new QGridLayout();

    m_pSoftwareName = new QLabel(this);
    m_pSoftwareVer = new QLabel(this);
    m_pLicenseNum = new QLabel(this);
    m_pLicenseTime = new QLabel(this);

    pGridLayout->addWidget(m_pSoftwareName, 1, 0, 1, -1, Qt::AlignLeft|Qt::AlignVCenter);
    pGridLayout->addWidget(m_pSoftwareVer, 2, 0, 1, -1, Qt::AlignLeft|Qt::AlignVCenter);
    pGridLayout->addWidget(m_pLicenseNum, 1, 0, 1, -1, Qt::AlignLeft|Qt::AlignVCenter);
    pGridLayout->addWidget(m_pLicenseTime, 2, 0, 1, -1, Qt::AlignLeft|Qt::AlignVCenter);

    QVBoxLayout *mainLayout = new QVBoxLayout();
    setLayout(mainLayout);
    mainLayout->addLayout(m_pGridLayout);
    mainLayout->addStretch(1);
    mainLayout->setContentsMargins(0, 0, 0, 0);
```

### 注：
1. 将控件加入布局后还要记得设置放置控件的Widget的布局，调用*setLayout函数*。
2. 在布局空白部分留白，空白占满剩余空间，布局最后加上*addStretch(1)*。

## ui中使用自动布局
1. 在Qt设计师界面上方，选中多个控件后点击即可：
![布局按钮](https://upload-images.jianshu.io/upload_images/2756183-676ba7964ac9154d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 加入布局后还要注意设置Widget的布局：![注意Widget布局](https://upload-images.jianshu.io/upload_images/2756183-43d171c187526cc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 控件在布局中的对齐方式：
控件上右键->Layout Alignment->...(选择对齐方式)

4.设置Widget的布局，选中需要布局的Widget，在空白处右键->布局。

5. 默认的水平布局等有一个默认的间距，不需要改成0即可：

![Layout间距](https://upload-images.jianshu.io/upload_images/2756183-05eebd9e6ecfc400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考资料
- [Qt Creator 窗体控件自适应窗口大小布局](http://www.cnblogs.com/emouse/archive/2013/05/19/3087708.html)

- [Qt 布局管理器](http://blog.csdn.net/xuguangsoft/article/details/8544827)

- [Qt之布局管理器](http://blog.csdn.net/liang19890820/article/details/51517002)

  ​

