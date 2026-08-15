---
title: Qt各类容器与函数
published: 2026-07-22
description: 分享我个人学习Qt时所记录的各类及API（第一篇）
image: "../Article_Image/Qt.png"
tags: [Qt, 函数, 容器]
category: Qt
draft: false
---

# QString::section
## 作用
以分隔符分割字符串，让分隔符与分隔符之间成为字段，索引从0开始。

## 函数原型
```cpp
//函数原型
QString QString::section(const QString &sep, qsizetype start, qsizetype end = -1, QString::SectionFlags flags = SectionDefault) const
//参数1：以什么字符作为分隔，参数2：起始索引，参数3：末尾索引，参数4：跳过空格字段（默认不跳过）
```
## 核心概念
| 概念           |  作用                                      |
|----------------|------------------------------------------ |
| 字段（Section） | 用分隔符将字符串分割后得到的子字符串。        |
| 字段索引        | 从 0 开始计数，分隔符之间的内容即为一个字段。 |

## ​示例说明​
假设字符串为 "A:B:C:D:E"，分隔符是 ":"
```cpp
QString str = "A:B:C:D:E";
```
分割后的字段​：
| 字段索引 | 字段值 | 解释                   |
|----------|--------|------------------------|
| 0        | "A"    | 第一个分隔符前的部分   |
| 1        | "B"    | 两个分隔符之间的部分   |
| 2        | "C"    | 以此类推               |
| 3        | "D"    | .....                  |
| 4        | "E"    | 最后一个分隔符后的部分 |

## 具体案例 
QString::section 的 ​**start 和 end 参数是基于分割后的字段索引**​（即分隔符将字符串切割成的“段”）的，而不是字符位置或分隔符的个数。具体来说：

- 案例1
```cpp
str.section(":", 1, 3)*
```
`​作用​：`获取字段索引从 1 到 3 的子字符串。    
`​结果​：`"B:C:D"（字段1是 "B"，字段2是 "C"，字段3是 "D"）。  
`​分隔符处理​：`字段之间的分隔符会被保留。

- 案例2
```cpp
str.section(":", 2, -1)
```
`​作用​：`end = -1 表示获取从字段 2 到最后一个字段的子字符串。  
`​结果​：`"C:D:E"（字段2是 "C"，字段3是 "D"，字段4是 "E"）。

- 案例3
```cpp
str.section(":", 0, 0)
```
`​作用​：`仅获取第一个字段。  
`​结果​：`"A"。

# QString::arg
## 作用
将参数替换到格式化字符串中，用于格式字符串。
## 函数原型
```cpp
template <typename... Args> QString QString::arg(Args &&... args) const
```
## 具体案例
```cpp
QString qstBook = QString("Id：%1，Name：%2").arg(book->m_Id).arg(bookName);
```
设置好格式后，将这个格式后的字符串返回到qstBook，此时qstBook就有了这样的格式，%1和%2是占位符，相当于%d，后面的arg就是将按照顺序将参数替换至字符串的占位符中。

# QTextCursor::mergeCharFormat
## mergeCharFormat()的真实行为
### 作用
直接用光标文本为选中的文本或后续的文本应用样式，比如我想加粗选中字体，那么就可以获取文本光标利用mergeCharFormat应用

### 场景
| 场景         | 行为                                     |
|--------------|------------------------------------------|
| 有选中文本时 | 只修改选中文本的格式，不改变光标位置格式 |
| 无选中文本时 | 修改光标位置格式，影响后续输入           |

### ​关键特性​
实际上不会更新当前字符格式状态，但会影响光标位置的格式属性。

### 示例代码
```cpp
void MainWindow::on_toolButton_clicked()
{
    bool buttonChecked = ui->toolButton->isChecked();//查询按钮当前状态

    if(buttonChecked)
    {
        QTextCharFormat format;
        format.setFontWeight(QFont::Bold);  // 设置加粗
        setTextEditInputFontWeight(format);
    }
    else
    {
        QTextCharFormat format;
        format.setFontWeight(QFont::Normal);
        setTextEditInputFontWeight(format);
    }
}
void MainWindow::setTextEditInputFontWeight(QTextCharFormat format)
{
    // 获取当前文本光标，QTextCursor是专门存储光标的类，里面有光标的相关信息，如选中的状态
    QTextCursor cursor = ui->textEditInput->textCursor();
    QTextCharFormat orignialFormat = ui->textEditInput->currentCharFormat();
    //hasSelection是判断用户是否选中了文本
    if (cursor.hasSelection())
    {
        // 对选中文本应用格式
        cursor.mergeCharFormat(format);//如果选中了，那就将旧格式与新格式合并，只改变字体粗线，保留其它属性，并且间接影响到后续输入

    }
    else
    {
        //如果没选中，则设置后续的文本都为加粗
        // 只设置光标位置的字符格式（不影响整个段落）
        QTextCharFormat currentFormat = cursor.charFormat();//获取当前光标字符格式位置
        currentFormat.merge(format);

        cursor.setCharFormat(format);//将格式应用在光标上

        // 更新光标
        ui->textEditInput->setTextCursor(cursor);//由于我们修改的光标是一个副本，所以要通过setTextCursor来将修改后的光标应用在原始光标上
    }
}   
```

## setCurrentCharFormat()的真实行为
通过控件更改光标字体样式，并更新当前格式状态

### 示例代码
```cpp
textEdit->setCurrentCharFormat(format);
```

### 场景与行为
| 场景         | 行为                                |
|--------------|-------------------------------------|
| 有选中文本时 | 修改选中文本格式 + 更新当前格式状态 |
| 无选中文本时 | 更新当前格式状态，影响后续输入      |

## 关键误会点
当有选中文本时，mergeCharFormat()确实不更新格式状态，但会改变光标位置的格式属性，后续当前光标字符格式会继承光标位置格式属性，所以就当选中文本时，会间接影响后续输入的格式，比如选中文本后进行加粗，后续的文本也会被加粗。

## mergeCharFormat vs setCurrentCharFormat
| 方法                   | 调用场景   | 对选中文本影响   | 对后续输入影响           |
|------------------------|------------|------------------|--------------------------|
| mergeCharFormat()      | 有选中文本 | 修改选中文本格式 | 间接影响（改变位置格式） |
| mergeCharFormat()      | 无选中文本 | 修改光标位置格式 | 直接影响                 |
| setCurrentCharFormat() | 有选中文本 | 修改选中文本格式 | 直接影响（更新状态）     |
| setCurrentCharFormat() | 无选中文本 | 修改光标位置格式 | 直接影响（更新状态）     |

# QFileInfo
## 示例代码
```cpp
//使用QFileInfo直接获取文件名（最推荐的方法）
QString filePath = "/home/user/docs/report.txt";
QFileInfo info(filePath);
QString lastField = info.fileName();//返回 "report.txt"

// 或者获取不带扩展名的文件名
QString baseName = info.baseName();   // 返回 "report"
```

# 怎么获取鼠标在整个屏幕的位置?
## 方法
1. QCursor::pos()
2. event.globalPos()	
    ```cpp
    #include<QCursor>
    QPoint p = event.globalPos();//当事件触发时获取鼠标在整个屏幕的位置
    ```
如果想通过事件触发坐标，那么用QContextMenuEvent类，否则直接QCursor::pos()。
# QContextMenuEvent类讲解
## 作用
可按下特定键或触发某个事件显示菜单，所以它本质上用来呼出菜单用的。

## 构造函数
```cpp
QContextMenuEvent::QContextMenuEvent(QContextMenuEvent::Reason reason, const QPoint &pos, const QPoint &globalPos, Qt::KeyboardModifiers modifiers = Qt::NoModifier)
```
`参数1（Reason）：`是指在什么事件被触发时会呼出菜单。  
`参数2（pos）`：触发时获取相对于接收部件的鼠标坐标。  
`参数3（globalPos）：`触发获取相对于屏幕的鼠标坐标。  
`参数4（modifiers）：`触发事件时用户按了什么辅助键，可以通过这个改变菜单内容。

# QMenu类
## 作用
创建一个菜单，用于给程序提供额外功能，比如提供显示复制粘贴等功能。

## 示例代码
```cpp
QMenu * menu = new QMenu(tr("复制界面"),this);//tr是国际翻译语言函数
QAction* copy = menu->addAction("复制此项");//往里面添加一个菜单项
```

# QClipboard和QApplication类
## 作用
获取windows的剪贴板并将文本设置到windows的剪贴板上
## 示例代码
```cpp
#include<QClipboard>//用于对windows的剪贴板进行操作
#include<QApplication>//用于获取windows的剪贴板
QClipboard* borad = QApplication::clipboard();//首先得通过clipboard获取windows的剪贴板，然后通过setText设置内容即可
borad->setText(clickingItem->text());//将item的文字复制到windows的剪贴板
```

# QDirIterator类
## 作用
QDirIterator是专门遍历目录和子目录的迭代器

## 构造函数
有4个，我放其中一最多的
```cpp
//QDirIterator it(路径, 过滤规则, 文件类型, 遍历选项);
QDirIterator::QDirIterator(const QString &path, const QStringList &nameFilters, QDir::Filters filters = QDir::NoFilter, QDirIterator::IteratorFlags flags = NoIteratorFlags)
```
`参数1（path）：`代表要遍历的起始路径。  
`参数2（nameFilters）`：代表要过滤的文件。  
`参数3（filters）：`目录查找方式，查找文件还是连着目录也查找。  
`参数4（modifiers）：`查找的文件状态，查找只读文件还是隐藏文件等。

## 示例代码
```cpp
/*
path代表要遍历的起始路径，QStringList() << "*.txt"代表要过滤的文件
QDir::Files | QDir::Readable | QDir::Hidden分别代表只查找文件(不包含目录)，只查找只读文件，查找隐藏文件
QDirIterator::Subdirectories | QDirIterator::FollowSymlinks分别代表查找所有目录和处理符号链接
*/
QDirIterator it(directoryPath,filteredFile,QDir::Files | QDir::Readable | QDir::Hidden ,QDirIterator::Subdirectories | QDirIterator::FollowSymlinks);
```

## 需要注意的点
初始状态的时候指针是指向第一个元素之前的，跟C++的有些不同

## 元素之间迭代器 vs 元素指向迭代器
| 特性           | "元素之间"迭代器 (如 Java/QDirIterator) | "元素指向"迭代器 (如 C++ 标准库) |
|----------------|-----------------------------------------|----------------------------------|
| 指针位置       | 位于元素之间的间隙                      | 直接指向元素本身                 |
| 初始状态       | 在第一个元素之前                        | 指向第一个元素                   |
| 获取当前元素   | 通过 next() 移动并返回                  | 直接解引用迭代器 *it             |
| 下一个元素     | next() 移动并返回下一个                 | ++it 只移动                      |
| 第一个元素获取 | 必须调用 next()                         | 初始 *begin 就是第一个元素       |

# QFile::remove删除指定文件
## 示例代码
```cpp
QFile::remove(deletePath)//deletePath填要删除文件名的相对路径或绝对路径，删除时不经过回收站

QFile::moveToTrash(deletePath);//删除时将文件放到回收站
```

# QListWidgetItem::listWidget()
## 作用
返回item所属的QListWidget组的指针，可用于判断item是否在QListWidget下。

## 示例代码
```cpp
item->listWidget() == listWidget
```
listWidget会返回它所属的QListWidget指针，如果是的话，就为真，反之为假。

# QList::indexOf
## 作用
从左往右找第一次出现的元素，并返回该元素的索引。

## 示例代码
```cpp
int index = list.indexOf("1");/找回来返回当前索引，找不到返回-1
```

# QVariantList（即 QList\<QVariant\>）
## 作用
能够存储不同类型元素的容器。
## 存储原理
当用append加入元素时，会在大容器内部创建一个QVariant实例(可以理解为小容器)这个小容器里面有两个值，一个是值副本，一个是值的类型，此时会识别出你元素的数据类型并拷贝到值的类型中，并以二进制的方式存储你的值到值副本中。
如当执行以下代码时
```cpp
array.append(42)
```
QVariant 创建一个内部存储空间，将整数 42 ​原样复制到自己的存储空间中，同时记录该值的类型信息​（int）。

## 取出方式
取出来时如果知道元素是什么类型可直接转换成对应类型，如果不知道，可通过userType来查看该元素类型后再进行转换，但通常建议先检查再转换，防止转换错误类型。
```cpp
//取出来时先检测我要转换的类型是否符合存储的类型，再进行转换，否则如果转换不对会崩溃
if(v.canConvert<QListWidget*>())//检测类型，<>里面是我要检测的类型
{
	转换之前value也会进行一次检测，只不过检测失败可能会导致崩溃，所以建议先检测后转换
	 RightClickListWidget* listWidget = v.value<RightClickListWidget*>();//<>里面是我要转换的类型
}
```

## 存储其它类型时的注意点
除了内置的数据类型以外，其它类型建议不要直接append加入，要先创建一个QVariant实例后，QVariant的fromValue将其它类型对象加入到QVariant后，再通过append加入至QVariantList容器当中。
如
```cpp
QVariant v = QVariant::fromValue(ui->listWidget);将QListWidget类型的地址加入到QVariant实例
varList.append(v);//再通过append加入至容器中
```

## 为什么存储其它类型时要使用fromValue来封装
qt知道的类型可以直接放到append参数中，因为qt知道怎么封装它，而其它类型类型需要我们自己创建实例手动利用fromValue将值拷贝进去，因为qt不知道怎么封装。

## 自定义类型存储方式
需要将声明自定义类型并注册到元对象系统中使用
```cpp
Q_DECLARE_METATYPE(RightClickListWidget*)//RightClickListWidget*是要声明的自定义类型
qRegisterMetaType<RightClickListWidget*>("RightClickListWidget*");//注册该类型
```
完整代码
```cpp
//声明的时候需要去自定义类中的全局作用域下声明，如#ifndef RIGHTCLICKLISTWIDGET_H
#define RIGHTCLICKLISTWIDGET_H
class RightClickListWidget : public QListWidget
{
    Q_OBJECT
public:
    explicit RightClickListWidget(QWidget* parent = nullptr);

signals:
    void itemRightClicked(QListWidgetItem*item);
protected:
    void mousePressEvent(QMouseEvent* event) override;
};
Q_DECLARE_METATYPE(RightClickListWidget*) // 声明指针类型支持，在这里声明
#endif // RIGHTCLICKLISTWIDGET_H

//注册的时候需要在程序启动前注册
#include "mainwindow.h"

#include <QApplication>
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    qRegisterMetaType<RightClickListWidget*>("RightClickListWidget*");//在这里注册
    MainWindow w;
    w.show();
    return a.exec();
}
```

# QStatusBar::showMessage
## 作用
在状态栏里显示一段时间的文字，但是会隐藏原本状态栏的组件，可通过函数重载版本提高显示时长。

# QElapsedTimer计时器
## 作用
用来计算时间的。

## 应用场景
如果想获取某个函数执行时间，可以在函数执行前利用start启动计算器，在函数执行后利用elapsed获取时间，此时的时间是毫秒，用qint64类型来接收，qint64其实就是long long，专门存储大的毫秒数。

## 注意事项
重复对同一个QElapsedTimer对象利用start启用，会将之前的记录清零，重新计时，所以再次计时的时候无需重新创建，对同一个对象start即可。

## start()和restart()的区别
| 特性         | start()          | restart()                            |
|--------------|------------------|--------------------------------------|
| 首次使用     | 只能调用一次     | 初始化计时器，必须已有计时器才能调用 |
| 重复调用效果 | 重置计时器       | 重置计时器并返回上次耗时长           |
| 返回值       | void（无返回值） | 返回上次启动到重置的时间             |
| 使用场景     | 首次启动计时器   | 循环或重复任务计时                   |

# 如何利用QTimer(定时器)和QProgressBar(进度条)做定时更新进度条
## 示例代码
```cpp
ui->progressBar->setFormat("当前进度:");
//初始化
ui->progressBar->setRange(0,100);//设置最小和最大
ui->progressBar->setValue(0);//设置进度条当前值
ui->progressBar->setTextVisible(true);//是否开启百分比模式

//如果想通过进度条显示进度，可以和QTimer进行配合，定时器开启后，每50毫秒会自动触发timeout信号，触发后再更新进度条
timer = new QTimer(this);
connect(ui->pushButton,&QPushButton::clicked,[=](){
    timer->start(50);
    ui->progressBar->show();
});
count = 0;
connect(timer,&QTimer::timeout,[this](){
//更新进度条
    count += 5;//每次更新百分之5
    ui->progressBar->setValue(count);
    if(count >= 100)
    {
        count = 0;
        ui->progressBar->hide();
        ui->progressBar->setValue(count);
        timer->stop();//停止定时器
    }
});
/*
如果想设置进度条高度，可以通过样式表设置QProgressBar的chunk(进度条)，进度条显示的时候有两个东西，一个是一条线，一个是显示进度时推进的线，设置为推进的线高度后，也要设置一条线的高度，否则显示的时候推进的线是高的，但是一条线的高度是低的，导致观感不好
*/
ui->progressBar->setStyleSheet(
"QProgressBar {"
"   border: 1px solid #CCCCCC;" // 边框
"   border-radius: 3px;"
"   background-color: #F0F0F0;"
"   height: 30px;"
"}"
"QProgressBar::chunk {"
"   background-color: #3498DB;"
"   width: 10px;" /* 每个块的宽度（水平进度条） */
"   margin: 0px;" /* 去掉块之间的间隔 */
"   height: 30px;" /* 或者设置100%来充满 */
"}");
```
如果想在进度条加个取消按钮，可以用QProgressDialog(进度条对话框)。