# Programación funcional en Java (Java 25+)

> **Alcance:** esta guía explica el **paradigma** de la programación funcional y cómo aplicarlo en Java 25 (versión LTS): funciones puras, inmutabilidad, funciones de orden superior, composición, evaluación perezosa, tipos de datos con `record` y `sealed`, y manejo de errores sin excepciones. La sintaxis de las lambdas y el catálogo completo de operaciones de `Stream` se tratan en detalle en la guía [Lambdas y Streams](<Lambdas y Streams.md>); aquí se resume lo imprescindible y se pone el foco en **cómo pensar** de forma funcional. Todos los programas completos se han compilado y ejecutado con OpenJDK 25 y su salida es la que se muestra.

---

## Índice

**Introducción**

0. [Cómo probar los ejemplos](#0-cómo-probar-los-ejemplos)
1. [Qué es la programación funcional](#1-qué-es-la-programación-funcional)
2. [Imperativo frente a declarativo](#2-imperativo-frente-a-declarativo)
3. [Repaso rápido: lambdas e interfaces funcionales](#3-repaso-rápido-lambdas-e-interfaces-funcionales)

**Principios**

4. [Funciones puras](#4-funciones-puras)
5. [Efectos secundarios: aislarlos, no eliminarlos](#5-efectos-secundarios-aislarlos-no-eliminarlos)
6. [Inmutabilidad](#6-inmutabilidad)
7. [Transparencia referencial](#7-transparencia-referencial)

**Funciones como valores**

8. [Funciones de primera clase](#8-funciones-de-primera-clase)
9. [Funciones de orden superior](#9-funciones-de-orden-superior)
10. [Composición de funciones](#10-composición-de-funciones)
11. [Clausuras, currificación y aplicación parcial](#11-clausuras-currificación-y-aplicación-parcial)

**Técnicas**

12. [Evaluación perezosa](#12-evaluación-perezosa)
13. [Memoización](#13-memoización)
14. [Recursividad en Java](#14-recursividad-en-java)
15. [Datos inmutables y pattern matching](#15-datos-inmutables-y-pattern-matching)
16. [Optional: ausencia de valor sin null](#16-optional-ausencia-de-valor-sin-null)
17. [Errores como valores: el tipo Resultado](#17-errores-como-valores-el-tipo-resultado)
18. [Streams con estilo funcional](#18-streams-con-estilo-funcional)
19. [Paralelismo y funciones puras](#19-paralelismo-y-funciones-puras)
20. [Cuándo usar el estilo funcional](#20-cuándo-usar-el-estilo-funcional)

**Para terminar**

21. [Ejemplo completo: procesar pedidos](#21-ejemplo-completo-procesar-pedidos)
22. [Errores frecuentes](#22-errores-frecuentes)
23. [Chuleta rápida](#23-chuleta-rápida)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Soluciones](#25-soluciones)
26. [Glosario](#26-glosario)
27. [Referencias](#27-referencias)

---

## 0. Cómo probar los ejemplos

Los programas completos usan un **fichero fuente compacto** de Java 25: un `void main()` y, alrededor, los métodos, *records* e interfaces que necesita el ejemplo. Un fichero compacto importa automáticamente todo el módulo `java.base`, así que `List`, `Map`, `Optional`, `Function`, `Stream` o `Collectors` se usan sin `import`. Se ejecutan así:

```bash
java Ejemplo.java
```

Los fragmentos cortos se pueden probar en **JShell** (`jshell`). En JShell, las interfaces de `java.util.function` hay que importarlas: `import java.util.function.*;`.

> 💡 Si necesitas repasar *records*, interfaces o clases selladas, consulta antes [Herencia, encapsulación y polimorfismo](<../01-Fundamentos-del-lenguaje/Herencia, encapsulacion y polimorfismo.md>).

---

## 1. Qué es la programación funcional

La **programación funcional** es un **paradigma**, es decir, una forma de plantear los programas. Su idea central es construir el programa **combinando funciones** que transforman datos, en lugar de dar instrucciones que van modificando el estado paso a paso.

Se basa en unas pocas ideas:

| Idea | En una frase |
|---|---|
| **Funciones puras** | Mismo resultado para los mismos datos y sin efectos ocultos |
| **Inmutabilidad** | Los datos no se modifican: se crean datos nuevos |
| **Funciones como valores** | Una función se puede guardar en una variable, pasar como argumento y devolver |
| **Composición** | Funciones pequeñas se combinan para formar otras más grandes |
| **Estilo declarativo** | Se describe **qué** se quiere obtener, no **cómo** obtenerlo paso a paso |

### 1.1. Java es multiparadigma

Java nació como un lenguaje **orientado a objetos**. Desde **Java 8** (2014) incorpora herramientas funcionales: lambdas, referencias a métodos, interfaces funcionales, `Stream` y `Optional`. Versiones posteriores han añadido piezas que encajan muy bien con el estilo funcional: *records* (16), clases selladas (17), `switch` con patrones (21) y *gatherers* (24).

Java **no es un lenguaje funcional puro** como Haskell: no obliga a nada. Lo habitual es **combinar** ambos estilos: objetos para organizar el programa y estilo funcional para transformar datos y escribir lógica de negocio fácil de probar.

### 1.2. ¿Por qué merece la pena?

- **Código más fácil de entender:** cada función hace una cosa y su resultado solo depende de su entrada.
- **Más fácil de probar:** una función pura se prueba con una llamada y una comprobación, sin preparar objetos ni simular bases de datos.
- **Menos errores:** sin estado compartido que cambia, desaparecen muchos errores difíciles de reproducir.
- **Concurrencia más segura:** los datos inmutables se pueden compartir entre hilos sin bloqueos.
- **Reutilización:** las funciones pequeñas se combinan como piezas de construcción.

---

## 2. Imperativo frente a declarativo

**Problema:** de una lista de personas, obtener en mayúsculas y ordenados alfabéticamente los nombres de los mayores de edad.

```java
// ImperativoDeclarativo.java
record Persona(String nombre, int edad) { }

void main() {
    List<Persona> personas = List.of(
        new Persona("lucía", 34), new Persona("mario", 15),
        new Persona("ana", 22), new Persona("pablo", 17));

    // Imperativo: CÓMO hacerlo, paso a paso
    List<String> resultado = new ArrayList<>();
    for (Persona p : personas) {
        if (p.edad() >= 18) {
            resultado.add(p.nombre().toUpperCase());
        }
    }
    Collections.sort(resultado);
    IO.println(resultado);

    // Declarativo: QUÉ se quiere obtener
    List<String> nombres = personas.stream()
        .filter(p -> p.edad() >= 18)
        .map(p -> p.nombre().toUpperCase())
        .sorted()
        .toList();
    IO.println(nombres);
}
```

```text
[ANA, LUCÍA]
[ANA, LUCÍA]
```

| | Imperativo | Declarativo (funcional) |
|---|---|---|
| Describe | Los pasos: crear lista, recorrer, comprobar, añadir, ordenar | El resultado: filtrar, transformar, ordenar |
| Estado | Una lista que se va modificando | Ningún estado intermedio visible |
| Lectura | Hay que «ejecutar mentalmente» el bucle | Se lee casi como una frase |
| Errores típicos | Índices, olvidar inicializar, modificar la lista equivocada | Menos ocasiones de error |

Ninguno es «mejor» siempre: un bucle sencillo es perfectamente válido. Pero cuando las transformaciones se encadenan, el estilo declarativo suele expresar la intención con más claridad.

---

## 3. Repaso rápido: lambdas e interfaces funcionales

Una **lambda** es una función anónima que se escribe en el sitio donde se usa:

```java
(parámetros) -> expresión
(parámetros) -> { sentencias; return valor; }
```

Una lambda siempre se asigna a una **interfaz funcional**: una interfaz con **un único método abstracto**. Las más usadas están en `java.util.function`:

| Interfaz | Recibe → devuelve | Método | Ejemplo |
|---|---|---|---|
| `Function<T, R>` | `T` → `R` | `apply` | `Function<String, Integer> longitud = String::length;` |
| `BiFunction<T, U, R>` | `T`, `U` → `R` | `apply` | `BiFunction<Integer, Integer, Integer> suma = Integer::sum;` |
| `UnaryOperator<T>` | `T` → `T` | `apply` | `UnaryOperator<String> mayus = String::toUpperCase;` |
| `BinaryOperator<T>` | `T`, `T` → `T` | `apply` | `BinaryOperator<Integer> max = Math::max;` |
| `Predicate<T>` | `T` → `boolean` | `test` | `Predicate<String> vacio = String::isBlank;` |
| `Supplier<T>` | nada → `T` | `get` | `Supplier<LocalDate> hoy = LocalDate::now;` |
| `Consumer<T>` | `T` → nada | `accept` | `Consumer<String> imprimir = IO::println;` |

Existen variantes para primitivos (`IntFunction`, `IntPredicate`, `ToIntFunction`, `IntUnaryOperator`...) que evitan convertir `int` a `Integer`.

Las **referencias a métodos** (`String::length`, `IO::println`, `Persona::new`) son una forma abreviada de escribir una lambda que solo llama a un método.

> 📚 Sintaxis completa, inferencia de tipos, captura de variables, referencias a métodos y excepciones dentro de lambdas: [Lambdas y Streams](<Lambdas y Streams.md>), secciones 2 a 10.

---

## 4. Funciones puras

Una función es **pura** si cumple dos condiciones:

1. **Determinista:** con los mismos argumentos, devuelve **siempre** el mismo resultado.
2. **Sin efectos secundarios:** no modifica nada fuera de ella ni depende de nada que pueda cambiar (variables externas, ficheros, consola, base de datos, hora actual...).

```java
// ✅ Pura
int sumar(int a, int b) {
    return a + b;
}

// ✅ Pura: crea y devuelve datos nuevos
List<String> enMayusculas(List<String> textos) {
    return textos.stream().map(String::toUpperCase).toList();
}
```

### 4.1. Ejemplos de funciones impuras

```java
int contador = 0;

// ❌ Modifica estado externo
int siguiente() {
    return ++contador;              // cada llamada devuelve algo distinto
}

// ❌ Modifica su argumento
void ordenar(List<String> lista) {
    Collections.sort(lista);        // quien llamó ve su lista cambiada
}

// ❌ Depende de algo que cambia
boolean esMayorDeEdad(LocalDate nacimiento) {
    return Period.between(nacimiento, LocalDate.now()).getYears() >= 18;   // depende de hoy
}

// ❌ Entrada/salida
double precioConIva(double precio) {
    IO.println("Calculando...");    // efecto secundario: escribe en consola
    return precio * 1.21;
}

// ❌ Aleatoriedad
int tirarDado() {
    return ThreadLocalRandom.current().nextInt(1, 7);
}
```

### 4.2. Cómo convertir una función impura en pura

Muchas veces basta con **pasar como parámetro** lo que la función consultaba por su cuenta:

```java
// ❌ Impura: depende de la fecha actual
boolean esMayorDeEdad(LocalDate nacimiento) {
    return Period.between(nacimiento, LocalDate.now()).getYears() >= 18;
}

// ✅ Pura: la fecha de referencia es un parámetro
boolean esMayorDeEdad(LocalDate nacimiento, LocalDate hoy) {
    return Period.between(nacimiento, hoy).getYears() >= 18;
}
```

Ahora la prueba es trivial y siempre da el mismo resultado:

```java
esMayorDeEdad(LocalDate.of(2008, 3, 1), LocalDate.of(2026, 2, 28));   // false
esMayorDeEdad(LocalDate.of(2008, 3, 1), LocalDate.of(2026, 3, 1));    // true
```

Y si la función modificaba su argumento, haz que **devuelva un resultado nuevo**:

```java
// ✅ No toca la lista original
List<String> ordenada(List<String> lista) {
    return lista.stream().sorted().toList();
}
```

### 4.3. Ventajas de las funciones puras

| Ventaja | Por qué |
|---|---|
| **Fáciles de probar** | Solo hay que comprobar salida para una entrada; sin preparar entorno |
| **Fáciles de entender** | Todo lo que necesitas saber está en su firma y su cuerpo |
| **Reutilizables** | No dependen de un contexto concreto |
| **Seguras en paralelo** | No comparten estado, no hay condiciones de carrera |
| **Cacheables** | Se puede guardar el resultado y reutilizarlo (sección 13) |

---

## 5. Efectos secundarios: aislarlos, no eliminarlos

Un programa sin ningún efecto secundario no serviría de nada: no podría leer datos, mostrar resultados ni guardar información. El objetivo no es eliminarlos, sino **aislarlos**.

Una forma de organizarlo es el patrón **«núcleo funcional, capa imperativa»** (*functional core, imperative shell*):

```text
┌────────────────────────────────────────────────────────────┐
│ Capa imperativa (pequeña)                                  │
│   leer fichero · consultar BD · pedir datos al usuario     │
│        │                                                   │
│        ▼                                                   │
│   ┌──────────────────────────────────────────────┐         │
│   │ Núcleo funcional (la mayor parte)            │         │
│   │   funciones puras: validar, calcular,        │         │
│   │   transformar, decidir                       │         │
│   └──────────────────────────────────────────────┘         │
│        │                                                   │
│        ▼                                                   │
│   escribir resultado · guardar en BD · enviar email        │
└────────────────────────────────────────────────────────────┘
```

```java
// Capa imperativa: lee, llama al núcleo y muestra
void main() {
    List<String> lineas = Files.readAllLines(Path.of("ventas.csv"));   // efecto
    Informe informe = generarInforme(lineas);                           // puro
    IO.println(informe.formatear());                                    // efecto
}

// Núcleo funcional: solo transforma datos
Informe generarInforme(List<String> lineas) { ... }
```

La lógica importante (`generarInforme`) se puede probar sin ficheros, y los efectos quedan concentrados en pocas líneas fáciles de revisar.

---

## 6. Inmutabilidad

Un dato **inmutable** no puede cambiar después de crearse. Para «modificarlo» se crea uno **nuevo** con los cambios. Es la base de la programación funcional: si los datos no cambian, las funciones no pueden tener efectos ocultos sobre ellos.

### 6.1. Herramientas de Java para la inmutabilidad

| Herramienta | Qué hace |
|---|---|
| `final` en variables y atributos | Impide reasignar la variable (pero no modificar el objeto al que apunta) |
| `record` | Clase de datos con atributos `private final`, sin setters |
| `List.of`, `Set.of`, `Map.of` | Crean colecciones **inmutables** |
| `List.copyOf`, `Set.copyOf`, `Map.copyOf` | Copia inmutable de una colección existente |
| `Stream.toList()` (Java 16) | Devuelve una lista inmutable |
| `String`, `Integer`, `LocalDate`, `BigDecimal` | Clases inmutables de la biblioteca estándar |

```java
List<String> colores = List.of("rojo", "verde");
colores.add("azul");            // 💥 UnsupportedOperationException
```

### 6.2. Modificar sin mutar: métodos «with» en records

Un record no tiene setters. Para cambiar un campo se devuelve una **copia** con ese campo cambiado:

```java
// ConMetodosWith.java
record Producto(String nombre, double precio, int stock) {
    Producto conPrecio(double nuevoPrecio) {
        return new Producto(nombre, nuevoPrecio, stock);
    }

    Producto conStock(int nuevoStock) {
        return new Producto(nombre, precio, nuevoStock);
    }
}

void main() {
    var original = new Producto("Teclado", 49.90, 10);
    var rebajado = original.conPrecio(39.90).conStock(8);

    IO.println(original);
    IO.println(rebajado);
}
```

```text
Producto[nombre=Teclado, precio=49.9, stock=10]
Producto[nombre=Teclado, precio=39.9, stock=8]
```

El original **sigue intacto**. Cualquier parte del programa que lo tuviera puede seguir confiando en sus valores.

### 6.3. Inmodificable no es lo mismo que inmutable

`Collections.unmodifiableList(lista)` devuelve una **vista** de solo lectura, pero si alguien modifica la lista original, la vista **también cambia**. `List.copyOf` crea una copia independiente:

```java
List<String> original = new ArrayList<>(List.of("a", "b"));
List<String> vista = Collections.unmodifiableList(original);
List<String> copia = List.copyOf(original);

original.add("c");
IO.println(vista);   // [a, b, c]  ← ha cambiado
IO.println(copia);   // [a, b]     ← independiente
```

### 6.4. ¿Y el rendimiento?

Crear objetos nuevos en lugar de modificar los existentes tiene un coste, pero en la mayoría de aplicaciones es **despreciable**: la JVM está muy optimizada para crear objetos pequeños de vida corta. Optimiza solo cuando una medición demuestre que es un problema.

---

## 7. Transparencia referencial

Una expresión tiene **transparencia referencial** si se puede **sustituir por su valor** sin que cambie el comportamiento del programa.

```java
int total = sumar(2, 3) * sumar(2, 3);
// sumar es pura → se puede sustituir por su valor:
int total = 5 * 5;                       // el programa se comporta igual
```

Con una función impura, la sustitución cambia el resultado:

```java
int a = siguiente() + siguiente();       // 1 + 2 = 3 (si contador empezaba en 0)
int x = siguiente();
int b = x + x;                           // 3 + 3 = 6  ← ¡distinto!
```

La transparencia referencial es lo que permite **razonar** sobre el código como si fueran ecuaciones matemáticas, refactorizar con seguridad (extraer variables, reordenar llamadas) y que el compilador o la biblioteca optimicen (cachear, paralelizar).

---

## 8. Funciones de primera clase

En Java, gracias a las interfaces funcionales, las funciones son **valores**: se pueden guardar en variables, en colecciones y en atributos, igual que un número o un `String`.

```java
// Calculadora.java
void main() {
    Map<String, BinaryOperator<Integer>> operaciones = Map.of(
        "+", Integer::sum,
        "-", (a, b) -> a - b,
        "*", (a, b) -> a * b,
        "max", Math::max
    );

    for (String op : List.of("+", "-", "*", "max")) {
        IO.println("8 " + op + " 3 = " + operaciones.get(op).apply(8, 3));
    }
}
```

```text
8 + 3 = 11
8 - 3 = 5
8 * 3 = 24
8 max 3 = 8
```

Este `Map` sustituye a un `switch` sobre el operador: añadir una operación nueva es añadir una entrada, sin tocar la lógica. Es el patrón **Strategy** expresado con funciones (ver [Patrones de comportamiento](<../07-Diseno/Patrones/Patrones comportamiento.md>)).

---

## 9. Funciones de orden superior

Una **función de orden superior** es la que **recibe funciones como parámetros** o **devuelve una función** como resultado. Ya las usas: `filter`, `map`, `sorted` o `forEach` son funciones de orden superior.

### 9.1. Recibir una función

Permite separar **qué se repite** de **qué cambia**:

```java
// Sin orden superior: tres métodos casi iguales
List<Producto> baratos(List<Producto> ps)   { /* bucle con precio < 10 */ }
List<Producto> sinStock(List<Producto> ps)  { /* bucle con stock == 0 */ }
List<Producto> deMarca(List<Producto> ps)   { /* bucle con marca == X */ }

// Con orden superior: un solo método, el criterio es un parámetro
<T> List<T> filtrar(List<T> elementos, Predicate<T> criterio) {
    List<T> resultado = new ArrayList<>();
    for (T e : elementos) {
        if (criterio.test(e)) {
            resultado.add(e);
        }
    }
    return List.copyOf(resultado);
}

filtrar(productos, p -> p.precio() < 10);
filtrar(productos, p -> p.stock() == 0);
filtrar(nombres, n -> n.startsWith("A"));      // funciona con cualquier tipo
```

### 9.2. Devolver una función

Una función puede **fabricar** otras funciones configuradas:

```java
// FabricaDeFunciones.java
Function<Double, Double> descuento(int porcentaje) {
    return precio -> precio * (100 - porcentaje) / 100;
}

Predicate<String> longitudEntre(int min, int max) {
    return s -> s.length() >= min && s.length() <= max;
}

void main() {
    Function<Double, Double> rebajas = descuento(20);
    Function<Double, Double> blackFriday = descuento(50);
    IO.println(rebajas.apply(100.0));
    IO.println(blackFriday.apply(100.0));

    Predicate<String> usuarioValido = longitudEntre(4, 12);
    IO.println(usuarioValido.test("ana"));
    IO.println(usuarioValido.test("ana_garcia"));
}
```

```text
80.0
50.0
false
true
```

### 9.3. Ejecutar «algo» alrededor de otra cosa

Un uso muy práctico es envolver un trozo de código con un comportamiento común (medir tiempo, reintentar, registrar):

```java
<T> T medir(String etiqueta, Supplier<T> tarea) {
    long inicio = System.nanoTime();
    T resultado = tarea.get();
    long ms = (System.nanoTime() - inicio) / 1_000_000;
    IO.println(etiqueta + ": " + ms + " ms");
    return resultado;
}

long primos = medir("Contar primos", () ->
    IntStream.rangeClosed(2, 100_000).filter(this::esPrimo).count());
```

---

## 10. Composición de funciones

**Componer** es encadenar funciones de modo que la salida de una sea la entrada de la siguiente, creando una función nueva. En matemáticas se escribe `g ∘ f`; en Java, con métodos `default` de las interfaces funcionales.

### 10.1. Function: andThen y compose

```java
Function<String, String> limpiar = String::strip;
Function<String, String> mayusculas = String::toUpperCase;
Function<String, Integer> longitud = String::length;

Function<String, String> normalizar = limpiar.andThen(mayusculas);   // primero limpiar, luego mayúsculas
Function<String, Integer> medir = normalizar.andThen(longitud);

normalizar.apply("  hola  ");     // "HOLA"
medir.apply("  hola  ");          // 4

// compose aplica primero el argumento: f.compose(g) = f(g(x))
Function<String, String> igual = mayusculas.compose(limpiar);        // equivale a limpiar.andThen(mayusculas)
```

```text
andThen:  x ──► limpiar ──► mayusculas ──► resultado
compose:  x ──► limpiar ──► mayusculas ──► resultado   (escrito al revés: mayusculas.compose(limpiar))
```

`Function.identity()` devuelve una función que no hace nada (`x -> x`). Es útil como punto de partida al componer muchas funciones:

```java
// ComponerPipeline.java
void main() {
    List<UnaryOperator<String>> pasos = List.of(
        String::strip,
        s -> s.replaceAll("\\s+", " "),
        String::toLowerCase,
        s -> s.replace(' ', '-')
    );

    Function<String, String> slug = Function.identity();
    for (UnaryOperator<String> paso : pasos) {
        slug = slug.andThen(paso);
    }

    IO.println(slug.apply("  Programación   Funcional en Java  "));
}
```

```text
programación-funcional-en-java
```

### 10.2. Predicate: and, or, negate y not

```java
Predicate<String> noVacio = Predicate.not(String::isBlank);       // Java 11
Predicate<String> corto = s -> s.length() <= 10;
Predicate<String> empiezaPorA = s -> s.startsWith("A");

Predicate<String> valido = noVacio.and(corto);
Predicate<String> interesante = empiezaPorA.or(corto.negate());
```

### 10.3. Comparator: comparing y thenComparing

`Comparator` también se compone:

```java
Comparator<Persona> orden = Comparator
    .comparing(Persona::apellido)
    .thenComparing(Persona::nombre)
    .thenComparing(Persona::edad, Comparator.reverseOrder());
```

Más en la guía [Comparable vs Comparator](<../01-Fundamentos-del-lenguaje/Comparable vs Comparator.md>).

### 10.4. Otras composiciones útiles

| Interfaz | Métodos para componer |
|---|---|
| `Function` | `andThen`, `compose`, `identity` |
| `UnaryOperator` | `identity` (y los de `Function`) |
| `BiFunction` | `andThen` |
| `BinaryOperator` | `minBy`, `maxBy` |
| `Predicate` | `and`, `or`, `negate`, `not`, `isEqual` |
| `Consumer` | `andThen` |
| `Comparator` | `comparing`, `thenComparing`, `reversed`, `nullsFirst`, `nullsLast` |

---

## 11. Clausuras, currificación y aplicación parcial

### 11.1. Clausuras

Una **clausura** (*closure*) es una función que **recuerda las variables del contexto** donde se creó, aunque ese contexto ya haya terminado. En la sección 9.2, la lambda que devuelve `descuento(20)` recuerda `porcentaje = 20` después de que el método haya terminado.

En Java, una lambda solo puede capturar variables locales **`final` o *effectively final*** (que no se reasignan después):

```java
int minimo = 18;
Predicate<Persona> adulto = p -> p.edad() >= minimo;    // ✅ minimo no cambia

int contador = 0;
lista.forEach(e -> contador++);     // ❌ error: local variables referenced from a lambda
                                    //    expression must be final or effectively final
```

Esta restricción es **intencionada**: empuja a no modificar estado desde las lambdas. En lugar de contar con una variable externa, se usa una operación que devuelve el resultado (`count()`, `reduce`, `collect`).

### 11.2. Currificación

**Currificar** es transformar una función de varios parámetros en una **cadena de funciones de un solo parámetro**:

```java
// Normal: dos parámetros a la vez
BiFunction<Integer, Integer, Integer> sumar = (a, b) -> a + b;
sumar.apply(3, 4);                   // 7

// Currificada: un parámetro cada vez
Function<Integer, Function<Integer, Integer>> sumarC = a -> b -> a + b;
sumarC.apply(3).apply(4);            // 7
```

### 11.3. Aplicación parcial

La ventaja de currificar es poder **fijar algunos argumentos** y obtener una función más específica, lista para reutilizar:

```java
Function<Integer, Integer> sumar10 = sumarC.apply(10);   // fija a = 10
sumar10.apply(5);                    // 15
sumar10.apply(-3);                   // 7

// Ejemplo práctico: un formateador configurable
Function<String, Function<String, String>> etiquetar = etiqueta -> texto -> "[" + etiqueta + "] " + texto;
Function<String, String> error = etiquetar.apply("ERROR");
Function<String, String> aviso = etiquetar.apply("AVISO");
error.apply("Disco lleno");          // "[ERROR] Disco lleno"
```

> 💡 En Java la currificación es menos natural que en lenguajes funcionales (los tipos se vuelven largos). En la práctica, el mismo efecto se consigue más claramente con un método que devuelve una función, como `descuento(int)` en la sección 9.2.

---

## 12. Evaluación perezosa

La **evaluación perezosa** (*lazy evaluation*) consiste en **no calcular un valor hasta que realmente se necesita**. Lo contrario es la evaluación **estricta** o **ansiosa** (*eager*), que es la habitual en Java: los argumentos de un método se calculan siempre antes de llamarlo.

### 12.1. Supplier: aplazar un cálculo

Un `Supplier<T>` envuelve un cálculo sin ejecutarlo; solo se ejecuta al llamar a `get()`:

```java
// Perezoso.java
String calcularCostoso() {
    IO.println("  (cálculo costoso ejecutado)");
    return "valor por defecto";
}

void main() {
    Optional<String> conValor = Optional.of("valor real");

    IO.println("orElse:");
    IO.println(conValor.orElse(calcularCostoso()));        // ansioso: siempre calcula

    IO.println("orElseGet:");
    IO.println(conValor.orElseGet(() -> calcularCostoso())); // perezoso: solo si hace falta
}
```

```text
orElse:
  (cálculo costoso ejecutado)
valor real
orElseGet:
valor real
```

Con `orElse` el valor por defecto se calcula **aunque no se use**. Con `orElseGet` solo se calcula si el `Optional` está vacío. Lo mismo ocurre con muchas API que aceptan un `Supplier`:

```java
Objects.requireNonNull(valor, () -> "Mensaje construido solo si falla: " + contexto);
logger.debug(() -> "Estado: " + objetoGrande.volcarTodo());     // Log4j 2: solo si DEBUG está activo
mapa.computeIfAbsent(clave, k -> crearValorCostoso(k));         // solo si falta la clave
```

### 12.2. Los streams son perezosos

Las operaciones intermedias de un `Stream` (`filter`, `map`...) **no hacen nada** hasta que se ejecuta una operación terminal. Y entonces procesan **elemento a elemento**, deteniéndose en cuanto tienen el resultado:

```java
// StreamPerezoso.java
void main() {
    Optional<Integer> primero = Stream.of(5, 12, 8, 30, 7)
        .peek(n -> IO.println("  revisando " + n))
        .filter(n -> n > 10)
        .findFirst();

    IO.println("Resultado: " + primero.orElseThrow());
}
```

```text
  revisando 5
  revisando 12
Resultado: 12
```

Solo se revisan **dos** elementos: en cuanto aparece el 12, `findFirst` tiene su respuesta.

### 12.3. Secuencias infinitas

Gracias a la pereza, se pueden describir secuencias **infinitas** y tomar solo lo necesario:

```java
// Fibonacci infinito; solo se calculan los 10 primeros
Stream.iterate(new long[] {0, 1}, f -> new long[] {f[1], f[0] + f[1]})
    .map(f -> f[0])
    .limit(10)
    .toList();                       // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Potencias de 2 menores que 1000 (iterate con condición, Java 9)
Stream.iterate(1, n -> n < 1000, n -> n * 2).toList();   // [1, 2, 4, ..., 512]
```

> ⚠️ Sin `limit`, `takeWhile` o una condición de parada, una operación terminal sobre un stream infinito **no termina nunca**.

---

## 13. Memoización

**Memoizar** es guardar el resultado de una función pura para no volver a calcularlo cuando se llama otra vez con los mismos argumentos. Solo es correcto con **funciones puras**: si el resultado dependiera de algo externo, la caché devolvería valores obsoletos.

```java
// Memoizar.java
<T, R> Function<T, R> memoizar(Function<T, R> funcion) {
    Map<T, R> cache = new ConcurrentHashMap<>();
    return entrada -> cache.computeIfAbsent(entrada, funcion);
}

int lenta(int n) {
    IO.println("  calculando para " + n + "...");
    return n * n;
}

void main() {
    Function<Integer, Integer> rapida = memoizar(this::lenta);

    IO.println(rapida.apply(4));
    IO.println(rapida.apply(4));     // ya está en caché
    IO.println(rapida.apply(5));
}
```

```text
  calculando para 4...
16
16
  calculando para 5...
25
```

`memoizar` es una función de orden superior: recibe una función y devuelve **otra** con el mismo comportamiento y una caché.

> ⚠️ No uses `computeIfAbsent` para memoizar una función **recursiva** (que se llama a sí misma desde dentro del cálculo): modificar el mapa mientras se está calculando una entrada lanza `ConcurrentModificationException` o `IllegalStateException`. Para esos casos, usa un bucle o un array de resultados (programación dinámica). También ten en cuenta que esta caché crece sin límite; en producción se usan bibliotecas como Caffeine.

---

## 14. Recursividad en Java

En los lenguajes funcionales puros no hay bucles: se usa **recursividad**. Muchos de ellos optimizan la **recursión de cola** (*tail call*) para que no consuma memoria de pila.

**Java no hace esa optimización.** Cada llamada recursiva ocupa espacio en la pila, y con miles de niveles se produce `StackOverflowError`:

```java
long sumarHasta(long n) {
    return n == 0 ? 0 : n + sumarHasta(n - 1);
}

sumarHasta(1_000);         // 500500
sumarHasta(1_000_000);     // 💥 StackOverflowError
```

En Java, para recorrer o acumular de forma funcional, usa **streams**:

```java
long suma = LongStream.rangeClosed(1, 1_000_000).sum();        // 500000500000, sin problemas de pila
```

La recursividad sigue siendo la mejor opción para **estructuras recursivas** de profundidad moderada, como árboles o expresiones, donde el número de niveles es pequeño (sección 15.2).

---

## 15. Datos inmutables y pattern matching

La programación funcional separa los **datos** (inmutables) de las **funciones** que operan sobre ellos. Java 21+ ofrece una combinación muy potente para este estilo:

- **`record`** para definir datos inmutables.
- **`sealed interface`** para decir «este dato es una de estas variantes, y ninguna más».
- **`switch` con patrones** para tratar cada variante, con comprobación de que no falta ninguna.

A esta combinación se la conoce como **tipos de datos algebraicos** (*algebraic data types*, ADT), un concepto clásico de los lenguajes funcionales.

### 15.1. Un ejemplo: figuras

```java
sealed interface Figura permits Circulo, Rectangulo, Triangulo { }
record Circulo(double radio) implements Figura { }
record Rectangulo(double base, double altura) implements Figura { }
record Triangulo(double base, double altura) implements Figura { }

double area(Figura f) {
    return switch (f) {
        case Circulo(double r) -> Math.PI * r * r;
        case Rectangulo(double b, double h) -> b * h;
        case Triangulo(double b, double h) -> b * h / 2;
    };       // exhaustivo: si se añade una figura, no compila hasta tratarla
}
```

`area` es una función **pura** que recibe datos **inmutables**. Los *records* no tienen comportamiento propio: la lógica está en funciones independientes, y se pueden añadir funciones nuevas (`perimetro`, `escalar`, `dibujar`) sin tocar los tipos.

### 15.2. Un ejemplo recursivo: expresiones aritméticas

Las estructuras recursivas (árboles) se modelan de forma natural:

```java
// Expresiones.java
sealed interface Expr permits Num, Suma, Mult, Neg { }
record Num(double valor) implements Expr { }
record Suma(Expr izq, Expr der) implements Expr { }
record Mult(Expr izq, Expr der) implements Expr { }
record Neg(Expr expr) implements Expr { }

double evaluar(Expr e) {
    return switch (e) {
        case Num(double v) -> v;
        case Suma(Expr a, Expr b) -> evaluar(a) + evaluar(b);
        case Mult(Expr a, Expr b) -> evaluar(a) * evaluar(b);
        case Neg(Expr x) -> -evaluar(x);
    };
}

String mostrar(Expr e) {
    return switch (e) {
        case Num(double v) -> String.valueOf(v);
        case Suma(Expr a, Expr b) -> "(" + mostrar(a) + " + " + mostrar(b) + ")";
        case Mult(Expr a, Expr b) -> mostrar(a) + " * " + mostrar(b);
        case Neg(Expr x) -> "-" + mostrar(x);
    };
}

// Simplificar devuelve una expresión NUEVA: no modifica la original
Expr simplificar(Expr e) {
    return switch (e) {
        case Mult(Num(double v), Expr _) when v == 0 -> new Num(0);
        case Mult(Num(double v), Expr x) when v == 1 -> simplificar(x);
        case Suma(Expr a, Num(double v)) when v == 0 -> simplificar(a);
        case Suma(Expr a, Expr b) -> new Suma(simplificar(a), simplificar(b));
        case Mult(Expr a, Expr b) -> new Mult(simplificar(a), simplificar(b));
        case Neg(Neg(Expr x)) -> simplificar(x);
        case Neg(Expr x) -> new Neg(simplificar(x));
        case Num n -> n;
    };
}

void main() {
    // (2 + 3) * 4
    Expr e = new Mult(new Suma(new Num(2), new Num(3)), new Num(4));
    IO.println(mostrar(e) + " = " + evaluar(e));

    // 1 * (x + 0) con x = -(-5)
    Expr f = new Mult(new Num(1), new Suma(new Neg(new Neg(new Num(5))), new Num(0)));
    IO.println(mostrar(f) + "  →  " + mostrar(simplificar(f)));
}
```

```text
(2.0 + 3.0) * 4.0 = 20.0
1.0 * (--5.0 + 0.0)  →  5.0
```

Observa los **patrones anidados** (`Mult(Num(double v), Expr x)`), las **guardas** (`when v == 1`) y `_` para lo que no interesa. Más sobre `switch` con patrones en [Fundamentos de Java](<../01-Fundamentos-del-lenguaje/Fundamentos de Java.md>) y sobre tipos sellados en [Herencia, encapsulación y polimorfismo](<../01-Fundamentos-del-lenguaje/Herencia, encapsulacion y polimorfismo.md>).

---

## 16. Optional: ausencia de valor sin null

`null` obliga a comprobar constantemente y, si se olvida, provoca `NullPointerException`. `Optional<T>` es un **contenedor** que puede tener un valor o estar vacío, y obliga a quien lo recibe a decidir qué hacer en cada caso.

Lo interesante desde el punto de vista funcional es que `Optional` se **transforma** con funciones, igual que un stream de cero o un elemento:

```java
record Direccion(String ciudad) { }
record Cliente(String nombre, Optional<Direccion> direccion) { }

Optional<Cliente> buscarCliente(String id) { ... }

// ❌ Estilo imperativo con null
String ciudad = "desconocida";
Cliente c = buscarClienteONull(id);
if (c != null && c.direccion() != null && c.direccion().ciudad() != null) {
    ciudad = c.direccion().ciudad().toUpperCase();
}

// ✅ Estilo funcional con Optional
String ciudad = buscarCliente(id)
    .flatMap(Cliente::direccion)          // Optional<Direccion>
    .map(Direccion::ciudad)               // Optional<String>
    .map(String::toUpperCase)
    .orElse("desconocida");
```

| Método | Qué hace |
|---|---|
| `map(f)` | Si hay valor, le aplica `f`; si no, sigue vacío |
| `flatMap(f)` | Como `map`, cuando `f` ya devuelve un `Optional` (evita `Optional<Optional<T>>`) |
| `filter(p)` | Se queda vacío si el valor no cumple `p` |
| `orElse(v)` / `orElseGet(s)` | Valor por defecto (ansioso / perezoso) |
| `orElseThrow()` | Lanza `NoSuchElementException` si está vacío |
| `ifPresentOrElse(c, r)` | Ejecuta una acción u otra |
| `or(s)` | Si está vacío, prueba con otro `Optional` |
| `stream()` | Convierte en un stream de 0 o 1 elementos |

Buenas prácticas: úsalo como **tipo de retorno** de métodos que pueden no encontrar resultado; no lo uses para atributos, parámetros ni colecciones (una lista vacía ya expresa «no hay nada»), y **no llames a `get()`** sin comprobar. Más detalles en [Lambdas y Streams](<Lambdas y Streams.md>), sección 17.

---

## 17. Errores como valores: el tipo Resultado

Las excepciones interrumpen el flujo normal: una función que lanza excepciones no es pura y su firma no dice qué puede fallar. Un enfoque funcional es **devolver el error como un valor**, con un tipo que represente «éxito o fallo».

Java no trae este tipo, pero se define en pocas líneas con una interfaz sellada:

```java
// Resultados.java
sealed interface Resultado<T> permits Exito, Fallo {

    // Transforma el valor si es un éxito; un fallo se propaga tal cual
    default <R> Resultado<R> map(Function<T, R> f) {
        return switch (this) {
            case Exito<T>(T valor) -> new Exito<>(f.apply(valor));
            case Fallo<T>(String error) -> new Fallo<>(error);
        };
    }

    // Encadena otra operación que también puede fallar
    default <R> Resultado<R> flatMap(Function<T, Resultado<R>> f) {
        return switch (this) {
            case Exito<T>(T valor) -> f.apply(valor);
            case Fallo<T>(String error) -> new Fallo<>(error);
        };
    }
}

record Exito<T>(T valor) implements Resultado<T> { }
record Fallo<T>(String error) implements Resultado<T> { }

Resultado<Integer> parsear(String texto) {
    try {
        return new Exito<>(Integer.parseInt(texto.strip()));
    } catch (NumberFormatException e) {
        return new Fallo<>("'" + texto + "' no es un número");
    }
}

Resultado<Integer> validarEdad(int edad) {
    return edad >= 0 && edad <= 120
        ? new Exito<>(edad)
        : new Fallo<>("edad fuera de rango: " + edad);
}

String describir(Resultado<?> r) {
    return switch (r) {
        case Exito<?>(var valor) -> "✅ " + valor;
        case Fallo<?>(var error) -> "❌ " + error;
    };
}

void main() {
    for (String entrada : List.of("42", " 7 ", "abc", "200")) {
        Resultado<String> r = parsear(entrada)
            .flatMap(this::validarEdad)
            .map(edad -> edad >= 18 ? "adulto (" + edad + ")" : "menor (" + edad + ")");
        IO.println(describir(r));
    }
}
```

```text
✅ adulto (42)
✅ menor (7)
❌ 'abc' no es un número
❌ edad fuera de rango: 200
```

Ventajas:

- La **firma** de `parsear` dice que puede fallar; el compilador obliga a tratar ambos casos en el `switch`.
- Las operaciones se **encadenan** con `map` y `flatMap`; en cuanto una falla, las siguientes se saltan automáticamente.
- Se pueden **acumular** resultados en una lista y procesar todos los errores juntos (por ejemplo, validar un formulario completo).

**¿Cuándo usar excepciones y cuándo un tipo resultado?**

| Situación | Recomendación |
|---|---|
| Errores **esperables** del negocio (dato no válido, saldo insuficiente) | Un tipo resultado, o `Optional` si basta con «hay o no hay» |
| Errores **excepcionales** o de programación (fichero corrupto, `null` inesperado, red caída) | Excepciones |
| Código que se integra con frameworks que esperan excepciones | Excepciones |

Más sobre excepciones en [Manejo eficaz de errores en Java](<../06-Excepciones/Manejo Eficaz de Errores en Java - Estrategias y Buenas Practicas.md>).

---

## 18. Streams con estilo funcional

Un `Stream` es la herramienta funcional por excelencia en Java. Sus tres operaciones fundamentales son las mismas que en cualquier lenguaje funcional:

| Operación | Idea | Ejemplo |
|---|---|---|
| **`filter`** | Quedarse con los elementos que cumplen una condición | `filter(p -> p.precio() > 10)` |
| **`map`** | Transformar cada elemento | `map(Producto::nombre)` |
| **`reduce`** (*fold*) | Combinar todos los elementos en un único valor | `reduce(0, Integer::sum)` |

`collect`, `count`, `sum`, `max`, `groupingBy`... son formas especializadas de reducción.

### 18.1. reduce: plegar una colección

```java
List<Integer> numeros = List.of(3, 1, 4, 1, 5, 9);

int suma = numeros.stream().reduce(0, Integer::sum);          // 23
int producto = numeros.stream().reduce(1, (a, b) -> a * b);   // 540
Optional<Integer> max = numeros.stream().reduce(Integer::max); // Optional[9]
```

```text
reduce(0, Integer::sum)
   0 + 3 = 3
   3 + 1 = 4
   4 + 4 = 8
   ...           = 23
```

El primer argumento es el **valor neutro** (el que no altera el resultado: `0` para sumar, `1` para multiplicar). Sin él, `reduce` devuelve un `Optional`, porque el stream podría estar vacío.

### 18.2. No uses efectos secundarios dentro de un stream

El error más habitual al empezar con streams es usarlos como un bucle disfrazado:

```java
// ❌ Efecto secundario: modifica una lista externa
List<String> nombres = new ArrayList<>();
personas.stream()
    .filter(p -> p.edad() >= 18)
    .forEach(p -> nombres.add(p.nombre()));

// ✅ Sin efectos: el stream produce el resultado
List<String> nombres = personas.stream()
    .filter(p -> p.edad() >= 18)
    .map(Persona::nombre)
    .toList();
```

La primera versión funciona en secuencial, pero **falla de forma impredecible en paralelo** (sección 19) y oculta la intención. Reglas:

- Las lambdas de `filter`, `map`, `sorted`, etc. deben ser **puras**.
- `forEach` solo para el **efecto final** (mostrar, guardar), nunca para construir resultados.
- `peek` solo para **depurar**.

### 18.3. Gatherers (Java 24+)

Los **gatherers** permiten crear operaciones intermedias que antes no existían, como agrupar en ventanas o acumular resultados parciales. La clase `Gatherers` trae varios listos para usar:

```java
// Ventanas.java
void main() {
    List<Integer> temperaturas = List.of(18, 21, 19, 25, 27, 22, 20);

    // Grupos fijos de 3
    IO.println(temperaturas.stream().gather(Gatherers.windowFixed(3)).toList());

    // Ventana deslizante de 3: media móvil
    IO.println(temperaturas.stream()
        .gather(Gatherers.windowSliding(3))
        .map(v -> v.stream().mapToInt(Integer::intValue).average().orElseThrow())
        .map(m -> Math.round(m * 10) / 10.0)
        .toList());

    // Acumulado: como reduce, pero emitiendo cada resultado parcial
    IO.println(temperaturas.stream().gather(Gatherers.scan(() -> 0, Integer::sum)).toList());
}
```

```text
[[18, 21, 19], [25, 27, 22], [20]]
[19.3, 21.7, 23.7, 24.7, 23.0]
[18, 39, 58, 83, 110, 132, 152]
```

> 📚 Todas las operaciones de `Stream`, los `Collectors` y los streams de primitivos están explicados en [Lambdas y Streams](<Lambdas y Streams.md>), secciones 11 a 16.

---

## 19. Paralelismo y funciones puras

Las funciones puras y los datos inmutables permiten **paralelizar** con seguridad: si ninguna operación modifica estado compartido, da igual en qué orden o en qué hilo se ejecuten.

```java
// Paralelo.java
void main() {
    // ❌ Efecto secundario en paralelo: ArrayList no es segura entre hilos
    List<Integer> mal = new ArrayList<>();
    try {
        IntStream.range(0, 10_000).parallel().forEach(mal::add);
        IO.println("Con efectos: " + mal.size() + " elementos (deberían ser 10000)");
    } catch (RuntimeException e) {
        IO.println("Con efectos: falló con " + e.getClass().getSimpleName());
    }

    // ✅ Sin efectos: el stream construye el resultado
    List<Integer> bien = IntStream.range(0, 10_000).parallel().boxed().toList();
    IO.println("Sin efectos: " + bien.size() + " elementos");
}
```

Una salida posible (varía en cada ejecución, y ese es precisamente el problema):

```text
Con efectos: 9467 elementos (deberían ser 10000)
Sin efectos: 10000 elementos
```

La primera versión puede **perder elementos**, lanzar `ArrayIndexOutOfBoundsException` o, a veces, funcionar, lo que hace el error muy difícil de detectar. La segunda es correcta siempre.

> ⚠️ `parallel()` no siempre acelera: para pocos elementos o tareas rápidas suele ser **más lento** por el coste de coordinar hilos. Mide antes de usarlo. Más en [Lambdas y Streams](<Lambdas y Streams.md>), secciones 18 y 19.

---

## 20. Cuándo usar el estilo funcional

| Úsalo para... | Mejor otro enfoque para... |
|---|---|
| Transformar, filtrar, agrupar y resumir colecciones | Bucles con lógica compleja de control (varios `break`, índices cruzados) |
| Lógica de negocio: cálculos, validaciones, reglas | Código que es esencialmente entrada/salida |
| Configurar comportamiento (estrategias, criterios, callbacks) | Algoritmos que dependen de modificar estructuras in situ por rendimiento |
| Modelar datos con variantes cerradas (`sealed` + `record`) | Jerarquías abiertas donde cada clase aporta su comportamiento |
| Código concurrente | Recursividad profunda (Java no la optimiza) |

Señales de que el estilo funcional **se está forzando**:

- Lambdas de más de 3 o 4 líneas: extráelas a un **método con nombre** y usa una referencia (`this::calcularDescuento`).
- Cadenas de streams tan largas que no caben en pantalla: divídelas en pasos con variables intermedias de nombre claro.
- `Function<A, Function<B, Function<C, D>>>`: probablemente un método normal sea más legible.
- Un `reduce` complicado que nadie entiende: un bucle puede ser más claro.

> 💡 La legibilidad manda. El objetivo no es «usar programación funcional», sino escribir código **correcto, claro y fácil de cambiar**. El estilo funcional es una herramienta excelente para conseguirlo, no un fin en sí mismo. Ver también [Principio KISS](<../07-Diseno/Principios/Principio KISS.md>).

---

## 21. Ejemplo completo: procesar pedidos

Este programa aplica las ideas de la guía:

- **Datos inmutables:** *records* y listas inmutables.
- **Tipos sellados** para los descuentos, con `switch` exhaustivo.
- **Funciones puras** para validar, calcular y resumir; los efectos (`IO.println`) quedan en `main`.
- **Funciones de orden superior** y **composición** de validaciones.
- **Errores como valores** con un tipo `Resultado`.
- **Streams** sin efectos secundarios y `Collectors` para agrupar.

```java
// Pedidos.java

// ── Datos ────────────────────────────────────────────────────
record Linea(String producto, String categoria, int cantidad, double precioUnitario) {
    double subtotal() { return cantidad * precioUnitario; }
}

sealed interface Descuento permits SinDescuento, Porcentaje, Fijo { }
record SinDescuento() implements Descuento { }
record Porcentaje(int valor) implements Descuento { }
record Fijo(double importe) implements Descuento { }

record Pedido(String id, String cliente, List<Linea> lineas, Descuento descuento) {
    Pedido {
        lineas = List.copyOf(lineas);           // copia defensiva: inmutable
    }
}

sealed interface Resultado<T> permits Aceptado, Rechazado { }
record Aceptado<T>(T valor) implements Resultado<T> { }
record Rechazado<T>(String motivo) implements Resultado<T> { }

// ── Validación: funciones que se componen ────────────────────
record Regla(String mensaje, Predicate<Pedido> cumple) { }

List<Regla> reglas = List.of(
    new Regla("el pedido no tiene líneas", p -> !p.lineas().isEmpty()),
    new Regla("hay cantidades no positivas", p -> p.lineas().stream().allMatch(l -> l.cantidad() > 0)),
    new Regla("el descuento supera el 50 %", p -> !(p.descuento() instanceof Porcentaje(int v) && v > 50))
);

Resultado<Pedido> validar(Pedido pedido) {
    return reglas.stream()
        .filter(r -> !r.cumple().test(pedido))
        .findFirst()
        .<Resultado<Pedido>>map(r -> new Rechazado<>(pedido.id() + ": " + r.mensaje()))
        .orElseGet(() -> new Aceptado<>(pedido));
}

// ── Cálculos puros ───────────────────────────────────────────
double bruto(Pedido p) {
    return p.lineas().stream().mapToDouble(Linea::subtotal).sum();
}

Function<Double, Double> aplicar(Descuento d) {
    return switch (d) {
        case SinDescuento _ -> Function.identity();
        case Porcentaje(int v) -> importe -> importe * (100 - v) / 100;
        case Fijo(double f) -> importe -> Math.max(0, importe - f);
    };
}

Function<Double, Double> conIva = importe -> importe * 1.21;
Function<Double, Double> redondear = importe -> Math.round(importe * 100) / 100.0;

double total(Pedido p) {
    return aplicar(p.descuento()).andThen(conIva).andThen(redondear).apply(bruto(p));
}

Map<String, Double> ventasPorCategoria(List<Pedido> pedidos) {
    return pedidos.stream()
        .flatMap(p -> p.lineas().stream())
        .collect(Collectors.groupingBy(Linea::categoria, TreeMap::new,
                 Collectors.summingDouble(Linea::subtotal)));
}

// ── Capa imperativa: datos de entrada y salida ───────────────
void main() {
    List<Pedido> pedidos = List.of(
        new Pedido("P-1", "Ana", List.of(
            new Linea("Teclado", "Periféricos", 1, 49.90),
            new Linea("Ratón", "Periféricos", 2, 19.95)), new Porcentaje(10)),
        new Pedido("P-2", "Bruno", List.of(
            new Linea("Monitor", "Pantallas", 1, 189.00)), new Fijo(20)),
        new Pedido("P-3", "Carla", List.of(), new SinDescuento()),
        new Pedido("P-4", "David", List.of(
            new Linea("Cable HDMI", "Accesorios", 3, 7.50)), new Porcentaje(70)),
        new Pedido("P-5", "Elena", List.of(
            new Linea("Portátil", "Ordenadores", 1, 899.00),
            new Linea("Funda", "Accesorios", 1, 25.00)), new SinDescuento())
    );

    Map<Boolean, List<Resultado<Pedido>>> resultados = pedidos.stream()
        .map(this::validar)
        .collect(Collectors.partitioningBy(r -> r instanceof Aceptado));

    List<Pedido> validos = resultados.get(true).stream()
        .map(r -> ((Aceptado<Pedido>) r).valor())
        .toList();

    IO.println("=== Pedidos válidos ===");
    validos.forEach(p -> IO.println("%-4s %-6s %8.2f €".formatted(p.id(), p.cliente(), total(p))));

    IO.println("\n=== Pedidos rechazados ===");
    resultados.get(false).forEach(r -> {
        if (r instanceof Rechazado<Pedido>(String motivo)) IO.println("- " + motivo);
    });

    IO.println("\n=== Ventas por categoría (sin IVA ni descuentos) ===");
    ventasPorCategoria(validos).forEach((cat, importe) ->
        IO.println("%-12s %8.2f €".formatted(cat, importe)));

    double facturado = validos.stream().mapToDouble(this::total).sum();
    IO.println("\nTotal facturado: %.2f €".formatted(facturado));
}
```

```text
=== Pedidos válidos ===
P-1  Ana       97,79 €
P-2  Bruno    204,49 €
P-5  Elena   1118,04 €

=== Pedidos rechazados ===
- P-3: el pedido no tiene líneas
- P-4: el descuento supera el 50 %

=== Ventas por categoría (sin IVA ni descuentos) ===
Accesorios      25,00 €
Ordenadores    899,00 €
Pantallas      189,00 €
Periféricos     89,80 €

Total facturado: 1420,32 €
```

**Para pensar:**

- Añade un descuento `DosPorUno(String producto)`. ¿Qué te dice el compilador? (Que `aplicar` no cubre todos los casos; además, este descuento necesitaría las líneas, no solo el importe: ¿cómo cambiarías la firma?).
- Añade una regla nueva («el pedido supera 5 000 €»). Solo hay que añadir un elemento a `reglas`.
- `total`, `bruto`, `validar` y `ventasPorCategoria` son puras: escribe una prueba con JUnit para cada una (ver [JUnit: aprende a escribir pruebas paso a paso](<../08-Testing/JUnit - Aprende a escribir pruebas paso a paso.md>)).

---

## 22. Errores frecuentes

| Error | Síntoma | Solución |
|---|---|---|
| Modificar una variable externa desde una lambda | `local variables referenced from a lambda expression must be final or effectively final` | Devuelve el resultado con `count`, `reduce`, `collect` o `toList` |
| Usar `forEach` para construir una colección | Código frágil; resultados erróneos en paralelo | `map` + `toList()` o `collect` |
| Esquivar la restricción con un array o `AtomicInteger` | Funciona, pero reintroduce estado mutable | Replantea con una operación de reducción |
| Modificar la colección de origen dentro de un stream | `ConcurrentModificationException` | Genera una colección nueva |
| Intentar modificar `List.of` o `toList()` | `UnsupportedOperationException` | Crea una copia mutable (`new ArrayList<>(lista)`) solo si de verdad la necesitas |
| Creer que `unmodifiableList` es inmutable | La «copia» cambia cuando cambia el original | `List.copyOf` |
| `orElse(calculoCostoso())` | El cálculo se ejecuta aunque haya valor | `orElseGet(() -> calculoCostoso())` |
| `optional.get()` sin comprobar | `NoSuchElementException` | `orElse`, `orElseThrow`, `map`, `ifPresentOrElse` |
| Funciones con efectos ocultos (consola, hora, aleatorios) | Pruebas que fallan a veces | Pasa la hora, el `Random` o los datos como parámetro |
| Recursión profunda | `StackOverflowError` | Streams o bucles |
| Memoizar una función impura | Resultados obsoletos | Solo se memoizan funciones puras |
| Stream infinito sin límite | El programa no termina | `limit`, `takeWhile` o `iterate` con condición |
| Lambdas largas y anidadas | Código ilegible | Extrae métodos con nombre y usa referencias a métodos |
| `parallel()` por defecto | Más lento o resultados incorrectos | Solo con funciones puras, muchos datos y tras medir |

---

## 23. Chuleta rápida

```java
// ── Interfaces funcionales ───────────────────────────────
Function<T, R>        f.apply(t)       // transforma
BiFunction<T, U, R>   f.apply(t, u)
UnaryOperator<T>      f.apply(t)       // T → T
BinaryOperator<T>     f.apply(a, b)    // (T, T) → T
Predicate<T>          p.test(t)        // condición
Supplier<T>           s.get()          // produce (perezoso)
Consumer<T>           c.accept(t)      // efecto final

// ── Composición ──────────────────────────────────────────
f.andThen(g)          // g(f(x))
f.compose(g)          // f(g(x))
Function.identity()   // x -> x
p.and(q)  p.or(q)  p.negate()  Predicate.not(p)
Comparator.comparing(A::x).thenComparing(A::y).reversed()

// ── Orden superior ───────────────────────────────────────
<T> List<T> filtrar(List<T> xs, Predicate<T> p) { ... }    // recibe función
Function<Double, Double> descuento(int pct) {              // devuelve función
    return precio -> precio * (100 - pct) / 100;
}

// ── Inmutabilidad ────────────────────────────────────────
record Punto(int x, int y) { Punto conX(int nx) { return new Punto(nx, y); } }
List.of(...)   List.copyOf(lista)   stream.toList()

// ── Datos + pattern matching ─────────────────────────────
sealed interface Forma permits Circulo, Cuadrado { }
double area(Forma f) {
    return switch (f) {
        case Circulo(double r) -> Math.PI * r * r;
        case Cuadrado(double l) -> l * l;
    };
}

// ── Optional ─────────────────────────────────────────────
opt.map(f).flatMap(g).filter(p).orElseGet(() -> porDefecto)

// ── Streams ──────────────────────────────────────────────
xs.stream().filter(p).map(f).toList()
xs.stream().reduce(0, Integer::sum)
xs.stream().collect(Collectors.groupingBy(A::clave))
xs.stream().gather(Gatherers.windowFixed(3)).toList()       // Java 24+
Stream.iterate(1, n -> n < 1000, n -> n * 2).toList()       // perezoso
```

**Regla de oro:** funciones pequeñas y puras + datos inmutables + efectos en los bordes.

---

## 24. Ejercicios propuestos

1. **¿Pura o impura?** Indica cuáles de estas funciones son puras y, para las impuras, cómo las harías puras:
   ```java
   double areaCirculo(double r) { return Math.PI * r * r; }
   String saludo() { return "Buenos días, son las " + LocalTime.now(); }
   List<Integer> duplicar(List<Integer> xs) { xs.replaceAll(x -> x * 2); return xs; }
   int longitudTotal(List<String> xs) { return xs.stream().mapToInt(String::length).sum(); }
   ```
2. **Del bucle al stream.** Reescribe en estilo declarativo: dada una lista de palabras, obtener las de más de 4 letras, sin repetir, en minúsculas y ordenadas por longitud y después alfabéticamente.
3. **Orden superior.** Escribe `<T> int contarSi(List<T> xs, Predicate<T> p)` y úsalo para contar los pares de una lista de enteros y las palabras que empiezan por mayúscula en una lista de cadenas.
4. **Fábrica de funciones.** Escribe `Function<Double, Double> convertir(double factor, double desplazamiento)` que devuelva `x -> x * factor + desplazamiento`. Úsala para crear `celsiusAFahrenheit` (factor 1,8 y desplazamiento 32) y `kmAMillas` (factor 0,621371 y desplazamiento 0).
5. **Composición de validaciones.** Con `Predicate<String>`, construye un validador de contraseñas combinando: al menos 8 caracteres, al menos un dígito, al menos una mayúscula y sin espacios. Pruébalo con varias contraseñas.
6. **Record con métodos «with».** Crea `record Cuenta(String titular, double saldo)` con `ingresar(double)` y `retirar(double)` que devuelvan una **nueva** `Cuenta`. `retirar` debe devolver un `Optional<Cuenta>` vacío si no hay saldo suficiente.
7. **Reduce.** Con `reduce`, calcula: (a) la palabra más larga de una lista, (b) la concatenación de todas separadas por comas, (c) el máximo común divisor de una lista de enteros.
8. **Tipos sellados.** Define `sealed interface Vehiculo` con `Bici()`, `Coche(int plazas)` y `Bus(int plazas, boolean articulado)`. Escribe `double tarifaPeaje(Vehiculo v)`: bici 0 €, coche 3 €, bus 8 € (12 € si es articulado). Hazlo con patrones de *record* y guardas `when`.

---

## 25. Soluciones

Hay muchas soluciones correctas; estas son una propuesta.

<details>
<summary>1. ¿Pura o impura?</summary>

- `areaCirculo`: **pura**.
- `saludo`: **impura**, depende de la hora actual. Versión pura: `String saludo(LocalTime hora) { return "Buenos días, son las " + hora; }`.
- `duplicar`: **impura**, modifica la lista que recibe. Versión pura: `return xs.stream().map(x -> x * 2).toList();`.
- `longitudTotal`: **pura**.

</details>

<details>
<summary>2. Del bucle al stream</summary>

```java
List<String> procesar(List<String> palabras) {
    return palabras.stream()
        .map(String::toLowerCase)
        .filter(p -> p.length() > 4)
        .distinct()
        .sorted(Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder()))
        .toList();
}
// procesar(List.of("Casa", "árbol", "Montaña", "río", "ÁRBOL", "puente", "lago"))
// → [árbol, puente, montaña]
```

Se pasa a minúsculas **antes** de `distinct` para que «árbol» y «ÁRBOL» se consideren repetidas.

</details>

<details>
<summary>3. Orden superior</summary>

```java
<T> int contarSi(List<T> xs, Predicate<T> p) {
    return (int) xs.stream().filter(p).count();
}

void main() {
    IO.println(contarSi(List.of(1, 2, 3, 4, 5, 6), n -> n % 2 == 0));                       // 3
    IO.println(contarSi(List.of("Ana", "pera", "Luis", "uva"),
                        s -> !s.isEmpty() && Character.isUpperCase(s.charAt(0))));          // 2
}
```

</details>

<details>
<summary>4. Fábrica de funciones</summary>

```java
Function<Double, Double> convertir(double factor, double desplazamiento) {
    return x -> x * factor + desplazamiento;
}

void main() {
    Function<Double, Double> celsiusAFahrenheit = convertir(1.8, 32);
    Function<Double, Double> kmAMillas = convertir(0.621371, 0);
    IO.println(celsiusAFahrenheit.apply(100.0));     // 212.0
    IO.println(kmAMillas.apply(42.195));             // 26.218749345
}
```

</details>

<details>
<summary>5. Composición de validaciones</summary>

```java
Predicate<String> largaSuficiente = s -> s.length() >= 8;
Predicate<String> tieneDigito = s -> s.chars().anyMatch(Character::isDigit);
Predicate<String> tieneMayuscula = s -> s.chars().anyMatch(Character::isUpperCase);
Predicate<String> sinEspacios = s -> s.chars().noneMatch(Character::isWhitespace);

Predicate<String> contrasenaValida =
    largaSuficiente.and(tieneDigito).and(tieneMayuscula).and(sinEspacios);

void main() {
    for (String c : List.of("corta1A", "sinmayuscula1", "SinDigitos", "Con Espacio1", "Segura2026")) {
        IO.println(c + " → " + contrasenaValida.test(c));
    }
}
// corta1A → false · sinmayuscula1 → false · SinDigitos → false · Con Espacio1 → false · Segura2026 → true
```

Mejora: guarda cada predicado en un `record Regla(String mensaje, Predicate<String> p)` para poder decir **qué** regla falla, como en el ejemplo de la sección 21.

</details>

<details>
<summary>6. Record con métodos «with»</summary>

```java
record Cuenta(String titular, double saldo) {
    Cuenta ingresar(double importe) {
        if (importe <= 0) throw new IllegalArgumentException("Importe no válido");
        return new Cuenta(titular, saldo + importe);
    }

    Optional<Cuenta> retirar(double importe) {
        return importe <= saldo
            ? Optional.of(new Cuenta(titular, saldo - importe))
            : Optional.empty();
    }
}

void main() {
    var cuenta = new Cuenta("Ana", 100).ingresar(50);
    IO.println(cuenta);                                       // Cuenta[titular=Ana, saldo=150.0]
    IO.println(cuenta.retirar(30));                           // Optional[Cuenta[titular=Ana, saldo=120.0]]
    IO.println(cuenta.retirar(500));                          // Optional.empty
    IO.println(cuenta.retirar(500).map(Cuenta::saldo).orElse(cuenta.saldo()));   // 150.0
}
```

</details>

<details>
<summary>7. Reduce</summary>

```java
List<String> palabras = List.of("sol", "montaña", "río", "mar");

String masLarga = palabras.stream()
    .reduce((a, b) -> a.length() >= b.length() ? a : b)
    .orElse("");                                             // "montaña"

String unidas = palabras.stream()
    .reduce((a, b) -> a + ", " + b)
    .orElse("");                                             // "sol, montaña, río, mar"
// (En la práctica, mejor: String.join(", ", palabras) o Collectors.joining(", "))

int mcd(int a, int b) { return b == 0 ? a : mcd(b, a % b); }

int mcdLista = Stream.of(24, 36, 60).reduce(this::mcd).orElseThrow();   // 12
```

</details>

<details>
<summary>8. Tipos sellados</summary>

```java
sealed interface Vehiculo permits Bici, Coche, Bus { }
record Bici() implements Vehiculo { }
record Coche(int plazas) implements Vehiculo { }
record Bus(int plazas, boolean articulado) implements Vehiculo { }

double tarifaPeaje(Vehiculo v) {
    return switch (v) {
        case Bici _ -> 0;
        case Coche _ -> 3;
        case Bus(_, boolean articulado) when articulado -> 12;
        case Bus _ -> 8;
    };
}

void main() {
    List<Vehiculo> trafico = List.of(new Bici(), new Coche(5), new Bus(50, false), new Bus(90, true));
    IO.println(trafico.stream().mapToDouble(this::tarifaPeaje).sum());   // 23.0
}
```

</details>

---

## 26. Glosario

| Término | Definición |
|---|---|
| **Aplicación parcial** | Fijar algunos argumentos de una función para obtener otra con menos parámetros |
| **Clausura** (*closure*) | Función que recuerda las variables del contexto donde se creó |
| **Composición** | Combinar funciones de modo que la salida de una sea la entrada de la siguiente |
| **Currificación** | Convertir una función de varios parámetros en una cadena de funciones de un parámetro |
| **Declarativo** | Estilo que describe qué resultado se quiere, no los pasos para obtenerlo |
| **Efecto secundario** | Cualquier cambio observable fuera de la función: modificar variables o argumentos, E/S, excepciones... |
| ***Effectively final*** | Variable que no se reasigna después de inicializarse, aunque no lleve `final` |
| **Evaluación perezosa** | Calcular un valor solo cuando se necesita |
| **Función de orden superior** | Función que recibe o devuelve otras funciones |
| **Función de primera clase** | Función que se puede tratar como un valor: guardar, pasar y devolver |
| **Función pura** | Función determinista y sin efectos secundarios |
| ***Gatherer*** | Operación intermedia personalizada de un stream (Java 24+) |
| **Imperativo** | Estilo que describe paso a paso cómo modificar el estado para llegar al resultado |
| **Inmutable** | Dato que no puede cambiar después de crearse |
| **Interfaz funcional** | Interfaz con un único método abstracto; se puede implementar con una lambda |
| **Memoización** | Guardar los resultados de una función pura para no recalcularlos |
| ***Reduce* / *fold*** | Combinar todos los elementos de una colección en un único valor |
| **Tipo de datos algebraico** | Tipo formado por un conjunto cerrado de variantes; en Java, `sealed` + `record` |
| **Transparencia referencial** | Propiedad de una expresión que se puede sustituir por su valor sin cambiar el programa |

---

## 27. Referencias

- [Dev.java – Lambda Expressions](https://dev.java/learn/lambdas/) y [The Stream API](https://dev.java/learn/api/streams/): tutoriales oficiales actualizados.
- Documentación de la API de Java SE 25: [`java.util.function`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/function/package-summary.html), [`java.util.stream`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/package-summary.html), [`Optional`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Optional.html) y [`Gatherers`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/Gatherers.html).
- [JEP 395: Records](https://openjdk.org/jeps/395), [JEP 409: Sealed Classes](https://openjdk.org/jeps/409), [JEP 440: Record Patterns](https://openjdk.org/jeps/440), [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) y [JEP 485: Stream Gatherers](https://openjdk.org/jeps/485).
- Brian Goetz, [*Data Oriented Programming in Java*](https://www.infoq.com/articles/data-oriented-programming-java/) (InfoQ): cómo combinar *records*, tipos sellados y patrones.
- Gary Bernhardt, [*Functional Core, Imperative Shell*](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell): el patrón de la sección 5.
- Venkat Subramaniam, *Functional Programming in Java* (2.ª ed., Pragmatic Bookshelf).
- Pierre-Yves Saumont, *Functional Programming in Java* (Manning).
