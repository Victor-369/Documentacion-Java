# Genéricos (Generics) en Java: guía para programadores

> **Público:** programadores en Java.
> **Versión de referencia:** Java 25 (LTS) en adelante.
> **Fuentes:** Baeldung ([The Basics of Java Generics](https://www.baeldung.com/java-generics)), W3Schools ([Java Generics](https://www.w3schools.com/java/java_generics.asp)) y The Java Tutorials de Oracle ([Lesson: Generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html)).

Este documento es un resumen propio y reorganizado de las tres fuentes, con ejemplos adaptados a Java moderno. Donde las fuentes usan código antiguo (por ejemplo, `new Integer(10)`), aquí se indica la alternativa actual.

---

## Índice

1. [¿Qué son los genéricos y por qué usarlos?](#1-qué-son-los-genéricos-y-por-qué-usarlos)
2. [Clases e interfaces genéricas](#2-clases-e-interfaces-genéricas)
3. [Tipos crudos (*raw types*)](#3-tipos-crudos-raw-types)
4. [Métodos genéricos](#4-métodos-genéricos)
5. [Inferencia de tipos](#5-inferencia-de-tipos)
6. [Tipos acotados (*bounded types*)](#6-tipos-acotados-bounded-types)
7. [Genéricos, herencia y subtipos](#7-genéricos-herencia-y-subtipos)
8. [Comodines (*wildcards*)](#8-comodines-wildcards)
9. [Borrado de tipos (*type erasure*)](#9-borrado-de-tipos-type-erasure)
10. [Genéricos y tipos primitivos](#10-genéricos-y-tipos-primitivos)
11. [Restricciones de los genéricos](#11-restricciones-de-los-genéricos)
12. [Genéricos con características modernas de Java](#12-genéricos-con-características-modernas-de-java)
13. [Errores habituales y buenas prácticas](#13-errores-habituales-y-buenas-prácticas)
14. [Chuleta rápida](#14-chuleta-rápida)
15. [Ejercicios propuestos](#15-ejercicios-propuestos)
16. [Referencias](#16-referencias)

---

## 1. ¿Qué son los genéricos y por qué usarlos?

Los genéricos permiten que **los tipos (clases e interfaces) sean parámetros** al definir clases, interfaces y métodos. Es parecido a los parámetros de un método, con una diferencia clave: los parámetros de un método reciben **valores**; los parámetros de tipo reciben **tipos**.

Los genéricos se introdujeron en Java 5 (JDK 5.0) con el objetivo de reducir errores y añadir una capa de abstracción sobre los tipos.

### El problema sin genéricos

```java
List lista = new LinkedList();
lista.add(Integer.valueOf(1));
Integer i = lista.iterator().next(); // ERROR de compilación: devuelve Object
Integer j = (Integer) lista.iterator().next(); // hace falta un cast explícito
```

Sin genéricos, la lista guarda cualquier `Object`. El compilador no puede garantizar qué tipo sale de ella, así que obliga a hacer un *cast*. Esto tiene dos problemas:

- Ensucia el código.
- Si te equivocas, el fallo aparece **en tiempo de ejecución** (`ClassCastException`), a veces lejos de donde está la causa real.

### La solución con genéricos

```java
List<Integer> lista = new LinkedList<>();
lista.add(1);
Integer i = lista.iterator().next(); // sin cast
```

Al indicar el tipo entre `<>`, el compilador sabe qué contiene la lista y **verifica la corrección en tiempo de compilación**.

### Ventajas (resumen de las tres fuentes)

| Ventaja | Explicación |
|---|---|
| **Comprobación de tipos más estricta** | Los errores se detectan al compilar, que es mucho más barato que depurarlos en ejecución. |
| **Eliminación de casts** | Código más limpio y menos propenso a errores. |
| **Reutilización de código** | Una sola clase o método sirve para muchos tipos distintos. |
| **Algoritmos genéricos** | Puedes escribir algoritmos que funcionan sobre colecciones de distintos tipos, de forma segura y legible. |

---

## 2. Clases e interfaces genéricas

### 2.1. Una clase genérica sencilla

Primero, una versión **no genérica** que acepta cualquier objeto:

```java
public class Caja {
    private Object objeto;

    public void set(Object objeto) { this.objeto = objeto; }
    public Object get() { return objeto; }
}
```

Problema: no hay forma de verificar en compilación cómo se usa. Alguien puede guardar un `Integer` y otra parte del código esperar un `String`, y el fallo saltará en ejecución.

Ahora la versión **genérica**:

```java
/**
 * Versión genérica de Caja.
 * @param <T> tipo del valor almacenado
 */
public class Caja<T> {
    private T valor; // T significa "Type" (tipo)

    public void set(T valor) { this.valor = valor; }
    public T get() { return valor; }
}
```

El formato general de una clase genérica es:

```java
class Nombre<T1, T2, ..., Tn> { /* ... */ }
```

La sección de parámetros de tipo va entre `<>` justo después del nombre de la clase. Dentro de la clase, `T` se puede usar como si fuera un tipo cualquiera.

### 2.2. Usar la clase genérica

```java
Caja<String> cajaTexto = new Caja<>();
cajaTexto.set("Hola");
System.out.println("Valor: " + cajaTexto.get());

Caja<Integer> cajaNumero = new Caja<>();
cajaNumero.set(50);
System.out.println("Valor: " + cajaNumero.get());
```

Cuando escribes `Caja<String>`, la `T` pasa a ser `String`; con `Caja<Integer>`, pasa a ser `Integer`. **La misma clase se reutiliza sin reescribirla.**

### 2.3. Parámetro de tipo vs. argumento de tipo

Mucha gente usa estos términos como sinónimos, pero no lo son:

- En `Caja<T>`, la `T` es un **parámetro de tipo** (parte de la declaración).
- En `Caja<String>`, el `String` es un **argumento de tipo** (parte del uso).

A `Caja<String>` se le llama **tipo parametrizado**. Y declarar `Caja<Integer> c;` **no crea** ninguna caja: solo declara una referencia a "una Caja de Integer".

### 2.4. Convenciones de nombres

Por convención, los parámetros de tipo son **una sola letra mayúscula**. Así se distinguen fácilmente de los nombres normales de clases.

| Letra | Significado | Dónde se ve |
|---|---|---|
| `T` | Type (tipo) | Uso general |
| `E` | Element (elemento) | Colecciones: `List<E>` |
| `K` | Key (clave) | `Map<K, V>` |
| `V` | Value (valor) | `Map<K, V>` |
| `N` | Number (número) | Tipos numéricos |
| `S`, `U`, `V`... | 2.º, 3.º, 4.º tipo | Varios parámetros |

### 2.5. El operador diamante `<>`

Desde Java 7, si el compilador puede deducir los argumentos de tipo por el contexto, puedes dejar los `<>` vacíos al construir:

```java
Caja<Integer> c1 = new Caja<Integer>(); // verboso
Caja<Integer> c2 = new Caja<>();        // recomendado
```

### 2.6. Varios parámetros de tipo

```java
public interface Par<K, V> {
    K getClave();
    V getValor();
}

public class ParOrdenado<K, V> implements Par<K, V> {
    private final K clave;
    private final V valor;

    public ParOrdenado(K clave, V valor) {
        this.clave = clave;
        this.valor = valor;
    }

    @Override public K getClave() { return clave; }
    @Override public V getValor() { return valor; }
}

Par<String, Integer> p1 = new ParOrdenado<>("Par", 8);
Par<String, String>  p2 = new ParOrdenado<>("hola", "mundo");
```

Gracias al *autoboxing*, puedes pasar un `int` (`8`) donde se espera un `Integer`.

Un argumento de tipo también puede ser otro tipo parametrizado:

```java
ParOrdenado<String, Caja<Integer>> p = new ParOrdenado<>("primos", new Caja<>());
```

### 2.7. Interfaces genéricas

Se declaran con las mismas reglas que las clases. Ya has visto `Par<K, V>`; también lo son `List<E>`, `Comparable<T>`, `Map<K, V>`, etc.

---

## 3. Tipos crudos (*raw types*)

Un **tipo crudo** es el nombre de una clase o interfaz genérica **sin argumentos de tipo**:

```java
Caja<Integer> cajaTipada = new Caja<>(); // tipo parametrizado
Caja cajaCruda = new Caja();             // tipo crudo (raw type)
```

Existen por **compatibilidad hacia atrás**: muchas clases del API (como las colecciones) no eran genéricas antes de Java 5. Con un tipo crudo obtienes el comportamiento "antiguo": todo es `Object`.

```java
Caja<String> cajaTexto = new Caja<>();
Caja cruda = cajaTexto;      // permitido
cruda.set(8);                // ADVERTENCIA: unchecked invocation (¡y rompe el contrato!)

Caja<Integer> cajaNum = cruda; // ADVERTENCIA: unchecked conversion
```

> **Regla práctica:** **evita siempre los tipos crudos** en código nuevo. Se saltan las comprobaciones de tipos y dejan los errores para tiempo de ejecución.

### Avisos "unchecked"

Si ves esto al compilar:

```text
Note: Ejemplo.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

significa que el compilador no tiene información suficiente para garantizar la seguridad de tipos. Para ver los detalles, compila con `-Xlint:unchecked`. La anotación `@SuppressWarnings("unchecked")` silencia el aviso, pero **úsala solo cuando estés seguro** de que el código es correcto, y con el menor alcance posible.

---

## 4. Métodos genéricos

Un método genérico se escribe **una sola vez** y se puede llamar con argumentos de distintos tipos. El compilador comprueba que el tipo usado sea coherente.

### 4.1. Características

- Llevan los parámetros de tipo **antes del tipo de retorno**: `public <T> List<T> ...`.
- El `<T>` es obligatorio aunque el método devuelva `void`.
- Pueden tener **varios parámetros de tipo** separados por comas.
- Los parámetros de tipo pueden estar **acotados**.
- El cuerpo es como el de un método normal.
- Pueden estar en clases genéricas o no genéricas.

### 4.2. Ejemplos

Imprimir cualquier array:

```java
public class Utilidades {
    public static <T> void imprimirArray(T[] array) {
        for (T elemento : array) {
            System.out.println(elemento);
        }
    }

    public static void main(String[] args) {
        String[] nombres = {"Jenny", "Liam"};
        Integer[] numeros = {1, 2, 3};

        imprimirArray(nombres);  // T = String
        imprimirArray(numeros);  // T = Integer
    }
}
```

Convertir un array en lista (usando `Stream.toList()`, disponible desde Java 16):

```java
public static <T> List<T> deArrayALista(T[] array) {
    return Arrays.stream(array).toList();
}
```

Con **dos** parámetros de tipo y una función de transformación:

```java
public static <T, G> List<G> deArrayALista(T[] array, Function<T, G> transformador) {
    return Arrays.stream(array)
                 .map(transformador)
                 .toList();
}

Integer[] enteros = {1, 2, 3, 4, 5};
List<String> textos = deArrayALista(enteros, Object::toString);
// ["1", "2", "3", "4", "5"]
```

### 4.3. Constructores genéricos

Los constructores también pueden declarar sus propios parámetros de tipo, tanto en clases genéricas como en no genéricas:

```java
class MiClase<X> {
    <T> MiClase(T t) {
        // ...
    }
}
```

---

## 5. Inferencia de tipos

La **inferencia de tipos** es la capacidad del compilador de deducir los argumentos de tipo mirando la invocación y la declaración. Por eso casi nunca necesitas escribirlos a mano.

```java
static <T> void imprimir(T dato) { System.out.println(dato); }

imprimir("texto"); // el compilador infiere T = String
imprimir(42);      // el compilador infiere T = Integer
```

### 5.1. Testigo de tipo (*type witness*)

Puedes indicar el tipo explícitamente si hace falta (rara vez es necesario):

```java
Utilidades.<Integer>imprimir(10);
```

### 5.2. Tipo objetivo (*target type*)

El compilador también usa el tipo que **espera** en ese punto del código:

```java
List<String> vacia = Collections.emptyList(); // infiere T = String por el destino
procesar(Collections.emptyList());            // también funciona en Java 8+ si procesar espera List<String>
```

### 5.3. Inferencia con el diamante

Para aprovechar la inferencia al instanciar una clase genérica **debes usar el diamante**. Sin él, estás usando el tipo crudo:

```java
Map<String, List<String>> bien = new HashMap<>();   // correcto
Map<String, List<String>> mal  = new HashMap();     // aviso: unchecked conversion
```

> La inferencia solo usa los argumentos de la invocación, el tipo objetivo y el tipo de retorno esperado. **No** usa información de líneas posteriores del programa.

---

## 6. Tipos acotados (*bounded types*)

A veces quieres **restringir** qué tipos se pueden usar como argumento. Para eso están los parámetros de tipo acotados.

### 6.1. Cota superior con `extends`

```java
class Estadisticas<T extends Number> {
    private final T[] numeros;

    Estadisticas(T[] numeros) { this.numeros = numeros; }

    double media() {
        double suma = 0;
        for (T n : numeros) {
            suma += n.doubleValue(); // posible porque T es, como mínimo, un Number
        }
        return suma / numeros.length;
    }
}

Estadisticas<Integer> e1 = new Estadisticas<>(new Integer[]{10, 20, 30, 40});
Estadisticas<Double>  e2 = new Estadisticas<>(new Double[]{1.5, 2.5, 3.5});
// Estadisticas<String> e3 = ...;  // ERROR: String no es un Number
```

Puntos importantes:

- `T extends Number` significa "`T` es `Number` o una subclase".
- En este contexto, `extends` se usa en sentido amplio: significa **"extiende"** (si la cota es una clase) o **"implementa"** (si es una interfaz).
- Además de limitar los tipos, la cota te permite **invocar los métodos de la cota** (aquí, `doubleValue()`).

### 6.2. Métodos genéricos acotados

```java
public static <T extends Number> double sumar(List<T> lista) {
    double total = 0;
    for (T n : lista) total += n.doubleValue();
    return total;
}
```

Un ejemplo clásico es el método `max` que requiere que los elementos sean comparables:

```java
public static <T extends Comparable<T>> T maximo(List<T> lista) {
    T mayor = lista.get(0);
    for (T elemento : lista) {
        if (elemento.compareTo(mayor) > 0) mayor = elemento;
    }
    return mayor;
}
```

### 6.3. Múltiples cotas

```java
<T extends B1 & B2 & B3>
```

Un tipo con varias cotas es subtipo de **todas** ellas. Si una de las cotas es una **clase**, debe ir **la primera**; si no, error de compilación.

```java
class A { }
interface B { }
interface C { }

class D<T extends A & B & C> { }   // correcto
// class E<T extends B & A & C> { } // ERROR: la clase A debe ir primera
```

> **Nota sobre Baeldung:** el artículo escribe `<T extends Number & Comparable>` con `Comparable` sin parametrizar (tipo crudo). En código real conviene escribir `<T extends Number & Comparable<T>>`.

---

## 7. Genéricos, herencia y subtipos

### 7.1. La trampa más común

Sabes que `Integer` es subtipo de `Number`, y que puedes hacer:

```java
Number n = Integer.valueOf(10); // OK
```

Pero esto **no** se extiende a los tipos genéricos:

```java
List<Integer> enteros = new ArrayList<>();
List<Number> numeros = enteros; // ERROR de compilación
```

**`List<Integer>` NO es subtipo de `List<Number>`**, aunque `Integer` sea subtipo de `Number`. Igualmente, `List<Object>` no es supertipo de `List<String>`.

¿Por qué? Porque si se permitiera, podrías hacer esto:

```java
numeros.add(3.14); // metería un Double en una lista que en realidad es de Integer
Integer i = enteros.get(0); // ¡ClassCastException!
```

> **Regla:** dados dos tipos concretos `A` y `B`, `MiClase<A>` **no tiene relación** con `MiClase<B>`, estén o no relacionados `A` y `B`. Su único ancestro común es `Object`.

### 7.2. Lo que sí es subtipo

Si **no cambias el argumento de tipo**, la relación de herencia entre las clases genéricas se mantiene:

```java
ArrayList<String> a = new ArrayList<>();
List<String> b = a;           // OK: ArrayList<E> implements List<E>
Collection<String> c = b;     // OK: List<E> extends Collection<E>
```

Para conseguir una relación parecida cuando los argumentos de tipo difieren, se usan los **comodines** (siguiente sección).

---

## 8. Comodines (*wildcards*)

El comodín se escribe `?` y representa un **tipo desconocido**. Sirve para relajar las restricciones que has visto en la sección anterior.

### 8.1. Comodín con cota superior: `? extends T`

Admite `T` **y cualquiera de sus subtipos**.

```java
public static double sumaLista(List<? extends Number> lista) {
    double suma = 0.0;
    for (Number n : lista) {
        suma += n.doubleValue();
    }
    return suma;
}

List<Integer> enteros = List.of(1, 2, 3);
List<Double> decimales = List.of(1.2, 2.3, 3.5);
sumaLista(enteros);   // 6.0
sumaLista(decimales); // 7.0
```

Comparación:

- `List<Number>` solo acepta listas de exactamente `Number`.
- `List<? extends Number>` acepta `List<Number>`, `List<Integer>`, `List<Double>`, etc.

### 8.2. Comodín sin cota: `?`

Significa "lista de tipo desconocido". Es útil cuando:

- Solo necesitas funcionalidad que ofrece `Object`.
- Usas métodos que no dependen del parámetro de tipo (`size()`, `clear()`...). Por eso se usa tanto `Class<?>`.

```java
// Solo imprime listas de Object: NO sirve para List<String>
public static void imprimirMal(List<Object> lista) { /* ... */ }

// Sirve para cualquier lista
public static void imprimir(List<?> lista) {
    for (Object elemento : lista) {
        System.out.print(elemento + " ");
    }
    System.out.println();
}

imprimir(List.of(1, 2, 3));
imprimir(List.of("uno", "dos"));
```

> **`List<Object>` y `List<?>` no son lo mismo.** En una `List<Object>` puedes insertar cualquier objeto. En una `List<?>` **solo puedes insertar `null`**.

### 8.3. Comodín con cota inferior: `? super T`

Admite `T` **y cualquiera de sus supertipos**.

```java
public static void anadirNumeros(List<? super Integer> lista) {
    for (int i = 1; i <= 10; i++) {
        lista.add(i); // seguro: la lista admite Integer
    }
}

anadirNumeros(new ArrayList<Integer>());
anadirNumeros(new ArrayList<Number>());
anadirNumeros(new ArrayList<Object>());
```

> Puedes especificar una cota superior **o** una inferior para un comodín, pero **no ambas a la vez**.

### 8.4. ¿Cuál uso? Regla "entrada / salida" (PECS)

Piensa en cada parámetro como de **entrada** ("in", te da datos) o de **salida** ("out", recibe datos). Piensa en `copiar(origen, destino)`: `origen` es de entrada y `destino` es de salida.

| Situación | Comodín |
|---|---|
| Variable de **entrada** (solo la lees) | `? extends T` |
| Variable de **salida** (solo escribes en ella) | `? super T` |
| Entrada que solo usa métodos de `Object` | `?` |
| Se usa **a la vez** como entrada y salida | **No uses comodín** |

Esta regla se conoce en inglés como **PECS** (*Producer Extends, Consumer Super*).

Ejemplo aplicado:

```java
public static <T> void copiar(List<? extends T> origen, List<? super T> destino) {
    for (T elemento : origen) {
        destino.add(elemento);
    }
}
```

**Reglas adicionales:**

- **Evita comodines en el tipo de retorno**: obligan a quien llama a lidiar con ellos.
- Una `List<? extends X>` se puede considerar informalmente "de solo lectura", pero no es una garantía estricta. Aún puedes: añadir `null`, llamar a `clear()`, obtener un iterador y llamar a `remove()`.

### 8.5. Comodines y subtipado

Con comodines sí hay relaciones de subtipo:

```java
List<? extends Integer> a = new ArrayList<Integer>();
List<? extends Number>  b = a;  // OK: ? extends Integer es subtipo de ? extends Number
List<?>                 c = b;  // OK
```

Y recuerda que para cualquier tipo concreto `A`, `List<A>` es subtipo de `List<?>`.

### 8.6. Captura de comodín y métodos auxiliares (nivel avanzado)

A veces el compilador no puede trabajar con `?` directamente, aunque tú sepas que el código es correcto:

```java
void intercambiarPrimeros(List<?> lista) {
    // lista.set(0, lista.get(1)); // ERROR: el compilador no sabe qué tipo es "?"
    ayudante(lista);               // truco: delegar en un método genérico
}

private <T> void ayudante(List<T> lista) {
    T temporal = lista.get(0);
    lista.set(0, lista.get(1));
    lista.set(1, temporal);
}
```

El compilador "captura" el `?` como un tipo concreto (pero anónimo) `T` al entrar en el método auxiliar. Es un patrón útil, pero no necesario al empezar.

---

## 9. Borrado de tipos (*type erasure*)

Los genéricos son una característica **de tiempo de compilación**. Para no añadir sobrecarga en ejecución y mantener compatibilidad con código antiguo, el compilador aplica el **borrado de tipos**:

1. Sustituye cada parámetro de tipo por su **cota** (o por `Object` si no tiene cota).
2. Inserta **casts** donde haga falta para mantener la seguridad de tipos.
3. Genera **métodos puente** (*bridge methods*) cuando es necesario para preservar el polimorfismo.

Resultado: el *bytecode* contiene solo clases, interfaces y métodos normales. No se crean tipos nuevos por cada parametrización.

### 9.1. Ejemplos

Sin cota, `T` se convierte en `Object`:

```java
// Código fuente
public <T> List<T> metodo(List<T> lista) { return lista; }

// Equivalente tras el borrado
public List metodo(List lista) { return lista; }
```

Con cota, `T` se sustituye por la cota:

```java
// Código fuente
public <T extends Edificio> void metodo(T t) { /* ... */ }

// Equivalente tras el borrado
public void metodo(Edificio t) { /* ... */ }
```

### 9.2. Métodos puente (nivel avanzado)

```java
public class Nodo<T> {
    private T dato;
    public void setDato(T dato) { this.dato = dato; }
}

public class MiNodo extends Nodo<Integer> {
    @Override
    public void setDato(Integer dato) { super.setDato(dato); }
}
```

Tras el borrado, `Nodo.setDato` pasa a ser `setDato(Object)`, mientras que `MiNodo.setDato` recibe `Integer`: **ya no coinciden las firmas**, y el polimorfismo se rompería. Para evitarlo, el compilador genera en `MiNodo` un método puente `setDato(Object)` que hace el cast a `Integer` y llama al método real. Normalmente ni lo ves, pero explica algunos errores o trazas extrañas.

### 9.3. Consecuencia práctica

**En tiempo de ejecución no se conoce el argumento de tipo.** Una `ArrayList<String>` y una `ArrayList<Integer>` son la misma clase para la JVM. De ahí vienen varias de las restricciones de la sección 11.

---

## 10. Genéricos y tipos primitivos

**El argumento de tipo no puede ser un tipo primitivo.**

```java
List<int> lista = new ArrayList<>();           // ERROR de compilación
List<Integer> lista = new ArrayList<>();       // correcto
```

¿Por qué? Porque, tras el borrado, los parámetros de tipo se convierten en `Object`, y los tipos primitivos (`int`, `double`, `boolean`...) **no extienden `Object`**.

La solución son las **clases envoltorio** (*wrappers*) junto con el *autoboxing* y *unboxing* automáticos:

| Primitivo | Envoltorio |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

```java
List<Integer> lista = new ArrayList<>();
lista.add(17);              // autoboxing: int -> Integer
int primero = lista.get(0); // unboxing: Integer -> int
```

El compilador lo traduce a algo equivalente a:

```java
lista.add(Integer.valueOf(17));
int primero = ((Integer) lista.get(0)).intValue();
```

> **Cuidado:** el *unboxing* de un `Integer` que vale `null` lanza `NullPointerException`. Además, el boxing tiene un coste de rendimiento frente a trabajar con primitivos puros.

> **Nota sobre el futuro:** el artículo de Baeldung menciona que versiones futuras de Java podrían permitir primitivos en genéricos gracias al **Proyecto Valhalla** (genéricos especializados, JEP 218). Es una línea de trabajo del OpenJDK que sigue evolucionando, pero **en Java 25 las restricciones descritas aquí siguen vigentes**. Consulta las notas de versión del JDK para ver novedades.

---

## 11. Restricciones de los genéricos

Estas son las limitaciones que debes conocer (las lista el tutorial de Oracle). Casi todas son consecuencia del borrado de tipos.

### 11.1. No se pueden usar tipos primitivos como argumento

```java
Par<int, char> p = new Par<>(8, 'a');                 // ERROR
Par<Integer, Character> p = new Par<>(8, 'a');        // OK (autoboxing)
```

### 11.2. No se pueden crear instancias de un parámetro de tipo

```java
public static <E> void anadir(List<E> lista) {
    E elemento = new E(); // ERROR de compilación
    lista.add(elemento);
}
```

Alternativa: pasar un `Class<E>` y usar reflexión. **Ojo:** `Class.newInstance()` (el que usa la documentación de Oracle) está *deprecado* desde Java 9. Hoy se escribe así:

```java
public static <E> void anadir(List<E> lista, Class<E> clase) throws ReflectiveOperationException {
    E elemento = clase.getDeclaredConstructor().newInstance();
    lista.add(elemento);
}
```

Otra alternativa más moderna y segura es recibir un `Supplier<E>`:

```java
public static <E> void anadir(List<E> lista, Supplier<E> fabrica) {
    lista.add(fabrica.get());
}

anadir(textos, String::new);
```

### 11.3. No se pueden declarar campos `static` de tipo parámetro

```java
public class Dispositivo<T> {
    private static T sistemaOperativo; // ERROR
}
```

Un campo `static` se comparte entre todas las instancias. Si fuera de tipo `T`, ¿sería `Smartphone`, `Tablet` o `Pager` a la vez? No tiene sentido, por eso no se permite.

### 11.4. No se pueden usar *casts* ni `instanceof` con tipos parametrizados

```java
if (lista instanceof ArrayList<Integer>) { }  // ERROR: en ejecución no existe <Integer>
if (lista instanceof ArrayList<?>) { }        // OK: el comodín sin cota sí es válido
```

Tampoco puedes hacer `(List<Number>) listaDeEnteros`. Solo se permite cuando el compilador sabe que es seguro, por ejemplo `(ArrayList<String>) listaDeStrings`.

### 11.5. No se pueden crear arrays de tipos parametrizados

```java
List<Integer>[] arrayDeListas = new List<Integer>[2]; // ERROR
```

Los arrays comprueban el tipo de sus elementos en ejecución (lanzan `ArrayStoreException`), pero con el borrado de tipos esa comprobación no podría distinguir `List<String>` de `List<Integer>`. Usa en su lugar una lista de listas:

```java
List<List<Integer>> listaDeListas = new ArrayList<>();
```

### 11.6. No se pueden crear, capturar ni lanzar objetos de tipos parametrizados

```java
class MiExcepcion<T> extends Exception { }  // ERROR: una clase genérica no puede extender Throwable

public static <T extends Exception> void ejecutar() {
    try { /* ... */ }
    catch (T e) { }                          // ERROR: no se puede capturar un parámetro de tipo
}
```

Lo que **sí** puedes es usar un parámetro de tipo en la cláusula `throws`:

```java
class Parser<T extends Exception> {
    public void parsear(Path fichero) throws T { /* ... */ }
}
```

### 11.7. No se puede sobrecargar con firmas que se borran al mismo tipo

```java
public class Ejemplo {
    public void imprimir(Set<String> conjuntoTexto) { }
    public void imprimir(Set<Integer> conjuntoEnteros) { } // ERROR: ambos son imprimir(Set) tras el borrado
}
```

---

## 12. Genéricos con características modernas de Java

Las tres fuentes están escritas con Java 8 como referencia. Estas son las mejoras posteriores que conviene combinar con los genéricos.

### 12.1. `var` (inferencia de variables locales, Java 10+)

`var` reduce el ruido, pero el tipo sigue siendo estático y completo:

```java
var mapa = new HashMap<String, List<Integer>>(); // tipo inferido: HashMap<String, List<Integer>>
```

> Cuidado: `var lista = new ArrayList<>();` infiere `ArrayList<Object>`. Si quieres otro tipo, indícalo: `var lista = new ArrayList<String>();`.

### 12.2. *Records* genéricos (Java 16+)

Los `record` pueden ser genéricos y son perfectos para tipos de datos sencillos como un par:

```java
public record Par<A, B>(A primero, B segundo) { }

Par<String, Integer> p = new Par<>("edad", 30);
System.out.println(p.primero() + " = " + p.segundo());
```

### 12.3. Interfaces selladas y *pattern matching* en `switch` (Java 21+)

Combinados con genéricos permiten modelar resultados de forma segura y exhaustiva:

```java
public sealed interface Resultado<T> permits Exito, Fallo { }
public record Exito<T>(T valor) implements Resultado<T> { }
public record Fallo<T>(String mensaje) implements Resultado<T> { }

static <T> String describir(Resultado<T> resultado) {
    return switch (resultado) {
        case Exito<T>(var valor)    -> "OK: " + valor;
        case Fallo<T>(var mensaje)  -> "Error: " + mensaje;
    };
}
```

El compilador comprueba que el `switch` cubre todos los casos posibles, sin necesidad de `default`.

### 12.4. Métodos de fábrica de colecciones y `Stream.toList()`

Los ejemplos modernos usan `List.of(...)`, `Map.of(...)` y `stream.toList()` en lugar de `Arrays.asList(...)` o `collect(Collectors.toList())` cuando no necesitas una lista modificable. Recuerda que las listas de `List.of` y `Stream.toList()` son **inmutables**.

### 12.5. Constructores de envoltorios obsoletos

Algunos ejemplos de la documentación de Oracle usan `new Integer(10)`. Esos constructores están **deprecados para su eliminación**. Usa siempre `Integer.valueOf(10)` o directamente el autoboxing (`Integer x = 10;`).

---

## 13. Errores habituales y buenas prácticas

### Errores típicos

| Error | Qué pasa | Solución |
|---|---|---|
| Usar tipos crudos (`List` a secas) | Pierdes comprobación de tipos; avisos *unchecked* | Parametriza siempre: `List<String>` |
| Pensar que `List<Integer>` es subtipo de `List<Number>` | Error de compilación | Usa `List<? extends Number>` |
| Intentar `List<int>` | Error de compilación | Usa `List<Integer>` |
| Intentar `new T()` | Error de compilación | Recibe un `Supplier<T>` |
| Escribir en una `List<? extends X>` | Error de compilación | Cambia el diseño o usa `? super X` si vas a escribir |
| `instanceof List<String>` | Error de compilación | Usa `instanceof List<?>` |
| Silenciar `unchecked` sin entender el motivo | Ocultas fallos reales | Soluciona la causa; suprime solo con justificación |

### Buenas prácticas

1. **Usa genéricos siempre** que trabajes con colecciones y estructuras de datos reutilizables.
2. **Nunca uses tipos crudos** en código nuevo.
3. **Usa el diamante `<>`** al instanciar.
4. **Sigue las convenciones de nombres** (`T`, `E`, `K`, `V`...). Si hay varios parámetros y las letras no aclaran, puedes usar nombres más descriptivos como `TipoEntrada`, siempre que sean fáciles de distinguir de las clases normales.
5. **Aplica PECS** para los parámetros de tus métodos: `extends` para leer, `super` para escribir.
6. **No uses comodines en tipos de retorno.**
7. **Acota solo lo necesario**: una cota innecesaria reduce la reutilización.
8. **Documenta los parámetros de tipo** con `@param <T>` en el Javadoc.
9. **Prefiere `List<T>` a arrays de tipos genéricos** (los arrays y los genéricos se llevan mal).

---

## 14. Chuleta rápida

```java
// Clase genérica
class Caja<T> { T valor; }

// Interfaz genérica
interface Par<K, V> { K clave(); V valor(); }

// Instanciación con diamante
Caja<String> c = new Caja<>();

// Método genérico
static <T> void imprimir(T dato) { }

// Varios parámetros de tipo
static <T, G> List<G> transformar(T[] a, Function<T, G> f) { }

// Cota superior (T es Number o subclase)
class Estadisticas<T extends Number> { }

// Múltiples cotas (la clase primero)
class D<T extends Number & Comparable<T>> { }

// Comodines
List<?>               // cualquier lista (solo lectura como Object)
List<? extends Number> // Number o subtipos  -> LEER
List<? super Integer>  // Integer o supertipos -> ESCRIBIR
```

| Quiero... | Uso |
|---|---|
| Reutilizar una clase con varios tipos | `class X<T>` |
| Reutilizar un método con varios tipos | `<T> void m(T x)` |
| Limitar los tipos permitidos | `<T extends Cota>` |
| Aceptar una lista de "algo que sea Number" para **leerla** | `List<? extends Number>` |
| Aceptar una lista donde pueda **escribir** Integer | `List<? super Integer>` |
| Aceptar cualquier lista sin importar el tipo | `List<?>` |
| Guardar primitivos en una colección | Usar el envoltorio: `List<Integer>` |

---

## 15. Ejercicios propuestos

1. **Caja genérica.** Crea `Caja<T>` con `set`, `get` y un método `estaVacia()`. Pruébala con `String`, `Integer` y `LocalDate`.
2. **Par.** Implementa un `record Par<A, B>` y un método genérico `static <A, B> Par<B, A> invertir(Par<A, B> par)`.
3. **Máximo.** Escribe `static <T extends Comparable<T>> T maximo(List<T> lista)` y pruébalo con `Integer`, `String` y `Double`.
4. **Suma con comodín.** Escribe `double sumar(Collection<? extends Number> numeros)` y llámalo con `List<Integer>` y `Set<Double>`.
5. **PECS.** Implementa `static <T> void copiar(List<? extends T> origen, List<? super T> destino)`. Prueba copiar de `List<Integer>` a `List<Number>`.
6. **Predice el error.** Sin compilar, indica por qué falla cada línea y luego comprueba tu respuesta:
   ```java
   List<Number> a = new ArrayList<Integer>();
   List<int> b = new ArrayList<>();
   List<? extends Number> c = new ArrayList<Integer>();
   c.add(Integer.valueOf(1));
   ```
7. **Resultado.** Implementa la interfaz sellada `Resultado<T>` de la sección 12.3 y un método `static Resultado<Integer> dividir(int a, int b)` que devuelva `Fallo` si `b == 0`.

---

## 16. Referencias

- Baeldung: [The Basics of Java Generics](https://www.baeldung.com/java-generics)
- W3Schools: [Java Generics](https://www.w3schools.com/java/java_generics.asp)
- Oracle, The Java Tutorials: [Lesson: Generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html) y sus secciones:
  [¿Por qué usar genéricos?](https://docs.oracle.com/javase/tutorial/java/generics/why.html) ·
  [Tipos genéricos](https://docs.oracle.com/javase/tutorial/java/generics/types.html) ·
  [Raw types](https://docs.oracle.com/javase/tutorial/java/generics/rawTypes.html) ·
  [Tipos acotados](https://docs.oracle.com/javase/tutorial/java/generics/bounded.html) ·
  [Herencia y subtipos](https://docs.oracle.com/javase/tutorial/java/generics/inheritance.html) ·
  [Inferencia de tipos](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html) ·
  [Comodines con cota superior](https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html) ·
  [Comodines sin cota](https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html) ·
  [Comodines con cota inferior](https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html) ·
  [Guía de uso de comodines](https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html) ·
  [Restricciones](https://docs.oracle.com/javase/tutorial/java/generics/restrictions.html)
- Oracle: [Dev.java](https://dev.java/learn/), tutoriales actualizados a las últimas versiones de Java.

> **Nota de alcance:** los tutoriales de Oracle están escritos para JDK 8. Las secciones 9.2 (métodos puente) y 8.6 (captura de comodín) amplían contenido de las subpáginas de Oracle con explicación y ejemplos propios, y la sección 12 (Java moderno) es aportación de este documento, no de las fuentes.
