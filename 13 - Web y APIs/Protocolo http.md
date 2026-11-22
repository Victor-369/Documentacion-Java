# Protocolo HTTP

## ¿Qué es HTTP?

**HTTP** (*HyperText Transfer Protocol*, protocolo de transferencia de hipertexto) es el conjunto de reglas que usan los programas para intercambiar información en la web. Cada vez que abres una página, una app móvil carga datos o un programa Java llama a una API, casi siempre hay HTTP por medio.

Sus características principales:

- **Cliente-servidor.** Un *cliente* (navegador, app, otro programa) envía una **petición** (*request*) y un *servidor* devuelve una **respuesta** (*response*). Siempre empieza el cliente.
- **Sin estado** (*stateless*). Cada petición es independiente: el servidor no recuerda las anteriores por sí mismo. Si hace falta "recordar" algo (por ejemplo, que un usuario ha iniciado sesión), se usan mecanismos adicionales como *cookies* o *tokens*.
- **Basado en texto y legible.** En HTTP/1.1 los mensajes son texto que una persona puede leer. HTTP/2 y HTTP/3 los envían en formato binario por eficiencia, pero el significado (métodos, cabeceras, códigos) es el mismo.
- **Extensible.** Mediante las **cabeceras** se pueden añadir funcionalidades nuevas sin cambiar el protocolo.

```
┌──────────┐      Petición HTTP       ┌──────────┐
│          │ ───────────────────────▶ │          │
│ Cliente  │                          │ Servidor │
│          │ ◀─────────────────────── │          │
└──────────┘      Respuesta HTTP      └──────────┘
```

## Dónde se sitúa HTTP

HTTP no viaja solo. Se apoya en otros protocolos que se encargan de llevar los datos de un ordenador a otro:

| Capa | Protocolo | Qué hace |
|---|---|---|
| Aplicación | **HTTP** | Define el significado de los mensajes (qué pido, qué respondo). |
| Seguridad | **TLS** | Cifra la comunicación. HTTP + TLS = **HTTPS**. |
| Transporte | **TCP** (HTTP/1.1 y HTTP/2) o **QUIC sobre UDP** (HTTP/3) | Garantiza que los datos llegan completos y en orden. |
| Red | **IP** | Lleva los paquetes de una dirección a otra. |

> **Importante:** en cualquier aplicación real usa siempre **HTTPS**. Con HTTP sin cifrar, cualquiera que esté en la misma red puede leer o modificar los datos.

### Versiones de HTTP

| Versión | Lo más destacado |
|---|---|
| HTTP/1.1 | Mensajes en texto. Conexiones reutilizables. Sigue muy presente. |
| HTTP/2 | Mensajes binarios. Varias peticiones a la vez por la misma conexión (*multiplexación*). Compresión de cabeceras. |
| HTTP/3 | Funciona sobre QUIC (UDP) en lugar de TCP. Conexiones más rápidas y mejor comportamiento en redes inestables. |

Como programador de APIs, la versión rara vez cambia tu código: los conceptos de este documento son válidos para todas.

## La URL

Una **URL** identifica el recurso al que quieres acceder:

```
https://api.tienda.com:443/productos/42?moneda=EUR#detalle
└─┬─┘   └──────┬─────┘└┬┘└─────┬─────┘└────┬─────┘└──┬──┘
esquema     host    puerto   ruta      consulta   fragmento
```

| Parte | Ejemplo | Para qué sirve |
|---|---|---|
| Esquema | `https` | Protocolo a usar. |
| Host | `api.tienda.com` | Servidor al que conectarse. |
| Puerto | `443` | Opcional. Por defecto 80 para `http` y 443 para `https`. |
| Ruta (*path*) | `/productos/42` | Qué recurso se pide dentro del servidor. |
| Consulta (*query string*) | `?moneda=EUR` | Parámetros adicionales `clave=valor`, separados por `&`. |
| Fragmento | `#detalle` | Lo usa solo el navegador; **no se envía** al servidor. |

## Anatomía de una petición

```http
POST /productos HTTP/1.1
Host: api.tienda.com
Content-Type: application/json
Accept: application/json
Content-Length: 41

{"nombre": "Teclado", "precio": 29.99}
```

Una petición tiene cuatro partes:

1. **Línea de petición:** método (`POST`), ruta (`/productos`) y versión (`HTTP/1.1`).
2. **Cabeceras:** pares `Nombre: valor` con información adicional.
3. **Una línea en blanco** que separa cabeceras y cuerpo.
4. **Cuerpo** (*body*), opcional: los datos que se envían. Las peticiones `GET` normalmente no llevan cuerpo.

## Anatomía de una respuesta

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /productos/42
Content-Length: 50

{"id": 42, "nombre": "Teclado", "precio": 29.99}
```

1. **Línea de estado:** versión, **código de estado** (`201`) y una frase descriptiva (`Created`).
2. **Cabeceras.**
3. **Línea en blanco.**
4. **Cuerpo**, opcional.

## Métodos HTTP

El **método** (también llamado *verbo*) indica qué acción quiere hacer el cliente sobre el recurso.

| Método | Para qué se usa | ¿Lleva cuerpo la petición? |
|---|---|---|
| `GET` | Obtener un recurso. | No |
| `POST` | Crear un recurso nuevo o lanzar un procesamiento. | Sí |
| `PUT` | Reemplazar un recurso completo (o crearlo en una URL conocida). | Sí |
| `PATCH` | Modificar **parte** de un recurso. | Sí |
| `DELETE` | Eliminar un recurso. | Normalmente no |
| `HEAD` | Igual que `GET`, pero la respuesta no tiene cuerpo (solo cabeceras). | No |
| `OPTIONS` | Preguntar qué métodos/opciones admite un recurso. | No |
| `CONNECT` | Abrir un túnel (lo usan los *proxies* para HTTPS). | No |
| `TRACE` | Prueba de eco de la petición (diagnóstico). | No |

En una API REST usarás casi siempre `GET`, `POST`, `PUT`, `PATCH` y `DELETE`.

### Métodos seguros e idempotentes

Dos propiedades muy importantes a la hora de diseñar una API:

- **Seguro:** el método **no modifica nada** en el servidor; solo lee.
- **Idempotente:** hacer la misma petición **una vez o diez veces deja el servidor en el mismo estado**.

| Método | Seguro | Idempotente |
|---|---|---|
| `GET` | Sí | Sí |
| `HEAD` | Sí | Sí |
| `OPTIONS` | Sí | Sí |
| `TRACE` | Sí | Sí |
| `PUT` | No | Sí |
| `DELETE` | No | Sí |
| `POST` | No | No |
| `PATCH` | No | No (no está garantizado) |
| `CONNECT` | No | No |

**Ejemplo para entender la idempotencia:**

- `DELETE /productos/42` → la primera vez borra el producto. La segunda, ya no existe. El estado final del servidor es el mismo: el producto 42 no está. **Idempotente.**
- `PUT /productos/42` con `{"precio": 10}` → da igual cuántas veces lo envíes: el producto acaba con precio 10. **Idempotente.**
- `POST /productos` con `{"nombre": "Ratón"}` → cada envío crea un producto nuevo. Tres envíos = tres ratones. **No idempotente.**

Esto importa porque, si una petición falla por un problema de red, **solo es seguro reintentarla automáticamente si es idempotente**.

## Códigos de estado

El código de estado es un número de tres cifras que resume qué ha pasado. La primera cifra indica la familia:

| Familia | Significado |
|---|---|
| `1xx` | Informativo. La petición se ha recibido y continúa el proceso. |
| `2xx` | Éxito. |
| `3xx` | Redirección. Hay que ir a otro sitio para completar la petición. |
| `4xx` | Error del cliente. La petición tiene algún problema. |
| `5xx` | Error del servidor. La petición era válida, pero el servidor ha fallado. |

### Los más habituales en una API

| Código | Nombre | Cuándo usarlo |
|---|---|---|
| `200` | OK | Petición correcta con datos en la respuesta (típico de `GET`, `PUT`, `PATCH`). |
| `201` | Created | Se ha creado un recurso (típico de `POST`). Suele acompañarse de la cabecera `Location` con la URL del recurso nuevo. |
| `204` | No Content | Correcto, pero no hay nada que devolver (típico de `DELETE`). |
| `301` | Moved Permanently | El recurso se ha movido para siempre a otra URL. |
| `304` | Not Modified | El cliente ya tiene la versión actual en caché. |
| `400` | Bad Request | La petición está mal formada (JSON inválido, parámetro con formato incorrecto…). |
| `401` | Unauthorized | Falta autenticarse o las credenciales no son válidas. |
| `403` | Forbidden | El usuario está identificado, pero no tiene permiso. |
| `404` | Not Found | El recurso no existe. |
| `405` | Method Not Allowed | El recurso existe, pero no admite ese método. |
| `409` | Conflict | La petición choca con el estado actual (por ejemplo, un email ya registrado). |
| `415` | Unsupported Media Type | El formato del cuerpo no es el que acepta el servidor. |
| `422` | Unprocessable Content | El JSON es correcto, pero los datos no son válidos (por ejemplo, un precio negativo). |
| `429` | Too Many Requests | El cliente ha superado el límite de peticiones. |
| `500` | Internal Server Error | Error inesperado en el servidor. |
| `502` | Bad Gateway | Un servidor intermedio ha recibido una respuesta inválida de otro. |
| `503` | Service Unavailable | El servidor no puede atender ahora (sobrecarga, mantenimiento). |

> **401 frente a 403:** `401` significa "no sé quién eres"; `403` significa "sé quién eres, pero no puedes hacer esto".

## Cabeceras importantes

| Cabecera | Dónde | Para qué sirve |
|---|---|---|
| `Host` | Petición | Nombre del servidor. Obligatoria en HTTP/1.1. |
| `Content-Type` | Ambas | Formato del cuerpo. Para JSON: `application/json`. |
| `Content-Length` | Ambas | Tamaño del cuerpo en bytes. |
| `Accept` | Petición | Formatos que el cliente acepta en la respuesta. |
| `Authorization` | Petición | Credenciales, por ejemplo `Bearer <token>`. |
| `Location` | Respuesta | URL de un recurso recién creado o de una redirección. |
| `Cache-Control` | Ambas | Reglas de caché. |
| `ETag` | Respuesta | "Huella" de una versión concreta de un recurso, útil para caché. |
| `User-Agent` | Petición | Qué programa hace la petición. |
| `Set-Cookie` / `Cookie` | Respuesta / Petición | Guardar y reenviar *cookies*. |

### Negociación de contenido

El cliente indica con `Accept` qué formato prefiere, y el servidor responde con `Content-Type` diciendo qué formato ha usado:

```http
GET /productos/42 HTTP/1.1
Host: api.tienda.com
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 42, "nombre": "Teclado", "precio": 29.99}
```

Si el servidor no puede devolver ninguno de los formatos pedidos, responde `406 Not Acceptable`.

## Probar HTTP desde la terminal con `curl`

`curl` viene instalado en la mayoría de sistemas. La opción `-i` muestra también las cabeceras de la respuesta y `-v` muestra la conversación completa.

```bash
# GET simple
curl -i https://httpbin.org/get

# POST con JSON
curl -i -X POST https://httpbin.org/post \
     -H "Content-Type: application/json" \
     -d '{"nombre": "Teclado", "precio": 29.99}'

# Ver la petición y la respuesta completas
curl -v https://httpbin.org/status/404
```

> `httpbin.org` es un servicio público de pruebas que devuelve lo que le envías. Es muy útil para experimentar.

## HTTP desde Java

Java incluye desde hace años un cliente HTTP moderno en el módulo `java.net.http`. Este ejemplo es un archivo compacto de Java 25 que puedes ejecutar con `java ClienteHttp.java`:

```java
// ClienteHttp.java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

void main() throws Exception {
    try (HttpClient cliente = HttpClient.newHttpClient()) {

        // 1. Construimos la petición
        HttpRequest peticion = HttpRequest.newBuilder()
                .uri(URI.create("https://httpbin.org/get"))
                .header("Accept", "application/json")
                .GET()
                .build();

        // 2. La enviamos y leemos el cuerpo como texto
        HttpResponse<String> respuesta =
                cliente.send(peticion, HttpResponse.BodyHandlers.ofString());

        // 3. Mostramos el resultado
        IO.println("Código de estado: " + respuesta.statusCode());
        IO.println("Content-Type: " + respuesta.headers().firstValue("Content-Type").orElse("?"));
        IO.println("Cuerpo:");
        IO.println(respuesta.body());
    }
}
```

Y un `POST` con cuerpo JSON:

```java
// ClientePost.java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

void main() throws Exception {
    String json = """
            {"nombre": "Teclado", "precio": 29.99}
            """;

    try (HttpClient cliente = HttpClient.newHttpClient()) {
        HttpRequest peticion = HttpRequest.newBuilder()
                .uri(URI.create("https://httpbin.org/post"))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(json))
                .build();

        HttpResponse<String> respuesta =
                cliente.send(peticion, HttpResponse.BodyHandlers.ofString());

        IO.println(respuesta.statusCode());
        IO.println(respuesta.body());
    }
}
```

Fíjate en:

- `try (...)`: `HttpClient` se cierra automáticamente al terminar el bloque (*try-with-resources*).
- `"""` … `"""`: es un **bloque de texto** (*text block*), muy cómodo para escribir JSON dentro del código.
- `java.net.http` no forma parte de `java.base`, por eso aquí sí hay que escribir los `import`.

## Resumen

- HTTP es un protocolo cliente-servidor, sin estado, basado en peticiones y respuestas.
- Una petición tiene **método + ruta + cabeceras + cuerpo opcional**.
- Una respuesta tiene **código de estado + cabeceras + cuerpo opcional**.
- Los métodos tienen un significado concreto; conviene conocer cuáles son **seguros** e **idempotentes**.
- Los códigos de estado se agrupan por familias: `2xx` éxito, `4xx` error del cliente, `5xx` error del servidor.
- Usa siempre HTTPS en entornos reales.

## Referencias

- MDN — Visión general de HTTP (español): <https://developer.mozilla.org/es/docs/Web/HTTP/Guides/Overview>
- MDN — Overview of HTTP (inglés): <https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview>
- MDN — Códigos de estado de respuesta HTTP: <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status>
- MDN — Métodos de petición HTTP: <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods>
- RFC 9110 — HTTP Semantics (la especificación oficial): <https://www.rfc-editor.org/rfc/rfc9110>
