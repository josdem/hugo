+++
date = "2015-12-10T15:33:18-06:00"
draft = true
title = "RESTClient, headers and data"

+++

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
curl -H "Content-Type: application/json" -H "Authorization: Bearer be41be7a-70ca-4988-89df-54beb212e6a9" -X POST -d '{"name":"josdem","email":"user@email.com"}' "http://myhost/services/user"
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
    body: [name:"josdem",email:"user@email"],
    requestContentType: 'application/json' )
  response.responseData
} catch(Exception ex) {
  log.warn "Error: ${ex.message}"
}
```

[Return to the main article](/techtalk/groovy)
