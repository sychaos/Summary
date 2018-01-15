## Activity：
* 启动 Activiy：onCreate -> onStart() -> onResume(), Activity 进入运行状态
* Activity 退居后台 ( Home 或启动新 Activity ): onPause() -> onStop()
* Activity 返回前台: onRestart() -> onStart() -> onResume()
* Activity 后台期间内存不足情况下当再次启动会重新执行启动流程
* 锁屏: onPause() -> onStop()
* 解锁: onStart() -> onResume()

## Fragment:
* 创建 Fragment: onAttach() -> onCreate() -> onCreateView() -> onActivityCreate() -> onStart() -> onResume()
* 销毁 Fragment: onPause() -> onStop() -> onDestroyView() -> onDestroy() -> onDetach()
* 从返回栈中回到上一个 Fragment: onDestroyView() -> onCreateView() -> onActivityCreated() -> onStart() -> onResume()

## Activity与Fragment数据传递:
* Fragment中通过getActivity()然后进行强制转化，调用Activity中的公有方法
* setArguments
* 广播

![](res/fragment与activity的关系.png)

## Activity 的四种启动模式和特点
一个应用默认只有一个任务栈
* standard ( 标准模式 ) : 每次启动都会创建一个新的实例，并放入栈顶。（无法使用非 Activity 类的 Context 启动该模式的 Activity，如果启动需要加上 FlAG_ACTIVITY_NEW_TASK标记创建一个栈）。
* singleTop ( 栈顶复用模式 ) : 如果启动的 Activity 在栈顶，则该 Activity 不会重建，同时 Activity 的 onNewIntent() 方法会被调用，如果不在栈顶则通 standard 模式。
* singleTask ( 栈内复用模式 ) : 若启动的 Activity 在栈内，则不会创建新的实例调用 onNewIntent() 方法，并将该 Activity 上面所以的 Activity 清空（ 如果其他应用启动该 Activity，若不存在则简历新的 Task，若存在在后台，则后台 Task 也切换到前台）。
* singleInstance ( 单例模式 ) : 新的 Activity 直接创建新的任务栈，当该模式的 Activity 存在于某个栈中，后面任何激活该 Activity 都会重用该实例。

## Activity 的缓存方法
Activity 由于异常终止时（被系统判定要回收时），系统会调用 onSaveInstanceState()来保存 Activity 状态 ( onStop() 之前和 onPause() 没有既定的时序关系 )。当重建时，会调用 onRestoreInstanceState()( onStart() 之后和 onPause() 没有既定的时序关系 )，并且把 Activity 销毁时 onSaveInstanceState() 方法所保存的 Bundle 对象参数同时传递给 onSaveInstanceState() 和 onCreate() 方法。因此，可通过 onRestoreInstanceState() 方法来恢复 Activity 的状态，该方法的调用时机是在 onStart() 之后。
onCreate()和 onRestoreInstanceState() 的区别：onRestoreInstanceState()回调则表明其中 Bundle 对象非空，不用加非空判断。onCreate() 需要非空判断。建议使用onRestoreInstanceState().

## 怎样退出终止App。
* 创建一个集合类对所有活动进行管理，ActivityCollector，通过 list 来管理 Activity。
* killProcess(android.os.Process.myPid()) 杀死当前程序进程。
    android中所有的activity都在主进程中，在Androidmanifest.xml中可以设置成启动不同进程，Service不是一个单独的进程也不是一个线程。
    当你Kill掉当前程序的进程时也就是说整个程序的所有线程都会结束，Service也会停止，整个程序完全退出。
* System.exit(0)
    当我们在写java程序时肯定用到过System.exit(0),它的意思是退出JVM（java虚拟机），在android中一样可以用，我们可以想像一下虚拟机都退出了当然执行System.exit的程序会完全退出，内存被释放。