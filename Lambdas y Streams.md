# Lambdas y Streams en Java 21+

Guía de estudio. Versión de referencia: **Java 21 (LTS)**, con notas sobre lo que aportan versiones posteriores (22 a 25).

> Este documento resume y reorganiza el contenido de seis fuentes (listadas al final) y lo amplía con material actual que esas fuentes no recogen: `record`, `Stream.toList()`, `SequencedCollection`, hilos virtuales, *pattern matching*, `Gatherers`, errores típicos y ejercicios. Varias de las fuentes están escritas para Java 8, así que aquí se han actualizado los ejemplos al estilo moderno.

---

## Índice

1. [Preparación: imports y dominio de ejemplo](#1-preparación-imports-y-dominio-de-ejemplo)
2. [Por qué existen las lambdas](#2-por-qué-existen-las-lambdas)
3. [Sintaxis de una lambda](#3-sintaxis-de-una-lambda)
4. [Interfaces funcionales](#4-interfaces-funcionales)
5. [Inferencia de tipos (target typing)](#5-inferencia-de-tipos-target-typing)
6. [Captura de variables y `this`](#6-captura-de-variables-y-this)
7. [Referencias a métodos](#7-referencias-a-métodos)
8. [Lambdas frente a clases anónimas](#8-lambdas-frente-a-clases-anónimas)
9. [Buenas prácticas con lambdas](#9-buenas-prácticas-con-lambdas)
10. [Excepciones comprobadas dentro de lambdas](#10-excepciones-comprobadas-dentro-de-lambdas)
11. [Streams: el modelo mental](#11-streams-el-modelo-mental)
12. [Crear streams](#12-crear-streams)
13. [Operaciones intermedias](#13-operaciones-intermedias)
14. [Operaciones terminales](#14-operaciones-terminales)
15. [Collectors](#15-collectors)
16. [Streams de primitivos](#16-streams-de-primitivos)
17. [Optional](#17-optional)
18. [Streams paralelos](#18-streams-paralelos)
19. [Rendimiento: ¿stream o bucle?](#19-rendimiento-stream-o-bucle)
20. [Novedades de Java 21 en adelante](#20-novedades-de-java-21-en-adelante)
21. [Errores típicos](#21-errores-típicos)
22. [Aclaraciones sobre las fuentes](#22-aclaraciones-sobre-las-fuentes)
23. [Ejercicios](#23-ejercicios)
24. [Soluciones](#24-soluciones)
25. [Chuleta final](#25-chuleta-final)
26. [Fuentes](#26-fuentes)

---

## 1. Preparación: imports y dominio de ejemplo

Todos los ejemplos asumen estos imports:

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
```

Y este dominio, que se reutiliza en todo el documento:

```java
record Empleado(String nombre, String departamento, double salario, int edad) {}

List<Empleado> empleados = List.of(
    new Empleado("Lucía",  "Ingeniería", 48_000, 34),
    new Empleado("Marcos", "Ingeniería", 39_500, 28),
    new Empleado("Elena",  "Ventas",     31_000, 45),
    new Empleado("Pablo",  "Ventas",     27_500, 23),
    new Empleado("Carmen", "RR. HH.",    35_000, 39)
);
```

Un `record` genera automáticamente constructor, `equals`, `hashCode`, `toString` y los accesores (`nombre()`, `salario()`...; sin prefijo `get`). Es ideal para datos inmutables y encaja muy bien con streams.

---

## 2. Por qué existen las lambdas

### Idea central: pasar comportamiento como si fuera un dato

Antes de Java 8 solo se podían pasar valores y objetos a un método. Las lambdas permiten pasar **un trozo de código** (una función) como parámetro, devolverlo desde un método o guardarlo en una variable. En programación funcional esto se llama tener **funciones como ciudadanos de primera clase**.

### La evolución de un problema real

Queremos filtrar empleados con distintos criterios.

**Intento 1: un método por criterio.** Se multiplican los métodos y el código se repite.

```java
static List<Empleado> mayoresDe(List<Empleado> lista, int edad) { /* bucle + if */ }
static List<Empleado> deDepartamento(List<Empleado> lista, String dep) { /* bucle + if */ }
// y así con cada combinación nueva...
```

**Intento 2: parámetros "comodín".** Un método con muchos parámetros y banderas booleanas. Es ilegible y frágil: cada criterio nuevo obliga a tocar la firma.

**Intento 3: patrón Estrategia con una interfaz.** El criterio pasa a ser un objeto.

```java
interface Criterio { boolean cumple(Empleado e); }

static List<Empleado> filtrar(List<Empleado> lista, Criterio criterio) {
    List<Empleado> resultado = new ArrayList<>();
    for (Empleado e : lista) {
        if (criterio.cumple(e)) resultado.add(e);
    }
    return resultado;
}
```

El problema: hay que crear una clase por cada criterio.

**Intento 4: clase anónima.** Evita crear clases con nombre, pero sigue siendo verboso:

```java
filtrar(empleados, new Criterio() {
    @Override
    public boolean cumple(Empleado e) {
        return e.edad() > 30;
    }
});
```

**Intento 5: lambda.** Solo queda lo esencial.

```java
filtrar(empleados, e -> e.edad() > 30);
```

**Intento 6: interfaz estándar del JDK.** No hace falta definir `Criterio`: ya existe `Predicate<T>`.

```java
static List<Empleado> filtrar(List<Empleado> lista, Predicate<Empleado> criterio) { /* ... */ }

filtrar(empleados, e -> e.edad() > 30);
```

**Intento 7: streams.** Ni siquiera hace falta escribir `filtrar`:

```java
empleados.stream().filter(e -> e.edad() > 30).toList();
```

Moraleja: una lambda **es una forma compacta de escribir la implementación de una interfaz que tiene un único método abstracto**.

---

## 3. Sintaxis de una lambda

Una lambda tiene tres partes:

```
(parámetros)  ->  cuerpo
```

1. **Parámetros**, entre paréntesis y separados por comas.
2. **Operador flecha** `->` (sin espacio entre `-` y `>`).
3. **Cuerpo**: una única expresión o un bloque entre llaves.

### Variantes habituales

| Caso | Ejemplo |
|---|---|
| Sin parámetros (paréntesis obligatorios) | `() -> System.out.println("hola")` |
| Un parámetro, sin tipo (paréntesis opcionales) | `n -> n * 2` |
| Un parámetro con tipo (paréntesis obligatorios) | `(int n) -> n * 2` |
| Varios parámetros | `(a, b) -> a + b` |
| Cuerpo de expresión (sin `return`, sin `;`) | `(a, b) -> a + b` |
| Cuerpo de bloque (llaves, `;` y `return` si devuelve algo) | `(a, b) -> { int s = a + b; return s; }` |
| Con `var` (desde Java 11) | `(var a, var b) -> a + b` |

### Reglas que conviene memorizar

- **Cuerpo de expresión:** se evalúa y su valor se devuelve automáticamente. No lleva `return` ni llaves.
- **Cuerpo de bloque:** necesita llaves, cada sentencia termina en `;` y, si la lambda devuelve algo, hay que escribir `return`.
- **Tipos de los parámetros:** normalmente se omiten porque el compilador los infiere. Si pones el tipo en uno, ponlo en todos. No se puede mezclar `(int a, b)`. Lo mismo con `var`.
- **Compatibilidad con `void`:** si el cuerpo es una única llamada a método, la lambda sirve tanto para una interfaz que devuelve un valor como para una que devuelve `void`. Por eso `s -> lista.add(s)` encaja en un `Consumer<String>` (se ignora el `boolean`) y también en un `Predicate<String>`.
- **Una lambda no tiene tipo propio.** Su tipo lo determina el contexto donde se usa (ver sección 5). Por eso esto no compila:

```java
var f = x -> x * 2;   // ERROR: no hay tipo destino del que inferir nada
```

### Ejemplos comentados

```java
Runnable saludo = () -> System.out.println("Hola");        // sin parámetros, sin retorno
Function<String, Integer> longitud = s -> s.length();      // un parámetro, devuelve valor
BinaryOperator<Integer> suma = (a, b) -> a + b;            // dos parámetros
Supplier<List<String>> nuevaLista = () -> new ArrayList<>();  // sin parámetros, devuelve objeto
Consumer<String> traza = s -> {                            // bloque con varias sentencias
    System.out.println("Procesando " + s);
    System.out.println("Longitud: " + s.length());
};
```

---

## 4. Interfaces funcionales

### Definición

Una **interfaz funcional** es una interfaz con **exactamente un método abstracto**. Esa es la que una lambda implementa. Puede tener además:

- Métodos `default` y métodos `static`.
- Métodos abstractos que coincidan con los de `java.lang.Object` (`equals`, `toString`...), que no cuentan.

### La anotación `@FunctionalInterface`

Es opcional, pero **ponla siempre en las tuyas**: si alguien añade por error un segundo método abstracto, el compilador falla en vez de romper en silencio a quien use la lambda.

```java
@FunctionalInterface
interface Validador<T> {
    boolean validar(T valor);                       // único método abstracto

    default Validador<T> y(Validador<T> otro) {     // los default no cuentan
        return v -> this.validar(v) && otro.validar(v);
    }

    static <T> Validador<T> siempreValido() {       // los static tampoco
        return v -> true;
    }
}
```

### Las cuatro familias básicas (paquete `java.util.function`)

| Familia | Interfaz | Método | Firma | Para qué se usa |
|---|---|---|---|---|
| Predicado | `Predicate<T>` | `test` | `T -> boolean` | Comprobar una condición |
| Proveedor | `Supplier<T>` | `get` | `() -> T` | Producir o crear un valor |
| Consumidor | `Consumer<T>` | `accept` | `T -> void` | Hacer algo con un valor (efectos laterales) |
| Función | `Function<T, R>` | `apply` | `T -> R` | Transformar un valor en otro |

### Variantes derivadas

| Interfaz | Firma | Notas |
|---|---|---|
| `BiPredicate<T, U>` | `(T, U) -> boolean` | Predicado de dos argumentos |
| `BiConsumer<T, U>` | `(T, U) -> void` | Típico en `Map.forEach` |
| `BiFunction<T, U, R>` | `(T, U) -> R` | Función de dos argumentos |
| `UnaryOperator<T>` | `T -> T` | `Function` donde entrada y salida son del mismo tipo |
| `BinaryOperator<T>` | `(T, T) -> T` | `BiFunction` con los tres tipos iguales. Típico en `reduce` |
| `Runnable` | `() -> void` | Tarea sin entrada ni salida (hilos) |
| `Callable<V>` | `() -> V` | Como `Supplier`, pero **puede lanzar excepciones** |
| `Comparator<T>` | `(T, T) -> int` | Ordenación |

### Variantes para primitivos

Existen versiones que evitan el *autoboxing* (convertir `int` en `Integer` y viceversa), que genera objetos innecesarios:

`IntPredicate`, `IntFunction<R>`, `IntUnaryOperator`, `IntBinaryOperator`, `IntSupplier`, `IntConsumer`, `ToIntFunction<T>`, `ToDoubleFunction<T>`, `BooleanSupplier`... y lo mismo para `Long` y `Double`.

Regla práctica: si trabajas con números en un camino caliente, prefiere `IntStream` y sus interfaces primitivas.

### Composición: lo que hay dentro de las interfaces estándar

Las interfaces estándar traen métodos `default` para combinar funciones:

```java
Predicate<Empleado> esJoven    = e -> e.edad() < 30;
Predicate<Empleado> esIngeniero = e -> e.departamento().equals("Ingeniería");

Predicate<Empleado> jovenIngeniero = esJoven.and(esIngeniero);
Predicate<Empleado> noEsJoven      = esJoven.negate();
Predicate<Empleado> cualquiera     = esJoven.or(esIngeniero);

// Predicate.not (Java 11) combina bien con referencias a métodos
List<String> noVacias = textos.stream().filter(Predicate.not(String::isBlank)).toList();

Function<String, String> limpiar = String::trim;
Function<String, Integer> longitud = String::length;
Function<String, Integer> limpiarYMedir = limpiar.andThen(longitud);   // primero limpiar, luego medir
Function<String, Integer> medirDespuesDe = longitud.compose(limpiar);  // equivalente: compose aplica el argumento primero

Consumer<String> imprimir = System.out::println;
Consumer<String> imprimirDosVeces = imprimir.andThen(imprimir);
```

### Conflictos con métodos `default`

Si una clase implementa dos interfaces que definen el mismo método `default`, está obligada a sobrescribirlo. Puede delegar en una de ellas con `InterfazA.super.metodo()`. Orden de resolución: primero la clase concreta, después las subinterfaces y, si sigue habiendo conflicto, elección explícita con `X.super.m()`.

---

## 5. Inferencia de tipos (target typing)

Una lambda **no tiene tipo por sí misma**: el compilador lo deduce del contexto, que se llama *tipo destino* (*target type*). Esto funciona en:

- Declaraciones de variables y asignaciones
- Argumentos de métodos y constructores
- Sentencias `return`
- Inicializadores de arrays
- Expresiones condicionales `?:` y conversiones (*casts*)
- Cuerpos de otras lambdas

```java
Predicate<String> p = s -> s.isEmpty();              // contexto de asignación: s es String
List<Empleado> r = filtrar(empleados, e -> e.edad() > 30);  // contexto de argumento: e es Empleado
```

La misma lambda puede ser de tipos distintos según el destino:

```java
Callable<Integer> c = () -> 42;
Supplier<Integer> s = () -> 42;
```

### Ambigüedad al sobrecargar métodos

Si dos métodos con el mismo nombre reciben interfaces funcionales distintas pero compatibles con la misma lambda, el compilador no puede elegir:

```java
void ejecutar(Callable<String> c) { /* ... */ }
void ejecutar(Supplier<String> s) { /* ... */ }

ejecutar(() -> "hola");   // ERROR: referencia ambigua
```

Soluciones, de mejor a peor:

1. **Dar nombres distintos** a los métodos (`ejecutarCallable`, `ejecutarSupplier`). Es la recomendación habitual.
2. Hacer un *cast* explícito: `ejecutar((Supplier<String>) () -> "hola")`. Funciona, pero afea el código.

---

## 6. Captura de variables y `this`

### Variables locales: deben ser *effectively final*

Una lambda puede leer variables locales del método donde se define, pero solo si son `final` o **efectivamente finales** (se asignan una sola vez, aunque no lleven la palabra `final`).

```java
int umbral = 30;                                 // efectivamente final: nunca se reasigna
empleados.stream().filter(e -> e.edad() > umbral);   // OK
```

```java
int contador = 0;
empleados.forEach(e -> contador++);              // ERROR de compilación
// "local variables referenced from a lambda expression must be final or effectively final"
```

Piensa que la lambda **copia el valor**, no la variable. Esto evita problemas de visibilidad y condiciones de carrera cuando la lambda se ejecuta en otro hilo.

Y si reasignas la variable en cualquier otro punto del método, deja de ser efectivamente final y el error aparece donde se usa en la lambda.

### El "truco" del array o del `AtomicInteger`: cuidado

Un array de una posición o un `AtomicInteger` compilan, porque la referencia no cambia, pero el **objeto sí se muta**:

```java
int[] total = new int[1];
empleados.forEach(e -> total[0] += e.edad());   // compila, pero es mala idea
```

Es legal pero frágil (y no es seguro en streams paralelos). Casi siempre hay una alternativa mejor:

```java
int total = empleados.stream().mapToInt(Empleado::edad).sum();
```

### Campos de instancia y estáticos

Los **campos** sí se pueden leer y modificar desde una lambda, porque se accede a ellos a través de `this` o de la clase. Aun así, mutar estado compartido desde una lambda es una fuente de errores: evítalo.

### `this` dentro de una lambda

Una lambda **no crea un ámbito nuevo**. `this` se refiere a la instancia de la clase que la contiene, no a la propia lambda (a diferencia de una clase anónima, donde `this` es la clase anónima). Tampoco se pueden declarar variables locales con el mismo nombre que otra del ámbito exterior:

```java
void metodo(int x) {
    Consumer<Integer> c = x -> { };   // ERROR: x ya está definida en el ámbito
}
```

---

## 7. Referencias a métodos

Cuando una lambda **solo llama a un método que ya existe**, se puede escribir de forma más corta con `::`.

```java
s -> s.toUpperCase()    //  se convierte en  →   String::toUpperCase
x -> System.out.println(x)   //  →   System.out::println
```

### Los cuatro tipos

| Tipo | Sintaxis | Lambda equivalente | Ejemplo |
|---|---|---|---|
| Método estático | `Clase::metodoEstatico` | `x -> Clase.metodoEstatico(x)` | `Integer::parseInt` |
| Método de instancia de un objeto concreto | `objeto::metodo` | `x -> objeto.metodo(x)` | `System.out::println` |
| Método de instancia de un tipo (el primer parámetro es el receptor) | `Tipo::metodo` | `x -> x.metodo()` | `String::toUpperCase` |
| Constructor | `Clase::new` | `() -> new Clase()` | `ArrayList::new` |

```java
Function<String, Integer> parsear  = Integer::parseInt;       // estático
Consumer<String> imprimir          = System.out::println;     // instancia de objeto concreto
Function<String, String> mayusc    = String::toUpperCase;     // instancia de tipo
Supplier<List<String>> nuevaLista  = ArrayList::new;          // constructor sin argumentos
Function<Integer, List<String>> conCapacidad = ArrayList::new;   // constructor con un argumento
BiFunction<String, Integer, Empleado> raro = (n, e) -> new Empleado(n, "X", 0, e);
IntFunction<int[]> crearArray      = int[]::new;              // constructor de array
```

Con constructores y `Function`/`BiFunction`, la interfaz funcional elegida determina qué sobrecarga del constructor se invoca (la de un argumento, la de dos...).

También son válidas `this::metodo` y `super::metodo` dentro de una clase.

### Cuándo preferir cada forma

- Si la lambda solo reenvía los parámetros a un método, usa la referencia: es más legible.
- Si hay lógica adicional, usa la lambda.
- Hay un caso ambiguo famoso: `Integer::toString` no compila como `Function<Integer, String>` porque existe tanto `toString()` (de instancia) como `Integer.toString(int)` (estático). En estos casos, usa la lambda.

---

## 8. Lambdas frente a clases anónimas

| Aspecto | Clase anónima | Lambda |
|---|---|---|
| Sintaxis | Verbosa | Compacta |
| Sirve para | Interfaces o clases con cualquier número de métodos | **Solo** interfaces funcionales |
| `this` | Se refiere a la propia clase anónima | Se refiere a la clase que la contiene |
| Ámbito | Crea uno nuevo (puede sombrear variables) | No crea ámbito nuevo (no sombrea) |
| Fichero `.class` | Uno por cada clase anónima | No genera `.class` propio (se resuelve en tiempo de ejecución con `invokedynamic`) |
| Estado propio | Puede tener campos | No puede |

```java
// Con una clase anónima, this.valor es el campo de la clase anónima
Foo fooAnonimo = new Foo() {
    String valor = "campo de la anónima";
    public String metodo(String s) { return this.valor; }
};

// Con una lambda, this.valor es el campo de la clase contenedora
Foo fooLambda = s -> this.valor;
```

Moraleja: **una lambda no es "una clase anónima abreviada"**. Se comporta distinto en ámbito y en `this`.

Que las lambdas no generen una clase por cada una se debe a que se enlazan en tiempo de ejecución mediante la instrucción de bytecode `invokedynamic`. Es un detalle de implementación que te conviene conocer, pero no necesitas manejarlo.

---

## 9. Buenas prácticas con lambdas

1. **Reutiliza las interfaces estándar.** Antes de crear una interfaz funcional propia, mira en `java.util.function`. Si tu interfaz es "un argumento, un resultado", ya existe: `Function`.
2. **Anota las interfaces propias con `@FunctionalInterface`.**
3. **Mantén las lambdas cortas.** Idealmente una línea. Si pasa de unas tres líneas, extrae un método y usa una referencia a método. Pero no lo conviertas en dogma: una lambda de dos o tres líneas legible no necesita extracción.

   ```java
   // Mejor
   .map(this::calcularPrecioFinal)
   // Peor
   .map(p -> { /* 15 líneas */ })
   ```

4. **Omite lo que el compilador infiere:** tipos de parámetros, paréntesis de un solo parámetro, llaves y `return` en cuerpos de una expresión.
5. **Prefiere referencias a métodos** cuando solo hay reenvío.
6. **Evita efectos laterales.** Idealmente, una lambda no modifica estado externo. Las lambdas "puras" (mismo resultado para la misma entrada y sin efectos) son más fáciles de probar y paralelizar.
7. **No muten objetos capturados.** Que una variable sea efectivamente final no impide que el objeto al que apunta cambie.
8. **Evita sobrecargar métodos que reciben interfaces funcionales distintas** (ver sección 5).
9. **No abuses de los métodos `default`** en interfaces funcionales. Su razón de ser original es evolucionar APIs sin romper compatibilidad.
10. **Prueba las lambdas complejas por separado.** Si hay lógica de negocio, que viva en un método que puedas testear sin montar un stream.
11. **Documenta los efectos laterales** de un `Consumer` que haga entrada/salida.
12. **No reescribas todo el código antiguo** solo por tener lambdas. Úsalas en código nuevo y al tocar código existente.
13. **No serialices lambdas.** Técnicamente es posible, pero es frágil y está desaconsejado.

---

## 10. Excepciones comprobadas dentro de lambdas

Las interfaces funcionales estándar (`Function`, `Consumer`...) **no declaran excepciones comprobadas** (*checked*). Por eso esto no compila:

```java
rutas.stream()
     .map(r -> Files.readString(r))   // ERROR: IOException no capturada
     .toList();
```

Estrategias, de más simple a más elaborada:

**1. `try/catch` dentro de la lambda, relanzando como no comprobada.**

```java
.map(r -> {
    try {
        return Files.readString(r);
    } catch (IOException e) {
        throw new UncheckedIOException(e);   // conserva la causa original
    }
})
```

**2. Un envoltorio reutilizable.** Para no repetir el `try/catch`:

```java
@FunctionalInterface
interface FuncionConExcepcion<T, R> {
    R aplicar(T t) throws Exception;
}

static <T, R> Function<T, R> sinChecked(FuncionConExcepcion<T, R> f) {
    return t -> {
        try {
            return f.aplicar(t);
        } catch (RuntimeException e) {
            throw e;
        } catch (Exception e) {
            throw new RuntimeException(e);   // no pierdas la causa
        }
    };
}

// Uso
rutas.stream().map(sinChecked(r -> Files.readString(r))).toList();
```

**3. Devolver un resultado que modele el fallo** (por ejemplo `Optional` o un tipo `Either`/`Result` propio o de una librería como Vavr), y filtrar los fallos después. Útil cuando quieres que el *pipeline* siga aunque algún elemento falle.

**4. No usar stream** si el manejo de errores es el protagonista: un bucle `for` con `try/catch` puede ser mucho más claro.

---

## 11. Streams: el modelo mental

### Qué es (y qué no es) un `Stream`

Un `Stream<T>` es una **secuencia de elementos sobre la que se aplican operaciones de forma declarativa**. Piensa en una cadena de montaje: los elementos entran por un extremo, pasan por estaciones que los filtran o transforman y salen por el otro.

- **No es una colección:** no almacena elementos. Los toma de una fuente (lista, array, fichero, generador...).
- **No modifica la fuente.** Las operaciones producen resultados nuevos.
- **Se consume una sola vez.** Tras una operación terminal, el stream queda cerrado.
- **Puede ser infinito** (por ejemplo `Stream.iterate`), siempre que lo limites.

### Estructura de un *pipeline*

```
FUENTE  →  operación intermedia  →  operación intermedia  →  OPERACIÓN TERMINAL
```

```java
List<String> resultado = empleados.stream()                        // 1. fuente
    .filter(e -> e.salario() > 30_000)                             // 2. intermedia
    .sorted(Comparator.comparingDouble(Empleado::salario).reversed())  // 2. intermedia
    .map(Empleado::nombre)                                         // 2. intermedia
    .toList();                                                     // 3. terminal
```

- Las **intermedias** devuelven otro `Stream` y son **perezosas** (*lazy*): no hacen nada hasta que hay una terminal.
- La **terminal** dispara la ejecución y produce un resultado (o un efecto) que no es un `Stream`.

### Evaluación perezosa y procesamiento "elemento a elemento"

Sin terminal no se ejecuta nada. Y los elementos **no pasan de operación en operación en bloque**, sino uno a uno por todo el pipeline:

```java
var resultado = Stream.of("uno", "dos", "tres", "cuatro")
    .filter(s -> { System.out.println("filter: " + s); return s.length() > 3; })
    .map(s -> { System.out.println("map: " + s); return s.toUpperCase(); })
    .findFirst();
```

Salida:

```
filter: uno
filter: dos
filter: tres
map: tres
```

Fíjate: se detiene en cuanto encuentra el primero (**cortocircuito**) y "cuatro" ni se mira. Esta es una de las razones por las que los streams pueden ser eficientes.

### Operaciones sin estado y con estado

- **Sin estado** (`filter`, `map`, `flatMap`...): procesan cada elemento sin necesitar ver los demás.
- **Con estado** (`sorted`, `distinct`, `limit`, `skip`...): necesitan recordar o ver otros elementos. `sorted` tiene que recibir todos los elementos antes de emitir el primero.

Consejo: coloca los `filter` lo antes posible para reducir el trabajo de las operaciones posteriores, sobre todo las costosas o con estado.

### Un stream no se reutiliza

```java
Stream<String> s = Stream.of("a", "b");
s.forEach(System.out::println);
s.forEach(System.out::println);   // IllegalStateException: stream has already been operated upon or closed
```

Si necesitas recorrer los datos varias veces, crea un stream nuevo cada vez a partir de la fuente (o guarda en una colección, o usa un `Supplier<Stream<T>>`).

---

## 12. Crear streams

```java
// Desde colecciones
empleados.stream();

// Desde arrays
Arrays.stream(new int[] {1, 2, 3});         // IntStream
Stream.of("a", "b", "c");

// Rangos de enteros (más seguros que un for clásico para generar secuencias)
IntStream.range(0, 5);          // 0, 1, 2, 3, 4
IntStream.rangeClosed(1, 5);    // 1, 2, 3, 4, 5

// Generados
Stream.iterate(1, n -> n * 2).limit(8);                   // 1, 2, 4, 8, 16, 32, 64, 128
Stream.iterate(1, n -> n < 200, n -> n * 2);              // igual, con condición de parada (Java 9)
Stream.generate(() -> "x").limit(3);                      // "x", "x", "x"

// Vacío y concatenado
Stream.empty();
Stream.concat(Stream.of(1, 2), Stream.of(3, 4));

// Cadenas
"hola".chars();                                           // IntStream de códigos de carácter
Pattern.compile(",").splitAsStream("a,b,c");

// Ficheros (necesitan cerrarse: usa try-with-resources)
try (Stream<String> lineas = Files.lines(Path.of("datos.txt"))) {
    lineas.filter(l -> !l.isBlank()).forEach(System.out::println);
}

// Optional a stream, y valor posiblemente nulo a stream (Java 9)
optional.stream();
Stream.ofNullable(valorQuizaNulo);

// Mapas: no tienen stream() propio, se parte de sus vistas
mapa.entrySet().stream();
mapa.keySet().stream();
mapa.values().stream();
```

Atención con los **streams infinitos** (`iterate`, `generate`): sin `limit` o `takeWhile`, la operación terminal no termina nunca.

---

## 13. Operaciones intermedias

### `filter`: quedarse con los que cumplen una condición

```java
empleados.stream().filter(e -> e.departamento().equals("Ventas"));
```

### `map`: transformar cada elemento

```java
empleados.stream().map(Empleado::nombre);            // Stream<String>
empleados.stream().map(e -> e.nombre().length());    // Stream<Integer>
```

### `flatMap`: transformar y aplanar

Cuando cada elemento produce **varios** elementos (una colección o un stream) y quieres un solo stream plano:

```java
List<List<Integer>> listas = List.of(List.of(1, 2), List.of(3), List.of(4, 5));

listas.stream().flatMap(List::stream).toList();       // [1, 2, 3, 4, 5]

// Palabras únicas de varias frases
List<String> frases = List.of("hola mundo", "hola java");
frases.stream()
      .flatMap(f -> Arrays.stream(f.split(" ")))
      .distinct()
      .toList();                                       // [hola, mundo, java]
```

Con `map` obtendrías un `Stream<Stream<...>>` o `Stream<List<...>>`; `flatMap` lo aplana.

### `distinct`: eliminar duplicados

Usa `equals`/`hashCode`. Con `record` funciona directamente.

### `sorted`: ordenar

```java
Stream.of("b", "a", "c").sorted();                                // orden natural (Comparable)
Stream.of("b", "a", "c").sorted(Comparator.reverseOrder());

empleados.stream().sorted(Comparator.comparing(Empleado::nombre));
empleados.stream().sorted(Comparator.comparingDouble(Empleado::salario).reversed());

// Varios criterios: primero departamento, luego salario descendente
empleados.stream().sorted(
    Comparator.comparing(Empleado::departamento)
              .thenComparing(Empleado::salario, Comparator.reverseOrder()));
```

Si los elementos no son `Comparable` y no pasas un `Comparator`, obtendrás `ClassCastException` en ejecución.

Para ordenar por varios criterios, **empieza con `comparing...` y encadena `thenComparing`**: la primera comparación tiene prioridad.

### `limit` y `skip`

```java
Stream.iterate(1, n -> n + 1).skip(10).limit(5);    // 11, 12, 13, 14, 15
```

### `takeWhile` y `dropWhile` (Java 9)

Útiles con datos **ordenados**: cogen o descartan mientras se cumple la condición.

```java
Stream.of(1, 2, 3, 10, 4, 5).takeWhile(n -> n < 5).toList();   // [1, 2, 3]
Stream.of(1, 2, 3, 10, 4, 5).dropWhile(n -> n < 5).toList();   // [10, 4, 5]
```

### `peek`: solo para depurar

Ejecuta una acción sobre cada elemento sin consumirlo. Úsalo para inspeccionar el pipeline mientras desarrollas, **no para lógica de negocio**: puede no ejecutarse (por cortocircuito) y su comportamiento con optimizaciones no está garantizado.

### `mapMulti` (Java 16)

Alternativa a `flatMap` cuando cada elemento produce pocos resultados y no quieres crear un stream por elemento:

```java
Stream.of(1, 2, 3)
      .<Integer>mapMulti((n, salida) -> { salida.accept(n); salida.accept(n * 10); })
      .toList();                                   // [1, 10, 2, 20, 3, 30]
```

### Conversión entre tipos de stream

```java
empleados.stream().mapToDouble(Empleado::salario);   // DoubleStream
empleados.stream().mapToInt(Empleado::edad);         // IntStream
IntStream.range(0, 3).boxed();                       // Stream<Integer>
IntStream.range(0, 3).mapToObj(i -> "n" + i);        // Stream<String>
```

---

## 14. Operaciones terminales

### `forEach`

Ejecuta una acción por elemento. No devuelve nada. Úsalo para el "último paso" (imprimir, enviar...), no para acumular resultados en una variable externa (ver sección 21).

```java
empleados.forEach(e -> System.out.println(e.nombre()));          // directamente sobre la colección
empleados.stream().forEachOrdered(System.out::println);          // respeta el orden incluso en paralelo
```

### Recolectar el resultado

```java
List<String> nombres = empleados.stream().map(Empleado::nombre).toList();          // Java 16+
List<String> modificable = empleados.stream().map(Empleado::nombre)
                                    .collect(Collectors.toList());                // ver nota
Set<String> deptos = empleados.stream().map(Empleado::departamento).collect(Collectors.toSet());
String[] array = empleados.stream().map(Empleado::nombre).toArray(String[]::new);
```

> **Nota sobre `toList()`:** `Stream.toList()` devuelve una lista **inmodificable**. Si luego haces `add` o `remove` obtendrás `UnsupportedOperationException`. Si de verdad necesitas una lista modificable, usa `collect(Collectors.toCollection(ArrayList::new))`. (`Collectors.toList()` tampoco promete mutabilidad, aunque hoy devuelva un `ArrayList`.)

### Contar y buscar extremos

```java
long cuantos = empleados.stream().filter(e -> e.edad() > 30).count();     // count devuelve long, no int

Optional<Empleado> masJoven = empleados.stream().min(Comparator.comparingInt(Empleado::edad));
Optional<Empleado> mejorPagado = empleados.stream().max(Comparator.comparingDouble(Empleado::salario));
```

### Comprobaciones (cortocircuito)

```java
empleados.stream().anyMatch(e -> e.salario() > 45_000);   // ¿alguno?
empleados.stream().allMatch(e -> e.edad() >= 18);          // ¿todos?
empleados.stream().noneMatch(e -> e.salario() < 0);        // ¿ninguno?
```

Curiosidad útil: `allMatch` sobre un stream vacío devuelve `true` y `anyMatch` devuelve `false`.

### Buscar un elemento

```java
Optional<Empleado> primero = empleados.stream()
    .filter(e -> e.departamento().equals("Ventas"))
    .findFirst();                          // el primero según el orden
// findAny(): cualquiera; más eficiente en paralelo cuando no importa cuál
```

`findFirst` **no recibe predicado**: se combina con `filter`.

### `reduce`: combinar todos en uno

Tres formas:

```java
// 1. Con valor inicial (identidad): devuelve el tipo directamente
double total = empleados.stream().map(Empleado::salario).reduce(0.0, Double::sum);

// 2. Sin valor inicial: devuelve Optional (puede que no haya elementos)
Optional<Double> maximo = empleados.stream().map(Empleado::salario).reduce(Double::max);

// 3. Con identidad, acumulador y combinador (para cambiar de tipo, sobre todo en paralelo)
int caracteres = Stream.of("a", "bb", "ccc")
    .reduce(0, (acum, s) -> acum + s.length(), Integer::sum);   // 6
```

Para sumas y medias, casi siempre es mejor un stream de primitivos (`mapToDouble(...).sum()`), más claro y más eficiente.

Para que `reduce` funcione bien (sobre todo en paralelo), la operación debe ser **asociativa** y la identidad debe ser neutra (0 para sumar, 1 para multiplicar, `""` para concatenar).

---

## 15. Collectors

`collect(...)` es la terminal más potente. Los `Collectors` ya preparados cubren la mayoría de casos.

### Básicos

```java
Collectors.toList();
Collectors.toSet();
Collectors.toUnmodifiableList();
Collectors.toCollection(TreeSet::new);          // la colección que tú elijas
Collectors.joining(", ", "[", "]");             // une Strings: "[a, b, c]"
Collectors.counting();
Collectors.summingDouble(Empleado::salario);
Collectors.averagingInt(Empleado::edad);
Collectors.summarizingDouble(Empleado::salario);   // count, sum, min, average, max de una vez
```

```java
String lista = empleados.stream().map(Empleado::nombre).collect(Collectors.joining(", "));
// "Lucía, Marcos, Elena, Pablo, Carmen"
```

### `toMap`

```java
Map<String, Double> salarioPorNombre = empleados.stream()
    .collect(Collectors.toMap(Empleado::nombre, Empleado::salario));
```

Si dos elementos producen **la misma clave**, lanza `IllegalStateException`. Para resolver colisiones, añade una función de fusión:

```java
Map<String, Double> masaSalarialPorDepto = empleados.stream()
    .collect(Collectors.toMap(Empleado::departamento, Empleado::salario, Double::sum));
```

### `groupingBy`: agrupar (el "GROUP BY" de SQL)

```java
// Departamento -> lista de empleados
Map<String, List<Empleado>> porDepto = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento));

// Departamento -> número de empleados
Map<String, Long> cuantosPorDepto = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento, Collectors.counting()));

// Departamento -> salario medio
Map<String, Double> mediaPorDepto = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento, Collectors.averagingDouble(Empleado::salario)));

// Departamento -> solo los nombres
Map<String, List<String>> nombresPorDepto = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento,
             Collectors.mapping(Empleado::nombre, Collectors.toList())));
```

El segundo argumento es un **collector "descendente"** que se aplica a cada grupo. Así se pueden combinar. También puedes elegir el tipo de mapa, por ejemplo `TreeMap::new`, para tener las claves ordenadas.

Contar repeticiones de cada elemento es el patrón clásico:

```java
Map<String, Long> frecuencias = palabras.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

### `partitioningBy`: dividir en dos grupos

```java
Map<Boolean, List<Empleado>> partes = empleados.stream()
    .collect(Collectors.partitioningBy(e -> e.salario() >= 35_000));

partes.get(true);    // los que cumplen
partes.get(false);   // los que no
```

### `teeing` (Java 12): dos collectors a la vez

Resuelve en una sola pasada lo que haría falta recorrer dos veces:

```java
var resumen = empleados.stream().collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.averagingDouble(Empleado::salario),
    (n, media) -> n + " empleados, media " + media));
```

### Otros útiles

- `Collectors.collectingAndThen(downstream, finisher)`: aplica una transformación final (por ejemplo, hacer inmodificable el resultado).
- `Collectors.filtering(...)` y `Collectors.flatMapping(...)` (Java 9): filtran/aplanan **dentro** de un `groupingBy`, de forma que los grupos vacíos siguen apareciendo (con `filter` antes del collector desaparecerían).

---

## 16. Streams de primitivos

`IntStream`, `LongStream` y `DoubleStream` evitan el *boxing* y traen operaciones numéricas propias.

```java
int sumaEdades = empleados.stream().mapToInt(Empleado::edad).sum();

OptionalDouble mediaEdad = empleados.stream().mapToInt(Empleado::edad).average();   // OptionalDouble
OptionalInt   edadMax    = empleados.stream().mapToInt(Empleado::edad).max();

IntSummaryStatistics stats = empleados.stream().mapToInt(Empleado::edad).summaryStatistics();
stats.getMin();  stats.getMax();  stats.getAverage();  stats.getSum();  stats.getCount();
```

Detalles a recordar:

- `average()`, `max()` y `min()` devuelven `Optional...` porque el stream puede estar vacío.
- `sum()` de un `IntStream` puede **desbordar** (overflow) silenciosamente. Si los valores son grandes, usa `mapToLong`.
- Para pasar de primitivo a objeto: `boxed()`, `mapToObj(...)`.
- `IntStream.range(0, n)` sustituye muchos bucles `for` con contador.

---

## 17. Optional

`Optional<T>` representa **"un valor que puede estar o no"** y evita devolver `null`. Muchas operaciones de stream (`findFirst`, `min`, `max`, `reduce` sin identidad) lo devuelven.

### Crear

```java
Optional.of(valor);            // valor NO puede ser null (si lo es, NullPointerException)
Optional.ofNullable(valor);    // admite null -> Optional vacío
Optional.empty();
```

### Consumir (de mejor a peor práctica)

```java
Optional<Empleado> oe = empleados.stream().filter(e -> e.edad() > 40).findFirst();

// Transformar encadenando
String nombre = oe.map(Empleado::nombre).orElse("nadie");

// Valor por defecto: orElse evalúa SIEMPRE el argumento; orElseGet solo si hace falta
oe.orElse(crearPorDefectoCaro());          // se crea siempre
oe.orElseGet(() -> crearPorDefectoCaro()); // se crea solo si el Optional está vacío

// Lanzar excepción si no hay valor
Empleado e = oe.orElseThrow();                                   // Java 10+, NoSuchElementException
Empleado e2 = oe.orElseThrow(() -> new IllegalStateException("No hay candidatos"));

// Acciones
oe.ifPresent(x -> System.out.println(x.nombre()));
oe.ifPresentOrElse(x -> System.out.println(x.nombre()),          // Java 9
                   () -> System.out.println("Nadie"));

// Otras
oe.filter(x -> x.salario() > 30_000);
oe.or(() -> Optional.of(otroEmpleado));                          // alternativa perezosa (Java 9)
oe.isPresent();   oe.isEmpty();                                  // isEmpty: Java 11
```

### Reglas de oro

- **Evita `get()` sin comprobar.** Si no hay valor, falla. Prefiere `orElseThrow()`, que al menos expresa claramente la intención.
- **No uses `Optional` como campo de clase ni como parámetro de método.** Está pensado principalmente como tipo de retorno.
- **Nunca devuelvas `null` desde un método que devuelve `Optional`.**
- No lo uses para colecciones: devuelve una colección vacía.

---

## 18. Streams paralelos

Se obtienen con `parallelStream()` o `.parallel()`. El trabajo se reparte entre hilos del *pool* común `ForkJoinPool`.

```java
long primos = IntStream.rangeClosed(2, 5_000_000).parallel().filter(Main::esPrimo).count();
```

**No es una "bala de plata".** Dividir y recombinar tiene coste; con pocos datos o tareas baratas suele ir **más lento** que el secuencial.

Condiciones para que merezca la pena:

- **Muchos datos** y/o **operaciones costosas de CPU**.
- La fuente se divide bien (`ArrayList`, arrays, `IntStream.range` sí; `LinkedList` o `Stream.iterate` no).
- Operaciones **sin estado**, **sin efectos laterales** y **asociativas**.
- No bloquear con entrada/salida dentro de las lambdas (se agotaría el *pool* común compartido por toda la aplicación).

Peligros:

```java
List<Integer> resultado = new ArrayList<>();
IntStream.range(0, 1000).parallel().forEach(resultado::add);   // ¡ArrayList no es thread-safe! Resultados corruptos
```

Solución: usa un `collect` (diseñado para ser seguro en paralelo) en vez de mutar estructuras compartidas.

**Regla práctica:** primero secuencial, luego medir, y solo paralelizar si hay una mejora comprobada.

---

## 19. Rendimiento: ¿stream o bucle?

Una de las fuentes compara tiempos de bucles clásicos frente a streams con una lista de dos millones de enteros. Sus conclusiones, resumidas:

- Para operaciones **muy simples** (por ejemplo, calcular el máximo), los bucles `for` con iterador fueron claramente más rápidos que el stream.
- Con **varios filtros encadenados**, el stream empató o ganó ligeramente.
- La conclusión general del autor es un compromiso entre **rendimiento** y **mantenibilidad**.

Cómo interpretarlo con criterio:

1. **Las diferencias de micro-rendimiento rara vez importan** frente a la legibilidad. El cuello de botella casi nunca es el bucle: suele ser la base de datos, la red o un algoritmo mal elegido.
2. Los benchmarks manuales engañan (calentamiento de la JVM, optimizaciones del compilador). Si de verdad necesitas medir, usa **JMH**.
3. **Elige el streams cuando** expresas "qué" quieres (filtrar, transformar, agrupar) y el pipeline se lee de un vistazo.
4. **Elige un bucle cuando** necesitas modificar estado, hacer `break`/`continue`, manejar excepciones comprobadas, o la lógica se vuelve más enrevesada en forma de pipeline.
5. En caminos calientes con números, usa **streams de primitivos** o un bucle.

Una analogía: `2*2*2*2` frente a `2^4`. Para cuatro dos da igual; para 2⁶⁴ la expresión compacta compensa.

---

## 20. Novedades de Java 21 en adelante

Esta sección no aparece en las fuentes (que se centran en Java 8) y es lo que diferencia el Java actual.

### 20.1. `record` + streams: modelos de datos concisos

Ya los hemos usado. Sus accesores (`e.nombre()`) se pasan genial como referencias a métodos (`Empleado::nombre`).

### 20.2. `Stream.toList()` (Java 16)

Sustituye en la mayoría de casos a `.collect(Collectors.toList())`. Devuelve una lista **inmodificable** y admite `null`.

### 20.3. `SequencedCollection` (Java 21)

Las colecciones con orden definido (`List`, `Deque`, `LinkedHashSet`, `LinkedHashMap`...) tienen ahora métodos para acceder a los extremos y para invertirlas:

```java
List<String> nombres = List.of("Ana", "Luis", "Eva");

nombres.getFirst();                      // "Ana"   (antes: get(0))
nombres.getLast();                       // "Eva"   (antes: get(size() - 1))
nombres.reversed().stream().toList();    // [Eva, Luis, Ana]   (vista invertida, sin copiar)
```

Cuando solo necesitas el primer elemento de una lista, `getFirst()` es más directo que `stream().findFirst()`. (Lanza `NoSuchElementException` si la lista está vacía.)

### 20.4. *Pattern matching* para `switch` y *record patterns* (Java 21, definitivos)

Se combinan estupendamente con lambdas y streams:

```java
sealed interface Forma permits Circulo, Rectangulo {}
record Circulo(double radio) implements Forma {}
record Rectangulo(double ancho, double alto) implements Forma {}

double areaTotal = formas.stream()
    .mapToDouble(f -> switch (f) {
        case Circulo c                -> Math.PI * c.radio() * c.radio();
        case Rectangulo(double a, double h) -> a * h;      // record pattern: desestructura
    })
    .sum();
```

Al ser `Forma` una interfaz `sealed`, el `switch` es exhaustivo y no necesita `default`: si añades una forma nueva y olvidas tratarla, el compilador avisa.

### 20.5. Hilos virtuales (Java 21, definitivos)

Los **hilos virtuales** permiten lanzar miles o millones de tareas concurrentes baratas, ideales para código que espera entrada/salida. Las lambdas son la forma natural de definir cada tarea:

```java
try (var ejecutor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<String>> futuros = urls.stream()
        .map(url -> ejecutor.submit(() -> descargar(url)))   // cada tarea en su propio hilo virtual
        .toList();

    for (Future<String> f : futuros) {
        System.out.println(f.get());      // get() puede lanzar excepciones comprobadas
    }
}   // el try-with-resources espera a que acaben las tareas
```

Distinción clave con los streams paralelos:

- **Stream paralelo:** reparte trabajo **de CPU** entre los núcleos.
- **Hilos virtuales:** gestionan mucha **concurrencia de E/S** (llamadas HTTP, base de datos...).

No los confundas ni uses un `parallelStream()` para peticiones de red.

### 20.6. Variables sin nombre `_` (preview en 21, definitivo en Java 22)

Cuando un parámetro de lambda no se usa, se puede escribir `_`:

```java
mapa.forEach((_, valor) -> System.out.println(valor));   // Java 22+
```

En Java 21 solo está disponible como característica en vista previa (hay que activar `--enable-preview`).

### 20.7. `Gatherers`: operaciones intermedias personalizadas (definitivo en Java 24)

Hasta ahora, el conjunto de operaciones intermedias era fijo. La API `Stream Gatherers` permite definirlas propias y trae algunas ya hechas:

```java
List<List<Integer>> ventanas = Stream.of(1, 2, 3, 4, 5, 6, 7)
    .gather(Gatherers.windowFixed(3))
    .toList();                              // [[1, 2, 3], [4, 5, 6], [7]]

Stream.of(1, 2, 3, 4).gather(Gatherers.windowSliding(2)).toList();   // [[1, 2], [2, 3], [3, 4]]
```

No existe en Java 21. Está disponible sin *flags* desde Java 24 y, por tanto, en Java 25 (LTS).

### 20.8. Resumen de "desde qué versión" de lo relevante

| Característica | Desde |
|---|---|
| Lambdas, `Stream`, `Optional`, `java.util.function`, referencias a métodos | 8 |
| `takeWhile`, `dropWhile`, `Stream.iterate` con predicado, `Optional.ifPresentOrElse/or/stream`, `Collectors.filtering/flatMapping` | 9 |
| `var` en parámetros de lambda, `Predicate.not`, `Optional.isEmpty` | 11 |
| `Collectors.teeing` | 12 |
| `Stream.toList()`, `mapMulti`, `record` (definitivo) | 16 |
| `SequencedCollection`, hilos virtuales, *pattern matching* en `switch`, *record patterns* | 21 |
| Variables sin nombre `_` | 22 |
| `Stream Gatherers` | 24 |

---

## 21. Errores típicos

| Error | Qué pasa | Cómo se evita |
|---|---|---|
| Reutilizar un stream | `IllegalStateException` | Crear un stream nuevo cada vez |
| Olvidar la operación terminal | No se ejecuta nada (y no hay error) | Comprobar que el pipeline acaba en terminal |
| Mutar la fuente mientras se recorre | `ConcurrentModificationException` o resultados raros | No modificar la colección que alimenta el stream |
| Acumular con `forEach` en una lista/contador externo | No compila (variable) o no es seguro en paralelo | Usar `collect`, `reduce` o `sum` |
| `toMap` con claves repetidas | `IllegalStateException` | Pasar función de fusión o usar `groupingBy` |
| Hacer `add` sobre el resultado de `.toList()` | `UnsupportedOperationException` | `Collectors.toCollection(ArrayList::new)` |
| `Optional.get()` sin comprobar | `NoSuchElementException` | `orElse`, `orElseThrow`, `map`, `ifPresent` |
| Stream infinito sin límite | El programa no termina | `limit` o `takeWhile` |
| Usar `peek` para lógica | Comportamiento no garantizado | `map` o `forEach` |
| `Stream<Integer>` para sumar millones | Boxing innecesario, más lento | `IntStream` / `mapToInt` |
| Ordenar con `(a, b) -> a - b` | Desbordamiento con valores extremos | `Integer.compare(a, b)` o `Comparator.comparingInt(...)` |
| `sorted()` sobre objetos no `Comparable` | `ClassCastException` | Pasar un `Comparator` |
| `parallelStream()` con estado compartido | Resultados incorrectos | `collect`; no mutar estructuras compartidas |
| `var f = x -> ...` | No compila | Declarar el tipo funcional explícito |
| Lambdas de 20 líneas | Código ilegible | Extraer método + referencia a método |

---

## 22. Aclaraciones sobre las fuentes

Al contrastar los textos con el JDK actual, hay detalles que conviene tener presentes para no aprender algo incorrecto:

- **Artículo de Arteco:** habla del paquete `java.util.functional`. El nombre correcto es **`java.util.function`**. Además, algunas cifras de rendimiento (como la mejora del 15 %) son anecdóticas de su experiencia, no garantías generales. Y que una lambda sin captura pueda reutilizarse como instancia única es un detalle de implementación de la JVM, no una garantía del lenguaje.
- **Tutorial de Adictos al Trabajo:** el ejemplo `aList.stream().findFirst(element -> ...)` no es válido, porque `findFirst()` **no recibe predicado**; lo correcto es `.filter(...).findFirst()`. Otros fragmentos son esquemáticos y no compilan tal cual: `() -> return createRandomInteger()` (un `return` exige llaves), `String message -> ...` (con tipo declarado, los paréntesis son obligatorios), `message.length > 50` (es el método `length()`) y un `println` con formato (para eso se usa `printf`).
- **Artículo de CleverIT:** `int cantidad = names.stream().limit(5).count();` no compila porque `count()` devuelve **`long`**. Usa `Lists.newArrayList` de Guava, que hoy se sustituye por `List.of` o `new ArrayList<>(List.of(...))`. Los benchmarks son ilustrativos; para decisiones reales, JMH.
- **Oracle (artículo LATAM y tutorial oficial):** están escritos para **Java 8**. El propio tutorial lo indica y remite a dev.java para material actualizado. Los conceptos (sintaxis, interfaces funcionales, captura, *target typing*, referencias a métodos) siguen plenamente vigentes; lo que falta es todo lo posterior (sección 20).
- **Baeldung:** su guía de buenas prácticas sigue siendo válida. Se ha integrado en la sección 9.

---

## 23. Ejercicios

Usa el dominio de `Empleado` de la sección 1, salvo que se indique otra cosa.

1. Obtén los **nombres** de los empleados de Ingeniería, ordenados alfabéticamente.
2. Calcula el **salario medio** de toda la plantilla.
3. Obtén el **empleado mejor pagado** (como `Optional`) y muestra su nombre o "sin datos" si no hay.
4. Agrupa los empleados por departamento y obtén, para cada uno, **cuántos hay**.
5. Obtén un `Map<String, Double>` con la **masa salarial** por departamento.
6. Divide la plantilla en **mayores y menores de 30 años** (dos grupos).
7. Dada `List<String> frases = List.of("hola mundo", "hola java", "java moderno")`, obtén la **lista de palabras distintas** ordenadas.
8. Dada `List<String> palabras`, calcula cuántas veces aparece cada palabra (`Map<String, Long>`).
9. Escribe **tu propia interfaz funcional** `Operacion` (dos `double`, devuelve `double`) con `@FunctionalInterface`, y úsala con lambdas para sumar, restar y multiplicar.
10. Convierte esta clase anónima en una lambda y luego en una referencia a método:

    ```java
    Comparator<String> porLongitud = new Comparator<String>() {
        @Override
        public int compare(String a, String b) {
            return Integer.compare(a.length(), b.length());
        }
    };
    ```

11. **Depuración:** este código no compila. Explica por qué y corrígelo.

    ```java
    int total = 0;
    empleados.forEach(e -> total += e.edad());
    ```

12. **Reto:** usando `teeing`, calcula en una sola pasada el salario máximo y el mínimo de la plantilla.

---

## 24. Soluciones

```java
// 1
empleados.stream()
    .filter(e -> e.departamento().equals("Ingeniería"))
    .map(Empleado::nombre)
    .sorted()
    .toList();

// 2
double media = empleados.stream().mapToDouble(Empleado::salario).average().orElse(0);

// 3
String nombre = empleados.stream()
    .max(Comparator.comparingDouble(Empleado::salario))
    .map(Empleado::nombre)
    .orElse("sin datos");

// 4
Map<String, Long> porDepto = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento, Collectors.counting()));

// 5
Map<String, Double> masaSalarial = empleados.stream()
    .collect(Collectors.groupingBy(Empleado::departamento,
             Collectors.summingDouble(Empleado::salario)));

// 6
Map<Boolean, List<Empleado>> partes = empleados.stream()
    .collect(Collectors.partitioningBy(e -> e.edad() >= 30));   // true: 30 o más; false: menores de 30

// 7
frases.stream()
    .flatMap(f -> Arrays.stream(f.split(" ")))
    .distinct()
    .sorted()
    .toList();

// 8
Map<String, Long> frecuencias = palabras.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

// 9
@FunctionalInterface
interface Operacion {
    double aplicar(double a, double b);
}

Operacion suma  = (a, b) -> a + b;
Operacion resta = (a, b) -> a - b;
Operacion mult  = (a, b) -> a * b;
// Equivalente con la interfaz estándar: DoubleBinaryOperator

// 10
Comparator<String> porLongitud = (a, b) -> Integer.compare(a.length(), b.length());
Comparator<String> porLongitudRef = Comparator.comparingInt(String::length);

// 11
// Error: 'total' se modifica dentro de la lambda, así que no es efectivamente final.
int total = empleados.stream().mapToInt(Empleado::edad).sum();

// 12
var extremos = empleados.stream().collect(Collectors.teeing(
    Collectors.maxBy(Comparator.comparingDouble(Empleado::salario)),
    Collectors.minBy(Comparator.comparingDouble(Empleado::salario)),
    (max, min) -> "máx=" + max.map(Empleado::salario).orElse(0.0)
                + ", mín=" + min.map(Empleado::salario).orElse(0.0)));
```

---

## 25. Chuleta final

**Lambdas**

- `(params) -> cuerpo`. Se asigna a una interfaz funcional (un solo método abstracto).
- Tipo por contexto: no existe `var f = x -> x;`.
- Solo capturan variables locales **final o efectivamente finales**. `this` es el de la clase contenedora.
- Referencias a métodos: `Clase::estatico`, `objeto::metodo`, `Tipo::metodoInstancia`, `Clase::new`.
- Corto, sin efectos laterales, y con `@FunctionalInterface` en las propias.

**Interfaces que debes saber de memoria**

| Necesito... | Uso |
|---|---|
| Comprobar algo (devuelve `boolean`) | `Predicate<T>` |
| Transformar | `Function<T,R>` / `UnaryOperator<T>` |
| Combinar dos del mismo tipo | `BinaryOperator<T>` |
| Producir sin entrada | `Supplier<T>` |
| Hacer algo sin devolver | `Consumer<T>` |
| Ordenar | `Comparator<T>` |

**Streams**

- `fuente → intermedias (perezosas) → terminal`. Se usa una vez.
- Filtrar pronto, ordenar tarde.
- Resultado: `toList()` (inmodificable), `collect(...)`, `count()`, `sum()` en primitivos, `reduce`, `findFirst`/`anyMatch`.
- Agrupar: `groupingBy`, `partitioningBy`, `toMap` (cuidado con claves duplicadas), `joining`, `teeing`.
- Primitivos: `mapToInt`/`IntStream` cuando hagas aritmética.
- `Optional`: `map`, `orElse`, `orElseGet`, `orElseThrow`, `ifPresent`. Nunca `get()` a ciegas.
- Paralelo: solo con muchos datos, CPU y sin estado compartido; medir antes.

**Java moderno**

- `record` para datos, `toList()`, `getFirst()/getLast()/reversed()`, `switch` con patrones sobre tipos `sealed`, hilos virtuales para E/S concurrente, `Gatherers` desde Java 24.

---

## 26. Fuentes

Contenido base resumido de:

1. Oracle LATAM, *Introducción a Expresiones Lambda y API Stream en Java SE 8 – Parte 1* (Alexis López): https://www.oracle.com/latam/technical-resources/articles/java/expresiones-lambda-api-stream-java.html
2. Oracle, *Lambda Expressions* (The Java Tutorials): https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html
3. Baeldung, *Lambda Expressions and Functional Interfaces: Tips and Best Practices*: https://www.baeldung.com/java-8-lambda-expressions-tips
4. Arteco Consulting, *Introducción a Java Lambda*: https://www.arteco-consulting.com/articulos/introduccion-java-lambda/
5. CleverIT Group, *Comenzando con Lambdas y Streams en Java*: https://www.cleveritgroup.com/es/blog/comenzando-con-lambdas-y-streams-en-java
6. Adictos al Trabajo, *Expresiones Lambda con Java 8*: https://adictosaltrabajo.com/2015/12/04/expresiones-lambda-con-java-8/

Para profundizar: la documentación oficial de la API de Java (paquetes `java.util.function` y `java.util.stream`) y el sitio https://dev.java/learn/.
