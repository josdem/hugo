+++
date = "2017-08-27T10:53:17-05:00"
title = "Google Sign-In for Android"
categories = ["techtalk","code","android"]
tags = ["josdem","techtalks","programming","technology","Google SignIn","Oauth2 Google"]
description = "This time I will show you how to integrate Google Sing-In to an Android application."
+++

This time I will show you how to integrate Google Sing-In to an Android application. First you need to add Google play services SDK. In Android Studio select: Tools > Android > SDK Manager, then in SDK Tools tab check the Google Play Services checkbox button.

Create a new project in android studio with an empty Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "26.0.0"
    defaultConfig {
        applicationId "com.jos.dem.oauth2"
        minSdkVersion 21
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
    compile 'com.google.android.gms:play-services-auth:11.0.4'  // 2
    testCompile 'junit:junit:4.12'
}

apply plugin: 'com.google.gms.google-services'  // 1
```

1. Adding Google Services plugin
2. Adding Google Play Services Auth

Also in the project's `gradle.properties` you need to add Google services

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'com.google.gms:google-services:3.1.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

You will need to get a configuration file, in order to do that follow this instructions: [Get a configuration file](https://developers.google.com/mobile/add?platform=android). At the end of this process you will get an `google-services.json` copy it to the app directory.


In the `activity_main.xml` create an Sign-In button and TextView status label components:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center"
              android:orientation="vertical">

    <TextView
        android:id="@+id/status"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="SignIn using Google"
        android:textSize="15sp"
        android:textStyle="bold"/>

    <com.google.android.gms.common.SignInButton
        android:id="@+id/sign_in_button"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

<img src="/img/techtalks/android/oauth2_google_1.png">


This is the MainActivity:

```java
package com.jos.dem.oauth2;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.TextView;

import com.google.android.gms.auth.api.Auth;
import com.google.android.gms.auth.api.signin.GoogleSignInAccount;
import com.google.android.gms.auth.api.signin.GoogleSignInOptions;
import com.google.android.gms.auth.api.signin.GoogleSignInResult;
import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.SignInButton;
import com.google.android.gms.common.api.GoogleApiClient;

public class MainActivity extends AppCompatActivity implements GoogleApiClient.OnConnectionFailedListener, View.OnClickListener {

  private static final int RC_SIGN_IN = 9001;
  private static final String TAG = "Oauth2Google";

  private GoogleApiClient googleApiClient;
  private TextView statusTextView;
  private SignInButton signInButton;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    statusTextView = (TextView) findViewById(R.id.status);

    GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN) // 1
            .requestEmail()
            .build();

    googleApiClient = new GoogleApiClient.Builder(this)  // 2
            .enableAutoManage(this, this)
            .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
            .build();

    signInButton = (SignInButton) findViewById(R.id.sign_in_button);
    signInButton.setOnClickListener(this);
  }

  @Override
  public void onClick(View view) {
    signIn();
  }

  private void signIn() {
    Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(googleApiClient);  // 3
    startActivityForResult(signInIntent, RC_SIGN_IN);
  }

  @Override
  public void onConnectionFailed(ConnectionResult connectionResult) {
    Log.d(TAG, "onConnectionFailed:" + connectionResult);
  }

  @Override
  public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == RC_SIGN_IN) {
      GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);  // 4
      handleSignInResult(result);
    }
  }

  private void handleSignInResult(GoogleSignInResult result) {
    Log.d(TAG, "handleSignInResult:" + result.isSuccess());
    if (result.isSuccess()) {
      GoogleSignInAccount account = result.getSignInAccount();
      statusTextView.setText("Signed as: " + account.getEmail());
      signInButton.setVisibility(View.GONE);
    } else {
      statusTextView.setText("Signed in unsuccessfully");
    }
  }

}
```

This is the Sign-In Google Intent:

<img src="/img/techtalks/android/oauth2_google_2.png">

1. A `GoogleSignInOptions` object with the `DEFAULT_SIGN_IN` parameter and requesting users' email addresses as well with the `requestEmail()` option.
2. A `GoogleApiClient` object with access to the Google Sign-In API. `enableAutoManage` requires `FragmentActivity` or `AppCompatActivity` as first parameter and `OnConnectionFailedListener` as second parameter, in our case this will be both our `MainActivity`.
3. Prompts the user to select a Google account to sign in with.
4. `onActivityResult` method, retrieve the sign-in result with `getSignInResultFromIntent`.

Here is when we `handleSignInResult`:

<img src="/img/techtalks/android/oauth2_google_3.png">

To download the code:

```bash
git clone https://github.com/josdem/android-google-oauth2.git
```


[Return to the main article](/techtalk/android)
