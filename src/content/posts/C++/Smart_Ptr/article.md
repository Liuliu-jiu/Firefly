---
title: 智能指针
published: 2026-06-27
pinned: false
description: "个人总结的智能指针知识点"
image: "../Article_Image/C++.png"
tags: ["C++", "智能指针"]
category: C++
draft: true
---

# 智能指针
## 为什么需要智能指针？
- 手动管理内存可能会导致内存泄露，为了解决这个问题，就使用智能指针解决。

## 智能指针是什么？
- 智能指针就是自动管理内存空间，当对象出作用域时，其指针会自动析构所指向的地址。
- 智能指针分为auto_ptr（已弃用）,unique_ptr，shared_ptr，weak_ptr。

## 智能指针的种类
### 1. auto_ptr（已弃用）
Auto_ptr是C++98引入的智能指针，它能在对象出作用域时，自动析构指向的内存地址，**但是有个严重缺陷，auto_ptr指针在赋值或拷贝时会进行所有权转移**。

示例代码如下：

    #include <iostream>
    #include <memory>
    int main() {
        std::auto_ptr<int> p1(new int(10));
        std::auto_ptr<int> p2 = p1; 
        std::cout << *p2 << std::endl; 
        std::cout << *p1 << std::endl; 
        return 0;
    }

p1指针赋值或拷贝到p2指针，就会导致p1指针成空指针，换句话说就是p1指针的所有东西剪切到p2指针了，如果此时访问p1指针很容易出事，这也是为什么要放弃使用的主要原因。

**并且它不能释放数组！它使用的是delete而不是delete[]。** 

### 2. unique_ptr
<!-- 
如果想在无序列表中嵌入单纯的文本，那么可以在无序列表后面加两个空格和一个换行，下行直接写文本即可，当然为了规范，在下行前面加4个空格或一个制表符 
-->
- 独占式概念  
    unique_ptr是C++11引入的智能指针，它与前一个指针不同的是它有独占式概念，什么叫独占式，其实就是同一个时刻只能有一个指针指向该地址，防止资源泄露，如果在有unique_ptr指针指向该地址的情况下拷贝或赋值，就会导致编译失败。

    示例代码如下：

        #include <iostream>
        #include <memory>
        int main() { 
            std::unique_ptr<int> u1(new int(10));
            // std::unique_ptr<int> u2 = u1;  // 编译错误，不允许拷贝
            std::unique_ptr<int> u3 = std::move(u1); //转移后U1变为空指针
            if (!u1) {
                std::cout << "u1 is nullptr" << std::endl;
            }
            std::cout << *u3 << std::endl; 
            return 0;
        }

    在这个例子中，如果直接将地址赋予其他的unique_ptr指针，会导致编译错误，如果想转移到别的指针进行管理，那么可以通过std::move进行所有权转移

- 所有权转移  
    unique_ptr指针支持管理数组，释放时，会自动使用delete[]来释放，同时能够通过返回值进行所有权转移

    示例代码如下：

        #include <iostream>
        #include <memory>
        std::unique_ptr<int> createInt() {
            return std::unique_ptr<int>(new int(20));
        }
        int main() {
            std::unique_ptr<int> ptr = createInt();
            std::cout << *ptr << std::endl; 
            return 0;
        }
    在这个例子中，将unique_ptr指针的所有权通过返回值转移到另外一个指针

### 3. shared_ptr
- 共享式概念  
    它拥有共享式拥有概念，它允许多个指针对象指向同一个内存地址，通过引用计数的方式来管理生命周期，当一个shared_ptr指针指向一个地址时，引用计数加1，当指针离开作用域或被重新赋值时，引用计数就会减1，当引用计数为0时，指针指向的地址就会被释放。

    示例代码如下：
    
        #include <iostream>
        #include <memory>
        int main() {
            std::shared_ptr<int> s1(new int(10));
            std::shared_ptr<int> s2 = s1; 
            std::cout << "s1 use_count: " << s1.use_count() << std::endl; 
            std::cout << "s2 use_count: " << s2.use_count() << std::endl; 
            s1.reset(); 
            std::cout << "s2 use_count: " << s2.use_count() << std::endl; 
            return 0;
        }
    在这个例子中，开辟了两个指针，s2指向s1指向的地址，所以引用计数为2，当释放时，引用计数减1，所以打印出来的引用数就为1。

- **注意事项**：  
    在shared_ptr中，引用计数的增减(+1，-1)的操作是原子操作，**但引用计数的修改(将计数修改为任意值)不是线程安全，从而可能引发数据竞争**，因此可以使用std::atomic或互斥锁保证引用计数不得随意修改。

    > 什么叫原子操作？其实就是不可被切割，不可被打断的最小操作单元。
    >> 为什么要进行原子操作？  
        比如进行i++时，通常分为3步  
        1.读取值  
        2.i + 1  
        3.重新赋值  
        CPU很可能在这三步中切换线程，从而导致数据竞争，所以我们可以利用原子操作将这三个步骤打包成一个整体，要么全都执行完，要么全都不执行，从而保证线程安全，而我们利用的就是C++模板的std::atomic，该模板可以将一个变量封装成“线程安全版”，从而保证线程安全。

### 4. weak_ptr
- 弱引用  
    weak_ptr是一种解决shared_ptr循环引用的弱引用指针，它不会增加引用计数，也不会控制对象的生命周期，只是记录shared_ptr指向的地址。

- 什么是循环引用？  
    两个地址内部有指针，并且指向对方，但这两个地址没有指针指向，导致无法找到这两个地址从而无法找到地址内部两个指针释放，就会导致内存泄露。
    
    示例代码如下：

        #include <iostream>
        #include <memory>
        class B;
        class A {
        public:
            std::shared_ptr<B> pb_;
            ~A() {
                std::cout << "A delete" << std::endl;
            }
        };
        class B {
        public:
            std::shared_ptr<A> pa_;
            ~B() {
                std::cout << "B delete" << std::endl;
            }
        };
        void fun() {
            std::shared_ptr<A> pa(new A());
            std::shared_ptr<B> pb(new B());
            pa->pb_ = pb;
            pb->pa_ = pa;
            std::cout << "pa use_count: " << pa.use_count() << std::endl; 
            std::cout << "pb use_count: " << pb.use_count() << std::endl; 
        }
        int main() {
            fun();
            return 0;
        }
    在这个例子中，A对象有一个指针指向B，B对象中有一个指针指向A，此时两个地址的引用计数都是为2，出作用域后，外部的pa和pb指针被释放，内部并没有，说明那两块空间并没有被释放，因此引用计数为1，但由于指向这两块空间的外部指针已经被释放了，因此没有任何外部指针指向这两块空间，从而无法释放，导致内存泄露。

    **循环引用根本原因：shared_ptr 靠引用计数管理内存，计数 = 0 才释放；循环引用让计数永远≥1。**

- 怎么解决循环引用？  
    使用weak_ptr，将其中的一方内部指针改为weak_ptr即可。

    示例代码如下：

        #include <iostream>
        #include <memory>
        class B;
        class A {
        public:
            std::shared_ptr<B> pb_;
            ~A() {
                std::cout << "A delete" << std::endl;
            }
        };
        class B {
        public:
            std::weak_ptr<A> pa_;
            ~B() {
                std::cout << "B delete" << std::endl;
            }
        };
        void fun() {
            std::shared_ptr<A> pa(new A());
            std::shared_ptr<B> pb(new B());
            pa->pb_ = pb;
            pb->pa_ = pa;
            std::cout << "pa use_count: " << pa.use_count() << std::endl; 
            std::cout << "pb use_count: " << pb.use_count() << std::endl; 
        }
    在这个例子中，创建时交给指针管理，此时A和B地址的引用计数都为1，A内部指针指向B时，由于A内部指针是shared_ptr，所以B地址的引用计数为2，B内部指针指向A地址时，由于B内部指针是弱引用指针，不会增加引用计数，所以A地址的引用计数为1，当出作用域时，各自地址的引用计数减一，发现A地址引用计数为0，因此将A地址释放，释放时指向B地址的指针也被释放，因此B地址的引用计数归0，B地址也被释放，从而解决循环引用，所以解决循环引用的方法就是释放时让某一方的引用计数归0。

- **注意事项**    
    使用weak_ptr时，要通过lock方法转换成shared_ptr，如果weak_ptr指向的地址是已释放的，那么会返回一个空的shared_ptr。
    为什么要转换成shared_ptr才使用？
    因为weak_ptr不控制对象生命周期且不拥有所有权，它只是单纯记录地址，不能对那块空间进行操作。

## 智能指针之间的区别
### 1. 所有权  
unique_ptr，shared_ptr，weak_ptr上有本质区别  
- unique_ptr是独占式所有权，它只允许再同一时刻只能有一个指针指向地址。  
- shared_ptr是共享式所有权，它允许多个对象指向同一个地址。  
- weak_ptr不拥有所有权，它只是对象的观察者，为了解决shared_ptr指针循环引用问题而存在。
### 2. 内存管理  
- auto_ptr指针出作用域即消耗，但由于赋值和拷贝会进行所有权转移，就会导致悬空指针的问题，因此在C++11被放弃使用。
- unique_ptr它只允许同一个时刻指向一个地址，也是出作用域后立即把这片空间消耗，但与auto_ptr不一样的是，它不允许赋值或拷贝，这就解决了  auto_ptr指针的悬空指针问题，也保证了内存释放的及时性和正确性。
- shared_ptr通过引用计数的方式管理内存，当引用计数为0时，自动释放内存。
- weak_ptr本身不管理内存，它只是内存的记录者，当weak_ptr指针指向的地址释放时，它不会至空，需要通过lock方法检测空指针。
### 3. 应用场景  
- unique_ptr指针应用于明确单一所有权场景。  
- shared_ptr应用于共享资源的场景。  
- weak_ptr应用于解决shared_ptr指针循环引用问题和观察对象状态但不影响生命周期的场景。

## std::shared_ptr 原理与手动实现
### 1. shared_ptr的原理
通过在堆区上分配控制块来记录引用数量，当引用数量变为0时，控制块会将这块地址释放，除第一个指针指向这块空间时会创建控制块，其余指针指向这片空间时则会共享该控制块。
如
std::shared_ptr<int> sp1(new int(10));
std::shared_ptr<int> sp2 = sp1;
此时，sp1和sp2共享同一个控制块，引用计数变为 2 
### 2. 手动实现
    #include <iostream>
    template <typename T>
    class MySharedPtr {
    public:
        MySharedPtr(T* ptr = nullptr) : m_ptr(ptr), m_refCount(new int(1)) {}

        //析构
        ~MySharedPtr() {
            release();
        }

        //构造
        MySharedPtr(const MySharedPtr& other) : m_ptr(other.m_ptr), m_refCount(other.m_refCount) {
            (*m_refCount)++;
        }

        //重载运算符等号
        MySharedPtr& operator=(const MySharedPtr& other) {
            if (this != &other) {
                release();
                m_ptr = other.m_ptr;
                m_refCount = other.m_refCount;
                (*m_refCount)++;
            }
            return *this;
        }

        //重载运算符星号
        T& operator*() const {
            return *m_ptr;
        }

        //重载运算符解引用号
        T* operator->() const {
            return m_ptr;
        }

        //统计引用数
        int use_count() const {
            return *m_refCount;
        }

    private:
        void release() {
            (*m_refCount)--;
            if (*m_refCount == 0) {
                delete m_ptr;
                delete m_refCount;
            }
        }

        T* m_ptr;			//数据块，指向地址的指针
        int* m_refCount;		//控制块，用来记录引用数量
    };

##  std::make_shared 与 std::shared_ptr (new T (args...)) 的对比
### 1. 性能对比
在频繁开辟空间的场景，使用make_shared比直接使用构造函数好，为什么?
make_shared能同时创建对象内存和控制块内存，而构造函数是先需要为对象分配内存然后再为控制块分配内存，多次分配不仅导致时间开销增加，也会使得内存碎片化严重，降低内存使用率。

### 2. 安全性对比
make_shared的胜于直接使用构造函数的，为什么?  
使用make_shared意味着减少了显式使用new的需求，减少了需求就意味着手动管理的场景少了，少了就能降低内存泄露的风险。

示例代码如下：

    void processWidget(std::shared_ptr<MyClass> ptr, int priority) {
        // 处理逻辑
    }

    void testException() {
        try {
            processWidget(std::shared_ptr<MyClass>(new MyClass), 1);
        } catch (...) {
            // 捕获异常
        }
    }
在这段场景中，如果创建空间的过程中发生异常，就会导致内存泄露
为什么?我们知道，使用构造函数时是有两步的，先为对象分配空间，再为控制块分配空间，如果为对象分配完空间后出现异常，就会导致控制块没被创建，对象的地址得不到记录，无法释放，就造成内存泄露。

换成make_shared就不用出现这种情况，为什么?  
因为对象和控制块一起创建，如果某一方出现问题，另一方也不会创建成功，就不会造成内存泄露，从而保证了安全

### 3. 简洁性
使用make_shared比直接使用构造函数更简洁。为什么？  
减少了手动new和delete的繁琐操作，同时让代码看起来更紧凑易读。

## 智能指针使用时要注意什么
### 1. unique_ptr 使用要点
1. 要注意独占资源的特性  
    当代码出现拷贝赋值时就会编译错误，如果想将一个地址完全交给另一个unique指针使用，那么就要通过移动语义move进行所有权转移。
    
        std::unique_ptr<int> ptr1 = std::make_unique<int>(10); 
        std::unique_ptr<int> ptr2 = std::move(ptr1); 

2. 利用返回值返回unique指针必须使用移动语义  
    为什么返回unique指针需要用到移动语义？  
    返回unique指针时是拷贝返回，因此会因为自身的特性导致拷贝失败，所以用到移动语义，为了解决返回值的问题，所以需要用到移动语义来解决。

    示例代码如下：

        std::unique_ptr<int> createUniquePtr() {
        return std::make_unique<int>(10);//等价于
        }
        auto ptr = createUniquePtr(); 
    在这段代码中，虽然没有用到移动语义，但是现代编译器(C++11以后的编译器)会为你自动添加移动语义，移动语义的发生点在return语句那。

    示例代码如下：

        return std::make_unique<int>(10); 
        // （编译器自动帮你加了 move） 
        return std::move(std::make_unique<int>(10));
    它会将局部对象指向的地址的所有权交给接受的对象管理，在这个例子中，局部对象unique指针指向的地址交给了ptr管理，同时将所有权转移到ptr，这就意味着局部对象指针是没有对这块空间管理的权限的，所以出作用域时，这块地址没被释放，外面的对象才能正确接受返回的地址，现代编译器不论返回什么值都会优先使用移动语义。

    > 之前编译器返回值方式(C++11之前)：在之前的编译器上，返回一般值是通过拷贝构造的方式返回，但这样会有一个问题，当返回局部对象时，由于局部对象和接受的对象都有对地址的所有权，因此局部对象出作用域后，会对该地址进行释放，就会导致接受的对象指向的地址是已释放的，使用时会出问题，所以C++11及以后，就优先采用移动语义解决

3. unique指针可以自定义删除器  
- 自定义删除器是什么？  
    自定义删除器就是像自定义析构函数那样，自己决定管理的资源应该怎么释放。

- 为什么需要自定义删除器？
    一般的unique指针只会使用delete释放，而有些资源不能使用delete，delete会将释放的地址当做堆地址看待，会查看内存头，标记内存块大小等，但有些地址不是堆地址，比如栈变量，全局变量，静态变量，再比如文件句柄，数据库连接，这些都有各自的关闭方式，如果使用delete，会导致程序崩溃，如果使用delete释放FILE*，就会导致文件句柄一直占着，从而打不开新文件。  
    >为什么会占着？delete调用的C++对象的析构函数，而FILE*是一个结构体，没有析构函数，它只会把FILE*指向的结构体暴力释放，而文件句柄是由操作系统内核管理的，释放后没有通知操作系统内核关闭文件句柄，就导致文件句柄一直占着，而句柄数量有限，打开多了新文件自然就打不开了，所以我们需要自定义删除器来去释放资源。

- 怎么去自定义删除器？  
    就是要自己自定义结构体，然后重载小括号来决定资源怎么释放。

    示例代码如下：

        struct FileCloser {
            void operator()(FILE* file) const {
                if (file) {
                    fclose(file);
                }
            }
        };
        std::unique_ptr<FILE, FileCloser> file_ptr(fopen("test.txt", "r")); 
    这里定义了一个 FileCloser 结构体作为删除器，当 file_ptr 被销毁时，会调用 FileCloser 的 operator() 来关闭文件，确保资源得到正确释放。 

    <!-- 
    如果想在引用中嵌入代码块的同时与文字共存，使用围栏式代码块，再引用中用一对三个反引号(`)包裹代码块，并且上下行留出一行空白行，每一行代码前面也要加右尖括号
     -->
    - 为什么要重载小括号？

            template<typename T, typename Deleter> 
            class unique_ptr { 
            T* ptr;
            Deleter deleter; // 这里创建了一个删除器对象！ 
            public:
            ~unique_ptr( ) { 
                deleter(ptr); // 调用删除器 
                }
            };
        在这段unique底层代码中，unique指针的第二个参数是删除器的类型，它决定了调用哪个方法来释放资源，我们可以发现传入的是一个类型，在内部会创建一个删除器对象，这个删除器对象会通过调用重载小括号的方法来释放资源,如果没有传入额外的删除器类型，就是一般的unique指针，调用默认的删除方法，此时只会用delete释放，如果传入了额外的删除器类型，就会调用起我写的释放资源的方法，而我们发现，它是通过对象名+小括号的方式调用释放资源的方法，这种方式调用相当于deleter.operator()，要想让它调用我写的释放资源代码，就要通过重载结构体中小括号的方式让它调用，所以，这就是为什么传入删除器类型时要重载小括号的原因，重写结构体的小括号来决定资源怎样释放。

### 2. shared_ptr 使用要点
- 使用shared_ptr时要注意线程安全  
    虽然引用计数的修改(增减，在控制块内)是线程安全，但是它本身的对象不是线程安全，比如对象在赋值或拷贝时，赋值和拷贝这个操作里面有几个步骤，这几个步骤不是连在一起执行的，就有可能执行的时候被CPU打断，如果同时操作一个指针，就可能会导致数据竞争
    >数据竞争就是多个线程在同一时刻读取或修改某一个资源，导致数据修改错乱，引发计数错误，甚至程序崩溃等现象。

    - 案例
    1. 数据拷贝不全现象  
        示例代码如下：

            std::shared_ptr<int> global_ptr = std::make_shared<int>(10); 
            std::shared_ptr<int> local_ptr;
            std::shared_ptr<int> ptr1;
            void threadFunction1() {
                ptr1 = local_ptr; // 赋值！
            }
            void threadFunction2() {
                local_ptr = global_ptr; // 拷贝！
            }
        这函数分别对同一个指针对象拷贝和赋值，我们首先来了解拷贝和赋值期间发生了什么，拷贝分为三步，将数据块(1)和控制块(2)的地址复制过去后，引用计数再加一(3)，而赋值分为四步，由于要指向新地址，所以引用计数减一(1)，减一后如果引用计数归0，则释放地址(2)，然后指向新对象(3)，引用计数再加一(4)，而这些步骤并不是连在一起执行的，就有可能在这过程中被CPU打断。
  
        在这段例子中，如果有两个线程同时操作指针，在赋值的期间（也就是赋值到一半）进行拷贝，此时出现数据竞争，就会导致数据拷贝不全的情况。

    2. 错误释放原地址现象

            std::shared_ptr<int> global_ptr = std::make_shared<int>(1); //此时状态是ptr = 旧地址，count = 1
            void threadFunction1() {
                global_ptr = std::make_shared<int>(10); // 赋值！
            }
            void threadFunction2() {
                auto local_ptr = global_ptr; // 拷贝！
            }
        在代码中，global_ptr的初始状态引用计数就为1，如果A，B线程的指针同时对一块地址进行拷贝和赋值，就有可能在拷贝或赋值的过程中进行另外一种操作，从而错误释放原地址。  
        比如有可能会在拷贝过程中刚复制完两块地址还没有将引用计数加一就中断A线程，去执行B线程的赋值操作，执行赋值操作时，由于要指向新对象，所以引用计数减一，减一后，引用计数归0，就把原地址释放了，这个地址是A线程正在操作管理的，**但由于被打断了操作，A线程是不知道已被释放，所以执行完B线程赋值操作后切换A线程继续执行时，还会将引用计数继续加一，此时引用计数虽然不为0，但是持有的是被B线程已释放的地址，后续使用A线程管理的地址就会崩溃。**

        **所以错误释放原地址现象出现的原因及程序崩溃的原因在于：同时读取或修改会出现数据竞争的问题，导致引用计数错乱**，B线程把A线程的地址误释放了，但A线程是不知道的，仍使用释放后的地址，导致后续使用时会崩溃。

        所以出现线程安全问题的原因就是同一时刻对同一个对象进行操作从而引发数据竞争，为了解决这个问题，要给指针对象上互斥锁，保证同一时刻只能有同一个线程对这个对象操作。

            std::shared_ptr<int> global_ptr; 
            std::mutex global_ptr_mutex; 
            void threadFunction1() {
                std::lock_guard<std::mutex> lock(global_ptr_mutex); 
                global_ptr = std::make_shared<int>(10); 
            }
            void threadFunction2() {
                std::lock_guard<std::mutex> lock(global_ptr_mutex); 
                auto local_ptr = global_ptr; 
            }
        global_ptr_mutex是锁本身，lock是锁的管理员，当它以锁本身为参数创建对象时，会对该锁上锁，出作用域时，锁就会被释放，就跟Qt的QMutex一样，或者从 C++20 开始，可以使用 std::atomic<std::shared_ptr> 来实现对 std::shared_ptr 的原子操作，从而避免线程安全问题。


### 3. weak_ptr使用要点
使用时一定要通过lock转换成std::shared_ptr才能使用，因为weak_ptr只是记录地址和观察对象的状态，不具有对象的所有权，因此转换成std::shared_ptr才能使用。

<!-- 为什么要放参考链接？因为我的笔记是看这篇文章总结而成的，大体框架相同，因此要标记下 -->
## 参考链接
[一文吃透C++智能指针：原理、区别与实战](https://zhuanlan.zhihu.com/p/1921579814285975670)