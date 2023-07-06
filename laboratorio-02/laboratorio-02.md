# Laboratorio MongoDB

Vamos a trabajar con el set de datos de Mongo Atlas _airbnb_. Lo puedes encontrar en este enlace: https://drive.google.com/drive/folders/1gAtZZdrBKiKioJSZwnShXskaKk6H_gCJ?usp=sharing

Para restaurarlo puede seguir las instrucciones de este videopost:
https://www.lemoncode.tv/curso/docker-y-mongodb/leccion/restaurando-backup-mongodb

> Acuerdate de mirar si en el directorio `/opt/app` del contenedor Mongo hay contenido de backups previos que haya que borrar

Para entregar las soluciones, añade un README.md a tu repositorio del bootcamp incluyendo enunciado y consulta (lo que pone '_Pega aquí tu consulta_').

## Introducción

En este base de datos puedes encontrar un montón de alojamientos y sus reviews, esto está sacado de hacer webscrapping.

**Pregunta**. Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?

```md
 ...

```

## Obligatorio

Esta es la parte mínima que tendrás que entregar para superar este laboratorio.

### Consultas

- Saca en una consulta cuantos alojamientos hay en España.

```js
 db.listingsAndReviews.count({ "address.country": "Spain" });
```

- Lista los 10 primeros:
  - Ordenados por precio de forma ascendente.
  - Sólo muestra: nombre, precio, camas y la localidad (`address.market`).

```js
db.listingsAndReviews
  .find({}, { _id: 0, name: 1, price: 1, beds: 1,"address.market": 1 })
  .sort({ price: -1 })
  .limit(10);
```

### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño o más.
  - Sólo muestra: nombre, precio, camas y baños.

```js
db.listingsAndReviews
.find({
    beds: {$gte: 4},
    bathrooms: {$gte: 2}
}, {_id: 0, name: 1, price: 1, beds: 1, bathrooms: 1})
```

- Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
  {
    beds: { $gte: 4 },
    bathrooms: { $gte: 2 },
    amenities: { $eq: "Wifi" },
  },
  { _id: 0, name: 1, price: 1, beds: 1, bathrooms: 1, amenities: 1 }
);

```

- Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota (_Pets allowed_).
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
  {
    beds: { $gte: 4 },
    bathrooms: { $gte: 2 },
    amenities: { $all: ["Wifi", "Pets allowed"] },
  },
  { _id: 0, name: 1, price: 1, beds: 1, bathrooms: 1, amenities: 1 }
);
```

### Operadores lógicos

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews (igual o superior a 88).
  - Sólo muestra: nombre, precio, camas, baños, rating y localidad.

```js
db.listingsAndReviews.find(
  {
    $and: [
      { price: { $lte: 50 } },
      { "review_scores.review_scores_rating": { $gte: 88 } },
      {
        $or: [
          { "address.market": "Barcelona" },
          { "address.country": "Portugal" },
        ],
      },
    ],
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
    "review_scores.review_scores_rating": 1,
    "address.location": 1
  }
);

```

- También queremos que el huésped sea un superhost y que no tengamos que pagar depósito de seguridad
  - Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de seguridad y localidad.

```js
db.listingsAndReviews.find(
   {
     $and: [
       { price: { $lte: 50 } },
       { "review_scores.review_scores_rating": { $gte: 88 } },
       { "host.host_is_superhost": true },
       { security_deposit: {$eq: 0}},
     ],
   },
   {
     _id: 0,
     name: 1,
     price: 1,
     beds: 1,
     bathrooms: 1,
     "address.location": 1,
     "host.host_is_superhost": 1,
     security_deposit: 1,
   }
 );
 ```

### Agregaciones

- Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
  - Nombre.
  - Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
  - Precio

```js
 db.listingsAndReviews.find(
  { "address.country": "Spain" },
  {
    name: 1, price: 1, "address.location": 1
});
```

- Queremos saber cuantos alojamientos hay disponibles por pais.

```js
db.listingsAndReviews.aggregate([
  {
    $group: {
      _id: "$address.country",
      totalRooms: { $sum: "$bedrooms" },
    },
  },
]);
```

## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España.

```js
db.listingsAndReviews.aggregate([
  {
    $group: {
      _id: null,
      avgAmount: { $avg: "$price" },
    },
  },
]);
```

- ¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```js
 db.listingsAndReviews.aggregate([
   {
     $group: {
       _id: "$address.country",
       avgAmount: { $avg: "$price" },
     },
   },
 ]);
```

- Repite los mismos pasos pero agrupando también por numero de habitaciones.

```js
 db.listingsAndReviews.aggregate([
   {
     $group: {
       _id: {
         country: "$address.country",
         bedrooms: "$bedrooms",
       },
       avgAmount: { $avg: "$price" },
     },
   },
 ]);
```

## Desafio

Queremos mostrar el top 5 de alojamientos más caros en España, con los siguentes campos:

- Nombre.
- Precio.
- Número de habitaciones
- Número de camas
- Número de baños
- Ciudad.
- Servicios, pero en vez de un array, un string con todos los servicios incluidos.

```js

db.listingsAndReviews.aggregate([
    {
        $match: {"address.country": "Spain"}
    },
  {
    $sort: {
      price: -1,
    },
  },
  {
    $limit: 5,
  },
  {
    $project: {
      _id: 0,
      name: 1,
      bedrooms: 1,
      beds: 1,
      bathrooms: 1,
      price: 1,
      amenities: 1
      "address.market": 1,
    },
  },
]);
```
