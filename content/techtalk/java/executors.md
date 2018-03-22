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

```


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
