---
title: 面试题总结（From J_Knight）
date: 2017-06-29 17:22:27
tags: iOS知识点
categories: iOS

---

>前一段时间看了[J_Knight](https://knightsj.github.io/)的[2017年5月iOS找人心得（附面试题）](http://www.jianshu.com/p/56e40ea56813)。作为一个在编程前线奋斗了将近两年的iOS从业人员，面对这些题目时，有些竟感觉生疏，甚至答不上来，很是惭愧。个人感觉，像runtime、线程、信号量相关的偏底层知识虽然平时基本用不到。特别是很多人可能都没参与过稍复杂项目的开发，优化，这些内容对于很多新手iOS开发来说只存在于理论。但并不是说这些知识不重要，相反，它是我们进阶的必经之路。此篇文章的目的一方面自己整理，一方面希望和大家共同学习进步。以下内容多数为整理，时间仓促可能有不准确的地方，如果缺漏，欢迎指正。

部分答案出处：[iOSInterviewQuestions](https://github.com/ChenYilong/iOSInterviewQuestions)
<!--more-->
# 基础
## 为什么说Objective-C是一门动态语言？

动态语言，是指程序在运行时可以改变其结构：新的函数可以被引进，已有的函数可以被删除等在结构上的变化。比如Ruby、Python等就是动态语言，而C、C++等语言则不属于动态语言。
所谓的动态类型语言，意思就是类型的检查是在运行时做的。

1、动态类型。 如id类型。实际上静态类型因为其固定性和可预知性而使用得更加广泛。静态类型是强类型，而动态类型属于弱类型。运行时决定接收者。

2、动态绑定。让代码在运行时判断需要调用什么方法，而不是在编译时。与其他面向对象语言一样，方法调用和代码并没有在编译时连接在一起，而是在消息发送时才进行连接。运行时决定调用哪个方法。

3、动态载入。让程序在运行时添加代码模块以及其他资源。用户可以根据需要加载一些可执行代码和资源，而不是在启动时就加载所有组件。可执行代码中可以含有和程序运行时整合的新类。

## 讲一下MVC和MVVM,MVP
[iOS 架构模式--解密 MVC，MVP，MVVM以及VIPER架构](http://www.cocoachina.com/ios/20160108/14916.html)
## 为什么代理要用weak？代理的delegate和dataSource有什么区别？block和代理的区别?
防止循环引用。

另外，不建议使用assign
weak 当计数器为0 时对象被释放，地址指针就置为了nil 了。 
assign 当计数器为0 时 对象被释放，地址指针还是指向那个地址，就会产生野指针

datasource协议里面东西是跟内容有关的，主要是cell的构造函数，各种属性
delegate协议里面的方法主要是操作相关的，移动编辑之类的，你都写上要用什么方法自己去翻就是了 
delegate控制的是UI，是上层的东西；而datasource控制的是数据。他们本质都是回调，只是回调的对象不同。

block 和 delegate 都可以通知外面。block 更轻型，使用更简单，能够直接访问上下文，这样类中不需要存储临时数据，使用 block 的代码通常会在同一个地方，这样读代码也连贯。delegate 更重一些，需要实现接口，它的方法分离开来，很多时候需要存储一些临时数据，另外相关的代码会被分离到各处，没有 block 好读。

多个相关方法，避免循环引用，建议用delegate。
临时性的，只在栈中，需要存储，只调用一次，一个完成周期用block

## 属性的实质是什么？包括哪几个部分？属性默认的关键字都有哪些？@dynamic关键字和@synthesize关键字是用来做什么的？
属性的本质就是实现实例变量和存取方法。
    @property = ivar + getter + setter;
1. 对应基本数据类型默认关键字是`atomic,readwrite,assign `
2. 对于普通的 Objective-C 对象`atomic,readwrite,strong`

`@property`有两个对应的词，一个是 `@synthesize`，一个是 `@dynamic`。如果 `@synthesize`和 `@dynamic`都没写，那么默认的就是`@syntheszie var = _var;`
`@synthesize` 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。
`@dynamic` 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 `@dynamic var`，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 `instance.var = someVar`，由于缺 setter 方法会导致程序崩溃；或者当运行到 `someVar = var` 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

## NSString为什么要用copy关键字，如果用strong会有什么问题？
因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本。

如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.

## 如何令自己所写的对象具有拷贝功能?
若想令自己所写的对象具有拷贝功能，则需实现 `NSCopying` 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 `NSCopying` 与 `NSMutableCopying` 协议。

## 可变集合类 和 不可变集合类的 copy 和 mutablecopy有什么区别？如果是集合是内容复制的话，集合里面的元素也是内容复制么？
```
    [immutableObject copy] // 浅复制
    [immutableObject mutableCopy] //深复制
    [mutableObject copy] //深复制
    [mutableObject mutableCopy] //深复制```
但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制

## 为什么IBOutlet修饰的UIView也适用weak关键字？
因为既然有外链那么视图在xib或者storyboard中肯定存在，视图已经对它有一个强引用了。
不过这个回答漏了个重要知识，使用storyboard（xib不行）创建的vc，会有一个叫`_topLevelObjectsToKeepAliveFromStoryboard`的私有数组强引用所有top level的对象，所以这时即便outlet声明成weak也没关系

## nonatomic和atomic的区别？atomic是绝对的线程安全么？为什么？如果不是，那应该如何实现？
`atomic`会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明 `nonatomic` 可以节省这些虽然很小但是不必要额外开销。

一般情况下并不要求属性必须是“原子的”，因为这并不能保证“线程安全” ( thread safety)，若要实现“线程安全”的操作，还需采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为 `atomic`，也还是会读到不同的属性值。

## UICollectionView自定义layout如何实现？
* 新建一个类继承`UICollectionViewFlowLayout`，实现`prepareLayout`方法。
* 新建`UICollectionViewFlowLayout`类，设置属性。
* 实现`UICollectionViewDelegateFlowLayout`方法。

## 用StoryBoard开发界面有什么弊端？如何避免？
* 难以维护  
* 性能瓶颈  
* 错误定位不准确

解决多Storyboard协作弊端，就是尽量将项目的界面分割在多个Storyboard文件中。一个最佳实践是，按照项目功能模块来区分故事板，例如`Login.Storyboard,Chat.Storyboard,Person.Storyboard`等。尽量把每个`Storyboard`的Scene数量控制在20个以内


## 进程和线程的区别？同步异步的区别？并行和并发的区别？
一个进程可以包括多个线程。一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。
一个程序至少有一个进程,一个进程至少有一个线程. 线程只能归属于一个进程并且它只能访问该进程所拥有的资源。

同步会造成阻塞，异步非阻塞，网络请求操作。

一个形象的例子：
你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。并发的关键是你有处理多个任务的能力，不一定要同时。并行的关键是你有同时处理多个任务的能力。所以我认为它们最关键的点就是：是否是『同时』。
“并行”概念是“并发”概念的一个子集

## 线程间的通信
`NSThread`可以先将自己的当前线程对象注册到某个全局的对象中去，这样相互之间就可以获取对方的线程对象，然后就可以使用下面的方法进行线程间的通信了，由于主线程比较特殊，所以框架直接提供了在主线程执行的方法

## GCD的一些常用的函数？（group，barrier，信号量，线程同步）
* group
队列组通知监听函数(异步函数)dispatch_group_notify
队列组等待函数(同步函数)dispatch_group_wait
应用场景:下载两张图片，拼接图片后到主线程中刷新

* barrier
栅栏函数执行顺序：栅栏函数之前的任务(执行完毕)--> 栅栏函数的任务(执行完毕)--> 栅栏函数之后的任务
栅栏函数前面和后面追加的操作执行顺序都不固定，但是前面的三个输出操作必然先执行，然后再执行栅栏函数中的操作，最后执行后面的三个输出操作。
[栅栏函数](http://www.jianshu.com/p/d63c3100dd63)

* 信号量
信号量大小是用于控制并发数量的
信号量就是一个资源计数器，对信号量有两个操作来达到互斥，分别是P和V操作。 一般情况是这样进行临界访问或互斥访问的： 
设信号量值为1， 当一个进程1运行是，使用资源，进行P操作，即对信号量值减1，也就是资源数少了1个。这是信号量值为0。系统中规定当信号量值为0是，必须等待，知道信号量值不为零才能继续操作。 这时如果进程2想要运行，那么也必须进行P操作，但是此时信号量为0，所以无法减1，即不能P操作，也就阻塞。这样就到到了进程1排他访问。 当进程1运行结束后，释放资源，进行V操作。资源数重新加1，这是信号量的值变为1. 这时进程2发现资源数不为0，信号量能进行P操作了，立即执行P操作。信号量值又变为0.次数进程2咱有资源，排他访问资源。 这就是信号量来控制互斥的原理

* 线程同步：
线程同步：@synchronized  NSLock  dispatch_semaphore(信号量设置为1)

## 如何使用队列来避免资源抢夺？
当我们使用多线程来访问同一个数据的时候，就有可能造成数据的不准确性。这个时候我么可以使用线程锁的来来绑定。也是可以使用串行队列来完成。
如：fmdb就是使用`FMDatabaseQueue`，来解决多线程抢夺资源。

## 数据持久化的几个方案（fmdb用没用过）
* plist
* CoreData 
* FMDB

## 说一下AppDelegate的几个方法？从后台到前台调用了哪些方法？第一次启动调用了哪些方法？从前台到后台调用了哪些方法？
    ```//第一次启动：
    didFinishLaunchingWithOptions:
    applicationDidBecomeActive:
    //前台到后台：
    applicationWillResignActive:
    applicationDidEnterBackground:
    //后台到前台：
    applicationWillEnterForeground:
    applicationDidBecomeActive:```
## NSCache优于NSDictionary的几点？
* 当系统资源将要耗尽时，NSCache具备自动删减缓冲的功能。并且还会先删减“最久未使用”的对象。
* NSCache不拷贝键，而是保留键。因为并不是所有的键都遵从拷贝协议（字典的键是必须要支持拷贝协议的，有局限性）。
* NSCache是线程安全的：不编写加锁代码的前提下，多个线程可以同时访问NSCache。

## 知不知道Designated Initializer？使用它的时候有什么需要注意的问题？
便利初始化函数只能调用自己类中的其他初始化方法
指定初始化函数才有资格调用父类的指定初始化函数
[构造便利函数](http://www.cnblogs.com/smileEvday/p/designated_initializer.html)

## 实现description方法能取到什么效果？
`description`方法默认返回对象的描述信息(默认实现是返回类名和对象的内存地址)，可定义输出自己想要的内容。

## objc使用什么机制管理对象内存？
1.Objective-C中所有对象都在堆区建立，由程序员负责释放对象所占用的内存。内存管理机制由3种：垃圾回收、引用计数、C语言方式。

2.垃圾回收是Mac OS10.5提供的新方案，在系统存在一个垃圾收集器。如果发现某个对象没有被任何对象使用，该对象被自动释放。

3.C语言方式，原始内存管理方式。用户手动调用malloc、calloc函数分配内存，free回收内存。

4.引用计数机制：对象创建后，运行时系统通过对象维护的一个计数器来描述有多少个其他对象在使用自己，当计数器为0时，释放该对象占用的内存空间（该对象调用dealloc方法）。

5,内存管理规则：当使用alloc，new或copy创建一个对象时，对象的引用计数被设置为1.；向对象发送retain消息，对象引用计数加1；向对象发送release消息时，对象引用计数减1；当对象引用计数为0时，运行时系统向对象发送dealloc消息并回收对象所占用的内存。

# 中级
## Block
### block的实质是什么？一共有几种block？都是什么情况下生成的？
Block是iOS开发中一种比较特殊的数据结构，它可以保存一段代码，在合适的地方再调用，具有语法简介、回调方便、编程思路清晰、执行效率高等优点，受到众多猿猿的喜爱。
* _NSConcreteGlobalBlock: 存储在全局数据区
* _NSConcreteStackBlock: 存储在栈区
* _NSConcreteMallocBlock: 存储在堆区
其中，`_NSConcreteGlobalBlock` 和 `_NSConcreteStackBlock` 可以由程序创建，而 `_NSConcreteMallocBlock` 则无法由程序创建，只能由 `_NSConcreteStackBlock` 通过拷贝生成。

### 为什么在默认情况下无法修改被block捕获的变量？ __block都做了什么？
Block不允许修改外部变量的值。Apple这样设计，应该是考虑到了block的特殊性，block也属于“函数”的范畴，变量进入block，实际就是已经改变了作用域。在几个作用域之间进行切换时，如果不加上这样的限制，变量的可维护性将大大降低。
又比如我想在block内声明了一个与外部同名的变量，此时是允许呢还是不允许呢？只有加上了这样的限制，这样的情景才能实现。于是栈区变成了红灯区，堆区变成了绿灯区。
将变量由栈区移到堆区。

### 模拟一下循环引用的一个情况？block实现界面反向传值如何实现？
一个对象中强引用了block，在block中又强引用了该对象，就会发射循环引用。
解决方法是将该对象使用`__weak`或者`__block`修饰符修饰之后再在block中使用.
`id weak weakSelf = self;` 或者 `weak __typeof(&*self)weakSelf = self`该方法可以设置宏
`id __block weakSelf = self;`
或者将其中一方强制制空 `xxx = nil`。

## Runtime
### objc在向一个对象发送消息时，发生了什么？
1. 通过对象的isa指针获取类的结构体。
2. 在结构体的方法表里查找方法的selector。
3. 如果没有找到selector，则通过objc_msgSend结构体中指向父类的指针找到父类，并在父类的方法表里查找方法的selector。
4. 依次会一直找到NSObject。
5. 一旦找到selector，就会获取到方法实现IMP。
6. 传入相应的参数来执行方法的具体实现。
7. 如果最终没有定位到selector，就会走消息转发流程。

### 什么时候会报unrecognized selector错误？iOS有哪些机制来避免走到这一步？
找不到执行方法。
* 动态方法解析
对象接收到未知的消息时，首先会调用所属类的类方法`+resolveInstanceMethod:`(实例方法)或 者`+resolveClassMethod:`(类方法)。
* 备用接收者
如果这个方法返回一个对象，则这个对象会作为消息的新接收者。注意这个对象不能是self自身，否则就是出现无限循环。如果没有指定对象来处理`aSelector`，则应该 `return [super forwardingTargetForSelector:aSelector]`。
* 完整消息转发
这是最后一次机会将消息转发给其它对象。创建一个表示消息的`NSInvocation`对象，把与消息的有关全部细节封装在`anInvocation`中，包括`selector`，目标(target)和参数。在`forwardInvocation` 方法中将消息转发给其它对象。

### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？
不能向编译后得到的类中增加实例变量；
能向运行时创建的类中添加实例变量；
解释下：
因为编译后的类已经注册在 runtime 中，类结构体中的 `objc_ivar_list` 实例变量的链表 和 `instance_size` 实例变量的内存大小已经确定，同时runtime 会调用 `class_setIvarLayout` 或 `class_setWeakIvarLayout` 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；

运行时创建的类是可以添加实例变量，调用 `class_addIvar` 函数。但是得在调用 `objc_allocateClassPair` 之后，`objc_registerClassPair`之前，原因同上。

### runtime如何实现weak变量的自动置nil？
runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

### 给类添加一个属性后，在类结构体里哪些元素会发生变化？
![Class 结构.jpg](http://upload-images.jianshu.io/upload_images/1059465-47afdfd1950a72c6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`instance_size` ：实例的内存大小
`objc_ivar_list *ivars`:属性列表

## RunLoop
### runloop是来做什么的？runloop和线程有什么关系？主线程默认开启了runloop么？子线程呢？

循环检测线程任务。为了在我们的应用可以在无人操作的时候休息，需要让它干活的时候又能立马响应。

1. 主线程的run loop默认是启动的。
2. 对其它线程来说，run loop默认是没有启动的，如果你需要更多的线程交互则可以手动配置和启动，如果线程只是去执行一个长时间的已确定的任务则不需要。

3. 在任何一个 Cocoa 程序的线程中，都可以通过以下代码来获取到当前线程的 run loop 。

    NSRunLoop *runloop = [NSRunLoop currentRunLoop];

### runloop的mode是用来做什么的？有几种mode？
model 主要是用来指定事件在运行循环中的优先级的，分为：

`NSDefaultRunLoopMode（kCFRunLoopDefaultMode）`：默认，空闲状态
`UITrackingRunLoopMode：ScrollView`滑动时
`UIInitializationRunLoopMode：`启动时
`NSRunLoopCommonModes（kCFRunLoopCommonModes）：`Mode集合
苹果公开提供的 Mode 有两个：
    NSDefaultRunLoopMode（kCFRunLoopDefaultMode）
    NSRunLoopCommonModes（kCFRunLoopCommonModes）
为什么把NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环以后，滑动scrollview的时候NSTimer却不动了？
RunLoop只能运行在一种mode下，如果要换mode，当前的loop也需要停下重启成新的。利用这个机制，ScrollView滚动过程中`NSDefaultRunLoopMode（kCFRunLoopDefaultMode）`的mode会切换到`UITrackingRunLoopMode`来保证ScrollView的流畅滑动：只能在`NSDefaultRunLoopMode`模式下处理的事件会影响ScrollView的滑动。

如果我们把一个NSTimer对象以`NSDefaultRunLoopMode（kCFRunLoopDefaultMode）`添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。

同时因为mode还是可定制的，所以：

Timer计时会被scrollView的滑动影响的问题可以通过将timer添加到`NSRunLoopCommonModes（kCFRunLoopCommonModes）`来解决。

    //将timer添加到NSDefaultRunLoopMode中
    [NSTimer scheduledTimerWithTimeInterval:1.0
                                     target:self
                                   selector:@selector(timerTick:)
                                   userInfo:nil
                                    repeats:YES];
    //然后再添加到NSRunLoopCommonModes里
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0
                                             target:self
                                           selector:@selector(timerTick:)
                                           userInfo:nil
                                            repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

### 苹果是如何实现Autorelease Pool的？
autoreleasepool 以一个队列数组的形式实现,主要通过下列三个函数完成.

`objc_autoreleasepoolPush`
`objc_autoreleasepoolPop`
`objc_autorelease`
看函数名就可以知道，对 autorelease 分别执行 push，和 pop 操作。销毁对象时执行release操作。

举例说明：我们都知道用类方法创建的对象都是 Autorelease 的，那么一旦 Person 出了作用域，当在 Person 的 dealloc 方法中打上断点，我们就可以看到这样的调用堆栈信息：
![autorealease pool.jpg](http://upload-images.jianshu.io/upload_images/1059465-b8314aaf33c5dd27.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
备注：[Objective-C Autorelease Pool 的实现原理](http://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/) 可能苹果在ARC的处理上又优化了，或者变更了。我自己测试的结果跟以上分析不一致，但原理可以参考。

## 类结构
### isa指针？（对象的isa，类对象的isa，元类的isa都要说）
每一个对象内部都有一个isa指针,指向他的类对象,类对象中存放着本对象的
* 对象方法列表（对象能够接收的消息列表，保存在它所对应的类对象中）
* 成员变量的列表,
* 属性列表,
它内部也有一个isa指针指向元对象(meta class),元对象内部存放的是类方法列表,类对象内部还有一个superclass的指针,指向他的父类对象。
类对象既然称为对象，那它也是一个实例。类对象中也有一个isa指针指向它的元类(meta class)，即类对象是元类的实例。元类内部存放的是类方法列表，根元类的isa指针指向自己，superclass指针指向NSObject类。

### 类方法和实例方法有什么区别？

* 类方法：
类方法是属于类对象的
类方法只能通过类对象调用
类方法中的self是类对象
类方法可以调用其他的类方法
类方法中不能访问成员变量
类方法中不能直接调用对象方法

* 实例方法：
实例方法是属于实例对象的
实例方法只能通过实例对象调用
实例方法中的self是实例对象
实例方法中可以访问成员变量
实例方法中直接调用实例方法
实例方法中也可以调用类方法(通过类名)

### 介绍一下分类，能用分类做什么？内部是如何实现的？它为什么会覆盖掉原来的方法？
分类可以在不知道系统类源代码的情况下，为这个类添加新的方法。分类只能用来添加方法，不能添加成员变量。通过分类增加的方法，系统会认为是该类类型的一部分
[Category实现原理](http://blog.leichunfeng.com/blog/2015/05/18/objective-c-category-implementation-principle/)

### 运行时能增加成员变量么？能增加属性么？如果能，如何增加？如果不能，为什么？
可以添加属性，不可以添加成员变量。
[OC类成员变量深度剖析](http://www.cocoachina.com/ios/20150526/11918.html)
### objc中向一个nil对象发送消息将会发生什么？（返回值是对象，是标量，结构体）

如果一个方法返回值是一个对象，那么发送给nil的消息将返回0(nil)。例如：

    Person *motherInlaw = [[aPerson spouse] mother];
1. 如果 spouse 对象为 nil，那么发送给 nil 的消息 mother 也将返回 nil。 
2. 如果方法返回值为指针类型，其指针大小为小于或者等于`sizeof(void*)，float，double，long double` 或者 `long long` 的整型标量，发送给 nil 的消息将返回0。 
3. 如果方法返回值为结构体,发送给 nil 的消息将返回0。结构体中各个字段的值将都是0。 
4. 如果方法的返回值不是上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的。

# 高级
## UITableview的优化方法（缓存高度，异步绘制，减少层级，hide，避免离屏渲染）
一般在网络请求结束后，在更新界面之前就把每个 cell 的高度算好，缓存到相对应的 model 中。

另外绘制 cell 不建议使用 UIView，建议使用 CALayer。
简单的形式参考：

    ```//异步绘制
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        CGRect rect = CGRectMake(0, 0, 100, 100);
        UIGraphicsBeginImageContextWithOptions(rect.size, YES, 0);
        CGContextRef context = UIGraphicsGetCurrentContext();
        [[UIColor lightGrayColor] set];
        CGContextFillRect(context, rect);

        //将绘制的内容以图片的形式返回，并调主线程显示
        UIImage *temp = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();

        // 回到主线程
        dispatch_async(dispatch_get_main_queue(), ^{
            //code
        });
    });```

CALayer 的 border、圆角、阴影、遮罩（mask），`CASharpLayer` 的矢量图形显示，通常会触发离屏渲染（offscreen rendering），而离屏渲染通常发生在 GPU 中。当一个列表视图中出现大量圆角的 `CALayer`，并且快速滑动时，可以观察到 GPU 资源已经占满，而 CPU 资源消耗很少。这时界面仍然能正常滑动，但平均帧数会降到很低。为了避免这种情况，可以尝试开启 `CALayer.shouldRasterize` 属性，但这会把原本离屏渲染的操作转嫁到 CPU 上去。对于只需要圆角的某些场合，也可以用一张已经绘制好的圆角图片覆盖到原本视图上面来模拟相同的视觉效果。最彻底的解决办法，就是把需要显示的图形在后台线程绘制为图片，避免使用圆角、阴影、遮罩等属性。





