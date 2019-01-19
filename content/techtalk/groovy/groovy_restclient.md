+++
date = "2015-12-10T15:33:18-06:00"
title = "RESTClient with Groovy WSLite"
categories = ["techtalk", "code","groovy"]
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "Essential Concepts"]
description = "Creating requests using RESTClient from Groovy WSLite"
+++

[Groovy-WSLite](https://github.com/jwagenleitner/groovy-wslite) is a library for Groovy that provides an easy way to consume SOAP and REST webservices. Let's consider the following [CURL](https://curl.haxx.se/) command:

```bash
curl https://weblux.josdem.io/sanity/ping
```

Response

```bash
pong
```

So using Groovy-WSLite we can create an script like this:

```groovy
#!/usr/bin/env groovy

@Grab('com.github.groovy-wslite:groovy-wslite:1.1.3')

import wslite.rest.RESTClient

def sanityStarter(){
  println 'Starting sanity check'
  def client = new RESTClient("https://webflux.josdem.io/")
  def response = client.get(path:'/sanity/ping')
  assert 200 == response.statusCode
  println 'Sanity check completed'
}

sanityStarter()
```

In order to execute previous script save it in a file named `sanityCheck.groovy` and provide the following privileges:

```bash
chmod 755 sanityCheck.groovy
```

Then you can execute this command:

```bash
./sanityCheck.groovy
```

## Post with Headers

Let's consider the following curl command:

```bash
curl -v -d 'username=myuser&password=mypassword' -X POST "http://myhost/oauth/token"
```

Here we are sending an username and password in a urlencoded format using POST method to the `http://myhost/oauth/token` url in a verbose mode.

Using `groovyx.net.http.RESTClient` could be this way:

```groovy
try{
  def rest = new RESTClient('http://myhost/')
  def response = rest.post(
    path: 'oauth/token',
    body: [username:'myuser',password:'mypassword'],
    requestContentType: URLENC )

  response.responseData
} catch(Exception ex) {
  log.warn "Error: ${ex.message}"
}
```

If we need to send some headers in curl format:

```bash
curl -H "Content-Type: application/json" -H "Authorization: Bearer be41be7a-70ca-4988-89df-54beb212e6a9" -X POST -d '{"name":"josdem","email":"joseluis.delacruz@gmail.com"}' "http://myhost/services/user"
```

Here we are sending JSON data with `Authorization: Bearer be41be7a-70ca-4988-89df-54beb212e6a9` as header.

RESTClient version:

```groovy
try{
  def rest = new RESTClient('http://myhost/')
  String token = 'mytoken'
  def response = rest.post(
    path: 'services/user',
    headers: [Authorization:"Bearer ${token}"],
    body: [name:"josdem",email:"joseluis.delacruz@gmail.com"],
    requestContentType: 'application/json' )
  response.responseData
} catch(Exception ex) {
  log.warn "Error: ${ex.message}"
}
```

[Return to the main article](/techtalk/groovy)
