#21. 并发
从性能的角度看，如果没有任务会阻塞，那么在单处理器机器上使用并发就没有任何意义  

**协作多线程和抢占多线程(java是抢占式)**  
协作多线程：允许线程自己决定什么时候放弃控制，程序员可以精确地决定某个线程何时被挂机；缺点是可能会有某个线程长时间占用CPU，导致其他线程饥饿  
抢占式线程：操作系统决定何时打断线程，可以在任何时候。通常会在一个时间片后打断线程，防止出现线程饥饿的情况  

###21.2 基本的线程机制
静态方法Thread.yield()告诉线程调度器：“本线程已经执行完生命周期中最重要的部分了，现在正是切换给其它任务执行一段时间的大好时机。”此方法完全是可选的  

当从Runnable导出一个类时，必须具有run()方法，但是它不会具有任何内在的线程能力。要实现线程行为，必须显示地将一个任务附着到线程上  

**Thread类**  
start方法在调用后立即返回，线程启动  

**使用Executor**  
CachedThreadPool, FixedThreadPool(线程池中线程数量固定), SingleThreadExecutor(线程数量固定为1的线程池)  

**从任务中产生返回值**  
可以实现Callable<Type>接口代替Runnable接口,覆盖call()方法，并用ExecutorService.submit()代替ExecutorService.exec()  

**sleep**  
老方式：`Thread.sleep(time)`  
新方式：`TimeUnit.MILLISECONDS.sleep(time)`  

**优先级**  
**让步**：yield()方法  
**后台线程**：当最后一个非后台线程终止时，所有后台线程会突然终止  

**21.2.9 编码的变体**：  

1. 直接继承Thread类，并在类中实现run()方法，再直接调用start()启动
2. 定义一个类实现Runnable接口，但在此类中声明一个Thread引用，并通过new Thread(this)创建对象，再通过对此Thread调用strat()方法启动线程
3. 匿名内部类实现：`t= new Thread(){ public void run(){...} }; t.start();`  
`t= new Thread(new Runnable(){ public void run(){...} }); t.start();`  

线程不是任务：Thread类自身不执行任何操作，它只是驱动赋予它的任务  

**加入一个线程**：如果在某个线程上调用另一个线程t.join()，此线程将被挂起，直到线程t结束；可以在join()中传入超时参数，在给定时间内t未完成则强制返回；可用interrupt()中断  

**线程的异常**：使用一般方法执行Executor中的线程，run()方法内出现异常时，无法在run()方法外捕获到异常；可以使用Thread.UncaughtExceptionHandler进行改进

###21.3 共享受限资源
synchronized, lock(), unlock()  

**原子性与易变性**：除long和double之外的基本类型，在读取和写入时，可以看成是原子操作；但对于long和double，在读取和写入时，是非原子的，可使用volatile

volatile可保证可见性  

**21.3.7 线程本地存储**  
（...............）  

（........）