+++
categories = ["techtalk", "code", "groovy"]
date = "2016-07-12T18:46:49-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "groovy", "bindable"]
title = "Bindable"

+++

Each class in Groovy has propeties, and a property is a combination of a private field and getters/setters

```groovy
class Player {
  String nickname
  Integer score
}

println new Player(nickname:'josdem', score:90).properties
```

Output:

```
[nickname:josdem, class:class Player, score:90]
```

As you can see, properties is a map, so we can do the following:

```groovy
class Player {
  String nickname
  Integer score
}

def map = new Player(nickname:'josdem', score:90).properties
println map['nickname']
```

Output:

```
josdem
```

`@groovy.beans.Bindable` is an annotation that can go either on a Groovy class property or a Groovy class itself. A bound property is a bean property for which a change to the property results in a notification being sent to some other bean.

```groovy
import groovy.beans.Bindable
import groovy.transform.ToString

@ToString
class Player {
  String nickname

  @Bindable
  Integer score
}

Player player = new Player(nickname:'josdem', score:90)

player.propertyChange = {
  println "Player has new score:"
  println it.properties
}

player.score = 91
```

Output:

```
Player has new score:
[newValue:91, propertyName:score, class:class java.beans.PropertyChangeEvent, oldValue:90, propagationId:null, source:Player(josdem, 91)]
```

That's it, when score changes which is a bindable field the closure propertyChange will respond with a "Player has new score:" and then print the properties from it.

[Return to the main article](/techtalk/groovy)
