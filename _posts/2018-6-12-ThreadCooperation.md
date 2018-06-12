---
layout: post
title: Redis持久化
tags: [Redis]
---

目录：
* [wait/notify](#wait/notify)
* [条件变量Condition](#条件变量condition)
* [CountDownLatch](#countdownlatch)
* [cyclibarrier](#cyclibarrier)


  ## wait/notify
  一个线程因其所执行的目标动作所需的保护条件未满足而被暂停的过程就称为等待。一个线程更新了系统的状态，使得其他线程所需的保护条件得以满足的时候唤醒那些被暂停的线程的过程就称为通知。

  Java中的等待与通知可以通过wait/notify来实现。

  wait/notify/notifyAll是Object类的方法，即所有类都拥有该方法，其中object.wait()方法执行的线程就称为等待线程，object.notify执行的线程就称为通知线程。一个线程只有持有一个对象内部锁的情况下，才能调用该对象的wait方法。

  等待线程在其被唤醒、继续运行到再次持有相应对象内部锁的这段时间，由于其他线程可能抢先获得相应内部锁并更新了相关共享变量而导致该线程所需保护的条件再次不成立，因此wait调用返回后需要再次判断此时保护条件是否成立，因此wait方法应该放在循环内

  同时wait 方法暂停时释放的锁只是该object对象的这个锁，线程中其他锁并不会被释放

  当wait方法被唤醒时，它会先申请该object对象的内部锁，接着wait方法才返回。
  ```java
  synchronized(object){
      while(条件不成立){
          object.wait();
      }

      dosomething();
  }

  synchronized(object){
      updateState;
      object.notify();
  }
  ```
  object.notify的执行线程持有的相应对象的内部锁只有在notify方法所在的临界区代码执行结束后才被释放，notify方法本身并不会释放锁。

  notify方法唤醒的线程只是该object对象上任意的一个等待线程，如果需要唤醒object上等待的所有线程，那么可以使用notifyAll方法。

  ### wait/notify的开销
  - 过早唤醒：就是等待线程在其所需的保护条件未成立的情况下被唤醒
  - 信号丢失：
  - 欺骗性唤醒：
  - 上下文切换：

  ### wait/notify与join的区别
  首先，join是Thread的方法，而wait/notify是object的方法。join方法其实就是调用wait来实现的。
  ```java
  public final synchronized void join(long millis) throws InterruptedException {
      long base = System.currentTimeMillis();
      long now = 0L;
      if (millis < 0L) {
          throw new IllegalArgumentException("timeout value is negative");
      } else {
          if (millis == 0L) {
              while(this.isAlive()) {
                  this.wait(0L);
              }
          } else {
              while(this.isAlive()) {
                  long delay = millis - now;
                  if (delay <= 0L) {
                      break;
                  }

                  this.wait(delay);
                  now = System.currentTimeMillis() - base;
              }
          }

      }
  }

  public final void join() throws InterruptedException {
          this.join(0L);
  }
  ```

  ## 条件变量Condition
  Condition接口可作为wait/notify的替代品，它为解决过早唤醒提供了支持，可以通过任意锁的Lock.newCondition来创建一个Condition实例。Condition实例被称为条件变量。每个Condition实例内部维护一个用于存储该条件等待线程的队列，所以唤醒的时候只会从该条件的队列中唤醒。

  ```java
  private final Lock lock = new ReentrantLock();
  private final Condition condition = lock.newCondition();

  public void doMethod() throws InterruptedException {
      lock.lock();
      try {
          while (条件不成立){
              condition.await();
          }
      }finally {
          lock.unlock();
      }
  }

  public void notifyMethod(){
      lock.lock();
      try {
          // 更新条件
          updateState;
          // 唤醒线程
          condition.notify();
      }finally {
          lock.unlock();
      }
  }    
  ```

  使用条件变量的开销与wait/notify差不多，但是其上下文切换比wait/notify要少

  ## CountDownLatch
  CountDownLatch可以用来实现一个或多个线程等待其他线程完成一组特定的操作之后才继续运行，即使用wait和join只能实现一个线程等待另一个线程，而CountDownLatch可以实现等待多个线程。另外CountDownLatch是一次性的，一个CountDownLatch实例只能实现一次等待。并且使用的时候不需要加锁。

  ## CycliBarrier
  与CountDownLatch不同的是CycliBarrier可以重复使用。
