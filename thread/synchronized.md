 # Synchronized

## 一、使用

### 1、修饰函数（同步方法）

实例方法

````java
//代码1，实例方法同步
public synchronized void method2() {
  //todo
}
````

静态方法

````java
//代码2
public synchronized static void method5() {
   //todo
}
````

### 2、代码块（同步代码块）

````java
//代码3
public void method6() {
   synchronized (this) {
      //todo
   }
}
````

````java
//代码4
private byte[] lock = new byte[0];

public void method7() {
    synchronized (lock) {
       //todo 
    }
}
````

````java
//代码5
public void method7() {
    synchronized (SomeClass.class) {
       //todo 
    }
}
````

## 二、More

### 1、两个概念

* 原子性：具有原子性的操作被称为原子操作。原子操作在操作完毕之前不会线程调度器中断。
* 可见性：多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

###2、Synchronized的关键

* 谁持有锁（锁持有者是对线程而言）
* 锁对象是谁（被锁对象）

## 三、实例

基础类

````java
package com.wj.thread.sync;

/**
 * @author : wangjia
 * @time : 2018/2/8 18:22
 */
public class ForSync {
    private final byte[] lock = new byte[0];

    public synchronized void syncMethod1() {
        System.out.println("syncMethod1 get lock");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("syncMethod1 free lock");
    }

    public synchronized void syncMethod2() {
        System.out.println("syncMethod2 get lock");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("syncMethod2 free lock");
    }

    public void method3() {
        System.out.println("method3");
    }

    public static void method4() {
        System.out.println("static method3");
    }

    public synchronized static void method5() {
        System.out.println("synchronized static method5() get lock");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("synchronized static method5() free lock");
    }

    public void method6() {
        synchronized (this) {
            System.out.println("synchronized(this) method6() get lock");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized(this) method6() free lock");
        }
    }

    public void method7() {
        synchronized (lock) {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized static method7");
        }
    }

    public synchronized void method8() {
        System.out.println("syncMethod8 get lock");
        this.syncMethod1();
        System.out.println("syncMethod get lock");
    }

    public void method9() {
        synchronized (ForSync.class) {
            System.out.println("synchronized(ForSync.class) method9 get lock");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized(ForSync.class) method9 free lock");
        }
    }

    public void method10() {
        synchronized (this.getClass()) {
            System.out.println("synchronized(getClass) method10 get lock");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized(getClass) method10 free lock");
        }
    }
}
````
### 1、多个线程，竞争同一对象锁

````java
/**
 * 多线程竞争容易一对象锁
 */
@Test
public void test1() throws Exception {
    ForSync forSync = new ForSync();
    new Thread(() -> forSync.syncMethod1()).start();
    new Thread(() -> forSync.syncMethod2()).start();
    Thread.sleep(5000);
}
/**输出
syncMethod1 get lock
syncMethod1 free lock
syncMethod2 get lock
syncMethod2 free lock
**/
````

第一个线程在 syncMethod1() 先获取锁，锁对象 forSync。第二个线程 syncMethod2() 也需要获取锁(forSync)，但是此时锁已经被持有了，必须等待持有者释放。因此，syncMethod2() 必须等待 syncMethod1() 执行完后才能执行。

两个方法肯定是一先一后顺序执行（其实也有可能是第二个线程先获取锁，先执行 syncMethod2()）。

### 2、多个线程分别持有不同对象锁

````java
/**
 * 多线程分别获取不同实例对象的锁
 */
@Test
public void test2() throws Exception {
    ForSync forSync1 = new ForSync();
    ForSync forSync2 = new ForSync();
    new Thread(() -> forSync1.syncMethod1()).start();
    new Thread(() -> forSync2.syncMethod2()).start();
    Thread.sleep(5000);
}
/**输出
syncMethod1 get lock
syncMethod2 get lock
syncMethod1 free lock
syncMethod2 free lock
**/
````

两个线程持有了两个不同对象的锁，因此互不干扰。

### 3、非 synchronized 方法和 synchronized 方法 互不影响

````java
/**
 * 非 synchronized 方法和 synchronized 方法 互不影响
 */
@Test
public void test3() throws Exception {
    ForSync forSync = new ForSync();
    new Thread(() -> forSync.syncMethod1()).start();
    new Thread(() -> forSync.method3()).start();
    Thread.sleep(5000);
}
/**
syncMethod1 get lock
method3
syncMethod1 free lock
**/
````

### 4、 synchronized (this) 和 synchronized (SomeClass.class)

````java
/**
 * synchronized (this) 和 synchronized (SomeClass.class)
 * 一个是实例对象锁 和 一个是 Class 锁
 */
@Test
public void test4() throws Exception {
    ForSync forSync = new ForSync();
    new Thread(() -> forSync.method6()).start();
    new Thread(() -> forSync.method10()).start();
    Thread.sleep(5000);
}
/**输出
synchronized(this) method6() get lock
synchronized(ForSync.class) method10 get lock
synchronized(ForSync.class) method10 free lock
synchronized(this) method6() free lock
**/
````

两个线程同时持有锁的对象是不同的，两个方法不会相互影响。

### 5、synchronized static 方法和 synchronized (SomeClass.class)

````java
/**
 * synchronized static 方法和 synchronized (SomeClass.class)
 */
@Test
public void test6() throws Exception {
    ForSync forSync = new ForSync();
    new Thread(() -> forSync.method5()).start();
    new Thread(() -> forSync.method9()).start();
    Thread.sleep(6000);
}
/**输出
synchronized static method5() get lock
synchronized static method5() free lock
synchronized(ForSync.class) method9 get lock
synchronized(ForSync.class) method9 free lock
**/
````

可以看到，两个线程产生了竞争，后面的线程必须等待前面的线程释放锁后才能执行。说明 synchronized 修饰在静态方法时，获取的锁是该类 Class , 与  synchronized (SomeClass.class）中需要获取的锁是同一个。

### 6、synchronized (this.getClass()) 和 synchronized (ForSync.class)

```java
/**
 * synchronized (this.getClass()) 和 synchronized (ForSync.class)
 *
 */
@Test
public void test8() throws Exception {
    ForSync forSync = new ForSync();
    new Thread(() -> forSync.method9()).start();
    new Thread(() -> forSync.method10()).start();
    Thread.sleep(6000);
}
/**
synchronized(ForSync.class) method9 get lock
synchronized(ForSync.class) method9 free lock
synchronized(ForSync.class) method10 get lock
synchronized(ForSync.class) method10 free lock
*/
```

可以看到，两种方式获取的是同一个锁。

但是再使用`synchronized (this.getClass())` 的时候，IDEA 会有这样一个提示

> If the class containing the synchronization is subclassed, the subclass will synchronize on a different class object. Usually the call to getClass() can be replaced with a class literal expression, for example String.class. An even better solution is synchronizing on a private static final lock object, access to which can be completely controlled.
>
> 如果包含同步的类是子类，那么子类将在不同的类对象上进行同步。 通常，对getClass（）的调用可以用一个类字面表达式替换，例如String.class。 **更好的解决方案是在一个私有静态最终锁定对象上进行同步，访问权限可以完全控制 ( private final byte[] lock = new byte[0]; )**

## 四、小结

* 用法：修饰方法(实例方法、静态方法)；代码块
* 哪个线程持有哪个锁对象