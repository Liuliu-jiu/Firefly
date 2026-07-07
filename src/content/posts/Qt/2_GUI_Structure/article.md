---
title: GUI程序结构与运行结构
published: 2026-07-01
description: 介绍Qt的GUI基本结构
image: "../Article_Image/Qt.png"
tags: [Qt, GUI结构与运行结构]
category: Qt
draft: true
---

# 项目配置文件
在项目配置文件中，有qmake构建工具，Qt变量，UI文件，main文件等等，下面进行一些简单的介绍。
## qmake构建系统
qmake是构建工具，它的作用是根据项目配置文件生成makefile文件，然后编译器就可以根据makefile进行编译和连接，对于Qt项目，还会为元对象编译器和用户界面编译器生成构建规则。

## qmake 配置文件中常见变量的含义
在pro文件中，如果单词是大写的，那么往往意味着这是Qt的变量。
| 变量        | 含义                                                                                 |
|-------------|--------------------------------------------------------------------------------------|
| QT          | 项目中使用的 Qt 模块列表，在用到某些模块时需要手动添加                               |
| CONFIG      | 项目的通用配置选项                                                                   |
| DEFINES     | 项目中的预处理定义列表，例如可以定义一些用于预处理的宏                               |
| TEMPLATE    | 项目使用的模板，项目模板可以是应用程序（app）或库（lib）。如果不设置就默认为应用程序 |
| HEADERS     | 项目中的头文件（.h 文件）列表                                                        |
| SOURCES     | 项目中的源程序文件（.cpp 文件）列表                                                  |
| FORMS       | 项目中的 UI 文件（.ui 文件）列表                                                     |
| RESOURCES   | 项目中的资源文件（.qrc 文件）列表                                                    |
| TARGET      | 项目构建后生成的应用程序的可执行文件名称，默认与项目名称相同                         |
| DESTDIR     | 目标可执行文件的存放路径                                                             |
| INCLUDEPATH | 项目用到的其他头文件的搜索路径列表                                                   |
| DEPENDPATH  | 项目其他依赖文件（如源程序文件）的搜索路径列表                                       |

在这些变量中，有几个变量是用来管理项目包含的文件和路径的，比如HEADERS，SOURCES，RESOURCES，INCLUDEPATH等。

##  Qt变量讲解
QT变量是定义(包含)Qt项目所用到的Qt模块，如果你想往你的项目中加一些附加模块，就可以将附加模块添加到QT变量中，比如我想加SQL模块，那么就在pro文件写QT += sql。

##  qmake替换函数
qmake提供了一种替换函数，它用来将变量值替换到当前位置以用于在配置过程中处理该值，而$$是替换函数的前缀，比如$${TARGET}，它就是一个替换函数，表示将TARGET变量值替换到当前位置处理，完整语句是这样的。

    qnx: target.path = /tmp/$${TARGET}/bin
整句话的作用就是拼接成可执行文件的路径，如果想了解更多关于qmake细节知识，就可以查看Qt帮助文档中的qmake Manual主题。

## UI文件
我们通常把**设计阶段的UI称为窗体（Form），运行阶段的UI称为窗口**
![对象查看器图片1](image/对象查看器图片1.png)
在UI文件中放了个label控件，我们可以在属性编辑器中看到继承关系是从下往上的
QLabel -> QFrame -> QWidget - QObejct
![对象查看器图片2](image/对象查看器图片2.png)
![对象查看器图片3](image/对象查看器图片3.png)
可以在对象查看器选择信号编辑器快速完成信号槽连接

## main文件
int main(int argc, char *argv[])
{
    //应用程序对象是运行在操作系统的必要条件，窗口是附着在应用程序对象运行的
    QApplication a(argc, argv);         //创建应用程序对象
    Widget w;                       //创建窗口对象
    w.show();                       //窗口显示
    return a.exec();                   //开启消息循环和事件处理
}
QApplication是标准应用程序类，通过创建应用程序和窗口对象来显示界面 

# 窗口相关的文件的作用
窗口界面设计和界面组件的事件处理是GUI程序设计的主要任务，那么
1. UI可视化结果是如何转换成代码的
2. 窗口事件是如何与处理程序关联起来的
根据这两个问题我们现在解答：
首先我们在项目构建中取消Shadow build，然后以realease的方式构建，我们就会发现项目根目录下生成了一个ui_widget.h，至于这个文件有什么用，待会再说。
而与窗口相关的文件有4个，分别是：

| 文件         |    作用                                                     |
|-------------|-------------------------------------------------------------|
| Widget.h    |定义了窗口类|
| Widget.cpp  | 实现widget类功能的源程序文件|
| Widget.ui   | 窗口UI文件，用于在QtDesigner中进行UI可视化设计。它是一个XML文件，用，来存储界面上各个组件的属性和布局内容的|
| Ui_widget.h | 经过UIC编译后的UI文件，这里面定义了一个类叫Ui_widget，这里是用C++语言来描述界面组件的属性布局，信号槽等关联内容的文件|

1. widget.h  
    在widget.h中，有几个重要部分  
    **第一部分：**

        QT_BEGIN_NAMESPACE
        namespace Ui {
        class Widget;
        }
        QT_END_NAMESPACE

    该代码声明了一个名称为UI命名空间，该空间包含名为class Widget的类，但这个类并不是widget.h中所声明的类，而是在ui_widget.h，在中的ui_widget这个类，也就是用来描述界面属性和布局的类，但名字都不同，那为什么是这个类，是因为在ui_widget.h中，它把ui_widget继承下来，重新命名为widget。  
    如：

            namespace Ui {
                class Widget: public Ui_Widget {};
            } // namespace Ui
            所以widget.h中UI命名空间包含的widget是ui_widget.h文件继承下来的子类。

    **第二部分：**

            class Widget : public QWidget
            {
                Q_OBJECT
            public:
                Widget(QWidget *parent = nullptr);
                ~Widget();

            private:
                Ui::Widget *ui;
            };
    第一行插入的是名叫 Q_OBJECT宏，这个宏是使用元对象系统必需的宏，有了它，可以使用信号槽，属性等功能，而在最后一行中有个ui指针，它是用前面命名空间中的类所创建的一个指针对象，可通过该对象找到ui_widget.h中所定义的控件对象，也就是说利用命名控件将ui_widget类导入到widget.h头文件，然后为该类创建指针对象就可以找到ui_widget所持有的控件对象。

2. widget.cpp  
在widget.cpp中，完整代码如下:

        #include "widget.h"
        #include "ui_widget.h"
        Widget::Widget(QWidget *parent)
            : QWidget(parent)
            , ui(new Ui::Widget)
        {
            ui->setupUi(this);
        }

        Widget::~Widget()
        {
            delete ui;
        }
    在这些代码中
    - #include "ui_widget.h"  
        表示 包含经UIC编译的widget.ui后生成文件，方便创建窗口对象。
    - 构造函数

            Widget::Widget(QWidget *parent)
                : QWidget(parent)   
                , ui(new Ui::Widget)
        这些代码的作用就是运行构造函数然后将创建的ui::widget赋予头文件的ui指针。
    - ui->setupUi(this);  
        它的作用是将this（窗口实例对象）指针作为参数传到setupUi中，然后它就会在该窗口对象指针上创建控件并初始化，在widget.ui中，它是一个xml文件，它存储了界面上所有控件的属性布局，信号槽等关联的内容。

3. ui_widget.h
    1. 里面定义了一个ui_widget类，用于封装可视化设计界面的类，它没有父类，没有从QWidget继承下来，所以它不是一个窗口类。
    2. 在该类的public部分会为每个组件定义一个指针，指针的名称就是你设置的对象名。
    3. setup函数中用C++描述各组件的属性，布局等关联内容，其中retranslateUi是设置组件的文字属性方法。
    4. namespace Ui

            namespace Ui {
                class Widget: public Ui_Widget {};
            } // namespace Ui
        这里定义了一个UI命名空间，并在里面声明一个由ui_widget类继承下来的widget类，这样在widget.h中就能引入该类，虽然跟widget窗口类同名，但是能用命名空间区分谁是窗口类谁不是。	
    5. 回过头来看，widget.ui经过UIC编译后生成ui_widget.h，通过widget.h引入的ui_widget的子类，然后在widget.cpp中创建该实例，调用setupUi初始化界面，这就是窗口创建与初始化过程。

# 可视化UI设计
## 布局管理  
Gird Layout：网格布局，当大小改变时，每个格子的大小都会被改变。  
Form layout：表单布局，跟网格布局，但是只有最右侧一列会改变大小，适用于只有两列组件时的布局。

## 按钮说明  
F3键：进入编辑状态，也就是正常设计状态。  
F4键：进入信号与槽的可视化设计状态，意思就是可以通过UI设计信号和槽。

## 伙伴关系  
可通过工具栏点击编辑伙伴关系进入。
![伙伴关系图1](./image/伙伴关系图1.png)
伙伴关系指的是一个标签和一个能够通过鼠标选中的组件相关联，它可以在运行时通过快捷键快速将焦点定位到标签关联的组件上，比如说
![伙伴关系图2](./image/伙伴关系图2.png)
![伙伴关系图3](./image/伙伴关系图3.png)

# 信号和槽
信号和槽是Qt的基础，使得各个组件的交互操作更简单直观。
## 信号
在特定情况下发生的通知。

## 槽
对信号响应的函数。

## 连接方式
它们之间使用`QObject::connect`静态函数关联，并且连接时可以将前面的QObject限定符去掉，比如connect。  

**Q. 在QObject::connect中，QObject限定符为什么可以去掉？**  
A. connect是QObject类的静态公有函数，当一个类继承QObject时，就把QObject相关成员函数包括connect继承下来，而子类访问父类的内容可以不加作用域，因此使用父类Qbject时是不需要加作用域。

## 语法与规则
略（内容在我xmind文档，到时候复制过来）

## 设置字体样式函数
设置下划线：`font.setUnderline(true);`  
设置斜体：`font.setItalic(true);`  
设置加粗：`font.setBold(true);`

## ui信号槽连接方式
**Q. 转到槽功能如何连接槽函数？**  
A. 如果是用转到槽功能的槽函数，那么会在`ui_widget.h`当中通过`QMetaObject::connectSlotsByName(Widget);`这行代码寻找界面上的组件将与相匹配的槽函数连接起来，具体的来说，这行代码的作用是找到该界面上的组件，然后将信号与相匹配的槽函数名称连接下来，连接的时候，又有规则，它是找到对象名和信号，按照`void on_<object name>_<signal name>(<signal parameters>);`这样的规则组成一个名称，根据名称寻找对应的槽函数，就能通过信号连接。
  
**Q. 信号编辑器是如何连接槽函数的？**  
A. ui文件编译成ui_widget文件中通过connnect连接。
![ui信号槽连接方式图1](./image/ui信号槽连接方式图1.png)
![ui信号槽连接方式图2](./image/ui信号槽连接方式图2.png)

## 自动生成函数定义步骤
`在函数名上右键 -> 重构 -> 添加定义`  
这样能够快速生成函数定义。
![自动生成函数定义步骤图1](./image/自动生成函数定义步骤图1.png)

## qmake设置图标：
1. 将图标文件放到项目根目录。
2. 在pro文件中添加RC_ICONS = 图标文件名 + 后缀即可，比如RC_ICONS = editor.ico。

# Qt项目构建过程基本原理
## 构建基本原理
除项目配置文件外，还有其它文件，  
头文件和源文件，UI文件，资源文件这三类文件都被编译成标准C++文件（就是将Qt语言转换成了正宗的C++的语言）然后通过C++编译器(GUN C++编译器或MSVC编译器)将这些文件编译和链接成可执行文件或库。  
如图所示：
![构建基本原理图](./image/构建基本原理图1.png)

# 参考书籍
[Qt 6 C++开发指南](https://baike.baidu.com/item/Qt%206%20C%2B%2B%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97/63406205)