# Gestión de registros (logging) con Log4j 2 en Java

Guía práctica para configurar y usar **Apache Log4j 2** en un proyecto Java, pensada para quien parte de cero.

---

## 1. ¿Qué es el logging y por qué usarlo?

Un **log** (registro) es un histórico de mensajes que tu aplicación genera mientras se ejecuta: qué ha hecho, qué ha fallado, con qué datos, en qué momento.

Usar `System.out.println()` para esto es una mala práctica porque:

- No distingue la **importancia** del mensaje (información, aviso, error grave…).
- No puedes **desactivarlo** o filtrarlo sin tocar el código.
- No permite enviar la salida a **varios destinos** (consola, fichero, base de datos…).
- No incluye **fecha, hilo, clase ni línea** automáticamente.
- No gestiona **rotación de ficheros** (los logs crecen sin control).

Log4j 2 resuelve todo esto y se controla desde un **fichero de configuración externo**, sin recompilar.

---

## 2. Conceptos clave

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Logger** | Objeto que usas en tu código para escribir mensajes. Se suele crear uno por clase. | `LogManager.getLogger(MiClase.class)` |
| **Level** (nivel) | Importancia del mensaje. | `INFO`, `ERROR`… |
| **Appender** | *Destino* del mensaje. | Consola, fichero, fichero rotativo, base de datos, email… |
| **Layout** | *Formato* del mensaje. | `PatternLayout`, `JsonLayout` |
| **Configuration** | Fichero que conecta todo lo anterior. | `log4j2.xml` |

Flujo simplificado:

```
Tu código ──► Logger ──► (¿nivel suficiente?) ──► Appender ──► Layout ──► Consola / Fichero
```

### Niveles de log (de menor a mayor gravedad)

| Nivel | Cuándo usarlo |
|---|---|
| `TRACE` | Detalle extremadamente fino (entrada/salida de métodos, valores internos). |
| `DEBUG` | Información útil para depurar durante el desarrollo. |
| `INFO` | Eventos normales relevantes (arranque, petición procesada, fichero cargado). |
| `WARN` | Algo inesperado pero la aplicación sigue funcionando. |
| `ERROR` | Un error que impide completar una operación. |
| `FATAL` | Error gravísimo; la aplicación probablemente no puede continuar. |

Además existen `ALL` (todo) y `OFF` (nada) para configurar.

**Regla clave:** si configuras un nivel, se muestran los mensajes de ese nivel **y de los superiores**.
Con nivel `INFO` verás `INFO`, `WARN`, `ERROR` y `FATAL`, pero no `DEBUG` ni `TRACE`.

---

## 3. Añadir Log4j 2 al proyecto

Log4j 2 se divide en dos módulos principales:

- **`log4j-api`**: las interfaces que usas en tu código.
- **`log4j-core`**: la implementación real (appenders, layouts, configuración…).

> Comprueba siempre la última versión estable en <https://logging.apache.org/log4j/2.x/> o en Maven Central. En el momento de escribir esta guía la última es la **2.25.2**.
> Usa siempre una versión reciente: las anteriores a la 2.17.1 tienen la vulnerabilidad crítica **Log4Shell** (CVE-2021-44228).

### Maven (`pom.xml`)

```xml
<properties>
    <log4j.version>2.25.2</log4j.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>${log4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>${log4j.version}</version>
    </dependency>
</dependencies>
```

### Gradle (`build.gradle`)

```groovy
dependencies {
    implementation 'org.apache.logging.log4j:log4j-api:2.25.2'
    implementation 'org.apache.logging.log4j:log4j-core:2.25.2'
}
```

### Gradle Kotlin DSL (`build.gradle.kts`)

```kotlin
dependencies {
    implementation("org.apache.logging.log4j:log4j-api:2.25.2")
    implementation("org.apache.logging.log4j:log4j-core:2.25.2")
}
```

---

## 4. Ubicación del fichero de configuración

Log4j 2 busca automáticamente un fichero llamado `log4j2.xml` (también admite `.json`, `.yaml` o `.properties`) en el **classpath**.

En un proyecto Maven/Gradle estándar:

```
mi-proyecto/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/ejemplo/App.java
    │   └── resources/
    │       └── log4j2.xml        ← aquí
    └── test/
        └── resources/
            └── log4j2-test.xml   ← (opcional) configuración solo para tests
```

> Si no encuentra ninguna configuración, Log4j 2 usa una **configuración por defecto**: solo muestra por consola mensajes de nivel `ERROR` o superior. Por eso, si "no ves nada", suele ser porque falta este fichero o está mal ubicado.

---

## 5. Configuración básica (`log4j2.xml`)

Configuración mínima: mensajes en la consola.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">

    <Appenders>
        <Console name="Consola" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <Root level="info">
            <AppenderRef ref="Consola"/>
        </Root>
    </Loggers>

</Configuration>
```

Explicación:

- **`status="WARN"`**: nivel de los mensajes *internos* de Log4j (útil para detectar errores de configuración). Usa `TRACE` o `DEBUG` si algo no funciona.
- **`<Appenders>`**: define los destinos. Aquí uno llamado `Consola`.
- **`<PatternLayout>`**: define el formato de cada línea (ver sección 7).
- **`<Loggers>`**: define los loggers y su nivel.
- **`<Root>`**: el logger raíz; recoge todo lo que no tenga una configuración más específica.
- **`<AppenderRef>`**: conecta el logger con un appender por su nombre.

---

## 6. Uso en el código Java

### 6.1. Ejemplo básico

```java
package com.ejemplo;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class App {

    // Un logger por clase, estático y final
    private static final Logger logger = LogManager.getLogger(App.class);

    public static void main(String[] args) {
        logger.trace("Mensaje TRACE");
        logger.debug("Mensaje DEBUG");
        logger.info("Aplicación iniciada");
        logger.warn("Esto es un aviso");
        logger.error("Ha ocurrido un error");
        logger.fatal("Error fatal");
    }
}
```

Salida con `Root level="info"`:

```
10:32:15.123 [main] INFO  com.ejemplo.App - Aplicación iniciada
10:32:15.125 [main] WARN  com.ejemplo.App - Esto es un aviso
10:32:15.125 [main] ERROR com.ejemplo.App - Ha ocurrido un error
10:32:15.126 [main] FATAL com.ejemplo.App - Error fatal
```

(`TRACE` y `DEBUG` no aparecen porque el nivel configurado es `info`.)

### 6.2. Mensajes parametrizados (¡recomendado!)

**No concatenes cadenas.** Usa `{}` como marcador de posición:

```java
String usuario = "ana";
int intentos = 3;

// ❌ Mal: la concatenación se ejecuta SIEMPRE, aunque el nivel esté desactivado
logger.debug("Usuario " + usuario + " con " + intentos + " intentos");

// ✅ Bien: solo se construye el mensaje si el nivel está activo
logger.debug("Usuario {} con {} intentos", usuario, intentos);
```

### 6.3. Registrar excepciones

Pasa la excepción como **último argumento** para que se imprima el *stack trace* completo:

```java
try {
    int resultado = 10 / 0;
} catch (ArithmeticException e) {
    logger.error("Error al calcular la división", e);
}
```

Con parámetros y excepción a la vez:

```java
logger.error("No se pudo procesar el pedido {}", idPedido, e);
```

> ❌ No hagas `logger.error(e.getMessage())` ni `e.printStackTrace()`: pierdes información o te saltas el sistema de logging.

### 6.4. Evaluación perezosa con lambdas

Si calcular el mensaje es costoso, usa un `Supplier`:

```java
logger.debug("Estado completo: {}", () -> objetoPesado.generarInformeCompleto());
```

El método solo se ejecuta si `DEBUG` está activo.

### 6.5. Comprobar si un nivel está activo

```java
if (logger.isDebugEnabled()) {
    logger.debug("Datos: {}", calcularAlgoCostoso());
}
```

### 6.6. Alternativa con Lombok

Si usas Lombok, te ahorras la declaración del logger:

```java
import lombok.extern.log4j.Log4j2;

@Log4j2
public class ServicioPedidos {

    public void procesar(int id) {
        log.info("Procesando pedido {}", id);   // la variable se llama "log"
    }
}
```

---

## 7. Formato de los mensajes: `PatternLayout`

El atributo `pattern` define qué aparece en cada línea.

| Símbolo | Significado |
|---|---|
| `%d{yyyy-MM-dd HH:mm:ss.SSS}` | Fecha y hora |
| `%level` / `%-5level` | Nivel del mensaje (`-5` = alineado a la izquierda, 5 caracteres) |
| `%t` / `%thread` | Nombre del hilo |
| `%logger{36}` | Nombre del logger (normalmente la clase), abreviado a 36 caracteres |
| `%C` | Nombre de la clase *(costoso)* |
| `%M` | Nombre del método *(costoso)* |
| `%L` | Número de línea *(costoso)* |
| `%msg` / `%m` | El mensaje |
| `%n` | Salto de línea |
| `%ex` / `%throwable` | Excepción con su stack trace |
| `%highlight{...}` | Colorea según el nivel (en consola) |
| `%X{clave}` | Valor del contexto MDC (ver sección 11) |

> Evita `%C`, `%M` y `%L` en producción: obligan a calcular la posición en el código y penalizan el rendimiento.

Ejemplo de patrón completo:

```
%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
```

Resultado:

```
2026-09-24 10:32:15.123 [main] INFO  com.ejemplo.App - Aplicación iniciada
```

Ejemplo con colores para la consola:

```xml
<PatternLayout pattern="%highlight{%d{HH:mm:ss.SSS} %-5level %logger{20} - %msg%n}"/>
```

---

## 8. Escribir en un fichero

### 8.1. Appender `File` (simple)

```xml
<File name="Fichero" fileName="logs/app.log">
    <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
</File>
```

Problema: el fichero crece indefinidamente.

### 8.2. Appender `RollingFile` (recomendado)

Crea un fichero nuevo por fecha y/o tamaño, comprime los antiguos y borra los más viejos:

```xml
<RollingFile name="FicheroRotativo"
             fileName="logs/app.log"
             filePattern="logs/app-%d{yyyy-MM-dd}-%i.log.gz">

    <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>

    <Policies>
        <!-- Rota cada día -->
        <TimeBasedTriggeringPolicy/>
        <!-- O cuando el fichero supere 10 MB -->
        <SizeBasedTriggeringPolicy size="10 MB"/>
    </Policies>

    <!-- Conserva como máximo 10 ficheros por periodo -->
    <DefaultRolloverStrategy max="10"/>
</RollingFile>
```

- `filePattern`: nombre de los ficheros archivados. Terminar en `.gz` o `.zip` los comprime automáticamente.
- `%i`: contador que se incrementa si hay varios ficheros en el mismo periodo.

### 8.3. Ejemplo completo: consola + fichero

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">

    <Properties>
        <Property name="PATRON">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n</Property>
        <Property name="DIR_LOGS">logs</Property>
    </Properties>

    <Appenders>
        <Console name="Consola" target="SYSTEM_OUT">
            <PatternLayout pattern="${PATRON}"/>
        </Console>

        <RollingFile name="FicheroRotativo"
                     fileName="${DIR_LOGS}/app.log"
                     filePattern="${DIR_LOGS}/app-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="${PATRON}"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>

    <Loggers>
        <Root level="info">
            <AppenderRef ref="Consola"/>
            <AppenderRef ref="FicheroRotativo"/>
        </Root>
    </Loggers>

</Configuration>
```

Con `<Properties>` defines variables reutilizables y evitas repetir el patrón.

---

## 9. Loggers específicos por paquete o clase

Puedes tener niveles distintos según la parte del código. Es muy útil para activar `DEBUG` solo en tu código y silenciar el ruido de librerías:

```xml
<Loggers>

    <!-- Tu código: detalle máximo -->
    <Logger name="com.ejemplo" level="debug" additivity="false">
        <AppenderRef ref="Consola"/>
        <AppenderRef ref="FicheroRotativo"/>
    </Logger>

    <!-- Una librería ruidosa: solo avisos y errores -->
    <Logger name="org.hibernate" level="warn"/>

    <!-- Todo lo demás -->
    <Root level="info">
        <AppenderRef ref="Consola"/>
    </Root>

</Loggers>
```

Sobre `additivity`:

- `additivity="true"` (por defecto): además de sus propios appenders, el mensaje se **propaga** al logger padre (y por tanto al `Root`).
- `additivity="false"`: el mensaje **no** se propaga. Ponlo a `false` cuando el logger tiene sus propios appenders, para evitar **mensajes duplicados**.

Los nombres son jerárquicos: un logger para `com.ejemplo` afecta también a `com.ejemplo.servicios.PedidoService`.

---

## 10. Separar errores en su propio fichero

Un patrón habitual: un fichero general y otro solo con errores, usando un **filtro**:

```xml
<RollingFile name="SoloErrores"
             fileName="logs/errores.log"
             filePattern="logs/errores-%d{yyyy-MM-dd}.log.gz">
    <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
    <PatternLayout pattern="${PATRON}"/>
    <Policies>
        <TimeBasedTriggeringPolicy/>
    </Policies>
</RollingFile>
```

Y lo enlazas en el `Root` junto a los demás:

```xml
<Root level="info">
    <AppenderRef ref="Consola"/>
    <AppenderRef ref="FicheroRotativo"/>
    <AppenderRef ref="SoloErrores"/>
</Root>
```

---

## 11. Contexto adicional: `ThreadContext` (MDC)

Sirve para añadir datos (usuario, id de petición…) que aparecerán en **todos** los mensajes del hilo actual, sin pasarlos manualmente.

```java
import org.apache.logging.log4j.ThreadContext;

public void atenderPeticion(String usuario, String idPeticion) {
    ThreadContext.put("usuario", usuario);
    ThreadContext.put("idPeticion", idPeticion);
    try {
        logger.info("Petición recibida");
        // ... lógica de negocio ...
        logger.info("Petición terminada");
    } finally {
        ThreadContext.clearMap();   // ¡limpiar siempre al terminar!
    }
}
```

En el patrón, usa `%X{clave}`:

```xml
<PatternLayout pattern="%d{HH:mm:ss.SSS} [%X{usuario}] [%X{idPeticion}] %-5level %logger{20} - %msg%n"/>
```

Salida:

```
10:40:02.311 [ana] [REQ-4521] INFO  c.e.PedidoService - Petición recibida
```

---

## 12. Configuración en formato `.properties` (alternativa)

Si prefieres no usar XML, este es el equivalente a la configuración básica en `src/main/resources/log4j2.properties`:

```properties
status = warn
name = ConfiguracionProperties

appender.consola.type = Console
appender.consola.name = Consola
appender.consola.target = SYSTEM_OUT
appender.consola.layout.type = PatternLayout
appender.consola.layout.pattern = %d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n

appender.fichero.type = RollingFile
appender.fichero.name = Fichero
appender.fichero.fileName = logs/app.log
appender.fichero.filePattern = logs/app-%d{yyyy-MM-dd}-%i.log.gz
appender.fichero.layout.type = PatternLayout
appender.fichero.layout.pattern = %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
appender.fichero.policies.type = Policies
appender.fichero.policies.time.type = TimeBasedTriggeringPolicy
appender.fichero.policies.size.type = SizeBasedTriggeringPolicy
appender.fichero.policies.size.size = 10MB
appender.fichero.strategy.type = DefaultRolloverStrategy
appender.fichero.strategy.max = 10

logger.miapp.name = com.ejemplo
logger.miapp.level = debug

rootLogger.level = info
rootLogger.appenderRef.consola.ref = Consola
rootLogger.appenderRef.fichero.ref = Fichero
```

---

## 13. Indicar el fichero de configuración manualmente

Si el fichero no está en el classpath o quieres cambiarlo sin recompilar:

```bash
java -Dlog4j2.configurationFile=/ruta/a/mi-log4j2.xml -jar mi-app.jar
```

Para que Log4j **recargue** la configuración automáticamente cuando el fichero cambie (sin reiniciar la app), añade `monitorInterval` (en segundos):

```xml
<Configuration status="WARN" monitorInterval="30">
```

---

## 14. Buenas prácticas

1. **Un logger por clase**, declarado `private static final`.
2. **Usa mensajes parametrizados** (`{}`), nunca concatenación.
3. **Elige bien el nivel**: `INFO` para hitos, `DEBUG` para diagnóstico, `ERROR` solo para fallos reales.
4. **Registra siempre la excepción completa**: `logger.error("mensaje", e)`.
5. **No registres datos sensibles**: contraseñas, tokens, números de tarjeta, datos personales.
6. **Producción en `INFO`** (o `WARN`) y desarrollo en `DEBUG`.
7. **Usa `RollingFile`** en lugar de `File` para no llenar el disco.
8. **Evita `%C`, `%M`, `%L`** en producción por rendimiento.
9. **No registres dentro de bucles muy intensivos** sin comprobar el nivel.
10. **Mensajes claros y con contexto**: `"Pedido 1234 no encontrado para el usuario ana"` es mejor que `"Error"`.
11. **Mantén Log4j actualizado** por motivos de seguridad.

---

## 15. Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| No aparece ningún log (salvo errores) | Falta `log4j2.xml` o está fuera de `resources` | Colócalo en `src/main/resources` y recompila |
| Aparece el aviso `ERROR StatusLogger Log4j2 could not find a logging implementation` | Falta la dependencia `log4j-core` | Añade `log4j-core` al `pom.xml` |
| Mensajes duplicados | Logger con `additivity` a `true` y los mismos appenders que el `Root` | Pon `additivity="false"` |
| No salen los mensajes `DEBUG` | El nivel del logger o del `Root` es `info` | Baja el nivel a `debug` |
| No se crea el fichero de log | Ruta sin permisos o error de configuración | Pon `status="DEBUG"` y revisa los mensajes internos |
| Error al parsear la configuración | XML mal formado o nombre de appender incorrecto en `AppenderRef` | Revisa que los `ref` coincidan con los `name` |
| Conflicto con otros frameworks de logging (SLF4J, Logback, Log4j 1.x) | Varias implementaciones en el classpath | Usa el puente correspondiente (ver sección 16) y deja una sola implementación |

**Truco de depuración:** cambia `status="WARN"` por `status="TRACE"` en `<Configuration>` y Log4j imprimirá en consola cómo está cargando la configuración.

---

## 16. Integración con SLF4J (si tus librerías lo usan)

Muchas librerías (Spring, Hibernate…) usan SLF4J como fachada. Para que sus logs también pasen por Log4j 2 añade el puente:

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j2-impl</artifactId>
    <version>${log4j.version}</version>
</dependency>
```

> Si usas **Spring Boot**, el starter incluye Logback por defecto. Para usar Log4j 2 hay que excluir `spring-boot-starter-logging` y añadir `spring-boot-starter-log4j2`.

---

## 17. Resumen rápido (chuleta)

**1. Dependencias** (`log4j-api` + `log4j-core`).

**2. Fichero** `src/main/resources/log4j2.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Consola" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Consola"/>
        </Root>
    </Loggers>
</Configuration>
```

**3. Código**:

```java
private static final Logger logger = LogManager.getLogger(MiClase.class);

logger.info("Usuario {} conectado", usuario);
logger.error("Fallo al guardar", excepcion);
```

---

## 18. Referencias

- Documentación oficial: <https://logging.apache.org/log4j/2.x/>
- Descargas y versiones: <https://logging.apache.org/log4j/2.x/download.html>
- Repositorio en Maven Central: <https://central.sonatype.com/artifact/org.apache.logging.log4j/log4j-core>
