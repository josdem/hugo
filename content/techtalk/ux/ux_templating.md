+++
title = "Templating with Swig & Express"
categories = ["techtalk","code","UX development"]
tags = ["josdem","techtalks","programming","technology","UX development"]
date = "2016-02-08T18:50:20-06:00"
description = "Express is a minimal and flexible Node.js web application framework."
+++

Express is a minimal and flexible Node.js web application framework. To install:

```bash
npm install express --save
```

Swig is a simple, powerful, and extendable JavaScript Template Engine. To install:

```bash
npm install swig --save
```

In this lines we are registering our swig engine in express as html and specifying our template directory

```javascript
application.engine('html', swig.renderFile)
application.set('view engine', 'html')
application.set('views', __dirname + '/views')
```

We can configure cache in our application as follows:

```javascript
application.set('view cache', false)
swig.setDefaults({cache:false})
```

Do not forget to set `cache:true` when you are going to production.
With this lines you start your server:

```javascript
application.listen(3000, function(){
 console.log('Server ready and listening at: 3000')
})
```

Let's consider the following server.js file where we specify an get function that renders to an index.html and displays an custom welcome message to our user:

```javascript
var = require('express')
var swig = require('swig')

var application = express()

application.engine('html', swig.renderFile)
application.set('view engine', 'html')
application.set('views', __dirname + '/views')

application.set('view cache', false)
swig.setDefaults({cache:false})

application.get("/",function(request, response){
  response.render("index", {
    title: "Templating",
    person: {"name":"josdem"}
  })
})

application.listen(3000, function(){
  console.log('Server ready and listening at: 3000')
})
```

Here is our index.html file

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{{title}}</title>
  </head>
  <body>Welcome {{person.name}}</body>
</html>
```

To execute this js file type in your terminal:

```bash
npm install
node server.js
```

To download the project:

```bash
git clone https://github.com/josdem/ux-development.git
git fetch
git checkout templating
```

[Return to the main article](/techtalk/ux)

