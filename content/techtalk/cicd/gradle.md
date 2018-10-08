+++
title = "Building Software with Gradle"
categories = ["techtalk", "code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2015-10-18T22:05:02-05:00"
description = "Using DSL language you write your build script like you write your code and this is based on Groovy."
+++

Gradle is an open-source build automation tool focused on flexibility and performance. Gradle build scripts are written using a Groovy or Kotlin DSL. Using DSL language you write your build script file as you write your development code. Some advantages in use Gradle are:

* Acelerate development productivity
* Build by convention
* Automate everything
* Deliver faster persuing performance
* Multy-module projects
* Reusable pieces of build logic
* Intelligent tasks that can check if they need to be executed or not

In this post I will show you how to build software using simply but useful Gradle functionality. Firstly lets create a `build.gradle` file with a single task called helloWorld and add an action to it. The scripting language we are using here is Groovy, if you want to know more please go here: [Groovy](/techtalk/groovy/)

```groovy
task helloWorld {
  println "I am saying hello in the year of our lord: ${new Date()}"
}
```

Then execute this command in your terminal:

```bash
gradle helloWorld
```

*Output:*

```bash
I am saying hello in the year of our lord: Sun Oct 07 15:28:16 EDT 2018
```

**Variables**

You can define variables in `build.gradle` as follow:

```groovy
def projectDescription = "Gradle is a build automation tool"

task showRootProject {
  println "Project Description: ${projectDescription}"
  println project.rootProject.properties
}
```

In previous task we are displaying `projectDescription` value using Groovy placeholders in addition we are desplaying the project properties, a variable already defined in Gradle for our convenience.

*Output:*

```bash
Project Description: Gradle is a build automation tool
[parent:null, classLoaderScope:org.gradle.api.internal.initialization.DefaultClassLoaderScope@71bf6081, buildDir:/Users/moralej3/Projects/gradle-workshop/build, configurations:[], plugins:[org.gradle.api.plugins.HelpTasksPlugin@179e437], scriptHandlerFactory:org.gradle.api.internal.initialization.DefaultScriptHandlerFactory@3ffbb4ae, objects:org.gradle.api.internal.model.DefaultObjectFactory@5d7e388e, logger:org.gradle.internal.logging.slf4j.OutputEventListenerBackedLogger@4105a437, deferredProjectConfiguration:org.gradle.api.internal.project.DeferredProjectConfiguration@4c61f1a0, rootDir:/Users/moralej3/Projects/gradle-workshop, project:root project 'gradle-workshop', projectRegistry:org.gradle.api.internal.project.DefaultProjectRegistry@46d02bc5, path::, normalization:org.gradle.normalization.internal.DefaultInputNormalizationHandler_Decorated@246cb975, repositories:[], childProjects:[:], scriptPluginFactory:org.gradle.configuration.ScriptPluginFactorySelector@87a5f3e, state:project state 'EXECUTING', resourceLoader:org.gradle.internal.resource.transfer.DefaultUriTextResourceLoader@4bd54360, serviceRegistryFactory:org.gradle.internal.service.scopes.ProjectScopeServices$4@c9cc7dc, tasks:[task ':helloWorld', task ':showRootProject'], group:, artifacts:org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@4dfa602b, ext:org.gradle.api.internal.plugins.DefaultExtraPropertiesExtension@17a34505, projectDir:/Users/moralej3/Projects/gradle-workshop, dependencyLocking:org.gradle.internal.locking.DefaultDependencyLockingHandler_Decorated@768cdad2, nexusUser:moralej3, configurationTargetIdentifier:org.gradle.configuration.ConfigurationTargetIdentifier$1@1c4451f8, projectEvaluationBroadcaster:ProjectEvaluationListener broadcast, projectPath::, module:org.gradle.api.internal.artifacts.ProjectBackedModule@21d5e0a, inheritedScope:org.gradle.api.internal.ExtensibleDynamicObject$InheritedDynamicObject@2219163, version:unspecified, script:false, dependencies:org.gradle.api.internal.artifacts.dsl.dependencies.DefaultDependencyHandler_Decorated@f27674f, fileResolver:org.gradle.api.internal.file.BaseDirFileResolver@1f6d7633, extensions:org.gradle.api.internal.plugins.DefaultConvention@21982fb7, modelRegistry:org.gradle.model.internal.registry.DefaultModelRegistry@2fbc086e, projectEvaluator:org.gradle.configuration.project.LifecycleProjectEvaluator@1b951fc9, projectConfigurator:org.gradle.api.internal.project.BuildOperationCrossProjectConfigurator@6f3d3e66, name:gradle-workshop, logging:org.gradle.internal.logging.services.DefaultLoggingManager@f3851a1, configurationActions:org.gradle.configuration.project.DefaultProjectConfigurationActionContainer@4088d750, buildscript:org.gradle.api.internal.initialization.DefaultScriptHandler@73d0e3df, gradleTechtalkVersion:0.0.1, helloWorld:task ':helloWorld', showRootProject:task ':showRootProject', status:release, processOperations:org.gradle.api.internal.file.DefaultFileOperations@75182dcd, subprojects:[], components:[], asDynamicObject:DynamicObject for root project 'gradle-workshop', displayName:root project 'gradle-workshop', identityPath::, parentIdentifier:null, description:null, antBuilderFactory:org.gradle.api.internal.project.DefaultAntBuilderFactory@3860cf96, buildPath::, fileOperations:org.gradle.api.internal.file.DefaultFileOperations@75182dcd, pluginManager:org.gradle.api.internal.plugins.DefaultPluginManager_Decorated@3e54bd9d, standardOutputCapture:org.gradle.internal.logging.services.DefaultLoggingManager@f3851a1, defaultTasks:[], modelSchemaStore:org.gradle.model.internal.manage.schema.extract.DefaultModelSchemaStore@49605da, class:class org.gradle.api.internal.project.DefaultProject_Decorated, buildScriptSource:org.gradle.groovy.scripts.TextResourceScriptSource@4a6e7f93, convention:org.gradle.api.internal.plugins.DefaultConvention@21982fb7, allprojects:[root project 'gradle-workshop'], baseClassLoaderScope:org.gradle.api.internal.initialization.DefaultClassLoaderScope@7eac4556, ant:org.gradle.api.internal.project.DefaultAntBuilder@400e396a, resources:org.gradle.api.internal.resources.DefaultResourceHandler@5baddca3, services:ProjectScopeServices, url:https://api.github.com/, gradle:build 'gradle-workshop', layout:org.gradle.api.internal.file.DefaultProjectLayout@6206a2a2, buildFile:/Users/moralej3/Projects/gradle-workshop/build.gradle, depth:0, nexusPass:Th3str4ng3r., rootProject:root project 'gradle-workshop', properties:(this Map), providers:org.gradle.api.internal.provider.DefaultProviderFactory@5b60eb8f]
```

If you want to know what tasks you can execute in a Gradle project, use this command:

```bash
gradle tasks
```

*Output:*

```bash
> Task :tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradle-workshop'.
components - Displays the components produced by root project 'gradle-workshop'. [incubating]
dependencies - Displays all dependencies declared in root project 'gradle-workshop'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradle-workshop'.
dependentComponents - Displays the dependent components of components in root project 'gradle-workshop'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'gradle-workshop'. [incubating]
projects - Displays the sub-projects of root project 'gradle-workshop'.
properties - Displays the properties of root project 'gradle-workshop'.
tasks - Displays the tasks runnable from root project 'gradle-workshop'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL in 0s
```

With `ext` section you can define extra properties to be added to existing domain project:

```groovy
ext{
  gradleTechtalkVersion = "0.0.1"
}

task showProjectContent {
  println project.tasks
  println "group:artifact:$gradleTechtalkVersion"
}
```

In previous task we are displaying gradleTechtalkVersion and all project tasks we defined. Usually we use `ext` section to define project dependency variables.

*Output:*

```bash
[task ':helloWorld', task ':showProjectContent', task ':showRootProject']
group:artifact:0.0.1
```

As you probably have guessed, you can declare tasks that depend on other tasks, let's consider the following example.

```groovy
task first {
  println "Hello World, this is my First Task!"
}

task second {
  println "Hello World from my Second Task!"
}

second.dependsOn rootProject.tasks['first']
```

That's it, `rootProject` is our main project task and we are defining first tasks as dependent on second task, we usually do this when we need to orquestate tasks in our project, if you want to see a bower task dependency example please go here: [Spring Boot Bower Plugin](http://josdem.io/techtalk/spring/spring_boot_bower_plugin/)

**Copy tasks**

Other cool functionality in Gradle is the copy task which copies files into a directory destination. This task can also rename and filter files as it copies, we usually do this when we need to copy extenalized config files. Let's define a new `environments.gradle` file with this content.

```groovy
ext {
	configurationDir = ".gradleWorkshop/"
	currentEnvironment = "development"
}

def filesToInclude = ["credentials-${currentEnvironment}.yaml"]
def toDirectory = 'src/main/resources/'
task settingEnvironment(type:Copy){
	from configurationDir
	into toDirectory
	include filesToInclude
	rename { String fileName -> fileName.replace("-${currentEnvironment}", '') }
}
```

Here is our `credentials-development.yaml` file

```yaml
credentials:
  username: josdem
  password: secret
```

In order to execute previous Gradle script file:

* Create a directory named: `.gradleWorkshop` in your root project directory
* Copy the `credentials-development.yaml` file in that directory
* Execute follwing command line

```bash
gradle -b environments.gradle settingEnvironment
```

To browse the code go [here](https://github.com/josdem/gradle-workshop), to download the code:

```bash
git clone git@github.com:josdem/gradle-workshop.git
```

[Return to the main article](/techtalk/continuous_integration_delivery)
