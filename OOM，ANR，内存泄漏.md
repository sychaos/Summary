# OOM，ANR，内存泄漏
## OOM
  OOM内存溢出，申请的内存超出了JVM能提供的内存大小，此时称之为溢出。
  一般而言，android中常见的原因主要有以下几个：
  1.数据库的cursor没有关闭。
  
  2.构造adapter没有使用缓存contentview。
 
  3.调用registerReceiver()后未调用unregisterReceiver().
  4.未关闭InputStream/OutputStream。
  
  5.Bitmap使用后未调用recycle()。
  
  6.Context泄漏。 当activity销毁时，如果有持有该activity的内部类，会导致activity无法正常销毁，解决办法设置为静态内部类，线程内部采用弱引用保存Context引用
  
  7.static关键字等。 定义为static的对象持有另一个对象会导致该对象无法被销毁
  
  8.使用bitmap时谨慎处理：及时回收，设置一定的采样率，巧妙的运用软引用
  
## 内存泄漏
  1.对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。
  
  2.静态内部类持有外部成员变量（或context）:可以使用弱引用或使用ApplicationContext。
  
  3.内部类持有外部类引用,异步任务中，持有外部成员变量。
  
  4.集合中没用的对象没有及时remove。
  
  5.不用的对象及时释放，如使用完Bitmap后掉用recycle（），再赋null。
  
  6.handler引起的内存泄漏，MessageQueue里的消息如果在activity销毁时没有处理完，就会引起内存的泄漏，可以使用弱引用解决。
  
  7.设置过的监听不用时，及时移除。如在Destroy时及时remove。尤其以addListener开头的，在Destroy中都需要remove。
  
  8.activity泄漏可以使用LeakCanary。
  
## ANR

  1.KeyDispatchTimeout(5 seconds) –主要类型按键或触摸事件在特定时间内无响应
  
  2.BroadcastTimeout(10 secends) –BroadcastReceiver在特定时间内无法处理完成
  
  3.ServiceTimeout(20 secends) –小概率事件 Service在特定的时间内无法处理完成
  
  ### 发生原因 ：
  主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
  
  主线程中存在耗时的计算
  
  主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况
  
  使用AsyncTask处理耗时IO操作。
