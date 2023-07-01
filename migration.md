# DB Connect + Migrations + Seeders con Sequelize en Express in Spanish

> **⚠️ Disclaimer:** la presente guía da por entendido que tienes conocimientos intermedios en NODE, EXPRESS y SEQUELIZE. Sin dichos conocimientos, esta guía resultará algo confusa 🤯.

Al usar [sequelize](https://sequelize.org) dentro de un proyecto de NODE + EXPRESS es posible crear las tablas de la base de datos a través de migrations. Y así mismo popular dichas tablas con datos aleaotrios por medio de los seeders.

Este esquema de trabajo es altamente recomendado, pues al trabajar en equipo, nos podemos asegurar que cada integrante del mismo pueda contar con la misma estructura de datos en su DB.

---

## Temario

1. [sequelize-cli](#seq-cli)
2. [Inicialización del proyecto](#init)
3. [Creación del primer modelo y migration](#model-migrate)
4. [Estructura de un modelo](#model)
5. [Estructura de una migration](#migration)
6. [Corriendo nuestra primer migration](#db-migrate)
7. [Deshaciendo las migrations](#db-migrate-undo)
8. [Creando nuestro primer seeder](#seq-seed)
9. [Estructura del archivo seeder](#seeder)
10. [Corriendo los seeders](#db-seed)
11. [Comando útiles](#comands)
12. [Demo](app/)

---

## 1. sequelize-cli <a name="seq-cli"></a>

Lo primero que se debe tener listo es el comando `sequelize` habilitado en nuestra terminal. Para ello instalaremos dicho paquete de **npm** de manera global en nuestra máquina para tenerlo siempre a disposición.

```
npm install -g sequelize-cli sequelize
```

Para corroborar que se instalo efectivamente el paquete, escribiremos en la terminal `sequelize` a lo cual se deberá mostrar algo así:

```
Sequelize CLI [Node: 12.14.1, CLI: 5.5.1, ORM: 5.21.3]

sequelize [command]

Commands:
  sequelize db:migrate                        Run pending migrations
  sequelize db:migrate:schema:timestamps:add  Update migration table to have timestamps
  sequelize db:migrate:status                 List the status of all migrations
  sequelize db:migrate:undo                   Reverts a migration
  sequelize db:migrate:undo:all               Revert all migrations ran
  sequelize db:seed                           Run specified seeder
  sequelize db:seed:undo                      Deletes data from the database
  sequelize db:seed:all                       Run every seeder
  sequelize db:seed:undo:all                  Deletes data from the database
  sequelize db:create                         Create database specified by configuration
  sequelize db:drop                           Drop database specified by configuration
  sequelize init                              Initializes project
  sequelize init:config                       Initializes configuration
  sequelize init:migrations                   Initializes migrations
  sequelize init:models                       Initializes models
  sequelize init:seeders                      Initializes seeders
  sequelize migration:generate                Generates a new migration file                       [aliases: migration:create]
  sequelize model:generate                    Generates a model and its migration                      [aliases: model:create]
  sequelize seed:generate                     Generates a new seed file
```

---

## 2. Inicialización del proyecto <a name="init"></a>

Ahora, inicializaremos las carpetas y archivos base que necesitamos para comenzar a trabajar. Para ello tomaremos el siguiente código:

```
const path = require('path');

module.exports = {
	config: path.resolve('./src/database/config', 'config.js'),
	'models-path': path.resolve('./src/database/models'),
	'seeders-path': path.resolve('./src/database/seeders'),
	'migrations-path': path.resolve('./src/database/migrations'),
};
```

Y guardaremos en mismo en un archivo llamado `.sequelizerc`, el cual deberá estar ubicado en la raíz de nuestro proyecto de NODE + EXPRESS.

```
.
├── node_modules
├── public
├── src
│   ├── app.js
│   └── etc...
├── .sequelizerc ← ¡Archivo Necesario!
├── package.json
└── ...
```

Con dicho archivo listo, podremos ejecutar en la terminal el siguiente comando:

```
npx sequelize init
```

Este comando creará dentro de la carpeta `/src` una sub-carpeta llamada `/database`, la cual tendrá la siguiente estructura:

```
.
├── src
│   ├── app.js
│   └── database
│       └── config
│           └── config.js
│       └── migrations
│       └── models
│           └── index.js
│       └── seeders
├── .sequelizerc
└── ...
```

Entendiendo que las carpetas **migrations** y **seeders** estarán vacías. Dentro del archivo `config.js` la estructura del mismo deberá configurarse así:

```js
module.exports = {
  development: {
    username: DB_USER, // ← Usuario de la DB
    password: DB_PASS, // ← Contraseña del usuario de la DB
    database: DB_NAME, // ← Nombre de la DB previamente creada
    host: "127.0.0.1",
    dialect: "mysql",
  },
  test: {
    username: "root",
    password: null,
    database: "database_test",
    host: "127.0.0.1",
    dialect: "mysql",
  },
  production: {
    username: "root",
    password: null,
    database: "database_production",
    host: "127.0.0.1",
    dialect: "mysql",
  },
};
```

Si hasta aquí todo salió bien, con estos pasos ya tendremos conexión a nuestra base de datos.

---

## 3. Creación del primer modelo (y su respectiva migration) <a name="model-migrate"></a>

Al crear un modelo con la terminal, `sequelize` creará a su vez la correspondiente _migration_ del mismo.

Para crear un modelo basta con ejecutar en la terminal el siguiente comando:

```
npx sequelize model:generate --name User --attributes firstName:string,lastName:string
```

Deshilvanemos el mismo para entenderlo un poco más a fondo:

- `model:generate`: indica a `sequelize` que deberá crear un modelo y su respectiva _migration_.
- `--name User`: creará el modelo `user.js` dentro de la carpeta `/database/models` y la _migration_ para crear la tabla `Users` dentro de la carpeta `/database/migrations`. El nombre del archivo de migración tendrá un _timestamp_ y el texto _create-user_, se verá algo así: `20200420214736-create-user.js`.
- `--attributes`: permite definir las columnas de la tabla y atributos del modelo. No es necesario definir todas las columnas/atributos, pues las mismas se podrán especificar una vez los archivos estén creados.

Con esto hecho, la estructura de archivos dentro de la carpeta `/database` deberá verse algo así:

```
.
├── database
│   └── config
│       └── config.js
│   └── migrations
│       └── 20200420214736-create-user.js
│   └── models
│       └── index.js
│       └── user.js
│   └── seeders
└── ...
```

## 4. Estructura del modelo <a name="model"></a>

Como bien es sabido, un modelo es la representación que el ORM tiene de una tabla en la base de datos. Generalmente un modelo se ve de la siguiente manera:

```js
"use strict";
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define(
    "User",
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
    },
    {}
  );

  User.associate = function (models) {
    // associations can be defined here
  };

  return User;
};
```

En este caso, dentro de éste archivo lo único que se debe definir son las columnas que se desean obtener de la tabla (_la columna id no es necesaria, viene implícita_), pues las mismas quedarán disponibles para lectura y escritura. Cualquier columna existente en la tabla y no referenciada en el modelo será ignorada.

---

## 5. Estructura de la migration <a name="migration"></a>

Tras haber ejecutado el comando:

```
npx sequelize model:generate --name User --attributes firstName:string,lastName:string
```

El archivo de la _migration_ se verá así:

```js
"use strict";
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("Users", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      firstName: {
        type: Sequelize.STRING,
      },
      lastName: {
        type: Sequelize.STRING,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("Users");
  },
};
```

Como se puede apreciar el objeto literal que se está exportando, contiene dós métodos: `up` y `down`. Los cuales permiten:

- `up`: crear la tabla al correr la _migration_.
- `down`: eliminar la tabla si se desea deshacer la _migration_.

Generalment el método `down` **no** se debe tocar. Mientras que en el método `up` es donde vamos a crear todas las columnas que deseamos tenga esa tabla.

En primera medida, `sequelize` nos da las columnas solicitadas `firstName` y `lastName`. Y por otro lado genera de manera implícita las columnas `id`, `createdAt` y `updatedAt`.

Si se desearan agregar las columnas `email` y `deletedAt`, es tan simple como agregar las mismas al listado de atributos.

```js
"use strict";
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("Users", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      firstName: {
        type: Sequelize.STRING,
      },
      lastName: {
        type: Sequelize.STRING,
      },
      // Columna agregada a mano en el archivo
      email: {
        type: Sequelize.STRING,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      // Columna agregada a mano en el archivo
      deletedAt: {
        type: Sequelize.DATE,
        allowNull: true,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("Users");
  },
};
```

> ⚠️ **Warning**: cualquier columna aquí agregada, deberá agregarse también en el modelo para tener acceso de lectura / escritura a la misma. De igual forma, las columnas `createdAt`, `updatedAt` y `deletedAd` no se hace necesario especificarlas en el modelo, pues las mismas ya vienen implícitas.

---

## 6. Corriendo nuestra primer migration <a name="db-migrate"></a>

Con el archivo de migración listo, ahora no nos queda otra cosa más que correr el mismo.

Para ello, desde la terminal ejecutaremos el siguiente comando:

```
npx sequelize db:migrate
```

El anterior comando ejecutará la migración, es decir, creará la tabla correspondiente en la base de datos y adicionalmente, la 1er vez que se ejecute una migración, se creará una tabla llamada `SequelizeMeta` la cual guardará un registro de cada una de las migraciones corridas.

---

## 7. Deshaciendo las migrations <a name="db-migrate-undo"></a>

Si por algún motivo quisieramos revertir el proceso de la migración, llevar a cabo esto es totalmente posible, pues para ello podremos ejecutar el comando:

```
npx sequelize db:migrate:undo
```

> Dicho comando, revertirá la última migración realizada.

Si nuestro objetivo es revertir **todas** las migraciones, el comando a ejecutar en la terminal será:

```
npx sequelize db:migrate:undo:all
```

Pero si quisieramos revertir a una _migration_ en específico, podríamos ejecutar este comando:

```
npx sequelize db:migrate:undo --to XXXXXXXXXXXXXX-create-TABLE.js
```

---

## 8. Creando nuestro primer seeder <a name="seq-seed"></a>

Un archivo _seeder_ básicamente servirá para poder popular las tablas de nuestra base de datos con información _ficticia_.

Para crear nuestro primer _seeder_ tendremos que ejecutar el siguiente comando en la terminal:

```
npx sequelize seed:generate --name demo-user
```

- `seed:generate`: indica a `sequelize` que deberá crear un archivo seeder.
- `--name`: indica el nombre que tendrá el archivo seeder.
- `demo-user`: será el nombre del archivo seeder. Dentro de la carpeta `/seeders/` se creará un archivo con el siguiente nombre `20200420215532-demo-user.js`. (como se puede observar, el nombre del archivo también tiene presente el _timestamp_).

---

## 9. Estructura del archivo seeder <a name="seeder"></a>

Un archivo seeder se verá de la siguiente manera:

```js
"use strict";

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert(
      "Users",
      [
        {
          firstName: "John",
          lastName: "Doe",
          email: "demo@demo.com",
          createdAt: new Date(),
          updatedAt: new Date(),
        },
      ],
      {}
    );
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete("Users", null, {});
  },
};
```

Cómo se puede ver, este archivo también tiene presente dos métodos: `up` y `down`.

- `up`: definé lo que sucederá al momento de ejecutar el archivo seeder.
- `down`: definé lo que sucederá si se quiere revertir el seed realizado.

Dentro del método `up` lo más importante sucede en el segundo parámetro del método `bulkInsert`. Pues el mismo deberá ser un array de objetos literales, los cuales serán los datos que se insertarán en la tabla.

> ⚠️ **Disclaimer**: Cada objeto literal del array deberá tener la misma estructura de atributos del modelo creado previamente.

## 10. Corriendo los seeders <a name="db-seed"></a>

Para poder correr los seeders y popular nuestras tablas con información, deberemos ejecutar el siguiente comando:

```
npx sequelize db:seed:all
```

Si quisieramos revertir la migración más reciente, podríamos ejecutar:

```
npx sequelize db:seed:undo
```

Y si quisieramos revertir todas migraciones realizadas, podríamos ejecutar:

```
npx sequelize db:seed:undo:all
```

---

## 11. Comándos útiles <a name="comands"></a>

- `npx sequelize init`: creará las carpeta y archivos necesarios.
- `npx sequelize model:generate`: creará el modelo y la respectiva migración.
- `npx sequelize db:migrate`: correrá las migraciones pendientes.
- `npx sequelize db:migrate:status`: mostrará las migraciones ejecutadas.
- `npx sequelize db:migrate:undo`: revertirá la última migración ejecutadas.
- `npx sequelize db:migrate:undo:all`: revertirá todas las migraciones ejecutadas.
- `npx sequelize seed:generate`: creará el seeder de datos _fake_.
- `npx sequelize db:seed`: correrá los seeders pendientes.
- `npx sequelize db:seed:all`: correrá todos seeders.
- `npx sequelize db:seed:undo`: revertirá el último seeder que se ejecutó.
- `npx sequelize db:seed:undo:all`: revertirá todos los seeders ejecutados.
- `npx sequelize db:seed:undo:all`: revertirá todos los seeders ejecutados.
- `npx sequelize migration:generate`: generará un archivo _custom_ de migración (Ej: `ALTER TABLE`).


# DB Connect + Migrations + Seeders con Sequelize en Express in English

> **⚠️ Disclaimer:** this guide assumes that you have intermediate knowledge in NODE, EXPRESS and SEQUELIZE. Without this knowledge, this guide will be somewhat confusing 🤯.

When using [sequelize](https://sequelize.org) within a NODE + EXPRESS project, it is possible to create the tables of the database through migrations. And likewise populate these tables with random data through the seeders.

This work scheme is highly recommended, because when working as a team, we can make sure that each member of the team can have the same data structure in their DB.


## Contents

1. [sequelize-cli](#seq-cli)
2. [Project initialization](#init)
3. [Creating the first model and migration](#model-migrate)
4. [Model structure](#model)
5. [Migration structure](#migration)
6. [Running our first migration](#db-migrate)


## 1. sequelize-cli <a name="seq-cli"></a>

The first thing to have ready is the `sequelize` command enabled in our terminal. For this we will install said **npm** package globally on our machine to always have it available.

```
npm install -g sequelize-cli sequelize
```

To verify that the package was installed effectively, we will write `sequelize` in the terminal to which it should show something like this:

```

Sequelize CLI [Node: 12.14.1, CLI: 5.5.1, ORM: 5.21.3]

sequelize [command]

Commands:
  sequelize db:migrate                        Run pending migrations
  sequelize db:migrate:schema:timestamps:add  Update migration table to have timestamps
  sequelize db:migrate:status                 List the status of all migrations
  sequelize db:migrate:undo                   Reverts a migration
  sequelize db:migrate:undo:all               Revert all migrations ran
  sequelize db:seed                           Run specified seeder
  sequelize db:seed:undo                      Deletes data from the database
  sequelize db:seed:all                       Run every seeder
  sequelize db:seed:undo:all                  Deletes data from the database
  sequelize db:create                         Create database specified by configuration
  sequelize db:drop                           Drop database specified by configuration
  sequelize init                              Initializes project
  sequelize init:config                       Initializes configuration
  sequelize init:migrations                   Initializes migrations
  sequelize init:models                       Initializes models
  sequelize init:seeders                      Initializes seeders
  sequelize migration:generate                Generates a new migration file                       [aliases: migration:create]
  sequelize model:generate                    Generates a model and its migration                      [aliases: model:create]
  sequelize seed:generate                     Generates a new seed file

```
---
## 2. Project initialization <a name="init"></a>

Now, we will initialize the base folders and files that we need to start working. For this we will take the following code:

```
const path = require('path');

module.exports = {
  config: path.resolve('./src/database/config', 'config.js'),
  'models-path': path.resolve('./src/database/models'),
  'seeders-path': path.resolve('./src/database/seeders'),
  'migrations-path': path.resolve('./src/database/migrations'),
};
```

And we will save it in a file called `.sequelizerc`, which must be located in the root of our NODE + EXPRESS project.

```
.
├── node_modules
├── public
├── src
│   ├── app.js
│   └── etc...
├── .sequelizerc ← ¡Necessary File!
├── package.json
└── ...
```

With said file ready, we can run the following command in the terminal:

```

npx sequelize init

```

This command will create a sub-folder called `/database` within the `/src` folder, which will have the following structure:

```
.
├── src
│   ├── app.js
│   └── database
│       └── config
│           └── config.js
│       └── migrations
│       └── models
│           └── index.js
│       └── seeders
├── .sequelizerc
└── ...
```

Understanding that the **migrations** and **seeders** folders will be empty. Within the `config.js` file the structure of the same should be configured like this:

```js
module.exports = {
  development: {
    username: DB_USER, // ← DB User
    password: DB_PASS, // ← DB User Password
    database: DB_NAME, // ← DB Name
    host: "
    dialect: "mysql",
  },
};
```

If everything went well so far, with these steps we will already have a connection to our database.

## 3. Creating the first model (and its respective migration) <a name="model-migrate"></a>

When creating a model with the terminal, `sequelize` will also create the corresponding _migration_ of it.

To create a model, just run the following command in the terminal:

```
npx sequelize model:generate --name User --attributes firstName:string,lastName:string
```

Let's unravel it to understand it a little more in depth:

- `model:generate`: indicates to `sequelize` that it should create a model and its respective _migration_.
- `--name User`: will create the `user.js` model within the `/database/models` folder and the _migration_ to create the `Users` table within the `/database/migrations` folder. The name of the migration file will have a _timestamp_ and the text _create-user_, it will look something like this: `20200420214736-create-user.js`.
- `--attributes`: allows you to define the columns of the table and attributes of the model. It is not necessary to define all the columns / attributes, since they can be specified once the files are created.

With this done, the file structure within the `/database` folder should look something like this:

```
.
├── database
│   └── config
│       └── config.js
│   └── migrations
│       └── 20200420214736-create-user.js
│   └── models
│       └── index.js
│       └── user.js
│   └── seeders
└── ...
```

## 4. Model structure <a name="model"></a>

As is well known, a model is the representation that the ORM has of a table in the database. Generally a model looks like this:

```js
"use strict";
  module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define(
    "User",
    {
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
    },
    {}
  );

  User.associate = function (models) {
    // associations can be defined here
  };

  return User;
};
```

In this case, within this file the only thing that must be defined are the columns that are desired to be obtained from the table (_the id column is not necessary, it is implicit_), since they will be available for reading and writing. Any column that exists in the table and is not referenced in the model will be ignored.


## 5. Migration structure <a name="migration"></a>

After having executed the command:

```
npx sequelize model:generate --name User --attributes firstName:string,lastName:string
```

The migration file will look like this:

```js
"use strict";
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("Users", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      firstName: {
        type: Sequelize.STRING,
      },
      lastName: {
        type: Sequelize.STRING,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("Users");
  },
};
```

As you can see, the object literal that is being exported contains two methods: `up` and `down`. Which allow:

- `up`: create the table when running the migration.
- `down`: delete the table if you want to undo the migration.

Generally the `down` method should **not** be touched. While in the `up` method is where we are going to create all the columns that we want that table to have.

In the first instance, `sequelize` gives us the requested columns `firstName` and `lastName`. And on the other hand, it implicitly generates the columns `id`, `createdAt` and `updatedAt`.

If the `email` and `deletedAt` columns were to be added, it is as simple as adding them to the attribute list.

```js
"use strict";
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("Users", {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      firstName: {
        type: Sequelize.STRING,
      },
      lastName: {
        type: Sequelize.STRING,
      },
      // Collumn added by hand in the file
      email: {
        type: Sequelize.STRING,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      // Collumn added by hand in the file
      deletedAt: {
        type: Sequelize.DATE,
        allowNull: true,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("Users");
  },
};
```

> ⚠️ **Warning**: any column added here, must also be added in the model to have read / write access to it. Similarly, the `createdAt`, `updatedAt` and `deletedAd` columns do not need to be specified in the model, since they are already implicit.

## 6. Running our first migration <a name="db-migrate"></a>

With the migration file ready, now we have no other choice but to run it.

For this, from the terminal we will execute the following command:

```
npx sequelize db:migrate
```

The previous command will execute the migration, that is, it will create the corresponding table in the database and additionally, the first time a migration is executed, a table called `SequelizeMeta` will be created which will save a record of each of the migrations run.

## 7. Undoing migrations <a name="db-migrate-undo"></a>

If for some reason we wanted to reverse the migration process, carrying out this is totally possible, because for this we can execute the command:

```
npx sequelize db:migrate:undo
```

> Said command will revert the last migration performed.

If our goal is to revert **all** migrations, the command to execute in the terminal will be:

```
npx sequelize db:migrate:undo:all
```

But if we wanted to revert to a specific _migration_, we could execute this command:

```
npx sequelize db:migrate:undo --to XXXXXXXXXXXXXX-create-TABLE.js
```

## 8. Creating our first seeder <a name="seq-seed"></a>

A _seeder_ file will basically serve to populate the tables of our database with _fake_ information.

To create our first _seeder_ we will have to execute the following command in the terminal:

```
npx sequelize seed:generate --name demo-user
```
- `seed:generate`: indicates to `sequelize` that it should create a seeder file.
- `--name`: indicates the name that the seeder file will have.
- `demo-user`: will be the name of the seeder file. Within the `/seeders/` folder a file will be created with the following name `20200420215532-demo-user.js`. (as you can see, the name of the file also has the _timestamp_ present).

## 9. Seeder file structure <a name="seeder"></a>

A seeder file will look like this:

```js
"use strict";

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert(
      "Users",
      [
        {
          firstName: "John",
          lastName: "Doe",
          email: "demo@demo.com",
          createdAt: new Date(),
          updatedAt: new Date(),
        },
      ],
      {}
    );
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete("Users", null, {});
  },
};
```

As you can see, this file also has two methods present: `up` and `down`.

- `up`: defines what will happen when executing the seeder file.
- `down`: defines what will happen if the performed seed is to be undone.

Within the `up` method, the most important thing happens in the second parameter of the `bulkInsert` method. Since it must be an array of object literals, which will be the data that will be inserted into the table.

> ⚠️ **Disclaimer**: Each object literal in the array must have the same attribute structure of the previously created model.

## 10. Running the seeders <a name="db-seed"></a>

To be able to run the seeders and populate our tables with information, we must execute the following command:

```
npx sequelize db:seed:all
```

If we wanted to revert the most recent migration, we could execute:

```
npx sequelize db:seed:undo
```

And if we wanted to revert all migrations performed, we could execute:

```
npx sequelize db:seed:undo:all
```

## 11. Useful commands <a name="comands"></a>

- `npx sequelize init`: will create the necessary folders and files.
- `npx sequelize model:generate`: will create the model and the respective migration.
- `npx sequelize db:migrate`: will run pending migrations.
- `npx sequelize db:migrate:status`: will show the executed migrations.
- `npx sequelize db:migrate:undo`: will revert the last executed migration.
- `npx sequelize db:migrate:undo:all`: will revert all executed migrations.
- `npx sequelize seed:generate`: will create the _fake_ data seeder.
- `npx sequelize db:seed`: will run pending seeders.
- `npx sequelize db:seed:all`: will run all seeders.
- `npx sequelize db:seed:undo`: will revert the last seeder that was executed.
- `npx sequelize db:seed:undo:all`: will revert all seeders executed.
- `npx sequelize db:seed:undo:all`: will revert all seeders executed.
- `npx sequelize migration:generate`: will generate a custom migration file (Ex: `ALTER TABLE`).

 ---

