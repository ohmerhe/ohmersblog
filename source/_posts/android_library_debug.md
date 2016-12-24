title: Library 不支持调试模式，不能忍
date: 2016-08-30 17:28:49
tags: 
- Gradle
- Android
  
thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170096155.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170096155.jpg?imageView2/1/w/1024/h/460 

---


tags: Android, Gradle

在 Android 开发过程中，`BuildConfig.Debug` 这个变量用来判断当前运行环境是不是支持调试模式。我们常常利用这个变量的判断在开发或者测试包中做一些代码追踪、测试工具开启、调试信息等工作。不过在 Android 依赖库中默认编译出来的包并不会像编译应用一样默认会自动生成 release 和 debug 两种包，它只会默认生成 release 一个版本的包，可以参考[这里](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Referencing-a-Library)。在 release 版本的包里面，除非你有做过改动，不然默认 debuggable 这个值是 false。

常见的依赖库的使用方式有两种，一种是把依赖库的作为一个模块和主项目一起编译，也就是文件依赖；另一种是使用 aar 的方式引用，下面分别针对两种不同的提供对应得解决方案。

## 文件依赖方式

其实 [这个问题](https://code.google.com/p/android/issues/detail?id=52962) 早在 2013 年就有人在 Google Group 上提出来。根据 [官方文档](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Library-Publication)，我们可以通过控制 `publishNonDefault` 这个变量的配置来使得依赖库在编译的时候默认生成所有变种的包，而不是仅仅生成 release 一种。

在依赖项目中添加这个配置：

```groovy
android {
    publishNonDefault true
    ...
}
```

这样在编译完成后，默认情况下我们可以在输出目录看到两个 aar 文件（之前只有一个）。然后在项目中声明依赖的时候，区分不同的编译类型进行依赖：

```groovy
dependencies {
    debugCompile project(path: ':myLocalLibrary', configuration: 'debug')
    releaseCompile project(path: ':myLocalLibrary', configuration: 'release')
}
```

这样在调试应用的过程中会使用依赖库的 debug 版本，而在正式发布应用的时候就会用 release 版本。

## AAR 依赖方式

Library 还有一种更为常见的依赖方式——aar 依赖。当然如果是正式发布的依赖库，不支持 debug 功能是很合理并且应该鼓励的。但是，不能忽视的是在我们开发过程频繁使用的 snapshot 版本，这是开发过程中的测试版本，在这个版本中支持调试功能即合理也很有必要。

在前面官方文档中，我们发现还有一个配置信息可以利用 `defaultPublishConfig`。这个变量用于指定使用哪个变种的包作为默认编译的版本。默认这个值是 `release`。你可以在项目的配置添加：

```
android {
    defaultPublishConfig "debug"
    ...
}
```

> 这个配置项后面的值是编译变种的全称，如果对于变种（variants）的概念不是很熟悉的话，可以回去再看看 Google 的 [定义](https://developer.android.com/studio/build/build-variants.html)。

> 如果没有定义针对变种做过配置的话，默认支持 `release` 和 `debug` 两种，这是根据这两种默认编译类型自动生成的。

这里有一个明显的 bug，必须在发布正式包和 snapshot 包的时候手动切换配置项的值。在我的项目中，我在发布 aar 包的时候，是通过在 `gradle.properties` 这个文件添加 `isRelease` 这个变量来区分的。当这个值是 `true` 的时候则会发布正式包，反之则发布 snapshot 版本。于是我也利用这个值来控制这个配置项的配置：

```
android {
	defaultPublishConfig System.properties['isRelease'].toBoolean() ? "release" : "debug"
    ...
}
```

> 在 Groovy 中看到熟悉的三目运算好亲切啊，想到在 Kotlin 中没有三目运算就心塞。

这样在发布依赖包时，就能自动实现在 snapshot 包支持调试功能，而不影响正式包。

## 其他方案

### 代码注入

如果不使用这种方式，在代码层面想办法绕过这个限制也不是特别困难。比如我们可以在依赖库中提供接口，然后在项目中将是否支持 Debug 状态的判断注入到依赖库中，从而实现依赖库和主项目之间的 Debug 状态保持一致。

虽然这种方案也可以解决问题，但是我个人不是很推荐。这种配置方式本身和依赖库的功能没有关联性，而且无形增加了依赖库的接入成本。

### 手动修改

还有一种方案，是在依赖库编译完成之后，通过判断当前编译变种的类型，手动去修改 `BuildConfig` 里面的值。这种方案无形增加了解决问题的复杂度，和前面利用官方配置项没有本质区别。根据这种方案的提出的时间，我猜测这应该是在官方支持如上描述的配置方案之前的方案。

## 参考资料

- [Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)
- [Building your own Android library](https://guides.codepath.com/android/Building-your-own-Android-library)
- [Issue 52962:	Gradle plugin does not propagate debug/release to dependencies](https://code.google.com/p/android/issues/detail?id=52962)
