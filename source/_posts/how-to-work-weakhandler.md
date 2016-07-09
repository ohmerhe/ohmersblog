title: WeakHandler 是怎么解决 Handler 的内存问题的
date: 2016-01-18 23:32:01
tags: 
- 内存泄露
- Android

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/bridge_huge.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/bridge_huge.jpg?imageView2/1/w/1024/h/460 

---


## 问题回顾

### 内存泄露

什么是内存泄露，请参考[Java的内存泄漏](https://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/)

### Handler内存泄露

对于安卓的初学者，常见的handler的写法如下：

<!--more-->

```java
public class SampleActivity extends Activity {
  private final Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ... 
    }
  }
}
```
在上面的代码中，mHandler作为SampleActivity的内部匿名类的对象，持有对父类对象的引用。为了保证在子线程（一般是后台线程，可能比较耗时）能够通过Handler和主线程进行通讯，一般该子线程需要持有对应Handler的对象。因此，如上的代码即使SampleActivity被关闭，mHandler由于子线程的引用并不会被回收，同样mHandler厘面引用的SampleActivity对象也不会被回收（内存泄露）。

### Runnable内存泄露

在发起一个延时操作时，通常会这样写：

```java
mHandler.postDelayed(new Runnable() {
  @Override
  public void run() { /* ... */ }
}, 10 * 60 * 1000);
```
这里的实现Runnable接口的内部匿名类同样也持有了外部类（通常是Activity）的引用，这个对象作为Message的属性会一直存在直到达到指定的延时时长。这就很有可能在Activity被关闭时导致Activity不能及时被回收。

有一些文章说这种情况也会造成mHandler的泄露，这一点我不太认同，因为Message对象或者后面保存Message的MessageQueue本身都没有持有对mHandler的引用，所以按理说这里应该不会造成mHandler的泄露。

## 解决方案

对于上面的问题，已经可以找到比较多的解决方案，如下：

- 静态内部类或独立类，一般针对Handler泄露的方案。既避免使用非静态内部类，这样Handler的对象就不会因为持有外部Activity的对象而造成泄露。
- 弱引用，能够从根源处解决内存泄露问题。为了编写代码的方便性，一般不建议Runnable使用静态内部类或独立类来解决Runnable的泄露问题，而是通过让系统Message持有一个弱引用（[参考](https://www.ibm.com/developerworks/cn/java/j-jtp11225/)）来解决这个问题。
- 在逻辑上控制，这个受限于业务的限制，不过如果逻辑允许的话，可以在页面关闭时将对应的线程结束，或者将Runnable对象从队列中移除`mHandler.removeCallback()`。

## WeakHandler

下面图片显示WeakHandler的实现逻辑

![Screenshot](https://raw.githubusercontent.com/badoo/android-weak-handler/master/WeakHandler.png)

### 解决Handler内存泄露

WeakHandler强引用一个Handler子类(ExecHandler)的对象，然后通过自定义的一个Callback将Handler的消息处理转发到这个callback中，这样就不必为了处理消息而构建一个匿名内部handler类对象。WeakHandler对象（通过ExecHandler对象）仅仅维持对callback对象的弱引用。这样即使callback对象持有对Activity对象的引用，由于其本身不会产生泄露，因此

```java
private static class ExecHandler extends Handler {
    private final WeakReference<Callback> mCallback;
    ExecHandler() {
        this.mCallback = null;
    }
    ExecHandler(WeakReference<Callback> callback) {
        this.mCallback = callback;
    }
    ExecHandler(Looper looper) {
        super(looper);
        this.mCallback = null;
    }
    ExecHandler(Looper looper, WeakReference<Callback> callback) {
        super(looper);
        this.mCallback = callback;
    }
    public void handleMessage(@NonNull Message msg) {
        if(this.mCallback != null) {
            Callback callback = (Callback)this.mCallback.get();
            if(callback != null) {
                callback.handleMessage(msg);
            }
        }
    }
}
```
但是，WeakHandler的对象会维持对callback对象的强引用。

### 解决Runnable泄露问题

WeakHandler内部定义一个WeakRunnable用来包装我们传递进去的Runnable对象，在WeakRunnable中维持对Runnable对象的弱引用，从而解决了Runnable对象不释放而造成的内存泄露问题。

```java
static class WeakRunnable implements Runnable {
    private final WeakReference<Runnable> mDelegate;
    private final WeakReference<WeakHandler.ChainedRef> mReference;
    WeakRunnable(WeakReference<Runnable> delegate, WeakReference<WeakHandler.ChainedRef> reference) {
        this.mDelegate = delegate;
        this.mReference = reference;
    }
    public void run() {
        Runnable delegate = (Runnable)this.mDelegate.get();
        WeakHandler.ChainedRef reference = (WeakHandler.ChainedRef)this.mReference.get();
        if(reference != null) {
            reference.remove();
        }
        if(delegate != null) {
            delegate.run();
        }
    }
}
```
记住，WeakHandler的对象会维持对runnable对象的强引用。

### 注意事项

在使用WeakHandler时，应该在activity（或者fragment）中声明一个全局变量，以保证WeakHandler的生命周期和activity保持一致。

```java
Activity{
	private WeakHandler mHandler = new WeakHandler();
}
```
如果只是定义一个临时变量，在内存不足时handler会被回收，导致callback和runnable对象也会被回收，从而不能拿到回调。

或者如果定义一个静态变量的话，会导致callback和runnable不能被释放，从而导致内存泄露。

项目地址：[android-weak-handler](https://github.com/badoo/android-weak-handler)


## 参考内容

- [Java的内存泄漏](https://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/)
- [Android中使用Handler造成内存泄露的分析和解决](http://my.oschina.net/rengwuxian/blog/181449)
- [Java 理论与实践: 用弱引用堵住内存泄漏](https://www.ibm.com/developerworks/cn/java/j-jtp11225/)
- [Android Handler Memory Leaks](https://techblog.badoo.com/blog/2014/08/28/android-handler-memory-leaks)
