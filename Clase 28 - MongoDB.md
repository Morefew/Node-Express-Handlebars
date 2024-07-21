# Clase 28 - MongoDB
__cont... [Clase 24 - 27 Node, Express, Handlebars](Clase%2024%20-%2027%20Node,%20Express,%20Handlebars.md)

Instalar MongoDb Community Edition
Instalar MongoDB Compass - GUI para realizar operaciones en la base de datos.

## Conexión con MongoDB usando Node.js.
La instalación del controlador Node.js se puede realizar utilizando el administrador de paquetes de nodo (NPM), como se muestra en el siguiente código:

```javascript
npm install mongodb
```




## Configurando MongoDB con Mongoose.

### ¿Qué es Mongoose?
Mongoose es una biblioteca de modelado de datos de objetos (ODM) para MongoDB y Node.js. 

### Características y beneficios clave cuando se trabaja con MongoDB:

- **Definición de esquema:** Mongoose permite a los desarrolladores definir esquemas para sus colecciones de MongoDB, proporcionando una forma estructurada de modelar datos de aplicaciones. Esto ayuda a hacer cumplir la coherencia y validación de los datos.
- **Modelado de datos:** permite a los desarrolladores crear modelos basados ​​en estos esquemas, que representan documentos en colecciones de MongoDB. Los modelos proporcionan una interfaz para crear, consultar, actualizar y eliminar registros.
- **Validación:** Mongoose incluye validación integrada y personalizada para los campos de datos, lo que garantiza que solo se guarden datos válidos en la base de datos.
- **Middleware:** ofrece enlaces previos y posteriores para diversas operaciones, lo que permite a los desarrolladores definir funciones que se ejecutan antes o después de ciertos eventos (por ejemplo, guardar un documento).
- **Creación de consultas:** Mongoose proporciona una potente API para crear consultas complejas, lo que facilita la interacción con los datos de MongoDB.
- **Conversión de tipos:** convierte automáticamente los campos en sus tipos definidos, lo que reduce la necesidad de verificación y conversión de tipos manuales.
- **Gestión de relaciones:** Mongoose ayuda a gestionar las relaciones entre datos, como completar documentos de referencia.
- **Gestión de conexiones:** simplifica el proceso de conexión a bases de datos MongoDB y gestión de grupos de conexiones.
- **Evolución del esquema:** Mongoose facilita la modificación y actualización de esquemas a medida que su aplicación evoluciona con el tiempo.

Mongoose actúa como una capa de abstracción sobre MongoDB, proporcionando una forma más estructurada y rica en funciones de trabajar con MongoDB en aplicaciones Node.js. Simplifica muchas tareas comunes y ayuda a mantener un código más limpio y organizado al interactuar con las bases de datos MongoDB.

Instalamos Mongoose en el proyecto:

```javascript
npm install mongoose
```

Estructura de Carpetas:
project-root/
├── public/ ("archivos estáticos del Front-end")
|	├── css/ 
|	├── html/ 
|	└── js/
└── src/ ("archivos relacionados con el servidor")
|	├── app.js 
|	├── routes/ 
|	├── controllers/
|	├── models/  
|	└── db/  
└── views/ ("vistas creadas con el motor de plantillas")
	├── partials/ 
	└── components/

### Archivo de conexión
`dbconnection.js`
```javascript
const mongoose = require('mongoose');

const database = 'nombre_base_de_datos';
const url = `mongodb://localhost:27017/${database}`;

mongoose.connect(url);
```

`app.js`
```javascript
//agregar la conexion
require('./db/dbconnection')
```


## Definiendo un esquema
Todo en Mongoose comienza con un esquema. Cada esquema se asigna a una colección de MongoDB y define la forma de los documentos dentro de esa colección [2](https://mongoosejs.com/docs/guide.html).

Ejemplo: Esquema de un blog: `blog.model.js`
```javascript

const { Schema } = mongoose;

const blogSchema = new Schema({
  title: String, required: true,
  author: String, trim: true,
  body: String,trim: true,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number, default: 0.0,
    favs: Number
  }
});
```

Si desea agregar claves adicionales más adelante, utilice el método `Schema.add()`:
```javascript
blogSchema.add({ authorDescrpt: 'string', articleCategory: 'string'});
```

Resultado del esquema de un blog después de `add.()`.
`blog.model.js`
```javascript

const { Schema } = mongoose;

const blogSchema = new Schema({
  title: String, required: true,
  author: String, trim: true,
  body: String,trim: true,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number, default: 0.0,
    favs: Number
  }
authorDescrpt: 'string', articleCategory: 'string'
});
```

## Creando los modelos

Para utilizar nuestra definición de esquema, necesitamos convertir nuestro blogSchema en un modelo con el que podamos trabajar. Para hacerlo, lo pasamos a: mongoose.model(modelName, esquema):

```javascript
const Blog = mongoose.model('Blog', blogSchema);

module.exports = Blog;
```

Luego importamos al archivo `app.js`
```javascript
const blogModel = require('./models/blog.model')
require('./db/dbconnection')
```

## Implementando las funciones de un CRUD

### Insertar Documentos (Create):
MongoDB proporciona los siguientes métodos para insertar documentos en una colección [3](https://www.mongodb.com/docs/mongodb-shell/crud/insert/):

 Para insertar un solo documento, use `db.collection.insertOne()`.
 Inserta un solo documento en MongoDB. Si los documentos pasados ​​no contienen el campo `_id`, el driver agregará uno a cada uno de los documentos que le falten, mutando el documento. Este comportamiento se puede anular configurando el indicador `forceServerObjectId`.
 
 ```javascript
let data = [
  {
    title: `${title}`,
    author: `${author}`,
    body: `${body}`,
    comments: [{ body: `${commentBody}`, date: Date }],
    date: { type: Date, default: Date.now },
    hidden: `${hidden}`,
    meta: {
      votes: `${votes}`,
      favs: `${favs}`
    },
    authorDescrpt: `${authorDesc}`,
    articleCategory: `${articleCategory}`
  }
];

const insertOneBlog = async (data) => {
	let blog = new blogModel (data)
	await mongoose.connect();
	try{
	await blog.insertOne()
	}
	catch(error){
	console.log(error)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
    }
}

insertOneBlog(data);
```

 Para insertar varios documentos, utilice `db.collection.insertMany()`.
 Dada una matriz de documentos, `insertMany()` inserta cada documento de la matriz en la colección.
 
 ```javascript
let blog1 = new blogModel(//data acording to the model);
let blog2 = new blogModel(//data acording to the model);

let blogsArray = [blog1, blog2];

const insertManyBlogs = async (array) => {
	await mongoose.connect();
	try{
	await blog.insertMany(array)
	}
	catch(error){
	console.log(error)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
	}
}

insertManyBlogs(blogsArray);
```

## Consultar documentos (Read)

Utilice el método `db.collection.find()` en MongoDB para consultar documentos en una colección.

Consultar todos los documentos:
```javascript
const findAll = async () => {
	let blogs = [];
	await mongoose.connect();
	try {
	blogs = await blog.model.find()
	}
	catch(error){
	console.log(error)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
	}
}

findAll();
```

Consultar un documento:
```javascript

const findByAuthor = async (blogAuthor = "") => {
	let blog = {};
	await mongoose.connect();
	try {
	blog = await blog.model.find({author: $author})
	}
	catch(error){
	console.log(error)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
	}
}

findByAuthor(author);
```
### Update:
El shell MongoDB proporciona los siguientes métodos para actualizar documentos en una colección:

 Para actualizar un solo documento, use `db.collection.updateOne()`.

 Para actualizar varios documentos, utilice `db.collection.updateMany()`.

 Para reemplazar un documento, use `db.collection.replaceOne()`.

Actualizar la sintaxis del operador

Para actualizar un documento, MongoDB proporciona operadores de actualización, como `$set`, para modificar los valores de los campos. Algunos operadores de actualización, como `$set`, crean el campo si el campo no existe.

Para utilizar los operadores de actualización, pase a los métodos de actualización un documento de actualización. 

Actualizar un documento:
```javascript

const update = async (blogAuthor = "", blogTitle = "", newTitle = "") => {
	await mongoose.connect();
	try {
		await blog.model.updateOne(
			{author: `${blogAuthor}`},
			{title: `${blogTitle}`},
			{$set: {title:`${newTitle}`}
		);
		console.log(`Blog entry: `${blogTitle}`, by `${blogAuthor}` is up to date`);
	}
	catch(error){
		console.log(error)
		console.log(`Blog entry: `${blogTitle}`, by `${blogAuthor}` was not found or updated`)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
	}
}

update("author","title", "newTitle" );
```

### Delete:

Para eliminar todos los documentos de una colección, pase un documento de filtro vacío {} al método `db.collection.deleteMany()`.

Para eliminar todos los documentos que coinciden con un criterio de eliminación, pase un parámetro de filtro al método `deleteMany()`.

Para eliminar como máximo un solo documento que coincida con un filtro específico (aunque varios documentos puedan coincidir con el filtro especificado), utilice el método `db.collection.deleteOne()`.

```javascript
const deleteOneBlog = async (blogAuthor="", blogTitle="") => {
	await mongoose.connect();
	try{
	await blog.deleteOne({"title": `${blogTitle}`, "author": `${blogAuthor}`});
	}
	catch(error){
	console.log(error)
	}
	finally {
     // Close the MongoDB client connection
    await mongoose.connection.close();
    }
}

deleteOneBlog("author", "title");
```


---
Referencias:
[1]
[2] https://mongoosejs.com/docs/guide.html
[3] https://www.mongodb.com/docs/mongodb-shell/crud/insert/