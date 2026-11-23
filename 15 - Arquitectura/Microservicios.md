# Microservicios

## Punto de partida: la aplicación monolítica

Una **aplicación monolítica** (o *monolito*) es una aplicación que se construye y se despliega como **una sola unidad**. Todo el código (usuarios, catálogo, pedidos, pagos…) vive en el mismo proyecto, se compila en un único ejecutable y se ejecuta en un único proceso.

```
┌──────────────────────────────────────────┐
│              Aplicación única            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ Usuarios │ │ Catálogo │ │ Pedidos  │  │
│  └──────────┘ └──────────┘ └──────────┘  │
│  ┌──────────┐ ┌──────────┐               │
│  │  Pagos   │ │  Envíos  │               │
│  └──────────┘ └──────────┘               │
└────────────────────┬─────────────────────┘
                     │
              ┌──────┴──────┐
              │ Base datos  │
              └─────────────┘
```

Un monolito **no es algo malo**. Es la forma más sencilla de empezar: un solo proyecto, un solo despliegue, llamadas entre módulos que son simples llamadas a métodos. Los problemas aparecen cuando la aplicación y el equipo crecen mucho:

- Un cambio pequeño obliga a volver a desplegar **toda** la aplicación.
- Si una parte necesita más recursos, hay que escalar **toda** la aplicación.
- Un fallo grave en un módulo puede tumbar todo el sistema.
- Con muchos equipos trabajando en el mismo código, es fácil pisarse.
- Cambiar de tecnología es muy difícil, porque afecta a todo.

## ¿Qué son los microservicios?

La **arquitectura de microservicios** construye una aplicación como un **conjunto de servicios pequeños e independientes**. Cada servicio:

- se ocupa de **una capacidad de negocio** concreta (catálogo, pedidos, pagos…),
- se ejecuta en **su propio proceso**,
- se comunica con los demás mediante mecanismos **ligeros**, normalmente HTTP (APIs REST) o mensajería,
- se **despliega de forma independiente**,
- puede usar **su propia tecnología** y **su propia base de datos**.

```
                     ┌──────────────┐
   Clientes ───────▶ │ API Gateway  │
                     └──────┬───────┘
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
   ┌────────────┐    ┌────────────┐    ┌────────────┐
   │  Catálogo  │    │  Pedidos   │───▶│   Pagos    │
   └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
         ▼                 ▼                 ▼
     ┌───────┐         ┌───────┐         ┌───────┐
     │  BD   │         │  BD   │         │  BD   │
     └───────┘         └───────┘         └───────┘
```

## Características de los microservicios

Martin Fowler y James Lewis describieron en 2014 las características comunes de los sistemas basados en microservicios. No todos los sistemas las cumplen todas, pero la mayoría comparten casi todas:

1. **Componentes en forma de servicios.** Cada parte del sistema es un servicio que se puede sustituir o actualizar por separado, en lugar de una librería dentro del mismo programa.
2. **Organizados en torno a capacidades de negocio.** Los servicios (y los equipos) se dividen por áreas del negocio (pedidos, facturación…) y no por capas técnicas (interfaz, lógica, base de datos).
3. **Productos, no proyectos.** El equipo que construye un servicio lo mantiene durante toda su vida ("tú lo construyes, tú lo operas").
4. **Extremos inteligentes, tuberías tontas** (*smart endpoints and dumb pipes*). La lógica está dentro de los servicios; la comunicación entre ellos es simple (HTTP, colas de mensajes ligeras).
5. **Gobierno descentralizado.** Cada equipo elige las herramientas más adecuadas para su servicio.
6. **Gestión de datos descentralizada.** Cada servicio es dueño de sus datos y de su base de datos. Ningún otro servicio accede a ella directamente.
7. **Automatización de la infraestructura.** Pruebas, integración y despliegue continuos (CI/CD). Sin automatización, gestionar decenas de servicios es inviable.
8. **Diseño para el fallo.** En un sistema distribuido, algún servicio fallará. Cada servicio debe tolerar que otro no responda.
9. **Diseño evolutivo.** El sistema está pensado para cambiar: se pueden añadir, reescribir o eliminar servicios sin rehacer todo.

## Principios de diseño

A partir de esas características se suelen aplicar estos principios:

- **Responsabilidad única.** Un servicio hace una cosa y la hace bien.
- **Alta cohesión, bajo acoplamiento.** Lo que cambia junto va junto; lo que es independiente se separa. Un cambio en un servicio no debería obligar a cambiar otros.
- **Contratos estables.** Los servicios se comunican a través de APIs bien definidas y versionadas. El interior de cada servicio puede cambiar libremente mientras el contrato se respete.
- **Despliegue independiente.** Si para desplegar el servicio A tienes que desplegar a la vez el B, no son realmente independientes.
- **Base de datos por servicio.** Compartir base de datos entre servicios crea un acoplamiento fuerte.
- **Observabilidad.** Registros (*logs*), métricas y trazas que permitan seguir una petición a través de varios servicios.

## Beneficios

| Beneficio | Explicación |
|---|---|
| **Despliegues independientes** | Se puede publicar una mejora en el servicio de pagos sin tocar el catálogo. |
| **Escalado selectivo** | Si el catálogo recibe muchas visitas, se ponen más instancias solo del catálogo. |
| **Aislamiento de fallos** | Si el servicio de recomendaciones cae, el resto de la tienda puede seguir funcionando. |
| **Libertad tecnológica** | Cada servicio puede usar el lenguaje, la versión o la base de datos que mejor le encaje. |
| **Equipos autónomos** | Equipos pequeños, cada uno responsable de sus servicios, que avanzan sin esperar a los demás. |
| **Código más pequeño** | Cada servicio es más fácil de entender que un monolito enorme. |

## Inconvenientes y retos

Los microservicios **no son gratis**. Cambian la complejidad de sitio: el código de cada servicio es más simple, pero el sistema en conjunto es más complejo.

| Reto | Explicación |
|---|---|
| **Complejidad de sistema distribuido** | Las llamadas por red son más lentas que una llamada a un método y pueden fallar. |
| **Consistencia de datos** | Con una base de datos por servicio, no hay transacciones que abarquen varios servicios. Hay que asumir *consistencia eventual* y usar patrones como **Saga**. |
| **Pruebas más difíciles** | Probar un flujo completo implica levantar varios servicios. |
| **Operación** | Más servicios que desplegar, monitorizar, actualizar y asegurar. |
| **Depuración** | Seguir un error a través de varios servicios requiere trazabilidad distribuida. |
| **Diseño de fronteras** | Decidir dónde termina un servicio y empieza otro es difícil. Una mala división produce servicios que se llaman sin parar entre sí. |

> **Consejo práctico:** muchos equipos empiezan con un **monolito bien modularizado** y extraen microservicios solo cuando hay una razón clara (escalado, equipos distintos, ritmos de despliegue diferentes). Martin Fowler llama a esta idea *MonolithFirst*.

## Monolito frente a microservicios

| Aspecto | Monolito | Microservicios |
|---|---|---|
| Despliegue | Una sola unidad | Cada servicio por separado |
| Escalado | Toda la aplicación | Cada servicio por separado |
| Tecnología | Una para todo | Puede variar por servicio |
| Base de datos | Normalmente una compartida | Una por servicio |
| Comunicación interna | Llamadas a métodos | Llamadas por red (HTTP, mensajería) |
| Complejidad inicial | Baja | Alta |
| Complejidad al crecer | Alta | Repartida |
| Fallos | Un fallo puede afectar a todo | Fallos más aislados (si se diseña bien) |

## Piezas habituales en una arquitectura de microservicios

| Pieza | Para qué sirve |
|---|---|
| **API Gateway** | Punto de entrada único para los clientes. Enruta peticiones, aplica seguridad, limita el tráfico. |
| **Descubrimiento de servicios** (*service discovery*) | Permite que un servicio encuentre la dirección de otro, aunque cambie. |
| **Configuración centralizada** | Guarda la configuración de todos los servicios en un único sitio. |
| **Balanceador de carga** | Reparte las peticiones entre varias instancias de un servicio. |
| **Circuit breaker** (cortacircuitos) | Si un servicio falla repetidamente, deja de llamarlo durante un tiempo para no saturarlo y responde con una alternativa. |
| **Mensajería** (colas, eventos) | Comunicación asíncrona entre servicios: uno publica un evento ("pedido creado") y otros reaccionan. |
| **Contenedores** (Docker) y **orquestadores** (Kubernetes) | Empaquetar, desplegar y escalar los servicios. |
| **Trazabilidad distribuida** | Seguir una petición a lo largo de todos los servicios por los que pasa. |

### Comunicación síncrona y asíncrona

- **Síncrona:** el servicio A llama al B (normalmente por HTTP) y **espera** la respuesta. Sencillo de entender, pero si B está caído o lento, A también se ve afectado.
- **Asíncrona:** A publica un mensaje o evento y **sigue trabajando**. B lo procesa cuando puede. Desacopla más los servicios, pero el flujo es más difícil de seguir.

## Microservicios y Spring

Spring Boot es una de las opciones más utilizadas para construir microservicios en Java, porque:

- crea aplicaciones **autónomas** que incluyen su propio servidor web y se arrancan con un solo comando,
- facilita crear **APIs REST**,
- ofrece **Spring Boot Actuator** para exponer información de salud y métricas de cada servicio,
- y el proyecto **Spring Cloud** aporta piezas para sistemas distribuidos (configuración centralizada, *gateway*, descubrimiento de servicios, *circuit breakers*…).

En los documentos de Spring Boot ([Spring y Spring Boot](<../14 - Spring-Boot/Fundamentos/Spring y Spring-Boot.md>), [Arquitectura por capas](<Arquitectura por capas.md>), [Spring Data](<../14 - Spring-Boot/Persistencia/Spring data.md>), [Testing y TDD](<../14 - Spring-Boot/Testing/Testing y TDD.md>) y [Docker](<../12 - Herramientas/Docker.md>)) has aprendido a construir y empaquetar **un servicio**, que es la pieza básica de cualquier arquitectura de microservicios.

## Resumen

- Un **monolito** es una aplicación que se despliega como una sola unidad. Es sencillo y adecuado para empezar.
- Los **microservicios** dividen la aplicación en servicios pequeños, independientes, organizados por capacidad de negocio y con sus propios datos.
- Aportan despliegue y escalado independientes, aislamiento de fallos y autonomía de equipos.
- A cambio, añaden la complejidad de un **sistema distribuido**: red, consistencia de datos, operación y depuración.
- No son una meta en sí mismos: se adoptan cuando sus beneficios compensan sus costes.

## Referencias

- HowToDoInJava — Microservices: definition, principles and benefits: <https://howtodoinjava.com/microservices/microservices-definition-principles-benefits/>
- Martin Fowler y James Lewis — Microservices: <https://martinfowler.com/articles/microservices.html>
- Red Hat — What are microservices?: <https://www.redhat.com/en/topics/microservices/what-are-microservices>
- Martin Fowler — MonolithFirst: <https://martinfowler.com/bliki/MonolithFirst.html>
- Spring Cloud: <https://spring.io/projects/spring-cloud>
