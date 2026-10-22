# Tutoriales de MongoDB

> **Fuente principal:** [Tutoriales de MongoDB — Manual de la base de datos](https://www.mongodb.com/es/docs/manual/tutorial/)
> **Revisado:** octubre de 2026 · **Versión del manual:** 9.0 (actual). Última versión de parche publicada: **9.0.4** (13 de octubre de 2026), con correcciones de seguridad.

La página oficial es un **índice de tutoriales** del manual. Este documento la reproduce organizada, añade una guía rápida de uso (CRUD, índices, agregación) y señala qué está obsoleto o ha cambiado en la versión 9.0.

---

## Índice

1. [Conceptos básicos antes de empezar](#1-conceptos-básicos-antes-de-empezar)
2. [Instalación](#2-instalación)
3. [Guía rápida con mongosh](#3-guía-rápida-con-mongosh)
4. [Administración: conjuntos de réplicas](#4-administración-conjuntos-de-réplicas)
5. [Administración: fragmentación (sharding)](#5-administración-fragmentación-sharding)
6. [Operaciones básicas de administración](#6-operaciones-básicas-de-administración)
7. [Seguridad](#7-seguridad)
8. [Patrones de desarrollo](#8-patrones-de-desarrollo)
9. [Patrones de búsqueda de texto ($text)](#9-patrones-de-búsqueda-de-texto-text)
10. [Patrones de modelado de datos](#10-patrones-de-modelado-de-datos)
11. [Novedades de MongoDB 9.0 que afectan a los tutoriales](#11-novedades-de-mongodb-90-que-afectan-a-los-tutoriales)
12. [Ruta de aprendizaje recomendada](#12-ruta-de-aprendizaje-recomendada)
13. [Recursos](#13-recursos)

---

## 1. Conceptos básicos antes de empezar

| Término | Significado |
|---|---|
| **Documento** | Registro en formato BSON (pares campo-valor). Tamaño máximo: **16 MB**. |
| **Colección** | Conjunto de documentos (equivalente aproximado a una tabla). |
| **Base de datos** | Conjunto de colecciones. |
| **`_id`** | Clave primaria de cada documento; si no se indica, se genera un `ObjectId`. |
| **mongod** | Proceso servidor de la base de datos. |
| **mongos** | Enrutador de consultas en un clúster fragmentado. |
| **mongosh** | *Shell* interactiva de MongoDB (sustituye a la antigua `mongo`). |
| **Conjunto de réplicas** (*replica set*) | Grupo de `mongod` que mantienen las mismas copias de los datos: un **primario** y varios **secundarios**. |
| **Fragmentación** (*sharding*) | Reparto de los datos entre varios fragmentos (*shards*) para escalar horizontalmente. |
| **Oplog** | Registro de operaciones que los secundarios replican desde el primario. |

### Ediciones y formas de despliegue

| Opción | Descripción |
|---|---|
| **MongoDB Atlas** | Servicio gestionado en la nube (AWS, Azure, Google Cloud). Incluye nivel gratuito. |
| **Community Edition** | Edición gratuita y autogestionada. |
| **Enterprise Advanced** | Edición comercial autogestionada con LDAP, Kerberos, auditoría, cifrado en reposo, etc. |

> Muchos tutoriales llevan la etiqueta **"autogestionado"** (*self-managed*): se refieren a despliegues que administras tú mismo. En Atlas, la mayoría de estas tareas (réplicas, *sharding*, copias de seguridad, usuarios) se realizan desde la interfaz web o la API.

---

## 2. Instalación

- [Instalar MongoDB Community Edition](https://www.mongodb.com/es/docs/manual/administration/install-community/) (Linux, macOS, Windows y Docker).
- [Instalar MongoDB Enterprise en Linux, macOS y Windows](https://www.mongodb.com/es/docs/manual/administration/install-enterprise/).

### Formas rápidas de tener MongoDB funcionando

**Con Docker:**

```bash
docker run -d --name mongo -p 27017:27017 \
  -v mongo_datos:/data/db \
  mongodb/mongodb-community-server:latest
```

**Qué hace cada parte:**

| Fragmento | Explicación |
|---|---|
| `docker run` | Descarga la imagen (si no la tienes) y crea y arranca un contenedor a partir de ella. |
| `-d` | *Detached*: el contenedor se ejecuta en segundo plano y te devuelve el control del terminal. |
| `--name mongo` | Le da un nombre al contenedor para poder referirte a él después (`docker stop mongo`, `docker logs mongo`…). |
| `-p 27017:27017` | Publica el puerto: `puerto_de_tu_máquina:puerto_del_contenedor`. **27017** es el puerto por defecto de MongoDB. |
| `-v mongo_datos:/data/db` | Crea un **volumen** llamado `mongo_datos` y lo monta en `/data/db`, la carpeta donde MongoDB guarda los datos. Así, aunque borres el contenedor, **los datos no se pierden**. |
| `mongodb/mongodb-community-server:latest` | Imagen oficial de MongoDB Community. `latest` = la versión más reciente (en proyectos reales es mejor fijar una versión, p. ej. `:9.0`). |
| `\` | Solo indica que el comando continúa en la línea siguiente. |

> Para comprobar que está funcionando: `docker ps` (debe aparecer el contenedor `mongo` con estado *Up*).

**Con Atlas CLI (clúster local o en la nube):**

```bash
atlas deployments setup   # asistente interactivo
```

`atlas deployments setup` lanza un **asistente** que te va haciendo preguntas (si quieres un despliegue local o en la nube, nombre, versión…) y al terminar te muestra la cadena de conexión. Requiere tener instalada la [Atlas CLI](https://www.mongodb.com/docs/atlas/cli/current/).

**Conectarse:**

```bash
mongosh "mongodb://localhost:27017"
# o, en Atlas:
mongosh "mongodb+srv://<cluster>.mongodb.net/" --username <usuario>
```

- `mongosh` abre la *shell* de MongoDB y se conecta al servidor indicado por la **cadena de conexión** (*connection string*).
- `mongodb://localhost:27017` → protocolo `mongodb://`, servidor `localhost` (tu propia máquina) y puerto `27017`.
- `mongodb+srv://...` → formato usado por Atlas: en lugar de listar todos los servidores del clúster, se consulta un registro DNS que los devuelve automáticamente.
- `<cluster>` y `<usuario>` son **marcadores**: hay que sustituirlos (sin los `<>`) por tus valores. Al usar `--username`, mongosh te pedirá la contraseña.
- Si la conexión va bien verás un *prompt* como `test>` (el nombre de la base de datos activa). Para salir: `exit`.

> **Novedad 9.0.4:** se añade compatibilidad del servidor con **Ubuntu 26.04**.

---

## 3. Guía rápida con mongosh

### 3.0. Cómo leer los comandos de este documento

> **¿Dónde se escribe cada comando?**
> - Los bloques marcados como `bash` se ejecutan en el **terminal del sistema** (Linux, macOS, PowerShell…).
> - Los bloques marcados como `javascript` se ejecutan **dentro de mongosh** (después de conectarte). mongosh entiende JavaScript, por eso puedes usar variables, `try/catch`, `new Date()`, etc.

Casi todos los comandos de mongosh siguen la misma estructura:

```javascript
db.<colección>.<método>(<argumentos>)
//  ejemplo:
db.productos.find({ precio: { $lt: 100 } })
```

| Pieza | Significado |
|---|---|
| `db` | La base de datos activa (la que elegiste con `use`). |
| `productos` | La colección sobre la que trabajas (similar a una tabla). |
| `find(...)` | La operación que quieres hacer. |
| `{ campo: valor }` | Los argumentos son **documentos** con sintaxis parecida a JSON. Las claves no necesitan comillas. |
| `$lt`, `$set`, `$match`… | Las palabras que empiezan por **`$` en una clave** son **operadores** (instrucciones para MongoDB, no nombres de campo). |
| `"$precio"` | En agregaciones, un **valor de texto que empieza por `$`** significa "el valor del campo `precio`". |
| `1` / `-1` | Según el contexto: orden ascendente / descendente, o incluir / excluir un campo. |

> **Analogía con SQL** (útil si ya conoces bases de datos relacionales): base de datos ≈ base de datos, colección ≈ tabla, documento ≈ fila, campo ≈ columna. La gran diferencia es que los documentos de una misma colección **no tienen que tener los mismos campos**.

### 3.1. Bases de datos y colecciones

```javascript
show dbs                      // listar bases de datos
use tienda                    // seleccionar (se crea al insertar el primer documento)
show collections              // listar colecciones
db.createCollection("productos")
```

| Comando | Qué hace |
|---|---|
| `show dbs` | Lista las bases de datos del servidor y su tamaño. Las bases de datos **vacías no aparecen**. |
| `use tienda` | Cambia la base de datos activa a `tienda`. Si no existe **no da error**: se creará de verdad cuando insertes el primer documento. A partir de aquí, `db` apunta a `tienda`. |
| `show collections` | Lista las colecciones de la base de datos activa. |
| `db.createCollection("productos")` | Crea la colección explícitamente. **Normalmente no hace falta**: MongoDB crea la colección automáticamente al insertar en ella. Se usa cuando quieres pasar opciones (reglas de validación, colecciones de tamaño fijo, series temporales…). |

### 3.2. Crear (Create)

```javascript
db.productos.insertOne({
  nombre: "Portátil",
  precio: 899.99,
  categorias: ["informática", "oficina"],
  stock: 12,
  creado: new Date()
})

db.productos.insertMany([
  { nombre: "Ratón", precio: 19.90, categorias: ["informática"], stock: 150 },
  { nombre: "Silla", precio: 129.00, categorias: ["oficina"], stock: 0 }
])
```

**Qué hace cada comando:**

- **`insertOne({...})`** inserta **un** documento en la colección `productos`.
  - `categorias: [...]` es un **array**: un campo puede guardar una lista de valores sin necesidad de otra tabla.
  - `new Date()` guarda la fecha y hora actuales como tipo fecha (no como texto).
  - No hemos indicado `_id`, así que MongoDB genera uno automáticamente (un `ObjectId`).
- **`insertMany([...])`** inserta **varios** documentos a la vez. Recibe un **array** (`[ ]`) de documentos separados por comas.
  - Fíjate en que estos documentos **no tienen el campo `creado`**: en MongoDB es válido, cada documento puede tener campos distintos.

**Qué devuelve** (respuesta de mongosh):

```javascript
// insertOne
{ acknowledged: true, insertedId: ObjectId('6718f0c2a1b2c3d4e5f60718') }
// insertMany
{ acknowledged: true, insertedIds: { '0': ObjectId('...'), '1': ObjectId('...') } }
```

`acknowledged: true` significa que el servidor ha confirmado la escritura; `insertedId(s)` son los `_id` generados.

### 3.3. Leer (Read)

```javascript
db.productos.find()                                        // todos
db.productos.find({ precio: { $lt: 100 } })               // precio < 100
db.productos.find({ categorias: "oficina" })              // contiene en el array
db.productos.find({ stock: { $gt: 0 }, precio: { $lte: 500 } })
db.productos.find({}, { nombre: 1, precio: 1, _id: 0 })   // proyección
db.productos.find().sort({ precio: -1 }).limit(3)          // ordenar y limitar
db.productos.countDocuments({ stock: 0 })
```

`find(filtro, proyección)` recibe dos argumentos opcionales: el **filtro** (qué documentos quieres) y la **proyección** (qué campos quieres ver de cada uno).

| Comando | Qué hace | Equivalente SQL aproximado |
|---|---|---|
| `find()` | Devuelve todos los documentos. mongosh muestra los 20 primeros; escribe `it` para ver los siguientes. | `SELECT * FROM productos` |
| `find({ precio: { $lt: 100 } })` | Productos cuyo precio es **menor que** (*less than*) 100. | `WHERE precio < 100` |
| `find({ categorias: "oficina" })` | Como `categorias` es un array, encuentra los documentos cuyo array **contiene** `"oficina"` entre sus elementos. | — (en SQL haría falta una tabla intermedia) |
| `find({ stock: { $gt: 0 }, precio: { $lte: 500 } })` | Varias condiciones en el mismo objeto se combinan con **Y** (*AND*): stock mayor que 0 **y** precio menor o igual que 500. | `WHERE stock > 0 AND precio <= 500` |
| `find({}, { nombre: 1, precio: 1, _id: 0 })` | `{}` = sin filtro (todos). La proyección indica qué campos mostrar: `1` = incluir, `0` = excluir. `_id` se muestra siempre salvo que pongas `_id: 0`. | `SELECT nombre, precio FROM productos` |
| `find().sort({ precio: -1 }).limit(3)` | Los métodos se **encadenan**: ordena por precio descendente (`-1`; `1` sería ascendente) y se queda con los 3 primeros → los 3 productos más caros. | `ORDER BY precio DESC LIMIT 3` |
| `countDocuments({ stock: 0 })` | Cuenta cuántos documentos cumplen el filtro (productos sin stock). | `SELECT COUNT(*) WHERE stock = 0` |

> **Error típico:** en la proyección no se pueden mezclar `1` y `0` (salvo en `_id`). O eliges qué campos incluir, o qué campos excluir.

**Operadores de consulta más usados:**

| Tipo | Operadores | Significado |
|---|---|---|
| Comparación | `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin` | igual, distinto, mayor, mayor o igual, menor, menor o igual, está en la lista, no está en la lista |
| Lógicos | `$and`, `$or`, `$not`, `$nor` | y, o, negación, ni uno ni otro |
| Elementos | `$exists`, `$type` | el campo existe / el campo es de un tipo concreto |
| Arrays | `$all`, `$elemMatch`, `$size` | contiene todos los valores / algún elemento cumple varias condiciones a la vez / el array tiene N elementos |
| Evaluación | `$regex`, `$expr`, `$jsonSchema`, `$mod` | expresión regular / comparar campos entre sí / validar contra un esquema / resto de una división |

**Ejemplos de uso:**

```javascript
// Productos de la categoría "oficina" O con precio menor que 20
db.productos.find({ $or: [ { categorias: "oficina" }, { precio: { $lt: 20 } } ] })

// Productos cuyo nombre es "Ratón" o "Teclado"   (SQL: WHERE nombre IN (...))
db.productos.find({ nombre: { $in: ["Ratón", "Teclado"] } })

// Productos que tienen el campo "creado"
db.productos.find({ creado: { $exists: true } })

// Productos cuyo nombre empieza por "S" (la "i" ignora mayúsculas/minúsculas)
db.productos.find({ nombre: { $regex: "^s", $options: "i" } })

// Productos con más unidades en stock que su precio (compara dos campos)
db.productos.find({ $expr: { $gt: ["$stock", "$precio"] } })
```

### 3.4. Actualizar (Update)

```javascript
db.productos.updateOne(
  { nombre: "Ratón" },
  { $set: { precio: 17.50 }, $inc: { stock: -1 } }
)

db.productos.updateMany(
  { stock: 0 },
  { $set: { disponible: false } }
)

db.productos.updateOne(
  { nombre: "Portátil" },
  { $push: { categorias: "gaming" } }
)

// upsert: actualiza o inserta si no existe
db.productos.updateOne(
  { nombre: "Teclado" },
  { $set: { precio: 49.90, stock: 30 } },
  { upsert: true }
)
```

Los métodos de actualización reciben hasta tres argumentos: `updateOne(filtro, cambios, opciones)`.

1. **Filtro**: qué documento(s) modificar (misma sintaxis que en `find`).
2. **Cambios**: qué modificar, siempre usando **operadores de actualización** (`$set`, `$inc`…).
3. **Opciones** (opcional): por ejemplo, `upsert`.

**Qué hace cada ejemplo:**

| Ejemplo | Explicación |
|---|---|
| 1.º `updateOne` | Busca el **primer** producto llamado "Ratón". `$set` cambia el precio a 17.50 y `$inc: { stock: -1 }` **resta 1** al stock (incrementar en -1). Se pueden combinar varios operadores en la misma actualización. |
| `updateMany` | Modifica **todos** los productos con `stock: 0`. `$set` añade el campo `disponible: false`; si el campo no existía, lo crea. (SQL: `UPDATE productos SET disponible = false WHERE stock = 0`). |
| 2.º `updateOne` | `$push` **añade un elemento al final del array** `categorias`. Si `"gaming"` ya estuviera, se duplicaría; para evitarlo se usa `$addToSet`. |
| `upsert: true` | *Upsert* = *update* + *insert*. Si existe un producto "Teclado", lo actualiza; si **no existe, lo crea** con `nombre: "Teclado"` más los campos de `$set`. |

**Qué devuelve:** `{ acknowledged: true, matchedCount: 1, modifiedCount: 1, upsertedCount: 0, insertedId: null }`. `matchedCount` = documentos que cumplían el filtro; `modifiedCount` = documentos que realmente cambiaron (si el valor ya era el mismo, no cuenta).

> **Error típico:** escribir `updateOne({ nombre: "Ratón" }, { precio: 17.50 })` sin `$set`. mongosh lo rechaza con *"Update document requires atomic operators"*. Si de verdad quieres sustituir el documento completo, se usa `replaceOne`.

**Operadores de actualización:**

| Operador | Qué hace | Ejemplo |
|---|---|---|
| `$set` | Asigna un valor (crea el campo si no existe). | `{ $set: { precio: 10 } }` |
| `$unset` | Elimina un campo. | `{ $unset: { disponible: "" } }` |
| `$inc` | Suma (o resta, con negativo) una cantidad. | `{ $inc: { stock: 5 } }` |
| `$mul` | Multiplica por un número. | `{ $mul: { precio: 1.21 } }` (añadir IVA) |
| `$rename` | Cambia el nombre de un campo. | `{ $rename: { stock: "existencias" } }` |
| `$min` / `$max` | Actualiza solo si el nuevo valor es menor / mayor que el actual. | `{ $min: { precio: 15 } }` |
| `$currentDate` | Pone la fecha actual. | `{ $currentDate: { modificado: true } }` |
| `$push` | Añade un elemento a un array. | `{ $push: { categorias: "nuevo" } }` |
| `$addToSet` | Añade al array solo si no está ya. | `{ $addToSet: { categorias: "oficina" } }` |
| `$pull` | Quita del array los elementos que coinciden. | `{ $pull: { categorias: "gaming" } }` |
| `$pop` | Quita el último (`1`) o el primer (`-1`) elemento del array. | `{ $pop: { categorias: 1 } }` |

### 3.5. Borrar (Delete)

```javascript
db.productos.deleteOne({ nombre: "Silla" })
db.productos.deleteMany({ disponible: false })
```

- **`deleteOne(filtro)`** borra **el primer** documento que cumple el filtro (aunque haya más).
- **`deleteMany(filtro)`** borra **todos** los documentos que cumplen el filtro.
- Devuelven `{ acknowledged: true, deletedCount: N }` con el número de documentos borrados.

> ⚠️ **Cuidado:** `db.productos.deleteMany({})` (filtro vacío) **borra todos los documentos** de la colección, igual que un `DELETE` sin `WHERE` en SQL. Buena práctica: ejecuta antes un `find()` con el mismo filtro para comprobar qué vas a borrar.

### 3.6. Índices

```javascript
db.productos.createIndex({ nombre: 1 })                     // simple, ascendente
db.productos.createIndex({ categorias: 1, precio: -1 })     // compuesto
db.usuarios.createIndex({ email: 1 }, { unique: true })     // único
db.sesiones.createIndex({ creado: 1 }, { expireAfterSeconds: 1800 }) // TTL
db.productos.getIndexes()
db.productos.find({ nombre: "Ratón" }).explain("executionStats")
```

**¿Qué es un índice?** Igual que el índice de un libro: en lugar de leer todas las páginas para encontrar un tema, vas directamente a la página correcta. Sin índice, MongoDB tiene que revisar **todos** los documentos de la colección para responder una consulta. A cambio, cada índice ocupa espacio y hace las escrituras un poco más lentas, así que solo se crean para las consultas frecuentes.

| Comando | Qué hace |
|---|---|
| `createIndex({ nombre: 1 })` | Índice sobre el campo `nombre`. El `1` indica orden ascendente (`-1`, descendente). En índices de un solo campo el orden da igual: MongoDB puede recorrerlo en ambos sentidos. |
| `createIndex({ categorias: 1, precio: -1 })` | Índice **compuesto** (varios campos). Sirve para consultas que filtran por `categorias`, o por `categorias` y `precio`, pero **no** para las que filtran solo por `precio`: el orden de los campos importa. Como `categorias` es un array, además es un índice **multiclave** (indexa cada elemento del array). |
| `createIndex({ email: 1 }, { unique: true })` | Índice **único**: impide que dos usuarios tengan el mismo email. Si lo intentas, obtendrás un error `E11000 duplicate key error`. |
| `createIndex({ creado: 1 }, { expireAfterSeconds: 1800 })` | Índice **TTL** (*time to live*): MongoDB borra automáticamente cada documento 1800 s (30 min) después de la fecha guardada en `creado`. El campo debe ser de tipo fecha. El borrado lo hace un proceso que se ejecuta cada 60 s, así que no es exacto al segundo. Ideal para sesiones, tokens o *logs* temporales. |
| `getIndexes()` | Lista los índices de la colección. Siempre verás `_id_`, que MongoDB crea automáticamente. |
| `find(...).explain("executionStats")` | No devuelve los datos, sino **cómo** ha ejecutado MongoDB la consulta. |

**Cómo interpretar `explain()`** — mira estos campos de la respuesta:

- `winningPlan.stage` (o `inputStage.stage`): **`IXSCAN`** = ha usado un índice ✅; **`COLLSCAN`** = ha recorrido toda la colección ❌ (en colecciones grandes, señal de que falta un índice).
- `nReturned`: documentos devueltos.
- `totalDocsExamined`: documentos que ha tenido que leer. Lo ideal es que se parezca mucho a `nReturned`.
- `executionTimeMillis`: tiempo que ha tardado.

> **Regla ESR** para índices compuestos: colocar primero los campos de **igualdad** (*Equality*), después los de **ordenación** (*Sort*) y por último los de **rango** (*Range*).
>
> Ejemplo: para la consulta `db.productos.find({ categorias: "oficina", precio: { $gte: 50 } }).sort({ nombre: 1 })`
> - Igualdad → `categorias` (`= "oficina"`)
> - Ordenación → `nombre`
> - Rango → `precio` (`>= 50`)
>
> Índice recomendado: `db.productos.createIndex({ categorias: 1, nombre: 1, precio: 1 })`.

### 3.7. Agregación

La **agregación** sirve para hacer cálculos y transformaciones sobre los datos (totales, medias, agrupaciones, *joins*…), parecido a `GROUP BY`, `SUM` o `JOIN` en SQL. Funciona como una **cadena de montaje** (*pipeline*): es un array de **etapas** y cada etapa recibe los documentos que produce la anterior, los transforma y los pasa a la siguiente.

Para entender el ejemplo, supón que cada documento de la colección `pedidos` tiene esta forma:

```javascript
{
  _id: 1001,
  cliente: "Ana",
  estado: "entregado",
  lineas: [
    { producto: "Ratón",    cantidad: 2, precio: 19.90 },
    { producto: "Portátil", cantidad: 1, precio: 899.99 }
  ]
}
```

Objetivo: **los 10 productos que más ingresos han generado en pedidos entregados.**

```javascript
db.pedidos.aggregate([
  { $match: { estado: "entregado" } },
  { $unwind: "$lineas" },
  { $group: {
      _id: "$lineas.producto",
      unidades: { $sum: "$lineas.cantidad" },
      ingresos: { $sum: { $multiply: ["$lineas.cantidad", "$lineas.precio"] } }
  } },
  { $sort: { ingresos: -1 } },
  { $limit: 10 }
])
```

**Etapa a etapa:**

1. **`$match: { estado: "entregado" }`** — Filtra: solo pasan los pedidos entregados (como un `WHERE`). Conviene ponerlo **al principio** para procesar menos documentos y poder usar índices.
2. **`$unwind: "$lineas"`** — "Desenrolla" el array `lineas`: crea **un documento por cada elemento** del array. El pedido 1001 de arriba se convierte en dos documentos:
   ```javascript
   { _id: 1001, cliente: "Ana", estado: "entregado", lineas: { producto: "Ratón",    cantidad: 2, precio: 19.90 } }
   { _id: 1001, cliente: "Ana", estado: "entregado", lineas: { producto: "Portátil", cantidad: 1, precio: 899.99 } }
   ```
   Fíjate en el `$` de `"$lineas"`: significa "el valor del campo `lineas`".
3. **`$group`** — Agrupa los documentos (como `GROUP BY`):
   - `_id: "$lineas.producto"` es la **clave de agrupación**: un grupo por cada producto distinto. El punto (`lineas.producto`) accede a un campo dentro de un subdocumento.
   - `unidades: { $sum: "$lineas.cantidad" }` suma las cantidades de cada grupo.
   - `ingresos: { $sum: { $multiply: [...] } }` multiplica cantidad × precio de cada línea y suma los resultados.
4. **`$sort: { ingresos: -1 }`** — Ordena de mayor a menor ingreso.
5. **`$limit: 10`** — Se queda con los 10 primeros.

**Resultado** (un documento por producto):

```javascript
{ _id: "Portátil", unidades: 37, ingresos: 33299.63 }
{ _id: "Ratón",    unidades: 412, ingresos: 8198.8 }
...
```

**Equivalente SQL aproximado** (con una tabla `lineas_pedido`):

```sql
SELECT producto, SUM(cantidad) AS unidades, SUM(cantidad * precio) AS ingresos
FROM pedidos p JOIN lineas_pedido l ON l.pedido_id = p.id
WHERE p.estado = 'entregado'
GROUP BY producto
ORDER BY ingresos DESC
LIMIT 10;
```

**Etapas más habituales:**

| Etapa | Qué hace | Parecido en SQL |
|---|---|---|
| `$match` | Filtra documentos. | `WHERE` / `HAVING` |
| `$project` | Elige, renombra o calcula los campos de salida. | lista del `SELECT` |
| `$addFields` / `$set` | Añade campos calculados manteniendo los demás. | columna calculada |
| `$group` | Agrupa y calcula totales (`$sum`, `$avg`, `$min`, `$max`, `$count`…). | `GROUP BY` |
| `$sort` | Ordena. | `ORDER BY` |
| `$limit` / `$skip` | Limita / salta resultados (útil para paginar). | `LIMIT` / `OFFSET` |
| `$unwind` | Crea un documento por cada elemento de un array. | — |
| `$lookup` | Une con documentos de otra colección. | `LEFT JOIN` |
| `$facet` | Ejecuta varios *pipelines* en paralelo sobre los mismos datos (p. ej., resultados + totales para una página de búsqueda). | — |
| `$bucket` | Agrupa en rangos (p. ej., precios de 0–50, 50–100…). | `CASE` + `GROUP BY` |
| `$graphLookup` | Búsqueda recursiva (jerarquías, árboles, redes). | `WITH RECURSIVE` |
| `$merge` / `$out` | Guarda el resultado en una colección. | `INSERT INTO ... SELECT` |

### 3.8. Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **todas o ninguna**. El ejemplo clásico es una transferencia bancaria: si se resta el dinero de una cuenta pero falla la suma en la otra, el dinero "desaparecería". Con una transacción, si algo falla se deshacen todos los cambios.

```javascript
const sesion = db.getMongo().startSession()
sesion.startTransaction()
try {
  const cuentas = sesion.getDatabase("banco").cuentas
  cuentas.updateOne({ _id: 1 }, { $inc: { saldo: -100 } })
  cuentas.updateOne({ _id: 2 }, { $inc: { saldo: 100 } })
  sesion.commitTransaction()
} catch (e) {
  sesion.abortTransaction()
  throw e
} finally {
  sesion.endSession()
}
```

**Línea a línea:**

| Código | Explicación |
|---|---|
| `db.getMongo().startSession()` | Abre una **sesión**: un "contexto" del que forman parte las operaciones de la transacción. |
| `sesion.startTransaction()` | Empieza la transacción. A partir de aquí, los cambios hechos a través de la sesión no son visibles para nadie más hasta confirmarlos. |
| `sesion.getDatabase("banco").cuentas` | Obtiene la colección `cuentas` **a través de la sesión**. Es importante: si usaras `db.cuentas` directamente, las operaciones quedarían **fuera** de la transacción. |
| `updateOne(... $inc: { saldo: -100 })` | Resta 100 € a la cuenta 1. |
| `updateOne(... $inc: { saldo: 100 })` | Suma 100 € a la cuenta 2. |
| `sesion.commitTransaction()` | **Confirma**: los dos cambios se guardan a la vez y pasan a ser visibles. |
| `catch` → `sesion.abortTransaction()` | Si algo falla, **deshace** todos los cambios de la transacción. `throw e` vuelve a lanzar el error para que no pase desapercibido. |
| `finally` → `sesion.endSession()` | Cierra la sesión pase lo que pase (éxito o error), para liberar recursos. |

> **¿Siempre necesito transacciones?** No. En MongoDB, una operación sobre **un solo documento** ya es atómica (se aplica entera o no se aplica). Las transacciones solo son necesarias cuando hay que modificar **varios documentos** de forma coordinada. Un buen modelado de datos (agrupar en un documento lo que cambia a la vez) reduce mucho su necesidad.

> Las transacciones requieren un **conjunto de réplicas** o un **clúster fragmentado** (no funcionan en un `mongod` independiente).

---

## 4. Administración: conjuntos de réplicas

Un conjunto de réplicas proporciona **alta disponibilidad** y **redundancia**. Se recomiendan al menos **tres miembros** con derecho a voto.

| Tutorial | Para qué sirve |
|---|---|
| [Implementar un conjunto de réplicas](https://www.mongodb.com/es/docs/manual/tutorial/deploy-replica-set/) | Crear un *replica set* desde cero. |
| [Convertir un mongod independiente en conjunto de réplicas](https://www.mongodb.com/es/docs/manual/tutorial/convert-standalone-to-replica-set/) | Pasar de un nodo único a un *replica set*. |
| [Añadir miembros](https://www.mongodb.com/es/docs/manual/tutorial/expand-replica-set/) | Ampliar el conjunto. |
| [Eliminar miembros](https://www.mongodb.com/es/docs/manual/tutorial/remove-replica-set-member/) | Retirar nodos. |
| [Sustituir un miembro](https://www.mongodb.com/es/docs/manual/tutorial/replace-replica-set-member/) | Reemplazar un nodo. |
| [Ajustar la prioridad](https://www.mongodb.com/es/docs/manual/tutorial/adjust-replica-set-member-priority/) | Influir en qué nodo se elige como primario. |
| [Resincronizar un miembro](https://www.mongodb.com/es/docs/manual/tutorial/resync-replica-set-member/) | Recuperar un nodo desfasado. |
| [Conjunto con redundancia geográfica](https://www.mongodb.com/es/docs/manual/tutorial/deploy-geographically-distributed-replica-set/) | Repartir nodos entre centros de datos. |
| [Cambiar el tamaño del oplog](https://www.mongodb.com/es/docs/manual/tutorial/change-oplog-size/) | Ampliar la ventana de replicación. |
| [Forzar que un miembro sea primario](https://www.mongodb.com/es/docs/manual/tutorial/force-member-to-be-primary/) | Mantenimiento controlado. |
| [Cambiar nombres de host](https://www.mongodb.com/es/docs/manual/tutorial/change-hostnames-in-a-replica-set/) | Migraciones de infraestructura. |
| [Añadir un árbitro](https://www.mongodb.com/es/docs/manual/tutorial/add-replica-set-arbiter/) | Nodo que vota pero no guarda datos. |
| [Convertir un secundario en árbitro](https://www.mongodb.com/es/docs/manual/tutorial/convert-secondary-into-arbiter/) | Reducir almacenamiento. |
| [Configurar el origen de sincronización](https://www.mongodb.com/es/docs/manual/tutorial/configure-replica-set-secondary-sync-target/) | Elegir de qué nodo copia un secundario. |
| [Miembro retrasado](https://www.mongodb.com/es/docs/manual/tutorial/configure-a-delayed-replica-set-member/) | Copia con retraso para recuperarse de errores humanos. |
| [Miembro oculto](https://www.mongodb.com/es/docs/manual/tutorial/configure-a-hidden-replica-set-member/) | Nodo invisible para las aplicaciones (analítica, copias). |
| [Miembro sin voto](https://www.mongodb.com/es/docs/manual/tutorial/configure-a-non-voting-replica-set-member/) | Más de 7 miembros o réplicas de lectura. |
| [Evitar que un secundario sea primario](https://www.mongodb.com/es/docs/manual/tutorial/configure-secondary-only-replica-set-member/) | Prioridad 0. |
| [Conjuntos de etiquetas](https://www.mongodb.com/es/docs/manual/tutorial/configure-replica-set-tag-sets/) | Dirigir lecturas/escrituras a centros de datos concretos. |
| [Replicación en cadena](https://www.mongodb.com/es/docs/manual/tutorial/manage-chained-replication/) | Secundarios que replican de otros secundarios. |
| [Reconfigurar con miembros no disponibles](https://www.mongodb.com/es/docs/manual/tutorial/reconfigure-replica-set-with-unavailable-members/) | Recuperación ante desastres. |
| [Recuperar tras un apagado inesperado](https://www.mongodb.com/es/docs/manual/tutorial/recover-data-following-unexpected-shutdown/) | Reparación de un nodo independiente. |
| [Solucionar problemas](https://www.mongodb.com/es/docs/manual/tutorial/troubleshoot-replica-sets/) | Diagnóstico. |
| [Renombrar un conjunto de réplicas](https://www.mongodb.com/es/docs/manual/tutorial/rename-unsharded-replica-set/) | Cambio de nombre. |

**Ejemplo mínimo de inicialización:**

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
rs.status()
```

**Qué hace:**

- **`rs.initiate({...})`** convierte varios `mongod` en un conjunto de réplicas. Se ejecuta **una sola vez**, conectado con mongosh a **uno** de los nodos.
  - `_id: "rs0"` es el **nombre del conjunto**. Debe coincidir con el valor de la opción `--replSet` con la que se arrancó cada `mongod` (ver más abajo).
  - `members` es la lista de nodos: cada uno con un `_id` numérico único y su `host` (`nombre_o_IP:puerto`). Todos los nodos deben poder resolver y alcanzar esos nombres entre sí.
  - Tras unos segundos, los nodos **eligen un primario** por votación; los demás pasan a ser secundarios.
- **`rs.status()`** muestra el estado del conjunto. En el campo `members`, el `stateStr` de cada nodo indica su papel: `PRIMARY`, `SECONDARY`, `STARTUP2` (sincronizando)…

Antes de `rs.initiate`, cada nodo debe arrancarse indicando a qué conjunto pertenece, por ejemplo:

```bash
mongod --replSet rs0 --port 27017 --dbpath /data/db --bind_ip localhost,mongo1
```

- `--replSet rs0`: este nodo forma parte del conjunto `rs0`.
- `--dbpath`: carpeta donde guarda los datos.
- `--bind_ip`: direcciones en las que acepta conexiones (por defecto solo `localhost`).

Otros métodos útiles: `rs.conf()` (ver la configuración), `rs.add("mongo4:27017")` (añadir un nodo), `rs.stepDown()` (pedir al primario que ceda su papel).

---

## 5. Administración: fragmentación (sharding)

El *sharding* distribuye las colecciones entre varios fragmentos según una **clave de fragmentación** (*shard key*). La elección de esa clave es la decisión de diseño más importante: debe tener alta cardinalidad, buena distribución y alinearse con las consultas más frecuentes.

| Tutorial | Para qué sirve |
|---|---|
| [Implementar un clúster fragmentado](https://www.mongodb.com/es/docs/manual/tutorial/deploy-shard-cluster/) | Montar *shards*, servidores de configuración y `mongos`. |
| [Convertir un conjunto de réplicas en clúster fragmentado](https://www.mongodb.com/es/docs/manual/tutorial/convert-replica-set-to-replicated-shard-cluster/) | Escalar un despliegue existente. |
| [Añadir fragmentos](https://www.mongodb.com/es/docs/manual/tutorial/add-shards-to-shard-cluster/) | Ampliar capacidad. |
| [Eliminar fragmentos](https://www.mongodb.com/es/docs/manual/tutorial/remove-shards-from-cluster/) | Reducir capacidad. |
| [De servidor de configuración integrado a dedicado](https://www.mongodb.com/es/docs/manual/tutorial/embedded-to-dedicated/) | Separar el servidor de configuración. |
| [Migrar rangos](https://www.mongodb.com/es/docs/manual/tutorial/migrate-chunks-in-sharded-cluster/) | Mover datos manualmente entre *shards*. |
| [Sustituir un servidor de configuración](https://www.mongodb.com/es/docs/manual/tutorial/replace-config-server/) | Mantenimiento. |
| [Migrar a hardware nuevo](https://www.mongodb.com/es/docs/manual/tutorial/migrate-sharded-cluster-to-new-hardware/) | Renovación de infraestructura. |
| [Copia de seguridad de metadatos](https://www.mongodb.com/es/docs/manual/tutorial/backup-sharded-cluster-metadata/) | Proteger la configuración del clúster. |
| [Copia con instantáneas del sistema de ficheros](https://www.mongodb.com/es/docs/manual/tutorial/backup-sharded-cluster-with-filesystem-snapshots/) | *Backup* físico. |
| [Copia con volcado de base de datos](https://www.mongodb.com/es/docs/manual/tutorial/backup-sharded-cluster-with-database-dumps/) | *Backup* lógico (`mongodump`). |
| [Restaurar desde instantáneas](https://www.mongodb.com/es/docs/manual/tutorial/restore-sharded-cluster/) | Recuperación. |
| [Programar una ventana de copia de seguridad](https://www.mongodb.com/es/docs/manual/tutorial/schedule-backup-window-for-sharded-clusters/) | Evitar migraciones durante el *backup*. |
| [Administrar zonas](https://www.mongodb.com/es/docs/manual/tutorial/manage-shard-zone/) | Ubicar datos por región (residencia de datos, RGPD). |

```javascript
sh.enableSharding("tienda")
sh.shardCollection("tienda.pedidos", { clienteId: "hashed" })
sh.status()
```

Estos comandos se ejecutan conectado con mongosh al **`mongos`** (el enrutador del clúster), no a un *shard* directamente.

| Comando | Qué hace |
|---|---|
| `sh.enableSharding("tienda")` | Marca la base de datos `tienda` como apta para fragmentarse. Desde MongoDB 6.0 este paso ya no es obligatorio (`shardCollection` lo hace implícitamente), pero sigue siendo válido. |
| `sh.shardCollection("tienda.pedidos", { clienteId: "hashed" })` | Fragmenta la colección `pedidos` (se indica como `"basededatos.colección"`) usando `clienteId` como **clave de fragmentación**. `"hashed"` significa que MongoDB calcula un *hash* del valor y reparte según ese *hash*: los datos quedan **distribuidos uniformemente** entre los *shards*, aunque los `clienteId` sean consecutivos. La contrapartida: las consultas por rangos (`clienteId` entre 100 y 200) tendrán que preguntar a todos los *shards*. Con `{ clienteId: 1 }` (fragmentación por **rangos**) ocurre lo contrario. |
| `sh.status()` | Muestra los *shards* del clúster, las bases de datos y colecciones fragmentadas y cómo se reparten los datos entre ellos. |

> La colección necesita un índice que empiece por la clave de fragmentación; si la colección está vacía, MongoDB lo crea automáticamente. Cambiar la clave más adelante es posible (`reshardCollection`) pero costoso, por eso conviene elegirla bien desde el principio.

> **Nuevos métodos de mongosh en 9.0:** `sh.shardDrainingStatus()` (estado del vaciado de un fragmento que se va a retirar) y `sh.getTransitionToDedicatedConfigServerStatus()`.

---

## 6. Operaciones básicas de administración

| Tutorial | Resumen |
|---|---|
| [Usar comandos de base de datos](https://www.mongodb.com/es/docs/manual/tutorial/use-database-commands/) | `db.runCommand()` y `db.adminCommand()`. |
| [Recuperar tras un apagado inesperado](https://www.mongodb.com/es/docs/manual/tutorial/recover-data-following-unexpected-shutdown/) | Reparar un nodo independiente. |
| [Expirar datos con TTL](https://www.mongodb.com/es/docs/manual/tutorial/expire-data/) | Borrado automático de documentos antiguos (sesiones, logs). |
| [Database Profiler](https://www.mongodb.com/es/docs/manual/tutorial/manage-the-database-profiler/) | Registrar operaciones lentas para analizarlas. |
| [Rotar ficheros de registro](https://www.mongodb.com/es/docs/manual/tutorial/rotate-log-files/) | Gestión de *logs*. |
| [Configurar procesos mongod](https://www.mongodb.com/es/docs/manual/tutorial/manage-mongodb-processes/) | Arranque, parada y opciones. |
| [Copias de seguridad con las herramientas de MongoDB](https://www.mongodb.com/es/docs/manual/tutorial/backup-and-restore-tools/) | `mongodump` / `mongorestore`. |
| [Copias con instantáneas del sistema de ficheros](https://www.mongodb.com/es/docs/manual/tutorial/backup-with-filesystem-snapshots/) | LVM, EBS, etc. |

```javascript
// Perfilador: registrar operaciones de más de 100 ms
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().sort({ ts: -1 }).limit(5)
```

El **perfilador** (*profiler*) registra las operaciones de la base de datos activa para que puedas encontrar las consultas lentas.

- **`db.setProfilingLevel(1, { slowms: 100 })`**
  - Nivel `0` = desactivado (por defecto); nivel `1` = registra solo las operaciones **lentas**; nivel `2` = registra **todas** (muy costoso, solo para depurar puntualmente).
  - `slowms: 100` define qué es "lento": más de 100 milisegundos.
- **`db.system.profile.find().sort({ ts: -1 }).limit(5)`**
  - Las operaciones registradas se guardan en la colección especial `system.profile`, y se consultan con un `find` normal.
  - `sort({ ts: -1 })` ordena por fecha (`ts` = *timestamp*) de más reciente a más antigua, y `limit(5)` muestra las 5 últimas.
  - En cada resultado, los campos más útiles son `op` (tipo de operación), `ns` (colección), `millis` (duración) y `command` (la consulta completa). Después, usa `explain()` sobre esa consulta para ver si le falta un índice.
- Al terminar, desactívalo con `db.setProfilingLevel(0)`.

**Copias de seguridad** (estos comandos se ejecutan en el **terminal del sistema**, no dentro de mongosh):

```bash
mongodump --uri="mongodb://localhost:27017" --db=tienda --out=./copia
mongorestore --uri="mongodb://localhost:27017" ./copia
```

| Comando / opción | Explicación |
|---|---|
| `mongodump` | Exporta los datos a ficheros (copia **lógica**). |
| `--uri=...` | Servidor al que conectarse (misma cadena de conexión que usa mongosh). |
| `--db=tienda` | Solo copia la base de datos `tienda` (sin esta opción, copia todas). |
| `--out=./copia` | Carpeta de destino. Se crea `./copia/tienda/` con un fichero `.bson` (datos) y un `.metadata.json` (índices y opciones) por cada colección. |
| `mongorestore ... ./copia` | Importa al servidor los datos de la carpeta indicada. Por defecto **no sobrescribe** documentos con el mismo `_id`; añade `--drop` si quieres borrar cada colección antes de restaurarla. |

> `mongodump` y `mongorestore` forman parte de las **MongoDB Database Tools**, que se instalan por separado del servidor. En la imagen Docker oficial ya vienen incluidas: `docker exec -it mongo mongodump ...`.

---

## 7. Seguridad

| Tutorial | Resumen |
|---|---|
| [Cortafuegos iptables (Linux)](https://www.mongodb.com/es/docs/manual/tutorial/configure-linux-iptables-firewall/) | Limitar el acceso de red. |
| [Cortafuegos netsh (Windows)](https://www.mongodb.com/es/docs/manual/tutorial/configure-windows-netsh-firewall/) | Igual, en Windows. |
| [Habilitar el control de acceso](https://www.mongodb.com/es/docs/manual/tutorial/enable-authentication/) | Activar autenticación (imprescindible en producción). |
| [Gestionar usuarios y roles](https://www.mongodb.com/es/docs/manual/tutorial/manage-users-and-roles/) | Control de acceso basado en roles (RBAC). |
| [Autenticación Kerberos en Linux](https://www.mongodb.com/es/docs/manual/tutorial/control-access-to-mongodb-with-kerberos-authentication/) | Solo Enterprise. |
| [Informar de una vulnerabilidad](https://www.mongodb.com/es/docs/manual/tutorial/create-a-vulnerability-report/) | Procedimiento de divulgación. |

```javascript
use admin
db.createUser({
  user: "admin",
  pwd: passwordPrompt(),
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase"]
})
```

**Qué hace:** crea un usuario administrador.

| Código | Explicación |
|---|---|
| `use admin` | Los usuarios administradores se crean en la base de datos especial `admin`. La base de datos donde se crea un usuario es su **base de datos de autenticación**: habrá que indicarla al conectarse. |
| `user: "admin"` | Nombre del usuario. |
| `pwd: passwordPrompt()` | En lugar de escribir la contraseña en el comando (quedaría en el historial de la *shell*), mongosh te la **pide de forma interactiva** sin mostrarla. |
| `roles: [...]` | Permisos del usuario. Un rol puede indicarse como objeto `{ role, db }` o como texto (entonces se refiere a la base de datos activa, aquí `admin`). |
| `userAdminAnyDatabase` | Permite crear y gestionar usuarios y roles en cualquier base de datos. |
| `readWriteAnyDatabase` | Permite leer y escribir datos en cualquier base de datos. |

Crear el usuario **no activa** la autenticación por sí solo. Hay que:

1. Activarla en el fichero de configuración `mongod.conf` y reiniciar `mongod`:
   ```yaml
   security:
     authorization: enabled
   ```
2. Conectarse indicando usuario y base de datos de autenticación (en el terminal):
   ```bash
   mongosh "mongodb://localhost:27017" -u admin -p --authenticationDatabase admin
   ```

Siguiendo el principio de mínimo privilegio, tu aplicación **no** debería usar este usuario, sino uno con permisos solo sobre su base de datos:

```javascript
use tienda
db.createUser({ user: "app_tienda", pwd: passwordPrompt(), roles: [{ role: "readWrite", db: "tienda" }] })
```

**Lista mínima de seguridad:**

- Activar autenticación y aplicar el **principio de mínimo privilegio**.
- No exponer el puerto 27017 a Internet; usar `bindIp`, cortafuegos o redes privadas.
- Cifrar las comunicaciones con **TLS**.
- Mantener el servidor actualizado (la **9.0.4** corrige varias CVE).
- Considerar cifrado en reposo y **Queryable Encryption** para datos sensibles.

> **Novedades de seguridad en 9.0:** las consultas por **prefijo, sufijo y subcadena** sobre campos cifrados con Queryable Encryption pasan a disponibilidad general; `mongocryptd` queda **obsoleto** en favor de la *Automatic Encryption Shared Library*.

---

## 8. Patrones de desarrollo

| Tutorial | Comentario |
|---|---|
| [Tutoriales completos del *pipeline* de agregación](https://www.mongodb.com/es/docs/manual/tutorial/aggregation-complete-examples/) | Ejemplos de filtrado, agrupación, *joins*, desanidado… Muy recomendable. |
| [Modelar datos para búsqueda por palabras clave](https://www.mongodb.com/es/docs/manual/tutorial/model-data-for-keyword-search/) | Array de palabras clave + índice multiclave. Para búsqueda avanzada, mejor **Atlas Search**. |
| [Map-Reduce incremental](https://www.mongodb.com/es/docs/manual/tutorial/perform-incremental-map-reduce/) | ⚠️ **Map-Reduce está obsoleto desde MongoDB 5.0**: usar el *pipeline* de agregación (`$group`, `$merge`). |
| [Solucionar problemas de la función Map](https://www.mongodb.com/es/docs/manual/tutorial/troubleshoot-map-function/) | ⚠️ Ídem (heredado). |
| [Solucionar problemas de la función Reduce](https://www.mongodb.com/es/docs/manual/tutorial/troubleshoot-reduce-function/) | ⚠️ Ídem (heredado). |
| [Almacenar una función JavaScript en el servidor](https://www.mongodb.com/es/docs/manual/tutorial/store-javascript-function-on-server/) | Desaconsejado en general; mejor mantener la lógica en la aplicación. |
| [Ajuste del *pool* de conexiones](https://www.mongodb.com/es/docs/manual/tutorial/connection-pool-performance-tuning/) | `maxPoolSize`, `minPoolSize`, `maxIdleTimeMS`… Clave en producción. |

> **Cambio relevante en 9.0:** el JavaScript del lado del servidor (`$where`, `$function`, `$accumulator`), declarado obsoleto en la 8.0, **deja de estar obsoleto**: ahora se ejecuta en un motor basado en **WebAssembly** con mejor aislamiento. No está disponible en arquitectura `ppc64le`.

---

## 9. Patrones de búsqueda de texto ($text)

- [Especificar el idioma de los índices de texto](https://www.mongodb.com/es/docs/manual/core/indexes/index-types/index-text/specify-text-index-language/) — importante para el español (`default_language: "spanish"`), que aplica *stemming* y palabras vacías.
- [Limitar las entradas de índice de texto examinadas](https://www.mongodb.com/es/docs/manual/core/indexes/index-types/index-text/limit-number-of-items-scanned-for-text-search/).

```javascript
db.articulos.createIndex(
  { titulo: "text", cuerpo: "text" },
  { default_language: "spanish", weights: { titulo: 3, cuerpo: 1 } }
)
db.articulos.find(
  { $text: { $search: "bases de datos documentales" } },
  { puntuacion: { $meta: "textScore" } }
).sort({ puntuacion: { $meta: "textScore" } })
```

**Creación del índice de texto:**

- `{ titulo: "text", cuerpo: "text" }` — en lugar de `1`/`-1`, el valor `"text"` crea un índice de **texto** sobre esos campos, que permite buscar palabras dentro de ellos. Solo puede haber **un índice de texto por colección** (pero puede incluir varios campos).
- `default_language: "spanish"` — aplica las reglas del español:
  - *Stemming* (raíz de la palabra): buscar "base" también encuentra "bases".
  - Palabras vacías (*stop words*): se ignoran palabras muy comunes como "de", "la", "el".
- `weights: { titulo: 3, cuerpo: 1 }` — una coincidencia en el título vale **3 veces más** que en el cuerpo al calcular la relevancia.

**Búsqueda:**

- `{ $text: { $search: "bases de datos documentales" } }` — busca documentos que contengan **alguna** de las palabras (funciona como un *O*). Variantes útiles:
  - Frase exacta, entre comillas escapadas: `$search: "\"bases de datos\""`.
  - Excluir una palabra con un guion: `$search: "mongodb -sql"`.
- `{ puntuacion: { $meta: "textScore" } }` — en la proyección, añade a cada resultado un campo `puntuacion` con la **relevancia** calculada por MongoDB.
- `.sort({ puntuacion: { $meta: "textScore" } })` — ordena del resultado más relevante al menos relevante.

> En Atlas (y, en versión preliminar, también en despliegues autogestionados mediante el proceso `mongot`), **MongoDB Search** ofrece búsqueda de texto completo basada en Lucene, con autocompletado, tolerancia a errores tipográficos y facetas, y **Vector Search** para búsqueda semántica.

---

## 10. Patrones de modelado de datos

| Tutorial | Idea clave |
|---|---|
| [Uno a uno con documentos embebidos](https://www.mongodb.com/es/docs/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/) | Incrustar el subdocumento (p. ej., usuario → dirección). |
| [Uno a muchos con documentos embebidos](https://www.mongodb.com/es/docs/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/) | Array de subdocumentos cuando el "muchos" es acotado. |
| [Muchos a muchos con documentos embebidos](https://www.mongodb.com/es/docs/manual/tutorial/model-embedded-many-to-many-relationships-between-documents/) | Incrustar en el lado más consultado; duplicar con criterio. |
| [Uno a muchos con referencias](https://www.mongodb.com/es/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/) | Guardar la referencia en el lado "muchos" si el array crecería sin límite. |
| [Modelo para operaciones atómicas](https://www.mongodb.com/es/docs/manual/tutorial/model-data-for-atomic-operations/) | Agrupar en un documento lo que debe actualizarse a la vez. |
| [Árbol con referencias al padre](https://www.mongodb.com/es/docs/manual/tutorial/model-tree-structures-with-parent-references/) | Cada nodo guarda `parent`. |
| [Árbol con referencias a los hijos](https://www.mongodb.com/es/docs/manual/tutorial/model-tree-structures-with-child-references/) | Cada nodo guarda `children: []`. |
| [Árbol con rutas materializadas](https://www.mongodb.com/es/docs/manual/tutorial/model-tree-structures-with-materialized-paths/) | Cada nodo guarda la ruta completa como cadena (`",Libros,Informática,"`). |
| [Árbol con conjuntos anidados](https://www.mongodb.com/es/docs/manual/tutorial/model-tree-structures-with-nested-sets/) | Valores `left`/`right`; lecturas rápidas, escrituras costosas. |

> El manual incluye además el modelo de **array de ancestros**. El detalle completo del modelado está en el fichero [MongoDB: modelado de datos y diseno](<MongoDB: modelado de datos y diseno.md>).

---

## 11. Novedades de MongoDB 9.0 que afectan a los tutoriales

| Novedad | Impacto práctico |
|---|---|
| **Límite de memoria por operación** | Por defecto, 1 GB o el 20 % de la memoria del proceso (lo que sea mayor). Las consultas que lo superen fallan con `ExceededMemoryLimit`. Configurable con `internalQueryMaxMemoryUsageBytesPerOperation`. El límite de 100 MB por etapa no cambia. |
| **Límite de transacciones concurrentes** | 10 000 por defecto (`maxConcurrentMultiDocumentTransactions`); al superarlo se devuelve `TooManyOpenTransactions`. |
| **Nivel de validación `constraint`** | Garantiza que *todos* los documentos de la colección cumplen las reglas de validación. |
| **Validación BSON por defecto** | El servidor valida todo el BSON recibido; `net.wireObjectCheck` queda obsoleto. |
| **Series temporales** | Se almacenan como un único espacio de nombres con *buckets* comprimidos (ya no son vistas materializadas). Se pueden renombrar con `renameCollection`. |
| **`querySettings` por comando** | Se pueden pasar ajustes de consulta directamente a `find`, `distinct` y `aggregate`, además de `maxTimeMS` y *query knobs* por forma de consulta. |
| **`$queryStats` activado por defecto** | Muestreo del 1 % de lecturas y escrituras; nueva estructura de métricas. |
| **JavaScript en servidor rehabilitado** | `$where`, `$function` y `$accumulator` vuelven a estar soportados (motor WASM). |

> **Problema conocido con Mongoose:** en FCV 9.0, `listIndexes` informa `collation: { locale: "simple" }`, lo que hace que `syncIndexes()` de versiones antiguas de Mongoose elimine y reconstruya índices en cada arranque. Solución: actualizar a **Mongoose 9.10.4 u 8.24.5**, o usar `createIndexes()` en lugar de `syncIndexes()` al iniciar la aplicación.

---

## 12. Ruta de aprendizaje recomendada

1. **Fundamentos:** documentos, colecciones, BSON y tipos de datos.
2. **Entorno:** Atlas gratuito o Docker + mongosh + Compass.
3. **CRUD:** inserción, consultas con operadores, actualizaciones y borrados.
4. **Índices:** simples, compuestos, multiclave, únicos, TTL; uso de `explain()`.
5. **Modelado de datos:** embeber frente a referenciar, patrones y antipatrones.
6. **Agregación:** `$match`, `$group`, `$lookup`, `$unwind`, `$facet`.
7. **Transacciones y consistencia:** *read/write concern*.
8. **Replicación y *sharding*:** conceptos y despliegue.
9. **Seguridad y operación:** usuarios, roles, TLS, copias de seguridad, monitorización.
10. **Búsqueda e IA:** MongoDB Search y Vector Search.

---

## 13. Recursos

- [Índice de tutoriales](https://www.mongodb.com/es/docs/manual/tutorial/)
- [Guías de inicio por lenguaje](https://www.mongodb.com/es/docs/manual/tutorial/getting-started/)
- [Operaciones CRUD](https://www.mongodb.com/es/docs/manual/crud/)
- [Índices](https://www.mongodb.com/es/docs/manual/indexes/)
- [Agregación](https://www.mongodb.com/es/docs/manual/aggregation/)
- [Notas de versión de MongoDB 9.0](https://www.mongodb.com/es/docs/manual/release-notes/9.0/)
- [Documentación de mongosh](https://www.mongodb.com/es/docs/mongodb-shell/)
- [MongoDB University](https://learn.mongodb.com/)
- Índice de la documentación para herramientas de IA: `https://www.mongodb.com/es/docs/llms.txt` (añadiendo `.md` a cualquier URL del manual se obtiene su versión Markdown).
