---
layout: post
title: JavascriptCore 浅析
---

React Native 的核心驱动力就来自于 JS Engine. 你写的所有代码都会被 JS Engine 来执行。
而 JavaScriptCore 是 iOS(iOS7+)/Android 平台上默认的 JS Engine, 来源于 Webkit，通俗来说 JavascriptCore 就是在原生代码中运行的 JS 解释器。
下面就来看看 JavaScriptCore 是怎样工作的

#### JSContext / JSValue

JSContext 是运行 Javascript 代码的环境，一个 JSContext 是一个全局环境的实例，创建一个 JSContext 实例后，
调用 .evaluateScript，传入写好的 JS 代码就可以运行了。

Swift
```Swift
let instance = JSContext()
instance.evaluateScript("var a = 10")
instance.evaluateScript("var names = ["Jack", "Mark", "Lily"]")
instance.evaluateScript("var nums = function(a, b){return a+b}")
```

Objective-C
```Objective-C
JSContext *instance = [[JSContext, alloc] init];
[instance evaluateScript:@"var a = 10"]
[instance evaluateScript:@"var names = ["Jack", "Mark", "Lily"]"]
[instance evaluateScript:@"var nums = function(a){return a}"]
```

任何出自 JSContext 的值都被包裹在一个 JSValue 对象中，所以 JSValue 包装了每一个可能的 JavaScript 值：字符串和数字；数组、对象和方法；甚至错误和特殊的 JavaScript 值诸如 null 和 undefined。

Swift
```Swift
let num: JSValue = instance.evaluateScript(nums(a))
```

Objective-C
```Objective-C
JSValue *num = [instance evaluateScript:@"nums(a)"];
```

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

Swift
```Swift
println("Num: \(num.toInt32())")
```

Objective-C
```Objective-C
NSLog(@"Num: %d", [num toInt32]);
```

#### 获取变量

初始化实例，成功创建变量后，我们可以使用下标值的访问我们之前创建的变量。

Swift
```Swift
let names = instance.objectForKeyedSubscript("names")
let name = names.objectAtIndexedSubscript(0)
let nameValue = name.toString()
// Jack
```

Objective-C
```Objective-C
JSValue *names = instance[@"names"];
JSValue *name = name[2];
NSString *nameValue = [name toString]
// Lily
```

#### 调用方法

若 JSValue 包装了一个 JavaScript 函数，在 iOS 中，我们可以从 Objective-C / Swift 代码中使用 Foundation 类型作为参数来直接调用该函数。

Swift
```Swift
let numsFunction = instance.objectForKeyedSubscript("nums")
let num = numsFunction.callWithArguments([15])
let numValue = num.toInt32()
// 15
```

Objective-C
```Objective-C
JSValue *numsFunction = instance[@"nums"];
JSValue *num = [numsFunction callWithArguments:@[5]]
NSNumber *numValue = [num toInt32]
// 5
```

#### 错误处理

JSContext 可以通过设置上下文的 exceptionHandler 属性，你可以观察和记录语法，类型以及运行时错误。 exceptionHandler 是一个接收一个 JSContext 引用和异常本身的回调处理。

Swift
```Swift
instance.exceptionHandler = { context, exception in
    println("JS Error: \(exception)")
}

instance.evaluateScript("function nums(a) { return a ")
// JS Error: SyntaxError: Unexpected end of script
```

Objective-C
```Objective-C
instance.exceptionHandler = ^(JSContext *context, JSValue *exception) {
   NSLog(@"JS Error: %@", exception);
};

[instance evaluateScript:@"function nums(a) { return a "];
// JS Error: SyntaxError: Unexpected end of script
// 5
```