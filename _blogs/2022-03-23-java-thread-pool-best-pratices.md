---
title: "Java thread pool best practices"
author: yiqi
permalink: /blog/2022-03-23-java-thread-pool
date: 2022-03-23
collection: blog
tag:
- Programming
- Java
---

Setting up a Java thread pool looks easy. One can just use static methods within [```Executors```](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executors.html) to create either single or multiple-thread pools. This approach may be good enough on course projects. When it comes to an industrial web service with potentially hunderds or even thousands of asynchronous job, we need to be more careful.  

# Set up thread pool with limited queue  

When using ```Executors``` APIs to create thread pools, an unbounded queue is used by default. For example, ```Executors.newFixedThreadPool(N)``` gives us a thread pool with at most ```N``` active threads processing at the same time. Additional tasks are added to the queue. Think of a scenario where tasks are constantly submitted at a higher rate than they can be processed. Pending tasks in the queue will keep growing and eventually consumes all available memory.  

One can provide a ```BlockingQueue``` object with limited capacity (say ```M```), so that at most M tasks can be pending in the queue. When the queue is full, additional task submitting receives ```RejectedExecutionException```, which should be caught and handled. (log an error, emit mornitoring metrics, fire an alarm, etc.)  

**BAD practices**

```java
ExecutorService threadPool = Executors.newFixedThreadPool(10);
threadPool.submit(xxx);
```

**GOOD practices**

```java
BlockingQueue<Runnable> workerQueue = new LinkedBlockingQueue<>(100);
ExecutorService threadPool = new ThreadPoolExecutor​(10, 10, 0L, TimeUnit.SECONDS, workerQueue);

try {
    threadPool.submit(xxx);
} catch (RejectedExecutionException e) {
    // handle exception
}
```

# Using thread pool factory  

Let's have the code first:

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
    .setDaemon(false)
    .setNameFormat("JavaThreadPoolDemo")
    .build();

BlockingQueue<Runnable> workerQueue = new LinkedBlockingQueue<>(100);
ExecutorService threadPool = new ThreadPoolExecutor​(10, 10, 0L, TimeUnit.SECONDS, workerQueue, threadFactory);

try {
    threadPool.submit(xxx);
} catch (RejectedExecutionException e) {
    // handle exception
}
```

Here we use [```ThreadFactoryBuilder```](https://guava.dev/releases/19.0/api/docs/com/google/common/util/concurrent/ThreadFactoryBuilder.html) provided by Google Guava to construct a thread factory.  

* Setting the thread as non-daemon, which blocks JVM from exiting until the active tasks are finished. You can also set a thread as daemon if needed, which will be directly abandoned when JVM exits.  
* Setting a unique and readable name for the thread. This is very useful in web services because one can easily identify specific thread in JVM logs.  

JVM log with default thread factory looks like

```
13:07:33.123 [pool-1-thread-1] INFO xxxxxx
```

With customized thread pool name the log looks like

```
13:07:33.123 [JavaThreadPoolDemo] INFO xxxxxx
```

This helps with log diving and debugging a lot.

# Set up rejection policy

As mentioned previously, when the queue is full, new submitted tasks are rejected by ```RejectedExecutionException```. We can actually customize this behaviour by providing an implementation of [```RejectedExecutionHandler```](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/RejectedExecutionHandler.html).  

Java already provides some useful implementations for us  

* ```ThreadPoolExecutor.AbortPolicy``` (default handler for ```ThreadPoolExecutor```): throws ```RejectedExecutionException``` when task is rejected.  
* ```ThreadPoolExecutor.CallerRunsPolicy```: runs the rejected task directly in the calling thread of the ```execute``` or ```submit``` method  
* ```ThreadPoolExecutor.DiscardOldestPolicy```: silently discards the oldest unhandled request and then retries ```execute```  
* ```ThreadPoolExecutor.DiscardPolicy```: silently discards the rejected task  

For example, if we want to silently discards the rejected task rather than getting ```RejectedExecutionException```

```java
new ThreadPoolExecutor​(10, 10, 0L, TimeUnit.SECONDS, workerQueue, threadFactory, ThreadPoolExecutor.DiscardPolicy);
```

# Shutting down thread pool  

In web services shutdown hook, all non-daemon thread pools must be shut down to avoid memory leak.  

The best practice to shutdown a thread pool is in official documentation [(link)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ExecutorService.html):

```java
void shutdownAndAwaitTermination(ExecutorService pool) {
    // Disable new tasks from being submitted
    pool.shutdown();
    
    try {
        // Wait a while for existing tasks to terminate
        if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
            // Cancel currently executing tasks
            pool.shutdownNow();
            // Wait a while for tasks to respond to being cancelled
            if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
                System.err.println("Pool did not terminate");
            }
     }
   } catch (InterruptedException ie) {
     // (Re-)Cancel if current thread also interrupted
     pool.shutdownNow();
     // Preserve interrupt status
     Thread.currentThread().interrupt();
   }
 }
```
