# Relaciones y modelos de datos

Cuando tu proyecto empieza a tener entidades relacionadas, tienes que tomar decisiones de arquitectura que marcarán la calidad del código. Más allá de qué anotación poner, lo importante es entender:

- **cuándo** quieres propagar los cambios de una entidad a las relacionadas (*cascade*),
- **cuándo** quieres cargar los datos relacionados automáticamente (`EAGER`) o solo bajo demanda (`LAZY`),
- **cuándo** tiene sentido una relación **bidireccional** y cuándo no.

> **Regla general en APIs REST:** mantén modelos limpios, evita relaciones bidireccionales cuando no sean necesarias y usa **DTOs** para no exponer de forma descontrolada el árbol completo de objetos.

## Parte A — Relaciones en JPA

Usaremos este modelo de ejemplo:

```
 PerfilCliente                                            Etiqueta
      1                                                       *
      │                                                       │
      1                                                       *
   Cliente 1 ──── * Pedido 1 ──── * LineaPedido * ──── 1 Producto
```

(`1` = uno, `*` = muchos)

- Un **cliente** tiene muchos **pedidos**; cada pedido es de un cliente.
- Un **pedido** tiene muchas **líneas**; cada línea pertenece a un pedido.
- Cada **línea** se refiere a un **producto**.
- Un **cliente** tiene un **perfil**.
- Un **producto** tiene muchas **etiquetas** y una etiqueta se aplica a muchos productos.

## Los cuatro tipos de relación

| Anotación | Significado | Ejemplo |
|---|---|---|
| `@ManyToOne` | Muchos de esta entidad se relacionan con uno de la otra. | Muchos pedidos → un cliente |
| `@OneToMany` | Uno de esta entidad se relaciona con muchos de la otra. | Un pedido → muchas líneas |
| `@OneToOne` | Uno con uno. | Un cliente → un perfil |
| `@ManyToMany` | Muchos con muchos. | Productos ↔ etiquetas |

### Unidireccional y bidireccional

- **Unidireccional:** solo una de las dos entidades conoce la relación. `Pedido` tiene un campo `cliente`, pero `Cliente` no tiene una lista de pedidos.
- **Bidireccional:** ambas la conocen. `Pedido.cliente` y `Cliente.pedidos`.

En la base de datos no hay diferencia: hay **una única clave foránea** (`pedidos.cliente_id`). La diferencia está solo en el modelo Java.

### El lado propietario (*owning side*)

En cada relación, uno de los dos lados es el **propietario**: el que tiene la clave foránea y el que JPA mira para guardar la relación.

- En `@ManyToOne` / `@OneToMany`, el propietario es **siempre el lado `@ManyToOne`**.
- El otro lado, si existe, se marca con **`mappedBy`**, indicando el nombre del campo del lado propietario.

> Si solo cambias el lado `mappedBy` (por ejemplo, añades un pedido a `cliente.getPedidos()` sin asignar `pedido.setCliente(...)`), **la relación no se guarda**.

## `@ManyToOne`: la relación más común

```java
@Entity
@Table(name = "pedidos")
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    // ...
}
```

- `@JoinColumn` define la columna de clave foránea en la tabla `pedidos`.
- `optional = false` indica que siempre debe haber un cliente.
- `fetch = FetchType.LAZY` → ver el apartado «Carga de datos: `EAGER` y `LAZY`».

En muchos casos **esto es todo lo que necesitas**. Si quieres los pedidos de un cliente, pídelos al repositorio:

```java
List<Pedido> findByClienteId(Long clienteId);
```

No hace falta una lista `pedidos` dentro de `Cliente`. Un cliente puede tener miles de pedidos: cargarlos todos como parte del cliente rara vez es buena idea.

## `@OneToMany`

Tiene sentido cuando el "uno" es el **dueño natural** de los "muchos" y se manejan juntos: un pedido y sus líneas. Las líneas no tienen sentido sin su pedido.

### Opción A: unidireccional con `@JoinColumn`

```java
@Entity
@Table(name = "pedidos")
public class Pedido {

    // ...

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "pedido_id", nullable = false)
    private List<LineaPedido> lineas = new ArrayList<>();

    public void agregarLinea(LineaPedido linea) {
        lineas.add(linea);
    }

    public List<LineaPedido> getLineas() {
        return Collections.unmodifiableList(lineas);
    }
}
```

`LineaPedido` no necesita saber a qué pedido pertenece en el modelo Java. Es la opción más sencilla.

> **Importante:** sin `@JoinColumn`, un `@OneToMany` unidireccional hace que JPA cree una **tabla intermedia** (`pedidos_lineas`), algo que casi nunca se quiere.

### Opción B: bidireccional con `mappedBy`

```java
@Entity
public class Pedido {

    @OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<LineaPedido> lineas = new ArrayList<>();

    // Método auxiliar: mantiene sincronizados los DOS lados
    public void agregarLinea(LineaPedido linea) {
        lineas.add(linea);
        linea.asignarPedido(this);
    }

    public void quitarLinea(LineaPedido linea) {
        lineas.remove(linea);
        linea.asignarPedido(null);
    }
}

@Entity
public class LineaPedido {

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "pedido_id")
    private Pedido pedido;   // lado propietario

    void asignarPedido(Pedido pedido) {
        this.pedido = pedido;
    }
}
```

Úsala cuando **realmente necesites navegar en los dos sentidos** (desde una línea llegar a su pedido). Si la eliges, **mantén siempre sincronizados ambos lados** con métodos auxiliares como `agregarLinea`.

## `@OneToOne`

```java
@Entity
public class Cliente {

    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @JoinColumn(name = "perfil_id")
    private PerfilCliente perfil;
}
```

Antes de crear una relación uno a uno, pregúntate si esos campos no podrían estar en la misma entidad. Tiene sentido cuando la parte separada es grande, opcional o se usa poco.

Una alternativa frecuente es que el perfil comparta la clave primaria del cliente con `@MapsId`:

```java
@Entity
public class PerfilCliente {

    @Id
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @MapsId
    private Cliente cliente;
}
```

## `@ManyToMany`

```java
@Entity
public class Producto {

    @ManyToMany
    @JoinTable(
            name = "productos_etiquetas",
            joinColumns = @JoinColumn(name = "producto_id"),
            inverseJoinColumns = @JoinColumn(name = "etiqueta_id"))
    private Set<Etiqueta> etiquetas = new HashSet<>();
}
```

JPA gestiona una **tabla intermedia** (`productos_etiquetas`) con las dos claves foráneas.

- Usa `Set` en lugar de `List` en las relaciones `@ManyToMany`: Hibernate las gestiona de forma más eficiente al borrar elementos.
- **No uses `CascadeType.REMOVE`** en un `@ManyToMany`: borrar un producto borraría las etiquetas, que pueden estar en otros productos.

### Cuando la relación tiene datos propios

Si la relación necesita atributos (por ejemplo, **cuándo** se asignó la etiqueta, o la **cantidad** de un producto en un pedido), no uses `@ManyToMany`: crea una **entidad intermedia** con dos `@ManyToOne`. Eso es exactamente `LineaPedido`, que une `Pedido` y `Producto` y además guarda `cantidad` y `precioUnitario`.

## Carga de datos: `EAGER` y `LAZY`

| Tipo | Comportamiento |
|---|---|
| `FetchType.EAGER` | Al cargar la entidad, se cargan **también** las relacionadas, siempre. |
| `FetchType.LAZY` | Las relacionadas **no** se cargan hasta que accedes a ellas. Hibernate pone en su lugar un *proxy*. |

### Valores por defecto (¡ojo!)

| Relación | Por defecto |
|---|---|
| `@ManyToOne` | **EAGER** |
| `@OneToOne` | **EAGER** |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

> **Recomendación:** declara **todas** las relaciones como `LAZY` (en especial `@ManyToOne` y `@OneToOne`, que por defecto son `EAGER`) y decide **en cada consulta** qué necesitas cargar. `EAGER` en el modelo obliga a cargar datos siempre, aunque no se usen, y no se puede desactivar después.

### `LazyInitializationException`

Si accedes a una relación `LAZY` **después** de que la transacción haya terminado, Hibernate ya no puede consultar la base de datos y lanza `LazyInitializationException`:

```java
// En el controlador, fuera de la transacción del servicio:
Pedido pedido = servicio.obtenerEntidad(15L);
pedido.getCliente().getNombre();   // ¡LazyInitializationException!
```

Soluciones correctas:

1. **Convertir a DTO dentro del servicio**, mientras la transacción está abierta, y devolver el DTO.
2. **Cargar lo necesario en la propia consulta** (apartado «El problema N+1»).

### `open-in-view`

Spring Boot activa por defecto la propiedad `spring.jpa.open-in-view`, que mantiene abierta la sesión de Hibernate durante toda la petición HTTP, incluida la generación del JSON. Así "desaparece" la `LazyInitializationException`… a cambio de lanzar consultas ocultas desde la capa web y mantener ocupadas las conexiones a la base de datos más tiempo. Por eso Spring Boot muestra un aviso al arrancar si no la configuras.

Lo recomendable es desactivarla y resolver la carga de forma explícita:

```properties
spring.jpa.open-in-view=false
```

## El problema N+1

Es el problema de rendimiento más habitual con JPA. Imagina:

```java
List<Pedido> pedidos = pedidoRepository.findAll();      // 1 consulta
for (Pedido p : pedidos) {
    IO.println(p.getCliente().getNombre());              // 1 consulta POR CADA pedido
}
```

Con 100 pedidos se ejecutan **101 consultas**: 1 para la lista y N para los clientes. Activa `spring.jpa.show-sql=true` y lo verás claramente.

### Solución 1: `JOIN FETCH` en JPQL

```java
@Query("""
        SELECT p FROM Pedido p
        JOIN FETCH p.cliente
        WHERE p.estado = :estado
        """)
List<Pedido> buscarConCliente(@Param("estado") EstadoPedido estado);
```

Una sola consulta con `JOIN` trae pedidos y clientes.

### Solución 2: `@EntityGraph`

Indica qué relaciones cargar, sin escribir la consulta:

```java
@EntityGraph(attributePaths = {"cliente", "lineas", "lineas.producto"})
Optional<Pedido> findConDetalleById(Long id);
```

### Solución 3: proyecciones a DTO

Si solo necesitas unos campos, pide solo esos:

```java
public record PedidoResumen(Long id, String nombreCliente, EstadoPedido estado) {}

@Query("""
        SELECT new com.ejemplo.tienda.pedido.PedidoResumen(p.id, c.nombre, p.estado)
        FROM Pedido p JOIN p.cliente c
        """)
List<PedidoResumen> listarResumen();
```

> **Cuidado** al hacer `JOIN FETCH` de una colección junto con paginación: Hibernate tiene que paginar en memoria y lo avisa con un mensaje. Para listados paginados, mejor proyecciones a DTO o cargar las colecciones en una segunda consulta.

## Propagación de cambios: `cascade`

`cascade` indica qué operaciones sobre una entidad se **propagan** a las entidades relacionadas.

| Tipo | Se propaga… |
|---|---|
| `PERSIST` | al guardar una entidad nueva. |
| `MERGE` | al fusionar una entidad separada. |
| `REMOVE` | al borrar. |
| `REFRESH` | al recargar desde la base de datos. |
| `DETACH` | al separar del contexto. |
| `ALL` | todas las anteriores. |

### `orphanRemoval`

Con `orphanRemoval = true`, si **quitas** un elemento de la colección, JPA **lo borra** de la base de datos (se ha quedado "huérfano").

```java
pedido.quitarLinea(linea);   // al confirmar la transacción: DELETE de esa línea
```

### ¿Cuándo usar cascada?

Usa `cascade = CascadeType.ALL` + `orphanRemoval = true` **solo** cuando la relación es de **composición**: el hijo pertenece a un único padre y no tiene sentido sin él.

| Relación | ¿Cascada? | Motivo |
|---|---|---|
| Pedido → líneas | **Sí** (`ALL` + `orphanRemoval`) | Las líneas son parte del pedido. |
| Cliente → perfil | **Sí** | El perfil es parte del cliente. |
| Pedido → cliente | **No** | El cliente existe independientemente del pedido. Borrar un pedido no debe borrar al cliente. |
| Línea → producto | **No** | El producto pertenece al catálogo. |
| Producto ↔ etiquetas | **No** (como mucho `PERSIST`/`MERGE`) | Las etiquetas se comparten. |

> **Regla práctica:** la cascada va siempre **del padre al hijo** y **nunca** en un `@ManyToOne` (de hijo a padre).

## `equals` y `hashCode` en entidades

Las entidades se guardan en colecciones (`Set`, `List`) y Hibernate compara instancias. Implementar mal `equals`/`hashCode` produce errores difíciles de encontrar. Pautas:

- **No** los generes incluyendo todos los campos ni las relaciones (puedes provocar cargas `LAZY` inesperadas o bucles infinitos).
- Si no los sobrescribes, Java compara por identidad del objeto, lo que funciona bien dentro de una misma transacción.
- Si necesitas igualdad lógica, básala en un **identificador de negocio único e inmutable** (un código, un email…) o en el id con cuidado de que sea `null` antes de guardar.

## Relaciones y JSON: por qué usar DTOs

Si devuelves una entidad bidireccional directamente en un controlador:

```java
@GetMapping("/{id}")
public Cliente obtener(@PathVariable Long id) { ... }   // ¡No!
```

pueden pasar varias cosas malas:

- **Bucle infinito:** Jackson serializa `cliente.pedidos`, cada pedido serializa su `cliente`, que serializa sus `pedidos`… hasta un `StackOverflowError`.
- **Consultas ocultas:** cada relación `LAZY` que Jackson recorre lanza una consulta (o una `LazyInitializationException`).
- **Exposición de datos:** se publica el árbol completo de objetos, incluyendo campos internos.
- **Acoplamiento:** cualquier cambio en el modelo de base de datos cambia el contrato de la API.

La solución es devolver **DTOs** que contengan exactamente lo que la API debe mostrar:

```java
public record PedidoResponse(
        Long id,
        String estado,
        ClienteResumen cliente,
        List<LineaResponse> lineas,
        BigDecimal total) {

    public record ClienteResumen(Long id, String nombre) {}

    public record LineaResponse(String producto, int cantidad, BigDecimal subtotal) {}
}
```

Las anotaciones de Jackson `@JsonManagedReference`/`@JsonBackReference` o `@JsonIgnore` "arreglan" el bucle, pero no los otros tres problemas. Los DTO sí.

## Parte B — Documentos embebidos en MongoDB

## Subdocumentos

En MongoDB es habitual tener **subdocumentos**:

```java
@Document(collection = "pedidos")
public record Pedido(@Id String id, ClienteResumen cliente, List<LineaPedido> lineas) {}

public record LineaPedido(String producto, int cantidad, BigDecimal precioUnitario) {}
```

A diferencia de JPA, aquí **no hay "relaciones" estrictas ni `JOIN`**: las líneas viven **dentro** del documento del pedido. No hay claves foráneas, ni cascadas, ni carga perezosa: al leer el pedido se leen sus líneas, y al guardarlo se guardan.

Este modelo es ideal cuando:

- los datos **tienen sentido juntos**,
- la **lectura** del documento es **más frecuente** que su modificación,
- **no hace falta consultar los subdocumentos de forma aislada**.

## Comparativa JPA frente a MongoDB

| Aspecto | JPA (relacional) | MongoDB (documental) |
|---|---|---|
| Pedido con líneas | Dos tablas + clave foránea + `@OneToMany` | Un documento con una lista dentro |
| Leer el pedido completo | `JOIN` o varias consultas | Una lectura |
| Borrar el pedido y sus líneas | `cascade` + `orphanRemoval` | Borrar el documento |
| Consultar "todas las líneas del producto X" | Consulta directa sobre la tabla de líneas | Posible (`lineas.producto`), pero devuelve pedidos completos; si es frecuente, quizá las líneas deban ser otra colección |
| Datos compartidos (cliente) | `@ManyToOne` | Referencia por id, o copia de los campos necesarios |
| Integridad | Garantizada por la base de datos (claves foráneas) | Responsabilidad de la aplicación |

## Resumen

- Empieza por **`@ManyToOne` unidireccional**; añade el otro lado solo si lo necesitas.
- Declara las relaciones como **`LAZY`** y carga lo necesario en cada consulta (`JOIN FETCH`, `@EntityGraph`, proyecciones).
- Vigila el **problema N+1** activando el registro de SQL.
- Usa **cascada y `orphanRemoval`** solo en relaciones de **composición** (padre → hijo).
- Considera desactivar **`open-in-view`**.
- En MongoDB, **embebe** lo que se lee junto y no se consulta por separado.
- Devuelve siempre **DTOs**, nunca el árbol de entidades.

## Referencias

- Spring Data JPA — Reference Documentation: <https://docs.spring.io/spring-data/jpa/reference/>
- Hibernate ORM — User Guide (asociaciones y *fetching*): <https://hibernate.org/orm/documentation/>
- MongoDB — Data Modeling: <https://www.mongodb.com/docs/manual/data-modeling/>
- Baeldung — The DTO Pattern: <https://www.baeldung.com/java-dto-pattern>
