# View事件传递机制

  跟touch事件相关的3个方法：
  public boolean dispatchTouchEvent(MotionEvent ev);    //用来分派event
  public boolean onInterceptTouchEvent(MotionEvent ev); //用来拦截event
  public boolean onTouchEvent(MotionEvent ev);          //用来处理event

  |  类       | 子类    |  方法  |
  | --------   | -----:   | :----: |
  | Activity类：        | Activity     | dispatchTouchEvent()、 onTouchEvent()、 |
  | Viewgroup（ViewGroup的子类）：       | FrameLayout、LinearLayout、ListView、ScrollVIew……      | dispatchTouchEvent()、onInterceptTouchEvent()、 onTouchEvent(); | 
  | View控件（非ViewGroup子类）：        | Button、TextView、EditText……      | 	dispatchTouchEvent()、onTouchEvent() |
  
  dispatchTouchEvent()	用来分派事件。其中调用了onInterceptTouchEvent()和onTouchEvent()，一般不重写该方法  
  onInterceptTouchEvent()	用来拦截事件。ViewGroup类中的源码实现就是{return false;}表示不拦截该事件，事件将向下传递（传递给其子View）；若手动重写该方法，使其返回true则表示拦截，事件将终止向下传递，事件由当前ViewGroup类来处理，就是调用该类的onTouchEvent()方法  
  onTouchEvent()	用来处理事件。返回true则表示该View能处理该事件，事件将终止向上传递（传递给其父View）；返回false表示不能处理，则把事件传递给其父View的onTouchEvent()方法来处理  
  
activitydispatchTouchEvent
```java

public boolean dispatchTouchEvent(MotionEvent ev) {  
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {  
        //这个函数其实是个空函数，啥也没干，如果你没重写的话，不用关心  
        onUserInteraction();  
    }  
    //这里事件开始交给Activity所附属的Window进行派发，如果返回true，整个事件循环就结束了  
    //返回false意味着事件没人处理，所有人的onTouchEvent都返回了false，那么Activity就要来做最后的收场。  
    if (getWindow().superDispatchTouchEvent(ev)) {  
        return true;  
    }  
    //这里，Activity来收场了，Activity的onTouchEvent被调用  
    return onTouchEvent(ev);  
}  
  
```



![](res/事件分发机制.png)
![](res/事件分发机制2.png)

如图所示 touch事件由 activity->viewgroup->view 的顺序调用 dispatchTouchEvent 如果 dispatchTouchEvent返回为false则事件不再向下传递，调用该层父类的onTouch（如果没有则调用onTouchEvent）然后如果该层的onTouch（onTouchEvent）返回为false则调用该层的上一层的onTouch（onTouchEvent）如果一路走来都没有就调用activity的onTouch（onTouchEvent）返回为true则该事件被消费

对于dispatchTouchEvent 返回 false 的含义应该是：事件停止往子View传递和分发同时开始往父控件回溯（父控件的onTouchEvent开始从下往上回传直到某个onTouchEvent return true），事件分发机制就像递归，return false 的意义就是递归停止然后开始回溯。

每个ViewGroup每次在做分发的时候，问一问拦截器要不要拦截（也就是问问自己这个事件要不要自己来处理）如果要自己处理那就在onInterceptTouchEvent方法中 return true就会交给自己的onTouchEvent的处理，如果不拦截就是继续往子控件往下传。默认是不会去拦截的，因为子View也需要这个事件，所以onInterceptTouchEvent拦截器return super.onInterceptTouchEvent()和return false是一样的，是不会拦截的，事件会继续往子View的dispatchTouchEvent传递。

## 总结
  对于 dispatchTouchEvent，onTouchEvent，return true是终结事件传递。return false 是回溯到父View的onTouchEvent方法。
ViewGroup 想把自己分发给自己的onTouchEvent，需要拦截器onInterceptTouchEvent方法return true 把事件拦截下来。
ViewGroup 的拦截器onInterceptTouchEvent 默认是不拦截的，所以return super.onInterceptTouchEvent()=return false；
View 没有拦截器，为了让View可以把事件分发给自己的onTouchEvent，View的dispatchTouchEvent默认实现（super）就是把事件分发给自己的onTouchEvent。


  activity把事件交给了 phoneWindow,向下传递。 如果下层没有处理这个事件，那么activity将调用自己的onTouchEvent来处理这个事件。  
  PhoneWindow的superDispatchTouchEvent将事件传递给了DecorView。   
  DecorView的superDispatchTouchEvent调用了ViewGoup的 dispatchTouchEvent方法。 
  ViewGroup的dispatchTouchEvent

