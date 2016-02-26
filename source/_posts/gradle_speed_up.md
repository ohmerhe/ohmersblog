title: 优化gradle编译速度实践
date: 2016-02-26 16:25:44
tags: 

- gradle
- android
- Android Studio
- 编译优化
- 安卓

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/plan_landed.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/plan_landed.jpg?imageView2/1/w/1024/h/460 

---


随着项目规模越来越大，编译速度越来越慢，每次修改代码以后的编译都是痛苦的等待。对于兄弟们来说，gradle已经变成了一个潜藏的‘岁月神偷’。So，现在是时候我们来优化一下gradle的编译速度。

<!--more-->

本文优化的实践主要是参考[Android Weekly](http://androidweekly.net/)上推荐的一篇文章[6 tips to speed up your Gradle build](https://medium.com/@shelajev/6-tips-to-speed-up-your-gradle-build-3d98791d3df9#.3ait6jmd3)和[gradle官方文档](https://docs.gradle.org/current/userguide/userguide.html)。

先看看我们都有神马方法可以用先：

## Gradle Daemon

Gradle Daemon是gradle官方极力推荐的一个优化gradle编译速度的方法在1.0之前的版本就已经提供，经过这么多的版本迭代，已经非常成熟。如果你的gradle版本足够新并且没有开启Daemon的话，在你的编译完成之后，经常会看到这样一句话：

```
This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.8/userguide/gradle_daemon.html
```

Gradle Daemon是一个长期生存（3个小时不被调用会自动结束）、能够提升编译速度的后台进程。它的优化原理有几个方面：

- 由于gradle是运行在JVM之上的，并且有较多的库依赖，长期运行在后台能够节省每次编译需要重新初始化的时间。
- 另外一个很重要的一点是通过运行时代码优化来提升编译性能。这种优化是循序渐进的，而不是立马见效的那种，也就是随着编译次数的增多，优化效果会越来越好，一般来讲在5-10次编译以后，这种优化效果会趋于稳定。
- Gradle Daemon通过编译缓存提高效率。如gradle能缓存一些编译时的输入和输出，支持增量编译。

gradle官方对于Gradle Daemon还有更多的期待，比如预下载依赖库等

### 开启Daemon

Gradle Daemon默认不是开启的，我们可以有多个方式开启的deamon，但是官方推荐的方法是在系统的gradle配置文件（$USER_HOME/.gradle/gradle.properties）中：

```
org.gradle.daemon=true
```

如果该文件不存在，需要创建该文件。

我们还可以在执行命令后面添加`--no-daemon`和`--daemon`指定某次编译过程是否开启deamon。

	不过在持续集成中，gradle官方建议不要开启deamon以保证每次编译的独立性。
	
## Configuration on demand

gradle编译区分为三个阶段：

- 初始化，gradle支持单个或多个项目同时编译，在初始化阶段，gradle决定哪些项目参与编译，并为每一个项目创建一个[Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)实例。
- 配置阶段，对所有的项目进行配置，会执行项目里的build.gradle文件，下载相关的插件和依赖等，决定需要执行哪些任务的集合。
- 执行阶段，执行在配置阶段确定的所有task。

按需配置（Configuration on demand）只对任务相关的项目进行配置，这在大型多项目编译过程中非常有用，能够大幅度的减少不必要的配置时间。gradle官方表示在长期的角度来看，按需配置会变成一项默认模式，甚至是唯一的模式。

不过在大部分的安卓项目中，由于项目会太，且一般多使用aar的方式引用，所以该配置项对于安卓开发编译的优化效果没有那么大。按需配置功能现在还是孵化中，并没有正式发布。

### Configuration on demand开启

和Daemon的配置类似，我们也可以在系统的`gradle.properties`中添加如下配置：

```
org.gradle.configureondemand=true
```

或者在编译命令后面添加`--configure-on-demand`。

## parallel

并行执行在多项目编译的项目中能有效提升编译的速度，但是并行执行的前提是每个项目已经被模块化，每个项目之间没有耦合。并行执行功能现在也还是孵化中，并没有正式发布。

### 开启parallel

```
org.gradle.parallel=true
```

或者在编译命令后面添加`--parallel`

## 项目解耦

Gradle允许任何项目在配置和执行阶段访问其他项目。这是一把双刃剑，一方面为编译提供了灵活性；另一方面也会带来一些负面的阻碍，对并行编译和按需配置产生影响。

两个互相解耦的项目之间最多只能有申明的依赖关系，任何其他形式的交互都被定为成耦合。一种常见的耦合方式就是使用配置注入，例如在项目中使用`allprojects`或`subprojects`关键字就会导致项目之间的耦合。通常这会被定义在一个只有一些公共配置的‘根项目’中，这导致这个‘根项目’被耦合到所有子项目中，但这是可以被接受的。但如果任何子项目中使用`allprojects`或`subprojects`是就会影响到并行编译和按需配置的效率。

为了充分利用项目之间解耦的优势，gradle官方已经在尝试引入不会产生耦合的新特性来处理配置注入这种常用功能。

关于如何解耦项目，官方给了两条建议：

- 避免在一个子项目的build.gradle中去引用其他的子项目，最好是从‘根项目’中引用；
- 避免在执行时改变其他项目的配置。

## 其他优化方法

- 优化依赖，gradle允许我们在依赖一个项目是可以指定一个版本范围（如下），这会导致gradle每次都会去检测当前版本是不是最新版本，这会带来不必要的资源消耗，尤其在网络环境差时。而且这种写法还会导致版本兼容和持续集成时的不一致性等问题。

```
dependencies {
	compile 'com.google.code.gson:gson:2.+'
}
```
- gradle的每次更新都会不断优化它的编译性能，同时也可能会提供更多新的特性去优化编译，所以我们应该尽量使用最新发布的正式版本去编译，如2.4这个版本在编译速度方面就做了非常大提升。

说了这么多，下面让我们来实践一下：

## 优化实践

为了比较优化前后的差异，我先记录一下优化前的编译实践。我这里选取的是`assembleDebug`的执行时间。

为了减少误差，我每次修改少量代码，编译了五次。虽然不能作为实验数据使用，但是用来说名问题应该已经够了。五次执行的时间如下，平均大概在1min30s。

## 

```
Executing external task 'assembleDebug'

...

BUILD SUCCESSFUL

Total time: 1 mins 32.81 secs

Total time: 1 mins 19.539 secs

Total time: 1 mins 22.601 secs

Total time: 1 mins 26.14 secs

Total time: 1 mins 38.686 secs

```

然后，开始优化我们的编译配置。

- 在我的gradle全局配置中添加如下配置：

```
org.gradle.daemon=true
org.gradle.configureondemand=true
org.gradle.parallel=true
```

- 由于Dexguard的阻碍，我们的项目的gradle版本不能高于2.8版本，所以只能放弃（最新版本已经是2.11）。
- 检查了一下所有的依赖没有使用版本范围的依赖。
- 检查所有项目除了根项目，没有项目间的耦合。

准备就绪，开始执行。

```
gradle assembleDebug
Parallel execution with configuration on demand is an incubating feature.

Total time: 1 mins 41.859 secs

Total time: 1 mins 1.363 secs

Total time: 1 mins 0.425 secs

Total time: 59.915 secs

Total time: 56.681 secs

Total time: 56.047 secs

Total time: 55.667 secs

```

可以看到编译时间经过几次编译以后，基本稳定在50-60s之间，编译速度有了明显的提示。

为了区分哪个优化效果最好，我还做了几个简单的实验，最后发现按需配置和并行编译并没有明显的提升，能够带来明显提升的就是deamon。这应该和我本地只有两个直接的代码依赖库有关。

最后需要说明的是，Android Studio在编译时已经开启一些优化方案，如deamon和按需配置。针对AS上的编译优化，我的建议就是在条件允许的情况下，及时升级最新的AS版本、Android的gradle插件版本、gradle的版本等相关工具的版本。

## 参考文件

- [6 tips to speed up your Gradle build](https://medium.com/@shelajev/6-tips-to-speed-up-your-gradle-build-3d98791d3df9#.3ait6jmd3)
- [Configuration on demand](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:configuration_on_demand)
- [Multi-project builds](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:build_phases)
- [The Gradle Daemon](https://docs.gradle.org/current/userguide/gradle_daemon.html)
- [The Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)
- [Decoupled Projects](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:decoupled_projects)
