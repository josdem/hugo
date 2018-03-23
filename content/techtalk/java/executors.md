+++
date = "2018-03-21T11:13:59-06:00"
title = "Executors"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

Executors is part of concurrency Java API, and it is a high level manging threads. Executors typically manage a pool of threads.

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
```

Previous code we are creating a thread-pool with 3 threads. Executors have to be stopped explicitly, otherwise they keep listening for new tasks, the preferred way to do it is following:

```java
executor.shutdown();
executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
```

Executor shuts down softly by waiting a certain amount of time for termination of currently running tasks. After a maximum of MAX_PERIOD_TIME seconds the executor finally shuts down by interrupting all running tasks.

Here is a complete code that counts how many time we are saying hello:

```java
import java.util.stream.IntStream;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

public class ExecutorCounter {

	private final static Integer MAX_PERIOD_TIME = 30;

	private Integer count = 0;
	private ExecutorService executor = Executors.newFixedThreadPool(3);

	private Integer start() throws InterruptedException {
		IntStream.range(0, 3).forEach(i -> executor.submit(this::incrementSync));
		executor.shutdown();

		executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
		return count;
	}

	private void incrementSync() {
		count++;
		System.out.println("I have been saying hello: " + count + " times");
	}

	public static void main(String[] args) throws InterruptedException {
		new ExecutorCounter().start();
	}

}
```

We might have this output:

```bash
I have been saying hello: 1 times
I have been saying hello: 2 times
I have been saying hello: 3 times
```

Or this one:

```bash
I have been saying hello: 2 times
I have been saying hello: 3 times
I have been saying hello: 1 times
```

This is because when executed, it will produce unpredictable results due to the threads running independently, that is common known as [Race Condition](https://en.wikipedia.org/wiki/Race_condition). Java supports thread-synchronization via the synchronized keyword. We can utilize synchronized to fix the above race conditions when incrementing the count

```java
import java.util.stream.IntStream;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

public class ExecutorCounter {

  private final static Integer MAX_PERIOD_TIME = 30;

  private Integer count = 0;
  private ExecutorService executor = Executors.newFixedThreadPool(3);

  private Integer start() throws InterruptedException {
    IntStream.range(0, 3).forEach(i -> executor.submit(this::incrementSync));
    executor.shutdown();

    executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
    return count;
  }

  private synchronized void incrementSync() {
    count++;
    System.out.println("I have been saying hello: " + count + " times");
  }

  public static void main(String[] args) throws InterruptedException {
    new ExecutorCounter().start();
  }

}
```

Another way to avoid race conditions is to use atomic operations such as `AtomicInteger`, besides it has a better performance than synchonizing locks. Let's create that version as follow:

```java
import java.util.stream.IntStream;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.atomic.AtomicInteger;

public class ExecutorAtomic {

	private final static Integer MAX_PERIOD_TIME = 30;
	private AtomicInteger atomic = new AtomicInteger(0);
	private ExecutorService executor = Executors.newFixedThreadPool(3);

	private Integer start() throws InterruptedException {
		IntStream.range(0, 3).forEach(i -> executor.submit(atomic::incrementAndGet));
		executor.shutdown();

		executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
		return atomic.get();
	}
	
	public static void main(String[] args) throws InterruptedException {
		Integer result = new ExecutorAtomic().start();
		System.out.println("I have been counting: " + result + " times");
	}

}
``` 

**Callables and Futures**

Executors support another kind of task named Callable, which is functional interfaces just like runnables but instead of being void they return a value.

Callables can be submitted to executor services. Since `submit()` doesn't wait until the task completes, the executor service cannot return the result of the callable directly. Instead the executor returns a special result of type Future which can be used to retrieve the actual result at a later point in time.

```java
import java.util.concurrent.Future;
import java.util.concurrent.Callable;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ExecutionException;

public class ExecutorCallable {

  private final static Integer MAX_PERIOD_TIME = 30;

  private ExecutorService executor = Executors.newFixedThreadPool(3);

  private void start() throws InterruptedException, ExecutionException {
    Future<Integer> future = executor.submit(new CallableThread());
    final Integer result = future.get();
    executor.shutdown();

    System.out.println("I have been sleeping: " + result + " seconds");
    executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
  }

  public static void main(String[] args) throws InterruptedException, ExecutionException {
    new ExecutorCallable().start();
  }
}

class CallableThread implements Callable<Integer> {

  @Override
  public Integer call() throws InterruptedException{
    final Integer wait = 3;
    TimeUnit.SECONDS.sleep(wait);
    return wait;
  }

}
```

*Output*

```bash
I have been sleeping: 3 seconds
```

In Java 8, the `CompletableFuture` class was introduced and also implements this `Future` interface.

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.CompletableFuture;

public class ExecutorCompletableFuture {

  private final static Integer MAX_PERIOD_TIME = 30;

  private ExecutorService executor = Executors.newFixedThreadPool(3);

  private void start() throws InterruptedException, ExecutionException {
    CompletableFuture<Integer> completableFuture  = new CompletableFuture<Integer>();

    executor.submit(() -> {
      try{
        final Integer wait = 3;
        TimeUnit.SECONDS.sleep(wait);
        completableFuture.complete(wait);
      } catch (InterruptedException ie){
        System.out.println("InterruptedException: " + ie.getMessage());
      }    
    });    

    final Integer result = completableFuture.get();
    executor.shutdown();

    System.out.println("I have been sleeping: " + result + " seconds");
    executor.awaitTermination(MAX_PERIOD_TIME, TimeUnit.SECONDS);
  }

  public static void main(String[] args) throws InterruptedException, ExecutionException {
    new ExecutorCompletableFuture().start();
  }
}

```

*Output*

```bash
I have been sleeping: 3 seconds
```

A common way to avoid boilerplate is simply execute some asynchronous methods with `runAsync`, `supplyAsync` and `applyAsync`:

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureAsynchronous {

  private void start() throws InterruptedException, ExecutionException {
    CompletableFuture<Integer> completableFuture  = CompletableFuture.completedFuture(3).thenApplyAsync(wait -> {
      try{
        TimeUnit.SECONDS.sleep(wait);        
      } catch(InterruptedException ie){
        System.out.println("InterruptedException: " + ie.getMessage());
      }
      return wait;
    });

    final Integer result = completableFuture.get();
    System.out.println("I have been sleeping: " + result + " seconds");
  }

  public static void main(String[] args) throws InterruptedException, ExecutionException {
    new CompletableFutureAsynchronous().start();
  }
}
```

*Output*

```bash
I have been sleeping: 3 seconds
```

To download the project:

```bash
git clone https://github.com/josdem/java-workshop.git
cd executors
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java ${JAVA_PROGRAM}
```


[Return to the main article](/techtalk/java)
