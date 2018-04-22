+++
date = "2017-08-01T08:23:35-05:00"
title = "Material Design Card View"
categories = ["techtalk","code","android"]
tags = ["josdem","techtalks","programming","technology", "android", "material design","card view"]
description = "Material design is used to create visual, motion, and interaction design across Android devices. I will show you how to create card views and how to respond to user event."
+++

Material design is used to create visual, motion, and interaction design across Android devices. Go [Here](https://developer.android.com/training/material/index.html) for more information. This time I will show you how to create card views and how to respond to user event.

## Setup

Create a new project in android studio with an empty Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

android {
  compileSdkVersion 25
  buildToolsVersion "26.0.0"
  defaultConfig {
      applicationId "dem.jos.com.card"
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
  compile 'com.android.support:cardview-v7:25.3.1'
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
    tools:context="com.jos.dem.card.MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</android.support.constraint.ConstraintLayout>
```

Now is time to connect it to a layout manager and attach an adapter for the data to be displayed using this `Activity`

```java
package com.jos.dem.card;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity implements ItemClickListener {

  private MainAdapter adapter;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
    recyclerView.setLayoutManager(new GridLayoutManager(this, 2, GridLayoutManager.VERTICAL, false)); // 1
    adapter = new MainAdapter(this, getDataSet());
    adapter.setClickListener(this);
    recyclerView.setAdapter(adapter);
  }

  private List<Fruit> getDataSet() {
    List<Fruit> fruits = new ArrayList<Fruit>();
    fruits.add(new Fruit("Lime", R.drawable.lime));
    fruits.add(new Fruit("Apple", R.drawable.apple));
    fruits.add(new Fruit("Watermelon", R.drawable.watermelon));
    fruits.add(new Fruit("Pear", R.drawable.pear));
    return fruits;
  }

  @Override
  public void onItemClick(View view, int position) {
    Toast.makeText(this, "You clicked " + adapter.getItem(position), Toast.LENGTH_SHORT).show();
  }

}
```

1. We are using a `GridLayoutManager` and this is the constructor:

```java
GridLayoutManager (Context context, 
                int spanCount, 
                int orientation, 
                boolean reverseLayout)
```

The data model class is given below:

```java
package com.jos.dem.card;

public class Fruit {
  private String name;
  private int thumbnail;

  public Fruit(String name, int thumbnail) {
    this.name = name;
    this.thumbnail = thumbnail;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getThumbnail() {
    return thumbnail;
  }

  public void setThumbnail(int thumbnail) {
    this.thumbnail = thumbnail;
  }
}
```

Here is the `ItemClickListener` interface

```java
package com.jos.dem.card;

import android.view.View;

public interface ItemClickListener {
  void onItemClick(View view, int position);
}
```

Here is the `card_view` layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <android.support.v7.widget.CardView
        xmlns:card_view="http://schemas.android.com/apk/res-auto"
        android:id="@+id/card_view"
        android:layout_width="180dp"
        android:layout_height="180dp"
        android:layout_gravity="center"
        card_view:cardCornerRadius="4dp">

        <ImageView
            android:id="@+id/thumbnail"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <TextView
            android:id="@+id/name"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:textColor="@color/colorText" />
    </android.support.v7.widget.CardView>

</LinearLayout>
```

Here is our adapter:

```java
package com.jos.dem.card;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.List;

public class MainAdapter extends RecyclerView.Adapter<MainAdapter.ViewHolder> {
  private List<Fruit> dataset;
  private Context context;
  private ItemClickListener clickListener;

  public MainAdapter(Context context, List<Fruit> dataset) {
    this.context = context;
    this.dataset = dataset;
  }

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {                   //2
    View view = LayoutInflater.from(context).inflate(R.layout.card_view, parent, false);
    return new ViewHolder(view);
  }

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {   // 3
    Fruit fruit = dataset.get(position);
    holder.name.setText(fruit.getName());
    holder.thumbnail.setImageResource(fruit.getThumbnail());
  }

  @Override
  public int getItemCount() {   // 4
    return dataset.size();
  }

  public String getItem(int position) {
    return dataset.get(position).getName();
  }

  public void setClickListener(ItemClickListener itemClickListener) {
    this.clickListener = itemClickListener;
  }

  public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {  // 1
    public TextView name;
    public ImageView thumbnail;

    public ViewHolder(View itemView) {
      super(itemView);
      name = (TextView) itemView.findViewById(R.id.name);
      thumbnail = (ImageView) itemView.findViewById(R.id.thumbnail);
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

<img src="/img/techtalks/android/material_design_cards.png">

To download the code:

```bash
git clone https://github.com/josdem/android-material-design-workshop.git
cd card
```

[Return to the main article](/techtalk/android)
