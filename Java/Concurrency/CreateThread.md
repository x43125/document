# Thread Creation

> Java中线程的几种创建方式



```java

import java.util.concurrent.*;

/**
 * @Descrption: 创建线程
 * @Author: x43125
 * @Date: 22/05/18
 */
public class CreateThread {
    static class ExtendThread extends Thread {
        @Override
        public void run() {
            System.out.println("1.直接继承Thread 创建线程 点进去可以发现Thread实现了Runnable接口" + currentThread().getName());
        }
    }

    static class ImplementRunnable implements Runnable {

        @Override
        public void run() {
            System.out.println("2.实现Runnable接口创建线程，需要依托Thread 或者线程池等创建线程 本身并非线程 " + Thread.currentThread().getName());
        }
    }

    static class ImplementCallable implements Callable<String> {

        @Override
        public String call() throws InterruptedException {
            System.out.println("3.实现Callable接口创建线程，可以通过FutureTask 接收返回值，" +
                    "调用接收方法即(FutureTask.get())方法的线程会被阻塞，直到获取到值 " +
                    "本身还需要依托Thread 并非线程 阻塞1秒..." + Thread.currentThread().getName());
            TimeUnit.SECONDS.sleep(1);
            return Thread.currentThread().getName();
        }
    }

    public static void main(String[] args) {
        // 1.继承Thread
        ExtendThread extendThread = new ExtendThread();
        extendThread.start();

        // 2.实现runnable
        ImplementRunnable runnable = new ImplementRunnable();
        new Thread(runnable).start();

        // 3.实现callable
        ImplementCallable callable = new ImplementCallable();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        new Thread(futureTask).start();
        try {
            System.out.println(" Callable的返回值 调用此方法的线程会被阻塞，直到得到线程的返回值" + futureTask.get() + "\n");
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }

        System.out.println("通过创建线程池来启动线程");
        System.out.println("线程池有四大主要参数：\n" +
                " corePoolSize: 线程池核心线程数，这部分线程创建之后会一直放在池子中供使用而不会回收。" +
                "当新请求到来，而当前活动线程已经等于核心线程数时，新的请求会被存放到阻塞队列中，等待线程释放\n" +
                " workQueue: 等待队列或阻塞队列，用于存放当核心线程已全在使用中时的新请求\n" +
                " maximumPoolSize: 最大线程数，池子中会同时存在的线程最大的数量。" +
                "当活动线程数已经达到核心线程数，且等待队列也被存满了的时候，又有新请求到来，这部分的请求将继续创建线程，直到活动线程数达到最大线程数。" +
                "当池子中活动线程数等于最大线程数，且又有新的请求到来，则会触发拒绝策略\n" +
                " rejectedExecutionHandler: 拒绝策略，即当活动线程数等于最大线程数，又有新的请求到来时将触发此拒绝策略");

        System.out.println("自定义线程池: ");
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                2,
                3,
                30,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2),
                // 四种拒绝策略：（默认是1.AbortPolicy）
                // 1.AbortPolicy: 直接抛异常，阻塞线程的运行
                // 2.CallerRunsPolicy: 让主线程来执行多出来的请求
                // 3.DiscardOldestPolicy: 抛弃掉等待队列中最早加进来且当前还未执行的请求
                // 4.DiscardPolicy: 直接抛弃新请求，不执行
                // ===================================================== 可逐次打开一行如下注释来观察执行结果的不同
//                 new ThreadPoolExecutor.AbortPolicy()
                new ThreadPoolExecutor.CallerRunsPolicy()
                // new ThreadPoolExecutor.DiscardOldestPolicy()
//                new ThreadPoolExecutor.DiscardPolicy()
        );
        for (int i = 0; i < 10; i++) {
            // execute 只可以提交runnable，无返回值
            int finalI = i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() + " " + finalI);
            });
            // submit 既可以提交runnable又可以提交callable，并且可以返回callable的返回值
//            executor.submit(runnable);
        }

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("\n 4种官方自带线程池：");
        // CachedThreadPool: maximumPoolSize = Integer.MAX_VALUE 即无线程上限，只要有请求就会一直创建线程
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        cachedThreadPool.execute(runnable);
        // FixedThreadPool: corePoolSize = maximumPoolSize = nThreads && workQueue = new LinkedBlockingQueue()
        // 核心线程数与最大线程数相同，即固定线程数，多的请求进入阻塞队列
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        executorService.execute(runnable);
        // todo DelayedWorkQueue
        // ScheduledThreadPool: corePoolSize = nThreads && maximumPoolSize = Integer.MAX_VALUE && workQueue = new DelayedWorkQueue()
        // 无线程上线，但阻塞队列使用了DelayedWorkQueue
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
        scheduledExecutorService.execute(runnable);
        // SingleThreadExecutor: corePoolSize = maximumPoolSize = 1 && workQueue = new LinkedBlockingQueue()
        // 单线程线程池，面试常问：单线程线程池的意义或使用场景是什么
        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        singleThreadExecutor.execute(runnable);

        // 线程池用完需要关闭， shutdown 方法会等待线程池中的任务跑完才关闭 shutdownNow会立刻打断任务，直接关闭
        executor.shutdown();
        cachedThreadPool.shutdown();
        scheduledExecutorService.shutdown();
        executorService.shutdown();
        singleThreadExecutor.shutdown();

//        ThreadFactory
    }
}
```