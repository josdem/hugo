+++
title =  "Spring Boot in Nginx server"
description = "In this technical post we will review how we can deploy a Spring Boot application in a Nginx server"
date = "2018-12-02T17:46:25-05:00"
tags = ["josdem", "techtalks","programming","technology","Nginx"]
categories = ["techtalk", "code", "sysadmin", "nginx"]
+++

In this technical post we will review how we can deploy a Spring Boot application in a [Nginx](https://www.nginx.com/) server. Please read this previous [Spring Boot Setup](/techtalk/spring/spring_boot) before continue this this information. I am using Nginx server 1.15.5 in Ubuntu 18.10. First, let's create a new file description service in: `/etc/systemd/system/spring-boot-setup.service` with this information:

```bash
[Unit]
Description=Spring Boot Setup
After=syslog.target
After=network.target[Service]
User=josdem
Type=simple

[Service]
ExecStart=/bin/java -jar /opt/spring-boot-setup0.0.1-SNAPSHOT.jar
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=spring-boot-setup

[Install]
WantedBy=multi-user.target
```

Start this service

```bash
sudo systemctl start spring-boot-setup
```

Check if is active

```bash
sudo systemctl status spring-boot-setup
```

Next step is to create a reverse proxy configuration in: `/etc/nginx/conf.d/spring-boot-setup.conf`

```bash
server {
  listen 80;
  listen [::]:80;

  server_name spring.josdem.io;

  location / {
    proxy_pass http://localhost:8080/;
  }
}
```

Test this configuration

```bash
sudo nginx -t
```

Output

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Now restart Nginx

```bash
sudo systemctl restart nginx
```

The application is now accessible. Navigate to the public domain address and the "Hello World!" message should appear.


[Return to the main article](/techtalk/sysadmin)
