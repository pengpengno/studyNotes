---
title: 多线程场景下线程间 事件协同解决模型-Future
date: 2022-01-03 17:43:35.408
updated: 2022-04-27 16:35:26.337
url: /archives/多线程场景下线程间事件协同解决模型-future
categories: 
tags: 
---



# 异步协同Future模型
## 应用场景
多线程的场景下非阻塞的线程卡可用通过集成Thread 实现Runable，Callable等实现并发操作，但上述方案都是 无法立即获取线程返回值，为了更高效的解决线程间协同时候需要等待的问题， 可以采用Future的线程模型。
同时Future 使用的局限性 则表现无法表现出业务线程之间的依赖关系，当前同时高度的解耦，也便于应用于大部分的场景下
## Future模型图解
![Future模型图解](http://180.76.240.8:8090/upload/2022/01/Future%E6%A8%A1%E5%9E%8B%E5%9B%BE%E8%A7%A3-433427116c8d483581c57d0d11a500c1.png)

### RunnableFuture

        这个接口同时继承Future接口和Runnable接口，在成功执行run（）方法后，可以通过Future访问执行结果。这个接口都实现类是FutureTask,一个可取消的异步计算，这个类提供了Future的基本实现，后面我们的demo也是用这个类实现，它实现了启动和取消一个计算，查询这个计算是否已完成，恢复计算结果。计算的结果只能在计算已经完成的情况下恢复。如果计算没有完成，get方法会阻塞，一旦计算完成，这个计算将不能被重启和取消，除非调用runAndReset方法。

        FutureTask能用来包装一个Callable或Runnable对象，因为它实现了Runnable接口，而且它能被传递到Executor进行执行。为了提供单例类，这个类在创建自定义的工作类时提供了protected构造函数。

### SchedualFuture

        这个接口表示一个延时的行为可以被取消。通常一个安排好的future是定时任务SchedualedExecutorService的结果

### CompleteFuture

        一个Future类是显示的完成，而且能被用作一个完成等级，通过它的完成触发支持的依赖函数和行为。当两个或多个线程要执行完成或取消操作时，只有一个能够成功。

### ForkJoinTask

        基于任务的抽象类，可以通过ForkJoinPool来执行。一个ForkJoinTask是类似于线程实体，但是相对于线程实体是轻量级的。大量的任务和子任务会被ForkJoinPool池中的真实线程挂起来，以某些使用限制为代价。
##  Future的主要实现
```
/*
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

/*
 * This file is available under and governed by the GNU General Public
 * License version 2 only, as published by the Free Software Foundation.
 * However, the following notice accompanied the original version of this
 * file:
 *
 * Written by Doug Lea with assistance from members of JCP JSR-166
 * Expert Group and released to the public domain, as explained at
 * http://creativecommons.org/publicdomain/zero/1.0/
 */

package java.util.concurrent;

/**
 * A {@code Future} represents the result of an asynchronous
 * computation.  Methods are provided to check if the computation is
 * complete, to wait for its completion, and to retrieve the result of
 * the computation.  The result can only be retrieved using method
 * {@code get} when the computation has completed, blocking if
 * necessary until it is ready.  Cancellation is performed by the
 * {@code cancel} method.  Additional methods are provided to
 * determine if the task completed normally or was cancelled. Once a
 * computation has completed, the computation cannot be cancelled.
 * If you would like to use a {@code Future} for the sake
 * of cancellability but not provide a usable result, you can
 * declare types of the form {@code Future<?>} and
 * return {@code null} as a result of the underlying task.
 *
 * <p>
 * <b>Sample Usage</b> (Note that the following classes are all
 * made-up.)
 * <pre> {@code
 * interface ArchiveSearcher { String search(String target); }
 * class App {
 *   ExecutorService executor = ...
 *   ArchiveSearcher searcher = ...
 *   void showSearch(final String target)
 *       throws InterruptedException {
 *     Future<String> future
 *       = executor.submit(new Callable<String>() {
 *         public String call() {
 *             return searcher.search(target);
 *         }});
 *     displayOtherThings(); // do other things while searching
 *     try {
 *       displayText(future.get()); // use future
 *     } catch (ExecutionException ex) { cleanup(); return; }
 *   }
 * }}</pre>
 *
 * The {@link FutureTask} class is an implementation of {@code Future} that
 * implements {@code Runnable}, and so may be executed by an {@code Executor}.
 * For example, the above construction with {@code submit} could be replaced by:
 *  <pre> {@code
 * FutureTask<String> future =
 *   new FutureTask<String>(new Callable<String>() {
 *     public String call() {
 *       return searcher.search(target);
 *   }});
 * executor.execute(future);}</pre>
 *
 * <p>Memory consistency effects: Actions taken by the asynchronous computation
 * <a href="package-summary.html#MemoryVisibility"> <i>happen-before</i></a>
 * actions following the corresponding {@code Future.get()} in another thread.
 *
 * @see FutureTask
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> The result type returned by this Future's {@code get} method
 */
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```