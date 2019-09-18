---
layout: post
title: JavascriptCore 浅析
---

React Native 的核心驱动力就来自于 JS Engine. 你写的所有代码都会被 JS Engine 来执行。
而 JavaScriptCore 是 iOS(iOS7+)/Android 平台上默认的 JS Engine, 来源于 Webkit，通俗来说 JavascriptCore 就是在原生代码中运行的 JS 解释器。
下面就来看看 JavaScriptCore 是怎样工作的

JSContext / JSValue
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