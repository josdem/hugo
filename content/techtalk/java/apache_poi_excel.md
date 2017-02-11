+++
date = "2017-02-10T19:21:27-06:00"
title = "Apache POI Excel"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","java"]

+++

Apache POI is a popular API that allows programmers to create, modify, and display MS Office files using Java programs. This time I will show you how to read Microsoft Office 2016. First we need to create a Java basic project using [lazybones](https://github.com/pledbrook/lazybones).

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

Edit your `build.gradle` file to add Apache POI dependencies.

```groovy
ext.apachePoiVersion = '3.15'
ext.groovyVersion = '2.4.8'
ext.spockVersion = '1.1-groovy-2.4-rc-3'

apply plugin: "java"
apply plugin: 'groovy'
apply plugin: "application"

repositories {
  mavenCentral()
}

dependencies {
  compile "org.apache.poi:poi:$apachePoiVersion"
  compile "org.apache.poi:poi-ooxml:$apachePoiVersion"
  testCompile "org.codehaus.groovy:groovy:$groovyVersion"
  testCompile "org.spockframework:spock-core:$spockVersion"
}
```

This is the Excel file we are going to read:

<img src="/img/techtalks/java/excel_poi.png">

The below code explains how to read an Excel file using Apache POI libraries.

```java
package example;

import java.util.List;
import java.util.ArrayList;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.FileNotFoundException;

import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.CellType;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class ExcelReader {

  List<String> readRows(Integer numberOfRows, File excelFile){
    List content = new ArrayList<ArrayList<String>>();
    List line = new ArrayList<String>();
    try{
      FileInputStream fileInputStream = new FileInputStream(excelFile);
      Workbook workbook = new XSSFWorkbook(fileInputStream);
      Sheet sheet = workbook.getSheetAt(0);

      for (Row row : sheet) {
        line = new ArrayList<String>();
        for (Cell cell : row) {
          if (cell.getCellTypeEnum() == CellType.STRING) {
            line.add(cell.getStringCellValue());
          } else if (cell.getCellTypeEnum() == CellType.NUMERIC) {
            line.add(String.valueOf(cell.getNumericCellValue()));
          }
        }
        content.add(line);
      }
    } catch(FileNotFoundException fne){
      throw new ExcelException(fne.getMessage(), fne);
    } catch(IOException ioe){
      throw new ExcelException(ioe.getMessage(), ioe);
    }

    return content;
  }

}
```

This is the ExelException:

```java
package example;

import java.lang.RuntimeException;
import java.lang.Throwable;

public class ExcelException extends RuntimeException {

  ExcelException(String message){
    super(message);
  }

  ExcelException(String message, Throwable cause){
    super(message, cause);
  }

}
```

To test read Excel funtionality we are using [Spock Framework](http://spockframework.org/spock/docs/1.1-rc-3/index.html)

```groovy
package example

import spock.lang.Specification

class ExcelReaderSpec extends Specification {

  ExcelReader excelReader = new ExcelReader()

  void "should read Excel rows"(){
    given:"Rows to read"
      Integer numberOfRows = 5
    and:"An excel file"
      File excelFile = new File("src/test/resources/input.xlsx")
    when:"We read rows"
      List<List<String>> result = excelReader.readRows(numberOfRows, excelFile)
    then:"We expect to get content"
    result.size() == 4
    result.get(0) == ['Name', 'Email', 'Ranking']
    result.get(1) == ['josdem','joseluis.delacruz@gmail.com','5.0']
    result.get(2) == ['eric','erich@email.com','5.0']
    result.get(3) == ['martin','martinv@email.com','5.0']
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
cd excel-reader
```


[Return to the main article](/techtalk/java)
