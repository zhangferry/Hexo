---
title: iOS获取来电和短信发送状态
date: 2016-12-12 15:35:51
tags: 短信来电
categories: iOS知识点
---

## 获取电话状态
在我想要了解iOS获取来电状态时，经常被这是不是允许的，是不是要调用私有库等问题困扰。费了好大劲终于解决了上面问题，你可以获取系统提供的电话相关状态，而且它不属于私有库。为了需要这方面资料的人查阅时少走弯路，我把这些东西写下来，废话少说，上代码。
<!--more-->
### 如何获取电话状态
首先要导入CoreTelephony框架：
`@import CoreTelephony;`

然后声明一个CTCallCenter变量：

    @interface ViewController () {  
    CTCallCenter *center_;   //为了避免形成retain cycle而声明的一个变量，指向接收通话中心对象
    }  
@end
然后监听电话状态：

    - (void) aboutCall{   
    //获取电话接入信息
    callCenter.callEventHandler = ^(CTCall *call){
    if ([call.callState isEqualToString:CTCallStateDisconnected]){
    NSLog(@"Call has been disconnected");

    }else if ([call.callState isEqualToString:CTCallStateConnected]){
    NSLog(@"Call has just been connected");

    }else if([call.callState isEqualToString:CTCallStateIncoming]){
    NSLog(@"Call is incoming");

    }else if ([call.callState isEqualToString:CTCallStateDialing]){
    NSLog(@"call is dialing");

    }else{
    NSLog(@"Nothing is done");
    }
    };
    }
还可以获取运营商信息：

    - (void)getCarrierInfo{
    // 获取运营商信息
    CTTelephonyNetworkInfo *info = [[CTTelephonyNetworkInfo alloc] init];
    CTCarrier *carrier = info.subscriberCellularProvider;
    NSLog(@"carrier:%@", [carrier description]);

    // 如果运营商变化将更新运营商输出
    info.subscriberCellularProviderDidUpdateNotifier = ^(CTCarrier *carrier) {
    NSLog(@"carrier:%@", [carrier description]);
    };

    // 输出手机的数据业务信息
    NSLog(@"Radio Access Technology:%@", info.currentRadioAccessTechnology);
    }    
当然这样在真机进行测试，以下为输出信息：

    2015-12-29 16:34:14.525 RWBLEManagerDemo[1489:543655] carrier:CTCarrier (0x134e065c0) {
    Carrier name: [中国移动]
    Mobile Country Code: [460]
    Mobile Network Code:[07]
    ISO Country Code:[cn]
    Allows VOIP? [YES]
    }
    2015-12-29 16:34:14.526 RWBLEManagerDemo[1489:543655] Radio Access Technology:CTRadioAccessTechnologyHSDPA    

### CoreTelephony框架是不是私有库
私有框架的目录为：
`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/System/Library/PrivateFrameworks/`

![FDC2B801-0F1C-41FD-A9A4-399592DF4BEF.png](http://upload-images.jianshu.io/upload_images/1059465-3d7bb9f8ff9ce3d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出CoreTelephony框架是在frameworks内而不是PrivateFrameworks，所以它是可以放心使用的。网上之所以有说CoreTelephony是私有库，是因为在iOS6的时候是私有框架，后来苹果又给公开了。
## 获取短信状态
关于短信的状态获取，我直接看了
`#import <MessageUI/MessageUI.h>`
里面就两个头文件：

`#import <MessageUI/MFMailComposeViewController.h>`
`#import <MessageUI/MFMessageComposeViewController.h>`
一个是邮件相关的方法，一个短信相关的方法。进到MFMessageComposeViewController.h有一个枚举值：

    enum MessageComposeResult {
    MessageComposeResultCancelled,
    MessageComposeResultSent,
    MessageComposeResultFailed
    };
typedef enum MessageComposeResult MessageComposeResult;   // available in iPhone 4.0
这是表示短信发送状态的值。要使用这个框架发送自己编辑的内容还需要添加代理：`MFMessageComposeViewControllerDelegate`

代码如下：

    - (void)showMessageView
    {
    if( [MFMessageComposeViewController canSendText] )// 判断设备能不能发送短信
        {
        MFMessageComposeViewController*picker = [[MFMessageComposeViewControlleralloc] init];
        // 设置委托
        picker.messageComposeDelegate= self;
        // 默认信息内容
        picker.body = @"nihao";
        // 默认收件人(可多个)
        picker.recipients = [NSArray arrayWithObject:@"12345678901", nil];
        [self presentModalViewController:picker animated:YES];
        [picker release];
        }
        else
        {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示信息"
        message:@"该设备不支持短信功能"
        delegate:self
        cancelButtonTitle:nil
        otherButtonTitles:@"确定", nil];
        [alert show];
        [alert release];
        }
    }

    - (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result
    {
        switch (result){
        case MessageComposeResultCancelled:
        NSLog(@"取消发送");
        break;
        case MessageComposeResultFailed:
        NSLog(@"发送失败");
        break;
        case MessageComposeResultSent:
        NSLog(@"发送成功");
        break;

        default:
        break;
        }
    }
对于来短信的通知没有找到，应该是不能获取的。

### 参考资料
* private framework使用
<http://chenjohney.blog.51cto.com/4132124/1288551> 
* CoreTelephony框架的简单使用
<http://blog.csdn.net/jymn_chen/article/details/19240903>  
* iOS关于系统短信和电话的调用
<http://blog.csdn.net/frank_jb/article/details/49815883>
