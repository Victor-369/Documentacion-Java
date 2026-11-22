# IoC e Inyección de Dependencias (los *beans* de Spring)

Este es el concepto más importante de Spring. Si lo entiendes bien, el resto del *framework* tiene sentido.

## El problema: clases que crean sus propias dependencias

Una **dependencia** es un objeto que otra clase necesita para hacer su trabajo. Imagina un servicio que envía avisos de pedidos por email:

```java
// SinInyeccion.java  →  java SinInyeccion.java
class EmailNotificador {
    void enviar(String mensaje) {
        IO.println("[EMAIL] " + mensaje);
    }
}

class PedidoService {
    // La clase crea ella misma su dependencia
    private final EmailNotificador notificador = new EmailNotificador();

    void confirmar(int idPedido) {
        notificador.enviar("Pedido " + idPedido + " confirmado");
    }
}

void main() {
    new PedidoService().confirmar(15);
}
```

Funciona, pero tiene varios problemas:

- **Acoplamiento fuerte:** `PedidoService` está "atado" a `EmailNotificador`. Si mañana quieres avisar por SMS, tienes que modificar `PedidoService`.
- **Difícil de probar:** en una prueba no puedes sustituir el email real por uno falso, porque el `new` está escondido dentro.
- **Responsabilidades mezcladas:** `PedidoService` decide *qué* hace (confirmar pedidos) y también *con qué* lo hace (qué notificador usar).

## La solución: inyección de dependencias

La **inyección de dependencias** (*Dependency Injection*, DI) consiste en que una clase **recibe** sus dependencias desde fuera en lugar de crearlas. Lo más habitual es recibirlas por el **constructor**. Además, la clase depende de una **interfaz** y no de una implementación concreta:

```java
// ConInyeccion.java  →  java ConInyeccion.java
interface Notificador {
    void enviar(String mensaje);
}

class EmailNotificador implements Notificador {
    public void enviar(String mensaje) {
        IO.println("[EMAIL] " + mensaje);
    }
}

class SmsNotificador implements Notificador {
    public void enviar(String mensaje) {
        IO.println("[SMS] " + mensaje);
    }
}

class PedidoService {
    private final Notificador notificador;

    // La dependencia llega desde fuera
    PedidoService(Notificador notificador) {
        this.notificador = notificador;
    }

    void confirmar(int idPedido) {
        notificador.enviar("Pedido " + idPedido + " confirmado");
    }
}

void main() {
    // Alguien de fuera decide qué implementación usar y la "inyecta"
    var conEmail = new PedidoService(new EmailNotificador());
    conEmail.confirmar(15);   // [EMAIL] Pedido 15 confirmado

    var conSms = new PedidoService(new SmsNotificador());
    conSms.confirmar(16);     // [SMS] Pedido 16 confirmado
}
```

Ahora `PedidoService` no sabe ni le importa cómo se envía el aviso. Para probarlo, basta con pasarle un `Notificador` de prueba:

```java
class NotificadorFalso implements Notificador {
    String ultimoMensaje;
    public void enviar(String mensaje) { ultimoMensaje = mensaje; }
}
```

## Inversión de Control (IoC)

En el ejemplo anterior, el método `main` hace de "montador": crea los objetos y los conecta. En una aplicación real con cientos de clases, hacer esto a mano sería muy pesado.

La **Inversión de Control** (*Inversion of Control*, IoC) es el principio de **ceder a un *framework* el control** de crear y conectar los objetos. Ya no es tu código el que llama a `new` y decide el orden; es el *framework* quien lo hace y te entrega los objetos listos.

> **IoC** es el principio general ("otro controla el flujo y la creación de objetos"). **DI** es la técnica concreta con la que Spring lo aplica ("te paso las dependencias que necesitas").

En Spring, quien asume ese control es el **contenedor IoC**.

## El contenedor IoC y los *beans*

El **contenedor IoC** de Spring es el componente que:

1. **Encuentra** las clases que debe gestionar.
2. **Crea** sus objetos (instancias).
3. **Inyecta** las dependencias de cada uno.
4. **Gestiona** su ciclo de vida (creación, uso y destrucción).

En código, el contenedor se representa con la interfaz `ApplicationContext`.

Un **bean** es, sencillamente, **un objeto creado y gestionado por el contenedor de Spring**. No es un tipo especial de clase: cualquier clase Java puede convertirse en un *bean* si se lo indicas a Spring.

```
  Tus clases                    Contenedor IoC                 Objetos listos
  + anotaciones  ──────────▶  (ApplicationContext)  ──────────▶  (beans)
  o configuración              crea y conecta                 con sus dependencias
```

## Cómo declarar *beans*

Hay dos formas principales.

### Opción 1: anotar la clase (escaneo de componentes)

Marcas la clase con `@Component` (o una de sus variantes) y Spring la detecta automáticamente gracias al `@ComponentScan` que incluye `@SpringBootApplication`.

```java
package com.ejemplo.tienda.notificacion;

import org.springframework.stereotype.Component;

@Component
public class EmailNotificador implements Notificador {

    @Override
    public void enviar(String mensaje) {
        System.out.println("[EMAIL] " + mensaje);
    }
}
```

Existen **anotaciones estereotipo**, que funcionan como `@Component` pero además indican el **papel** de la clase:

| Anotación | Papel | Capa |
|---|---|---|
| `@Component` | Componente genérico | Cualquiera |
| `@Service` | Lógica de negocio | Servicio |
| `@Repository` | Acceso a datos. Además, traduce las excepciones de acceso a datos a las de Spring. | Persistencia |
| `@Controller` | Controlador web que devuelve vistas | Presentación |
| `@RestController` | Controlador web que devuelve datos (JSON). Es `@Controller` + `@ResponseBody`. | Presentación |
| `@Configuration` | Clase que define *beans* con métodos `@Bean` | Configuración |

### Opción 2: métodos `@Bean` en una clase `@Configuration`

Útil cuando **no puedes anotar la clase** (por ejemplo, porque pertenece a una librería externa) o cuando crearla requiere lógica:

```java
package com.ejemplo.tienda.config;

import java.time.Clock;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    // El objeto devuelto se registra como bean. Su nombre es "reloj".
    @Bean
    public Clock reloj() {
        return Clock.systemDefaultZone();
    }
}
```

A partir de aquí, cualquier *bean* que necesite un `Clock` lo recibirá automáticamente.

## Cómo inyectar dependencias

### Inyección por constructor (la recomendada)

```java
package com.ejemplo.tienda.pedido;

import org.springframework.stereotype.Service;

import com.ejemplo.tienda.notificacion.Notificador;

@Service
public class PedidoService {

    private final Notificador notificador;

    public PedidoService(Notificador notificador) {
        this.notificador = notificador;
    }

    public void confirmar(int idPedido) {
        notificador.enviar("Pedido " + idPedido + " confirmado");
    }
}
```

Spring ve que el constructor necesita un `Notificador`, busca un *bean* de ese tipo (`EmailNotificador`) y lo pasa al crear `PedidoService`.

> Si la clase tiene **un único constructor**, no hace falta escribir `@Autowired`. Spring lo usa automáticamente.

Ventajas de la inyección por constructor:

- Las dependencias pueden ser `final`: no cambian después de crear el objeto.
- Es imposible crear el objeto sin sus dependencias: no habrá `NullPointerException` por olvidos.
- Se ve claramente qué necesita la clase. Si el constructor tiene demasiados parámetros, es una señal de que la clase hace demasiadas cosas.
- Se puede probar sin Spring: `new PedidoService(new NotificadorFalso())`.

### Inyección por *setter*

```java
@Service
public class InformeService {

    private Notificador notificador;

    @Autowired
    public void setNotificador(Notificador notificador) {
        this.notificador = notificador;
    }
}
```

Tiene sentido para dependencias **opcionales** o que pueden cambiar. En la mayoría de casos, mejor por constructor.

### Inyección en el atributo (*field injection*)

```java
@Service
public class InformeService {

    @Autowired
    private Notificador notificador;   // no recomendado
}
```

Es la más corta, pero **no se recomienda**: el atributo no puede ser `final`, la dependencia queda "escondida" y la clase no se puede crear correctamente sin Spring (por ejemplo, en una prueba unitaria). La verás en código antiguo y en muchos tutoriales.

## Varias implementaciones de la misma interfaz

Si hay dos *beans* que implementan `Notificador`, Spring no sabe cuál elegir y la aplicación **no arranca**: lanza un error indicando que esperaba un único *bean* y ha encontrado dos.

```java
@Component
public class EmailNotificador implements Notificador { ... }

@Component
public class SmsNotificador implements Notificador { ... }
```

Hay varias formas de resolverlo:

### `@Primary`: una implementación por defecto

```java
@Component
@Primary
public class EmailNotificador implements Notificador { ... }
```

Cuando no se indique otra cosa, se usará `EmailNotificador`.

### `@Qualifier`: elegir por nombre

Por defecto, el nombre de un *bean* anotado es el de la clase con la primera letra en minúscula (`smsNotificador`).

```java
@Service
public class AlertaService {

    private final Notificador notificador;

    public AlertaService(@Qualifier("smsNotificador") Notificador notificador) {
        this.notificador = notificador;
    }
}
```

### Recibir todas las implementaciones

```java
@Service
public class DifusionService {

    private final List<Notificador> notificadores;

    public DifusionService(List<Notificador> notificadores) {
        this.notificadores = notificadores;
    }

    public void avisarATodos(String mensaje) {
        notificadores.forEach(n -> n.enviar(mensaje));
    }
}
```

Spring inyecta una lista con **todos** los *beans* de tipo `Notificador`.

## Ámbitos (*scopes*) de un *bean*

El **ámbito** indica cuántas instancias crea Spring de un *bean* y cuánto tiempo viven.

| Ámbito | Instancias | Uso típico |
|---|---|---|
| `singleton` (**por defecto**) | Una sola para toda la aplicación. | Servicios, repositorios, controladores. |
| `prototype` | Una nueva cada vez que se pide. | Objetos con estado propio que no deben compartirse. |
| `request` | Una por cada petición HTTP. | Datos ligados a una petición. |
| `session` | Una por cada sesión HTTP. | Datos ligados a la sesión de un usuario. |

```java
@Component
@Scope("prototype")
public class GeneradorInforme { ... }
```

> **Importante:** como los *singleton* se comparten entre todas las peticiones (que pueden llegar a la vez desde distintos hilos), **no guardes en sus atributos datos que cambien con cada petición**. Un servicio debería contener solo sus dependencias (`final`) y no "estado" de un usuario concreto.

## Ciclo de vida de un *bean*

```
 Spring crea el objeto ──▶ Inyecta dependencias ──▶ @PostConstruct ──▶ En uso ──▶ @PreDestroy ──▶ Destruido
```

Puedes ejecutar código justo después de que el *bean* esté listo, o justo antes de destruirlo (al parar la aplicación):

```java
package com.ejemplo.tienda.cache;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class CacheProductos {

    @PostConstruct
    void cargar() {
        System.out.println("Cargando caché de productos...");
    }

    @PreDestroy
    void liberar() {
        System.out.println("Liberando caché de productos...");
    }
}
```

> `@PreDestroy` no se llama en los *beans* `prototype`: Spring los crea y los entrega, pero no gestiona su destrucción.

## Dependencias circulares

Si `A` necesita a `B` y `B` necesita a `A` (directa o indirectamente) mediante sus constructores, Spring no puede crear ninguno de los dos y **la aplicación no arranca**. Spring Boot, por defecto, no permite referencias circulares.

Casi siempre indica un **problema de diseño**: las dos clases están demasiado acopladas. La solución habitual es extraer la parte común a una tercera clase de la que dependan ambas.

## Ver el contenedor en acción

Este ejemplo muestra cómo pedir *beans* directamente al contenedor. En una aplicación normal casi nunca harás esto (para eso está la inyección), pero ayuda a entender qué ocurre por dentro:

```java
package com.ejemplo.tienda;

import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

import com.ejemplo.tienda.pedido.PedidoService;

// Un CommandLineRunner se ejecuta al terminar de arrancar la aplicación
@Component
public class InspectorDeBeans implements CommandLineRunner {

    private final ApplicationContext contexto;

    public InspectorDeBeans(ApplicationContext contexto) {
        this.contexto = contexto;
    }

    @Override
    public void run(String... args) {
        PedidoService servicio = contexto.getBean(PedidoService.class);
        servicio.confirmar(15);

        System.out.println("Número de beans en el contenedor: "
                + contexto.getBeanDefinitionCount());
    }
}
```

Al arrancar verás el aviso del pedido y el número total de *beans*: muchos de ellos los ha creado la autoconfiguración de Spring Boot.

## Resumen

- Una **dependencia** es un objeto que otra clase necesita.
- **Inyección de dependencias:** la clase recibe sus dependencias desde fuera (mejor por constructor y mejor como interfaces).
- **Inversión de Control:** un *framework* se encarga de crear y conectar los objetos.
- El **contenedor IoC** de Spring (`ApplicationContext`) crea y gestiona los **beans**.
- Se declaran *beans* con `@Component` y sus estereotipos (`@Service`, `@Repository`, `@RestController`…) o con métodos `@Bean` dentro de una clase `@Configuration`.
- Con varias implementaciones: `@Primary`, `@Qualifier` o inyectar una `List`.
- El ámbito por defecto es **singleton**: no guardes en él estado de cada petición.

## Referencias

- Baeldung — Intro to Inversion of Control and Dependency Injection with Spring: <https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring>
- Baeldung — What Is a Spring Bean?: <https://www.baeldung.com/spring-bean>
- Spring Framework — The IoC Container: <https://docs.spring.io/spring-framework/reference/core/beans.html>
- Martin Fowler — Inversion of Control Containers and the Dependency Injection pattern: <https://martinfowler.com/articles/injection.html>
