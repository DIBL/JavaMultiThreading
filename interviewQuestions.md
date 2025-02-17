#### 1. Multi-thread
- Two ways to create thread
	- extends from Thread class
	- implement runnable interface (prefer)
- `yield()` and `sleeping()`
	- `yield()` changes thread from running state to runnable state
	- `sleeping()` changes thread from running state to waiting state
- Threads can communicate with each other by `wait()`, `notify()`, `notifyAll()` method.
Used in user interface, like when you open the interface, it takes a long time to open the whole page, in the same time you can click your mouse on some button to do something.

#### 2. Synchronized type?
synchronized methods and synchronized block.

#### 3. What's volatile?
Essentially, volatile is used to indicate that a variable's value will be modified by different threads.

#### 4. Daemon Thread.
- The daemon threads are basically the low priority threads that provides the background support to the user threads. It provides services to the user threads.
- A daemon thread is a thread, that does not prevent the JVM from exiting when the program finishes but the thread is still running. An example for a daemon thread is the garbage collection.
- You can use the `setDaemon()` method to change the Thread daemon properties.

#### 5. What’s difference between `notify()` and `notifyAll()`? [1]
- First and main difference between `notify()` and `notifyAll()` method is that, if multiple threads is waiting on any lock in Java, notify method send notification to only one of waiting thread while notifyAll informs all threads waiting on that lock.

- If you use notify method , It's not guaranteed that, which thread will be informed, but if you use notifyAll since all thread will be notified, they will compete for lock and the lucky thread which gets lock will continue. In a way, **notifyAll method is safer because it sends notification to all threads**, so if any thread misses the notification, there are other threads to do the job, while in the case of `notify()` method if the notified thread misses the notification then it could create subtle, hard to debug issues. 

- Some people argue that using notifyAll can drain more CPU cycles than notify itself but if you really want to sure that your notification doesn't get wasted by any reason, use notifyAll. Since wait method is called from the loop and they check condition even after waking up, calling notifyAll won't lead any side effect, instead it ensures that notification is not dropped.

- Prefer notifyAll over notify whenever in doubt and if you can, avoid using notify and notifyAll altogether, instead use concurrency utility like `CountDownLatch`, `CyclicBarrier`, and `Semaphore` to write your concurrency code. 

#### 6. What’s the states of a thread? [2]
In Java, to get the current state of the thread, use `Thread.getState()` method to get the current state of the thread. Java provides `java.lang.Thread.State` class that defines the `ENUM` constants for the state of a thread. A thread can be in one of the following states:
- **NEW**: A thread that has not yet started is in this state.
- **RUNNABLE**: A thread executing in the Java virtual machine is in this state.
- **BLOCKED**: A thread that is blocked waiting for a monitor lock is in this state.
- **WAITING**: A thread that is waiting indefinitely for another thread to perform a particular action is in this state.
- **TIMED_WAITING**: A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state.
- **TERMINATED**: A thread that has exited is in this state.
![Alt text](https://media.geeksforgeeks.org/wp-content/uploads/threadLifeCycle.jpg)

#### 7. What is difference between forking a process and spawning a thread? 
- Forking processes: new process will run same code as parent process but in different memory space
- Spawning thread: creates another independent path of execution but share same memory space

#### 8. What is the relationship between threads and processes? 
- Thread is subset of Process, in other words one process can contain multiple threads. 
- Two process runs on different memory space, but all threads share same memory space.
 
#### 9. You have thread T1, T2 and T3, how will you ensure that thread T2 run after T1 and thread T3 run after T2?
- `t.join()` instructs the current thread - the one that call `t.join()` - to wait until the `t` thread completes its job before continuing with the statements that follows.
- Use `join()` method, T3 calls T2. join, and T2 calls T1.join, this ways T1 will finish first and T3 will finish last. 

#### 10. What is the advantage of new Lock interface over synchronized block in Java? You need to implement a high performance cache which allows multiple reader but single writer to keep the integrity how will you implement it? [3]
- Lock implementations provide more extensive locking operations than can be obtained using synchronized methods and statements. They allow more flexible structuring, may have quite different properties, and may support multiple associated Condition objects (`lock.newCondition()`). [4]
- A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource: only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first. However, some locks may allow concurrent access to a shared resource, such as the read lock of a ReadWriteLock. [4]
- A ReadWriteLock maintains a pair of associated locks, one for read-only operations and one for writing. The read lock may be held simultaneously by multiple reader threads, so long as there are no writers. The write lock is exclusive. [7]
- The major advantage of readwritelock is it provides two separate lock for reading and writing which enables you to write high performance data structure like ConcurrentHashMap and conditional blocking. 
- When you execute the following code sample you'll notice that both read tasks have to wait the whole second until the write task has finished. After the write lock has been released both read tasks are executed in parallel and print the result simultaneously to the console. They don't have to wait for each other to finish because read-locks can safely be acquired concurrently as long as no write-lock is held by another thread.

```
ExecutorService executor = Executors.newFixedThreadPool(2);
Map<String, String> map = new HashMap<>();
ReadWriteLock lock = new ReentrantReadWriteLock();

executor.submit(() -> {
    lock.writeLock().lock();
    try {
        sleep(1);
        map.put("foo", "bar");
    } finally {
        lock.writeLock().unlock();
    }
});

Runnable readTask = () -> {
    lock.readLock().lock();
    try {
        System.out.println(map.get("foo"));
        sleep(1);
    } finally {
        lock.readLock().unlock();
    }
};

executor.submit(readTask);
executor.submit(readTask);

stop(executor);
```

#### 11. What are differences between wait and sleep method in java?
- `wait()` 
	- release the lock or monitor
	- wait is used for inter-thread communication
	- `java.lang.Object`类中，提供了`wait()`
- `sleep()` 
	- doesn't release any lock or monitor while waiting
  	- sleep is used to introduce pause on execution.
	- `java.lang.Thread`类中，提供了`sleep()`
	- sleep need try catch interruption

#### 12. What is difference between CyclicBarriar  and CountdownLatch in Java ?
- CountdownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
A CountDownLatch is initialized with a given count. The await methods block until the current count reaches zero due to invocations of the countDown() method, after which all waiting threads are released and any subsequent invocations of await return immediately. This is a one-shot phenomenon -- the count cannot be reset. If you need a version that resets the count, consider using a CyclicBarrier.

- One difference is that you can reuse CyclicBarrier once barrier is broken but you can not reuse CountdownLatch.
 Though both CyclicBarrier and CountDownLatch wait for number of threads on one or more events, main difference between them is that you can not re-use CountDownLatch once count reaches to zero, but you can reuse same CyclicBarrier even after barrier is broken. 

#### 13. Difference between Runnable and Callable in Java?
both Runnable and Callable represent task which is intended to be executed in separate thread. Runnable is there from JDK 1.0, while Callable was added on JDK 1.5. Main difference between these two is that 
Callable's call() method can return value and throw Exception, which was not possible with Runnable's run() method

#### 14. How to stop thread in Java?
There was some control methods in JDK 1.0 e.g. `stop()`, `suspend()` and `resume()` which was deprecated in later releases due to potential deadlock threats,. To manually stop, programmers either take advantage of volatile boolean variable and check in every iteration if run method has loops or interrupt threads to abruptly cancel tasks

#### 15.How do you share data between two thread in Java?
You can share data between threads by using shared object, or concurrent data-structure like BlockingQueue. 

#### 16. Why we call `start()` method which in turns calls `run() method`, why not we directly call `run()` method ?
Another classic java multi-threading interview question This was my original doubt when I started programming in thread. Now days mostly asked in phone interview or first round of interview at mid and junior level java interviews. Answer to this question is that, when you call start() method it creates new Thread and execute code declared in run() while directly calling run() method doesn’t create any new thread and execute code on same calling thread.

#### 17. How will you awake a blocked thread in java?
This is tricky question on threading, blocking can result on many ways, if thread is blocked on IO then I don't think there is a way to interrupt the thread, let me know if there is any, on the other hand if thread is blocked due to result of calling `wait()`, `sleep()` or `join()` method you can interrupt the thread and it will awake by throwing InterruptedException. 

#### 18. What is FutureTask in Java? 
FutureTask represents a cancellable asynchronous computation in concurrent Java application. This class provides a base implementation of Future, with methods to start and cancel a computation, query to see if the computation is complete, and retrieve the result of the computation. The result can only be retrieved when the computation has completed; the get methods will block if the computation has not yet completed. A FutureTask object can be used to wrap a Callable or Runnable object. Since FutureTask also implements Runnable, it can be submitted to an Executor for execution.

#### 19. What is thread pool? Why should you use thread pool in Java?
Creating thread is expensive in terms of time and resource. If you create thread at time of request processing it will slow down your response time, also there is only a limited number of threads a process can create. To avoid both of these issues, a pool of thread is created when application starts-up and threads are reused for request processing. This pool of thread is known as "thread pool" and threads are known as worker thread. From JDK 1.5 release, Java API provides Executor framework, which allows you to create different types of thread pools e.g. single thread pool, which process one task at a time, fixed thread pool (a pool of fixed number of threads) or cached thread pool (an expandable thread pool suitable for applications with many short lived tasks). See this article to learn more about thread pools in Java to prepare detailed answer of this question.

#### 20. How do you check if a Thread holds a lock or not?
There is a method called `Thread.holdsLock(Object obj)` on `java.lang.Thread`, it returns true if and only if the current thread holds the monitor lock on the specified object.

#### 21. What happens if you submit a task when the queue of the thread pool is already filled? 
ThreadPoolExecutor's `submit()` method throws `RejectedExecutionException` if the task cannot be scheduled for execution.

#### 22. What is the difference between the `submit()` and `execute()` method thread pool in Java? 
> Thread is subset of Process, in other words one process can contain multiple threads. Two process runs on different memory space, but all threads share same memory space. [5]

A main difference between the `submit()` and `execute()` method is that `ExecuterService.submit()` can return result of computation because it has a return type of `Future`, but `execute()` method cannot return anything because it's return type is void. 

Both methods are ways to submit a task to thread pools but there is a slight difference between them. `execute(Runnable command)` is defined in `Executor` interface and executes given task in future, but more importantly, it does not return anything. Its return type is void. 

On other hand `submit()` is an overloaded method, it can take either `Runnable` or `Callable` task and can return `Future` object which can hold the pending result of computation. This method is defined on `ExecutorService` interface, which extends `Executor` interface, and every other thread pool class e.g. `ThreadPoolExecutor` or `ScheduledThreadPoolExecutor` gets these methods.

#### 23. Is it possible to start a thread twice?
No, there is no possibility to start a thread twice. If we does, it throws an exception.
> It is never legal to start a thread more than once. In particular, a thread may not be restarted once it has completed execution.
> Throws:
> IllegalThreadStateException - if the thread was already started. [6]

#### 24. What is static synchronization?
If you make any static method as synchronized, the lock will be on the class not on object. 

#### 25. Thread Communication Methods
- `wait()`
- `sleep()`
- `notify()`
- `notifyAll()`

#### 26. Use Executor create Thread Pool
```
public class TestThreadPool {  
     public static void main(String[] args) {  
        ExecutorService executor = Executors.newFixedThreadPool(5);//creating a pool of 5 threads  
        for (int i = 0; i < 10; i++) {  
            Runnable worker = new WorkerThread("" + i);  
            executor.execute(worker);//calling execute method of ExecutorService  
          }  
        executor.shutdown();  
        while (!executor.isTerminated()) {   }    
        System.out.println("Finished all threads");  
    }  
 }
 ```
 
 #### 27. Optimistic Locking using StampedLock
- In contrast to normal read locks an optimistic lock doesn't prevent other threads to obtain a write lock instantaneously. After sending the first thread to sleep for one second the second thread obtains a write lock without waiting for the optimistic read lock to be released. From this point the optimistic read lock is no longer valid. Even when the write lock is released the optimistic read locks stays invalid.

- So when working with optimistic locks you have to validate the lock every time after accessing any shared mutable variable to make sure the read was still valid.


```
ExecutorService executor = Executors.newFixedThreadPool(2);
StampedLock lock = new StampedLock();

executor.submit(() -> {
    long stamp = lock.tryOptimisticRead();
    try {
        System.out.println("Optimistic Lock Valid: " + lock.validate(stamp));
        sleep(1);
        System.out.println("Optimistic Lock Valid: " + lock.validate(stamp));
        sleep(2);
        System.out.println("Optimistic Lock Valid: " + lock.validate(stamp));
    } finally {
        lock.unlock(stamp);
    }
});

executor.submit(() -> {
    long stamp = lock.writeLock();
    try {
        System.out.println("Write Lock acquired");
        sleep(2);
    } finally {
        lock.unlock(stamp);
        System.out.println("Write done");
    }
});

stop(executor);
```
 
[1]: https://www.java67.com/2013/03/difference-between-wait-vs-notify-vs-notifyAll-java-thread.html
[2]: https://www.geeksforgeeks.org/lifecycle-and-states-of-a-thread-in-java/
[3]: https://winterbe.com/posts/2015/04/30/java8-concurrency-tutorial-synchronized-locks-examples/
[4]: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Lock.html
[5]: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html#submit(java.lang.Runnable)
[6]: https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html#start()
[7]: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/ReadWriteLock.html
