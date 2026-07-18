---
title: 桌面宠物项目问题及解决方法（第1篇）
published: 2026-07-08
description: 分享我个人写桌面宠物时遇到的问题及解决方案
image: "../Article_Image/Qt.png"
tags: [Qt, 桌面宠物]
category: Qt
draft: false
---

# 为什么要发这篇文章
这是我个人写项目时的一些问题及思考，**旨在帮助大家能够更好的理解项目，同时避开一些坑。**  
小部分代码被优化过，所以在这篇文章中并不是和桌面宠物2.0代码完全相同。

# 桌面宠物项目链接
gitee链接：<https://gitee.com/liulijiu/project_repository/tree/master/%E9%A1%B9%E7%9B%AE%E4%BB%93%E5%BA%93/%E6%A1%8C%E9%9D%A2%E5%AE%A0%E7%89%A9/%E6%A1%8C%E9%9D%A2%E5%AE%A0%E7%89%A92.0>  

github链接：<https://github.com/Liuliu-jiu/TablePet>  
github访问不了下载[Watt Toolkit(原名steam++)](https://steampp.net/)加速

# 问题分类
## 拖拽功能
### 1.为什么会获取屏幕区域时输出1440和900
```cpp
//statemanagement.cpp

this->mainScreenSize = QGuiApplication::primaryScreen()->size();
```
分辨率有两种，一种是物理分辨率，对于人眼而言能够观察到的，一种是逻辑分辨率，是对于程序而言的分辨率，为了能够更好的处理，Qt将物理分辨率缩小了一些，而缩小后的尺寸就是逻辑分辨率，虽然缩小了，但是对于程序而言仍然能够获取到整个屏幕的信息，我的物理分辨率是2880  x1800，对于程序而言缩小为1440x900，这也就是为什么会输出1440x900，因为输出的是逻辑分辨率，是正常的。
| 视角     | 分辨率      | 说明                           |
|----------|-------------|--------------------------------|
| 人眼视角 | 2880 × 1800 | 物理分辨率，屏幕实际的像素数量 |
| 程序视角 | 1440 × 900  | 逻辑分辨率，经过缩放后的坐标系 |
| 缩放关系 | 2:1         | 设备像素比(DPR)为2.0           |

### 2.将标题栏边框隐藏，背景透明后，用左键点击宠物无法暂停移动
```cpp
//mainwidget.cpp

this->setAttribute(Qt::WA_TranslucentBackground);//设置透明背景
this->setWindowFlags(Qt::FramelessWindowHint | Qt::WindowStaysOnTopHint);//设置隐藏标题栏和边框并且使得窗口一直置于顶层
```
- 原因  
    可拖拽的部分是在标题栏左边的图标和右边的按钮之间的，但由于我窗口设置的大小了，导致可拖拽的部分缩小到最小甚至没了，就无法进行拖拽了。
- 解决方法  
    把窗口放大，让可拖拽区域显示出来。但这样会把标题栏显示出来，效果不好，所以要实现自定义拖拽。    

### 3.点击QWidget(不含标题栏)为什么不能移动
- 原因  
    拖拽功能是基于标题栏进行移动的，只有我在标题栏点击的左键后移动，才能移动整个窗口，而我的图片是在QWidget内的，当我点击QWidget试图移动时，由于Qt没有自带的QWidget的拖拽功能，所以没办法移动。

- 解决方法  
    实现自定义拖拽。

### 4.怎么实现自定义拖拽功能
```cpp
//mainwidget.cpp
void MainWidget::mousePressEvent(QMouseEvent* event)
{
    if(event->button() == Qt::LeftButton){
        this->isPress = true;                       //禁止窗口移动
        this->eventPos = event->pos();              //获取鼠标在窗口中的坐标
    }
    else if(event->button() == Qt::RightButton){
        this->isRightMenu = true;                   //当右键菜单时，窗口禁止移动
        this->rightClickMenu->move(QCursor::pos()); //移动至鼠标基于屏幕的位置
        this->rightClickMenu->show();
    }
    else{
        QWidget::mousePressEvent(event);
    }
}
void MainWidget::mouseReleaseEvent(QMouseEvent* event)
{
    this->isPress = false;//允许窗口移动
}
void MainWidget::mouseMoveEvent(QMouseEvent* event)
{
    //拖拽功能
    if(this->isPress && event->buttons() == Qt::LeftButton){ /移动时是否按的是左键
        QPoint windowPos = QCursor::pos() - this->eventPos;  //获取鼠标相对于屏幕的坐标，减去鼠标相对于窗口的坐标，得出窗口左上角的坐标
        if(windowPos.x() < 0){ //检测是否超出左边界
            windowPos.setX(0); //将窗口边框设置在边界线上
        }
        else if(windowPos.x() + this->width() > this->avaliableSize.width()){ //检测是否超出右边界
            windowPos.setX(this->avaliableSize.width()- this->width());
        }
        if(windowPos.y() < 0){  //检测是否超出上边界
            windowPos.setY(0);
        }
        else if(windowPos.y() + this->height() > this->avaliableSize.height()){   //检测是否超出下边界
            windowPos.setY(this->avaliableSize.height() - this->height());
        }
        this->logSystemPtr->writeLog(LogSystem::Info,QString("用户拖拽的坐标：(%1,%2)").arg(windowPos.x()).arg(windowPos.y()));
        this->move(windowPos);

        if(this->performanceWindowAction->isChecked()){  //如果性能窗口选项处于选中状态，性能窗口会不断处于桌面宠物下方
            emit this->performanceWindow.requestPerformanceWindowMove(calculatePerformanceWindowPos(windowPos));
        }
    }
}
```
<p>拖拽时想要窗口内被点击的位置跟着鼠标移动，就需要用到mouseMoveEvent，这个事件能够监听鼠标移动，当每移动一次，就会触发mouseMoveEvent事件，如果写了窗口跟随鼠标位置代码，那么窗口就会跟着鼠标一起移动。</p>

那么究竟怎么跟随移动呢？  
首先明确一点，move是把窗口的左上角的位置移动到新位置，当有标题栏时，是基于标题栏左上角移动的，当标题栏隐藏时，是基于客户区左上角进行移动的，而我的程序是隐藏标题栏的，当我移动的时候，我点击的是客户区里面的区域，如果此时直接移动，那么就会导致窗口的左上角会直接移动至鼠标位置，而我希望的是被点击区域能够跟随着是鼠标指针，所以我得写一个逻辑，怎么让客户区里面的被点击区域移动至鼠标位置
由于move是基于左上角移动的，所以移动后我需要根据被点击区域计算出左上角的坐标，然后利用move移动至左上角的坐标，这样被点击区域就能跟随着鼠标位置移动了。  

那我怎么计算呢？
由于被点击的区域是跟随着鼠标进行移动的，因此移动后无法直接得出窗口左上角的坐标，我可以获取被点击区域相对于屏幕的位置和相对于窗口的位置，然后利用被点击区域相对于屏幕的位置减去相对于窗口的位置(被点击区域相对于窗口位置的偏移量)即可得出左上角窗口的位置。

比如我窗口是在100，100位置  
`200宽，200高`  
我点击了客户区里面的区域，被点击区域相对于屏幕坐标是（150，150），相对于窗口坐标是（50，50），我通过屏幕位置（150，150）减去窗口位置（50，50）就得出（100，100）而这个坐标，正好是窗口左上角的位置。

100，100 左上角坐标  
200，200 窗口大小  

### 5.我怎么限制拖动区域
```cpp
//statemanagement.cpp    

this->avaliableSize = QGuiApplication::primaryScreen()->availableSize();  //获取可用空间大小(不包括任务栏)
if(windowPos.x() < 0){  //检测是否超出左边界
    windowPos.setX(0);  //将窗口边框设置在边界线上
}
else if(windowPos.x() + this->width() > this->avaliableSize.width()){/  /检测是否超出右边界
    windowPos.setX(this->avaliableSize.width()- this->width());
}
if(windowPos.y() < 0){  //检测是否超出上边界
    windowPos.setY(0);
}
else if(windowPos.y() + this->height() > this->avaliableSize.height()){   //检测是否超出下边界
    windowPos.setY(this->avaliableSize.height() - this->height());
}
```
首先，我希望可移动的区域是中心部分，不包括任务栏，通知栏，所以我需要获取可用部分的尺寸（不包括任务栏，通知栏），然后根据可用部分的尺寸限制窗口移动，当移动出可用尺寸时，就将窗口的位置置于边界。

### 6.QWidget::pos基于标题栏左上角的点还是基于客户区左上角的点
(0, 0) 点是父窗口部件的客户区（Client Area，即不包括边框和标题栏的部分）的左上角。

## 行走功能
### 1.QWidget在屏幕移动的时候为什么y轴会发生变化
```cpp
QPoint p = this->mapToGlobal(QPoint(0,0));
```
- 原因  
    mapToGlobal获取的是客户区左上角(0,0)点的位置，不包含标题栏，而move是基于标题栏左上角的点进行移动的
    当我获取客户区左上角的0,0点后，往右加x像素，本来客户区的0，0点是在标题栏的0，0点的正下方的，往右加就变成了右下方了，当我移动的时候标题栏就被移动至右下角那个点了，就呈现了是往右下角移动的，本质是使用的坐标系统不同，mapToGlobal使用的是客户区的坐标系统(标题栏下方的区域)，而move使用的是整个窗口的坐标系统(包含标题栏的)。
- 解决方法  
    通过this->pos获取整个窗口在父窗口(屏幕)的坐标。

### 2.当我点击宠物后就隐藏窗口了，但依旧还在动，这是怎么回事
- 原因  
    当我点击狐狸以外的应用程序时，被点击的应用程序就会被置于顶层，就把狐狸程序覆盖下去了。
- 解决方法  
    将窗口一直置于顶层。

### 3.窗口标志设置问题
```cpp
//mainwidget.cpp

//设置应用程序一直顶层（单字段）
this->setWindowFlags(Qt::WindowStaysOnTopHint);

//隐藏标题栏和边框（单字段）
this->setWindowFlags(Qt::FramelessWindowHint);

//隐藏标题栏和边框（多字段）
this->setWindowFlags(Qt::FramelessWindowHint | Qt::WindowStaysOnTopHint);
```
为什么字段分开来设置却设置不了顶层，合在一起就行？
- 原因  
    每次调用setWindowFlags会覆盖前一次的标志，当我设置隐藏标题栏和边框标志时，就会隐藏上一次设置的一直置于顶层标志。
- 解决方法
    利用按位或运算符将标志位合在一起设置。

## 日志系统
### 1.日志文件同步写入问题
`Q1. `我有一个日志系统类，写入日志时希望使用同一个file对象，**但问题是，我有多个源文件，要想调用写入日志函数，那么就要在每个源文件上创建一个日志系统对象，但同时我又想将多个源文件调试信息写入至同一个日志文件中，如果按照我现在的办法，会导致多个file对象以写的形式打开，最终只能由一个file对象写入，其它文件的信息会丢失，我在想有没有一种方法，能够将file对象注册到qt中，这样所有文件就可以直接使用同一个file对象写入了**  
`A1.` **使用单例模式的懒汉式解决我的问题(创建一次实例后就不再创建，通过静态指针和静态函数实现)**  
具体来说通过使用静态函数和静态指针来去使得不同源文件获取到同一个的实例，调用静态函数时，查看静态指针指向的是否有效，如果无效，说明未创建实例，那就创建实例后返回该地址，下次调用时，发现静态指针指向有效地址，就不再创建，把该地址返回过来即可，这样确保多个对象调用静态函数时获取到的都是同一个实例，把构造函数的权限改为私有防止调用创建多个不同的实例。

`Q2.` 为什么不同源文件可以访问到同一个静态变量呢？  
`A2.` 静态变量在定义时会放在一个单独的存储空间中，使得不同源文件可以访问到该指针的地址。
```cpp
//logsystem.h/logsystem.cpp

static std::shared_ptr<LogSystem> logSystemPtr;//静态指针，用于指向自己
explicit LogSystem(QObject *parent = nullptr);//将构造函数变为私有，防止创建多个对象
LogSystem(const LogSystem& logSystem) = delete;//将拷贝构造删了，防止利用拷贝构造创建出不同的实例
std::shared_ptr<LogSystem> LogSystem::getLogSystemObject()
{
    if(logSystemPtr == nullptr){//如果未开辟对应的空间
        logSystemPtr = std::shared_ptr<LogSystem>(new LogSystem());//使用new开辟空间，将这块空间交给智能指针管理
    }
    return logSystemPtr;//否则返回静态指针指向的地址使其不同源文件获取到同一个对象
}
this->logSystemPtr = LogSystem::getLogSystemObject();//获取日志系统对象
```

### 2.关于静态变量问题
`Q.` 在类内声明静态变量，在类内分初始化为什么会报错?  
错误信息：logsystem.cpp.obj:-1: error: LNK2001: 无法解析的外部符号 "public: static class std::shared_ptr<class LogSystem> LogSystem::logSystemPtr" (?logSystemPtr@LogSystem@@2V?$shared_ptr@VLogSystem@@@std@@A)

`A.`
1. 静态变量不属于任何一个对象，所以不能在类内定义初始化，需要在类外单独初始化，初始化的过程就是在分配一个空间。
2. 类内声明静态变量并不是分配空间，只是告诉编译器有这个存在。
3. 避免重定义错误，如果每个源文件都在类内初始化静态变量，会导致重名。


### 3.大小轮换机制`LogSystem::logFileExistsMechanism`问题
```cpp
//logsystem.cpp

QString LogSystem::logFileExistsMechanism(int index)
{
    //通过索引递增寻找未创建的日志文件
    QString tempPath;
    do{
        //先转换为整数递增后加附加至字符串末尾，防止出现字符9加到字符冒号的问题出现
        tempPath = QCoreApplication::applicationDirPath() + "/" +QCoreApplication::applicationName() +"_LogFile_"+ QString::number(index++) + ".log";
    }while(QFile(tempPath).exists());
    return tempPath;
}
```

`Q1.` 怎么实现?  
`A1.` 通过检查大小来去重名命文。

`Q2.` 怎么重名命文件去开一个新文件?  
`A2.` 拿到文件名最后一个字符，使其ascll码加1，能够从字符1变为字符2，然后替换文件名末尾的字符，就能成为一个新的日志文件。  
`Q2.1` 为什么要这么做可以呢？  
`A2.1` 文件名由两部分组成，前缀和文件的索引，文件的索引决定了要打开哪个文件，比如TableLogFile1，后面的1代表这是第一个文件，同时也代码文件数量为1，我打开的第一个日志文件，当更换文件时，我将字符1加1变成字符2后，代表这是第二个文件，同时文件数量变为2，再替换掉最后一个字符，变为TableLogFile2，这样打开的时候就能打开新的日志文件了，同时第一个日志文件也不会被删除，下次打开时可以通过读出来的文件名确定写入的日志文件。

`Q3.` 关于保存日志文件名配置问题  
为什么不把日志类的配置写入至桌面宠物的数据库中，而是直接写入文件?  
`A3.` 最主要的原因是简单，不用写连接数据库的代码，不用将文件名传入至其它文件写入配置。

### 日志配置文件不存在的情况
`Q.` 当配置文件不存在时怎么处理，重新创建配置文件后日志文件名怎么办，日志文件名我希望是应用程序名+LogFile + 索引 组成日志文件名  
`A.` 使用日志文件存在检测机制
当日志配置文件不存在时，说明程序无法确定打开的日志文件，因此我需要重新创建配置文件的同时确定打开的日志文件，而我希望打开的是新的日志文件，同时我不希望旧的日志文件被覆盖，所以我从索引1开始，依次往后递增，递增的期间检查当前索引的文件是否存在，如果存在则说明该索引的日志文件已被创建，属于旧的日志文件，不能被覆盖，索引就继续递增，直到该名字的日志文件没被创建。
```cpp
//logsystem.cpp

char index = '1';//文件索引
do
{
    //this->logFileName记录的是不带后缀的文件名
    this->logFileName = QCoreApplication::applicationName() + "LogFile" + index ;
    index += 1;
}while(QFile(this->logFileName + ".log").exists());
```

## 切换状态功能
### 1.怎么实现双击切换状态功能，从走路姿势切换成站立呼吸姿势?
利用两个定时器在同一后台进程中双击来回切换姿势即可
```cpp
//mainwidget.cpp

void MainWidget::mouseDoubleClickEvent(QMouseEvent* event)
{
    //双击切换状态
    this->isWalk = !(this->isWalk);
    qDebug() << "双击切换后的状态：" << (this->isWalk ? "行走" : "站立");

    //重载运算符+没有准备两个char字符串相加的情况，因此无法拼接，重载运算符+准备的场景有两个QString拼接或一个QString一个char字符串拼接
    //所以为了达到拼接的要求，将其中char字符串一个转为QString后，使用重载运算符+就能将char字符串转为QString字符串后拼接
    this->logSystemPtr->writeLog(LogSystem::Info,QString("双击切换后的状态：") + (this->isWalk ? "行走" : "站立"));
    if(this->isWalk){
        int walkFrequency = configFile.readConfigValue(APP_CONFIG_JSON_FILE,"walkFrequency").toInt();
        s.walk(walkFrequency);
    }
    else{
        int idleFrequency = configFile.readConfigValue(APP_CONFIG_JSON_FILE,"idleFrequency").toInt();
        s.idle(idleFrequency);
    }
}
```

### 2.定时器线程错误
1. 跨线程定时停止错误
```cpp
connect(this,&Widget::requestChangeCatStatus,this,[this,&isRun,&catRunTimer,&catIdleTimer](){
                isRun = !isRun;
                if(isRun)
                {
                    //双击切换成小猫奔跑效果
                    //停止小猫站立呼吸定时器
                    catIdleTimer.stop();

                    //开启小猫奔跑移动定时器
                    catRunTimer.start(SWITCH_RUN_IMAGE_TIEM);
                }
                else
                {
                    //双击切换成小猫站立呼吸效果
                    //停止小猫奔跑定时器
                    catRunTimer.stop();

                    //开启小猫站立定时器
                    catIdleTimer.start(SWITCH_IDLE_IMAGE_TIME);
                }
            });
错误1：QObject::killTimer: Timers cannot be stopped from another thread
错误2：QObject::startTimer: Timers cannot be started from another thread
``` 
`Q1.` 为什么会触发这个槽函数后会报这样的错?  
`A1.` Qt是根据接收方来决定使用哪个连接字段的，当看到发送方和接受方属于同一线程后，则使用DirectConnection直接调用，当不处于同一线程后，就把函数调用请求封装成事件放到接收方线程中调用，而问题在于，接受方是this，它是属于主线程。

`Q1.1` 为什么属于主线程?  
`A1.1` 因为创建widget的时候是在主线程创建的，因此属于主线程，属于哪个线程是根据创建对象时所在的线程决定的，以后不会变，所以当this传到后台线程后，虽然内容被复制了，但依旧是主线程，而Qt就把我连接的槽函数发放到主线程执行了，在主线程操作后台进程的定时器导致了报错。

2. `Q.` 为什么获取当前后台进程的地址作为接收方连接槽函数这样依旧不行?  
`A.` 后台进程对象创建的时候是在主线程中创建的，但它管理的是另一个线程(后台进程)，QThread::currentThread()获取的是后台进程对象地址，而这个对象是在主线程创建的，所以会把事件放到主事件中执行，还是导致了跨线程操作定时器问题。

### 3.切换取反值问题
```cpp
//mainwidget.cpp

void Widget::mouseDoubleClickEvent(QMouseEvent* event)
{
    //通过先将标志位取反，然后再根据取反的值却去切换状态
    isRun = !isRun;
    if(isRun)
    {
        //双击切换成小猫走路效果
        //停止小猫站立呼吸定时器
        catIdleTimer.stop();

        //开启小猫奔跑移动定时器
        catRunTimer.start(SWITCH_RUN_IMAGE_TIEM);
    }   
    else
    {
        //双击切换成小猫站立呼吸效果
        //停止小猫走路定时器
        catRunTimer.stop();

        //开启小猫站立定时器
        catIdleTimer.start(SWITCH_IDLE_IMAGE_TIME);
    }
}
```
`Q.` 为什么取反后，取反的值就是这次要切换的状态呢?  
`A.` 拿电路开关的电流进行对比，isRun比作电流，切换的状态比作灯的状态，当我关灯时，我电流从有到消失，灯才能灭，那同理，我isRun状态从true到false，才能转换成站立状态，我得通过标志位确定它应处于哪种状态，才能切换至对应的状态。

## 设置功能
### 1.关于调用initSpinBoxRelatedAttributes函数(初始化spinBox相关属性)的问题
`Q.` 为什么要延迟调用？
```cpp    
QTimer::singleShot(10,[=](){
        initSpinBoxRelatedAttributes();
    });
```
`A.` 由于我在setWindow.cpp初始化spinBox属性的时候需要获取到图片宽高度，我希望在setWinodw.cpp通过信号获取widget.cpp的图片宽高值，因此我创立了连接，但由于这个连接是创建完SetWindow后才在widget.cpp连接的，导致创建获取图片的时候，是无法正确获取到宽高的，因为此时还没创建连接，那么我希望创建连接完后再调用initSpinBoxRelatedAttributes函数，那么我就使用QTimer进行延迟调用，确保连接完后再初始化控件的值。

### 2.关于spinBox最大值的问题(窗口左上角可设置点的坐标范围)
`Q1.` 为什么乘2?
```cpp
mainScreen->availableSize().width() * 2
```
`A1.` *2是为了让用户以物理分辨率的方式去设置并查看可用尺寸，用户输入后除以2就是逻辑分辨率。

`Q2.` 为什么减去图片宽度/高度的可用宽高度，怎么获取?  
`A2.` move是基于窗口左上角的点进行移动的，如果生成至x的2880，那么就会导致左上角的点是基于右边界绘制的，小猫就会在窗口的外边了
会导致一直翻转从而出不来，所以我需要减去图片的宽度来获取可用宽高度去确保图片右边是贴在右边界而不是左上角贴在右边界。
```cpp 
//statemanagement.cpp

/*                                  
为什么新位置到达左右边界时需要将窗口设置在各自的边界线上呢
就是因为之前新位置到达左右边界外时，我的做法不能移动，下次移动时直接反弹
这样造成的视觉效果就是还没完全碰到边界线时直接反弹，导致效果不是特别的好
我希望完全碰壁之后再进行反弹，所以写了判断语句
*/
if(isArriveBorder(windowPos)){//判断新位置是否到达了左右边界
        if(this->isRight){    //如果到达了边界，就看行驶方向
            windowPos.setX(this->mainScreenSize.width()- (emit requestGetWindowWidth()));//如果isRight为true，那么就说明一直往右走新位置到达了右边界外，需要将窗口的右边贴近右边界线
        }
        else{
            windowPos.setX(0);//如果isRight为false，说明一直往左边走新位置超出了左边界外，需要将位置设置在左边界线上
        }
}
```

`Q3.` 图片的宽高是按物理分辨率算还是逻辑分辨率?  
`A3.` 如果用物理分辨率算，那么用物理分辨率减去图片宽度再除以2是没问题的，但如果是按逻辑分辨率算减去图片宽度除以2就有问题了  
图片的宽高包括设置的宽高都是按照逻辑分辨率来进行计算的，我想获取减去图片宽度的可用的物理宽度范围，我可以先获取减去图片宽度的逻辑分辨率，再乘2就是减去图片宽度的可用的物理宽度范围了，也就是先获取减去图片宽度的逻辑分辨率，再乘2就能获取到减去图片宽度的物理可设置宽度。

### 3.move延迟调用问题
`Q1.` widget.cpp的move为什么要延迟调用
```cpp
//延迟调用readyStart，确保初始化完成
QTimer::singleShot(10,[this](){
    //为什么要延迟调用
    this->move(this->setWindowPtr->getWindowOriginalPosition());
    this->readyStart();
});
```
`A1.`因为初始化spinBox的函数是在widget构造函数后开始执行的，而如果在此之前移动位置，就无法将窗口移动至用户所选的坐标了，因为初始化spinBox的函数是决定了窗口生成的位置。

`Q1.1.` 但有时候setWindow的延迟调用函数比widget.cpp的延迟调用函数要慢，因此会造成将窗口设置在屏幕外的情况。  
`A1.1.` 将setWindow的延迟调用时间缩短，确保先执行setWindow的初始化spinBox属性函数在调用widget的move函数。
```cpp
//mainwidget.cpp

MainWidget::MainWidget(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::MainWidget)
{
    ui->setupUi(this);

    initVariable();       //初始化变量
    initConnect();        //初始化连接
    initWindowAttribute();//初始化窗口相关属性
    s.initVariable();     //先让mainwidget连接再初始化发射信号，防止信号发送时信号还没连接    
}
```

### 4.跨文件传递数据问题
`Q.` 怎么根据widget.cpp初始值设置setWindow.的下拉框的值?
`A.` 在setWindow定义信号，通过widget触发setWindow信号将值传过去即可。

### 5.点击确定按钮获取并设置属性开启定时器的bug(有确定按钮的情况下) 
```cpp
void Widget::restoreWindowMove()
{
    //先将设置属性，再恢复移动

    //获取速度值改变移动速度
    QString speedString = this->setWindowPtr->getSpeedComboxValue();
    this->imageRun.switchRunImageTime = speedString.toInt();

    //获取呼吸频率值改变呼吸频率
    QString breathString = this->setWindowPtr->getBreathComboxValue();
    this->imageIdle.switchIdleImageTime = breathString.toInt();
    if(isRun)
    {
        this->catRunTimer.start(this->imageRun.switchRunImageTime);//重新开启定时器设置速度
    }
    else
    {
        this->catIdleTimer.start(this->imageIdle.switchIdleImageTime);
    }
    this->isRun = true;
}
```
`Q.` 有个bug，当我右键时isRun改为false，如果处于走路状态后更新定时器时间时会导致开启了站立呼吸的定时器，两个定时器对同一个窗口进行切换图片操作，又因为下面，设置完属性后，isRun改为true，导致两个定时器同时对窗口移动了，就显得窗体移动的速度很快。  
`A.` 根据定时器的状态来决定更新哪个状态的定时器，如果走路状态的定时器处于活跃状态，说明切换的图片是走路的图片
则更新走路定时器，否则更新呼吸定时器。
```cpp 
if(this->catRunTimer.isActive())
{
    this->isRun = true;
    this->catRunTimer.start(this->imageRun.switchRunImageTime);//重新开启定时器设置速度
}
else
{
    this->catIdleTimer.start(this->imageIdle.switchIdleImageTime);
}
```

### 6. Q.不明确坐标转换规则，是逻辑分辨率还是物理分辨率? 
A. 对于程序而言使用的是逻辑尺寸，而对于用户而言使用的是物理尺寸，为什么这么做？物理尺寸是给用户看的，便于用户理解，而逻辑尺寸则便于程序对屏幕的操作。

## 开机自启动问题
### 1. Q1.怎么样让程序开机自启动?
`A1`.我想要让该程序开机自启动，那就要将该exe程序放到开机自启动配置中。

`Q1.1`. 怎么放？  
`A1.1` 首先利用QSettings找到注册表的开机自启动配置路径，然后在这个路径中将该程序名和程序路径作为键值放到该配置中
这样开机后，系统就能通过该路径启动exe程序。

### 2. Q2.为什么将程序名和程序路径添加至开机自启动配置后依然启动不了，而通过将快捷方式路径加入至配置中却可以?  
`A2.` 工作目录路径不同导致的加载资源异常从而启动程序失败。  
`A2.详细版原因：`
当我开机时，系统会在C:\Windows\System32去自启动应用，如果配置里配置的是exe程序路径，那么系统就会在C:\Windows\System32路径下通过你的路径找到exe程序后，在C:\Windows\System32为其创建新线程执行你的exe程序，同时工作目录被设置为C:\Windows\System32，又由于我资源文件是相对路径，导致加载资源时会在此路径下查找，但资源不在这个路径，所以查找不到无法启动程序。  
而如果配置的是快捷方式路径，系统会在C:\Windows\System32找到该快捷方式，但由于这是个快捷方式，它有着明确的工作目录，工作目录就是快捷方式指向的exe程序所在的目录路径，所以系统会通过快捷方式找到exe程序，并在exe程序所在目录下创建一个新线程，并将工作目录路径设置为该exe程序所在的目录路径，加载资源时就从该路径查找，就能查找到，从而启动程序。  
`A2.简略版原因：`
当系统自启动时，会在C:\Windows\System32调用exe程序，导致exe程序认为自己的当前位置是在C:\Windows\System32这个路径，从而寻找资源失败而导致启动失败。

`A2.` 解决方法
1. 路径指向快捷方式
2. 启动时将工作目录从C:\Windows\System32修改为程序所在目录
```cpp
//setwindow.cpp 

//获取程序所在目录路径设置为当前工作目录
QDir::setCurrent(QCoreApplication::applicationDirPath());
```

`Q2.1` 工作目录是什么？  
`A2.1` 那这引出一个概念，工作目录和程序所在目录的区别  
程序所在目录是你exe所在的目录路径，而工作目录是你调用exe程序时所在的目录路径，也就是我在C:\Windows\System32调用B目录的程序时，会在C:\Windows\System32创建一个新线程，同时将工作目录设置为你调用程序所在的目录路径(C:\Windows\System32)，此时应用路径被设置为了工作目录，导致资源加载不对。  

(工作目录是什么省略版)  
工作目录 vs 程序所在目录的区别​  
**程序所在目录：**exe文件物理存储的位置  
**工作目录：**程序运行时认为的"当前所在位置"    

`Q2.2` 工作目录的作用？  
`A2.2` 使用相对路径时，会优先使用工作目录作为路径

`Q2.3` 为什么在C:\Windows\System32(A目录)能执行我程序(B目录的程序)呢？  
`A2.3` 因为这是windows的机制，只是在A目录创建了一个新线程找到B目录路径后执行，程序所在的目录不一定就是运行时的目录路径

## 线程问题
### 1.定时器开启时后台进程结束问题
`Q.`   
现在有一个需求，我需要在后台进程播放动画的同时不造成界面卡顿，之前使用QThread会冻结界面，所以我使用了QTimer，但按我现在这种写法，QTimer发出的信号是在主线程执行的，我的代码是这样的：
```cpp
//延迟调用start，确保初始化完成
QTimer::singleShot(10,[this](){
    this->readyStart();//
    //定时器开启后，每100毫秒切换一张图片
    this->timer.start(100);
});
```
当我调用readyStart时，把start放到后台进程中执行，然后开启定时器，我的想法是可以在后台进程使用定时器触发start函数，可以保证界面不卡顿，但问题是在定时器发出timeout信号之前，我的后台进程就执行完毕了，因为没有循环，导致我的timeout信号都是在主线程执行的。  
补充一点原因：QTimer是在主线程创建的，即使后台进程没结束，依旧是在主线程触发信号并执行，依旧会冻结界面。

`A.` 解决办法  
将定时器的创建和连接放在后台进程中执行，并创建事件循环使其后台进程一直运行，这样定时器就能一直在后台进程中发出timeout信号并执行start函数了，并且需要创建事件循环，防止后台进程退出。

## 跨平台问题(Linux，Ubuntu)
### 1.无法设置窗口标志
`Q.` 在ubuntu不能置顶，无边框我认为也没设置成功，但是看不出来，我推测是在ubuntu下窗口的背景和边框都是白色，导致即使有边框看不出来。  
`A.` 跟使用的wayland协议有关，回退为x11协议即可。
```cpp
//main.cpp

int main(int argc, char *argv[])
{
#ifdef Q_OS_UNIX
    qputenv("QT_QPA_PLATFORM","xcb");//linux系统使用x11协议确保窗口正确移动
#endif
}
```

### 2.无法进行鼠标拖拽事件
`Q1.` 看到的现象：能够捕获鼠标事件，但是无法移动，使用move移动不了，我发现了一个离谱的点，宠物虽然在表面上没移动，但是能够触发边界检测，说明宠物确实移动了，但是不知道为什么没法显示，鼠标拖拽事件也能够正常拖拽，但是看不见效果。  
`A1.` 解决办法
1. 退回至x11协议，x11允许应用程序高度拥有对窗口的权限，因此调用move时，应用窗口就能自己命令自己直接移动至该坐标位置
```cpp
//main.cpp

int main(int argc, char *argv[])
{
#ifdef Q_OS_UNIX
    qputenv("QT_QPA_PLATFORM","xcb");//linux系统使用x11协议确保窗口正确移动
#endif
}
```
2. 使用 startSystemMove(推荐)，这是符合wayland的移动标准。

`Q1.1.` 为什么在高版本ubuntu使用move函数移动却移动不了?  
`A1.1.` ubuntu有两个协议，一个是x11，一个是wayland，它们都有一部分是负责窗口移动的，高版本默认使用的是wayland，而qt也对wayland进行了支持，但wayland处于安全性和现代性考虑，将全局坐标系统的概念去除，而Qt的GUI高度依赖该概念，导致了应用程序不能对窗口拥有高权限，每次移动时，都需要经过Wayland协议的检测和批准，但对于wayland协议来说，随意请求窗口移动的应用程序通常被定义为不好的，所以wayland通常会忽视这类请求，导致窗口无法移动。

`Q1.2` 既然无法移动，那为什么会触发边界检测?  
`A1.2` 窗口的坐标是在程序内部进行计算的，而不是wayland协议决定的，每切换一张图片坐标就会加加，但加到一定程度时，自然超出了屏幕范围，因此触发边界检测。

`Q1.3` wayland协议既然没有全局变量概念，又如何获取到光标位置呢?  
`A1.3` 准确来说Wayland协议为了安全性考虑，不向应用程序泄漏窗口的坐标，获取到的光标位置其实是基于应用程序内部获取的，而获取到的坐标又会被qt转化为全局坐标，但这个全局坐标的位置是模拟的，并不是绝对值，因此获取的只是模拟后的全局坐标。

### 3.开机自启动配置问题
`Q1.` window的自启动配置路径不适于linux，那怎么实现Linux开机自启动?  
`A1.`   
linux平台开机自启动配置：  
要想linux开机自启动程序，我得需要一个.desktop文件和autoStart目录，.desktop告诉了linux平台如何找到程序并启动，程序名称等相关信息，而autoStart类似于windows平台的自启动目录，我通过将.desktop放到linux的autoStart目录后，linux平台就会根据这个目录去寻找到该.desktop文件并正确启动。

```cpp
//setwindow.cpp

void SetWindow::setLinuxAutoStart(bool isChecked)
{
    //1.获取linux开机自启动路径
    QString autoStartPath = "/etc/xdg/autostart";
    //如果当前目录不存在，则递归创建
    QDir dir(autoStartPath);
    if(!dir.exists()){
        dir.mkdir(".");//mkpath(".")代表创建指向的当前目录(autoStart)，关键特性是会递归创建，如果某个父目录不存在，那么会将该目录创建出来
    }

    QString appName = QCoreApplication::applicationName();
    QString appPath = QDir::fromNativeSeparators(QCoreApplication::applicationFilePath());
    //2.编写程序开机自启动内容
    QString string = QString(
        "[Desktop Entry]\n"                 //桌面文件入口标准开头
        "Type=Application\n"                //程序类型
        "Name=%1\n"                         //程序名称
        "Exec=%2\n"                         //程序路径
        "Hidden=false\n"                    //是否在菜单中隐藏
        "NoDisplay=false\n"                 //是否在应用列表中不显示
        "X-GNOME-Autostart-enabled=true\n").arg(appName).arg(appPath);//GNOME桌面(ubuntu)特定的自启动启用标志

    //3.将该内容放到开机自启动路径下的.desktop文件中
    QFile file(autoStartPath + "/.desktop");
    if(isChecked){
        if(file.open(QIODevice::WriteOnly)){
            file.write(string.toUtf8());
            QString successText = QString("已启用开机自启动，路径：%1" ).arg(file.fileName());
            qDebug() << successText;
            this->logSystemPtr->writeLog(LogSystem::Info,successText);
        }
        else{
            QString errorText = "启用开机自启动失败！路径：" + file.fileName();
            qDebug() << errorText;
            this->logSystemPtr->writeLog(LogSystem::Error,errorText);
        }
    }
    else{
        if(file.remove()){
            QString successText = "已禁用开机自启动，路径：" + file.fileName();
            qDebug() << successText;
            this->logSystemPtr->writeLog(LogSystem::Info,successText);
        }
        else{
            QString errorText = "禁用开机自启动失败！路径：" + file.fileName();
            qDebug() << errorText;
            this->logSystemPtr->writeLog(LogSystem::Error,errorText);
        }
    }
}
```

`Q1.1` 那linux平台是怎么判断这个autoStart目录下.desktop文件就是需要自启动的应用程序呢?  
`A1.1` 这就关乎到一个概念，标准位置，通常来说，linux为了他统一各个发行版的配置目录，用户目录等系统级别的路径，就定义了一套标准
你放自己的配置文件，可以，但是你要在我指定了目录路径下放，这样我linux才能通过这些路径找到不同的系统级别路径从而去设置相关信息
而自启动路径就被设置在/home/用户/.config这个路径下，其目录文件就叫autoStart，我把.desktop文件放到autoStart里面
linux就能通过这个目录找到.desktop文件从而启动程序。
那实现思路就很明确了，首先找到配置路径下的autoStart目录，如果没有就创建一个，然后往这个目录下添加并编写.desktop文件
如果不需要开机自启动，则直接删除文件即可。
另外，.desktop文件放在不同目录下有着不同的表现，比如放在~/.local/share/applications/这个目录下，就会出现在开始菜单中
(~/.代表/home/用户名)。  

放在~/.config/autostart/，就能让程序自启动  
桌面图标 = .desktop文件放在桌面  
开始菜单项 = .desktop文件放在 applications目录  
开机启动项 = .desktop文件放在 autostart目录

标准路径的完整框架如下：
| 目录位置 | 作用  | 对应Windows概念 |
|-----------|------------|---------------------------|
| ~/.config/autostart/ | 用户开机自启动 |HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run  |
| /etc/xdg/autostart/ | 系统级开机自启动（所有用户）|HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run |
| ~/.local/share/applications/ | 用户应用菜单项               | \`%APPDATA%\Microsoft\Windows\Start Menu\Programs\`  |
| /usr/share/applications/     | 系统应用菜单项（所有用户）   |  \`C:\ProgramData\Microsoft\Windows\Start Menu\Program`|
| ~/Desktop/                   | 桌面快捷方式                 | \`%USERPROFILE%\Desktop\` |
| ~/Desktop/                   | 桌面快捷方式                 | \`%USERPROFILE%\Desktop\` |