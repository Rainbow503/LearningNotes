# JUC   &    Java 并发编程

编写优质的`并发代码`是一件难度极高的事情。Java语言从第一版本开始内置了对多线程的支持，这一点在当年是非常了不起的，但是当我们对并发编程有了更深刻的认识和更多的实践后，实现并发编程就有了更多的方案和更好的选择。本文是对并发编程的一点总结和思考，同时也分享了Java 5以后的版本中如何编写并发代码的一点点经验。



> 并发其实是一种解耦合的策略，它帮助我们把做什么（目标）和什么时候做（时机）分开。
>
> 这样做可以明显改进应用程序的吞吐量（获得更多的CPU调度时间）和结构（程序有多个部分在协同工作）。
>
> 做过`Java Web`开发的人都知道，Java Web中的`Servlet`程序在`Servlet``容器`的支持下采用单实例[多线程](https://so.csdn.net/so/search?q=多线程&spm=1001.2101.3001.7020)的工作模式，`Servlet`容器为你处理了并发问题。









# J.U.C

## Lock接口及ReentrantLock对象分析及应用

并发编程领域，有两大核心问题：一个是互斥，即同一时刻只允许一个线程访问共享资源；另一个是同步，即线程之间如何通信、协作。

这两大问题，在Java SDK 并发包可通过 Lock 和 Condition 两个接口来实现，其中Lock 用于解决互斥问题，Condition 用于解决同步问题。

Java SDK 并发包里的 Lock 接口中，不仅有支持类似 `synchronized `的隐式加锁方法，还支持超时、非阻塞、可中断的方式获取锁， 这三种方式为我们编写更加安全、健壮的并发程序提供了很大的便利。

我们来一起看看Lock接口常用方法，关键方法如下：

- 1)void lock() 获取锁对象，优先考虑是锁的获取，而非中断。
- 2)void lockInterruptibly() 获取锁，但优先响应中断而非锁的获取。
- 3)boolean tryLock() 试图获取锁。
- 4)boolean tryLock(long timeout, TimeUnit timeUnit) 试图获取锁，并设置等待时长。
- 5)void unlock()释放锁对象。

Java SDK 并发包里的`ReentrantLock`实现了`Lock`接口，是一个可重入的互斥锁(“独占锁”), 同时提供了”公平锁”和”非公平锁”的支持。所谓公平锁和非公平锁其含义如下：

- 1)公平锁：在多个线z程争用锁的情况下，公平策略倾向于将访问权授予等待时间最长的线程。也就是说，相当于有一个线程等待队列，先进入等待队列的线程后续会先获得锁

- 2)非公平锁：在多个线程争用锁的情况下，能够最终获得锁的线程是随机的（由底层OS调度）。
  `ReetrantLock`简易应用如下（默认为非公平策略）

  ```java
  class Counter{
    ReentrantLock lock = new ReentrantLock();
    int count = 0;
    void increment() {
      lock.lock();
      try {
        count++;
      } finally {
        lock.unlock();
      }
    }
  }
  ```

  其中，这里的锁通过`lock（）`方法获取锁,通过`unlock（）`方法释放锁。重要的是将代码包装成`try/finally`块，以确保在出现异常时解锁。

  这个方法和`synchronized`关键字修饰的方法一样是线程安全的。在任何给定的时间，只有一个线程可以持有锁。
  `ReetrantLock`对象在构建时，可以基于`ReentrantLock(boolean fair)`构造方法参数，设置对象的”公平锁”和”非公平锁”特性。

  其中`fair`的值`true`表示“公平锁”。这种公平锁，会影响其性能，但是在一些公平比效率更加重要的场合中公平锁特性就会显得尤为重要。关键代码示例如下：

  ```java
  public void performFairLock(){
  		  ReentrantLock lock = new ReentrantLock(true);
  		   try {
  		        //Critical section here
  		        } finally {
  		            lock.unlock();
  		        }
  		    //...
  }
  ```

  `ReetrantLock`对象在基于业务获取锁时，假如希望有等待时间，可以借助tryLock实现，关键代码示例如下：

  ```java
   public void performTryLock(){
  		    //...
  		    boolean isLockAcquired = lock.tryLock(1, TimeUnit.SECONDS);
  		     
  		    if(isLockAcquired) {
  		        try {
  		            //Critical section here
  		        } finally {
  		            lock.unlock();
  		        }
  		    }
  		    //...
  		}
  }
  ```

  

