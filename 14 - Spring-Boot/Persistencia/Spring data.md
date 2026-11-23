# Spring Data: un motor para unificar el acceso a datos

## El problema

Cada tecnología de almacenamiento tiene su propia forma de trabajar:

- Con una base de datos relacional escribes SQL, abres conexiones, recorres resultados y los conviertes en objetos.
- Con MongoDB usas su propio *driver*, construyes documentos BSON y los conviertes en objetos.
- Con Redis, Elasticsearch o Cassandra, cada una tiene su API.

Gran parte de ese código es repetitivo: guardar, buscar por id, listar, borrar, paginar… es casi igual en todas.

## ¿Qué es Spring Data?

**Spring Data** es un proyecto paraguas de Spring que ofrece **un modelo de programación común** para acceder a datos, sea cual sea el almacenamiento, sin perder las características propias de cada uno.

Está formado por módulos:

| Módulo | Almacenamiento |
|---|---|
| **Spring Data JPA** | Bases de datos relacionales a través de JPA (Hibernate) |
| **Spring Data JDBC** | Bases de datos relacionales con un enfoque más simple, sin JPA |
| **Spring Data MongoDB** | MongoDB (documentos) |
| **Spring Data Redis** | Redis (clave-valor) |
| **Spring Data Elasticsearch** | Elasticsearch (búsqueda) |
| **Spring Data Cassandra**, **Neo4j**, **Couchbase**, **R2DBC**… | Otras tecnologías |
| **Spring Data REST** | Expone automáticamente los repositorios como una API REST |

La pieza central, común a todos los módulos, es la **abstracción de repositorio**.

## Repositorios: declaras una interfaz, Spring escribe la clase

Con Spring Data no implementas el repositorio. **Declaras una interfaz** y Spring genera la implementación al arrancar la aplicación:

```java
public interface ProductoRepository extends ListCrudRepository<Producto, Long> {
}
```

Los dos parámetros de tipo son:

1. El tipo de la **entidad** o **documento** que gestiona (`Producto`).
2. El tipo de su **identificador** (`Long`).

Solo con eso ya tienes `save`, `findById`, `findAll`, `deleteById`, `count`, `existsById`… Y la misma interfaz sirve, con mínimos cambios, para JPA o para MongoDB.

## La jerarquía de interfaces

```
Repository<T, ID>                          ← interfaz "marcador", sin métodos
  └── CrudRepository<T, ID>                ← CRUD básico (devuelve Iterable)
        ├── ListCrudRepository<T, ID>      ← igual, pero devuelve List
        └── PagingAndSortingRepository<T, ID> (*)
              └── ListPagingAndSortingRepository<T, ID>

Específicas de cada módulo:
  JpaRepository<T, ID>       ← CRUD + paginación + extras de JPA (flush, lotes…)
  MongoRepository<T, ID>     ← CRUD + paginación + extras de MongoDB (insert…)
```

(*) `PagingAndSortingRepository` añade `findAll(Sort)` y `findAll(Pageable)`. No incluye los métodos CRUD por sí misma, por eso se suele combinar con `CrudRepository` o usar directamente las interfaces específicas.

| Interfaz | Cuándo usarla |
|---|---|
| `ListCrudRepository` | Solo necesitas CRUD y quieres trabajar con `List`. |
| `JpaRepository` | Usas JPA y quieres todo: CRUD, paginación, ordenación y extras. Es la más habitual. |
| `MongoRepository` | Usas MongoDB y quieres todo. |
| `Repository` | Quieres exponer **solo** los métodos que tú declares (control total). |

### Exponer solo lo necesario

Si un repositorio no debe permitir borrar, no heredes de `CrudRepository`: hereda de `Repository` y declara solo lo que quieras:

```java
public interface FacturaRepository extends Repository<Factura, Long> {

    Factura save(Factura factura);

    Optional<Factura> findById(Long id);

    List<Factura> findAll();
}
```

Spring implementa esos métodos igual que si vinieran heredados.

## Métodos CRUD más usados

| Método | Qué hace |
|---|---|
| `save(entidad)` | Inserta si es nueva, actualiza si ya existe. Devuelve la entidad guardada (úsala: puede tener el id asignado). |
| `saveAll(lista)` | Guarda varias. |
| `findById(id)` | Devuelve un `Optional<T>`. |
| `findAll()` | Devuelve todos (¡cuidado con tablas grandes!). |
| `findAllById(ids)` | Devuelve los que coinciden con una lista de ids. |
| `existsById(id)` | `true` si existe. |
| `count()` | Número total. |
| `deleteById(id)` | Borra por id. |
| `delete(entidad)` | Borra una entidad concreta. |

## Consultas derivadas del nombre del método

Spring Data puede **generar la consulta a partir del nombre del método**. Analiza el nombre, identifica las propiedades de la entidad y construye la consulta (SQL en JPA, consulta de MongoDB en MongoDB).

```java
public interface ProductoRepository extends JpaRepository<Producto, Long> {

    List<Producto> findByCategoria(String categoria);

    Optional<Producto> findByNombreIgnoreCase(String nombre);

    List<Producto> findByPrecioLessThan(BigDecimal precio);

    List<Producto> findByCategoriaAndPrecioBetween(String categoria, BigDecimal min, BigDecimal max);

    List<Producto> findByNombreContainingIgnoreCaseOrderByPrecioAsc(String texto);

    List<Producto> findTop5ByOrderByFechaAltaDesc();

    boolean existsByNombreIgnoreCase(String nombre);

    long countByCategoria(String categoria);

    void deleteByCategoria(String categoria);
}
```

### Cómo se construye el nombre

```
find      By   Categoria   And   Precio   Between
└─┬─┘     └┬┘  └───┬───┘   └┬┘   └──┬─┘   └──┬──┘
prefijo   inicio propiedad  unión propiedad  operador
```

| Parte | Opciones habituales |
|---|---|
| Prefijo | `find…By`, `read…By`, `get…By`, `query…By`, `search…By`, `stream…By`, `count…By`, `exists…By`, `delete…By` |
| Limitar resultados | `First`, `Top`, `Top3`, `Distinct` (entre el prefijo y `By`) |
| Unir condiciones | `And`, `Or` |
| Operadores | `Is`/`Equals`, `Not`, `LessThan`, `LessThanEqual`, `GreaterThan`, `Between`, `IsNull`, `IsNotNull`, `Like`, `Containing`, `StartingWith`, `EndingWith`, `In`, `NotIn`, `True`, `False`, `Before`, `After` |
| Modificadores | `IgnoreCase`, `AllIgnoreCase` |
| Ordenación | `OrderBy<Propiedad>Asc` / `Desc` |

### Propiedades anidadas

Si `Pedido` tiene un `Cliente` con un `email`:

```java
List<Pedido> findByClienteEmail(String email);
```

> **Ventaja:** si escribes mal el nombre de una propiedad (`findByNombr`), la aplicación **falla al arrancar** con un mensaje claro. El error se detecta pronto, no en producción.
>
> **Límite:** si el nombre se hace larguísimo, es mejor escribir la consulta a mano con `@Query` (lo verás en los documentos de JPA y MongoDB).

## Tipos de retorno

| Tipo | Uso |
|---|---|
| `T` | Un único resultado; `null` si no hay. Si hay más de uno, lanza excepción. |
| `Optional<T>` | Un único resultado que puede no existir. **Recomendado** para búsquedas de un elemento. |
| `List<T>` | Varios resultados. |
| `Stream<T>` | Varios resultados procesados de uno en uno (hay que cerrarlo, por ejemplo con *try-with-resources*). |
| `Page<T>` | Una página de resultados + información del total. |
| `Slice<T>` | Una página sin calcular el total (más barato). |
| `long`, `boolean` | Para `count…` y `exists…`. |

## Paginación y ordenación

Nunca devuelvas una colección grande entera. Spring Data lo resuelve con `Pageable`:

```java
public interface ProductoRepository extends JpaRepository<Producto, Long> {

    Page<Producto> findByCategoria(String categoria, Pageable paginacion);
}
```

```java
// Página 0 (la primera), 20 elementos, ordenados por precio descendente
Pageable paginacion = PageRequest.of(0, 20, Sort.by("precio").descending());

Page<Producto> pagina = repositorio.findByCategoria("perifericos", paginacion);

pagina.getContent();        // List<Producto> con los 20 elementos
pagina.getTotalElements();  // total de elementos que cumplen la condición
pagina.getTotalPages();     // número total de páginas
pagina.getNumber();         // número de la página actual
pagina.hasNext();           // ¿hay más páginas?
```

> Las páginas empiezan en **0**.

### Paginación desde el controlador

Spring MVC puede construir el `Pageable` directamente a partir de los parámetros `?page=`, `?size=` y `?sort=`:

```java
@GetMapping
public Page<ProductoResponse> listar(Pageable paginacion) {
    return servicio.listar(paginacion);
}
```

```
GET /productos?page=1&size=10&sort=precio,desc
```

> Devolver directamente un `Page` como JSON funciona, pero su estructura interna no está pensada como contrato estable de una API. Para APIs públicas es mejor devolver un DTO propio con el contenido y los datos de paginación que tú elijas (verás un ejemplo en el documento [DTOs, validación y diseño de entrada/salida](<../APIs-REST/DTOs y validacion.md>)).

## Proyecciones: devolver solo algunos campos

A veces no necesitas la entidad completa. Una **proyección** permite recuperar solo algunos campos. La forma más sencilla en Java moderno es un `record`:

```java
public record ProductoResumen(Long id, String nombre) {}

public interface ProductoRepository extends JpaRepository<Producto, Long> {

    List<ProductoResumen> findByCategoria(String categoria);
}
```

Spring Data rellena el `record` usando los nombres de sus componentes, que deben coincidir con propiedades de la entidad. La consulta solo trae esas columnas.

## Cómo encuentra Spring tus repositorios

Spring Boot busca automáticamente las interfaces de repositorio en el paquete de la clase principal (la de `@SpringBootApplication`) y en sus subpaquetes. No hace falta anotar las interfaces con `@Repository`.

## Spring Data REST (mención)

**Spring Data REST** va un paso más allá: **expone los repositorios como una API REST** sin escribir controladores. Con la dependencia `spring-boot-starter-data-rest`:

```java
@RepositoryRestResource(collectionResourceRel = "personas", path = "personas")
public interface PersonaRepository extends MongoRepository<Persona, String> {

    List<Persona> findByApellido(@Param("apellido") String apellido);
}
```

Genera *endpoints* como `GET /personas`, `POST /personas`, `GET /personas/{id}` o `GET /personas/search/findByApellido?apellido=…`, con respuestas en formato hipermedia HAL (nivel 3 del modelo de madurez de Richardson).

Es muy útil para prototipos y herramientas internas. En APIs con lógica de negocio, validaciones y DTOs, lo habitual es escribir los controladores a mano, como en el resto de esta documentación.

## Resumen

- **Spring Data** ofrece un modelo común para acceder a datos en distintas tecnologías.
- Declaras una **interfaz de repositorio** y Spring genera la implementación.
- Elige la interfaz base según lo que quieras exponer: `Repository`, `ListCrudRepository`, `JpaRepository`, `MongoRepository`.
- Las **consultas derivadas** se generan a partir del nombre del método.
- Usa `Optional` para un resultado y `Page`/`Pageable` para colecciones grandes.
- Las **proyecciones** con `record` traen solo los campos necesarios.

## Referencias

- Spring Data (proyecto): <https://spring.io/projects/spring-data>
- Baeldung — Spring Data Repositories: <https://www.baeldung.com/spring-data-repositories>
- Spring Data JPA — Query Methods: <https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html>
- Spring Data JPA — Projections: <https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html>
- Spring — Accessing JPA Data with REST (Spring Data REST): <https://spring.io/guides/gs/accessing-data-rest/>
