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

**Con Atlas CLI (clúster local o en la nube):**

```bash
atlas deployments setup   # asistente interactivo
```

**Conectarse:**

```bash
mongosh "mongodb://localhost:27017"
# o, en Atlas:
mongosh "mongodb+srv://<cluster>.mongodb.net/" --username <usuario>
```

> **Novedad 9.0.4:** se añade compatibilidad del servidor con **Ubuntu 26.04**.

---

## 3. Guía rápida con mongosh

### 3.1. Bases de datos y colecciones

```javascript
show dbs                      // listar bases de datos
use tienda                    // seleccionar (se crea al insertar el primer documento)
show collections              // listar colecciones
db.createCollection("productos")
```

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

**Operadores de consulta más usados:**

| Tipo | Operadores |
|---|---|
| Comparación | `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin` |
| Lógicos | `$and`, `$or`, `$not`, `$nor` |
| Elementos | `$exists`, `$type` |
| Arrays | `$all`, `$elemMatch`, `$size` |
| Evaluación | `$regex`, `$expr`, `$jsonSchema`, `$mod` |

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

**Operadores de actualización:** `$set`, `$unset`, `$inc`, `$mul`, `$rename`, `$min`, `$max`, `$currentDate`, `$push`, `$pull`, `$addToSet`, `$pop`.

### 3.5. Borrar (Delete)

```javascript
db.productos.deleteOne({ nombre: "Silla" })
db.productos.deleteMany({ disponible: false })
```

### 3.6. Índices

```javascript
db.productos.createIndex({ nombre: 1 })                     // simple, ascendente
db.productos.createIndex({ categorias: 1, precio: -1 })     // compuesto
db.usuarios.createIndex({ email: 1 }, { unique: true })     // único
db.sesiones.createIndex({ creado: 1 }, { expireAfterSeconds: 1800 }) // TTL
db.productos.getIndexes()
db.productos.find({ nombre: "Ratón" }).explain("executionStats")
```

> **Regla ESR** para índices compuestos: colocar primero los campos de **igualdad** (*Equality*), después los de **ordenación** (*Sort*) y por último los de **rango** (*Range*).

### 3.7. Agregación

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

**Etapas más habituales:** `$match`, `$project`, `$addFields`/`$set`, `$group`, `$sort`, `$limit`, `$skip`, `$unwind`, `$lookup`, `$facet`, `$bucket`, `$graphLookup`, `$merge`, `$out`.

### 3.8. Transacciones

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

```bash
mongodump --uri="mongodb://localhost:27017" --db=tienda --out=./copia
mongorestore --uri="mongodb://localhost:27017" ./copia
```

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

> El manual incluye además el modelo de **array de ancestros**. El detalle completo del modelado está en el fichero `03-modelado-de-datos-y-diseno.md`.

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
