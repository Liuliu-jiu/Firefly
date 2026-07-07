---
title: 随机库
published: 2026-06-27
pinned: false
description: "个人总结的C++随机库知识点"
image: "./image/咖啡馆.png"
tags: ["C++", "随机库"]
category: C++
draft: false
---

# random随机库
## rand的缺陷
它只能利用随机算法生成[0, RAND_MAX]的整数，如果你想要其它类型，如随机小数，还需要自己转换，麻烦的同时也容易出错，引擎(算法)和分布绑定在一起了，引擎有一个，模具也只有一个，且是整数模具，还绑死在一起

## random概念及优势：
C++11引入了random库，而random则是将着两个东西分开了，引擎就是黑盒子，负责产生无规律组成的0和1的比特流(实际上是是由无规律的比特流组成的无符号大整数)，分布就是模具，负责将原始比特流加工成我想要的随机数类型
random库相比于rand函数，把算法和分布进行分离，引擎只有一个，但模具可以有多个，相对于rand来说灵活且引擎的随机性更好

random相比于rand有多种高质量引擎，随机性也更好，如下：
| 引擎类型         | 典型类名                  | 特点                                            |
|------------------|---------------------------|-------------------------------------------------|
| 线性同余生成器   | minstd_rand0, minstd_rand | 简单快速，周期约 2^31，适合对质量要求不高的场景 |
| 梅森旋转算法     | mt19937, mt19937_64       | 周期极长（2^19937-1），随机性好，通用首选       |
| 带进位减法生成器 | ranlux24_base, ranlux48   | 质量高但速度稍慢，用于高精度科学计算            |
| 默认引擎         | default_random_engine     | 实现定义的引擎，通常是 mt19937 或 minstd_rand   |
  
而分布(模具)通常也有很多种，如下：
|           分布类          |              用途              |
|:-------------------------:|:------------------------------:|
|  uniform_int_distribution |        整数区间均匀分布        |
| uniform_real_distribution |       浮点数区间均匀分布       |
|    normal_distribution    |        正态（高斯）分布        |
|   bernoulli_distribution  |   伯努利试验（真/假，按概率）  |
|    poisson_distribution   |            泊松分布            |
|  exponential_distribution |            指数分布            |
|   binomial_distribution   |            二项分布            |
|             …             | 还包括离散分布、片断常数分布等 |

    int main() 
    {
        //生成1-6的随机数
        std::random_device rd;			//创建种子对象
        std::mt19937 engine(rd());		//创建引擎，利用种子对象的重载小括号生成种子随机数，然后用该随机数作为引擎生成的标准
        std::uniform_int_distribution<int> dic(1, 6);//创建分布，指定区间，但目前只是个空壳
        std::cout << dic(engine);		//将引擎对象通过作为参数传到分布的重载小括号中，使其引擎生成的随机数能够转换成我指定区间的类型，通过返回值传回来
        return 0;
    }