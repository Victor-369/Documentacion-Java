# Persistencia NoSQL con MongoDB

> 📚 **Documentación relacionada en este repositorio:** este documento se centra en usar MongoDB desde Spring Boot. Para conocer el motor en profundidad, consulta [NoSQL y diferencias con SQL](<../../10 - Bases de datos/NoSQL/Diferencias entre nosql y sql.md>), [MongoDB: modelado de datos y diseño](<../../10 - Bases de datos/NoSQL/MongoDB - modelado de datos y diseno.md>) y [MongoDB: tutoriales](<../../10 - Bases de datos/NoSQL/MongoDB - tutoriales.md>) (uso de `mongosh`, CRUD, índices y agregación).

## ¿Qué es NoSQL?

**NoSQL** agrupa a las bases de datos que **no** siguen el modelo de tablas relacionales. Hay varios tipos:

| Tipo | Cómo guarda los datos | Ejemplos |
|---|---|---|
| Documentales | Documentos tipo JSON | MongoDB, Couchbase |
| Clave-valor | Pares clave → valor | Redis |
| Columnares | Familias de columnas | Cassandra |
| Grafos | Nodos y relaciones | Neo4j |

## ¿Qué es MongoDB?

**MongoDB** es una base de datos **documental**. En lugar de filas en tablas, guarda **documentos** con una estructura muy parecida a JSON (internamente usa **BSON**, una versión binaria de JSON con más tipos de datos, como fechas y decimales).

| Relacional | MongoDB |
|---|---|
| Base de datos | Base de datos |
| Tabla | **Colección** |
| Fila | **Documento** |
| Columna | **Campo** |
| Clave primaria | Campo `_id` |
| `JOIN` | Documentos embebidos o referencias (con `$lookup` en agregaciones) |

Un pedido en MongoDB puede guardarse como **un único documento**, con sus líneas dentro:

```json
{
  "_id": { "$oid": "6740a1f2c3e4b5a6d7e8f901" },
  "cliente": { "nombre": "Laura Martín", "email": "laura@ejemplo.com" },
  "estado": "PAGADO",
  "lineas": [
    { "producto": "Teclado", "cantidad": 1, "precioUnitario": 59.90 },
    { "producto": "Ratón",   "cantidad": 2, "precioUnitario": 24.50 }
  ],
  "creadoEn": { "$date": "2026-11-22T10:30:00Z" }
}
```

### Características principales

- **Esquema flexible:** los documentos de una colección no tienen por qué tener exactamente los mismos campos. (Aun así, conviene mantener una estructura coherente, y MongoDB permite definir reglas de validación.)
- **Datos anidados:** un documento puede contener objetos y listas, lo que evita muchos `JOIN`.
- **Escalado horizontal:** está diseñado para repartir los datos entre varios servidores (*sharding*).
- **Límite de tamaño:** un documento no puede superar los **16 MB**.

### ¿Cuándo elegir MongoDB y cuándo una base de datos relacional?

| Encaja mejor MongoDB | Encaja mejor una relacional |
|---|---|
| Los datos se leen y escriben "en bloque" (un pedido con sus líneas). | Hay muchas relaciones entre entidades que se consultan de formas variadas. |
| La estructura cambia a menudo o varía entre elementos (catálogos con atributos distintos por producto). | Se necesitan transacciones complejas entre muchas tablas y restricciones estrictas. |
| Grandes volúmenes con necesidad de escalar horizontalmente. | Informes con agregaciones y cruces complejos (aunque MongoDB tiene un potente *aggregation framework*). |

> MongoDB **sí** admite transacciones con varios documentos, pero su modelo invita a diseñar documentos que se puedan modificar de forma atómica por sí solos.

## Levantar MongoDB con Docker

`compose.yaml`:

```yaml
services:
  mongodb:
    image: mongo:8
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_DATABASE: tienda
```

```bash
docker compose up -d
```

> Para explorar los datos puedes usar **MongoDB Compass** (aplicación gráfica oficial) o `mongosh` (consola).

## Dependencias y configuración

En Spring Initializr añade **Spring Data MongoDB**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-test</artifactId>
    <scope>test</scope>
</dependency>
```

`application.properties`:

```properties
spring.mongodb.uri=mongodb://localhost:27017/tienda
```

Si no configuras nada, Spring Boot intenta conectarse a `mongodb://localhost/test`.

> **Spring Boot 4:** las propiedades de conexión de MongoDB pasaron de `spring.data.mongodb.*` a `spring.mongodb.*`. En tutoriales antiguos verás `spring.data.mongodb.uri`.

Igual que con PostgreSQL, si añades `spring-boot-docker-compose`, Spring Boot arranca el `compose.yaml` y configura la conexión automáticamente.

## Documentos

Un **documento** se representa con una clase anotada con `@Document`. A diferencia de JPA, aquí **sí puedes usar `record`**: Spring Data MongoDB construye el objeto a través de su constructor.

```java
package com.ejemplo.tienda.pedido;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

@Document(collection = "pedidos")
public record Pedido(
        @Id String id,
        Cliente cliente,
        EstadoPedido estado,
        List<LineaPedido> lineas,
        @Field("creado_en") Instant creadoEn
) {

    public record Cliente(String nombre, String email) {}

    public record LineaPedido(String producto, int cantidad, BigDecimal precioUnitario) {

        public BigDecimal subtotal() {
            return precioUnitario.multiply(BigDecimal.valueOf(cantidad));
        }
    }

    public enum EstadoPedido { PENDIENTE, PAGADO, ENVIADO, CANCELADO }

    public BigDecimal total() {
        return lineas.stream()
                .map(LineaPedido::subtotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    /** Devuelve una copia del pedido con otro estado (el record es inmutable). */
    public Pedido conEstado(EstadoPedido nuevoEstado) {
        return new Pedido(id, cliente, nuevoEstado, lineas, creadoEn);
    }
}
```

### Anotaciones principales

| Anotación | Paquete | Qué hace |
|---|---|---|
| `@Document(collection = "…")` | `org.springframework.data.mongodb.core.mapping` | Marca la clase como documento y define la colección. |
| `@Id` | `org.springframework.data.annotation` | Campo que se guarda como `_id`. Si es `String` y vale `null`, MongoDB genera un `ObjectId`. |
| `@Field("…")` | `org.springframework.data.mongodb.core.mapping` | Nombre del campo en la base de datos. |
| `@Indexed` | `org.springframework.data.mongodb.core.index` | Declara un índice sobre el campo. |
| `@Transient` | `org.springframework.data.annotation` | Campo que no se guarda. |
| `@DocumentReference` / `@DBRef` | `org.springframework.data.mongodb.core.mapping` | Referencia a otro documento (ver «Referencias con Spring Data MongoDB»). |

> **Cuidado con `@Id`:** usa la de `org.springframework.data.annotation`, **no** la de `jakarta.persistence` (esa es de JPA).
>
> **Índices:** por defecto, Spring Data MongoDB **no crea automáticamente** los índices declarados con `@Indexed`. Lo recomendable es crearlos de forma explícita (por ejemplo, con scripts o con `MongoTemplate`) o activar la creación automática en la configuración.

## Repositorios

Funcionan exactamente igual que en JPA, gracias a Spring Data:

```java
package com.ejemplo.tienda.pedido;

import java.time.Instant;
import java.util.List;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

import com.ejemplo.tienda.pedido.Pedido.EstadoPedido;

public interface PedidoRepository extends MongoRepository<Pedido, String> {

    // Consultas derivadas: igual que en JPA
    List<Pedido> findByEstado(EstadoPedido estado);

    // Propiedad de un subdocumento: cliente.email
    List<Pedido> findByClienteEmail(String email);

    Page<Pedido> findByCreadoEnAfter(Instant desde, Pageable paginacion);

    // Consulta escrita a mano con la sintaxis de MongoDB (JSON)
    @Query("{ 'lineas.producto': ?0 }")
    List<Pedido> buscarQueContienenProducto(String nombreProducto);
}
```

- `?0` es el primer parámetro del método, `?1` el segundo, etc.
- `findByClienteEmail` busca dentro del subdocumento `cliente`. Spring Data genera `{ "cliente.email": … }`.

## `MongoTemplate`: control total

Para operaciones que los repositorios no cubren bien (actualizaciones parciales, consultas dinámicas, agregaciones), Spring ofrece `MongoTemplate`, que se inyecta como cualquier *bean*:

```java
package com.ejemplo.tienda.pedido;

import static org.springframework.data.mongodb.core.query.Criteria.where;

import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import com.ejemplo.tienda.pedido.Pedido.EstadoPedido;

@Service
public class PedidoEstadoService {

    private final MongoTemplate mongo;

    public PedidoEstadoService(MongoTemplate mongo) {
        this.mongo = mongo;
    }

    /** Cambia solo el campo "estado", sin leer ni reescribir el documento completo. */
    public boolean marcarComoEnviado(String idPedido) {
        var consulta = new Query(where("_id").is(idPedido).and("estado").is(EstadoPedido.PAGADO));
        var cambio = new Update().set("estado", EstadoPedido.ENVIADO);

        var resultado = mongo.updateFirst(consulta, cambio, Pedido.class);
        return resultado.getModifiedCount() == 1;
    }
}
```

La condición `estado = PAGADO` dentro de la propia actualización garantiza que solo se envían pedidos pagados, **de forma atómica**: no hay riesgo de que otro proceso cambie el estado entre la lectura y la escritura.

### Agregaciones

El *aggregation framework* de MongoDB procesa documentos en una cadena de etapas (filtrar, agrupar, ordenar…). Ejemplo: número de líneas de pedido por estado:

```java
import static org.springframework.data.mongodb.core.aggregation.Aggregation.group;
import static org.springframework.data.mongodb.core.aggregation.Aggregation.newAggregation;
import static org.springframework.data.mongodb.core.aggregation.Aggregation.sort;
import static org.springframework.data.mongodb.core.aggregation.Aggregation.unwind;

import org.springframework.data.domain.Sort;

// "id" recibe el valor de _id, que tras el group es el estado
public record VentasPorEstado(String id, long numeroLineas) {}

// ...
var agregacion = newAggregation(
        unwind("lineas"),                              // una salida por cada línea
        group("estado").count().as("numeroLineas"),    // agrupar por estado y contar
        sort(Sort.Direction.DESC, "numeroLineas"));

List<VentasPorEstado> resultado = mongo
        .aggregate(agregacion, "pedidos", VentasPorEstado.class)
        .getMappedResults();
```

## Modelado de datos en MongoDB

> 💡 Aquí se resume lo imprescindible. Los patrones de diseño de esquemas, los antipatrones y un ejemplo completo están en [MongoDB: modelado de datos y diseño](<../../10 - Bases de datos/NoSQL/MongoDB - modelado de datos y diseno.md>).

En una base de datos relacional se **normaliza**: cada dato en un único sitio y relaciones mediante claves. En MongoDB la pregunta clave es otra: **¿cómo va a usar la aplicación estos datos?** La propia documentación de MongoDB recomienda diseñar el esquema pensando en los patrones de acceso de la aplicación.

Hay dos herramientas: **embeber** y **referenciar**.

### Embeber (documentos anidados)

Los datos relacionados se guardan **dentro** del mismo documento:

```json
{
  "_id": "p-15",
  "cliente": { "nombre": "Laura", "email": "laura@ejemplo.com" },
  "lineas": [ { "producto": "Teclado", "cantidad": 1 } ]
}
```

Es la opción adecuada cuando:

- Los datos **tienen sentido juntos** (un pedido y sus líneas).
- Se **leen juntos** casi siempre, y la lectura es más frecuente que la modificación.
- **No necesitas consultar los subdocumentos por separado** (nadie pide "la línea 3" sin su pedido).
- El número de elementos anidados está **acotado** (no crece sin límite).

Ventajas: una sola lectura trae todo; las actualizaciones de un documento son **atómicas**.

### Referenciar

Se guarda el **identificador** de otro documento, como una clave foránea:

```json
// colección "pedidos"
{ "_id": "p-15", "clienteId": "c-7", "lineas": [ ... ] }

// colección "clientes"
{ "_id": "c-7", "nombre": "Laura", "email": "laura@ejemplo.com", "direcciones": [ ... ] }
```

Es la opción adecuada cuando:

- El dato referenciado tiene **vida propia** y se consulta o modifica por separado (un cliente).
- Se **comparte** entre muchos documentos y duplicarlo sería costoso de mantener.
- La lista crecería **sin límite** (todos los comentarios de un producto muy popular).

### Duplicar a propósito

En MongoDB es habitual **copiar** algunos datos en lugar de referenciarlos, cuando lo que importa es el valor **en un momento concreto**. Por ejemplo, el precio y el nombre del producto en una línea de pedido: si mañana cambia el precio del catálogo, el pedido de ayer debe conservar el precio con el que se compró.

### Resumen de decisión

| Pregunta | Si la respuesta es "sí"… |
|---|---|
| ¿Se leen siempre juntos? | Embeber |
| ¿El subdato tiene sentido por sí solo y se consulta aislado? | Referenciar |
| ¿La lista puede crecer sin límite? | Referenciar |
| ¿Se comparte entre muchos documentos y cambia a menudo? | Referenciar |
| ¿Necesito el valor "congelado" de un momento? | Embeber una copia |

## Referencias con Spring Data MongoDB

La forma más simple y explícita es guardar el id a mano:

```java
@Document(collection = "pedidos")
public record PedidoConReferencia(@Id String id, String clienteId, List<LineaPedido> lineas) {}
```

Y cargar el cliente cuando haga falta con su propio repositorio. Así siempre sabes cuántas consultas se hacen.

Spring Data MongoDB también ofrece `@DocumentReference` (y la más antigua `@DBRef`) para que la referencia se resuelva automáticamente al leer. Son cómodas, pero cada referencia resuelta implica consultas adicionales, así que úsalas con conocimiento.

## Transacciones en MongoDB

MongoDB admite transacciones de varios documentos, pero **solo en un *replica set* o un clúster** (no en un servidor independiente). En Spring se usan con `@Transactional` después de registrar un `MongoTransactionManager`. Si diseñas bien los documentos (embebiendo lo que cambia junto), muchas veces no las necesitarás.

## Resumen

- **MongoDB** guarda **documentos** tipo JSON en **colecciones**.
- Con Spring Data MongoDB, los documentos son clases (o `record`) con `@Document`, y los repositorios funcionan igual que en JPA.
- `MongoTemplate` permite actualizaciones parciales, consultas dinámicas y agregaciones.
- Modela según **cómo se usan los datos**: **embebe** lo que se lee junto y está acotado; **referencia** lo que tiene vida propia, se comparte o crece sin límite.
- En Spring Boot 4 la conexión se configura con `spring.mongodb.uri`.

## Referencias

- Spring Data MongoDB — Reference Documentation: <https://docs.spring.io/spring-data/mongodb/reference/>
- Spring — Accessing Data with MongoDB: <https://spring.io/guides/gs/accessing-data-mongodb/>
- MongoDB — Data Modeling: <https://www.mongodb.com/docs/manual/data-modeling/>
- Spring Boot — Working with NoSQL Technologies (MongoDB): <https://docs.spring.io/spring-boot/reference/data/nosql.html>
