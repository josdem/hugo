+++
title = "Apache POI Excel Filter Reader"
date = "2017-06-19T14:12:19-05:00"
categories = ["techtalk","code", "excel", "apache POI"]
tags = ["josdem","techtalks","programming","technology","hidden cells", "apache poi"]
description = "Apache POI is a popular API that allows programmers to create, modify, and display MS Office files using Java programs. This time I will show you how to read an Excel file."
+++


Apache POI is a popular API that allows programmers to create, modify, and display MS Office files using Java programs. This time I will show you how to read an Excel file. First we need to create a Spring Boot basic project using `spring init` command from installed [SDKMAN](http://sdkman.io/index.html)

```bash
spring init --build=gradle --language=groovy excel-filter-reader
```

Previous command will create this structure

```bash
excel-filter-reader
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    ├── main
    │   ├── groovy
    │   │   └── com
    │   │       └── example
    │   │           
    │   │           
    │   └── resources
    │       └── application.properties
    └── test
        └── groovy
            └── com
                └── example                   
```

Edit your `build.gradle` file to add Apache POI dependencies.

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.1.RELEASE'
		apachePoiVersion = '3.15'
		spockVersion = '1.1-groovy-2.4-rc-3'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'apache-filter-reader'
	version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile 'org.springframework.boot:spring-boot-starter'
	compile 'org.codehaus.groovy:groovy'
	compile "org.apache.poi:poi:$apachePoiVersion"
	compile "org.apache.poi:poi-ooxml:$apachePoiVersion"
	testCompile "org.spockframework:spock-core:$spockVersion"
	testCompile 'org.springframework.boot:spring-boot-starter-test'
}

```


We are filtering ranking column, so we are showing only ranking 5 rows 

<img src="/img/techtalks/java/excel_filter.png">

This is the file without filter applied::

<img src="/img/techtalks/java/excel_without_filter.png">

The below code explains how to read an Excel file using Apache POI but only those rows not filtered, **see `isHidden()` method below**.

```groovy
package com.example

import org.apache.poi.ss.usermodel.Row
import org.apache.poi.ss.usermodel.Cell
import org.apache.poi.ss.usermodel.Sheet
import org.apache.poi.ss.usermodel.Workbook
import org.apache.poi.ss.usermodel.CellType
import org.apache.poi.xssf.usermodel.XSSFWorkbook

class ExcelFilterReader {

	List<String> readRows(Integer numberOfRows, File excelFile){
		List content = new ArrayList<ArrayList<String>>()
		List line = new ArrayList<String>()
		try{
			FileInputStream fileInputStream = new FileInputStream(excelFile)
			Workbook workbook = new XSSFWorkbook(fileInputStream)
			Sheet sheet = workbook.getSheetAt(0)

			for (Row row : sheet) {
				line = new ArrayList<String>()
				if(!isHidden(row)){
					for (Cell cell : row) {
						if (cell.getCellTypeEnum() == CellType.STRING) {
							line.add(cell.getStringCellValue())
							} else if (cell.getCellTypeEnum() == CellType.NUMERIC) {
								line.add(String.valueOf(cell.getNumericCellValue()))
							}
						}
						content.add(line)
					}
				}
		} catch(FileNotFoundException fne){
			  throw new ExcelException(fne.getMessage(), fne)
		} catch(IOException ioe){
				throw new ExcelException(ioe.getMessage(), ioe)
		}

		return content
  }

  private Boolean isHidden(Row row){
    row.getZeroHeight()
  }

}
```

This is the ExcelException class:

```groovy
package com.example

class ExcelException extends RuntimeException {

	ExcelException(String message){
		super(message)
	}

	ExcelException(String message, Throwable cause){
		super(message, cause)
	}
	
}
```

To test read Excel funtionality we are using [Spock Framework](http://spockframework.org/spock/docs/1.1-rc-3/index.html)

```groovy
package com.example

import spock.lang.Specification

class ExcelFilterReaderSpec extends Specification {

  ExcelFilterReader excelReader = new ExcelFilterReader()

	void "should read filtered Excel rows"(){
		given:"Rows to read"
      Integer numberOfRows = 5
    and:"An excel file"
      File excelFile = new File("src/test/resources/input.xlsx")
    when:"We read rows"
      List<List<String>> result = excelReader.readRows(numberOfRows, excelFile)
    then:"We expect to get content"
    result.size() == 3
    result.get(0) == ['Name', 'Email', 'Ranking']
    result.get(1) == ['josdem','joseluis.delacruz@gmail.com','5.0']
    result.get(2) == ['martin','martinv@email.com','5.0']
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
cd excel-filter-reader
```


[Return to the main article](/techtalk/java)
