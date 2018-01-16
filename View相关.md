# View相关

## View的绘制流程

Activity启动时，ActivityThread.handleResumeActivity()方法中建立了decorView与ViewRoot的关联关系，当建立好了decorView与ViewRoot的关联后，ViewRoot类的requestLayout()方法会被调用，以完成应用程序用户界面的初次布局。实际被调用的是ViewRootImpl类的requestLayout()方法
上面的方法中调用了scheduleTraversals()方法来调度一次完成的绘制流程，该方法会向主线程发送一个“遍历”消息，最终会导致ViewRootImpl的performTraversals()方法被调用。
![](res/view绘制.png)
最终会导致ViewRootImpl的performTraversal会依次调用measure，layout，draw
measure阶段：TODO
layout阶段：TODO
draw阶段： TODO

自定义View如何考虑机型适配
自定义View如何提供获取View属性的接口；TODO 点击事件之类的

## 自定义View过程：onMeasure()、onLayout()、onDraw()
    * onMeasure() 方法：单一 View，一般重写此方法，针对 wrap_ content 情况，规定 View 默认的大小值，避免于 match_parent 情况一致。ViewGroup，若不重写，就会执行和单子View 中相同逻辑，不会测量子 View。一般会重写 onMeasure() 方法，循环测量子 View。
    * onLayout()方法:单一 View，不需要实现该方法。ViewGroup 必须实现，该方法是个抽象方法，实现该方法，来对子 View 进行布局。
    * onDraw() 方法：无论单一View，或者 ViewGroup 都需要实现该方法，因其是个空方法