+++
date = "2016-02-01T11:27:02-06:00"
draft = true
title = "Mongoose"

+++

Mongoose allows us to have access to the MongoDB commands, to include mongoose library type in your terminal:

```bash
npm install mongoose --save
```

In order to use mongoose in your js, you need:

```javascript
//Grab it in our project
var mongoose = require('mongoose')

//Connect to a MongoDB database
mongoose.connect('mongodb://localhost/mongoosedb')

//Create an Schema (is what is used to define attributes for our documents)
var Schema = mongoose.Schema

//A model using it
var Person = mongoose.model('Person', person)
```

Let's consider the following example:

```javascript
var mongoose = require('mongoose')
var Schema = mongoose.Schema

mongoose.connect('mongodb://localhost/mongoosedb')

var person = new Schema({
  name: String,
  lastName: String
})

var Person = mongoose.model('Person', person)

var object = {"name":"Jose Luis", "lastName":"De la Cruz"}

Person.create(object, function(err, doc){
  if(err){
    console.log('There is an error', err)
    return
  }

  console.log('Person was created', doc)
})

Person.find({}, function(err, docs){
  if(err){
    console.log('There is an error', err)
    return
  }

  docs.forEach(function(person){
    console.log('person:', person.name)
  })
})
```

In this example we are doing:

* Defining an Schema Person with name and lastname as String attributes
* Creating a new Json object with those attributes
* Creating a Person model
* Using `Person.create` to create a new person in MongoDB
* Using `Person.find` to get person collection from MongoDB

Don't forget to create package.json with `npm init` and install MongoDB in your computer before create any js file.

To execute this js file type in your terminal:

```bash
node mongoose_example.js
```

To download the project:

```bash
git clone https://github.com/josdem/ux-development.git
git fetch
git checkout mongoose
```

[Return to the main article](/techtalk/ux)

