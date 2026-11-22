# Gestión profesional de errores

## Objetivos

Una API profesional gestiona los errores de forma que:

1. **El cliente entiende qué ha pasado**: código de estado correcto y mensaje claro.
2. **Todas las respuestas de error tienen el mismo formato**, venga el error de donde venga.
3. **No se filtran detalles internos**: ni trazas de pila (*stack traces*), ni nombres de tablas, ni mensajes de excepciones técnicas.
4. **El equipo puede diagnosticar**: los errores inesperados quedan registrados en los *logs* con todo el detalle.
5. **El código de negocio queda limpio**: sin bloques `try/catch` repartidos por todos los controladores.

## El formato estándar: *Problem Details* (RFC 9457)

El estándar **RFC 9457** define un formato JSON para los errores de las APIs HTTP, con el tipo de contenido `application/problem+json`:

```json
{
  "type": "https://api.tienda.com/errores/stock-insuficiente",
  "title": "Stock insuficiente",
  "status": 422,
  "detail": "Solo quedan 2 unidades de 'Teclado mecánico' y se han pedido 5",
  "instance": "/api/v1/pedidos",
  "productoId": 42,
  "disponible": 2
}
```

| Campo | Significado |
|---|---|
| `type` | URI que identifica el **tipo** de error. Puede apuntar a documentación. Si no hay uno específico: `about:blank`. |
| `title` | Resumen corto y estable del tipo de error. |
| `status` | Código de estado HTTP. |
| `detail` | Explicación de **este** caso concreto. |
| `instance` | URI de la petición que ha fallado. |
| *Extensiones* | Campos adicionales propios (`productoId`, `errores`…). |

Spring lo implementa con la clase **`ProblemDetail`**.

## Las herramientas de Spring

| Herramienta | Para qué sirve |
|---|---|
| `@ResponseStatus` en una excepción | Asociar una excepción propia a un código de estado. |
| `ResponseStatusException` | Lanzar un error HTTP "al vuelo", sin crear una clase. |
| `ErrorResponseException` | Excepción base que ya contiene un `ProblemDetail`. |
| `@ExceptionHandler` | Método que captura una excepción y construye la respuesta. |
| `@RestControllerAdvice` | Clase con `@ExceptionHandler` que aplica a **todos** los controladores. |
| `ResponseEntityExceptionHandler` | Clase base con el tratamiento de todas las excepciones propias de Spring MVC en formato *Problem Details*. |
| `spring.mvc.problemdetails.enabled` | Propiedad de Spring Boot que activa ese tratamiento sin escribir código. |

### `@ResponseStatus`: lo más simple

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ProductoNoEncontradoException extends RuntimeException {
    public ProductoNoEncontradoException(Long id) {
        super("No existe el producto " + id);
    }
}
```

Funciona, pero da poco control sobre el cuerpo de la respuesta.

### `ResponseStatusException`: rápido, para casos puntuales

```java
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No existe el producto " + id);
```

Cómodo, pero mete HTTP dentro del código donde se lanza. **No lo uses en la capa de servicio**.

## Diseño recomendado

```
 Servicio / dominio                      Capa web
 ───────────────────                     ────────────────────────────────
 lanza excepciones de NEGOCIO  ──────▶   @RestControllerAdvice las traduce
 (sin saber nada de HTTP)                a ProblemDetail + código de estado
```

### Una jerarquía de excepciones de negocio

Con Java 25 se puede usar una **clase sellada** (`sealed`) para dejar explícito qué tipos de error de negocio existen:

```java
package com.ejemplo.tienda.comun.error;

public abstract sealed class ErrorDeNegocio extends RuntimeException
        permits RecursoNoEncontradoException, ConflictoException, ReglaDeNegocioException {

    protected ErrorDeNegocio(String mensaje) {
        super(mensaje);
    }
}
```

```java
package com.ejemplo.tienda.comun.error;

/** El recurso pedido no existe → 404 */
public final class RecursoNoEncontradoException extends ErrorDeNegocio {

    public RecursoNoEncontradoException(String recurso, Object id) {
        super("No existe " + recurso + " con id " + id);
    }
}
```

```java
package com.ejemplo.tienda.comun.error;

/** La operación choca con el estado actual (duplicados, versiones…) → 409 */
public final class ConflictoException extends ErrorDeNegocio {

    public ConflictoException(String mensaje) {
        super(mensaje);
    }
}
```

```java
package com.ejemplo.tienda.comun.error;

/** Los datos son correctos en formato, pero violan una regla de negocio → 422 */
public final class ReglaDeNegocioException extends ErrorDeNegocio {

    public ReglaDeNegocioException(String mensaje) {
        super(mensaje);
    }
}
```

> Las clases `sealed` deben estar en el mismo paquete que sus subclases (o en el mismo módulo). Si prefieres no usarlas, una clase abstracta normal funciona igual.

### Un manejador global

```java
package com.ejemplo.tienda.comun.error;

import java.net.URI;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.HttpStatusCode;
import org.springframework.http.ProblemDetail;
import org.springframework.http.ResponseEntity;
import org.springframework.orm.ObjectOptimisticLockingFailureException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

@RestControllerAdvice
public class ManejadorGlobalErrores extends ResponseEntityExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(ManejadorGlobalErrores.class);
    private static final String BASE_TIPOS = "https://api.tienda.com/errores/";

    // ---------- Errores de negocio ----------

    @ExceptionHandler(ErrorDeNegocio.class)
    public ProblemDetail errorDeNegocio(ErrorDeNegocio ex) {
        // switch con patrones sobre una jerarquía sellada: el compilador comprueba que están todos
        HttpStatus estado = switch (ex) {
            case RecursoNoEncontradoException _ -> HttpStatus.NOT_FOUND;
            case ConflictoException _ -> HttpStatus.CONFLICT;
            case ReglaDeNegocioException _ -> HttpStatus.UNPROCESSABLE_CONTENT;
        };

        ProblemDetail problema = ProblemDetail.forStatusAndDetail(estado, ex.getMessage());
        problema.setType(URI.create(BASE_TIPOS + tipoDe(ex)));
        return problema;
    }

    // ---------- Concurrencia optimista (@Version) ----------

    @ExceptionHandler(ObjectOptimisticLockingFailureException.class)
    public ProblemDetail conflictoDeVersion(ObjectOptimisticLockingFailureException ex) {
        ProblemDetail problema = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT,
                "El recurso ha sido modificado por otra petición. Vuelve a cargarlo e inténtalo de nuevo.");
        problema.setTitle("Conflicto de versión");
        return problema;
    }

    // ---------- Errores de validación de @Valid (personalizamos el de Spring) ----------

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex, HttpHeaders headers,
            HttpStatusCode status, WebRequest request) {

        record ErrorCampo(String campo, String mensaje) {}

        List<ErrorCampo> errores = ex.getBindingResult().getFieldErrors().stream()
                .map(e -> new ErrorCampo(e.getField(), e.getDefaultMessage()))
                .toList();

        ProblemDetail problema = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST,
                "La petición contiene datos no válidos");
        problema.setTitle("Datos no válidos");
        problema.setType(URI.create(BASE_TIPOS + "validacion"));
        problema.setProperty("errores", errores);

        return ResponseEntity.badRequest().body(problema);
    }

    // ---------- Cualquier otro error inesperado ----------

    @ExceptionHandler(Exception.class)
    public ProblemDetail errorInesperado(Exception ex) {
        // Se registra TODO el detalle en el log…
        log.error("Error inesperado", ex);
        // …pero al cliente solo se le da un mensaje genérico
        ProblemDetail problema = ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR,
                "Se ha producido un error interno. Inténtalo más tarde.");
        problema.setTitle("Error interno");
        return problema;
    }

    private static String tipoDe(ErrorDeNegocio ex) {
        return switch (ex) {
            case RecursoNoEncontradoException _ -> "no-encontrado";
            case ConflictoException _ -> "conflicto";
            case ReglaDeNegocioException _ -> "regla-de-negocio";
        };
    }
}
```

Qué aporta cada parte:

- **Heredar de `ResponseEntityExceptionHandler`** hace que **todas las excepciones propias de Spring MVC** (JSON mal formado, método no permitido, tipo de contenido no soportado, parámetro con tipo incorrecto…) se devuelvan automáticamente como *Problem Details* con su código correcto.
- Se **sobrescribe** `handleMethodArgumentNotValid` para dar una lista de errores por campo.
- El **`switch` con patrones** sobre la clase sellada no necesita `default`: si mañana añades un tipo de error nuevo a `permits`, el compilador te obliga a tratarlo.
- El manejador de `Exception` es la **red de seguridad**: registra el error completo y responde con un mensaje genérico.

> Spring busca el `@ExceptionHandler` **más específico**. Una excepción `ConflictoException` la trata `errorDeNegocio`, no `errorInesperado`.
>
> `HttpStatus.UNPROCESSABLE_CONTENT` es el nombre actual del código 422 según la RFC 9110. En Spring Framework 7 el nombre antiguo, `UNPROCESSABLE_ENTITY`, está marcado como obsoleto.
>
> `case RecursoNoEncontradoException _` usa un **patrón sin nombre** (`_`): indica que solo interesa el tipo, no la variable.

### Ejemplo de respuesta de validación

```http
HTTP/1.1 400
Content-Type: application/problem+json

{
  "type": "https://api.tienda.com/errores/validacion",
  "title": "Datos no válidos",
  "status": 400,
  "detail": "La petición contiene datos no válidos",
  "instance": "/api/v1/clientes",
  "errores": [
    { "campo": "email", "mensaje": "El email no tiene un formato válido" },
    { "campo": "nombre", "mensaje": "El nombre es obligatorio" }
  ]
}
```

## Si no quieres escribir código: la propiedad de Spring Boot

```properties
spring.mvc.problemdetails.enabled=true
```

Spring Boot registra entonces su propio `ResponseEntityExceptionHandler`, y las excepciones de Spring MVC se devuelven en formato *Problem Details*. Es un buen punto de partida. En cuanto necesites tratar tus propias excepciones o personalizar las de Spring, escribe tu `@RestControllerAdvice` como en el apartado anterior.

## Qué código de estado usar

| Situación | Código | Origen típico |
|---|---|---|
| JSON mal formado, parámetro con tipo incorrecto | `400 Bad Request` | Spring MVC |
| Falla una validación de formato (`@Valid`) | `400 Bad Request` | Bean Validation |
| Falta autenticación | `401 Unauthorized` | Spring Security |
| Sin permiso | `403 Forbidden` | Spring Security |
| El recurso no existe | `404 Not Found` | Negocio |
| Método HTTP no soportado | `405 Method Not Allowed` | Spring MVC |
| Duplicado, conflicto de versión, estado incompatible | `409 Conflict` | Negocio / JPA |
| Tipo de contenido no soportado | `415 Unsupported Media Type` | Spring MVC |
| Datos bien formados que violan una regla de negocio | `422 Unprocessable Content` | Negocio |
| Error inesperado | `500 Internal Server Error` | Cualquiera |
| Un servicio externo no responde | `502` / `503` / `504` | Integraciones |

> Hay equipos que usan `400` también para las reglas de negocio en lugar de `422`. Las dos opciones son defendibles; lo importante es **documentarlo y ser coherente**.

## Errores de la base de datos

Algunos errores llegan desde la base de datos, por ejemplo, una restricción `UNIQUE` violada. Spring los traduce a su jerarquía `DataAccessException`, como `DataIntegrityViolationException`.

Se pueden capturar en el manejador para devolver `409`, pero **no expongas el mensaje original**: suele contener nombres de tablas y restricciones. Lo ideal es comprobar la regla en el servicio (por ejemplo, `existsByEmail`) para dar un mensaje claro, y dejar la restricción de la base de datos como **última barrera** ante peticiones simultáneas.

## *Logging* de errores

| Tipo de error | Nivel de *log* recomendado | Motivo |
|---|---|---|
| Errores del cliente (4xx) | `DEBUG`, `INFO` o `WARN` | Son "normales": el cliente envió algo incorrecto. |
| Errores inesperados (5xx) | `ERROR`, con la traza completa | Hay que investigarlos. |

No registres datos sensibles (contraseñas, *tokens*, datos personales completos).

## Buenas prácticas

1. El **servicio lanza excepciones de negocio**; la **capa web las traduce** a HTTP.
2. Un único `@RestControllerAdvice` (o pocos, bien organizados) para toda la aplicación.
3. **Formato único** para todos los errores: *Problem Details*.
4. Usa el **código de estado correcto**.
5. **Nunca** devuelvas trazas de pila ni mensajes técnicos al cliente.
6. Registra los errores 5xx con todo el detalle.
7. Documenta los tipos de error (`type`) de tu API.
8. **Prueba** los casos de error igual que los de éxito ([Testing con Spring Boot y TDD](<../Testing/Testing y TDD.md>)).

## Resumen

- Usa **`ProblemDetail`** (RFC 9457) como formato de error.
- Crea una **jerarquía de excepciones de negocio** sin dependencias de HTTP.
- Centraliza la traducción en un **`@RestControllerAdvice`** que herede de **`ResponseEntityExceptionHandler`**.
- Añade un manejador genérico como red de seguridad que registre el error y responda con un mensaje neutro.

## Referencias

- Baeldung — Error Handling for REST with Spring: <https://www.baeldung.com/exception-handling-for-rest-with-spring>
- Spring Framework — Error Responses: <https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html>
- Spring Framework — Exceptions (`@ExceptionHandler`): <https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html>
- RFC 9457 — Problem Details for HTTP APIs: <https://www.rfc-editor.org/rfc/rfc9457>
