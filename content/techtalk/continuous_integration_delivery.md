+++
title =  "Continuous Integration & Delivery"
description = "Continuous_integration_delivery"
date = "2018-10-07T11:42:11-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

**Introduction**

The most important problem that we face as software professionals is this: How we deliver to users as quickly as possible? Development pipeline is, in essence, an automated implementation of your application’s build, deploy, test and releases process. The aim in the deployment pipe line is to make every part of the process if building,deploying, testing, and releasing software visible to everybody involved, aiding collaboration, it improves feedback so that problems are identified, and so resolved, as early in the process as possible, finally it enables teams to deploy and release any version of their software to any environment at will through a fully automated process.


*TIP: So while it is important to deliver our software as quickly as possible, it is essential to maintain an appropiate level of quality.*

**Every change should trigger the feedback process**

A working software application can be usefull decomposed into four components: excecutable code, configuration, host environment, and data. If any of them changes, it can lead to a change in the behaviour of the application. Therefore we need to keep all four fo these components under control and ensure that a change in any of them is verified.

* The process of creating the executable code must work.
* The software’s unit test must pass.
* The software should fulfill certain quiality criteria such as test coverage and other technology-specific metrics.
* The software’s functional acceptance test must pass.
* The software’s nonfunctional test must pass.
* The software must go through exploratory testing and a demosntration to the customer and a selection of users.

**The feedback must be received as soon as possible**

The key to fast feedback is automation. If you have manual processes, you are dependent on people to get the job done. People take longer, they introduce errors, and they are not auditable. Perform an automatic build, test and deploy, and you should do as follow:

Tests should run fast

* They are comprehensive as possible
* Coverage must be more than 75%, so when they pass, we have a good level of confidence that the application works.
* If any of them fails, it means our application has a critical fault and should not be released under any circumstances.

**The delivery team must receive feedback and then act on it**

It is essential that everybody involved in the process of delivering software is involved in the feedback process. That includes developers, testers, operations staff, infrastructure specialists, and managers. Being able to react to feedback also means broadcasting information. Using big and visible dashboards and other notification mechanisms.

Finally, feedback is no good unless it is acted upon. This requires discipline and planning. When something needs doing, it is responsibility of the team decide on a course of action.

* [Git Basics](/techtalk/cicd/git_basics)
* [Building Software with Gradle](/techtalk/cicd/gradle)


[Return to the main article](/techtalk/techtalks)
