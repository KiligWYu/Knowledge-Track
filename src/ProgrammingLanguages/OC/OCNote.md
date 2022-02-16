
# OC 底层学习笔记

- [OC 底层学习笔记](#oc底层学习笔记)
  - [面向对象](#面向对象)
    - [OC -> C/C++](#oc-cc)
    - [一个 NSObject 对象占用多少内存？](#一个nsobject对象占用多少内存？)
    - [对象的 isa 指针指向哪里？](#对象的isa指针指向哪里？)
    - [OC 的类信息存放在哪里？](#oc的类信息存放在哪里？)
  - [KVO](#kvo)
    - [iOS 使用什么方式实现一个对象的 KVO？（KVO 的本质是什么？）](#ios使用什么方式实现一个对象的-kvo？（kvo的本质是什么？）)
    - [如何手动触发 KVO](#如何手动触发kvo)
    - [直接修改成员变量的值会触发 KVO 么？](#直接修改成员变量的值会触发kvo么？)
  - [KVC](#kvc)
    - [通过 KVC 修改属性会触发 KVO 么？](#通过kvc修改属性会触发-kvo么？)
    - [`setValue:forKey:` 原理](#setvalue-forkey原理)
    - [`valueForKey:` 原理](#valueforkey原理)
  - [Category](#category)
    - [Category 的原理](#category的原理)
    - [Category 和 Class Extension 的区别](#category和-class-extension的区别)
    - [+load 方法](#load方法)
    - [+initialize 方法](#initialize方法)
    - [给 Category 添加成员变量](#给category添加成员变量)
      - [关联对象](#关联对象)
      - [关联对象的 key 的常见用法](#关联对象的key的常见用法)
      - [objc_AssociationPolicy](#objc-associationpolicy)
      - [关联对象的原理](#关联对象的原理)
  - [Block](#block)
    - [Block 的本质](#block的本质)
    - [Block 的变量捕获机制](#block的变量捕获机制)
    - [Block 类型](#block类型)
    - [Block 的 copy](#block的-copy)
    - [对象类型的 auto 变量](#对象类型的auto变量)
    - [__block 修饰符](#block修饰符)
    - [__block 的内存管理](#block的内存管理)
    - [被 __block 修饰的对象类型](#被block修饰的对象类型)
  - [Runtime](#runtime)
    - [Class 结构](#class结构)
    - [方法缓存](#方法缓存)
    - [OC 消息机制](#oc消息机制)
    - [`super` 关键字](#super关键字)
  - [RunLoop](#runloop)
    - [RunLoop 本质](#runloop本质)
    - [RunLoop 与线程](#runloop与线程)
    - [RunLoop 的运行逻辑](#runloop的运行逻辑)
    - [线程保活](#线程保活)
  - [多线程](#多线程)
    - [各种队列的执行效果](#各种队列的执行效果)
    - [请问下面代码的打印结果是什么？](#请问下面代码的打印结果是什么？)
    - [线程同步](#线程同步)
    - [线程同步方案性能比较](#线程同步方案性能比较)
  - [Timer](#timer)
    - [CADisplayLink、NSTimer、dispatch_source_t](#cadisplaylink、nstimer、dispatch-source-t)
  - [内存管理](#内存管理)
    - [iOS 程序的内存布局](#ios程序的内存布局)
    - [Tagged Pointer](#tagged-pointer)

## 面向对象

### OC -> C/C++

``` bash
$ clang -rewrite-objc main.m -o main.cpp
// 指定平台和架构。模拟器(i386)、32bit(armv7)、64bit(arm64)
$ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main.cpp
// cannot create __weak reference in file using manual reference 解决方法：支持 ARC，指定运行时版本
$ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-14.0.0 main.m
```

### 一个 NSObject 对象占用多少内存？

- 系统分配了 16 个字节给 NSObject 对象（通过 `malloc_size` 函数获得）
- 但 NSObject 对象内部只用了 8 个字节的空间（64 位下，可以通过 `class_getInstanceSize` 函数获得）

``` objectivec
// 获取 NSObjcet 类实例对象的成员变量所占用的空间，其实例对象的成员变量只用 isa，所以返回 8
NSLog(@"%zd", class_getInstanceSize([NSObjcet class]);
// 获取 obj 指针所指向内存的大小，返回 16
NSLog(@"%zd", malloc_size((__bridge const void *)obj));
// 编译时，上方两个方法是运行时
NSLog(@"%zd", sizeof(obj));
```

- 结构体内存对齐：最大成员大小的倍数
- iOS 内存对齐：16 的倍数

### 对象的 isa 指针指向哪里？

- instance 对象的指针指向 class 对象
- class 对象的指针指向 meta-class 对象
- meta-class 对象的指针指向基类的 meta-class 对象、

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202160957523.png)

### OC 的类信息存放在哪里？

- 对象方法、属性、成员变量、协议信息，存放在 class 对象中
- 类方法存放在 meta-class 对象中
- 成员变量的具体值，存放在 instance 对象中

---

## KVO

### iOS 使用什么方式实现一个对象的 KVO？（KVO 的本质是什么？）

- 利用 Runtime API 动态生成一个子类（NSKVONotifying_SomeClassName），并且让 instance 对象的 isa 指针指向这个全新的子类
- 当修改 instance 对象的属性时，会调用 Foundation 的 `_NSSet(*: Int、Double、Char 等)ValueAndNotify` 函数，这个函数的实现：

> 1. `willChangeValueForKey:`  
> 2. 父类原来的 setter  
> 3. `didChangeValueForKey:`  

`didChangeValueForKey:` 内部会调用 observer 的 `observerValueForKeyPath:ofObject:change:context` 方法

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202160959104.png)

### 如何手动触发 KVO

- 手动调用 `willChangeValueForKey:` 和 `didChangeValueForKey:`

### 直接修改成员变量的值会触发 KVO 么？

- 不会

---

## KVC

### 通过 KVC 修改属性会触发 KVO 么？

- 会

### `setValue:forKey:` 原理

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161000082.png)

### `valueForKey:` 原理

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161001048.png)

---

## Category

### Category 的原理

* Category 编译之后的底层结构是 `struct category_t`，里面存储着分类的对象方法、类方法、属性、协议信息
* 在程序运行时候，runtime 会将 Category 的数据合并到类信息中（类对象、元类对象中）

### Category 和 Class Extension 的区别

* **Class Extension 是在编译时**，它的数据就已经包含在类信息中
* **Category 是在运行时**，才会将数据合并到类信息中

### +load 方法

- `+load` 方法会在 runtime 加载**类、分类**时调用
- 每个**类、分类**的 `+load` 在程序运行时只调用一次
- 调用顺序
  1. **先调用类的 +load**
    - **按照编译顺序调用**（先编译先调用）
    - 调用子类的 +load 之前会调用父类的 +load
  2. **再调分类的 +load**
    - 按照编译顺序调用（先编译先调用）

- **+load 方法是根据方法地址直接调用，并不是经过 objc_msgSend 函数调用**

### +initialize 方法

- +initialize 方法会在类第一次收到消息时调用
- 调用顺序
  - 先调用父类的 +initialize，再调用子类的 +initialize （先初始化父类，再初始化子类，每个类只初始化一次）
- **如果子类没有实现 +initialize，会调用父类的 +initialize （所以父类的 +initialize 可能会被调用多次）**
- **如果分类实现了 +initialize，就覆盖类本身的 +initialize**

### 给 Category 添加成员变量

- 不能直接给 Category 添加成员变量，但可以通过 **关联对象** 间接实现

#### 关联对象

- 添加关联对象

`void objc_setAssociatedObject(id object, const void * key, id value, objc_AssociationPolicy policy)`

- 获取关联对象

`id objc_getAssociatedObject(id object, const void * key)`

- 移除所有关联对象

`void objc_removeAssociatedObjects(id object)`

#### 关联对象的 key 的常见用法

``` objectivec
static void *MyKey = &MyKey;
objc_setAssociatedObject(obj, MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, MyKey)

static char MyKey;
objc_setAssociatedObject(obj, &MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, &MyKey)

// 使用属性名作为key
objc_setAssociatedObject(obj, @"property", value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
objc_getAssociatedObject(obj, @"property");

// 使用get方法的@selecor作为key
objc_setAssociatedObject(obj, @selector(getter), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, @selector(getter))
```

#### objc_AssociationPolicy

| objc_AssociationPolicy | 对应的修饰符 |
| --- | --- |
| OBJC_ASSOCIATION_ASSIGN | assign |
| OBJC_ASSOCIATION_RETAIN_NONATOMIC | strong, nonatomic |
| OBJC_ASSOCIATION_COPY_NONATOMIC | copy, nonatomic |
| OBJC_ASSOCIATION_RETAIN | strong, atomic |
| OBJC_ASSOCIATION_COPY | copy, atomic |

#### 关联对象的原理

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161003141.png)

---

## Block

### Block 的本质

- block 本质上也是一个 OC 对象
- block 是封装了函数调用和函数调用环境的 OC 对象

### Block 的变量捕获机制

| 变量类型 | 捕获到 block 内部 | 访问方式 |
| --- | :---: | --- |
| 局部变量 auto | ✅ | 值传递 |
| 局部变量 static | ✅ | 指针传递 |
| 全局变量 | ❌ | 直接访问 |

### Block 类型

| block 类型 | 环境 | 内存分配 | 复制效果
| :---: | :---: | :---: | --- |
| `__NSGlobalBlock__` | 没有访问 auto 变量 | 数据区（.data 区）| 什么也不做 |
| `__NSStackBlock__` | 访问了 auto 变量 | 栈 | 从栈复制到堆 |
| `__NSMallocBlock__` | `__NSStackBlock__` 调用了 copy | 堆 | 引用计数增加 |

> 程序区（.text 区） —— 低地址  
> 数据区（.data 区）：全局变量  
> 堆：动态分配  
> 栈 —— 高地址  

### Block 的 copy

在 ARC 环境下，以下情况编译器会自动将栈上的 block 复制到堆上：

- block 作为返回值时
- 将 block 赋值给 __strong 指针时
- block 作为 Cocoa API 中方法名含有 `usingBlock` 的方法参数时
- block 作为 GCD API 的方法参数时

### 对象类型的 auto 变量

当 block 访问了对象类型的 auto 变量时：

- 如果 block 在栈上，将不会对 auto 变量产生强引用
- 如果 block 被拷贝到堆上
  - 会调用 block 内部的 copy 函数
  - copy 函数内部会调用 `_Block_object_assign` 函数
  - `_Block_object_assign` 函数会根据 auto 变量的修饰符（`__strong`, `__weak`, `__unsafe_unretained`）做出相应的操作，形成强引用或弱引用
- 如果 block 从堆上移除
  - 会调用 block 内部的 dispose 函数
  - dispose 函数内部会调用 `_Block_object_dispose` 函数
  - `_Block_object_dispose` 函数会自动释放引用的 auto 变量（release）

### __block 修饰符

- `__block` 可以解决 block 内部无法修改 auto 变量值的问题
- `__block` 不能修饰全局变量、静态变量
- 编译器会将 `__block` 修饰的变量包装成一个对象

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161004760.png)
![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161004077.png)

### __block 的内存管理

- 当 block 在栈上时，并不会对 __block 变量产生强引用
- 当 block 被 copy 到堆时
  - 会调用 block 内部的 copy 函数
  - copy 函数内部会调用 `_Block_object_assign` 函数
  - `_Block_object_assign` 函数会对 __block 变量形成强引用（retain）
- 当 block 从堆中移除时
  - 会调用 block 内部的 dispose 函数
  - dispose 函数内部会调用 `_Block_object_dispose` 函数
  - `_Block_object_dispose` 函数会自动释放引用的 __block 变量（release）

### 被 __block 修饰的对象类型

- 当 __block 变量在栈上时，不会对指向的对象产生强引用
- 当 __block 变量被 copy 到堆时
  - 会调用 __block 变量内部的 copy 函数
  - copy 函数内部会调用 `_Block_object_assign`  函数
  - `_Block_object_assign` 函数会根据所指向对象的修饰符（`__strong`、`__weak`、`__unsafe_unretained`）做出相应的操作，形成强引用（retain）或者弱引用（注意：这里仅限于 ARC 时会 retain，MRC 时不会 retain）
- 如果 __block 变量从堆上移除
  - 会调用 __block 变量内部的 dispose 函数
  - dispose 函数内部会调用 `_Block_object_dispose` 函数
  - `_Block_object_dispose` 函数会自动释放指向的对象（release）

---

## Runtime

### Class 结构

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161005645.png)

### 方法缓存

Class 内部结构中有个方法缓存（cache_t），用**散列表（哈希表）**来缓存曾经调用过的方法，可以提高方法的查找速度

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161005849.png)

### OC 消息机制

OC 中的方法调用，都是转换为 `objc_msgSend` 函数的调用，`objc_msgSend` 的执行流程：

1. 消息发送

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161006685.png)

2. 动态方法解析

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161006903.png)

3. 消息转发

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161006143.png)

### `super` 关键字

super 调用，底层会转换为 `objc_msgSendSuper2` 函数的调用，接收 2 个参数：

1. struct objc_super2
2. SEL

``` objectivec
struct objc_super2 {
  id receiver; // 消息接收者
  Class current_class; // receiver 的 Class 对象
}
```

- **消息接收着仍然是子类对象**
- **从父类开始查找方法的实现**

---

## RunLoop

### RunLoop 本质

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161007756.png)

**CFRunLoopModeRef**
  
  - CFRunLoopModeRef  代表 RunLoop 的运行模式
  - 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source0/Source1/Timer/Observer
  - RunLoop 启动时只能选择其中一个 Mode，作为 currentMode
  - **如果需要切换 Mode，只能退出当前 Loop，再重新选择一个 Mode 进入**
    - 不同组的Source0/Source1/Timer/Observer能分隔开来，互不影响
  - **如果 Mode 里没有任何 Source0/Source1/Timer/Observer，RunLoop 会立马退出**

1. **Source0**
  - 触摸事件处理
  - `performSelector:onThread:`
2. **Source1**
  - 基于 Port 的线程间通信
  - 系统事件捕捉
3. **Timers**
  - NSTimer
  - `performSelector:withObject:afterDelay:`
4. **Observers**
  - 用于监听 RunLoop 的状态
  - UI 刷新（BeforeWaiting）
  - Autorelease pool（BeforeWaiting）

### RunLoop 与线程

- 每条线程都有唯一的一个与之对应的 RunLoop 对象
- RunLoop 保存在一个全局的 Dictionary 里，线程作为 key，RunLoop 作为 value
- 线程刚创建时并没有 RunLoop 对象，**RunLoop 会在第一次获取它时创建**
- RunLoop 会在线程结束时销毁
- 主线程的 RunLoop 已经自动获取（创建），**子线程默认没有开启 RunLoop**

### RunLoop 的运行逻辑

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161008263.png)

### 线程保活

- NSRunLoop 的 `run` 方法是无法停止的
> If no input sources or timers are attached to the run loop, this method exits immediately; otherwise, it runs the receiver in the NSDefaultRunLoopMode by repeatedly invoking  `runMode:beforeDate:`. In other words, this method effectively begins an infinite loop that processes data from the run loop’s input sources and timers.  

---

## 多线程

### 各种队列的执行效果

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161008460.png)

### 请问下面代码的打印结果是什么？

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161008254.png)

- 打印结果是：1、3
- 因为 `performSelector:withObject:afterDelay:` 的本质是往 Runloop 中添加定时器，而子线程默认没有启动 Runloop

### 线程同步

1. **OSSPinLock**
  - `OSSpinLock` 叫做**自旋锁，等待锁的线程会处于忙等（busy-wait）状态**，一直占用着 CPU 资源
  - **目前已不再安全，可能会出现优先级反转问题**
  - 如果等待锁的线程优先级较高，它会一直占用着 CPU 资源，优先级低的线程就无法释放锁
  - 需要导入头文件 `#import <libkern/OSAtomic.h>`

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161009301.png)

2. **os_unfair_lock**
  - 互斥锁
  - `os_unfair_lock` 用于取代不安全的 `OSSpinLock` ，从 iOS 10 开始才支持
  - 等待 `os_unfair_lock` 锁的线程会处于休眠状态，并非忙等
  - 需要导入头文件 `#import <os/lock.h>`

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161009209.png)

3. **pthread_mutex** 
  - `mutex` 叫做”互斥锁”，等待锁的线程会处于休眠状态
  - 需要导入头文件 `#import <pthread.h>`

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161009346.png)

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161009567.png)

  - **pthread_mutex – 递归锁** 

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161010697.png)

  - **pthread_mutex – 条件** 

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161010454.png)

4. **NSLock、NSRecursiveLock** 
  - `NSLock` 是对 `mutex` 普通锁的封装

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161010366.png)

  - `NSRecursiveLock` 是对 mutex 递归锁的封装，API 跟 NSLock 基本一致

5. **NSCondition** 

  - `NSCondition` 是对 mutex 和 cond 的封装

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161011849.png)
    
6. **NSConditionLock** 

  - `NSConditionLock` 是对 `NSCondition` 的进一步封装，可以设置具体的条件值

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161011803.png)

7. **dispatch_semaphore** 
  
  - semaphore 叫做”信号量”
  - 信号量的初始值，可以用来控制线程并发访问的最大数量
  - 信号量的初始值为 1，代表同时只允许 1 条线程访问资源，保证线程同步

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161011979.png)

8. **dispatch_queue** 

  - 直接使用 GCD 的串行队列，也是可以实现线程同步的

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161012258.png)

9. **@synchronized** 

  - `@synchronized` 是对 mutex 递归锁的封装
  - 源码查看：objc4 中的 `objc-sync.mm` 文件
  - `@synchronized(obj)` 内部会生成 obj 对应的递归锁，然后进行加锁、解锁操作

10. **pthread_rwlock**  读写锁

  - 等待锁的线程会进入休眠

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161012025.png)

11. **dispatch_barrier_async** 

  - **这个函数传入的并发队列必须是自己通过 dispatch_queue_cretate 创建的**

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161012194.png)

- **iOS 中的读写安全方案 10、11**

###  线程同步方案性能比较

- 性能从高到低排序

> os_unfair_lock  
> OSSpinLock  
> dispatch_semaphore  
> pthread_mutex  
> dispatch_queue(DISPATCH_QUEUE_SERIAL)  
> NSLock  
> NSCondition  
> pthread_mutex(recursive)  
> NSRecursiveLock  
> NSConditionLock  
> @synchronized  

---

## Timer

### CADisplayLink、NSTimer、dispatch_source_t

- `CADisplayLink`、`NSTimer` 会对 target 产生强引用，如果 target 又对它们产生强引用，那么就会引发循环引用，解决方案：

    1. 使用 block

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161013253.png)

    2. 使用 NSProxy

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161013164.png)

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161013391.png)

- `NSTimer` 依赖于 RunLoop，如果 RunLoop 的任务过于繁重，可能会导致 NSTimer 不准时，而 GCD 的定时器会更加准时

    ![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161014689.png)

---

## 内存管理

### iOS 程序的内存布局

![](https://raw.githubusercontent.com/KiligWYu/Pics/master/202202161014281.png)

### Tagged Pointer

- 从 64bit 开始，iOS 引入了 Tagged Pointer 技术，用于优化 NSNumber、NSDate、NSString 等小对象的存储
- 在没有使用 Tagged Pointer 之前， NSNumber 等对象需要动态分配内存、维护引用计数等，NSNumber 指针存储的是堆中 NSNumber 对象的地址值
- 使用 Tagged Pointer 之后，NSNumber 指针里面存储的数据变成了：Tag + Data，也就是**将数据直接存储在了指针中**
- 当指针不够存储数据时，才会使用动态分配内存的方式来存储数据
- `objc_msgSend` 能识别 Tagged Pointer，比如 NSNumber 的 `intValue` 方法，直接从指针提取数据，节省了以前的调用开销
- 如何判断一个指针是否为 Tagged Pointer？
  - iOS 平台，最高有效位是 1（第 64bit）
  - Mac 平台，最低有效位是 1
