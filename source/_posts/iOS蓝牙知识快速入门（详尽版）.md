---
title: iOS蓝牙知识快速入门（详尽版）
date: 2017-01-13 22:36:41
tags: 
    -蓝牙
categories: 蓝牙总结

---

![iOS-bluetooth](http://upload-images.jianshu.io/upload_images/1059465-d8149efe4c37d452.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以前写过几篇蓝牙相关的文章，但是没有涉及扫描、收发指令这些基础功能的实现。所以打算写一篇尽可能详尽的蓝牙知识汇总，一方面给有需要的同学看，一方面是对自己学习蓝牙的一个总结。

这篇文章的目的：教你实现设备的扫描，连接，数据收发，蓝牙数据解析。如果在实现上面任一功能遇到问题时，欢迎留下你的问题，我将进行补充，对于说法有误的地方也请老司机予以指正。
<!--more-->
## 目录
> [0、思维导图](#thought)
> [1、苹果对蓝牙设备有什么要求](#1)
> [2、操作蓝牙设备使用什么库](#2)
> [3、如何扫描](#3)
> [4、如何连接](#4)
> [5、如何发送数据和接收数据](#5)
> [6、如何解析数据](#6)
> [7、扩展](#7)

## 思维导图
![思维导图](http://upload-images.jianshu.io/upload_images/1059465-46a0e68cfd9c4c9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第一次做图，大家凑合着看哈。这张是我总结的蓝牙知识的结构图，下面的内容将围绕这些东西展开进行。
![连接设备流程](http://upload-images.jianshu.io/upload_images/1059465-18ebcb6eb1790121.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这张是蓝牙连接发送数据的流程图，下文进入coding阶段的讲解顺序，大家先有个大概印象，等阅读完本文再回来看这张图将理解的更深一些。

## 苹果对蓝牙设备有什么要求

BLE：bluetouch low energy，蓝牙4.0设备因为低功耗，所有也叫作BLE。苹果在iphone4s及之后的手机型号开始支持蓝牙4.0，这也是最常见的蓝牙设备。低于蓝牙4.0协议的设备需要进行MFI认证，关于MFI认证的申请工作可以看这里：[关于MFI认证你所必须要知道的事情](http://www.jianshu.com/p/b90b0c45398d)

在进行操作蓝牙设备前，我们先下载一个蓝牙工具`LightBlue`，它可以辅助我们的开发，在进行蓝牙开发之前建议先熟悉一下[LightBlue](https://itunes.apple.com/us/app/lightblue-explorer-bluetooth/id557428110?mt=8)这个工具。

## 操作蓝牙设备使用什么库
苹果自身有一个操作蓝牙的库`CoreBluetooth.framework`，这个是大多数人员进行蓝牙开发的首选框架，除此之外目前github还有一个比较流行的对原生框架进行封装的三方库[BabyBluetooth](https://github.com/coolnameismy/BabyBluetooth)，它的机制是将CoreBluetooth中众多的delegate写成了block方法，有兴趣的同学可以了解下。下面主要介绍的是原生蓝牙库的知识。

### 中心和外围设备

![central-peripheral](http://upload-images.jianshu.io/upload_images/1059465-d4c7cabd9f2ca98a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图所示，电脑、Pad、手机作为中心，心跳监听器作为外设，这种中心外设模式是最常见的。简单理解就是，发起连接的是中心设备（Central），被连接的是外围设备（Peripheral），对应传统的客户机-服务器体系结构。Central能够扫描侦听到，正在播放广告包的外设。

### 服务与特征
外设可以包含一个或多个服务（CBService），服务是用于实现装置的功能或特征数据相关联的行为集合。
而每个服务又对应多个特征（CBCharacteristic）,特征提供外设服务进一步的细节，外设，服务，特征对应的数据结构如下所示

![CBService-CBCharacteristic](http://upload-images.jianshu.io/upload_images/1059465-bbcf073456d1f3be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 如何扫描蓝牙
在进行扫描之前我们需要，首先新建一个类作为蓝牙类，例如`FYBleManager`，写成单例，作为处理蓝牙操作的管理类。引入头文件`#import <CoreBluetooth/CoreBluetooth.h>`
`CBCentralManager`是蓝牙中心的管理类，控制着蓝牙的扫描，连接，蓝牙状态的改变。

### 1、初始化
    dispatch_queue_t centralQueue = dispatch_queue_create(“centralQueue",DISPATCH_QUEUE_SERIAL);
        NSDictionary *dic = @{CBCentralManagerOptionShowPowerAlertKey : YES,
        CBCentralManagerOptionRestoreIdentifierKey : @"unique identifier"
    };
    self.centralManager = [[CBCentralManager alloc] initWithDelegate:self queue:centralQueue options:dic];

`CBCentralManagerOptionShowPowerAlertKey`对应的BOOL值，当设为YES时，表示CentralManager初始化时，如果蓝牙没有打开，将弹出Alert提示框
`CBCentralManagerOptionRestoreIdentifierKey`对应的是一个唯一标识的字符串，用于蓝牙进程被杀掉恢复连接时用的。

### 2、扫描

    //不重复扫描已发现设备        
    NSDictionary *option = @{CBCentralManagerScanOptionAllowDuplicatesKey : [NSNumber numberWithBool:NO],CBCentralManagerOptionShowPowerAlertKey:YES};        
    [self.centralManager scanForPeripheralsWithServices:nil options:option];
    - (void)scanForPeripheralsWithServices:(nullable NSArray<CBUUID *> *)serviceUUIDs options:(nullable NSDictionary<NSString *, id> *)options;
扫面方法，`serviceUUIDs`用于第一步的筛选，扫描此UUID的设备
options有两个常用参数：`CBCentralManagerScanOptionAllowDuplicatesKey`设置为NO表示不重复扫瞄已发现设备，为YES就是允许。`CBCentralManagerOptionShowPowerAlertKey`设置为YES就是在蓝牙未打开的时候显示弹框

### 3、CBCentralManagerDelegate代理方法

在初始化的时候我们调用了代理，在CoreBluetooth中有两个代理，
* CBCentralManagerDelegate
* CBPeripheralDelegate

iOS的命名很友好，我们通过名字就能看出，上面那个是关于中心设备的代理方法，下面是关于外设的代理方法。我们这里先研究`CBCentralManagerDelegate`中的代理方法

    - (void)centralManagerDidUpdateState:(CBCentralManager *)central;
这个方法标了`@required`是必须添加的，我们在self.centralManager初始换之后会调用这个方法，回调蓝牙的状态。状态有以下几种：

    typedef NS_ENUM(NSInteger, CBCentralManagerState{
        CBCentralManagerStateUnknown = CBManagerStateUnknown,//未知状态
        CBCentralManagerStateResetting = CBManagerStateResetting,//重启状态
        CBCentralManagerStateUnsupported = CBManagerStateUnsupported,//不支持
        CBCentralManagerStateUnauthorized = CBManagerStateUnauthorized,//未授权
        CBCentralManagerStatePoweredOff = CBManagerStatePoweredOff,//蓝牙未开启
        CBCentralManagerStatePoweredOn = CBManagerStatePoweredOn,//蓝牙启
    } NS_DEPRECATED(NA, NA, 5_0, 10_0, "Use CBManagerState instead”);
该枚举在iOS10之后已经废除了，系统推荐使用`CBManagerState`，类型都是对应的

    typedef NS_ENUM(NSInteger, CBManagerState{
        CBManagerStateUnknown = 0,
        CBManagerStateResetting,
        CBManagerStateUnsupported,
        CBManagerStateUnauthorized,
        CBManagerStatePoweredOff,
        CBManagerStatePoweredOn,
    } NS_ENUM_AVAILABLE(NA, 10_0);

    - (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *, id> *)advertisementData RSSI:(NSNumber *)RSSI;
peripheral是外设类
`advertisementData`是广播的值，一般携带设备名，`serviceUUIDs`等信息
RSSI绝对值越大，表示信号越差，设备离的越远。如果想装换成百分比强度，（RSSI+100）/100，（这是一个约数，蓝牙信号值并不一定是-100 - 0的值，但近似可以如此表示）

    - (void)centralManager:(CBCentralManager *)central willRestoreState:(NSDictionary<NSString *, id> *)dict;

在蓝牙于后台被杀掉时，重连之后会首先调用此方法，可以获取蓝牙恢复时的各种状态

## 如何连接

在扫面的代理方法中，我们连接外设名是MI的蓝牙设备

    - (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI{    
        NSLog(@"advertisementData:%@，RSSI:%@",advertisementData,RSSI);      
        if([peripheral.name isEqualToString:@"MI"]){        
        [self.centralManager connectPeripheral:peripheral options:nil];//发起连接的命令       
              self.peripheral = peripheral;     
        }
    }
__连接的状态__
对应另外的`CBCentralManagerDelegate`代理方法
连接成功的回调

    - (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral;

连接失败的回调

    - (void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(nullable NSError *)error;

连接断开的回调

    - (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(nullable NSError *)error;

连接成功之后并没有结束，还记得`CBPeripheral`中的`CBService`和`CBService`中的`CBCharacteristic`吗，对数据的读写是由`CBCharacteristic`控制的。我们先用lightblue连接小米手环为例，来看一下，手环内部的数据是不是我们说的那样。

![lightblue](http://upload-images.jianshu.io/upload_images/1059465-1ab902ebe252094e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中`ADVERTISEMENT DATA`显示的就是广播信息。
>**iOS蓝牙无法直接获取设备蓝牙MAC地址，可以将MAC地址放到这里广播出来**

`FEEO`是`ServiceUUIDs`,里面的`FF01`、`FF02`是`CBCharacteristic的UUID`

`Properties`是特征的属性，可以看出`FF01`具有读的权限，`FF02`具有读写的权限。特征拥有的权限类别有如下几种：

    typedef NS_OPTIONS(NSUInteger, CBCharacteristicProperties{
        CBCharacteristicPropertyBroadcast = 0x01,
        CBCharacteristicPropertyRead = 0x02,
        CBCharacteristicPropertyWriteWithoutResponse = 0x04,
        CBCharacteristicPropertyWrite = 0x08,
        CBCharacteristicPropertyNotify = 0x10,
        CBCharacteristicPropertyIndicate = 0x20,
        CBCharacteristicPropertyAuthenticatedSignedWrites = 0x40,
        CBCharacteristicPropertyExtendedProperties = 0x80,
        CBCharacteristicPropertyNotifyEncryptionRequired NS_ENUM_AVAILABLE(NA, 6_0) = 0x100,
        CBCharacteristicPropertyIndicateEncryptionRequired NS_ENUM_AVAILABLE(NA, 6_0) = 0x200};

## 如何发送并接收数据

通过上面的步骤我们发现`CBCentralManagerDelegate`提供了蓝牙状态监测、扫描、连接的代理方法，但是`CBPeripheralDelegate`的代理方法却还没使用。别急，马上就要用到了，通过名称判断这个代理的作用，肯定是跟`Peripheral`有关，我们进入系统API，看它的代理方法都有什么，因为这里的代理方法较多，我就挑选几个常用的拿出来说明一下。

### 1、**代理方法**

    //发现服务的回调
    - (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(nullable NSError *)error;
    //发现特征的回调
    - (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(nullable NSError *)error;
    //读数据的回调
    - (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(nullable NSError *)error;
    //是否写入成功的回调
     - (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(nullable NSError *)error;

### 2、**步骤**
通过这几个方法我们构建一个流程：连接成功->获取指定的服务->获取指定的特征->订阅指定特征值->通过具有写权限的特征值写数据->在`didUpdateValueForCharacteristic`回调中读取蓝牙反馈值

解释一下订阅特征值：特征值具有Notify权限才可以进行订阅，订阅之后该特征值的value发生变化才会回调`didUpdateValueForCharacteristic`

### 3、**实现上面流程的实例代码**

    //连接成功
    - (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral{
            //连接成功之后寻找服务，传nil会寻找所有服务
            [peripheral discoverServices:nil];
    }

    //发现服务的回调
    - (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error{   
        if (!error) {        
            for (CBService *service in peripheral.services) {                
                NSLog(@"serviceUUID:%@", service.UUID.UUIDString);            
                if ([service.UUID.UUIDString isEqualToString:ST_SERVICE_UUID]) {
                            //发现特定服务的特征值               
                    [service.peripheral discoverCharacteristics:nil forService:service];            
                }        
            }    
        }
    }

    //发现characteristics，由发现服务调用（上一步），获取读和写的characteristics
    - (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {    
        for (CBCharacteristic *characteristic in service.characteristics) {        
        //有时读写的操作是由一个characteristic完成        
            if ([characteristic.UUID.UUIDString isEqualToString:ST_CHARACTERISTIC_UUID_READ]) {   
                self.read = characteristic;           
                [self.peripheral setNotifyValue:YES forCharacteristic:self.read];        
            } else if ([characteristic.UUID.UUIDString isEqualToString:ST_CHARACTERISTIC_UUID_WRITE]) {  
                 self.write = characteristic;        
            }    
        }
    }

    //是否写入成功的代理
    - (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{   
        if (error) {        
            NSLog(@"===写入错误：%@",error);    
        }else{        
            NSLog(@"===写入成功");    
        }
    }

    //数据接收
    - (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {    
            if([characteristic.UUID.UUIDString isEqualToString:ST_CHARACTERISTIC_UUID_READ]){
            //获取订阅特征回复的数据
            NSData *value = characteristic.value;        
            NSLog(@"蓝牙回复：%@",value);
            }
    }

比如我们要获取蓝牙电量，由硬件文档查询得知该指令是`**0x1B9901**`,那么获取电量的方法就可以写成

    - (void)getBattery{
        Byte value[3]={0};
        value[0]=x1B;
        value[1]=x99;
        value[2]=x01;
        NSData * data = [NSData dataWithBytes:&value length:sizeof(value)];
        //发送数据
        [self.peripheral writeValue:data forCharacteristic:self.write type:CBCharacteristicWriteWithoutResponse];
    }

如果写入成功，我们将会在`didUpdateValueForCharacteristic`方法中获取蓝牙回复的信息。

## 如何解析蓝牙数据

如果你顺利完成了上一步的操作，并且看到了蓝牙返回的数据，那么恭喜你，蓝牙的常用操作你已经了解大半了。因为蓝牙的任务大部分就是围绕发送指令，获取指令，将蓝牙数据呈现给用户。上一步我们已经获取了蓝牙指令，但是获取的却是`0x567b0629`这样的数据，这是什么意思呢。这时我们参考硬件文档，看到这样一段:

![device-document](http://upload-images.jianshu.io/upload_images/1059465-141b274b7ed8e976.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
那么我们就可以得出设备电量是 60%。

对数据解析的流程就是：判断校验和是否正确，是不是一条正确的数据->该条数据是不是我们需要的电量数据，即首字节为`0x567b`->根据定义规则解析电量，传给view显示。其中第一步校验数据，视情况而定，也有不需要的情况。

## 扩展

[iOS蓝牙中的进制转换](http://www.jianshu.com/p/a5e25206df39)
[蓝牙固件升级](http://www.jianshu.com/p/0d956862ffa1)
[nRF芯片设备DFU升级](http://www.jianshu.com/p/eb5b1e26adf7)
