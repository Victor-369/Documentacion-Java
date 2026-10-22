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

El **modelado de datos** consiste en decidir cómo se organiza la información dentro de la base de datos y cómo se vinculan las entidades relacionadas.

MongoDB tiene un **modelo de datos flexible** que permite almacenar **datos polimórficos**:

- Los documentos de una misma colección **no tienen por qué compartir los mismos campos**.
- El **tipo de dato** de un campo puede variar entre documentos.

Esta flexibilidad permite:

- Hacer corresponder el modelo de datos **directamente con los objetos del código**.
- **Iterar** el modelo a medida que evoluciona la aplicación.
- Añadir **validación** solo en las partes del documento que requieran un control más estricto.

### Límites técnicos que condicionan el diseño

| Límite | Valor |
|---|---|
| Tamaño máximo de un documento BSON | **16 MB** (para ficheros mayores: **GridFS**) |
| Niveles máximos de anidamiento | **100** |
| Memoria por etapa de agregación | **100 MB** (con posibilidad de volcar a disco) |
| Memoria por operación de consulta (nuevo en 9.0) | 1 GB o el 20 % de la memoria del proceso, lo que sea mayor |
| Espacio mínimo por índice | ~8 kB |

---

## 2. Principio fundamental

> **Los datos que se consultan juntos deben almacenarse juntos.**

El esquema se diseña a partir de los **patrones de acceso** de la aplicación (qué consultas se hacen, con qué frecuencia y qué datos devuelven), no a partir de las entidades en abstracto como en la normalización relacional.

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

### Paso 2. Mapear las relaciones

- Identificar las entidades y sus relaciones (1:1, 1:N, N:M).
- Decidir para cada relación si se **embebe** o se **referencia** (ver sección 6).
- Estimar la **cardinalidad**: ¿uno a pocos, a muchos o a muchísimos?

### Paso 3. Aplicar patrones de diseño

- Usar patrones (sección 11) para optimizar lecturas y escrituras.

### Paso 4. Crear índices

- Crear índices que den soporte a las consultas más comunes (sección 9).

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

### 5.3. Uno a muchos (referencias en el lado "muchos")

```javascript
// colección editoriales
{ _id: "anaya", nombre: "Anaya", fundada: 1959, pais: "España" }

// colección libros
{ _id: 123, titulo: "Aprende MongoDB", editorial_id: "anaya" }
{ _id: 456, titulo: "Bases de datos NoSQL", editorial_id: "anaya" }
```

> Guardar la referencia en el lado "muchos" evita que el documento de la editorial contenga un **array que crece sin límite**.

### 5.4. Muchos a muchos

```javascript
// estudiantes
{ _id: "e1", nombre: "Marta", asignaturas: ["bd", "redes"] }

// asignaturas
{ _id: "bd", nombre: "Bases de Datos", creditos: 6 }
```

Alternativas: arrays de referencias en uno o ambos lados, o una **colección intermedia** (`matriculas`) si la relación tiene atributos propios (nota, fecha) o es muy numerosa.

### 5.5. Unir datos referenciados con `$lookup`

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

---

## 6. Embeber o referenciar: cómo decidir

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

Incrustar datos relacionados puede implicar **duplicarlos** en varias colecciones. Antes de hacerlo, hay que valorar:

- **El beneficio en lectura:** evitar uniones entre colecciones.
- **La frecuencia de actualización** del dato duplicado: si cambia poco, mantenerlo es barato; si cambia a menudo, puede ser mejor referenciar.

### Tipos de datos duplicados

| Tipo | Descripción | Ejemplo |
|---|---|---|
| **Inmutable** | Nunca cambia. Candidato ideal a duplicar. | Fecha de alta de un usuario. |
| **Temporal** | Cambia, pero interesa conservar el valor histórico. | Dirección de envío en el momento del pedido. |
| **Sensible a la obsolescencia** | Debe estar siempre al día en todas sus copias. | Stock de un producto. |
| **Tolerante a la obsolescencia** | Puede estar desfasado un tiempo. | Lista de productos recomendados. |

### Cómo mantener la coherencia

| Método | Cuándo usarlo |
|---|---|
| **Embeber los datos relacionados** | Coherencia inmediata y atómica en un solo documento. |
| **Transacciones** | Actualizar varias copias de forma atómica (más coste de rendimiento). |
| **Atlas Database Triggers / Change Streams** | Propagar cambios de forma asíncrona (coherencia eventual). |
| **Tareas en segundo plano** | Datos tolerantes a la obsolescencia; recálculo periódico. |

---

## 8. Validación de esquemas

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
        notaMedia: { bsonType: "double", minimum: 0, maximum: 10 },
        curso:     { enum: ["1º", "2º", "3º", "4º"] }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
})
```

### Niveles y acciones de validación

| Opción | Valor | Efecto |
|---|---|---|
| `validationLevel` | `strict` (por defecto) | Valida todas las inserciones y actualizaciones. |
| | `moderate` | Solo valida actualizaciones de documentos que ya eran válidos. |
| | **`constraint`** (nuevo en 9.0) | Además de validar inserciones y actualizaciones, **garantiza que todos los documentos de la colección cumplen las reglas**. |
| | `off` | Desactiva la validación. |
| `validationAction` | `error` | Rechaza el documento no válido. |
| | `warn` | Lo acepta y registra un aviso. |

Ejemplos de uso:

- Asegurar que `fechaInicio` de un evento sea una **fecha** y no una cadena.
- Restringir `tarjetasAceptadas` a `["Visa", "MasterCard", "American Express"]`.
- Garantizar que `notaMedia` sea un número positivo.

> También existe una guía específica para **validar colecciones polimórficas** (documentos de varios tipos en la misma colección) usando `oneOf`/`anyOf` en `$jsonSchema`.

---

## 9. Índices y modelado

Si un campo se usa con frecuencia para **filtrar, ordenar o unir**, conviene indexarlo. Los índices permiten:

- Devolver resultados más rápido.
- Ordenar de forma más eficiente.
- Optimizar `$lookup` y `$group`.
- Reducir el uso de CPU y E/S.

Pero tienen coste:

- Cada índice ocupa **al menos 8 kB** y consume disco y memoria.
- Cada escritura debe **actualizar todos los índices** de la colección: en colecciones con muchas escrituras, los índices son caros.
- En colecciones con muchas lecturas, los índices adicionales suelen compensar.
- Hay que vigilar que el **conjunto de trabajo** (*working set*: datos e índices de uso frecuente) quepa en RAM.

**Tipos de índices disponibles:** simple, compuesto, multiclave (arrays), texto, geoespacial (`2dsphere`), *hashed*, comodín (*wildcard*), parcial, *sparse*, único, TTL, agrupado (*clustered*). Además, en Atlas: índices de **Search** y **Vector Search**.

```javascript
// Supervisar el uso real de los índices
db.pedidos.aggregate([{ $indexStats: {} }])
```

---

## 10. Otras consideraciones

### 10.1. Atomicidad

- En MongoDB, una escritura es **atómica a nivel de documento**, aunque modifique varios subdocumentos.
- Un modelo **desnormalizado** (subdocumentos y arrays) permite operaciones atómicas sin transacciones.
- Con referencias, la aplicación necesita **varias operaciones** de lectura y escritura.
- Para atomicidad entre varios documentos o colecciones existen **transacciones distribuidas**.

> **Importante:** las transacciones tienen un coste de rendimiento mayor que las escrituras de un solo documento y **no sustituyen a un buen diseño del esquema**. En la mayoría de los casos, modelar bien los datos minimiza la necesidad de transacciones. Recuerda además los límites de tiempo de ejecución y de tamaño del oplog, y que en 9.0 hay un máximo de 10 000 transacciones simultáneas por defecto.

### 10.2. Gestión del ciclo de vida de los datos

- **Índices TTL:** borrado automático tras un tiempo (p. ej., sesiones que caducan a los 30 minutos de inactividad).
- **Colecciones de tamaño fijo (*capped*):** comportamiento FIFO, útiles cuando solo interesan los documentos más recientes.
- **Patrón de archivo:** mover datos antiguos a otra colección o a almacenamiento más barato (Atlas Online Archive).

### 10.3. Restricciones de hardware

- Los documentos grandes consumen más RAM y pueden obligar a leer de disco.
- Diseñar para que las consultas devuelvan **solo los campos necesarios** (proyecciones) y que el *working set* no crezca innecesariamente.

### 10.4. Documentos pequeños

Cuando hay muchísimos documentos diminutos, la sobrecarga por documento se vuelve significativa:

- **Usar `_id` con un valor propio y significativo** en lugar de un `ObjectId` de 12 bytes que no aporta información.
- **Acortar nombres de campo** (MongoDB los guarda en cada documento). Ojo: reduce la legibilidad y no afecta al tamaño de los índices; suele ser más eficaz replantear el modelo.
- **Agrupar en documentos embebidos** (patrón *bucket*).

```javascript
// Antes:  { last_name: "Smith", best_score: 3.9 }
// Después: { lname: "Smith", score: 3.9 }   // ahorra 9 bytes por documento
```

### 10.5. Fragmentación (*sharding*)

Si la colección se va a fragmentar, el modelo debe contemplar una **clave de fragmentación** con alta cardinalidad, buena distribución de escrituras y alineada con las consultas más frecuentes (para evitar consultas *scatter-gather*).

---

## 11. Patrones de diseño de esquemas

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

Precalcular resultados que se leen muy a menudo.

```javascript
// En cada nueva reseña se actualizan los totales del producto
db.productos.updateOne(
  { _id: 42 },
  { $inc: { "valoraciones.total": 1, "valoraciones.suma": 5 } }
)
```

**Aproximación (*Approximation*):** actualizar un contador solo cada N eventos (p. ej., visitas) cuando no se necesita exactitud absoluta.

### 11.2. Agrupación (*Bucket*)

Agrupar muchas mediciones pequeñas en un documento por intervalo.

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

> Para este caso, las **colecciones de series temporales** nativas aplican el *bucketing* automáticamente (en 9.0 se almacenan como un único espacio de nombres comprimido).

### 11.3. Valores atípicos (*Outlier*)

Cuando unos pocos documentos tienen arrays enormes (un libro superventas con millones de compradores), se marca el documento (`tieneDesbordamiento: true`) y el exceso se guarda en documentos adicionales.

### 11.4. Subconjunto (*Subset*)

Embeber solo la parte más usada (las 5 últimas reseñas) y dejar el resto en otra colección.

### 11.5. Referencia extendida (*Extended Reference*)

Copiar en el documento que referencia los **pocos campos** que siempre se muestran (p. ej., nombre y dirección del cliente dentro del pedido), evitando `$lookup` constantes.

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

### 11.7. Versionado

- **Versionado de esquema:** campo `versionEsquema` para que la aplicación convierta los documentos antiguos de forma progresiva, sin migraciones masivas ni paradas.
- **Versionado de documentos:** guardar revisiones anteriores en una colección de histórico (útil en auditoría, normativa, documentos legales).

```javascript
{ _id: 1, versionEsquema: 2, nombre: "Ana", contacto: { email: "...", tel: "..." } }
```

### 11.8. Archivo (*Archive*)

Mover documentos antiguos (pedidos de hace más de 5 años) a otra colección, base de datos o almacenamiento de bajo coste, usando por ejemplo `$merge` o Atlas Online Archive.

### 11.9. Colección única (*Single Collection*)

Guardar entidades relacionadas de distintos tipos en una sola colección, con un campo que las distingue y referencias comunes, para obtenerlas con **una sola consulta** indexada.

```javascript
{ _id: "pedido#1001",        tipoDoc: "pedido", pedidoId: 1001, cliente: "c7", total: 120 }
{ _id: "pedido#1001#linea1", tipoDoc: "linea",  pedidoId: 1001, producto: "Ratón", cantidad: 2 }
{ _id: "pedido#1001#envio",  tipoDoc: "envio",  pedidoId: 1001, estado: "en reparto" }

db.ventas.find({ pedidoId: 1001 })   // trae el pedido completo
```

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

> **Antipatrón clásico adicional:** consultas **insensibles a mayúsculas** con expresiones regulares (`/^ana$/i`) que no aprovechan los índices. Solución: crear un índice con **intercalación** (*collation*) de fuerza 1 o 2 y consultar con la misma intercalación.

```javascript
db.usuarios.createIndex({ nombre: 1 }, { collation: { locale: "es", strength: 2 } })
db.usuarios.find({ nombre: "ana" }).collation({ locale: "es", strength: 2 })
```

### Herramientas para detectarlos

- **Performance Advisor de Atlas** (clústeres M10 o superiores): tarjeta *Improve Schema* con recomendaciones.
- **Performance Insights de MongoDB Compass**.
- **`$queryStats`** (en 9.0, activado por defecto con muestreo del 1 %).

---

## 13. Modelado de estructuras de árbol

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

**Índices:**

```javascript
db.productos.createIndex({ sku: 1 }, { unique: true })
db.productos.createIndex({ "atributos.k": 1, "atributos.v": 1 })
db.resenas.createIndex({ productoId: 1, fecha: -1 })
db.pedidos.createIndex({ clienteId: 1, fecha: -1 })
db.clientes.createIndex({ email: 1 }, { unique: true })
```

**Decisiones justificadas:**

- Las **líneas del pedido** se embeben: se leen siempre con el pedido, están acotadas y deben guardarse de forma atómica.
- El **precio en la línea** se copia (dato temporal): si el producto cambia de precio, el pedido conserva el original.
- Las **reseñas** se referencian: crecen sin límite. Solo las 5 últimas se embeben en el producto.
- Para dinero se usa **`Decimal128`** (`NumberDecimal`), no `double`, para evitar errores de redondeo.

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
