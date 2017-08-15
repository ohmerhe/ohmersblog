title: Kotlin 迷人的语言特性（下）
date: 2017-08-15 22:46:11
tags: 
- Kotlin
  
thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170116497.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170116497.jpg?imageView2/1/w/1024/h/460 

---


在上一篇文章，我们介绍了 Kotlin 许多迷人的语言特性，包括空安全、类型推断、操作符重载等等，接下来我们继续领略 Kotlin 给我们带来的迷人特性。

## 委托属性

Kotlin 没有字段（field）的概念，只有属性，Kotlin 为所有的属性自动生成 Setter 和 Getter 方法（常量只有 Getter）方法，对 Kotlin 属性的设置和访问，也都是通过 Setter 和 Getter 方法。利用这个特性，Kotlin 为开发者提供了委托属性，可以将对属性的操作和访问委托个另外一个对象。懒加载和属性观察者，用起来 666。

![](http://7xpox6.com1.z0.glb.clouddn.com/2017-08-06-22-14-42.png)


## Lambdas 表达式和高阶函数

Lambdas 表达式是 Kotlin 带给 Android 程序员的又一个期待已久的礼包，在 Java 8 只是 Lambdas 表达式的时候，大家就各种期待能用 Java 8 开发 Android，然而过了这么久，Jack 依然存在各种问题，而且现在还被官方放弃了。不过现在有了 Kotlin，Lambdas 表达式和高阶函数都不在是遥不可及。

![](http://7xpox6.com1.z0.glb.clouddn.com/2017-08-06-22-20-28.png)

## 类型别名

当碰到一个嵌套多级的泛型类型的是，要定义这个类型的变量是通过的，因为我们需要申明这样 `MutableMap<K, MutableList<File>>` 一个类型，除了书写起来非常费力外，可读性也非常差，有时可能还要比对尖括号的位置来阅读代码。为了解决这样的问题（同样也适用于 Kotlin 的函数类型），Kotlin 提供了类型别名的方式，让你可以为一个复杂的类型定义一个简单类型别名。

![](http://7xpox6.com1.z0.glb.clouddn.com/2017-08-06-22-26-41.png)

## 解构声明

还记得第一次接触到 Go 的时候，我的同事就对 Go 的同时返回多参数赞不绝口，都后来才发现，只要支持元祖的语言，都可以做到这一点。Kotlin 的解构声明在早期其实也是叫元祖，但是可能由于对元祖功能支持的并不完整，所以后面放弃了这个叫法。针对特定的类型（如：数据类）我们可以快速结构一个对象的得到多个变量，常用在多参数返回和集合迭代中。

![](http://7xpox6.com1.z0.glb.clouddn.com/2017-08-06-22-36-04.png)

## Kotlin 协程

Kotlin 协程是 1.1 后引入的一个大的语言特性，现在还是体验阶段，虽然还是体验阶段，但是这个特性实在是太赞了，以至于已经吸引了大家太多的眼球了。Kotlin 协程旨在帮助程序员更方便的处理异步程序，Kotlin 官方提供协程实现库，囊括了现在流行语言中使用协程的一些经典用法，如# 和 ECMAScript 的 async/await 及源于 C# 和 Python generators/yield。

![](http://7xpox6.com1.z0.glb.clouddn.com/8B94B216-105A-47C0-A801-80B8A1237DF3.png)

## 内联函数

当我们建一个函数或表达式标记成内联函数的时候，Kotlin 编译器会将对应的代码生成在该内联函数调用的地方，这样一方面解决了高阶函数带来的一些运行时的效率损失，同样也为具体化的类型参数（泛型）的实现提供了实现基础。

![](http://7xpox6.com1.z0.glb.clouddn.com/2017-08-06-23-00-17.png)

## 总结

Kotlin 有太多让人赞不绝口的迷人特性，其中有很多特性本身就值得我们花上好几个篇章来介绍，这里只是简单做了一些汇总，具体各种语言特性的体验，还需要各位看官亲自体验。

距离 Google I/O 大会已经过去一段时间了，Kotlin 的风暴似乎已经过去，你是不是已经找到自己的理由继续回去写自己的 Java 了。

---
![](http://7xpox6.com1.z0.glb.clouddn.com/kotlin-three-wechat.jpg)
