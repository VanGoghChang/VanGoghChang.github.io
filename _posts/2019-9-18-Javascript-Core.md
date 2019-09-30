---
layout: post
title: JavascriptCore 浅析
---

React Native 的核心驱动力就来自于 JS Engine. 你写的所有代码都会被 JS Engine 来执行。
而 JavaScriptCore 是 iOS(iOS7+)/Android 平台上默认的 JS Engine, 来源于 Webkit，通俗来说 JavascriptCore 就是在原生代码中运行的 JS 解释器。
下面就来看看 JavaScriptCore 是怎样工作的

## Native 调用处理 JavaScript

### JSContext / JSValue

JSContext 是运行 Javascript 代码的环境，一个 JSContext 是一个全局环境的实例，创建一个 JSContext 实例后，
调用 .evaluateScript，传入写好的 JS 代码就可以运行了。

#### Swift
![](/images/19_09_18/core_0.png)

#### Objective-C
![](/images/19_09_18/core_1.png)



任何出自 JSContext 的值都被包裹在一个 JSValue 对象中，所以 JSValue 包装了每一个可能的 JavaScript 值：字符串和数字；数组、对象和方法；甚至错误和特殊的 JavaScript 值诸如 null 和 undefined。

#### Swift
![](/images/19_09_18/core_2.png)

#### Objective-C
![](/images/19_09_18/core_3.png)




JSValue 包括一系列方法用于访问其可能的值以保证有正确的 Foundation 类型，包括：

|  JavaScript Type   | JSValue method  |  Objective-C Type   | Swift Type  |
|  ----  | ----  |  ----  | ----  |
| string   | toString | NSString | String! |
| boolean  | toBool | BOOL | Bool |
| number   | toNumber/toDouble/toInt32/toUInt32 | NSNumber/double/int32_t/uint32_t | NSNumber!/Double/Int32/UInt32 |
| Date     | toDate | NSDate | NSDate! |
| Array    | toArray | NSArray | [AnyObject]! |
| Object   | toDictionary | NSDictionary | [NSObject : AnyObject]! |
| Object   | toObject/toObjectOfClass: | custom type | custom type |




获取的 JSValue 的值调取相应的方法转换成目标类型的值，如上面的例子 num 应：

#### Swift
![](/images/19_09_18/core_4.png)

#### Objective-C
![](/images/19_09_18/core_5.png)



### 获取变量

初始化实例，成功创建变量后，我们可以使用下标值的访问我们之前创建的变量。

#### Swift
![](/images/19_09_18/core_6.png)

#### Objective-C
![](/images/19_09_18/core_7.png)




### 调用方法

若 JSValue 包装了一个 JavaScript 函数，在 iOS 中，我们可以从 Objective-C / Swift 代码中使用 Foundation 类型作为参数来直接调用该函数。

#### Swift
![](/images/19_09_18/core_8.png)

#### Objective-C
![](/images/19_09_18/core_9.png)




### 错误处理

JSContext 可以通过设置上下文的 exceptionHandler 属性，你可以观察和记录语法，类型以及运行时错误。 exceptionHandler 是一个接收一个 JSContext 引用和异常本身的回调处理。

#### Swift
![](/images/19_09_18/core_10.png)

#### Objective-C
![](/images/19_09_18/core_11.png)



## JavaScript 调用处理 Native

上述是原生层内怎样访问 Javascript 内定义的方法和对象，那么反向，Javascript 怎样访问原生文件中定义的对象和方法？
让 JSContext 访问我们的本地客户端代码的方式主要是两种：block 和 JSExport 协议。

### Blocks
当一个 Objective-c block 被赋给 JSContext 里的一个标识符，JavascriptCore 会自动的把 block 封装在 Javascript 函数里。
这使得在 Javascript 中可以简单的使用 Foundation 和 Cocoa 类，所有的桥接都已经封装好了。

在这儿，Swift 还有一个坑，请注意，这仅适用于 Objective-c 的 block，而不是 Swift 的闭包。要在 JSContext 中使用 Swift 闭包，
它需要（a）与 @ objc_block 属性一起声明，以及（b）使用 Swift 那个令人恐惧的 unsafeBitCast() 函数转换为 AnyObject。

#### Swift
![](/images/19_09_18/core_12.png)

#### Objective-C
![](/images/19_09_18/core_13.png)

### 内存管理
由于 block 可以保有变量引用，而且 JSContext 也强引用它所有的变量，为了避免强引用循环需要特别小心。避免保有你的 JSContext 或一个 block 里的任何 JSValue。相反，使用 [JSContext currentContext] 得到当前上下文，并把你需要的任何值用参数传递。

### JSExport 协议
另一种在 JavaScript 代码中访问原生文件中定义的对象的方法是添加 JSExport 协议。无论我们在 JSExport 里声明的属性，实例还是类方法，继承的协议都会自动提供给任何 Javascript 代码。