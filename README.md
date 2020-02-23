# InterviewSet
1JVM/GC
2.多线程与高并发
3.Java集合类

1.volatile
	JVM(Java虚拟机)提供的轻量级的同步机制
		特性：
			1.保证可见性
			2.不保证原子性
			3.禁止指令重排
  JMM:Java Memory Model（Java内存模型） 本身是一种抽象的概念并不真实存在，描述的是一组规则或者规范，通过这组规范定义了程序中各个变量的访问方式
	重要特性：
			可见性：（改变是否立即可见，volatile可以保证）  aTO60的实验证明 分别一个线程和主线程 主线程去判断值
			原子性：（不可分割，完整性，当某个线程在执行某个业务的时候，不可以被加塞或者分割，要么同时成功，要么同时失败，volatile不可保证）	20个线程同时执行1000次++操作  主线程去读取 值不相等 出现丢失（写回的过程缺失）
			解决方法：
				1.sync 同步锁
				2.AtomicInteger 推荐 （JUC下的原子类）
			有序性：指令重排的概念
				计算机在执行程序的时候，为了提高性能，编译器和处理器会对指令做重排，一般有编译器的优化重排，指令并行的重排，内存系统的重排，单线程环境中不会影响其结果，在重排的时候必须考虑到指令之间的数据依赖性，多线程环境中由于线程的交替执行，多个线程中使用的变量的一致性无法保证
			单例模式在多线程下出现的问题（不安全）：
				1.sync解决  （重量）
				2.DCL (Double Check Lock) 双端检锁机制  在加锁的前后都 进行判断 （锁的是代码段  不是那么重量级）
					注意：DCL不一定线程安全，因为存在指令重排，可以加入volatile来禁止指令重排
2.CAS
	CAS:比较与交换，Compare-And-Swap 它是一条CPU并发原语，判定内存中某个位置的值是否为预期值，如果是则进行修改，这个过程是原子性的，不会造成所谓的数据不一致的问题。
	底层原理：自旋（CAS思想） + Unsafe
	Unsafe:CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地方法访问，Unsafe相当于是留了一个后门，该类可以直接操作特定内存的数据。其中valueOffset表示的是在内存中的偏移地址，在Unsafe类中，就是根据内存偏移量来获取数据。
	CAS的缺点：
			1.循环时间长、开销大 （底层使用do while进行判断）
			2.只能保证一个共享变量的原子操作 （当前对象this中的变量）
			3.引发出ABA问题（CAS算法实现的一个重要的前提是取出内存中某时刻的数据并在当前时刻比较并替换，在时间差内会导致数据的变化）
3.原子类AtomicInteger的ABA问题？原子更新引用？
	原子引用  + 时间戳 解决ABA问题 AtomicStampedReference
	使用原子引用的过程  还是会出现	ABA
	在使用原子引用 + 时间戳 类似乐观锁 通过当前的版本号进行判断 来解决ABA问题 （todo）

4.集合中不安全的类(多线程环境下导致并发修改异常)
  List --ArrayList	解决方案：
						1.new Vector<>();	效率低 方法都是synchronized的修饰
						2.Collections.synchronizedList()	Collections工具类的转换
						3.CopyOnWriteArrayList	读写分离  写时复制
	Set --HashSet		解决方案：
						1.Collections.synchronizedSet();
						2.CopyOnWriteArraySet
	Map --HashMap		解决方案：
						1.Collections.synchronizedMap()
						2.ConcurrentHashMap
						注意比较 hashmap和concurrentHashMap


5.公平锁、非公平锁、可重入锁、递归锁、自旋锁的理解？ 手写自旋锁
	公平锁：是指多个线程按照申请锁的顺序来获取锁，类似排队取票，先来后到
	非公平锁：是指多个线程获取锁的顺序并不是按照申请的顺序，有可能后申请的线程执行顺序在前，在高并发的情况下，可能造成优先级反转和饥饿现象
	可重入锁（也叫递归锁）：值的是同一线程外层函数获得锁之后，内层递归函数依然可以获取该锁的代码，在同一个线程在外层的方法获取锁的时候，在进入内层方法会自定获取锁，也就是说，线程可以进入任何一个它已经拥有的锁所同步着的代码(重点)。
	注意：ReentrantLock/Synchronized就是一个典型的可重入锁，可重入锁最大的作用就是避免死锁。
	自旋锁：指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程的上下文切换的消耗，循环比较直到成功为止，缺点是会消耗CPU。
	独占锁（写锁）/共享锁（读锁）/互斥锁
		独占锁：该锁一次只能被一个线程锁占有，对于ReentrantLock和synchronized都是独占锁
		共享锁：该锁能够被多个线程占有，对于ReentrantReadWriteLock（重点）其读锁是共享锁，写锁是独占锁
		读写、写读、写写是互斥的
	CountDownLatch 递减到零完成操作  闭锁  可以结合枚举类进行操作 类似于倒计数的实现 
	CyclicBarrier 递增到一定值完成操作   类似于开会 得等人到齐才能进行
	Semaphore 信号量  类似于抢车位 多个线程对于共享资源的抢占使用

6.阻塞队列：首先是一个队列，当阻塞队列为空的时候，从队列中获取元素的操作将会被阻塞；当阻塞队列为满的时候，往队列中添加元素将会被阻塞
	1.阻塞队列有没有好的一面？
		在线程领域，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会重新被唤醒。使用阻塞队列不用我们去关心什么时候去
		阻塞，什么时候需要唤醒，我们不必去控制这些细节，阻塞队列帮我们自动实现了。
		解放程序员的控制工作量。
	2.不得不阻塞，如何进行管理？
		阻塞队列的API
	重要的是3中实现类  ArrayBlockingQueue		LinkedBlockingQueue		SynchronousQueue
		其中SynchronousQueue没有容量，与其他的阻塞队列不同，它不存储元素，每一个put必须要等待一个take操作，否则不能继续添加元素 Demo放数据和取数据的演示

  阻塞队列的作用区域：
		1.生产者与消费者模式
			传统版：
				1.1 synchronized+wait的版本  
				1.2 lock+condition的版本	 加减一个数 多个线程对其进行获取

  			sync与lock的不同：
					1.构成上：sync是关键字，属于JVM层面。Lock是类，api层面。
					2.使用上：sync不需要使用者去维护，会自动进行释放，lock需要手动释放。
					3.等待时不可中断:sync不可中断，lock更为灵活，可以设置锁时间和中断。
					4.lock可以精确控制，可以绑定多条件condition,比sync更为好用
			阻塞队列版：
				package com.ecjtu.juc.blockingqueueCase;
				import java.util.concurrent.ArrayBlockingQueue;
				import java.util.concurrent.BlockingQueue;
				import java.util.concurrent.TimeUnit;
				import java.util.concurrent.atomic.AtomicInteger;

				class ShareData {
				    // 默认开启 进行生产+消费
				    private volatile boolean FLAG = true;
				    private AtomicInteger atomicInteger = new AtomicInteger();
				    BlockingQueue<String> blockingQueue;

				    public ShareData(BlockingQueue<String> blockingQueue) {
				        this.blockingQueue = blockingQueue;
				        System.out.println(blockingQueue.getClass().getName());
				    }

				    // 生产
				    public void Prod() throws Exception {
				        String data;
				        boolean offer;
				        while (FLAG) {
				            data = atomicInteger.incrementAndGet() + "";
				            offer = blockingQueue.offer(data, 2L, TimeUnit.SECONDS);
				            if (offer) {
				                System.out.println(Thread.currentThread().getName() + "\t生产数据" + data);
				            } else {
				                System.out.println(Thread.currentThread().getName() + "\t生产数据异常");
				            }
				            TimeUnit.SECONDS.sleep(2);
				        }
				        System.out.println("我这边停止生产了...");
				    }

				    // 消费
				    public void Cons() throws Exception {
				        String data;
				        while (FLAG) {
				            data = blockingQueue.poll(2L, TimeUnit.SECONDS);
				            if (null == data || data.equalsIgnoreCase("")) {
				                FLAG = false;
				                System.out.println("我这边消费不到 要溜了...");
				                return;
				            }
				            System.out.println(Thread.currentThread().getName() + "\t消费数据" + data);
				        }
				    }

				    public void stop() {
				        this.FLAG = false;
				    }
				}


				/**
				 * volatile\CAS\atomicInteger\BlockingQueue\线程交互\原子引用
				 */
				public class BlockingQueuePro_Demo {

				    public static void main(String[] args) throws InterruptedException {
				        ShareData shareData = new ShareData(new ArrayBlockingQueue<>(10));
				        new Thread(() -> {
				            try {
				                shareData.Prod();
				            } catch (InterruptedException e) {
				                e.printStackTrace();
				            } catch (Exception e) {
				                e.printStackTrace();
				            }
				        }, "A").start();
				        new Thread(() -> {
				            try {
				                shareData.Cons();
				            } catch (InterruptedException e) {
				                e.printStackTrace();
				            } catch (Exception e) {
				                e.printStackTrace();
				            }
				        }, "B").start();
				        TimeUnit.SECONDS.sleep(5);
				        shareData.stop();
				    }
				}

2.线程池
			线程的创建方式：
				1.继承Thread
				2.实现Runnable
				3.实现Callable	带返回值 和FutureTask结合使用
				4.匿名内部类
				5.线程池的创建
			为什么要用线程池？
				1.降低资源消耗
				2.提高响应速度
				3.提高线程的可管理性，利用线程池进行统一的管理和分配.做到线程复用。
			线程池如何使用？
				常用的三种方式：底层为ThreadPoolExecutor
					Executors.newFixedThreadPool(int)	创建定长的线程池，可以控制线程的最大并发数	，超出的会在队列中等待，底层使用的LinkedBlockingQueue
					适用执行长期任务，性能好很多
					Executors.newSingleThreadExecutor()	
					创建一个单线程化的线程池，它只会使用唯一的线程来执行任务，保证任务按照顺序进行执行，底层使用的LinkedBlockingQueue
					适用于一个任务一个任务的执行
					Executors.newCachedThreadPool()
					创建一个可缓存的线程池，如果长度超过处理需求，可以灵活回收空闲线程，保证任务按照顺序进行执行，底层使用的SynchronousQueue
					适用于执行短期异步的程序

线程池重要参数配置？
				7大参数的配置：
					底层源码：
					ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
                    corePoolSize：线程池中常驻核心线程数，初始化数
                    maximumPoolSize：能够容纳同时执行的最大线程数，必须大于等于1
                    keepAliveTime：多余的空闲线程的存活时间，超过这个时间的多余线程将会被销毁
                    unit：keepAliveTime的单位
                    workQueue：任务队列，被提交但是尚未被执行的任务
                    threadFactory：表示生成线程池中工作线程的线程工厂，用于创建线程
                    handler：拒绝策略，表示队列满的时候，工作线程数要大于等于线程池的最大线程数时如何拒绝
			线程池的底层工作原理？
				想象银行办理业务的场景，将7大参数结合起来。
				其中4种拒绝策略的扩展：
					AbortPolicy(默认)：直接抛出RejectedExecutionException异常阻止系统正常运行。
					CallerRunsPolicy:既不抛弃业务，也不会抛出异常。调用者模式，回退。
					DiscardOldestPolicy:抛弃队列中等待最久的任务，然后将当前任务加入队列中尝试再次提交当前任务
					DiscardPolicy:直接丢弃任务，不予任何处理也不抛出任何异常，如果允许任务丢失，那么这是个好的方案
				工程上如何友好的进行参数的配置：
					1.CPU密集型：一直在运行
						一般公式：CPU核数+1个线程的线程池
					2.IO密集型：并不是一直在运行
						一般公式:CPU核数*2
						另外一种工程经验总结：CPU核数/1-阻塞系数（其中阻塞系数在0.8-0.9）
						比如8核CPU： 8/1-0.9=80个线程


		3.消息中间件
7.死锁定位及其分析：
	死锁：两个或者两个以上的进程在执行的过程中，因争夺资源而造成一种互相等待的现象，若无外力的干涉，那么将无法推进下去。
	代码：
	public class DeadLockDemo {
	    public static void main(String[] args) {
	        String lockA = "lockA";
	        String lockB = "lockB";
	        new Thread(new MyLock(lockA,lockB),"AAA").start();
	        new Thread(new MyLock(lockB,lockA),"BBB").start();
	    }
	}

	class MyLock implements Runnable{

	    private String lockA;
	    private String lockB;

	    public MyLock(String lockA, String lockB) {
	        this.lockA = lockA;
	        this.lockB = lockB;
	    }


	    @Override
	    public void run() {
	        synchronized (lockA){
	            try {
	                System.out.println(Thread.currentThread().getName() + "\t持有" + lockA + "还想要" + lockB);
	                TimeUnit.SECONDS.sleep(1);
	                synchronized (lockB) {
	                    System.out.println(Thread.currentThread().getName() + "\t持有" + lockB + "还想要" + lockA);
	                }
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	}

	问题解决：
		1.jps -l 类似于linux中查看java的应用程序
		2.jstack ID 对应的进程号去查看报告

JVM--GC
	JVM运行在操作系统之上，与硬件没有直接交互。
	JVM体系：
	PC寄存器：每个线程都有一个程序计数器，是线程私有的，就是一个指针，指向方法区中的方法字节码。程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器
	方法区：被线程共享，所有方法和字段的字节码。静态变量+常量+类信息+运行时常量池。
	JAVA栈（虚拟机栈）:
	堆：

堆内存调优：
		-Xms:设置初始的分配大小，默认为物理内存的1/64
		-Xmx:最大分配内存，默认为物理内存的1/4
		-XX:+PrintGCDetails:输出详细的GC处理日志
		模拟OOM可以使用while（true）不断的new 字符串，并将初始和最大内存设置小 
	GC:
分代收集算法：次数上频繁收集Young区，次数上较少收集Old区，基本不动Perm区

四大算法：
			1.引用计数法（循环引用的问题，弃用）
			2.复制算法（Copying):年轻代使用的是MinorGC(来也匆匆，去也匆匆)，JVM把年轻代分为了三部分：1个Eden和2个Survivor区(分别叫from和to),默认比例为8:1:1
				优点：无内存碎片
				缺点：占用空间
			3.标记清除(Mark-Sweep):精准打击，一般用于老年代
				优点：不浪费空间
				缺点：产生内存碎片，两次扫描，浪费时间
			4.标记压缩（Mark-Compact）:一般用于老年代
				优点：没有内存碎片
				缺点：需要移动对象的成本，两次扫描，浪费时间

  GCRoots(一组必须活跃的引用):
		什么是垃圾：内存中不再被使用到的空间就是垃圾
		1.引用计数法：存在循环引用的问题
		2.可达性分析：通过一系列名为“GCRoots”的对象作为起始点，从GCRoot对象进行链路扫描，判断是否可达
		GcRoot对象是如何确定的？
		GCRooot对象四个区域：
			虚拟机栈中引用的对象
			方法区中类静态属性引用的对象 
			方法区中常量引用的对象
			本地方法栈中JNI（Native方法）引用的对象

   JVM的参数类型：
    	标配参数：-version\-help\showversion
    	X参数：-Xint(解释执行)、-Xcomp(第一次使用编译成本地代码)、-Xmixed(混合模式)
    	XX参数（重点）：
    		Boolean类型：公式==>-XX:+或者-某个属性，+表示开启，-表示关闭 
    		举个例子：
    			是否打印GC的收集细节-->-XX:+PrintGCDetails
    		KV设置类型：公式==>-XX:属性key=属性值value
    		举个例子：
    			-XX:MetaspaceSize=128m
    	JVM中的默认值的查看方式：	
    	1.	jps
    		jinfo -flag 具体参数 java进程编号
    		jinfo -flags java进程编号
    	2.	java -XX:+PrintFlagsInitial
    		盘点JVM初始家底
    	3.  java -XX:+PrintFlagsFinal
    		主要查看修改更新之后的内容 可以在运行的时候进行修改
    		hava -xx:+PrintFlagsFinal -XX:MetaspaceSize=512m
    	4.  java -XX:+PrintCommandLineFlags
    		方便查询当前所使用的垃圾回收算法

   常用参数(注意-Xms和-Xmx属于XX型参数)：
    		-Xms:初始大小内存 默认为物理内存的1/64，等价于-XX：InitalHeapSize
    		-Xmx:最大分配内存，默认为物理内存的1/4,
    		等价于-XX：MaxHeapSize
    		-Xss:单个线程栈的大小,一般默认为512k~1024k,等价于-XX：ThreadStackSize
    		-Xmn:设置年轻代的大小

   垃圾回收分析举例：
    		[GC (Allocation Failure) [PSYoungGen: 1972K->480K(2560K)] 1972K->760K(9728K), 0.0013425 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
			[GC (Allocation Failure) [PSYoungGen: 480K->480K(2560K)] 760K->768K(9728K), 0.0009561 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
			[Full GC (Allocation Failure) [PSYoungGen: 480K->0K(2560K)] [ParOldGen: 288K->723K(7168K)] 768K->723K(9728K), [Metaspace: 3351K->3351K(1056768K)], 0.0086582 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

	    【GC的类型：youngGC前新生代的内存占用->youngGC后新生代的内存占用（新生代总共大小）】youngGC前JVM堆内存占用->youngGC后JVM堆内存占用，youngGC耗时】【Times:youngGC用户耗时，youngGC系统耗时，youngGC实际耗时】
    	栈管运行，堆管存储。
    		规律：名称：GC前内存占用->GC后内存占用（该内存总大小）

    	-XX:SurvivorRatio:设置新生代中eden和s0/s1空间的比例 默认为eden:s0:s1= 8:1:1
    	-XX:NewRatio:设置年轻代与老年代在堆内存的占比，默认为1：2
    	-XX:MaxTenuringThreshold：设置垃圾的最大年龄

   强/软/弱/虚/的引用(Reference)：
    	强：最常见的普通引用，只要有强引用指向对象，垃圾回收器就不会碰这种对象，这也是造成Java内存泄漏的主要原因之一。
    	软：SoftReference当系统内存充足的时候它不会被回收，当系统内存不足的时候，它会被回收。（可以手动设置JVM参数，模拟情况）
    	弱：不管内存是否充足，只要发生GC，都会被回收
    	虚：形同虚设，任何时候都可能被垃圾回收，虚引用必须和引用队列联合使用。主要作用是跟踪对象被垃圾回收的状态，提供了一种确保对象被finalize之后，做某些事的机制。
    软引用和弱引用的适用场景：
    	大量读取比较影响性能，又或者一次性需要把内容加载到内存中，这个时候可以使用软引用解决问题

	

  OOM的认识：
		StackOverFlowError:栈方法递归调用
		OutOfMemoryError:Java heap space 堆内存空间不足
		GC OverHead limit exceeded(未成功模拟)
		Direct buffer memeroy
		unable to create new native thread
		Metaspace    	
    垃圾收集器的回收种类：
    	1.Serial	串行
    		它为单线程环境设置并且只使用一个线程进行垃圾回收，会暂停所有用户线程，所以不适合服务器环境。
    	2.Parallel	并行
    		多个线程进行垃圾回收，提高效率。
    	3.CMS       并发
    	4.G1		

   查看垃圾回收器的命令：java -XX:+PrintCommandLineFlags -version

   GC之七大垃圾收集器：
    	Young Gen:Serial Copying  	Parallel Scavenge 		parNew
    	Old Gen:Serial MSC 			Parallel Compacting 	CMS
    	G1

   新生代: 
    			串行GC(Serial)/(Serial Copying)、
    			并行GC(ParNew)、
    			并行回收GC(Parallel)/(Parallel Scavenge)
   老年代:  
    			串行GC(Serial Old)/(Serial MSC)、
    			并行GC(Parallel Old)/(Parallel MSC)、
    			并发标记清除GC(CMS)



Java8新特性：
	Lambda 表达式：
	  可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
		可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
		可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
		可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。
  Java 8 默认方法：
		Java 8 新增了接口的默认方法。
		简单说，默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法。
		我们只需在方法名前面加个 default 关键字即可实现默认方法。
  Java 8 Stream：
		Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。
		Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。
		Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。
		这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。
		元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

  Java 8 Optional 类:
		Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。
  	Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。
		Optional 类的引入很好的解决空指针异常。

RestFull API:
	四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。


Linux用于生产环境的常用命令:
	整机:
		top、uptime
		主要查看CPU和内存使用率以及右上角的负载均衡
	CPU:
		vmstat -n 秒数 采样数
		mpstat -P ALL 秒数:查看所有cpu核信息
		pidstat -u 1 -p:每个进程使用cpu的用量分解信息	
	内存：
		free
	硬盘：
		df -h
	磁盘IO：
		iostat
	网络IO：
		ifstat
