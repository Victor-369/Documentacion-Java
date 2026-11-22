# Spring y Spring Boot

## ¿Qué es Spring?

**Spring Framework** es un *framework* de código abierto para construir aplicaciones Java. Un *framework* es un "esqueleto" que ya resuelve problemas comunes (gestionar objetos, recibir peticiones HTTP, acceder a bases de datos, seguridad…) para que tú te centres en la lógica de tu aplicación.

La idea central de Spring es el **contenedor de Inversión de Control (IoC)**: en lugar de que tu código cree y conecte todos los objetos a mano, Spring los crea, los configura y los conecta por ti. Esto se explica en detalle en el documento [IoC e Inyección de Dependencias (los *beans* de Spring)](<IoC e inyeccion de dependencias.md>).

Alrededor del núcleo hay muchos módulos y proyectos:

| Proyecto / módulo | Para qué sirve |
|---|---|
| **Spring Core** | Contenedor IoC, inyección de dependencias. |
| **Spring Web MVC** | Aplicaciones web y APIs REST. |
| **Spring Data** | Acceso a bases de datos con muy poco código. |
| **Spring Security** | Autenticación y autorización. |
| **Spring Cloud** | Herramientas para microservicios y sistemas distribuidos. |
| **Spring Boot** | Arranque y configuración rápida de todo lo anterior. |

## ¿Qué es Spring Boot?

Configurar una aplicación Spring "a mano" exige muchas decisiones: qué librerías y versiones usar, qué servidor web, cómo conectarlo todo… **Spring Boot** elimina casi todo ese trabajo. Sus pilares son:

### Autoconfiguración

Spring Boot mira qué librerías tienes en el proyecto y **configura automáticamente** lo necesario. Por ejemplo, si añades la dependencia de web, arranca un servidor web y configura la conversión a JSON sin que escribas nada.

La autoconfiguración **no es invasiva**: si defines tu propia configuración para algo, Spring Boot se aparta y usa la tuya.

### *Starters*

Un **starter** es una dependencia que agrupa todo lo necesario para una funcionalidad. En lugar de añadir diez librerías con sus versiones, añades una:

| Starter | Qué aporta |
|---|---|
| `spring-boot-starter-webmvc` | Spring MVC, servidor Tomcat embebido y Jackson (JSON). Para APIs REST. |
| `spring-boot-starter-data-jpa` | Acceso a bases de datos relacionales con JPA/Hibernate. |
| `spring-boot-starter-validation` | Validación de datos (`@NotBlank`, `@Positive`…). |
| `spring-boot-starter-security` | Seguridad. |
| `spring-boot-starter-actuator` | *Endpoints* de salud, métricas e información de la aplicación. |

> **Nota sobre nombres:** en Spring Boot 4 el *starter* para aplicaciones web con Spring MVC se llama `spring-boot-starter-webmvc`. El nombre antiguo, `spring-boot-starter-web`, sigue existiendo pero está obsoleto (*deprecated*). Además, cada tecnología tiene ahora su propio *starter* de pruebas, como `spring-boot-starter-webmvc-test`.

### Servidor embebido

Una aplicación Spring Boot incluye su propio servidor web (Tomcat por defecto). No hace falta instalar ni configurar un servidor aparte: se genera un `.jar` y se ejecuta con `java -jar`.

### Valores por defecto razonables

Spring Boot es "opinado" (*opinionated*): toma decisiones sensatas por ti (puerto 8080, formato JSON, *logs* en consola…). Todas se pueden cambiar mediante configuración.

### Versiones de referencia

| Componente | Versión |
|---|---|
| Spring Boot | 4.x (basado en Spring Framework 7) |
| Java mínimo para Spring Boot 4 | 17 |
| Java usado en esta documentación | 25 |
| Librería JSON | Jackson 3 |

## Crear un proyecto con Spring Initializr

**Spring Initializr** es una web oficial que genera el esqueleto de un proyecto: <https://start.spring.io>

1. Abre <https://start.spring.io>.
2. Rellena las opciones:

   | Opción | Valor |
   |---|---|
   | Project | **Maven** (o Gradle, si lo prefieres) |
   | Language | **Java** |
   | Spring Boot | La última versión estable 4.x (sin `SNAPSHOT` ni `M`/`RC`) |
   | Group | `com.ejemplo` |
   | Artifact | `tienda` |
   | Packaging | **Jar** |
   | Java | **25** |

3. En **Dependencies**, añade **Spring Web**.
4. Pulsa **Generate**. Se descarga un `.zip`.
5. Descomprímelo y ábrelo con tu IDE.

> Los IDE más comunes (IntelliJ IDEA, VS Code, Eclipse con Spring Tools) también permiten crear proyectos con Initializr sin salir del editor.

## Estructura del proyecto generado

```
tienda/
├── mvnw                     ← Maven Wrapper (Linux/macOS)
├── mvnw.cmd                 ← Maven Wrapper (Windows)
├── pom.xml                  ← Dependencias y configuración de construcción
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/ejemplo/tienda/
    │   │       └── TiendaApplication.java   ← Clase principal
    │   └── resources/
    │       ├── application.properties       ← Configuración
    │       ├── static/                      ← Ficheros estáticos (HTML, CSS, JS)
    │       └── templates/                   ← Plantillas de vistas (si se usan)
    └── test/
        └── java/
            └── com/ejemplo/tienda/
                └── TiendaApplicationTests.java
```

El **Maven Wrapper** (`mvnw`) descarga automáticamente la versión correcta de Maven, así que no necesitas instalarlo.

### El fichero `pom.xml`

Es el fichero de configuración de Maven. Las partes clave son:

```xml
<!-- Hereda la configuración de Spring Boot (versiones de librerías, plugins...) -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.2</version> <!-- usa la versión que te haya generado Initializr -->
    <relativePath/>
</parent>

<groupId>com.ejemplo</groupId>
<artifactId>tienda</artifactId>
<version>0.0.1-SNAPSHOT</version>

<properties>
    <java.version>25</java.version>
</properties>

<dependencies>
    <!-- API REST con Spring MVC + Tomcat + Jackson -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- Pruebas -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Permite ejecutar y empaquetar la aplicación -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

Fíjate en que **las dependencias no llevan versión**: la hereda del `parent`, que garantiza que todas las librerías son compatibles entre sí.

## La clase principal

```java
package com.ejemplo.tienda;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TiendaApplication {

    public static void main(String[] args) {
        SpringApplication.run(TiendaApplication.class, args);
    }
}
```

- `SpringApplication.run(...)` arranca Spring: crea el contenedor, configura todo y levanta el servidor web.
- `@SpringBootApplication` es un atajo que equivale a tres anotaciones:

| Anotación | Qué hace |
|---|---|
| `@SpringBootConfiguration` | Marca la clase como fuente de configuración. |
| `@EnableAutoConfiguration` | Activa la autoconfiguración. |
| `@ComponentScan` | Busca componentes (`@RestController`, `@Service`, `@Repository`…) en este paquete **y en sus subpaquetes**. |

> **Importante:** como `@ComponentScan` busca a partir del paquete de la clase principal, coloca tus clases **dentro** de `com.ejemplo.tienda` o en subpaquetes (`com.ejemplo.tienda.producto`, etc.). Si las pones fuera, Spring no las encontrará.

## Tu primer *endpoint*

Crea el fichero `src/main/java/com/ejemplo/tienda/SaludoController.java`:

```java
package com.ejemplo.tienda;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SaludoController {

    @GetMapping("/saludo")
    public String saludar(@RequestParam(defaultValue = "mundo") String nombre) {
        return "¡Hola, " + nombre + "!";
    }
}
```

- `@RestController` indica que esta clase atiende peticiones HTTP y que lo que devuelven sus métodos se escribe directamente en el cuerpo de la respuesta.
- `@GetMapping("/saludo")` asocia el método a las peticiones `GET /saludo`.
- `@RequestParam` lee un parámetro de la consulta (`?nombre=...`). Si no llega, usa `"mundo"`.

## Ejecutar la aplicación

Desde la carpeta del proyecto:

```bash
# Linux / macOS
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

O bien ejecuta el método `main` de `TiendaApplication` desde el IDE.

Cuando veas en la consola un mensaje parecido a `Tomcat started on port 8080`, prueba:

```bash
curl http://localhost:8080/saludo
# ¡Hola, mundo!

curl "http://localhost:8080/saludo?nombre=Laura"
# ¡Hola, Laura!
```

### Devolver JSON

Si el método devuelve un objeto en lugar de un `String`, Spring Boot lo convierte automáticamente a JSON con Jackson:

```java
package com.ejemplo.tienda;

import java.time.LocalDateTime;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SaludoController {

    record Saludo(String mensaje, LocalDateTime fecha) {}

    @GetMapping("/saludo")
    public Saludo saludar(@RequestParam(defaultValue = "mundo") String nombre) {
        return new Saludo("¡Hola, " + nombre + "!", LocalDateTime.now());
    }
}
```

```bash
curl -i http://localhost:8080/saludo
```

```http
HTTP/1.1 200
Content-Type: application/json

{"mensaje":"¡Hola, mundo!","fecha":"2026-11-22T10:30:00.123456"}
```

Cómo funciona esta conversión se explica en el documento [Jackson y JSON](<../../13-Web-y-APIs/Jackson y json.md>).

## Empaquetar y ejecutar como `.jar`

```bash
./mvnw clean package
java -jar target/tienda-0.0.1-SNAPSHOT.jar
```

El `.jar` contiene tu código, todas las dependencias y el servidor web. Es lo que se despliega en un servidor o se mete en un contenedor Docker.

## Configuración: `application.properties`

El fichero `src/main/resources/application.properties` permite cambiar el comportamiento de la aplicación sin tocar el código:

```properties
# Nombre de la aplicación
spring.application.name=tienda

# Puerto del servidor (por defecto 8080)
server.port=8081

# Nivel de detalle de los logs de tu paquete
logging.level.com.ejemplo.tienda=DEBUG
```

También se puede usar el formato YAML en un fichero `application.yml`:

```yaml
spring:
  application:
    name: tienda
server:
  port: 8081
```

Usa **uno u otro** formato, no los dos a la vez, para evitar confusiones.

### Propiedades propias

Puedes definir tus propias propiedades:

```properties
tienda.nombre=Mi Tienda
tienda.productos-por-pagina=20
```

Y leerlas en una clase con `@ConfigurationProperties`. Un `record` encaja muy bien:

```java
package com.ejemplo.tienda;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "tienda")
public record TiendaProperties(String nombre, int productosPorPagina) {}
```

Para que Spring las registre, añade `@ConfigurationPropertiesScan` a la clase principal:

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class TiendaApplication { ... }
```

Spring convierte automáticamente `productos-por-pagina` (en el fichero) a `productosPorPagina` (en Java).

Para un valor suelto también existe `@Value`:

```java
@Value("${tienda.nombre}")
private String nombreTienda;
```

> `@ConfigurationProperties` es preferible cuando tienes varias propiedades relacionadas: agrupa la configuración, comprueba los tipos y es más fácil de probar.

### Perfiles

Los **perfiles** permiten tener configuraciones distintas para cada entorno (desarrollo, pruebas, producción):

```
src/main/resources/
├── application.properties          ← común a todos
├── application-dev.properties      ← solo con el perfil "dev"
└── application-prod.properties     ← solo con el perfil "prod"
```

Se activa un perfil con la propiedad `spring.profiles.active`:

```bash
java -jar target/tienda-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

Las propiedades del perfil activo sustituyen a las del fichero común.

## Hilos virtuales

Java 21 introdujo los **hilos virtuales** (*virtual threads*), hilos muy ligeros gestionados por la propia JVM. Permiten atender muchas peticiones simultáneas que pasan tiempo esperando (a una base de datos, a otra API…) con pocos recursos. En Spring Boot se activan con una propiedad:

```properties
spring.threads.virtual.enabled=true
```

## Herramientas útiles

### Spring Boot DevTools

Reinicia la aplicación automáticamente cada vez que cambias el código. Se añade como dependencia (en Initializr: **Spring Boot DevTools**). Solo actúa en desarrollo; se desactiva al ejecutar el `.jar` empaquetado.

### Spring Boot Actuator

Añade *endpoints* para supervisar la aplicación. Con la dependencia `spring-boot-starter-actuator`:

```bash
curl http://localhost:8080/actuator/health
# {"status":"UP"}
```

Por defecto, por HTTP solo se expone `health`. Para exponer más:

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

> No expongas todos los *endpoints* de Actuator en producción sin protegerlos: algunos muestran información sensible.

### Ver qué se ha autoconfigurado

Si arrancas la aplicación con `--debug`, Spring Boot muestra un informe de qué autoconfiguraciones se han aplicado y por qué.

## Resumen

- **Spring** es un *framework* cuyo núcleo es un contenedor que crea y conecta los objetos de la aplicación.
- **Spring Boot** simplifica Spring con **autoconfiguración**, **starters**, **servidor embebido** y valores por defecto razonables.
- Los proyectos se crean con **Spring Initializr** (<https://start.spring.io>).
- `@SpringBootApplication` arranca la aplicación y busca componentes en su paquete y subpaquetes.
- `@RestController` + `@GetMapping` crean un *endpoint*; si devuelves un objeto, se convierte a JSON.
- La configuración va en `application.properties` (o `.yml`), y puede variar por **perfil**.

## Referencias

- Baeldung — Learn Spring Boot: <https://www.baeldung.com/spring-boot>
- Spring Quickstart Guide: <https://spring.io/quickstart>
- Spring Initializr: <https://start.spring.io>
- Documentación de referencia de Spring Boot: <https://docs.spring.io/spring-boot/reference/>
- Guía de migración a Spring Boot 4 (wiki oficial): <https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide>
