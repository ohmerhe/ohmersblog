title: 使用 Espresso 实现完整覆盖的功能测试
date: 2017-04-18 16:59:14
tags: 
- 测试
- Android
  
thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170003723.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170003723.jpg?imageView2/1/w/1024/h/460 

---


tags: Android, 测试

对于基于 UI 的功能测试的需求其实一直存在，理由其实很简单，不想一直让人去做重复机械的事情，而且可靠性完全是靠人力的堆积产生。然而现在行业大多数公司的功能测试工作依然主要是依靠人工来完成，从我们公司的实践来看我觉得有几个方面的因素的影响。

- 之前的 UI 测试框架的表现差强人意。就拿我们公司来说，其实测试部门在去年已经实现并推广一套主要基于 UIAutomator 实现的测试平台，但由于对复杂功能的处理能力较弱，基本只能实现部分功能的检测。这样导致的一个结果是，并不能有效减少测试的工作，而只能增加测试的额外工作，因此测试编写测试代码的积极性不是很高。同时由于测试代码的可重复利用性差，导致测试脚本的编写成本和维护成本偏高，实践中大家只用 UI 测试跑一些主流程业务，覆盖范围非常有限。

- 部分测试人员的编码能力不是很强。由于大部分测试人员可能并没有过多的开发经验，所以在编写测试代码时并不能很顺畅的完成自己想要的效果，这样也会导致测试代码项目的推广阻力会比较大。

- 对于怎么编写 UI 测试，并没有一个被大家接受认可的最佳实践。虽然我用 Espresso 实现了一套完整的覆盖方案，但是其实我用的方法和 Google 官方所建议的写法还是有蛮多差异的。

对比上面的几个因素，我觉得更为主要的原因还是在于现有测试平台对于复杂逻辑处理的能力不够，导致对于 UI 测试的依赖性仅仅局限在安装测试和兼容性测试，只能用来跑一些主流程的东西，对于大多数功能还只能依靠人工的方式完成。

## Espresso

Espresso 是 Google 在 2013 年推出的 Android UI 测试的开源框架。其实之前我们团队也多多少少对 Espresso 有过一些尝试，但遗憾的是都没有深入的进行实践。一季度我们将 UI 测试作为一个很明确的坑来填以后，发现 Espresso 已经很强大了，经过实践下来我们发现用 Espresso 实现 80-90% 的功能性覆盖测试基本没有什么问题。而且 Espresso 的测试脚本编写起来非常简单，如果测试和开发共同来完成测试代码的编写，能够有效替代测试大量的重复机械的工作。

下面我就来描述一下我们是怎么用 Espresso 来实现这一样一个完整覆盖的功能性测试平台。这篇文章会讲到一些在使用 Espresso 中遇到的坑，但是并不会在 How-to 的事情上面花太多的精力，如果你对 Espresso 还不是很了解的话，建议先去 [官方文档](https://google.github.io/android-testing-support-library/docs/espresso/index.html) 了解一些，并先进行一些简单的实践。

## ETP 测试方案

在介绍我们的测试平台方案之前，需要提前说明的是，我们在使用 Espresso 的方式可能和官方 Demo 里所展示的方式不完全一样。为了让我们写的测试代码能够更加灵活和方便的被复用在不同的测试用例中，从而实现更低成本的全功能覆盖，我们进行了一些方案设计，最后实现了我们现在的测试平台。

给我们的测试方案取了一个高大上的名称——ETP 测试方案，也就是 Espresso Test Platform 的简称。大家如果看过 Google 官方的 Demo 的话，就应该能理解官方的思路其实是每个测试是完成一条完整的逻辑测试，比如完成添加一个笔记的测试逻辑。

![](http://7xpox6.com1.z0.glb.clouddn.com/4A9E3284-F1C5-4ED5-8B7F-3E5D4ACF0475.png)

这样的测试流程总结下来有两个比较明显的问题：

- 每条测试用例需要单独编写，代码的复用性差，导致编写单元测试的成本较高，虽然 Google 官方提供录制测试脚本的功能，但是生成的代码可用性并不强，大多数情况下还是需要靠人来编写。

- 复杂场景的处理能力，这样一条单独的测试流程容易被触发性的弹窗或者引导提示打断，如果在单条测试中做过多的判断，又会让单条测试用例变得臃肿，从而让所有的测试用例都变得臃肿而且难于维护和更新。

### 单页面测试

这是我为我们测试平台定义的一个基本测试单位，也是我们整个测试方案里面的一个核心概念。这里有两个方面需要明确一下。

- 有些人会说这是 Espresso 的标准写法，因为你在定义一个 Espresso 测试文件时就必须要指定一个启动的 Activity。但是从官方的 Demo 来看，他们倾向于这个 Activity 仅仅是一个入口，你应该在完成一条完整的测试用例（你可以根据需要跳转到任何页面）以后再进行相应验证。但是在我们的这里，启动的这个页面（可以是 Activity 或者 Fragment）以后就只做当前页面相关的逻辑测试，如果有页面跳转也仅仅是测试是否能成功跳转，不会再对应的界面产生更多的交互。

- 由于 Espresso 是基于 JUnit 实现的，所以你可以针对单个页面编写多个独立得测试用例，它们会以随机的顺序被调用。在我们的单页面测试中，只有单一的测试入口，然后顺序执行这个页面需要执行的所有测试。

在单页面测试中，会根据需求尽可能覆盖这个页面的所有的功能。这个时候有人可能会说，在不同的应用状态下（比如：登录是否），通过 UI 测试所能产生的逻辑并不一致，怎么做到全覆盖能。我们的解决方案是在这个页面的测试代码中，需要全部覆盖该页面所有的逻辑分支，当开始执行这个单页面测试的时候，是怎么样的状态，就进入怎样的逻辑分支。这个时候又有人开始有疑问了，这样怎么做到全功能覆盖呢。想象力丰富的人可能已经想到了解决方案，我们先给到一张图启发一下，后面再介绍我们引入的下一个概念——测试流。

![](http://7xpox6.com1.z0.glb.clouddn.com/D2653BAE-B2DB-43E4-8A91-0FB9240DBA9A.png?imageView2/1/w/720)

PS：如果使用的 MVP 的模式来编写代码的话，你会发现在单个页面需要那些逻辑是非常清晰的。

虽然上面已经明确定义了单页面测试的写法，但是在实际应用过程中，还是会遇到一些场景，不在定义里面被约束，应该怎么处理会让人产生疑惑，会对单页面测试的能力的覆盖性产生怀疑。下面我列出来的几种情况常见的情况来解释应该怎么坚持单页面测试作为基本单位。

#### 页面关联性测试

当涉及到页面互相关联的逻辑，一个典型的场景就是在订阅页面订阅一个任务，然后在订阅列表页需要及时显示最新添加的订阅内容。官方的用例是把它放在一条测试里面就不用说了，在我们的测试方案里面应该怎么处理。

最后我们选择的方案是分开测试：

- 在订阅页面进行订阅相关测试，在这个页面只验证是否订阅成功。

- 在订阅列表界面，通过模拟订阅操作，发送一个订阅成功通知给这个页面，测试该页面是否会及时显示最新的订阅内容。

#### Activity 嵌套多个 Fragment 

由于 Espresso 是以 Activity 为入口的，所以可能会导致产生一些误会，我们的单页面就是完全对应到 Activity，其实在我们的设计里面单页面对应到的独立子页面的，所以 Fragment 可以作为一个单独的「单页面测试存在」。在我们的应用里面，首界面有三个 tab，所以加上 MainActivityTest 这个单页面测试首界面总共有四个单页面测试。

#### 随机触发性逻辑

在应用中完成某些任务后，会发送一个全局的广播，然后会在后台触发一些和当前页面没有什么关联弹窗。这种场景对我们的 UI 测试是个挑战，由于的随机性，除非在每次都做大量重复的判断，否则很容易导致 UI 测试被中断失败。

针对这样的场景我们有个小伙伴提了一个特别好的解决方案：

- 为这些随机的弹窗测试单独写测试，可以嵌入到在业务上认为合理的单页面测试中

- 一般这样的触发性的弹窗都会有相应的全局性的变量用来控制，在执行其他的单页面测试的时候，则手动将对应的控制开关关闭。

### 测试流和全功能覆盖

在每个页面的单页面测试都完成以后，接下来的任务就是怎么有效的将这些单页面组合起来。在单元测试中每个单元测试都是独立的，所以只要保证所有的测试用例被执行过就可以了。但是现在我们的目的是实现功能测试，所以一定会有一些状态下的逻辑需要测试。于是在单页面的基础上我们加入了测试流的概念。

![](http://7xpox6.com1.z0.glb.clouddn.com/BA340370-0A57-4C0A-8FB4-E93B41D51950.png?imageView2/1/w/720)

- 一条测试流其实是不同单页面测试的顺序执行。通过前面的单页面测试来后对应用的产生的输出，变成后面一个单页面测试的输入（如：在某条测试流中需要应用处于登录状态，则可以在整个测试流的第一个单页面测试应该是登录页面测试）。
- 单条测试流可以对应到某条业务的一条完整流程，一般会覆盖多个测试用例。
- 通过不同的测试流，来测试同一页面中不同的逻辑分支。
- 通过测试流的叠加来实现全功能的测试覆盖。这里的逻辑是当每个页面的所有逻辑都被测试过，则实现了全功能的测试覆盖。

## Espresso 的坑

虽然 Espresso 已经很强大了，但是从2.2这个版本以后，已经很久没有更行新版本了，其实里面还是有很多坑的，在使用 Espresso 的时候需要尽量避免。

### Idling 后面需要有 onView 的阻塞操作才能产生效果

在刚开始接触 IdlingResource 的时候，对它抱有太多的幻想，以为可以肆无忌惮的处理异步的问题，使用之后才发现问题其实也不少，甚至还有一些明显的 bug。

在 IdlingResource 的说明文档里说可以总结成这样一句话 `you have to use Idling Resources to inform Espresso of the app’s long-running operations.`。在这里我们并不能很直白的理解出 IdlingResource 只能用来等待 ` UI events`。也就是说在官方的设计里面，要使用 IdlingResource 来维持对应的后台操作，后面的紧跟着一条 UI 操作或验证，否则将不会生效。这样导致的结果是在一个后台操作完成以后是关闭当前页面的场景，根本无法测试。

如果这算是 Espresso 的特性的话，下面这个就是一个明显的 bug。

### 不同线程 IdlingResource 的 bug

在进行 UI 测试的时候，有两个主线程需要区分一下，一个是主 App 运行的主线程（[main,5,main]），另一个是 UI 测试跑的主线程（[Instr: android.support.test.runner.AndroidJUnitRunner,5,main]）。我们触发的UI事件都是在 App 主线程里面执行的，如果我们想要在 App 的线程里面做一些操作需要切换到对应的线程操作。如下面的代码：

```
mActivityTestRule.getActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                LogUtils.d(TAG, "runOnUiThread..." + Thread.currentThread());
                TaskApi.Companion.getMyTasks(0, 10000, "",  new HSAPICallback<TaskListResult>() {
                    public void onRequestSuccess(TaskListResult data, int httpStatus,Boolean fromCache) {
                        super.onRequestSuccess(data, httpStatus, fromCache);
                        mTasks = data.getDatas();
                    }
                });
            }
        });
```

理论上这里进行的异步操作应该和 App 里面执行的异步操作是一样的，可以用 IdlingResource 去守护这样一个后台操作，但是实际使用下来，虽然 IdlingResource 已经接受到对应的异步完成回调，但是并没有回调到被注册的 ResourceCallback。

### hasProperty 异常

Espresso 用的是 Hamcrest 的语法来进行的验证，理论上应该支持所有 Hamcrest 的写法，但是当我们在使用 hasProperty 这个方法的时候，会发现下面这样的错误。这主要是由于 Android SDK 里面并没有完整 JDK 的库，我们用到这部分刚好在 Android SDK 没有。

```
java.lang.NoClassDefFoundError: Failed resolution of: Ljava/beans/Introspector;
at org.hamcrest.beans.PropertyUtil.propertyDescriptorsFor(PropertyUtil.java:47)
at org.hamcrest.beans.PropertyUtil.getPropertyDescriptor(PropertyUtil.java:28)
at org.hamcrest.beans.HasPropertyWithValue.propertyOn(HasPropertyWithValue.java:94)
at org.hamcrest.beans.HasPropertyWithValue.matchesSafely(HasPropertyWithValue.java:81)
at org.hamcrest.TypeSafeDiagnosingMatcher.matches(TypeSafeDiagnosingMatcher.java:55)
at org.hamcrest.core.AllOf.matches(AllOf.java:27)
at org.hamcrest.DiagnosingMatcher.matches(DiagnosingMatcher.java:12)
at android.support.test.espresso.action.AdapterDataLoaderAction.perform(AdapterDataLoaderAction.java:83)
at android.support.test.espresso.ViewInteraction$1.run(ViewInteraction.java:144)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:422)
at java.util.concurrent.FutureTask.run(FutureTask.java:237)
at android.os.Handler.handleCallback(Handler.java:739)
at android.os.Handler.dispatchMessage(Handler.java:95)
at android.os.Looper.loop(Looper.java:135)
```

根据 [Android espresso onData error](http://baiduhix.blogspot.com/2015/07/android-espresso-ondata-error.html) 这篇文章可以找到对应的解决方案，但是实际使用下来效果并不好，主要是 gradle 的 Android 插件在不同的版本里面对于引入 Java Core 的代码处理方式有差别，而且我用的 2.2.3 的版本根本就不能用，所以这里的建议绕过不要使用这个方法，我们最后是通过自己定义了一个 Match 来解决这个问题的。

### Drawerlayout 的坑导致用例执行失败

在官方 Support 包里面的 Drawerlayout 控件有个特性，就是当你触摸到它的触发区域就会发出一个延时操作（160ms），如果这段时间内没有子试图触发事件（如没有及时抬起手指）就会自动触发 Peek 动画，从而导致子试图的事件得不到响应。

Espresso 的 perform(click()) 操作是先后发送了一个 EventDown 和 EventUp 事件，由于是异步发送的，所以没法保证每次两个事件的执行间隔在 160ms 之内，所以如果在 Drawerlayout 的拖拽相应区域内针对其他子试图的点击事件有可能得不到相应，从而导致用例执行失败。

这种失败并不是业务逻辑出错，完全是 Drawerlayout 控件设计不合理导致的，而且 Google 的大神们根本没有提供关闭这个功能的接口（他们这么任性不是一次两次了）。 Drawerlayout 是官方提供的控件，你可以选择不使用这个控件自定义一个。我们的解决方案是通过 AOP 的方式在 Drawerlayout 发送这个延时操作以后马上将他从执行队列里移除，从而实现将它这个功能关闭的效果。

```
@Pointcut("execution(* android.support.v4.widget.DrawerLayout.ViewDragCallback.onEdgeTouched(..))")
public void edgeTouchedPoint() {}

@After("edgeTouchedPoint()")
public void onExecutionPoint(final JoinPoint joinPoint) throws Throwable {
    Reflect.on(joinPoint.getTarget()).call("removeCallbacks");
}
```

## 结束语

经过诸多尝试和实践，虽然过程中碰到了很多的问题，但最后都逐个被解决，证明用 Espresso 实现全功能覆盖的可行性。我们的方案已经开始实施，截止到现在虽然还没有完全实现覆盖，但是已经完成的部分已经开始产生效果，多次检测到由于开发不小心而导致的 bug，提高了产品的提测质量。可以预见在该方案完整实施后，将对产品质量有一个稳定、高效的保证。

## 参考内容

- [espresso](https://google.github.io/android-testing-support-library/docs/espresso/index.html)
- [Espresso Idling Resource doesn't work
](http://stackoverflow.com/questions/33120493/espresso-idling-resource-doesnt-work)
- [Android espresso onData error](http://baiduhix.blogspot.com/2015/07/android-espresso-ondata-error.html)
