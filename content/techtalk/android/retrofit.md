+++
date = "2017-07-24T22:02:39-05:00"
title = "REST Calls using Retrofit"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology", "android", "retrofit2"]

+++

Retrofit is a HTTP client for Android and Java, it turns your HTTP API into a Java interface. This time I will show you how to create a basic project using [Retrofit](http://square.github.io/retrofit/).

## Setup

Create a new project in android studio with an empty Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

android {
  compileSdkVersion 25
  buildToolsVersion "26.0.0"
  defaultConfig {
      applicationId "dem.jos.com.retrofit"
      minSdkVersion 15
      targetSdkVersion 25
      versionCode 1
      versionName "1.0"
      testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
  }
  buildTypes {
    release {
      minifyEnabled false
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}

dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
      exclude group: 'com.android.support', module: 'support-annotations'
  })
  compile 'com.android.support:appcompat-v7:25.3.1'
  compile 'com.android.support.constraint:constraint-layout:1.0.2'
  testCompile 'junit:junit:4.12'

  compile 'com.squareup.retrofit2:retrofit:2.3.0'
  compile 'com.squareup.retrofit2:converter-gson:2.3.0'
}
```

Here we are using [Gson](https://github.com/google/gson) converter to transform the JSON responses to the model classes.

In this example we are going to use a Jugoterapia API to get juice categories: http://jugoterapia.josdem.io/jugoterapia-server/beverage/categories

This is the Json output

```json
[
  {
    id: 1,
    name: "Curativos"
  },
  {
    id: 2,
    name: "Energizantes"
  },
  {
    id: 3,
    name: "Saludables"
  },
  {
    id: 4,
    name: "Estimulantes"
  }
]
```

Let's create a basic model Category class

```java
package com.jos.dem.retrofit.model;

public class Category {
  private Integer id;
  private String name;

  public Integer getId() {
    return id;
  }

  public void setId(Integer id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  @Override
  public String toString() {
    return "id: " + this.id + " name: " + this.name;
  }
}
```

To download the code:

```bash
git clone https://github.com/josdem/android-retrofit-workshop.git
```


[Return to the main article](/techtalk/android)
