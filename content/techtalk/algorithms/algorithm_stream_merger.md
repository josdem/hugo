+++
date = "2016-06-07T20:50:43-05:00"
draft = true
title = "Stream Merger"

+++

Let's consider the following class:

```java
import java.util.Set;
import java.util.HashSet;

class InputStream {
  int getStreamId(){
    return 0;
  }
}

class OutputStream {
  void emitValue(int value){}
}

public class StreamMerger {
  private Set<InputStream> streams;

  public StreamMerger(Set<InputStream> streams){
    this.streams = streams;
  }

  public void mergeInto(OutputStream stream){
  }

  public static void main(String[] args){
    new StreamMerger(new HashSet<InputStream>());
  }
}
```

Our goal is to get a collection of streams order it by stream id and then merge those elements into an output stream.

```java
import java.util.Set;
import java.util.HashSet;

class InputStream implements Comparable<InputStream>{
  private Integer streamId;

  public InputStream(Integer streamId){
    this.streamId = streamId;
  }

  public Integer getStreamId(){
    return streamId;
  }

  @Override
  public int compareTo(InputStream that){
    return this.getStreamId().compareTo(that.getStreamId());
  }
}

class OutputStream {
  void emitValue(int value){}
}

public class StreamMerger {
  private Set<InputStream> streams;

  public StreamMerger(Set<InputStream> streams){
    this.streams = streams;
  }

  public void mergeInto(OutputStream stream){
  }

  public static void main(String[] args){
    new StreamMerger(new HashSet<InputStream>());
  }
}
```

Java provides the Comparable interface, which contains only one method, called compareTo. This method compares two objects, in order to impose an order between them.

```java
import java.util.Set;
import java.util.HashSet;

class InputStream implements Comparable<InputStream>{
  private Integer streamId;

  public InputStream(Integer streamId){
    this.streamId = streamId;
  }

  public Integer getStreamId(){
    return streamId;
  }

  @Override
  public int compareTo(InputStream that){
    return this.getStreamId().compareTo(that.getStreamId());
  }

  @Override
  public String toString(){
    return this.getStreamId().toString();
  }
}

class OutputStream {
  void emitValue(int value){}
}

public class StreamMerger {
  private Set<InputStream> streams;

  public StreamMerger(Set<InputStream> streams){
    this.streams = streams;
  }

  public void mergeInto(OutputStream stream){
    void emitValue(int value){}
  }

  public static void main(String[] args){
    InputStream ist1 = new InputStream(1);
    InputStream ist2 = new InputStream(2);

    Set<InputStream> streams = new HashSet<InputStream>();
    streams.add(ist1);
    streams.add(ist2);

    new StreamMerger(streams).mergeInto(new OutputStream());
  }
}
```

Then we fill our collection with some input streams.

```java
import java.util.Set;
import java.util.HashSet;
import java.util.TreeSet;

class InputStream implements Comparable<InputStream>{
  private Integer streamId;

  public InputStream(Integer streamId){
    this.streamId = streamId;
  }

  public Integer getStreamId(){
    return streamId;
  }

  @Override
  public int compareTo(InputStream that){
    return this.getStreamId().compareTo(that.getStreamId());
  }

  @Override
  public String toString(){
    return this.getStreamId().toString();
  }
}

class OutputStream {
  void emitValue(int value){
    System.out.println(value);
  }
}

public class StreamMerger {
  private Set<InputStream> streams;

  public StreamMerger(Set<InputStream> streams){
    this.streams = streams;
  }

  public void mergeInto(OutputStream stream){
    TreeSet<InputStream> treeSet = new TreeSet<InputStream>();
    treeSet.addAll(streams);
    OutputStream outputStream = new OutputStream();
    treeSet.forEach(item -> outputStream.emitValue(item.getStreamId()));
  }

  public static void main(String[] args){
    InputStream ist1 = new InputStream(2);
    InputStream ist2 = new InputStream(1);

    Set<InputStream> streams = new HashSet<InputStream>();
    streams.add(ist1);
    streams.add(ist2);

    new StreamMerger(streams).mergeInto(new OutputStream());
  }
}
```

Finally we are using a TreeSet since a HashSet does not guarantee any order of its elements.


[Return to the main article](/techtalk/algorithms)
