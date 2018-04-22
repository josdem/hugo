+++
tags = ["josdem","techtalks","programming","technology", "android", "material design"]
date = "2017-07-29T10:25:03-05:00"
title = "Material Design Creating Lists"
categories = ["techtalk","code","android"]
description = "Material design is used to create visual, motion, and interaction design across Android devices. I will show you how to create lists and how to respond to user event."
+++

Material design is used to create visual, motion, and interaction design across Android devices. Go [Here](https://developer.android.com/training/material/index.html) for more information. This time I will show you how to create lists and how to respond to user event.

## Setup

Create a new project in android studio with an empty Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

android {
  compileSdkVersion 25
  buildToolsVersion "26.0.0"
  defaultConfig {
    applicationId "com.jos.dem.list"
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
  compile 'com.android.support:recyclerview-v7:25.3.1'
  compile 'com.android.support.constraint:constraint-layout:1.0.2'
  testCompile 'junit:junit:4.12'
}
```

In order to create a lists we need to use a `RecyclerView` which is more advance and flexible that `ListView`. `RecyclerView` needs:

* `LayoutManager` for positioning item
* Adapter provides access to set and update data in views
* Dataset is the model

Here we are adding the `RecyclerView` to the main layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context="com.jos.dem.list.MainActivity">

  <android.support.v7.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:scrollbars="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

</android.support.constraint.ConstraintLayout>
```

Now is time to connect it to a layout manager and attach an adapter for the data to be displayed using this `Activity`

```java
package com.jos.dem.list;

import android.app.Activity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends Activity implements ItemClickListener {

  private MainAdapter adapter;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    List<String> dataset = new ArrayList<String>();
    dataset.add("josdem");
    dataset.add("skuarch");
    dataset.add("tgrip");
    dataset.add("erich");
    dataset.add("martinvillegas");

    RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
    recyclerView.setLayoutManager(new LinearLayoutManager(this));
    recyclerView.setHasFixedSize(true);       // 1
    adapter = new MainAdapter(this, dataset);
    adapter.setClickListener(this);
    recyclerView.setAdapter(adapter);
  }

  @Override
  public void onItemClick(View view, int position) {
    Toast.makeText(this, "You clicked " + adapter.getItem(position), Toast.LENGTH_SHORT).show();
  }

}
```

1. Use this setting to improve performance if you know that changes in content do not change the layout size of the `RecyclerView`

Here is the `ItemClickListener` interface

```java
package com.jos.dem.list;

import android.view.View;

public interface ItemClickListener {
  void onItemClick(View view, int position);
}
```

Here is the `recycler_view` layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:orientation="horizontal"
  android:padding="10dp">

  <TextView android:id="@+id/nicknames"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">

  </TextView>

</LinearLayout>

```

Here is our adapter:

```java
package com.jos.dem.list;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.List;

public class MainAdapter extends RecyclerView.Adapter<MainAdapter.ViewHolder> {
  private List<String> dataset;
  private LayoutInflater inflater;
  private ItemClickListener clickListener;

  public MainAdapter(Context context, List<String> dataset) {
    this.inflater = LayoutInflater.from(context);
    this.dataset = dataset;
  }

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {  // 2
    View view = inflater.inflate(R.layout.recycler_view, parent, false);
    ViewHolder viewHolder = new ViewHolder(view);
    return viewHolder;
  }

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {  // 3
    holder.textView.setText(dataset.get(position));
  }

  @Override
  public int getItemCount() {  // 4
    return dataset.size();
  }

  public String getItem(int id) {
    return dataset.get(id);
  }

  public void setClickListener(ItemClickListener itemClickListener) {
    this.clickListener = itemClickListener;
  }

  public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener { // 1
    public TextView textView;

    public ViewHolder(View itemView) {
      super(itemView);
      textView = (TextView) itemView.findViewById(R.id.nicknames);
      itemView.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
      clickListener.onItemClick(view, getAdapterPosition());
    }
  }

}
```

1. You need to create a `ViewHolder` implementation wich provide a reference to the views for each data item
2. `onCreateViewHolder` Creates new views invoked by the layout manager
3. `onBindViewHolder` replaces the contents of a view invoked by the layout manager
4. `getItemCount` returns the size of your dataset invoked by the layout manager

This is the List view using `RecyclerView`

<img src="/img/techtalks/android/material_design_lists.png">

To download the code:

```bash
git clone https://github.com/josdem/android-material-design-workshop.git
cd list
```

[Return to the main article](/techtalk/android)
