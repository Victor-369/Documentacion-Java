# Test Driven Development (TDD) en Java: guía paso a paso desde cero

> Guía pensada para quien está empezando. Solo necesitas saber lo básico de Java (clases, métodos, `if`, bucles `for`). Todo lo demás se explica por el camino.
>
> Inspirada en el artículo [*Test-Driven Development (TDD) in Java: A Comprehensive Guide with Examples*](https://medium.com/@berrachdim/test-driven-development-tdd-in-java-a-comprehensive-guide-with-examples-c66a77afe036), de Berrachdi Mohamed, ampliada con más explicaciones y ejemplos completos.

---

## Índice

1. [¿Qué es TDD?](#1-qué-es-tdd)
2. [El ciclo Rojo → Verde → Refactor](#2-el-ciclo-rojo--verde--refactor)
3. [¿Por qué merece la pena?](#3-por-qué-merece-la-pena)
4. [Preparar el proyecto (Maven + JUnit 5)](#4-preparar-el-proyecto-maven--junit-5)
5. [Anatomía de un test](#5-anatomía-de-un-test)
6. [Ejemplo 1: calcular el factorial, paso a paso](#6-ejemplo-1-calcular-el-factorial-paso-a-paso)
7. [Ejemplo 2: validador de contraseñas](#7-ejemplo-2-validador-de-contraseñas)
8. [Buenas prácticas](#8-buenas-prácticas)
9. [Errores típicos de quien empieza](#9-errores-típicos-de-quien-empieza)
10. [Ejercicios para practicar](#10-ejercicios-para-practicar)
11. [Mini glosario](#11-mini-glosario)
12. [Conclusión](#12-conclusión)

---

## 1. ¿Qué es TDD?

**TDD** son las siglas de *Test Driven Development*, que en castellano se traduce como **desarrollo guiado por pruebas**.

La idea es muy simple, aunque al principio choca:

> **Primero escribes la prueba y después escribes el código que la hace pasar.**

Lo normal, cuando uno empieza a programar, es hacer justo lo contrario: escribir el código y, si acaso, probarlo a mano al final. En TDD damos la vuelta a ese orden.

Una **prueba** (o *test*) es un pequeño programa que comprueba automáticamente que otro trozo de código hace lo que debe. Por ejemplo: "si llamo a `sumar(2, 3)`, el resultado tiene que ser `5`".

### Una analogía

Imagina que vas a construir una estantería. Antes de coger el taladro, decides: "tiene que aguantar 20 kilos de libros". Ese es tu criterio de éxito. Después la construyes y compruebas que lo cumple. En TDD, el test es ese criterio de éxito, y lo defines **antes** de construir.

---

## 2. El ciclo Rojo → Verde → Refactor

TDD se basa en repetir un ciclo muy corto, de pocos minutos, con tres fases:

```
        ┌────────────────────────────────┐
        │                                │
        ▼                                │
   🔴 ROJO  ──────►  🟢 VERDE  ──────►  🔵 REFACTOR
 (test que falla)  (código mínimo)   (mejorar sin romper)
```

| Fase | Qué haces | Cómo sabes que has terminado |
|------|-----------|------------------------------|
| 🔴 **Rojo** (*Red*) | Escribes un test para una funcionalidad que **todavía no existe**. | Ejecutas el test y **falla**. |
| 🟢 **Verde** (*Green*) | Escribes el código **mínimo** para que el test pase. No hace falta que sea bonito. | Ejecutas el test y **pasa**. |
| 🔵 **Refactor** | Mejoras el código (nombres, duplicaciones, estructura) **sin cambiar su comportamiento**. | Todos los tests siguen en verde. |

Y vuelta a empezar con el siguiente pequeño requisito.

### ¿Por qué es importante ver el test fallar primero?

Porque si escribes un test y nunca lo has visto fallar, **no sabes si realmente comprueba algo**. Un test que siempre pasa es peor que no tener test: da una falsa sensación de seguridad.

---

## 3. ¿Por qué merece la pena?

- **Especificaciones claras.** Para escribir el test tienes que pensar antes qué debe hacer exactamente el código.
- **Menos errores.** Cada funcionalidad nace con su prueba.
- **Depuración más rápida.** Si algo se rompe, el test que falla te dice dónde mirar.
- **Refactorizar sin miedo.** Con una buena batería de tests puedes mejorar el código y saber al instante si has estropeado algo.
- **Documentación viva.** Los tests muestran cómo se usa el código y qué resultados se esperan, y a diferencia de un comentario, **no se quedan obsoletos** (si dejan de ser ciertos, fallan).
- **Mejor diseño.** Si un código es difícil de testear, suele ser señal de que está mal diseñado.
- **Trabajo en equipo.** Los tests son un contrato compartido sobre cómo debe comportarse el código.

Y una advertencia honesta: al principio parece que vas más lento. Es normal. Con la práctica se compensa de sobra, porque pasas mucho menos tiempo persiguiendo errores.

---

## 4. Preparar el proyecto (Maven + JUnit 5)

Para escribir tests en Java necesitamos un *framework* de pruebas. El más extendido es **JUnit 5** (también llamado *JUnit Jupiter*). Para gestionar el proyecto y sus dependencias usaremos **Maven**.

### Requisitos

- **JDK 17 o superior** instalado (`java -version` para comprobarlo).
- **Maven** instalado (`mvn -version`) o un IDE como IntelliJ IDEA, Eclipse o VS Code, que ya lo incluyen.

### Estructura de carpetas

Maven espera esta estructura. Respétala tal cual:

```
tdd-java/
├── pom.xml
└── src/
    ├── main/java/es/ejemplo/tdd/     ← aquí va el código de producción
    │   └── CalculadoraFactorial.java
    └── test/java/es/ejemplo/tdd/     ← aquí van los tests
        └── CalculadoraFactorialTest.java
```

La regla de oro: **el código real en `src/main`, los tests en `src/test`**, y la clase de test con el mismo nombre que la clase que prueba, terminada en `Test`.

### El fichero `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>es.ejemplo</groupId>
    <artifactId>tdd-java</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- JUnit 5: incluye las anotaciones, las aserciones y el motor de ejecución -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Necesario para que Maven ejecute los tests de JUnit 5 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### Cómo ejecutar los tests

Desde la carpeta del proyecto, en una terminal:

```bash
mvn test
```

En un IDE también puedes hacer clic derecho sobre la clase de test y elegir *Run* (o pulsar el icono verde de "play" junto al test).

---

## 5. Anatomía de un test

Antes de empezar con el ejemplo, veamos las piezas de un test con JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class SumadorTest {

    @Test
    void sumaDosNumerosPositivos() {
        // Arrange (preparar)
        Sumador sumador = new Sumador();

        // Act (actuar)
        int resultado = sumador.sumar(2, 3);

        // Assert (comprobar)
        assertEquals(5, resultado);
    }
}
```

### Las piezas

- **`@Test`**: anotación que le dice a JUnit "este método es un test, ejecútalo".
- **El método de test**: sin parámetros y con un nombre que **describe lo que se comprueba**.
- **`assertEquals(esperado, real)`**: una *aserción*. Compara el valor esperado con el obtenido. Si no coinciden, el test falla. Ojo con el orden: primero **lo esperado**, después **lo real**.

### El patrón AAA (Arrange – Act – Assert)

Casi todos los tests siguen tres pasos:

1. **Arrange** (preparar): creas los objetos y datos necesarios.
2. **Act** (actuar): ejecutas lo que quieres probar.
3. **Assert** (comprobar): verificas que el resultado es el esperado.

Si respetas esta estructura, tus tests serán fáciles de leer.

### Aserciones más habituales

| Aserción | Para qué sirve |
|----------|----------------|
| `assertEquals(esperado, real)` | Comprueba que dos valores son iguales. |
| `assertTrue(condicion)` | Comprueba que una condición es verdadera. |
| `assertFalse(condicion)` | Comprueba que una condición es falsa. |
| `assertNull(valor)` / `assertNotNull(valor)` | Comprueba si un valor es (o no es) `null`. |
| `assertThrows(Excepcion.class, () -> ...)` | Comprueba que el código lanza una excepción concreta. |

---

## 6. Ejemplo 1: calcular el factorial, paso a paso

**El requisito:** queremos una clase que calcule el factorial de un número.

Un recordatorio de matemáticas: el factorial de `n` (se escribe `n!`) es el producto de todos los enteros de 1 a `n`.

- `0! = 1` (por definición)
- `1! = 1`
- `5! = 5 × 4 × 3 × 2 × 1 = 120`
- Los números negativos no tienen factorial.

Vamos a construirlo con TDD, **en pequeños pasos**. Fíjate en que no empezamos por lo difícil, sino por el caso más sencillo.

---

### Iteración 1: el factorial de 0 es 1

#### 🔴 Rojo: escribimos el test

Creamos `src/test/java/es/ejemplo/tdd/CalculadoraFactorialTest.java`:

```java
package es.ejemplo.tdd;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculadoraFactorialTest {

    @Test
    void elFactorialDeCeroEsUno() {
        CalculadoraFactorial calculadora = new CalculadoraFactorial();

        long resultado = calculadora.calcular(0);

        assertEquals(1, resultado);
    }
}
```

Si intentas ejecutarlo ahora, **ni siquiera compila**, porque la clase `CalculadoraFactorial` no existe. Eso cuenta como "rojo": el test no puede pasar.

Para poder ver el fallo real, creamos primero un esqueleto vacío en `src/main/java/es/ejemplo/tdd/CalculadoraFactorial.java`:

```java
package es.ejemplo.tdd;

public class CalculadoraFactorial {

    public long calcular(int n) {
        return -1; // valor provisional, solo para que compile
    }
}
```

Ejecutamos `mvn test` y vemos algo así:

```
[ERROR] CalculadoraFactorialTest.elFactorialDeCeroEsUno:14
        expected: <1> but was: <-1>
```

✅ Tenemos el **rojo**: el test falla por la razón correcta (el resultado no es el esperado).

#### 🟢 Verde: el código mínimo

¿Cuál es lo mínimo imprescindible para que pase? Devolver `1`. Sí, literalmente:

```java
public long calcular(int n) {
    return 1;
}
```

Ejecutamos `mvn test`: **verde**. 🎉

> 💡 **"Pero esto es hacer trampa, ¿no?"** No: es TDD puro. Escribimos solo lo necesario para satisfacer los tests que existen *hasta ahora*. Nuestro siguiente test nos obligará a generalizar.

#### 🔵 Refactor

Con tan poco código no hay nada que mejorar. Pasamos a la siguiente iteración.

---

### Iteración 2: el factorial de 5 es 120

#### 🔴 Rojo

Añadimos un segundo test a la clase de test:

```java
@Test
void elFactorialDeCincoEsCientoVeinte() {
    CalculadoraFactorial calculadora = new CalculadoraFactorial();

    long resultado = calculadora.calcular(5);

    assertEquals(120, resultado);
}
```

Ejecutamos:

```
[ERROR] CalculadoraFactorialTest.elFactorialDeCincoEsCientoVeinte
        expected: <120> but was: <1>
```

🔴 Rojo. Nuestro `return 1;` ya no sirve: el test nos **obliga** a implementar la lógica real.

#### 🟢 Verde

Implementamos el cálculo con un bucle:

```java
public long calcular(int n) {
    long resultado = 1;
    for (int i = 2; i <= n; i++) {
        resultado = resultado * i;
    }
    return resultado;
}
```

Comprobamos con `n = 0`: el bucle empieza en `i = 2`, no entra ni una vez y devuelve `1`. ✅
Con `n = 5`: `1 × 2 × 3 × 4 × 5 = 120`. ✅

Ejecutamos `mvn test`: los dos tests en **verde**.

#### 🔵 Refactor

El código es corto y claro. Lo único que podemos mejorar está en los tests: ambos crean su propia `CalculadoraFactorial`. Podemos extraerla a un campo:

```java
class CalculadoraFactorialTest {

    private final CalculadoraFactorial calculadora = new CalculadoraFactorial();

    @Test
    void elFactorialDeCeroEsUno() {
        assertEquals(1, calculadora.calcular(0));
    }

    @Test
    void elFactorialDeCincoEsCientoVeinte() {
        assertEquals(120, calculadora.calcular(5));
    }
}
```

JUnit crea una instancia nueva de la clase de test para cada método, así que no hay riesgo de que un test afecte a otro. Volvemos a ejecutar: siguen en verde. ✅

---

### Iteración 3: los números negativos no son válidos

#### 🔴 Rojo

¿Qué debería pasar con `calcular(-3)`? Tomamos una decisión de diseño: lanzar una excepción `IllegalArgumentException`. Lo expresamos con un test:

```java
import static org.junit.jupiter.api.Assertions.assertThrows;

@Test
void lanzaExcepcionSiElNumeroEsNegativo() {
    assertThrows(IllegalArgumentException.class, () -> calculadora.calcular(-3));
}
```

Ejecutamos. Ahora mismo `calcular(-3)` devuelve `1` sin quejarse, así que el test falla:

```
[ERROR] lanzaExcepcionSiElNumeroEsNegativo
        Expected IllegalArgumentException to be thrown, but nothing was thrown.
```

🔴 Rojo.

#### 🟢 Verde

Añadimos la validación al principio del método:

```java
public long calcular(int n) {
    if (n < 0) {
        throw new IllegalArgumentException("El número no puede ser negativo: " + n);
    }
    long resultado = 1;
    for (int i = 2; i <= n; i++) {
        resultado = resultado * i;
    }
    return resultado;
}
```

Ejecutamos: los tres tests en **verde**. ✅

#### 🔵 Refactor

De momento el código está limpio. Seguimos.

---

### Iteración 4: probar muchos valores a la vez

Ya tenemos la base. Ahora conviene comprobar más casos: `1`, `2`, `3`, `10`... Escribir un test por cada uno sería repetitivo. JUnit 5 ofrece **tests parametrizados**, que ejecutan el mismo test con distintos datos.

Necesitamos importar unas anotaciones más (ya vienen incluidas en la dependencia `junit-jupiter` que pusimos en el `pom.xml`):

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

@ParameterizedTest
@CsvSource({
    "0, 1",
    "1, 1",
    "2, 2",
    "3, 6",
    "4, 24",
    "5, 120",
    "10, 3628800",
    "20, 2432902008176640000"
})
void calculaElFactorialDeVariosNumeros(int entrada, long esperado) {
    assertEquals(esperado, calculadora.calcular(entrada));
}
```

Cada línea de `@CsvSource` es un caso: el primer valor va a `entrada` y el segundo a `esperado`. Al ejecutar, JUnit lo tratará como 8 tests independientes.

Ejecutamos y... **todos pasan a la primera**. ¿Es un problema? No: aquí no estamos añadiendo funcionalidad nueva, sino reforzando la confianza en la que ya tenemos. Es normal que a veces un test nuevo nazca en verde. Lo importante es que, cuando sí añadimos comportamiento nuevo, empecemos siempre por un test rojo.

---

### Iteración 5: ¿y si el resultado no cabe en un `long`?

Un `long` en Java llega, como mucho, a unos 9,22 × 10¹⁸. El factorial de 20 (`2 432 902 008 176 640 000`) cabe, pero el de 21 **no**. ¿Qué pasa si lo pedimos?

#### 🔴 Rojo

Vamos a decidir que, en ese caso, queremos un error claro y no un número absurdo:

```java
@Test
void lanzaExcepcionSiElResultadoDesbordaUnLong() {
    assertThrows(ArithmeticException.class, () -> calculadora.calcular(21));
}
```

Ejecutamos y falla: Java **desborda en silencio** y devuelve un número negativo sin avisar (`-4249290049419214848`). Este es justo el tipo de fallo traicionero que los tests ayudan a descubrir.

#### 🟢 Verde

Java tiene un método que lanza `ArithmeticException` cuando una multiplicación desborda: `Math.multiplyExact`. Lo usamos:

```java
resultado = Math.multiplyExact(resultado, i);
```

Ejecutamos: todo **verde**. ✅

#### 🔵 Refactor

Cambiamos el mensaje de error por una constante más legible y dejamos el código final. Este es el resultado tras todas las iteraciones:

```java
package es.ejemplo.tdd;

public class CalculadoraFactorial {

    public long calcular(int n) {
        if (n < 0) {
            throw new IllegalArgumentException(
                "No existe el factorial de un número negativo: " + n);
        }

        long resultado = 1;
        for (int i = 2; i <= n; i++) {
            resultado = Math.multiplyExact(resultado, i); // falla si desborda
        }
        return resultado;
    }
}
```

Y la clase de test completa:

```java
package es.ejemplo.tdd;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class CalculadoraFactorialTest {

    private final CalculadoraFactorial calculadora = new CalculadoraFactorial();

    @Test
    void elFactorialDeCeroEsUno() {
        assertEquals(1, calculadora.calcular(0));
    }

    @Test
    void elFactorialDeCincoEsCientoVeinte() {
        assertEquals(120, calculadora.calcular(5));
    }

    @Test
    void lanzaExcepcionSiElNumeroEsNegativo() {
        assertThrows(IllegalArgumentException.class, () -> calculadora.calcular(-3));
    }

    @ParameterizedTest
    @CsvSource({
        "0, 1",
        "1, 1",
        "2, 2",
        "3, 6",
        "4, 24",
        "5, 120",
        "10, 3628800",
        "20, 2432902008176640000"
    })
    void calculaElFactorialDeVariosNumeros(int entrada, long esperado) {
        assertEquals(esperado, calculadora.calcular(entrada));
    }

    @Test
    void lanzaExcepcionSiElResultadoDesbordaUnLong() {
        assertThrows(ArithmeticException.class, () -> calculadora.calcular(21));
    }
}
```

### Qué hemos aprendido con este ejemplo

- Empezamos por el caso más simple y fuimos **añadiendo un requisito cada vez**.
- El test rojo nos **obligó** a escribir cada trozo de código.
- Encontramos un problema real (el desbordamiento) **gracias a pensar en casos límite** al escribir tests.
- Al final tenemos código funcionando **y** una red de seguridad que lo protege.

---

## 7. Ejemplo 2: validador de contraseñas

Practiquemos con otro caso, esta vez más rápido. Vamos a ver solo lo esencial de cada iteración; intenta seguirlo tú en tu ordenador.

**Requisitos:** una contraseña es válida si:

1. Tiene al menos 8 caracteres.
2. Contiene al menos un dígito.
3. Contiene al menos una letra mayúscula.

Empezamos con esqueleto y test.

### Iteración 1: longitud mínima

🔴 **Rojo**

```java
package es.ejemplo.tdd;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

class ValidadorContrasenaTest {

    private final ValidadorContrasena validador = new ValidadorContrasena();

    @Test
    void rechazaContrasenasDeMenosDeOchoCaracteres() {
        assertFalse(validador.esValida("Ab1"));
    }
}
```

Con un esqueleto que devuelva siempre `true`, el test falla.

🟢 **Verde**

```java
package es.ejemplo.tdd;

public class ValidadorContrasena {

    public boolean esValida(String contrasena) {
        return contrasena.length() >= 8;
    }
}
```

### Iteración 2: hace falta un dígito

🔴 **Rojo**

```java
@Test
void rechazaContrasenasSinDigitos() {
    assertFalse(validador.esValida("Abcdefghi"));
}
```

Falla: `"Abcdefghi"` tiene 9 caracteres y nuestro código la da por buena.

🟢 **Verde**

```java
public boolean esValida(String contrasena) {
    return contrasena.length() >= 8
        && contrasena.chars().anyMatch(Character::isDigit);
}
```

### Iteración 3: hace falta una mayúscula

🔴 **Rojo**

```java
@Test
void rechazaContrasenasSinMayusculas() {
    assertFalse(validador.esValida("abcdefg1"));
}
```

🟢 **Verde**

```java
public boolean esValida(String contrasena) {
    return contrasena.length() >= 8
        && contrasena.chars().anyMatch(Character::isDigit)
        && contrasena.chars().anyMatch(Character::isUpperCase);
}
```

### Iteración 4: el caso feliz

No olvides comprobar también que lo **correcto** se acepta:

```java
@Test
void aceptaUnaContrasenaValida() {
    assertTrue(validador.esValida("Segura123"));
}
```

Este test pasa directamente. Está bien: cubre el camino que más nos importa.

### Iteración 5: ¿y si la contraseña es `null`?

🔴 **Rojo**

```java
@Test
void rechazaContrasenasNulas() {
    assertFalse(validador.esValida(null));
}
```

Falla con un `NullPointerException`, porque intentamos hacer `null.length()`.

🟢 **Verde** y 🔵 **Refactor**: aprovechamos para dejar el código más legible, separando cada regla en un método con nombre expresivo:

```java
package es.ejemplo.tdd;

public class ValidadorContrasena {

    private static final int LONGITUD_MINIMA = 8;

    public boolean esValida(String contrasena) {
        return contrasena != null
            && tieneLongitudMinima(contrasena)
            && contieneDigito(contrasena)
            && contieneMayuscula(contrasena);
    }

    private boolean tieneLongitudMinima(String contrasena) {
        return contrasena.length() >= LONGITUD_MINIMA;
    }

    private boolean contieneDigito(String contrasena) {
        return contrasena.chars().anyMatch(Character::isDigit);
    }

    private boolean contieneMayuscula(String contrasena) {
        return contrasena.chars().anyMatch(Character::isUpperCase);
    }
}
```

Ejecutamos todos los tests: siguen en verde. Gracias a ellos hemos podido reorganizar el código **con total tranquilidad**. Esa es la magia del refactor en TDD.

---

## 8. Buenas prácticas

1. **Da pasos pequeños.** Un test, un comportamiento. Si un ciclo te lleva más de 10 minutos, el paso era demasiado grande.
2. **Un test, una idea.** Cada test debería comprobar una sola cosa, para que al fallar sepas exactamente qué se ha roto.
3. **Nombres descriptivos.** `rechazaContrasenasSinDigitos` dice mucho más que `test2`.
4. **Tests independientes.** Ningún test debe depender de que otro se haya ejecutado antes.
5. **Tests rápidos.** Deberían ejecutarse en milisegundos. Si son lentos, dejarás de ejecutarlos.
6. **Escribe el código mínimo en verde.** No te adelantes ni "aproveches" para añadir cosas que nadie ha pedido todavía.
7. **Refactoriza siempre con los tests en verde.** Nunca mezcles cambiar comportamiento y mejorar estructura a la vez.
8. **Prueba los casos límite:** el valor cero, valores negativos, `null`, cadenas vacías, el máximo y el mínimo permitidos...
9. **Trata el código de test con el mismo cariño que el de producción.** También se lee y se mantiene.
10. **Ejecuta los tests continuamente.** Cuantas más veces, antes detectas los problemas.

---

## 9. Errores típicos de quien empieza

| Error | Por qué es un problema | Qué hacer |
|-------|------------------------|-----------|
| Escribir el código primero y el test después | Pierdes los beneficios de diseño y no sabes si el test detecta fallos. | Resiste la tentación: test primero. |
| No ver el test fallar | Podrías tener un test que siempre pasa. | Ejecútalo siempre en rojo antes de implementar. |
| Escribir tests enormes | Cuando fallan, no sabes por qué. | Divide en varios tests pequeños. |
| Implementar de golpe la solución final | Te saltas la guía que proporcionan los tests. | Haz el mínimo para pasar el test actual. |
| Confundir el orden en `assertEquals` | Los mensajes de error salen al revés y confunden. | Recuerda: `assertEquals(esperado, real)`. |
| Saltarse el refactor | El código se va ensuciando poco a poco. | Dedica un momento en cada ciclo a limpiar. |
| Probar detalles internos en lugar del comportamiento | Los tests se rompen con cualquier cambio interno. | Prueba **qué hace** el código, no **cómo** lo hace. |
| Querer el 100 % de cobertura a toda costa | Genera tests inútiles solo para cumplir un número. | Céntrate en probar lo que importa. |

---

## 10. Ejercicios para practicar

Hazlos siguiendo estrictamente el ciclo 🔴 → 🟢 → 🔵. Empieza siempre por el caso más simple.

### Ejercicio 1: `FizzBuzz`
Escribe una clase que, dado un número, devuelva:
- `"Fizz"` si es múltiplo de 3,
- `"Buzz"` si es múltiplo de 5,
- `"FizzBuzz"` si es múltiplo de ambos,
- el propio número convertido a texto en cualquier otro caso.

### Ejercicio 2: `Palindromo`
Un método que diga si una palabra es un palíndromo (se lee igual del derecho que del revés, como `"reconocer"`). Pistas para los tests: cadena vacía, una sola letra, mayúsculas y minúsculas mezcladas, `null`.

### Ejercicio 3: `NumerosRomanos`
Convierte un número entero (1 a 3999) a numeración romana. Empieza por `1 → "I"`, luego `2 → "II"`, `4 → "IV"`... y deja que los tests te guíen.

### Ejercicio 4: `CarritoDeLaCompra`
Una clase con métodos para añadir productos (nombre y precio), calcular el total y aplicar un descuento porcentual. Piensa qué ocurre con un carrito vacío o con un descuento superior al 100 %.

### Ejercicio 5: `AnoBisiesto`
Un año es bisiesto si es divisible entre 4, excepto los divisibles entre 100, salvo que también lo sean entre 400. Prueba con 1900, 2000, 2023 y 2024.

---

## 11. Mini glosario

- **Test unitario:** prueba automática de una pequeña pieza de código (normalmente una clase o un método) de forma aislada.
- **Aserción (*assertion*):** comprobación dentro de un test. Si no se cumple, el test falla.
- **Regresión:** error que aparece en algo que antes funcionaba, normalmente por un cambio reciente.
- **Refactorizar:** reorganizar el código para mejorarlo sin cambiar lo que hace.
- **Caso límite (*edge case*):** situación extrema o poco habitual (vacío, cero, máximo, `null`...).
- **Cobertura (*coverage*):** porcentaje del código que es ejecutado por los tests.
- **Test parametrizado:** un mismo test que se ejecuta varias veces con datos distintos.
- **Framework de pruebas:** herramienta que facilita escribir y ejecutar tests (por ejemplo, JUnit).

---

## 12. Conclusión

TDD no consiste en "escribir muchos tests". Consiste en **dejar que las pruebas guíen el diseño** del código, avanzando con pasos pequeños y seguros:

1. 🔴 Escribe un test que falle.
2. 🟢 Haz lo mínimo para que pase.
3. 🔵 Mejora el código con la tranquilidad de que los tests te cubren las espaldas.
4. 🔁 Repite.

Al principio te parecerá lento y antinatural. Es completamente normal. Practica con los ejercicios anteriores y, en pocas semanas, empezarás a notar que programas con más calma, más seguridad y menos sustos.

### Para seguir aprendiendo

- Documentación oficial de JUnit 5: <https://junit.org/junit5/docs/current/user-guide/>
- Cuando domines lo básico, investiga **Mockito** (para simular dependencias en los tests) y **AssertJ** (aserciones más expresivas).
- Libro recomendado: *Test-Driven Development: By Example*, de Kent Beck, quien popularizó esta técnica.

¡Ánimo y a practicar! 🚀
