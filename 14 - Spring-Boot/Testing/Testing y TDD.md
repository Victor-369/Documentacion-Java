# Testing con Spring Boot y TDD

> 📚 **Documentación relacionada en este repositorio:** este documento aplica las pruebas a una aplicación Spring Boot. Si quieres repasar primero las herramientas por separado, consulta las guías de [JUnit](<../../08 - Testing/JUnit - Aprende a escribir pruebas paso a paso.md>), [AssertJ](<../../08 - Testing/AssertJ - Guia para empezar desde cero.md>), [Mockito](<../../08 - Testing/Mockito - Aprende a escribir pruebas paso a paso.md>) y [TDD](<../../08 - Testing/Test-Driven Development (TDD).md>).

## ¿Por qué escribir pruebas automatizadas?

- **Detectan errores pronto**, cuando son baratos de arreglar.
- **Permiten cambiar el código con confianza**: si rompes algo, una prueba te avisa.
- **Documentan** el comportamiento esperado con ejemplos ejecutables.
- **Mejoran el diseño**: el código difícil de probar suele estar mal diseñado (demasiadas responsabilidades, dependencias ocultas).

## Tipos de prueba: la pirámide

```
                 ▲
                / \        Extremo a extremo (E2E)
               /   \       Pocas, lentas, prueban el sistema entero
              /─────\
             /       \     Integración
            /         \    Varias piezas juntas (web + servicio + BD real)
           /───────────\
          /             \  Unitarias
         /               \ Muchas, rápidas, una clase aislada
        └─────────────────┘
```

| Tipo | Qué prueba | Velocidad | En Spring Boot |
|---|---|---|---|
| **Unitaria** | Una clase aislada, sustituyendo sus dependencias por dobles. | Milisegundos | JUnit + Mockito, **sin** Spring |
| **De porción** (*slice*) | Una capa con Spring, pero solo esa capa. | Segundos | `@WebMvcTest`, `@DataJpaTest`, `@DataMongoTest`… |
| **De integración** | La aplicación completa con sus dependencias reales. | Segundos o más | `@SpringBootTest` + Testcontainers |

La idea de la pirámide: **muchas pruebas unitarias**, bastantes de porción e integración, y pocas de extremo a extremo.

## Dependencias

En Spring Boot 4 cada tecnología tiene su *starter* de pruebas. Initializr los añade automáticamente según las dependencias elegidas:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
```

Estos *starters* incluyen las librerías de pruebas habituales:

| Librería | Para qué sirve |
|---|---|
| **JUnit Jupiter** | El *framework* de pruebas: `@Test`, `@BeforeEach`, `@ParameterizedTest`… |
| **AssertJ** | Comprobaciones legibles: `assertThat(x).isEqualTo(y)`. |
| **Mockito** | Dobles de prueba (*mocks*): objetos falsos que simulan dependencias. |
| **Spring Test** | Soporte de Spring: contexto de pruebas, `MockMvc`, `MockMvcTester`… |
| **JsonPath**, **JSONassert** | Comprobaciones sobre JSON. |

Ejecuta las pruebas con:

```bash
./mvnw test
```

## Anatomía de una buena prueba

Sigue el patrón **AAA** (*Arrange, Act, Assert*), también conocido como **Dado / Cuando / Entonces** (*Given / When / Then*):

```java
@Test
void aplicaEnvioGratisEnPedidosDeMasDeCincuentaEuros() {
    // Dado (Arrange): preparamos los datos
    var calculadora = new CalculadoraEnvio();

    // Cuando (Act): ejecutamos lo que queremos probar
    BigDecimal coste = calculadora.costeEnvio(new BigDecimal("60.00"));

    // Entonces (Assert): comprobamos el resultado
    assertThat(coste).isEqualByComparingTo("0.00");
}
```

Pautas:

- **Una prueba, un comportamiento.** Si falla, debe quedar claro qué se ha roto.
- **Nombres descriptivos**: `noPermiteCrearClienteConEmailRepetido` dice más que `test3`. También puedes usar `@DisplayName("…")` para un texto libre.
- **Independientes**: ninguna prueba debe depender de que otra se haya ejecutado antes.
- **Deterministas**: mismo resultado siempre. Cuidado con la hora actual, los números aleatorios o los datos compartidos.

> Con `BigDecimal` usa `isEqualByComparingTo`: `new BigDecimal("0.0")` y `new BigDecimal("0.00")` no son iguales con `equals` (tienen distinta escala), pero sí con `compareTo`.

## Pruebas unitarias con Mockito

> 💡 Los fundamentos de Mockito (*stubbing*, *matchers*, `verify`, `ArgumentCaptor`, *spies*…) se explican en detalle en [Mockito: aprende a escribir pruebas paso a paso](<../../08 - Testing/Mockito - Aprende a escribir pruebas paso a paso.md>).

Un servicio que depende de un repositorio:

```java
@Service
public class ClienteService {

    private final ClienteRepository repositorio;

    public ClienteService(ClienteRepository repositorio) {
        this.repositorio = repositorio;
    }

    @Transactional
    public ClienteResponse crear(CrearClienteRequest peticion) {
        if (repositorio.existsByEmailIgnoreCase(peticion.email())) {
            throw new ConflictoException("El email " + peticion.email() + " ya está registrado");
        }
        var guardado = repositorio.save(new Cliente(peticion.nombre(), peticion.email()));
        return ClienteResponse.desde(guardado);
    }
}
```

La prueba sustituye el repositorio por un **mock** de Mockito. No arranca Spring ni una base de datos:

```java
package com.ejemplo.tienda.cliente;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.verify;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import com.ejemplo.tienda.cliente.dto.CrearClienteRequest;
import com.ejemplo.tienda.comun.error.ConflictoException;

@ExtendWith(MockitoExtension.class)
class ClienteServiceTest {

    @Mock
    ClienteRepository repositorio;      // doble de prueba

    @InjectMocks
    ClienteService servicio;            // se crea inyectando los @Mock

    @Test
    void creaElClienteSiElEmailEstaLibre() {
        // Dado
        given(repositorio.existsByEmailIgnoreCase("ana@ejemplo.com")).willReturn(false);
        given(repositorio.save(any(Cliente.class))).willAnswer(inv -> inv.getArgument(0));

        // Cuando
        var respuesta = servicio.crear(new CrearClienteRequest("Ana", "ana@ejemplo.com", null));

        // Entonces
        assertThat(respuesta.nombre()).isEqualTo("Ana");
        verify(repositorio).save(any(Cliente.class));
    }

    @Test
    void noPermiteEmailsRepetidos() {
        given(repositorio.existsByEmailIgnoreCase("ana@ejemplo.com")).willReturn(true);

        assertThatThrownBy(() -> servicio.crear(new CrearClienteRequest("Ana", "ana@ejemplo.com", null)))
                .isInstanceOf(ConflictoException.class)
                .hasMessageContaining("ana@ejemplo.com");

        verify(repositorio, never()).save(any());
    }
}
```

| Elemento | Qué hace |
|---|---|
| `@ExtendWith(MockitoExtension.class)` | Activa Mockito en JUnit. |
| `@Mock` | Crea un objeto falso. Por defecto, sus métodos devuelven valores "vacíos" (`null`, `0`, `false`, listas vacías). |
| `@InjectMocks` | Crea el objeto a probar inyectándole los mocks por constructor. |
| `given(...).willReturn(...)` | Programa la respuesta del mock. |
| `willAnswer(inv -> inv.getArgument(0))` | Devuelve el mismo objeto que recibe (simula un `save`). |
| `verify(mock).metodo(...)` | Comprueba que se ha llamado. |
| `verify(mock, never())` | Comprueba que **no** se ha llamado. |

> **Sin mocks también se puede.** Si el repositorio es un puerto (una interfaz propia), a menudo es más sencillo y legible escribir una implementación falsa en memoria (un "*fake*") que programar un mock.

### Pruebas parametrizadas

Para probar el mismo comportamiento con varios datos:

```java
@ParameterizedTest
@CsvSource({
        "49.99, 4.95",
        "50.00, 0.00",
        "120.00, 0.00"
})
void calculaElCosteDeEnvio(BigDecimal importe, BigDecimal costeEsperado) {
    assertThat(new CalculadoraEnvio().costeEnvio(importe)).isEqualByComparingTo(costeEsperado);
}
```

## Probar la capa web con `@WebMvcTest`

`@WebMvcTest` arranca **solo la capa web** (controladores, `@RestControllerAdvice`, conversión JSON, validación) sin servicios ni base de datos. Los servicios se sustituyen por mocks con **`@MockitoBean`**.

Para hacer las peticiones se usa **`MockMvcTester`**, que simula peticiones HTTP sin levantar un servidor real y permite comprobar el resultado con AssertJ.

```java
package com.ejemplo.tienda.cliente;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;

import java.time.Instant;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.webmvc.test.autoconfigure.WebMvcTest;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.servlet.assertj.MockMvcTester;

import com.ejemplo.tienda.cliente.dto.ClienteResponse;
import com.ejemplo.tienda.comun.error.RecursoNoEncontradoException;

@WebMvcTest(ClienteController.class)
class ClienteControllerTest {

    @Autowired
    MockMvcTester mvc;

    @MockitoBean
    ClienteService servicio;

    @Test
    void devuelveElCliente() {
        given(servicio.obtener(1L))
                .willReturn(new ClienteResponse(1L, "Ana", "ana@ejemplo.com", null, Instant.now()));

        assertThat(mvc.get().uri("/api/v1/clientes/{id}", 1))
                .hasStatusOk()
                .bodyJson()
                .extractingPath("$.nombre").isEqualTo("Ana");
    }

    @Test
    void devuelve404SiNoExiste() {
        given(servicio.obtener(99L)).willThrow(new RecursoNoEncontradoException("cliente", 99L));

        assertThat(mvc.get().uri("/api/v1/clientes/{id}", 99))
                .hasStatus(HttpStatus.NOT_FOUND);
    }

    @Test
    void rechazaDatosNoValidos() {
        String json = """
                {"nombre": "", "email": "no-es-un-email"}
                """;

        assertThat(mvc.post().uri("/api/v1/clientes")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(json))
                .hasStatus(HttpStatus.BAD_REQUEST);
    }

    @Test
    void creaElClienteYDevuelve201() {
        given(servicio.crear(any()))
                .willReturn(new ClienteResponse(7L, "Ana", "ana@ejemplo.com", null, Instant.now()));

        assertThat(mvc.post().uri("/api/v1/clientes")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                                {"nombre": "Ana", "email": "ana@ejemplo.com"}
                                """))
                .hasStatus(HttpStatus.CREATED)
                .bodyJson()
                .extractingPath("$.id").isEqualTo(7);
    }
}
```

Lo que comprueba esta prueba y una unitaria no puede:

- Que la **URL y el método** HTTP están bien asociados.
- Que el JSON de entrada se **convierte** correctamente y que la **validación** funciona.
- Que las excepciones se **traducen** al código de estado correcto.
- Que el JSON de salida tiene la **forma** esperada.

> **Spring Boot 4:** `@WebMvcTest` está en `org.springframework.boot.webmvc.test.autoconfigure` y requiere `spring-boot-starter-webmvc-test`. `@MockitoBean` sustituye a la antigua `@MockBean`.
>
> `MockMvcTester` necesita AssertJ en el *classpath* (los *starters* de pruebas ya lo incluyen). También existe el clásico `MockMvc`, basado en Hamcrest, que verás en muchos tutoriales.

## Probar la capa de persistencia con `@DataJpaTest`

`@DataJpaTest` arranca **solo la capa JPA**: entidades, repositorios y base de datos. Por defecto:

- configura una **base de datos embebida** si hay una en el *classpath* (por ejemplo, H2);
- cada prueba es **transaccional** y se **deshace** al terminar, así que las pruebas no se contaminan entre sí;
- muestra el SQL ejecutado;
- ofrece un `TestEntityManager` para preparar datos.

Añade H2 solo para las pruebas:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

```java
package com.ejemplo.tienda.producto;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigDecimal;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.jpa.test.autoconfigure.DataJpaTest;
import org.springframework.boot.jpa.test.autoconfigure.TestEntityManager;

@DataJpaTest
class ProductoRepositoryTest {

    @Autowired
    TestEntityManager em;

    @Autowired
    ProductoRepository repositorio;

    @Test
    void buscaPorNombreSinDistinguirMayusculas() {
        em.persistAndFlush(new Producto("Teclado mecánico", new BigDecimal("59.90"), 10));

        assertThat(repositorio.findByNombreIgnoreCase("TECLADO MECÁNICO")).isPresent();
    }

    @Test
    void devuelveLosProductosMasBaratosQueElLimite() {
        em.persist(new Producto("Ratón", new BigDecimal("15.00"), 5));
        em.persist(new Producto("Monitor", new BigDecimal("199.00"), 2));
        em.flush();

        var baratos = repositorio.findByPrecioLessThanOrderByPrecioAsc(new BigDecimal("100"));

        assertThat(baratos).extracting(Producto::getNombre).containsExactly("Ratón");
    }
}
```

Merece la pena probar los repositorios cuando tienen **consultas propias** (derivadas complejas, `@Query`). No hace falta probar `save` o `findById`: ya los prueba Spring Data.

> H2 es rápido, pero **no es PostgreSQL**: algunas funciones o tipos se comportan distinto. Para tener total seguridad, prueba contra la base de datos real con Testcontainers (siguiente apartado).

## Pruebas de integración con Testcontainers

**Testcontainers** es una librería que, desde una prueba, **arranca contenedores Docker** (PostgreSQL, MongoDB, Kafka…) y los elimina al terminar. Así pruebas contra la **misma tecnología que en producción**. Requiere tener Docker funcionando.

Dependencias (Initializr las añade si eliges **Testcontainers** junto con tu base de datos):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

> **Testcontainers 2:** los artefactos llevan el prefijo `testcontainers-` (`testcontainers-postgresql`, `testcontainers-mongodb`…) y las clases de cada módulo están en su propio paquete (`org.testcontainers.postgresql.PostgreSQLContainer`, `org.testcontainers.mongodb.MongoDBContainer`). Spring Boot gestiona las versiones.

### Prueba de integración completa

```java
package com.ejemplo.tienda;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.testcontainers.service.connection.ServiceConnection;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.assertj.MockMvcTester;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.postgresql.PostgreSQLContainer;

@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
class ClienteIntegracionTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:17");

    @Autowired
    MockMvcTester mvc;

    @Test
    void creaYRecuperaUnCliente() {
        var creado = mvc.post().uri("/api/v1/clientes")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                        {"nombre": "Ana", "email": "ana@ejemplo.com"}
                        """)
                .exchange();

        assertThat(creado).hasStatus(HttpStatus.CREATED);

        // El cliente se ha guardado de verdad en PostgreSQL: al pedir la lista, aparece
        assertThat(mvc.get().uri("/api/v1/clientes"))
                .hasStatusOk()
                .bodyJson()
                .extractingPath("$.contenido[0].nombre").isEqualTo("Ana");
    }
}
```

| Elemento | Qué hace |
|---|---|
| `@SpringBootTest` | Arranca la aplicación completa. |
| `@AutoConfigureMockMvc` | Configura `MockMvc`/`MockMvcTester` para hacer peticiones a la aplicación. |
| `@Testcontainers` + `@Container` | Arranca el contenedor antes de las pruebas y lo para al final. El campo es `static` para compartir el contenedor entre todas las pruebas de la clase. |
| `@ServiceConnection` | Spring Boot obtiene del contenedor la URL, el usuario y la contraseña y configura la conexión automáticamente. Sin esto habría que usar `@DynamicPropertySource`. |

> Arrancar contenedores lleva unos segundos. Spring **reutiliza el contexto** de la aplicación entre clases de prueba con la misma configuración, así que conviene que las pruebas de integración compartan configuración (por ejemplo, con una clase base o un `@TestConfiguration` común).

## Probar MongoDB con `@DataMongoTest`

```java
package com.ejemplo.tienda.pedido;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.mongodb.test.autoconfigure.DataMongoTest;
import org.springframework.boot.testcontainers.service.connection.ServiceConnection;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.mongodb.MongoDBContainer;

import com.ejemplo.tienda.pedido.Pedido.Cliente;
import com.ejemplo.tienda.pedido.Pedido.EstadoPedido;
import com.ejemplo.tienda.pedido.Pedido.LineaPedido;

@DataMongoTest
@Testcontainers
class PedidoRepositoryTest {

    @Container
    @ServiceConnection
    static MongoDBContainer mongo = new MongoDBContainer("mongo:8");

    @Autowired
    PedidoRepository repositorio;

    @Test
    void buscaPedidosPorEmailDelCliente() {
        repositorio.save(new Pedido(null,
                new Cliente("Ana", "ana@ejemplo.com"),
                EstadoPedido.PAGADO,
                List.of(new LineaPedido("Teclado", 1, new BigDecimal("59.90"))),
                Instant.now()));

        assertThat(repositorio.findByClienteEmail("ana@ejemplo.com")).hasSize(1);
    }
}
```

Necesita `spring-boot-starter-data-mongodb-test` y `org.testcontainers:testcontainers-mongodb`.

## TDD: desarrollo guiado por pruebas

> 💡 Para una explicación más detallada del ciclo, con ejemplos completos fuera de Spring, consulta [Test-Driven Development (TDD)](<../../08 - Testing/Test-Driven Development (TDD).md>).

**TDD** (*Test-Driven Development*) es una forma de desarrollar en la que **la prueba se escribe antes que el código**. Se trabaja en ciclos muy cortos de tres pasos:

```
     ┌──────────────┐
     │   1. ROJO    │  Escribe una prueba pequeña que FALLA
     └──────┬───────┘  (el comportamiento aún no existe)
            ▼
     ┌──────────────┐
     │  2. VERDE    │  Escribe el código MÍNIMO para que pase
     └──────┬───────┘
            ▼
     ┌──────────────┐
     │ 3. REFACTOR  │  Mejora el código sin cambiar su comportamiento
     └──────┬───────┘  (las pruebas siguen en verde)
            │
            └──────────▶ vuelta al paso 1 con el siguiente caso
```

### Ejemplo paso a paso

**Requisito:** el coste de envío es 4,95 €, pero es gratis a partir de 50 €.

**Ciclo 1 — Rojo.** Primero, la prueba del caso básico:

```java
class CalculadoraEnvioTest {

    @Test
    void cobraElEnvioEnPedidosPequenos() {
        var calculadora = new CalculadoraEnvio();
        assertThat(calculadora.costeEnvio(new BigDecimal("20.00"))).isEqualByComparingTo("4.95");
    }
}
```

No compila: la clase no existe. Eso cuenta como "rojo".

**Ciclo 1 — Verde.** El código mínimo:

```java
public class CalculadoraEnvio {

    public BigDecimal costeEnvio(BigDecimal importePedido) {
        return new BigDecimal("4.95");
    }
}
```

Parece "hacer trampa", pero es intencionado: el código solo crece cuando una prueba lo exige.

**Ciclo 2 — Rojo.** Siguiente caso:

```java
@Test
void elEnvioEsGratisAPartirDeCincuentaEuros() {
    var calculadora = new CalculadoraEnvio();
    assertThat(calculadora.costeEnvio(new BigDecimal("50.00"))).isEqualByComparingTo("0");
}
```

Falla.

**Ciclo 2 — Verde.**

```java
public BigDecimal costeEnvio(BigDecimal importePedido) {
    if (importePedido.compareTo(new BigDecimal("50.00")) >= 0) {
        return BigDecimal.ZERO;
    }
    return new BigDecimal("4.95");
}
```

**Ciclo 2 — Refactor.** Damos nombre a los valores, sin cambiar el comportamiento:

```java
public class CalculadoraEnvio {

    private static final BigDecimal UMBRAL_ENVIO_GRATIS = new BigDecimal("50.00");
    private static final BigDecimal COSTE_ESTANDAR = new BigDecimal("4.95");

    public BigDecimal costeEnvio(BigDecimal importePedido) {
        return importePedido.compareTo(UMBRAL_ENVIO_GRATIS) >= 0
                ? BigDecimal.ZERO
                : COSTE_ESTANDAR;
    }
}
```

Las pruebas siguen en verde. Siguiente ciclo: ¿qué pasa con un importe negativo? Se escribe la prueba (por ejemplo, que lance `IllegalArgumentException`) y se repite.

### Beneficios de TDD

- Cada línea de código tiene una prueba que la justifica.
- Obliga a pensar primero en **qué** debe hacer el código antes que en **cómo**.
- Produce diseños con dependencias explícitas, fáciles de probar.
- Los pasos pequeños hacen que los errores se localicen enseguida.

### TDD en una API Spring Boot

Un flujo habitual para una funcionalidad nueva:

1. Prueba **`@WebMvcTest`** del *endpoint* (URL, códigos de estado, JSON) → crea el controlador y el DTO.
2. Pruebas **unitarias** del servicio con sus reglas de negocio → implementa el servicio.
3. Prueba **`@DataJpaTest`** si hay consultas nuevas → implementa el repositorio.
4. Una prueba de **integración** del flujo completo con Testcontainers.

## Buenas prácticas

- Prueba **comportamiento**, no detalles de implementación. Si refactorizas sin cambiar el comportamiento, las pruebas no deberían romperse.
- Prueba **los casos de error** tanto como los de éxito.
- Evita la lógica (`if`, bucles) dentro de las pruebas.
- No abuses de los mocks: si para probar algo necesitas diez, probablemente la clase hace demasiado.
- Mantén las pruebas **rápidas**: las unitarias deberían ejecutarse en milisegundos.
- Las pruebas son código: mantenlas limpias y legibles.

## Resumen

- **Unitarias** (JUnit + AssertJ + Mockito, sin Spring) para la lógica de negocio.
- **`@WebMvcTest`** + `MockMvcTester` + `@MockitoBean` para los controladores.
- **`@DataJpaTest`** / **`@DataMongoTest`** para los repositorios con consultas propias.
- **`@SpringBootTest`** + **Testcontainers** + `@ServiceConnection` para la integración con la base de datos real.
- **TDD:** rojo → verde → refactor, en ciclos pequeños.

## Referencias

- Spring Boot — Testing: <https://docs.spring.io/spring-boot/reference/testing/index.html#testing>
- Spring — Testing the Web Layer: <https://spring.io/guides/gs/testing-web/>
- Baeldung — Testing in Spring Boot: <https://www.baeldung.com/spring-boot-testing>
- Spring Boot — Testcontainers: <https://docs.spring.io/spring-boot/reference/testing/testcontainers.html>
- JUnit: <https://junit.org/>
- AssertJ: <https://assertj.github.io/doc/>
- Mockito: <https://site.mockito.org/>
- Testcontainers para Java: <https://java.testcontainers.org/>
