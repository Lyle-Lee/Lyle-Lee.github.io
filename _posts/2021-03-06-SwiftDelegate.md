---
layout:     post
title:      Swift Delegate
subtitle:   Swift中如何使用代理模式
date:       2021-04-06
author:     Lyle
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - iOS
    - Swift
    - 设计模式
---

> Xcode 12.4 | Swift 5.0

在iOS开发中，无论是 **Objective-C** 还是 **Swift** ，**Delegate** 有着具足轻重的位置，如`TabelViewDelegate` 与 `TableViewDataSource`。

## Swift 代理模式

Swift 代理模式 与 Objective-C 一样，只是语法不同。

#### 委托方（子控制器）

- 创建协议 、声明协议方法

		protocol SubViewDelegate {
	    func backStr(str: String)
		}	
- 创建一个代理属性

		var delegate: SubViewDelegate?
- 执行协议方法

		/// 执行代理方法，将值回传
        delegate?.backStr(str: textField.text ?? "")
        
#### 代理方(主控制器)
- 继承协议

		class ViewController: UIViewController, SubViewDelegate
- 将代理设为自己

		subVC.delegate = self
		
- 实现代理方法

	```swift
	func backStr(str: String) {
        self.textF.text = str
    }
    ```
    
    
## 继承

Swift 的扩展 `extension`可以用来继承协议，实现代码隔离，便于维护。

```swift
/// 使用扩展继承协议 实现协议方法 可以分离代码
extension ViewController: SubViewDelegate{
    /// 实现代理方法
    func backStr(str: String) {
        self.textF.text = str
    }
}
```


## Delegate应用开源项目

最后附上一个应用实例[Real Time Weather](https://github.com/Lyle-Lee/RealTimeWeather)

如果对你有帮助的话，**Star**✨下一吧！
