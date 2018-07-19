# Callable、Future

````
/**
 *
 * <p>The {@code Callable} interface is similar to {@link
 * java.lang.Runnable}, in that both are designed for classes whose
 * instances are potentially executed by another thread.  
 */
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
````
Callable 和 Runnable 很相似，一般都用于多线程情景，但是 Callable 可以返回结果并抛出异常
````java
public interface Future<V> {
	
    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

````

Future 中定义的接口不多，包括取消任务、查询任务取消或完成、获取结果这些功能



简单使用

异步计算求和操作，并获取计算结果

````java
//单任务
public class CallableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Callable<Integer> callable = new AsyncTask();
        Future<Integer> future = executorService.submit(callable);
        System.out.println("提交任务后,主线程执行其他操作……");
        Integer res = future.get();
        System.out.println("异步任务结果：" + res);
    }

}

//多任务
public class CallableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        List<AsyncTask> tasks = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            tasks.add(new AsyncTask());
        }
        List<Future<Integer>> futures = executor.invokeAll(tasks);
        executor.shutdown();
        for (Future<Integer> future : futures) {
            System.out.println(future.get());
        }
    }
}

class AsyncTask implements Callable<Integer> {
    /**
     * 一般是耗时IO操作等
     */
    @Override
    public Integer call() throws Exception {
        System.out.println("异步任务开始");
        Thread.sleep(3000);
        int sum = 0;
        for (int i = 0; i <= 100; i++) {
            sum = sum + i;
        }
        System.out.println("异步任务结束");
        return sum;
    }
}
````



FutureTask<V> implements  RunnableFuture<V>，RunnableFuture<V>实现了 Runnable 和 Future。

> 可取消的异步计算。 此类提供Future的基本实现，包括启动和取消计算的方法，查询计算是否完成的查询以及检索计算结果。 只有在计算完成后才能检索结果; 如果计算尚未完成，get方法将阻塞。 计算完成后，无法重新启动或取消计算（除非使用runAndReset（）调用计算）。
>
>  FutureTask可用于包装Callable或Runnable对象。 因为FutureTask实现了Runnable，所以可以将FutureTask提交给Executor执行。
>
> 除了作为独立类之外，此类还提供了在创建自定义任务类时可能有用的受保护功能。
>
> ——来自	Java Doc 

FutureTask 使用

````java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        FutureTask<Integer> asyncTaskFutureTask = new FutureTask<Integer>(new AsyncTask()) {
            @Override
            protected void done() {
                System.out.println("异步任务结束");
            }
        };
        executor.submit(asyncTaskFutureTask);
        executor.shutdown();
    }
````

