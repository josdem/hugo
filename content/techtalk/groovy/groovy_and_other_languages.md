+++
title = "Comparison between Groovy and other languages"
date = "2015-08-29T14:25:41-05:00"
categories = ["techtalk", "code","groovy"]
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "From Java to Groovy"]
description = "Groovy is very assertive, let’s compare the different mechanisms to obtain today’s date in varioys languages to demonstrate it"
+++

Groovy is very assertive, let’s compare the different mechanisms to obtain today’s date in varioys languages to demonstrate it

```
import java.util.*; //Java
Date today = new Date(); //Java

today = new Date() //Groovy

require = 'date' #Ruby
today = Date.new #Ruby

import java.util._ //Scala
var today = new Date //Scala

(import '(java.util.Date)') :Clojure
(def today (new Date)) :Clojure
(def today (Date.)) :Clojure
alternative :Clojure
```

Groovy solution is short, precise and more compact than regular Java. Groovy doesn’t need to import the java.util package or specify the Date type.

[Return to the main article](/techtalk/groovy)
