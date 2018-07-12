# LockSupport

### 概述

>本章介绍JUC(java.util.concurrent)包中的LockSupport。内容包括：
>
>1. LockSupport 概述
>2. LockSupport源码分析
>3. LockSupport示例

### LockSupport概述

>LockSupport 和 CAS 是Java并发包中很多并发工具控制机制的基础，它们底层其实都是依赖Unsafe实现。
>
>LockSupport是用来创建锁和其他同步类的基本**线程阻塞**原语。
>
>LockSupport 提供park()和unpark()方法实现阻塞线程和解除线程阻塞，LockSupport和每个使用它的线程都与一个许可(permit)关联。permit相当于1，0的开关，默认是0，调用一次unpark就加1变成1，调用一次park会消费permit, 也就是将1变成0，同时park立即返回。再次调用park会变成block（因为permit为0了，会阻塞在这里，直到permit变为1）, 这时调用unpark会把permit置为1。每个线程都有一个相关的permit, permit最多只有一个，**重复调用unpark也不会积累**。
>
>park()和unpark()不会有 “Thread.suspend和Thread.resume所可能引发的死锁” 问题，由于许可的存在，调用 park 的线程和另一个试图将其 unpark 的线程之间的竞争将保持活性。
>
>如果调用线程被中断，则park方法会返回。同时park也拥有可以设置超时时间的版本。
>
>需要特别注意的一点：**park 方法还可以在其他任何时间“毫无理由”地返回，因此通常必须在重新检查返回条件的循环里调用此方法**。从这个意义上说，park 是“忙碌等待”的一种优化，它不会浪费这么多的时间进行自旋，但是必须将它与 unpark 配对使用才更高效。
>
>#### LockSupport 函数列表
>
>```java
>
>// 获得当前线程阻塞在哪个对象之上被阻塞,
>// 如果该调用不受阻塞，则返回 null。
>static Object getBlocker(Thread t)
>
>// 为了线程调度，禁用当前线程，除非许可可用。
>static void park()
>
>// 为了线程调度，在许可可用之前禁用当前线程。
>static void park(Object blocker)
>
>// 为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。
>static void parkNanos(long nanos)
>
>// 为了线程调度，在许可可用前禁用当前线程，并最多等待指定的等待时间。
>static void parkNanos(Object blocker, long nanos)
>
>// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
>static void parkUntil(long deadline)
>
>// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
>static void parkUntil(Object blocker, long deadline)
>
>// 如果给定线程的许可尚不可用，则使其可用。
>static void unpark(Thread thread)
>```
>
>

### 2.LockSupport源码分析

>```java
>package java.util.concurrent.locks;
>
>import sun.misc.Unsafe;
>
>//阻塞当前线程，和suspend的区别
>//1. resume 在suspend之前调用没有效果而unpark在park调用之前是有效果的
>public class LockSupport {
>    //禁止初始化，当前类的所有方法都是静态方法，可以直接调用
>    private LockSupport() {
>    }
>
>    //设置当前线程是在哪个对象上阻塞的
>    private static void setBlocker(Thread t, Object arg) {
>        // Even though volatile, hotspot doesn't need a write barrier here.
>        UNSAFE.putObject(t, parkBlockerOffset, arg);
>    }
>
>    /**
>     * 唤醒thread线程
>     * 会出现以下情况：
>     * 1. 在thread线程阻塞之后调用了unpark操作，会唤醒park的线程。这是正常操作
>     * 2. 在thread线程阻塞之前调用了unpark操作，当你在调用park的时候，不会产生阻塞
>     *    注意的一点是：无论thread线程阻塞之前调用过多少次unpark，调用一次park
>     *    操作就会将之前所有的unpark操作给清零
>     *    代码示例：下面这样写是会发生阻塞的
>     *      LockSupport.unpark(Thread.currentThread());
>     *      LockSupport.unpark(Thread.currentThread());
>     *      LockSupport.park();
>     *      LockSupport.park();
>     */
>    public static void unpark(Thread thread) {
>        if (thread != null)
>            UNSAFE.unpark(thread);
>    }
>
>    /**
>     * 阻塞当前线程
>     * blocker是给定当前线程是在哪个对象上阻塞的
>     */
>    public static void park(Object blocker) {
>        Thread t = Thread.currentThread();
>        setBlocker(t, blocker);
>        UNSAFE.park(false, 0L);
>        setBlocker(t, null);
>    }
>
>    /**
>     * 阻塞当前线程nanoseconds，或者中间被唤醒
>     * blocker 是当前线程阻塞在哪个对象上
>     * nanos   是阻塞多少nanoseconds
>     */
>    public static void parkNanos(Object blocker, long nanos) {
>        if (nanos > 0) {
>            Thread t = Thread.currentThread();
>            setBlocker(t, blocker);
>            UNSAFE.park(false, nanos);
>            setBlocker(t, null);
>        }
>    }
>
>    /**
>     * 阻塞当前线程到deadline结束，或者在时间未到之前被唤醒
>     * blocker   是当前线程阻塞在哪个对象上
>     * deadline  是阻塞截止的时间
>     */
>    public static void parkUntil(Object blocker, long deadline) {
>        Thread t = Thread.currentThread();
>        setBlocker(t, blocker);
>        UNSAFE.park(true, deadline);
>        setBlocker(t, null);
>    }
>
>    //获得当前线程阻塞在哪个对象之上被阻塞
>    public static Object getBlocker(Thread t) {
>        if (t == null)
>            throw new NullPointerException();
>        return UNSAFE.getObjectVolatile(t, parkBlockerOffset);
>    }
>
>    //阻塞当前线程直到被唤醒，
>    public static void park() {
>        UNSAFE.park(false, 0L);
>    }
>
>    /**
>     * 阻塞当前线程nanoseconds，或者中间被唤醒
>     * nanos   是阻塞多少nanoseconds
>     */
>    public static void parkNanos(long nanos) {
>        if (nanos > 0)
>            UNSAFE.park(false, nanos);
>    }
>
>    /**
>     * 阻塞当前线程到deadline结束，或者在时间未到之前被唤醒
>         * deadline  是阻塞截止的时间
>     */
>    public static void parkUntil(long deadline) {
>        UNSAFE.park(true, deadline);
>    }
>
>    /**
>     * Returns the pseudo-randomly initialized or updated secondary seed.
>     * Copied from ThreadLocalRandom due to package access restrictions.
>     */
>    static final int nextSecondarySeed() {
>        int r;
>        Thread t = Thread.currentThread();
>        if ((r = UNSAFE.getInt(t, SECONDARY)) != 0) {
>            r ^= r << 13; // xorshift
>            r ^= r >>> 17;
>            r ^= r << 5;
>        } else if ((r = java.util.concurrent.ThreadLocalRandom.current().nextInt()) == 0)
>            r = 1; // avoid zero
>        UNSAFE.putInt(t, SECONDARY, r);
>        return r;
>    }
>
>    // Hotspot implementation via intrinsics API
>    private static final sun.misc.Unsafe UNSAFE;
>    private static final long parkBlockerOffset;
>    private static final long SEED;
>    private static final long PROBE;
>    private static final long SECONDARY;
>    static {
>        try {
>            UNSAFE = sun.misc.Unsafe.getUnsafe();
>            Class<?> tk = Thread.class;
>            parkBlockerOffset = UNSAFE.objectFieldOffset(tk.getDeclaredField("parkBlocker"));
>            SEED = UNSAFE.objectFieldOffset(tk.getDeclaredField("threadLocalRandomSeed"));
>            PROBE = UNSAFE.objectFieldOffset(tk.getDeclaredField("threadLocalRandomProbe"));
>            SECONDARY = UNSAFE.objectFieldOffset(tk.getDeclaredField("threadLocalRandomSecondarySeed"));
>        } catch (Exception ex) {
>            throw new Error(ex);
>        }
>    }
>
>}
>
>```
>
>

### 3. LockSupport示例

>示例1
>
>```
>public class WaitTest1 {
>
>    public static void main(String[] args) {
>
>        ThreadA ta = new ThreadA("ta");
>
>        synchronized(ta) { // 通过synchronized(ta)获取“对象ta的同步锁”
>            try {
>                System.out.println(Thread.currentThread().getName()+" start ta");
>                ta.start();
>
>                System.out.println(Thread.currentThread().getName()+" block");
>                // 主线程等待
>                ta.wait();
>
>                System.out.println(Thread.currentThread().getName()+" continue");
>            } catch (InterruptedException e) {
>                e.printStackTrace();
>            }
>        }
>    }
>
>    static class ThreadA extends Thread{
>
>        public ThreadA(String name) {
>            super(name);
>        }
>
>        public void run() {
>            synchronized (this) { // 通过synchronized(this)获取“当前对象的同步锁”
>                System.out.println(Thread.currentThread().getName()+" wakup others");
>                notify();    // 唤醒“当前对象上的等待线程”
>            }
>        }
>    }
>}
>```
>
>示例2
>
>```
>import java.util.concurrent.locks.LockSupport;
>
>public class LockSupportTest1 {
>
>    private static Thread mainThread;
>
>    public static void main(String[] args) {
>
>        ThreadA ta = new ThreadA("ta");
>        // 获取主线程
>        mainThread = Thread.currentThread();
>
>        System.out.println(Thread.currentThread().getName()+" start ta");
>        ta.start();
>
>        System.out.println(Thread.currentThread().getName()+" block");
>        // 主线程阻塞
>        LockSupport.park(mainThread);
>
>        System.out.println(Thread.currentThread().getName()+" continue");
>    }
>
>    static class ThreadA extends Thread{
>
>        public ThreadA(String name) {
>            super(name);
>        }
>
>        public void run() {
>            System.out.println(Thread.currentThread().getName()+" wakup others");
>            // 唤醒“主线程”
>            LockSupport.unpark(mainThread);
>        }
>    }
>}
>```
>
>#### **说明**：park和wait的区别。wait让线程阻塞前，必须通过synchronized获取同步锁。



