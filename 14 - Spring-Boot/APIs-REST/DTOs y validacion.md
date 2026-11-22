# DTOs, validación y diseño de entrada/salida

## Parte A — El patrón DTO

## ¿Qué es un DTO?

Un **DTO** (*Data Transfer Object*, objeto de transferencia de datos) es un objeto que **solo transporta datos** entre dos partes de un sistema, sin lógica de negocio. El patrón lo describió Martin Fowler en *Patterns of Enterprise Application Architecture*; su objetivo original era agrupar en una sola llamada datos que, si no, requerirían varias llamadas remotas.

En una API REST, los DTO definen **el contrato** con el exterior: qué datos se aceptan en cada petición y qué datos se devuelven en cada respuesta.

```
  Cliente HTTP                       API                                Base de datos
      │       JSON       ┌────────────┐    ┌─────────┐    ┌────────────┐
      │ ───────────────▶ │ Request    │──▶ │ Service │──▶ │  Entidad   │ ──▶ tablas
      │                  │  DTO       │    │         │    │            │
      │ ◀─────────────── │ Response   │◀── │         │◀── │            │
      │       JSON       │  DTO       │    └─────────┘    └────────────┘
                         └────────────┘
```

## ¿Por qué no devolver directamente las entidades?

| Problema | Ejemplo |
|---|---|
| **Exponer datos sensibles** | La entidad `Usuario` tiene `passwordHash`, y acaba en el JSON. |
| **Aceptar datos que no deberían llegar** | El cliente envía `"rol": "ADMIN"` o `"id": 5` y se asigna a la entidad (*mass assignment*). |
| **Acoplar la API a la base de datos** | Renombrar una columna rompe a los clientes de la API. |
| **Problemas con relaciones** | Bucles infinitos, consultas ocultas y `LazyInitializationException` (ver [Relaciones y modelos de datos](<../Persistencia/Relaciones y modelado.md>)). |
| **Un único modelo para todo** | Crear, actualizar y consultar suelen necesitar datos distintos. |

## Diseñar los DTO

En Java 25, los DTO se escriben como **`record`**: inmutables, concisos y con soporte directo en Jackson y en Bean Validation.

Una buena práctica es tener **un DTO por caso de uso**, no un único DTO "para todo":

```java
// Entrada para crear: sin id, sin fechas (los pone el servidor)
public record CrearClienteRequest(String nombre, String email, String telefono) {}

// Entrada para modificar parcialmente: todos los campos opcionales
public record ActualizarClienteRequest(String nombre, String telefono) {}

// Salida detallada
public record ClienteResponse(Long id, String nombre, String email, String telefono, Instant creadoEn) {}

// Salida resumida, para listados
public record ClienteResumen(Long id, String nombre) {}
```

### Convenciones de nombres habituales

| Sufijo | Uso |
|---|---|
| `…Request` | Datos de entrada de una petición. |
| `…Response` | Datos de salida. |
| `…Resumen` / `…Summary` | Versión reducida para listados. |

Lo importante es que el equipo use **siempre las mismas**.

## Conversión entre entidades y DTO (*mapping*)

### Opción 1: métodos estáticos en el DTO

Sencilla y explícita, ideal para empezar:

```java
public record ClienteResponse(Long id, String nombre, String email, String telefono, Instant creadoEn) {

    public static ClienteResponse desde(Cliente cliente) {
        return new ClienteResponse(
                cliente.getId(),
                cliente.getNombre(),
                cliente.getEmail(),
                cliente.getTelefono(),
                cliente.getCreadoEn());
    }
}
```

### Opción 2: una clase *mapper* dedicada

Cuando la conversión es más compleja o necesita otros servicios:

```java
@Component
public class PedidoMapper {

    public PedidoResponse aResponse(Pedido pedido) {
        var lineas = pedido.getLineas().stream()
                .map(l -> new PedidoResponse.LineaResponse(
                        l.getProducto().getNombre(), l.getCantidad(), l.subtotal()))
                .toList();

        return new PedidoResponse(
                pedido.getId(),
                pedido.getEstado().name(),
                new PedidoResponse.ClienteResumen(pedido.getCliente().getId(), pedido.getCliente().getNombre()),
                lineas,
                pedido.total());
    }
}
```

### Opción 3: librerías de *mapping*

Librerías como **MapStruct** generan el código de conversión en tiempo de compilación a partir de una interfaz. Son útiles cuando hay muchos DTO con conversiones repetitivas. Escribir los *mappers* a mano tiene la ventaja de que todo se ve y se depura fácilmente.

### ¿Dónde se convierte?

Lo habitual es que el **servicio** reciba DTO de entrada y devuelva DTO de salida, haciendo la conversión **dentro de la transacción**. Así las relaciones `LAZY` se pueden cargar sin errores y el controlador nunca ve entidades.

## Respuestas paginadas

En lugar de devolver directamente el `Page` de Spring Data, define tu propio contrato:

```java
public record PaginaResponse<T>(
        List<T> contenido,
        int pagina,
        int tamano,
        long totalElementos,
        int totalPaginas) {

    public static <E, T> PaginaResponse<T> desde(Page<E> pagina, Function<E, T> conversor) {
        return new PaginaResponse<>(
                pagina.getContent().stream().map(conversor).toList(),
                pagina.getNumber(),
                pagina.getSize(),
                pagina.getTotalElements(),
                pagina.getTotalPages());
    }
}
```

```java
@Transactional(readOnly = true)
public PaginaResponse<ClienteResumen> listar(Pageable paginacion) {
    return PaginaResponse.desde(
            repositorio.findAll(paginacion),
            c -> new ClienteResumen(c.getId(), c.getNombre()));
}
```

## Parte B — Validación

## ¿Qué validar y dónde?

| Tipo de validación | Ejemplo | Dónde |
|---|---|---|
| **Sintáctica / de formato** | El email tiene formato de email; el nombre no está vacío; la cantidad es positiva. | En el **DTO de entrada**, con Bean Validation. |
| **Semántica / de negocio** | El email no está ya registrado; hay stock suficiente; el pedido no está ya enviado. | En el **servicio** (o en el modelo de dominio). |
| **De integridad** | La clave única no se repite; la clave foránea existe. | En la **base de datos** (restricciones), como última barrera. |

> No confíes nunca en que el cliente (una web, una app) haya validado: **la API debe validar siempre**.

## Bean Validation

**Jakarta Bean Validation** es la especificación estándar para validar objetos con anotaciones. Su implementación de referencia es **Hibernate Validator**, que Spring Boot incluye con el *starter*:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### Restricciones más usadas (paquete `jakarta.validation.constraints`)

| Anotación | Comprueba que el valor… |
|---|---|
| `@NotNull` | no es `null`. |
| `@NotEmpty` | no es `null` ni vacío (cadenas, colecciones). |
| `@NotBlank` | no es `null`, ni vacío, ni solo espacios (cadenas). |
| `@Size(min, max)` | tiene una longitud o tamaño dentro del rango. |
| `@Min`, `@Max` | es un número mayor/menor o igual que el límite. |
| `@Positive`, `@PositiveOrZero` | es mayor que 0 / mayor o igual que 0. |
| `@Negative`, `@NegativeOrZero` | es menor que 0 / menor o igual que 0. |
| `@DecimalMin`, `@DecimalMax` | igual que `@Min`/`@Max`, admitiendo decimales como texto (`"0.01"`). |
| `@Digits(integer, fraction)` | tiene como máximo ese número de dígitos enteros y decimales. |
| `@Email` | tiene formato de email. |
| `@Pattern(regexp)` | cumple una expresión regular. |
| `@Past`, `@PastOrPresent` | es una fecha pasada (o presente). |
| `@Future`, `@FutureOrPresent` | es una fecha futura (o presente). |
| `@AssertTrue`, `@AssertFalse` | es `true` / `false`. |

> Casi todas las restricciones consideran **válido** el valor `null` (salvo `@NotNull`, `@NotEmpty` y `@NotBlank`). Si un campo es obligatorio **y** tiene formato, combina ambas: `@NotBlank @Email String email`.

### Un DTO validado

```java
package com.ejemplo.tienda.cliente.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;
import jakarta.validation.constraints.Size;

public record CrearClienteRequest(

        @NotBlank(message = "El nombre es obligatorio")
        @Size(max = 100, message = "El nombre no puede superar {max} caracteres")
        String nombre,

        @NotBlank(message = "El email es obligatorio")
        @Email(message = "El email no tiene un formato válido")
        String email,

        @Pattern(regexp = "^\\+?[0-9]{9,15}$", message = "El teléfono debe tener entre 9 y 15 dígitos")
        String telefono   // opcional: si llega null, @Pattern lo da por válido
) {}
```

En los mensajes puedes usar los atributos de la restricción entre llaves: `{max}`, `{min}`, `{value}`.

### Activar la validación en el controlador

```java
@PostMapping
public ResponseEntity<ClienteResponse> crear(@Valid @RequestBody CrearClienteRequest peticion) {
    ...
}
```

- `@Valid` (de `jakarta.validation`) hace que Spring valide el objeto **antes** de ejecutar el método.
- Si algo falla, el método no se ejecuta y Spring lanza `MethodArgumentNotValidException`, que por defecto se traduce en `400 Bad Request`. En el documento [Gestión profesional de errores](<Gestion de errores.md>) verás cómo devolver un error claro.

### Validar objetos anidados y listas

`@Valid` también se usa **dentro** de un DTO para validar objetos y elementos de listas:

```java
public record CrearPedidoRequest(

        @NotNull Long clienteId,

        @NotEmpty(message = "El pedido debe tener al menos una línea")
        @Size(max = 50, message = "Un pedido no puede tener más de {max} líneas")
        List<@Valid LineaRequest> lineas
) {
    public record LineaRequest(
            @NotNull Long productoId,
            @Positive(message = "La cantidad debe ser mayor que cero") int cantidad) {}
}
```

Sin el `@Valid` dentro de `List<@Valid LineaRequest>`, las restricciones de cada línea **no se comprobarían**.

### Validar parámetros de ruta y de consulta

Spring Framework aplica también la validación a parámetros simples de los métodos del controlador cuando llevan restricciones:

```java
@GetMapping
public PaginaResponse<ClienteResumen> listar(
        @RequestParam(defaultValue = "0") @PositiveOrZero int pagina,
        @RequestParam(defaultValue = "20") @Min(1) @Max(100) int tamano) {
    ...
}
```

Si fallan, Spring lanza `HandlerMethodValidationException` (también se traduce en `400`).

## Restricciones personalizadas

Cuando ninguna restricción estándar encaja, puedes crear la tuya. Ejemplo: validar un código de producto con formato `ABC-1234`.

**1. La anotación:**

```java
package com.ejemplo.tienda.comun.validacion;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.RECORD_COMPONENT})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = CodigoProductoValidator.class)
public @interface CodigoProducto {

    String message() default "El código debe tener el formato ABC-1234";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

**2. El validador:**

```java
package com.ejemplo.tienda.comun.validacion;

import java.util.regex.Pattern;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class CodigoProductoValidator implements ConstraintValidator<CodigoProducto, String> {

    private static final Pattern FORMATO = Pattern.compile("^[A-Z]{3}-\\d{4}$");

    @Override
    public boolean isValid(String valor, ConstraintValidatorContext contexto) {
        // null se considera válido: combínalo con @NotBlank si es obligatorio
        return valor == null || FORMATO.matcher(valor).matches();
    }
}
```

**3. Uso:**

```java
public record CrearProductoRequest(@NotBlank @CodigoProducto String codigo, ...) {}
```

### Validaciones entre varios campos

Para reglas que implican varios campos (por ejemplo, "la fecha de fin debe ser posterior a la de inicio"), una forma sencilla con `record` es un método con `@AssertTrue`:

```java
public record CrearPromocionRequest(
        @NotNull LocalDate inicio,
        @NotNull LocalDate fin) {

    @AssertTrue(message = "La fecha de fin debe ser posterior a la de inicio")
    public boolean isRangoValido() {
        return inicio == null || fin == null || fin.isAfter(inicio);
    }
}
```

> El método debe empezar por `is` (o `get`) para que Bean Validation lo trate como una propiedad.

## Grupos de validación

A veces un mismo DTO se valida de forma distinta según la operación. Los **grupos** permiten activar unas restricciones u otras:

```java
public interface AlCrear {}
public interface AlActualizar {}

public record ProductoRequest(
        @Null(groups = AlCrear.class, message = "No envíes el id al crear")
        @NotNull(groups = AlActualizar.class)
        Long id,

        @NotBlank(groups = {AlCrear.class, AlActualizar.class})
        String nombre) {}
```

En el controlador se usa `@Validated` (de Spring) indicando el grupo:

```java
@PostMapping
public ProductoResponse crear(@Validated(AlCrear.class) @RequestBody ProductoRequest peticion) { ... }
```

> Los grupos añaden complejidad. Muchas veces es más claro tener **dos DTO distintos** (`CrearProductoRequest` y `ActualizarProductoRequest`).

## Validación en la capa de servicio

Bean Validation también puede validar los parámetros de cualquier *bean* de Spring. Anota la clase con `@Validated` y los parámetros con las restricciones:

```java
@Service
@Validated
public class ClienteService {

    public ClienteResponse crear(@Valid CrearClienteRequest peticion) { ... }

    public ClienteResponse obtener(@Positive Long id) { ... }
}
```

Si una restricción falla, se lanza `ConstraintViolationException`. Es útil cuando el servicio se llama desde varios sitios, no solo desde un controlador.

## Validación de negocio en el servicio

Las reglas que dependen del estado del sistema no se pueden expresar con anotaciones. Van en el servicio y se comunican con **excepciones de negocio**:

```java
@Transactional
public ClienteResponse crear(CrearClienteRequest peticion) {
    if (repositorio.existsByEmailIgnoreCase(peticion.email())) {
        throw new EmailYaRegistradoException(peticion.email());
    }
    var cliente = new Cliente(peticion.nombre(), peticion.email().toLowerCase(), peticion.telefono());
    return ClienteResponse.desde(repositorio.save(cliente));
}
```

## Buenas prácticas de diseño de entrada/salida

1. **Un DTO por caso de uso**, en lugar de uno genérico.
2. **Nunca** recibas ni devuelvas entidades en los controladores.
3. Los DTO de **entrada** no llevan los campos que asigna el servidor (`id`, fechas de creación, estado inicial).
4. Valida **formato** en el DTO y **negocio** en el servicio.
5. Normaliza los datos de entrada en un único sitio (recortar espacios, emails en minúsculas…).
6. Usa tipos adecuados: `BigDecimal` para dinero, `Instant`/`LocalDate` para fechas, `enum` para valores cerrados.
7. Escribe **mensajes de error claros** y pensados para quien usa la API.
8. Mantén el contrato **estable**: añadir campos opcionales no rompe a los clientes; renombrar o eliminar sí.

## Resumen

- Los **DTO** definen el contrato de la API y protegen el modelo interno. Escríbelos como `record`.
- Convierte entidades ↔ DTO en el **servicio**, dentro de la transacción.
- **Bean Validation** valida el formato con anotaciones; activa la validación con `@Valid`.
- Usa `@Valid` dentro de los DTO para validar objetos anidados y listas.
- Crea **restricciones personalizadas** cuando las estándar no basten.
- Las **reglas de negocio** se validan en el servicio y se comunican con excepciones.

## Referencias

- Baeldung — Validation in Spring Boot: <https://www.baeldung.com/spring-boot-bean-validation>
- Baeldung — The DTO Pattern: <https://www.baeldung.com/java-dto-pattern>
- Spring Boot — Validation: <https://docs.spring.io/spring-boot/reference/io/validation.html>
- Spring Framework — Validation (incluida la validación de métodos): <https://docs.spring.io/spring-framework/reference/core/validation/beanvalidation.html>
- Jakarta Bean Validation: <https://beanvalidation.org/>
- Martin Fowler — Data Transfer Object: <https://martinfowler.com/eaaCatalog/dataTransferObject.html>
