+++
date = "2015-09-07T18:02:25-05:00"
draft = true
title = "Tomcat listening in a domain"

+++
When we want to publish our services in a specific domain such as api.josdem.io and that services are hosted in a Tomcat, the best way I can recommend you to do it is using a AJP connector. In this post I'll show you how to do it.

## Light Tomcat
The first thing you must do is get rid of configuration you don't need in your Tomcat, this give you the next advantages:

* Less code
* Faster Tomcat
* Better knowledge what you have
* Better control

This is a file example how  you can configure your "light" Tomcat
file: $TOMCAT_HOME/config/server.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8001" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <Service name="Catalina">
    <Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jmetadata">
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```

Notice jvmRoute parameter is added as AJP Connector.

## A record configuration
It is important you have your domain pointing to your IP address. Usually, you can do this creating an A record in your domain management. As soon as you got it, you should do something like this:

```bash
$nslookup jmetadata.josdem.io

Server:   127.0.1.1
Address:  127.0.1.1#53

Non-authoritative answer:
Name: jmetadata.josdem.io
Address: 52.8.244.12
```

## Creating a virtual host in your Apache Server
Next step is create a file named josdem.io.conf in /etc/httpd/conf.d with this your Apache will know how connect your Tomcat, here is my example file:

```xml
<VirtualHost *:80>
  DocumentRoot "/var/www/html/website"
  ServerName josdem.io
  ServerAlias jmetadata.josdem.io
  <Directory "/var/www/html/website">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order Deny,Allow
    Allow from all
  </Directory>
  ErrorLog logs/jmetadata.josdem.io-error_log
  CustomLog logs/jmetadata.josdem.io-access_log common

  <Proxy balancer://clusterJmetadata>
    BalancerMember ajp://localhost:8010 route=jmetadata
    ProxySet stickysession=JSESSIONID
  </Proxy>

  ProxyPass /manage-balancer !
  ProxyPass / balancer://clusterJmetadata/ stickysession=JSESSIONID
  ProxyPassReverse / balancer://clusterJmetadata/

  <Location /manage-balancer>
    SetHandler balancer-manager
  </Location>
</VirtualHost>
```

## Tomcat as a service
Finally you may want to start, stop your tomcat service as any service in your server, so here is an example.
File: /etc/rc.d/init.d/tomcatJmetadata

```bash
#!/bin/bash
#
# tomcat7     This shell script takes care of starting and stopping Tomcat
#
# chkconfig: - 80 20
#
## BEGIN INIT INFO
# Provides: tomcat7
# Required-Start: $network $syslog
# Required-Stop: $network $syslog
# Default-Start:
# Default-Stop:
# Description: Release implementation for Servlet 2.5 and JSP 2.1
# Short-Description: start and stop tomcat
## END INIT INFO

## Source function library.
#. /etc/rc.d/init.d/functions
TOMCAT_HOME=/opt/tomcat_jmetadata
export JAVA_HOME=/opt/jdk1.7.0_79
export PATH=$JAVA_HOME/bin:$PATH
SHUTDOWN_WAIT=20

tomcat_pid() {
  echo `ps aux | grep tomcat_jmetadata | grep -v grep | awk '{ print $2 }'`
}

start() {
  pid=$(tomcat_pid)
  if [ -n "$pid" ]
  then
    echo "Tomcat is already running (pid: $pid)"
  else
    # Start tomcat
    echo "Starting tomcat"
    #ulimit -n 100000
    umask 007
    #/bin/su -p -s /bin/sh tomcat $TOMCAT_HOME/bin/startup.sh
    /bin/sh $TOMCAT_HOME/bin/startup.sh
  fi


  return 0
}

stop() {
  pid=$(tomcat_pid)
  if [ -n "$pid" ]
  then
    echo "Stoping Tomcat"
    #/bin/su -p -s /bin/sh tomcat $TOMCAT_HOME/bin/shutdown.sh
    /bin/sh $TOMCAT_HOME/bin/shutdown.sh

    let kwait=$SHUTDOWN_WAIT
    count=0;
    until [ `ps -p $pid | grep -c $pid` = '0' ] || [ $count -gt $kwait ]
    do
      echo -n -e "\nwaiting for processes to exit";
      sleep 1
      let count=$count+1;
    done

    if [ $count -gt $kwait ]; then
      echo -n -e "\nkilling processes which didn't stop after $SHUTDOWN_WAIT seconds"
      kill -9 $pid
    fi
  else
    echo "Tomcat is not running"
  fi

return 0
}

case $1 in
start)
  start
;;
stop)
  stop
;;
restart)
  stop
  start
;;
status)
  pid=$(tomcat_pid)
  if [ -n "$pid" ]
  then
    echo "Tomcat is running with pid: $pid"
  else
    echo "Tomcat is not running"
  fi
;;
esac
exit 0
```

Follow this final steps:

1. Give execution provilegies to users: `sudo hmod 755 tomcatJmetadata`
2. Add service to the system with: `sudo chkconfig --add tomcatJmetadata`
3. List services to confirm refistration: `sudo chkconfig --list`
4. Execute: `service tomcatJemtadata start` to start your tomcat service

Now, you should get your service in the: http://jmetadata.josdem.io base

You may see your new service listed.

[Return to the main article](/techtalk/techtalks)
