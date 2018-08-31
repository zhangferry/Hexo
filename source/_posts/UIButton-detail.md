---
title: 可能被忽略的UIButton细节
date: 2018-07-17 10:55:35
tags: iOS
---
## 关于System Button
---------------
看一个简单的例子：
```swift
button.setTitle("Title", for: .normal)
button.setImage(UIImage(named: "icon"), for: .normal)
```

buttonType分别设置为system和custom，仅做如上设置，显示效果对比（上面的custom，下面的是system）
<div align=center>
![button_normal](https://ws4.sinaimg.cn/large/006tKfTcgy1ftcrof65cij305x00xa9u.jpg)
![button_system](https://ws1.sinaimg.cn/large/006tKfTcgy1ftcrokww28j305x010q2r.jpg)
</div>
system Button显示出蓝色其实是tintColor的效果，关于tintColor的说法是：
<!--more-->
This property has no default effect for buttons with type custom. For custom buttons, you must implement any behavior related to tintColor yourself.

在custom类型的button中设置tintColor是不生效的，需要自定义样式。在system类型的button里有一个默认蓝色的tintColor，当然我们可以修改它为其他颜色，会对image和title同时生效。

另外可以发现image不是原始图片，而是被填充为tintColor的颜色。这是因为system类型下button的image被默认以`alwaysTemplate`类型渲染的，如果想要显示原始图片可以做如下操作：

```swift
let image = UIImage(named: "icon")?.withRenderingMode(.alwaysOriginal)
button.setImage(image, for: .normal)
```

## 关于触摸反馈
------

看一个常见的代码：

```swift
let button = UIButton()//默认样式custom
button.setTitle("Title", for: .normal)
button.setTitleColor(UIColor.blue, for: .normal)
button.backgroundColor = UIColor.red
view.addSubview(button)
```

以上是的button的常见写法。遗憾的是这种写法，不会带触摸反馈效果。那如果我们想加触摸反馈，需要如何处理：

### 1、仅文字的触摸反馈

```swift
//system:
let button = UIButton(type: .system)
button.setTitleColor(UIColor.blue, for: .normal)//自动添加反馈效果
button.setTitleColor(UIColor.green, for: .highlighted)//会和系统效果叠加，不可控，不建议写
//custom
let button = UIButton(type: .custom)
button.setTitleColor(UIColor.blue, for: .normal)
button.setTitleColor(UIColor.green, for: .highlighted)//自定义反馈样式
```

### 2、带图片和文字的触摸反馈

```swift
button.setTitle("Title", for: .normal)
button.setImage(UIImage(named: "icon"), for: .normal)
//system:会同时对图片文字添加反馈效果
//custom:默认仅对图片有触摸反馈
```

### 3、带背景图的按钮

```swift
button.setBackgroundImage(UIImage(named: "background"), for: .normal)
//system是按钮整体反馈
//custom是仅背景图片反馈，title，image无反馈
```

### 4、关闭触摸反馈

```swift
button.isUserInteractionEnabled = false
//custome，system均会关闭触摸反馈
button.adjustsImageWhenHighlighted = false
//custom:会关闭image，backgroundImage的反馈
//system:此设置无效
```

### 5、showsTouchWhenHighlighted

这个属性是系统提供的一种highlighted样式，点击时按钮高亮。但是效果确实有点丑丑的，基本不用这种效果

## 其他特性

### 全局修改UIButton的样式可以：

```swift
let gobalBtn = UIButton.appearance()//所有继承UIView的类都可以使用这个方法
gobalBtn.setTitle("Good", for: .normal)//会将所有button的title改为Good
```

### setAttributedTitle方法

这个方法可以将button的title以富文本的形式进行设置，支持对不同state的设置。需要注意它和setTitle的优先级

```swift
//用attributed方式设置button的title和titleColor
let string = "Title"
let attributed = NSMutableAttributedString(string: string)
let range = NSMakeRange(0, string.count)
attributed.addAttributes([NSAttributedStringKey.foregroundColor : UIColor.green], range: range)
button.setAttributedTitle(attributed, for: .normal)
//此时用setTitle重新设置title样式，不会生效，attributed优先级大于直接设置
button.setTitle("Next Button", for: .normal)
button.setTitleColor(UIColor.blue, for: .normal)
```

