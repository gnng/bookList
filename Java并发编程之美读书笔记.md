# 第一章 线程基础知识

## 1. 什么是线程？

​	线程是进程的执行单元。

## 2. 什么是进程？

​	进程是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，例：一个简单的项目相当于一个进程。

## 3. 线程和进程的区别和联系？

​	线程是进程的执行路径，一个进程至少有一个线程，进程中多个线程共享进程资源。操作系统是把资源分配给进程的，但是CPU资源比较特殊是被分配到线程，因为真正占用CPU使用的是线程，也可以说线程是CPU分配的基本单位。

## 4. 线程的资源分配

​	一个进程中有多个线程，多个线程共享进程的堆和方法区资源，但是每个线程有自己的程序计数器和栈资源。

​	程序计数器是一块内存区域，用来记录当前线程要执行的命令地址，CPU是通过时间片轮转方式让线程轮询占用。程序计数器就是记录线程上次轮询执行地址，以便再次轮询到可以继续执行，所以要设置为线程私有化，如果执行的是native方法，程序计数器记录的是undefined地址。

​	每个线程有自己的栈资源，用来存储局部变量，这些局部变量为线程私有，别的线程访问不了。

## 5. 线程的创建。

​	线程的创建有三种方式。

	1. 继承Thread类（Thread类也实现了Runnable接口）：
		public class MyThread extend Thread{
	        @override
	        public void run(){
	            system.out.println("用extend创建线程！");
	        }
		}
		//创建线程
		MyThread myThread = new MyThread();
		//启动线程
		myThread.start();
		调用start()方法后线程没有立即执行，而是进入就绪状态，等待CPU的调度。
	2.实现Runnable接口：
		实现Runnable()接口，重写run()方法。
		new Thread(task).start();
	3. 实现Callable接口（可以得到返回值get()方法）：
	    public class CallerTask implements Callable {
	        @Override
	        public Object call() throws Exception {
	            System.out.println("Callable创建线程并返回执行结果");
	            return "Callable";
	        }
	        public static void main(String[] args) {
	            //创建异步任务
	            FutureTask<String> futureTask = new FutureTask<>(new CallerTask());
	            //启动线程
	            new Thread(futureTask).start();
	            try {
	                //等待任务异步完成并返回 结构
	                System.out.println(futureTask.get());
	            } catch (Exception e) {
	                e.printStackTrace();
	            }
	        }
	    }

​	小结：使用继承方法方便传参，可以在子类里面添加成员变量，通过set方式设置参数或者通过构造函数进行传递；使用Runnable方式，只能使用主线程中用final声明的变量，前两者都不能回去返回参数。

# 第二章 并发编程基础知识

## 1. 线程安全和共享资源的概念

​	共享资源就是说该资源被多个线程所持有或者多个线程都可以访问该资源。线程安全问题是指当多个线程同时读写共享资源并且没有任何同步措施，导致出现脏数据或者其他不可预料结果的问题。

## 2. synchronize关键字

​	synchronize同步关键字，可以修饰方法和同步代码块（修饰代码块可以控制锁的粒度）。synchronize块是Java提供的一种原子性内置锁，java的每个对象都可以把它当做一个同步锁来使用，这些java内置的使用者看不到的锁被称为内部锁，也叫监视器锁。

​	线程的代码块在进入synchronize块前自动获取内部锁，这个时候其他线程访问该同步代码块，该线程会被阻塞挂起，拿到内部锁的线程会在正常退出同步代码块或者抛出异常后或者在同步代码块内部调用了内置锁资源的wait系列方法时会释放该内置锁。内置锁时排它锁，也就是当一个线程获得了这个锁后，其余线程只有等该锁释放才能获取该锁。synchronize的使用会导致上下文的切换，非常耗时。

​	synchronize保证了内存可见性，和操作的原子性。

## 3. volatile关键字

​	volatile确保对一个变量的更新对其他线程马上可见，当一个变量被声明为volatile时，线程在写入该变量时不会把值缓存到寄存器或者其他地方，而是把值刷新回主内存，当其他线程读取该共享变量，会直接从主内存重新获取最新值，而不是当前线程工作内存中的值。

​	volatile保证了内存可见性，避免重排，不保证操作的原子性。

## 4. 原子性操作

​	所谓的原子性操作，指的是一系列操作时，要么全部执行，要么全部不执行。

## 5. CAS操作

​	CAS即Compare And Swap，其是JDK提供的非阻塞原子性操作，它通过硬件保证  比较--更新  操作的 原子性。JDK 里面的 Unsafe 类提供了一系列的 compareAndSwap＊方法 ，boolean compareAndSwapLong(Object obj,long valueOffset,long expect, long update）;等等。    CAS操作会有ABA问题，即变量的状态值出现环状转换，如果变量只能朝一个方向转换就不会出现ABA问题。

## 6. 锁的概述

- 悲观锁：

  ​	悲观锁指对数据被外界修改保持保守状态，认为数据很容易被其他线程修改，所以处理数据前先对数据加锁，整个处理过程中，数据处于锁定状态。

- 乐观锁：

  ​	乐观锁是相对悲观锁来说的， 它认为一般情况下数据是不会被修改的，所以在访问前不会加锁，而进行数据更新时才会对数据是否冲突进行检测。

- 公平锁：

  ​	公平锁表示线程获取锁的顺序是按照线程请求顺序确定的。

  ​	ReentrantLock pairLock = new ReentrantLock(true);    

- 非公平锁：

  ​	线程获取锁的顺序通过抢占确定。

  ​	ReentrantLock pairLock = new ReentrantLock(false);  

- 独占锁：

  ​	独占锁保证任何时候都只有一个线程得到锁，是一种悲观锁。

- 共享锁：

  ​	共享锁是乐观锁，放宽了加锁的条件，允许多个线程同时进行读操作。

- 可重入锁：

  ​	当一个线程获取别其他线程持有的独占锁时，该线程会被阻塞，那么当线程再次获取它之前获取的锁时会不会被阻塞？如果不被阻塞，说明锁是可重入的。synchronized 内部锁是可重入锁 ，可重入锁原理是在锁的内部维护了一个线程标示，用来标示该锁目前是被那个线程占用，然后关联一个计数器。   

- 自旋锁：

  ​	当一个线程获取独占锁失败后，会被切换到内核状态而被挂起，当线程获取到锁后又需要将切换内核状态而唤醒该线程，这样内存开销较大，影响并发性能。

  ​	自旋锁则是，当线程在获取锁时，如果该锁已经被其他线程占有，它不会别立即阻塞，在不放弃CPU使用权的情况下，多次尝试获取该锁（默认10次，可以使用－XX :PreBlockSpinsh参数设置），很有可能在后续几次尝试中其他线程释放了该锁。如果尝试的指定次数内没有释放锁则当前线程才会被挂起。

  ​	自旋锁是使用CPU时间来换取线程阻塞与调度的开销，但是可能这些CPU时间会白白浪费。

	​	