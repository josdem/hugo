+++
date = "2015-10-01T19:28:10-05:00"
draft = true
title = "Jugoterapia API"

+++

Jugoterapia has three services

**Get juice categories**

Returns all juice categories

* URL: http://jugoterapia.josdem.io/jugoterapia-server/beverage/categories
* Method: GET
* Response format: Json

Example request using httpie:
```bash
http http://jugoterapia.josdem.io/jugoterapia-server/beverage/categories
```

Example result:
```json
[
   {
      "id":1,
      "name":"Curativos"
   },
   {
      "id":2,
      "name":"Energizantes"
   },
   {
      "id":3,
      "name":"Saludables"
   },
   {
      "id":4,
      "name":"Estimulantes"
   }
]
```

**Get beverages from category**

Returns all beverages given a category id

* URL http://jugoterapia.josdem.io/jugoterapia-server/beverage/beverages
* Parameter: categoryId
* Method: POST
* Response format: Json

Example request using httpie:
```bash
http --form POST http://jugoterapia.josdem.io/jugoterapia-server/beverage/beberages categoryId=1
```

Example result:
```json
[
   {
      "id":1,
      "ingredients":"* 3 Tallos de apio\n* ¼ De pepino",
      "name":"Jugo para evitar los calambres",
      "recipe":"Mezcla 3 tallos de apio y ¼ de pepino y procésalos en el extractor de jugos. Si lo deseas, puedes rebajarlo con agua."
   },
   {
      "id":6,
      "ingredients":"* ¼ Taza de remolacha\n\n* 1 Zanahoria",
      "name":"Jugo para la anemia y amenorrea",
      "recipe":"Mezcla los dos ingredientes en el extractor de jugos, agrega un vaso con agua y si lo prefieres endulza con una cucharada de miel de abeja. Cuando se consumen grandes cantidades la piel a veces se torna amarilla. Esta coloración es inocua."
   },
   {
      "id":7,
      "ingredients":"* ¼ Taza de espinaca\n\n* Un Tomate pequeño\n\n* Un diente de ajo",
      "name":"Jugo para el dolor de muelas",
      "recipe":"Mezcla todos los ingredientes en el extractor o en la licuadora, agregando un vaso de agua, si prefieres puede endulzar con un poco de miel de abeja."
   },
   {
      "id":8,
      "ingredients":"* Remolacha mediana",
      "name":"Jugo para los problemas de la digestión",
      "recipe":"Licua la remolacha junto con un vaso de agua, si lo prefieres puedes endulzarlo con una cucharada de miel de abeja. Su contenido de fibra dietética y pectina como antidiarréico ayuda a corregir los problemas de digestión"
   }
]
```

**Get beverage from id**

Returns beverage given a beverage id

* URL http://jugoterapia.josdem.io/jugoterapia-server/beverage/beverage
* Parameter: beverageId
* Method: POST
* Response format: Json

Example request using httpie:
```bash
http --form POST http://jugoterapia.josdem.io/jugoterapia-server/beverage/beverage beverageId=83
```

Example result:
```json
{
    "id": 83,
    "ingredients": "* 2 Naranjas\n* 1 Toronja\n* 1 Ramito de perejil",
    "name": "Jugo antigripal (Perejil)",
    "recipe": "Exprime el jugo de las naranjas y de la toronja, pasa por el extractor de jugos el perejil, revuelve todos los jugos en un vaso y toma recién hecho diariamente."
}
```

[Return to the main article](/jugoterapia/jugoterapia)
