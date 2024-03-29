---
title: synchronized工作方式
date: 2021-10-18 09:38:36.479
updated: 2022-04-27 16:35:26.358
url: /archives/synchronized工作方式
categories: 
- Java | 锁
tags: 
---



synchronized算是多线程中非常常用的加锁方式了，但很多人都不太理解其底层的工作原理。本篇文章博主用尽可能通俗易懂的方式来带大家去看看synchronized究竟是怎么加锁的。在学习本篇文章时，如果有不太懂的地方，大家也可以先看看博主上一篇文章，锁的这部分内容是面试中很常见的问题，多学学对自己是非常有帮助的。同时，朋友们如果有什么问题都可以随时和我探讨，大家一起进步！

synchronized原理
一. 特性
二. 加锁过程（锁升级/锁膨胀）
1. 无锁状态
2. 偏向锁
3. 轻量级锁
4. 重量级锁
5. 总结
三. 锁优化
1. 锁消除
2. 锁粗化
一. 特性
这部分内容在上篇文章中的 synchronized充当了哪些锁部分已经介绍过了哦，没有看的小伙伴可以去看看synchronized的特性

二. 加锁过程（锁升级/锁膨胀）
在Java中JVM虚拟机将synchronized锁分为无锁、偏向锁、轻量级锁、重量级锁状态。会根据不同的情况，进行不同的升级操作


1. 无锁状态
此状态理解起来较为简单，没有进行线程任务时最开始的状态就是无锁状态。

2. 偏向锁
偏向锁类似于一种乐观锁，当一个线程在执行任务时，偏向锁会给这个线程设定一个标记（并不是真正地加锁），如果后续没有其他线程来竞争这个锁，那么这个偏向锁就不会再进行其他的任何操作了，有效避免了因为加锁过程而产生的内存开销问题
若有其他线程也竞争这把锁，那么此时第一个线程会立马把锁拿到（因为之前第一个线程已经有了偏向锁标记，所以很容易拿到）然后进入轻量级锁的状态
偏向锁的大体思路是能不加锁就尽量不加锁避免内存开销，只做上标记即可，但如果实在要加锁，也会因为标记的存在而立马把锁拿到（类似于高考填志愿保底心态）

3. 轻量级锁
当进入轻量级锁锁状态（自适应自旋锁）后，是完全在用户态上实现的，且是基于CAS来完成的操作，因为这个状态不涉及到内核态和用户态的切换，也不涉及到线程的阻塞和调度过程。所以并不会对系统的内存有着过于高的开销，因此可以保证更高效地获取到锁（一个线程释放锁后，另一个线程会马上获取到锁）
具体步骤

通过 CAS 检查并更新一块内存 (比如 null => 该线程引用)
如果更新成功, 则认为加锁成功
如果更新失败, 则认为锁被占用, 继续自旋式的等待(并不放弃 CPU)
由于自旋操作可能会一直让CPU 空转，比较浪费 CPU 资源，因此此处的自旋不会一直持续进行，而是达到一定的时间（重试）次数，就不再自旋了，也就是所谓的 “自适应”（根据情况来）
4. 重量级锁
当锁的竞争变得非常激烈时，如果再按照之前自旋的方式，那么对于CPU的开销是非常高的，而且此时自旋还不能快速地获取到锁的状态，那么此时就会变成重量级锁（挂起等待锁），对于挂起等待锁来说，锁的等待过程是释放CPU的过程，此时会节省CPU的开销，但付出的代价是引入了线程的阻塞和调度的开销（以CPU资源换取性能）
具体过程
此处的重量级锁就是用到了内核提供的mutex，要执行加锁操作，首先会进入内核态，在内核态判定当前的锁是否已经被占用，若该锁没有被占用，则加锁成功，切换回用户态；若该锁被占用，则加锁失败，此时线程进入锁的等待队列去挂起等待，直到锁被其他线程释放后，操作系统才会唤醒挂起等待锁的这个线程，最后这个线程才会获取到锁

5. 总结
锁升级（锁膨胀）的过程完全是synchronized内部自适应完成的，即根据不同的情况（即锁冲突的高或低状态）来升级或降级成对应的状态，不需要用户或者程序员去干预，因此使用起来会比较方便。
2.注意， synchronized在有些JVM版本上是可以同时实现降级和升级的自适应的，但在有些JVM上只能实现升级的自适应。
三. 锁优化
1. 锁消除
JVM和编译器提供了一个很好的功能，能判断某段代码是否有加锁的必要（根据情况选择是否需要加锁），以防开发人员加错锁而造成无缘无故开销很大系统内存的情况。
例子
Java提供了两个类，StringBuilder和StringBuffer，其中，前者不会考虑线程安全问题，而后者中的每个方法都带有了synchronized以确保线程安全。但我们平常在单线程下，这个加锁是没有必要的，会白白浪费很多内存资源，这时候，如果开发人员不小心在单线程中使用了StringBuffer，那么编译器和JVM也会对其进行一定的优化，去把这个锁消除。
注意，编译器的判断也不是每次完全都是正确的，不会每次都会锁消除，因此，还是要提醒大家在写代码过程中自己还是要尽量避免出现此类错误

2. 锁粗化
介绍锁粗化之前，首先大家得知道锁的粗细的含义：

如果synchronized代码块中包含的代码比较多，则认为锁比较粗
如果synchronized代码块中包含的代码比较少，则认为锁比较细
那么实际开发中，到底是锁粗一点好还是细一点好呢？这个还是根据情况来决定的，虽然当锁里面的代码量少时会减少线程之间的锁冲突概率，但是有的情况下，反而当锁里面的代码量较多时，运行效率才会更高：

void func(){
        synchronized (this){
            //任务1
        }
        synchronized (this){
            //任务2
        }
        synchronized (this){
            //任务3
        }
    }

大家先来看一看这段代码，这段代码是细锁的情况（一个锁中的代码量较少），那么其执行效率怎么样呢？显然是不太高的，每次加一次锁都会进行一定的内存开销，因此我们很有必要对其进行一定地改进，让锁粗化：

void func(){
        synchronized (this){
            //任务1
            //任务2
            //任务3
        }
    }
可以看出来，当锁粗化的时候，会大大提高代码的执行效率
就好比出门买物品A和物品B，我们通常会尽可能地出一次门，将二者一块买好再回家，而不是先把物品A买好放到家里再出门买物品B，这样显然效率是非常低的，而且还很浪费体力