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

If you want to know how to enable IIS features, please go [here](https://computingforgeeks.com/install-and-configure-iis-web-server-on-windows-server/); also, we need to install some third-party applications:

- IIS URL rewrite, you can find it [here](https://www.iis.net/downloads/microsoft/url-rewrite)
- ASP .NET Core 5.0 Runtime, you can find it [here](https://dotnet.microsoft.com/en-us/download/dotnet/5.0). Make sure you install the Hosting Bundle installer.

HTTPS protocol is mandatory for all communication between robots and Orchestrator. Therefore we need to create a self-certificate, which is not recommended for Production. Type this command in a PowerShell console:

```bash
$pass = ConvertTo-SecureString -String "Secret" -Force -AsPlainText
$imported = Import-PfxCertificate -FilePath "%USERPROFILE%\UIPathCertificate.pfx" -CertStoreLocation Cert:\LocalMachine\My\ -Exportable -Password $pass
$store = New-Object System.Security.Cryptography.X509Certificates.X509Store( "Root", "LocalMachine")
$store.Open("MaxAllowed")
$store.Add($imported)
```

## Single-Node Installation

Run the Windows Installer (UiPathOrchestrator.msi). The UiPath Orchestrator Setup wizard is displayed.

<img src="/img/techtalks/uipath/orchestrator1.png">

Click Next, and the Orchestrator IIS Settings step is displayed.


<img src="/img/techtalks/uipath/orchestrator2.png">


Make sure your certificate is imported in both Personal and Trusted Root Certification Authorities; you can manage that configuration by opening the Manage Computer Certificates located in Control Panel. When the Application Pool Settings step is displayed select custom account and provide your server user's credentials. Next step is Database Settings.

<img src="/img/techtalks/uipath/orchestrator3.png">

In this setup, we will have UiPath databases on the same server, but it is recommended to setup the databases on a different server. Also, you should choose Windows Integrated Authentication instead of SQL Server Authentication.

[Return to the main article](/techtalk/techtalks)
