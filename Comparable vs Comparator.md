# Comparable y Comparator en Java

### Guía de estudio para programadores (Java 21 en adelante)

> **Cómo se ha elaborado esta guía**
>
> - Contenido contrastado con la documentación oficial de Oracle (Javadoc de Java SE 21) y con el artículo de GeeksforGeeks [Java Comparable vs Comparator](https://www.geeksforgeeks.org/java/comparable-vs-comparator-in-java/).
> - **Todos los ejemplos de código se han compilado y ejecutado** con OpenJDK 21.0.10. Las salidas que ves son las salidas reales, no inventadas.
> - Los nombres de clases, variables y métodos están en inglés británico (`Film`, `forename`, `surname`, `mark`…). Las explicaciones están en castellano.
> - Cuando algo procede de mi conocimiento del lenguaje y no de una página consultada, se indica en la sección de [fuentes](#14-fuentes-y-cómo-se-ha-verificado).

## Índice

- [0. Cómo ejecutar los ejemplos](#0-cómo-ejecutar-los-ejemplos)
- [1. El problema de ordenar](#1-el-problema-de-ordenar)
- [2. Conceptos previos](#2-conceptos-previos)
- [3. Comparable](#3-comparable)
- [4. Comparator](#4-comparator)
- [5. Dónde se usan](#5-dónde-se-usan)
- [6. Comparable frente a Comparator](#6-comparable-frente-a-comparator)
- [7. Errores típicos](#7-errores-típicos)
- [8. Novedades y estado en Java 21](#8-novedades-y-estado-en-java-21)
- [9. Ejemplo completo](#9-ejemplo-completo)
- [10. Ejercicios](#10-ejercicios)
- [11. Chuleta de repaso](#11-chuleta-de-repaso)
- [12. Autoevaluación](#12-autoevaluación)
- [13. Observaciones sobre el artículo de GeeksforGeeks](#13-observaciones-sobre-el-artículo-de-geeksforgeeks)
- [14. Fuentes y cómo se ha verificado](#14-fuentes-y-cómo-se-ha-verificado)

---

## 0. Cómo ejecutar los ejemplos

Cada ejemplo es **un único fichero** `.java`. Solo necesitas un JDK 21 o superior:

1. Guarda el código en un fichero con el nombre de la clase principal (por ejemplo, `ComparableFilmDemo.java`).
2. Ejecútalo con:

```bash
java ComparableFilmDemo.java
```

Este modo (ejecutar directamente un fichero fuente) compila y ejecuta en un solo paso. En los ejemplos, la clase con el método `main` va siempre la primera del fichero; las demás clases (`record`, `enum`…) van debajo.

> **Si ves signos `?` en lugar de tildes** en la consola, es que tu consola no está en UTF-8. Ejecuta con `java -Dstdout.encoding=UTF-8 NombreDemo.java`. Las salidas de esta guía se generaron así.

---

## 1. El problema de ordenar

Con números y textos, ordenar es trivial porque Java ya sabe compararlos:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class SortingProblemDemo {

    public static void main(String[] args) {
        // Con tipos que ya saben ordenarse, Collections.sort funciona directamente
        List<Integer> numbers = new ArrayList<>(List.of(42, 7, 19, 3));
        Collections.sort(numbers);
        System.out.println(numbers);

        List<String> words = new ArrayList<>(List.of("pear", "apple", "cherry"));
        Collections.sort(words);
        System.out.println(words);
    }
}
```

Salida:

```text
[3, 7, 19, 42]
[apple, cherry, pear]
```

Pero ¿qué ocurre con una clase nuestra, por ejemplo `Film` (película)? Java no puede adivinar si queremos ordenar por título, por año de estreno o por valoración. Si intentamos ordenar sin darle esa información, **el código ni siquiera compila**:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class NotComparableDemo {

    public static void main(String[] args) {
        List<Film> films = new ArrayList<>();
        films.add(new Film("Star Wars", 8.7, 1977));
        films.add(new Film("The Empire Strikes Back", 8.8, 1980));

        Collections.sort(films); // ¿Cómo sabe Java qué película va antes?
    }
}

class Film {
    private final String title;
    private final double rating;
    private final int releaseYear;

    Film(String title, double rating, int releaseYear) {
        this.title = title;
        this.rating = rating;
        this.releaseYear = releaseYear;
    }
}
```

Mensaje del compilador (primeras líneas):

```text
NotComparableDemo.java:12: error: no suitable method found for sort(List<Film>)
        Collections.sort(films); // ¿Cómo sabe Java qué película va antes?
                   ^
    method Collections.<T#1>sort(List<T#1>) is not applicable
      (inference variable T#1 has incompatible bounds
        equality constraints: Film
        upper bounds: Comparable<? super T#1>)
    method Collections.<T#2>sort(List<T#2>,Comparator<? super T#2>) is not applicable
[...]
```

Lo importante está en la línea `upper bounds: Comparable<? super T#1>`: el método `Collections.sort(List<T>)` exige que el tipo `T` de la lista implemente `Comparable`.

Hay **dos formas** de darle a Java la información que le falta:

| Enfoque | Idea |
|---|---|
| `Comparable` | La propia clase sabe compararse con otra de su mismo tipo ("me ordeno a mí mismo"). |
| `Comparator` | Un objeto aparte, externo a la clase, decide cómo se comparan dos objetos ("un árbitro externo"). |

---

## 2. Conceptos previos

### 2.1 Qué significa "ordenar"

La documentación de Java describe un `Comparator` como una función de comparación que impone un **orden total** sobre una colección de objetos. En la práctica, significa que para cualquier par de elementos siempre se puede decir cuál va antes, cuál va después o que son equivalentes, y que las decisiones son coherentes entre sí (más adelante verás las reglas exactas, llamadas *contrato*).

### 2.2 El resultado de una comparación

Tanto `compareTo` (en `Comparable`) como `compare` (en `Comparator`) devuelven un `int`:

| Resultado | Significado | Al ordenar de forma ascendente |
|---|---|---|
| **Negativo** | El primero es *menor* que el segundo | El primero va **antes** |
| **Cero** | Son equivalentes según este orden | Ninguno prevalece sobre el otro |
| **Positivo** | El primero es *mayor* que el segundo | El primero va **después** |

> **Solo importa el signo.** Nunca dependas del valor concreto. Por ejemplo, con `String`, `"apple".compareTo("banana")` devuelve `-1`, pero `"apple".compareTo("cherry")` devuelve `-2` (lo puedes ver en la [sección 3.3](#33-el-orden-natural-de-las-clases-del-jdk)). Ambos son negativos y eso es todo lo que el contrato garantiza.

### 2.3 Por qué aparece `<Film>` en `Comparable<Film>`

`Comparable` e `Comparator` son **interfaces genéricas**. El tipo entre `< >` indica *qué tipo de objetos se comparan*. `Comparable<Film>` significa "un `Film` sabe compararse con otro `Film`", y por eso el método recibe directamente un `Film` (sin necesidad de convertir tipos).

### 2.4 Qué es un `record`

Muchos ejemplos usan `record` (disponible desde Java 16). Un `record` es una forma compacta de declarar una clase de datos inmutable:

```java
record Film(String title, double rating, int releaseYear) { }
```

Java genera automáticamente el constructor, los métodos de acceso (`title()`, `rating()`, `releaseYear()`; ojo, **sin** el prefijo `get`), `equals`, `hashCode` y `toString`. Un `record` puede implementar interfaces (como `Comparable`) y tener campos `static`. Si prefieres las clases tradicionales, todo lo que se explica sirve igual; en la [sección 3.2](#32-primer-ejemplo-ordenar-películas-por-año) verás también una clase tradicional.

---

## 3. Comparable

### 3.1 Definición

`Comparable<T>` (paquete `java.lang`) define el **orden natural** de una clase. Tiene un único método:

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

`a.compareTo(b)` devuelve un valor negativo, cero o positivo según `a` sea menor, igual o mayor que `b`.

**Para hacer que una clase sea `Comparable`:**

1. Añade `implements Comparable<MiClase>` a la declaración.
2. Sobrescribe `public int compareTo(MiClase other)`.
3. Devuelve negativo, cero o positivo comparando `this` con `other`.
4. Para comparar números, usa `Integer.compare`, `Double.compare`, etc. (no restes: mira el [error 7.1](#71-comparar-restando)).

### 3.2 Primer ejemplo: ordenar películas por año

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ComparableFilmDemo {

    public static void main(String[] args) {
        List<Film> films = new ArrayList<>();
        films.add(new Film("Return of the Jedi", 8.4, 1983));
        films.add(new Film("Star Wars", 8.7, 1977));
        films.add(new Film("The Empire Strikes Back", 8.8, 1980));

        // Comparación directa de dos películas
        Film starWars = films.get(1);
        Film empire = films.get(2);
        System.out.println("starWars.compareTo(empire) = " + starWars.compareTo(empire));
        System.out.println("empire.compareTo(starWars) = " + empire.compareTo(starWars));
        System.out.println("starWars.compareTo(starWars) = " + starWars.compareTo(starWars));

        // Collections.sort usa compareTo internamente
        Collections.sort(films);
        System.out.println();
        System.out.println("Films sorted by release year:");
        for (Film film : films) {
            System.out.println(film);
        }
    }
}

class Film implements Comparable<Film> {

    private final String title;
    private final double rating;
    private final int releaseYear;

    Film(String title, double rating, int releaseYear) {
        this.title = title;
        this.rating = rating;
        this.releaseYear = releaseYear;
    }

    String getTitle() {
        return title;
    }

    double getRating() {
        return rating;
    }

    int getReleaseYear() {
        return releaseYear;
    }

    @Override
    public int compareTo(Film other) {
        // Orden natural: por año de estreno, de menor a mayor
        return Integer.compare(this.releaseYear, other.releaseYear);
    }

    @Override
    public String toString() {
        return title + " (" + releaseYear + ", rating " + rating + ")";
    }
}
```

Salida:

```text
starWars.compareTo(empire) = -1
empire.compareTo(starWars) = 1
starWars.compareTo(starWars) = 0

Films sorted by release year:
Star Wars (1977, rating 8.7)
The Empire Strikes Back (1980, rating 8.8)
Return of the Jedi (1983, rating 8.4)
```

Los datos de las películas son los del artículo de GeeksforGeeks y sirven solo como ejemplo.

**Qué está pasando:**

- `Film implements Comparable<Film>`: declara que sus objetos tienen un orden natural.
- En `compareTo`, `this` es el objeto sobre el que se llama y `other` es el argumento. `Integer.compare(this.releaseYear, other.releaseYear)` devuelve negativo si `this` se estrenó antes.
- `starWars.compareTo(empire)` es `-1` (Star Wars, 1977, es anterior). Al invertir los operandos sale `1`, y comparar un objeto consigo mismo da `0`.
- `Collections.sort(films)` **no recibe ningún criterio**: llama internamente a `compareTo` tantas veces como necesite.

### 3.3 El orden natural de las clases del JDK

Muchas clases del JDK ya son `Comparable`. Por eso `Collections.sort` funciona con `String`, `Integer` o `LocalDate` sin más:

```java
import java.text.Collator;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Locale;

public class NaturalOrderOfJdkClassesDemo {

    enum Priority { LOW, MEDIUM, HIGH }

    public static void main(String[] args) {
        // Strings: orden por valor de los caracteres (las mayúsculas van antes que las minúsculas)
        List<String> names = new ArrayList<>(List.of("charlotte", "Bob", "alice", "Zoe"));
        Collections.sort(names);
        System.out.println("Strings:    " + names);

        // Fechas
        List<LocalDate> dates = new ArrayList<>(List.of(
                LocalDate.of(2024, 3, 1), LocalDate.of(2023, 12, 25), LocalDate.of(2024, 1, 15)));
        Collections.sort(dates);
        System.out.println("LocalDate:  " + dates);

        // Enumerados: el orden es el de declaración de las constantes
        List<Priority> priorities = new ArrayList<>(List.of(Priority.HIGH, Priority.LOW, Priority.MEDIUM));
        Collections.sort(priorities);
        System.out.println("Enum:       " + priorities);

        // Del contrato solo se garantiza el signo, no el valor exacto
        System.out.println("\"apple\".compareTo(\"banana\") = " + "apple".compareTo("banana"));
        System.out.println("\"apple\".compareTo(\"cherry\") = " + "apple".compareTo("cherry"));

        // Texto con tildes: el orden natural de String no es el del diccionario
        List<String> cities = new ArrayList<>(List.of("Zaragoza", "Ávila", "Álava", "Badajoz", "Almería"));
        Collections.sort(cities);
        System.out.println("\nString natural order: " + cities);

        Collator spanishCollator = Collator.getInstance(Locale.of("es", "ES"));
        cities.sort(spanishCollator);
        System.out.println("Collator es-ES:       " + cities);
    }
}
```

Salida:

```text
Strings:    [Bob, Zoe, alice, charlotte]
LocalDate:  [2023-12-25, 2024-01-15, 2024-03-01]
Enum:       [LOW, MEDIUM, HIGH]
"apple".compareTo("banana") = -1
"apple".compareTo("cherry") = -2

String natural order: [Almería, Badajoz, Zaragoza, Álava, Ávila]
Collator es-ES:       [Álava, Almería, Ávila, Badajoz, Zaragoza]
```

Lo que se observa:

| Tipo | Orden natural (comprobado en la salida) |
|---|---|
| `String` | Por el valor de los caracteres: las mayúsculas van antes que las minúsculas (`Bob`, `Zoe`, `alice`, `charlotte`). |
| `LocalDate` | Cronológico. |
| `enum` | El orden en que se declaran las constantes (`LOW`, `MEDIUM`, `HIGH`). |

> **Cuidado con el texto en castellano.** El orden natural de `String` compara el valor numérico de los caracteres, y las letras con tilde (`Á`) tienen un valor mayor que la `Z`. Por eso `Álava` y `Ávila` quedan después de `Zaragoza`. Para obtener un orden "de diccionario" hay que usar un `Collator` con el `Locale` adecuado (`java.text.Collator` implementa `Comparator`, así que se pasa directamente a `sort`). `Locale.of(...)` está disponible desde Java 19.

### 3.4 Comparar por varios campos

Muchas veces un solo campo no basta: dos personas pueden tener el mismo apellido. La regla es: **compara el primer campo; si son distintos, devuelve ese resultado; si empatan (resultado 0), pasa al siguiente campo.**

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class MultiFieldComparableDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(
                new Person("Alice", "Smith", 30),
                new Person("Bob", "Jones", 25),
                new Person("Charlotte", "Smith", 22),
                new Person("Oliver", "Jones", 25),
                new Person("Amelia", "Brown", 41)));

        Collections.sort(people);
        people.forEach(System.out::println);

        System.out.println();
        List<PersonManual> manual = new ArrayList<>(List.of(
                new PersonManual("Alice", "Smith"),
                new PersonManual("Bob", "Jones"),
                new PersonManual("Oliver", "Jones")));
        Collections.sort(manual);
        manual.forEach(System.out::println);
    }
}

// Versión recomendada: reutilizar un Comparator dentro de compareTo
record Person(String forename, String surname, int age) implements Comparable<Person> {

    private static final Comparator<Person> NATURAL_ORDER =
            Comparator.comparing(Person::surname)
                    .thenComparing(Person::forename)
                    .thenComparingInt(Person::age);

    @Override
    public int compareTo(Person other) {
        return NATURAL_ORDER.compare(this, other);
    }
}

// Versión manual: se compara campo a campo y se devuelve en cuanto uno desempata
record PersonManual(String forename, String surname) implements Comparable<PersonManual> {

    @Override
    public int compareTo(PersonManual other) {
        int result = this.surname.compareTo(other.surname);
        if (result != 0) {
            return result;
        }
        return this.forename.compareTo(other.forename);
    }
}
```

Salida:

```text
Person[forename=Amelia, surname=Brown, age=41]
Person[forename=Bob, surname=Jones, age=25]
Person[forename=Oliver, surname=Jones, age=25]
Person[forename=Alice, surname=Smith, age=30]
Person[forename=Charlotte, surname=Smith, age=22]

PersonManual[forename=Bob, surname=Jones]
PersonManual[forename=Oliver, surname=Jones]
PersonManual[forename=Alice, surname=Smith]
```

Aquí hay dos versiones:

- `Person` (recomendada): construye un `Comparator` con `Comparator.comparing(...).thenComparing(...)` y lo reutiliza dentro de `compareTo`. Es corta y difícil de equivocar. Los detalles de `comparing` y `thenComparing` se explican en la [sección 4.3](#43-métodos-de-fábrica-y-composición).
- `PersonManual`: hace el desempate a mano, campo a campo. Es más larga, pero deja claro qué ocurre por dentro.

### 3.5 El contrato de `compareTo`

La documentación de `Comparable` exige que tu implementación cumpla estas reglas (donde `sgn` es el signo: -1, 0 o 1):

1. **Antisimetría:** `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))` para cualquier `x` e `y`. Si `x` va antes que `y`, entonces `y` debe ir después que `x`.
2. **Transitividad:** si `x.compareTo(y) > 0` y `y.compareTo(z) > 0`, entonces `x.compareTo(z) > 0`.
3. **Coherencia de la equivalencia:** si `x.compareTo(y) == 0`, entonces `x` e `y` deben compararse igual con cualquier otro `z`.
4. **Recomendado, pero no obligatorio:** que `(x.compareTo(y) == 0) == x.equals(y)`. Se dice entonces que el orden es *coherente con equals*. Si no lo es, la documentación recomienda avisarlo. Consulta el [error 7.2](#72-orden-incoherente-con-equals) para ver por qué importa.
5. `e.compareTo(null)` debería lanzar `NullPointerException`, aunque `e.equals(null)` devuelva `false`.

### 3.6 La limitación de Comparable

Una clase solo puede tener **un** `compareTo`, es decir, un único orden natural. Si hoy quieres ordenar películas por año y mañana por valoración, `Comparable` por sí solo no basta. Para eso existe `Comparator`.

---

## 4. Comparator

### 4.1 Definición

`Comparator<T>` (paquete `java.util`) define un criterio de ordenación **externo** a la clase. Su método abstracto es:

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    // ...además de métodos estáticos y por defecto (sección 4.3)
}
```

`compare(a, b)` devuelve negativo, cero o positivo con el mismo significado que en `compareTo`. Según la documentación, un `Comparator` sirve para:

- Ordenar con métodos como `Collections.sort` o `Arrays.sort`.
- Controlar el orden de estructuras como los conjuntos y mapas ordenados (`TreeSet`, `TreeMap`).
- Ordenar objetos que **no tienen** orden natural.

Como el criterio vive fuera de la clase, puedes tener **tantos comparadores como quieras** para los mismos objetos, sin modificar la clase.

### 4.2 Cuatro formas de escribir un Comparator

Este ejemplo ordena las mismas películas con las cuatro formas más habituales:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class ComparatorFormsDemo {

    public static void main(String[] args) {
        List<Film> films = new ArrayList<>(List.of(
                new Film("Star Wars", 8.7, 1977),
                new Film("The Force Awakens", 8.3, 2015),
                new Film("The Empire Strikes Back", 8.8, 1980)));

        // 1) Clase independiente
        films.sort(new ByRatingDescending());
        System.out.println("Separate class:  " + titles(films));

        // 2) Clase anónima
        films.sort(new Comparator<Film>() {
            @Override
            public int compare(Film first, Film second) {
                return Double.compare(first.rating(), second.rating()); // ascendente
            }
        });
        System.out.println("Anonymous class: " + titles(films));

        // 3) Expresión lambda
        films.sort((first, second) -> Double.compare(second.rating(), first.rating()));
        System.out.println("Lambda:          " + titles(films));

        // 4) Comparator.comparingDouble + referencia a método
        films.sort(Comparator.comparingDouble(Film::rating));
        System.out.println("Factory method:  " + titles(films));

        // Un Comparator se puede guardar en una variable y reutilizar
        Comparator<Film> byTitle = Comparator.comparing(Film::title);
        films.sort(byTitle);
        System.out.println("By title:        " + titles(films));
    }

    private static List<String> titles(List<Film> films) {
        return films.stream().map(Film::title).toList();
    }
}

record Film(String title, double rating, int releaseYear) { }

class ByRatingDescending implements Comparator<Film> {
    @Override
    public int compare(Film first, Film second) {
        return Double.compare(second.rating(), first.rating());
    }
}
```

Salida:

```text
Separate class:  [The Empire Strikes Back, Star Wars, The Force Awakens]
Anonymous class: [The Force Awakens, Star Wars, The Empire Strikes Back]
Lambda:          [The Empire Strikes Back, Star Wars, The Force Awakens]
Factory method:  [The Force Awakens, Star Wars, The Empire Strikes Back]
By title:        [Star Wars, The Empire Strikes Back, The Force Awakens]
```

| Forma | Cuándo la verás |
|---|---|
| **Clase independiente** (`ByRatingDescending`) | Útil si el comparador es complejo o se reutiliza mucho. Es la forma que muestra el artículo de GeeksforGeeks. |
| **Clase anónima** | Es la forma anterior a Java 8. La encontrarás en código antiguo. |
| **Expresión lambda** | Cómoda para criterios puntuales. |
| **Método de fábrica** (`Comparator.comparingDouble(...)`) | La más legible y la recomendada para casos habituales. |

> **Truco para invertir el orden a mano:** intercambia los argumentos. `Double.compare(second.rating(), first.rating())` ordena de mayor a menor, mientras que `Double.compare(first.rating(), second.rating())` lo hace de menor a mayor.

### 4.3 Métodos de fábrica y composición

Desde Java 8, `Comparator` incluye métodos que evitan escribir `compare` a mano (todos aparecen en la documentación de Java 21):

**Métodos estáticos**

| Método | Qué hace |
|---|---|
| `comparing(keyExtractor)` | Compara por una clave que implemente `Comparable` (por ejemplo, `Person::surname`). |
| `comparing(keyExtractor, keyComparator)` | Compara por una clave usando otro `Comparator` para esa clave. |
| `comparingInt`, `comparingLong`, `comparingDouble` | Igual, pero para claves primitivas. Evitan convertir cada valor en un objeto (*boxing*). |
| `naturalOrder()` | Orden natural (`Comparable`). |
| `reverseOrder()` | Inverso del orden natural. |
| `nullsFirst(cmp)`, `nullsLast(cmp)` | Aceptan `null` y lo colocan al principio o al final. |

**Métodos de instancia (`default`)**

| Método | Qué hace |
|---|---|
| `reversed()` | Devuelve un comparador con el orden inverso. |
| `thenComparing(...)` | Si el comparador actual considera iguales dos elementos (`compare == 0`), usa otro criterio para desempatar. |
| `thenComparingInt`, `thenComparingLong`, `thenComparingDouble` | Desempate con clave primitiva. |

Ejemplo con todos ellos:

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ComparatorFactoryMethodsDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(
                new Person("Alice", "Smith", 30),
                new Person("Bob", "Jones", 25),
                new Person("Charlotte", "Smith", 22),
                new Person("Oliver", "Jones", 25),
                new Person("Amelia", "Brown", 41)));

        // comparing: clave Comparable (aquí, un String)
        people.sort(Comparator.comparing(Person::surname));
        System.out.println("By surname:              " + names(people));

        // comparingInt: clave int, sin boxing
        people.sort(Comparator.comparingInt(Person::age));
        System.out.println("By age:                  " + names(people));

        // thenComparing: desempate cuando el primer criterio da 0
        people.sort(Comparator.comparing(Person::surname).thenComparing(Person::forename));
        System.out.println("Surname, then forename:  " + names(people));

        // reversed: invierte todo el comparador sobre el que se llama
        people.sort(Comparator.comparingInt(Person::age).reversed());
        System.out.println("Age descending:          " + names(people));

        // Edad descendente y, si empatan, apellido ascendente
        people.sort(Comparator.comparingInt(Person::age).reversed()
                .thenComparing(Person::surname));
        System.out.println("Age desc, then surname:  " + names(people));

        // comparing con un segundo argumento: cómo comparar la clave
        List<String> words = new ArrayList<>(List.of("banana", "Apple", "cherry", "apple"));
        words.sort(Comparator.comparing(word -> word, String.CASE_INSENSITIVE_ORDER));
        System.out.println("\nCase-insensitive:        " + words);

        // Por longitud y luego sin distinguir mayúsculas (ejemplo de la Javadoc)
        List<String> mixed = new ArrayList<>(List.of("ccc", "A", "bb", "a", "B"));
        mixed.sort(Comparator.comparingInt(String::length)
                .thenComparing(String.CASE_INSENSITIVE_ORDER));
        System.out.println("Length, then ignore case: " + mixed);

        // naturalOrder y reverseOrder
        List<Integer> numbers = new ArrayList<>(List.of(5, 1, 9, 3));
        numbers.sort(Comparator.naturalOrder());
        System.out.println("\nnaturalOrder:            " + numbers);
        numbers.sort(Comparator.reverseOrder());
        System.out.println("reverseOrder:            " + numbers);
        Collections.sort(numbers, Collections.reverseOrder());
        System.out.println("Collections.reverseOrder:" + " " + numbers);
    }

    private static List<String> names(List<Person> people) {
        return people.stream().map(person -> person.forename() + " " + person.surname() + " (" + person.age() + ")").toList();
    }
}

record Person(String forename, String surname, int age) { }
```

Salida:

```text
By surname:              [Amelia Brown (41), Bob Jones (25), Oliver Jones (25), Alice Smith (30), Charlotte Smith (22)]
By age:                  [Charlotte Smith (22), Bob Jones (25), Oliver Jones (25), Alice Smith (30), Amelia Brown (41)]
Surname, then forename:  [Amelia Brown (41), Bob Jones (25), Oliver Jones (25), Alice Smith (30), Charlotte Smith (22)]
Age descending:          [Amelia Brown (41), Alice Smith (30), Bob Jones (25), Oliver Jones (25), Charlotte Smith (22)]
Age desc, then surname:  [Amelia Brown (41), Alice Smith (30), Bob Jones (25), Oliver Jones (25), Charlotte Smith (22)]

Case-insensitive:        [Apple, apple, banana, cherry]
Length, then ignore case: [A, a, B, bb, ccc]

naturalOrder:            [1, 3, 5, 9]
reverseOrder:            [9, 5, 3, 1]
Collections.reverseOrder: [9, 5, 3, 1]
```

Cómo leer una cadena de comparadores: `Comparator.comparingInt(Person::age).reversed().thenComparing(Person::surname)` significa "edad de mayor a menor y, si dos personas tienen la misma edad, apellido de la A a la Z".

### 4.4 Ojo con la inferencia de tipos en lambdas

Con referencias a método (`Person::surname`), encadenar `.reversed()` funciona sin problemas. Pero con una lambda sin tipo, el compilador puede no deducir el tipo del parámetro:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class InferenceProblemDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(new Person("Bob", "Jones"), new Person("Alice", "Smith")));

        // Con lambda: el compilador no puede deducir el tipo de "person"
        people.sort(Comparator.comparing(person -> person.surname()).reversed());
    }
}

record Person(String forename, String surname) { }
```

Resultado al compilar:

```text
InferenceProblemDemo.java:11: error: cannot find symbol
        people.sort(Comparator.comparing(person -> person.surname()).reversed());
                                                         ^
  symbol:   method surname()
  location: variable person of type Object
1 error
error: compilation failed
```

Tres formas de resolverlo:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class OkInferenceDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(new Person("Bob", "Jones"), new Person("Alice", "Smith")));

        // Opción 1: referencia a método
        people.sort(Comparator.comparing(Person::surname).reversed());
        System.out.println("Method reference:  " + people);

        // Opción 2: declarar el tipo del parámetro de la lambda
        people.sort(Comparator.comparing((Person person) -> person.surname()).reversed());
        System.out.println("Typed lambda:      " + people);

        // Opción 3: pasar el orden inverso como comparador de la clave
        people.sort(Comparator.comparing(person -> person.surname(), Comparator.reverseOrder()));
        System.out.println("Key comparator:    " + people);
    }
}

record Person(String forename, String surname) { }
```

Salida:

```text
Method reference:  [Person[forename=Alice, surname=Smith], Person[forename=Bob, surname=Jones]]
Typed lambda:      [Person[forename=Alice, surname=Smith], Person[forename=Bob, surname=Jones]]
Key comparator:    [Person[forename=Alice, surname=Smith], Person[forename=Bob, surname=Jones]]
```

### 4.5 Comparadores y valores `null`

Según la documentación, a diferencia de `Comparable`, un `Comparator` **puede** permitir comparar argumentos `null`. Los métodos `nullsFirst` y `nullsLast` envuelven otro comparador para conseguirlo.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class NullHandlingDemo {

    public static void main(String[] args) {
        List<String> nicknames = new ArrayList<>(Arrays.asList("zed", null, "amy", null, "bob"));

        nicknames.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
        System.out.println("nullsFirst: " + nicknames);

        nicknames.sort(Comparator.nullsLast(Comparator.naturalOrder()));
        System.out.println("nullsLast:  " + nicknames);

        // nullsLast combinado con reversed
        nicknames.sort(Comparator.nullsLast(Comparator.<String>naturalOrder().reversed()));
        System.out.println("nullsLast + reversed order: " + nicknames);

        // Sin tratamiento de null: NullPointerException
        try {
            Collections.sort(nicknames);
        } catch (NullPointerException exception) {
            System.out.println("\nCollections.sort with null elements -> NullPointerException");
        }

        try {
            "abc".compareTo(null);
        } catch (NullPointerException exception) {
            System.out.println("\"abc\".compareTo(null) -> NullPointerException");
        }

        // El null puede estar en la clave: se protege la clave, no el objeto
        List<Person> people = new ArrayList<>(List.of(
                new Person("Alice", null),
                new Person("Bob", "Jones"),
                new Person("Charlotte", "Brown")));
        people.sort(Comparator.comparing(Person::surname, Comparator.nullsLast(Comparator.naturalOrder())));
        System.out.println("\nBy surname, null surname last: " + people);
    }
}

record Person(String forename, String surname) { }
```

Salida:

```text
nullsFirst: [null, null, amy, bob, zed]
nullsLast:  [amy, bob, zed, null, null]
nullsLast + reversed order: [zed, bob, amy, null, null]

Collections.sort with null elements -> NullPointerException
"abc".compareTo(null) -> NullPointerException

By surname, null surname last: [Person[forename=Charlotte, surname=Brown], Person[forename=Bob, surname=Jones], Person[forename=Alice, surname=null]]
```

Distingue dos casos:

- **Elementos `null` dentro de la lista:** se envuelve el comparador completo con `nullsFirst`/`nullsLast`.
- **Objetos válidos con un campo `null`** (última parte del ejemplo): se protege solo la clave con `comparing(Person::surname, Comparator.nullsLast(...))`.

Sin ese tratamiento, se lanza `NullPointerException`, como se ve en el ejemplo.

### 4.6 El contrato de `compare`

Igual que `compareTo`, `compare(x, y)` debe cumplir (según la documentación):

1. `sgn(compare(x, y)) == -sgn(compare(y, x))` para todo `x` e `y`.
2. Transitividad: `compare(x, y) > 0` y `compare(y, z) > 0` implican `compare(x, z) > 0`.
3. Si `compare(x, y) == 0`, entonces `x` e `y` dan el mismo signo al compararse con cualquier `z`.
4. Recomendado (no obligatorio): que `(compare(x, y) == 0) == x.equals(y)`.

Puede lanzar `NullPointerException` si recibe un `null` y no admite nulos, y `ClassCastException` si los tipos de los argumentos impiden compararlos.

### 4.7 Por qué `Comparator` es una interfaz funcional

Una **interfaz funcional** es una interfaz con **un solo método abstracto**. Puede usarse como destino de una lambda o de una referencia a método. `Comparator` está anotada con `@FunctionalInterface` y su único método abstracto es `compare`.

Puede parecer que tiene dos, porque también declara `equals(Object)`. Pero `equals` ya existe en `Object`, y la especificación de las interfaces funcionales no cuenta como abstracto un método que sobrescribe un método público de `Object`. Por eso `Comparator` cumple la condición.

---

## 5. Dónde se usan

### 5.1 Ordenar y consultar

Un mismo `Film` (con orden natural por año) y un `Comparator` para cada uso:

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.TreeMap;
import java.util.TreeSet;

public class WhereToUseDemo {

    public static void main(String[] args) {
        Comparator<Film> byRatingDescending = Comparator.comparingDouble(Film::rating).reversed();

        List<Film> films = new ArrayList<>(List.of(
                new Film("Star Wars", 8.7, 1977),
                new Film("The Force Awakens", 8.3, 2015),
                new Film("The Empire Strikes Back", 8.8, 1980),
                new Film("Return of the Jedi", 8.4, 1983)));

        // List.sort
        films.sort(byRatingDescending);
        System.out.println("List.sort:         " + titles(films));

        // Collections.sort con Comparator
        Collections.sort(films, Comparator.comparing(Film::title));
        System.out.println("Collections.sort:  " + titles(films));

        // List.sort(null) = orden natural (Film es Comparable por año)
        films.sort(null);
        System.out.println("List.sort(null):   " + titles(films));

        // Arrays.sort con un array de objetos
        Film[] array = films.toArray(new Film[0]);
        Arrays.sort(array, byRatingDescending);
        System.out.println("Arrays.sort:       " + Arrays.stream(array).map(Film::title).toList());

        // Streams
        System.out.println("stream.sorted():   " + films.stream().sorted(byRatingDescending).map(Film::title).toList());
        System.out.println("stream.min:        " + films.stream().min(Comparator.comparingInt(Film::releaseYear)).get().title());
        System.out.println("stream.max:        " + films.stream().max(Comparator.comparingDouble(Film::rating)).get().title());

        // Collections.min / max
        System.out.println("Collections.max:   " + Collections.max(films, Comparator.comparing(Film::title)).title());

        // TreeSet: mantiene los elementos ordenados al insertarlos
        TreeSet<Film> sortedByTitle = new TreeSet<>(Comparator.comparing(Film::title));
        sortedByTitle.addAll(films);
        System.out.println("\nTreeSet:           " + titles(sortedByTitle));

        // TreeMap: ordena por clave
        TreeMap<String, Integer> yearByTitle = new TreeMap<>(Comparator.reverseOrder());
        films.forEach(film -> yearByTitle.put(film.title(), film.releaseYear()));
        System.out.println("TreeMap (reverse): " + yearByTitle);

        // PriorityQueue: poll() devuelve siempre el "menor" según el comparador
        PriorityQueue<Film> queue = new PriorityQueue<>(byRatingDescending);
        queue.addAll(films);
        System.out.print("PriorityQueue:     ");
        while (!queue.isEmpty()) {
            System.out.print(queue.poll().title() + " | ");
        }
        System.out.println();
    }

    private static List<String> titles(Iterable<Film> films) {
        List<String> titles = new ArrayList<>();
        films.forEach(film -> titles.add(film.title()));
        return titles;
    }
}

record Film(String title, double rating, int releaseYear) implements Comparable<Film> {
    @Override
    public int compareTo(Film other) {
        return Integer.compare(this.releaseYear, other.releaseYear);
    }
}
```

Salida:

```text
List.sort:         [The Empire Strikes Back, Star Wars, Return of the Jedi, The Force Awakens]
Collections.sort:  [Return of the Jedi, Star Wars, The Empire Strikes Back, The Force Awakens]
List.sort(null):   [Star Wars, The Empire Strikes Back, Return of the Jedi, The Force Awakens]
Arrays.sort:       [The Empire Strikes Back, Star Wars, Return of the Jedi, The Force Awakens]
stream.sorted():   [The Empire Strikes Back, Star Wars, Return of the Jedi, The Force Awakens]
stream.min:        Star Wars
stream.max:        The Empire Strikes Back
Collections.max:   The Force Awakens

TreeSet:           [Return of the Jedi, Star Wars, The Empire Strikes Back, The Force Awakens]
TreeMap (reverse): {The Force Awakens=2015, The Empire Strikes Back=1980, Star Wars=1977, Return of the Jedi=1983}
PriorityQueue:     The Empire Strikes Back | Star Wars | Return of the Jedi | The Force Awakens | 
```

| Uso | Con orden natural (`Comparable`) | Con `Comparator` |
|---|---|---|
| Ordenar una lista | `Collections.sort(list)` o `list.sort(null)` | `list.sort(cmp)` o `Collections.sort(list, cmp)` |
| Ordenar un array de objetos | `Arrays.sort(array)` | `Arrays.sort(array, cmp)` |
| Streams | `stream.sorted()` | `stream.sorted(cmp)`, `stream.min(cmp)`, `stream.max(cmp)` |
| Mínimo y máximo | `Collections.min(c)`, `Collections.max(c)` | `Collections.min(c, cmp)`, `Collections.max(c, cmp)` |
| Conjunto ordenado | `new TreeSet<>()` | `new TreeSet<>(cmp)` |
| Mapa ordenado por clave | `new TreeMap<>()` | `new TreeMap<>(cmp)` |
| Cola con prioridad | `new PriorityQueue<>()` | `new PriorityQueue<>(cmp)` |

Notas:

- Según la documentación de `Collections.sort`, pasar un comparador `null` significa "usa el orden natural". Por eso `list.sort(null)` funciona (se ve en la salida).
- En una `PriorityQueue`, `poll()` devuelve siempre el elemento "menor" según el orden; en el ejemplo, con el comparador de valoración descendente, salen de mayor a menor valoración.
- Los arrays de tipos primitivos (`int[]`, `double[]`…) se ordenan con `Arrays.sort(array)` y no admiten `Comparator`, porque este trabaja con objetos.

### 5.2 La ordenación es estable

La documentación de `Collections.sort` garantiza que la ordenación es **estable**: los elementos que se consideran iguales **no se reordenan** entre sí.

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class StableSortDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(
                new Person("Alice", 30),
                new Person("Bob", 25),
                new Person("Charlotte", 30),
                new Person("Oliver", 25),
                new Person("Amelia", 30)));

        System.out.println("Original:        " + people);

        // Ordenar solo por edad: los empatados conservan su orden relativo original
        people.sort(Comparator.comparingInt(Person::age));
        System.out.println("Sorted by age:   " + people);
    }
}

record Person(String forename, int age) { }
```

Salida:

```text
Original:        [Person[forename=Alice, age=30], Person[forename=Bob, age=25], Person[forename=Charlotte, age=30], Person[forename=Oliver, age=25], Person[forename=Amelia, age=30]]
Sorted by age:   [Person[forename=Bob, age=25], Person[forename=Oliver, age=25], Person[forename=Alice, age=30], Person[forename=Charlotte, age=30], Person[forename=Amelia, age=30]]
```

Alice, Charlotte y Amelia (todas con 30 años) conservan su orden original relativo, igual que Bob y Oliver (25 años). Esto permite ordenar en varias pasadas, aunque `thenComparing` suele ser más claro.

---

## 6. Comparable frente a Comparator

| Característica | `Comparable<T>` | `Comparator<T>` |
|---|---|---|
| **Qué define** | El orden **natural** de la clase | Un orden **externo** o personalizado |
| **Paquete** | `java.lang` | `java.util` |
| **Método abstracto** | `int compareTo(T o)` | `int compare(T o1, T o2)` |
| **Dónde se implementa** | En la **propia clase** a ordenar | En otra clase, una lambda o mediante métodos de fábrica |
| **Número de órdenes** | Uno solo | Tantos como se necesiten |
| **Requiere modificar la clase** | Sí | No |
| **Admite `null`** | No: `compareTo(null)` debería lanzar `NullPointerException` | Puede admitirlo (`nullsFirst`, `nullsLast`) |
| **Se usa como lambda** | No es lo habitual | Sí (`@FunctionalInterface`) |

**Criterio práctico para elegir:**

- Usa **`Comparable`** cuando exista un orden "obvio" y único para tu clase y esa clase sea tuya (números de empleado, fechas, versiones…).
- Usa **`Comparator`** cuando necesites varios criterios, cuando no puedas modificar la clase (por ejemplo, viene de una librería), o cuando el orden sea puntual para una pantalla o un informe.
- **Puedes combinar ambos:** una clase con un orden natural (`Comparable`) y varios comparadores adicionales para otros criterios. Es lo que hace el ejemplo completo de la [sección 9](#9-ejemplo-completo).

---

## 7. Errores típicos

### 7.1 Comparar restando

Es un error muy frecuente escribir `return a - b;` en un comparador de enteros. Falla porque la resta de `int` puede **desbordarse**. Con decimales, convertir la resta a `int` **pierde la parte decimal**:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class SubtractionPitfallDemo {

    public static void main(String[] args) {
        // Problema 1: la resta de enteros puede desbordarse
        Comparator<Integer> bySubtraction = (first, second) -> first - second; // MAL

        System.out.println("MAX_VALUE - (-1) = " + (Integer.MAX_VALUE - (-1)) + "   (debería ser positivo)");
        System.out.println("bySubtraction.compare(MAX_VALUE, -1) = "
                + bySubtraction.compare(Integer.MAX_VALUE, -1) + "   (dice que MAX_VALUE < -1)");

        List<Integer> numbers = new ArrayList<>(List.of(-1, Integer.MAX_VALUE));
        numbers.sort(bySubtraction);
        System.out.println("Sorted with subtraction:     " + numbers + "   (orden incorrecto)");

        numbers.sort(Integer::compare); // BIEN
        System.out.println("Sorted with Integer.compare: " + numbers);

        // Problema 2: convertir a int una resta de decimales pierde la parte decimal
        List<Film> films = new ArrayList<>(List.of(
                new Film("Star Wars", 8.7),
                new Film("The Empire Strikes Back", 8.8),
                new Film("Return of the Jedi", 8.4),
                new Film("The Force Awakens", 8.3)));

        films.sort((first, second) -> (int) (first.rating() - second.rating())); // MAL
        System.out.println("\n(int) (a - b):   " + films.stream().map(Film::title).toList());

        films.sort((first, second) -> Double.compare(first.rating(), second.rating())); // BIEN
        System.out.println("Double.compare:  " + films.stream().map(Film::title).toList());

        // Double.compare también define qué hacer con casos especiales
        System.out.println("\nDouble.compare(0.0, -0.0)        = " + Double.compare(0.0, -0.0));
        System.out.println("Double.compare(Double.NaN, 1.0)  = " + Double.compare(Double.NaN, 1.0));
    }
}

record Film(String title, double rating) { }
```

Salida:

```text
MAX_VALUE - (-1) = -2147483648   (debería ser positivo)
bySubtraction.compare(MAX_VALUE, -1) = -2147483648   (dice que MAX_VALUE < -1)
Sorted with subtraction:     [2147483647, -1]   (orden incorrecto)
Sorted with Integer.compare: [-1, 2147483647]

(int) (a - b):   [Star Wars, The Empire Strikes Back, Return of the Jedi, The Force Awakens]
Double.compare:  [The Force Awakens, Return of the Jedi, Star Wars, The Empire Strikes Back]

Double.compare(0.0, -0.0)        = 1
Double.compare(Double.NaN, 1.0)  = 1
```

Lo que muestra:

- `MAX_VALUE - (-1)` desborda y da un número negativo, así que el comparador cree que `MAX_VALUE` es menor que `-1` y ordena mal.
- `(int) (a - b)` con estas valoraciones da siempre `0`, porque todas las diferencias son menores que 1 y se truncan. Todas las películas parecen "iguales" y la lista queda sin ordenar.
- `Double.compare` también define el comportamiento con casos especiales como `-0.0` y `NaN`.

**Regla:** usa siempre `Integer.compare`, `Long.compare`, `Double.compare`… o `Comparator.comparingInt(...)` y compañía.

### 7.2 Orden incoherente con `equals`

`TreeSet` y `TreeMap` deciden si dos elementos son "el mismo" usando **el comparador** (`compare`/`compareTo`), **no** `equals`. Si tu orden devuelve `0` para objetos distintos, la colección descartará uno de ellos:

```java
import java.math.BigDecimal;
import java.util.Comparator;
import java.util.HashSet;
import java.util.Set;
import java.util.TreeSet;

public class EqualsConsistencyPitfallDemo {

    public static void main(String[] args) {
        // 1) Comparator que solo mira la longitud: "cat" y "dog" son "iguales" para el TreeSet
        Set<String> byLength = new TreeSet<>(Comparator.comparingInt(String::length));
        byLength.add("cat");
        byLength.add("dog");
        byLength.add("bird");
        System.out.println("TreeSet by length: " + byLength + "  size = " + byLength.size());

        // 2) Orden natural de Film: solo por año. Dos películas distintas del mismo año chocan
        Set<Film> films = new TreeSet<>();
        films.add(new Film("Film A", 1980));
        films.add(new Film("Film B", 1980));
        System.out.println("TreeSet<Film>:     " + films + "  size = " + films.size());

        // 3) Caso real del JDK: BigDecimal
        BigDecimal fourPointZero = new BigDecimal("4.0");
        BigDecimal fourPointZeroZero = new BigDecimal("4.00");
        System.out.println("\n4.0 equals 4.00:      " + fourPointZero.equals(fourPointZeroZero));
        System.out.println("4.0 compareTo 4.00:   " + fourPointZero.compareTo(fourPointZeroZero));

        Set<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(fourPointZero);
        hashSet.add(fourPointZeroZero);
        Set<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(fourPointZero);
        treeSet.add(fourPointZeroZero);
        System.out.println("HashSet size (uses equals):     " + hashSet.size());
        System.out.println("TreeSet size (uses compareTo):  " + treeSet.size());
    }
}

record Film(String title, int releaseYear) implements Comparable<Film> {
    @Override
    public int compareTo(Film other) {
        return Integer.compare(this.releaseYear, other.releaseYear);
    }
}
```

Salida:

```text
TreeSet by length: [cat, bird]  size = 2
TreeSet<Film>:     [Film[title=Film A, releaseYear=1980]]  size = 1

4.0 equals 4.00:      false
4.0 compareTo 4.00:   0
HashSet size (uses equals):     2
TreeSet size (uses compareTo):  1
```

- `"cat"` y `"dog"` tienen la misma longitud; el `TreeSet` cree que son el mismo elemento y se queda con el primero.
- Dos `Film` distintos del mismo año colisionan en un `TreeSet` porque el orden natural solo mira el año.
- `BigDecimal` es un caso del propio JDK: `4.0` y `4.00` **no** son `equals`, pero su `compareTo` da `0`. La documentación de `Comparable` lo cita como excepción a la regla de que el orden natural sea coherente con `equals`.

**Cómo evitarlo:** si un orden puede dar `0` para objetos distintos, añade campos de desempate hasta que solo den `0` los objetos realmente iguales, o ten presente esta limitación al usar `TreeSet`/`TreeMap`.

### 7.3 Poner mal el `reversed()`

`reversed()` invierte **el comparador sobre el que se llama**, que es toda la cadena construida hasta ese punto:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class ReversedPlacementPitfallDemo {

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(
                new Person("Alice", "Smith"),
                new Person("Bob", "Jones"),
                new Person("Charlotte", "Smith"),
                new Person("Oliver", "Jones")));

        // reversed() al final invierte TODA la cadena (apellido y nombre)
        people.sort(Comparator.comparing(Person::surname)
                .thenComparing(Person::forename)
                .reversed());
        System.out.println("reversed at the end:   " + names(people));

        // reversed() justo después del primer criterio solo invierte ese criterio
        people.sort(Comparator.comparing(Person::surname)
                .reversed()
                .thenComparing(Person::forename));
        System.out.println("reversed after first:  " + names(people));

        // Otra forma: indicar el comparador de la clave
        people.sort(Comparator.comparing(Person::surname, Comparator.reverseOrder())
                .thenComparing(Person::forename));
        System.out.println("reverseOrder for key:  " + names(people));
    }

    private static List<String> names(List<Person> people) {
        return people.stream().map(person -> person.forename() + " " + person.surname()).toList();
    }
}

record Person(String forename, String surname) { }
```

Salida:

```text
reversed at the end:   [Charlotte Smith, Alice Smith, Oliver Jones, Bob Jones]
reversed after first:  [Alice Smith, Charlotte Smith, Bob Jones, Oliver Jones]
reverseOrder for key:  [Alice Smith, Charlotte Smith, Bob Jones, Oliver Jones]
```

- Al final de la cadena, invierte apellido **y** nombre.
- Justo después del primer criterio, invierte solo el apellido y el nombre queda ascendente.
- `comparing(clave, Comparator.reverseOrder())` es otra forma clara de invertir solo un criterio.

### 7.4 `TreeSet` (o `TreeMap`) con una clase que no es `Comparable`

Este código **compila** pero falla al ejecutarse si la clase no es `Comparable` y no se da ningún `Comparator`:

```java
import java.util.Comparator;
import java.util.TreeSet;

public class NotComparableAtRuntimeDemo {

    public static void main(String[] args) {
        // Este código COMPILA, pero Film no es Comparable y no se ha dado ningún Comparator
        TreeSet<Film> films = new TreeSet<>();
        try {
            films.add(new Film("Star Wars", 1977));
        } catch (ClassCastException exception) {
            System.out.println("ClassCastException: " + exception.getMessage());
        }

        // Solución: pasar un Comparator al constructor
        TreeSet<Film> fixed = new TreeSet<>(Comparator.comparingInt(Film::releaseYear));
        fixed.add(new Film("Star Wars", 1977));
        System.out.println("With Comparator: " + fixed);
    }
}

record Film(String title, int releaseYear) { }
```

Salida:

```text
ClassCastException: class Film cannot be cast to class java.lang.Comparable (Film is in unnamed module of loader com.sun.tools.javac.launcher.Main$MemoryClassLoader @161479c6; java.lang.Comparable is in module java.base of loader 'bootstrap')
With Comparator: [Film[title=Star Wars, releaseYear=1977]]
```

**Solución:** haz que la clase implemente `Comparable` o pasa un `Comparator` al constructor.

### 7.5 Comparadores que rompen el contrato

Un comparador incoherente (por ejemplo, uno que devuelva resultados aleatorios) rompe las reglas de antisimetría y transitividad. La documentación indica que `Collections.sort` **puede** lanzar `IllegalArgumentException` si detecta la violación, pero dice que la detección es **opcional**:

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class ContractViolationDemo {

    public static void main(String[] args) {
        Random random = new Random(1);
        List<Integer> numbers = new ArrayList<>();
        for (int i = 0; i < 5000; i++) {
            numbers.add(random.nextInt(1000));
        }

        try {
            // Comparador que rompe el contrato: ¡devuelve un resultado aleatorio!
            numbers.sort((first, second) -> random.nextInt(3) - 1);
            System.out.println("No exception this time (the check is optional).");
        } catch (IllegalArgumentException exception) {
            System.out.println("IllegalArgumentException: " + exception.getMessage());
        }
    }
}
```

Salida (en mi prueba con OpenJDK 21.0.10):

```text
IllegalArgumentException: Comparison method violates its general contract!
```

En las pruebas que hice, con la misma técnica y listas de 50 a 1000 elementos **no** se lanzó ninguna excepción; con 5000 sí. Conclusión: **no cuentes con que Java te avise**. Un comparador que rompe el contrato puede producir un orden incorrecto sin ningún error.

### 7.6 Olvidar los `null`

`Comparable.compareTo(null)` debería lanzar `NullPointerException`, y `Collections.sort` falla si la lista contiene `null` y no usas `nullsFirst`/`nullsLast` (ver [sección 4.5](#45-comparadores-y-valores-null)).

---

## 8. Novedades y estado en Java 21

- **El contrato de `Comparable` y `Comparator` descrito en la Javadoc de Java 21 es el que se ha usado en esta guía.** `Comparator.comparing`, `thenComparing`, `reversed`, `nullsFirst` y compañía siguen vigentes.
- **Records** (Java 16): permiten definir clases de datos en una línea y se llevan muy bien con `Comparator.comparing(Film::title)`.
- **Colecciones secuenciadas (Java 21, JEP 431):** `List` y los conjuntos ordenados como `TreeSet` disponen de `getFirst()`, `getLast()` y `reversed()`. Combinados con un buen orden, dan código muy legible:

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.TreeSet;

public class Java21FeaturesDemo {

    public static void main(String[] args) {
        List<Film> films = new ArrayList<>(List.of(
                new Film("Star Wars", 8.7, 1977),
                new Film("The Empire Strikes Back", 8.8, 1980),
                new Film("Return of the Jedi", 8.4, 1983),
                new Film("The Force Awakens", 8.3, 2015)));

        films.sort(Comparator.comparingDouble(Film::rating).reversed());

        // Java 21: List tiene getFirst(), getLast() y reversed()
        System.out.println("Best rated:  " + films.getFirst().title());
        System.out.println("Worst rated: " + films.getLast().title());
        System.out.println("Reversed view: " + films.reversed().stream().map(Film::title).toList());

        // Java 21: los conjuntos ordenados también tienen getFirst(), getLast() y reversed()
        TreeSet<Film> byYear = new TreeSet<>(Comparator.comparingInt(Film::releaseYear));
        byYear.addAll(films);
        System.out.println("\nOldest: " + byYear.getFirst().title());
        System.out.println("Newest: " + byYear.getLast().title());
        System.out.println("Newest first: " + byYear.reversed().stream().map(Film::title).toList());
    }
}

record Film(String title, double rating, int releaseYear) { }
```

Salida:

```text
Best rated:  The Empire Strikes Back
Worst rated: The Force Awakens
Reversed view: [The Force Awakens, Return of the Jedi, Star Wars, The Empire Strikes Back]

Oldest: Star Wars
Newest: The Force Awakens
Newest first: [The Force Awakens, Return of the Jedi, The Empire Strikes Back, Star Wars]
```

Estas novedades **no sustituyen** a `Comparable` ni a `Comparator`: son un complemento para trabajar con el resultado de la ordenación. No he contrastado esta guía con versiones posteriores a Java 21 ejemplo por ejemplo.

---

## 9. Ejemplo completo

Un pequeño informe de empleados que combina todo: un orden natural (`Comparable`) por número de empleado, varios comparadores reutilizables publicados como constantes, consultas de mínimo y máximo, y un `TreeSet`.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.TreeSet;

public class EmployeeReportDemo {

    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>(List.of(
                new Employee(104, "Alice", "Smith", Department.FINANCE, 48_000),
                new Employee(101, "Bob", "Jones", Department.ENGINEERING, 55_000),
                new Employee(103, "Charlotte", "Smith", Department.ENGINEERING, 61_000),
                new Employee(102, "Oliver", "Brown", Department.MARKETING, 41_000),
                new Employee(105, "Amelia", "Jones", Department.FINANCE, 52_000)));

        // 1) Orden natural (Comparable): por número de empleado
        Collections.sort(employees);
        print("Natural order (employee number)", employees);

        // 2) Comparadores alternativos (Comparator)
        employees.sort(Employee.BY_SURNAME_THEN_FORENAME);
        print("By surname, then forename", employees);

        employees.sort(Employee.BY_SALARY_DESCENDING);
        print("By salary, highest first", employees);

        employees.sort(Employee.BY_DEPARTMENT_THEN_SALARY_DESCENDING);
        print("By department, then salary (highest first)", employees);

        // 3) Consultas con los mismos comparadores
        System.out.println("Highest paid: " + Collections.max(employees, Comparator.comparingInt(Employee::salary)));
        System.out.println("Lowest paid:  " + Collections.min(employees, Comparator.comparingInt(Employee::salary)));

        // 4) Colección que se mantiene ordenada
        TreeSet<Employee> directory = new TreeSet<>(Employee.BY_SURNAME_THEN_FORENAME);
        directory.addAll(employees);
        System.out.println("\nFirst in directory: " + directory.first());
        System.out.println("Last in directory:  " + directory.last());
    }

    private static void print(String heading, List<Employee> employees) {
        System.out.println("\n" + heading + ":");
        employees.forEach(employee -> System.out.println("  " + employee));
    }
}

enum Department { ENGINEERING, FINANCE, MARKETING }

record Employee(int employeeNumber, String forename, String surname, Department department, int salary)
        implements Comparable<Employee> {

    // Comparadores reutilizables, publicados como constantes
    static final Comparator<Employee> BY_SURNAME_THEN_FORENAME =
            Comparator.comparing(Employee::surname).thenComparing(Employee::forename);

    static final Comparator<Employee> BY_SALARY_DESCENDING =
            Comparator.comparingInt(Employee::salary).reversed();

    static final Comparator<Employee> BY_DEPARTMENT_THEN_SALARY_DESCENDING =
            Comparator.comparing(Employee::department)
                    .thenComparing(Comparator.comparingInt(Employee::salary).reversed());

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.employeeNumber, other.employeeNumber);
    }
}
```

Salida:

```text

Natural order (employee number):
  Employee[employeeNumber=101, forename=Bob, surname=Jones, department=ENGINEERING, salary=55000]
  Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]
  Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]
  Employee[employeeNumber=104, forename=Alice, surname=Smith, department=FINANCE, salary=48000]
  Employee[employeeNumber=105, forename=Amelia, surname=Jones, department=FINANCE, salary=52000]

By surname, then forename:
  Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]
  Employee[employeeNumber=105, forename=Amelia, surname=Jones, department=FINANCE, salary=52000]
  Employee[employeeNumber=101, forename=Bob, surname=Jones, department=ENGINEERING, salary=55000]
  Employee[employeeNumber=104, forename=Alice, surname=Smith, department=FINANCE, salary=48000]
  Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]

By salary, highest first:
  Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]
  Employee[employeeNumber=101, forename=Bob, surname=Jones, department=ENGINEERING, salary=55000]
  Employee[employeeNumber=105, forename=Amelia, surname=Jones, department=FINANCE, salary=52000]
  Employee[employeeNumber=104, forename=Alice, surname=Smith, department=FINANCE, salary=48000]
  Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]

By department, then salary (highest first):
  Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]
  Employee[employeeNumber=101, forename=Bob, surname=Jones, department=ENGINEERING, salary=55000]
  Employee[employeeNumber=105, forename=Amelia, surname=Jones, department=FINANCE, salary=52000]
  Employee[employeeNumber=104, forename=Alice, surname=Smith, department=FINANCE, salary=48000]
  Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]
Highest paid: Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]
Lowest paid:  Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]

First in directory: Employee[employeeNumber=102, forename=Oliver, surname=Brown, department=MARKETING, salary=41000]
Last in directory:  Employee[employeeNumber=103, forename=Charlotte, surname=Smith, department=ENGINEERING, salary=61000]
```

Puntos a observar:

- `Employee` es `Comparable` (número de empleado) y además expone comparadores alternativos como constantes `static final`.
- `Comparator.comparing(Employee::department)` funciona con el `enum` `Department` porque los `enum` son `Comparable`.
- `BY_DEPARTMENT_THEN_SALARY_DESCENDING` combina el orden natural de un `enum` con un desempate invertido, mediante `thenComparing(Comparator...)`.

---

## 10. Ejercicios

Intenta resolverlos sin mirar. Después, despliega las soluciones.

1. Ordena una lista de palabras por **longitud** y, si empatan, **alfabéticamente sin distinguir mayúsculas de minúsculas**. Datos: `kiwi`, `Fig`, `banana`, `apple`, `Plum`, `date`.
2. Dado un `record Student(String forename, String surname, int mark)`, ordena por **nota descendente** y, en caso de empate, por **apellido** y luego por **nombre**.
3. Crea un `record Book(String title)` que sea `Comparable` con orden natural por título **sin distinguir mayúsculas**. Pista: `String.CASE_INSENSITIVE_ORDER`. Piensa: ¿es este orden coherente con `equals` del `record`?
4. Obtén la persona **más joven** y la **más mayor** de una lista sin ordenarla. Pista: `Collections.min` y `Collections.max`.
5. Ordena personas por apellido, que **puede ser `null`** (los `null` al final), y luego por nombre.

<details>
<summary><strong>Ver soluciones (código completo ejecutado)</strong></summary>

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ExerciseSolutions {

    public static void main(String[] args) {
        exercise1();
        exercise2();
        exercise3();
        exercise4();
        exercise5();
    }

    // Ejercicio 1: ordenar palabras por longitud y, en empate, alfabéticamente sin distinguir mayúsculas
    static void exercise1() {
        List<String> words = new ArrayList<>(List.of("kiwi", "Fig", "banana", "apple", "Plum", "date"));
        words.sort(Comparator.comparingInt(String::length)
                .thenComparing(String.CASE_INSENSITIVE_ORDER));
        System.out.println("Exercise 1: " + words);
    }

    // Ejercicio 2: estudiantes por nota descendente, luego apellido y luego nombre
    static void exercise2() {
        List<Student> students = new ArrayList<>(List.of(
                new Student("Alice", "Smith", 72),
                new Student("Bob", "Jones", 85),
                new Student("Charlotte", "Brown", 85),
                new Student("Oliver", "Smith", 72),
                new Student("Amelia", "Jones", 91)));
        students.sort(Comparator.comparingInt(Student::mark).reversed()
                .thenComparing(Student::surname)
                .thenComparing(Student::forename));
        System.out.println("Exercise 2:");
        students.forEach(student -> System.out.println("  " + student));
    }

    // Ejercicio 3: libros con orden natural por título sin distinguir mayúsculas
    static void exercise3() {
        List<Book> books = new ArrayList<>(List.of(
                new Book("the hobbit"), new Book("Emma"), new Book("Dracula"), new Book("Persuasion")));
        Collections.sort(books);
        System.out.println("Exercise 3: " + books);
    }

    // Ejercicio 4: persona más joven y más mayor
    static void exercise4() {
        List<Person> people = List.of(
                new Person("Alice", "Smith", 30),
                new Person("Bob", "Jones", 25),
                new Person("Charlotte", "Brown", 41));
        Person youngest = Collections.min(people, Comparator.comparingInt(Person::age));
        Person oldest = Collections.max(people, Comparator.comparingInt(Person::age));
        System.out.println("Exercise 4: youngest = " + youngest.forename() + ", oldest = " + oldest.forename());
    }

    // Ejercicio 5: apellidos que pueden ser null (null al final), luego por nombre
    static void exercise5() {
        List<Person> people = new ArrayList<>(Arrays.asList(
                new Person("Alice", null, 30),
                new Person("Bob", "Jones", 25),
                new Person("Charlotte", "Brown", 41),
                new Person("Amelia", null, 19)));
        people.sort(Comparator
                .comparing(Person::surname, Comparator.nullsLast(Comparator.<String>naturalOrder()))
                .thenComparing(Person::forename));
        System.out.println("Exercise 5:");
        people.forEach(person -> System.out.println("  " + person));
    }
}

record Student(String forename, String surname, int mark) { }

record Person(String forename, String surname, int age) { }

record Book(String title) implements Comparable<Book> {
    @Override
    public int compareTo(Book other) {
        return String.CASE_INSENSITIVE_ORDER.compare(this.title, other.title);
    }
}
```

Salida:

```text
Exercise 1: [Fig, date, kiwi, Plum, apple, banana]
Exercise 2:
  Student[forename=Amelia, surname=Jones, mark=91]
  Student[forename=Charlotte, surname=Brown, mark=85]
  Student[forename=Bob, surname=Jones, mark=85]
  Student[forename=Alice, surname=Smith, mark=72]
  Student[forename=Oliver, surname=Smith, mark=72]
Exercise 3: [Book[title=Dracula], Book[title=Emma], Book[title=Persuasion], Book[title=the hobbit]]
Exercise 4: youngest = Bob, oldest = Charlotte
Exercise 5:
  Person[forename=Charlotte, surname=Brown, age=41]
  Person[forename=Bob, surname=Jones, age=25]
  Person[forename=Alice, surname=null, age=30]
  Person[forename=Amelia, surname=null, age=19]
```

**Comentario del ejercicio 3:** no es coherente con `equals`. El `equals` generado por el `record` distingue mayúsculas, así que `Book("emma")` y `Book("Emma")` no serían `equals`, pero su `compareTo` daría `0`. Es el mismo problema de la [sección 7.2](#72-orden-incoherente-con-equals).

</details>

---

## 11. Chuleta de repaso

- `Comparable` → orden **natural**, `compareTo(T o)`, paquete `java.lang`, se implementa **en la clase**, **un solo orden**.
- `Comparator` → orden **externo**, `compare(T a, T b)`, paquete `java.util`, se define **fuera de la clase**, **varios órdenes**.
- Resultado: **negativo** = el primero va antes; **cero** = equivalentes; **positivo** = el primero va después. Solo importa el signo.
- **No restes** para comparar: usa `Integer.compare`, `Double.compare`… o `Comparator.comparingInt(...)`.
- Forma recomendada de escribir comparadores: `Comparator.comparing(...)`, `.thenComparing(...)`, `.reversed()`.
- `reversed()` invierte **todo lo que hay antes** en la cadena.
- Los `null` se gestionan con `Comparator.nullsFirst` / `nullsLast`.
- `Collections.sort` y `List.sort` son **estables**.
- `TreeSet`/`TreeMap` usan el comparador para decidir si dos elementos son iguales: cuidado con órdenes que den `0` para objetos distintos.
- Sin `Comparable` ni `Comparator`, un `TreeSet<Film>` compila pero falla en ejecución.
- Un comparador que rompe el contrato puede ordenar mal **sin avisar**.

---

## 12. Autoevaluación

1. ¿En qué paquetes están `Comparable` y `Comparator`?
2. ¿Qué significa que `a.compareTo(b)` devuelva un valor negativo?
3. ¿Por qué `Comparator` es una interfaz funcional aunque declare `equals`?
4. ¿Qué dos problemas tiene `return first - second;` en un comparador de enteros, y `(int) (a - b)` con decimales?
5. ¿Cómo ordenarías por valoración descendente y, en empate, por título?
6. ¿Qué ocurre si dos objetos distintos dan `0` en el `compareTo` y los añades a un `TreeSet`?
7. ¿Qué diferencia hay entre `comparing(A).thenComparing(B).reversed()` y `comparing(A).reversed().thenComparing(B)`?
8. ¿Qué pasa al hacer `new TreeSet<Film>().add(film)` si `Film` no es `Comparable`?
9. ¿Cuándo elegirías `Comparable` y cuándo `Comparator`?

<details>
<summary><strong>Ver respuestas</strong></summary>

1. `Comparable` está en `java.lang`; `Comparator` está en `java.util`.
2. Que `a` es menor que `b` según el orden, y por tanto en una ordenación ascendente `a` va antes que `b`.
3. Porque `equals(Object)` ya existe en `Object` y no cuenta como método abstracto; el único abstracto es `compare`.
4. La resta de enteros puede desbordarse y dar un signo incorrecto; el `(int)` sobre una resta de decimales trunca la parte decimal y hace que valores distintos parezcan iguales. La solución es `Integer.compare` / `Double.compare` o `Comparator.comparingInt` / `comparingDouble`.
5. `Comparator.comparingDouble(Film::rating).reversed().thenComparing(Film::title)`.
6. El `TreeSet` los considera el mismo elemento y no añade el segundo.
7. La primera invierte los dos criterios; la segunda invierte solo el criterio A y mantiene B ascendente.
8. Compila, pero lanza `ClassCastException` al añadir el primer elemento.
9. `Comparable` para un orden natural único de una clase propia; `Comparator` para varios criterios, para clases que no puedes modificar o para órdenes puntuales.

</details>

---

## 13. Observaciones sobre el artículo de GeeksforGeeks

Al contrastar el artículo con la documentación y con la ejecución de los ejemplos:

1. **Lo esencial es correcto:** `Comparable` define el orden natural con `compareTo`, en `java.lang`, y `Comparator` un orden personalizado con `compare`, en `java.util`; con `Comparator` puede haber varios órdenes. La explicación sobre por qué `Comparator` es interfaz funcional también coincide con la documentación.
2. **Resta en `compareTo`:** el ejemplo usa `this.year - m.year`. Con años de películas no habrá desbordamiento, pero es un mal hábito (ver [7.1](#71-comparar-restando)). Es preferible `Integer.compare`.
3. **"Ordenar por valoración y luego por nombre":** el ejemplo del artículo hace **dos ordenaciones independientes** (una por valoración y otra, después, por nombre), no un desempate. Para ordenar por valoración y desempatar por nombre hay que encadenar con `thenComparing` ([4.3](#43-métodos-de-fábrica-y-composición)).
4. **Qué no cubre:** los métodos de fábrica (`comparing`, `thenComparing`, `reversed`, `nullsFirst`/`nullsLast`), el contrato, la coherencia con `equals` ni los errores típicos. Esta guía los añade.

---

## 14. Fuentes y cómo se ha verificado

**Fuentes consultadas**

- GeeksforGeeks, *Java Comparable vs Comparator*: <https://www.geeksforgeeks.org/java/comparable-vs-comparator-in-java/> (página leída completa).
- Oracle, Javadoc de `java.util.Comparator` (Java SE 21): <https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Comparator.html> (página leída completa).
- Oracle, Javadoc de `java.lang.Comparable` (Java SE 21): <https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Comparable.html> (consultada mediante fragmentos devueltos por la búsqueda, junto con las versiones 8, 15 y 23 de la misma página, que mencionan la misma excepción de `BigDecimal`).
- Oracle, Javadoc de `java.util.Collections` (Java SE 21): <https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html> (consultada mediante fragmentos de la búsqueda; de ahí proceden la estabilidad de `sort`, el significado del comparador `null` y la excepción opcional `IllegalArgumentException`).

**Comprobado ejecutando código** (OpenJDK 21.0.10): todas las salidas mostradas, el error de compilación de la sección 1 y de la 4.4, el desbordamiento de la resta, el comportamiento de `TreeSet`, `BigDecimal`, `reversed()`, el `ClassCastException`, la excepción de contrato roto y los métodos `getFirst`, `getLast` y `reversed` de Java 21.

**Datos que proceden de mi conocimiento del lenguaje** y que no he contrastado con una página concreta en esta sesión:

- Los `record` se incorporaron de forma definitiva en Java 16.
- `Locale.of` está disponible desde Java 19.
- Las colecciones secuenciadas de Java 21 corresponden al JEP 431.
- La regla de la especificación de interfaces funcionales por la cual un método abstracto que sobrescribe un método público de `Object` no cuenta como abstracto.
- Los arrays de tipos primitivos no admiten `Comparator` en `Arrays.sort`.
