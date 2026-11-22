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

## Referencias

- REST API Tutorial: <https://www.restapitutorial.com/>
- REST: buenas prácticas para el diseño de APIs (Hashmap, Medium): <https://medium.com/hashmapinc/rest-good-practices-for-api-design-881439796dc9>
- RESTful API: <https://restfulapi.net/>
- Martin Fowler — Richardson Maturity Model: <https://martinfowler.com/articles/richardsonMaturityModel.html>
- RFC 9457 — Problem Details for HTTP APIs: <https://www.rfc-editor.org/rfc/rfc9457>
- Especificación OpenAPI: <https://www.openapis.org/>
