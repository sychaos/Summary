# Binder机制
在Activity和Service进行通讯的时候，用到了Binder。
    * 1.当属于同个进程我们可以继承Binder然后在Activity中对Service进行操作
    * 2.当不属于同个进程，那么要用到AIDL让系统给我们创建一个Binder，然后在Activity中对远端的Service进行操作。

写一个消息...
预览帮助
选择文件

Looper阻塞为什么不ANR


# Handler机制
    Android消息机制包含: MessageQueue、Handler、Looper、Message.
    Message: 需要传递的消息，可以传递数据。
    MessageQueue: 消息队列，通过单链表数据结构来维护消息列表，（投递和删除消息）。
    Handler: 消息辅助类，向消息池发送各种消息事件
    Handler.sendMessage 发送消息
    Handler.handleMessage 处理消息
    Looper: 不断从消息队里中读取消息，按分发机制将消息分发给目标处理者。

# MessageQuene、Handler 和 looper 三者之间的关系：
    每个线程中只能存在一个 Looper，保存在 ThreadLocal 中。主线程（UI线程）已经创建了一个 Looper，所以在主线程中不需要再创建 Looper，其他线程中需要创建 Looper。每个线程中可以有多个 Handler，即一个 Looper 可以处理来自多个 Handler 的消息。 Looper 中维护一个 MessageQueue，来维护消息队列，消息队列中的 Message 可以来自不同的 Handler。
    当调用 handler.sendMessage() 发送 message 时，实际上发送到与当前线程绑定的 MessageQuene 中，然后当前线程绑定的 Looper 不断从 MessageQueue 取出新的 Message，调用 msg.target.disspatchMessage(msg) 方法将消息分发到与 Message 绑定的 handler.handleMessage()中。
