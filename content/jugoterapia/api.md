+++
date = "2015-10-01T19:28:10-05:00"
draft = true
title = "Jugoterapia API"

+++

Jugoterapia has three services

**Get recipe categories**

Returns all juice categories

* URL: https://webflux.josdem.io/categories/
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl https://webflux.josdem.io/categories/
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

* URL https://webflux.josdem.io/categories/{categoryId}/beverages
* Path Parameter: categoryId
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl https://webflux.josdem.io/categories/1/beverages
```

Example result:
```json
[
  {
    id: 1,
    name: "Jugo para evitar los calambres",
    ingredients: "3 tallos de apio 1/4 de pepino",
    recipe: "Mezcla 3 tallos de apio y 1/4 de pepino y procésalos en el extractor de jugos. Si lo deseas puedes debajarlo con agua"
  },
  {
    id: 2,
    name: "Jugo para la anemia y amenorrea",
    ingredients: "1/4 Taza de remolacha,1 Zanahoria",
    recipe: "Mezcla los dos ingredientes en el extractor de jugos, agrega un vaso con agua y si lo prefieres endulza con una cucharada de miel de abeja. Cuando se consumen grandes cantidades la piel a veces se torna amarilla. Esta coloración es inocua."
  },
  {
    id: 3,
    name: "Jugo para el dolor de muelas",
    ingredients: "1/4 Taza de espinaca,1 Tomate pequeño,1 diente de ajo",
    recipe: "Mezcla todos los ingredientes en el extractor o en la licuadora, agregando un vaso de agua, si prefieres puede endulzar con un poco de miel de abeja."
  },
  {
    id: 4,
    name: "Jugo para los problemas de la digestión",
    ingredients: "1 Remolacha mediana",
    recipe: "Licua la remolacha junto con un vaso de agua, si lo prefieres puedes endulzarlo con una cucharada de miel de abeja. Su contenido de fibra dietética y pectina como antidiarréico ayuda a corregir los problemas de digestión"
  },
  {
    id: 5,
    name: "Jugo para curar el acné (Pepino)",
    ingredients: "2 Zanahorias,1 Pepino,1 Rama de apio,1/2 Betabel,2 Hojas de espinacas,1 Jitomate,1 Hoja de lechuga,5 Hojas de perejil",
    recipe: "Limpia la piel de los excesos de producción de las glándulas sebáceas. Empleando el extractor obtenga el jugo de dos zanahorias crudas y peladas; a continuación, licúa con tres tazas de agua, un pepino sin cáscara, una rama de apio y 1/2 de betabel crudo y pelado, así como dos hojas de espinaca, un jitomate pelado, una hoja de lechuga y 5 hojas de perejil. Cuela el resultado y toma un vaso a cualquier hora del día."
  },
  {
    id: 6,
    name: "Jugo antireumático",
    ingredients: "4 Zanahorias,2 Pepinos,1/2 Col,1 Diente de ajo",
    recipe: "Usa el extactor para obtener el jugo de cuatro zanahorias crudas y peladas, el cual se mezclará en la licuadora con dos pepinos sin cáscara, 1/2 de col cruda y un diente de ajo pelado. La bebida se sirve sin colar, pudiendo agregar un poco de hielo picado, de manera que sea aún más fresca. Además de disminuir las molestias reumáticas, límpia al organismo y previene anemia."
  },
  {
    id: 7,
    name: "Jugo para el colesterol",
    ingredients: "1 toronja,1 rama de apio,5 hojas de perejíl,1 trozo de sábila",
    recipe: "Gracias a sus propiedades depurativas este jugo reduce los niveles de los compuestos grasos que se acumulan en la sangre. Exprime una toronja, pon al extractor una rama y mezcla ambos jugos en la licuadora con cinco hojas de perejil y un trozo de sábila (de aproximadamente cinco centimetros por lado). Bebe, sin colar, un vaso en ayunas."
  },
  {
    id: 8,
    name: "Jugo para el extreñimiento",
    ingredients: "1 Manzana,1 Pera,Azucar al gusto",
    recipe: "Comienza el dia con jugo de manzana y pera sin cáscara utilizando el extractor. El jugo de manzana contiene sorbitol, azucar natural con cualidades laxantes."
  },
  {
    id: 9,
    name: "Jugo para el estrés",
    ingredients: "2 Pepinos,1 Centro de lechuga,Sal y pimienta al gusto",
    recipe: "Mezcla en la licuadora dos pepinos sin cáscara, el centro de una lechuga y una taza de agua; posteriormente bate la mezcla obtenida en una coctelera,con hielo picado sal y pimienta. Sus propiedades sedantes tienen gran resultado en personas nerviosas; se recomienda tomarlo por la noche."
  },
  {
    id: 10,
    name: "Jugo para el insomnio",
    ingredients: "1 Lechuga orejona",
    recipe: "La clorofila característica de las hojas verdes es un gran calmante del sistema nervioso. Tome un vaso de jugo de lechuga orejona (preparado en el extractor) una hora antes de dormir."
  },
  {
    id: 11,
    name: "Jugo para la irritación de la garganta",
    ingredients: "2 Rebanadas de piña,1 Rodaja de jengibre",
    recipe: "Licue el jugo de 2 rebanadas de piña (obtenido usando el extractor) junto con una rodaja de medio centí­metro de jengibre; además de delicioso coctel es muy efectivo para acelerar la curación de una garganta irritada, ya que ambos ingredientes contienen agentes naturales antiinflamatorios."
  },
  {
    id: 12,
    name: "Jugo para los dolores menstruales",
    ingredients: "1 Naranja,1/2 Nopal,2 cm de sábila,1/4 De limón,200 Mililitros de agua mineral",
    recipe: "Vierte en la licuadora el jugo de una naranja, 1/2 nopal, 2 centímetros de sábila, 1/4 de limón sin cáscara y 200 mililitros de agua mineral. Mézclalo y tómalo sin colar en ayunas, durante los cinco dias previos al inicio de su periodo."
  },
  {
    id: 13,
    name: "Jugo para la perdida de potasio",
    ingredients: "1/2 Vaso de jugo de naranja,2 Zanahorias en jugo,1 Plátano",
    recipe: "La falta de este mineral afecta el funcionamiento de los riñones y, por tanto, la eliminación de toxinas a través de la orina, así­ como al adecuado ritmo cardiaco y presión arterial. Para recuperar el potasio perdido por diarrea o por el efecto de diuréticos, mezcla en la licuadora 1/2 vaso de jugo de naranja, el jugo de 2 zanahorias peladas (usando el extractor) y un plátano."
  },
  {
    id: 14,
    name: "Jugo para la retención de lí­quidos",
    ingredients: "1/2 Jugo de uva natural,1/2 Jugo de melón",
    recipe: "Mezcla el jugo de uvas frescas y de melón, sin añadir agua, una vez al día; se trata de diuréticos naturales"
  },
  {
    id: 15,
    name: "Jugo para la tos y la bronquitis",
    ingredients: "1/2 Piña,3 Ciruelas,1/2 Mango",
    recipe: "Licua el jugo de 1/2 piña (Empleando el extractor), 1/2 mango y 3 ciruelas peladas. La bebida lubrica las membranas de la garganta, además que la piña contiene una enzima llamada biomelina, que actúa como antiinflamatoria y contrarrestra las afecciones en las vías respiratorias, gracias a sus propiedades antibióticas."
  },
  {
    id: 16,
    name: "Jugo contra la obesidad",
    ingredients: "2 Zanahorias,1 Betabel,1 Apio",
    recipe: "En este punto son más indicados los jugos de verduras. Tómalos a cualquier hora. Mezcla varias verduras: Zanahoria, apio y betabel ó zanahoria, lechuga, rábano, pepino, betabel, apio, jitomate y dos dientes de ajo. Otra opción de mezcla es alfalfa con piña y miel. Prepáralos en el extractor."
  },
  {
    id: 17,
    name: "Jugo para limpiar el organismo",
    ingredients: "5 Hojas de acelga,1 Apio,1/4 Piña",
    recipe: "Mezcla todos los ingredientes y licualos, agregando un vaso de agua y endulzando con una cucharada de miel de abeja. Tómalo en ayunas diariamente."
  },
  {
    id: 18,
    name: "Jugo para desintoxicar",
    ingredients: "1 Mango,1 Durazno,1 Manzana,10 Uvas",
    recipe: "Mezcla durazno con mango y manzana o uva con zanahoria y zarzamora, toma este jugo en las mañanas"
  },
  {
    id: 19,
    name: "Jugo anticolesterol",
    ingredients: "1 Toronja,1 Rama de apio,1 Rama de perejil,1 trozo de sábila",
    recipe: "Mezcla los ingredientes en el extractor de jugos, toma este jugo durante un mes en la mañana, si deseas lo puede endulzar con una cucharada de miel de abeja."
  },
  {
    id: 20,
    name: "Jugo laxante suave",
    ingredients: "1 Manzana,1 Pera",
    recipe: "Inicia el dí­a con un jugo de manzana y pera. El extracto de manzana contiene sorbitol, que es una azúcar natural con mil propiedades laxantes."
  },
  {
    id: 21,
    name: "Jugo para combatir el mal aliento",
    ingredients: "250 Gramos de zanahoria,125 Gramos de espinaca,125 Gramos de pepino",
    recipe: "Licua todos los ingredientes, sin agregarle agua ni azúcar. Toma medio vaso de la mezcla, después de las comidas."
  },
  {
    id: 22,
    name: "Jugo para el estrés, nervios e insomnio",
    ingredients: "1 Limón,3 Hojas de Lechuga orejona,1 Pepino,Todas las verduras verdes",
    recipe: "Toma jugos de verduras y ensaladas verdes. Un vaso de lechuga orejona (en extractor). Una hora antes de dormir."
  },
  {
    id: 23,
    name: "Jugo para combatir las imperfecciones de la piel",
    ingredients: "125 Gramos de zanahoria,50 Gramos de espinacas,50 Gramos de espárragos",
    recipe: "Licúa todos los ingredientes, sin agregar agua, ya que tiene que quedar el zumo de los ingredientes, si lo deseas tómalo en ayunas un vaso al despertar."
  },
  {
    id: 24,
    name: "Jugo contra la diarrea",
    ingredients: "1 Manzana,1 Cucharada de salvado de trigo",
    recipe: "Para comenzar, realiza un jugo de la manzana ayudándose con el extractor de jugos. Enseguida agrega una cucharada de salvado de trigo y mezclalo perfectamente. Tómalo al medio dia, despues de la comida."
  },
  {
    id: 25,
    name: "Jugo para la gripe",
    ingredients: "150 Gramos de col,150 Gramos de naranja",
    recipe: "Licúa todos los ingredientes, agregando un poco de agua si es necesario y una cucharada de miel para darle un poco mas de sabor. Toma el jugo tres veces al día."
  },
  {
    id: 26,
    name: "Jugo antioxidante",
    ingredients: "1 Vaso de yogurt natural,1 Cucharada de almendras picadas,1 Cucharada de cacahuate natural picado,1 Cucharada de semillas de girasol picadas,1 Cucharada de miel de abeja",
    recipe: "Bate todos los ingredientes a velocidad alta, hasta que no le queden grumos. Bebe en seguida. Este jugo es energético, antioxidante ya que contiene una buena cantidad de fibra, ayuda incluso a disminuir el colesterol"
  },
  {
    id: 27,
    name: "Jugo para las molestias de las varices",
    ingredients: "6 Fresas,1 Toronja,1 Cucharada de miel de abeja",
    recipe: "Lava perfectamente bien todos los ingredientes. Pasa las fresas por el extractor de jugos. Exprime la toronja. Mezcla ambos jugos en la licuadora y tómalo de inmediato"
  },
  {
    id: 28,
    name: "Jugo para la menopausia",
    ingredients: "5 Fresas,2 Tazas de leche de soya natural,1 Kiwi maduro,1 Miel de abeja",
    recipe: "Primero lava y desinfecta muy bien las fresas, retira el rabo y las dejas otra vez con las gotas para desinfectar. Ahora retira la cáscara del kiwi y lo partes en cuatro. En la licuadora vierte la leche, las fresas y los trozos de kiwi. Licúa muy bien hasta que todo este incorporado por la mañana y otra al medio dia. Si gustas puede endulzar con la miel a su gusto, decora con unas hojuelas de almendras."
  },
  {
    id: 29,
    name: "Jugo para reforzar las uñas",
    ingredients: "2 Rabanitos de bolita,1 Pepino,1 Papa,4 Zanahorias",
    recipe: "Lava las zanahorias y el pepino, corta en tiras este último. Lava y retira la cáscara de la papa y la corta en tiras gruesas de manera que entren en el extractor. También lava y corta a la mitad los rabanitos. Mete los pepinos, rabanos, papa y zanahorias en el extractor. Tome este jugo a tragos durante todo el día."
  },
  {
    id: 30,
    name: "Jugo para el higado",
    ingredients: "1 Tallo de apio,1/2 Taza de col picada,1 Toronja",
    recipe: "Lava y desinfecta el apio y la col. Expime la toronja o dos según el jugo que necesite. Pase por el extractor la col y el apio para obtener un jugo. Enseguida revuelve los tres jugos muy bien. Toma un vaso por la mañana en ayunas 2 veces a la semana."
  },
  {
    id: 31,
    name: "Jugo para prevenir el cancer",
    ingredients: "2 Cucharas de perejil,2 Naranjas,1 Limón,5 Jitomates,1 Vaso de agua",
    recipe: "Lava y desinfecta muy bien el perejil y pica finalmente. Exprime el limón, las naranjas y en la licuadora pon el jitomate para hacer el jugo de este. Enseguida vierte los jugos de limón y de naranja. Licúa muy bien hasta que se integren todos los ingredientes. Cuela el jugo en un vaso y toma un vaso en ayunas dos veces al mes."
  },
  {
    id: 32,
    name: "Jugo para las reumas",
    ingredients: "4 Jitomates,4 Cubos de hielo,2 Limones,2 Naranjas",
    recipe: "Lava las naranjas, los limones y los jitomates. Exprime los limones, de la misma forma las naranjas hasta tener 1 taza de jugo. Pon los dos jugos en la licuadora, los hielos y los tomates. Licúa perfectamente bien hasta que se muela todo muy bien. Cuela en un vaso el jugo y toma una vez al dia por las mañanas."
  },
  {
    id: 33,
    name: "Jugo para los huesos",
    ingredients: "600 Gramos de coco rallado,1/2 Taza de leche,6 Cubos de hielo",
    recipe: "Pon en la licuadora, el coco rallado, la leche y una taza de agua. Licúa todo muy bien hasta que se incorporen todos los ingredientes. Este licuado lo puedes tomar como sustituto de refresco y es saludable para los huesos."
  },
  {
    id: 34,
    name: "Jugo para la hipertensión",
    ingredients: "1/2 Taza de agua,1 Taza de fresas,1 Plátano,1 Pera",
    recipe: "Lava y desinfecta muy bien las fresas. Retira la cáscara del plátano y lo cortalo en trozos. Quita la cáscara de la pera y corta a la mitad en trozos. Pon en la licuadora el agua, las fresas, el plátano y la pera, licua muy bien. Vacía el licuado en un vaso y toma de inmediato, por una vez a la semana"
  },
  {
    id: 35,
    name: "Jugo para aliviar las hemorroides",
    ingredients: "8 Zanahorias,1 Ramo de espinacas,1 Vaso de agua",
    recipe: "Lava y desinfecta muy bien las espinacas, pon el agua en la licuadora y licúa perfectamente bien las espinacas de manera que salga un jugo. Cuela el jugo de espinacas. Lava las zanahorias, pasa estas por el extractor, cuela el jugo. Junta el jugo de espinacas con el de zanahoria. Toma un vaso por la mañana una vez a la semana por un mes."
  },
  {
    id: 36,
    name: "Jugo para las articulaciones",
    ingredients: "1 Taza de leche descremada de soya,4 Cubos de hielo,1/2 Vaso de piña,1 Pizca de canela molida,1/2 Taza de fresas picadas,1 Cucharada de extracto de vainilla,1/2 Taza de melón chino picado,1 Cucharada de miel de abeja",
    recipe: "Primero lavas y desinfecta perfectamente bien las fresas. Pon en la licuadora la leche de soya, el jugo de piña, las fresas picadas, la vainilla, el melón y la miel. Licúa muy bien para que los ingredientes se incorporen y se forme una mezcla homogénea. Enseguida sirve el licuado en un vaso grande con los hielos y espolvorea la canela, toma de inmediato"
  },
  {
    id: 37,
    name: "Jugo para la osteoporosis",
    ingredients: "1/2 Taza de piña en trozos,1 Taza de helado sabor fresa,1 Taza de leche de coco,1 Cucharadita de miel",
    recipe: "Vierte en la licuadora la piña y agrega poco a poco la leche. Después añde los demás ingredientes, procurando que la mezcla quede espumosa."
  },
  {
    id: 38,
    name: "Jugo para quemar grasa corporal",
    ingredients: "1/2 Pimiento verde,1 Manojo de berros,1 Taza de agua mineral,1 Nabo mediano,3 Tallos de apio",
    recipe: "Mezclar todos los ingredientes perfectamente (licuado o en extractor de jugos), se cuelan. Tómarlo recién preparado."
  },
  {
    id: 39,
    name: "Para el funcionamiento de los riñones",
    ingredients: "3 Zanahorias,1 Rama de apio,3 Espárragos verdes,2 Rábanos rojo,1 y 1/2 Taza de espinacas",
    recipe: "Se mezclan todos los ingredientes perfectamente (licuado o en extractor de jugos) se cuelan, tómalo recién preparado con comida del mediodia."
  },
  {
    id: 40,
    name: "Licuado con vegetales para bajar de peso",
    ingredients: "1 Vaso de toronja,1 Pedazo de papaya,1 Pedazo de piña,1 Pedazo de nopal,1/2 Manzana,1 Cucharada de algas marinas",
    recipe: "Licuar y tomar sin colar. La toronja, papaya, nopal, piña, manzana y algas marinas son los principales ingredientes naturales que solos o combinados pueden ayudarte a bajar de peso quemando grasa, facilitando la eliminación de líquidos retenidos y mejorando la evacuación del intestino, además que las algas marinas mejoran el funcionamiento de la glándula tiroides que regula nuestro metabolismo."
  },
  {
    id: 41,
    name: "Jugo con col para problemas de digestión",
    ingredients: "1 Vaso de jugo de zanahoria,1 Jugo de una hoja de col,1 Jugo de una papa cruda,1 Jugo de una manzana.",
    recipe: "Extraer el jugo y tomar, puedes agregar miel de abeja al gusto. La col pertenece a la familia de las crucíferas, el jugo de la col combinado con el aceite de oliva aplicado directamente en la piel ayuda a granos, ampollas y arrugas prematuras. El jugo de col tomado ayuda en casos de artritis, diarreas, úlceras gástricas, afecciones de ví­as respiratorias y eliminación de líquidos."
  },
  {
    id: 42,
    name: "Jugo con col para enfermedades respiratorias",
    ingredients: "1 Vaso de jugo de naranja,1 Hoja de col,1 Mango,1 Kiwi,1 Pizca de jengibre,Miel de abeja al gusto",
    recipe: "Licuar y tomar preferentemente sin colar. La col contiene 50 mg. de vitamina C, 100 mg. de vitamina A, 68 mg. de azufre por cada 100 grs. Micronutrientes necesarios para prevenir enfermedades infeccionsas y hacer mas lento el envejecimiento."
  },
  {
    id: 43,
    name: "Jugo con avena y manzana para problemas de colesterol alto",
    ingredients: "1 Vaso de jugo de naranja,1/4 De tallo de apio,1 Pedazo de chayote,1 Cucharada de avena,2 Cucharadas de miel de abeja",
    recipe: "Licuar y tomar sin colar. El chayote, avena, manzana y apio son algunos alimentos de origen vegetal que consumidos frecuentemente; ayudan a bajar los niveles de colesterol malo en la sangre."
  },
  {
    id: 44,
    name: "Jugo con zanahoria y espinaca para problemas de la piel",
    ingredients: "1 Vaso de jugo de naranja,1 Zanahoria chica,1 Hoja de espinacas,1 Cucharada sopera de aceite de olivo,1 Jugo de limón chico,1 Pizca de sal",
    recipe: "Licuar y tomar sin colar. La vitamina A, la vitamina C y la vitamina E, son necesarias para la buena salud de la piel. Las podemos encontrar en la zanahoria, la espinaca, el limón y el aceite de olivo."
  },
  {
    id: 45,
    name: "Jugo con citricos para prevenir infecciones y evitar la oxidación celular",
    ingredients: "1 Vaso de jugo de piña,1 Tallo de apio,1 Durazno",
    recipe: "Licuar y tomar sin colar. Las frutas cítricas se caracterizan por su sabor ácido y por su alto contenido de vitamina C, la vitamina C nos ayuda para hacernos mas resistentes a las enfermedades infecciosas o a luchar en contra de ellas. Las frutas cítricas son la toronja, naranja, mandarina, lima y limón."
  },
  {
    id: 46,
    name: "Jugo de toronja para los problemas de presión alta",
    ingredients: "1 Vaso de jugo de toronja,1 Manzana,1 Rebanada de papaya,Miel de abeja al gusto",
    recipe: "Licuar y tomar sin colar. La toronja, además de su conocido efecto adelgazante, tiene un bajísimo contenido de sodio y grasa, así como una elevada proporción de potacio lo cual lo hace recomendable para aquellos que padecen insuficiencia cárdiaca, especialmente los hipertensos. La toronja tiene un ligero efecto diurético especialmente si se consume el fruto con su pulpa, lo que hace que tenga un mejor efecto hacia el aparato circulatorio."
  },
  {
    id: 47,
    name: "Licuado de semilla de calabaza para prevenir la osteoporosis",
    ingredients: "1 Vaso de leche de vaca o soya,1 Cucharada de semillas de calabaza pelada,1 Cucharada de amaranto,1 Cucharada de ajonjoli,1 Mango o fruta de temporada,Miel de abeja al gusto",
    recipe: "Licuar y tomar sin colar. Las semillas de calabaza contienen por cada 100 gramos 1,100 miligramos de fósforo, uno de los minerales mas importantes en la composición de los huesos. El 40% de los huesos es el calcio, y el 45% de este calcio esta en forma de fostato cálcico. Por eso para prevenir problemas de osteoporosis es importante consumir semillas oleoginosas en el desayuno o cena."
  },
  {
    id: 48,
    name: "Licuado con leche de soya para problemas de menopausia",
    ingredients: "1 Vaso de agua,2 Cucharadas soperas de leche de soya,1 Cucharada de linaza molida,1 Pera",
    recipe: "Licuar y tomar sin colar. La leche de soya y el queso derivado de ella conocido como tofu. Aportan isoflavonas (hormonas femeninas de origen vegetal) que ejercen una acción similar a los estrógenos, pero sin efectos secundarios. Al unirse a los receptores celulares de los estrógenos, las isoflavonas indicen efectos favorables de los estrógenos naturales: aumento de la mineralización ósea, protección frente a la arteosclerosis y sensación de bienestar."
  },
  {
    id: 49,
    name: "Recuperarse de diarrea",
    ingredients: "1/2 Vaso de jugo de naranja,1/2 Melón,2 Zanahorias licuadas,1 Plátano",
    recipe: "Licuar y tomar sin colar. Este jugo es indicado para recuperar el potasio perdido por diarrea o por efecto de diuréticos."
  },
  {
    id: 50,
    name: "Jugo contra el estreñimiento",
    ingredients: "1 Manzana,1 Pera,1 Vaso de jugo de naranja",
    recipe: "La manzana contiene sorbitol una azúcar natural con propiedades laxantes, la fibra de la pera ayudará a mejorar tu sistema digestivo."
  },
  {
    id: 52,
    name: "Jugo contra el acné (Espárrago)",
    ingredients: "125 Gramos de zanahoria,50 Gramos de espinacas,50 Gramos de espárragos",
    recipe: "Se licúan los elementos y se toman en ayunas una vez al día"
  },
  {
    id: 53,
    name: "Jugo contra la anemia",
    ingredients: "200 Gramos de zanahoria,100 Gramos de espinaca",
    recipe: "Este jugo cumple la función de aumentar la producción de glóbulos rojos. Se licúan los ingredientes y se toman después del almuerzo y de la cena"
  },
  {
    id: 54,
    name: "Jugo contra la diarrea",
    ingredients: "1 Jugo de manzana,1 Cucharada de salvado de trigo",
    recipe: "Tómalo a medio día, después de la comida"
  },
  {
    id: 55,
    name: "Jugo contra el dolor de cabeza",
    ingredients: "300 Gramos de col,100 Gramos de apio",
    recipe: "Una vez licuados los ingredientes se toman en ayunas una vez al dia. La col contiene magnesio, potasio, calcio, vitaminas A, B1, B2. El apio contiene magnesio, hierro, yodo, vitaminas A, B y C."
  },
  {
    id: 56,
    name: "Jugo antigripal (Perejil)",
    ingredients: "2 Naranjas,1 Toronja,1 Rama de perejil",
    recipe: "Exprime el jugo de las naranjas y de la toronja, pasa por el extractor de jugos el perejil, revuelve todos los jugos en un vaso y toma recién hecho diariamente."
  }
]
```

**Get beverage by id**

Returns beverage given its id

* URL https://webflux.josdem.io/beverages/{beverageId}
* Path Variable: beverageId
* Method: GET
* Response format: Json

Example request using curl:

```bash
curl https://webflux.josdem.io/beverages/66
```

Example result:
```json
{
  id: 66,
  name: "Jugo nutritivo (Zanahoria)",
  ingredients: "4 Zanahorias,1 Tallo de apío,1 Pera,5 hojas de espinacas",
  recipe: "Lava perfectamente todos los ingrendientes. Pasa la zanahoria por el extractor, el apio, las espinacas y la pera. Mezcla todo perfectamente y bebe de inmediato. La espinaca es una excelente fuente de hierro. Promueve el transporte y depósito de oxí­geno en los tejidos, aumenta la fuerza muscular, ayuda a bajar de peso, favorece el tránsito intestinal, beneficia a mujeres embarazadas y niños debido a su contenido de ácido fólico (vitamina B9), mejora la visión y mantiene la presión arterial balanceada."
}
```

[Return to the main article](/jugoterapia/jugoterapia)
