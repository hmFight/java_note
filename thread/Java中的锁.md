# Java中的锁

####参考文章

* [[并发编程网 - ifeve.com](http://ifeve.com/)Java锁的种类以及辨析](http://ifeve.com/java_lock_see/)

​        在并发环境下，为了保证数据操作的一致性，于是引进锁进制，用于保证临界区代码的安全。

#### 自旋锁

​	如果线程竞争不激烈（或者说参加竞争的线程数不多），而且**处理器阻塞一个线程引起的线程上下文的切换的代价高于等待资源的代价**的时候（锁的已保持者保持锁时间比较短），那么等待锁的线程可以不放弃CPU时间片，而是在原地”忙等"，直到锁的持有者释放了该锁，这就是自旋锁的原理，可见自旋锁是一种非阻塞锁。

​	自旋锁的实现

````java
public class SpinLock {

  private AtomicReference<Thread> sign =new AtomicReference<>();

  public void lock(){
    Thread current = Thread.currentThread();
    while(!sign .compareAndSet(null, current)){
    }
  }

  public void unlock (){
    Thread current = Thread.currentThread();
    sign.compareAndSet(current, null);
  }
}
````

​	自旋锁的问题：当自旋锁脱离的自己的适用范围时，第一 线程数多，竞争激烈，由于每个线程都会忙等占用CPU时间，会造成CPU负担; 第二如果锁的释放时间本来就不慢，那其实线程切换的代价不一定比等待的代价大，自旋锁的意义就不大。

#### 乐观锁

#### 悲观锁

