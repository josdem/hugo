+++
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology","java"]
date = "2017-01-20T12:10:01-06:00"
title = "Java NIO File Copy"

+++

Before Java 7, if we need to copy a file we need to make a method or call the `copyFile(File srcFile, File destFile)` method of `FileUtils` using Apache commons io package. In Java 7, file copy function is very simple, let's consider the following example:

```java
import java.nio.file.StandardCopyOption;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class FileCopier {

  private void copy(Path source, Path target){
    try {
      Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
      System.out.println("file: " + source + " was copied successfully");
    } catch (IOException ioe) {
      System.out.println("Error: " + ioe.getMessage());
    }
  }

  public static void main(String[] args) {
    Path source = Paths.get("source/HappyFace.jpg");
    Path target = Paths.get("destination/HappyFace.jpg");
    FileCopier fileCopier = new FileCopier();
    fileCopier.copy(source, target);
  }

}
```

One notable option that we have used is the usage of the static variable `REPLACE_EXISTING` of `StandardCopyOption` class of `java.nio.file package`.

In some cases we get an `InputStream` since is used to read data from a source, `java.nio.file.Files` also support to copy from an `InputStream` to a path: `copy(InputStream in, Path target, CopyOption... options)`, see [here](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html) for more information.

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd java-nio-copy
```

[Return to the main article](/techtalk/java)



