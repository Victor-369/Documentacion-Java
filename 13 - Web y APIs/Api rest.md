# API REST

## ¿Qué es una API?

Una **API** (*Application Programming Interface*, interfaz de programación de aplicaciones) es un contrato que define cómo un programa puede pedirle cosas a otro. Cuando ese contrato se ofrece por HTTP, hablamos de una **API web**.

Ejemplo: la app de una tienda pide a un servidor la lista de productos. La app no sabe cómo están guardados (base de datos, ficheros…); solo sabe que si hace `GET /productos` recibirá una lista en JSON. Eso es la API.

## ¿Qué es REST?

**REST** (*REpresentational State Transfer*) es un **estilo de arquitectura** para diseñar APIs web. Lo definió Roy Fielding en su tesis doctoral en el año 2000. No es un protocolo ni una librería: es un conjunto de principios. Una API que los sigue se llama **API REST** o **API RESTful**.

### Los seis principios (restricciones) de REST

1. **Cliente-servidor.** Cliente y servidor están separados y evolucionan de forma independiente. El cliente no sabe cómo se guardan los datos; el servidor no sabe cómo se muestran.
2. **Sin estado** (*stateless*). Cada petición lleva toda la información necesaria para entenderla. El servidor no guarda "la sesión" del cliente entre peticiones.
3. **Cacheable.** Las respuestas deben indicar si se pueden guardar en caché o no, para no repetir peticiones innecesarias.
4. **Interfaz uniforme.** Todos los recursos se manejan de la misma manera:
   - Cada recurso se identifica con una URI (`/productos/42`).
   - Se manipulan mediante **representaciones** (por ejemplo, un JSON).
   - Los mensajes son autodescriptivos (método, `Content-Type`, código de estado…).
   - Las respuestas pueden incluir enlaces a acciones relacionadas (**HATEOAS**, se explica más abajo).
5. **Sistema en capas.** Entre cliente y servidor puede haber intermediarios (*proxies*, balanceadores, cachés) sin que el cliente lo note.
6. **Código bajo demanda** (opcional). El servidor puede enviar código ejecutable al cliente (por ejemplo, JavaScript). Es la única restricción opcional.

## Recursos y representaciones

En REST todo gira en torno a los **recursos**: cualquier "cosa" con identidad propia que la API expone. Un producto, un usuario, un pedido, una factura.

- **Recurso:** el concepto (el producto 42).
- **URI:** su dirección (`/productos/42`).
- **Representación:** la forma en que se envía (normalmente JSON):

```json
{
  "id": 42,
  "nombre": "Teclado",
  "precio": 29.99
}
```

Un mismo recurso puede tener varias representaciones (JSON, XML…). El cliente elige con la cabecera `Accept`.

## Diseño de URIs

Las URIs son la cara visible de tu API. Unas buenas URIs se entienden sin documentación.

### Reglas básicas

| Regla | Bien | Mal |
|---|---|---|
| Usa **sustantivos**, no verbos. La acción la indica el método HTTP. | `GET /productos` | `GET /obtenerProductos` |
| Usa el **plural** para colecciones. | `/productos/42` | `/producto/42` |
| Usa **minúsculas** y guiones para separar palabras. | `/lineas-pedido` | `/LineasPedido`, `/lineas_pedido` |
| Expresa **jerarquías** con `/`. | `/clientes/7/pedidos` | `/pedidosDeCliente?id=7` |
| No pongas la extensión del formato. | `/productos/42` + `Accept` | `/productos/42.json` |
| No termines en `/`. | `/productos` | `/productos/` |
| Usa la consulta (`?`) para **filtrar, ordenar y paginar**. | `/productos?categoria=perifericos&orden=precio` | `/productos/perifericos/ordenados-por-precio` |

### Métodos sobre una colección y un elemento

Con solo dos URIs y los métodos HTTP se cubren todas las operaciones básicas (**CRUD**: *Create, Read, Update, Delete*):

| Operación | Método | URI | Respuesta típica |
|---|---|---|---|
| Listar productos | `GET` | `/productos` | `200 OK` + lista |
| Obtener un producto | `GET` | `/productos/42` | `200 OK` o `404 Not Found` |
| Crear un producto | `POST` | `/productos` | `201 Created` + `Location: /productos/42` |
| Reemplazar un producto | `PUT` | `/productos/42` | `200 OK` (o `204 No Content`) |
| Modificar parte de un producto | `PATCH` | `/productos/42` | `200 OK` |
| Borrar un producto | `DELETE` | `/productos/42` | `204 No Content` |

### Recursos anidados

Cuando un recurso pertenece a otro:

```
GET  /clientes/7/pedidos        → pedidos del cliente 7
GET  /clientes/7/pedidos/15     → pedido 15 del cliente 7
POST /clientes/7/pedidos        → crear un pedido para el cliente 7
```

> **Consejo:** no anides más de uno o dos niveles. `/clientes/7/pedidos/15/lineas/3/producto` es difícil de usar. Si el pedido 15 tiene identidad propia, `/pedidos/15/lineas` es más sencillo.

### Acciones que no encajan en CRUD

A veces hay operaciones que no son "crear, leer, actualizar o borrar", como cancelar un pedido. Hay dos enfoques habituales:

- Tratarlo como un cambio de estado: `PATCH /pedidos/15` con `{"estado": "CANCELADO"}`.
- Tratar la acción como un sub-recurso: `POST /pedidos/15/cancelacion`.

Ambos son válidos; lo importante es ser **coherente** en toda la API.

## Filtrado, ordenación y paginación

Cuando una colección es grande, no se devuelve entera. Se usan parámetros de consulta:

```
GET /productos?categoria=perifericos          → filtrar
GET /productos?orden=precio                   → ordenar ascendente
GET /productos?orden=-precio                  → ordenar descendente (convención común)
GET /productos?pagina=2&tamano=20             → paginar
GET /productos?campos=id,nombre               → elegir campos
```

No hay un estándar único para los nombres de estos parámetros. Elige unos y úsalos igual en toda la API.

Una respuesta paginada suele incluir información sobre la página:

```json
{
  "contenido": [
    { "id": 21, "nombre": "Ratón", "precio": 15.0 },
    { "id": 22, "nombre": "Alfombrilla", "precio": 8.5 }
  ],
  "pagina": 2,
  "tamano": 20,
  "totalElementos": 134,
  "totalPaginas": 7
}
```

## Uso correcto de los códigos de estado

Devolver siempre `200` con un mensaje de error dentro del cuerpo es un error frecuente. El código de estado **es parte de la respuesta** y los clientes lo usan para decidir qué hacer.

```http
GET /productos/999 HTTP/1.1
```

Mal:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"error": "Producto no encontrado"}
```

Bien:

```http
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "No existe el producto con id 999",
  "instance": "/productos/999"
}
```

### Formato estándar de errores: *Problem Details*

El ejemplo anterior sigue el estándar **RFC 9457** (*Problem Details for HTTP APIs*), con el tipo de contenido `application/problem+json`. Sus campos son:

| Campo | Significado |
|---|---|
| `type` | URI que identifica el tipo de problema. `about:blank` si no hay uno específico. |
| `title` | Resumen corto del problema. |
| `status` | El código de estado HTTP. |
| `detail` | Explicación concreta de este caso. |
| `instance` | URI de la petición que ha fallado. |

Spring Boot tiene soporte para este formato mediante la clase `ProblemDetail`, como verás en los documentos siguientes.

## Versionado de la API

Una API pública cambia con el tiempo, y no puedes romper a los clientes que ya la usan. Las formas más habituales de versionar son:

| Estrategia | Ejemplo |
|---|---|
| En la ruta | `GET /v1/productos` |
| En un parámetro de consulta | `GET /productos?version=1` |
| En una cabecera | `GET /productos` + `API-Version: 1` |
| En el tipo de contenido | `Accept: application/vnd.tienda.v1+json` |

La más sencilla y extendida es poner la versión en la ruta.

## Modelo de madurez de Richardson

Leonard Richardson propuso un modelo para medir "cuánto de REST" tiene una API. Martin Fowler lo popularizó en un artículo. Tiene cuatro niveles; cada uno se apoya en el anterior.

```
            ┌─────────────────────────────────────┐
  Nivel 3   │ Controles hipermedia (HATEOAS)      │  ← "la gloria de REST"
            ├─────────────────────────────────────┤
  Nivel 2   │ Verbos HTTP y códigos de estado     │  ← donde están la mayoría de APIs
            ├─────────────────────────────────────┤
  Nivel 1   │ Recursos                            │
            ├─────────────────────────────────────┤
  Nivel 0   │ Un único punto de entrada (POX)     │
            └─────────────────────────────────────┘
```

### Nivel 0 — Un único punto de entrada

HTTP se usa solo como "tubería". Hay **una sola URL** y todas las peticiones son `POST`. Lo que se quiere hacer va dentro del cuerpo.

```http
POST /api HTTP/1.1
Content-Type: application/json

{"accion": "obtenerProducto", "id": 42}
```

```http
POST /api HTTP/1.1
Content-Type: application/json

{"accion": "borrarProducto", "id": 42}
```

### Nivel 1 — Recursos

Aparecen URLs distintas para cada recurso, pero se sigue usando un único método.

```http
POST /productos/42 HTTP/1.1

{"accion": "obtener"}
```

### Nivel 2 — Verbos HTTP

Se usan los métodos HTTP con su significado y los códigos de estado correctos.

```http
GET /productos/42 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 42, "nombre": "Teclado", "precio": 29.99}
```

```http
DELETE /productos/42 HTTP/1.1
```

```http
HTTP/1.1 204 No Content
```

Gracias a que `GET` es seguro, las respuestas se pueden cachear; gracias a los códigos de estado, el cliente sabe qué ha pasado sin leer el cuerpo.

### Nivel 3 — Controles hipermedia (HATEOAS)

**HATEOAS** (*Hypermedia As The Engine Of Application State*) significa que las respuestas incluyen **enlaces** a las acciones posibles. El cliente descubre qué puede hacer a partir de la respuesta, igual que una persona navega por una web haciendo clic.

```json
{
  "id": 15,
  "estado": "PENDIENTE",
  "total": 45.50,
  "_links": {
    "self":      { "href": "/pedidos/15" },
    "cliente":   { "href": "/clientes/7" },
    "pagar":     { "href": "/pedidos/15/pago" },
    "cancelar":  { "href": "/pedidos/15/cancelacion" }
  }
}
```

Si el pedido ya estuviera pagado, el enlace `pagar` no aparecería. El servidor guía al cliente.

> El formato `_links` del ejemplo es el de **HAL**, uno de los formatos hipermedia más usados. En el ecosistema Spring existe el proyecto **Spring HATEOAS** para generarlos.

En la práctica, la mayoría de APIs se quedan en el **nivel 2**, que ya aporta casi todos los beneficios del día a día.

## Buenas prácticas resumidas

1. URIs con **sustantivos en plural**, en minúsculas y con guiones.
2. Usa el **método HTTP** adecuado para cada acción.
3. Devuelve el **código de estado** correcto. No uses `200` para todo.
4. Usa **JSON** como formato por defecto y declara siempre `Content-Type`.
5. Devuelve los errores con un formato **uniforme** (por ejemplo, *Problem Details*).
6. **Pagina** las colecciones grandes.
7. **Versiona** la API desde el principio.
8. Usa siempre **HTTPS**.
9. **No expongas detalles internos** (trazas de error, nombres de tablas, contraseñas…) en las respuestas.
10. **Documenta** la API. El estándar más usado es **OpenAPI** (antes conocido como Swagger).
11. Sé **coherente**: mismos nombres de campos, mismo formato de fechas (ISO 8601: `2026-11-22T10:30:00Z`), mismo estilo en todos los recursos.

## Ejemplo completo de diseño

API de una biblioteca:

| Método | URI | Descripción | Éxito | Errores típicos |
|---|---|---|---|---|
| `GET` | `/v1/libros` | Lista libros (admite `?autor=` y `?pagina=`) | `200` | — |
| `GET` | `/v1/libros/{id}` | Obtiene un libro | `200` | `404` |
| `POST` | `/v1/libros` | Crea un libro | `201` | `400`, `422` |
| `PUT` | `/v1/libros/{id}` | Reemplaza un libro | `200` | `400`, `404`, `422` |
| `DELETE` | `/v1/libros/{id}` | Borra un libro | `204` | `404` |
| `GET` | `/v1/socios/{id}/prestamos` | Préstamos de un socio | `200` | `404` |
| `POST` | `/v1/socios/{id}/prestamos` | Presta un libro a un socio | `201` | `404`, `409` (libro ya prestado) |

## De la teoría al código: tu primera API REST en Java

Todo lo anterior es independiente del lenguaje. Ahora veremos cómo se convierte en código Java con **Spring Boot**, el *framework* más utilizado para crear APIs REST en Java.

> Para ejecutar el ejemplo necesitas un proyecto Spring Boot con la dependencia **Spring Web**. Cómo crearlo y arrancarlo se explica paso a paso en [Spring y Spring Boot](<../14 - Spring-Boot/Fundamentos/Spring y Spring-Boot.md>). Si todavía no lo has leído, puedes quedarte por ahora con la idea general del código.

### ¿Qué es un *endpoint*?

Un ***endpoint*** es la combinación de **método HTTP + URI** que atiende la API. Por ejemplo, `GET /productos` y `POST /productos` son dos *endpoints* distintos, aunque compartan la URI.

En Spring, cada *endpoint* es un **método Java** marcado con una anotación que indica el método HTTP y la ruta:

| Anotación | Para qué sirve |
|---|---|
| `@RestController` | Marca la clase como un controlador REST: lo que devuelven sus métodos se escribe en el cuerpo de la respuesta (como JSON si es un objeto). |
| `@RequestMapping("/productos")` | Ruta base común a todos los *endpoints* de la clase. |
| `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping` | Asocian el método Java al método HTTP correspondiente. |
| `@PathVariable` | Lee una parte variable de la ruta: el `42` de `/productos/42`. |
| `@RequestParam` | Lee un parámetro de la consulta: el `30` de `/productos?precioMax=30`. |
| `@RequestBody` | Convierte el JSON del cuerpo de la petición en un objeto Java. |
| `ResponseEntity<T>` | Permite elegir el **código de estado**, las **cabeceras** y el **cuerpo** de la respuesta. |

### Cómo funciona un controlador

Antes de escribir código, una idea clave: **tú no creas el controlador ni llamas a sus métodos**. Lo hace Spring:

1. Al arrancar la aplicación, Spring crea **una única instancia** de cada clase marcada con `@RestController`.
2. Cuando llega una petición (por ejemplo, `GET /productos/1`), Spring busca el método cuya anotación coincide con ese método HTTP y esa ruta.
3. Lee de la petición los datos que pide el método (`@PathVariable`, `@RequestParam`, `@RequestBody`) y lo llama.
4. Convierte lo que devuelve el método en la respuesta HTTP. Si es un objeto, lo transforma en JSON con la librería Jackson (ver [Jackson y JSON](<Jackson y json.md>)).

Tu trabajo es, por tanto, escribir **un método por cada *endpoint*** y decidir qué devuelve.

Vamos a construir, paso a paso, una API para gestionar productos con las operaciones de la tabla del apartado *Métodos sobre una colección y un elemento*. Para centrarnos en los *endpoints*, los productos se guardan **en memoria** (en un `Map`), sin base de datos. Al reiniciar la aplicación se pierden.

> **Sobre los `import`:** en cada paso solo se muestra el código nuevo. Tu IDE añade los `import` automáticamente (en IntelliJ, `Alt + Enter` sobre la clase en rojo). Al final del apartado tienes el controlador completo con todos ellos.

### Paso 1: el controlador y los datos

Crea el fichero `src/main/java/com/ejemplo/tienda/ProductoController.java`:

```java
package com.ejemplo.tienda;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/productos")
public class ProductoController {

    // Lo que devuelve la API: un producto con su id.
    record Producto(long id, String nombre, double precio) {}

    // Lo que envía el cliente para crear o modificar: sin id, porque lo asigna el servidor.
    record DatosProducto(String nombre, double precio) {}

    // Nuestra "base de datos": id → producto.
    private final Map<Long, Producto> productos = new ConcurrentHashMap<>();

    // Contador para generar ids: 1, 2, 3...
    private final AtomicLong siguienteId = new AtomicLong(1);

    // Al crearse el controlador, guardamos dos productos para tener datos con los que probar.
    public ProductoController() {
        guardarNuevo("Teclado", 29.99);
        guardarNuevo("Ratón", 15.00);
    }

    private Producto guardarNuevo(String nombre, double precio) {
        long id = siguienteId.getAndIncrement();   // devuelve el valor actual y lo incrementa
        Producto producto = new Producto(id, nombre, precio);
        productos.put(id, producto);
        return producto;
    }
}
```

- `@RequestMapping("/productos")` hace que todos los *endpoints* de esta clase empiecen por `/productos`.
- Los datos se modelan con [*records*](<../01 - Fundamentos del lenguaje/Records.md>). Hay dos porque **lo que entra no es igual que lo que sale**: el cliente nunca envía el `id`.
- **¿Por qué `ConcurrentHashMap` y `AtomicLong` en lugar de `HashMap` y `long`?** Como Spring usa **una sola instancia** del controlador para todas las peticiones, si llegan dos a la vez se ejecutan en paralelo sobre el mismo `Map`. `HashMap` y `long` no están preparados para eso y podrían perder datos o repetir ids. Sus versiones *concurrentes* se usan exactamente igual, pero son seguras. Cuando uses una base de datos real, este problema lo resolverá ella.
- El precio es `double` para simplificar. En una aplicación real, las cantidades de dinero se representan con `BigDecimal`, porque `double` comete pequeños errores de redondeo (`0.1 + 0.2` da `0.30000000000000004`).

Todavía no hay ningún *endpoint*. Vamos a añadirlos.

### Paso 2: leer productos (`GET`)

#### Listar todos

Añade este método dentro de la clase:

```java
    // GET /productos → 200 OK + lista de productos
    @GetMapping
    public List<Producto> listar() {
        return List.copyOf(productos.values());
    }
```

`@GetMapping` sin ruta atiende la ruta base de la clase: `GET /productos`. El método devuelve una lista y Spring la convierte en un array JSON con código `200 OK`.

Arranca la aplicación y pruébalo desde otra terminal (la opción `-i` muestra el código de estado y las cabeceras; ver [Protocolo HTTP](<Protocolo http.md>)):

```bash
curl -i http://localhost:8080/productos
```

```http
HTTP/1.1 200
Content-Type: application/json

[{"id":1,"nombre":"Teclado","precio":29.99},{"id":2,"nombre":"Ratón","precio":15.0}]
```

> El orden de los productos puede variar: un `Map` no garantiza ningún orden.

#### Filtrar con un parámetro de consulta

Ahora queremos poder pedir `GET /productos?precioMax=20`. Sustituye el método `listar` por este:

```java
    // GET /productos             → todos
    // GET /productos?precioMax=20 → solo los que cuestan 20 o menos
    @GetMapping
    public List<Producto> listar(@RequestParam(required = false) Double precioMax) {
        if (precioMax == null) {
            return List.copyOf(productos.values());
        }
        return productos.values().stream()
                .filter(p -> p.precio() <= precioMax)
                .toList();
    }
```

- `@RequestParam` lee el parámetro `precioMax` de la URL y lo convierte a número.
- `required = false` indica que es opcional. Si no llega, Spring pasa `null`.
- Por eso el tipo es **`Double`** (la clase) y no `double` (el primitivo): un `double` no puede valer `null`.
- El filtrado usa un *stream* (ver [Lambdas y Streams](<../02 - Programacion funcional/Lambdas y Streams.md>)).

```bash
curl "http://localhost:8080/productos?precioMax=20"
# [{"id":2,"nombre":"Ratón","precio":15.0}]
```

#### Obtener uno por su id

```java
    // GET /productos/1 → 200 OK + producto,  o 404 Not Found si no existe
    @GetMapping("/{id}")
    public ResponseEntity<Producto> obtener(@PathVariable long id) {
        Producto producto = productos.get(id);
        if (producto == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(producto);
    }
```

- `"/{id}"` es una ruta con una parte **variable**. `@PathVariable long id` recoge su valor: para `/productos/1`, `id` vale `1`.
- Aquí aparece **`ResponseEntity`**. En `listar` no hacía falta porque la respuesta siempre es `200 OK`. Pero este método tiene **dos respuestas posibles** (`200` o `404`), y `ResponseEntity` representa la respuesta HTTP completa: código de estado, cabeceras y cuerpo. El tipo entre `< >` es el del cuerpo.

`ResponseEntity` se crea con métodos estáticos. Algunos devuelven la respuesta ya terminada; otros devuelven un objeto a medio construir que se completa con `.body(...)` (si lleva cuerpo) o `.build()` (si no lo lleva). Estas son las formas que usaremos:

| Respuesta | Código |
|---|---|
| `200 OK` con cuerpo | `ResponseEntity.ok(producto)` |
| `201 Created` con cabecera `Location` y cuerpo | `ResponseEntity.created(uri).body(producto)` |
| `204 No Content` sin cuerpo | `ResponseEntity.noContent().build()` |
| `404 Not Found` sin cuerpo | `ResponseEntity.notFound().build()` |

```bash
curl -i http://localhost:8080/productos/1     # HTTP/1.1 200 + {"id":1,"nombre":"Teclado","precio":29.99}
curl -i http://localhost:8080/productos/999   # HTTP/1.1 404
```

### Paso 3: crear un producto (`POST`)

```java
    // POST /productos → 201 Created + cabecera Location + producto creado
    @PostMapping
    public ResponseEntity<Producto> crear(@RequestBody DatosProducto datos) {
        Producto producto = guardarNuevo(datos.nombre(), datos.precio());
        URI ubicacion = URI.create("/productos/" + producto.id());
        return ResponseEntity.created(ubicacion).body(producto);
    }
```

- `@RequestBody` le pide a Spring que convierta el JSON del cuerpo de la petición en un objeto `DatosProducto`. Los nombres de los campos del JSON deben coincidir con los del *record*.
- Como dice la teoría, al crear se responde **`201 Created`** con la cabecera **`Location`**, que indica la URI del nuevo recurso.

```bash
curl -i -X POST http://localhost:8080/productos \
     -H "Content-Type: application/json" \
     -d '{"nombre": "Monitor", "precio": 149.90}'
```

```http
HTTP/1.1 201
Location: /productos/3
Content-Type: application/json

{"id":3,"nombre":"Monitor","precio":149.9}
```

La cabecera `Content-Type: application/json` es necesaria: sin ella, Spring no sabe que el cuerpo es JSON y responde `415 Unsupported Media Type`.

> En Windows (PowerShell o CMD), las comillas simples de `-d '...'` pueden dar problemas. Si ocurre, usa comillas dobles y escapa las interiores: `-d "{\"nombre\": \"Monitor\", \"precio\": 149.90}"`. También puedes probar la API con herramientas gráficas como **Postman**, **Bruno** o el cliente HTTP de IntelliJ.

### Paso 4: reemplazar y borrar (`PUT` y `DELETE`)

Estos dos métodos combinan lo que ya has visto.

```java
    // PUT /productos/1 → 200 OK + producto reemplazado,  o 404 Not Found
    @PutMapping("/{id}")
    public ResponseEntity<Producto> reemplazar(@PathVariable long id,
                                               @RequestBody DatosProducto datos) {
        if (!productos.containsKey(id)) {
            return ResponseEntity.notFound().build();
        }
        Producto producto = new Producto(id, datos.nombre(), datos.precio());
        productos.put(id, producto);
        return ResponseEntity.ok(producto);
    }

    // DELETE /productos/1 → 204 No Content,  o 404 Not Found
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> borrar(@PathVariable long id) {
        Producto borrado = productos.remove(id);   // devuelve null si no existía
        if (borrado == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.noContent().build();
    }
```

- `reemplazar` usa a la vez `@PathVariable` (qué producto) y `@RequestBody` (sus nuevos datos).
- `borrar` no devuelve cuerpo en ningún caso. Por eso el tipo es **`ResponseEntity<Void>`**: `Void` significa "sin cuerpo".

```bash
curl -i -X PUT http://localhost:8080/productos/1 \
     -H "Content-Type: application/json" \
     -d '{"nombre": "Teclado mecánico", "precio": 59.90}'
# HTTP/1.1 200 + {"id":1,"nombre":"Teclado mecánico","precio":59.9}

curl -i -X DELETE http://localhost:8080/productos/1   # HTTP/1.1 204
curl -i -X DELETE http://localhost:8080/productos/1   # HTTP/1.1 404 (ya no existe)
```

### El controlador completo

Este es el resultado de todos los pasos, con todos los `import`:

```java
package com.ejemplo.tienda;

import java.net.URI;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/productos")
public class ProductoController {

    record Producto(long id, String nombre, double precio) {}

    record DatosProducto(String nombre, double precio) {}

    private final Map<Long, Producto> productos = new ConcurrentHashMap<>();
    private final AtomicLong siguienteId = new AtomicLong(1);

    public ProductoController() {
        guardarNuevo("Teclado", 29.99);
        guardarNuevo("Ratón", 15.00);
    }

    // GET /productos  y  GET /productos?precioMax=20
    @GetMapping
    public List<Producto> listar(@RequestParam(required = false) Double precioMax) {
        if (precioMax == null) {
            return List.copyOf(productos.values());
        }
        return productos.values().stream()
                .filter(p -> p.precio() <= precioMax)
                .toList();
    }

    // GET /productos/1
    @GetMapping("/{id}")
    public ResponseEntity<Producto> obtener(@PathVariable long id) {
        Producto producto = productos.get(id);
        if (producto == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(producto);
    }

    // POST /productos
    @PostMapping
    public ResponseEntity<Producto> crear(@RequestBody DatosProducto datos) {
        Producto producto = guardarNuevo(datos.nombre(), datos.precio());
        URI ubicacion = URI.create("/productos/" + producto.id());
        return ResponseEntity.created(ubicacion).body(producto);
    }

    // PUT /productos/1
    @PutMapping("/{id}")
    public ResponseEntity<Producto> reemplazar(@PathVariable long id,
                                               @RequestBody DatosProducto datos) {
        if (!productos.containsKey(id)) {
            return ResponseEntity.notFound().build();
        }
        Producto producto = new Producto(id, datos.nombre(), datos.precio());
        productos.put(id, producto);
        return ResponseEntity.ok(producto);
    }

    // DELETE /productos/1
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> borrar(@PathVariable long id) {
        Producto borrado = productos.remove(id);
        if (borrado == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.noContent().build();
    }

    private Producto guardarNuevo(String nombre, double precio) {
        long id = siguienteId.getAndIncrement();
        Producto producto = new Producto(id, nombre, precio);
        productos.put(id, producto);
        return producto;
    }
}
```

### Qué reglas de diseño se han aplicado

- Las URIs usan un **sustantivo en plural** (`/productos`), y la acción la decide el **método HTTP**, no el nombre de la ruta.
- Cada operación devuelve el **código de estado** adecuado: `201` al crear, `204` al borrar y `404` si el producto no existe. Nunca un `200` con un error dentro.
- Al crear, la cabecera **`Location`** indica la URI del nuevo recurso.
- El **filtrado** se hace con un parámetro de consulta (`?precioMax=`), no con una ruta nueva.
- El cliente **no envía el `id`**: lo asigna el servidor.

### Qué le falta a este ejemplo

Este controlador es intencionadamente mínimo para centrarse en los *endpoints*. En una aplicación real se añade:

| Mejora | Dónde se explica |
|---|---|
| Separar el código en controlador, servicio y repositorio | [Arquitectura por capas](<../15 - Arquitectura/Arquitectura por capas.md>) |
| Validar los datos de entrada (nombre obligatorio, precio positivo…) | [DTOs y validación](<../14 - Spring-Boot/APIs-REST/DTOs y validacion.md>) |
| Devolver errores con formato *Problem Details* (`ProblemDetail`) | [Gestión de errores](<../14 - Spring-Boot/APIs-REST/Gestion de errores.md>) |
| Guardar los datos en una base de datos | [Spring Data JPA](<../14 - Spring-Boot/Persistencia/Spring data jpa.md>) |
| Probar los *endpoints* automáticamente | [Testing y TDD](<../14 - Spring-Boot/Testing/Testing y TDD.md>) |

## Referencias

- REST API Tutorial: <https://www.restapitutorial.com/>
- REST: buenas prácticas para el diseño de APIs (Hashmap, Medium): <https://medium.com/hashmapinc/rest-good-practices-for-api-design-881439796dc9>
- RESTful API: <https://restfulapi.net/>
- Martin Fowler — Richardson Maturity Model: <https://martinfowler.com/articles/richardsonMaturityModel.html>
- RFC 9457 — Problem Details for HTTP APIs: <https://www.rfc-editor.org/rfc/rfc9457>
- Especificación OpenAPI: <https://www.openapis.org/>
