#Java 线程状态转换

> [Java线程状态分析](https://fangjian0423.github.io/2016/06/04/java-thread-state/)

## Thread 源码中线程状态

````java
/**
A thread state. A thread can be in one of the following states:
NEW
A thread that has not yet started is in this state.
RUNNABLE
A thread executing in the Java virtual machine is in this state.
BLOCKED
A thread that is blocked waiting for a monitor lock is in this state.
WAITING
A thread that is waiting indefinitely for another thread to perform a particular action is in this state.
TIMED_WAITING
A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state.
TERMINATED
A thread that has exited is in this state.
A thread can be in only one state at a given point in time. These states are virtual machine states which do not reflect any operating system thread states.
 */
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
````
## NEW

>Thread state for a thread which has not yet started.
>
>线程未启动时的状态

````java
@Test
public void testNew() throws Exception {
    MyThread myThread = new MyThread();
    Assert.assertEquals(Thread.State.NEW,myThread.getState());
}
````

## RUNNABLE

> Thread state for a runnable thread. A thread in the runnable state is executing in the Java virtual machine but it may be waiting for other resources from the operating system such as processor.
>
> 在运行状态的线程在java虚拟机执行，但它可能在等待操作系统的其他资源例如处理器。

````java
@Test
public void testRunable() throws Exception {
    MyThread myThread = new MyThread();
    myThread.start();
    Assert.assertEquals(Thread.State.RUNNABLE,myThread.getState());
}
````

## BLOCKED

> Thread state for a thread blocked waiting for a monitor lock. A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method or reenter a synchronized block/method after calling [`Object.wait`](file:///Users/wangjia/Library/Application%20Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/lang/Object.html#wait--).
>
>  处于阻塞状态的线程正在等待 monitor lock ，以便进入同步块/方法或者在调用 Object.wait 之后重新进入同步块/方法。

````java

/**
 * 两个线程，先被调度的线程进入 RUNNABLE ,后一个线程进入 BLOCKED
 */
@Test
public void testBlock() throws Exception {
    Thread thread1 = new Thread(() -> {
        synchronized (lock) {
            while (true) {
            }
        }
    });
    Thread thread2 = new Thread(() -> {
        synchronized (lock) {
            while (true) {
            }
        }
    });
    thread1.start();
    Thread.sleep(1000);
    thread2.start();
    Assert.assertEquals(Thread.State.RUNNABLE, thread1.getState());
    Assert.assertEquals(Thread.State.BLOCKED, thread2.getState());
}
````

## WAITING

> Thread state for a waiting thread. A thread is in the waiting state due to calling one of the following methods:
>
> - [`Object.wait`](file:///Users/wangjia/Library/Application%20Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/lang/Object.html#wait--) with no timeout
> - [`Thread.join`](file:///Users/wangjia/Library/Application%20Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/lang/Thread.html#join--) with no timeout
> - [`LockSupport.park`](file:///Users/wangjia/Library/Application%20Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/util/concurrent/locks/LockSupport.html#park--)
>
> A thread in the waiting state is waiting for another thread to perform a particular action. For example, a thread that has called `Object.wait()` on an object is waiting for another thread to call`Object.notify()` or `Object.notifyAll()` on that object. A thread that has called `Thread.join()` is waiting for a specified thread to terminate.