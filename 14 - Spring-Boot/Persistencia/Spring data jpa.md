# Persistencia relacional con Spring Data JPA

## Conceptos previos

### Bases de datos relacionales

Una base de datos relacional guarda la información en **tablas** con filas y columnas. Las tablas se relacionan entre sí mediante **claves**: cada fila tiene una **clave primaria** que la identifica, y otras tablas la referencian con **claves foráneas**. Se consultan con **SQL**. Ejemplos: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, H2.

### ORM

En Java trabajamos con **objetos**; la base de datos trabaja con **tablas**. Un **ORM** (*Object-Relational Mapping*, mapeo objeto-relacional) traduce entre ambos mundos: tú guardas un objeto y el ORM genera el `INSERT`; pides un objeto y el ORM ejecuta el `SELECT` y lo construye.

### JPA, Hibernate y Spring Data JPA

| Pieza | Qué es |
|---|---|
| **JPA / Jakarta Persistence** | La **especificación** estándar de Java para ORM: anotaciones (`@Entity`, `@Id`…) y la API (`EntityManager`). Solo define reglas. |
| **Hibernate** | La **implementación** más usada de JPA. Es quien realmente genera y ejecuta el SQL. Spring Boot la usa por defecto. |
| **Spring Data JPA** | Capa de Spring **por encima** de JPA que aporta los repositorios automáticos del documento [Spring Data: un motor para unificar el acceso a datos](<Spring data.md>). |

```
  Tu código  ──▶  Spring Data JPA  ──▶  JPA (EntityManager)  ──▶  Hibernate  ──▶  JDBC  ──▶  Base de datos
 (repositorio)    (implementa la         (API estándar)          (genera SQL)
                   interfaz)
```

## Dependencias

En Spring Initializr añade **Spring Data JPA** y el *driver* de tu base de datos (por ejemplo, **PostgreSQL Driver**, o **H2 Database** para una base de datos en memoria):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Driver de PostgreSQL -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Pruebas de la capa JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
```

## Configuración de la conexión

`application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/tienda
spring.datasource.username=tienda
spring.datasource.password=tienda

# Mostrar en consola el SQL que genera Hibernate (útil mientras aprendes)
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

> **Nunca** subas contraseñas reales al repositorio de código. En entornos reales se pasan mediante variables de entorno, por ejemplo `SPRING_DATASOURCE_PASSWORD`, que Spring Boot lee automáticamente.

### Levantar PostgreSQL con Docker

La forma más cómoda de tener PostgreSQL en desarrollo es con Docker. Crea un fichero `compose.yaml` en la raíz del proyecto:

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: tienda
      POSTGRES_USER: tienda
      POSTGRES_PASSWORD: tienda
    ports:
      - "5432:5432"
```

```bash
docker compose up -d
```

Si añades la dependencia `spring-boot-docker-compose` (en Initializr: **Docker Compose Support**), Spring Boot **arranca el `compose.yaml` automáticamente** al ejecutar la aplicación y configura la conexión por ti, sin necesidad de escribir las propiedades `spring.datasource.*`. Docker se explica en el documento [Docker y *multi-stage build*](<../../12 - Herramientas/Docker.md>).

## Tu primera entidad

Una **entidad** es una clase Java que se corresponde con una tabla.

```java
package com.ejemplo.tienda.producto;

import java.math.BigDecimal;
import java.time.Instant;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "productos")
public class Producto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 100)
    private String nombre;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;

    @Column(nullable = false)
    private int stock;

    @Column(name = "fecha_alta", nullable = false, updatable = false)
    private Instant fechaAlta;

    // JPA necesita un constructor sin argumentos. "protected" evita que se use por error.
    protected Producto() {
    }

    public Producto(String nombre, BigDecimal precio, int stock) {
        this.nombre = nombre;
        this.precio = precio;
        this.stock = stock;
        this.fechaAlta = Instant.now();
    }

    // Métodos de negocio en lugar de setters sueltos
    public void cambiarPrecio(BigDecimal nuevoPrecio) {
        this.precio = nuevoPrecio;
    }

    public void retirarStock(int unidades) {
        if (unidades > stock) {
            throw new IllegalStateException("Stock insuficiente para " + nombre);
        }
        this.stock -= unidades;
    }

    public Long getId() { return id; }
    public String getNombre() { return nombre; }
    public BigDecimal getPrecio() { return precio; }
    public int getStock() { return stock; }
    public Instant getFechaAlta() { return fechaAlta; }
}
```

### Anotaciones básicas

| Anotación | Qué hace |
|---|---|
| `@Entity` | Marca la clase como entidad JPA. |
| `@Table(name = "…")` | Nombre de la tabla. Si no se pone, se deriva del nombre de la clase. |
| `@Id` | Marca la clave primaria. Obligatoria. |
| `@GeneratedValue` | El valor del id lo genera la base de datos. |
| `@Column` | Ajusta la columna: nombre, `nullable`, `unique`, `length`, `precision`/`scale`, `updatable`… |
| `@Enumerated(EnumType.STRING)` | Guarda un `enum` por su nombre (`"PAGADO"`) en lugar de por su posición. **Usa siempre `STRING`**: si guardas la posición y luego reordenas el `enum`, los datos dejan de cuadrar. |
| `@Transient` | Campo que **no** se guarda en la base de datos. |
| `@Version` | Control de concurrencia optimista (ver «Concurrencia optimista con `@Version`»). |

### Estrategias de generación de id

| Estrategia | Funcionamiento |
|---|---|
| `IDENTITY` | La base de datos asigna el id al insertar (columna autoincremental). Sencilla y muy común. |
| `SEQUENCE` | Usa una secuencia de la base de datos. Permite a Hibernate optimizar inserciones en lote. |
| `UUID` | Genera un identificador UUID. Útil si no quieres ids predecibles o si se generan en varios sistemas. |
| `AUTO` | Deja que el proveedor elija. |

### Requisitos de una entidad

- Debe tener un **constructor sin argumentos** (puede ser `protected`).
- **No** puede ser `final`, ni sus métodos persistentes.
- Debe tener un campo `@Id`.
- Por eso **un `record` no puede ser una entidad JPA** (es `final` e inmutable). Los `record` se usan para los DTO y las proyecciones.

## Creación del esquema

¿Quién crea la tabla `productos`? Hay dos opciones.

### Opción 1: que Hibernate la genere (`ddl-auto`)

```properties
spring.jpa.hibernate.ddl-auto=update
```

| Valor | Comportamiento |
|---|---|
| `none` | No toca el esquema. |
| `validate` | Comprueba que el esquema coincide con las entidades; si no, la aplicación no arranca. |
| `update` | Crea o modifica tablas y columnas para que coincidan (nunca borra). |
| `create` | Borra y crea el esquema al arrancar. |
| `create-drop` | Como `create`, y además lo borra al parar. |

Spring Boot usa por defecto `create-drop` con bases de datos embebidas (como H2) y `none` con las demás.

> `update` es cómodo para aprender, pero **no se usa en producción**: no controla bien los cambios, no puede renombrar columnas sin perder datos y no queda registro de qué ha cambiado.

### Opción 2: migraciones con Flyway o Liquibase (la profesional)

Las **migraciones** son scripts SQL versionados que se aplican en orden y quedan registrados en la base de datos. Así, todos los entornos tienen exactamente el mismo esquema.

Con **Flyway** (en Spring Boot 4 se añade el *starter* `spring-boot-starter-flyway` y el módulo de tu base de datos, por ejemplo `flyway-database-postgresql`):

```
src/main/resources/db/migration/
├── V1__crear_tabla_productos.sql
└── V2__anadir_columna_categoria.sql
```

```sql
-- V1__crear_tabla_productos.sql
CREATE TABLE productos (
    id          BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    nombre      VARCHAR(100)   NOT NULL UNIQUE,
    precio      NUMERIC(10, 2) NOT NULL,
    stock       INTEGER        NOT NULL,
    fecha_alta  TIMESTAMP WITH TIME ZONE NOT NULL
);
```

Y en `application.properties`:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

Flyway aplica los scripts pendientes al arrancar; Hibernate solo **valida** que las entidades coinciden con el esquema.

## El repositorio

```java
package com.ejemplo.tienda.producto;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductoRepository extends JpaRepository<Producto, Long> {

    Optional<Producto> findByNombreIgnoreCase(String nombre);

    List<Producto> findByPrecioLessThanOrderByPrecioAsc(BigDecimal precioMaximo);

    boolean existsByNombreIgnoreCase(String nombre);
}
```

## Consultas personalizadas con `@Query`

Cuando una consulta derivada se queda corta o su nombre es demasiado largo, escríbela tú.

### JPQL

**JPQL** (*Jakarta Persistence Query Language*) se parece a SQL, pero trabaja con **entidades y sus atributos**, no con tablas y columnas:

```java
@Query("""
        SELECT p FROM Producto p
        WHERE p.stock < :minimo
        ORDER BY p.stock ASC
        """)
List<Producto> buscarConPocoStock(@Param("minimo") int minimo);
```

- `Producto` es el nombre de la **clase**, no de la tabla.
- `:minimo` es un **parámetro con nombre**; `@Param` lo enlaza con el argumento.
- Los bloques de texto `"""` hacen las consultas largas mucho más legibles.

### SQL nativo

Si necesitas algo específico de tu base de datos:

```java
// ILIKE es específico de PostgreSQL: búsqueda sin distinguir mayúsculas
@Query(value = "SELECT * FROM productos WHERE nombre ILIKE CONCAT('%', :texto, '%')", nativeQuery = true)
List<Producto> buscarPorTextoNativo(@Param("texto") String texto);
```

> Usa SQL nativo solo cuando JPQL no sea suficiente: te ata a una base de datos concreta.

### Consultas que modifican datos

```java
@Modifying
@Query("UPDATE Producto p SET p.precio = p.precio * :factor WHERE p.stock = 0")
int ajustarPrecioSinStock(@Param("factor") BigDecimal factor);
```

`@Modifying` indica que no es un `SELECT`. Debe ejecutarse dentro de una transacción (siguiente apartado). Devuelve el número de filas afectadas.

### Proyecciones con JPQL

```java
public record ProductoResumen(Long id, String nombre, BigDecimal precio) {}

@Query("SELECT new com.ejemplo.tienda.producto.ProductoResumen(p.id, p.nombre, p.precio) FROM Producto p")
List<ProductoResumen> listarResumen();
```

## Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **todas o ninguna**. Si a mitad de un pedido falla el descuento de stock, no debe quedar un pedido a medias.

En Spring se declaran con `@Transactional`, normalmente en la **capa de servicio**:

```java
package com.ejemplo.tienda.producto;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class ProductoService {

    private final ProductoRepository repositorio;

    public ProductoService(ProductoRepository repositorio) {
        this.repositorio = repositorio;
    }

    @Transactional(readOnly = true)
    public Producto obtener(Long id) {
        return repositorio.findById(id)
                .orElseThrow(() -> new ProductoNoEncontradoException(id));
    }

    @Transactional
    public void cambiarPrecio(Long id, java.math.BigDecimal nuevoPrecio) {
        Producto producto = obtener(id);
        producto.cambiarPrecio(nuevoPrecio);
        // No hace falta llamar a save(): ver "dirty checking" más abajo
    }
}
```

Reglas importantes:

- Si dentro del método se lanza una **excepción no comprobada** (`RuntimeException`), la transacción se **deshace** (*rollback*). Con las excepciones comprobadas, por defecto **no**.
- `readOnly = true` indica que solo se lee; permite optimizaciones.
- `@Transactional` funciona porque Spring envuelve el *bean* en un *proxy*. Por eso **no tiene efecto si llamas al método desde otro método de la misma clase**, ni en métodos `private`.
- Los métodos de los repositorios de Spring Data ya son transaccionales por sí mismos. `@Transactional` en el servicio sirve para **agrupar varias operaciones**.

### *Dirty checking*: guardado automático

Dentro de una transacción, las entidades que cargas están **gestionadas** (*managed*) por el **contexto de persistencia**. Hibernate recuerda su estado y, al terminar la transacción, **detecta los cambios y ejecuta el `UPDATE` automáticamente**. Por eso, en `cambiarPrecio`, no hace falta llamar a `save()`.

### Estados de una entidad

| Estado | Significado |
|---|---|
| **Nueva** (*transient*) | Creada con `new`, Hibernate no la conoce. |
| **Gestionada** (*managed*) | Asociada al contexto de persistencia: sus cambios se guardan solos. |
| **Separada** (*detached*) | Lo estuvo, pero la transacción terminó. Sus cambios ya no se guardan solos. |
| **Eliminada** (*removed*) | Marcada para borrar al confirmar la transacción. |

## Auditoría: fechas de creación y modificación

Spring Data JPA puede rellenar automáticamente quién y cuándo creó o modificó una entidad.

1. Activa la auditoría en una clase de configuración:

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

2. Anota la entidad:

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Producto {

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private Instant creadoEn;

    @LastModifiedDate
    private Instant modificadoEn;

    // ...
}
```

Para `@CreatedBy` y `@LastModifiedBy` (el usuario) hace falta además un *bean* `AuditorAware` que indique quién es el usuario actual.

## Concurrencia optimista con `@Version`

Si dos personas editan el mismo producto a la vez, la segunda podría sobrescribir sin saberlo los cambios de la primera. Con un campo `@Version`:

```java
@Version
private long version;
```

Hibernate incrementa la versión en cada `UPDATE` y comprueba que no ha cambiado desde que se leyó. Si ha cambiado, lanza una excepción de bloqueo optimista (`ObjectOptimisticLockingFailureException` en Spring), que la API puede traducir a `409 Conflict`.

## Paginación en JPA

```java
Page<Producto> findByStockGreaterThan(int stock, Pageable paginacion);
```

Spring Data genera dos consultas: una para la página (`LIMIT`/`OFFSET`) y otra `COUNT` para el total. Si no necesitas el total, devuelve `Slice<Producto>` y te ahorras el `COUNT`.

## Buenas prácticas

- Pon `@Transactional` en el **servicio**, no en el controlador.
- No expongas las entidades en la API: usa **DTOs** ([DTOs, validación y diseño de entrada/salida](<../APIs-REST/DTOs y validacion.md>)).
- Usa `ddl-auto=validate` + **migraciones** fuera de la fase de aprendizaje.
- Usa `BigDecimal` para dinero e `Instant`/`OffsetDateTime` para momentos en el tiempo.
- Guarda los `enum` con `EnumType.STRING`.
- Activa `spring.jpa.show-sql=true` en desarrollo para ver qué SQL se ejecuta realmente.
- Cuidado con las relaciones y la carga perezosa: lo verás a fondo en el documento [Relaciones y modelos de datos](<Relaciones y modelado.md>).

## Resumen

- **JPA** es la especificación, **Hibernate** la implementación y **Spring Data JPA** los repositorios automáticos.
- Una **entidad** es una clase con `@Entity` e `@Id` y un constructor sin argumentos.
- Crea el esquema con **migraciones** (Flyway/Liquibase) y valida con `ddl-auto=validate`.
- Usa consultas derivadas, `@Query` con JPQL o, si no hay más remedio, SQL nativo.
- `@Transactional` agrupa operaciones; dentro de una transacción los cambios se guardan solos (*dirty checking*).

## Referencias

- Spring Data JPA — Reference Documentation: <https://docs.spring.io/spring-data/jpa/reference/>
- Spring — Accessing Data with JPA: <https://spring.io/guides/gs/accessing-data-jpa/>
- Baeldung — Spring Data JPA tutorial: <https://www.baeldung.com/spring-data-jpa-tutorial>
- Spring Boot — SQL Databases: <https://docs.spring.io/spring-boot/reference/data/sql.html>
- Spring Boot — Database initialization (Flyway/Liquibase): <https://docs.spring.io/spring-boot/how-to/data-initialization.html>
- Documentación de Hibernate ORM: <https://hibernate.org/orm/documentation/>
