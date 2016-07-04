+++
date = "2016-01-23T21:18:41-06:00"
draft = true
title = "Inquirer & Callback"

+++

Inquirer.js strives to be an easily embeddable and beautiful command line interface for Node.js

Inquirer.js should ease the process of:

* Providing error feedback
* Asking questions
* Parsing input
* Validating answers
* Managing hierarchical prompts

In order to install inquirer type in your terminal:

```bash
npm install inquirer --save
```

Usage prompt functionality:

```javascript
var inquirer = require("inquirer");
inquirer.prompt([/* Pass your questions in here */], function( answers ) {
  // Use user feedback for... whatever!!
})
```

Let's consider the following code:

```javascript
var inquirer = require('inquirer')
var guess = 8

var questions = [{
  type:'input',
  name:'number',
  message:'Think of a number between 1 and 10:'
}]

inquirer.prompt(questions, function(answers){
  var number = Number.parseInt(answers.number)
  if(number == guess){
    console.log('That is the number')
  } else {
    console.log('Try again')
  }
})
```

## Questions

A question object is a hash containing question related values:

* type: (String) Type of the prompt. Defaults: input - Possible values: input, confirm, list, rawlist, password
* name: (String) The name to use when storing the answer in the answers hash.
* message: (String|Function) The question to print. If defined as a function, the first parameter will be the current inquirer session answers.

inquirer.prompt uses a callback to receive the user response from command line as answers in our example.

To execute our js file type in your terminal:

```bash
node inquirer.js
```

To download the project:

```bash
git clone git@github.com:josdem/ux-development.git
git fetch
git checkout inquirer
```

[Return to the main article](/techtalk/ux)
