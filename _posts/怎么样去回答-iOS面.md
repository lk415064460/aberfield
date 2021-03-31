### 怎么样去回答iOS面试问题

+ 日常工作中能用，原理了解清楚了，但是在面试中很难清晰流畅的去表达，所有在此，用文字去记录问题的回答答案以及方式，可能会偏口语一点，意在加深印象同时学会回答。
+ 主要从iOS相关面试题开始，基础，方案，框架性的东西逐一整理。


#### iOS基础面试题


##### OC相关
1. assign和weak修饰符的区别
    + assign一般用来修饰值类型，存储在栈空间的属性，由系统管理其内存生命周期
    + weak属性一般用来打断引用环，被weak引用的对象它的引用计数不会增加，而且在这个对象被释放的时候被Weak修饰的变量会自动置空，不会造成野指针问题。
    + weak属性实现：
        1. 通过全局的SideTables查找到对应属性(指针地址为Key)的SideTable。(哈希查找)
        2. SideTable存在一个自旋锁，RefcountMap, weak_table。其中自旋锁用于保证同一时间只能由一个线程进行修改，RefcountMap代表属性的应用关系，weak_table_t保存着弱引用实体。结构体如下:

        + SideTable结构体

            ```
            struct SideTable {
                spinlock_t slock;
                RefcountMap refcnts;
                weak_table_t weak_table;

                SideTable() {
                    memset(&weak_table, 0, sizeof(weak_table));
                }

                ~SideTable() {
                    _objc_fatal("Do not delete SideTable.");
                }

                void lock() { slock.lock(); }
                void unlock() { slock.unlock(); }
                void forceReset() { slock.forceReset(); }

                // Address-ordered lock discipline for a pair of side tables.

                template<HaveOld, HaveNew>
                static void lockTwo(SideTable *lock1, SideTable *lock2);
                template<HaveOld, HaveNew>
                static void unlockTwo(SideTable *lock1, SideTable *lock2);
            };
            ```



        + weak_table_t结构体
            ```
            struct weak_table_t {
                weak_entry_t *weak_entries;
                size_t    num_entries;
                uintptr_t mask;
                uintptr_t max_hash_displacement;
            };
            ```

        
        ![SideTable结构](https://raw.githubusercontent.com/aberfield/figureStore/master/iOS/weak01.webp)

    + 概念问题：
        + SideTabls:为了管理所有对象的引用计数和weak指针，苹果创建了一个全局的SideTables，虽然名字后面有个"s"不过他其实是一个全局的Hash表，里面的内容装的都是SideTable结构体而已。它使用对象的内存地址当它的key。管理引用计数和weak指针就靠它了。


2. NSObject *obj = [[NSObject alloc] init]占有多少空间？
    + obj指向的内存空间大小取决于NSObject内部实现的需要占有多少内存空间，如下代码块Object存在一个对象 Class isa，该实例占有8个字节的空间。所有obj里面的实例对象占有8bytes空间，但是obj占有的空间不是8bit而是16bytes，具体原因如下：


        + NSObject 对象结构
        ```
            @interface NSObject <NSObject> {
                Class isa  OBJC_ISA_AVAILABILITY;
            }

            /// 翻译成C和C++如下：
            struct NSObject_IMPL {
                Class isa;
            }

        ```

        + alloc时处理: CF requires all objects be at least 16 bytes
        ```
        inline size_t instanceSize(size_t extraBytes) const {
           if (fastpath(cache.hasFastInstanceSize(extraBytes))) {
                return cache.fastInstanceSize(extraBytes);
            }

            size_t size = alignedInstanceSize() + extraBytes;
            // CF requires all objects be at least 16 bytes.
            if (size < 16) size = 16;
            return size;
        }

        ```


##### Swift相关


##### iOS Widget开发
1. Widget如何和主App进行数据交互
    + 在Widget和主APP间使用App Groups进行数据交互。在Capability中勾选App Groups选项。
    + 使用NSUserDefault进行数据传输交互
    + 使用NSFileManage进行数据交互
    + Widgets和主App以服务端数据为桥接进行数据交互

2. Widget如何支持动画
    + 不支持播放动画Gif、视频
    + 不支持滚动
    + 不支持主动刷新视图
    + 貌似有私有API可以旋转动画(后期可以去调查)

3. Widget生命周期
    + 核心是时间线，在对应的时间点上显示对应的UI内容
    + 时间线由一个或者多个时间线入口以及重载策略组成。
    + 时间线刷新:
        1. System Reloads: 由系统主动、按时发起，会relad下一时间的数据。
        2. App Reloads： 应用在前台时调用Widget Center相关API进行触发；处于后台时通过后台推送刷新时间。



##### 第三方库



#### 底层原理



#### 设计方案



#### 架构思路



#### 网络传输


#### 跨平台