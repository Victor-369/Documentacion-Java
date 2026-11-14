# Consideraciones generales de diseño: modelado de datos en MongoDB

> **Nota sobre la fuente:** la URL original (`/docs/manual/applications/data-models/`) ya no existe. Su contenido se ha reorganizado en la sección **[Modelado de datos](https://www.mongodb.com/es/docs/manual/data-modeling/)** del manual y sus subpáginas (buenas prácticas, proceso de diseño, patrones y antipatrones).
> **Revisado:** octubre de 2026 · **Versión del manual:** 9.0 (actual).

---

## Índice

1. [Qué es el modelado de datos en MongoDB](#1-qué-es-el-modelado-de-datos-en-mongodb)
2. [Principio fundamental](#2-principio-fundamental)
3. [Diferencias de diseño frente a una base de datos relacional](#3-diferencias-de-diseño-frente-a-una-base-de-datos-relacional)
4. [Proceso de diseño del esquema (4 pasos)](#4-proceso-de-diseño-del-esquema-4-pasos)
5. [Relaciones entre documentos](#5-relaciones-entre-documentos)
6. [Embeber o referenciar: cómo decidir](#6-embeber-o-referenciar-cómo-decidir)
7. [Duplicación de datos y coherencia](#7-duplicación-de-datos-y-coherencia)
8. [Validación de esquemas](#8-validación-de-esquemas)
9. [Índices y modelado](#9-índices-y-modelado)
10. [Otras consideraciones](#10-otras-consideraciones)
11. [Patrones de diseño de esquemas](#11-patrones-de-diseño-de-esquemas)
12. [Antipatrones de diseño de esquemas](#12-antipatrones-de-diseño-de-esquemas)
13. [Modelado de estructuras de árbol](#13-modelado-de-estructuras-de-árbol)
14. [Ejemplo completo: tienda en línea](#14-ejemplo-completo-tienda-en-línea)
15. [Lista de comprobación del diseño](#15-lista-de-comprobación-del-diseño)
16. [Recursos](#16-recursos)

---

## 1. Qué es el modelado de datos en MongoDB

El **modelado de datos** consiste en decidir cómo se organiza la información dentro de la base de datos y cómo se vinculan las entidades relacionadas. Dicho de forma sencilla: decidir **qué colecciones vas a tener y qué forma tiene cada documento**.

MongoDB tiene un **modelo de datos flexible** que permite almacenar **datos polimórficos** (del griego "muchas formas"):

- Los documentos de una misma colección **no tienen por qué compartir los mismos campos**.
- El **tipo de dato** de un campo puede variar entre documentos.

```javascript
// Dos documentos válidos en la MISMA colección "productos"
{ nombre: "Camiseta", talla: "M", color: "azul" }
{ nombre: "Libro", isbn: "978-84-0000-000-0", paginas: 320 }
```

Esta flexibilidad permite:

- Hacer corresponder el modelo de datos **directamente con los objetos del código**: un objeto Java con una lista dentro se guarda como un documento con un array dentro, sin repartirlo en varias tablas.
- **Iterar** el modelo a medida que evoluciona la aplicación: añadir un campo nuevo no requiere un `ALTER TABLE`.
- Añadir **validación** solo en las partes del documento que requieran un control más estricto.

> ⚠️ "Flexible" **no** significa "sin diseño". Que MongoDB acepte cualquier cosa hace que la responsabilidad de mantener los datos ordenados recaiga en ti. Este documento explica cómo hacerlo bien.

### 1.1. Vocabulario básico

Estos términos aparecen a lo largo de todo el documento:

| Término | Significado | Ejemplo |
|---|---|---|
| **Embeber** (*embed*) | Guardar los datos relacionados **dentro** del mismo documento, como subdocumento o array. | Las direcciones de un usuario dentro del documento del usuario. |
| **Referenciar** (*reference*) | Guardar en un documento el `_id` de **otro** documento de otra colección (parecido a una clave foránea). | Un libro guarda `editorial_id: "anaya"`. |
| **Subdocumento** | Un objeto dentro de otro documento. | `historial: { grupoSanguineo: "A+" }` |
| **Patrón de acceso** | Cómo usa la aplicación los datos: qué consulta, con qué frecuencia y qué campos necesita. | "La ficha de producto se abre 10 000 veces por minuto". |
| **Cardinalidad** | Cuántos elementos hay en el lado "muchos" de una relación. | Un usuario tiene 2 direcciones (baja) o 1 millón de seguidores (muy alta). |
| **Normalizar** | Evitar datos repetidos repartiéndolos en varias tablas/colecciones. | El nombre del cliente solo está en `clientes`. |
| **Desnormalizar** | Repetir datos a propósito para leer más rápido. | Copiar el nombre del cliente dentro de cada pedido. |
| **Atomicidad** | Una operación se aplica **entera o no se aplica**; nunca a medias. | Si falla a mitad, no queda nada guardado. |
| **Conjunto de trabajo** (*working set*) | Datos e índices que la aplicación usa a menudo y que conviene que quepan en RAM. | Los productos activos y sus índices. |

### Límites técnicos que condicionan el diseño

| Límite | Valor | Qué significa en la práctica |
|---|---|---|
| Tamaño máximo de un documento BSON | **16 MB** (para ficheros mayores: **GridFS**) | Un documento no puede crecer indefinidamente. 16 MB es mucho texto (miles de páginas), pero un array que crece sin control acabará llegando. **GridFS** es el mecanismo de MongoDB para guardar ficheros grandes (vídeos, PDF) partiéndolos en trozos. |
| Niveles máximos de anidamiento | **100** | Subdocumentos dentro de subdocumentos, hasta 100 niveles. En la práctica, si pasas de 3–4 niveles, el diseño probablemente es demasiado complejo. |
| Memoria por etapa de agregación | **100 MB** (con posibilidad de volcar a disco) | Una etapa como `$group` o `$sort` que necesite más memoria fallará, o usará disco (más lento) si se permite (`allowDiskUse`). |
| Memoria por operación de consulta (nuevo en 9.0) | 1 GB o el 20 % de la memoria del proceso, lo que sea mayor | Límite total para una consulta; si se supera, falla con `ExceededMemoryLimit`. |
| Espacio mínimo por índice | ~8 kB | Cada índice tiene un coste aunque la colección esté vacía: no crees índices "por si acaso". |

---

## 2. Principio fundamental

> **Los datos que se consultan juntos deben almacenarse juntos.**

El esquema se diseña a partir de los **patrones de acceso** de la aplicación (qué consultas se hacen, con qué frecuencia y qué datos devuelven), no a partir de las entidades en abstracto como en la normalización relacional.

**Ejemplo:** una entrada de blog con sus comentarios, que la aplicación siempre muestra juntos.

En una base de datos **relacional** se repartiría en dos tablas y habría que unirlas en cada lectura:

```sql
-- tabla entradas:     id | titulo          | texto
--                      1 | "Hola MongoDB"  | "..."
-- tabla comentarios:  id | entrada_id | autor  | texto
--                      7 |          1 | "Ana"  | "¡Genial!"
--                      8 |          1 | "Luis" | "Gracias"
SELECT * FROM entradas e JOIN comentarios c ON c.entrada_id = e.id WHERE e.id = 1;
```

En **MongoDB**, como se consultan juntos, se guardan juntos en un solo documento:

```javascript
{
  _id: 1,
  titulo: "Hola MongoDB",
  texto: "...",
  comentarios: [
    { autor: "Ana",  texto: "¡Genial!" },
    { autor: "Luis", texto: "Gracias" }
  ]
}
// Una sola lectura, sin uniones:
db.entradas.findOne({ _id: 1 })
```

> Pero si una entrada pudiera recibir **millones** de comentarios, este diseño ya no serviría (el documento crecería sin límite). Por eso la pregunta clave no es solo "¿se leen juntos?", sino también "¿cuántos hay?". Las secciones 5 y 6 explican cómo decidir.

### Ejemplos de uso de la flexibilidad del modelo

- **Empleados y departamentos:** incrustar la información del departamento en cada documento de empleado permite obtenerla en una sola consulta.
- **Reseñas de productos:** la ficha del producto muestra las 5 reseñas más recientes embebidas; el histórico completo se guarda en una colección aparte, porque se consulta poco (patrón *subset*).
- **Catálogo de una tienda de ropa:** productos con atributos distintos (tallas, materiales, dimensiones) conviven en la misma colección (patrón polimórfico).

---

## 3. Diferencias de diseño frente a una base de datos relacional

| Base de datos relacional | Base de datos documental (MongoDB) |
|---|---|
| El esquema de la tabla se define **antes** de insertar datos; cambiarlo requiere planificar el impacto en las referencias dependientes. | El modelo se puede **cambiar con facilidad** a medida que evoluciona la aplicación. |
| A menudo hay que **unir varias tablas** (JOIN) para obtener lo que necesita la aplicación. | Los datos se almacenan **según los patrones de acceso**; incrustar datos evita uniones complejas y reduce la carga del servidor. |
| Se normaliza para **evitar redundancia**. | Se **desnormaliza con criterio** cuando mejora las lecturas. |
| La integridad referencial la impone el motor (claves foráneas). | La integridad la gestiona el modelo, la aplicación, las transacciones o la validación. |
| Se diseña pensando en **qué datos hay**. | Se diseña pensando en **cómo se usan los datos**. |

---

## 4. Proceso de diseño del esquema (4 pasos)

Conviene **planificar el esquema al principio** del desarrollo: aunque sea flexible, modificar esquemas grandes en producción sigue siendo costoso. Una estrategia razonable es empezar con un esquema sencillo que cubra la funcionalidad básica y optimizar después.

### Paso 1. Identificar la carga de trabajo

- Enumerar las **operaciones más frecuentes** (lecturas y escrituras).
- Estimar volumen de datos, frecuencia de cada operación, relación lectura/escritura y requisitos de latencia.
- Priorizar las operaciones críticas para el negocio.

| Operación | Tipo | Frecuencia | Datos | Prioridad |
|---|---|---|---|---|
| Ver ficha de producto | Lectura | 10 000/min | Producto + 5 reseñas | Alta |
| Añadir reseña | Escritura | 50/min | Reseña | Media |
| Informe mensual de ventas | Lectura | 1/mes | Pedidos agregados | Baja |

**Cómo leer esta tabla:** la ficha de producto se lee 10 000 veces por minuto y las reseñas se escriben solo 50 veces por minuto. Por tanto, compensa que **leer** la ficha sea lo más rápido posible (por ejemplo, embebiendo las 5 últimas reseñas en el producto), aunque eso haga **escribir** una reseña un poco más costoso (hay que actualizar también el producto). El informe mensual, en cambio, puede ser lento: se ejecuta una vez al mes.

### Paso 2. Mapear las relaciones

- Identificar las entidades y sus relaciones (1:1, 1:N, N:M). *Ejemplo: un cliente tiene muchos pedidos (1:N); un pedido contiene muchos productos y un producto aparece en muchos pedidos (N:M).*
- Decidir para cada relación si se **embebe** o se **referencia** (ver sección 6).
- Estimar la **cardinalidad**: ¿uno a pocos, a muchos o a muchísimos? No es lo mismo "un pedido tiene entre 1 y 20 líneas" que "un producto tiene un número ilimitado de reseñas".

### Paso 3. Aplicar patrones de diseño

- Usar patrones (sección 11) para optimizar lecturas y escrituras. Los patrones son **soluciones ya probadas** a problemas habituales (por ejemplo, "guardar solo las 5 últimas reseñas en el producto" es el patrón *Subset*). No hace falta inventarlas.

### Paso 4. Crear índices

- Crear índices que den soporte a las consultas más comunes (sección 9). Vuelve a la tabla del paso 1: cada operación frecuente debería tener un índice que la acelere.

> **Herramientas de apoyo:** MongoDB Compass incluye una función de **modelado de datos** (diagramas del esquema) y **Relational Migrator** ayuda a convertir esquemas relacionales a documentales.

---

## 5. Relaciones entre documentos

El valor de un campo puede ser cualquier tipo BSON, incluidos **subdocumentos**, **arrays** y **arrays de subdocumentos**. Con ellos se representan:

| Relación | Ejemplo |
|---|---|
| **Uno a uno** | Un paciente tiene exactamente un historial médico. |
| **Uno a muchos** | Un usuario tiene muchas publicaciones o comentarios. |
| **Muchos a muchos** | Un estudiante cursa varias asignaturas y cada asignatura tiene varios estudiantes. |

### 5.1. Uno a uno (embebido)

```javascript
{
  _id: "paciente01",
  nombre: "Lucía Martín",
  historial: {
    grupoSanguineo: "A+",
    alergias: ["penicilina"],
    ultimaRevision: ISODate("2026-09-14")
  }
}
```

- El historial es un **subdocumento** (un objeto dentro del documento del paciente).
- **Por qué embeber:** cada paciente tiene un solo historial, el historial no tiene sentido sin el paciente y normalmente se consultan juntos.
- `ISODate("...")` es la forma de escribir una **fecha** en mongosh (se guarda como tipo fecha, no como texto).
- Para consultar un campo de un subdocumento se usa la **notación de punto**: `db.pacientes.find({ "historial.grupoSanguineo": "A+" })` (las comillas son obligatorias cuando la clave lleva punto).

### 5.2. Uno a pocos (embebido)

```javascript
{
  _id: "usuario01",
  nombre: "Jorge Ruiz",
  direcciones: [
    { tipo: "casa", calle: "Gran Vía 1", ciudad: "Madrid" },
    { tipo: "trabajo", calle: "Av. Diagonal 200", ciudad: "Barcelona" }
  ]
}
```

- `direcciones` es un **array de subdocumentos**.
- **Por qué embeber:** un usuario tiene **pocas** direcciones (2, 3, quizá 5), así que el array nunca crecerá demasiado, y al mostrar el perfil o hacer un pedido se necesitan a la vez que el usuario.
- Consulta de ejemplo (usuarios con alguna dirección en Madrid): `db.usuarios.find({ "direcciones.ciudad": "Madrid" })`.

### 5.3. Uno a muchos (referencias en el lado "muchos")

```javascript
// colección editoriales
{ _id: "anaya", nombre: "Anaya", fundada: 1959, pais: "España" }

// colección libros
{ _id: 123, titulo: "Aprende MongoDB", editorial_id: "anaya" }
{ _id: 456, titulo: "Bases de datos NoSQL", editorial_id: "anaya" }
```

- Hay **dos colecciones**. Cada libro guarda en `editorial_id` el `_id` de su editorial (como una clave foránea en SQL, aunque MongoDB **no comprueba** que exista).
- **Por qué referenciar:** una editorial puede publicar miles de libros, y los libros tienen sentido por sí mismos (se buscan, se listan y se editan sin necesidad de cargar la editorial).
- Para obtener los libros de una editorial: `db.libros.find({ editorial_id: "anaya" })` (conviene un índice sobre `editorial_id`).

> Guardar la referencia en el lado "muchos" evita que el documento de la editorial contenga un **array que crece sin límite**. La alternativa errónea sería:
>
> ```javascript
> // ❌ Mal: el array libros crece con cada libro publicado
> { _id: "anaya", nombre: "Anaya", libros: [123, 456, 789, /* ... miles ... */] }
> ```

### 5.4. Muchos a muchos

```javascript
// estudiantes
{ _id: "e1", nombre: "Marta", asignaturas: ["bd", "redes"] }

// asignaturas
{ _id: "bd", nombre: "Bases de Datos", creditos: 6 }
```

- Cada estudiante guarda un **array de referencias** (`asignaturas`) con los `_id` de las asignaturas que cursa.
- Funciona bien porque un estudiante cursa **pocas** asignaturas. No se guarda el array inverso (asignatura → lista de estudiantes), porque una asignatura puede tener cientos.
- Estudiantes de Bases de Datos: `db.estudiantes.find({ asignaturas: "bd" })`.

Alternativas: arrays de referencias en uno o ambos lados, o una **colección intermedia** (`matriculas`) si la relación tiene atributos propios (nota, fecha) o es muy numerosa:

```javascript
// matriculas: un documento por cada par estudiante-asignatura (como una tabla intermedia en SQL)
{ estudianteId: "e1", asignaturaId: "bd",    nota: 8.5, fecha: ISODate("2026-09-15") }
{ estudianteId: "e1", asignaturaId: "redes", nota: 7.0, fecha: ISODate("2026-09-15") }
```

### 5.5. Unir datos referenciados con `$lookup`

Cuando los datos están referenciados y necesitas verlos juntos, `$lookup` hace algo parecido a un `LEFT JOIN` de SQL dentro de una agregación:

```javascript
db.libros.aggregate([
  { $match: { _id: 123 } },
  { $lookup: {
      from: "editoriales",
      localField: "editorial_id",
      foreignField: "_id",
      as: "editorial"
  } },
  { $unwind: "$editorial" }
])
```

| Parte | Explicación |
|---|---|
| `$match: { _id: 123 }` | Se queda solo con el libro 123. |
| `from: "editoriales"` | Colección con la que se une. |
| `localField: "editorial_id"` | Campo del **libro** que contiene la referencia. |
| `foreignField: "_id"` | Campo de la **editorial** con el que debe coincidir. |
| `as: "editorial"` | Nombre del campo nuevo donde se guardan las coincidencias. **Siempre es un array**, aunque solo haya una. |
| `$unwind: "$editorial"` | Convierte ese array de un elemento en un subdocumento normal, para que sea más cómodo de usar. |

**Resultado:**

```javascript
{
  _id: 123,
  titulo: "Aprende MongoDB",
  editorial_id: "anaya",
  editorial: { _id: "anaya", nombre: "Anaya", fundada: 1959, pais: "España" }
}
```

> `$lookup` es útil, pero si lo necesitas en **casi todas** las consultas, es señal de que esos datos deberían estar embebidos (ver el antipatrón "exceso de `$lookup`" en la sección 12).

---

## 6. Embeber o referenciar: cómo decidir

**Resumen rápido** — hazte estas tres preguntas sobre los datos "hijo":

1. **¿Se leen casi siempre junto con el padre?** Si sí → tiende a **embeber**.
2. **¿Cuántos puede haber?** Pocos y acotados → **embeber**. Muchos o sin límite → **referenciar**.
3. **¿Tienen sentido por sí solos?** (se buscan, listan o editan sin el padre) Si sí → **referenciar**.

En caso de duda, empieza embebiendo (es más sencillo) y pasa a referenciar cuando aparezca uno de los problemas de la tabla siguiente.

### Tabla de decisión oficial

| Escenario | Método recomendado |
|---|---|
| Tener los datos juntos simplifica el modelo y el código | **Embeber** |
| Relación de tipo "tiene un" o "contiene" | **Embeber** |
| La aplicación consulta esa información a la vez | **Embeber** |
| Los datos se actualizan a menudo juntos | **Embeber** |
| Los datos deben archivarse al mismo tiempo | **Embeber** |
| El lado hijo de la relación tiene **alta cardinalidad** | **Referenciar** |
| La duplicación sería demasiado compleja de mantener | **Referenciar** |
| El tamaño combinado consume demasiada memoria o ancho de banda | **Referenciar** |
| Los datos embebidos **crecerían sin límite** | **Referenciar** |
| Los datos se escriben en momentos distintos en una carga intensiva en escrituras | **Referenciar** |
| El hijo puede existir por sí mismo sin el padre | **Referenciar** |

### Regla práctica de cardinalidad

| Cardinalidad | Estrategia habitual |
|---|---|
| Uno a **pocos** (decenas) | Array de subdocumentos embebidos |
| Uno a **muchos** (cientos o miles) | Array de referencias en el padre, o referencia en el hijo |
| Uno a **muchísimos** (sin límite) | Referencia al padre en cada hijo |

```javascript
// Uno a POCOS → hijos embebidos dentro del padre
{ _id: "u1", nombre: "Jorge", direcciones: [ { ciudad: "Madrid" }, { ciudad: "Barcelona" } ] }

// Uno a MUCHOS → el padre guarda un array con los _id de los hijos
{ _id: "prod1", nombre: "Coche", piezas: ["pieza1", "pieza2", /* ... cientos ... */] }

// Uno a MUCHÍSIMOS → cada hijo guarda el _id del padre (el padre no sabe nada de sus hijos)
{ _id: "log1", servidorId: "srv1", mensaje: "Arranque correcto" }
{ _id: "log2", servidorId: "srv1", mensaje: "Disco al 80 %" }
```

### Ventajas e inconvenientes

| | Embeber | Referenciar |
|---|---|---|
| **Lecturas** | Una sola consulta | Varias consultas o `$lookup` |
| **Atomicidad** | Garantizada en un documento | Requiere transacción para varias colecciones |
| **Tamaño del documento** | Crece (límite de 16 MB) | Documentos pequeños |
| **Duplicación** | Posible | Mínima |
| **Independencia del hijo** | No | Sí |

### Ejemplo de evolución del modelo

Una plataforma de formación en línea empieza guardando cada curso con sus **lecciones embebidas** (título, descripción y diapositivas). Cuando se añaden vídeos, cuestionarios, tareas y enlaces externos, el documento del curso se vuelve demasiado complejo. La solución es pasar las lecciones a una **colección `lecciones`** referenciada desde el curso, lo que da más flexibilidad para nuevos formatos.

---

## 7. Duplicación de datos y coherencia

Incrustar datos relacionados puede implicar **duplicarlos** en varias colecciones. Por ejemplo, si cada pedido guarda una copia del nombre del cliente, ese nombre está en `clientes` **y** en cada uno de sus pedidos. El problema aparece cuando el dato cambia: ¿hay que actualizar todas las copias? Antes de duplicar, hay que valorar:

- **El beneficio en lectura:** evitar uniones entre colecciones.
- **La frecuencia de actualización** del dato duplicado: si cambia poco, mantenerlo es barato; si cambia a menudo, puede ser mejor referenciar.

### Tipos de datos duplicados

| Tipo | Descripción | Ejemplo |
|---|---|---|
| **Inmutable** | Nunca cambia. Candidato ideal a duplicar. | Fecha de alta de un usuario. |
| **Temporal** | Cambia, pero interesa conservar el valor histórico. | Dirección de envío en el momento del pedido. |
| **Sensible a la obsolescencia** | Debe estar siempre al día en todas sus copias. | Stock de un producto. |
| **Tolerante a la obsolescencia** | Puede estar desfasado un tiempo. | Lista de productos recomendados. |

> **Fíjate en el caso "temporal":** aquí la copia **no es un problema, es lo que se quiere**. Si un cliente se muda, sus pedidos antiguos deben seguir mostrando la dirección a la que se enviaron, no la nueva. Lo mismo pasa con el precio de un producto dentro de una factura.

### Cómo mantener la coherencia

"Coherencia" significa que todas las copias de un dato digan lo mismo.

| Método | Cuándo usarlo |
|---|---|
| **Embeber los datos relacionados** | Coherencia inmediata y atómica en un solo documento: si no hay copias, no hay nada que sincronizar. |
| **Transacciones** | Actualizar varias copias de forma atómica (más coste de rendimiento). |
| **Atlas Database Triggers / Change Streams** | Propagar cambios de forma asíncrona (**coherencia eventual**: durante unos instantes alguna copia puede estar desactualizada, pero acaba sincronizándose). Un *change stream* es un "aviso" que MongoDB envía a tu aplicación cada vez que cambia un documento. |
| **Tareas en segundo plano** | Datos tolerantes a la obsolescencia; recálculo periódico (p. ej., un proceso nocturno que actualiza las recomendaciones). |

Ejemplo de propagación manual cuando un cliente cambia su nombre:

```javascript
db.clientes.updateOne({ _id: 7 }, { $set: { nombre: "Lucía Martín López" } })
db.pedidos.updateMany({ clienteId: 7 }, { $set: { "cliente.nombre": "Lucía Martín López" } })
// Si ambas actualizaciones deben ocurrir sí o sí a la vez, se envuelven en una transacción.
```

---

## 8. Validación de esquemas

Por defecto MongoDB acepta cualquier documento. La **validación de esquema** permite definir reglas (campos obligatorios, tipos, rangos de valores…) para que la base de datos **rechace** los documentos que no las cumplan, igual que una tabla SQL rechaza un texto en una columna numérica.

La validación es más útil en aplicaciones **maduras** con una estructura de datos ya definida. No es obligatorio cubrir todos los campos: puede aplicarse solo a los críticos.

```javascript
db.createCollection("estudiantes", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["nombre", "email", "notaMedia"],
      properties: {
        nombre:    { bsonType: "string", description: "obligatorio y de tipo cadena" },
        email:     { bsonType: "string", pattern: "^.+@.+\\..+$" },
        notaMedia: { bsonType: "number", minimum: 0, maximum: 10 },
        curso:     { enum: ["1º", "2º", "3º", "4º"] }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
})
```

**Qué significa cada parte:**

| Parte | Explicación |
|---|---|
| `validator: { $jsonSchema: {...} }` | Las reglas se escriben con **JSON Schema**, un estándar para describir la forma de un documento JSON. |
| `bsonType: "object"` | El documento en sí debe ser un objeto (siempre se pone así en el nivel raíz). |
| `required: [...]` | Campos **obligatorios**: si falta alguno, el documento se rechaza. |
| `properties` | Reglas para cada campo. Un campo que **no** aparece aquí se acepta sin comprobar. |
| `nombre: { bsonType: "string" }` | Debe ser texto. `description` es solo un comentario que aparece en el mensaje de error. |
| `email: { pattern: "..." }` | Debe cumplir una **expresión regular**: algo, `@`, algo, un punto, algo. (La doble barra `\\.` es necesaria para escribir un punto literal dentro de una cadena). |
| `notaMedia: { bsonType: "number", minimum: 0, maximum: 10 }` | Debe ser un número entre 0 y 10. `"number"` acepta cualquier tipo numérico (entero, decimal…). |
| `curso: { enum: [...] }` | Solo admite uno de los valores de la lista. Como `curso` no está en `required`, puede faltar; pero si está, debe ser válido. |
| `validationLevel` / `validationAction` | Cuándo se valida y qué hacer si falla (ver tabla siguiente). |

> **Cuidado con los tipos numéricos:** mongosh guarda `8` como entero (`int`) y `8.5` como `double`. Si hubiéramos puesto `bsonType: "double"`, un documento con `notaMedia: 8` sería **rechazado**. Por eso se usa `"number"` (o una lista: `bsonType: ["int", "double"]`).

**Qué pasa si se incumple una regla:**

```javascript
db.estudiantes.insertOne({ nombre: "Pablo", email: "pablo-sin-arroba", notaMedia: 12 })
// MongoServerError: Document failed validation
// (el detalle del error indica que email no cumple el patrón y notaMedia supera el máximo)
```

Para añadir o cambiar la validación de una colección **que ya existe** se usa `db.runCommand({ collMod: "estudiantes", validator: { $jsonSchema: {...} } })`.

### Niveles y acciones de validación

| Opción | Valor | Efecto |
|---|---|---|
| `validationLevel` | `strict` (por defecto) | Valida todas las inserciones y actualizaciones. |
| | `moderate` | Valida las inserciones, y las actualizaciones solo de documentos que ya eran válidos. Útil al añadir validación a una colección con datos antiguos "imperfectos": no te impide modificarlos. |
| | **`constraint`** (nuevo en 9.0) | Además de validar inserciones y actualizaciones, **garantiza que todos los documentos de la colección cumplen las reglas**. |
| | `off` | Desactiva la validación. |
| `validationAction` | `error` | Rechaza el documento no válido. |
| | `warn` | Lo acepta y registra un aviso en el *log* del servidor. Útil para probar reglas nuevas sin romper la aplicación. |

Ejemplos de uso:

- Asegurar que `fechaInicio` de un evento sea una **fecha** y no una cadena.
- Restringir `tarjetasAceptadas` a `["Visa", "MasterCard", "American Express"]`.
- Garantizar que `notaMedia` sea un número positivo.

> También existe una guía específica para **validar colecciones polimórficas** (documentos de varios tipos en la misma colección) usando `oneOf`/`anyOf` en `$jsonSchema`.

---

## 9. Índices y modelado

Un **índice** funciona como el índice de un libro: en lugar de revisar todos los documentos de la colección, MongoDB va directamente a los que le interesan. Si un campo se usa con frecuencia para **filtrar, ordenar o unir**, conviene indexarlo. Los índices permiten:

- Devolver resultados más rápido.
- Ordenar de forma más eficiente.
- Optimizar `$lookup` y `$group`.
- Reducir el uso de CPU y E/S.

Pero tienen coste:

- Cada índice ocupa **al menos 8 kB** y consume disco y memoria.
- Cada escritura debe **actualizar todos los índices** de la colección: en colecciones con muchas escrituras, los índices son caros.
- En colecciones con muchas lecturas, los índices adicionales suelen compensar.
- Hay que vigilar que el **conjunto de trabajo** (*working set*: datos e índices de uso frecuente) quepa en RAM. Si no cabe, MongoDB tiene que leer continuamente del disco, que es mucho más lento.

**Tipos de índices disponibles:**

| Tipo | Para qué sirve |
|---|---|
| Simple | Un solo campo: `{ email: 1 }`. |
| Compuesto | Varios campos; el orden importa: `{ clienteId: 1, fecha: -1 }`. |
| Multiclave | Se crea automáticamente al indexar un array: indexa cada elemento. |
| Texto | Búsqueda de palabras dentro de textos. |
| Geoespacial (`2dsphere`) | Consultas por ubicación ("tiendas a menos de 5 km"). |
| *Hashed* | Reparte los valores de forma uniforme; se usa sobre todo en *sharding*. |
| Comodín (*wildcard*) | Indexa campos cuyo nombre no conoces de antemano. |
| Parcial | Solo indexa los documentos que cumplen un filtro (p. ej., solo pedidos activos). |
| *Sparse* | Solo indexa los documentos que tienen el campo. |
| Único | Impide valores repetidos (p. ej., emails). |
| TTL | Borra documentos automáticamente pasado un tiempo. |
| Agrupado (*clustered*) | Guarda los documentos físicamente ordenados por `_id`. |

Además, en Atlas: índices de **Search** (búsqueda de texto avanzada) y **Vector Search** (búsqueda semántica para IA).

```javascript
// Supervisar el uso real de los índices
db.pedidos.aggregate([{ $indexStats: {} }])
```

`$indexStats` devuelve un documento por índice. Mira el campo `accesses.ops`: indica **cuántas veces se ha usado** el índice desde que arrancó el servidor (`accesses.since`). Un índice con `ops: 0` tras semanas de funcionamiento probablemente sobra y se puede eliminar con `db.pedidos.dropIndex("nombre_del_indice")`.

---

## 10. Otras consideraciones

### 10.1. Atomicidad

- En MongoDB, una escritura es **atómica a nivel de documento**, aunque modifique varios subdocumentos.
- Un modelo **desnormalizado** (subdocumentos y arrays) permite operaciones atómicas sin transacciones.
- Con referencias, la aplicación necesita **varias operaciones** de lectura y escritura.
- Para atomicidad entre varios documentos o colecciones existen **transacciones distribuidas**.

**Ejemplo:** al registrar una venta hay que restar stock y anotar la venta.

```javascript
// ✅ Modelo embebido: UNA sola operación, atómica sin transacción
db.productos.updateOne(
  { _id: 42, stock: { $gte: 1 } },                       // solo si queda stock
  { $inc: { stock: -1 }, $push: { ventas: { fecha: new Date(), cantidad: 1 } } }
)

// Modelo referenciado: DOS operaciones sobre dos colecciones.
// Si la segunda falla, la primera ya se ha aplicado → hace falta una transacción.
db.productos.updateOne({ _id: 42 }, { $inc: { stock: -1 } })
db.ventas.insertOne({ productoId: 42, fecha: new Date(), cantidad: 1 })
```

> **Importante:** las transacciones tienen un coste de rendimiento mayor que las escrituras de un solo documento y **no sustituyen a un buen diseño del esquema**. En la mayoría de los casos, modelar bien los datos minimiza la necesidad de transacciones. Recuerda además los límites de tiempo de ejecución y de tamaño del oplog, y que en 9.0 hay un máximo de 10 000 transacciones simultáneas por defecto.

### 10.2. Gestión del ciclo de vida de los datos

- **Índices TTL:** borrado automático tras un tiempo (p. ej., sesiones que caducan a los 30 minutos de inactividad).
- **Colecciones de tamaño fijo (*capped*):** comportamiento FIFO (*first in, first out*: cuando se llenan, los documentos más antiguos se borran automáticamente para dejar sitio a los nuevos). Útiles cuando solo interesan los documentos más recientes, como los últimos *logs*.
- **Patrón de archivo:** mover datos antiguos a otra colección o a almacenamiento más barato (Atlas Online Archive).

```javascript
// Índice TTL: borra cada sesión 30 minutos (1800 s) después de su "ultimoAcceso"
db.sesiones.createIndex({ ultimoAcceso: 1 }, { expireAfterSeconds: 1800 })

// Colección capped: como máximo 10 MB; al llenarse, se sobrescriben los más antiguos
db.createCollection("logs", { capped: true, size: 10 * 1024 * 1024 })
```

### 10.3. Restricciones de hardware

- Los documentos grandes consumen más RAM y pueden obligar a leer de disco.
- Diseñar para que las consultas devuelvan **solo los campos necesarios** (proyecciones) y que el *working set* no crezca innecesariamente. Por ejemplo, para un listado de productos basta con `db.productos.find({}, { nombre: 1, precio: 1 })`; no hace falta traer la descripción larga ni las reseñas.

### 10.4. Documentos pequeños

Cuando hay muchísimos documentos diminutos, la sobrecarga por documento se vuelve significativa:

- **Usar `_id` con un valor propio y significativo** en lugar de un `ObjectId` de 12 bytes que no aporta información.
- **Acortar nombres de campo** (MongoDB los guarda en cada documento). Ojo: reduce la legibilidad y no afecta al tamaño de los índices; suele ser más eficaz replantear el modelo.
- **Agrupar en documentos embebidos** (patrón *bucket*).

```javascript
// Antes:  { last_name: "Smith", best_score: 3.9 }
// Después: { lname: "Smith", score: 3.9 }   // ahorra 9 bytes por documento
```

> El ahorro sale de los nombres: `last_name` + `best_score` = 19 caracteres; `lname` + `score` = 10. Nueve bytes parecen poco, pero en 1 000 millones de documentos son unos 9 GB. Aun así, **en una aplicación normal no merece la pena**: prioriza nombres claros.

### 10.5. Fragmentación (*sharding*)

La fragmentación reparte una colección entre varios servidores (*shards*) cuando ya no cabe o no rinde en uno solo. Para decidir en qué servidor va cada documento se usa la **clave de fragmentación** (*shard key*), un campo (o varios) de cada documento.

Si la colección se va a fragmentar, el modelo debe contemplar una clave de fragmentación con:

- **Alta cardinalidad:** muchos valores distintos (`clienteId` sí; `pais` no, porque con pocos valores no se puede repartir bien).
- **Buena distribución de escrituras:** que las inserciones no vayan todas al mismo servidor (una fecha creciente, por ejemplo, manda siempre los documentos nuevos al último *shard*).
- **Alineada con las consultas más frecuentes:** si la consulta incluye la clave, MongoDB sabe a qué servidor preguntar. Si no, tiene que preguntar a **todos** y juntar las respuestas (consulta *scatter-gather*, "dispersar y recoger"), que es más lenta.

---

## 11. Patrones de diseño de esquemas

Un **patrón de diseño** es una solución ya probada para un problema que aparece una y otra vez (igual que los patrones de diseño en Java, como *Singleton* o *Builder*). No hace falta memorizarlos todos: lo importante es reconocer el problema cuando aparezca y saber que existe una solución conocida.

La documentación actual agrupa los patrones en seis categorías:

| Categoría oficial | Objetivo | Patrones clásicos relacionados |
|---|---|---|
| **Valores calculados** | Calcular en escritura para que la lectura sea inmediata. | *Computed*, *Approximation* |
| **Agrupar datos** | Agrupar en series para mejorar rendimiento y tratar valores atípicos. | *Bucket*, *Outlier*, *Subset*, *Extended Reference* |
| **Datos polimórficos** | Documentos con campos y tipos distintos en una misma colección. | *Polymorphic*, *Attribute* |
| **Versionado de documentos y esquemas** | Prepararse para cambios de esquema y conservar históricos. | *Schema Versioning*, *Document Versioning* |
| **Archivo** | Mover datos antiguos para liberar espacio y mejorar el rendimiento. | *Archive* |
| **Colección única** | Agrupar documentos relacionados de distintos tipos en una sola colección, vinculados por referencias. | *Single Collection* |

### 11.1. Valores calculados (*Computed*)

- **Problema:** mostrar la valoración media de un producto obligaría a leer y sumar **todas** sus reseñas cada vez que alguien abre la ficha (miles de veces por minuto).
- **Solución:** calcular el resultado **al escribir** (pocas veces) y guardarlo, para que leer (muchas veces) sea inmediato.

```javascript
// En cada nueva reseña (aquí, de 5 estrellas) se actualizan los totales del producto
db.productos.updateOne(
  { _id: 42 },
  { $inc: { "valoraciones.total": 1, "valoraciones.suma": 5 } }
)
```

- `$inc` suma 1 al número de reseñas (`total`) y suma la nota (5) a `suma`.
- `"valoraciones.total"` usa la notación de punto para modificar un campo dentro del subdocumento `valoraciones`.
- La media se obtiene sin recorrer las reseñas: `suma / total`.

**Aproximación (*Approximation*):** actualizar un contador solo cada N eventos (p. ej., visitas) cuando no se necesita exactitud absoluta. Por ejemplo, en lugar de escribir en la base de datos en cada visita a una página, la aplicación suma 100 al contador cada 100 visitas: el número mostrado es aproximado, pero se hacen 100 veces menos escrituras.

### 11.2. Agrupación (*Bucket*)

- **Problema:** un sensor que envía una temperatura cada minuto genera 525 600 documentos diminutos al año. Muchos documentos pequeños desperdician espacio (cada uno repite `sensor`, tiene su `_id`, su entrada en cada índice…).
- **Solución:** agrupar muchas mediciones pequeñas en **un documento por intervalo** (aquí, uno por hora con 60 medidas).

```javascript
{
  sensor: "temp-07",
  inicio: ISODate("2026-10-22T10:00:00Z"),
  fin:    ISODate("2026-10-22T10:59:59Z"),
  num: 60,
  medidas: [ { t: ISODate("2026-10-22T10:00:00Z"), v: 21.4 }, /* ... */ ],
  suma: 1290.5
}
```

- `inicio` / `fin`: el intervalo que cubre el documento (el "cubo" o *bucket*).
- `medidas`: array con las mediciones de esa hora (`t` = momento, `v` = valor).
- `num` y `suma`: valores precalculados (patrón *Computed*); la media de la hora es `suma / num` sin recorrer el array.
- Cuando `num` llega a 60, la siguiente medida va a un documento nuevo. Así el array **nunca crece sin límite**.

> Para este caso, las **colecciones de series temporales** nativas aplican el *bucketing* automáticamente (en 9.0 se almacenan como un único espacio de nombres comprimido).

### 11.3. Valores atípicos (*Outlier*)

- **Problema:** el diseño funciona para el 99 % de los casos, pero unos pocos documentos son excepcionales. Por ejemplo, guardar los compradores de cada libro en un array va bien para casi todos los libros (decenas de compradores), pero un superventas tiene millones.
- **Solución:** no cambiar el diseño para todos por culpa de unos pocos. Se marca el documento excepcional (`tieneDesbordamiento: true`) y el exceso se guarda en documentos adicionales.

```javascript
// Libro normal: todo en un documento
{ _id: "libro1", titulo: "Poesía local", compradores: ["u1", "u2", "u3"] }

// Libro superventas: los primeros 1000 aquí y el resto en documentos de desbordamiento
{ _id: "libro2", titulo: "Superventas", compradores: [/* 1000 ids */], tieneDesbordamiento: true }
{ libroId: "libro2", pagina: 1, compradores: [/* 1000 ids más */] }
{ libroId: "libro2", pagina: 2, compradores: [/* ... */] }
```

La aplicación solo consulta los documentos extra cuando ve `tieneDesbordamiento: true`.

### 11.4. Subconjunto (*Subset*)

- **Problema:** un producto con 5 000 reseñas embebidas genera un documento enorme, aunque la ficha solo muestre las 5 últimas.
- **Solución:** embeber solo la parte más usada (las 5 últimas reseñas) y dejar el resto en otra colección.

```javascript
// Al añadir una reseña: se guarda completa en "resenas"...
db.resenas.insertOne({ productoId: 42, usuario: "ana", nota: 5, texto: "Genial" })

// ...y se añade también al producto, conservando solo las 5 más recientes
db.productos.updateOne(
  { _id: 42 },
  { $push: { ultimasResenas: { $each: [{ usuario: "ana", nota: 5, texto: "Genial" }], $slice: -5 } } }
)
```

`$push` con `$each` y `$slice: -5` añade la reseña al final del array y luego **recorta** el array dejando solo los 5 últimos elementos. El histórico completo se consulta en `resenas` solo cuando el usuario pulsa "ver todas".

### 11.5. Referencia extendida (*Extended Reference*)

- **Problema:** cada vez que se muestra un pedido hay que hacer `$lookup` a `clientes` solo para enseñar el nombre del cliente.
- **Solución:** copiar en el documento que referencia los **pocos campos** que siempre se muestran (p. ej., nombre y dirección del cliente dentro del pedido), evitando `$lookup` constantes.

```javascript
{
  _id: "pedido1001",
  clienteId: 7,                                       // la referencia de siempre...
  cliente: { nombre: "Lucía Martín", ciudad: "Madrid" }, // ...más una copia de lo que se muestra
  total: 120
}
```

Se mantiene `clienteId` para acceder al cliente completo cuando haga falta. Copia **solo campos que cambian poco** (ver sección 7: duplicar datos que cambian a menudo obliga a actualizar muchas copias).

### 11.6. Polimórfico y atributos (*Polymorphic* / *Attribute*)

```javascript
// Polimórfico: distintos tipos de producto en la misma colección
{ tipo: "libro",   titulo: "...", isbn: "...",   paginas: 320 }
{ tipo: "camiseta", titulo: "...", talla: "M",   color: "azul" }

// Atributos: convertir muchos campos dispersos en un array indexable
{
  nombre: "Portátil X",
  atributos: [
    { k: "ram_gb", v: 16 },
    { k: "pantalla_pulgadas", v: 14 },
    { k: "peso_kg", v: 1.3 }
  ]
}
// Un único índice cubre todos los atributos:
db.productos.createIndex({ "atributos.k": 1, "atributos.v": 1 })
```

**Polimórfico:**

- **Problema:** productos de tipos distintos tienen campos distintos. En SQL acabarías con una tabla por tipo o con muchas columnas vacías.
- **Solución:** guardarlos todos en la misma colección, con un campo (`tipo`) que indica qué clase de documento es. La aplicación lee `tipo` para saber qué campos esperar (como el polimorfismo en Java: una clase `Producto` con subclases `Libro` y `Camiseta`).

**Atributos:**

- **Problema:** si cada portátil tiene campos `ram_gb`, `pantalla_pulgadas`, `peso_kg`… y los móviles tienen otros diferentes, para poder buscar por todos ellos harían falta **decenas de índices** (uno por campo).
- **Solución:** convertir esos campos en un array de pares **clave-valor** (`k` = nombre del atributo, `v` = valor). Así **un solo índice** sirve para buscar por cualquier atributo.

```javascript
// Portátiles con 16 GB de RAM
db.productos.find({ atributos: { $elemMatch: { k: "ram_gb", v: 16 } } })
```

`$elemMatch` exige que **el mismo elemento** del array cumpla ambas condiciones. Sin él, `{ "atributos.k": "ram_gb", "atributos.v": 16 }` también encontraría un producto con `ram_gb: 8` y `peso_kg: 16`, porque cada condición podría cumplirse en un elemento distinto.

### 11.7. Versionado

- **Versionado de esquema:** campo `versionEsquema` para que la aplicación convierta los documentos antiguos de forma progresiva, sin migraciones masivas ni paradas.
- **Versionado de documentos:** guardar revisiones anteriores en una colección de histórico (útil en auditoría, normativa, documentos legales).

```javascript
// Versión 1 (antigua): email y teléfono sueltos
{ _id: 2, nombre: "Luis", email: "...", tel: "..." }

// Versión 2 (nueva): agrupados en "contacto"
{ _id: 1, versionEsquema: 2, nombre: "Ana", contacto: { email: "...", tel: "..." } }
```

Ambas versiones conviven en la colección. La aplicación mira `versionEsquema` (si no existe, asume la versión 1) y lee los campos de la forma correcta. Cuando modifica un documento antiguo, lo guarda ya en el formato nuevo. Así la migración se hace **poco a poco**, sin detener la aplicación para convertir millones de documentos de golpe.

### 11.8. Archivo (*Archive*)

- **Problema:** la colección de pedidos acumula años de datos que casi nadie consulta, pero que ocupan espacio y memoria y hacen más lentos los índices.
- **Solución:** mover documentos antiguos (pedidos de hace más de 5 años) a otra colección, base de datos o almacenamiento de bajo coste, usando por ejemplo `$merge` o Atlas Online Archive.

```javascript
const limite = new Date("2021-10-01")

// 1. Copiar los pedidos antiguos a la colección de archivo
db.pedidos.aggregate([
  { $match: { fecha: { $lt: limite } } },
  { $merge: { into: "pedidos_archivo" } }
])

// 2. Borrarlos de la colección principal (después de comprobar que se han copiado)
db.pedidos.deleteMany({ fecha: { $lt: limite } })
```

### 11.9. Colección única (*Single Collection*)

- **Problema:** mostrar un pedido completo exige consultar tres colecciones (pedido, líneas, envío).
- **Solución:** guardar entidades relacionadas de distintos tipos en una sola colección, con un campo que las distingue y referencias comunes, para obtenerlas con **una sola consulta** indexada.

```javascript
{ _id: "pedido#1001",        tipoDoc: "pedido", pedidoId: 1001, cliente: "c7", total: 120 }
{ _id: "pedido#1001#linea1", tipoDoc: "linea",  pedidoId: 1001, producto: "Ratón", cantidad: 2 }
{ _id: "pedido#1001#envio",  tipoDoc: "envio",  pedidoId: 1001, estado: "en reparto" }

db.ventas.find({ pedidoId: 1001 })   // trae el pedido completo
```

- `tipoDoc` indica qué representa cada documento (pedido, línea o envío).
- `pedidoId` es el campo **común** a todos ellos: con un índice sobre `pedidoId`, una sola consulta trae todas las piezas del pedido.
- El `_id` se construye con un formato legible (`pedido#1001#linea1`) que deja claro a qué pertenece cada documento.
- Es un patrón avanzado: úsalo solo cuando las piezas se lean juntas muy a menudo y no puedan embeberse (por ejemplo, porque son demasiadas o se escriben por separado).

---

## 12. Antipatrones de diseño de esquemas

Formas ineficientes de estructurar el esquema que provocan complejidad y problemas de rendimiento.

| Antipatrón | Problema | Solución |
|---|---|---|
| **Arrays ilimitados** | Un array crece sin control: puede superar los 16 MB y degrada los índices multiclave. | Referenciar en el lado "muchos", patrón *subset* o *bucket*. |
| **Demasiadas colecciones** | Cada colección y sus índices consumen recursos del motor de almacenamiento. | Unificar colecciones con la misma estructura (p. ej., no crear una colección por día o por cliente). |
| **Índices innecesarios** | Ocupan disco y RAM y ralentizan las escrituras. | Eliminar índices no usados o redundantes (revisar con `$indexStats`). |
| **Documentos sobredimensionados** | Documentos con datos que casi nunca se leen juntos penalizan las consultas frecuentes. | Separar los datos poco consultados (patrón *subset*). |
| **Exceso de `$lookup`** | Uniones constantes que complican y ralentizan las consultas. | Embeber o usar referencia extendida para los datos que se leen juntos. |

Un **antipatrón** es lo contrario de un patrón: una forma de diseñar que parece razonable al principio pero que causa problemas cuando crecen los datos. Conocerlos ayuda a detectarlos a tiempo.

> **Antipatrón clásico adicional:** consultas **insensibles a mayúsculas** con expresiones regulares (`/^ana$/i`) que no aprovechan los índices. Solución: crear un índice con **intercalación** (*collation*) de fuerza 1 o 2 y consultar con la misma intercalación.

```javascript
db.usuarios.createIndex({ nombre: 1 }, { collation: { locale: "es", strength: 2 } })
db.usuarios.find({ nombre: "ana" }).collation({ locale: "es", strength: 2 })
```

- La **intercalación** (*collation*) son las reglas de un idioma para comparar y ordenar textos.
- `locale: "es"`: reglas del español (por ejemplo, la "ñ" va después de la "n").
- `strength` indica qué diferencias se tienen en cuenta al comparar:
  - `1`: ignora **mayúsculas y tildes** → "ana" = "Ana" = "Aná".
  - `2`: ignora **mayúsculas**, pero no las tildes → "ana" = "Ana", pero "ana" ≠ "Aná".
  - `3` (por defecto): distingue ambas.
- La consulta debe usar **la misma** intercalación que el índice; si no, MongoDB no podrá usarlo.

### Herramientas para detectarlos

- **Performance Advisor de Atlas** (clústeres M10 o superiores): tarjeta *Improve Schema* con recomendaciones.
- **Performance Insights de MongoDB Compass**.
- **`$queryStats`** (en 9.0, activado por defecto con muestreo del 1 %).

---

## 13. Modelado de estructuras de árbol

Un **árbol** es una jerarquía en la que cada elemento tiene un "padre" (salvo la raíz) y puede tener "hijos": categorías de una tienda, carpetas de un disco, organigramas de una empresa… Todos los ejemplos de esta sección usan esta jerarquía de categorías:

```text
Libros
└── Informática
    └── Bases de datos
        ├── MongoDB
        └── PostgreSQL
```

Hay varias formas de guardarla. Cada una hace más fácil un tipo de consulta:

| Modelo | Cómo se guarda | Ventajas | Inconvenientes |
|---|---|---|---|
| **Referencias al padre** | `{ _id: "MongoDB", padre: "Bases de datos" }` | Sencillo; mover subárboles es fácil | Obtener un subárbol requiere varias consultas o `$graphLookup` |
| **Referencias a los hijos** | `{ _id: "Bases de datos", hijos: ["MongoDB", "PostgreSQL"] }` | Rápido para obtener hijos directos | Difícil obtener ancestros |
| **Array de ancestros** | `{ _id: "MongoDB", ancestros: ["Libros", "Informática", "Bases de datos"], padre: "Bases de datos" }` | Consultar ancestros y descendientes con un índice | Más espacio; actualizar al mover nodos |
| **Rutas materializadas** | `{ _id: "MongoDB", ruta: ",Libros,Informática,Bases de datos," }` | Subárboles con expresión regular anclada `/^,Libros,Informática,/` | Depende de cadenas; reorganizar es costoso |
| **Conjuntos anidados** | `{ _id: "MongoDB", izq: 4, der: 5 }` | Subárboles muy rápidos en árboles estáticos | Cada inserción obliga a renumerar |

```javascript
// Recorrido recursivo con $graphLookup (referencias al padre)
db.categorias.aggregate([
  { $match: { _id: "MongoDB" } },
  { $graphLookup: {
      from: "categorias",
      startWith: "$padre",
      connectFromField: "padre",
      connectToField: "_id",
      as: "ancestros"
  } }
])
```

`$graphLookup` es como un `$lookup` que **se repite** siguiendo una cadena de referencias:

| Parte | Explicación |
|---|---|
| `$match: { _id: "MongoDB" }` | Empezamos por la categoría "MongoDB". |
| `from: "categorias"` | Busca en la misma colección (el árbol está dentro de ella). |
| `startWith: "$padre"` | Primer valor a buscar: el padre de MongoDB → "Bases de datos". |
| `connectToField: "_id"` | Busca el documento cuyo `_id` coincide con ese valor. |
| `connectFromField: "padre"` | De ese documento toma su `padre` ("Informática") y repite la búsqueda, y así sucesivamente hasta llegar a la raíz (un documento sin padre). |
| `as: "ancestros"` | Guarda todos los documentos encontrados en este array: "Bases de datos", "Informática" y "Libros". |

> El array `ancestros` **no garantiza ningún orden**. Si necesitas saber el nivel de cada uno, añade `depthField: "nivel"` (0 = padre directo, 1 = abuelo…).

---

## 14. Ejemplo completo: tienda en línea

**Patrones de acceso principales:**

1. Ver ficha de producto con valoración media y últimas reseñas (muy frecuente).
2. Ver un pedido completo (frecuente).
3. Listar pedidos de un cliente (frecuente).
4. Añadir reseñas (moderado).

```javascript
// productos — patrones computed + subset + attribute
{
  _id: ObjectId("..."),
  sku: "PORT-X14",
  nombre: "Portátil X14",
  precio: NumberDecimal("899.99"),
  stock: 12,
  atributos: [{ k: "ram_gb", v: 16 }, { k: "ssd_gb", v: 512 }],
  valoraciones: { total: 248, media: 4.6 },
  ultimasResenas: [                       // máximo 5
    { usuario: "lucia_m", nota: 5, texto: "Muy ligero", fecha: ISODate("2026-10-20") }
  ]
}

// resenas — histórico completo, referenciado
{ _id: ObjectId("..."), productoId: ObjectId("..."), usuario: "lucia_m", nota: 5, texto: "...", fecha: ISODate("...") }

// clientes
{ _id: ObjectId("..."), nombre: "Lucía Martín", email: "lucia@ejemplo.es", direcciones: [ /* pocas */ ] }

// pedidos — referencia extendida + dato temporal duplicado
{
  _id: ObjectId("..."),
  clienteId: ObjectId("..."),
  cliente: { nombre: "Lucía Martín", email: "lucia@ejemplo.es" },      // referencia extendida
  direccionEnvio: { calle: "Gran Vía 1", ciudad: "Madrid", cp: "28013" }, // valor histórico
  lineas: [
    { sku: "PORT-X14", nombre: "Portátil X14", precioUnidad: NumberDecimal("899.99"), cantidad: 1 }
  ],
  total: NumberDecimal("899.99"),
  estado: "enviado",
  fecha: ISODate("2026-10-21T09:15:00Z")
}
```

**Cómo leer este modelo:**

- Hay **cuatro colecciones**: `productos`, `resenas`, `clientes` y `pedidos`. Las líneas de los pedidos y las direcciones de los clientes **no** tienen colección propia: van embebidas.
- `ObjectId("...")` es el identificador generado automáticamente por MongoDB; `productoId` y `clienteId` guardan el `ObjectId` de otro documento (son **referencias**).
- `NumberDecimal("899.99")` guarda un número **decimal exacto** (tipo `Decimal128`). `ISODate("...")` guarda una fecha.
- Cada colección está anotada con los patrones de la sección 11 que aplica.

**Índices:**

```javascript
db.productos.createIndex({ sku: 1 }, { unique: true })
db.productos.createIndex({ "atributos.k": 1, "atributos.v": 1 })
db.resenas.createIndex({ productoId: 1, fecha: -1 })
db.pedidos.createIndex({ clienteId: 1, fecha: -1 })
db.clientes.createIndex({ email: 1 }, { unique: true })
```

Cada índice existe para dar servicio a una consulta concreta:

| Índice | Consulta a la que da servicio |
|---|---|
| `productos: { sku: 1 }`, único | Buscar un producto por su código (`find({ sku: "PORT-X14" })`) y evitar dos productos con el mismo código. |
| `productos: { "atributos.k": 1, "atributos.v": 1 }` | Filtrar por características (patrón *Attribute*): "portátiles con 16 GB de RAM". |
| `resenas: { productoId: 1, fecha: -1 }` | "Ver todas las reseñas" de un producto, de la más reciente a la más antigua. |
| `pedidos: { clienteId: 1, fecha: -1 }` | Patrón de acceso 3: listar los pedidos de un cliente, los últimos primero. |
| `clientes: { email: 1 }`, único | Iniciar sesión por email e impedir dos cuentas con el mismo email. |

Fíjate en que el patrón de acceso 2 ("ver un pedido completo") no necesita un índice nuevo: se busca por `_id`, que siempre está indexado.

**Decisiones justificadas:**

- Las **líneas del pedido** se embeben: se leen siempre con el pedido, están acotadas y deben guardarse de forma atómica.
- El **precio en la línea** se copia (dato temporal): si el producto cambia de precio, el pedido conserva el original.
- Las **reseñas** se referencian: crecen sin límite. Solo las 5 últimas se embeben en el producto.
- Para dinero se usa **`Decimal128`** (`NumberDecimal`), no `double`, para evitar errores de redondeo (con `double`, `0.1 + 0.2` da `0.30000000000000004`; es el mismo motivo por el que en Java se usa `BigDecimal` para importes).
- Los **datos del cliente** se copian parcialmente en el pedido (referencia extendida): el pedido se muestra sin `$lookup`, y `clienteId` permite llegar al cliente completo si hace falta.

---

## 15. Lista de comprobación del diseño

- [ ] ¿He identificado las consultas más frecuentes y su volumen?
- [ ] ¿Los datos que se leen juntos están juntos?
- [ ] ¿Algún array puede crecer sin límite?
- [ ] ¿Algún documento puede acercarse a los 16 MB?
- [ ] ¿Sé qué datos duplicados existen y cómo se mantienen coherentes?
- [ ] ¿Necesito realmente transacciones o puede resolverlo el modelo?
- [ ] ¿Cada consulta frecuente tiene un índice adecuado (regla ESR)?
- [ ] ¿Hay índices que no se usan?
- [ ] ¿He definido validación de esquema en los campos críticos (`constraint` en 9.0 si quiero garantías totales)?
- [ ] ¿He previsto el versionado del esquema?
- [ ] ¿He pensado en el ciclo de vida de los datos (TTL, archivo)?
- [ ] Si voy a fragmentar, ¿tengo una buena clave de *sharding*?

---

## 16. Recursos

- [Modelado de datos en MongoDB](https://www.mongodb.com/es/docs/manual/data-modeling/)
- [Buenas prácticas de modelado](https://www.mongodb.com/es/docs/manual/data-modeling/best-practices/)
- [Diseñar el esquema](https://www.mongodb.com/es/docs/manual/data-modeling/schema-design-process/)
- [Patrones de diseño de esquemas](https://www.mongodb.com/es/docs/manual/data-modeling/design-patterns/)
- [Antipatrones de diseño de esquemas](https://www.mongodb.com/es/docs/manual/data-modeling/design-antipatterns/)
- [Datos embebidos](https://www.mongodb.com/es/docs/manual/data-modeling/embedding/) · [Datos referenciados](https://www.mongodb.com/es/docs/manual/data-modeling/referencing/)
- [Datos duplicados](https://www.mongodb.com/es/docs/manual/data-modeling/handle-duplicate-data/) · [Coherencia de datos](https://www.mongodb.com/es/docs/manual/data-modeling/data-consistency/)
- [Validación de esquemas](https://www.mongodb.com/es/docs/manual/core/schema-validation/)
- [Modelar relaciones](https://www.mongodb.com/es/docs/manual/applications/data-models-relationships/)
- [Estructuras de árbol](https://www.mongodb.com/es/docs/manual/applications/data-models-tree-structures/)
- [Notas de versión de MongoDB 9.0](https://www.mongodb.com/es/docs/manual/release-notes/9.0/)
- [Curso *Data Modeling for MongoDB* (MongoDB University)](https://learn.mongodb.com/learning-paths/data-modeling-for-mongodb)
