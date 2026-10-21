# Guía práctica de JUnit: aprende a escribir pruebas paso a paso

> Guía para personas que empiezan con las pruebas unitarias en Java.
> Basada en la documentación oficial de **JUnit 6.1.3**: <https://docs.junit.org/6.1.3/overview.html>

---

## Índice

1. [¿Qué es JUnit y por qué probar el código?](#1-qué-es-junit-y-por-qué-probar-el-código)
2. [Requisitos y configuración del proyecto](#2-requisitos-y-configuración-del-proyecto)
3. [Tu primera prueba](#3-tu-primera-prueba)
4. [Anatomía de una clase de prueba](#4-anatomía-de-una-clase-de-prueba)
5. [Ciclo de vida: preparar y limpiar](#5-ciclo-de-vida-preparar-y-limpiar)
6. [Aserciones: comprobar resultados](#6-aserciones-comprobar-resultados)
7. [Probar excepciones](#7-probar-excepciones)
8. [Nombres legibles para las pruebas](#8-nombres-legibles-para-las-pruebas)
9. [Suposiciones (assumptions)](#9-suposiciones-assumptions)
10. [Desactivar pruebas y ejecución condicional](#10-desactivar-pruebas-y-ejecución-condicional)
11. [Pruebas parametrizadas](#11-pruebas-parametrizadas)
12. [Pruebas anidadas](#12-pruebas-anidadas)
13. [Etiquetas y filtrado](#13-etiquetas-y-filtrado)
14. [Orden de ejecución](#14-orden-de-ejecución)
15. [Pruebas repetidas](#15-pruebas-repetidas)
16. [Tiempos límite (timeouts)](#16-tiempos-límite-timeouts)
17. [Ficheros temporales con `@TempDir`](#17-ficheros-temporales-con-tempdir)
18. [Pruebas dinámicas](#18-pruebas-dinámicas)
19. [Cómo ejecutar las pruebas](#19-cómo-ejecutar-las-pruebas)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Resumen rápido (chuleta)](#21-resumen-rápido-chuleta)
22. [Para seguir aprendiendo](#22-para-seguir-aprendiendo)

---

## 1. ¿Qué es JUnit y por qué probar el código?

**JUnit** es el marco de pruebas (*testing framework*) más utilizado en Java. Te permite escribir código que comprueba automáticamente que tu código de producción hace lo que debe.

Una **prueba unitaria** verifica una pequeña pieza de código (normalmente un método o una clase) de forma aislada. Sus ventajas:

- Detectas errores **antes** de que lleguen a producción.
- Puedes **modificar el código con confianza** (refactorizar) porque las pruebas te avisan si rompes algo.
- Las pruebas sirven como **documentación viva** de cómo se comporta el código.

### Los módulos de JUnit

Según la documentación oficial, JUnit está formado por tres subproyectos:

```
JUnit 6.1.3 = JUnit Platform + JUnit Jupiter + JUnit Vintage
```

| Módulo | Para qué sirve |
|---|---|
| **JUnit Platform** | Base para lanzar marcos de pruebas en la JVM. Es lo que usan los IDE y las herramientas de construcción. |
| **JUnit Jupiter** | El modelo de programación (anotaciones, aserciones…) y de extensión para **escribir tus pruebas**. Es lo que vas a usar tú. |
| **JUnit Vintage** | Permite ejecutar pruebas antiguas de JUnit 3 y 4. Está **obsoleto** y solo debería usarse mientras migras. |

> **En esta guía trabajaremos con JUnit Jupiter**, que es donde se escriben las pruebas.

---

## 2. Requisitos y configuración del proyecto

### Requisitos

- **Java 17 o superior** en tiempo de ejecución. Puedes probar código compilado con versiones anteriores del JDK.
- Una herramienta de construcción (**Maven** o **Gradle**) o un IDE con soporte para JUnit (IntelliJ IDEA, Eclipse, NetBeans y Visual Studio Code lo tienen).

### Estructura de carpetas estándar

```
mi-proyecto/
├── pom.xml                      (o build.gradle)
└── src/
    ├── main/java/               ← código de producción
    │   └── com/ejemplo/Calculadora.java
    └── test/java/               ← código de pruebas
        └── com/ejemplo/CalculadoraTest.java
```

La clase de prueba se coloca en el **mismo paquete** que la clase que prueba, pero dentro de `src/test/java`.

### Configuración con Maven

Añade la dependencia de JUnit Jupiter y el plugin Surefire (para JUnit 6 la versión mínima de Surefire es la 3.0.0):

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>6.1.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.5.5</version>
        </plugin>
    </plugins>
</build>
```

> **Truco:** si usas varias librerías de JUnit, importa el *BOM* (`org.junit:junit-bom:6.1.3`) en `<dependencyManagement>` y así puedes omitir la versión en cada dependencia.

### Configuración con Gradle

```groovy
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:6.1.3")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

test {
    useJUnitPlatform()
}
```

`useJUnitPlatform()` es imprescindible: le dice a Gradle que ejecute las pruebas con la plataforma JUnit.

### Proyectos de ejemplo

El repositorio oficial [`junit-examples`](https://github.com/junit-team/junit-examples) contiene proyectos de arranque listos para copiar (por ejemplo `junit-jupiter-starter-maven` y `junit-jupiter-starter-gradle`).

---

## 3. Tu primera prueba

Imagina que tenemos esta clase de producción, en `src/main/java/com/ejemplo/Calculadora.java`:

```java
package com.ejemplo;

public class Calculadora {

    public int sumar(int a, int b) {
        return a + b;
    }

    public int dividir(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("No se puede dividir entre cero");
        }
        return a / b;
    }
}
```

Y esta es su clase de prueba, en `src/test/java/com/ejemplo/CalculadoraTest.java`:

```java
package com.ejemplo;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void sumaDosNumerosPositivos() {
        Calculadora calculadora = new Calculadora();

        int resultado = calculadora.sumar(2, 3);

        assertEquals(5, resultado);
    }
}
```

### Qué está pasando aquí

1. `@Test` marca el método como **método de prueba**. JUnit lo descubrirá y lo ejecutará.
2. Creamos el objeto a probar.
3. Ejecutamos la operación.
4. `assertEquals(esperado, real)` comprueba que el resultado es el esperado. Si no lo es, la prueba **falla**.

Si el método termina sin lanzar ninguna excepción, la prueba **pasa**.

---

## 4. Anatomía de una clase de prueba

### Reglas básicas

Una **clase de prueba** es cualquier clase que contenga al menos un método de prueba. Los **métodos de prueba** son los anotados con `@Test` (y algunas otras anotaciones que veremos, como `@RepeatedTest` o `@ParameterizedTest`).

| Elemento | Regla |
|---|---|
| Clase de prueba | No puede ser `abstract` y debe tener un constructor. Puede ser de visibilidad de paquete (no necesita ser `public`). |
| Método de prueba | **No** puede ser `abstract`, **no** debe ser `private` y **debe devolver `void`**. También puede ser de visibilidad de paquete. |

> A diferencia de JUnit 4, **no hace falta poner `public`** en la clase ni en los métodos. Lo habitual es dejarlos sin modificador de acceso.

### El patrón Arrange–Act–Assert (AAA)

Una buena prueba sigue tres pasos, que conviene separar visualmente:

```java
@Test
void sumaDosNumerosPositivos() {
    // Arrange (preparar)
    Calculadora calculadora = new Calculadora();

    // Act (actuar)
    int resultado = calculadora.sumar(2, 3);

    // Assert (comprobar)
    assertEquals(5, resultado);
}
```

### Convención de nombres

- La clase de prueba suele llamarse como la clase probada más el sufijo `Test` o `Tests`: `CalculadoraTest`.
- Maven Surefire busca por defecto clases que coincidan con `Test*.java`, `*Test.java`, `*Tests.java` o `*TestCase.java`.
- Para nombrar los **métodos** de prueba, consulta el patrón `método_condición_resultadoEsperado` en la [sección 8](#convención-de-nombres-método_condición_resultadoesperado).

---

## 5. Ciclo de vida: preparar y limpiar

Cuando varias pruebas necesitan el mismo punto de partida, no repitas código: usa los métodos de ciclo de vida.

| Anotación | Cuándo se ejecuta | ¿`static`? |
|---|---|---|
| `@BeforeAll` | Una vez, **antes** de todas las pruebas de la clase | Sí (por defecto) |
| `@BeforeEach` | **Antes de cada** método de prueba | No |
| `@AfterEach` | **Después de cada** método de prueba | No |
| `@AfterAll` | Una vez, **después** de todas las pruebas de la clase | Sí (por defecto) |

### Ejemplo: cuenta bancaria

Clase de producción:

```java
package com.ejemplo;

public class CuentaBancaria {

    private double saldo;

    public CuentaBancaria(double saldoInicial) {
        if (saldoInicial < 0) {
            throw new IllegalArgumentException("El saldo inicial no puede ser negativo");
        }
        this.saldo = saldoInicial;
    }

    public void ingresar(double cantidad) {
        if (cantidad <= 0) {
            throw new IllegalArgumentException("La cantidad debe ser positiva");
        }
        saldo += cantidad;
    }

    public void retirar(double cantidad) {
        if (cantidad > saldo) {
            throw new IllegalStateException("Saldo insuficiente");
        }
        saldo -= cantidad;
    }

    public double getSaldo() {
        return saldo;
    }
}
```

Clase de prueba con ciclo de vida:

```java
package com.ejemplo;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class CuentaBancariaTest {

    private CuentaBancaria cuenta;

    @BeforeAll
    static void iniciarTodo() {
        System.out.println("Empieza la batería de pruebas");
    }

    @BeforeEach
    void prepararCuenta() {
        // Se crea una cuenta NUEVA antes de cada prueba
        cuenta = new CuentaBancaria(100.0);
    }

    @Test
    void ingresarAumentaElSaldo() {
        cuenta.ingresar(50.0);

        assertEquals(150.0, cuenta.getSaldo());
    }

    @Test
    void retirarDisminuyeElSaldo() {
        cuenta.retirar(30.0);

        assertEquals(70.0, cuenta.getSaldo());
    }

    @AfterEach
    void limpiar() {
        System.out.println("Prueba terminada");
    }

    @AfterAll
    static void terminarTodo() {
        System.out.println("Fin de la batería de pruebas");
    }
}
```

### Detalle importante: una instancia nueva por cada prueba

Por defecto, JUnit crea **una instancia nueva de la clase de prueba para cada método de prueba**. Por eso `cuenta` empieza siempre con 100 € en cada prueba, sin importar lo que hayan hecho las demás. Esto garantiza que las pruebas son **independientes** entre sí.

Si necesitas cambiar este comportamiento, puedes anotar la clase con `@TestInstance(TestInstance.Lifecycle.PER_CLASS)`. En ese caso los métodos `@BeforeAll` y `@AfterAll` ya no tienen que ser `static`.

---

## 6. Aserciones: comprobar resultados

Las aserciones están en la clase `org.junit.jupiter.api.Assertions`. Se suelen importar de forma estática:

```java
import static org.junit.jupiter.api.Assertions.*;
```

### Las más utilizadas

| Aserción                                           | Qué comprueba                                                   |
| -------------------------------------------------- | --------------------------------------------------------------- |
| `assertEquals(esperado, real)`                     | Que ambos valores son iguales                                   |
| `assertNotEquals(a, b)`                            | Que los valores son distintos                                   |
| `assertTrue(condicion)` / `assertFalse(condicion)` | Que la condición es verdadera / falsa                           |
| `assertNull(obj)` / `assertNotNull(obj)`           | Que el objeto es / no es `null`                                 |
| `assertSame(a, b)` / `assertNotSame(a, b)`         | Que son (o no) **la misma referencia**                          |
| `assertArrayEquals(esperado, real)`                | Que dos arrays tienen el mismo contenido                        |
| `assertIterableEquals(esperado, real)`             | Que dos iterables tienen los mismos elementos en el mismo orden |
| `assertInstanceOf(Clase.class, obj)`               | Que el objeto es de un tipo determinado                         |
| `assertThrows(...)`                                | Que se lanza una excepción (ver sección 7)                      |
| `assertAll(...)`                                   | Agrupa varias aserciones (ver más abajo)                        |
| `assertTimeout(...)`                               | Que un código termina antes de un tiempo dado                   |
| `fail("mensaje")`                                  | Hace fallar la prueba a propósito                               |

> **Atención al orden de los parámetros:** primero el valor **esperado** y después el **real**. Si los pones al revés, el mensaje de error resultará engañoso.

### Ejemplos

```java
import static org.junit.jupiter.api.Assertions.*;

import java.util.List;
import org.junit.jupiter.api.Test;

class EjemplosAsercionesTest {

    @Test
    void asercionesBasicas() {
        assertEquals(4, 2 + 2);
        assertNotEquals(5, 2 + 2);
        assertTrue(10 > 3);
        assertFalse(3 > 10);

        String texto = null;
        assertNull(texto);
        assertNotNull("hola");
    }

    @Test
    void comparacionDeColeccionesYArrays() {
        assertArrayEquals(new int[] {1, 2, 3}, new int[] {1, 2, 3});
        assertIterableEquals(List.of("a", "b"), List.of("a", "b"));
    }

    @Test
    void comprobarElTipo() {
        Object valor = "Soy un texto";
        assertInstanceOf(String.class, valor);
    }
}
```

### Mensajes personalizados

Todas las aserciones admiten un mensaje final para que el fallo sea más claro. Puede ser un `String` o, mejor, una **expresión lambda** (solo se construye si la prueba falla):

```java
assertEquals(5, resultado, "La suma de 2 y 3 debería ser 5");
assertEquals(5, resultado, () -> "Se esperaba 5 pero se obtuvo " + resultado);
```

### Comparar números decimales

Los `double` y `float` no son exactos. Usa un margen de error (*delta*):

```java
assertEquals(0.3, 0.1 + 0.2, 0.0001);
```

### Aserciones agrupadas con `assertAll`

Con varias aserciones seguidas, si la primera falla, las demás **no se ejecutan**. Con `assertAll` se ejecutan **todas** y se te informa de **todos** los fallos a la vez:

```java
@Test
void datosDeUnaPersona() {
    Persona persona = new Persona("Lucía", "García", 30);

    assertAll("Datos de la persona",
        () -> assertEquals("Lucía", persona.getNombre()),
        () -> assertEquals("García", persona.getApellido()),
        () -> assertEquals(30, persona.getEdad())
    );
}
```

### ¿Y si quiero usar otra librería de aserciones?

JUnit Jupiter incluye todo lo necesario para la mayoría de casos. Si buscas aserciones más expresivas, puedes combinarlo con librerías de terceros como **AssertJ** o **Hamcrest**.

---

## 7. Probar excepciones

A veces lo que quieres comprobar es que el código **falla correctamente**. Para eso está `assertThrows`.

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import org.junit.jupiter.api.Test;

class CalculadoraExcepcionesTest {

    private final Calculadora calculadora = new Calculadora();

    @Test
    void dividirEntreCeroLanzaExcepcion() {
        ArithmeticException excepcion = assertThrows(
            ArithmeticException.class,
            () -> calculadora.dividir(10, 0)
        );

        assertEquals("No se puede dividir entre cero", excepcion.getMessage());
    }
}
```

Puntos clave:

- El primer argumento es el **tipo de excepción esperado**.
- El segundo es una **lambda** con el código que debería lanzarla.
- Devuelve la excepción capturada, así puedes comprobar su mensaje u otros detalles.
- Si el código **no** lanza la excepción (o lanza otra distinta), la prueba falla.

### Cuando NO debe lanzarse ninguna excepción

```java
import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;

@Test
void ingresarUnaCantidadValidaNoFalla() {
    CuentaBancaria cuenta = new CuentaBancaria(0);

    assertDoesNotThrow(() -> cuenta.ingresar(10));
}
```

### Ejemplo con la cuenta bancaria

```java
@Test
void retirarMasDelSaldoLanzaExcepcion() {
    CuentaBancaria cuenta = new CuentaBancaria(100.0);

    IllegalStateException e = assertThrows(
        IllegalStateException.class,
        () -> cuenta.retirar(500.0)
    );

    assertEquals("Saldo insuficiente", e.getMessage());
}

@Test
void noSePuedeCrearUnaCuentaConSaldoNegativo() {
    assertThrows(IllegalArgumentException.class, () -> new CuentaBancaria(-1));
}
```

---

## 8. Nombres legibles para las pruebas

Con `@DisplayName` puedes dar a las pruebas nombres descriptivos (con espacios, tildes e incluso emojis) que aparecerán en los informes y en el IDE:

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@DisplayName("Pruebas de la calculadora")
class CalculadoraDisplayNameTest {

    @Test
    @DisplayName("Sumar dos números positivos da su suma")
    void sumaPositivos() {
        assertEquals(5, new Calculadora().sumar(2, 3));
    }

    @Test
    @DisplayName("Dividir entre cero lanza ArithmeticException")
    void divisionEntreCero() {
        assertThrows(ArithmeticException.class, () -> new Calculadora().dividir(1, 0));
    }
}
```

### Generación automática de nombres

Si no quieres escribir un `@DisplayName` en cada método, puedes usar `@DisplayNameGeneration`. Por ejemplo, para convertir los guiones bajos en espacios:

```java
import org.junit.jupiter.api.DisplayNameGeneration;
import org.junit.jupiter.api.DisplayNameGenerator;

@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class CalculadoraGeneradorNombresTest {

    @Test
    void sumar_dos_numeros_positivos() {
        // Aparecerá como: "sumar dos numeros positivos"
    }
}
```

### Convención de nombres: `método_condición_resultadoEsperado`

Un buen nombre de test funciona como un mensaje de error anticipado. Cuando una prueba falla en el informe del IDE o en la integración continua, lo primero que ves es el nombre. Si ese nombre es suficientemente claro, sabes **qué se rompió sin abrir el código**.

Un patrón muy extendido es dividir el nombre en tres partes separadas por guiones bajos:

```
método_condición_resultadoEsperado
```

| Parte | Responde a... | Ejemplo |
|---|---|---|
| **método** | ¿Qué se está probando? | `dividir` |
| **condición** | ¿En qué situación o con qué datos? | `entreCero` |
| **resultadoEsperado** | ¿Qué debería ocurrir? | `lanzaArithmeticException` |

Resultado: `dividir_entreCero_lanzaArithmeticException`.

Los guiones bajos separan las tres ideas, y dentro de cada parte se usa camelCase.

#### Antes y después

Nombres de los ejemplos de esta guía reescritos con el patrón:

| Nombre original | Con el patrón |
|---|---|
| `sumaDosNumerosPositivos` | `sumar_dosPositivos_devuelveLaSuma` |
| `dividirEntreCeroLanzaExcepcion` | `dividir_entreCero_lanzaArithmeticException` |
| `ingresarAumentaElSaldo` | `ingresar_cantidadValida_aumentaElSaldo` |
| `retirarMasDelSaldoLanzaExcepcion` | `retirar_masDelSaldo_lanzaExcepcion` |
| `noSePuedeCrearUnaCuentaConSaldoNegativo` | `crearCuenta_saldoNegativo_lanzaExcepcion` |

Los nombres originales ya eran descriptivos. El patrón añade **consistencia**: todos los tests dicen lo mismo en el mismo orden. Además te obliga a pensar en las tres partes, y si no sabes qué poner en alguna, quizá el test prueba demasiado.

#### Combinarlo con `@DisplayNameGeneration`

El generador `ReplaceUnderscores` encaja con este patrón, porque convierte cada guion bajo en un espacio:

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class CalculadoraTest {

    @Test
    void dividir_entreCero_lanzaArithmeticException() {
        assertThrows(ArithmeticException.class, () -> new Calculadora().dividir(1, 0));
    }
}
```

En el informe aparecerá como: `dividir entreCero lanzaArithmeticException`.

Ten en cuenta que `ReplaceUnderscores` **no separa el camelCase**, solo cambia los guiones bajos. Si quieres frases completas con espacios y tildes, usa `@DisplayName` y deja el nombre del método con el patrón para localizarlo en el código.

#### Buenas prácticas para los nombres

1. **Mantén las tres partes.** No omitas la condición ni el resultado, aunque parezcan obvios.
2. **Sé concreto en el resultado.** `lanzaArithmeticException` informa más que `lanzaExcepcion`, y `devuelveLaSuma` más que `funciona`.
3. **Un nombre largo es aceptable.** Los métodos de test no se llaman desde otro código, así que la claridad pesa más que la brevedad.
4. **Si el nombre necesita un "y", divide el test.** `ingresar_cantidadValida_aumentaSaldoYRegistraMovimiento` indica que hay dos comportamientos y que conviene escribir dos tests.
5. **Con `@Nested`, la condición puede ir en la clase.** Si agrupas tests por situación (por ejemplo, una clase `CuandoLaPilaEstaVacia`), el método puede quedarse en `pop_lanzaExcepcion`.

> **Nota:** los guiones bajos en nombres de método se salen de la convención habitual de Java (camelCase), aunque muchas guías de estilo los permiten expresamente en los tests. Lo importante es que **todo el equipo use la misma convención**.

---

## 9. Suposiciones (assumptions)

Una **suposición** (`assume...`) es una condición que debe cumplirse para que la prueba tenga sentido. Si no se cumple, la prueba **se aborta** (no se marca como fallida ni como correcta, sino como *abortada*).

Se encuentran en `org.junit.jupiter.api.Assumptions`:

```java
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

@Test
void pruebaSoloEnEntornoDeDesarrollo() {
    assumeTrue("DEV".equals(System.getenv("ENTORNO")),
        "Esta prueba solo tiene sentido en desarrollo");

    // Este código solo se ejecuta si la suposición se cumple
    assertEquals(2, 1 + 1);
}

@Test
void ejecutarSoloUnaParteSegunCondicion() {
    assumingThat("CI".equals(System.getenv("ENTORNO")),
        () -> {
            // Se ejecuta solo en integración continua
            assertEquals(2, 1 + 1);
        });

    // Esto se ejecuta siempre
    assertEquals(4, 2 + 2);
}
```

**Diferencia clave con las aserciones:** una aserción fallida = prueba fallida. Una suposición no cumplida = prueba abortada (no es un error).

---

## 10. Desactivar pruebas y ejecución condicional

### Desactivar temporalmente una prueba: `@Disabled`

```java
import org.junit.jupiter.api.Disabled;

@Disabled("Pendiente de corregir el error #123")
@Test
void pruebaQueAunNoFunciona() {
    // ...
}
```

También puede aplicarse a toda una clase. **Escribe siempre el motivo**; y no dejes pruebas desactivadas eternamente.

### Ejecutar solo en ciertas condiciones

JUnit Jupiter trae anotaciones para activar o desactivar pruebas según el entorno:

```java
import org.junit.jupiter.api.condition.*;

class PruebasCondicionalesTest {

    @Test
    @EnabledOnOs(OS.LINUX)
    void soloEnLinux() { /* ... */ }

    @Test
    @DisabledOnOs(OS.WINDOWS)
    void noEnWindows() { /* ... */ }

    @Test
    @EnabledOnJre(JRE.JAVA_21)
    void soloConJava21() { /* ... */ }

    @Test
    @EnabledIfEnvironmentVariable(named = "ENTORNO", matches = "staging")
    void soloEnStaging() { /* ... */ }

    @Test
    @EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
    void soloEn64Bits() { /* ... */ }
}
```

---

## 11. Pruebas parametrizadas

Imagina que quieres probar la misma lógica con muchos valores distintos. En vez de copiar y pegar métodos, escribe **una** prueba y dale varios conjuntos de datos.

Las pruebas parametrizadas usan `@ParameterizedTest` en lugar de `@Test` y necesitan al menos una **fuente de argumentos**. (Están incluidas en la dependencia `junit-jupiter` que añadiste.)

### `@ValueSource`: una lista de valores simples

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class PalindromosTest {

    @ParameterizedTest
    @ValueSource(strings = {"ana", "reconocer", "oso", "radar"})
    void detectaPalindromos(String palabra) {
        assertTrue(esPalindromo(palabra));
    }

    private boolean esPalindromo(String s) {
        return new StringBuilder(s).reverse().toString().equals(s);
    }
}
```

Admite `ints`, `longs`, `doubles`, `strings`, `booleans`, etc.

### `@CsvSource`: varios argumentos por ejecución

```java
import org.junit.jupiter.params.provider.CsvSource;

class SumaParametrizadaTest {

    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "-4, 4, 0",
        "10, 20, 30"
    })
    void sumaCorrectamente(int a, int b, int esperado) {
        assertEquals(esperado, new Calculadora().sumar(a, b));
    }
}
```

Cada fila es una ejecución; los valores se convierten automáticamente al tipo del parámetro.

### `@NullSource`, `@EmptySource` y `@NullAndEmptySource`

Ideales para probar casos límite:

```java
import org.junit.jupiter.params.provider.NullAndEmptySource;

@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {" ", "   "})
void textoEnBlancoNoEsValido(String texto) {
    assertTrue(texto == null || texto.isBlank());
}
```

### `@EnumSource`: todos los valores de un enum

```java
import org.junit.jupiter.params.provider.EnumSource;
import java.time.DayOfWeek;

@ParameterizedTest
@EnumSource(value = DayOfWeek.class, names = {"SATURDAY", "SUNDAY"})
void esFinDeSemana(DayOfWeek dia) {
    assertTrue(dia == DayOfWeek.SATURDAY || dia == DayOfWeek.SUNDAY);
}
```

### `@MethodSource`: datos generados por un método

Útil cuando los datos son objetos complejos:

```java
import java.util.stream.Stream;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.MethodSource;

class DivisionParametrizadaTest {

    @ParameterizedTest
    @MethodSource("casosDeDivision")
    void divideCorrectamente(int a, int b, int esperado) {
        assertEquals(esperado, new Calculadora().dividir(a, b));
    }

    static Stream<Arguments> casosDeDivision() {
        return Stream.of(
            Arguments.of(10, 2, 5),
            Arguments.of(9, 3, 3),
            Arguments.of(-8, 2, -4)
        );
    }
}
```

### Personalizar el nombre de cada ejecución

```java
@ParameterizedTest(name = "{0} + {1} = {2}")
@CsvSource({"1, 1, 2", "2, 3, 5"})
void suma(int a, int b, int esperado) {
    assertEquals(esperado, new Calculadora().sumar(a, b));
}
```

En el informe verás `1 + 1 = 2`, `2 + 3 = 5`… en lugar de un nombre genérico. `{0}`, `{1}`… son los argumentos por posición.

---

## 12. Pruebas anidadas

Con `@Nested` puedes **agrupar pruebas relacionadas** dentro de clases internas, organizadas por escenario. Es muy útil para expresar «cuando… entonces…».

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

@DisplayName("Una pila (Stack)")
class PilaTest {

    private java.util.Stack<String> pila;

    @Nested
    @DisplayName("cuando es nueva")
    class CuandoEsNueva {

        @BeforeEach
        void crearPila() {
            pila = new java.util.Stack<>();
        }

        @Test
        @DisplayName("está vacía")
        void estaVacia() {
            assertTrue(pila.isEmpty());
        }

        @Test
        @DisplayName("lanza EmptyStackException al hacer pop")
        void popLanzaExcepcion() {
            assertThrows(java.util.EmptyStackException.class, pila::pop);
        }

        @Nested
        @DisplayName("después de apilar un elemento")
        class DespuesDeApilar {

            @BeforeEach
            void apilar() {
                pila.push("elemento");
            }

            @Test
            @DisplayName("ya no está vacía")
            void noEstaVacia() {
                assertFalse(pila.isEmpty());
            }

            @Test
            @DisplayName("hacer pop devuelve el elemento")
            void popDevuelveElemento() {
                assertEquals("elemento", pila.pop());
            }
        }
    }
}
```

Los métodos `@BeforeEach` de las clases externas se ejecutan **antes** que los de las clases internas, de modo que cada nivel añade contexto al anterior.

> Las clases `@Nested` deben ser **clases internas no estáticas**.

---

## 13. Etiquetas y filtrado

Con `@Tag` clasificas las pruebas para poder ejecutar solo algunas (por ejemplo, las rápidas en tu equipo y las lentas en integración continua).

```java
import org.junit.jupiter.api.Tag;

class EtiquetasTest {

    @Test
    @Tag("rapida")
    void pruebaRapida() { /* ... */ }

    @Test
    @Tag("lenta")
    @Tag("integracion")
    void pruebaLenta() { /* ... */ }
}
```

### Filtrar por etiquetas en Maven

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.5.5</version>
    <configuration>
        <groups>rapida</groups>
        <excludedGroups>lenta</excludedGroups>
    </configuration>
</plugin>
```

### Filtrar por etiquetas en Gradle

```groovy
test {
    useJUnitPlatform {
        includeTags("rapida")
        excludeTags("lenta")
    }
}
```

Puedes usar expresiones, por ejemplo `rapida & !integracion`.

---

## 14. Orden de ejecución

**Por defecto**, el orden de las pruebas es determinista pero deliberadamente no obvio. Y esto es una característica: **tus pruebas nunca deben depender del orden**.

Aun así, si necesitas fijar un orden (por ejemplo, en pruebas de integración), puedes hacerlo:

```java
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrdenadasTest {

    @Test
    @Order(1)
    void primero() { /* ... */ }

    @Test
    @Order(2)
    void segundo() { /* ... */ }

    @Test
    @Order(3)
    void tercero() { /* ... */ }
}
```

También existen otros ordenadores, como `MethodOrderer.DisplayName` (alfabético por nombre visible) o `MethodOrderer.Random`.

---

## 15. Pruebas repetidas

`@RepeatedTest` ejecuta el mismo método un número determinado de veces. Es útil, por ejemplo, para detectar comportamientos intermitentes.

```java
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;

class RepetidasTest {

    @RepeatedTest(5)
    void seRepiteCincoVeces() {
        assertEquals(4, 2 + 2);
    }

    @RepeatedTest(value = 3, name = "Repetición {currentRepetition} de {totalRepetitions}")
    void conInformacion(RepetitionInfo info) {
        System.out.println("Ejecución número " + info.getCurrentRepetition());
    }
}
```

---

## 16. Tiempos límite (timeouts)

### Con la anotación `@Timeout`

```java
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.api.Timeout;

class TimeoutTest {

    @Test
    @Timeout(value = 2, unit = TimeUnit.SECONDS)
    void debeTerminarEnMenosDeDosSegundos() throws InterruptedException {
        Thread.sleep(100); // simula un trabajo rápido
    }
}
```

Si la prueba tarda más, **falla**.

### Con `assertTimeout`

Para limitar solo un fragmento de código:

```java
import static org.junit.jupiter.api.Assertions.assertTimeout;
import java.time.Duration;

@Test
void operacionRapida() {
    assertTimeout(Duration.ofMillis(500), () -> {
        // código que debe tardar menos de 500 ms
        new Calculadora().sumar(1, 2);
    });
}
```

---

## 17. Ficheros temporales con `@TempDir`

Cuando pruebes código que lee o escribe ficheros, no ensucies tu disco: JUnit puede crear un directorio temporal y borrarlo al terminar.

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import org.junit.jupiter.api.io.TempDir;

class FicheroTest {

    @Test
    void escribeYLeeUnFichero(@TempDir Path directorioTemporal) throws IOException {
        Path fichero = directorioTemporal.resolve("saludo.txt");

        Files.writeString(fichero, "Hola, JUnit");

        assertEquals("Hola, JUnit", Files.readString(fichero));
    }
}
```

Al terminar la prueba, JUnit elimina el directorio y su contenido automáticamente.

---

## 18. Pruebas dinámicas

Las pruebas normales (`@Test`) se definen en tiempo de compilación. Las **dinámicas** se generan en tiempo de ejecución con un método `@TestFactory`, que devuelve una colección o un flujo de objetos `DynamicTest`.

```java
import java.util.stream.Stream;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;

import static org.junit.jupiter.api.DynamicTest.dynamicTest;

class DinamicasTest {

    @TestFactory
    Stream<DynamicTest> tablaDelDos() {
        return Stream.of(1, 2, 3, 4, 5)
            .map(n -> dynamicTest(
                "2 x " + n + " = " + (2 * n),
                () -> assertEquals(2 * n, n * 2)
            ));
    }
}
```

> Para empezar, quédate con `@Test` y `@ParameterizedTest`. Las pruebas dinámicas solo son necesarias en casos avanzados.

---

## 19. Cómo ejecutar las pruebas

### Desde el IDE

IntelliJ IDEA, Eclipse, NetBeans y Visual Studio Code tienen soporte integrado: haz clic en el icono verde junto a la clase o al método, o clic derecho → *Run tests*.

### Con Maven

```bash
mvn test                       # ejecuta todas las pruebas
mvn test -Dtest=CalculadoraTest            # solo una clase
mvn test -Dtest=CalculadoraTest#sumaDosNumerosPositivos   # solo un método
```

### Con Gradle

```bash
./gradlew test
./gradlew test --tests "com.ejemplo.CalculadoraTest"
```

### Cómo interpretar el resultado

| Estado | Significado |
|---|---|
| ✅ **Pasa** | El método terminó sin excepciones |
| ❌ **Falla** | Una aserción no se cumplió (`AssertionFailedError`) |
| 💥 **Error** | Se lanzó una excepción inesperada |
| ⏭️ **Omitida / abortada** | La prueba estaba desactivada o una suposición no se cumplió |

Ejemplo de fallo:

```
org.opentest4j.AssertionFailedError: expected: <5> but was: <6>
```

Te dice qué esperabas (`5`) y qué obtuviste (`6`).

---

## 20. Buenas prácticas

1. **Una idea por prueba.** Cada prueba debería comprobar un único comportamiento.
2. **Nombres que expliquen el caso.** Mejor `retirarMasDelSaldoLanzaExcepcion` que `test1`.
3. **Pruebas independientes.** Ninguna debe depender de otra ni del orden de ejecución.
4. **Sigue el patrón AAA** (preparar, actuar, comprobar) y separa visualmente los bloques.
5. **Prueba los casos límite:** valores cero, negativos, nulos, cadenas vacías, colecciones vacías, máximos y mínimos.
6. **Prueba también los caminos de error**, no solo el caso feliz.
7. **Que sean rápidas.** Las pruebas unitarias deberían ejecutarse en milisegundos.
8. **Sin lógica compleja dentro de la prueba.** Evita bucles y condicionales; si los necesitas, plantéate una prueba parametrizada.
9. **Valor esperado primero** en `assertEquals(esperado, real)`.
10. **No dejes pruebas desactivadas** indefinidamente; arréglalas o bórralas.
11. **Las pruebas también son código:** mantenlas limpias y legibles.

---

## 21. Resumen rápido (chuleta)

### Anotaciones principales

| Anotación | Función |
|---|---|
| `@Test` | Marca un método de prueba |
| `@ParameterizedTest` | Prueba que se ejecuta con varios conjuntos de datos |
| `@RepeatedTest(n)` | Repite la prueba *n* veces |
| `@TestFactory` | Genera pruebas dinámicas |
| `@BeforeEach` / `@AfterEach` | Antes / después de cada prueba |
| `@BeforeAll` / `@AfterAll` | Una vez antes / después de todas (métodos `static`) |
| `@DisplayName("...")` | Nombre legible |
| `@Nested` | Agrupa pruebas en clases internas |
| `@Tag("...")` | Etiqueta para filtrar |
| `@Disabled` | Desactiva la prueba |
| `@Timeout` | Tiempo máximo de ejecución |
| `@TempDir` | Directorio temporal |
| `@TestMethodOrder` / `@Order` | Orden de ejecución |
| `@EnabledOnOs`, `@EnabledOnJre`, `@EnabledIf…` | Ejecución condicional |

### Aserciones principales

```java
assertEquals(esperado, real);
assertTrue(condicion);
assertFalse(condicion);
assertNull(objeto);
assertNotNull(objeto);
assertArrayEquals(arrayEsperado, arrayReal);
assertThrows(Excepcion.class, () -> codigo());
assertDoesNotThrow(() -> codigo());
assertAll(() -> ..., () -> ...);
assertTimeout(Duration.ofSeconds(1), () -> codigo());
```

### Plantilla mínima para copiar

```java
package com.ejemplo;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@DisplayName("Pruebas de MiClase")
class MiClaseTest {

    private MiClase objeto;

    @BeforeEach
    void preparar() {
        objeto = new MiClase();
    }

    @Test
    @DisplayName("hace lo que se espera en el caso normal")
    void casoNormal() {
        // Arrange
        // Act
        // Assert
    }

    @Test
    @DisplayName("lanza una excepción con datos no válidos")
    void casoDeError() {
        assertThrows(IllegalArgumentException.class, () -> objeto.metodo(null));
    }
}
```

---

## 22. Para seguir aprendiendo

- **Guía oficial de JUnit 6.1.3:** <https://docs.junit.org/6.1.3/overview.html>
- **Escribir pruebas (Writing Tests):** <https://docs.junit.org/6.1.3/writing-tests/intro.html>
- **Soporte de construcción (Maven, Gradle…):** <https://docs.junit.org/6.1.3/running-tests/build-support.html>
- **Migrar desde JUnit 4:** <https://docs.junit.org/6.1.3/migrating-from-junit4.html>
- **Proyectos de ejemplo:** <https://github.com/junit-team/junit-examples>
- **Dudas:** [Stack Overflow](https://stackoverflow.com/questions/tagged/junit5) o el apartado Q&A de [GitHub Discussions](https://github.com/junit-team/junit-framework/discussions/categories/q-a)

### Siguientes pasos recomendados

1. Crea un proyecto con Maven o Gradle y copia los ejemplos de esta guía.
2. Escribe pruebas para una clase tuya, empezando por el caso más sencillo.
3. Aprende a usar un *mock* (por ejemplo, con **Mockito**) para aislar las dependencias externas.
4. Mide la cobertura de tus pruebas con **JaCoCo**.
5. Practica el desarrollo guiado por pruebas (**TDD**): escribe primero la prueba y después el código.

---

*Contenido elaborado a partir de la documentación oficial de JUnit, publicada bajo la licencia Eclipse Public License v2.0.*
