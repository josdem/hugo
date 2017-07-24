+++
categories = ["techtalk","code"]
date = "2017-07-23T14:12:24-05:00"
title = "Dependency Injection with Dagger"
tags = ["josdem","techtalks","programming","technology", "android", "dagger"]

+++

Dependency injecton is when objects define their dependencies. Dependency Injection is build upon the concept Inversion of Control it is used to increase modularity of the program, make it extensible and make it easy to test.

Dagger is a dependency injection framework for both Java and Android. This time I will show you how to create a basic project using [Dagger2](https://google.github.io/dagger/).

## Setup

Create a new project in android studio with an empty Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

android {
  compileSdkVersion 25
  buildToolsVersion "26.0.0"
  defaultConfig {
    applicationId "com.jos.dem.dagger"
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
  testCompile 'junit:junit:4.12'

  compile "com.google.dagger:dagger:2.11"
  annotationProcessor "com.google.dagger:dagger-compiler:2.11"
  provided 'javax.annotation:jsr250-api:1.0'
  compile 'javax.inject:javax.inject:1'

}
```
Here we are using the `annotationProcessor` for generating the dependency graph classes during build time.

Let's create a basic model user class

```java
package com.jos.dem.dagger.model;

public class User {

  private String username;
  private String email;

  public User(String username, String email) {
    this.username = username;
    this.email = email;
  }

  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public String getEmail() {
    return email;
  }

  public void setEmail(String email) {
    this.email = email;
  }

}
```

Now let's reate few custom annotations: ActivityContext, ApplicationContext and PerActivity.

**Activity Context**

```java
package com.jos.dem.dagger.context;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

import javax.inject.Qualifier;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityContext {}
```

**Application Context**

```java
package com.jos.dem.dagger.context;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

import javax.inject.Qualifier;


@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ApplicationContext {}
```

**Per Activity**

```java
package com.jos.dem.dagger.context;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

import javax.inject.Scope;

@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {}
```

`@Qualifier` is used to distinguish between objects of the same type but with different instances. Now let's create a `UserService` as dependency to inject

```java
package com.jos.dem.dagger.service;

import javax.inject.Inject;
import javax.inject.Singleton;

import com.jos.dem.dagger.model.User;

@Singleton
public class UserService {

  User user = new User("josdem", "joseluis.delacruz@gmail.com");

  @Inject
  public UserService(){}


  public User getUser() {
    return user;
  }

}
```

`@Singleton` Ensure a single instance exist globally. `@Inject` on the constructor instructs the Dagger object when the class is being constructed.

Next, create `DemoApplication` class that extends `android.app.Application`

```java
package com.jos.dem.dagger;

import android.app.Application;
import android.content.Context;

import com.jos.dem.dagger.component.ApplicationComponent;
import com.jos.dem.dagger.component.DaggerApplicationComponent;
import com.jos.dem.dagger.module.ApplicationModule;

public class DemoApplication extends Application {

  protected ApplicationComponent applicationComponent;

  public static DemoApplication get(Context context) {
    return (DemoApplication) context.getApplicationContext();
  }

  @Override
  public void onCreate() {
    super.onCreate();
    applicationComponent = DaggerApplicationComponent
            .builder()
            .applicationModule(new ApplicationModule(this))
            .build();
    applicationComponent.inject(this);
  }

  public ApplicationComponent getComponent(){
    return applicationComponent;
  }

}
```

You should add this class as application name in the `AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.jos.dem.dagger">

  <application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="Dagger"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:name=".DemoApplication"
    android:theme="@style/AppTheme">
    <activity android:name=".MainActivity">
      <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>
  </application>

</manifest>
```

Now let's create class `MainActivity`

```java
package com.jos.dem.dagger;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.TextView;

import com.jos.dem.dagger.component.ActivityComponent;
import com.jos.dem.dagger.component.DaggerActivityComponent;
import com.jos.dem.dagger.module.ActivityModule;
import com.jos.dem.dagger.service.UserService;

import javax.inject.Inject;

public class MainActivity extends AppCompatActivity {

  @Inject
  UserService userService;

  private ActivityComponent activityComponent;

  public ActivityComponent getActivityComponent() {
    if (activityComponent == null) {
      activityComponent = DaggerActivityComponent.builder()
              .activityModule(new ActivityModule(this))
              .applicationComponent(DemoApplication.get(this).getComponent())
              .build();
    }
    return activityComponent;
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    getActivityComponent().inject(this);
    setContentView(R.layout.activity_main);
    TextView usernameTextView = (TextView) findViewById(R.id.usernameLabel);
    TextView emailTextView = (TextView) findViewById(R.id.emailLabel);
    usernameTextView.setText("Username: " + userService.getUser().getUsername());
    emailTextView.setText("Email: " + userService.getUser().getEmail());
  }

}
```

With the `MainActivity` there is a `activity_main.xml` file to represent graphical elements

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          android:id="@+id/activity_main"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:gravity="center"
          android:orientation="vertical">

  <TextView
    android:id="@+id/usernameLabel"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>

  <TextView
    android:id="@+id/emailLabel"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>

</LinearLayout>
```

Now let's create `ApplicationModule` This class defines the methods that provide the dependency. A Module class is identified by `@Module` and the dependency provider method in identified by `@Provides`.

```java

ckage com.jos.dem.dagger.module;

import android.app.Application;
import android.content.Context;

import com.jos.dem.dagger.context.ApplicationContext;

import dagger.Module;
import dagger.Provides;

@Module
public class ApplicationModule {

  private final Application application;

  public ApplicationModule(Application application) {
    this.application = application;
  }

  @Provides
  @ApplicationContext
  Context provideContext() {
    return application;
  }

  @Provides
  Application provideApplication() { return application; }

}
```

Now let's create a component which links the `DemoApplication` with the `ApplicationModule`

```java
package com.jos.dem.dagger.component;

import android.app.Application;
import android.content.Context;

import com.jos.dem.dagger.context.ApplicationContext;
import com.jos.dem.dagger.DemoApplication;
import com.jos.dem.dagger.module.ApplicationModule;
import com.jos.dem.dagger.service.UserService;

import javax.inject.Singleton;

import dagger.Component;

@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {

  void inject(DemoApplication demoApplication);

  @ApplicationContext
  Context getContext();

  Application getApplication();

  UserService getUserService();

}
```

That's it, we are injecting `UserService` into out `MainActivity` and with that we are getting user data information.

<img src="/img/techtalks/android/dagger.png">

To download the code:

```bash
git clone https://github.com/josdem/android-dagger2-workshop.git
```


[Return to the main article](/techtalk/android)
