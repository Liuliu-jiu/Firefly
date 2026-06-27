---
title: C++知识点(不定时更新)
published: 2026-06-25
pinned: false
description: "个人总结的C++知识点"
image: "./image/祈祷.webp"
tags: ["C++", "知识点"]
category: C++
draft: false
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

# 移动语义
## 基本认识
### 1. 移动语义是什么？
其实就是通过某种方法从一个对象的数据转移到另一个对象，从而减小拷贝开销，这种方式称为移动语义。

### 2. 为什么出现移动语义？
就是因为拷贝过程中如果数据量过大，那么开销过大，为了减小开销，就出现了移动语义。

## 移动语义的实现
### 1. 左值引用和右值引用
左值引用就是我们常用的引用，它的符号是”&”，可以引用变量名
  
而右值引用，是可以引用没有名称的临时对象以及被std::move标记的对象(也可以说是等号右边的值)，C++11引入右值引用就是为了配合移动语义的使用实现

示例代码如下：

    int val{ 0 };
    int&& rRef0{ getTempValue() };  // OK，引用临时对象
    int&& rRef1{ val };  // Error，不能引用左值
    int&& rRef2{ std::move(val) };  // OK，引用使用std::move标记的对象
以下两种情况会让编译器将对象作为右值进行引用  
1.一行代码执行完后就会销毁的临时对象，俗称匿名对象  
2.被std::move所转移的非const对象
  
让编译器将对象视为右值引用，是一切的基础

### 2. 区分拷贝和移动操作
拷贝操作和移动操作是不一样的，拷贝是申请一块新空间，然后将旧空间的数据复制到新空间去，而移动操作是将原来的空间交给另一个对象管理。  

举一个例子，示例代码如下：

    class MyClass
    {
    public:
        MyClass(const std::string& s)
            : str{ s }
        {};
        // 假设已经实现了移动语义
    private:
        std::string str;
    };
    std::vector<MyClass> myClasses;
    MyClass tmp{ "hello" };
    myClasses.push_back(tmp);  // 这里执行拷贝操作，将tmp中的数据拷贝给容器中的元素
    myClasses.push_back(std::move(tmp));  // 这里执行移动操作，容器中的元素直接将tmp的数据转移给自己
在这个例子中，我们有类和容器，将类对象插入至容器中，插入的时候进行两种不同的操作，一个是将原对象拷贝到容器中，一个是将原对象转移至容器中，这样有什么特点，元素1持有的是原对象的拷贝，而元素2持有的是原对象本身，我们可以发现，第二次插入时，开销变小，因为它不会执行申请空间和复制操作，开销减少了。
  
![内存示意图](image/内层示意图.png)

为什么会进行两种不同的操作？  
原因在于传递的值，我们把有名称和无名称的临时对象当作参数传递了过去，从而被编译器识别成了左值引用和右值引用，然后通过参数类型的不同触发push_back的重载函数，从而触发不同的操作。

    push_back重载函数伪代码如下
    class vector
    {
    public:
        void push_back(const MyClass& value)  // const MyClass& 左值引用
        {
            // 执行拷贝操作
        }

        void push_back(MyClass&& value)  // MyClass&& 右值引用
        {
            // 执行移动操作
        }
    };	
在这段伪代码中，我们发现一个是用左值引用触发，一个是通过右值引用触发，综上所述，通过传递左值引用和右值引用，就能触发不同重载版本的push_back，就能触发不同的操作。

### 3. 移动拷贝构造
C++11引入的新机制，移动拷贝构造，如果我们在创建对象时构造函数接收的是右值引用，那么就会触发移动拷贝构造 。 

示例代码如下：

    class MyClass{
    public:
    // 移动构造函数    
    MyClass(MyClass&& rValue) noexcept  // 关于noexcept我们稍后会介绍        
    : str{ std::move(rValue.str) }      // 看这里，调用std::string类型的移动构造函数 ，这里是将string类型转换成右值{}
        MyClass(const std::string& s)
            : str{ s }
        {}
    private:
        std::string str;};
假设我们自己写的类是官方的类，在这个类中，可以发现有一个构造函数，是以右值引用接收的。

    MyClass tmp{ "hello" };
    MyClass A{ std::move(tmp) };  // 调用移动构造函数，这里是将A对象转换成右值
为什么调用两次std::move？  
第一次是为了让编译器调用起MyClass类的移动构造函数而将A对象转换成右值，第二次是为了移动A对象里面的str对象，所以才调用第二次将str转换成右值并且触发移动拷贝构造，move转换后就会触发转换对象的移动构造。

然后创建两个对象，一个是tmp，一个是A，创建A时由于A的构造函数的参数是move返回的临时对象，因此被编译器识别为右值引用，因此触发移动拷贝构造，将tmp对象中的str转移到A中的str，从而减少了开销，std::move也有重载版本，如果传的左值引用，那么就会将左值转换成右值，如果传的是右值，那么会触发该类的移动拷贝函数。
  
**注意：std::movd只是将左值转换成右值引用，真正的移动操作是发生在各类成员的移动拷贝函数当中。**

### 4. 自己实现移动拷贝(移动语义)
    class MyClass
    {
    public:
        MyClass()
            : val{ 998 }
        {
            name = new char[5] {'P','e','t','e','r'};
        }
        //移动拷贝构造指的是将原有的对象数据转移到另一个对象中，原有对象数据需要清空，所以不能加const，以防修改不了
        MyClass(MyClass&& c) noexcept{      
            std::cout << "触发移动构造" << std::endl;
            this->val = c.val;
            this->name = c.name;
            c.val = 0;
            c.name = nullptr;
        }

        ~MyClass()
        {
            if (nullptr != name)
            {
                delete[] name;
                name = nullptr;
            }
        }
    private:
        int val;
        char* name;
    };
    int main()
    {
        MyClass A{};
        MyClass B{ std::move(A) };
        return 0;
    }

- 关于自己实现移动语义的问题
    1. 转移数据时，将基础类型变量比如int通过std::move移动到另一个变量时，会把临时地址转移到另一个对象中吗?  
    不会，由于基础变量对象是存储于栈空间，虽然有地址，但是没有内存管理的概念，那是属于堆区的概念，继而没有转移地址的概念，既然没有转移地址的概念，那就不会在转移数据时栈上将别的地址交给其他对象管理，在栈上所做的，只有开辟空间和赋值操作，所以通过std::move在栈上转移数据时，所做的只是赋值操作。

    2. 既然在栈上转移的基础类型的数据(比如int类型)是赋值操作，那使用std::move有意义吗?  
    答案是没有的，因为基础类型的数据是没有移动拷贝构造的，比如int，所以编译器会自动将move进行忽略，直接进行赋值操作，比如val{ std::move(rValue.val) }，这里的val是int类型变量，编译器会将move自动忽略，这里的代码相当于val{rValue.val}。

### 5. 移动赋值运算符(重载版本)
它是C++11引入的机制。  
它与拷贝构造和拷贝赋值运算符一样，可以接收右值引用，但它所做的事是通过等号的方式将对象中的数据转移到另一个对象中，跟移动拷贝一样，只不过触发的方式不同。

    class MyClass
    {
    public:
        MyClass()
            : val{ 998 }
        {
            name = new char[5] {'P','e','t','e','r'};
        }
        ~MyClass()
        {
            if (nullptr != name)
            {
                delete[] name;
                name = nullptr;
            }
        }
        void operator=(MyClass&& c) noexcept{
            std::cout << "触发重载移动赋值运算符" << std::endl;
            this->val = c.val;
            this->name = c.name;
            c.val = 0;
            c.name = nullptr;
        }
    private:
        int val;
        char* name;
    };
    int main()
    {
        MyClass A{};
        MyClass B{};
        B = std::move(A);
        return 0;
    }

## 移动构造函数和移动赋值运算符的生成规则
C++11之前有构造，析构，拷贝构造和拷贝赋值，而11后有移动拷贝和移动赋值。

### 1. deleted functions
在讲新加入的两个移动时，先讲已删除的函数，有一种语法，它可以使得某一个函数被删除，如果使用了该函数，则会报错，这种语法被称为“已删除的函数”(deleted functions)。  
    `语法：函数返回值 函数名(函数参数) = delete;`

    class MyClass{
    public:
    void Test() = delete;
    };
    MyClass value;
    value.Test();  // 编译错误：attempting to reference a deleted function

### 2. C++11的编译器会自动生成两个移动
我们知道C++11之前如果定义了一个空类，会为我们生成构造，析构，拷贝构造，拷贝赋值函数，而11及以后，还会编译器则还会生成移动拷贝和移动赋值。

    class MyClass{};
    MyClass A{};  // OK，执行编译器默认生成的构造函数
    MyClass B{ A };  // OK，执行编译器默认生成的拷贝构造函数
    MyClass C{ std::move(A) };  // OK，执行编译器默认生成的移动构造函数

### 3. 如果自定义了拷贝构造或拷贝赋值会发生什么事？
如果定义了一个空类，则会生成6个函数，即两个拷贝，两个移动，构造和析构，但当我自己定义两个拷贝中的一个，编译器则会禁止生成移动拷贝和移动赋值，如果此时接收了右值，会因为没有移动函数则转头执行拷贝操作。

    class Text 
    {
    public:
        Text() {
            std::cout << "触发构造" << std::endl;
        }

        Text(const Text& t) {
            std::cout << "触发拷贝构造" << std::endl;
        }
    };
    int main()
    {
        Text t;
        Text b{t};
        Text a{ std::move(t) };//由于编译器禁止了移动函数，虽然接收了右值，但仍会执行拷贝构造
        return 0;
    }

### 4. 如果自定义析构函数会发生什么事
如果自定义析构函数编译器则不会自动生成两个移动。

    class MyBaseClass
    {
    public:
        virtual ~MyBaseClass()
    {}
    };
    class MyClass : MyBaseClass  // 子类没有实现自己的析构函数{};
    MyClass A{};
    MyClass B{ std::move(A) };  // 这里将执行编译器自动生成的移动构造函数
如果没有virtual关键字，子类继承父类时编译器仍会自动生成移动拷贝和移动赋值，为什么？
加了virtual关键字，相当于子类要重写析构，析构自定义后，就无法自动生成了。
  
子类继承父类时，父类的析构要加virtual关键字，为什么？请看如下  
`Q.` 在C++中，子类继承父类时，父类的析构函数为什么要加virtual，在我的理解中，是因为子类可能有些内存没释放，需要父类调用子类的析构函数以释放，但父类无法调用，所以要加virtual，但是我不理解子类自己都可以调用析构，那为什么需要父类调用，加了virtual关键字后又有什么效果？
  
`A.` 这些问题发生在多态之中，当父类指针指向子类对象需要释放时，由于父类指针只看到自己那一部分，无法看到子类那部分内容，因为它是静态绑定，通过类型调用析构，所以只调用父类自己的析构，没有调用子类的析构函数，导致子类内存泄露，所以我们需要一种方法使父类看到子类析构并调用。  
当在父类加入virtual关键字后，父类会多个虚指针和虚函数表，子类继承下来时，不仅会将父类的属性行为继承下来，也会将虚指针和虚函数表继承下来，并要求子类重写含有virtual关键字的函数，当重写完毕之时，子类虚函数表中的父类函数地址会被替换为子类的虚函数地址，因此在释放时，由于析构函数被重写，delete能通过虚函数表找到子类的析构函数并调用，子类释放后父类也会自动释放。
  
总结：
静态绑定时delete只会通过类型调用析构函数，而动态绑定时则会查找虚函数表调用析构函数。

### 5. 移动构造函数和移动赋值运算符的相互影响
当自定义两个移动中的其中一个时，编译器不会自动生成另一个，如果使用没有自动生成的移动函数，则会直接进行报错，会被定义为使用“已删除的函数”。

    class MyClass
    {
    public:
        MyClass()
        {}

        // 我们定义了移动构造函数，这会禁止编译器自动生成移动赋值运算符，并且对移动赋值运算符的调用会产生编译错误
        MyClass(MyClass&& rValue) noexcept
        {}
    };
    MyClass A{};
    MyClass B{};
    B = std::move(A);  // 对移动赋值运算符的调用产生编译错误：attempting to reference a deleted function

### 6. 总结
当自定义拷贝构造，拷贝赋值，析构函数时，不会自定义生成两个移动，当执行移动操作时，会因为没有两个移动函数则会执行拷贝函数，当定义两个移动之间的一个，使用另一个没生成的会报错。

## noexcept
### 1. 为什么需要noexcept？
Push_back具有强异常保护(即当我们调用一个函数时，如果发生了异常，那么应用程序的状态能够回滚到函数调用之前)，在拷贝异常时，就会恢复原数据，根据这个概念，拷贝构造显然具有此特性，但移动函数在移动过程中会损坏原数据，无法恢复到调用之前的状态，因此只能在发生异常时，抛出异常，因此编译器一般情况下是不敢用移动拷贝的，只敢用拷贝构造，编译器觉得与其丢失原数据，不如不用，就导致性能下降，而我们希望性能提升，因此就可以加个noexcpet告诉编译器这个移动构造具有强异常保护，你放心大胆的用，所以编译器就会采用移动构造了。  

**所以加noexcpet就是告诉编译器大胆用移动构造，使得性能提升，这是在异常安全和性能之间的考量。**

如：

    class MyClass{
    public:
        MyClass()
        {}
        MyClass(const MyClass& lValue)
        {
            std::cout << "拷贝构造函数" << std::endl;
        }
        MyClass(MyClass&& rValue)  // 注意这里，我们没有对移动构造函数使用noexcept    {
            std::cout << "移动构造函数" << std::endl;
        }	
    private:
        std::string str{ "hello" };};

    MyClass A{};
    std::vector<MyClass> classes;
    classes.push_back(A);
    classes.push_back(A);
在这个例子中，A被push_back了两次，而默认的时候，classes容器元素为1，当push_back第二次时，由于空间不够，所以要去内存找一片新空间，并且将之前的数据拷贝过去，如果在第二次拷贝的过程中发生异常，那么可以通过原数据回到之前的状态(第一次push_back的状态)，但移动拷贝构造不行，会导致原有数据丢失的同时抛不出异常，回不到调用函数之前的状态，所以编译器不敢用移动拷贝，可以加个noexcpet让编译器更敢用noexcpet

加了noexcept后就会调用移动构造的形式将旧数据移到新空间当中

    #include<vector>
    class MyClass
    {
    public:
        MyClass()
        {}

        MyClass(const MyClass& lValue)
        {
            std::cout << "拷贝构造函数" << std::endl;
        }

        MyClass(MyClass&& rValue) noexcept  // 注意这里，为移动构造函数使用noexcept
        {
            std::cout << "移动构造函数" << std::endl;
        }

    private:
        std::string str{ "hello" };
    };
    int main()
    {
        MyClass A{};
        std::vector<MyClass> classes;
        classes.push_back(A);
        classes.push_back(A);
        return 0;
    }
![执行结果](image/image.png)

当第二次调用push_back时，我发现调用了一次拷贝构造和一次移动构造，这是为什么？  
第二次调用push_back时，由于空间已满，所以要开辟一块新空间，然后把旧空间的数据挪到新空间中在插入新元素，开辟完空间后，由于A是左值，因此调用了拷贝构造将元素拷贝到第二个位置上，由于我加了noexcpet，编译器就认为旧数据复制到新空间时需要采用移动构造，因此将旧数据的搬到新空间时，采用的是移动构造，而旧数据只有一个，因此只触发了一次移动构造。

## 使用移动语义的需要注意的其他内容：
### 1. 编译器生成的移动构造和移动赋值额外知识
编译器生成的移动构造和移动赋值生成的是逐成员的移动语义，什么意思？意思就是你有几个成员变量，编译器就会生成几个移动语义  
比如：

    class MyClass
    {
    private:
        int val;
        std::string str;
    };
    编译器自动生成的移动拷贝和移动赋值如下：
    class MyClass
    {
    public:
        // 编译器自动生成的移动构造函数类似这样，执行逐成员的移动语义
        MyClass(MyClass&& rValue) noexcept
            : val{ std::move(rValue.val) }
            , str{ std::move(rValue.str) }
        {}

        // 编译器自动生成的移动赋值运算符类似这样，执行逐成员的移动语义
        MyClass& operator=(MyClass&& rValue) noexcept
        {
            val = std::move(rValue.val);
            str = std::move(rValue.str);

            return *this;
        }

    private:
        int val;
        std::string str;
    };
在这个例子中，我们发现每个数学都有与之对应的移动语义

### 2. 被移动的对象指针必须保证置空，这样才能保证正确释放
把被移动对象管理的资源移走后，被移动对象就称为“有效未定义”的状态  
>有效：被移动对象仍是一个合法对象，可以重新赋值，正常销毁  
>未定义：标准没有对被移动对象规定长什么样

所以当资源被转移后，被移动对象就成了“有效未定义”的有力体现，而如果转移后，被移动对象没有及时置空，就会导致移动对象和被移动对象指针都指向同一空间，就会导致释放两次从而程序崩溃，因此及时置空保证正确析构。

### 3. 避免非必要的std::move调用
由于现代C++编译器中启用名为”NRVO“技术特性存在，函数返回值时不需要手动调用std::move进行转换
- NRVO：named return value optimization（命名返回值优化）  

NRVO就是在返回一个具体名字的局部对象时，会将调用方预留的内存地址传过去，让函数直接将该地址当作返回的对象使用，这样能少一次拷贝构造

调用方的地址：
函数调用时，会为等号左边的对象(调用方)开辟一段内存地址，方便用于接收数据，这段地址被称为调用方的地址
比如：

    MyClass xmyClass = GetTemporary(); 
调用GetTemporary方法前，会提前为myClass 创建一片空间，而myClass 则被称为调用方，因此这片空间被称为调用方的地址

不启用NRVO时，如果返回一个临时对象，那么会进行两次操作，函数内构造一次局部对象，返回时拷贝一次给调用方

启用NRVO时，编译器会让函数签名多增加一个参数，这个参数是用来接收调用方的函数地址，调用时会直接将调用方预留内存的地址传到函数内，关于返回对象的操作都会在传过来的地址(调用方预留的内存)上操作
传过来的参数地址其实就是返回的对象，任何对该返回对象的操作都是在传过来的参数地址(调用方预留的内存)操作，相当于带回型参数，将地址传过去，让函数直接在这片地址进行操作
  
比如：

    class MyClass{};
    MyClass GetTemporary(){	
        MyClass A{};
    return A;
    }
    MyClass myClass = GetTemporary();  // 注意这里
调用GetTemporary函数时，编译器会为myClass对象提前开辟一段空间，用于接收返回的对象数据，编译器检测到返回的是一个具体名字的局部对象，因此它将函数签名改为

    void GetTemporary(void* __result);  // 伪代码
这样，就是多增加了一个参数，调用时编译器会自动将调用方预留内存地址(也就是&myClass)传到GetTemporary函数内，使得调用方对象取代A对象，在函数中，A其实就是myClass对象，对A的构造操作等同于对myClass的构造操作


## 参考链接
[一文入魂：妈妈再也不担心我不懂C++移动语义了](https://zhuanlan.zhihu.com/p/455848360)
