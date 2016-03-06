+++
date = "2016-03-05T19:24:14-06:00"
draft = true
title = "groovy-wslite SOAP"

+++

Recently I needed to consume an SOAP service from an Spring application, my first approach was creating an snippet that works from Groovy console, my code worked and looks like this:

```groovy
@Grab('com.github.groovy-wslite:groovy-wslite:1.1.2')

import wslite.soap.SOAPClient

def file = new File("/tmp/file.xml")
assert file.exists()

String wsdl = 'https://server/servicios/soap/stamp.wsdl'

def client = new SOAPClient(wsdl)
def response = client.send(SOAPAction: 'stamp') {
  body {
    stamp('xmlns': 'https://server/WSDL') {
      xml(file.bytes.encodeBase64().toString())
      username(username)
      password(password)
    }
  }
}
println response.dump()
```

Unfortunately when I convert this to a Spring bean was not working anymore, since I can not find a response in Stack Overflow, I needed to figure out was happening by my own and after that I wrote this post. My code in Spring looks like this:

```groovy
package com.tim.one.billing.services.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired
import javax.annotation.PostConstruct
import wslite.soap.SOAPClient

import com.tim.one.billing.services.StampProvider
import com.tim.one.billing.state.ApplicationState

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class StampProviderImpl implements StampProvider {

  String wsdl
  String namespace

  @Autowired
  Properties properties

  Log log = LogFactory.getLog(getClass())

  @PostConstruct
  void initialize(){
    wsdl = properties.getProperty(ApplicationState.STAMP_WSDL)
    namespace = properties.getProperty(ApplicationState.STAMP_NAMESPACE)
  }

  @Override
  def stamp(File factura, String username, String password){
    def client = new SOAPClient(wsdl)
    def response = client.send(SOAPAction: 'stamp') {
      body {
        stamp('xmlns':namespace) {
          delegate.username(username)
          delegate.password(password)
          delegate.xml(factura.bytes.encodeBase64().toString())
        }
      }
    }
  }

}
```

The issue was a closure scope, and using delegate inside the nested closure, wslite sent the right parameters to the service.

[Return to the main article](/techtalk/spring)
