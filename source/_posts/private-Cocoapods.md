---
title: 使用Cocoapods管理私有库组件
date: 2018-08-30 14:47:34
tags: cocoapods
---
![cocoapods.png](https://upload-images.jianshu.io/upload_images/1059465-ab8debcc283dcc57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

CocoaPods是OS X和iOS下的一个第三方开源类库管理工具，通过CocoaPods工具我们可以为项目添加依赖库（这些类库必须是CocoaPods本身所支持的），并且可以轻松管理其版本。它是目前iOS开发中使用最广泛的开源库管理工具，如果我们内部协作的组件化能够使用这种方式管理的话，那将是很便利的。
在通过Cocoapods建立内部私有库之前，我们需要再熟悉下Cocoapods的工作流程，我们创建内部私有库时也会依照这个流程来。

>本文目录
一、Cocoapods的工作流程
二、建立Cocoapods私有库
三、使用私有库
四、问题总结

## Cocoapods工作流程
工作流程如图所示：
![cocoapods_work_flows](https://upload-images.jianshu.io/upload_images/1059465-4799e6f105e76e18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 远程索引库：
这里存放了各个框架的描述文件，托管在github上：
[CocoaPods/Specs](https://github.com/CocoaPods/Specs)

### 本地索引库：
在安装cocoapods时，执行的`pod setup`就是讲远程索引克隆到本地，本地索引的目录为：
```
~/.cocoapods/repos/master
```
本地索引和远程索引的目录一致，结构如下：
![localRepo.png](https://upload-images.jianshu.io/upload_images/1059465-de2a30d6996c2107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个库的每个版本都对应一个json格式的描述文件：
```
{
"name": "YYImage",
"summary": "Image framework for iOS to display/encode/decode animated WebP, APNG, GIF, and more.",
"version": "1.0",
"license": {
"type": "MIT",
"file": "LICENSE"
},
"authors": {
"ibireme": "ibireme@gmail.com"
},
"social_media_url": "http://blog.ibireme.com",
"homepage": "https://github.com/ibireme/YYImage",
"platforms": {
"ios": "6.0"
},
"source": {
"git": "https://github.com/ibireme/YYImage.git",
"tag": "1.0"
},
"requires_arc": true,
"default_subspecs": "Core",
"subspecs": [
{
"name": "Core",
"source_files": "YYImage/*.{h,m}",
"public_header_files": "YYImage/*.{h}",
"libraries": "z",
"frameworks": [
"UIKit",
"CoreFoundation",
"QuartzCore",
"AssetsLibrary",
"ImageIO",
"Accelerate",
"MobileCoreServices"
]
},
{
"name": "WebP",
"dependencies": {
"YYImage/Core": [

]
},
"ios": {
"vendored_frameworks": "Vendor/WebP.framework"
}
}
]
}
```
### 本地索引文件
当执行pod search命令时，如果本地索引不存在，就会创建出来：
```
$ pod search afn
Creating search index for spec repo 'master'..
```
本地索引文件路径为：
```
~/Library/Cache/Cocoapods/Pods
```
### 远程框架库
以YYImage为例，它的远程框架库就是json文件中的source：
https://github.com/ibireme/YYImage.git

所以再用文字总结下Cocoapods工作流程大概就是
1、本地安装cocoapods，建立本地索引库和远程索引库的映射
2、本地项目pod install
3、查找本地索引文件，然后找到各个库对应版本的json文件
4、通过json文件source字段找到引用库的git地址
5、把库文件拉到本地项目

## 建立Cocoapods私有库（framework）
建议采用framework的形式创建私有库，这可以很好的在开发阶段检查出库的不兼容或者文件权限出现的问题，Swift编写的代码通过Cocoapods生成的都是framework。
### 0、准备工作：
**如何建立远程索引库**
首先我们需要建立一个内部的远程索引库，类似`Cocoapods/Spec`的功能，之后添加的库文件索引文件都会存放到这里：https://zhangferry@bitbucket.org/sealcn/sealrepo.git
建立本地和远程索引仓库的关联：
```
pod repo add SealRepo https://zhangferry@bitbucket.org/sealcn/sealrepo.git
```
执行`pod repo`
![91bd1bf2.png](https://upload-images.jianshu.io/upload_images/1059465-bc4880496ed40db6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到我们有了两个索引仓库，可以去在这个目录`~/.cocoapods/repos`看到我们刚建立的SealRepo。

**如何组织文件结构**
我们可以看下[Alamofire](https://github.com/Alamofire/Alamofire)的文件组织结构：

![image.png](https://upload-images.jianshu.io/upload_images/1059465-a2e182ab582670fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们看到这几个文件：
* `Source`用于存放Framework源文件，
* `Example`用于放Demo项目
* `docs`和`Documentation`放说明文档，这个是可选的，
* `Tests`测试文件也是可选。
我们制作私有库时会仿照这个格式。
### 一、制作framework
![3f2a44c9.png](https://upload-images.jianshu.io/upload_images/1059465-273e8e64bfbb617d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为是Swift的工程，接口的开放直接通过open、public等关键字指定，所以工程中的ABTest.h头文件可以删除，加入我们自己的库文件。
![5f8d2d4d.png](https://upload-images.jianshu.io/upload_images/1059465-312ee35e4b2f5ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：在写公有库文件时，对外界开放的属性，方法需要带上`public`或者`open`关键字。

### 二、添加Example工程
通过Xcode菜单栏`File->New->Target...`添加一个Example工程。
**引入第三方库**
如果无第三库引用可以跳过这一步。
注意：引入Podfile文件，需要framework和Example两个target都添加上。
**测试项目**
需要先编译framework，没有问题之后，导入到Demo项目里
```
import ABTest
```
运行Dome，测试开发功能有没有问题。

**push项目到远程库**
如果已经关联过远程私有仓库，这一步可以跳过。
在远程配置一个git地址，然后将本地项目关联到远程私有仓库：
```
git remote add origin 仓库地址
```
如过是首次关联远程仓库，在push之前我们一般需要先拉去远程分支
```
git pull origin master
```
如果提示：
```
There is no tracking information for the current branch.
```
那是因为本地库和远程库没有建立联系，git认为这两个仓库可能不是同一个，如果我们确认对应库没问题，可以使用：
```
$ git pull origin master --allow-unrelated-histories 
```
将远程库文件强制拉到本地仓库。
之后再执行push命令将项目推到远程仓库。
```
git push -u origin master
```

### 三、Cocoapods配置文件
**1、添加.swift-version**
.swift-version文件用来告诉cocoapods当前文件swift的版本，用命令行建立：
```
$ echo "3.0" > .swift-version
```
**2、添加LICENSE**
每个使用cocoapods添加的库都需要准守开源协议，一般是`MIT`协议，因为bitbucket没法自动生成，我们可以手动生成这个同名文件，然后把协议内容复制进去：
```
MIT License

Copyright (c) [year] [fullname]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
**3、创建库描述文件**
可以通过命令行生成描述文件：
```
$ pod spec create ABTest
```
然后我们编辑ABTest.podspec文件，可以仿照下面的写法
```
Pod::Spec.new do |s|

s.name         = "ABTest"
s.version      = "0.0.1"
s.summary      = "ABTest with Firebase"
s.description  = "This is a ABTest Framworks on swift"
s.homepage     = "https://bitbucket.org/sealcn/remoteabtest/src/master/"
s.license      = { :type => "MIT", :file => "LICENSE" }
s.author             = { "zhangferry" => "zhangfei@dailyinnovation.biz" }

# ――― Platform Specifics ――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
s.platform     = :ios, "8.0"
# ――― Source Location ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
s.source       = { :git => "https://zhangferry@bitbucket.org/sealcn/remoteabtest.git", :tag => s.version }
s.source_files  = "Source", "Source/*.swift"

# ――― Resources ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
# s.resource  = "icon.png"
# s.resources = "Resources/*.png"

# ――― Project Settings ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
s.requires_arc = true
s.static_framework = true
s.dependency "Firebase/Core"
s.dependency "Firebase/RemoteConfig"
#s.ios.vendored_frameworks = "ABTest.framework"
s.xcconfig = { 'SWIFT_INCLUDE_PATHS' => '$(PODS_ROOT)/Firebase/CoreOnly/Sources' }
end
```
此时我们的文件目录看起来应该大概是这个样子：
![framework_catalog.png](https://upload-images.jianshu.io/upload_images/1059465-22fe03c83eebed9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**4、验证本地`podspec`文件**
```
pod lib lint
```
该命令用于检查podspec文件书写是否正确，如果有error需要解决，warning可以不用管（可能会遇到较多问题，需耐心解决0。0）。解决之后再次运行检查命令，当命令行显示：
```
-> ABTest (0.0.1)
ABTest passed validation.
```
说明我们本地配置成功了，到这里本地的第一个版本就算完成了！
然后我们需要将本次修改提交打上tag，提交到远程仓库。
```
git add .
git commit -m "build v0.0.1"
git push origin master
git tag 0.0.1
git push --tags
```
**5、验证远程索引文件**
上传代码成功之后，我们需要再次验证它跟远程仓库（ABTest远程库和.podspec）是否匹配正确，执行：
```
pod spec lint
```
当出现：
```
ABTest.podspec passed validation
```
时，说明我们远程仓库匹配正确。

**6、提交podspec文件到SpecsRepo**
```
$ pod repo push SealRepo ABTest.podspec
```
这个命令会包含`pod spec lint`命令，验证通过之后，会添加.podspec文件到本地索引库：

![7dffd809.png](https://upload-images.jianshu.io/upload_images/1059465-8df2158ae8033cfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

和远程索引库：
![7e052503.png](https://upload-images.jianshu.io/upload_images/1059465-749a69f468b86656.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 使用私有库
### 引用私有库
我们可以像使用其他库文件一样在Podfile文件中添加使用私有库了，引入方法有两种：

**1、全局添加**
在Podfile文件最上面添加一行：
```
source 'https://zhangferry@bitbucket.org/sealcn/sealrepo.git'
```
注意：如果私有仓库和cocoapods仓库出现同名库，会出现不可预期的情况（随机拉下来公有库或者私有库文件）。这时我们需要使用单独添加的方式。

**2、单独添加**
```
pod 'ABTest', :git => 'https://zhangferry@bitbucket.org/sealcn/remoteabtest.git'
```
使用时通过import方法导入库就可以了。
### 更新私有库
当我们需要升级私有库，添加或者修改方法时，只需要：

1、修改`.podspec`文件中`s.version`的版本号

2、提交本地修改至远程，打上对应tag

3、使用项目的工程执行`pod update`

## 可能遇到的问题
1、pod search 查不到本地库
这个可能是cocoadpods本身问题，pod install安装没有问题

2、更新了版本，但是pod update没有找到
我们可以采用如下形式，手动指定版本号：
```
pod 'ABTest', :git => 'https://zhangferry@bitbucket.org/sealcn/remoteabtest.git', :tag => '0.0.4'
```

3、提示`The 'Pods-App' target has transitive dependencies that include static binaries`
这是因为引入的库被编译成了静态库，我们可以在`.podspec`文件中加入：
```
s.static_framework = true
```

4、引入的第三方库，在`pod lint`时提示找不到
可以手动指定pod目录，将firsebase替换成你的库文件路径:
```
s.xcconfig = { 'SWIFT_INCLUDE_PATHS' => '$(PODS_ROOT)/Firebase/CoreOnly/Sources' }
```

5、提示source_files对应文件为空
每次`pod lint`时都是根据版本号进行查找的，可以检查下当前修改跟版本号是否对应。
