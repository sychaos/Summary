# 多线程相关

## Thread、AsyncTask、IntentService的使用场景与特点。
* Thread线程，独立运行与于 Activity 的，当 Activity 被 finish 后，如果没有主动停止 Thread 或者 run 方法没有执行完，其会一直执行下去。
* AsyncTask 封装了两个线程池和一个 Handler，（SerialExecutor 用于排队，THREAD_POOL_EXECUTOR 为真正的执行任务，Handler 将工作线程切换到主线程），其必须在 UI 线程中创建，execute 方法必须在 UI 线程中执行，一个任务实例只允许执行一次，执行多次将抛出异常，用于网络请求或者简单数据处理。
* IntentService：处理异步请求，实现多线程，在 onHandleIntent 中处理耗时操作，多个耗时任务会依次执行，执行完毕后自动结束。

AsyncTask的cancel：AsyncTask不会不考虑结果而直接结束一个线程。调用cancel()其实是给AsyncTask设置一个"canceled"状态。这取决于你去检查AsyncTask是否已经取消，之后决定是否终止你的操作。
对于mayInterruptIfRunning——它所作的只是向运行中的线程发出interrupt()调用。在这种情况下，你的线程是不可中断的，也就不会终止该线程。

## 静态内部类实现单例的原理
    因为内部静态类是要在有引用了以后才会装载到内存的。所以在你第一次调用getInstance()之前，SingletonHolder是没有被装载进来的，
    只有在你第一次调用了getInstance()之后，里面涉及到了return return SingletonHolder.instance; 产生了对SingletonHolder的引用，内部静态类的实例才会真正装载。这也就是懒加载的意思

## 单例模式的双层锁原理
    ```java
    public class SingleTon {

        private static SingleTon singleTon = null;


        private SingleTon() {
        }

        public static SingleTon getInstance(){
            if (singleTon == null) {
                synchronized (SingleTon.class) {
                    if (singleTon == null) {
                        singleTon = new SingleTon();
                    }
                }
            }
            return singleTon;
        }

    }
    ```

    为何要使用双重检查锁定呢？上文已经大概说了一下。
    考虑这样一种情况，就是有两个线程同时到达，即同时调用 getInstance() 方法，
    此时由于 singleTon == null ，所以很明显，两个线程都可以通过第一重的 singleTon == null ，
    进入第一重 if 语句后，由于存在锁机制，所以会有一个线程进入 lock 语句并进入第二重 singleTon == null ，
    而另外的一个线程则会在 lock 语句的外面等待。
    而当第一个线程执行完 new  SingleTon（）语句后，便会退出锁定区域，此时，第二个线程便可以进入 lock 语句块，
    此时，如果没有第二重 singleTon == null 的话，那么第二个线程还是可以调用 new  SingleTon （）语句，
    这样第二个线程也会创建一个 SingleTon实例，这样也还是违背了单例模式的初衷的，
    所以这里必须要使用双重检查锁定。
    细心的朋友一定会发现，如果我去掉第一重 singleton == null ，程序还是可以在多线程下完好的运行的，
    考虑在没有第一重 singleton == null 的情况下，
    当有两个线程同时到达，此时，由于 lock 机制的存在，第一个线程会进入 lock 语句块，并且可以顺利执行 new SingleTon（），
    当第一个线程退出 lock 语句块时， singleTon 这个静态变量已不为 null 了，所以当第二个线程进入 lock 时，
    还是会被第二重 singleton == null 挡在外面，而无法执行 new Singleton（），
    所以在没有第一重 singleton == null 的情况下，也是可以实现单例模式的？那么为什么需要第一重 singleton == null 呢？
    这里就涉及一个性能问题了，因为对于单例模式的话，new SingleTon（）只需要执行一次就 OK 了，
    而如果没有第一重 singleTon == null 的话，每一次有线程进入 getInstance（）时，均会执行锁定操作来实现线程同步，
    这是非常耗费性能的，而如果我加上第一重 singleTon == null 的话，
    那么就只有在第一次，也就是 singleTton ==null 成立时的情况下执行一次锁定以实现线程同步，
    而以后的话，便只要直接返回 Singleton 实例就 OK 了而根本无需再进入 lock 语句块了，这样就可以解决由线程同步带来的性能问题了。

## 死锁
    ### 写一个死锁
    ```java
    /**
    * 一个简单的死锁类
    * 当DeadLock类的对象flag==1时（td1），先锁定o1,睡眠500毫秒
    * 而td1在睡眠的时候另一个flag==0的对象（td2）线程启动，先锁定o2,睡眠500毫秒
    * td1睡眠结束后需要锁定o2才能继续执行，而此时o2已被td2锁定；
    * td2睡眠结束后需要锁定o1才能继续执行，而此时o1已被td1锁定；
    * td1、td2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。
    */
    public class DeadLock implements Runnable {
        public int flag = 1;
        //静态对象是类的所有对象共享的
        private static Object o1 = new Object(), o2 = new Object();
        @Override
        public void run() {
            System.out.println("flag=" + flag);
            if (flag == 1) {
                synchronized (o1) {
                    try {
                        Thread.sleep(500);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    synchronized (o2) {
                        System.out.println("1");
                    }
                }
            }
            if (flag == 0) {
                synchronized (o2) {
                    try {
                        Thread.sleep(500);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    synchronized (o1) {
                        System.out.println("0");
                    }
                }
            }
        }

        public static void main(String[] args) {

            DeadLock td1 = new DeadLock();
            DeadLock td2 = new DeadLock();
            td1.flag = 1;
            td2.flag = 0;
            //td1,td2都处于可执行状态，但JVM线程调度先执行哪个线程是不确定的。
            //td2的run()可能在td1的run()之前运行
            new Thread(td1).start();
            new Thread(td2).start();

        }
    }
    ```
    ### 什么是死锁
    死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，
    若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。
    ### 死锁产生的原因
    * 因竞争资源发生死锁 现象：系统中供多个进程共享的资源的数目不足以满足全部进程的需要时，就会引起对诸资源的竞争而发生死锁现象

    （1）可剥夺资源和不可剥夺资源：可剥夺资源是指某进程在获得该类资源时，该资源同样可以被其他进程或系统剥夺，不可剥夺资源是指当系统把该类资源分配给某个进程时，不能强制收回，只能在该进程使用完成后自动释放

    （2）竞争不可剥夺资源：系统中不可剥夺资源的数目不足以满足诸进程运行的要求，则发生在运行进程中，不同的进程因争夺这些资源陷入僵局。

    举例说明：  资源A,B； 进程C,D

    资源A,B都是不可剥夺资源：一个进程申请了之后，不能强制收回，只能进程结束之后自动释放。内存就是可剥夺资源

    进程C申请了资源A，进程D申请了资源B。

    接下来C的操作用到资源B，D的资源用到资源A。但是C,D都得不到接下来的资源，那么就引发了死锁。

    （3）竞争临时资源

    * 进程推进顺序不当发生死锁
    ### 死锁的四个必要条件
    （1）互斥条件：进程对所分配到的资源不允许其他进程进行访问，若其他进程访问该资源，只能等待，直至占有该资源的进程使用完成后释放该资源

    （2）请求和保持条件：进程获得一定的资源之后，又对其他资源发出请求，但是该资源可能被其他进程占有，此事请求阻塞，但又对自己获得的资源保持不放

    （3）不可剥夺条件：是指进程已获得的资源，在未完成使用之前，不可被剥夺，只能在使用完后自己释放

    （4）环路等待条件：是指进程发生死锁后，必然存在一个进程--资源之间的环形链
    ### 怎样防止死锁 (小例子 todo)
    1.预防死锁：通过设置一些限制条件，去破坏产生死锁的必要条件
    2.避免死锁：在资源分配过程中，使用某种方法避免系统进入不安全的状态，从而避免发生死锁
    3.检测死锁：允许死锁的发生，但是通过系统的检测之后，采取一些措施，将死锁清除掉
    4.解除死锁：该方法与检测死锁配合使用

## 为什么不能在子线程更新UI
    如果子线程都能更新UI的话，问题还很多的。。

## 如何保证多线程读写文件的安全？TODO 我曹好高端啊
    多线程下载大概思路就是通过Range 属性实现文件分段
    RandomAccessFile是否可以实现写文件不加锁

线程同步，CopyOnWriteArrayList怎样实现。TODO
volatile原理
synchronized与Lock的区别
锁机制
wait/notify
手写生产者/消费者模式
线程如何关闭，以及如何防止线程的内存泄漏
ThreadLocal原理，实现及如何保证Local属性
长连接是什么？为什么用长连接？心跳时间是如何确定的？

线程池的源码；当达到核心线程数后，新加的任务是怎样操作的？RejectHandler是在哪个线程中处理的？可重入锁的源码实现，可重入锁是如何保证可见性和原子性的；volatile是如何保证可见性的。