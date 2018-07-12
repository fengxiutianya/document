# Thread

>api
>
>```Java
>static class 	 Thread.State  //线程状态
>static interface Thread.UncaughtExceptionHandler
>static int	MAX_PRIORITY   线程可以设置的最大优先级大小
>static int	MIN_PRIORITY   线程可以设置的最小优先级大小
>static int	NORM_PRIORITY  默认分给线程的优先级大小
>    
>Thread()   //默认构造函数
>
>Thread(Runnable target) //使用指定对象构造对象
>
>Thread(Runnable target, String name) //使用指定对象和线程名字构造对象
>
>Thread(String name)  //指定当前线程的名字
>
>Thread(ThreadGroup group, Runnable target)
>    
>Thread(ThreadGroup group, Runnable target, String name)
>    
>Thread(ThreadGroup group, Runnable target, String name, long stackSize)
>    
>Thread(ThreadGroup group, String name)    
>
>    
>static int activeCount()  //返回当前线程所在的ThreadGroup以及子group里面的活跃线程数
>void   checkAccess()   //检查当前cpu运行的线程是否可以修改此线程
>static THread currentThread()  //返回当前正在运行的Thread对象引用
>static void dumpStack()  //打印当前线程的栈记录去标准错误流
>static int enumerate(Thread[] tarray) //拷贝当前线程组中所有的线程到数组
>static Map<Thread,StackTraceElement[]>	getAllStackTraces() //返回当前所有线程的记录
>
>    
>    //得到线程遇到不可处理的错误时对应的异常处理器
>static Thread.UncaughtExceptionHandler	getDefaultUncaughtExceptionHandler()
>    //设置处理突然线程中断的情形
>static void	setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh)
>    
>long	getId()  //返回当前线程的表示
>String  getName() //返回当前线程的名字
>int getPriority()  //返回当前线程的优先级
>StackTraceElement[]	getStackTrace()  //打印本线程栈记录
>Thread.State  getState() //返回当前线程的状态
>ThreadGroup getThreadGroup()  //返回当前线程所属的线程组
>        //得到本线程遇到不可处理的错误时对应的异常处理器
>Thread.UncaughtExceptionHandler	getUncaughtExceptionHandler()
>void	setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh)    
>static boolean	holdsLock(Object obj)  //返回本线程是否持有锁
>void interrupt() //打断当前线程
>static boolean	interrupted() //判断当前线程是否结束
>boolean	isAlive()  //测试当前线程是否存活
>boolean	isDaemon() //判断当前线程是否是一个后台线程
>boolean	isInterrupted() //判断当前线程是否是被打断
>void	join()   //等待知道这个线程死亡
>void	join(long millis)
>void	join(long millis, int nanos)
>void	run()
>void	setContextClassLoader(ClassLoader cl)
>ClassLoader	getContextClassLoader()    //返回当前线程的加载器
>
>void	setDaemon(boolean on)  //设置当前线程为后台线程
>void	setName(String name)
>void	setPriority(int newPriority)
>void	start()    
>String	toString()
>    
>static void	sleep(long millis)
>static void	sleep(long millis, int nanos)
>static void	yield()  //让出当前线程正在使用处理器，进入可运行状态，但有可能当前线程还会获得执行
>
>
>protected Object clone()  //克隆当前对象
>    
>    
>    
>
>```
>
>

