---
title: nRF设备DFU升级
date: 2016-12-28 22:17:59
tags: 蓝牙
categories: 蓝牙总结
---
![Nordic.png](http://upload-images.jianshu.io/upload_images/1059465-1afdc502de9bec75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>这里主要参考这个项目：[iOS-nRF-Toolbox](https://github.com/NordicSemiconductor/IOS-nRF-Toolbox)，它是Nordic公司开发的测试工程，包含一整套nRF设备的测试解决方案。

项目是用Swift写的，不过之前还是有OC版本的，但是后来由于一些**（不可描述的问题），才变成了现在的纯Swift版本。对于使用Swift开发的人员，直接仿照Demo操作即可。如果你是用Swift开发的，那下面的内容你可以不用看了。接下来我就讲一下针对OC引用DFU升级的操作步骤和我遇到的问题。
<!--more-->
## 代码研究
nRF-Toolbox项目包含BGM，HRM，HTM，DFU等多个模块，我们今天只关注其中的DFU升级模块。打开项目，在对应的`NORDFUViewController.swift`中我们能够看到有三个引用库
`import UIKit`,`import CoreBluetooth`,`import iOSDFULibrary`，这里的[iOSDFULibrary](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library)就是DFU升级的库，也是解决DFU升级最重要的组件。我们只要把这个库集成到我们的项目中，就能够完成nRF设备的DFU升级了。

## 集成步骤
有两种方案集成：
* 通过cocoapods集成
* 编译出framework然后把库导入项目

第一种方案是作者推荐的，但是我试了很久，引入DFULibrary会出现头文件找不到等一系列问题，无奈只能放弃，如果有人通过这种方式成功，还望告知。下面讲的是通过第二种方案的集成。

**第一步：导出iOSDFULibrary**

这一步是最关键也是最容易出问题的，这个库也是由Swift写成的，我们将这个库clone到本地，然后选择iOSDFULibrary进行编译

![01.png](http://upload-images.jianshu.io/upload_images/1059465-163cdfb2d7b5da14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后生成两个framework:
* iOSDFULibrary.framework
* Zip.framework

这时库内的代码已经变成了我们熟悉的OC语言。理论上这个库应该是没问题的了，但是事实还是有问题的，见[issues#39](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library/issues/39)。作者给出的解决方法是：

![02.png](http://upload-images.jianshu.io/upload_images/1059465-463d5757a37b71f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[carthage](https://github.com/Carthage/Carthage#installing-carthage)是一种和cocoapods相似的的类库管理工具，如果不会使用的话可以参照Demo，将framework文件导入到自己的项目。

**第二步、导入framework**
Target->General
![128F494E-C863-49E7-AC44-A7B53B3EB463.png](http://upload-images.jianshu.io/upload_images/1059465-9633f3e7feab7f5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接拖入项目默认只会导入到`Linked Frameworks and Libraries`，我们还需要在Embeded Binaries中引入。

**第三步、使用iOSDFULibrary**

    //create a DFUFirmware object using a NSURL to a Distribution Packer(ZIP)
    DFUFirmware *selectedFirmware = [[DFUFirmware alloc] initWithUrlToZipFile:url];// or
    //Use the DFUServiceInitializer to initialize the DFU process.
    DFUServiceInitiator *initiator = [[DFUServiceInitiator alloc] initWithCentralManager: centralManager target:selectedPeripheral];
    [initiator withFirmware:selectedFirmware];
    // Optional:
    // initiator.forceDfu = YES/NO; // default NO
    // initiator.packetReceiptNotificationParameter = N; // default is 12
    initiator.logger = self; // - to get log info
    initiator.delegate = self; // - to be informed about current state and errors 
    initiator.progressDelegate = self; // - to show progress bar
    // initiator.peripheralSelector = ... // the default selector is used

    DFUServiceController *controller = [initiator start];

库中有三个代理方法`DFUProgressDelegate`，`DFUServiceDelegate`，`LoggerDelegate`，它们的作用分别为监视DFU升级进度，DFU升级及蓝牙连接状态，打印状态日志。

## 完整OC项目
这个是对应Swift版本用OC写的完整项目，应该是OC停止维护之前的版本。会有一些bug。在将DFUFramework更新之后，我把它搬到了我的github上，有需要的同学可以下载研究：[OC-nRFTool-box](https://github.com/zhangferry/nRF-Toolbox)。
