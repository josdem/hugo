+++
date = "2017-07-24T22:02:39-05:00"
title = "REST Calls using Retrofit"
categories = ["techtalk","code","android"]
tags = ["josdem","techtalks","programming","technology", "android", "retrofit2"]
description = "Retrofit is a HTTP client for Android and Java, it turns your HTTP API into a Java interface. This time I will show you how to create a basic project using Retrofit."
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

This is the JSON output

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

Now we can create the `JugoterapiaService` interface that will embody our HTTP response.

```java
package com.jos.dem.retrofit.service;

import com.jos.dem.retrofit.model.Category;

import java.util.List;

import retrofit2.Call;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;
import retrofit2.http.GET;

public interface JugoterapiaService {

  @GET("/jugoterapia-server/beverage/categories")
  public Call<List<Category>> getCategories();

  public static final Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("http://jugoterapia.josdem.io/")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

}
```

Finally we are going to call asynchronously and provide the callback to be executed upon completion.

```java
package com.jos.dem.retrofit;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import com.jos.dem.retrofit.model.Category;
import com.jos.dem.retrofit.service.JugoterapiaService;

import java.util.List;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class MainActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    JugoterapiaService jugoterapiaService = JugoterapiaService.retrofit.create(JugoterapiaService.class);
    Call<List<Category>> call = jugoterapiaService.getCategories();
    call.enqueue(new Callback<List<Category>>() {

      @Override
      public void onResponse(Call<List<Category>> call, Response<List<Category>> response) {
        for(Category category: response.body()){
          Log.d("category", category.toString());
        }
      }

      @Override
      public void onFailure(Call<List<Category>> call, Throwable t) {
        Log.d("error", t.getMessage());
      }
    });

  }

}
```

That's it when you run the project, you will see the categories in the logcat Android Monitor.

```bash
07-24 21:53:46.364 14516-14516/dem.jos.com.retrofit D/category: id: 1 name: Curativos
07-24 21:53:46.364 14516-14516/dem.jos.com.retrofit D/category: id: 2 name: Energizantes
07-24 21:53:46.364 14516-14516/dem.jos.com.retrofit D/category: id: 3 name: Saludables
07-24 21:53:46.365 14516-14516/dem.jos.com.retrofit D/category: id: 4 name: Estimulantes
```

To download the code:

```bash
git clone https://github.com/josdem/android-retrofit-workshop.git
git fetch
git checkout feature/get
```

Now, we are going to see what we need to do in order to send a JSON post message, first you need to create a POJO.

```java
package com.jos.dem.retrofit.model;

public class Credentials {

  private String name;
  private String email;
  private String token;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getEmail() {
    return email;
  }

  public void setEmail(String email) {
    this.email = email;
  }

  public String getToken() {
    return token;
  }

  public void setToken(String token) {
    this.token = token;
  }

  @Override
  public String toString() {
    return "name:" + this.name + " email:" + this.email + " token:" + this.token;
  }

}
```
Then we need to add a `@Post` method in Retrofit service:

```java
package com.jos.dem.retrofit.service;

import com.jos.dem.retrofit.model.Category;
import com.jos.dem.retrofit.model.Credentials;

import java.util.List;

import retrofit2.Call;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;
import retrofit2.http.Body;
import retrofit2.http.GET;
import retrofit2.http.Headers;
import retrofit2.http.POST;

public interface JugoterapiaService {

  @GET("/jugoterapia-server/beverage/categories")
  public Call<List<Category>> getCategories();

  @Headers("Content-Type: application/json")
  @POST("http://jugoterapia.josdem.io/auth/validate")
  public Call<Credentials> sendCredentials(@Body Credentials credentials);

  public static final Retrofit retrofit = new Retrofit.Builder()
          .baseUrl("http://jugoterapia.josdem.io/")
          .addConverterFactory(GsonConverterFactory.create())
          .build();

}
```

Finally we are going to call asynchronously and provide the callback to be executed upon completion.

```java
package com.jos.dem.retrofit;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import com.jos.dem.retrofit.model.Category;
import com.jos.dem.retrofit.model.Credentials;
import com.jos.dem.retrofit.service.JugoterapiaService;

import java.util.List;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class MainActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    JugoterapiaService jugoterapiaService = JugoterapiaService.retrofit.create(JugoterapiaService.class);

    Credentials credentials = createCredentials();

    Call<Credentials> call = jugoterapiaService.sendCredentials(credentials);
    call.enqueue(new Callback<Credentials>() {

      @Override
      public void onResponse(Call<Credentials> call, Response<Credentials> response) {
        Log.d("credentials:", response.body().toString());
      }

      @Override
      public void onFailure(Call<Credentials> call, Throwable t) {
        Log.d("error", t.getMessage());
      }
    });

  }

  private Credentials createCredentials() {
    Credentials credentials = new Credentials();
    credentials.setName("josdem");
    credentials.setEmail("joseluis.delacruz@gmail.com");
    credentials.setToken("token");
    return credentials;
  }

}
```

To download the code:

```bash
git clone https://github.com/josdem/android-retrofit-workshop.git
git fetch
git checkout feature/post
```


[Return to the main article](/techtalk/android)
