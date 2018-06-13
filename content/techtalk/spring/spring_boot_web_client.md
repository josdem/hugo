+++
title =  "Spring Boot WebClient"
description = "Spring_boot_web_client"
date = "2018-06-13T15:54:27-04:00"
tags = ["josdem", "techtalks","programming","technology", "Spring Boot WebClient", "WebFlux WebClient"]
categories = ["techtalk", "code", "WebFlux"]
+++

WebClient is a reactive client that provides an alternative to the RestTemplate. It exposes a functional, fluent API and relies on non-blocking I/O which allows it to support high concurrency more efficiently than the RestTemplate. WebClient is a natural fit for streaming scenarios and depends on a lower level HTTP client library to execute requests and that support is pluggable.

WebClient uses the same codecs as WebFlux server applications do, and shares a common base package, some common APIs, and infrastructure with the server functional web framework. The API exposes Reactor Flux and Mono types. By default it uses it uses Reactor Netty as the HTTP client library but others can be plugged in through a custom ClientHttpConnector.

By comparison to the RestTemplate, the WebClient is:

* Non-blocking, reactive, and supports higher concurrency with less hardware resources.
* Provides a functional API that takes advantage of Java 8 lambdas.
* Supports both synchronous and asynchronous scenarios.
* Supports streaming up or down from a server.

The RestTemplate is not a good fit for use in non-blocking applications, and therefore Spring WebFlux application should always use the WebClient. The WebClient should also be preferred in Spring MVC, in most high concurrency scenarios, and for composing a sequence of remote, inter-dependent calls.

## Retrieve 

The `retrieve()` method is the easiest way to get a response body and decode it:

```java
WebClient client = WebClient.create("http://jugoterapia.josdem.io/jugoterapia-server");

Mono<Beverage> result = client.get()
  .uri("/beverages/{id}", id).accept(MediaType.APPLICATION_JSON)
  .retrieve()
  .bodyToMono(Beverage.class);
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot)