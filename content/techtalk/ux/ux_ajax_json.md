+++
date = "2015-10-25T12:36:25-06:00"
draft = true
title = "Parsing Json with Ajax"

+++

We want to consume a GET service and parsing an JSON response using Ajax and JQuery, so here is what we need.

JSON example:

```json
{ uuid: "7d6bc74b9a33257e29bb0206e2b00f434deb51a0", nombre: "Jose Luis", apellidoPaterno: "De la Cruz", apellidoMaterno: "Morales", rfc: "rfc", fechaNacimiento: 1445640529000, 18 more… }
```

Let's imagine this scenario: when user type rfc in a form and click on send button, we consume our service retrieve an JSON and fill some input text fields in a form, so our html file looks like this:

```html
<html>
<head>
  <meta content="text/html;charset=utf-8" http-equiv="Content-Type">
</head>

<body>
<form id="simulator">
  <div>
    <label for="rfc">RFC</label>
    <input type="text" name="rfc" id="rfc"/>
  </div>
  <div>
    <label for="nombre">nombre</label>
    <input type="text" name="nombre" id="nombre"/>
  </div>
  <div>
    <label for="apellidoPaterno">apellidoPaterno</label>
    <input type="text" name="apellidoPaterno" id="apellidoPaterno"/>
  </div>
  <div>
    <label for="apellidoMaterno">apellidoMaterno</label>
    <input type="text" name="apellidoMaterno" id="apellidoMaterno"/>
  </div>
</form>
<button>Send</button>
</body>
</html>
```

And we define an ajax to do the trick as follow:

```javascript
<head>
<script type="text/javascript" src="https://code.jquery.com/jquery-2.1.4.min.js"></script>
<script type="text/javascript">
$(document).ready(function(){
  $("button").click(function(){

    $.ajax('//localhost:8080/client', {
      method: 'GET',
      dataType: 'json',
      data: {
        rfc: $('#rfc').val()
      }
    })
    .done(function(data) {
      var simulator = $('#simulator');
      simulator.find('#rfc').val(data.rfc);
      simulator.find('#nombre').val(data.nombre);
      simulator.find('#apellidoPaterno').val(data.apellidoPaterno);
      simulator.find('#apellidoMaterno').val(data.apellidoMaterno);
    })
    .fail(function(data, status){
      console.log(data, status)
    });
  });
});
</script>
</head>
```

In this code we can see the following aspects:

* Setting the dataType to 'json' to convert data automatically
* Sending rfc information as `rfc: $('#rfc').val()`
* When we receive data as JSON we fill our simulator form with find method
* If fails we send data and status to the console

[Return to the main article](/techtalk/ux)
