+++
categories = ["projects", "java"]
date = "2015-08-23T19:24:08-05:00"
tags = ["josdem", "jmetadata"]
title = "Developers"
+++
Here you can find all information regarding how configure JMetadata code in your environment.

**Introduction**

Usually it’s not enough only download the source code and "running the code", so this section was created by supporting all issues regarding running JMetadata from eclipse IDE as application and as a Unit or Integration testing, you also are welcome to do it by command line and vim editor :)

**To the developers**

* Is my intention that you can take advantage from this code using it as a example resource.
* I’ll be honored if you would like to extend this code or do some advice.
* Jmetadata is still work in progress so, there are some issues that I’m working on.
* I was trying to reduce the number of bugs using JUnit and Integration test, but certainly there are some bugs detected, those are in the issues section, but feel free to add a new one.

**Details**

In order to set up you environment I recommend you to install the following tools, if you are using Eclipse:

* Maven plugin
* EMMA plugin
* GIT plugin

For Eclipse and Command Line:

* Maven in your system environment path
* Install Musicbrainz and LastFM third party jar’s manually in your .m2 repository

```bash
mvn install:install-file -Dfile=path-to-file -DgroupId=de.umass.lastfm -DartifactId=lastfm_bindings -Dversion=0.1.0 -Dpackaging=jar
mvn install:install-file -Dfile=path-to-file -DgroupId=com.skychief.javamusicbrainz -DartifactId=javamusicbrainz -Dversion=1.0 -Dpackaging=jar
```

*Note:* You should find the third party Jars in `lib` folder in Jmetadata source code.

**To run JMetadata**

Download code from repository:

```bash
git clone https://github.com/josdem/jmetadata.git
```
Execute this command where you downloaded the code

```bash
mvn -Dmaven.test.skip clean assembly:assembly
```

Go to the `$JMETADATA_HOME/target/` directory and unzip the `JMetadata-bin.zip` file and execute

```bash
sh JMetadata.sh
```

*Note:* To create installers read the `INSTALL` file included.


[Return to the main article](/jmetadata/jmetadata)
