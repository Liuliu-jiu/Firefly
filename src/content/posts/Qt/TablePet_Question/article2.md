---
title: 桌面宠物项目问题及解决方法（第2篇）
published: 2026-07-09
description: 分享我个人写桌面宠物时遇到的问题及解决方案
image: "../Article_Image/Qt.png"
tags: [Qt, 桌面宠物]
category: Qt
draft: false
---

# 为什么要发这篇文章
这是我个人写项目时的一些问题及，思考，
**旨在帮助大家能够更好的理解项目，同时避开一些坑。**  
小部分代码被优化过，所以在这篇文章中并不是和桌面宠物2.0代码完全相同。

# 桌面宠物项目链接
gitee链接：<https://gitee.com/liulijiu/project_repository/tree/master/%E9%A1%B9%E7%9B%AE%E4%BB%93%E5%BA%93/%E6%A1%8C%E9%9D%A2%E5%AE%A0%E7%89%A9/%E6%A1%8C%E9%9D%A2%E5%AE%A0%E7%89%A92.0>  

github链接：<https://github.com/Liuliu-jiu/TablePet>  
github访问不了下载[Watt Toolkit(原名steam++)](https://steampp.net/)加速

# 问题分类
## 行走功能
### Q1. 当原位置还距离窗口边界一小段距离时，就直接反方向走了
`A1.`这是因为移动窗口临近边界时，新位置已经超出边界，但原位置还距离边界一小段距离，此事就会直接反方向走，我希望窗口位置完全到达边界后才反方向移动

### Q2. 但为什么1.0版本会完全贴近甚至超出屏幕后才进行反方向移动呢?
`A2.` 1.0版本判断的是原位置有没有超出边界，而2.0版本判断是新位置有没有超出边界，所以导致了这种现象

```cpp
//左右边界检测
//左边界用左上角点检测，右边界用右上角点检测
if(windowPos.x() < 0)
{
    windowPos.setX(0);
}
else if(windowPos.x() + IMAGE_WIDTH > rect.width())
{
    windowPos.setX(rect.width() - IMAGE_WIDTH);
}

//上下边界检测
//上边界用左上角点检测，下边界用左下角点检测
if(windowPos.y() < 0)
{
    windowPos.setY(0);
}
else if(windowPos.y() + IMAGE_HEIGHT > rect.height())
{
    windowPos.setY(rect.height() - IMAGE_HEIGHT);
}
```

### Q3.我期望新位置到达边界外时，就把窗口贴在边界线上，下次移动时就直接反弹，效果更好。
`A3.`增加贴边检测机制，当新位置到达边界时，根据原行驶方向将窗体边框设置在对应的边界线上。
```
//statemanagement.cpp

/*
为什么新位置到达左右边界时需要将窗口设置在各自的边界线上呢
就是因为之前新位置到达左右边界外时，我的做法不能移动，下次移动时直接反弹
这样造成的视觉效果就是还没完全碰到边界线时直接反弹，导致效果不是特别的好
我希望完全碰壁之后再进行反弹，所以写了判断语句
*/
if(isArriveBorder(windowPos)){ //判断新位置是否到达了左右边界
    if(this->isRight){         //如果到达了边界，就看行驶方向
        windowPos.setX(this->mainScreenSize.width()- (emit requestGetWindowWidth()));//如果isRight为true，那么就说明一直往右走新位置到达了右边界外，需要将窗口的右边贴近右边界线
    }
    else{
        windowPos.setX(0);//如果isRight为false，说明一直往左边走新位置超出了左边界外，需要将位置设置在左边界线上
    }
}
```

## 受伤功能
### Q. 现在有个问题，我改变受伤选项状态后无法立即获取到该状态，因为受伤选项是在主窗口类，而受伤状态的触发是在状态管理类，状态管理类无法包含主窗口类，因此不能在主窗口类发送信号至状态管理类，因此无法到该状态。
`A.`既然无法改变受伤选项状态时在该状态管理类立即获取到选项状态，那么我可以在边界检测时，获取该选项状态从而决定是否要触发受伤状态。

## 日志系统类问题
### Q1.为什么不使用std::make_shared()来开辟空间呢？
`A1.`构造函数是私有的，std::make_shared()无法调用私有的构造函数，我希望构造函数私有的同时能够交给智能指针管理，而只有new才能调用私有构造，因此我手动使用new来开辟空间后，主动将这块空间交给std::shared_ptr管理，std::shared_ptr<LogSystem>是创建出指向LogSystem地址的智能指针，而(new LogSystem())则是指向的地址，所以这行代码的意思是，创建一个指向LogSystem的智能指针后，将new出来的空间交给智能指针管理。

```cpp
//logsystem.cpp

if(logSystemPtr == nullptr){//如果未开辟对应的空间
    logSystemPtr = std::shared_ptr<LogSystem>(new LogSystem());//使用new开辟空间，将这块空间交给智能指针管理
}
return logSystemPtr;
```

### Q2.日志文件存在检测机制是否需要写？
`A2.` AI给我的建议是需要添加。  
1. 不同时间段的日志混合会导致可读性差。
2. 当使用已存在的文件时，这个文件可能本来就很大超出100mb，继续使用越来越大，从而不受100mb检测机制的管理，会导致失控。  
所以这两点仍需要被添加。

### Q2.1 什么时候需要该机制？
`A2.1` 当索引加1后的重名命的文件是存在时，需要该机制找到不存在文件的名称，本意是为了防止覆盖日志文件而准备的，但问题是即使日志文件存在，也不会被覆盖，而是会继续打开使用。

### Q2.2 那么还有要写的必要吗
`A2.2` 写了可以当超出100mb之后，找到一个全新的日志文件，也就是说，写了后当超出100mb后找到一个新的日志文件使用，不写则新旧混合使用。

### Q3. 我是怎么解决多位数索引的问题的?
`A3.`   
之前有个严重的逻辑缺陷，我之前的做法是将字符索引传过去，然后字符9加1时不是变10而是变成冒号，导致冒号成了索引，但我希望9+1变10，让10成为索引，因此我把字符9传过去时转为整数，然后再加1，这样9就变成了10，然后转换为字符串附加在日志文件末尾就可以解决多位数索引问题，其实就是转为整数后加，而不是在字符类型时加加。
```cpp
//logsystem.cpp

//通过索引递增寻找未创建的日志文件
QString tempPath;
do{
    //先转换为整数递增后加附加至字符串末尾，防止出现字符9加到字符冒号的问题出现
    tempPath = QCoreApplication::applicationDirPath() + "/" +QCoreApplication::applicationName() +"_LogFile_"+ QString::number(index++) + ".log";
}while(QFile(tempPath).exists());
return tempPath;
```

### Q3.1 多位数索引问题解决了，但传索引时会有个问题，按我之前的写法，我传的是不带后缀的日志文件名的最后一个字符，本意是想传索引过去，但是当多位数索引时，这种就会发生错误，比如TablePet2_LogFile_10，如果只传最后一个字符，那就导致只传了0，没有传到完整的数字，就会出错。
`A3.1` 我的日志名是通过下划线隔开各个部分的，而索引在下划线的最后一个部分，因此我以下划线切割字符串，拿到最后一个部分(索引)
这样一来不管是单位还是多位，都能正确传过去。
```cpp
//logsystem.cpp

//检查大小是否超过100mb
QFileInfo fileInfo(logFilePath);
if(fileInfo.size() < 100 * MB){
    //更换日志文件(利用日志文件存在检测机制找到全新的日志文件，并将带后缀的路径返回来)
    logFilePath = logFileExistsMechanism(fileInfo.baseName().split("_").back().toInt());     //切割字符串拿到索引部分的值
    qDebug() << "更换文件后的路径：" << logFilePath;
}
```

## 性能窗口问题
### Q1. 为什么创建线程后显示窗口会报这些错？(当时只有进度条类，编译器为这个类单独创建了一个widget)
`A1.`  
> 错误信息：QObject: Cannot create children for a parent that is in a different thread.(Parent is QMenu(0x1d671579110), parent's thread is QThread(0x1d66fd96a70), current thread is QThreadPoolThread(0x1d672647cd0)
```cpp 
QFuture<void > future = QtConcurrent::run([=](){
            QEventLoop loop;
            this->memoryProgressBarPtr->show();
            loop.exec();
        });
```

### Q2. 性能窗口移动问题
`Q2.1` 希望的效果：性能窗口的中心部分显示在桌面宠物正下方中心。  
`A2.1` 性能窗口的宽度是远长于桌面宠物的宽度的，如果要使得性能窗口中心部分显示在桌面宠物的正下方，就要计算性能窗口x坐标和桌面宠物x坐标相差的值。

`Q2.2` 为什么要进行计算这个值呢?  
`A2.2` 如果处于正下方，那么性能窗口的左上角的点和桌面宠物左上角的点在左边有一段距离，我需要计算出这段距离，然后利用当前x坐标减去这段距离就能得到性能窗口应该要设置的x坐标。

`Q2.3` 怎么计算性能窗口的坐标呢?  
`A2.3` 我们假设性能窗口和桌面宠物的左上角点之间的距离设置为x，由于性能窗口中心部分处于桌面宠物下方，因此除中间部分外，两边的距离一样的，因此另外一边也是x，而中间部分的值就是桌面宠物窗口的宽度，因此列成的公式就是2x + 桌面宠物宽度 = 性能窗口宽度，把x解出来就能得到之间相差的距离了，怎么解，(性能窗口宽度-桌面宠物宽度)/2 = x，然后再拿桌面宠物的x坐标减去这个x就能得到性能窗口的x坐标，就可以使得性能窗口处于中心位置。
```cpp
//mainWidget.cpp

QPoint MainWidget::calculatePerformanceWindowPos(QPoint mainWidgetPos)
{
    //计算性能窗口的坐标，使其性能窗口中心部分处于桌面宠物下方
    QPoint performanceWindowPos = mainWidgetPos;
    int value = (this->performanceWindow.getWindowWidth()-this->width())/2;
    performanceWindowPos.setX(mainWidgetPos.x() - value);
    performanceWindowPos.setY(mainWidgetPos.y() + this->height());
    return performanceWindowPos;
}
```

### Q3. 当主窗口关闭时，性能窗口由于忽略关闭事件因此无法关闭。
当关闭时，我忽略的关闭事件导致无法关闭性能窗口，希望的效果：当主窗口关闭时，性能窗口也跟着关闭，但当窗口未关闭时，性能窗口无法关闭。
`A3.` 利用标志位和信号来关闭窗口，当主窗口关闭时，发送性能窗口关闭信号，同时将标志位改为true当为true时，则响应关闭事件，否则忽略关闭事件。
```cpp
//performanceWindow.cpp

void PerformanceWindow::closeEvent(QCloseEvent* event)
{
    if(this->windowIsClose){
        QWidget::closeEvent(event);     //如果标志位为true，则响应关闭事件
    }
    else{
        event->ignore();                //忽略关闭事件，防止性能窗口意外关闭
    }
}
```

### Q4.1 如何获取内存情况?
`A4.1.` 可以通过GlobalMemoryStatusEx获取，获取前先声明MEMORYSTATUSEX 结构体，这是用来存储物理和虚拟内存情况的结构体，并且将dwLength 初始化为当前结构体的大小。
```cpp
memoryStatus.dwLength = sizeof(memoryStatus);
```
将MEMORYSTATUSEX当作GlobalMemoryStatusEx参数传过去，传过去后，就可以获取了，可以通过返回值判断是否获取成功，通过结构体的变量得知内存情况，memoryStatus.ullTotalPhys为物理内存的总字节数，emoryStatus.ullAvailPhys为物理内存的可用字节数。
```cpp
//performanceWindow.cpp

void PerformanceWindow::getWindowMemory(int& usedPresent,double& used)
{
    MEMORYSTATUSEX memoryStatusEx;                              //声明结构体，用来存储物理和虚拟内存情况的结构体
    memoryStatusEx.dwLength = sizeof(memoryStatusEx);           //将dwLength初始化为结构体大小，用来确认结构体版本，防止写入错误
    if(GlobalMemoryStatusEx(&memoryStatusEx)){
        double total = memoryStatusEx.ullTotalPhys / (1.0*GB);  //总共物理空间，转换为GB单位存储在变量中
        double avail = memoryStatusEx.ullAvailPhys / (1.0*GB);  //可用物理空间
        used = total-avail;                                     //已用物理空间
        usedPresent = used/total * 100;                         //已用物理空间百分比

        qDebug() << "------------------";
        qDebug() << "总共物理内存：" << total;
        qDebug() << "已用物理内存：" << avail;
        qDebug() << "可用物理内存：" << used;
        qDebug() << "------------------";
    }
    else{
        qDebug() << "Get memory is fail！";
    }
}
```

### Q4.2 将dwLength 初始化为当前结构体的大小memoryStatus.dwLength = sizeof(memoryStatus); 为什么这么做?
`A4.2` 这是因为要确认程序所使用的结构体版本，如果版本不一致，就会可能导致写入错误。

### Q5. 怎么使得CPU进度条一秒更新一次?
`A5.` 开启定时器前采集一次CPU状态时间，然后存储到第一次变量中，作为第一次采样，发出timeout信号后再采集一次，存储到第二次变量中，作为第二采样，然后与第一次进行对比计算出增量，计算完将第二次采集的数据放到第一次变量中，作为下次对比的第一次采样，1秒后再次发出timeout信号，然后采集数据放到第二次变量中，作为对比的第二次采样，再与前一次的第二次数据(现在存储到第一次变量中)进行对比计算，周而复始，就能1秒更新一次，发出timeout信号时，与上一次值进行对比。
```cpp
//performanceWindow.cpp

connect(&this->memoryTimer,&QTimer::timeout,this,[=](){
    int present = 0;//已使用的百分比
    double used = 0;//已使用的字节数(GB)
#ifdef Q_OS_WIN
    getWindowMemory(present,used);
#elif defined(Q_OS_UNIX)
    getLinuxMemory(present,used);
#endif
    ui->memoryProgressBar->setValue(present);
    ui->showMemoryByteLabel->setText(QString(" 已用：%1G").arg(used,0,'f',2));//arg参数中，0代表总字段宽度，'f'代表定点表示法，4代表只保留4位小数
});

void PerformanceWindow::responseWindowShow(QPoint windowPos)
{
    //CPU定时器开启前采样一次，作位第一次采样的数据
#ifdef Q_OS_WIN
    getCPUStatusTime(&this->fristIdleTime,&this->fristKernelTime,&this->fristUserTime);
#elif defined(Q_OS_UNIX)
    getLinuxCPUTimes(fristTotal,fristTotalIdle);
#endif
    this->cpuTimer.start(1000);     //每1秒更新一次CPU使用情况
    this->windowIsClose = false;
    this->memoryTimer.start(1000);  //每一秒更新一次内存情况
    this->move(windowPos);          //将窗口移动至指定的位置
    this->show();                   //再进行显示
}

void PerformanceWindow::getCPUStatusTime(ULONGLONG* longIdleTime,ULONGLONG* longKernelTime,ULONGLONG* longUserTime)
{
    _FILETIME idleTime,kernelTime,userTime;//定义第一次能存储空闲时间，内核时间，用户时间的结构体
    if(GetSystemTimes(&idleTime,&kernelTime,&userTime)){
        *longIdleTime = concatenatedTimeDigits(idleTime.dwLowDateTime,idleTime.dwHighDateTime);//拼接成可运算的时间，然后通过参数带回值
        *longKernelTime = concatenatedTimeDigits(kernelTime.dwLowDateTime,kernelTime.dwHighDateTime);
        *longUserTime = concatenatedTimeDigits(userTime.dwLowDateTime,userTime.dwHighDateTime);
    }
    else{
        qDebug() << "Get cpu status is fail！";
    }
}
```


`Q5.1` 怎么计算总时间?  
`A5.1` 我要计算总时间，其实就是内核时间+用户时间+空闲时间的总和，单由于内核时间包含了空闲时间，因此计算总时间时只需要内核时间(包括空闲时间)+用户时间=总时间，但计算总时间时，我不能直接将两次时间直接相加，因为我们要获取的是在制定时间间隔内的总时间，因此，先计算各个时间的增量，然后在相加就能得到两次之间的总时间，然后利用总时间-空闲时间就能得到非空闲时间。
```cpp
//performanceWindow.cpp

qreal PerformanceWindow::calculateWindowsCpuUsage()
{
    //计算使用率时，程序设置的时间间隔和实际系统所使用的时间间隔不同，就会导致展示的使用率与系统的使用率略有差异
    getCPUStatusTime(&this->sencondIdleTime,&this->sencondKernelTime,&this->sencondUserTime);   //第二次采集数据
    ULONGLONG idle = this->sencondIdleTime - this->fristIdleTime;                               //通过第二次数据减去第一次数据得到1秒内空闲时间
    ULONGLONG kernel = this->sencondKernelTime - this->fristKernelTime;                         //1秒内内核时间
    ULONGLONG user = this->sencondUserTime - this->fristUserTime;                               //1秒内用户时间
    ULONGLONG total = kernel + user;                                                            //1秒内总时间
    ULONGLONG work = total - idle;                                                              //1秒内非空闲时间

    qreal usedPresend = (double)work/total*100;                                                 //非空闲时间在1秒内的占比

    this->fristIdleTime = this->sencondIdleTime;                                                //将这一次第二次采样的数据放到第一次数据的变量中，作为下次对比的第一次采样数据
    this->fristKernelTime = this->sencondKernelTime;
    this->fristUserTime = this->sencondUserTime;
    return usedPresend;
}
```

### Q6. 怎么获取linux的内存情况?
`A6.`通过读取/proc/meminfo文件获取，因为在Linux中，使用了meminfo虚拟文件存储内存相关情况，因此我只需要读取然后解析就能获取到了
读取的时候我需要了解meminfo的文件内部结构是什么样才能读取，meminfo文件的每一行都是内存某个具体方面的使用情况，比如第一行是内存总和，第二行是可用空间，第三行是缓冲区使用情况，而每一行都分为两部分，第一部分是具体方面的名字，第二个部分是具体方面的字节数，比如内存总和，`MemTotal: 8028124 kB`，一行就包含了内存总和的名字及字节数，中间通过冒号隔，要想正确读取出来，就需要一行行读取，读取到一行时，就需要以冒号分割字符串，以便于将字节数存储至正确的变量中。

/proc/meminfo文件内容解析
| 变量名       | 大小       | 名称         |
|--------------|------------|--------------|
| MemTotal     | 8028124 kB | 总内存       |
| MemFree      | 234568 kB  | 空闲内存     |
| MemAvailable | 3456789 kB | 可用内存     |
| Buffers      | 123456 kB  | 块设备缓冲区 |
| Cached       | 2345678 kB | 页面缓存     |
| SwapCached   | 0 kB       | 交换区缓存   |
| Active       | 3456789 kB | 活跃内存     |
| Inactive     | 1234567 kB | 非活跃内存   |

### Q7.为什么计算已用空间时，要把缓存和缓冲区内存减去呢，不也是内存使用的一部分吗?
```cpp
    double used = static_cast<double>(memTotal - memAvailable);
```
`A7.`因为缓存和缓冲区的即时回收性，当一个程序需要占用资源时，linux自动会将一部分缓存释放从而让给这个程序，而让出来的这部分程序就可以用，不管是什么需要占用资源时会自动释放缓存，既然谁都可以占用缓存资源，那我们就将这些资源视作可用空间了。

### Q8. 读取虚拟文件meminfo时即使有内容atEnd方法也会返回true?
QFile读取的是真实文件，读虚拟文件时可能会导致指针位置不对，从而判断错误，即使调整指针位置也没用  
`A8.` 先将所有内容读取出来，然后按"\n"分割字符串，最后遍历，

`Q8.1` 为什么按"\n"分割呢?  
`A8.1` 内存的情况是按一行行排列的，我按"\n"分割，就相当于把内存情况的具体部分都分割了出来。

### Q9.怎么获取linux的CPU使用率情况?
`A9.`获取CPU使用率通常是读取/proc/stat文件所获取，stat是linux内核用于存储CPU各个时间值的虚拟文件，所以我只需要读取该文件的数据然后解析就能得到CPU使用率。
怎么解析呢？  
首先我们要知道stat文件结构是什么样的，第一行是全部CPU的各个时间的时间总和，第二行及以下就是单个CPU的各个时间值。

示例文件内容
| <!-- -->   | <!-- -->  | <!-- --> | <!-- -->  | <!-- -->| <!-- -->  | <!-- --> | <!-- --> | <!-- -->| <!-- --> | <!-- -->    |
|-----------|------------|------|-------|--------|------|-----|------|-----|-----|-----|
| cpu       | 123456     | 7890 | 12345 | 678901 | 2345 | 678 | 9012 | 0   | 0   | 0   |
| cpu1      | 12345      | 678  | 901   | 23456  | 789  | 123 | 456  | 0   | 0   | 0   |
| cpu0      | 12345      | 678  | 901   | 23456  | 789  | 123 | 456  | 0   | 0   | 0   |
| cpu2      | 12345      | 678  | 901   | 23456  | 789  | 123 | 456  | 0   | 0   | 0   |
| cpu3      | 12345      | 678  | 901   | 23456  | 789  | 123 | 456  | 0   | 0   | 0   |
| intr      | 123456789  | ...  | ...   | ...    | ...  | ... | ...  | ... | ... | ... |
| ctxt      | 12345678   | ...  | ...   | ...    | ...  | ... | ...  | ... | ... | ... |
| btime     | 1234567890 | ...  | ...   | ...    | ...  | ... | ...  | ... | ... | ... |
| processes | 12345      | ...  | ...   | ...    | ...  | ... | ...  | ... | ... | ... |
| ...       | ...        | ...  | ...   | ...    | ...  | ... | ...  | ... | ... | ... |

而这些字段分别代表(第1到第8列，从0开始)
| 字段    | 说明               | 计算示例                       |
|---------|--------------------|--------------------------------|
| user    | 用户态时间         | 普通应用程序运行时间           |
| nice    | 低优先级用户态时间 | nice命令调整优先级的进程时间   |
| system  | 内核态时间         | 系统调用、内核任务时间         |
| idle    | 空闲时间           | CPU完全空闲时间                |
| iowait  | I/O等待时间        | 等待磁盘/网络I/O的时间         |
| irq     | 硬中断时间         | 处理硬件中断的时间             |
| softirq | 软中断时间         | 处理软件中断的时间             |
| steal   | 被偷走的时间       | 虚拟化环境中被宿主机占用的时间 |

CPU总时间就是这8个字段的和，CPU总空闲时间就是空闲时间+i/0等待的时间，所以要计算CPU已用时间，就是将8个字段值加起来后减去(空闲时间+i/o等待的时间)，我们计算CPU使用率时，计算的是瞬时值，意思是计算固定时间段的工作占比，也就是计算1秒内工作的占比为多少，因此需要采样两次，中间隔一秒，用后一次的时间减去前一次的时间得出增量值(固定时间)，查看这固定时间内工作占比为多少。

## 显示字体窗口问题
### Q.为什么播放完受伤状态后立即再次播放会导致字体没有显示?(第一次播放完后第二次立即播放时"hp-1"字样未显示)
`A.` 字体播放的时长远小于受伤状态播放的时长，当受伤状态播放完成后，此时字体动画还没播放完，如果再次触发受伤状态，就会在第一次播放字体动画的途中播放第二次受伤状态，这就是因为为什么立即触发时没法显示，因为动画还没播放完就触发第二次受伤状态了。

## 预加载问题
### Q1.预加载类要不要使用单例模式呢?
`A1.`   
- 使用单例模式时
    - 好处  
        能够资源共享，A文件能够使用B文件加载的图片资源。
    - 缺点：
        1. 耦合度较高。
        2. 加载太多不知道图片是从哪里加载的。

- 不使用单例模式时
    - 优点  
        每个源文件都独自享有自己的资源，耦合度较低。
    - 缺点
        1. 空间占用可能会比单例模式大一点。
        2. 如果两个文件都需要那张图片，则需重复加载。

我打算采用不使用单例模式，为什么？  
原因：  
1. 希望耦合度低。
2. 目前除主窗口类，其它文件并不需要用到图片资源。
3. 算以后扩展，我估摸着就只有设置窗口可能会用到，但跟主窗口类用到的并不是一样，也就是说各个窗口之间使用的图片资源应该是不一样的。

### Q2.我需要加载两个方向的图片，该怎么做?
`A2.`用两个容器存储，调用loadPixmap时，不仅加载左边，也加载右边。
```cpp
//preload.cpp

//哈希容器说明：<状态,<图片索引，图片对象>>，为了统一管理模型的状态，图片路径，图片对象之间的关系
QHash<QString,QHash<int,QPixmap>> pixmapHash;      //存储图片的容器
QHash<QString,QHash<int,QPixmap>> mirrorPixmapHash;//存储翻转后的图片容器的
void Preload::loadPixmap(QString status,QStringList pixmapPathList)
{
    QHash<QString,QHash<int,QPixmap>>::Iterator it = this->pixmapHash.find(status); //首先查看该状态是否已经加载至容器中
    if(it == this->pixmapHash.end()){                                               //未加载则将状态，图片路径和图片对象关联后加载至容器中
        QHash<int,QPixmap> hash;                                                    //存储原图片的容器
        QHash<int,QPixmap> mirrorHash;                                              //存储翻转图片的容器
        int i = 0;                                                                  //整数作为图片对象的键名，方便查找
        foreach(QString pixmapPath,pixmapPathList){
            QPixmap pixmap(pixmapPath);
            hash.insert(i,pixmap);                                                  //将图片路径和图片对象关联后放入至容器中存储
            mirrorHash.insert(i,QPixmap::fromImage(pixmap.toImage().mirrored(true,false)));//关联图片路径和水平翻转后的图片对象
            i++;
            qDebug() << "预加载的图片资源：" << pixmapPath;
            if(pixmap.isNull()){
                emit requestPromatQPixmapIsNull();
                qDebug() << "图片为空！路径" << pixmapPath;
            }
        }
        this->pixmapHash.insert(status,hash);                                       //关联状态和原容器
        this->mirrorPixmapHash.insert(status,mirrorHash);                           //关联状态和翻转容器
    }
}
```

`Q2.1` 那为什么需要两个容器存储呢？  
`A2.1` 因为如果左右方向都存储在一个容器中，会导致找不到我所需要方向的图片路径，两方向的路径是一样的，因此需要按方向分开存储有利用查找，我希望的效果就是我可以自由指定这个图片是否需要翻转，如果不翻转则默认存储在左容器中(原容器中)，如果翻转则不仅存储在左容器，也存储在右容器(存储翻转后图片的容器中)，这样就比较清晰明了，同时也为获取不同方向图片打下基础，而获取图片的时候我希望也能通过参数获取指定方向的图片。

`Q2.1.1` 那为什么要这么做呢?  
`A2.1.1 `就是因为后续的图片如果不需要翻转则直接存储至左容器中，获取的时候只需要获取左容器的图片就行，比较方便。

### Q3.Preload::getImageAndPathHash为什么返回哈希容器?
```cpp
//preload.cpp

QHash<int,QPixmap> Preload::getImageAndPathHash(QString status,bool isGetMirrored)
{ 
    return isGetMirrored == true ? this->mirrorPixmapHash.value(status) : this->pixmapHash.value(status);//根据布尔值选择要返回的哈希容器
}
```
`A3.`如果返回图片对象，那么就需要知道图片路径，但当前做法是在切换图片时是在状态管理类切换的，而我是在设置窗口类读取配置文件，因此状态管理类是不知道图片路径的，所以我只能将关联路径与对象的哈希容器作一个返回，然后遍历哈希内容进行一个图片切换。

## Qt常见问题
### Q.为什么不推荐用信号返回值?
`A.`原因如下；  
1. 可读性差：通常来说信号只是通知一个事件的发生，不期望返回值，这种写法可能会另其他开发者感到困惑。
2. 灵活性差：如果是跨线程操作，那么这种写法将会失效，使用队列连接跨线程工作时，事件的发生是在接收方发生的，而发送方和接收方的线程不同，是无法获取返回值的。

## 配置类
### Q.我怎么在json文件中修改无限深度的嵌套值?
`A.`我希望能够遍历json文件的所有键，但json文件会出现嵌套键值的情况，因此单纯利用for循环遍历是行不通的，而嵌套值的情况是出现在一个对象当中的，因此遍历键时，如果遇到了个对象，证明里面有嵌套值，就应该往里面遍历，但遍历完并不知道从哪个位置继续往下遍历，因此用递归来记录位置，当遇到一个对象时，就递归一次，往更深层遍历，遍历完后归时，就能在刚刚的位置继续往下遍历，当修改完键时，我就在归时逐一更新之前的对象值。

`简易版：`  
怎么修改嵌套的值呢，一层层遍历，如果发现是我需要修改的键，就停下来，如果不是，则一直遍历，如果有对象，那么就到对象里面遍历。

```cpp
//configfile.cpp

QJsonObject ConfigFile::updateConfigValue(QJsonObject jsonObject,QString updateKey,QVariant value,JsonType jsonType,bool& isOk)
{
    //一次递归视作一个对象，在这个对象下遍历所有键，如果与我的键相符，则修改对应的值，然后往前更新，返回QJsonObject
    //如果该键是一个object对象，则进入到这个对象遍历，如果没有则退出来继续往下遍历
    QJsonObject newObject;
    foreach(QString key ,jsonObject.keys()){
        //遍历过程中发现是个对象则继续递归遍历
        if(jsonObject.value(key).isObject()){
            newObject = updateConfigValue(jsonObject.value(key).toObject(),updateKey,value,jsonType,isOk);
            //当函数返回时，当找到该键并更新时，就将该对象的值进行一个更新，更新完后返回
            if(isOk){
                jsonObject[key] = newObject;
                break;
            }
        }
        else{
            //当找到需要更新的键时，就按照传入类型转换成对应的类型更新对应的值
            //然后退出循环将当前对象进行返回
            if(key.contains(updateKey)){
                switch(jsonType){
                case Int:
                    jsonObject[key] = value.toInt();
                    break;
                case String:
                    jsonObject[key] = value.toString();
                    break;
                }
                isOk = true;
                break;
            }
        }
    }
    return jsonObject;
}
```

## 设置窗口类
### Q1.我怎么知道用户点击了哪个模型单选框?
`A1.`让所有radioButton连接同一个槽函数，当某个按钮点击时，我就将该单选框地址传过来，然后根据文本选择加载的配置文件。
```cpp
//setwindow.cpp

connect(button,&QRadioButton::toggled,this,[=](bool isChecked){
if(isChecked){
    bool isOk;
    configFile.writeConfig(APP_CONFIG_JSON_FILE,"userSelectedModelName",button->text(),ConfigFile::String,isOk);
    if(!isOk){
        qWarning() << "userSelectedModelName更新失败！模型名：" << button->text();
    }
    loadModelConfig(button);
    loadOtherSet();//根据模型更新最初可生成的位置
}
});
```
### Q2.通过宏定义决定json文件的目录路径
```cpp
//setwindow.cpp

#ifdef Qt_NO_DEBUG_OUTPUT
    //模型JSON文件的目录路径
    #define MODEL_JSON_FILE_DIRETORY_PATH QCoreApplication::applicationDirPath()

    //项目配置文件路径
    #define APP_CONFIG_JSON_FILE QCoreApplication::applicationDirPath() + "/appConfig.json"

#else
    //模型JSON文件的目录路径
    #define MODEL_JSON_FILE_DIRETORY_PATH QFileInfo(__FILE__).absoluteDir().absolutePath()

    //项目配置文件路径
    #define APP_CONFIG_JSON_FILE QFileInfo(__FILE__).absoluteDir().absolutePath() + "/appConfig.json"
#endif
```
`Q2.1`为什么这么分？  
`A2.1`就是因为调试版和发行版的json文件路径不同，调试版的json文件在源文件目录下，而发行版则在exe程序目录下，如果不分版本，就会导致有一个版本找不到json文件
例如只使用调试版本的json路径，此时json目录路径是在源文件下的，而发行版使用了调试版的json目录路径，但在发行版中，是找不到该路径的，因此就找不到json文件从而导致加载资源失败。

## 自定义配置界面
### Q1.为什么要去除首尾中括号?
```cpp
//custommodelwindow.cpp

QString CustomModelWindow::concatenateFilePaths(QStringList filePathList)
{
    QString result;
    //将多条路径拼接成一个字符串，路径与路径之间用逗号隔开
    for(int i = 0;i < filePathList.size();i++){
        result += ("\"" + filePathList[i]+ "\"");       //图片路径的首尾加双引号代表在json文件中这是一个字符串
        if(i < (filePathList.size() - 1)){              //当来到最后一个字符串时，末尾不需要加逗号
            result += ",\n";                            //加\n是为了在label上显示时更好看，写入json文件加不加都可以
        }
    }
    result.insert(0,"[");                               //在字符串首尾加左右中括号表示这是一个数组类型
    result.insert(result.size(),"]");
    return result;
}
//读取行走状态图片
QStringList walkImageList = configFile.readConfigValue(jsonPath,"walkImagePath").toStringList();
walkImagePath = concatenateFilePaths(walkImageList);
setFilePathsToLabel(walkImagePath,ui->walkImageLabel);
```
`A1.`通过concatenateFilePaths拼接后，返回来的字符串是符合json格式的，因此可以直接插入json格式字符串中，但问题在于，该字符串不仅要写入，而且要放到界面上看，而界面又有中括号，字符串也有中括号，如果直接放会导致有两对中括号，而我只希望只有界面有中括号，字符串没有，因此我将返回过来的值通过setFilePathsToLabel去除首尾并放到label上显示。

### Q2.我怎么设置spinbox的最大值?
`A2.`我们知道，spinbox设置的是宠物的宽高，而我希望宠物不超过可用部分的4分之一，因此获取可用部分后，将高除2，宽除2。
```cpp
//custommodelwindow.cpp

void CustomModelWindow::initSpinBox()
{
    //初始化spinbox的最小，最大值
    QSize availableSize = QApplication::primaryScreen()->availableSize();
    ui->modelWidthSpinBox->setRange(0,availableSize.width()*2);
    ui->modelHeightSpinBox->setRange(0,availableSize.height()*2);
    ui->maxWidthLabel->setText(QString("最大：%1").arg(availableSize.width()*2));    //显示能设置的最大宽高，最大不能超过屏幕的4分之一
    ui->maxHeightLabel->setText(QString("最大：%1").arg(availableSize.height()*2));
}
```
### Q3.为什么能直接将spinbox最大值设置为可用部分的宽高呢，不是要除以2缩小至4分之一吗?
```cpp
//custommodelwindow.cpp

void CustomModelWindow::initSpinBox()
{
    //初始化spinbox的最小，最大值
    QSize availableSize = QApplication::primaryScreen()->availableSize();
    ui->modelWidthSpinBox->setRange(0,availableSize.width()*2);
    ui->modelHeightSpinBox->setRange(0,availableSize.height()*2);
    ui->maxWidthLabel->setText(QString("最大：%1").arg(availableSize.width()*2));    //显示能设置的最大宽高，最大不能超过屏幕的4分之一
    ui->maxHeightLabel->setText(QString("最大：%1").arg(availableSize.height()*2));
}
```
`A3.`因为程序获取的是逻辑分辨率，又因为显示在界面和文件中的宽高是物理分辨率，即使，不显示除以2，由于坐标系统的改变，从逻辑分辨率转为物理分辨率，值是没动的，概念是变了，相当于除以2了，其实就是写入文件时概念的转变，将逻辑分辨率的值当作物理分辨率了，所以不需要除以2。

### Q4.为什么要创建一个通用的提示错误方法?
`A4.`如果直接使用Qt给的静态方法对话框，由于子类会继承父类，就会导致子类会继承父类的颜色风格，导致不美观，为了解决这个问题，我单独创建一个对话框，又由于有许多地方需要提示错误，因此我创建一个通用的方法来进行提示。

### Q5.当修改后的模型名与读取的模型名不一致时，无法删除旧模型文件的问题。
```cpp
//custommodelwindow.cpp

void CustomModelWindow::on_editModelFilePushButton_clicked()
{
    //选取json文件
    QString jsonPath = QFileDialog::getOpenFileName(this,"获取模型文件",MODEL_JSON_FILE_DIRETORY_PATH,"*.json");
    if(jsonPath == APP_CONFIG_JSON_FILE){//当用户选择的是项目文件时，我就让用户选择正确的模型文件
        promptError("文件选择错误！请选择正确的模型文件");
        on_editModelFilePushButton_clicked();
        return;
    }
    if(!jsonPath.isEmpty()){
        readFilePath = jsonPath;//记录读取json路径
    }
}
void CustomModelWindow::updateModelFile()
{
    //如果读取的路径和我生成时拼接的路径不一致，就代表模型名不一致，就将旧模型文件删除，防止新旧模型混合
    if(!readFilePath.isEmpty() && readFilePath != file.fileName()){
        QFile(readFilePath).remove();
        qDebug() << "模型名不一致！删除  " << readFilePath << "  旧模型文件";
        readFilePath = file.fileName();         //记录此次模型名，生成时便于判断模型名是否改变
    }
}
```
`Q5.1` 当更新配置文件时，如果模型名被修改，那么基于模型名创建一个新的文件，而不是在原有的模型名修改，这就导致加载时新旧模型混合。  
`A5.1` 用个变量记录文件名，修改时如果发现模型名被修改，那么创建新模型文件的同时将旧模型文件删除，如果只是单纯的生成配置文件时如果该变量名还有值就会导致误删，针对于这种情况，我们可以在编辑并生成模型后将变量值进行清空，这样下次只是生成时就不会误删了，当然，还有一种情况，就是读取完模型文件但不想修改时关闭窗口，此时也要将变量值进行清空，否则下次打开时会导致误删。

`Q5.2` 当生成完模型文件时，如果我需要再次修改模型名，就会导致生成新的模型文件。  
`A5.2` readFilePath赋有效值操作是在点击编辑按钮并选中json文件之后的，当一次编辑后多次更新并生成模型文件就会导致没法删除旧文件，例如，读取出一个json文件，就将该模型名记录下来（实际记录的是路径，不过路径包含模型名），第一次更新模型名时，会将旧文件删除并将字符串赋值为空，第二次更新模型名时，就会因为字符串为空导致没法删除旧文件。

### Q6.当点击编辑后，如果我不想在原有的基础上改，而是想新创建一个模型，此时如果点击生成模型文件会将旧文件删除，而我希望的是新旧都保留。
`A6. `解决办法：既然用户希望在编辑后创建一个新的模型文件，不如直接给个新建按钮，编辑完点击新建即可新建一个新的模板，不会删除模型文件。

### Q7.新建时我希望能够检测该模型文件是否存在，防止被覆盖，因此加了个检测方法，但编辑json文件更新配置时，由于检测文件代码是放在生成配置代码中的，因此会导致在没修过模型名的情况下更新时检测出一样的模型名，然后导致触发检测方法，就无法更新配置。
`A7.`既然编辑状态不需要检测方法，那么只需要用户是在新建的操作下才进行检测即可，当readFilePath不为空，说明在编辑json文件，反之在新建

```cpp
//custommodelwindow.cpp

if(readFilePath.isEmpty() && QFile(file.fileName()).exists()){
    promptError(modelName + " 该模型文件已存在！请修改模型名");
    return;
}
```

`A7全面解答版：`  
判断是否在新建场景下，如果是则判断是否文件存在，防止其它模型文件被覆盖。  
    
`Q7.1` 为什么要在新建场景下才判断文件是否存在?  
`A7.1` 因为新建和编辑都会经过这段生成配置代码而编辑生成时可能会在相同的文件下更新配置，如果无脑判断文件是否存在，就会导致无法正常更新，所以编辑场景下是不需要检测方法的。

`Q7.2` 怎么去判断是在新建场景呢?  
`A7.2` 就是通过readFilePath值，当值不为空，说明在编辑状态，此时生成就不需要经过检测方法，反之需要。