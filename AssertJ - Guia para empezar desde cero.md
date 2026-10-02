# AssertJ: guía para empezar desde cero

> Documentación introductoria sobre **AssertJ**, la librería de aserciones fluidas para Java y la JVM.
> Basada en la documentación oficial (<https://assertj.github.io/doc/>) y en el repositorio del proyecto (<https://github.com/assertj/assertj>). Las versiones de los ejemplos corresponden a AssertJ Core **3.27.7**, la que figuraba en la documentación oficial en el momento de escribir esta guía.

---

## Índice

1. [¿Qué es AssertJ?](#1-qué-es-assertj)
2. [El proyecto en GitHub: finalidad y uso](#2-el-proyecto-en-github-finalidad-y-uso)
3. [Módulos de AssertJ](#3-módulos-de-assertj)
4. [Requisitos e instalación](#4-requisitos-e-instalación)
5. [Tu primer test](#5-tu-primer-test)
6. [Cómo funciona: el patrón `assertThat(...)`](#6-cómo-funciona-el-patrón-assertthat)
7. [Aserciones por tipo de dato](#7-aserciones-por-tipo-de-dato)
8. [Colecciones y arrays en profundidad](#8-colecciones-y-arrays-en-profundidad)
9. [Aserciones sobre excepciones](#9-aserciones-sobre-excepciones)
10. [Mejorar los mensajes de error](#10-mejorar-los-mensajes-de-error)
11. [Errores típicos que debes evitar](#11-errores-típicos-que-debes-evitar)
12. [Soft assertions (aserciones "suaves")](#12-soft-assertions-aserciones-suaves)
13. [Comparación recursiva campo a campo](#13-comparación-recursiva-campo-a-campo)
14. [Otros puntos de entrada: `WithAssertions` y estilo BDD](#14-otros-puntos-de-entrada-withassertions-y-estilo-bdd)
15. [Configuración del IDE](#15-configuración-del-ide)
16. [Recursos y dónde pedir ayuda](#16-recursos-y-dónde-pedir-ayuda)
17. [Resumen rápido (chuleta)](#17-resumen-rápido-chuleta)

---

## 1. ¿Qué es AssertJ?

**AssertJ** es una librería Java que ofrece un conjunto amplio de **aserciones** (comprobaciones) para tus tests, con **mensajes de error muy útiles**, y que mejora la **legibilidad** del código de pruebas. Está pensada para ser muy fácil de usar desde tu IDE gracias al autocompletado.

### ¿Qué es una aserción?

Una aserción es una comprobación dentro de un test: "esto debe valer tal cosa". Si no se cumple, el test falla.

### ¿Por qué usar AssertJ?

Con JUnit "a secas" escribirías algo así:

```java
assertEquals("Frodo", personaje.getNombre());
assertTrue(lista.contains("Sam"));
```

Con AssertJ, el mismo código se lee casi como una frase en inglés:

```java
assertThat(personaje.getNombre()).isEqualTo("Frodo");
assertThat(lista).contains("Sam");
```

Ventajas principales:

- **Fluidez**: encadenas varias comprobaciones en una sola sentencia.
- **Autocompletado**: escribes `assertThat(objeto).` y el IDE te muestra solo las aserciones que tienen sentido para ese tipo de dato.
- **Mensajes de error claros**: te dicen qué esperabas y qué obtuviste.
- **Independiente del framework de test**: funciona con JUnit, TestNG o cualquier otro.

---

## 2. El proyecto en GitHub: finalidad y uso

🔗 **Repositorio oficial:** <https://github.com/assertj/assertj>

### ¿Cuál es la finalidad del proyecto?

El repositorio `assertj/assertj` es el **código fuente oficial** de AssertJ. Su descripción es *"Fluent testing assertions for Java and the JVM"* (aserciones de test fluidas para Java y la JVM). La ambición declarada del proyecto es ofrecer un conjunto **rico e intuitivo de aserciones fuertemente tipadas** para tests unitarios.

La idea central es que las aserciones deben ser **específicas del tipo de objeto** que estás comprobando:

- ¿Compruebas un `String`? Usas aserciones de `String`.
- ¿Compruebas un `Map`? Usas aserciones de `Map`.

Todo empieza igual: escribes `assertThat(loQueSeaQueEstasProbando).` y el autocompletado te muestra lo que puedes verificar.

### ¿Para qué se usa este repositorio?

| Uso | Descripción |
| --- | --- |
| **Código fuente** | Contiene la implementación de la librería (por ejemplo, los módulos `assertj-core` y `assertj-guava`, además de `assertj-bom`, `assertj-parent` y `assertj-tests`). |
| **Seguimiento de incidencias** | En la pestaña *Issues* puedes reportar errores o proponer aserciones que echas en falta. |
| **Contribuciones** | Si falta una aserción útil, el proyecto anima a abrir una *issue* para discutirla y, mejor aún, a enviar un *pull request*. Las normas están en el archivo `CONTRIBUTING.md`. |
| **Discusiones y wiki** | El repositorio incluye *Discussions* y *Wiki* como espacios de comunidad y documentación. |
| **Licencia** | El proyecto usa la licencia **Apache-2.0**, una licencia de código abierto. |

> **Nota para quien aprende:** normalmente **no necesitas clonar este repositorio** para usar AssertJ en tus proyectos. Basta con añadir la dependencia con Maven o Gradle (ver [sección 4](#4-requisitos-e-instalación)). El repositorio es útil si quieres leer el código, reportar un problema o contribuir.

### Otros enlaces útiles del repositorio

- Documentación oficial: <https://assertj.github.io/doc/>
- Javadoc de AssertJ Core: <https://www.javadoc.io/doc/org.assertj/assertj-core/latest/index.html>
- Preguntas y respuestas: [Stack Overflow, etiqueta `assertj`](https://stackoverflow.com/questions/tagged/assertj)
- Guía de contribución: <https://github.com/assertj/assertj/blob/main/CONTRIBUTING.md>
- Proyecto de ejemplos: <https://github.com/assertj/assertj-examples>

---

## 3. Módulos de AssertJ

AssertJ está dividido en varios módulos. Como principiante, **solo necesitas el módulo Core**.

| Módulo | Para qué sirve |
| --- | --- |
| **Core** | Aserciones para tipos del JDK: `String`, `Iterable`, `Stream`, `Path`, `File`, `Map`, etc. **Es el que usarás el 99 % del tiempo.** |
| **Guava** | Aserciones para tipos de la librería Guava (`Multimap`, `Optional`, etc.). |
| **Joda Time** | Aserciones para tipos de Joda Time (`DateTime`, `LocalDateTime`). |
| **Neo4J** | Aserciones para tipos de Neo4J (`Path`, `Node`, `Relationship`...). |
| **DB** | Aserciones para bases de datos relacionales (`Table`, `Row`, `Column`...). |
| **Swing** | API sencilla para tests funcionales de interfaces gráficas Swing. |

---

## 4. Requisitos e instalación

### Requisitos

- **Java 8 o superior** para AssertJ Core.
- Compatible con **Kotlin 1.9 o superior**.
- **Android**: no está oficialmente soportado, pero es compatible con API Level 26+ (excepto *soft assertions* y *assumptions*).

### Maven

```xml
<dependency>
  <groupId>org.assertj</groupId>
  <artifactId>assertj-core</artifactId>
  <version>3.27.7</version>
  <scope>test</scope>
</dependency>
```

### Gradle

```groovy
testImplementation("org.assertj:assertj-core:3.27.7")
```

### Spring Boot

Si usas Spring Boot, `spring-boot-starter-test` ya incluye AssertJ Core (junto con JUnit Jupiter, Mockito, etc.) y gestiona su versión automáticamente. No necesitas añadirlo a mano.

Si quisieras forzar otra versión, se sobrescribe la propiedad `assertj.version`:

```xml
<properties>
  <assertj.version>3.27.7</assertj.version>
</properties>
```

```groovy
ext['assertj.version'] = '3.27.7'
```

---

## 5. Tu primer test

```java
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;

class PrimerTest {

  @Test
  void unas_cuantas_aserciones_simples() {
    assertThat("El Señor de los Anillos")
        .isNotNull()
        .startsWith("El")
        .contains("Señor")
        .endsWith("Anillos");
  }
}
```

Qué está pasando, paso a paso:

1. Importamos estáticamente `assertThat`, el punto de entrada de AssertJ.
2. Pasamos **el objeto que queremos comprobar** como único argumento de `assertThat(...)`.
3. Encadenamos tantas aserciones como necesitemos. Salvo `isNotNull()`, que es común a todos los tipos, las demás (`startsWith`, `contains`, `endsWith`) son **específicas de `String`** porque el objeto comprobado es un `String`.

---

## 6. Cómo funciona: el patrón `assertThat(...)`

Todo en AssertJ sigue la misma estructura:

```java
assertThat( objetoAComprobar ).aserción1().aserción2().aserciónN();
```

### El import más habitual

La clase `Assertions` es la única que necesitas para empezar. La forma más cómoda es un import estático con comodín:

```java
import static org.assertj.core.api.Assertions.*;
```

O, si prefieres imports individuales:

```java
import static org.assertj.core.api.Assertions.assertThat;  // el principal
import static org.assertj.core.api.Assertions.atIndex;      // para listas
import static org.assertj.core.api.Assertions.entry;        // para mapas
import static org.assertj.core.api.Assertions.tuple;        // al extraer varias propiedades
import static org.assertj.core.api.Assertions.offset;       // para números decimales
```

### Usa el autocompletado

Escribe `assertThat(objeto).` y pulsa la tecla de autocompletar de tu IDE: verás únicamente las aserciones aplicables a ese tipo. Es la mejor forma de descubrir qué puedes comprobar.

---

## 7. Aserciones por tipo de dato

AssertJ tiene aserciones específicas para muchos tipos. Los grupos principales que soporta son:

- **Comunes**: `BigDecimal`, `BigInteger`, `CharSequence`/`String`, `Class`, `Date`, `File`, `Future`/`CompletableFuture`, `InputStream`, `Iterable` (cualquier `Collection`), `Iterator`, `List`, `Map`, `Object`, arrays, `Optional` (y `OptionalInt`/`Long`/`Double`), `Path`, `Predicate`, `Stream`, `Throwable`/`Exception`.
- **Primitivos y sus wrappers**: `short`, `int`, `long`, `byte`, `char`, `float`, `double`, y sus arrays (1D y 2D).
- **Tipos temporales de Java 8**: `Instant`, `LocalDate`, `LocalDateTime`, `LocalTime`, `OffsetDateTime`, `OffsetTime`, `ZonedDateTime`, `Period`.
- **Tipos atómicos**: `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, etc.

Para ver **todas** las aserciones disponibles de cada tipo, consulta el [Javadoc](https://www.javadoc.io/doc/org.assertj/assertj-core/latest/index.html), donde cada aserción está explicada, muchas con ejemplos.

### Ejemplos básicos

```java
// Objetos en general
assertThat(personaje).isNotNull();
assertThat(frodo).isNotEqualTo(sauron);
assertThat(frodo).isIn(comunidadDelAnillo);

// Strings
assertThat(frodo.getNombre()).startsWith("Fro")
                             .endsWith("do")
                             .isEqualToIgnoringCase("FRODO");

// Números
assertThat(frodo.getEdad()).isEqualTo(33)
                           .isGreaterThan(18)
                           .isLessThan(100);

// Booleanos
assertThat(frodo.esHobbit()).isTrue();

// Números decimales (con margen de error)
assertThat(3.14159).isCloseTo(3.14, within(0.01));

// Mapas
assertThat(mapaEdades).containsEntry("Frodo", 33)
                      .containsKey("Sam")
                      .hasSize(3);

// Optional
assertThat(optionalConValor).isPresent();
assertThat(optionalVacio).isEmpty();
```

---

## 8. Colecciones y arrays en profundidad

Las colecciones son donde AssertJ más brilla. Estos son los conceptos clave.

### 8.1. Comprobar el contenido: la familia `contains`

| Aserción | Qué verifica |
| --- | --- |
| `contains` | Contiene los valores dados, **en cualquier orden** (puede tener más). |
| `containsOnly` | Contiene **solo** esos valores, en cualquier orden e ignorando duplicados. |
| `containsExactly` | Contiene **exactamente** esos valores y **en ese orden**. |
| `containsExactlyInAnyOrder` | Contiene exactamente esos valores, **en cualquier orden**. |
| `containsSequence` | Contiene esa secuencia, en orden y **sin valores intermedios**. |
| `containsSubsequence` | Contiene esa subsecuencia en orden, **pudiendo haber valores intermedios**. |
| `containsOnlyOnce` | Contiene los valores dados **una sola vez**. |
| `containsAnyOf` | Contiene **al menos uno** de los valores dados (como un "o"). |

Ejemplo:

```java
List<String> hobbits = List.of("Frodo", "Sam", "Merry", "Pippin");

assertThat(hobbits).hasSize(4)
                   .contains("Frodo", "Sam")
                   .doesNotContain("Sauron")
                   .containsExactly("Frodo", "Sam", "Merry", "Pippin");
```

### 8.2. Comprobar sobre algunos elementos: `satisfy` y `match`

```java
// TODOS los elementos deben cumplir las aserciones
assertThat(hobbits).allSatisfy(h -> assertThat(h.getRaza()).isEqualTo(HOBBIT));

// AL MENOS UNO debe cumplirlas
assertThat(hobbits).anySatisfy(h -> assertThat(h.getNombre()).isEqualTo("Sam"));

// NINGUNO debe cumplirlas
assertThat(hobbits).noneSatisfy(h -> assertThat(h.getRaza()).isEqualTo(ELFO));

// Variante con predicados (y descripción opcional para el mensaje de error)
assertThat(hobbits).allMatch(h -> h.getRaza() == HOBBIT, "son hobbits")
                   .anyMatch(h -> h.getNombre().contains("pp"))
                   .noneMatch(h -> h.getRaza() == ORCO);
```

### 8.3. Navegar a un elemento concreto

Con `first()`, `last()`, `element(indice)` y `singleElement()` te "posicionas" en un elemento para comprobarlo:

```java
assertThat(hobbits).first().isEqualTo(frodo);
assertThat(hobbits).last().isEqualTo(pippin);

// Para tener aserciones de String tras navegar, indica el tipo con as(STRING)
assertThat(nombres).first(as(STRING)).startsWith("fro").endsWith("do");
```

> Tras navegar, por defecto solo tienes aserciones genéricas de objeto, salvo que indiques el tipo (como `as(STRING)` en el ejemplo).

### 8.4. Filtrar antes de comprobar (`filteredOn`)

```java
// Con una expresión lambda
assertThat(comunidad).filteredOn(p -> p.getNombre().contains("o"))
                     .containsOnly(aragorn, frodo, legolas, boromir);

// Por nombre de propiedad/campo y valor
assertThat(comunidad).filteredOn("raza", HOBBIT)
                     .containsOnly(sam, frodo, pippin, merry);

// Con operadores: not, in, notIn
assertThat(comunidad).filteredOn("raza", not(HOBBIT))
                     .containsOnly(gandalf, boromir, aragorn, gimli, legolas);

// Propiedades anidadas
assertThat(comunidad).filteredOn("raza.nombre", "Hombre")
                     .containsOnly(aragorn, boromir);
```

### 8.5. Extraer valores antes de comprobar (`extracting`)

Muchas veces no quieres construir objetos completos para compararlos; solo te interesa un campo. `extracting` lo resuelve:

```java
// Un valor por elemento
assertThat(comunidad).extracting(Personaje::getNombre)
                     .contains("Frodo", "Gandalf")
                     .doesNotContain("Sauron");

// Varios valores por elemento, agrupados en "tuplas"
assertThat(comunidad).extracting("nombre", "edad", "raza.nombre")
                     .contains(tuple("Sam", 38, "Hobbit"),
                               tuple("Legolas", 1000, "Elfo"));
```

También existen `flatExtracting` / `flatMap` para "aplanar" listas de listas (equivalente a `flatMap` en programación funcional) y `map` como alias de `extracting`.

### 8.6. Comparar con un criterio propio

`usingElementComparator` cambia la forma de comparar elementos (en lugar de usar `equals`):

```java
assertThat(comunidad)
    .usingElementComparator((a, b) -> a.getRaza().compareTo(b.getRaza()))
    .contains(sauron); // se compara solo por raza
```

---

## 9. Aserciones sobre excepciones

### 9.1. `assertThatThrownBy`

La forma más directa: ejecuta el código y comprueba la excepción lanzada.

```java
assertThatThrownBy(() -> { throw new IllegalArgumentException("importe incorrecto 123"); })
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessageContaining("importe");
```

> Si el código **no** lanza ninguna excepción, la aserción falla inmediatamente.

### 9.2. `assertThatExceptionOfType`

Sintaxis alternativa que a algunas personas les resulta más natural:

```java
assertThatExceptionOfType(IOException.class)
    .isThrownBy(() -> { throw new IOException("boom!"); })
    .withMessage("boom!")
    .withNoCause();
```

Hay atajos para excepciones comunes: `assertThatNullPointerException`, `assertThatIllegalArgumentException`, `assertThatIllegalStateException`, `assertThatIOException`.

### 9.3. Estilo BDD: `catchThrowable`

Separa el "cuando" (WHEN) del "entonces" (THEN):

```java
// GIVEN
String[] nombres = { "Pier", "Pol", "Jak" };
// WHEN
Throwable thrown = catchThrowable(() -> System.out.println(nombres[9]));
// THEN
assertThat(thrown).isInstanceOf(ArrayIndexOutOfBoundsException.class)
                  .hasMessageContaining("9");
```

`catchThrowableOfType` permite además acceder a campos personalizados de tu excepción.

### 9.4. Comprobar el mensaje

```java
Throwable t = new IllegalArgumentException("wrong amount 123");

assertThat(t).hasMessage("wrong amount 123")
             .hasMessageStartingWith("wrong")
             .hasMessageContaining("amount")
             .hasMessageEndingWith("123")
             .hasMessageMatching("wrong amount .*")   // expresión regular
             .hasMessageNotContaining("right");
```

### 9.5. Comprobar la causa

```java
assertThat(throwable).cause()
                     .hasMessage("boom!")
                     .isInstanceOf(NullPointerException.class);

// También existe rootCause() para la causa raíz
```

### 9.6. Comprobar que NO se lanza excepción

```java
assertThatNoException().isThrownBy(() -> System.out.println("OK"));

// o bien:
assertThatCode(() -> System.out.println("OK")).doesNotThrowAnyException();
```

---

## 10. Mejorar los mensajes de error

### 10.1. Describir la aserción con `as()`

Muy útil, sobre todo con booleanos, donde el mensaje por defecto solo dice "esperaba `true` pero fue `false`".

```java
assertThat(frodo.getEdad()).as("comprobar la edad de %s", frodo.getNombre())
                           .isEqualTo(100);
```

Mensaje de error resultante:

```text
[comprobar la edad de Frodo] expected:<100> but was:<33>
```

> ⚠️ **`as()` debe ir ANTES de la aserción.** Si va después, se ignora (porque la aserción fallida corta la cadena).

### 10.2. Cambiar el mensaje por completo

Con `withFailMessage()` u `overridingErrorMessage()`:

```java
assertThat(frodo.getEdad()).withFailMessage("debería ser %s", sam)
                           .isEqualTo(sam.getEdad());
```

Si construir el mensaje es costoso, usa la variante con `Supplier<String>` (solo se calcula si la aserción falla):

```java
assertThat(jugador.esNovato())
    .withFailMessage(() -> "Se esperaba que el jugador fuera novato.")
    .isTrue();
```

---

## 11. Errores típicos que debes evitar

### 11.1. Olvidar la aserción final

Este es **el error más común y más peligroso**: el test pasa siempre porque no comprueba nada.

```java
// ❌ MAL: no comprueba nada, el test pasa igualmente
assertThat(actual.equals(esperado));
assertThat(1 == 2);

// ✅ BIEN
assertThat(actual).isEqualTo(esperado);
assertThat(1).isEqualTo(2);           // falla, como debe
assertThat(1 == 2).isTrue();          // válido, aunque menos elegante
```

Herramientas como SpotBugs o SonarQube (regla S2970) pueden detectar este error.

### 11.2. Poner `as()` o `withFailMessage()` después de la aserción

```java
// ❌ MAL: se ignoran
assertThat(actual).isEqualTo(esperado).as("descripción");
assertThat(actual).isEqualTo(esperado).withFailMessage("mensaje");

// ✅ BIEN: antes de la aserción
assertThat(actual).as("descripción").isEqualTo(esperado);
assertThat(actual).withFailMessage("mensaje").isEqualTo(esperado);
```

### 11.3. Configurar un comparador después de la aserción

```java
// ❌ MAL: el comparador no se usa
assertThat(actual).isEqualTo(esperado).usingComparator(new MiComparador());

// ✅ BIEN
assertThat(actual).usingComparator(new MiComparador()).isEqualTo(esperado);
```

**Regla de oro:** todo lo que *configura* la aserción (`as`, `withFailMessage`, `usingComparator`...) va **antes** de la aserción que comprueba.

---

## 12. Soft assertions (aserciones "suaves")

Normalmente, un test se detiene en la **primera** aserción que falla. Con las *soft assertions*, AssertJ **recoge todos los errores** y los muestra juntos al final. Es muy útil en tests largos (por ejemplo, de extremo a extremo), porque te permite corregir varios fallos de una sola vez.

```java
@Test
void ejemplo_soft_assertions() {
  SoftAssertions softly = new SoftAssertions();

  softly.assertThat("George Martin").as("grandes autores").isEqualTo("JRR Tolkien");
  softly.assertThat(42).as("respuesta a todo").isGreaterThan(100);
  softly.assertThat("Gandalf").isEqualTo("Sauron");

  // ¡No olvides esta llamada! Si no, no se reporta ningún error.
  softly.assertAll();
}
```

Salida (resumida):

```text
Multiple Failures (3 failures)
-- failure 1 --
[grandes autores]
...
-- failure 2 --
[respuesta a todo]
...
-- failure 3 --
...
```

### Formas de no tener que llamar a `assertAll()` a mano

| Enfoque | Descripción |
| --- | --- |
| **Regla de JUnit 4** | `JUnitSoftAssertions` llama a `assertAll()` al terminar cada test. |
| **Extensión de JUnit 5** | `SoftAssertionsExtension` inyecta el objeto y llama a `assertAll()` por ti. |
| **`AutoCloseableSoftAssertions`** | Usable con `try-with-resources`. |
| **`assertSoftly`** | Método estático que recibe una lambda. |

Ejemplo con `assertSoftly`:

```java
SoftAssertions.assertSoftly(softly -> {
  softly.assertThat("Gandalf").isEqualTo("Gandalf");
  softly.assertThat(42).isGreaterThan(10);
});
```

Existe también una versión **BDD** (`BDDSoftAssertions`) donde `assertThat` se sustituye por `then`.

---

## 13. Comparación recursiva campo a campo

Por defecto, `isEqualTo` usa el método `equals` del objeto. Si tu clase **no** lo redefine, dos objetos con los mismos datos se consideran distintos (comparan referencias). La comparación recursiva compara **campo por campo**, incluidos los objetos anidados:

```java
Persona sherlock  = new Persona("Sherlock", 1.80);
Persona sherlock2 = new Persona("Sherlock", 1.80);

// ✅ Pasa: los datos de ambos objetos son iguales
assertThat(sherlock).usingRecursiveComparison()
                    .isEqualTo(sherlock2);

// ❌ Falla: Persona no redefine equals y se comparan referencias
assertThat(sherlock).isEqualTo(sherlock2);
```

### Opciones útiles

```java
// Ignorar campos (útil para ids, fechas generadas, etc.)
assertThat(actual).usingRecursiveComparison()
                  .ignoringFields("id", "home.address.street")
                  .isEqualTo(esperado);

// Ignorar campos nulos del objeto esperado
assertThat(actual).usingRecursiveComparison()
                  .ignoringExpectedNullFields()
                  .isEqualTo(esperado);

// Exigir que los tipos sean compatibles
assertThat(actual).usingRecursiveComparison()
                  .withStrictTypeChecking()
                  .isEqualTo(esperado);

// Comparar un tipo concreto con un criterio propio
assertThat(actual).usingRecursiveComparison()
                  .withEqualsForType((a, b) -> Math.abs(a - b) <= 0.5, Double.class)
                  .isEqualTo(esperado);
```

Notas importantes:

- La comparación **no es simétrica**: se limita a los campos del objeto *actual*.
- Las opciones de configuración van **antes** de `isEqualTo`.
- Desde la versión 3.17.0 no se usan los métodos `equals` redefinidos de las clases; para recuperar ese comportamiento existe `usingOverriddenEquals()`.

---

## 14. Otros puntos de entrada: `WithAssertions` y estilo BDD

### `WithAssertions`

Si tu clase de test implementa la interfaz `WithAssertions`, tienes `assertThat` disponible **sin import estático**:

```java
import org.assertj.core.api.WithAssertions;

class MiTest implements WithAssertions {

  @Test
  void ejemplo() {
    assertThat(frodo.getEdad()).isEqualTo(33);
  }
}
```

### `BDDAssertions`

Para quien prefiere el estilo BDD (*Given / When / Then*), `assertThat` se sustituye por `then`:

```java
import static org.assertj.core.api.BDDAssertions.then;

class MiTestBdd {

  @Test
  void ejemplo() {
    then(frodo.getEdad()).isEqualTo(33);
    then(frodo.getNombre()).isEqualTo("Frodo");
  }
}
```

---

## 15. Configuración del IDE

Para que `assertThat` aparezca al escribir `as` y lanzar el autocompletado:

- **IntelliJ IDEA**: no hace falta configurar nada. Escribe `asser` e invoca el autocompletado (Ctrl+Espacio) dos veces.
- **Eclipse**: `Window > Preferences > Java > Editor > Content Assist > Favorites > New Type…`, introduce `org.assertj.core.api.Assertions` y acepta. Comprueba que aparece `org.assertj.core.api.Assertions.*` en la lista de favoritos.

---

## 16. Recursos y dónde pedir ayuda

| Recurso | Enlace |
| --- | --- |
| Documentación oficial | <https://assertj.github.io/doc/> |
| Repositorio en GitHub | <https://github.com/assertj/assertj> |
| Javadoc de AssertJ Core | <https://www.javadoc.io/doc/org.assertj/assertj-core/latest/index.html> |
| Ejemplos de código | <https://github.com/assertj/assertj-examples> |
| Preguntas (Stack Overflow) | <https://stackoverflow.com/questions/tagged/assertj> |
| Reportar problemas / proponer aserciones | <https://github.com/assertj/assertj/issues> |
| Cómo contribuir | <https://github.com/assertj/assertj/blob/main/CONTRIBUTING.md> |

### Ruta de aprendizaje sugerida

1. Añade la dependencia y escribe tu primer test con `assertThat(...)`.
2. Practica con `String`, números y booleanos.
3. Aprende las aserciones de colecciones (`contains`, `containsExactly`, `extracting`, `filteredOn`).
4. Aprende a comprobar excepciones (`assertThatThrownBy`).
5. Usa `as()` para mejorar tus mensajes de error.
6. Descubre las *soft assertions* y la comparación recursiva cuando las necesites.
7. Consulta el Javadoc cuando quieras saber qué hace una aserción concreta.

---

## 17. Resumen rápido (chuleta)

```java
import static org.assertj.core.api.Assertions.*;

// Básicas
assertThat(x).isEqualTo(y);
assertThat(x).isNotNull();
assertThat(x).isInstanceOf(Clase.class);

// String
assertThat(s).startsWith("a").contains("b").endsWith("c");

// Números
assertThat(n).isPositive().isBetween(1, 10);
assertThat(d).isCloseTo(3.14, within(0.01));

// Colecciones
assertThat(lista).hasSize(3).contains(a, b).doesNotContain(c);
assertThat(lista).containsExactly(a, b, c);
assertThat(lista).extracting(Persona::getNombre).contains("Ana");
assertThat(lista).filteredOn(p -> p.getEdad() > 18).hasSize(2);

// Excepciones
assertThatThrownBy(() -> codigo()).isInstanceOf(X.class).hasMessageContaining("...");
assertThatNoException().isThrownBy(() -> codigo());

// Descripción del test (¡ANTES de la aserción!)
assertThat(x).as("descripción").isEqualTo(y);

// Comparación recursiva
assertThat(a).usingRecursiveComparison().ignoringFields("id").isEqualTo(b);

// Soft assertions
SoftAssertions.assertSoftly(softly -> {
  softly.assertThat(a).isEqualTo(1);
  softly.assertThat(b).isEqualTo(2);
});
```

---

*Documento elaborado con fines didácticos a partir de la documentación pública de AssertJ. Para información completa y actualizada, consulta siempre la [documentación oficial](https://assertj.github.io/doc/) y el [repositorio](https://github.com/assertj/assertj).*
