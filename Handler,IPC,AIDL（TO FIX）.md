# IPC机制
   是指进程间通信的机制 android的IPC方式有Binder机制和Socket，ContentProvider

# Binder机制
在Activity和Service进行通讯的时候，用到了Binder
1. 当属于同个进程我们可以继承Binder然后在Activity中对Service进行操作  
1. 当不属于同个进程，那么要用到AIDL让系统给我们创建一个Binder，然后在Activity中对远端的Service进行操作。

Looper阻塞为什么不ANR

# Handler机制
   Android消息机制包含: MessageQueue、Handler、Looper、Message.
   1. Message: 需要传递的消息，可以传递数据。
   1. MessageQueue: 消息队列，通过单链表数据结构来维护消息列表，（投递和删除消息）。
   1. Handler: 消息辅助类，向消息池发送各种消息事件
   Handler.sendMessage 发送消息
   Handler.handleMessage 处理消息
   1. Looper: 不断从消息队里中读取消息，按分发机制将消息分发给目标处理者。

# Looper有什么作用？在一个非UI线程中使用Looper是怎样的步骤？在Looper.loop()方法的打印语句会执行吗？
   1. Looper: 不断从消息队里中读取消息，按分发机制将消息分发给目标处理者。
   1. 要先调用Looper.prepare()  创建该线程的Looper对象
   1. 写在Looper.loop()之后的代码不会被立即执行，当调用后mHandler.getLooper().quit()后，loop才会中止，其后的代码才能得以运行

# MessageQueue、Handler和Looper三者之间的关系：
   1. 每个线程中只能存在一个 Looper，保存在 ThreadLocal 中。主线程（UI线程）已经创建了一个 Looper，所以在主线程中不需要再创建 Looper，其他线程中需要创建 Looper。
   1. 每个线程中可以有多个 Handler，即一个 Looper 可以处理来自多个 Handler 的消息。 Looper 中维护一个 MessageQueue，来维护消息队列，消息队列中的 Message 可以来自不同的 Handler。
   1. 当调用 handler.sendMessage() 发送 message 时，实际上发送到与当前线程绑定的 MessageQueue 中，然后当前线程绑定的 Looper 不断从 MessageQueue 取出新的 Message，
    调用 msg.target.dispatchMessage(msg) 方法将消息分发到与 Message 绑定的 handler.handleMessage()中。
