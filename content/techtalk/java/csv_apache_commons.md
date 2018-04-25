+++
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology","Java"]
date = "2017-01-28T11:13:59-06:00"
title = "CSV with Apache Commons"
description = "Apache Commons CSV Commons software library provides a write and read files funtionality in variations of the Comma Separated Value (CSV) format."
+++

Apache Commons CSV Commons software library provides a write and read files funtionality in variations of the Comma Separated Value (CSV) format, for more information go [here](https://commons.apache.org/proper/commons-csv/). This time I will show how to write/read CSV tab delimeted files in java. First we need to create a Java basic project this time using [lazybones](https://github.com/pledbrook/lazybones).

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

Edit your `build.gradle` file to create a make a Jar task and add Apache commons dependencies.

```groovy
ext.apacheCSVVersion = '1.4'
ext.groovyVersion = '2.4.8'
ext.spockVersion = '1.1-groovy-2.4-rc-3'

apply plugin: "java"
apply plugin: "groovy"
apply plugin: "application"

version = '0.0.1'

task buildJar(type: Jar) {
  manifest {
    attributes 'Implementation-Title': 'Read Write an CSV file with Apache Commons',
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
  compile "org.apache.commons:commons-csv:$apacheCSVVersion"
  testCompile "org.codehaus.groovy:groovy:$groovyVersion"
  testCompile "org.spockframework:spock-core:$spockVersion"
}
```

Then define Java classes under `project-dir/src/main/java/example/`

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
      for (CSVRecord record : records) {
        List<String> row = new ArrayList<String>();
        String id = record.get("id");
        String name = record.get("name");
        String email = record.get("email");
        row.add(id);
        row.add(name);
        row.add(email);
        elements.add(row);
      }
    } catch(IOException ioe){
      throw new CsvException(ioe.getMessage());
    }
    return elements;
  }

}
```

`CsvFileReader` reads the CSV file in java using `CSVFormat.TDF.withHeader("id", "name", "email").parse(input)` where:

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

In order to read the CSV file, we need to create this simple POJO to contain the target id, name and email.

```java
package example;

public class Target {
  private String id;
  private String name;
  private String email;

  public void setId(String id){
    this.id = id;
  }

  public String getId(){
    return id;
  }

  public void setName(String name){
    this.name = name;
  }

  public String getName(){
    return name;
  }

  public void setEmail(String email){
    this.email = email;
  }

  public String getEmail(){
    return email;
  }

}
```

`CsvFileWriter` writes the CSV file in java using `CSVFormat.TDF.withHeader("id", "name", "email").withRecordSeparator(NEW_LINE_SEPARATOR).print(output)` where:

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

  public void write(List<Target> targets, String path){
    List elements = new ArrayList<List<String>>();
    try{
      FileWriter output = new FileWriter(path);
      CSVPrinter printer = CSVFormat.TDF.withHeader("id", "name", "email").withRecordSeparator(NEW_LINE_SEPARATOR).print(output);

      for(Target target: targets){
        List<String> record = new ArrayList<String>();
        record.add(target.getId());
        record.add(target.getName());
        record.add(target.getEmail());
        printer.printRecord(record);
      }

      output.flush();
      output.close();
    } catch(IOException ioe){
      throw new CsvException(ioe.getMessage());
    }
  }

}
```

This are the Groovy test cases to prove the functionality:

**CsvFileReaderSpec.groovy**

```groovy
package example;

import spock.lang.Specification;

class CsvFileReaderSpec extends Specification {

  CsvFileReader reader = new CsvFileReader()

  void "Should read csv file"(){
    given:"A path"
    String path = 'src/test/resources/csv.txt'
    when:
    def result = reader.read(path);
    then:
    result.size() == 4
    result.get(0) == ['id','name','email']
    result.get(1) == ['1','eric','erich@email.com']
    result.get(2) == ['2','martin','martinv@email.com']
    result.get(3) == ['3','josdem','josdem@email.com']
  }

}
```

**CsvFileWriterSpec.groovy**

```groovy
package example;

import spock.lang.Specification;

class CsvFileWriterSpec extends Specification {

  CsvFileWriter writer = new CsvFileWriter()

  void "Should write to a csv file"(){
    given:"A path"
      String path = 'src/test/resources/csv_written.txt'
    and:"Some targets"
      Target t1 = new Target(id:'1',name:'eric',email:'erich@email.com')
      Target t2 = new Target(id:'2',name:'martin',email:'martinv@email.com')
      Target t3 = new Target(id:'3',name:'josdem',email:'josdem@email.com')
      List<Target> targets = [t1,t2,t3]
    when:
    writer.write(targets,path);
    File file = new File(path)
    then:"We expect file exist"
    List<String> lines = []
    file.eachLine { line ->
      lines << line
    }
    lines.get(0) == 'id name  email'
    lines.get(1) == '1  eric  erich@email.com'
    lines.get(2) == '2  martin  martinv@email.com'
    lines.get(3) == '3  josdem  josdem@email.com'
  }

}

```

To run the project:

```bash
gradle test
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd aws-csv-transformer
```


[Return to the main article](/techtalk/java)
