---
title: 认识Qt
published: 2026-06-27
description: 介绍Qt的基本知识
image: "../Article_Image/Qt.png"
tags: [Qt, 基本认识]
category: Qt
draft: false
---

# 基本概念
Qt是由C++编写的跨平台框架，它包含大量的类，支持GUI，数据，多媒体，网络等各自应用的开发，在计算机中，你想在哪个平台上运行软件，就要在哪个平台编译，Qt跨平台的本质上运行目标平台编译生成好的二进制文件，所以才能在不同的设备上运行。

# Qt许可类型
1. 商业许可
允许源代码不公开，且使用的模块更多。
2. 开源许可
分为GPLv2/GPLv3许可，如果开发用到了GPL许可的Qt代码，则要求开发者开源，但可以商业销售，GPLv3则在此基础上额外要求公开相关硬件信息。
3. LGPLv3许可
跟GPL许可差不多，只不过宽松些，具体的来说就是如果使用了LGPL许可的Qt代码，则要求开发者开源，如果只是以库的形式链接或调用LGPL许可的Qt代码，则开源闭源，不管哪种方式都可以商业化销售。

# Qt安装包
根据开发目标的不同，分为三种Qt安装包
1. Qt for Appliaction Development
为计算机和移动设备所准备开发套件安装包，具有商业和开源版本，有windows，Linux，macos平台版本。
2. Qt for Device Creation
为嵌入式设备所准备的开发应用套件安装包，只有商业版本，具有windows，Linux平台版本。
3. Qt for MCUs
为MCU GUI开发所准备的开发套件安装包，同样只有商业版本，有widonws和Linux平台版本。

# Qt支持的语言
1. C++和QML
2. python

# Qt6相对于Qt5的有哪些新特性
1. 支持C++17标准
2. Qt核心库的改动。比如字符串全面支持Unicode，修改了QList类的实现方式，将QVector和QList统一为QList，QMetaType和QVariant被Qt几乎重写，这两个也是元对象系统的基础
3. 添加Cmake构建系统，但仍兼容qmake

# Qt安装模块讲解(目前安装的模块，部分)
1. MSVC 2022 64-bit
这是使用Qt的widget插件时所需要的编译器套件，如果想使用这个套件，不仅要下载这个套件，而且还要安装vistual studio 2022，社区版即可，如果不想安装那么庞大的软件，那么只需要下载mingw即可
2. MinGw 13.1.0 64-bit
这是由MinGw 编译器的开发套件，在windows平台上使用的是GNU工具集，包含GNU C++编译器
3. Source
这是Qt框架的源代码
4. Qt 5 Compatibility Module
这是为了在Qt6兼容Qt5所设计的模块，它里面包含着被Qt6移除的Qt功能，当看到某个类或函数的提示为强烈不赞成，淘汰的，那么就代表改类或函数是从这个模块来的，为了兼容性，应当选择安装
5. Qt Shared Tools
Qt的着色器工具，用于3D图形的着色的模块
6. Cmake 3.30.5
Cmake构建系统的开发套件，适用于大型软件项目的的构建工具
7. Ninja 1.12.1
它是一个小型的构建系统，专注于构建速度，可与Cmake结合使用

# 参考书籍
[Qt 6 C++开发指南](https://baike.baidu.com/item/Qt%206%20C%2B%2B%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97/63406205)