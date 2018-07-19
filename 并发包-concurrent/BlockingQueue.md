## BlockingQueue

BlockingQueue<E> 的唯一父接口是  Queue<E>，所以在BlockingQueue<E>  Java Doc文档一开始是：

> A [`Queue`](http://127.0.0.1:57801/Dash/loxudajq/java/util/Queue.html) that additionally supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available in the queue when storing an element.
>
> 额外支持了空队列取元素会等待（阻塞），满队列存元素会等待（阻塞）的队列

所以 BlockingQueue 的核心要点就是 Block。

#### 常用方法总结

​							BlockingQueue 方法总结

|             | *Throws exception* | *Special value* | *Blocks*         | *Times out* 【offer-false】 【poll-null】 |
| ----------- | ------------------ | --------------- | ---------------- | ----------------------------------------- |
| **Insert**  | [`add(e)`](~)      | [`offer(e)`](~) | [`put(e)`](~)    | [`offer(e, time, unit)`](~)               |
| **Remove**  | [`remove()`](~)    | [`poll()`](~)   | [`take()`](~)    | [`poll(time, unit)`](~)                   |
| **Examine** | [`element()`](~)   | [`peek()`](~)   | *not applicable* | *not applicable*                          |

其中前两列的方法，属于正常 Queue 中的方法，而后两列中的方法才是 BlockingQueue 扩展方法。其中 put(e)和 take() 当不满足条件时会无限时阻塞，而 *Times out* 列中的 offer(e, time, unit) 和  poll(time, unit) 也会阻塞但是会有时间限制，指定的 timeout 时间到了后，**offer 会因为入队失败返回 false，poll 因为取不到新元素返回 Null**

#### 解决的问题

在非阻塞队列中，多线程环境下的同步和线程休眠唤醒问题都是需要额外去考虑和处理的。而 BlockingQueue则，在内部解决了这些问题，屏蔽的这些容易出错和繁杂的细节。

#### Example

````java
class Producer implements Runnable {
    private final BlockingQueue<Object> queue;

    Producer(BlockingQueue q) {
        queue = q;
    }

    @Override
    public void run() {
        try {
            while (true) {
                queue.put(produce());
            }
        } catch (InterruptedException ex) {
            System.out.println("InterruptedException");
        }
    }

    Object produce() {
        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("produce a obj");
        return new Object();
    }
}

class Consumer implements Runnable {
    private final BlockingQueue<Object> queue;

    Consumer(BlockingQueue q) {
        queue = q;
    }

    @Override
    public void run() {
        try {
            while (true) {
                consume(queue.take());
            }
        } catch (InterruptedException ex) {
            System.out.println("InterruptedException");
        }
    }

    void consume(Object x) {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("consume a obj");
    }
}

public class Setup {
    public static void main(String[] args) {
        BlockingQueue<Object> q = new LinkedBlockingDeque<>();
        Producer p = new Producer(q);
        Consumer c = new Consumer(q);
        new Thread(c).start();
        new Thread(p).start();
    }
}

````

#### BlockingQueue 实现类

ArrayBlockingQueue,

DelayQueue

LinkedBlockingDeque, 

LinkedBlockingQueue, 

LinkedTransferQueue, 

PriorityBlockingQueue, 

SynchronousQueue