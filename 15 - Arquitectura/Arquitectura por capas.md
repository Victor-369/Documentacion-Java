# Arquitectura por capas (Controller – Service – Repository)

## ¿Por qué organizar el código en capas?

Si toda la lógica de una aplicación (leer la petición HTTP, aplicar reglas de negocio, guardar en la base de datos) estuviera en una única clase, sería difícil de leer, de probar y de cambiar.

La **arquitectura por capas** separa el código según su **responsabilidad**. Cada capa se ocupa de una cosa y solo habla con la capa que tiene justo debajo.

```
          Petición HTTP (JSON)
                  │
                  ▼
   ┌───────────────────────────────┐
   │  Capa de presentación         │   @RestController
   │  (Controller)                 │   Recibe la petición y devuelve la respuesta
   └──────────────┬────────────────┘
                  │  DTO
                  ▼
   ┌───────────────────────────────┐
   │  Capa de negocio              │   @Service
   │  (Service)                    │   Reglas y lógica de la aplicación
   └──────────────┬────────────────┘
                  │  Entidad / modelo
                  ▼
   ┌───────────────────────────────┐
   │  Capa de persistencia         │   @Repository
   │  (Repository)                 │   Guarda y recupera datos
   └──────────────┬────────────────┘
                  │
                  ▼
           Base de datos
```

## Las tres capas

### Controller (capa de presentación o web)

Es la **puerta de entrada** desde el exterior. Sus responsabilidades:

- Asociar URLs y métodos HTTP a métodos Java (`@GetMapping`, `@PostMapping`…).
- Leer datos de la petición: ruta, parámetros, cabeceras, cuerpo.
- Validar el **formato** de la entrada.
- Llamar al servicio adecuado.
- Construir la respuesta HTTP: código de estado, cabeceras y cuerpo.

**No debe** contener reglas de negocio ni acceder directamente a la base de datos.

### Service (capa de negocio)

Es el **corazón** de la aplicación. Sus responsabilidades:

- Aplicar las **reglas de negocio** ("no se puede vender un producto sin stock", "un pedido de más de 50 € tiene envío gratis").
- Coordinar varias operaciones (por ejemplo, crear un pedido y descontar stock).
- Gestionar **transacciones** (que varias operaciones se hagan todas o ninguna).
- Lanzar excepciones de negocio cuando algo no es válido.

**No debe** saber nada de HTTP: ni códigos de estado, ni peticiones, ni respuestas. Así podría reutilizarse desde otra entrada (una tarea programada, un mensaje de una cola…).

### Repository (capa de persistencia o acceso a datos)

Es la que **habla con el almacenamiento**. Sus responsabilidades:

- Guardar, buscar, actualizar y borrar datos.
- Ocultar cómo se guardan realmente (base de datos relacional, NoSQL, ficheros, memoria…).

**No debe** contener reglas de negocio.

## Reglas de dependencia

1. Cada capa **solo llama a la capa inmediatamente inferior**: Controller → Service → Repository.
2. Las capas inferiores **no conocen** a las superiores. Un servicio nunca usa un controlador; un repositorio nunca usa un servicio.
3. **No te saltes capas.** Aunque una operación parezca trivial, el controlador no llama directamente al repositorio. Si hoy no hay lógica de negocio, mañana puede haberla.

```
   Controller ──▶ Service ──▶ Repository       ✔
   Controller ─────────────▶ Repository        ✘ (saltarse la capa de negocio)
   Repository ──▶ Service                      ✘ (dependencia hacia arriba)
```

## Modelos de datos: entidades y DTOs

Los datos que circulan entre capas no tienen por qué ser los mismos objetos.

| Tipo | Qué representa | Dónde se usa |
|---|---|---|
| **Entidad** o **modelo de dominio** | Cómo la aplicación representa y guarda la información internamente. | Service y Repository. |
| **DTO** (*Data Transfer Object*) | Los datos que entran o salen por la API. | Controller (y en la frontera con el Service). |

¿Por qué separarlos?

- **Seguridad:** la entidad `Usuario` puede tener el *hash* de la contraseña; el DTO de respuesta no debe incluirlo.
- **Independencia:** puedes cambiar cómo se guardan los datos sin cambiar el contrato de la API, y al revés.
- **Claridad:** la petición para crear un producto no lleva `id` (lo genera el servidor), pero la respuesta sí.

En Java 25, los `record` son perfectos para los DTO:

```java
// Lo que llega para crear un producto: sin id
public record CrearProductoRequest(String nombre, BigDecimal precio, int stock) {}

// Lo que se devuelve: con id
public record ProductoResponse(Long id, String nombre, BigDecimal precio, int stock) {}
```

> Para importes de dinero usa `BigDecimal` en lugar de `double`. `double` no representa con exactitud muchos decimales (por ejemplo, `0.1 + 0.2` da `0.30000000000000004`).

## Un recorrido completo, paso a paso

Supongamos la petición `GET /productos/42`.

**1. El controlador recibe la petición:**

```java
@RestController
@RequestMapping("/productos")
public class ProductoController {

    private final ProductoService servicio;

    public ProductoController(ProductoService servicio) {
        this.servicio = servicio;
    }

    @GetMapping("/{id}")
    public ProductoResponse obtener(@PathVariable Long id) {
        return servicio.buscarPorId(id);
    }
}
```

**2. El servicio aplica la lógica:**

```java
@Service
public class ProductoService {

    private final ProductoRepository repositorio;

    public ProductoService(ProductoRepository repositorio) {
        this.repositorio = repositorio;
    }

    public ProductoResponse buscarPorId(Long id) {
        Producto producto = repositorio.findById(id)
                .orElseThrow(() -> new ProductoNoEncontradoException(id));
        return new ProductoResponse(producto.id(), producto.nombre(),
                producto.precio(), producto.stock());
    }
}
```

**3. El repositorio busca el dato:**

```java
@Repository
public class ProductoRepository {

    private final Map<Long, Producto> almacen = new ConcurrentHashMap<>();

    public Optional<Producto> findById(Long id) {
        return Optional.ofNullable(almacen.get(id));
    }
}
```

**4. La respuesta vuelve hacia arriba:** el repositorio devuelve un `Optional<Producto>`, el servicio lo transforma en un `ProductoResponse` (o lanza una excepción si no existe) y el controlador lo devuelve; Spring lo convierte a JSON.

## Gestión de errores entre capas

El servicio **no sabe de HTTP**, así que no puede devolver un `404`. Lo que hace es **lanzar una excepción de negocio**:

```java
public class ProductoNoEncontradoException extends RuntimeException {
    public ProductoNoEncontradoException(Long id) {
        super("No existe el producto con id " + id);
    }
}
```

Y una clase de la capa web **traduce** esa excepción a una respuesta HTTP. Para ello Spring ofrece `@RestControllerAdvice` (un "manejador global" de excepciones para todos los controladores) junto con `@ExceptionHandler`:

```java
@RestControllerAdvice
public class ManejadorErrores {

    @ExceptionHandler(ProductoNoEncontradoException.class)
    public ProblemDetail noEncontrado(ProductoNoEncontradoException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }
}
```

`ProblemDetail` es la clase de Spring que representa el formato estándar de errores *Problem Details* (RFC 9457) visto en el documento [API REST](<../13-Web-y-APIs/Api rest.md>). La respuesta queda así:

```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "No existe el producto con id 42",
  "instance": "/productos/42"
}
```

## Organización de paquetes

Hay dos formas habituales de organizar los paquetes.

### Por capa (*package by layer*)

```
com.ejemplo.tienda
├── controller
│   ├── ProductoController.java
│   └── PedidoController.java
├── service
│   ├── ProductoService.java
│   └── PedidoService.java
├── repository
│   ├── ProductoRepository.java
│   └── PedidoRepository.java
├── model
│   ├── Producto.java
│   └── Pedido.java
└── dto
    ├── ProductoResponse.java
    └── PedidoResponse.java
```

Fácil de entender al principio. Pero cuando la aplicación crece, para tocar una funcionalidad hay que saltar entre muchas carpetas.

### Por funcionalidad (*package by feature*)

```
com.ejemplo.tienda
├── producto
│   ├── ProductoController.java
│   ├── ProductoService.java
│   ├── ProductoRepository.java
│   ├── Producto.java
│   └── dto
│       ├── CrearProductoRequest.java
│       └── ProductoResponse.java
├── pedido
│   ├── PedidoController.java
│   ├── PedidoService.java
│   └── ...
└── comun
    └── ManejadorErrores.java
```

Todo lo relacionado con una funcionalidad está junto. Escala mejor y encaja con la idea de microservicios (cada paquete podría convertirse en un servicio).

> Las dos son válidas. Lo importante es **elegir una y ser coherente**.

## El repositorio con Spring Data JPA

En los ejemplos de esta documentación el repositorio guarda los datos **en memoria** (en un `Map`), para centrarnos en la arquitectura sin necesitar una base de datos.

En una aplicación real, lo normal es usar una base de datos con **Spring Data JPA**. Con esta librería, basta con declarar una **interfaz**; Spring genera la implementación automáticamente:

```java
public interface ProductoRepository extends JpaRepository<Producto, Long> {

    // Spring genera la consulta a partir del nombre del método
    List<Producto> findByNombreContainingIgnoreCase(String texto);
}
```

`JpaRepository` ya incluye métodos como `findById`, `findAll`, `save`, `deleteById` o `existsById`. Lo importante es que **las capas superiores no cambian**: el servicio sigue llamando a `repositorio.findById(id)`, venga el dato de un `Map` o de una base de datos. Esa es precisamente la ventaja de separar en capas.

## Ventajas de la arquitectura por capas

| Ventaja | Explicación |
|---|---|
| **Separación de responsabilidades** | Cada clase tiene un único motivo para cambiar. |
| **Facilidad de prueba** | Se puede probar el servicio con un repositorio falso, o el controlador con un servicio falso. |
| **Mantenibilidad** | Es fácil saber dónde está cada cosa. |
| **Sustituibilidad** | Se puede cambiar la base de datos o la forma de exponer la API sin reescribir la lógica de negocio. |
| **Trabajo en equipo** | Distintas personas pueden trabajar en distintas capas con menos conflictos. |

## Errores frecuentes

- **Lógica de negocio en el controlador.** Si ves un `if` con reglas de negocio en un controlador, probablemente debe ir al servicio.
- **El servicio devuelve `ResponseEntity` o usa `HttpStatus`.** El servicio no debe saber de HTTP.
- **Devolver entidades directamente en la API.** Expones detalles internos y acoplas la API al almacenamiento. Usa DTOs.
- **Controlador que usa el repositorio directamente.** Te saltas la capa de negocio.
- **Servicios "anémicos" que solo reenvían llamadas.** Si el servicio no aporta nada, revisa si la lógica está en el sitio correcto.

## Resumen

- **Controller:** HTTP de entrada y salida. Sin reglas de negocio.
- **Service:** reglas de negocio. Sin HTTP.
- **Repository:** acceso a datos. Sin reglas de negocio.
- Las dependencias van siempre **hacia abajo**, sin saltarse capas.
- Usa **DTOs** (`record`) para la API y **entidades** para el interior.
- Los errores de negocio se lanzan como **excepciones** y se traducen a HTTP en un `@RestControllerAdvice`.

## Referencias

- GeeksforGeeks — Spring Boot Code Structure: <https://www.geeksforgeeks.org/java/spring-boot-code-structure/>
- GeeksforGeeks — Spring Boot Architecture: <https://www.geeksforgeeks.org/springboot/spring-boot-architecture/>
- CompileMyMind — Understanding Controller, Service and Repository in Spring Boot: <https://www.compilemymind.com/posts/spring-boot-layered-architecture/>
- Spring Framework — Error Responses (`ProblemDetail`): <https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html>
- Spring Data JPA: <https://spring.io/projects/spring-data-jpa>
