# Jackson y JSON

## ¿Qué es JSON?

**JSON** (*JavaScript Object Notation*) es un formato de texto para representar datos. Es el formato más utilizado en las APIs REST porque es ligero, fácil de leer para las personas y fácil de procesar para los programas.

```json
{
  "id": 42,
  "nombre": "Teclado mecánico",
  "precio": 59.90,
  "disponible": true,
  "etiquetas": ["periférico", "gaming"],
  "fabricante": {
    "nombre": "TeclaPro",
    "pais": "ES"
  },
  "descuento": null
}
```

### Tipos de datos en JSON

| Tipo JSON | Ejemplo | Equivalente habitual en Java |
|---|---|---|
| Objeto | `{"clave": valor}` | Una clase, un `record` o un `Map` |
| Array | `[1, 2, 3]` | `List`, `Set` o array |
| Cadena | `"texto"` | `String` (también fechas, `enum`…) |
| Número | `42`, `59.90` | `int`, `long`, `double`, `BigDecimal`… |
| Booleano | `true`, `false` | `boolean` |
| Nulo | `null` | `null` |

### Reglas de sintaxis

- Las claves van **siempre entre comillas dobles**: `"nombre"`, nunca `nombre` ni `'nombre'`.
- Las cadenas también van entre comillas dobles.
- **No se permiten comentarios.**
- **No se permite una coma después del último elemento**: `[1, 2, 3,]` es inválido.

## Serializar y deserializar

- **Serializar:** convertir un objeto Java en texto JSON.
- **Deserializar:** convertir texto JSON en un objeto Java.

```
                     serializar
   Objeto Java  ───────────────────▶  JSON
   (Producto)   ◀───────────────────  (texto)
                    deserializar
```

## ¿Qué es Jackson?

**Jackson** es la librería de Java más utilizada para trabajar con JSON. Spring Boot la usa por defecto: cuando un controlador devuelve un objeto, Jackson lo serializa; cuando recibe un cuerpo JSON, Jackson lo deserializa.

### Jackson 3

Spring Boot 4 usa **Jackson 3**, que tiene diferencias importantes con Jackson 2 (que verás en muchos tutoriales antiguos):

| Aspecto | Jackson 2 | Jackson 3 |
|---|---|---|
| Paquete Java | `com.fasterxml.jackson…` | `tools.jackson…` |
| *Group id* de Maven | `com.fasterxml.jackson.core` | `tools.jackson.core` |
| Paquete de las anotaciones (`@JsonProperty`, `@JsonIgnore`…) | `com.fasterxml.jackson.annotation` | **El mismo**: `com.fasterxml.jackson.annotation` |
| Clase principal recomendada para JSON | `ObjectMapper` (modificable) | `JsonMapper` (inmutable, se configura con un *builder*) |
| Excepciones | Comprobadas (`JsonProcessingException`, hereda de `IOException`) | No comprobadas (`JacksonException`, hereda de `RuntimeException`) |
| Fechas de `java.time` | Requería registrar un módulo aparte | Soporte incluido |
| Formato por defecto de las fechas | Números (*timestamps*) | Texto ISO 8601 (`"2026-11-22T10:30:00"`) |
| Propiedades desconocidas al deserializar | Error por defecto | Se ignoran por defecto |
| Orden de los campos al serializar | Sin orden garantizado | Alfabético por defecto (en los `record`, el orden de declaración) |
| Java mínimo | 8 | 17 |

> **Cuidado con los tutoriales antiguos.** Si copias un `import com.fasterxml.jackson.databind.ObjectMapper` en un proyecto Spring Boot 4, estarás usando Jackson 2 (o no compilará). En Jackson 3 es `tools.jackson.databind…`. **Las anotaciones son la excepción**: siguen en `com.fasterxml.jackson.annotation`.

## Preparar el entorno para los ejemplos

Dentro de un proyecto Spring Boot con `spring-boot-starter-webmvc`, Jackson ya está incluido. Si quieres usarlo en un proyecto Maven sin Spring, añade:

```xml
<dependency>
    <groupId>tools.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>VERSION</version> <!-- consulta la última 3.x en Maven Central -->
</dependency>
```

Para que los ejemplos sean fáciles de ejecutar, están escritos como **clases normales con `main`**. Puedes ponerlos en la carpeta `src/test/java` de tu proyecto Spring Boot (donde Jackson ya está disponible) y ejecutarlos desde el IDE.

## `JsonMapper`: la clase principal

`JsonMapper` es el objeto que convierte entre Java y JSON. Es **inmutable** y **seguro entre hilos**: créalo una vez y reutilízalo.

```java
import tools.jackson.databind.SerializationFeature;
import tools.jackson.databind.json.JsonMapper;

// Con la configuración por defecto
JsonMapper mapper = new JsonMapper();

// Con configuración personalizada (mediante el builder)
JsonMapper mapperBonito = JsonMapper.builder()
        .enable(SerializationFeature.INDENT_OUTPUT)   // JSON con saltos de línea y sangría
        .build();
```

> En una aplicación Spring Boot **no crees tu propio `JsonMapper`**: Spring Boot ya registra uno como *bean*, configurado de forma coherente con el resto de la aplicación. Inyéctalo donde lo necesites (ver el apartado «Jackson dentro de Spring Boot»).

## Serializar: de objeto Java a JSON

```java
package com.ejemplo.json;

import java.math.BigDecimal;
import java.util.List;

import tools.jackson.databind.SerializationFeature;
import tools.jackson.databind.json.JsonMapper;

public class EjemploSerializar {

    record Fabricante(String nombre, String pais) {}

    record Producto(Long id, String nombre, BigDecimal precio,
                    List<String> etiquetas, Fabricante fabricante) {}

    public static void main(String[] args) {
        JsonMapper mapper = JsonMapper.builder()
                .enable(SerializationFeature.INDENT_OUTPUT)
                .build();

        var producto = new Producto(
                42L,
                "Teclado mecánico",
                new BigDecimal("59.90"),
                List.of("periférico", "gaming"),
                new Fabricante("TeclaPro", "ES"));

        String json = mapper.writeValueAsString(producto);
        IO.println(json);
    }
}
```

Resultado:

```json
{
  "id" : 42,
  "nombre" : "Teclado mecánico",
  "precio" : 59.90,
  "etiquetas" : [ "periférico", "gaming" ],
  "fabricante" : {
    "nombre" : "TeclaPro",
    "pais" : "ES"
  }
}
```

Jackson usa los **componentes del `record`** (o los *getters* de una clase normal) para decidir qué campos escribir y con qué nombre. En un `record` los campos salen en el mismo orden en que se declaran; en una clase normal, Jackson 3 los ordena alfabéticamente. Si necesitas otro orden, usa `@JsonPropertyOrder` (ver el apartado «Anotaciones de Jackson»).

## Deserializar: de JSON a objeto Java

```java
package com.ejemplo.json;

import java.math.BigDecimal;

import tools.jackson.databind.json.JsonMapper;

public class EjemploDeserializar {

    record Producto(Long id, String nombre, BigDecimal precio) {}

    public static void main(String[] args) {
        JsonMapper mapper = new JsonMapper();

        String json = """
                {
                  "id": 42,
                  "nombre": "Teclado mecánico",
                  "precio": 59.90
                }
                """;

        Producto producto = mapper.readValue(json, Producto.class);

        IO.println(producto.nombre());   // Teclado mecánico
        IO.println(producto.precio());   // 59.90
    }
}
```

Para clases normales (no `record`), Jackson necesita normalmente un **constructor sin argumentos** y *setters*, o un constructor anotado. Con los `record` funciona directamente.

### Listas y tipos genéricos

Por cómo funcionan los genéricos en Java, `List<Producto>.class` no existe. Para indicar a Jackson el tipo completo se usa `TypeReference`:

```java
package com.ejemplo.json;

import java.util.List;

import tools.jackson.core.type.TypeReference;
import tools.jackson.databind.json.JsonMapper;

public class EjemploLista {

    record Producto(Long id, String nombre) {}

    public static void main(String[] args) {
        JsonMapper mapper = new JsonMapper();

        String json = """
                [
                  {"id": 1, "nombre": "Ratón"},
                  {"id": 2, "nombre": "Alfombrilla"}
                ]
                """;

        List<Producto> productos = mapper.readValue(json, new TypeReference<List<Producto>>() {});

        for (Producto p : productos) {
            IO.println(p.id() + " - " + p.nombre());
        }
    }
}
```

## El modelo de árbol: `JsonNode`

A veces no tienes (o no quieres crear) una clase para el JSON, por ejemplo cuando solo necesitas un par de campos de una respuesta grande. Jackson puede leer el JSON como un **árbol** de nodos:

```java
package com.ejemplo.json;

import tools.jackson.databind.JsonNode;
import tools.jackson.databind.json.JsonMapper;

public class EjemploArbol {

    public static void main(String[] args) {
        JsonMapper mapper = new JsonMapper();

        String json = """
                {
                  "pedido": 15,
                  "cliente": { "nombre": "Laura", "ciudad": "Valencia" },
                  "lineas": [
                    { "producto": "Ratón", "cantidad": 2 },
                    { "producto": "Teclado", "cantidad": 1 }
                  ]
                }
                """;

        JsonNode raiz = mapper.readTree(json);

        int numPedido = raiz.get("pedido").asInt();
        String ciudad = raiz.get("cliente").get("ciudad").asString();

        IO.println("Pedido " + numPedido + " para " + ciudad);

        for (JsonNode linea : raiz.get("lineas")) {
            IO.println(linea.get("cantidad").asInt() + " x " + linea.get("producto").asString());
        }

        // path() no devuelve null si el campo no existe: devuelve un nodo "vacío"
        IO.println(raiz.path("noExiste").isMissingNode()); // true
    }
}
```

| Método | Qué hace |
|---|---|
| `get("campo")` | Devuelve el nodo hijo, o `null` si no existe. |
| `path("campo")` | Devuelve el nodo hijo, o un nodo "ausente" si no existe (evita `NullPointerException`). |
| `asString()`, `asInt()`, `asBoolean()`… | Convierte el valor del nodo. |
| `isMissingNode()`, `isNull()`, `isArray()`, `isObject()` | Comprueba qué tipo de nodo es. |

> En Jackson 2 el método para obtener el texto era `asText()`. En Jackson 3 se llama `asString()` (`asText()` queda obsoleto).

## Anotaciones de Jackson

Las anotaciones permiten ajustar cómo se convierte cada clase o campo. Recuerda que se importan desde `com.fasterxml.jackson.annotation`.

### `@JsonProperty`: cambiar el nombre de un campo

```java
import com.fasterxml.jackson.annotation.JsonProperty;

record Usuario(
        @JsonProperty("nombre_completo") String nombreCompleto,
        String email) {}
```

```json
{"nombre_completo": "Laura Martín", "email": "laura@ejemplo.com"}
```

### `@JsonIgnore`: no incluir un campo

```java
import com.fasterxml.jackson.annotation.JsonIgnore;

record Usuario(String email, @JsonIgnore String passwordHash) {}
```

`passwordHash` no aparecerá en el JSON generado.

> Aunque `@JsonIgnore` funciona, la forma más segura de no exponer datos sensibles es **no incluirlos en el DTO de respuesta** (ver el documento [Arquitectura por capas (Controller – Service – Repository)](<../15-Arquitectura/Arquitectura por capas.md>)).

### `@JsonInclude`: omitir valores vacíos o nulos

```java
import com.fasterxml.jackson.annotation.JsonInclude;

@JsonInclude(JsonInclude.Include.NON_NULL)
record Producto(Long id, String nombre, String descripcion) {}
```

Si `descripcion` es `null`, no aparecerá en el JSON. Otras opciones: `NON_EMPTY` (omite también cadenas y listas vacías), `ALWAYS` (por defecto).

### `@JsonFormat`: formato de fechas y números

```java
import java.time.LocalDate;

import com.fasterxml.jackson.annotation.JsonFormat;

record Evento(
        String titulo,
        @JsonFormat(pattern = "dd/MM/yyyy") LocalDate fecha) {}
```

```json
{"titulo": "Lanzamiento", "fecha": "22/11/2026"}
```

> Sin `@JsonFormat`, Jackson 3 escribe las fechas en formato **ISO 8601** (`"2026-11-22"`), que es el recomendado para APIs. Usa `@JsonFormat` solo si necesitas otro formato.

### `@JsonPropertyOrder`: orden de los campos

```java
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonPropertyOrder({"id", "nombre", "precio"})
record Producto(String nombre, java.math.BigDecimal precio, Long id) {}
```

### `@JsonAlias`: aceptar varios nombres al leer

```java
import com.fasterxml.jackson.annotation.JsonAlias;

record Cliente(@JsonAlias({"mail", "correo"}) String email) {}
```

Al deserializar acepta `"email"`, `"mail"` o `"correo"`. Al serializar, escribe `"email"`.

### `@JsonIgnoreProperties`: controlar propiedades desconocidas

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
record Producto(Long id, String nombre) {}
```

En Jackson 3 las propiedades desconocidas ya se ignoran por defecto, pero esta anotación deja la intención explícita y es útil si el *mapper* se ha configurado para fallar con ellas.

### `enum`

Los `enum` se serializan como texto:

```java
enum EstadoPedido { PENDIENTE, PAGADO, ENVIADO }

record Pedido(Long id, EstadoPedido estado) {}
```

```json
{"id": 15, "estado": "PAGADO"}
```

### Resumen de anotaciones

| Anotación | Para qué sirve |
|---|---|
| `@JsonProperty` | Cambiar el nombre de un campo en el JSON. |
| `@JsonIgnore` | Excluir un campo. |
| `@JsonInclude` | Omitir campos nulos o vacíos. |
| `@JsonFormat` | Formato de fechas, números… |
| `@JsonPropertyOrder` | Orden de los campos al serializar. |
| `@JsonAlias` | Nombres alternativos al deserializar. |
| `@JsonIgnoreProperties` | Ignorar campos concretos o desconocidos. |

## Fechas y horas

Jackson 3 entiende los tipos de `java.time` sin configuración adicional y los escribe en formato **ISO 8601**:

| Tipo Java | Ejemplo en JSON |
|---|---|
| `LocalDate` | `"2026-11-22"` |
| `LocalTime` | `"10:30:00"` |
| `LocalDateTime` | `"2026-11-22T10:30:00"` |
| `Instant` | `"2026-11-22T09:30:00Z"` |
| `OffsetDateTime` | `"2026-11-22T10:30:00+01:00"` |

> Para APIs que usan personas de distintas zonas horarias, `Instant` u `OffsetDateTime` son más seguros que `LocalDateTime`, porque incluyen la información de zona.

## Errores al procesar JSON

Si el JSON está mal escrito o no encaja con la clase, Jackson lanza una **`JacksonException`** (paquete `tools.jackson.core`). En Jackson 3 es una excepción **no comprobada**, así que no estás obligado a capturarla, pero puedes hacerlo:

```java
package com.ejemplo.json;

import tools.jackson.core.JacksonException;
import tools.jackson.databind.json.JsonMapper;

public class EjemploError {

    record Producto(Long id, String nombre) {}

    public static void main(String[] args) {
        JsonMapper mapper = new JsonMapper();

        String jsonRoto = """
                {"id": 1, "nombre": "Ratón"
                """;  // falta la llave de cierre

        try {
            mapper.readValue(jsonRoto, Producto.class);
        } catch (JacksonException e) {
            IO.println("JSON no válido: " + e.getMessage());
        }
    }
}
```

En una API con Spring Boot no tienes que hacerlo tú: si el cuerpo de una petición no es un JSON válido, Spring responde automáticamente con `400 Bad Request`.

## Jackson dentro de Spring Boot

En una aplicación Spring Boot casi nunca llamarás a Jackson directamente. Spring lo hace por ti:

```java
@RestController
@RequestMapping("/productos")
public class ProductoController {

    // Petición: el JSON del cuerpo se convierte en CrearProductoRequest
    // Respuesta: el ProductoResponse devuelto se convierte en JSON
    @PostMapping
    public ProductoResponse crear(@RequestBody CrearProductoRequest peticion) {
        ...
    }
}
```

- `@RequestBody` indica a Spring que **deserialice** el cuerpo de la petición.
- El objeto devuelto se **serializa** automáticamente (porque la clase es `@RestController`).
- La cabecera `Content-Type: application/json` se pone sola en la respuesta.

### Usar el `JsonMapper` de Spring

Si necesitas convertir JSON manualmente dentro de la aplicación, **inyecta** el `JsonMapper` que Spring Boot ya ha configurado:

```java
@Service
public class AuditoriaService {

    private final JsonMapper mapper;

    public AuditoriaService(JsonMapper mapper) {
        this.mapper = mapper;
    }

    public String comoJson(Object objeto) {
        return mapper.writeValueAsString(objeto);
    }
}
```

### Configurar Jackson desde `application.properties`

```properties
# JSON con sangría (útil en desarrollo)
spring.jackson.serialization.indent-output=true

# No incluir campos nulos en ninguna respuesta
spring.jackson.default-property-inclusion=non_null
```

### Configurar Jackson desde código

Para ajustes más avanzados, Spring Boot 4 ofrece la interfaz `JsonMapperBuilderCustomizer` (paquete `org.springframework.boot.jackson.autoconfigure`). Permite modificar el *builder* del `JsonMapper` autoconfigurado sin sustituirlo:

```java
package com.ejemplo.tienda.config;

import org.springframework.boot.jackson.autoconfigure.JsonMapperBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import tools.jackson.databind.SerializationFeature;

@Configuration
public class JacksonConfig {

    @Bean
    public JsonMapperBuilderCustomizer personalizarJackson() {
        return builder -> builder.enable(SerializationFeature.INDENT_OUTPUT);
    }
}
```

> Si declaras tu propio *bean* de tipo `JsonMapper`, Spring Boot dejará de autoconfigurar el suyo y perderás sus ajustes por defecto. Normalmente es mejor usar un `JsonMapperBuilderCustomizer` o las propiedades `spring.jackson.*`.

## Resumen

- **JSON** es el formato estándar de las APIs REST.
- **Jackson** convierte entre objetos Java y JSON. Spring Boot 4 usa **Jackson 3**.
- En Jackson 3 el paquete es `tools.jackson…`, salvo las **anotaciones**, que siguen en `com.fasterxml.jackson.annotation`.
- `JsonMapper`: `writeValueAsString` para serializar, `readValue` para deserializar, `readTree` para leer como árbol.
- Los `record` funcionan directamente con Jackson y son ideales para DTOs.
- Las fechas de `java.time` se escriben en ISO 8601 sin configuración extra.
- En Spring Boot, `@RequestBody` y el valor devuelto por un `@RestController` se convierten automáticamente. Configura Jackson con `spring.jackson.*` o con `JsonMapperBuilderCustomizer`.

## Referencias

- Baeldung — Intro to the Jackson ObjectMapper: <https://www.baeldung.com/jackson-object-mapper-tutorial> (usa Jackson 2: cambia los paquetes `com.fasterxml.jackson.databind` por `tools.jackson.databind`)
- Baeldung — Spring Boot: Consuming and Producing JSON: <https://www.baeldung.com/spring-boot-json>
- Spring Boot — JSON: <https://docs.spring.io/spring-boot/reference/features/json.html>
- Blog de Spring — Introducing Jackson 3 support in Spring: <https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring/>
- Guía oficial de migración a Jackson 3: <https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md>
- Especificación de JSON: <https://www.json.org/json-es.html>
