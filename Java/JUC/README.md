# 进程与线程认知强化

## 如何理解进程与线程？

- 进程：操作系统进行资源调度和分配的基本单位（例如浏览器，APP，JVM）
- 线程：进程中的最小执行单位（可以理解为一个顺序的执行流）

![在这里插入图片描述](pics/20210708213529822.png)

说明：同一个进程内的多个线程共享资源。

## 如何理解多线程中的并行与并发？

并发：多线程抢占CPU，可能不同时执行，侧重于多个任务交替执行。

![在这里插入图片描述](pics/20210708213558189.png)

> 现在的操作系统无论是`windows`，`linux`还是`macOS`等其实都是多用户多任务分时操作系统，使用这些操作系统的的用户可以“同时”干多件事情。
>
> 但实际上，对于单机`CPU`的计算机而言，在同一时间只能干一件事，为了看起来像是“同时干多件事”分时操作系统把CPU的时间划分成了长短进本相同的时间区间，即“时间片”，通过操作系统的管理，把时间片依次轮流的分配给各个线程任务使用。
>
> 我们看似的“同时干多件事”，其实是通过CPU时间片技术并发完成的。例如：多个线程并发使用一个CPU资源并发执行任务的线程时序图。

![在这里插入图片描述](pics/20210708213639490.gif)

并行：线程可以不共享CPU，可每个线程一个CPU同时执行多个任务。

![在这里插入图片描述](pics/20210708213752956.png)

总之：个人认为并行只出现在多CPU或多核CPU中，而并发可理解为并行中的一个子集。

## 如何理解线程的生命周期及状态变化？

一个线程从创建，运行，到最后销毁的这个过程称之为线程的生命周期，在这个生命周期过程中线程可能会经历如下几个状态：![在这里插入图片描述](pics/20210708213825726.png)

这些状态可归纳为：状态分别为新建状态，就绪状态，运行状态，阻塞状态，死亡状态。

线程并发安全问题认知强化
如何理解线程安全与不安全？
多个线程并发执行时，仍旧能够保证数据的正确性，这种现象称之为线程安全。
多个线程并发执行时，不能能够保证数据的正确性，这种现象称之为线程不安全。
案例分享：如何保证12306中的订票操作的安全。

第一步：编写售票任务类：

```java
class TicketTask implements Runnable{
  int ticket=10;
  @Override
  public void run() {
    doTicket();
  }
  public void doTicket() {
    while(true) {
      if(ticket <= 0) break;
      System.out.println(ticket--);
    }
  }
}
```

第二步：编写售票测试方法：

```java
public static void main(String[] args) {
  TicketTask task=new TicketTask();
  Thread t1=new Thread(task);
  Thread t2=new Thread(task);
  Thread t3=new Thread(task);

  t1.start();
  t2.start();
  t3.start();  
}
```

## 导致线程不安全的因素有哪些？

1. 多个线程并发执行。
2. 多个线程并发执行时存在共享数据集(临界资源)。
3. 多个线程在共享数据集上的操作不是原子操作。

例如：现有一生产者消费者模型，生产者和消费者并发操作容器对象。

![在这里插入图片描述](pics/20210708214051873.png)

如何保证并发线程的安全性？

- 第一；对共享进行限制访问（例如加锁：syncronized，Lock）
- 第二：基于CAS实现非阻塞同步（基于CPU硬件技术支持）
- 第三：取消共享，每个线程一个对象实例（例如threadlocal）



说明：Java中的线程安全问题的主要关注点有3个：可见性，有序性，原子性；

> Java内存模型（JMM）解决了可见性和有序性问题，而解决了原子性问题。 

<hr>

# 如何理解JAVA中的悲观锁和乐观锁？
JAVA中为了保证多线程并发访问的安全性，提供了基于锁的应用，大体可归纳为两大类，即悲观锁和乐观锁。

悲观锁 & 乐观锁定义说明：

1. 悲观锁：假定会发生并发冲突，屏蔽一切可违反数据完整性的操作，例如java中可以基于syncronized,Lock，ReadWriteLock等实现。

2. 乐观锁实现：假设不会发生冲突，只在提交操作时检查是否违反数据完整性，例如java中可借助CAS（ Compare And Swap）算法实现(此算法依赖硬件CPU)。

悲观锁&乐观锁应用场景说明：
1)悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
2)乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升

悲观锁&乐观锁应用案例分析

悲观锁实现计数器：
方案1：

```java
class Counter{
		private int count;
		public synchronized int count() {
			count++;
			return count;
		}
}
```

方案2：

```java
class Counter{
  private int count;
  private Lock lock = new ReentrantLock();
  public  int count() {
    lock.lock();
    try {
      count++;
      return count;
    }finally {
      lock.unlock();
    }
  }
}
```

乐观锁实现计数器：

```java
class Counter{
	private AtomicInteger at = new AtomicInteger();
	public  int count() {
		return at.incrementAndGet();
	}
}
```

其中 AtomicInteger 是基于CAS算法实现。

## 如何理解线程的上下文切换？

一个线程得到CPU执行的时间是有限的。当此线程用完为其分配的CPU时间以后，cpu会切换到下一个线程执行。

但是在这之前，线程需要将当前的状态进行保存，以便下次再次获得CPU时间片时可以加载对应的状态以继续执行剩下的任务。

而这个切换过程是需要耗费时间的，会影响多线程程序的执行效率，所以在在使用多线程时要减少线程的频繁切换。

减少多线程上下文切换的方案如下：

- 无锁并发编程：锁的竞争会带来线程上下文的切换
- CAS算法：CAS算法在数据更新方面，可以达到锁的效果
- 使用最少线程：避免不必要的线程等待
- 使用协程：单线程完成多任务的调度和切换，避免多线程



## 如何理解死锁以及避免死锁问题？

多个线程互相等待已经被对方线程正在占用的锁，导致陷入彼此等待对方释放锁的状态，这个过程称之为死锁。
如何避免死锁呢？

- 避免一个线程中同时获取多个锁
- 避免一个线程在一个锁中获取其他的锁资源
- 考虑使用定时锁来替换内部锁机制，如lock.tryLock(timeout)

可能出现死锁的案例

```java
class SyncThread implements Runnable {
  private Object obj1;
  private Object obj2;
  public SyncThread(Object o1, Object o2) {
    this.obj1 = o1;
    this.obj2 = o2;
  }
  @Override
  public void run() {
    synchronized (obj1) {
      work();
      synchronized (obj2) {
        work();
      }
    }
  }
  private void work() {
    try {Thread.sleep(30000);} catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

死锁测试

```java
public class TestDeadLock01 {
	public static void main(String[] args)throws Exception {
		 Object obj1 = new Object();
		 Object obj2 = new Object();
		 Thread t1 = new Thread(new SyncThread(obj1, obj2), "t1");
		 Thread t2 = new Thread(new SyncThread(obj2, obj1), "t2");
		 t1.start();
		 t2.start();
	}
}
```

# 线程通讯与进程通讯应用增强

## 如何理解进程与线程通讯？

- 线程通讯：java中的多线程通讯主要是共享内存（变量）等方式
- 进程通讯：java中进程通讯（IPC）主要是Socket，MQ等

![在这里插入图片描述](pics/20210708214329828.png)

## 如何实现进程内部线程之间的通讯？

### 基于wait/nofity/notifyall实现

- Wait:阻塞正在使用监视器对象的线程，同时释放监视器对象
- notify: 唤醒在监视器对象上等待的单个线程，但不释放监视器对象，此时调用该方法的代码继续执行，直到执行结束才释放对象锁
- notifyAll: 唤醒在监视器对象上等待的所有线程，但不释放监视器对象，此时调用该方法的代码继续执行，直到执行结束才释放对象锁

> 1） 这些方法必须应用在同步代码块或同步方法中
> 2） 这些方法必须由监视器对象（对象锁）调用

说明：使用wait/notify/notifyAll的作用一般是为了避免轮询带来的性能损失。

手动实现阻塞式队列，并基于wait()/notifyAll()方法实现实现线程在队列上的通讯

```java
/**
 * 有界消息队列：用于存取消息
 * 1)数据结构：数组(线性结构)
 * 2)具体算法：FIFO(先进先出)-First in First out
 */
public class BlockContainer<T> {//类泛型 
	/**用于存储数据的数组*/
	private Object[] array;
	/**记录有效元素个数*/
	private int size;
	public BlockContainer () {
		this(16);//this(参数列表)表示调用本类指定参数的构造函数
	}
    public BlockContainer (int cap) {
		array=new Object[cap];//每个元素默认值为null
	}
}
```

向容器添加put方法，用于放数据。

```java
/**
     * 生产者线程通过put方法向容器放数据
     * 数据永远放在size位置
     * 说明：实例方法内部的this永远指向
     * 调用此方法的当前对象(当前实例)
     * 注意：静态方法中没有this，this只能
     * 应用在实例方法，构造方法，实例代码块中
     */
public synchronized void put(T t){//同步锁：this
  //1.判定容器是否已满，满了则等待
  while(size==array.length)
    try{this.wait();}catch(Exception e){}
  //2.放数据
  array[size]=t;
  //3.有效元素个数加1
  size++;
  //4.通知消费者取数据
  this.notifyAll();
}
```

向容器类添加take方法，用于从容器取数据。

```java
/**
     * 消费者通过此方法取数据
     * 位置：永远取下标为0的位置的数据
     * @return
     */
@SuppressWarnings("unchecked")
public synchronized T take(){
  //1.判定容器是否为空，空则等待
  while(size==0)
    try{this.wait();}catch(Exception e){}
  //2.取数据
  Object obj=array[0];
  //3.移动元素
  System.arraycopy(
    array,//src 原数组
    1, //srcPos 从哪个位置开始拷贝
    array,  //dest 放到哪个数组
    0, //destPost 从哪个位置开始放
    size-1);//拷贝几个
  //4.有效元素个数减1
  size--;
  //5.将size位置为null
  array[size]=null;
  //6.通知生产者放数据
  this.notifyAll();//通知具备相同锁对象正在wait线程
  return (T)obj;
}
```

### 基于Condition实现

Condition 是一个用于多线程间协同的工具类，基于此类可以方便的对持有锁的线程进行阻塞或唤醒阻塞的线程。它的强大之处在于它可以为多个线程间建立不同的Condition，通过signal()/signalall()方法指定要唤醒的不同线程。

1. 基于Lock对象获取Condition对象
2. 基于Condition对象的await()/signal()/signalall()方法实现线程阻塞或唤醒。

手动实现阻塞式队列，并基于wait()/notifyAll()方法实现实现线程在队列上的通讯。

```java
/**
 * 有界消息队列：用于存取消息
 * 1)数据结构：数组(线性结构)
 * 2)具体算法：FIFO(先进先出)-First in First out
 */
public class BlockContainer<T> {//类泛型 

  /**用于存储数据的数组*/
  private Object[] array;
  /**记录有效元素个数*/
  private int size;
  public BlockContainer() {
    this(16);//this(参数列表)表示调用本类指定参数的构造函数
  }
  public BlockContainer(int cap) {
    array=new Object[cap];//每个元素默认值为null
  }
  //JDK1.5以后引入的可重入锁(相对于synchronized灵活性更好)
  private ReentrantLock lock=new ReentrantLock(true);// true表示使用公平锁，默认是非公平锁
  private Condition producerCondition=lock.newCondition();//通讯条件
  private Condition consumerCondition=lock.newCondition();//通讯条件
}
```

向容器中添加put方法，用于向容器放数据

````java
/**
     * 生产者线程通过put方法向容器放数据
     * 数据永远放在size位置
     * 说明：实例方法内部的this永远指向
     * 调用此方法的当前对象(当前实例)
     * 注意：静态方法中没有this，this只能
     * 应用在实例方法，构造方法，实例代码块中
     */
public void put(T t){//同步锁：this
  System.out.println("put");
  lock.lock();
  try{
    //1.判定容器是否已满，满了则等待
    while(size==array.length)
      //等效于Object类中的wait方法
      try{producerCondition.await();}catch(Exception e){e.printStackTrace();}
    //2.放数据
    array[size]=t;
    //3.有效元素个数加1
    size++;
    //4.通知消费者取数据
    consumerCondition.signalAll();//等效于object类中的notifyall()
  }finally{
    lock.unlock();
  }
}
````

在容器类中添加take方法用于从容器取数据

```java
/**
     * 消费者通过此方法取数据
     * 位置：永远取下标为0的位置的数据
     * @return
     */
@SuppressWarnings("unchecked")
public  T take(){
  System.out.println("take");
  lock.lock();
  try{
    //1.判定容器是否为空，空则等待
    while(size==0)
      try{consumerCondition.await();}catch(Exception e){}
    //2.取数据
    Object obj=array[0];
    //3.移动元素
    System.arraycopy(
      array,//src 原数组
      1, //srcPos 从哪个位置开始拷贝
      array,  //dest 放到哪个数组
      0, //destPost 从哪个位置开始放
      size-1);//拷贝几个
    //4.有效元素个数减1
    size--;
    //5.将size位置为null
    array[size]=null;
    //6.通知生产者放数据
    producerCondition.signalAll();//通知具备相同锁对象正在wait线程
    return (T)obj;
  }finally{
    lock.unlock();
  }
}
```

## 如何实现进程之间间通讯（IPC）？

基于BIO实现的简易server服务器

```java
public class BioMainServer01 {
  private Logger log=LoggerFactory.getLogger(BioMainServer01.class);	
  private ServerSocket server;
  private volatile boolean isStop=false;
  private int port;
  public BioMainServer01(int port) {
    this.port=port;
  }
  public void doStart()throws Exception {
    server=new ServerSocket(port);
    while(!isStop) {
      Socket socket=server.accept();
      log.info("client connect");
      doService(socket);
    }
    server.close();
  }
  public void doService(Socket socket) throws Exception{
    InputStream in=socket.getInputStream();
    byte[] buf=new byte[1024];
    int len=-1;
    while((len=in.read(buf))!=-1) {
      String content=new String(buf,0,len);
      log.info("client say {}", content);
    }
    in.close();
    socket.close();
  }
  public void doStop() {
    isStop=false;
  }
  public static void main(String[] args)throws Exception {
    BioMainServer01 server=new BioMainServer01(9999);
    server.doStart();
  }
}
```

启动服务，然后打开浏览器进行访问或者通过如下客户端端访问

```java
public class BioMainClient {
	public static void main(String[] args) throws Exception{
		Socket socket=new Socket();
		socket.connect(new InetSocketAddress("127.0.0.1", 9999));
		OutputStream out=socket.getOutputStream();
		Scanner sc=new Scanner(System.in);
		System.out.println("client input:");
		out.write(sc.nextLine().getBytes());
		out.close();
		sc.close();
		socket.close();
	}
}
```

# 总结（Summary）

本小节主要从线程、进程基本概念着手，逐步进行了线程安全、线程通讯、进程通讯等维度方面的讲解。
