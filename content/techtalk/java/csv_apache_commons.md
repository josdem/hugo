+++
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology","Java"]
date = "2017-01-28T11:13:59-06:00"
title = "CSV with Apache Commons"
description = "Apache Commons CSV Commons software library provides a write and read files funtionality in variations of the Comma Separated Value (CSV) format."
+++

Apache Commons CSV makes read and write files funtionality easy whether is Comma or Tab Separated (CSV), for more information go [here](https://commons.apache.org/proper/commons-csv/). This time we will review how to write/read CSV tab delimeted files in java. Let's create a Java basic project structure using [lazybones](https://github.com/pledbrook/lazybones).

```bash
lazybones create java-basic csv-apache-commons
```

Previous command will create this structure

```bash
<proj>
      |
      +- src
          |
          +- main
          |   |
          |   +- java
          |
          +- test
          |   |
          |   +- java
```

Edit your `build.gradle` file in order to create a Jar task builder plus adding Apache commons and Junit 5 dependencies.

```groovy
ext {
  springVersion = '5.2.2.RELEASE'
  apacheCSVVersion = '1.7'
  junitJupiterVersion = '5.5.2'
}

apply plugin: "java"
apply plugin: "application"

version = '1.0.0'

task buildJar(type: Jar) {
  manifest {
    attributes 'Implementation-Title': 'Transforming an Excel file using AWS Lambda',
    'Implementation-Version': version,
    'Main-Class': 'example.Application'
  }
  baseName = project.name + '-all'
  from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
  with jar
}

repositories {
  mavenCentral()
}

dependencies {
  implementation "org.springframework:spring-context:$springVersion"
  implementation "org.springframework:spring-core:$springVersion"
  implementation "org.apache.commons:commons-csv:$apacheCSVVersion"
  testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
  testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
}

test {
  useJUnitPlatform()
}
```

Now is time to create a CSV file reader under `project-dir/src/main/java/example/`

```java
package example;

import java.io.FileReader;
import java.io.IOException;
import java.util.List;
import java.util.ArrayList;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVRecord;

public class CsvFileReader {

  public List<List<String>> read(String path){
    List elements = new ArrayList<List<String>>();
    try{
      FileReader input = new FileReader(path);
      Iterable<CSVRecord> records = CSVFormat.TDF.withHeader("id", "name", "email").parse(input);

      records.forEach(record -> {
            List<String> row = new ArrayList<String>();
            row.add(record.get("id"));
            row.add(record.get("name"));
            row.add(record.get("email"));
            elements.add(row);
          });
    } catch(IOException ioe){
      throw new CsvException(ioe.getMessage());
    }
    return elements;
  }

}
```

`CsvFileReader` reads our CSV file using `CSVFormat.TDF.withHeader("id", "name", "email").parse(input)` where:

*  `TDF` is tab delimited specification
* `parse` action reads from `FileReader` input

This is the `CsvException`

```java
package example;

import java.lang.RuntimeException;

public class CsvException extends RuntimeException {

  CsvException(String message){
    super(message);
  }

}
```

And this is the CSV example file:

```
id  name  email
1 eric  erich@email.com
2 martin  martinv@email.com
3 josdem  josdem@email.com
```

For reading our the CSV file, we need to create this simple POJO containing target id, name and email.

```java
package example;

public class Target {

  private String id;
  private String name;
  private String email;

  public Target(String id, String name, String email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getId() {
    return id;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void setEmail(String email) {
    this.email = email;
  }

  public String getEmail() {
    return email;
  }

}
```

`CsvFileWriter` writes the CSV file using `CSVFormat.TDF.withHeader("id", "name", "email").withRecordSeparator(NEW_LINE_SEPARATOR).print(output)` where:

* TDF is tab delimited specification
* `withRecordSeparator` specify an '\n' character at the ending of new record written
* `print` action writes to the FileWriter output

```java
package example;

import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.ArrayList;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVRecord;
import org.apache.commons.csv.CSVPrinter;

public class CsvFileWriter {

  private static final String NEW_LINE_SEPARATOR = "\n";

  public void write(List<Target> targets, String path) {
    List elements = new ArrayList<List<String>>();
    try {
      FileWriter output = new FileWriter(path);
      CSVPrinter printer = CSVFormat.TDF.withHeader("id", "name", "email")
          .withRecordSeparator(NEW_LINE_SEPARATOR).print(output);

      for (Target target : targets) {
        List<String> record = new ArrayList<String>();
        record.add(target.getId());
        record.add(target.getName());
        record.add(target.getEmail());
        printer.printRecord(record);
      }

      output.flush();
      output.close();
    } catch (IOException ioe) {
      throw new CsvException(ioe.getMessage());
    }
  }

}
```

This are our test cases written in [Junit 5](https://junit.org/junit5/)

```java
package example;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.List;
import java.util.Arrays;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

class CsvFileReaderTest {

  private CsvFileReader reader = new CsvFileReader();

  @Test
  @DisplayName("Should read csv file")
  void shouldReadCsvFile(){
    String path = "src/test/resources/csv.txt";
    List<List<String>> result = reader.read(path);

    assertAll("rows",
        () -> assertEquals(4, result.size(), "should have four rows"),
        () -> assertTrue(result.contains(Arrays.asList("id","name","email")), "should have header"),
        () -> assertTrue(result.contains(Arrays.asList("1","eric","erich@email.com")), "should contain Eric"),
        () -> assertTrue(result.contains(Arrays.asList("2","martin","martinv@email.com")), "should contain Martin"),
        () -> assertTrue(result.contains(Arrays.asList("3","josdem","josdem@email.com")), "should contain josdem")
        );
  }
}
```

And

```java
package example;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;
import java.util.Arrays;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

class CsvFileWriterTest {

  private CsvFileWriter writer = new CsvFileWriter();

  @Test
  @DisplayName("Should write csv file")
  void shouldWriteCsvFile() throws Exception {
    String path = "src/test/resources/csv.txt";
    Target t1 = new Target("1", "eric", "erich@email.com");
    Target t2 = new Target("2", "martin", "martinv@email.com");
    Target t3 = new Target("3", "josdem", "josdem@email.com");

    List<Target> targets = Arrays.asList(t1, t2, t3);

    writer.write(targets, path);

    Files.lines(Paths.get(path)).forEach(line ->
        assertTrue(line.contains("email"), "should contains email word")
    );

  }

}
```

To run the project:

```bash
gradle test
```

To browse the project go [here](https://github.com/josdem/java-workshop), to download the project:

```bash
git clone git@github.com:josdem/java-workshop.git
cd aws-csv-transformer
```


[Return to the main article](/techtalk/java)
