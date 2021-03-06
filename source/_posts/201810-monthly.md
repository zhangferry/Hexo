---
title: iOS开发月报#4|201810
date: 2018-10-31 16:00:50
tags: 月报
---

记录本月开发遇到的知识点，小tips，和bug总结。
## 大事件
---
新版iPad Pro、MacBook Air、Mac mini发布，全线涨价，但是真香。。。
![image.png](https://upload-images.jianshu.io/upload_images/1059465-d64ed4ef11a3ee86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Tips
----
### 适配swift4.2
1、利用xcode快速迁移
升级到Xcode10之后，我们打开项目会出现如下提示，
![image.png](https://upload-images.jianshu.io/upload_images/1059465-ac05882ccd098391.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击会有一个版本升级窗口，如果你的的项目包含一些第三方库的话，第三方库的选型也会出现在上面：
![image.png](https://upload-images.jianshu.io/upload_images/1059465-8041b545d62a3095.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认勾选第三方库，但是我们适配的时候不应该让Xcode去自动检索第三方库代码。只对我们的app进行代码迁移就够了。
适配完后可以在这里查看我们当前的swift版本：
![image.png](https://upload-images.jianshu.io/upload_images/1059465-6e113852a3fb4b35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于第三方库，如果都适配了swift4.2，那么更新到对应版本就行了。如果有没适配的，可以通过制定版本的方式解决冲突，在Podfile文件末尾添加如下代码：
```ruby
post_install do |installer|
installer.pods_project.targets.each do |target|
target.build_configurations.each do |config|
config.build_settings['SWIFT_VERSION'] = '4.0'
end
end
end
```

### 属性访问权限
swift中提供的访问权限关键词由低到高有以下五种：
private < fileprivate < internal < public < open
其中internal是Swift中的默认控制级，一下介绍了这几种关键字的区别：
* private：当前作用域或者当前类中访问。
* fileprivate：表示代码可以在当前文件中访问。
* internal：在当前target中访问
* public：可跨target使用，但不能被集成或者重写。
* open：可跨target使用，允许被集成或者重写。

对于一个严格的项目来说，精确的最小化访问控制级别对于代码的维护来说是很重要的。能用public的就别用open。

### 修改文件权限
当我们执行某些命令行操作，收到如下提示时：
```
Linking /usr/local/Cellar/the_silver_searcher/2.1.0... 
Error: Could not symlink etc/bash_completion.d/ag.bashcomp.sh
/usr/local/etc/bash_completion.d is not writable.
```
就表明我们对需要修改的文件权限不够，我们可以给文件加上修改的权限：
```
sudo chown -R $(whoami) /usr/local/etc/bash_completion.d
```
其中`-R`表示对目前目录下的所有文件与子目录进行相同的拥有者变更(即以递回的方式逐个变更)

### 默认关键字
```
public static let `default` = ImageCache(name: "default")
```
default是默认关键字，如果使用要加单斜号。


### tabbar手动跳转
```
func tabBarController(UITabBarController, didSelect: UIViewController)
```
> In iOS v3.0 and later, the tab bar controller calls this method regardless of whether the selected view controller changed. In addition, it is called only in response to user taps in the tab bar and is not called when your code changes the tab bar contents programmatically.
> In versions of iOS prior to version 3.0, this method is called only when the selected view controller actually changes. In other words, it is not called when the same view controller is selected. In addition, the method was called for both programmatic and user-initiated changes to the selected view controller.

tabbar的didSelect代理方法，只有在手动点击的时候才会触发。通过代码跳转是不会触发的。
### 自定义tabbar动画
```swift
// 当点击tabBar的时候,自动执行该代理方法(不需要手动设置代理)  
override func tabBar(_ tabBar: UITabBar, didSelect item: UITabBarItem) {  
for view in tabBar.subviews {
//控制view实现各种动画效果
}
}
```

### NS_ASSUME_NONNULL_BEGIN 和 NS_ASSUME_NONNULL_END

从xcode6.3开始，为了能让OC也能表示swift的?(optional)和!功能，增加了对对象的可选指定。指定属性是否可选，可以：
```Objection-c
@property (nonatomic, copy, nonnull) NSString * tickets;
//或者
@property (nonatomic, copy) NSString * __nonnull tickets;
```

如果属性多了，每一个都这么写会很麻烦，苹果增加了一对新的宏命令，就是`NS_ASSUME_NONNULL_BEGIN`和`NS_ASSUME_NONNULL_END`。放在里面的对象就相当于都增加了`nonnull`命令，`nonull`为默认值。

### 一个自定义collectionView布局的bug
**bug描述：**
```xcode
NSInternalInconsistencyException

UICollectionView received layout attributes for a cell with an index path that does not exist: <NSIndexPath: 0x280d2b200> {length = 2, path = 1 - 5}
```
**原因：**
`layoutAttributesForElementsInRect`返回的`UICollectionViewLayoutAttributes`数组有`indexPath`没有被 `[NSIndexPath indexPathForRow:numberOfSection]`覆盖。
当数据量增加时不会出问题，当数量减少时出现。有人反映这是iOS10的bug，但实际上，我拿iOS10模拟器跑并没有问题，反而是在崩溃后台看到是iOS12的用户上报的。那究竟什么原因不详，附stackoverflow([iOS 10 bug: UICollectionView received layout attributes for a cell with an index path that does not exist - Stack Overflow](https://stackoverflow.com/questions/39867325/ios-10-bug-uicollectionview-received-layout-attributes-for-a-cell-with-an-index)]地址。
**解决方案：**
当我们自定义layout时，需要清除UICollectionViewLayoutAttributes的缓存
```swift
//方案一： 在自定义layout的类里
override func prepare() {
super.prepare()
attributesArr.removeAll()
}
//方案二：在collectionview刷新出
collectionView.reloaData()
collectionView.collectionViewLayout.invalidateLayout()
```

### 配置git SSH
1、设置git的user name和email
```
$ git config --global user.name "username"
$ git config --global user.email "username@gmail.com"
```
2、生成秘钥
```
$ ssh-keygen -t rsa -C "username@gmail.com"
```
如果不需要设置密码，连按三个回车。最后得到了两个文件：`id_rsa`和`id_rsa.pub`。
3、添加秘钥到ssh-agent中
```
$ ssh-add ~/.ssh/id_rsa
```
4、登录git仓库（github或者bitbucket），添加ssh
将`id_rsa.pub`文件的内容复制到对应的地方。

### Apple Watch开发注意事项
1、watch没有UIKit，对于UI的操作只能通过`storyboard`进行
2、watch只支持帧动画，我们只能通过png序列来实现动画效果。`WKInterfaceGroup` 和 `WKInterfaceImage`均可以实现帧动画。
3、开发的watch应用内存被限定为80M，太多帧的动画会不支持
4、提交应用watch也需要配置市场截图。
5、watch分为两个target。当新建一个Target为WatchDemo，xcode会自动生成一个WatchDemo Extension。前者负责UI，后者负责逻辑。引用cocoapods可以这么写：
```
target 'WatchDemo Extension' do
platform :watchos, '3.0'
use_frameworks!
pod 'Alamofire'
end
```
6、Always Embed Swift Standard Libraries
在Build Settings里面，这两个target，需要将WatchDemo Extension中设置为Yes，另一个设置为No
## Github
---
[Sizes](https://github.com/marcosgriselli/Sizes)
可以在一个界面，显示各个屏幕尺寸。这样我们就不用每个模拟器跑一遍看效果了。方便调试。

[iOS-DeviceSupport](https://github.com/iGhibli/iOS-DeviceSupport)
当手机升级，而xcode未升级时，我们会遇到Device Support的弹框，此时要么升级xcode，要么需要导入对应的Device Support文件。这个库就是提供这种文件的。

[InjectionIII](https://github.com/johnno1962/InjectionIII)
用于解决烦人的UI调试问题。当修改了一些UI属性之后，在xcode中我们只能运行程序才能看到效果，如果是处理大量的UI问题，这个过程是很烦人的。好在InjectionIII帮我们解决了这个问题，一起了解一下吧！


