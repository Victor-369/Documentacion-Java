# NoSQL y diferencias con SQL

> **Fuente principal:** [¿Qué es NoSQL? — MongoDB](https://www.mongodb.com/es/resources/basics/databases/nosql-explained)
> **Revisado:** octubre de 2026 · **Versión actual de MongoDB:** 9.0 (la página original todavía anuncia la 8.0 en su cabecera; la 9.0 se publicó el 29 de septiembre de 2026).

---

## Índice

1. [¿Qué es NoSQL?](#1-qué-es-nosql)
2. [Tipos de bases de datos NoSQL](#2-tipos-de-bases-de-datos-nosql)
3. [Breve historia](#3-breve-historia)
4. [Características principales](#4-características-principales)
5. [ACID, BASE y el teorema CAP](#5-acid-base-y-el-teorema-cap)
6. [Ejemplo práctico: relacional frente a documental](#6-ejemplo-práctico-relacional-frente-a-documental)
7. [Diferencias entre SQL (RDBMS) y NoSQL](#7-diferencias-entre-sql-rdbms-y-nosql)
8. [Equivalencia de conceptos SQL ↔ MongoDB](#8-equivalencia-de-conceptos-sql--mongodb)
9. [Equivalencia de consultas SQL ↔ MQL](#9-equivalencia-de-consultas-sql--mql)
10. [Casos de uso y cuándo elegir NoSQL](#10-casos-de-uso-y-cuándo-elegir-nosql)
11. [Mitos habituales](#11-mitos-habituales)
12. [Primeros pasos](#12-primeros-pasos)
13. [Preguntas frecuentes](#13-preguntas-frecuentes)
14. [Resumen y recursos](#14-resumen-y-recursos)

---

## 1. ¿Qué es NoSQL?

Se llama **NoSQL** a cualquier base de datos **no relacional**, es decir, que no organiza la información en tablas con filas y columnas fijas unidas por claves foráneas. El término se interpreta de dos formas:

- **"No SQL"**: no usa el modelo relacional ni SQL.
- **"Not only SQL" (no solo SQL)**: la interpretación más extendida hoy, ya que muchas de estas bases de datos admiten lenguajes de consulta potentes (e incluso dialectos parecidos a SQL).

Conviene distinguir dos ideas que a menudo se mezclan:

| Concepto | Qué es |
|---|---|
| **SQL** | Un *lenguaje* de consulta estructurado. |
| **Modelo relacional** | Una *forma de organizar* los datos (tablas normalizadas). |
| **NoSQL** | Un *enfoque* de gestión de datos con modelos alternativos al relacional. |

Rasgos comunes de las bases de datos NoSQL:

- **Esquema flexible**: no es obligatorio definir la estructura antes de insertar datos.
- **Escalado horizontal**: se reparten los datos entre varios servidores.
- **Modelos de datos cercanos al código**: los documentos se corresponden de forma natural con los objetos de los lenguajes de programación.
- **Pensadas para grandes volúmenes** de datos y de usuarios concurrentes.

---

## 2. Tipos de bases de datos NoSQL

Existen cuatro familias principales, más las multimodelo:

### 2.1. Documentales

Guardan la información en **documentos** de tipo JSON (en MongoDB, en su formato binario **BSON**). Cada documento es un conjunto de pares campo-valor, y los valores pueden ser cadenas, números, booleanos, fechas, *arrays* u otros documentos anidados.

- **Ejemplos:** MongoDB, Couchbase, Amazon DocumentDB, Firestore.
- **Ideales para:** uso general, catálogos, perfiles de usuario, gestión de contenidos, datos semiestructurados y jerárquicos.

```json
{
  "_id": "12345",
  "nombre": "Ana García",
  "email": "ana@ejemplo.es",
  "direccion": {
    "calle": "Calle Mayor, 10",
    "ciudad": "Valencia",
    "cp": "46001"
  },
  "aficiones": ["música", "guitarra", "lectura"]
}
```

### 2.2. Clave-valor

La estructura más sencilla: cada **clave única** apunta a un **valor** (que la base de datos trata como un bloque opaco). Suelen trabajar en memoria y ofrecen lecturas y escrituras muy rápidas.

- **Ejemplos:** Redis, Valkey, Amazon DynamoDB, Memcached.
- **Ideales para:** cachés, sesiones de usuario, carritos de la compra, contadores, colas.

```text
Clave:  usuario:12345
Valor:  {"nombre": "Ana García", "email": "ana@ejemplo.es", "puesto": "desarrolladora"}
```

### 2.3. Columnas anchas (*wide-column*)

Organizan los datos en tablas, filas y columnas, pero **cada fila puede tener un conjunto de columnas distinto**. Aplican compresión por columnas y son muy eficientes con datos dispersos.

- **Ejemplos:** Apache Cassandra, Apache HBase, ScyllaDB, Google Bigtable.
- **Ideales para:** series temporales, IoT, registros (*logs*), grandes volúmenes con patrones de consulta predecibles.

| nombre | id | email | fecha_nac | ciudad |
|---|---|---|---|---|
| Ana García | 12345 | ana@ejemplo.es | | Valencia |
| Luis Pérez | 34521 | luis@ejemplo.es | 12-05-1972 | |

### 2.4. Grafos

Representan la información como **nodos** (entidades: personas, lugares, productos) y **aristas** (relaciones entre ellas), ambas con propiedades.

- **Ejemplos:** Neo4j, Amazon Neptune, ArangoDB.
- **Ideales para:** redes sociales, motores de recomendación, detección de fraude, grafos de conocimiento.
- MongoDB ofrece recorridos de grafo mediante la etapa de agregación **`$graphLookup`**.

### 2.5. Multimodelo

Admiten varios modelos (documental, grafo, clave-valor…) sobre un único motor.

- **Ejemplos:** Azure Cosmos DB, ArangoDB, SurrealDB.

### 2.6. Comparativa rápida

| Tipo | Unidad de datos | Punto fuerte | Limitación típica |
|---|---|---|---|
| Documental | Documento JSON/BSON | Flexibilidad y consultas ricas | Requiere pensar el modelo según los accesos |
| Clave-valor | Par clave → valor | Velocidad máxima | Consultas solo por clave |
| Columnas anchas | Fila con columnas variables | Escrituras masivas y escalado | Consultas poco flexibles |
| Grafos | Nodos y aristas | Recorrer relaciones complejas | Menos adecuadas para agregaciones masivas |
| Multimodelo | Varios | Versatilidad | Complejidad y menor especialización |

> **Tendencia actual (2025-2026):** a estas familias se suman las **bases de datos vectoriales** y la **búsqueda vectorial** integrada (por ejemplo, Atlas Vector Search en MongoDB), muy utilizadas en aplicaciones de IA generativa, RAG y memoria de agentes.

---

## 3. Breve historia

- **Años 70-90:** el almacenamiento es caro. El modelo relacional y la **normalización** buscan evitar a toda costa la duplicación de datos.
- **Principios de los 2000:** Google publica el artículo sobre **Bigtable** (2006) y Amazon el de **Dynamo** (2007), que inspiran muchos sistemas distribuidos posteriores.
- **2009:** auge del término NoSQL y aparición de bases de datos documentales como **MongoDB** y **CouchDB**.
- **Década de 2010:** se generalizan los distintos tipos de NoSQL, impulsados por:
  - El abaratamiento del almacenamiento (deja de ser prioritario evitar la duplicación; lo importante pasa a ser la **productividad del desarrollador**).
  - El crecimiento de datos **estructurados, semiestructurados y no estructurados**.
  - Las **metodologías ágiles**, que exigen iterar rápido y cambiar el modelo de datos con frecuencia.
  - La **computación en la nube** y la necesidad de distribuir datos entre servidores y regiones.
- **Década de 2020:** convergencia. Las bases de datos NoSQL incorporan transacciones ACID y validación de esquemas, y las relacionales añaden soporte JSON. Hoy el foco está en el análisis en tiempo real y las cargas de trabajo de IA.

---

## 4. Características principales

| Característica | Descripción |
|---|---|
| **Computación distribuida** | Los datos se reparten entre varios nodos. |
| **Escalabilidad** | Principalmente horizontal (añadir servidores) mediante *sharding* o particionado. |
| **Esquema flexible** | Los documentos de una misma colección pueden tener campos distintos. |
| **Lenguaje de consulta rico** | Filtros, proyecciones, agregaciones, búsqueda de texto, geoespacial, vectorial… |
| **Facilidad para el desarrollador** | El modelo de datos coincide con los objetos del código. |
| **Tolerancia a particiones** | El sistema sigue funcionando aunque falle la comunicación entre nodos. |
| **Alta disponibilidad** | Replicación automática y conmutación por error (*failover*). |

---

## 5. ACID, BASE y el teorema CAP

### ACID (habitual en bases de datos relacionales)

- **A**tomicidad: la operación se ejecuta completa o no se ejecuta.
- **C**onsistencia: los datos pasan de un estado válido a otro estado válido.
- a**I**slamiento: las transacciones concurrentes no interfieren entre sí.
- **D**urabilidad: una vez confirmada, la operación persiste aunque haya un fallo.

### BASE (habitual en muchos sistemas NoSQL distribuidos)

- **B**asically **A**vailable (disponibilidad básica): el sistema tolera fallos parciales, como la caída de un nodo.
- **S**oft state (estado blando): se admiten inconsistencias temporales.
- **E**ventual consistency (consistencia eventual): con el tiempo, todos los nodos convergen al mismo valor.

> **En MongoDB:** las operaciones sobre **un único documento son siempre atómicas**. Además, desde la versión 4.0 (conjuntos de réplicas) y 4.2 (clústeres fragmentados) admite **transacciones ACID multidocumento**. El nivel de consistencia se ajusta con *read concern* y *write concern*. En **MongoDB 9.0**, el servidor limita por defecto a **10 000** las transacciones multidocumento abiertas simultáneamente (parámetro `maxConcurrentMultiDocumentTransactions`).

### Teorema CAP

Un sistema distribuido no puede garantizar a la vez, **en presencia de una partición de red**, las tres propiedades siguientes:

- **C**onsistencia (*Consistency*): todas las lecturas ven la última escritura.
- **D**isponibilidad (*Availability*): toda petición recibe respuesta.
- **T**olerancia a **P**articiones (*Partition tolerance*): el sistema sigue funcionando aunque se corte la comunicación entre nodos.

Como las particiones de red son inevitables en sistemas distribuidos, en la práctica la elección real es entre **priorizar consistencia (CP)** o **priorizar disponibilidad (AP)** cuando se produce una partición. MongoDB, con su configuración por defecto, se comporta como un sistema **CP**.

> **Matiz útil:** el teorema **PACELC** amplía CAP: incluso sin particiones (*Else*), hay que elegir entre **latencia** y **consistencia**.

---

## 6. Ejemplo práctico: relacional frente a documental

Queremos guardar el nombre, apellidos, móvil, ciudad y aficiones de cada usuario.

### Modelo relacional (dos tablas)

**Tabla `usuarios`**

| id | nombre | apellidos | movil | ciudad |
|---|---|---|---|---|
| 1 | Ana | García López | 600111222 | Valencia |

**Tabla `aficiones`**

| id | usuario_id | aficion |
|---|---|---|
| 10 | 1 | música |
| 11 | 1 | guitarra |

```sql
SELECT u.nombre, u.apellidos, a.aficion
FROM usuarios u
JOIN aficiones a ON a.usuario_id = u.id
WHERE u.id = 1;
```

Para recuperar el usuario completo hace falta un **JOIN**, y para actualizar usuario y aficiones de forma conjunta hace falta una **transacción**.

### Modelo documental (un único documento)

```javascript
db.usuarios.insertOne({
  nombre: "Ana",
  apellidos: "García López",
  movil: "600111222",
  ciudad: "Valencia",
  aficiones: ["música", "guitarra"]
})

db.usuarios.findOne({ nombre: "Ana" })
```

Todo se recupera con **una sola lectura**, sin JOIN, y se actualiza de forma **atómica** sin necesidad de transacción multidocumento:

```javascript
db.usuarios.updateOne(
  { nombre: "Ana" },
  { $set: { ciudad: "Madrid" }, $push: { aficiones: "lectura" } }
)
```

---

## 7. Diferencias entre SQL (RDBMS) y NoSQL

| Aspecto | RDBMS (SQL) | NoSQL |
|---|---|---|
| **Modelo de datos** | Tablas de filas y columnas; idóneo para datos estructurados | Documentos, clave-valor, columnas anchas o grafos; idóneo para datos semiestructurados y no estructurados |
| **Esquema** | Fijo y definido de antemano; cambiarlo con datos existentes es costoso (`ALTER TABLE`, migraciones) | Flexible; se puede evolucionar sobre la marcha (y, si se quiere, restringir con validación) |
| **Lenguaje de consulta** | SQL estándar | Depende del producto: MQL (MongoDB), CQL (Cassandra), Cypher (Neo4j)… |
| **Escalabilidad** | Tradicionalmente vertical (más CPU/RAM); la horizontal es posible pero más compleja | Diseñadas para escalado horizontal (y también vertical) |
| **Relaciones** | Claves foráneas y JOIN | Anidadas (embebidas), referencias explícitas o implícitas |
| **Transacciones** | ACID | ACID o BASE según el producto y la configuración |
| **Rendimiento** | Muy bueno en cargas transaccionales con muchos JOIN | Muy bueno en tiempo real, big data y entornos distribuidos |
| **Consistencia** | Fuerte por defecto | Eventual en muchos casos; configurable en otros (MongoDB es fuerte por defecto en el primario) |
| **Distribución** | Clústeres y réplicas, no pensado originalmente para ello | Particionado (*sharding*), replicación y clúster nativos |
| **Tolerancia a fallos** | Replicación, copias de seguridad y recuperación; a menudo requiere medidas adicionales | Replicación y conmutación automática integradas |
| **Particionado** | Por tablas, con poda de particiones | *Sharding* automático entre nodos |
| **Correspondencia con objetos** | Requiere ORM (Hibernate, JPA, Entity Framework…) por el desajuste objeto-relacional | Correspondencia directa; existen ODM opcionales (Mongoose, Spring Data MongoDB…) |

> **Novedad 2026:** MongoDB 9.0 incluye una extensión para **Hibernate ORM**, de modo que los desarrolladores Java pueden seguir usando las API y HQL de Hibernate trabajando sobre MongoDB.

---

## 8. Equivalencia de conceptos SQL ↔ MongoDB

| SQL | MongoDB |
|---|---|
| Base de datos | Base de datos |
| Tabla | Colección |
| Fila (registro) | Documento |
| Columna | Campo |
| Clave primaria | Campo `_id` (indexado automáticamente) |
| Índice | Índice |
| JOIN | Documentos embebidos o etapa `$lookup` |
| Clave foránea | Referencia (no forzada por la base de datos) |
| `GROUP BY` | Etapa `$group` del *pipeline* de agregación |
| Vista | Vista (estándar o materializada con `$merge`/`$out`) |
| `CHECK` / restricciones | Validación de esquema (`$jsonSchema`) |
| Transacción | Transacción multidocumento (o atomicidad de documento) |

---

## 9. Equivalencia de consultas SQL ↔ MQL

| Operación | SQL | MongoDB (mongosh) |
|---|---|---|
| Insertar | `INSERT INTO usuarios (nombre, edad) VALUES ('Ana', 30);` | `db.usuarios.insertOne({ nombre: "Ana", edad: 30 })` |
| Seleccionar todo | `SELECT * FROM usuarios;` | `db.usuarios.find()` |
| Filtrar | `SELECT * FROM usuarios WHERE edad > 25;` | `db.usuarios.find({ edad: { $gt: 25 } })` |
| Proyección | `SELECT nombre FROM usuarios;` | `db.usuarios.find({}, { nombre: 1, _id: 0 })` |
| AND | `WHERE edad > 25 AND ciudad = 'Madrid'` | `find({ edad: { $gt: 25 }, ciudad: "Madrid" })` |
| OR | `WHERE ciudad = 'Madrid' OR ciudad = 'Bilbao'` | `find({ $or: [{ ciudad: "Madrid" }, { ciudad: "Bilbao" }] })` |
| IN | `WHERE ciudad IN ('Madrid','Bilbao')` | `find({ ciudad: { $in: ["Madrid", "Bilbao"] } })` |
| LIKE | `WHERE nombre LIKE 'An%'` | `find({ nombre: /^An/ })` |
| Ordenar y limitar | `ORDER BY edad DESC LIMIT 5` | `find().sort({ edad: -1 }).limit(5)` |
| Contar | `SELECT COUNT(*) FROM usuarios;` | `db.usuarios.countDocuments()` |
| Actualizar | `UPDATE usuarios SET edad = 31 WHERE nombre = 'Ana';` | `db.usuarios.updateOne({ nombre: "Ana" }, { $set: { edad: 31 } })` |
| Borrar | `DELETE FROM usuarios WHERE edad < 18;` | `db.usuarios.deleteMany({ edad: { $lt: 18 } })` |
| Agrupar | `SELECT ciudad, COUNT(*) FROM usuarios GROUP BY ciudad;` | `db.usuarios.aggregate([{ $group: { _id: "$ciudad", total: { $sum: 1 } } }])` |
| JOIN | `... JOIN pedidos p ON p.usuario_id = u.id` | `aggregate([{ $lookup: { from: "pedidos", localField: "_id", foreignField: "usuario_id", as: "pedidos" } }])` |
| Crear índice | `CREATE INDEX idx_email ON usuarios(email);` | `db.usuarios.createIndex({ email: 1 })` |

---

## 10. Casos de uso y cuándo elegir NoSQL

### Casos de uso típicos

- Análisis en tiempo real y paneles de control.
- Gestión de contenidos (CMS) y catálogos de productos con atributos variables.
- Internet de las cosas (IoT) y series temporales.
- Perfiles de usuario, personalización y sistemas de recomendación.
- Detección de fraude.
- Videojuegos (estado de partidas, clasificaciones).
- Aplicaciones móviles y *edge*.
- Aplicaciones de IA: almacenamiento de *embeddings*, RAG, memoria de agentes.

### Señales de que NoSQL (documental) encaja

- Desarrollo ágil con requisitos cambiantes.
- Datos estructurados **y** semiestructurados.
- Grandes volúmenes de datos o de usuarios.
- Necesidad de escalado horizontal.
- Arquitecturas modernas: microservicios, *streaming* en tiempo real, *serverless*.

### Señales de que un RDBMS puede ser mejor opción

- Esquema muy estable y altamente relacional, con consultas *ad hoc* que cruzan muchas entidades.
- Ecosistema existente muy ligado a SQL (herramientas de BI heredadas, informes, equipos formados).
- Requisitos regulatorios o de integridad referencial que se quieren delegar por completo en el motor.

> **Recomendación práctica:** la elección no es excluyente. Muchas arquitecturas actuales son **políglotas** y combinan, por ejemplo, PostgreSQL, MongoDB y Redis según la necesidad de cada servicio.

---

## 11. Mitos habituales

### Mito 1: "Los datos relacionados solo encajan en bases de datos relacionales"

Falso. NoSQL guarda relaciones, pero **de otra manera**: anidando datos relacionados en una misma estructura o mediante referencias. A menudo resulta incluso más sencillo, porque no hay que trocear la información entre tablas.

### Mito 2: "NoSQL no admite transacciones ACID"

Falso para muchos productos. MongoDB admite transacciones ACID multidocumento. Además, un buen modelo documental **reduce la necesidad** de transacciones: si usuario y aficiones están en el mismo documento, basta una única escritura atómica.

### Mito 3: "NoSQL significa sin esquema"

Más preciso: **esquema flexible**. En MongoDB se puede aplicar **validación de esquemas** con `$jsonSchema`, y en la versión 9.0 se añade el nivel de validación **`constraint`**, que garantiza que *todos* los documentos de la colección cumplen las reglas.

### Mito 4: "NoSQL no es consistente"

Depende del producto y de la configuración. MongoDB ofrece consistencia fuerte en lecturas al primario y permite ajustar el equilibrio entre consistencia y latencia.

---

## 12. Primeros pasos

1. **Crear un clúster gratuito en MongoDB Atlas** (servicio gestionado en la nube). El nivel gratuito incluye **512 MB** de almacenamiento y conjuntos de datos de ejemplo.
2. **Explorar los datos** con *Atlas Data Explorer* o con **MongoDB Compass** (interfaz gráfica de escritorio).
3. **Practicar consultas** con **mongosh** (la *shell* de MongoDB).
4. **Construir *pipelines* de agregación** para análisis más avanzados.
5. **Visualizar datos** con **Atlas Charts**.
6. **Formarse** en [MongoDB University](https://learn.mongodb.com/) (cursos gratuitos, insignias de habilidad y certificaciones).
7. **Conectar desde tu lenguaje** con los *drivers* oficiales (Node.js, Python, Java, C#, Go, PHP, Rust…).

Alternativa local: instalar **MongoDB Community Edition** o levantarla con Docker:

```bash
docker run -d --name mongo -p 27017:27017 mongodb/mongodb-community-server:latest
mongosh "mongodb://localhost:27017"
```

---

## 13. Preguntas frecuentes

**¿Cuáles son las ventajas de NoSQL?**
Esquemas flexibles, escalado horizontal, consultas rápidas gracias a un modelo de datos orientado al acceso y facilidad de uso para los desarrolladores.

**¿Qué es la consistencia eventual?**
Una propiedad de las bases de datos distribuidas: tras una actualización, todos los nodos acabarán reflejándola, aunque durante un breve intervalo algunos puedan devolver el valor anterior.

**¿Para qué se usa NoSQL?**
Depende del tipo: las documentales son de propósito general; las clave-valor, para accesos simples y muy rápidos; las de columnas anchas, para grandes volúmenes con consultas predecibles; las de grafos, para analizar relaciones.

**¿Es difícil de aprender?**
No especialmente. Los documentos se corresponden con estructuras de datos de los lenguajes más populares. Quien viene del mundo relacional necesitará adaptarse a **modelar según los patrones de acceso** en lugar de normalizar.

**¿JSON es NoSQL?**
JSON es un formato de datos. Las bases de datos documentales almacenan documentos en JSON o, como MongoDB, en **BSON** (JSON binario con más tipos: fechas, `ObjectId`, `Decimal128`, binarios…).

**¿Qué lenguaje se usa para consultar?**
Cada producto tiene el suyo. MongoDB utiliza **MQL** (MongoDB Query Language) y el *framework* de agregación.

**¿NoSQL tiene esquema?**
Tiene un esquema flexible. En MongoDB se puede "cerrar" tanto como se desee mediante validación de esquemas.

---

## 14. Resumen y recursos

- NoSQL agrupa las bases de datos **no relacionales**: documentales, clave-valor, columnas anchas, grafos y multimodelo.
- Sus puntos fuertes son el **esquema flexible**, el **escalado horizontal**, la **alta disponibilidad** y la **productividad del desarrollador**.
- La diferencia clave con SQL está en el **modelado**: en NoSQL se diseña a partir de **cómo accede la aplicación a los datos**.
- MongoDB es la base de datos NoSQL más popular según DB-Engines; su versión actual es la **9.0**.

### Recursos

- [¿Qué es NoSQL? (MongoDB)](https://www.mongodb.com/es/resources/basics/databases/nosql-explained)
- [NoSQL frente a SQL](https://www.mongodb.com/es/resources/basics/databases/nosql-explained/nosql-vs-sql)
- [Cuándo usar NoSQL](https://www.mongodb.com/es/resources/basics/databases/nosql-explained/when-to-use-nosql)
- [Tipos de bases de datos NoSQL](https://www.mongodb.com/es/resources/basics/databases/types)
- [Transacciones ACID en MongoDB](https://www.mongodb.com/es/resources/basics/databases/acid-transactions)
- [Consultar documentos (manual)](https://www.mongodb.com/es/docs/manual/tutorial/query-documents/)
- [Notas de versión de MongoDB 9.0](https://www.mongodb.com/es/docs/manual/release-notes/9.0/)
- [MongoDB University](https://learn.mongodb.com/)
