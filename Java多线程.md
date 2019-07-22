### Java多线程

#### 1.进程与线程

##### 进程

```
进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。
```

##### 线程

```
线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。
```

##### 临界区

```
临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。在并行程序中，临界区资源是保护的对象。
```

#### 2.线程的运行模式

```
 a:分时调度
      所有线程轮流使用 CPU 的使用权，平均分配每个线程占用 CPU 的时间。
```

```
b:抢占式调度
 优先让优先级高的线程使用 CPU，如果线程的优先级相同，那么会随机选择一个(线程随机性)，Java使用的为抢占式调度。

 大部分操作系统都支持多进程并发运行，现在的操作系统几乎都支持同时运行多个程序。比如：现在我们上课一边使用编辑器，一边使用录屏软件，同时还开着画图板，dos窗口等软件。此时，这些程序是在同时运行，”感觉这些软件好像在同一时刻运行着“。

 实际上，CPU(中央处理器)使用抢占式调度模式在多个线程间进行着高速的切换。对于CPU的一个核而言，某个时刻，只能执行一个线程，而 CPU的在多个线程间切换速度相对我们的感觉要快，看上去就是在同一时刻运行。
 其实，多线程程序并不能提高程序的运行速度，但能够提高程序运行效率，让CPU的使用率更高。
```

#### 3.多线程

```
多线程就是多个线程同时运行或交替运行。单核CPU的话是顺序执行，也就是交替运行。多核CPU的话，因为每个CPU有自己的运算器，所以在多个CPU中可以同时运行。
```

开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

线程就是轻量级进程，是程序执行的最小单位。使用多线程而不是用多进程去进行并发程序的设计，是因为线程间的切换和调度的成本远远小于进程。

#### 4.使用多线程常见的三种方式

前两种实际上很少使用，一般都是用线程池的方式比较多一点。

##### 1.继承Thread类

```
public class MyThread extends Thread {
	@Override
	public void run() {
		super.run();
		System.out.println("MyThread");
	}
}

public class Run {
	public static void main(String[] args) {
		MyThread mythread = new MyThread();
		mythread.start();
		System.out.println("运行结束");
	}
}

运行结束
MyThread
从上面的运行结果可以看出：线程是一个子任务，CPU以不确定的方式，或者说是以随机的时间来调用线程中的run方法。
```

##### 2.实现Runnable接口

推荐实现Runnable接口方式开发多线程，因为Java单继承但是可以实现多个接口。

```
public class MyRunnable implements Runnable {
	@Override
	public void run() {
		System.out.println("MyRunnable");
	}
}

public class Run {
	public static void main(String[] args) {
		Runnable runnable=new MyRunnable();
		Thread thread=new Thread(runnable);
		thread.start();
		System.out.println("运行结束！");
	}
}

运行结束
MyThread
```

##### 3.使用线程池

```
 1.在java中，如果每个请求到达就创建一个新线程，开销是相当大的。
 2.在实际使用中，创建和销毁线程花费的时间和消耗的系统资源都相当大，甚至可能要比在处理实际的用户请求的时间和资源要多的多。
 3.除了创建和销毁线程的开销之外，活动的线程也需要消耗系统资源。
	如果在一个jvm里创建太多的线程，可能会使系统由于过度消耗内存或“切换过度”而导致系统资源不足。
    为了防止资源不足，需要采取一些办法来限制任何给定时刻处理的请求数目，尽可能减少创建和销毁线程的次数，特别是一些资源耗费比较大的线程的创建和销毁，尽量利用已有对象来进行服务。
     线程池主要用来解决线程生命周期开销问题和资源不足问题。通过对多个任务重复使用线程，线程创建的开销就被分摊到了多个任务上了，而且由于在请求到达时线程已经存在，所以消除了线程创建所带来的延迟。这样，就可以立即为请求服务，使用应用程序响应更快。另外，通过适当的调整线程中的线程数目可以防止出现资源不足的情况。
     
```

```
降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
```

###### 步骤

```
使用工厂类 Executors中的静态方法创建线程对象,指定线程的个数

static ExecutorService newFixedThreadPool(int 个数) 返回线程池对象

返回的是ExecutorService接口的实现类 (线程池对象)

接口实现类对象,调用方法submit (Ruunable r) 提交线程执行任务
```

```
 public class ThreadPoolDemo {
        public static void main(String[] args) {
          //调用工厂类的静态方法,创建线程池对象
          //返回线程池对象,是返回的接口
          ExecutorService es = Executors.newFixedThreadPool(2);
            //调用接口实现类对象es中的方法submit提交线程任务
          //将Runnable接口实现类对象,传递
          es.submit(new ThreadPoolRunnable());
          es.submit(new ThreadPoolRunnable());
          es.submit(new ThreadPoolRunnable());
          }
  }
public class ThreadPoolRunnable implements Runnable{
    public void run(){
      System.out.println(Thread.currentThread().getName()+" 线程提交任务");
    }
  }
```

```
除此之外，ExecutorService还继承了Executor接口（注意区分Executor接口和Executors工厂类），这个接口只有一个execute()方法，最后我们看一下整个继承树：

    使用Executors执行多线程任务的步骤如下：

  • 调用Executors类的静态工厂方法创建一个ExecutorService对象，该对象代表一个线程池；

  • 创建Runnable实现类或Callable实现类的实例，作为线程执行任务；

  • 调用ExecutorService对象的submit()方法来提交Runnable实例或Callable实例；

  • 当不想提交任务时，调用ExecutorService对象的shutdown()方法来关闭线程池。   
```



#### 5.实例变量和线程安全

##### 1.不共享数据的情况

```
public class MyThread extends Thread {

	private int count = 5;

	public MyThread(String name) {
		super();
		this.setName(name);
	}

	@Override
	public void run() {
		super.run();
		while (count > 0) {
			count--;
			System.out.println("由 " + MyThread.currentThread().getName() + " 计算，count=" + count);
		}
	}

	public static void main(String[] args) {
		MyThread a = new MyThread("A");
		MyThread b = new MyThread("B");
		MyThread c = new MyThread("C");
		a.start();
		b.start();
		c.start();
	}
}
每个线程都有一个属于自己的实例变量count，它们之间互不影响。

```

##### 2.共享数据的情况

```
public class SharedVariableThread extends Thread {
	private int count = 5;

	@Override
	public void run() {
		super.run();
		count--;
		System.out.println("由 " + SharedVariableThread.currentThread().getName() + " 计算，count=" + count);
	}

	public static void main(String[] args) {

		SharedVariableThread mythread = new SharedVariableThread();
		// 下列线程都是通过mythread对象创建的
		Thread a = new Thread(mythread, "A");
		Thread b = new Thread(mythread, "B");
		Thread c = new Thread(mythread, "C");
		Thread d = new Thread(mythread, "D");
		Thread e = new Thread(mythread, "E");
		a.start();
		b.start();
		c.start();
		d.start();
		e.start();
	}
}
结果出现了错误count并不是依次递减
因为在大多数jvm中，count–的操作分为如下下三步：

取得原有count值
计算i -1
对i进行赋值
所以多个线程同时访问时出现问题就是难以避免的了。

解决办法：一种是利用 synchronized 关键字（保证任意时刻只能有一个线程执行该方法），一种是利用 AtomicInteger 类（JUC 中的 Atomic 原子类）。
```

#### 6.一些常用方法

##### currentThread()

返回对当前正在执行的线程对象的引用。

##### getId()

返回此线程的标识符

##### getName()

返回此线程的名称

##### getPriority()

返回此线程的优先级

##### isAlive()

测试这个线程是否还处于活动状态。

活动状态就是线程已经启动且尚未终止。线程处于正在运行或准备运行的状态。

##### sleep(long millis)

使当前正在执行的线程以指定的毫秒数“休眠”（暂时停止执行），具体取决于系统定时器和调度程序的精度和准确性。

##### interrupt()

中断这个线程。

##### interrupted() 和isInterrupted()

interrupted()：测试当前线程是否已经是中断状态，执行后具有将状态标志清除为false的功能

isInterrupted()： 测试线程Thread对相关是否已经是中断状态，但部清楚状态标志

##### setName(String name)

将此线程的名称更改为等于参数 name 。

##### isDaemon()

测试这个线程是否是守护线程。

##### setDaemon(boolean on)

将此线程标记为 daemon线程或用户线程。

##### join()

在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是 主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行

##### yield()

yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU时间。注意：放弃的时间不确定，可能一会就会重新获得CPU时间片。

##### setPriority(int newPriority)

更改此线程的优先级

#### 7.线程的优先级

每个线程都具有各自的优先级，线程的优先级可以在程序中表明该线程的重要性，如果有很多线程处于就绪状态，系统会根据优先级来决定首先使哪个线程进入运行状态。但这个并不意味着低优先级的线程得不到运行，而只是它运行的几率比较小，如垃圾回收机制线程的优先级就比较低。所以很多垃圾得不到及时的回收处理。

线程优先级具有继承特性比如A线程启动B线程，则B线程的优先级和A是一样的。

线程优先级具有随机性也就是说线程优先级高的不一定每一次都先执行完。

Thread类中包含的成员变量代表了线程的某些优先级。如Thread.MIN_PRIORITY（常数1），Thread.NORM_PRIORITY（常数5）,

Thread.MAX_PRIORITY（常数10）。其中每个线程的优先级都在Thread.MIN_PRIORITY（常数1） 到Thread.MAX_PRIORITY（常数10） 之间，在默认情况下优先级都是Thread.NORM_PRIORITY（常数5）。

#### 8.Java多线程分类

```
用户线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程

守护线程：运行在后台，为其他前台线程服务.也可以说守护线程是JVM中非守护线程的 “佣人”。

特点：一旦所有用户线程都结束运行，守护线程会随JVM一起结束工作

应用：数据库连接池中的检测线程，JVM虚拟机启动后的检测线程

最常见的守护线程：垃圾回收线程

如何设置守护线程？
public class MyThread extends Thread {
	private int i = 0;

	@Override
	public void run() {
		try {
			while (true) {
				i++;
				System.out.println("i=" + (i));
				Thread.sleep(100);
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

#### 9.线程同步

 java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查），将会导致数据不准确，相互之间产生冲突，因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用，从而保证了该变量的唯一性和准确性。

##### **1、同步方法**     】

即有synchronized关键字修饰的方法。由于java的每个对象都有一个内置锁，当用此关键字修饰方法时，内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。

```
public synchronized void save(){}
synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类
```

##### **2、同步代码块**     

即有synchronized关键字修饰的语句块。被该关键字修饰的语句块会自动被加上内置锁，从而实现同步。

```
public class Bank {  
     
        private int count =0;//账户余额  
     
        //存钱  
        public void addMoney(int money){  
     
            synchronized (this) {  
                count +=money;  
            }  
            System.out.println(System.currentTimeMillis()+"存进："+money);  
        }  
     
        //取钱  
        public   void subMoney(int money){  
     
            synchronized (this) {  
                if(count-money < 0){  
                    System.out.println("余额不足");  
                    return;  
                }  
                count -=money;  
            }  
            System.out.println(+System.currentTimeMillis()+"取出："+money);  
        }  
     
        //查询  
        public void lookMoney(){  
            System.out.println("账户余额："+count);  
        } 
    }
```

同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。

##### **3、使用特殊域变量(volatile)实现线程同步**   

```
	volatile关键字为域变量的访问提供了一种免锁机制；

   使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新；

   因此每次使用该域就要重新计算，而不是使用寄存器中的值；

   volatile不会提供任何原子操作，它也不能用来修饰final类型的变量。
```

```
public class SynchronizedThread {
 
        class Bank {
 
            private volatile int account = 100;
 
            public int getAccount() {
                return account;
            }
 
            /**
             * 用同步方法实现
             * 
             * @param money
             */
            public synchronized void save(int money) {
                account += money;
            }
 
            /**
             * 用同步代码块实现
             * 
             * @param money
             */
            public void save1(int money) {
                synchronized (this) {
                    account += money;
                }
            }
        }
 
        class NewThread implements Runnable {
            private Bank bank;
 
            public NewThread(Bank bank) {
                this.bank = bank;
            }
 
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    // bank.save1(10);
                    bank.save(10);
                    System.out.println(i + "账户余额为：" +bank.getAccount());
                }
            }
 
        }
 
        /**
         * 建立线程，调用内部类
         */
        public void useThread() {
            Bank bank = new Bank();
            NewThread new_thread = new NewThread(bank);
            System.out.println("线程1");
            Thread thread1 = new Thread(new_thread);
            thread1.start();
            System.out.println("线程2");
            Thread thread2 = new Thread(new_thread);
            thread2.start();
        }
 
        public static void main(String[] args) {
            SynchronizedThread st = new SynchronizedThread();
            st.useThread();
        }
```

多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。用final域，有锁保护的域和volatile域可以避免非同步的问题。

##### **4、使用重入锁（Lock）实现线程同步**

在[Java](http://www.2cto.com/kf/ware/Java/)SE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。ReenreantLock类的常用方法有：

```
ReentrantLock() : 创建一个ReentrantLock实例         
 lock() : 获得锁        
 unlock() : 释放锁
 ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用
```

```
class Bank {
            
            private int account = 100;
            //需要声明这个锁
            private Lock lock = new ReentrantLock();
            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized 
            public void save(int money) {
                lock.lock();
                try{
                    account += money;
                }finally{
                    lock.unlock();
                }
                
            }
        ｝ 
```

#### 10.线程通信

##### **1、借助于Object类的wait()、notify()和notifyAll()实现通信**

```
	线程执行wait()后，就放弃了运行资格，处于冻结状态；

     线程运行时，内存中会建立一个线程池，冻结状态的线程都存在于线程池中，notify()执行时唤醒的也是线程池中的线程，线程池中有多个线程时唤醒第一个被冻结的线程。
      notifyall(), 唤醒线程池中所有线程。
      注： （1） wait(), notify(),notifyall()都用在同步里面，因为这3个函数是对持有锁的线程进行操作，而只有同步才有锁，所以要使用在同步中；
       （2） wait(),notify(),notifyall(),  在使用时必须标识它们所操作的线程持有的锁，因为等待和唤醒必须是同一锁下的线程；而锁可以是任意对象，所以这3个方法都是Object类中的方法。
```

##### **2、使用Condition控制线程通信**

```
 jdk1.5中，提供了多线程的升级解决方案为：

     （1）将同步synchronized替换为显式的Lock操作；

     （2）将Object类中的wait(), notify(),notifyAll()替换成了Condition对象，该对象可以通过Lock锁对象获取;

     （3）一个Lock对象上可以绑定多个Condition对象，这样实现了本方线程只唤醒对方线程，而jdk1.5之前，一个同步只能有一个锁，不同的同步只能用锁来区分，且锁嵌套时容易死锁。
```

```
class Resource{  
        private String name;  
        private int count=1;  
        private boolean flag=false;  
        private Lock lock = new ReentrantLock();/*Lock是一个接口，ReentrantLock是该接口的一个直接子类。*/  
        private Condition condition_pro=lock.newCondition(); /*创建代表生产者方面的Condition对象*/  
        private Condition condition_con=lock.newCondition(); /*使用同一个锁，创建代表消费者方面的Condition对象*/  
          
        public void set(String name){  
            lock.lock();//锁住此语句与lock.unlock()之间的代码  
            try{  
                while(flag)  
                    condition_pro.await(); //生产者线程在conndition_pro对象上等待  
                this.name=name+"---"+count++;  
                System.out.println(Thread.currentThread().getName()+"...生产者..."+this.name);  
                flag=true;  
                 condition_con.signalAll();  
            }  
            finally{  
                lock.unlock(); //unlock()要放在finally块中。  
            }  
        }  
        public void out(){  
            lock.lock(); //锁住此语句与lock.unlock()之间的代码  
            try{  
                while(!flag)  
                    condition_con.await(); //消费者线程在conndition_con对象上等待  
            System.out.println(Thread.currentThread().getName()+"...消费者..."+this.name);  
            flag=false;  
            condition_pro.signqlAll(); /*唤醒所有在condition_pro对象下等待的线程，也就是唤醒所有生产者线程*/  
            }  
            finally{  
                lock.unlock();  
            }  
        }  
    }  
```

##### **3、使用阻塞队列（BlockingQueue）控制线程通信**

```
BlockingQueue是一个接口，也是Queue的子接口。BlockingQueue具有一个特征：当生产者线程试图向BlockingQueue中放入元素时，如果该队列已满，则线程被阻塞；但消费者线程试图从BlockingQueue中取出元素时，如果队列已空，则该线程阻塞。程序的两个线程通过交替向BlockingQueue中放入元素、取出元素，即可很好地控制线程的通信。

BlockingQueue提供如下两个支持阻塞的方法：

  （1）put(E e)：尝试把Eu元素放如BlockingQueue中，如果该队列的元素已满，则阻塞该线程。

  （2）take（）：尝试从BlockingQueue的头部取出元素，如果该队列的元素已空，则阻塞该线程。

BlockingQueue继承了Queue接口，当然也可以使用Queue接口中的方法，这些方法归纳起来可以分为如下三组：

  （1）在队列尾部插入元素，包括add（E e）、offer（E e）、put（E e）方法，当该队列已满时，这三个方法分别会抛出异常、返回false、阻塞队列。

  （2）在队列头部删除并返回删除的元素。包括remove（）、poll（）、和take（）方法，当该队列已空时，这三个方法分别会抛出异常、返回false、阻塞队列。

  （3）在队列头部取出但不删除元素。包括element（）和peek（）方法，当队列已空时，这两个方法分别抛出异常、返回false。
```

BlockingQueue接口包含如下5个实现类：

```
ArrayBlockingQueue ：基于数组实现的BlockingQueue队列。

LinkedBlockingQueue：基于链表实现的BlockingQueue队列。

PriorityBlockingQueue：它并不是保准的阻塞队列，该队列调用remove（）、poll（）、take（）等方法提取出元素时，并不是取出队列中存在时间最长的元素，而是队列中最小的元素。
                       它判断元素的大小即可根据元素（实现Comparable接口）的本身大小来自然排序，也可使用Comparator进行定制排序。

SynchronousQueue：同步队列。对该队列的存、取操作必须交替进行。

DelayQueue：它是一个特殊的BlockingQueue，底层基于PriorityBlockingQueue实现，不过，DelayQueue要求集合元素都实现Delay接口（该接口里只有一个long getDelay（）方法），
            DelayQueue根据集合元素的getDalay（）方法的返回值进行排序。
```

#### 11.死锁

```
互斥条件：资源不能被共享，只能被同一个进程使用

请求与保持条件：已经得到资源的进程可以申请新的资源

非剥夺条件：已经分配的资源不能从相应的进程中被强制剥夺

循环等待条件：系统中若干进程组成环路，该环路中每个进程都在等待相邻进程占用的资源
```

eg

```
例子：进程A中包含资源A,进程B中包含资源B，A的下一步需要资源B，B的下一步需要资源A，所以它们就互相等待对方占有的资源释放，所以也就产生了一个循环等待死锁。
```

处理死锁的方法

```
忽略该问题，也即鸵鸟算法。当发生了什么问题时，不管他，直接跳过，无视它；
检测死锁并恢复；
资源进行动态分配；
破除上面的四种死锁条件之一。
```

#### 12.线程相关类

##### **ThreadLocal**

 ThreadLocal它并不是一个线程，而是一个可以在每个线程中存储数据的数据存储类，通过它可以在指定的线程中存储数据，数据存储之后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到该线程的数据。 即多个线程通过同一个ThreadLocal获取到的东西是不一样的，就算有的时候出现的结果是一样的（偶然性，两个线程里分别存了两份相同的东西），但他们获取的本质是不同的。使用这个工具类可以简化多线程编程时的并发访问，很简洁的隔离多线程程序的竞争资源。

​     对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。ThreadLocal类提供了如下的三个public方法：

```
ThreadLocal()
          创建一个线程本地变量。
T get()
          返回此线程局部变量的当前线程副本中的值，如果这是线程第一次调用该方法，则创建并初始化此副本。
protected  T initialValue()
          返回此线程局部变量的当前线程的初始值。
```

```
ThreadLocal在日常开发中使用到的地方较少，但是在某些特殊的场景下，通过ThreadLocal可以轻松实现一些看起来很复杂的功能。一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑使用ThreadLocal。例如在Handler和Looper中。对于Handler来说，它需要获取当前线程的Looper，很显然Looper的作用域就是线程并且不同的线程具有不同的Looper，这个时候通过ThreadLocal就可以轻松的实现Looper在线程中的存取。如果不采用ThreadLocal，那么系统就必须提供一个全局的哈希表供Handler查找指定的Looper，这样就比较麻烦了，还需要一个管理类。
ThreadLocal的另一个使用场景是复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程中的任务过于复杂，就可能表现为函数调用栈比较深以及代码入口的多样性，这种情况下，我们又需要监听器能够贯穿整个线程的执行过程。这个时候就可以使用到ThreadLocal，通过ThreadLocal可以让监听器作为线程内的全局对象存在，在线程内通过get方法就可以获取到监听器。如果不采用的话，可以使用参数传递，但是这种方式在设计上不是特别好，当调用栈很深的时候，通过参数来传递监听器这个设计太糟糕。而另外一种方式就是使用static静态变量的方式，但是这种方式存在一定的局限性，拓展性并不是特别的强。比如有10个线程在执行，就需要提供10个监听器对象。
```

ThreadLocal和其他所有的同步机制一样，都是为了解决多线程中对于同一变量的访问冲突。值普通的同步机制中，通过对象加锁来实现多线程对同一变量的安全访问，且该变量是多线程共享的，所有需要使用这种同步机制来明确分开是在什么时候对变量进行读写，在什么时候需要锁定该对象。此种情况下，系统并没有将这个资源复制多份，而是采取安全机制来控制访问而已。ThreadLocal只是从另一个角度解决多线程的并发访问，即将需要并发访问的资源复制多份，每个线程拥有一份资源，每个线程都有自己的资源副本。

若多个线程之间需要共享资源，以达到线程间的通信时，就使用同步机制；若仅仅需要隔离多线程之间的关系资源，则可以使用ThreadLocal。