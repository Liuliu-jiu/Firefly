---
title: Qt各类容器与函数
published: 2026-07-30
description: 分享我个人学习Qt时所记录的各类及API（第二篇）
image: "../Article_Image/Qt.png"
tags: [Qt, 函数, 容器]
category: Qt
draft: false
---

# QValidator输入验证器
## 作用
可在输入的时候验证是否输入的是你指定的类型，比如只能输入整型，就可以创建一个整型输入验证器QIntValidator，再将其设置到lineEdit中。
```cpp
ui->lineEdit->setValidator(new QIntValidator(this));//设置整型验证器到lineEdit中
```

## 示例代码
```cpp
//示例1：限制输入范围（0-100）
//创建范围限制验证器（最小值0，最大值100）
QIntValidator *validator = new QIntValidator(0, 100, this);
ui->lineEdit->setValidator(validator);

//示例2：限制输入正整数
//只允许输入正整数（1到99999）
ui->lineEdit->setValidator(new QIntValidator(1, 99999, this));

//示例3：修改已有验证器范围
//先获取已有验证器
QIntValidator *validator = qobject_cast<QIntValidator*>(ui->lineEdit->validator());
if(validator) {
    // 修改范围
    validator->setRange(10, 50);
}
```

# QLocale().toString()
## 作用
将内置类型转换为本地语言的字符串。

## 示例代码
```cpp
//​QLocale()：创建临时语言环境对象（默认使用系统语言）
//.toString(bool_value)，接收布尔值参数（true或false），返回对应的本地化字符串
QLocale().toString(bool);
```
将bool值转换为对应本地语言的字符串(true/false)，所有内置类型都可以转。  
在Qt中，使用QLocale().toString(bValue)将bool转换为QString的语法是直接调用QLocale对象的toString()方法，并将布尔值作为参数，语法
。  
## ​注意点​
该方法在Qt 5.15及以上版本可用。

# QFile的copy和rename和path,errorString
## copy
### 作用
将原来的文件复制到新路径当中。

### 函数原型
```cpp
bool QFile::copy(const QString &sourceFilePath, const QString &destinationFilePath)
```

### 返回值
true：复制成功。
false：复制失败（可通过 errorString() 获取错误信息）。

### 重要特性：
目标文件如果已存在，复制操作不会覆盖而是直接失败。

## rename：重命名文件或移动文件
### 作用
将旧的路径改为新的路径，相当于将文件直接移动至别的地方。

### 函数原型
```cpp
bool QFile::rename(const QString &originalPath, const QString &newPath)
```

### 示例代码
```cpp
// 重命名文件（同目录）同目录不同文件名
QFile::rename("/home/user/report-old.txt", "/home/user/report-new.txt");

// 移动文件到新位置	不同目录同文件名
QFile::rename("C:/docs/file.txt", "D:/archive/file.txt");
```

## copy VS rename
| 特性           | copy()       | rename()          |
|----------------|--------------|-------------------|
| 文件系统操作   | 创建副本     | 移动/重命名       |
| 原始文件       | 保留         | 移除              |
| 目标文件存在   | 失败         | 需要先删除        |
| 性能（同分区） | 需要复制数据 | 仅修改元数据      |
| 原子性         | 无           | 保证              |
| 权限保留       | 否           | 是                |
| 跨分区处理     | 直接支持     | 自动降级复制+删除 |

## path
### 作用
获取文件路径的纯目录路径，不包含文件名。

### 示例代码
```cpp
QFileInfo info(oldfilePath);
QString fileDirectory = info.path();//获取纯目录，不包含文件名
```

## QFile下errorString
### 作用
专门获取文件打开失败时的错误信息。

# QDesktopServices::openUrl(QUrl::fromLocalFile(filePath));
## 作用
我想利用使用默认程序进行打开，可以利用QDesktopServices桌面服务类的openUrl来使用默认程序进行打开，但openUrl它的参数需要URL对象，所以我通过QUrl下的fromLocalFile将文件路径转为URL后通过openUrl用默认程序对URL对象(文件路径)来进行处理。

| 组件                        | 作用           | 详细说明                        |
|-----------------------------|-----------------------------------------------------|--------------------------------------------------------------------------------------------|
| filePath  | 文件路径  | QString类型变量，表示文件在磁盘上的位置，如："C:/Docs/report.docx" 或 /home/user/pic.png） |
| QUrl::fromLocalFile() | 路径转URL  | 静态函数，将本地路径转换为标准文件URL格式（如：file:///C:/Docs/report.docx） |
| QDesktopServices::openUrl() | 桌面服务类，Qt跨平台接口，提供系统级功能访问打开URL | Qt跨平台接口，提供系统级功能访问|

## QFileInfo的suffix
## 作用
截取文件后缀名。

## 示例代码
```cpp
QString fileExtension = info.suffix();//利用suffix截取文件后缀名(不带点)
```

# 关于QHash的疑问
## 哈希容器在插入和遍历时的顺序是一致吗？
答案是不一致，因为哈希容器中有多个桶，每一对键值都存储在其中一个桶中，这个桶有可能是连续的空间，有可能是链表，每一块空间都存着一对键值，而决定放哪个桶，就需要通过哈希值%桶数量来进行决定，而哈希值是由键的内容通过哈希函数计算出桶的位置来进行决定的，如果计算出数据要存放的桶里面已经有数据了，则在这个桶中新添一块内存空间即可，可重复在桶中放数据，所以这个数据被放在了最后一个数据的后一个位置。

## ​键值对存储在桶中​
每个桶对应哈希表中的一个位置，桶的索引由 hash(key) % bucket_count 计算确定。

## ​哈希值决定桶位置​
哈希值基于键的内容通过哈希函数计算得出。

## ​桶的实现结构​
每个桶是一个链表头指针。  
链表节点结构：
```cpp
struct Node {
    Key key;
    T value;
    Node* next; // 指向桶内下一个节点
};
```

## 桶数组的结构
```cpp
Node* buckets[8];//初始大小为质数
```

# 如何在对话框添加自定义按钮？
## 示例代码
创建出一个QMessBox对象，利用addButton将你所想要显示的按钮加到对话框
```cpp   
QMessageBox msBox;
QPushButton* confrim = msBox.addButton("确认",QMessageBox::YesRole);
QPushButton* cancel = msBox.addButton("取消",QMessageBox::NoRole);
```

## 代码讲解
第一个参数是你按钮显示的文本，第二个参数是角色。
那角色是什么？  
角色是决定该按钮是哪个功能类型的，比如我确认按钮是yes(QMessageBox::YesRole)功能，就将该按钮放到yes的位置上，当然按钮不仅仅决定位置，还决定它的行为绑定，如acceptRole绑定的就是回车键，总结而言就是，角色就是看该按钮划分到哪个功能区，从而决定它的布局位置，行为绑定（按键绑定，一般只有acceptRole和rejectRole有快捷键绑定），如果设置多个按钮为相同角色，是没有问题的，只是将相同角色的按钮放在同一区域。
  
通过addButton来达到我添加自定义按钮的目的，然后通过addButton的返回值来获取该按钮本身，方便后续判断用户点击了哪些按钮，点击按钮时，通过clickedButton的返回值与我按钮本身进行对比，可得知用户点击了哪个按钮。

## 一开始创建对话框的叉号是不可点击状态，这是为什么？
原因是没有关联的按钮，我想让叉号关联至取消按钮，则将取消按钮的角色设置为QMessageBox::RejectRole，这样叉号关联到取消按钮，esc也绑定取消按钮，也能正常点击了。

## 获取提问对话框的图标并设置到自定义对话框上
```cpp
QMessageBox mb(QMessageBox::Question,"","");//创建一个对话框，第一个参数将该对话框设置为提问消息对话框类型，第二个参数是标题，第三个是文本
msBox.setIconPixmap(mb.iconPixmap());//获取提问对话框的图标将其设置在自定义对话框上
```

# QGraphicsView（视图）
## 作用
如果你想让一张或多张图片或或形状（矩形，三角形等）显示在窗口并能进行复杂操作（如滚动，缩放，放大，旋转，用户输入事件处理等等），则可以用QGraphicsView视图类来进行操作，**QGraphicsView是一个强大的视图可视化窗口，能对显示的图片进行复杂的操作。**

## 如何使用
首先利用场景（容器）QGraphicsScene来存放多个图形项QGraphicsPixmapItem（QGraphicsItem的子类，专门存放pixmap图片），最后通过视图QGraphicsView显示场景（容器）中的图形项。
```cpp
//创建场景(一个容器)，用于存放图形项
QGraphicsScene* scene = new QGraphicsScene(this);

//创建图形项
QPixmap pixmap(":/new/prefix1/image/ppt.png");//将图片加载至pixmap中
QGraphicsPixmapItem* pixmapItem = new QGraphicsPixmapItem(pixmap);//将pixmap包装成图形项
scene->addItem(pixmapItem);//将图形项加入至场景（容器）中

//创建视图（可视化窗口）
QGraphicsView* view = new QGraphicsView(this);
view->setScene(scene);//视图关联场景，代表该视图要显示的是该场景下的图形项
view->resize(800,600);
```
通过布局算法来显示多张图片。

# QStyledItemDelegate
## 作用
QStyledItemDelegate是通过一些方法来进行显示数据的一个类。

## 主要负责
1. 数据的显示（在paint()方法中）。
2. 编辑器的创建和销毁。
3. 为项提供大小提示。
4. 处理特殊事件（如工具提示）。

## 自定义委托
自定义编辑，显示时的行为，自己定义如何进行显示，编辑等行为。

### 自定义委托的意义
通过创建自定义委托，可以：  
​1.定制显示效果​ - 改变项的外观。  
2.修改交互行为​ - 添加特定事件处理。  
​3.扩展功能​ - 添加额外的显示元素（如本例的工具提示(悬停显示文本)）。  

```cpp
class TooltipDelegate : public QStyledItemDelegate {
public:
    explicit TooltipDelegate(QObject* parent = nullptr) : QStyledItemDelegate(parent) {}

    bool helpEvent(QHelpEvent* event, //事件发生的对象
                   QAbstractItemView* view, //发生的组件
                   const QStyleOptionViewItem& option, //当前的样式
                   const QModelIndex& index) override //当前的索引
    {

	//空指针检查
        if (!event || !view)
            return false;

        if (event->type() == QEvent::ToolTip) {//如果事件属于消息提示类型
            // 从ItemData获取路径（角色设为Qt::UserRole）
            QVariant pathData = index.data(Qt::UserRole);//获取路径
            if (!pathData.isNull()) {//查看是否为空
                QToolTip::showText(event->globalPos(), pathData.toString(), view);//进行悬停提示
                return true;
            }
        }
	//如果不是消息提示类型，则按照默认处理下·
        return QStyledItemDelegate::helpEvent(event, view, option, index);
    }
};
```

# QMultiMap 和 QMultiHash 的语法与用法
这两个容器都是能够存储一键多值或多键一值的容器，QMultiMap和QMultiHash插入时候，将每一对键值对进行独立存储。  
比如：
```cpp
multiMap.insert("key1", "value1");
multiMap.insert("key1", "value2");
multiMap.insert("key2", "value3");
```
内部多了3个条目
```cpp
("key1", "value1")
("key1", "value2")
("key2", "value3")
```
如果使用values进行查找值的话，则会将所有的值放在QList容器上返回来。

## QMultiMap的插入和遍历顺序是否一致?
QMultiMap插入和遍历的顺序是不一样的，QMultiMap是基于红黑树实现，插入的时候自动根据键的比较结果(默认升序)进行排序，遍历则按照排序时的顺序输出。

## QMultiHash的插入和遍历顺序是否一致?
QMultiHash插入时也是将每对键值对独立存储，但是不一样的是，QMultiHash是将所有相同键放在同一桶下，使用values查找到对应的键后，遍历桶将值输出出来，同样不能保证插入和遍历的顺序一样。

## QMultiMap与QMultiMap 的比较
| 特性     | QMultiHash  | QMultiMap |
|----------|-------------|-----------|
| 底层结构 | 哈希表+链表 | 红黑树    |
| 排序     | 无序        | 按键排序  |
| 查找性能 | 平均 O(1)   | O(log n)  |
| 插入性能 | 平均 O(1)   | O(log n)  |
| 内存使用 | 较高        | 较低      |
| 遍历顺序 | 不确定      | 按键排序  |

# std::shared_ptr和 std::weak_ptr智能指针
## std::shared_ptr
### 作用
可自动管理对象的生命周期，采用引用计数方式，但对象出作用域时，该对象会自动销毁，引用计数减1。

### 示例代码
```cpp
//创建语法
auto ptr1 = std::make_shared<int>(42);//在堆区上开辟

//重置指针，释放当前对象（引用计数减1）
ptr1.reset();

//指向新对象
ptr1.reset(new int(300));

//置空
ptr1 = nullptr;
```

### 销毁机制
当多个智能指针对象指向同一块地址时，会采用引用计数方式计数有多少个指针指向这块地址，当销毁最后一个指针(即引用计数为1变0时)，就会自动销毁指针指向的地址。

### 引用计数变化规则
| 操作          | 引用计数变化                         |
|---------------|--------------------------------------|
| 创建新对象    | 1                                    |
| 拷贝构造      | +1                                   |
| 拷贝赋值      | 等号左侧对象计数-1，右侧对象计数+1   |
| 移动构造/赋值 | 计数不变（所有权转移）               |
| reset()       | -1（如果指向其他对象，原对象计数-1） |
| 销毁          | -1                                   |

### 注意事项
在qt中，对象树和智能指针如果指向同一块内存空间，就会导致对象树释放了该地址，但智能指针引用计数却没有减一，导致指针不知道这块地址释放了，所以要么只用对象树释放，要么只用智能指针。

## 循环引用
两个指针互相引用对方称为循环引用。
### 风险
导致两个指针指向的地址永久无法释放且无法访问。
比如：
```cpp
struct Node {
    std::shared_ptr<Node> next;
};

auto node1 = std::make_shared<Node>(); // 引用计数 = 1
auto node2 = std::make_shared<Node>(); // 引用计数 = 1

node1->next = node2; // 引用计数变化：
                     // node2: 1 → 2
                     // node1: 1 → 1（不变）

node2->next = node1; // 引用计数变化：
                     // node1: 1 → 2
                     // node2: 2 → 2（不变）
```
创建两个指向结构体的指针，这个结构体里面也有一个智能指针，然后让他们各自的成员指针指向对方，由于node1和node2指针指向的地址赋值了给其它指针，所以node1和node2的引用计数从1变为2，当销毁时，node1和node2的指针销毁，引用计数分别从2变为1，但由于node1和node2指向的地址被他们各自成员引用对方地址，因此引用次数都是1，结构体本身也就没销毁，但是指向结构体的指针被销毁了，因此无法释放且无法访问。  

`Q.`为什么指向结构体的指针销毁时不顺带将里面的指针销毁，这样就没有地址被引用了？  
`A.`我销毁的只是指向结构体的指针，指针只是存储结构体的地址来去访问到它，能够直接指向结构体的指针是在栈上开辟的，但结构体是在堆区上开辟的，那么结构体里面的指针也是在堆区上开辟的，因为成员指针在结构体内部，结构体怎么样开辟成员指针也就怎么开辟，当离开作用域，销毁直接指向结构体的指针，也就是在栈上开辟的变量，引用计数会减1，但结构体里面的成员指针是在堆区上开辟的，因此不会被销毁，虽然结构体里面的指针指向了对方的结构体，但是需要访问到结构体才能访问到结构体里面的指针，但是没有指针能够直接指向结构体地址，所以没得访问且无法释放。

### 解决循环引用方法
使用weak_ptr。  

`Q.`weak_ptr是什么？  
`A.`weak_ptr是弱引用指针，不增加引用次数，因此 避免循环引用行为，通过控制块来得知是否有其它指针指向该地址，强引用(shared_ptr)是拥有对象所有权进而影响生命周期，而弱引用(weak_ptr)则无对象所有权进而不能影响生命周期，只有拥有对象所有权才能增加引用次数。

## 智能指针内存结构
每次开辟智能指针时，一次都会开辟两个东西，一个是对象，一个是控制块，对象是你存储的数据，而控制块是存储强弱引用次数的地方。

## 引用会增加引用计数吗
以引用的方式传递不会增加引用计数，值传递会增加。

## 智能指针销毁规则
智能指针销毁时先销毁指向那块地址的智能指针，当发现引用次数变为0时，才会销毁那块地址，不是智能指针则先调用析构函数再销毁对象。
 
# QPalette
## 作用
QPalette是统一管理各个控件各个部分颜色的类。

## 两个重要的概念
### ColorRole (颜色角色)
告诉编译器我要指定颜色的控件是哪个（如WindowText用于文本，Button用于按钮背景）。

### ColorGroup (颜色组)
告诉编译器在哪个状态下使用哪部分颜色(如Active激活状态，Disabled禁用状态）。

## 例子
比如说QPalette::WindowText是管理窗口/控件文本颜色，而我想设置窗口/控件文本颜色
```cpp
QPalette palette = label->palette();//获取调色板
palette.setColor(QPalette::WindowText,QColor(0,0,0));//设置窗口/控件文本颜色
label->setPalette(palette);//设置调色板
```

## 注意点
当创建新的调色板时，会导致其它角色的颜色丢失，从而使用默认值，而这些默认值通常是无效状态，就可能会导致显示异常。
当我执行
```cpp
QPalette(QPalette::WindowText,QColor(255,255,255,a))
```
这行的时候，由于只指定了窗口文本颜色，就会导致其它控件的角色颜色丢失，从而使用默认值，但默认值通常是无效颜色，因此就会造成显示异常。  
`解决方法：`获取现成调色板，只修改文本颜色，就不会导致其它角色颜色丢失。

## QPalette管理的控件如下
| 颜色角色 (ColorRole)                          | 核心用途说明                                            |
|-----------------------------------------------|---------------------------------------------------------|
| QPalette::Window                              | 窗口或控件的一般背景色。                                |
| QPalette::WindowText                          | 窗口或控件的一般前景色（通常是文字颜色）。              |
| QPalette::Base                                | 文本输入控件（如 QLineEdit、QTextEdit）的背景色。       |
| QPalette::Text                                | 与 Base角色搭配使用，作为文本输入控件的前景（文字）色。 |
| QPalette::Button                              | 按钮控件（如 QPushButton）的背景色。                    |
| QPalette::ButtonText                          | 按钮控件上的文字颜色。                                  |
| QPalette::Highlight                           | 选中项（如列表中的高亮项）的背景色。                    |
| QPalette::HighlightedText                     | 选中项上的文字颜色，需要与 Highlight形成对比。          |
| QPalette::Light / Dark / Mid 等               | 主要用于生成3D斜面和阴影效果。                          |
| QPalette::Link / QPalette::LinkVisited        | 分别用于设置未访问和已访问超链接的颜色。                |
| QPalette::ToolTipBase / QPalette::ToolTipText | 分别用于设置工具提示（ToolTip）的背景色和文字颜色。     |
| QPalette::PlaceholderText                     | 设置文本输入框中的占位符文字颜色（Qt 5.12引入）。       |

# QPropertyAnimation属性动画类
## 作用
QPropertyAnimation这个类是在一定时间内平滑改变某个控件的属性，从而达到特定的动画效果，比如平滑淡出(不断修改透明度)，平滑缩放(不断修改窗口大小)，平滑移动(不断修改窗口坐标)等。

## 构造函数
```cpp
QPropertyAnimation::QPropertyAnimation(QObject *target, const QByteArray &propertyName, QObject *parent = nullptr)
```
第一个参数指定要动画化的对象，第二个参数指定动画化的属性(不一定是控件对象，通常在动画化对象里面的属性，但这个属性能够操控控件属性)，第三个是父对象。

## 例子
比如说：我想实现平滑淡出效果，平滑淡出效果的本质其实就是修改控件透明度，因此我需要用到QGraphicsOpacityEffect来去修改控件透明度。
```cpp
//方法1：使用QGraphicsOpacityEffect实现整体透明
QGraphicsOpacityEffect *opacityEffect = new QGraphicsOpacityEffect(label);//创建透明度效果对象，使其操作label的透明度
label->setGraphicsEffect(opacityEffect);//将对象应用至label上，这样这个对象在渲染时会经过这个透明度对象进行渲染，就能操作label的透明度，生命周期由label管理

//我想实现平滑淡出动画效果，就创建一个QPropertyAnimation，这个在一定时间内用来平滑改变控件属性类，可用于平滑淡出，缩放，移动
QPropertyAnimation *animation = new QPropertyAnimation(opacityEffect, "opacity", this);//创建动画类，指定要动画化的对象，第二个参数指定要平滑改变的属性(opacity是透明度，在opacityEffect对象中有opacity属性，可以通过这个属性操作label的透明度)
animation->setDuration(2000);// 动画时长2秒
animation->setStartValue(1.0);//从完全不透明
animation->setEndValue(0.0);//完全透明
animation->setEasingCurve(QEasingCurve::InOutQuad);//设置动画的缓动曲线，就是在动画过程中速度要发生怎么样的变化，QEasingCurve::InOutQuad就把速度设置为开始末尾快，不设置默认是匀速
animation->start();
``` 

# QGraphicsOpacityEffect透明度对象
## 作用
QGraphicsOpacityEffect是用来修改控件的透明度，比如修改窗口，按钮的透明度。

## 示例代码
例如：我要修改label控件的透明度，你就把控件对象放到QGraphicsOpacityEffect构造函数中，代表要通过这个透明度对象修改控件透明度。
```cpp
QGraphicsOpacityEffect *opacityEffect = new QGraphicsOpacityEffect(label);  //创建透明度效果对象，使其操作label的透明度
label->setGraphicsEffect(opacityEffect);          

int windowY = pos.y() - this->label->height();
this->smoothMoveAnimation->setStartValue(QPoint(pos.x(),windowY));
this->smoothMoveAnimation->setEndValue(QPoint(pos.x(),windowY - 20));
```

# QMutex和QMutexLocker 锁机制
## 作用
在日志系统中，多个线程同时写入时，如果没有锁机制，就会导致日志文件顺序错乱，甚至引发程序崩溃问题，为了解决这样的问题，就需要引入锁机制来保证线程安全。

## QMutex
### 作用
Qt提供的互斥锁，作用是将线程并行改为线程同步，确保只有一个线程访问共享资源。

## QMutexLocker
### 作用
Qt提供的一种基于RAII（Resource Acquisition Is Initialization，资源获取即初始化）的机制辅助类，通常用来更好的管理QMutex的锁定和解锁状态。
### 工作原理
`构造时：`传入QMutex对象，会在构造函数中锁定这个锁，此时只有一个线程访问共享资源。
`析构时：`当对象离开作用域时，会自动析构QMutexLocker 对象，同时会将之前锁定的锁解锁。

## 当多个线程写入时会发生什么？
A,B,C线程同时调用writeLog方法，只会有一个线程获取到锁，其它线程被操作系统挂起休眠，当该线程执行完后，会释放该锁，操作系统会将其中一个阻塞线程拿来执行，同时上锁，直到全部线程执行完毕。  
通过这个现象我们发现，锁机制就是将线程并行改为线程同步来保证线程安全。

## 优势
1. 异常安全：即使临界区代码中抛出了异常，QMutexLocker的析构函数仍然会被调用，锁会被正确释放，避免了因异常导致锁永远无法释放的死锁问题。
2. 代码简洁：你不需要手动调用 lock()和 unlock()，减少了忘记解锁的风险。
3. 可读性强：在代码中清晰标明了临界区的开始和结束（即 QMutexLocker对象的生命周期）。
