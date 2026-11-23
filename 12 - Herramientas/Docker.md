# Docker y *multi-stage build*

## ¿Qué problema resuelve Docker?

"En mi ordenador funciona" es una frase clásica. La aplicación depende de una versión concreta de Java, de variables de entorno, de una base de datos… y cada máquina es distinta.

**Docker** empaqueta la aplicación **junto con todo lo que necesita para ejecutarse** (sistema base, Java, ficheros, configuración) en una **imagen**. Esa imagen se ejecuta igual en tu portátil, en el de un compañero, en el servidor de integración continua y en producción.

## Conceptos básicos

| Concepto | Qué es | Analogía |
|---|---|---|
| **Imagen** | Plantilla inmutable con el sistema de ficheros y la configuración de la aplicación. | Una clase |
| **Contenedor** | Una imagen **en ejecución**, aislada del resto del sistema. | Un objeto (instancia de la clase) |
| **Dockerfile** | Fichero de texto con las instrucciones para construir una imagen. | El código fuente de la clase |
| **Capa** (*layer*) | Cada instrucción del Dockerfile genera una capa. Las capas se guardan en caché y se reutilizan. | — |
| **Registro** (*registry*) | Almacén de imágenes. El público más conocido es **Docker Hub**. | Un repositorio de librerías como Maven Central |
| **Volumen** | Almacenamiento que sobrevive al contenedor (por ejemplo, los datos de una base de datos). | — |
| **Docker Compose** | Herramienta para definir y arrancar **varios contenedores** juntos con un fichero YAML. | — |

### Contenedor frente a máquina virtual

Una **máquina virtual** incluye un sistema operativo completo. Un **contenedor** comparte el núcleo del sistema anfitrión y solo aísla los procesos y ficheros de la aplicación. Por eso los contenedores son mucho **más ligeros y arrancan en segundos**.

## Comandos esenciales

```bash
docker build -t tienda-api:1.0 .        # construir una imagen a partir del Dockerfile
docker images                           # listar imágenes locales
docker run -d -p 8080:8080 --name api tienda-api:1.0   # crear y arrancar un contenedor
docker ps                               # contenedores en ejecución
docker ps -a                            # todos, también los parados
docker logs -f api                      # ver (y seguir) los logs
docker exec -it api sh                  # abrir una consola dentro del contenedor
docker stop api                         # parar
docker rm api                           # eliminar el contenedor
docker rmi tienda-api:1.0               # eliminar la imagen
```

En `docker run`:

- `-d` → en segundo plano (*detached*).
- `-p 8080:8080` → publica el puerto: `puerto_del_anfitrión:puerto_del_contenedor`.
- `--name api` → nombre del contenedor.
- `-e NOMBRE=valor` → variable de entorno.

## Un primer Dockerfile (sencillo, pero mejorable)

Partiendo de un `.jar` ya construido con `./mvnw package`:

```dockerfile
FROM eclipse-temurin:25-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

| Instrucción | Qué hace |
|---|---|
| `FROM` | Imagen base. `eclipse-temurin:25-jre` contiene solo el entorno de ejecución de Java 25 (sin compilador). |
| `WORKDIR` | Directorio de trabajo dentro de la imagen. |
| `COPY` | Copia ficheros del ordenador a la imagen. |
| `EXPOSE` | Documenta el puerto que usa la aplicación (no lo publica por sí solo). |
| `ENTRYPOINT` | Comando que se ejecuta al arrancar el contenedor. |

Problemas de este enfoque:

1. **Hay que compilar fuera de Docker**, así que depende de que la máquina tenga el JDK correcto.
2. **Todo el `.jar` va en una sola capa.** Cada vez que cambias una línea de tu código, se vuelve a copiar el `.jar` entero, con decenas de megas de dependencias que no han cambiado.
3. El proceso se ejecuta como **root** dentro del contenedor.

## *Multi-stage build*

Un **Dockerfile multi-etapa** tiene varias instrucciones `FROM`. Cada una inicia una **etapa**, y una etapa puede copiar ficheros de otra anterior. **Solo la última etapa forma la imagen final.**

Así se separa:

- una etapa de **construcción**, con el JDK completo, Maven y el código fuente;
- una etapa de **ejecución**, solo con el JRE y lo imprescindible.

La imagen final es más **pequeña** y más **segura**: no contiene el código fuente, ni Maven, ni el compilador.

### Las capas de Spring Boot

Spring Boot sabe dividir el `.jar` en **capas** según la frecuencia con que cambian:

| Capa | Contenido | Cambia… |
|---|---|---|
| `dependencies` | Librerías con versión estable | Rara vez |
| `spring-boot-loader` | El cargador de Spring Boot | Rara vez |
| `snapshot-dependencies` | Librerías en versión `SNAPSHOT` | A veces |
| `application` | Tus clases y recursos | En cada cambio |

Se extraen con el modo *tools* del propio `.jar`:

```bash
java -Djarmode=tools -jar app.jar extract --layers --destination extracted
```

Si cada capa se copia en una instrucción `COPY` distinta, Docker **reutiliza de la caché** las capas que no han cambiado. Al modificar tu código, solo se reconstruye la capa `application`, que ocupa muy poco.

### Dockerfile multi-etapa completo

```dockerfile
# syntax=docker/dockerfile:1

# ============ Etapa 1: compilar ============
FROM eclipse-temurin:25-jdk AS build
WORKDIR /workspace

# 1) Primero solo lo necesario para descargar dependencias.
#    Mientras pom.xml no cambie, Docker reutiliza esta capa desde la caché.
COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN chmod +x mvnw && ./mvnw -B dependency:go-offline

# 2) Después, el código fuente (cambia a menudo)
COPY src src
RUN ./mvnw -B package -DskipTests

# ============ Etapa 2: extraer las capas de Spring Boot ============
FROM eclipse-temurin:25-jre AS extract
WORKDIR /builder
COPY --from=build /workspace/target/*.jar application.jar
RUN java -Djarmode=tools -jar application.jar extract --layers --destination extracted

# ============ Etapa 3: imagen final de ejecución ============
FROM eclipse-temurin:25-jre
WORKDIR /application

# Usuario sin privilegios
RUN groupadd --system spring && useradd --system --gid spring spring
USER spring:spring

# Una instrucción COPY por capa, de la que menos cambia a la que más
COPY --from=extract /builder/extracted/dependencies/ ./
COPY --from=extract /builder/extracted/spring-boot-loader/ ./
COPY --from=extract /builder/extracted/snapshot-dependencies/ ./
COPY --from=extract /builder/extracted/application/ ./

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "application.jar"]
```

Puntos clave:

- **`AS build`, `AS extract`**: nombran las etapas para referirse a ellas.
- **`COPY --from=build`**: copia ficheros de una etapa anterior.
- **Orden de las instrucciones**: lo que cambia poco, arriba; lo que cambia mucho, abajo. Así se aprovecha la caché.
- `-DskipTests`: las pruebas se ejecutan en la integración continua (las de Testcontainers necesitan Docker, que no suele estar disponible dentro de un `docker build`).
- **`USER spring`**: la aplicación no se ejecuta como `root`.
- El `application.jar` final no es el *fat jar*: solo contiene tu código y referencias a las librerías extraídas. Esta disposición arranca más rápido.

> La documentación oficial de Spring Boot usa como imagen de ejecución `bellsoft/liberica-openjre-debian:25-cds` en sus ejemplos. `eclipse-temurin:25-jre` es otra opción muy extendida. Ambas sirven.

### Fichero `.dockerignore`

Igual que `.gitignore`, evita que se envíen a Docker ficheros innecesarios (y acelera la construcción):

```
target/
.git/
.idea/
.vscode/
*.iml
```

### Construir y ejecutar

```bash
docker build -t tienda-api:1.0 .
docker run -d -p 8080:8080 --name api tienda-api:1.0
```

## Configuración mediante variables de entorno

Spring Boot lee automáticamente las **variables de entorno** y las convierte en propiedades: en mayúsculas, los puntos se convierten en `_` y los guiones desaparecen.

| Propiedad | Variable de entorno |
|---|---|
| `spring.datasource.url` | `SPRING_DATASOURCE_URL` |
| `spring.datasource.password` | `SPRING_DATASOURCE_PASSWORD` |
| `spring.profiles.active` | `SPRING_PROFILES_ACTIVE` |
| `server.port` | `SERVER_PORT` |

```bash
docker run -d -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/tienda \
  -e SPRING_DATASOURCE_USERNAME=tienda \
  -e SPRING_DATASOURCE_PASSWORD=secreto \
  tienda-api:1.0
```

La **misma imagen** sirve para todos los entornos; solo cambia la configuración. **Nunca** metas contraseñas dentro de la imagen.

### Memoria de la JVM en contenedores

La JVM detecta los límites de memoria del contenedor. Si quieres controlar qué parte usa para el *heap*, usa un porcentaje en lugar de un valor fijo:

```bash
docker run -e JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75" ...
```

## Docker Compose: aplicación + base de datos

`compose.yaml`:

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: tienda
      POSTGRES_USER: tienda
      POSTGRES_PASSWORD: tienda
    volumes:
      - datos-postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tienda -d tienda"]
      interval: 5s
      timeout: 3s
      retries: 10

  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/tienda
      SPRING_DATASOURCE_USERNAME: tienda
      SPRING_DATASOURCE_PASSWORD: tienda
    depends_on:
      db:
        condition: service_healthy

volumes:
  datos-postgres:
```

- Dentro de Compose, los servicios se encuentran **por su nombre**: la API se conecta a `db:5432`, no a `localhost`.
- El **volumen** `datos-postgres` conserva los datos aunque se borre el contenedor.
- `healthcheck` + `depends_on … service_healthy` hace que la API espere a que PostgreSQL esté **listo**, no solo arrancado.

```bash
docker compose up -d --build   # construir la imagen de la API y arrancar todo
docker compose logs -f api     # ver los logs de la API
docker compose down            # parar y eliminar los contenedores (los volúmenes se conservan)
docker compose down -v         # … y también los volúmenes (¡borra los datos!)
```

### Docker Compose en desarrollo con Spring Boot

Con la dependencia `spring-boot-docker-compose`, al ejecutar la aplicación desde el IDE Spring Boot **arranca automáticamente los servicios del `compose.yaml`** y configura la conexión. Es muy cómodo en desarrollo: no hay que acordarse de levantar la base de datos.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-docker-compose</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

## Alternativa sin Dockerfile: *Buildpacks*

Spring Boot puede construir una imagen optimizada **sin escribir un Dockerfile**, usando **Cloud Native Buildpacks**:

```bash
./mvnw spring-boot:build-image
```

Genera una imagen por capas, con un usuario sin privilegios y una JVM configurada para contenedores. Es una opción excelente cuando no necesitas controlar cada detalle de la imagen.

## Buenas prácticas

1. Usa **multi-stage builds**: imagen final sin JDK ni código fuente.
2. Usa una imagen base **JRE** y con **versión concreta** (`25-jre`, no `latest`).
3. Ordena el Dockerfile para **aprovechar la caché** (dependencias antes que código).
4. Aprovecha las **capas de Spring Boot**.
5. Ejecuta como **usuario sin privilegios**.
6. Configura con **variables de entorno**; nada de secretos dentro de la imagen.
7. Añade un **`.dockerignore`**.
8. Una aplicación por contenedor.
9. Expón un *endpoint* de salud (Spring Boot Actuator: `/actuator/health`) para que el orquestador sepa si la aplicación está viva.

## Resumen

- Una **imagen** empaqueta la aplicación con todo lo necesario; un **contenedor** es una imagen en ejecución.
- Un **Dockerfile multi-etapa** compila en una etapa con JDK y ejecuta en otra con solo el JRE.
- Las **capas de Spring Boot** (`java -Djarmode=tools … extract --layers`) aprovechan la caché de Docker.
- **Docker Compose** arranca la aplicación con su base de datos.
- Alternativa sin Dockerfile: `./mvnw spring-boot:build-image`.

## Referencias

- Documentación de Docker: <https://docs.docker.com/>
- Docker — Multi-stage builds: <https://docs.docker.com/build/building/multi-stage/>
- Spring — Spring Boot with Docker: <https://spring.io/guides/gs/spring-boot-docker/>
- Spring Boot — Dockerfiles: <https://docs.spring.io/spring-boot/reference/packaging/container-images/dockerfiles.html>
- Spring Boot — Docker Compose support: <https://docs.spring.io/spring-boot/reference/features/dev-services.html>
- Eclipse Temurin (imágenes oficiales): <https://hub.docker.com/_/eclipse-temurin>
