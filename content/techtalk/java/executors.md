+++
date = "2018-03-21T11:13:59-06:00"
title = "Executors"
tags = ["josdem", "techtalks","programming","technology","java"]
categories = ["techtalk", "code","java"]
description = "Executors is part of concurrency Java API, and it is a high level manging threads. Executors typically manage a pool of threads."
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

Executors support another kind of task named Callable, which is functional interfaces just like runnables but instead of being void they return a value. Callables can be submitted to executor services. Since `submit()` doesn't wait until the task completes, the executor service cannot return the result of the callable directly. Instead the executor returns a special result of type Future which can be used to retrieve the actual result at a later point in time.

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

In Java 8, the `CompletableFuture` class was introduced and also implements this `Future` interface. It provides an `isDone()` method to check whether the computation is done or not, and a `get()` method to retrieve the result of the computation when it is done.

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

A cool functionality is to use this asynchronous methods with `runAsync`, `supplyAsync` and `thenApplyAsync`. If you want to run some background task asynchronously and don’t want to return anything from the task, then you use `runAsync()`. It returns `CompletableFuture<Void>`

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureRunAsync {

  private void start() throws InterruptedException, ExecutionException {
    CompletableFuture<Void> completableFuture  = CompletableFuture.runAsync( () -> {
      try{
        TimeUnit.SECONDS.sleep(3);
      } catch(InterruptedException ie){
        System.out.println("InterruptedException: " + ie.getMessage());
      }
    });
    completableFuture.get();
    System.out.println("I have been sleeping: 3 seconds");
  }

  public static void main(String[] args) throws InterruptedException, ExecutionException {
    new CompletableFutureRunAsync().start();
  }

}
```

*Output*

```bash
I have been sleeping: 3 seconds
```

`CompletableFuture.supplyAsync()` is use when you provide a `Supplier<T>` and returns `CompletableFuture<T>` where `T` is the type of the value obtained by calling the given supplier, then you apply that `T`.


```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureSupplyAsync {

  private void start() throws InterruptedException, ExecutionException {
    CompletableFuture<String> completableFuture  = CompletableFuture.supplyAsync( () -> {
      try{
        TimeUnit.SECONDS.sleep(3);
      } catch(InterruptedException ie){
        System.out.println("InterruptedException: " + ie.getMessage());
      }
      return "3 seconds";
    }).thenApply(message -> {
      return "I have been sleeping " + message;
    });
    System.out.println(completableFuture.get());
  }

  public static void main(String[] args) throws InterruptedException, ExecutionException {
    new CompletableFutureSupplyAsync().start();
  }

}
```

*Output*

```bash
I have been sleeping: 3 seconds
```

If you want to run async logic in a separate thread tou can use `thenApplyAsync()`

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.CompletableFuture;

public class CompletableFutureThenApplyAsync {

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
    new CompletableFutureThenApplyAsync().start();
  }
}
```

*Output*

```bash
I have been sleeping: 3 seconds
```

To browse the project go [here](https://github.com/josdem/java-workshop), to download the project:

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
