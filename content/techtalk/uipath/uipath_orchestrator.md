+++
title =  "UiPath Orchestrator"
description = "UiPath Orchestrator is a web application that enables you to orchestrate robots in executing repetitive business processes"
date = "2022-04-24T17:55:43-04:00"
tags = ["josdem", "techtalks","programming","technology","uipath"]
categories = ["techtalk", "code"]
+++

[UiPath](https://www.uipath.com/) Is a technology running on Windows that helps you to automate repetitive tasks, those tasks can be executed by robots and the main component in UIPath to coordinate those robots is [UiPath Orchestrator](https://docs.uipath.com/orchestrator). In this technical post we will go over the process of installing and configuring UiPath Orchestrator in a Windows Server 2019. First, we need to be sure we meet the [hardware requirements](https://docs.uipath.com/installation-and-upgrade/docs/orchestrator-hardware-requirements) and [software requirements](https://docs.uipath.com/installation-and-upgrade/docs/orchestrator-software-requirements). Let's start by enabling and configuring IIS in our Windows Server enabling this features:

- IIS WebServer Sockets
- IIS Application Init
- IIS ISAPI Extensions
- IIS ISAPI Filter
- IIS ASPNET 4.7
- IIS Windows Authentication
- IIS URL Authorization

If you want to know how to enble IIS features, please go [here](https://computingforgeeks.com/install-and-configure-iis-web-server-on-windows-server/)

[Return to the main article](/techtalk/techtalks)
