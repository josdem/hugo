+++
date = "2015-10-01T19:28:10-05:00"
draft = true
title = "Jugoterapia API"

+++

Jugoterapia has three services

**Get recipe categories**

Returns all juice categories

* URL: http://jugoterapia.josdem.io/jugoterapia-server/categories
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl http://jugoterapia.josdem.io/jugoterapia-server/beverage/categories
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

* URL http://jugoterapia.josdem.io/jugoterapia-server/categories/{id}
* Path Parameter: category id
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl http://jugoterapia.josdem.io/jugoterapia-server/categories/1
```

Example result:
```json
[
  {
    "id": 1,
    "name": "Jugo para evitar los calambres",
    "ingredients": "3 Tallos de apio,1/4 De pepino",
    "recipe": "Mezcla 3 tallos de apio y 1/4 de pepino y procésalos en el extractor de jugos. Si lo deseas, puedes rebajarlo con agua."
  },
  {
    "id": 6,
    "name": "Jugo para la anemia y amenorrea",
    "ingredients": "1/4 Taza de remolacha,1 Zanahoria",
    "recipe": "Mezcla los dos ingredientes en el extractor de jugos, agrega un vaso con agua y si lo prefieres endulza con una cucharada de miel de abeja. Cuando se consumen grandes cantidades la piel a veces se torna amarilla. Esta coloración es inocua."
  },
  {
    "id": 7,
    "name": "Jugo para el dolor de muelas",
    "ingredients": "1/4 Taza de espinaca,1 Tomate pequeño,1 diente de ajo",
    "recipe": "Mezcla todos los ingredientes en el extractor o en la licuadora, agregando un vaso de agua, si prefieres puede endulzar con un poco de miel de abeja."
  },
  {
    "id": 8,
    "name": "Jugo para los problemas de la digestión",
    "ingredients": "1 Remolacha mediana",
    "recipe": "Licua la remolacha junto con un vaso de agua, si lo prefieres puedes endulzarlo con una cucharada de miel de abeja. Su contenido de fibra dietética y pectina como antidiarréico ayuda a corregir los problemas de digestión"
  },
  {
    "id": 9,
    "name": "Jugo para curar el acné (Pepino)",
    "ingredients": "2 Zanahorias,1 Pepino,1 Rama de apio,1/2 Betabel,2 Hojas de espinacas,1 Jitomate,1 Hoja de lechuga,5 Hojas de perejil",
    "recipe": "Limpia la piel de los excesos de producción de las glándulas sebáceas. Empleando el extractor obtenga el jugo de dos zanahorias crudas y peladas; a continuación, licúa con tres tazas de agua, un pepino sin cáscara, una rama de apio y 1/2 de betabel crudo y pelado, así como dos hojas de espinaca, un jitomate pelado, una hoja de lechuga y 5 hojas de perejil. Cuela el resultado y toma un vaso a cualquier hora del día."
  }
]
```

**Get beverage by id**

Returns beverage given its id

* URL http://jugoterapia.josdem.io/jugoterapia-server/beverages/{id}
* Path Variable: beverage id
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl http://jugoterapia.josdem.io/jugoterapia-server/beverages/35
```

Example result:
```json
{
  "id": 35,
  "name": "Jugo nutritivo (Zanahoria)",
  "ingredients": "4 Zanahorias,1 Tallo de apío,1 Pera,5 hojas de espinacas",
  "recipe": "Lava perfectamente todos los ingrendientes. Pasa la zanahoria por el extractor, el apio, las espinacas y la pera. Mezcla todo perfectamente y bebe de inmediato. La espinaca es una excelente fuente de hierro. Promueve el transporte y depósito de oxí­geno en los tejidos, aumenta la fuerza muscular, ayuda a bajar de peso, favorece el tránsito intestinal, beneficia a mujeres embarazadas y niños debido a su contenido de ácido fólico (vitamina B9), mejora la visión y mantiene la presión arterial balanceada."
}
```

[Return to the main article](/jugoterapia/jugoterapia)
