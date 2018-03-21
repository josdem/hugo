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

Previous code we are creating a thread-pool with 3 threads

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
