+++
title =  "Firebase Remote Config"
description = "Firebase_remote_config"
date = "2019-03-03T09:25:34-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code", "android"]
+++


[Firebase Remote Config](https://firebase.google.com/docs/remote-config/) is a Google cloud service that lets you change the behavior and appearance of your app without rebuild. In this technical post we will cover the basic configuration in an Android application in order to change the REST API to override environments (QA and Production).

The first thing you have to do is to add Firebase Remote Config to your `build.gradle` file.

```groovy
implementation 'com.google.firebase:firebase-config:16.3.0'
```

Next step is to create a reference to the singleton Remote Config Object and add default values.

```java
private FirebaseRemoteConfig firebaseRemoteConfig = FirebaseRemoteConfig.getInstance();
private HashMap<String, Object> defaults = new HashMap<>();

public void initializeFirebaseRemoteConfig(){
  firebaseRemoteConfig.setConfigSettings(new FirebaseRemoteConfigSettings.Builder()
						.setDeveloperModeEnabled(true)
						.build());
  defaults.put("serviceUrl", "https://webflux.josdem.io/";
  firebaseRemoteConfig.setDefaults(defaults);

  final Task<Void> fetch = firebaseRemoteConfig.fetch(TimeUnit.HOURS.toSeconds(12));
  fetch.addOnSuccessListener( it -> firebaseRemoteConfig.activateFetched() );
}
```

Where:

* `FirebaseRemoteConfigSettings` Enables developer mode
* `defaults` Store defaults values for Remote Config
* `firebaseRemoteConfig.fetch()` Tells to Remote Config to download all the parameters from Firebase console
* `fetch.addOnSuccessListener` Tracks the progress of the download
* `firebaseRemoteConfig.activateFetched()` Activates the parameters
*  `TimeUnit.HOURS.toSeconds(12)` Sets catched parameter values for 12 hours

So now we are able to get the server URL with this method

```java
public String getServiceUrl(){
  return firebaseRemoteConfig.getString("serviceUrl");
}
```

In order to change this `serviceUrl` dinamically, follow this steps. Go to the [Firebase Console](https://console.firebase.google.com/) and select Remote Config from the menu.

![Remote Config](/img/techtalks/android/firebase_remote_config.png)

So now, you are good to go and create a new parameter and click on Publish Change button.


![Remote Config](/img/techtalks/android/firebase_remote_config1.png)


Please notice that you need to have already added Firebase to your project in order to use Firebase Remote Config, if you want to learn how to setup Firebase in your project go [here](https://firebase.google.com/docs/android/setup). To browse the project go [here](https://github.com/josdem/jugoterapia-mobile), to download the project:

```bash
git clone git@github.com:josdem/jugoterapia-mobile.git
```

[Return to the main article](/techtalk/android)
