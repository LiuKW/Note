## Java并发工具类

### CountDownLatch

> **CountDownLatch的作用**
>
> * 让一个线程或多个线程等待其他线程各自执行完毕后再执行。
>
>   ​	
>
> CountDownLatch通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。
>
> ​				
>
> ```java
> /**
> 	假设一个使用场景，一个程序有很多线程，不同线程执行自己的任务，
> 	要求等所有线程执行完毕之后提示一句已完成；
>  */
> public static void main(String[] args) throws InterruptedException {
>     CountDownLatch countDownLatch = new CountDownLatch(10);
>     for(int i = 0; i < 10; i++) {
>         new Thread(()->{
>             System.out.println(Thread.currentThread().getName() + "执行完了");
>             countDownLatch.countDown();
>         }, "" + i).start();
>     }
>     countDownLatch.await();
>     System.out.println("所有任务都已执行完毕");
> }
> ```
>
> ​					
>
> **重要方法**
>
> * `await()`：调用await方法的线程会被挂起，它会等待直到count值为0才继续执行
>
> * `await(long timeout, TimeUnit unit)`：和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
>
> * `countDown()`：将计数值count减一

​			

​			

​			

### CyclicBarrier

> **CyclicBarrier的作用**
>
> * 让一组线程到达一个屏障（同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。barrier是屏障，栅栏的意思，也可以理解为线程到达栅栏
>
> * CyclicBarrier多用于多线程计算数据，最后合并计算结果的场景。或者一个线程组的线程需要等待所有线程完成任务后再继续执行下一次任务
>
>   ​			
>
> **两个构造方法**
>
> * `CyclicBarrier(int parties)`：parties是参与线程的个数；
>
> * `CyclicBarrier(int parties, Runnable barrierAction)`：barrierAction是最后一个到达线程要做的任务
>
>   ​			
>
> **重要方法**
>
> * `await()`：线程调用await()，表示自己已经到达屏障
>
> * `await(long timeout, TimeUnit unit)`：这个方法会抛出一个BrokenBarrierException，表示屏障已经被破坏，破坏的原因可能是其中一个线程 await() 时被中断或者超时
>
> * `reset()`：将屏障重置为初始状态
>
>   
>
> ```java
> static int a = 0;
> static int b = 0;
> public static void main(String[] args) {
>     CyclicBarrier c = new CyclicBarrier(3, ()->{
>         System.out.println(
>             Thread.currentThread().getName() + 
>             "是最后一个执行的线程，最终的结果为：" + (a + b));
>     });
> 
>     for (int i = 0; i < 2; i++) {
>         new Thread(new Runnable() {
>             @Override
>             public void run() {
>                 if(Thread.currentThread().getName().equals("0"))
>                     for (int j = 0; j < 10; j++) a++;
> 
>                 if(Thread.currentThread().getName().equals("1"))
>                     for (int j = 0; j < 20; j++) b++;
> 
>                 try{
>                     c.await();
>                 } catch(Exception e) {
>                     e.printStackTrace();
>                 }
>             }
>         }, ""+ i).start();
>     }
> 
>     try{
>         TimeUnit.SECONDS.sleep(1);
>         c.await();
>     } catch(Exception e) {}
> }
> ```
>
> ​			
>
> **CyclicBarrier 和 CountDownLatch 的区别**
>
> * CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用 reset() 方法重置，所以 CyclicBarrier 可以处理更为复杂的场景。例如，计算发生错误，可以重新计算
> * CyclicBarrier 还提供了其他方法，getNumberWaitting() 可以获得 CyclicBarrier 阻塞的线程数量。isBroken() 可以用来获得阻塞的线程是否被中断
> * CountDownLatch允许一个或多个线程等待一组事件的产生，而CyclicBarrier用于等待其他线程运行到屏障

​			

​			

​			

### Semphore

> **Semphore信号量**
>
> 用于控制同时访问特定资源的线程数量，通过协调各个线程以保证合理的使用公共资源
>
> ​				
>
> **应用场景**
>
> 用于做流量控制，例如：控制数据库连接数
>
> ​				
>
> **重要的方法**
>
> * `acquire()`：从此信号量获取许可证，如果此时没有许可证，则被阻塞，直到获取到许可证为止
>
> * `tryAcquire()`：立即获得许可证，不会阻塞线程。如果获取成功，返回true；否则返回false
>
> * `release()`：归还许可证
>
>   ​		
>
> ```java
> private static final int THREAD_CNT = 30;
> public static void main(String[] args) throws Exception {
>     ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_CNT);
>     Semaphore s = new Semaphore(10);  // 最多允许是个线程访问资源
>     long start = System.currentTimeMillis();
>     CyclicBarrier cyclicBarrier = new CyclicBarrier(THREAD_CNT, ()->{
>         System.out.println((System.currentTimeMillis() - start) / 1000.0);
>     });
>     for (int i = 0; i < THREAD_CNT; i++) {
>         threadPool.execute(()->{
>             try{
>                 s.acquire(); //获取许可证
>                 TimeUnit.SECONDS.sleep(1);
>                 System.out.println(Thread.currentThread().getName());
>                 s.release();  //归还许可证
>                 cyclicBarrier.await();
>             } catch(Exception e) {}
>         });
>     }
>     threadPool.shutdown();
> }
> 
> 
> // 代码最终输出的是3秒钟，因为限制了最多只能10个线程并发执行。
> // 所以超过10个之后的线程需要等待之前的线程执行完毕才有可能拿到许可证
> // 如果把和Semphore相关的代码注释掉，最终耗时只有1s
> ```

​				

​				

​				

### Exchanger

> **Exchanger(交换者)**
>
> 用于进行线程间的数据交换。它提供了一个同步点，在这个同步点两个线程可以交换彼此的数据。
>
> ​					
>
> ```java
> public static void main(String[] args) {
>     Exchanger<String> exchanger = new Exchanger<>();
>     ExecutorService threadPool = Executors.newFixedThreadPool(2);
> 
>     threadPool.execute(new Runnable() {
>         @Override
>         public void run() {
>             String a = "我是线程A";
>             try {
>                 String exchange = exchanger.exchange(a);
>                 System.out.println(a + " == "  + exchange);
>             } catch (InterruptedException e) {
>                 e.printStackTrace();
>             }
>         }
>     });
> 
>     threadPool.execute(new Runnable() {
>         @Override
>         public void run() {
>             String b = "我是线程B";
>             try {
>                 String exchange = exchanger.exchange(b);
>                 System.out.println(b + " == " +exchange);
>             } catch (InterruptedException e) {
>                 e.printStackTrace();
>             }
>         }
>     });
> 
>     threadPool.shutdown();
> }
> ```