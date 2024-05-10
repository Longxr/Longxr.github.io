---
title: Qt之QListView使用
categories:
  - Qt
tags:
  - Qt
  - QListView
date: 2017-08-10 18:11:00
---

<Excerpt in index | 摘要> 
记录下使用QListView遇到的各种问题<!-- more -->
<The rest of contents | 余下全文>

QListView可以用来以列表的形式展示数据，在Qt中使用model/View结构来管理数据与视图的关系，model负责数据的存取，数据的交互通过delegate来实现。

## 添加数据模型
> QT提供了一些现成的models用于处理数据项：
> QStringListModel 用于存储简单的QString列表。
> QStandardItemModel 管理复杂的树型结构数据项，每项都可以包含任意数据。
> QDirModel  提供本地文件系统中的文件与目录信息。
> QSqlQueryModel, QSqlTableModel,QSqlRelationTableModel用来访问数据库。

使用Qt自带的模型类`QStandardItemModel`即可。模型中的每个数据项都有一个与之对应的role来存储某一类数据。需要存取自定义数据可以使用UserRole，UserRole+1...

对于自定义数据类型，如果要使用QVariant，就必须使用Q_DECLARE_METATYPE注册。
```
struct ItemData{
    QString name;
    QString tel;
};

Q_DECLARE_METATYPE(ItemData)
```

### 单一数据存取
```
//存
Item->setData(itemStatus,Qt::UserRole);  // 单一存取

//取
ItemStatus status = (ItemStatus)(index.data(Qt::UserRole).toInt());
```
### 结构体数据存取
```
//存
Item->setData(QVariant::fromValue(itemData),Qt::UserRole+1);//整体存取

//取
QVariant variant = index.data(Qt::UserRole+1);
ItemData data = variant.value<ItemData>();
```

## 自定义delegate
模型的交互和绘制通过自定义delegate来实现，暂时没用到交互，先说下item的绘制。继承了QStyledItemDelegate后，重写paint函数处理item的样式，以及sizeHint函数返回item的大小：

### 绘制item
```
void ItemDelegate::paint(QPainter *painter, const QStyleOptionViewItem &option, const QModelIndex &index) const
{


    if(index.isValid())
    {
        painter->save();

        ItemStatus status = (ItemStatus)(index.data(Qt::UserRole).toInt());

        QVariant variant = index.data(Qt::UserRole+1);
        ItemData data = variant.value<ItemData>();

        QStyleOptionViewItem viewOption(option);//用来在视图中画一个item

        QRectF rect;
        rect.setX(option.rect.x());
        rect.setY(option.rect.y());
        rect.setWidth( option.rect.width()-1);
        rect.setHeight(option.rect.height()-1);

        //QPainterPath画圆角矩形
        const qreal radius = 7;
        QPainterPath path;
        path.moveTo(rect.topRight() - QPointF(radius, 0));
        path.lineTo(rect.topLeft() + QPointF(radius, 0));
        path.quadTo(rect.topLeft(), rect.topLeft() + QPointF(0, radius));
        path.lineTo(rect.bottomLeft() + QPointF(0, -radius));
        path.quadTo(rect.bottomLeft(), rect.bottomLeft() + QPointF(radius, 0));
        path.lineTo(rect.bottomRight() - QPointF(radius, 0));
        path.quadTo(rect.bottomRight(), rect.bottomRight() + QPointF(0, -radius));
        path.lineTo(rect.topRight() + QPointF(0, radius));
        path.quadTo(rect.topRight(), rect.topRight() + QPointF(-radius, -0));

        if(option.state.testFlag(QStyle::State_Selected))
        {
            painter->setPen(QPen(Qt::blue));
            painter->setBrush(QColor(229, 241, 255));
            painter->drawPath(path);
        }
        else if(option.state.testFlag(QStyle::State_MouseOver))
        {
            painter->setPen(QPen(Qt::green));
            painter->setBrush(Qt::NoBrush);
            painter->drawPath(path);
        }
        else{
            painter->setPen(QPen(Qt::gray));
            painter->setBrush(Qt::NoBrush);
            painter->drawPath(path);
        }

        //绘制数据位置
        QRect NameRect = QRect(rect.left() +10, rect.top()+10, rect.width()-30, 20);
        QRect circle = QRect(NameRect.right(), rect.top()+10, 10, 10);
        QRect telRect = QRect(rect.left() +10, rect.bottom()-25, rect.width()-10, 20);


        switch (status) {
        case S_RED:
            painter->setBrush(Qt::red);
            painter->setPen(QPen(Qt::red));
            break;
        case S_BLUE:
            painter->setBrush(Qt::blue);
            painter->setPen(QPen(Qt::blue));
            break;
        case S_YELLOW:
            painter->setBrush(Qt::yellow);
            painter->setPen(QPen(Qt::yellow));
            break;
        }

        painter->drawEllipse(circle);     //画圆圈

        painter->setPen(QPen(Qt::black));
        painter->setFont(QFont("Times", 12, QFont::Bold));
        painter->drawText(NameRect,Qt::AlignLeft,data.name); //绘制名字

        painter->setPen(QPen(Qt::gray));
        painter->setFont(QFont("Times", 10));
        painter->drawText(telRect,Qt::AlignLeft,data.tel); //绘制电话

        painter->restore();
    }
}

QSize ItemDelegate::sizeHint(const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    return QSize(160, 60);
}
```

### 设置item不同状态下的样式
在paint函数中，还可以获得当前item的状态，并设置不同的样式：
```
 if(option.state.testFlag(QStyle::State_Selected)) //选中状态
        {
            painter->setPen(QPen(Qt::blue));
            painter->setBrush(QColor(229, 241, 255));
            painter->drawPath(path);
        }
        else if(option.state.testFlag(QStyle::State_MouseOver))//鼠标划过状态
        {
            painter->setPen(QPen(Qt::green));
            painter->setBrush(Qt::NoBrush);
            painter->drawPath(path);
        }
        else{
            painter->setPen(QPen(Qt::gray));
            painter->setBrush(Qt::NoBrush);
            painter->drawPath(path);
        }
```

设置好模型后，再对QListView进行下属性设置：
```
    ui->listView->setItemDelegate(m_delegate);       //为视图设置委托
    ui->listView->setSpacing(15);                   //为视图设置控件间距
    ui->listView->setModel(m_model);                  //为委托设置模型
    ui->listView->setViewMode(QListView::IconMode); //设置Item图标显示
    ui->listView->setDragEnabled(false); 
```

模型的数据和展示都处理好后，运行效果如下：

![listview展示](http://upload-images.jianshu.io/upload_images/2756183-971bece84e07396e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 过滤item
Qt中提供了一个方便处理模型排序和过滤的类`QSortFilterProxyModel`，通过他可以非常方便的处理我们的model。将QListView展示的model设置成代理模型：
```
    ui->listView->setItemDelegate(m_delegate);       //为视图设置委托
    ui->listView->setSpacing(15);                   //为视图设置控件间距
    m_proxyModel = new QSortFilterProxyModel(ui->listView);
    m_proxyModel->setSourceModel(m_model);
    m_proxyModel->setFilterRole(Qt::UserRole);
    m_proxyModel->setDynamicSortFilter(true);
    ui->listView->setModel(m_proxyModel);                  //为委托设置模型
    ui->listView->setViewMode(QListView::IconMode); //设置Item图标显示
    ui->listView->setDragEnabled(false);            //控件不允许拖动
```
其中，`m_proxyModel->setFilterRole(Qt::UserRole);`设置根据模型的某一项数据来处理模型的过滤。proxyModel可以设置过滤的方式，根据QString或者正则表达式来过滤：
```
m_proxyModel->setFilterFixedString(QString::number(S_RED));//根据字符串过滤
m_proxyModel->setFilterRegExp(QRegExp("^[0|2]$")); //根据正则表达式过滤
```

![过滤模型](http://upload-images.jianshu.io/upload_images/2756183-5f2a4324c47ac469.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 获取选中item
对于列表中item的操作，可以是在delegate中处理交互事件，也可以通过QListView获取到所有选中item的QModelIndex，然后对模型本身进行修改。这里我选择的后者:
```
QModelIndexList modelIndexList = ui->listView->selectionModel()->selectedIndexes();
    
```

### 设置多选
将QListView的`selectionBehavior`设置成`MultiSelection`即可。

对于多选的时候，模型的修改有一个坑。在设置了代理模型后，由于开启了动态排序模式，如果修改代理模型的数据，在第一个item修改数据后可能就不在当前过滤模型中，会被过滤掉，后面的item的QModelIndex就会变化，导致后续的修改失败。

> dynamicSortFilter : bool
> This property holds whether the proxy model is dynamically sorted and filtered whenever the contents of the source model change.
> Note that you should not update the source model through the proxy model when dynamicSortFilter is true. For instance, if you set the proxy model on a QComboBox, then using functions that update the model, e.g., addItem(), will not work as expected. An alternative is to set dynamicSortFilter to false and call sort() after adding items to the QComboBox.
> The default value is true.

有两个方法处理这个坑，一是不修改代理模型，修改源模型的数据。二是在修改模型数据的时候关闭代理模型的动态排序功能。

### 修改数据
```
QModelIndexList sourceIndexList;
    foreach (QModelIndex modelIndex, modelIndexList){
        sourceIndexList<<m_proxyModel->mapToSource(modelIndex); //获取源model的modelIndex
    }

//    g_proxyModel->setDynamicSortFilter(false);
    foreach (QModelIndex sourceIndex, sourceIndexList){
        ItemStatus status = (ItemStatus)(sourceIndex.data(Qt::UserRole).toInt());
        qDebug() << "Index : " << sourceIndex.row();

        switch (status) {
            case S_RED:
                redNum--;
                break;
            case S_BLUE:
                blueNum--;
                break;
            case S_YELLOW:
                yellowNum--;
                break;
        }

        status = S_RED;
        redNum++;

        m_model->setData(sourceIndex,status,Qt::UserRole);
    }
//    g_proxyModel->setDynamicSortFilter(true);
```

弄完大概是酱紫的：

![最终效果](http://upload-images.jianshu.io/upload_images/2756183-12a31ca99cfca2d1.gif?imageMogr2/auto-orient/strip)

Demo在这里:->[Github链接地址](https://github.com/Longxr/QListViewDemo)。
