# Arquitectura avanzada en Spring Boot

## ¿Qué es la arquitectura de software?

La **arquitectura** de una aplicación es su estructura de alto nivel: cómo se divide en partes, qué responsabilidad tiene cada una y cómo se comunican. Son las decisiones que más cuesta cambiar después.

Una buena arquitectura consigue que la aplicación sea:

- **fácil de entender**: se sabe dónde está cada cosa;
- **fácil de cambiar**: un cambio afecta a pocas partes;
- **fácil de probar**: cada parte se puede probar por separado;
- **independiente de los detalles**: la base de datos o el *framework* se pueden sustituir con un coste razonable.

## Patrones de arquitectura de software

En *Software Architecture Patterns* (O'Reilly), Mark Richards describe cinco patrones muy extendidos:

| Patrón | Idea principal | Cuándo encaja |
|---|---|---|
| **Por capas** (*layered*) | El código se divide en capas horizontales (presentación, negocio, persistencia…), cada una con una responsabilidad. | Punto de partida habitual; aplicaciones de gestión. |
| **Dirigida por eventos** (*event-driven*) | Componentes desacoplados que se comunican publicando y reaccionando a eventos. | Sistemas asíncronos, con mucho volumen o muchos consumidores de la misma información. |
| **Microkernel** (*plug-in*) | Un núcleo mínimo al que se añaden funcionalidades como módulos enchufables. | Productos que se personalizan por cliente; herramientas extensibles (como un IDE). |
| **Microservicios** | La aplicación es un conjunto de servicios pequeños que se despliegan por separado. | Sistemas grandes con varios equipos y necesidades de escalado distintas. |
| **Basada en espacio** (*space-based*) | Los datos se mantienen en memoria replicada entre varias unidades de procesamiento para evitar el cuello de botella de la base de datos. | Picos de carga muy altos e impredecibles. |

No son excluyentes: un microservicio puede estar organizado por capas internamente y comunicarse con otros mediante eventos.

## Arquitectura por capas, en profundidad

```
 ┌──────────────────────────┐
 │      Presentación        │  @RestController, DTOs
 ├──────────────────────────┤
 │        Negocio           │  @Service, reglas
 ├──────────────────────────┤
 │      Persistencia        │  Repositorios
 ├──────────────────────────┤
 │      Base de datos       │
 └──────────────────────────┘
```

### Capas cerradas y abiertas

- Una capa **cerrada** obliga a pasar por ella: la presentación no puede llamar directamente a la persistencia, tiene que pasar por el negocio. Así se consigue el **aislamiento entre capas**: cambiar una capa no afecta a las que no la usan directamente.
- Una capa **abierta** se puede saltar. Se usa a veces para capas de servicios compartidos (utilidades, registro de *logs*…).

Por defecto, conviene que todas las capas sean **cerradas**.

### El antipatrón "sumidero" (*architecture sinkhole*)

Ocurre cuando muchas peticiones atraviesan las capas **sin que ninguna haga nada**: el controlador llama al servicio, que solo llama al repositorio, que solo hace una consulta. Richards propone como referencia la regla 80-20: si alrededor del 20 % de las peticiones son así, es normal; si son la gran mayoría, la arquitectura por capas quizá no es la adecuada o las capas están mal definidas.

### El límite de la arquitectura por capas clásica

En la versión clásica, **la capa de negocio depende de la de persistencia**: el servicio usa directamente un `JpaRepository` y las entidades JPA. Eso significa que las reglas de negocio quedan atadas a JPA y a la base de datos. Las arquitecturas de los siguientes apartados resuelven precisamente esto.

## Arquitectura limpia y hexagonal

### La idea central: la regla de dependencia

La **arquitectura limpia** (*Clean Architecture*, Robert C. Martin) y la **arquitectura hexagonal** (*Ports and Adapters*, Alistair Cockburn) comparten una idea:

> **El núcleo de la aplicación (el dominio y los casos de uso) no depende de nada externo. Son los detalles (web, base de datos, *frameworks*) los que dependen del núcleo.**

```
          ┌───────────────────────────────────────────────┐
          │        Infraestructura (detalles)             │
          │   Controladores REST, JPA, MongoDB, Kafka…    │
          │      ┌─────────────────────────────────┐      │
          │      │   Aplicación (casos de uso)     │      │
          │      │      ┌───────────────────┐      │      │
          │      │      │     Dominio       │      │      │
          │      │      │ entidades y reglas│      │      │
          │      │      └───────────────────┘      │      │
          │      └─────────────────────────────────┘      │
          └───────────────────────────────────────────────┘

            Las dependencias apuntan siempre HACIA DENTRO
```

| Capa | Contiene | Depende de |
|---|---|---|
| **Dominio** | Entidades de negocio, reglas, excepciones de dominio. Java puro. | Nada |
| **Aplicación** | Casos de uso ("crear pedido", "cancelar pedido") y **puertos** (interfaces que el caso de uso necesita). | Dominio |
| **Infraestructura** | **Adaptadores**: controladores REST, repositorios JPA, clientes HTTP, configuración de Spring. | Aplicación y dominio |

### Puertos y adaptadores

- Un **puerto** es una **interfaz** definida por el núcleo que describe lo que necesita ("guardar un pedido", "enviar una notificación").
- Un **adaptador** es una **implementación** de ese puerto con una tecnología concreta (JPA, MongoDB, email…).
  - Adaptadores **de entrada** (*driving*): llaman al núcleo. Ejemplo: un controlador REST.
  - Adaptadores **de salida** (*driven*): el núcleo los usa a través de un puerto. Ejemplo: un repositorio JPA.

```
   Petición HTTP                                                  Base de datos
        │                                                               ▲
        ▼                                                               │
 ┌──────────────┐   usa    ┌──────────────┐  usa   ┌────────┐ implementa ┌──────────────┐
 │ Controlador  │ ───────▶ │ Caso de uso  │ ─────▶ │ Puerto │ ◀───────── │ Adaptador JPA│
 │ (adaptador   │          │ (aplicación) │        │(interf.)│           │ (infraestr.) │
 │  de entrada) │          └──────────────┘        └────────┘            └──────────────┘
 └──────────────┘
```

Gracias a que el caso de uso solo conoce la **interfaz**, la flecha de dependencia del adaptador JPA apunta **hacia el núcleo** (lo implementa), y no al revés. Es la **inversión de dependencias** aplicada a la arquitectura.

## Ejemplo con Spring Boot

Caso de uso: registrar un cliente con email único.

### Estructura de paquetes

```
com.ejemplo.tienda.cliente
├── dominio
│   ├── Cliente.java                       ← Java puro
│   └── EmailYaRegistradoException.java
├── aplicacion
│   ├── RegistrarCliente.java              ← caso de uso
│   └── puerto
│       └── ClienteRepositorio.java        ← puerto de salida (interfaz)
└── infraestructura
    ├── web
    │   ├── ClienteController.java         ← adaptador de entrada
    │   └── RegistrarClienteRequest.java
    ├── persistencia
    │   ├── ClienteJpaEntity.java          ← entidad JPA (detalle técnico)
    │   ├── ClienteJpaRepository.java      ← Spring Data
    │   └── ClienteRepositorioJpa.java     ← adaptador de salida
    └── ClienteConfig.java                 ← conecta todo con Spring
```

### Dominio: Java puro, sin anotaciones de Spring ni de JPA

```java
package com.ejemplo.tienda.cliente.dominio;

import java.util.Objects;

public record Cliente(Long id, String nombre, String email) {

    public Cliente {
        Objects.requireNonNull(nombre, "El nombre es obligatorio");
        Objects.requireNonNull(email, "El email es obligatorio");
        email = email.strip().toLowerCase();
    }

    public static Cliente nuevo(String nombre, String email) {
        return new Cliente(null, nombre, email);
    }
}
```

```java
package com.ejemplo.tienda.cliente.dominio;

public class EmailYaRegistradoException extends RuntimeException {

    public EmailYaRegistradoException(String email) {
        super("El email " + email + " ya está registrado");
    }
}
```

El **constructor compacto** del `record` (`public Cliente { ... }`) valida y normaliza los datos: es imposible crear un `Cliente` inválido.

### Aplicación: el puerto y el caso de uso

```java
package com.ejemplo.tienda.cliente.aplicacion.puerto;

import com.ejemplo.tienda.cliente.dominio.Cliente;

public interface ClienteRepositorio {

    boolean existeEmail(String email);

    Cliente guardar(Cliente cliente);
}
```

```java
package com.ejemplo.tienda.cliente.aplicacion;

import com.ejemplo.tienda.cliente.aplicacion.puerto.ClienteRepositorio;
import com.ejemplo.tienda.cliente.dominio.Cliente;
import com.ejemplo.tienda.cliente.dominio.EmailYaRegistradoException;

public class RegistrarCliente {

    private final ClienteRepositorio repositorio;

    public RegistrarCliente(ClienteRepositorio repositorio) {
        this.repositorio = repositorio;
    }

    public Cliente ejecutar(String nombre, String email) {
        Cliente nuevo = Cliente.nuevo(nombre, email);
        if (repositorio.existeEmail(nuevo.email())) {
            throw new EmailYaRegistradoException(nuevo.email());
        }
        return repositorio.guardar(nuevo);
    }
}
```

Fíjate: **ni una sola importación de Spring o de JPA**. Este código se puede probar con un repositorio falso en memoria, en milisegundos.

### Infraestructura: el adaptador de persistencia

```java
package com.ejemplo.tienda.cliente.infraestructura.persistencia;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "clientes")
class ClienteJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    @Column(nullable = false)
    String nombre;

    @Column(nullable = false, unique = true)
    String email;

    protected ClienteJpaEntity() {
    }

    ClienteJpaEntity(String nombre, String email) {
        this.nombre = nombre;
        this.email = email;
    }
}
```

```java
package com.ejemplo.tienda.cliente.infraestructura.persistencia;

import org.springframework.data.jpa.repository.JpaRepository;

interface ClienteJpaRepository extends JpaRepository<ClienteJpaEntity, Long> {

    boolean existsByEmail(String email);
}
```

```java
package com.ejemplo.tienda.cliente.infraestructura.persistencia;

import org.springframework.stereotype.Component;

import com.ejemplo.tienda.cliente.aplicacion.puerto.ClienteRepositorio;
import com.ejemplo.tienda.cliente.dominio.Cliente;

@Component
class ClienteRepositorioJpa implements ClienteRepositorio {

    private final ClienteJpaRepository jpa;

    ClienteRepositorioJpa(ClienteJpaRepository jpa) {
        this.jpa = jpa;
    }

    @Override
    public boolean existeEmail(String email) {
        return jpa.existsByEmail(email);
    }

    @Override
    public Cliente guardar(Cliente cliente) {
        var guardado = jpa.save(new ClienteJpaEntity(cliente.nombre(), cliente.email()));
        return new Cliente(guardado.id, guardado.nombre, guardado.email);
    }
}
```

Las clases de persistencia no son `public`: **nadie fuera de su paquete puede usarlas**. El resto de la aplicación solo ve el puerto `ClienteRepositorio`.

### Infraestructura: configuración y adaptador web

Como `RegistrarCliente` no tiene anotaciones de Spring, se registra como *bean* en una clase de configuración. Aquí también se declara la transacción:

```java
package com.ejemplo.tienda.cliente.infraestructura;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.ejemplo.tienda.cliente.aplicacion.RegistrarCliente;
import com.ejemplo.tienda.cliente.aplicacion.puerto.ClienteRepositorio;

@Configuration
class ClienteConfig {

    @Bean
    RegistrarCliente registrarCliente(ClienteRepositorio repositorio) {
        return new RegistrarCliente(repositorio);
    }
}
```

```java
package com.ejemplo.tienda.cliente.infraestructura.web;

import java.net.URI;

import org.springframework.http.ResponseEntity;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.ejemplo.tienda.cliente.aplicacion.RegistrarCliente;

import jakarta.validation.Valid;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

@RestController
@RequestMapping("/api/v1/clientes")
class ClienteController {

    record RegistrarClienteRequest(@NotBlank String nombre, @NotBlank @Email String email) {}

    record ClienteResponse(Long id, String nombre, String email) {}

    private final RegistrarCliente registrarCliente;

    ClienteController(RegistrarCliente registrarCliente) {
        this.registrarCliente = registrarCliente;
    }

    @PostMapping
    @Transactional
    public ResponseEntity<ClienteResponse> registrar(@Valid @RequestBody RegistrarClienteRequest peticion) {
        var cliente = registrarCliente.ejecutar(peticion.nombre(), peticion.email());
        var respuesta = new ClienteResponse(cliente.id(), cliente.nombre(), cliente.email());
        return ResponseEntity.created(URI.create("/api/v1/clientes/" + cliente.id())).body(respuesta);
    }
}
```

> Poner `@Transactional` en el adaptador de entrada es una de varias opciones. Otra muy común es permitir anotaciones de Spring (`@Service`, `@Transactional`) en la capa de aplicación, aceptando esa dependencia a cambio de menos código de configuración. Elige una y aplícala en todo el proyecto.

### Ventajas e inconvenientes

| Ventajas | Inconvenientes |
|---|---|
| Las reglas de negocio son Java puro, fáciles de leer y de probar. | Más clases y más conversiones (dominio ↔ JPA ↔ DTO). |
| Cambiar la base de datos o el *framework* afecta solo a los adaptadores. | Más difícil de entender al principio. |
| Las dependencias son explícitas y controladas. | Puede ser excesivo para un CRUD sencillo. |

> **No hay una arquitectura "mejor" en abstracto.** Para un CRUD sencillo, la arquitectura por capas basta. Cuando las reglas de negocio son ricas y la aplicación va a vivir muchos años, la arquitectura limpia o hexagonal compensa su coste.

## *Package by feature*

### *Package by layer* frente a *package by feature*

```
Por capa                           Por funcionalidad
──────────────                     ─────────────────
controller/                        cliente/
  ClienteController                  ClienteController
  PedidoController                   ClienteService
service/                             ClienteRepository
  ClienteService                     Cliente
  PedidoService                    pedido/
repository/                          PedidoController
  ClienteRepository                  PedidoService
  PedidoRepository                   PedidoRepository
model/                               Pedido
  Cliente
  Pedido
```

En **package by layer**, el primer nivel de carpetas dice **qué tipo de clase** es. En **package by feature**, dice **de qué parte del negocio** trata.

### Ventajas de organizar por funcionalidad

- **Alta cohesión:** todo lo que cambia junto está junto. Para modificar "pedidos" no hay que saltar entre cinco carpetas.
- **Encapsulación real:** las clases que no necesitan verse desde fuera pueden ser **package-private** (sin `public`). Con *package by layer* todo tiene que ser `public`, porque el controlador y el servicio están en paquetes distintos.
- **Visión del negocio:** al abrir el proyecto, la estructura cuenta qué hace la aplicación.
- **Camino hacia la modularidad:** un paquete bien aislado se puede convertir más adelante en un módulo o un microservicio.

### Combinación con capas o con arquitectura hexagonal

Ambas ideas se combinan: primer nivel por funcionalidad y, **dentro** de cada una, capas o puertos y adaptadores (como en el apartado «Ejemplo con Spring Boot»).

### Reglas para que funcione

- Cada funcionalidad expone **pocas clases públicas** (su API); el resto es interno.
- Una funcionalidad no accede a las clases internas de otra: lo hace a través de su API pública.
- Lo compartido de verdad (configuración global, gestión de errores común) va en un paquete aparte, por ejemplo `comun`.
- Evita las dependencias circulares entre funcionalidades.

> **Spring Modulith** es un proyecto de Spring que ayuda a estructurar aplicaciones por módulos de negocio y permite **verificar en una prueba** que los módulos no se saltan sus límites.

## Principios de diseño que sustentan todo lo anterior

| Principio | En la práctica |
|---|---|
| **Responsabilidad única** (SRP) | Cada clase tiene un único motivo para cambiar. |
| **Abierto/cerrado** (OCP) | Añadir comportamiento nuevo sin modificar el existente (por ejemplo, una nueva implementación de un puerto). |
| **Sustitución de Liskov** (LSP) | Cualquier implementación de una interfaz debe poder usarse en lugar de otra. |
| **Segregación de interfaces** (ISP) | Interfaces pequeñas y específicas mejor que una enorme. |
| **Inversión de dependencias** (DIP) | Los módulos de alto nivel dependen de abstracciones, no de detalles. Es la base de los puertos y adaptadores. |

Juntos se conocen como **SOLID**.

## Resumen

- Los patrones clásicos son: **por capas, dirigida por eventos, microkernel, microservicios y basada en espacio**.
- La arquitectura **por capas** es el punto de partida; mantén las capas **cerradas** y vigila el antipatrón **sumidero**.
- La arquitectura **limpia/hexagonal** pone el dominio en el centro: los detalles técnicos dependen del núcleo a través de **puertos** (interfaces) y **adaptadores** (implementaciones).
- **Package by feature** agrupa el código por área de negocio y permite ocultar clases internas.
- Elige la arquitectura según la **complejidad real** del problema.

## Referencias

- Mark Richards — Software Architecture Patterns (O'Reilly): <https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/>
- Baeldung — Clean Architecture with Spring Boot: <https://www.baeldung.com/spring-boot-clean-architecture>
- Martin Fowler — Package by Feature: <https://martinfowler.com/bliki/PackageByFeature.html>
- Alistair Cockburn — Hexagonal Architecture: <https://alistair.cockburn.us/hexagonal-architecture/>
- Spring Modulith: <https://spring.io/projects/spring-modulith>
