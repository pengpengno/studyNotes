---
title: ReentrantLock
date: 2021-11-11 13:36:40.392
updated: 2022-04-27 16:35:19.342
url: /archives/reentrantlock
categories: 
- 锁
tags: 
---



 
## ReentrantLock
1.什么是ReentrantLock
1.1ReentrantLock 与Synchronized区别
在面试中询问ReentrantLock与Synchronized区别时，一般回答都是

### 什么是ReentrantLock

ReentrantLock是JDK方法，需要手动声明上锁和释放锁，因此语法相对复杂些；如果忘记释放锁容易导致死锁
ReentrantLock具有更好的细粒度，可以在ReentrantLock里面设置内部Condititon类，可以实现分组唤醒需要唤醒的线程
RenentrantLock能实现公平锁
Synchronized

Synchoronized语法上简洁方便
Synchoronized是JVM方法，由编辑器保证枷锁和释放
 构造方法

```
public class ReentrantLock implements Lock, java.io.Serializable {

 public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

}
```

- ReentrantLock有两个构造方法，一个是无参的 ReentrantLock() ；另一个含有布尔参数public ReentrantLock(boolean fair)。后面一个构造函数说明ReentrantLock可以新建公平锁；而Synchronized只能建立非公平锁。

（公平锁：公平锁，是按照通过CLH等待线程按照先来先得的规则，公平的获取锁；而非公平锁，则当线程要获取锁时，它会无视CLH等待队列而直接获取锁

转装于：https://www.cnblogs.com/kexianting/p/8550975.html


Lock中的newCondition方法
```
 public Condition newCondition() {
        return sync.newCondition();
    }
```
### Condition详解：https://www.cnblogs.com/gemine/p/9039012.html 与Object的wait与notify用法类似。

获取锁的方式

void lock()获取锁。

void lockInterruptibly()如果当前线程未被中断，则获取锁。

boolean tryLock() 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。

boolean tryLock(long timeout, TimeUnit unit) timeout：时间long值如10L   TimeUnit 时间单位如TimeUnit.SECONDS 代表等待10秒若未获取到锁则放弃。

```
public class MyLock implements Runnable {
    int i =10;
    int num = 100;
    boolean flag = true;
    ReentrantLock l = new ReentrantLock(true);
    public void run() {
        while (i >= 1) {
            //加锁
            //如果当前获取不到锁则返回false 不等待继续执行下面
            if (l.tryLock()) {
                try {
                    flag=false;
                    Thread.sleep(500); //睡眠一会
                    if (i > 0) {
                        System.out.println(Thread.currentThread().getName() + "正在售卖:" + (i--) + "张票");
                    }
                } catch (InterruptedException e) {
//                  e.printStackTrace();
                } finally {
                    flag=true;
                    l.unlock();
                }

            }
     if(flag&&num>0)
            {

                    int i = 5;
//                System.out.println(Thread.currentThread().getName());
                for (int r = 0 ;r<10;r++){
                    try {
                        Thread.sleep(900);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
//                    if(i>0)
                    System.out.println(i--+""+Thread.currentThread().getName());
                }
                i=5;
            }
 } }
```
得出结论：当线程没拿到锁后，会开始执行下面的代码，且当执行完之后才会继续去尝试获取锁
复制代码
 

 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。

void unlock()释放锁，建议放在finally中 避免出现异常导致死锁的发生。