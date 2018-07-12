### ExecutorService

>
>
>```java
>//阻塞当前主线程直到线程池中所有的任务完成，或者超过了指定的时间，无论哪一个发生，都直接结束
>boolean	awaitTermination(long timeout, TimeUnit unit)
>    
>//执行给定的所有任务并返回和这些任务关联的future
><T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks)
>
>//执行给定的所有任务并返回和这些任务关联的future，或者超过了指定时间，则结束还没执行的任务
><T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
>
>//执行给定的所有任务，并返回任何一个已经完成任务的结果
><T> T	invokeAny(Collection<? extends Callable<T>> tasks)
>//执行给定的所有任务，并返回任何一个已经完成任务的结果,并且是指定的时间到达前执行完成
><T> T	invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
>
>//当前Executor是否被关闭
>boolean	isShutdown()
>
>//当前任务是否全部执行完成
>boolean	isTerminated()
>
>//按照顺序的关闭任务，不去接受新的任务
>void	shutdown()
>
>    //关闭所有的任务，并返回当前在等待的任务
>List<Runnable>	shutdownNow()
>
>//提交任务，并返回对应的future来进行检测任务的执行
><T> Future<T>	submit(Callable<T> task)
>Future<?>	submit(Runnable task)
><T> Future<T>	submit(Runnable task, T result)
>
>```
>
>
>
>
>
>

### AbstractExecutorService

>
>
>```java
>// 构造函数
>AbstractExecutorService()
>
>// API
>    //执行给定任务，并返回持有这些任务的状态和完成后的结果
><T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks)
>
>//执行给定的所有任务并返回和这些任务关联的future，或者超过了指定时间，则结束还没执行的任务
><T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
>
><T> T	invokeAny(Collection<? extends Callable<T>> tasks)
><T> T	invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
>
>    //返回一个给定callable对应的RunnableFuture
>protected <T> RunnableFuture<T>	newTaskFor(Callable<T> callable)
>  
>protected <T> RunnableFuture<T>	newTaskFor(Runnable runnable, T value)
>
><T> Future<T>	submit(Callable<T> task)
>Future<?>	submit(Runnable task)
><T> Future<T>	submit(Runnable task, T result)
>
>
>```
>
>
>
>



### Executors

>
>
>```java
>static Callable<Object>	callable(PrivilegedAction<?> action)
>
>static Callable<Object>	callable(PrivilegedExceptionAction<?> action)
>
>static Callable<Object>	callable(Runnable task)
>
>static <T> Callable<T>	callable(Runnable task, T result)
>
>static ThreadFactory	defaultThreadFactory()
>
>//创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒处于等待任务到来）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池的最大值是Integer的最大值(2^31-1)。
>static ExecutorService	newCachedThreadPool()
>static ExecutorService	newCachedThreadPool(ThreadFactory threadFactory)
>
>//创建固定大小的线程池，每次提交一个任务就创建一个线程，直到线程池达到最大线程限制，线程池的大小一旦达到最大值就会保持不变，在提交新任务，任务将会进入等待队列中等待。如果某个线程因为执行异常结束，那么线程池将会补充一个新线程
>static ExecutorService	newFixedThreadPool(int nThreads)
>static ExecutorService	newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
>
>//创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
>static ScheduledExecutorService	newScheduledThreadPool(int corePoolSize)
>static ScheduledExecutorService	newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
>    
>//创建一个单线程的线程池，这个线程池只有一个县城在工作，也就是相当于单线程串行执行任务。如果这个唯一的线程因为一场结束，那么会有一个新的线程来替代它，次线程池保证所有的任务的执行顺序是按照任务的提交顺序执行
>static ExecutorService	newSingleThreadExecutor()
>static ExecutorService	newSingleThreadExecutor(ThreadFactory threadFactory)
>
>static ScheduledExecutorService	newSingleThreadScheduledExecutor()
>static ScheduledExecutorService	newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
>
>static ExecutorService	newWorkStealingPool()
>
>static ExecutorService	newWorkStealingPool(int parallelism)
>
>static <T> Callable<T>	privilegedCallable(Callable<T> callable)
>
>static <T> Callable<T>	privilegedCallableUsingCurrentClassLoader(Callable<T> callable)
>
>static ThreadFactory	privilegedThreadFactory()
>
>static ExecutorService	unconfigurableExecutorService(ExecutorService executor)
>
>static ScheduledExecutorService	  unconfigurableScheduledExecutorService(ScheduledExecutorService executor)
>```
>
>
>
>

### ThreadPoolExecutor

>
>
>```java
>//构造方法
>
>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
>                   TimeUnit unit, BlockingQueue<Runnable> workQueue)
>
>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
>                   TimeUnit unit, BlockingQueue<Runnable> workQueue, 
>				RejectedExecutionHandler handler)
>
>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
>                   TimeUnit unit, BlockingQueue<Runnable> workQueue, 
>                   ThreadFactory threadFactory)
>
>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
>                   TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory
>                   threadFactory, RejectedExecutionHandler handler)
>
>```
>
>- corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有 任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者 prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建 corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线 程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
>
>- maximumPoolSize：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；
>
>- keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize 时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize。即当线程池中的线程数大于corePoolSize 时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了 allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize 时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；
>
>- unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：
>
>  ```
>  TimeUnit.DAYS;               //天
>  TimeUnit.HOURS;             //小时
>  TimeUnit.MINUTES;           //分钟
>  TimeUnit.SECONDS;           //秒
>  TimeUnit.MILLISECONDS;      //毫秒
>  TimeUnit.MICROSECONDS;      //微妙
>  TimeUnit.NANOSECONDS;       //纳秒
>  ```
>
>  ​
>
>- workQueue：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：
>
>  ```
>  ArrayBlockingQueue; //有界队列
>  LinkedBlockingQueue; //无界队列
>  SynchronousQueue; //特殊的一个队列，只有存在等待取出的线程时才能加入队列，可以说容量为0，是无界队列
>  ```
>
>   ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和Synchronous。线程池的排队策略与BlockingQueue有关。
>
>- threadFactory：线程工厂，主要用来创建线程；
>
>- handler：表示当拒绝处理任务时的策略，有以下四种取值：
>
>
>```
>ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
>ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
>ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
>ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
>```
>
>### API
>
>```Java
>//当给定runnable执行完成后方法执行
>protected void	afterExecute(Runnable r, Throwable t)
>
>    //设置core thread是否被取代，当一断时间没有执行任务
>void	allowCoreThreadTimeOut(boolean value)
>//返回当前线程池是否是超时就取代线程
>boolean	allowsCoreThreadTimeOut()
>//阻塞直到当前所有的任务完成，或者超时或者现在的线程被中断，无论哪一个先发生，都直接退出
>boolean	awaitTermination(long timeout, TimeUnit unit)
>//method被执行先于给定线程上执行的runnable
>protected void	beforeExecute(Thread t, Runnable r)
>//执行给定的任务
>void	execute(Runnable command)
>//Invokes shutdown when this executor is no longer referenced and it has no threads.
>protected void	finalize()
>
> //返回大概的线程数，正在执行任务的线程   
>int	getActiveCount()
>//返回已经完成的任务数量
>long	getCompletedTaskCount()
>//得到core threads数量
>int	getCorePoolSize()
>//得到线程保持存活的时间
>long	getKeepAliveTime(TimeUnit unit)
>//返回同时存在线程池中最大线程的个数
>int	getLargestPoolSize()
>
>int	getMaximumPoolSize()
>
>int	getPoolSize()
>
>BlockingQueue<Runnable>	getQueue()
>
>RejectedExecutionHandler	getRejectedExecutionHandler()
>
>long	getTaskCount()
>ThreadFactory	getThreadFactory()
>//返回当前executor是否被关闭
>boolean	isShutdown()
>//返回当前所有的任务是否被完成
>boolean	isTerminated()
>//Returns true if this executor is in the process of terminating after shutdown() or shutdownNow() but has not completely terminated.
>boolean	isTerminating()
>    //开启所有的core thread去等待任务的到来
>int	prestartAllCoreThreads()
>//开启一个核心线程区等待任务的到来
>boolean	prestartCoreThread()
>//尝试从任务队列中删除所有已经取消的任务
>void	purge()
>//删除指定的任务
>boolean	remove(Runnable task)
>
>void	setCorePoolSize(int corePoolSize)
>
>void	setKeepAliveTime(long time, TimeUnit unit)
>
>void	setMaximumPoolSize(int maximumPoolSize)
>
>void	setRejectedExecutionHandler(RejectedExecutionHandler handler)
>
>void	setThreadFactory(ThreadFactory threadFactory)
>void	shutdown()
>List<Runnable>	shutdownNow()
>protected void	terminated()
>
>```
>
>
>
>
>
>



