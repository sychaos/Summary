# AsyncTask
## 原理：
  AsyncTask 封装了两个线程池和一个 Handler，（SerialExecutor 用于排队，THREAD_POOL_EXECUTOR 为真正的执行任务，Handler 将工作线程切换到主线程），
  其必须在 UI 线程中创建，execute 方法必须在 UI 线程中执行，一个任务实例只允许执行一次，执行多次将抛出异常，用于网络请求或者简单数据处理。
## 注意：
  AsyncTask的类必须在UI线程加载（从4.1开始系统会帮我们自动完成）
  
  ### AsyncTask对象必须在UI线程创建
  AsyncTask实例必须UI线程创建的原因如下：
  需要在主线程创建InternalHandler，以便onProgressUpdate， onPostExecute ， onCancelled 可以正常更新UI。
  AsyncTask实例的execute方法必须在主线程调用的原因如下：
  保证 onPreExecute 正常的更新UI。
  
  execute方法必须在UI线程调用

  不要在你的程序中去直接调用onPreExecute(), onPostExecute, doInBackground, onProgressUpdate方法
  
  一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常
  
  AsyncTask不是被设计为处理耗时操作的，耗时上限为几秒钟，如果要做长耗时操作，强烈建议你使用Executor，ThreadPoolExecutor以及FutureTask
  在1.6之前，AsyncTask是串行执行任务的，1.6的时候AsyncTask开始采用线程池里处理并行任务，但是从3.0开始，为了避免AsyncTask所带来的并发错误，AsyncTask又采用一个线程来串行执行任务
  
  关于线程池：asyncTask对应的线程池ThreadPoolExecutor都是进程范围内共享的，都是static的，所以是asyncTask控制着进程范围内所有的子类实例。由于这个限制的存在，
  当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0后默认串行执行，不会出现这个问题)。针对这种情况，可以尝试自定义线程池，配合asynctask使用。
  
  关于默认线程池：核心线程池中最多有CPU_COUNT+1个，最多有CPU_COUNT*2+1个，线程等待队列的最大等待数为128，但是可以自定义线程池。
  线程池是由AsyncTask来管理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉，类似volatile变量。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。
  
  通过查阅官方文档发现，AsyncTask首次引入时，异步任务是在一个独立的线程中顺序的执行，也就是说一次只能执行一个任务，
  不能并行的执行，从1.6开始，AsyncTask引入了线程池，支持同时执行5个异步任务，也就是说同时只能有5个线程运行，超过的线程只能等待，等待前面的线程某个执行完了才被调度和运行。
  换句话说，如果一个进程中的AsyncTask实例个数超过5个，那么假如前5个都运行很长时间的话，那么第6个只能等待机会了。这是AsyncTask的一个限制，而且对于2.3以前的版本无法解决。
  如果你的应用需要大量的后台线程去执行任务，那么你只能放弃使用AsyncTask，自己创建线程池来管理Thread。不得不说，虽然AsyncTask较Thread使用起来方便，但是它最多只能同时运行5个线程，
  这也大大局限了它的实力，你必须要小心设计你的应用，错开使用AsyncTask的时间，尽力做到分时，或者保证数量不会大于5个，否则就会遇到上次提到的问题。可能是Google意识到了AsyncTask的局限性了，
  从Android3.0开始对AsyncTask的API作出了一些调整：每次只启动一个线程执行一个任务，完成之后再执行第二个任务，也就是相当于只有一个后台线程在执行所提交的任务。

## 缺陷
  1. 生命周期
  很多开发者会认为一个在Activity中创建的AsyncTask会随着Activity的销毁而销毁。然而事实并非如此。AsyncTask会一直执行，直到doInBackground()方法执行完毕。然后，如果cancel(boolean)被调用,那么onCancelled(Result result)方法会被执行；否则，执行onPostExecute(Result result)方法。如果我们的Activity销毁之前，没有取消AsyncTask，这有可能让我们的AsyncTask崩溃(crash)。因为它想要处理的view已经不在了。所以，我们总是必须确保在销毁活动之前取消任务。总之，我们使用AsyncTask需要确保AsyncTask正确的取消。
  2. 内存泄漏
如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄漏。
  3. 结果丢失
屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。
  4. 并行还是串行
在Android1.6之前的版本，AsyncTask是串行的，在1.6至2.3的版本，改成了并行的。在2.3之后的版本又做了 修改，可以支持并行和串行，当想要串行执行时，直接执行execute()方法，如果需要执行executeOnExecutor(Executor)。

如何取消AsyncTask

## Activity销毁但Task如果没有销毁掉，当Activity重启时这个AsyncTask该如何解决？
比如屏幕旋转这个例子，在重建Activity的时候，会回调  Activity.onRetainNonConfigurationInstance()重新传递一个新的对象给AsyncTask，完成引用的更新。
若Activity已经销毁,此时AsyncTask执行完并返回结果,会报异常么?当一个App旋转时，整个Activity会被销毁和重建。
当Activity重启时，AsyncTask中对该Activity的引用是无效的，因此onPostExecute()就不会起作用
若AsyncTask正在执行，折会报 view not attached to window manager 异常
同样也是生命周期的问题，在 Activity 的onDestroy()方法中调用AsyncTask.cancel方法，让二者的生命周期同步
