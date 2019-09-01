# 「Java并行」Callable 、Future、FutureTask、和线程池

## 为什么需要Callable？

有两种创建多线程的方式：一个是继承Thread类，另一个是实现Runnable接口。但是这样的一个问题是实现Runnable的run()方法后并不能返回任何结果，因为run()是一个void类型。为了弥补这个缺点，Callable在Java中诞生了。

## Callable vs Runnable
* Runnable不能返回任何结果，Callable可以返回结果。需要注意的是，并不能直接使用Callable创建线程，只有Runnable可以。
* Callable运行过程中可以抛出异常，而Runnable不能。
Callable的call的函数签名如下：

    ``` java
    public Object call() throws Exception;
    ```
## Future

Future是用来保存线程返回结果的——它并不能直接获取多线程的结果，而是在【未来】线程执行结束。Future是在主线程跟踪其他线程的进度和结果。

## FutureTask

创建一个线程Runnable是必须的，要保存线程的返回结果Future是必须的，因此有了FutureTask，FutureTask同时实现了Runnable和Future接口。


FutureTask对象可以通过传入Callable对象的构造器创建对象，然后FutureTask对象中也有包含要创建的线程对象。因此间接地可以使用Callable创建一个线程。再次强调Callable并不能直接创建线程对象。

要给完成的示例：

``` java
class CallableExample implements Callable 
{ 
  
  public Object call() throws Exception 
  { 
    Random generator = new Random(); 
    Integer randomNumber = generator.nextInt(5); 
    Thread.sleep(randomNumber * 1000); 
    return randomNumber; 
  } 
  
} 
  
public class CallableFutureTest 
{ 
  public static void main(String[] args) throws Exception 
  { 
  
    // FutureTask 是要给实现了Runnable和Future接口的实现类
    FutureTask[] randomNumberTasks = new FutureTask[5]; 
  
    for (int i = 0; i < 5; i++) 
    { 
      Callable callable = new CallableExample(); 
  
      //通过Callable创建FutureTask
      randomNumberTasks[i] = new FutureTask(callable); 
  
      // 因为FutureTask实现了Runnable接口, 因此可以通过 FutureTask创建一个线程对象。 
      Thread t = new Thread(randomNumberTasks[i]); 
      t.start(); 
    } 
  
    for (int i = 0; i < 5; i++) 
    { 
      //当线程结束可以通过get()方法获取线程的运行结果。
      System.out.println(randomNumberTasks[i].get()); 
      //get方法在没有获得结果的时候是阻塞状态的。
    } 
  } 
} 

```

## FutureTask搭配线程池使用

![](images/「Java并行」Callable&#32;、Future、FutureTask、和线程池/pool2.png)

``` java
MyRunnable myrunnableobject1 = new MyRunnable(1000); 
MyRunnable myrunnableobject2 = new MyRunnable(2000); 
  
FutureTask<String> futureTask1 = new FutureTask<>(myrunnableobject1, "FutureTask1 is complete"); 

FutureTask<String> futureTask2 = new FutureTask<>(myrunnableobject2, "FutureTask2 is complete"); 
  
//为ExecutorService创建一个大小是2的线程池 
ExecutorService executor = Executors.newFixedThreadPool(2); 
  
//提交futureTask1给ExecutorService 
executor.submit(futureTask1); 
  
//提交futureTask2给ExecutorService 
executor.submit(futureTask2); 
```





