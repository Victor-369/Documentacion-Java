# JUnit 6.1.3: visión general y guía de inicio

> **Origen:** traducción al castellano de España de la página
> [Overview – JUnit User Guide 6.1.3](https://docs.junit.org/6.1.3/overview.html).
> **Licencia del contenido original:** © 2015-2026 los autores originales. Contenido publicado bajo la [Eclipse Public License v2.0](https://www.eclipse.org/legal/epl-v20.html).
>
> **Cómo está organizado este documento**
>
> - **Parte 1** es la traducción fiel de la página oficial.
> - **Parte 2** es un complemento pensado para quien parte de cero con JUnit: conceptos básicos, puesta en marcha y buenas prácticas. **No forma parte de la documentación oficial**; está basado en el modelo de programación de JUnit Jupiter, y para cualquier detalle conviene consultar siempre la guía oficial enlazada en cada apartado.

---

## Índice

- [Parte 1. Traducción de la página oficial](#parte-1-traducción-de-la-página-oficial)
  - [Descripción general](#descripción-general)
  - [¿Qué es JUnit?](#qué-es-junit)
  - [Versiones de Java compatibles](#versiones-de-java-compatibles)
  - [Obtener ayuda](#obtener-ayuda)
  - [Primeros pasos](#primeros-pasos)
- [Parte 2. Complemento para principiantes](#parte-2-complemento-para-principiantes)
  - [1. Conceptos previos](#1-conceptos-previos)
  - [2. Cuál de las tres piezas necesito](#2-cuál-de-las-tres-piezas-necesito)
  - [3. Puesta en marcha con Maven](#3-puesta-en-marcha-con-maven)
  - [4. Puesta en marcha con Gradle](#4-puesta-en-marcha-con-gradle)
  - [5. Tu primer test](#5-tu-primer-test)
  - [6. Anotaciones esenciales](#6-anotaciones-esenciales)
  - [7. Aserciones más usadas](#7-aserciones-más-usadas)
  - [8. Comprobar excepciones](#8-comprobar-excepciones)
  - [9. Ciclo de vida: preparar y limpiar](#9-ciclo-de-vida-preparar-y-limpiar)
  - [10. Tests parametrizados](#10-tests-parametrizados)
  - [11. Desactivar, etiquetar y limitar el tiempo](#11-desactivar-etiquetar-y-limitar-el-tiempo)
  - [12. Ejecutar los tests](#12-ejecutar-los-tests)
  - [13. Buenas prácticas profesionales](#13-buenas-prácticas-profesionales)
  - [14. Errores frecuentes de principiantes](#14-errores-frecuentes-de-principiantes)
  - [15. Hoja de ruta para seguir aprendiendo](#15-hoja-de-ruta-para-seguir-aprendiendo)
  - [Glosario](#glosario)

---

# Parte 1. Traducción de la página oficial

## Descripción general

El objetivo de este documento es ofrecer una documentación de referencia completa para
quienes escriben tests, para los autores de extensiones y de motores de pruebas
(*engines*), así como para los proveedores de herramientas de compilación (*build tools*) y de IDE.

## ¿Qué es JUnit?

JUnit se compone de varios módulos distintos pertenecientes a tres subproyectos.

**JUnit 6.1.3 = *JUnit Platform* + *JUnit Jupiter* + *JUnit Vintage***

**JUnit Platform** sirve como base para [lanzar frameworks de pruebas](https://docs.junit.org/6.1.3/advanced-topics/launcher-api.html) en la JVM. También define la API `TestEngine` ([Javadoc](https://docs.junit.org/6.1.3/api/org.junit.platform.engine/org/junit/platform/engine/TestEngine.html)), que sirve para desarrollar un framework de pruebas que se ejecute sobre la plataforma. Además, la plataforma proporciona un [Console Launcher](https://docs.junit.org/6.1.3/running-tests/console-launcher.html) (lanzador de consola) para arrancar la plataforma desde la línea de comandos, y el [JUnit Platform Suite Engine](https://docs.junit.org/6.1.3/advanced-topics/junit-platform-suite-engine.html) para ejecutar una suite de pruebas personalizada utilizando uno o varios motores de pruebas sobre la plataforma. Los IDE más populares ofrecen también soporte de primer nivel para JUnit Platform (consulta [IntelliJ IDEA](https://docs.junit.org/6.1.3/running-tests/ide-support.html#intellij-idea), [Eclipse](https://docs.junit.org/6.1.3/running-tests/ide-support.html#eclipse), [NetBeans](https://docs.junit.org/6.1.3/running-tests/ide-support.html#netbeans) y [Visual Studio Code](https://docs.junit.org/6.1.3/running-tests/ide-support.html#vscode)), al igual que las herramientas de compilación (consulta [Gradle](https://docs.junit.org/6.1.3/running-tests/build-support.html#gradle), [Maven](https://docs.junit.org/6.1.3/running-tests/build-support.html#maven), [Ant](https://docs.junit.org/6.1.3/running-tests/build-support.html#ant), [Bazel](https://docs.junit.org/6.1.3/running-tests/build-support.html#bazel) y [sbt](https://docs.junit.org/6.1.3/running-tests/build-support.html#sbt)).

**JUnit Jupiter** es la combinación del [modelo de programación](https://docs.junit.org/6.1.3/writing-tests/intro.html) y el [modelo de extensiones](https://docs.junit.org/6.1.3/extensions/overview.html) para escribir tests y extensiones de JUnit. El subproyecto Jupiter proporciona un `TestEngine` para ejecutar sobre la plataforma los tests basados en Jupiter.

**JUnit Vintage** proporciona un `TestEngine` para ejecutar sobre la plataforma los tests basados en JUnit 3 y JUnit 4. Requiere que JUnit 4.12 o posterior esté presente en el *classpath* o en el *module path*. Ten en cuenta, no obstante, que el motor JUnit Vintage está **obsoleto (*deprecated*)** y solo debería utilizarse de forma temporal mientras se migran los tests a JUnit Jupiter o a otro framework de pruebas con soporte nativo para JUnit Platform.

## Versiones de Java compatibles

JUnit requiere **Java 17 (o superior)** en tiempo de ejecución. Aun así, puedes seguir probando código que haya sido compilado con versiones anteriores del JDK.

## Obtener ayuda

Puedes plantear tus dudas sobre JUnit en [Stack Overflow](https://stackoverflow.com/questions/tagged/junit5) o en la [categoría Q&A de GitHub Discussions](https://github.com/junit-team/junit-framework/discussions/categories/q-a).

## Primeros pasos

### Descargar los artefactos de JUnit

Para saber qué artefactos hay disponibles para descargar e incluir en tu proyecto, consulta los [metadatos de dependencias](https://docs.junit.org/6.1.3/appendix.html#dependency-metadata) (*Dependency Metadata*). Para configurar la gestión de dependencias de tu compilación, consulta [Build Support](https://docs.junit.org/6.1.3/running-tests/build-support.html) y los [proyectos de ejemplo](#proyectos-de-ejemplo).

### Funcionalidades de JUnit

Para saber qué funcionalidades ofrece JUnit 6.1.3 y cómo utilizarlas, lee los apartados correspondientes de esta guía de usuario, organizados por temas.

- [Escribir tests en JUnit Jupiter](https://docs.junit.org/6.1.3/writing-tests/intro.html)
- [Migrar de JUnit 4 a JUnit Jupiter](https://docs.junit.org/6.1.3/migrating-from-junit4.html)
- [Ejecutar tests](https://docs.junit.org/6.1.3/running-tests/intro.html)
- [Modelo de extensiones para JUnit Jupiter](https://docs.junit.org/6.1.3/extensions/overview.html)
- Temas avanzados
  - [API Launcher de JUnit Platform](https://docs.junit.org/6.1.3/advanced-topics/launcher-api.html)
  - [Test Kit de JUnit Platform](https://docs.junit.org/6.1.3/advanced-topics/testkit.html)

### Proyectos de ejemplo

Para ver ejemplos completos y funcionales de proyectos que puedes copiar y con los que puedes experimentar, un buen punto de partida es el repositorio [`junit-examples`](https://github.com/junit-team/junit-examples). Contiene una colección de proyectos de ejemplo basados en JUnit Jupiter, JUnit Vintage y otros frameworks de pruebas. En ellos encontrarás los scripts de compilación adecuados (por ejemplo, `build.gradle`, `pom.xml`, etc.). Los enlaces siguientes destacan algunas de las combinaciones entre las que puedes elegir.

- Para Gradle y Java: [`junit-jupiter-starter-gradle`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-gradle)
- Para Gradle y Kotlin: [`junit-jupiter-starter-gradle-kotlin`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-gradle-kotlin)
- Para Gradle y Groovy: [`junit-jupiter-starter-gradle-groovy`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-gradle-groovy)
- Para Maven: [`junit-jupiter-starter-maven`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-maven)
- Para Ant: [`junit-jupiter-starter-ant`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-ant)
- Para Bazel: [`junit-jupiter-starter-bazel`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-bazel)
- Para sbt: [`junit-jupiter-starter-sbt`](https://github.com/junit-team/junit-examples/tree/r6.1.3/junit-jupiter-starter-sbt)

### Mapa del resto de la guía oficial (en inglés)

| Sección | Enlace |
|---|---|
| Escribir tests | <https://docs.junit.org/6.1.3/writing-tests/intro.html> |
| Anotaciones | <https://docs.junit.org/6.1.3/writing-tests/annotations.html> |
| Aserciones | <https://docs.junit.org/6.1.3/writing-tests/assertions.html> |
| Suposiciones (*assumptions*) | <https://docs.junit.org/6.1.3/writing-tests/assumptions.html> |
| Gestión de excepciones | <https://docs.junit.org/6.1.3/writing-tests/exception-handling.html> |
| Desactivar tests | <https://docs.junit.org/6.1.3/writing-tests/disabling-tests.html> |
| Etiquetado y filtrado | <https://docs.junit.org/6.1.3/writing-tests/tagging-and-filtering.html> |
| Ciclo de vida de la instancia de test | <https://docs.junit.org/6.1.3/writing-tests/test-instance-lifecycle.html> |
| Tests anidados | <https://docs.junit.org/6.1.3/writing-tests/nested-tests.html> |
| Tests repetidos | <https://docs.junit.org/6.1.3/writing-tests/repeated-tests.html> |
| Clases y tests parametrizados | <https://docs.junit.org/6.1.3/writing-tests/parameterized-classes-and-tests.html> |
| Tests dinámicos | <https://docs.junit.org/6.1.3/writing-tests/dynamic-tests.html> |
| Timeouts | <https://docs.junit.org/6.1.3/writing-tests/timeouts.html> |
| Ejecución en paralelo | <https://docs.junit.org/6.1.3/writing-tests/parallel-execution.html> |
| Extensiones integradas | <https://docs.junit.org/6.1.3/writing-tests/built-in-extensions.html> |
| Migrar desde JUnit 4 | <https://docs.junit.org/6.1.3/migrating-from-junit4.html> |
| Soporte de IDE | <https://docs.junit.org/6.1.3/running-tests/ide-support.html> |
| Soporte de herramientas de compilación | <https://docs.junit.org/6.1.3/running-tests/build-support.html> |
| Parámetros de configuración | <https://docs.junit.org/6.1.3/running-tests/configuration-parameters.html> |
| Modelo de extensiones | <https://docs.junit.org/6.1.3/extensions/overview.html> |
| Notas de la versión | <https://docs.junit.org/6.1.3/release-notes.html> |
| Apéndice (metadatos de dependencias) | <https://docs.junit.org/6.1.3/appendix.html> |
| Javadoc | <https://docs.junit.org/6.1.3/api/index.html> |

---

# Parte 2. Complemento para principiantes

> Esta parte **no pertenece a la página oficial**. Es una introducción práctica para quien nunca ha usado JUnit. Todos los ejemplos usan JUnit Jupiter (paquete `org.junit.jupiter.api`) y Java 17 o superior.

## 1. Conceptos previos

**Test unitario.** Es un pequeño programa que ejecuta un fragmento de tu código (normalmente un método) y comprueba automáticamente que el resultado es el esperado. Si no lo es, el test falla y te avisa.

**¿Por qué escribirlos?**

- Detectas errores antes de que lleguen a producción.
- Puedes modificar el código con confianza: si rompes algo, los tests te lo dicen.
- Sirven como documentación ejecutable de cómo debe comportarse el código.
- Son imprescindibles en la integración continua (CI).

**Anatomía de un test: patrón AAA**

1. **Arrange (preparar):** creas los objetos y datos necesarios.
2. **Act (actuar):** ejecutas el código que quieres probar.
3. **Assert (comprobar):** verificas que el resultado es el esperado.

**Resultados posibles de un test**

| Resultado | Significado |
|---|---|
| Correcto (*passed*) | Todas las comprobaciones se cumplieron. |
| Fallido (*failed*) | Una aserción no se cumplió: el código no hace lo esperado. |
| Error | Ocurrió una excepción inesperada durante el test. |
| Omitido (*skipped*) | El test estaba desactivado o no se cumplían sus condiciones. |

## 2. Cuál de las tres piezas necesito

La fórmula oficial es **JUnit = Platform + Jupiter + Vintage**. Traducido a decisiones prácticas:

| Pieza | ¿Para qué sirve? | ¿La necesito? |
|---|---|---|
| **Jupiter** | Es la API con la que **escribes** tus tests (`@Test`, `assertEquals`…) y el motor que los ejecuta. | **Sí, siempre.** Es lo que vas a usar a diario. |
| **Platform** | Es la base que **lanza** los tests y permite que IDE y Maven/Gradle los ejecuten. | Sí, pero llega automáticamente como dependencia de Jupiter. |
| **Vintage** | Ejecuta tests antiguos de JUnit 3 y 4. | **Solo** si tienes tests antiguos. Está obsoleto; úsalo únicamente de forma transitoria durante una migración. |

> **Regla práctica:** si empiezas un proyecto nuevo, añade solo Jupiter.

## 3. Puesta en marcha con Maven

Estructura de carpetas estándar:

```
mi-proyecto/
├── pom.xml
└── src/
    ├── main/java/...     ← tu código de producción
    └── test/java/...     ← tus tests
```

Fragmento del `pom.xml`:

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
    <junit.version>6.1.3</junit.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <!-- Usa una versión 3.x reciente del plugin -->
            <version>3.5.2</version>
        </plugin>
    </plugins>
</build>
```

Ejecutar los tests:

```bash
mvn test
```

> **Importante:** el plugin `maven-surefire-plugin` es el que lanza los tests en Maven. Las versiones antiguas no reconocen JUnit Platform; usa una versión 3.x reciente. Consulta [Build Support](https://docs.junit.org/6.1.3/running-tests/build-support.html) para la configuración oficial y comprueba cuál es la última versión estable disponible.

## 4. Puesta en marcha con Gradle

`build.gradle.kts` (Kotlin DSL):

```kotlin
plugins {
    java
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:6.1.3")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.test {
    useJUnitPlatform()   // imprescindible: activa JUnit Platform
}
```

Ejecutar los tests:

```bash
./gradlew test
```

> Si prefieres gestionar todas las versiones de JUnit desde un único punto, existe el BOM `org.junit:junit-bom` (*Bill of Materials*). Consulta el [apéndice oficial](https://docs.junit.org/6.1.3/appendix.html#dependency-metadata) para conocer los artefactos disponibles.

## 5. Tu primer test

Código de producción (`src/main/java/com/ejemplo/Calculadora.java`):

```java
package com.ejemplo;

public class Calculadora {

    public int sumar(int a, int b) {
        return a + b;
    }

    public int dividir(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("El divisor no puede ser cero");
        }
        return a / b;
    }
}
```

Test (`src/test/java/com/ejemplo/CalculadoraTest.java`):

```java
package com.ejemplo;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void sumaDosNumerosPositivos() {
        // Arrange
        Calculadora calculadora = new Calculadora();

        // Act
        int resultado = calculadora.sumar(2, 3);

        // Assert
        assertEquals(5, resultado);
    }
}
```

**Puntos clave:**

- La clase y el método de test **no necesitan ser `public`**; en JUnit Jupiter basta con visibilidad de paquete.
- Los métodos de test **no deben ser `private`** ni `static`, y no deben devolver nada (`void`).
- La convención habitual es que la clase de test se llame como la clase probada más el sufijo `Test` (`CalculadoraTest`) y esté en el **mismo paquete** dentro de `src/test/java`.
- En `assertEquals(esperado, real)` el **primer argumento es el valor esperado** y el segundo el obtenido. Invertirlos provoca mensajes de error confusos.

## 6. Anotaciones esenciales

Resumen de las más utilizadas (todas en `org.junit.jupiter.api`). La lista completa está en [Annotations](https://docs.junit.org/6.1.3/writing-tests/annotations.html).

| Anotación | Función |
|---|---|
| `@Test` | Marca un método como test. |
| `@DisplayName("...")` | Da un nombre legible al test o a la clase en los informes. |
| `@BeforeEach` | Se ejecuta **antes de cada** test. |
| `@AfterEach` | Se ejecuta **después de cada** test. |
| `@BeforeAll` | Se ejecuta **una vez antes de todos** los tests (el método debe ser `static`, salvo con ciclo de vida `PER_CLASS`). |
| `@AfterAll` | Se ejecuta **una vez tras todos** los tests (igual que `@BeforeAll`, `static`). |
| `@Nested` | Declara una clase interna para agrupar tests de forma jerárquica. |
| `@Disabled` | Desactiva un test o una clase. |
| `@Tag("...")` | Etiqueta tests para filtrarlos. |
| `@ParameterizedTest` | Ejecuta el mismo test con varios conjuntos de datos. |
| `@RepeatedTest(n)` | Repite un test `n` veces. |
| `@Timeout` | Falla el test si tarda más de lo indicado. |
| `@TestMethodOrder` | Define el orden de ejecución de los tests de una clase. |

## 7. Aserciones más usadas

Las aserciones son los métodos estáticos de `org.junit.jupiter.api.Assertions` con los que compruebas resultados. Referencia oficial: [Assertions](https://docs.junit.org/6.1.3/writing-tests/assertions.html).

```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void ejemplosDeAserciones() {
    assertEquals(4, 2 + 2);                       // igualdad
    assertNotEquals(5, 2 + 2);                    // desigualdad
    assertTrue(10 > 3);                           // condición verdadera
    assertFalse(3 > 10);                          // condición falsa
    assertNull(null);                             // es null
    assertNotNull("hola");                        // no es null
    assertSame(objetoA, objetoA);                 // misma referencia (==)
    assertArrayEquals(new int[]{1, 2}, new int[]{1, 2});
    assertEquals(0.3, 0.1 + 0.2, 0.0001);         // decimales: con tolerancia
    assertIterableEquals(List.of(1, 2), List.of(1, 2));
}
```

**Mensajes personalizados.** Todas admiten un mensaje final para facilitar el diagnóstico:

```java
assertEquals(5, resultado, "La suma de 2 y 3 debería ser 5");

// Con Supplier: el mensaje solo se construye si el test falla
assertEquals(5, resultado, () -> "Resultado inesperado: " + resultado);
```

**Varias comprobaciones a la vez con `assertAll`.** Ejecuta todas y muestra *todos* los fallos, no solo el primero:

```java
@Test
void comprobarUsuario() {
    Usuario u = new Usuario("Ana", "García");

    assertAll("usuario",
        () -> assertEquals("Ana", u.nombre()),
        () -> assertEquals("García", u.apellidos())
    );
}
```

## 8. Comprobar excepciones

Con `assertThrows` compruebas que el código lanza la excepción esperada. Referencia: [Exception Handling](https://docs.junit.org/6.1.3/writing-tests/exception-handling.html) y [Assertions](https://docs.junit.org/6.1.3/writing-tests/assertions.html).

```java
@Test
void dividirEntreCeroLanzaExcepcion() {
    Calculadora calculadora = new Calculadora();

    IllegalArgumentException ex = assertThrows(
        IllegalArgumentException.class,
        () -> calculadora.dividir(10, 0)
    );

    assertEquals("El divisor no puede ser cero", ex.getMessage());
}
```

## 9. Ciclo de vida: preparar y limpiar

Por defecto, JUnit crea **una instancia nueva de la clase de test para cada método de test**. Así, los atributos de instancia no se comparten entre tests y cada uno parte de un estado limpio. Referencia: [Test Instance Lifecycle](https://docs.junit.org/6.1.3/writing-tests/test-instance-lifecycle.html).

```java
class CarritoTest {

    private Carrito carrito;

    @BeforeAll
    static void inicioGlobal() {
        System.out.println("Una vez, antes de todos los tests");
    }

    @BeforeEach
    void prepararCarrito() {
        carrito = new Carrito();          // estado limpio para cada test
    }

    @AfterEach
    void limpiar() {
        // liberar recursos si hiciera falta
    }

    @AfterAll
    static void finGlobal() {
        System.out.println("Una vez, después de todos los tests");
    }

    @Test
    @DisplayName("Un carrito nuevo está vacío")
    void carritoNuevoEstaVacio() {
        assertTrue(carrito.estaVacio());
    }

    @Test
    @DisplayName("Al añadir un producto, el carrito ya no está vacío")
    void anadirProducto() {
        carrito.anadir("Libro");
        assertFalse(carrito.estaVacio());
    }
}
```

Orden de ejecución para cada test: `@BeforeEach` → `@Test` → `@AfterEach`.

## 10. Tests parametrizados

Sirven para ejecutar la misma lógica con muchos datos sin duplicar código. Requieren que el módulo de parametrización esté disponible; el artefacto agregado `org.junit.jupiter:junit-jupiter` ya lo incluye. Referencia: [Parameterized Classes and Tests](https://docs.junit.org/6.1.3/writing-tests/parameterized-classes-and-tests.html).

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import org.junit.jupiter.params.provider.ValueSource;

class ParametrizadosTest {

    @ParameterizedTest
    @ValueSource(strings = {"radar", "reconocer", "ana"})
    void esPalindromo(String palabra) {
        assertTrue(new StringBuilder(palabra).reverse().toString().equals(palabra));
    }

    @ParameterizedTest(name = "{0} + {1} = {2}")
    @CsvSource({
        "1, 2, 3",
        "0, 0, 0",
        "-1, 1, 0"
    })
    void sumar(int a, int b, int esperado) {
        assertEquals(esperado, new Calculadora().sumar(a, b));
    }
}
```

Fuentes de datos habituales: `@ValueSource`, `@CsvSource`, `@CsvFileSource`, `@EnumSource`, `@MethodSource`, `@NullSource`, `@EmptySource`.

## 11. Desactivar, etiquetar y limitar el tiempo

```java
@Test
@Disabled("Pendiente de corregir el error #123")
void testDesactivado() { }

@Test
@Tag("lento")
void testLento() { }

@Test
@Timeout(value = 2, unit = TimeUnit.SECONDS)
void debeTerminarEnDosSegundos() { }
```

- **`@Disabled`** — indica siempre el motivo. Referencia: [Disabling Tests](https://docs.junit.org/6.1.3/writing-tests/disabling-tests.html).
- **`@Tag`** — permite ejecutar solo un subconjunto (por ejemplo, tests rápidos en local y todos en CI). Referencia: [Tagging and Filtering](https://docs.junit.org/6.1.3/writing-tests/tagging-and-filtering.html).
- **`@Timeout`** — evita que un test colgado bloquee la compilación. Referencia: [Timeouts](https://docs.junit.org/6.1.3/writing-tests/timeouts.html).

## 12. Ejecutar los tests

| Entorno | Cómo ejecutarlos |
|---|---|
| **IntelliJ IDEA** | Icono verde junto a la clase o el método, o clic derecho → *Run*. |
| **Eclipse** | Clic derecho sobre la clase → *Run As → JUnit Test*. |
| **Visual Studio Code** | Con el *Extension Pack for Java*, panel *Testing* o iconos junto al método. |
| **Maven** | `mvn test` (o `mvn -Dtest=CalculadoraTest test` para una clase concreta). |
| **Gradle** | `./gradlew test` (o `./gradlew test --tests "com.ejemplo.CalculadoraTest"`). |
| **Consola** | Con el [Console Launcher](https://docs.junit.org/6.1.3/running-tests/console-launcher.html) de JUnit Platform. |

Más detalle en [IDE Support](https://docs.junit.org/6.1.3/running-tests/ide-support.html) y [Build Support](https://docs.junit.org/6.1.3/running-tests/build-support.html).

## 13. Buenas prácticas profesionales

1. **Un test, un comportamiento.** Cada test debe comprobar una sola cosa y fallar por una sola razón.
2. **Nombres descriptivos.** Usa nombres de método claros o `@DisplayName` que expliquen el escenario y el resultado esperado (por ejemplo, `dividirEntreCeroLanzaExcepcion`).
3. **Sigue el patrón AAA** (preparar, actuar, comprobar) y separa visualmente las tres fases.
4. **Tests independientes.** Ningún test debe depender de que otro se haya ejecutado antes ni del orden de ejecución.
5. **Deterministas.** Evita depender de la hora actual, números aleatorios, red, o el orden de colecciones sin garantía. Si es necesario, inyecta esas dependencias para poder controlarlas.
6. **Rápidos.** Los tests unitarios deben ejecutarse en milisegundos. Marca con `@Tag` los que sean lentos.
7. **Prueba comportamiento, no implementación.** Verifica qué hace el código, no cómo lo hace internamente; así los tests sobreviven a las refactorizaciones.
8. **Cubre casos límite.** Valores nulos, cadenas vacías, cero, negativos, listas vacías, valores máximos, entradas inválidas.
9. **No uses lógica compleja en los tests** (bucles, condicionales). Un test debe ser tan simple que sea evidentemente correcto.
10. **Mensajes de fallo útiles.** Añade mensajes cuando el fallo por defecto no sea suficientemente claro.
11. **Ejecútalos siempre en integración continua** y no aceptes cambios que rompan la compilación de tests.
12. **No comentes ni ignores tests fallidos** sin motivo: usa `@Disabled` con una explicación y un enlace a la incidencia.
13. **Un test que nunca ha fallado no se ha demostrado útil.** Al escribir uno nuevo, comprueba que falla cuando el código es incorrecto.

## 14. Errores frecuentes de principiantes

| Error | Solución |
|---|---|
| El IDE o Maven dice «0 tests ejecutados». | Comprueba que importas `org.junit.jupiter.api.Test` (y **no** `org.junit.Test` de JUnit 4), que el test está en `src/test/java` y, en Maven, que el plugin Surefire es una versión 3.x reciente. En Gradle, que tienes `useJUnitPlatform()`. |
| Confundir `Assertions` de JUnit con `assert` de Java. | Usa siempre `assertEquals`, `assertTrue`, etc. La palabra clave `assert` de Java está desactivada por defecto. |
| Método de test `private` o `static`. | Debe ser no privado, no estático y devolver `void`. |
| `@BeforeAll` / `@AfterAll` que no se ejecutan o lanzan error. | Deben ser `static` (salvo que uses `@TestInstance(Lifecycle.PER_CLASS)`). |
| Compartir estado entre tests con atributos `static`. | Usa atributos de instancia inicializados en `@BeforeEach`. |
| Comparar `double` con `assertEquals(a, b)`. | Usa la versión con tolerancia: `assertEquals(a, b, delta)`. |
| Usar anotaciones de JUnit 4 (`@Before`, `@After`, `@RunWith`, `@Ignore`). | En Jupiter son `@BeforeEach`, `@AfterEach`, `@ExtendWith`, `@Disabled`. Consulta [Migrar desde JUnit 4](https://docs.junit.org/6.1.3/migrating-from-junit4.html). |
| Probar la excepción con `try/catch`. | Usa `assertThrows`. |
| Invertir los argumentos de `assertEquals`. | El orden correcto es `(esperado, real)`. |

## 15. Hoja de ruta para seguir aprendiendo

Recorrido sugerido de la guía oficial una vez asimilado lo anterior:

1. [Definitions](https://docs.junit.org/6.1.3/writing-tests/definitions.html) y [Test Classes and Methods](https://docs.junit.org/6.1.3/writing-tests/test-classes-and-methods.html) — terminología y reglas exactas.
2. [Display Names](https://docs.junit.org/6.1.3/writing-tests/display-names.html) — nombres legibles.
3. [Assumptions](https://docs.junit.org/6.1.3/writing-tests/assumptions.html) y [Conditional Test Execution](https://docs.junit.org/6.1.3/writing-tests/conditional-test-execution.html) — ejecutar tests solo si se cumplen ciertas condiciones (sistema operativo, versión de Java, variables de entorno…).
4. [Nested Tests](https://docs.junit.org/6.1.3/writing-tests/nested-tests.html) — organizar tests por contexto.
5. [Dependency Injection for Constructors and Methods](https://docs.junit.org/6.1.3/writing-tests/dependency-injection-for-constructors-and-methods.html) — recibir parámetros como `TestInfo` o `@TempDir`.
6. [Built-in Extensions](https://docs.junit.org/6.1.3/writing-tests/built-in-extensions.html) — por ejemplo, directorios temporales para tests con ficheros.
7. [Parallel Execution](https://docs.junit.org/6.1.3/writing-tests/parallel-execution.html) — acelerar suites grandes.
8. [Extension Model](https://docs.junit.org/6.1.3/extensions/overview.html) — para integrar con Mockito, Spring, bases de datos, etc.
9. [Configuration Parameters](https://docs.junit.org/6.1.3/running-tests/configuration-parameters.html) — ajustar el comportamiento global.
10. [Migrating from JUnit 4](https://docs.junit.org/6.1.3/migrating-from-junit4.html) — imprescindible si trabajas con código heredado.

**Herramientas complementarias** que suelen usarse junto a JUnit en proyectos profesionales (no forman parte de JUnit): *Mockito* (dobles de prueba), *AssertJ* (aserciones fluidas), *Testcontainers* (bases de datos y servicios reales en contenedores), *JaCoCo* (cobertura de código).

## Glosario

| Término | Definición |
|---|---|
| **Aserción (*assertion*)** | Comprobación que hace fallar el test si no se cumple. |
| **Classpath / module path** | Rutas donde la JVM busca clases y módulos. |
| **Artefacto** | Librería empaquetada (por ejemplo, un `.jar`) identificada por `groupId`, `artifactId` y versión. |
| **BOM** | *Bill of Materials*: fichero que fija de forma coherente las versiones de un conjunto de librerías. |
| **Test engine (motor de pruebas)** | Componente que sabe descubrir y ejecutar un tipo concreto de tests (Jupiter, Vintage…). |
| **Launcher (lanzador)** | Componente de JUnit Platform que descubre y ejecuta los tests a través de los motores. |
| **Suite** | Conjunto de tests agrupados para ejecutarse juntos. |
| **Extensión** | Pieza que añade comportamiento a JUnit Jupiter (inyección de parámetros, callbacks, condiciones…). |
| **Fixture** | Estado o datos que deben estar preparados antes de ejecutar un test. |
| **Doble de prueba / mock** | Objeto que sustituye a una dependencia real para aislar lo que se prueba. |
| **Regresión** | Error que reaparece en algo que antes funcionaba; los tests automáticos sirven para detectarlas. |
| **CI (integración continua)** | Sistema que compila y ejecuta los tests automáticamente en cada cambio. |

---

*Traducción de la página oficial y material complementario elaborados para la versión 6.1.3 de JUnit. La documentación de referencia y vinculante es la publicada en <https://docs.junit.org/6.1.3/>.*
