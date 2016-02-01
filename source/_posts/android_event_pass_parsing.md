title: 安卓事件传递机制分析
date: 2016-02-01 18:38:00
tags: 
- android
- touchevent
- 事件传递

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-137723945.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-137723945.jpg?imageView2/1/w/1024/h/460 

---

这篇文章基于Android4.2的源码分析得出，写的比较早，拿出来晒晒。

这片文章讲解的事件传递的起源从dispatchTouchEvent(event)开始，根据事件的处理流程逐渐展开，直至事件被可预料的处理掉结束。

<!--more-->

先贴一张个人总结的事件传递的流程图，如果可以将这张图清楚的理解，下面的文章就可以不用看了，因为这篇文章的主要内容也就是围绕这幅图展开。

![事件分发流程图](https://gitlab.yeshj.com/uploads/android_demos/android_issues/ea9effbc2b/android_view_.png)

## ViewGroup中的事件处理

在用户触碰屏幕后，经过系统一系列处理后，会分发到的View的dispatchTouchEvent方法中，事件将在这个方法中进行分发，决定该事件的去向。

由于安卓的事件处理顺序是由外至里的，既外层视图最先拿到对应的事件，既事件会优先传递到ViewGroup的dispatchTouchEvent方法中。在自定义视图中可以重写dispatchTouchEvent这个方法定义事件的进一步分发，本文分析的是ViewGroup默认的分发机制。

在ViewGroup中默认先将事件分发给onInterceptTouchEvent方法，通过该方法的返回来判断当前视图是否中断事件的进一步分发，如果onInterceptTouchEvent返回true，则该事件认为已经被消耗不会继续分发下去。

```
if (!disallowIntercept) {  
    intercepted = onInterceptTouchEvent(ev);  
    ev.setAction(action); // restore action in case it was changed  
    } else {  
    intercepted = false;  
}
```

ViewGroup默认并不会中断该事件，而是直接返回false。在自定义的视图中，可以通过重写onInterceptTouchEvent返回true而中断所有事件的分发。

```
public boolean onInterceptTouchEvent(MotionEvent ev) {  
    return false;  
} 
``` 

如果事件在当前视图没有被截取，ViewGroup会继续分发事件，判断自己是否有子视图符合接收该事件的条件，如果有的话，则直接将事件分发给该子视图，并返回true代表在这层事件已经被分发出去。该视图的子视图可以是一个普通的view，也可以是一个Viewgroup。当子视图是一个View的时候，请参考下面View的事件处理部分。当子视图是一个ViewGroup，则重复前面描述的分发逻辑。

如果没有子视图消耗掉当前的事件，这个事件最终被传递到ViewGroup本身，这个时候ViewGroup将作为一个普通的View继续处理事件（详细参见后面的View处理事件部分）
    
```
if (!canViewReceivePointerEvents(child)|| !isTransformedTouchPointInView(x, y, child, null)) {  
    continue;  
}
newTouchTarget = getTouchTarget(child);  
if (newTouchTarget != null) {  
    // Child is already receiving touch within its bounds.  
    // Give it the new pointer in addition to the ones it is handling.  
    newTouchTarget.pointerIdBits |= idBitsToAssign;  
    break;  
}  
  
resetCancelNextUpFlag(child);  
if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {  
    // Child wants to receive touch within its bounds.  
    mLastTouchDownTime = ev.getDownTime();  
    mLastTouchDownIndex = childIndex;  
    mLastTouchDownX = ev.getX();  
    mLastTouchDownY = ev.getY();  
    newTouchTarget = addTouchTarget(child, idBitsToAssign);  
    alreadyDispatchedToNewTouchTarget = true;  
    break;  
} 
```

## View中的事件处理

View中的事件分发仍然是从dispatchTouchEvent方法开始。该方法中首先会将事件分发给调用到mOnTouchListener，mOnTouchListener是我们在使用view的setOnTouchListener方法时注册进去的监听。如果我们在注册进去监听的onTouch方法中处理了该事件并且返回了true则代表该事件已经被消耗，事件将不会在继续传递。
      
```
ListenerInfo li = mListenerInfo;  
if (li != null && li.mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED  
    && li.mOnTouchListener.onTouch(this, event)) {  
    return true;  
}if (onTouchEvent(event)) {  
    return true;  
}
```

而如果没有设置监听或返回为false的话，该事件将会被传递到onTouchEvent方法。在View类默认的onTouchEvent方法中，会将事件分发到视图的click或者longClick事件。

用户需用通过setOnClickListener或者setOnLongClickListener设置click的处理，如果我们设置了listener，则onTouchEvent分发完事件后会返回true通知该事件已经被消耗。

```
if (mPerformClick == null) {  
    mPerformClick = new PerformClick();  
}  
if (!post(mPerformClick)) {  
    performClick();  
}
ListenerInfo li = mListenerInfo;  
if (li != null && li.mOnClickListener != null) {  
    playSoundEffect(SoundEffectConstants.CLICK);  
    li.mOnClickListener.onClick(this);  
    return true;  
}
```

ps: 从上面的代码指导view的click方法并不是直接调用执行的，而是通过post将click的处理延迟以保证视觉效果的优先执行。

## 总结

- 首先接收到事件的视图是最外层的视图，然后再往子视图上传递
- 事件的传递是一个递归过程
- 在上述每个环节都可以通过返回true的方法消耗该事件，结束事件的传递
- 自定义视图的事件传递过程决定于其对应继承的方法，但应该遵守上述的规则
