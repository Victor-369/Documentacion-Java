# Guía de Reflection en Java (Java 25 o superior)

**Referencias base:**

- [The Reflection API – Java Tutorials (Oracle)](https://docs.oracle.com/javase/tutorial/reflect/index.html)
- [Guide to Java Reflection – Baeldung](https://www.baeldung.com/java-reflection)

> Nota: el tutorial de Oracle está escrito para JDK 8. Esta guía mantiene sus conceptos y los actualiza con lo que cambió hasta Java 25 (records, clases selladas, módulos, `MethodHandle`, fin del `SecurityManager`, etc.).

---

## Índice

1. [¿Qué es Reflection?](#1-qué-es-reflection)
2. [Casos de uso reales](#2-casos-de-uso-reales)
3. [Inconvenientes y precauciones](#3-inconvenientes-y-precauciones)
4. [Clases de ejemplo que usaremos](#4-clases-de-ejemplo-que-usaremos)
5. [Obtener el objeto `Class`](#5-obtener-el-objeto-class)
6. [Inspeccionar una clase](#6-inspeccionar-una-clase)
7. [Constructores](#7-constructores)
8. [Campos (fields)](#8-campos-fields)
9. [Métodos](#9-métodos)
10. [Acceso a miembros privados y `setAccessible`](#10-acceso-a-miembros-privados-y-setaccessible)
11. [Excepciones habituales](#11-excepciones-habituales)
12. [Arrays y enumerados](#12-arrays-y-enumerados)
13. [Records y clases selladas](#13-records-y-clases-selladas)
14. [Anotaciones en tiempo de ejecución](#14-anotaciones-en-tiempo-de-ejecución)
15. [Genéricos y borrado de tipos](#15-genéricos-y-borrado-de-tipos)
16. [Proxies dinámicos](#16-proxies-dinámicos)
17. [Reflection y el sistema de módulos](#17-reflection-y-el-sistema-de-módulos)
18. [`MethodHandle` y `VarHandle`](#18-methodhandle-y-varhandle)
19. [Novedades y cambios relevantes (Java 17 → 25 y más allá)](#19-novedades-y-cambios-relevantes-java-17--25-y-más-allá)
20. [Rendimiento](#20-rendimiento)
21. [Reflection y GraalVM Native Image](#21-reflection-y-graalvm-native-image)
22. [Mini proyecto: un validador con anotaciones](#22-mini-proyecto-un-validador-con-anotaciones)
23. [Buenas prácticas](#23-buenas-prácticas)
24. [Ejercicios propuestos](#24-ejercicios-propuestos)
25. [Resumen rápido (cheat sheet)](#25-resumen-rápido-cheat-sheet)

---

## 1. ¿Qué es Reflection?

**Reflection** (reflexión) es la capacidad de un programa Java de **examinar y modificar su propia estructura y comportamiento en tiempo de ejecución**.

Normalmente, al escribir código, el compilador conoce de antemano los tipos, métodos y campos que usas:

```java
Persona p = new Persona();
p.setNombre("Ana");
```

Con Reflection puedes hacer lo mismo **sin conocer los nombres en tiempo de compilación**:

```java
Class<?> clase = Class.forName(nombreDeClaseLeidoDeUnFichero);
Object objeto = clase.getDeclaredConstructor().newInstance();
```

Con Reflection puedes:

- Descubrir el nombre, modificadores, campos, métodos, constructores, superclase e interfaces de una clase.
- Crear objetos de una clase cuyo nombre solo conoces en ejecución.
- Leer y escribir campos (incluso privados, con ciertas condiciones).
- Invocar métodos (incluso privados, con ciertas condiciones).
- Leer anotaciones, tipos genéricos y componentes de `record`.
- Crear implementaciones de interfaces "al vuelo" (proxies).

No necesitas librerías externas: todo está en el paquete `java.lang.reflect` y en `java.lang.Class` (módulo `java.base`).

```java
import java.lang.reflect.*;
```

---

## 2. Casos de uso reales

El tutorial de Oracle destaca tres grandes familias de usos; Baeldung añade el mapeo de objetos a bases de datos:

| Uso | Ejemplo |
|---|---|
| **Extensibilidad** | Cargar *plugins* o clases definidas por el usuario a partir de su nombre completo. |
| **Herramientas de desarrollo** | IDEs, navegadores de clases, depuradores que inspeccionan miembros privados. |
| **Frameworks de test** | JUnit descubre y ejecuta métodos anotados con `@Test`. |
| **Mapeo objeto–datos** | Hibernate/JPA lee los campos de una entidad para generar SQL. |
| **Serialización** | Jackson/Gson convierten objetos a JSON leyendo sus campos o *getters*. |
| **Inyección de dependencias** | Spring/Jakarta CDI/Guice crean objetos e inyectan campos y constructores. |
| **Proxies/AOP** | Transacciones, seguridad o *logging* sin tocar el código de negocio. |

> Si trabajas con Spring, Hibernate, JUnit o Jackson, **ya usas Reflection todos los días** aunque no la escribas tú.

---

## 3. Inconvenientes y precauciones

Oracle lo resume bien: *si puedes hacerlo sin Reflection, hazlo sin Reflection*.

1. **Rendimiento**: al resolverse tipos dinámicamente, la JVM no puede aplicar algunas optimizaciones. Evítala en código muy llamado (ver [sección 20](#20-rendimiento)).
2. **Seguridad y encapsulación**: permite saltarse `private`. Rompe abstracciones y puede dejar objetos en estados inválidos.
3. **Fragilidad**: si renombras un método, el compilador **no** avisa de que un `getMethod("nombreViejo")` ya no existe. El error aparece en ejecución.
4. **Sin ayuda del compilador ni del IDE**: no hay autocompletado ni comprobación de tipos para los nombres que pasas como `String`.
5. **Portabilidad futura**: el JDK está cerrando poco a poco el acceso "profundo" (módulos fuertemente encapsulados, advertencias sobre `final`, etc.).

> **Sobre el `SecurityManager`:** el tutorial de Oracle menciona restricciones de seguridad bajo un *security manager* (por ejemplo, en *applets*). Ese mecanismo está **permanentemente deshabilitado desde Java 24** (JEP 486) y los *applets* ya no existen, por lo que hoy el control de acceso reflexivo se hace principalmente con el **sistema de módulos**.

---

## 4. Clases de ejemplo que usaremos

Para seguir esta guía crea un fichero `Main.java` que contenga estas clases (la clase con `main` debe ser la primera). Los fragmentos posteriores se pegan dentro del método `main`.

```java
import java.lang.reflect.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws Throwable {
        // aquí pegaremos los ejemplos
    }
}

interface Eating {
    String eats();
}

interface Locomotion {
    String getLocomotion();
}

abstract class Animal implements Eating {
    public static String CATEGORY = "domestic";
    private String name;

    Animal(String name) { this.name = name; }

    protected abstract String getSound();

    public String getName() { return name; }
}

class Goat extends Animal implements Locomotion {
    public Goat() { super("goat"); }

    @Override protected String getSound() { return "bleat"; }
    @Override public String getLocomotion() { return "walks"; }
    @Override public String eats() { return "grass"; }
}

class Bird extends Animal {
    private boolean walks;

    public Bird()                           { super("bird"); }
    public Bird(String name)                { super(name); }
    public Bird(String name, boolean walks) { super(name); this.walks = walks; }

    public boolean walks()          { return walks; }
    public void setWalks(boolean w) { this.walks = w; }

    private String secreto() { return "shh"; }

    @Override protected String getSound() { return "tweet"; }
    @Override public String eats() { return "seeds"; }
    @Override public String toString() { return "Bird[" + getName() + ", walks=" + walks + "]"; }
}
```

Ejecutar:

```bash
java Main.java
```

---

## 5. Obtener el objeto `Class`

Todo comienza en `java.lang.Class<T>`: es el "carnet de identidad" de un tipo en ejecución. Hay varias formas de obtenerlo:

```java
// 1) Desde una instancia
Class<?> a = "hola".getClass();

// 2) Desde el literal de clase (comprobado en compilación)
Class<?> b = String.class;

// 3) Desde el nombre completo (en ejecución, puede fallar)
Class<?> c = Class.forName("java.lang.String");

System.out.println(a == b);  // true
System.out.println(b == c);  // true: solo hay un Class por tipo y class loader

// Tipos primitivos y arrays también tienen Class
System.out.println(int.class);            // int
System.out.println(int[].class.getName()); // [I
```

Puntos importantes:

- `Class.forName("x")` exige el **nombre completo con paquete**. Si no existe lanza `ClassNotFoundException` (una excepción comprobada).
- Cada tipo tiene **un único** objeto `Class` por cada *class loader*.
- Primitivos (`int.class`), `void.class` y arrays también tienen su `Class`.

---

## 6. Inspeccionar una clase

```java
Class<?> clazz = Class.forName("Goat");

System.out.println(clazz.getSimpleName());   // Goat
System.out.println(clazz.getName());         // Goat (con paquete sería com.x.Goat)
System.out.println(clazz.getPackage());      // paquete (en el paquete por defecto: package )
System.out.println(Modifier.toString(clazz.getModifiers())); // (vacío: sin modificador)

System.out.println(clazz.getSuperclass().getSimpleName());   // Animal
System.out.println(Arrays.toString(clazz.getInterfaces()));  // [interface Locomotion]
```

### Modificadores

`getModifiers()` devuelve un `int` con *bits*. La clase `Modifier` ayuda a interpretarlo:

```java
int mods = Animal.class.getModifiers();
System.out.println(Modifier.isAbstract(mods)); // true
System.out.println(Modifier.isPublic(mods));   // false (es package-private)
System.out.println(Modifier.toString(mods));   // abstract
```

### `getInterfaces()` solo devuelve las declaradas directamente

`Goat` declara `implements Locomotion`, y hereda `Eating` de `Animal`. Por eso:

```java
System.out.println(Goat.class.getInterfaces().length);   // 1 (solo Locomotion)
System.out.println(Animal.class.getInterfaces().length); // 1 (solo Eating)
```

### Nombres: `getName`, `getSimpleName`, `getCanonicalName`, `getTypeName`

| Método | Para `int[]` | Para `Map.Entry` |
|---|---|---|
| `getName()` | `[I` | `java.util.Map$Entry` |
| `getSimpleName()` | `int[]` | `Entry` |
| `getCanonicalName()` | `int[]` | `java.util.Map.Entry` |
| `getTypeName()` | `int[]` | `java.util.Map$Entry` |

---

## 7. Constructores

### Listar constructores

```java
Constructor<?>[] publicos = Bird.class.getConstructors();         // solo públicos
Constructor<?>[] todos    = Bird.class.getDeclaredConstructors(); // todos los de ESTA clase

System.out.println(publicos.length); // 3
```

### Elegir uno por sus tipos de parámetros

Dos constructores no pueden tener la misma firma, así que los parámetros identifican de forma única:

```java
Constructor<Bird> c0 = Bird.class.getConstructor();
Constructor<Bird> c1 = Bird.class.getConstructor(String.class);
Constructor<Bird> c2 = Bird.class.getConstructor(String.class, boolean.class);
```

Si no existe, lanza `NoSuchMethodException`.

### Crear instancias

```java
Bird b1 = c0.newInstance();
Bird b2 = c1.newInstance("Weaver bird");
Bird b3 = c2.newInstance("dove", true);

System.out.println(b1); // Bird[bird, walks=false]
System.out.println(b3); // Bird[dove, walks=true]
```

> **Ojo:** `Class.newInstance()` está **obsoleto desde Java 9**. Usa siempre `clazz.getDeclaredConstructor().newInstance()`.

```java
// Forma moderna cuando solo hay constructor sin argumentos
Object o = Goat.class.getDeclaredConstructor().newInstance();
```

---

## 8. Campos (fields)

| Método | Qué devuelve |
|---|---|
| `getFields()` / `getField(nombre)` | Campos **públicos**, incluidos los heredados. |
| `getDeclaredFields()` / `getDeclaredField(nombre)` | **Todos** los campos declarados en **esa** clase (cualquier visibilidad), sin heredados. |

```java
// Públicos, incluyendo heredados: Bird no declara públicos, pero hereda CATEGORY
Field[] publicos = Bird.class.getFields();
System.out.println(publicos[0].getName()); // CATEGORY

// Declarados en Bird (incluye privados)
for (Field f : Bird.class.getDeclaredFields()) {
    System.out.println(Modifier.toString(f.getModifiers()) + " "
            + f.getType().getSimpleName() + " " + f.getName());
}
// private boolean walks
```

El campo privado `name` es de `Animal`, así que `Bird.class.getDeclaredField("name")` lanzaría `NoSuchFieldException`. Para llegar a él hay que consultar `Animal.class`.

### Leer y escribir valores

```java
Bird bird = new Bird("paloma", false);

Field walks = Bird.class.getDeclaredField("walks");
walks.setAccessible(true);                // necesario porque es private

System.out.println(walks.getBoolean(bird)); // false
walks.set(bird, true);
System.out.println(bird.walks());          // true
```

`Field` tiene variantes tipadas (`getInt`, `getBoolean`, `setDouble`...) y genéricas (`get`/`set`, que hacen *boxing*).

### Campos estáticos: se pasa `null` como instancia

```java
Field cat = Animal.class.getField("CATEGORY");
System.out.println(cat.get(null)); // domestic
```

---

## 9. Métodos

| Método | Qué devuelve |
|---|---|
| `getMethods()` / `getMethod(nombre, tipos...)` | Métodos **públicos**, incluidos los heredados (también los de `Object`). |
| `getDeclaredMethods()` / `getDeclaredMethod(nombre, tipos...)` | **Todos** los métodos declarados en **esa** clase. |

```java
for (Method m : Animal.class.getDeclaredMethods()) {
    System.out.println(m.getName());   // getName, getSound
}

// Públicos de Bird: incluye equals, hashCode, toString, notifyAll...
List<String> nombres = Arrays.stream(Bird.class.getMethods())
        .map(Method::getName)
        .sorted()
        .toList();
System.out.println(nombres);
```

### Invocar un método

```java
Bird bird = new Bird();

Method setWalks = Bird.class.getMethod("setWalks", boolean.class);
Method walks    = Bird.class.getMethod("walks");

System.out.println(walks.invoke(bird));   // false
setWalks.invoke(bird, true);
System.out.println(walks.invoke(bird));   // true
```

Para métodos **estáticos** se pasa `null` como primer argumento:

```java
Method valueOf = Integer.class.getMethod("valueOf", String.class);
Object n = valueOf.invoke(null, "42");
System.out.println(n); // 42
```

### Información de un método

```java
Method m = Bird.class.getMethod("setWalks", boolean.class);
System.out.println(m.getReturnType());                         // void
System.out.println(Arrays.toString(m.getParameterTypes()));    // [boolean]
System.out.println(m.getParameterCount());                     // 1
System.out.println(Modifier.toString(m.getModifiers()));       // public
```

> **Nombres de parámetros:** por defecto el compilador **no** guarda los nombres (`arg0`, `arg1`). Compila con `javac -parameters` para poder leerlos con `Parameter.getName()`. Las **componentes de un `record`** siempre conservan su nombre.

### Excepciones dentro del método invocado: `InvocationTargetException`

Si el método que invocas lanza una excepción, `invoke` la **envuelve** en `InvocationTargetException`. La excepción real está en `getCause()`:

```java
Method boom = Main.class.getDeclaredMethod("boom");
try {
    boom.invoke(null);
} catch (InvocationTargetException e) {
    System.out.println("Causa real: " + e.getCause()); // java.lang.IllegalStateException: boom
}

// En la clase Main:
static void boom() { throw new IllegalStateException("boom"); }
```

---

## 10. Acceso a miembros privados y `setAccessible`

Por defecto, Reflection **respeta** los modificadores de acceso al invocar o acceder:

```java
Method secreto = Bird.class.getDeclaredMethod("secreto");
try {
    secreto.invoke(new Bird());
} catch (IllegalAccessException e) {
    System.out.println("No se puede: es private");
}
```

Para saltarse la comprobación se llama a `setAccessible(true)`:

```java
secreto.setAccessible(true);
System.out.println(secreto.invoke(new Bird())); // shh
```

Otras opciones útiles de `AccessibleObject` (de la que heredan `Field`, `Method` y `Constructor`):

```java
boolean ok = secreto.trySetAccessible();  // devuelve false en lugar de lanzar excepción
boolean visible = secreto.canAccess(new Bird()); // ¿puede el código actual usarlo ya?
```

### Límites importantes

`setAccessible(true)` **no es mágico**. Falla (`InaccessibleObjectException`) cuando el miembro pertenece a un **módulo que no abre su paquete** a tu código. Es lo que pasa con casi todo el JDK:

```java
Field value = String.class.getDeclaredField("value");
value.setAccessible(true);  // InaccessibleObjectException (módulo java.base no abre java.lang)
```

Ver [sección 17](#17-reflection-y-el-sistema-de-módulos).

### Campos `final`

- Un campo `final` **de instancia** en una clase normal *todavía* puede modificarse con `setAccessible(true)` + `set(...)` en Java 25, pero es una mala práctica.
- Los campos `final` de **`record`** y de **clases ocultas (*hidden classes*)** **nunca** pueden modificarse por Reflection: `set` lanza `IllegalAccessException`.
- Los **campos estáticos finales** no pueden modificarse.
- Si el campo es una **constante en tiempo de compilación** (`private final int v = 1;`), el compilador **copia su valor** en los sitios donde se usa. Aunque cambies el campo, el código seguirá viendo `1`.

```java
class Const {
    private final int v = 1;
    int v() { return v; }
}

Const cc = new Const();
Field ff = Const.class.getDeclaredField("v");
ff.setAccessible(true);
ff.set(cc, 9);
System.out.println(cc.v()); // sigue imprimiendo 1 (valor en línea)
```

> **Mira hacia delante (JDK 26, JEP 500):** a partir de Java 26 la modificación de campos `final` por Reflection profunda **emite una advertencia** por defecto, y una versión futura la bloqueará. En Java 25 aún no hay advertencia, pero es un buen motivo para **no depender de ello**. Más detalles en la [sección 19](#19-novedades-y-cambios-relevantes-java-17--25-y-más-allá).

---

## 11. Excepciones habituales

| Excepción | Cuándo ocurre | Solución típica |
|---|---|---|
| `ClassNotFoundException` | `Class.forName` con un nombre inexistente o sin paquete. | Usa el nombre completo y comprueba el *classpath*. |
| `NoSuchMethodException` | `getMethod`/`getConstructor` con nombre o tipos de parámetros incorrectos. | Revisa los tipos exactos (`int.class` ≠ `Integer.class`). |
| `NoSuchFieldException` | `getField`/`getDeclaredField` con nombre inexistente o campo heredado privado. | Busca en la clase que lo declara. |
| `IllegalAccessException` | Acceso a un miembro no accesible o `set` sobre un campo final no modificable. | `setAccessible(true)` si procede (y es legítimo). |
| `InaccessibleObjectException` | `setAccessible` sobre un paquete no abierto de un módulo. | `opens` en `module-info.java` o `--add-opens`. |
| `InvocationTargetException` | El método/constructor invocado lanzó una excepción. | Mira `e.getCause()`. |
| `InstantiationException` | Crear una instancia de una clase abstracta o interfaz. | Usa una clase concreta. |
| `IllegalArgumentException` | Número o tipo de argumentos incorrecto en `invoke`/`newInstance`/`set`. | Revisa los argumentos. |

> **Ojo con primitivos:** `getMethod("setWalks", Boolean.class)` **no** encuentra `setWalks(boolean)`. Hay que usar `boolean.class`.

---

## 12. Arrays y enumerados

### Arrays

Los arrays se crean en tiempo de ejecución. `java.lang.reflect.Array` permite manipularlos sin conocer su tipo:

```java
Object arr = Array.newInstance(int.class, 3);   // int[3]
Array.setInt(arr, 0, 7);

System.out.println(Array.getLength(arr));        // 3
System.out.println(Array.get(arr, 0));           // 7
System.out.println(arr.getClass().isArray());    // true
System.out.println(arr.getClass().getComponentType()); // int
```

### Enumerados

```java
enum Color { ROJO, VERDE, AZUL }

Class<Color> k = Color.class;
System.out.println(k.isEnum());                              // true
System.out.println(Arrays.toString(k.getEnumConstants()));   // [ROJO, VERDE, AZUL]
System.out.println(Color.valueOf("VERDE"));                  // VERDE
```

---

## 13. Records y clases selladas

Dos novedades del lenguaje (finalizadas en Java 16 y 17) que Reflection soporta de forma específica.

### Records

```java
record Punto(int x, int y) {}

Class<Punto> k = Punto.class;
System.out.println(k.isRecord()); // true

Punto p = new Punto(3, 4);
for (RecordComponent rc : k.getRecordComponents()) {
    Object valor = rc.getAccessor().invoke(p);
    System.out.println(rc.getName() + " (" + rc.getType().getSimpleName() + ") = " + valor);
}
// x (int) = 3
// y (int) = 4
```

Los *records* son inmutables: sus campos son `final` y **no se pueden cambiar por Reflection**, aunque uses `setAccessible(true)`.

Construir un record reflexivamente:

```java
Constructor<Punto> ctor = Punto.class.getDeclaredConstructor(int.class, int.class);
Punto nuevo = ctor.newInstance(10, 20);
System.out.println(nuevo); // Punto[x=10, y=20]
```

### Clases e interfaces selladas

```java
sealed interface Forma permits Circulo, Cuadrado {}
record Circulo(double r) implements Forma {}
record Cuadrado(double lado) implements Forma {}

System.out.println(Forma.class.isSealed());                         // true
System.out.println(Arrays.toString(Forma.class.getPermittedSubclasses()));
// [class Circulo, class Cuadrado]
```

---

## 14. Anotaciones en tiempo de ejecución

Las anotaciones solo están disponibles por Reflection si tienen `@Retention(RetentionPolicy.RUNTIME)`.

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)  // visible en ejecución
@Target(ElementType.FIELD)           // solo aplicable a campos
@interface NoVacio {
    String mensaje() default "no puede estar vacío";
}

class Usuario {
    @NoVacio(mensaje = "el nombre es obligatorio")
    String nombre;
    int edad;

    Usuario(String nombre, int edad) { this.nombre = nombre; this.edad = edad; }
}
```

Leerlas:

```java
for (Field f : Usuario.class.getDeclaredFields()) {
    if (f.isAnnotationPresent(NoVacio.class)) {
        NoVacio nv = f.getAnnotation(NoVacio.class);
        System.out.println(f.getName() + " -> " + nv.mensaje());
    }
}
// nombre -> el nombre es obligatorio
```

Así funcionan `@Test` en JUnit, `@Entity` en JPA o `@Autowired` en Spring: un framework **escanea** las anotaciones y actúa en consecuencia. Verás un ejemplo completo en la [sección 22](#22-mini-proyecto-un-validador-con-anotaciones).

---

## 15. Genéricos y borrado de tipos

En Java los genéricos se **borran** (*type erasure*) en tiempo de compilación, así que en ejecución un `List<String>` y un `List<Integer>` son la misma clase `List`.

Sin embargo, la **declaración** de campos, parámetros y retornos conserva la información genérica y se puede leer:

```java
class Holder {
    List<String> lista;
}

Field f = Holder.class.getDeclaredField("lista");

System.out.println(f.getType());         // interface java.util.List  (tipo "borrado")
System.out.println(f.getGenericType());  // java.util.List<java.lang.String>

ParameterizedType pt = (ParameterizedType) f.getGenericType();
System.out.println(pt.getRawType());                       // interface java.util.List
System.out.println(Arrays.toString(pt.getActualTypeArguments())); // [class java.lang.String]
```

Esto es lo que usan Jackson o Gson para saber que un campo es `List<Usuario>` y no solo `List`.

> Lo que **no** puedes saber es el tipo genérico de un objeto concreto creado con `new ArrayList<String>()`: esa información no existe en ejecución.

---

## 16. Proxies dinámicos

Un **proxy dinámico** es una implementación de una o varias interfaces creada en ejecución con `java.lang.reflect.Proxy`. Todas las llamadas pasan por un `InvocationHandler`.

```java
interface Saludo {
    String saluda(String nombre);
}

Saludo real = nombre -> "Hola " + nombre;

Saludo conLog = (Saludo) Proxy.newProxyInstance(
        Saludo.class.getClassLoader(),
        new Class<?>[] { Saludo.class },
        (proxy, metodo, argumentos) -> {
            System.out.println("Llamando a " + metodo.getName());
            return metodo.invoke(real, argumentos);
        });

System.out.println(conLog.saluda("Ana"));
// Llamando a saluda
// Hola Ana
```

Los proxies son la base de muchas funcionalidades de frameworks: transacciones (`@Transactional`), repositorios (Spring Data genera la implementación de tus interfaces), clientes HTTP declarativos, etc.

> Limitación: `Proxy` solo funciona con **interfaces**. Para clases concretas, los frameworks usan generación de *bytecode* (por ejemplo, ByteBuddy). Desde Java 24 existe además la **Class-File API** estándar (JEP 484) para leer y generar ficheros `.class`.

---

## 17. Reflection y el sistema de módulos

Desde Java 9 (y con *encapsulación fuerte* por defecto desde Java 17), el JDK **no permite** hacer Reflection profunda sobre sus clases internas.

### Conceptos clave

- **`exports`**: hace públicos los tipos públicos de un paquete para **compilar y ejecutar**.
- **`opens`**: permite **Reflection profunda** (acceso a miembros privados con `setAccessible`) **en tiempo de ejecución**.

```java
module mi.app {
    requires java.sql;
    exports com.miempresa.api;           // API pública
    opens com.miempresa.modelo;          // un framework puede usar Reflection profunda aquí
    // opens com.miempresa.modelo to com.fasterxml.jackson.databind;  // solo para un módulo concreto
}
```

### Consultas desde código

```java
Module m = String.class.getModule();
System.out.println(m.getName());                  // java.base
System.out.println(m.isOpen("java.lang"));        // false (no abierto a todos)
System.out.println(m.isExported("java.lang"));    // true
```

### Abrir paquetes desde la línea de comandos

Si una librería antigua necesita acceder a internals del JDK:

```bash
java --add-opens java.base/java.lang=ALL-UNNAMED -jar app.jar
```

> **Sugerencia:** si ves `InaccessibleObjectException: ... does not "opens ..."`, el problema es de **módulos**, no de tu código. La solución limpia es actualizar la librería; el parche temporal es `--add-opens`.

---

## 18. `MethodHandle` y `VarHandle`

Desde Java 7 existe una alternativa más moderna y rápida a `Method.invoke`: `java.lang.invoke.MethodHandle`.

```java
import java.lang.invoke.*;

MethodHandles.Lookup lookup = MethodHandles.lookup();
MethodType tipo = MethodType.methodType(int.class);          // () -> int

MethodHandle length = lookup.findVirtual(String.class, "length", tipo);
int n = (int) length.invokeExact("hola");                     // 4
System.out.println(n);
```

Comparativa:

| | `Method.invoke` | `MethodHandle` |
|---|---|---|
| Comprobación de acceso | En **cada** llamada (sobre el `Method`, o una vez con `setAccessible`) | **Una sola vez**, al crear el *handle* |
| Rendimiento | Bueno desde Java 18 | Muy bueno si se guarda en `static final` |
| Facilidad | Muy sencilla | Más compleja (firmas exactas con `invokeExact`) |
| Uso típico | Código de aplicación, frameworks | Frameworks de alto rendimiento, JVM languages |

> **Dato de interés (JDK 18, JEP 416):** la implementación interna de `Method.invoke`, `Constructor.newInstance` y `Field.get/set` fue reescrita para usar *method handles*. Es un cambio interno: tu código no cambia, pero el rendimiento y el comportamiento en la JVM son más coherentes.

### Acceso a campos privados con `VarHandle`

```java
MethodHandles.Lookup privado = MethodHandles.privateLookupIn(Bird.class, MethodHandles.lookup());
VarHandle walksVH = privado.findVarHandle(Bird.class, "walks", boolean.class);

Bird bird = new Bird();
walksVH.set(bird, true);
System.out.println(walksVH.get(bird)); // true
```

`privateLookupIn` también respeta la encapsulación de módulos: tu módulo debe tener permiso (por ejemplo, que el paquete esté `opens` a tu módulo).

---

## 19. Novedades y cambios relevantes (Java 17 → 25 y más allá)

Resumen de lo que conviene saber al usar Reflection con Java 25:

| Versión | Cambio | Impacto |
|---|---|---|
| **9** | `Class.newInstance()` obsoleto. Sistema de módulos (JPMS). | Usa `getDeclaredConstructor().newInstance()`. `opens`/`exports` condicionan el acceso. |
| **16** | Records (`isRecord`, `getRecordComponents`). | Inmutables: no se pueden modificar por Reflection. |
| **17** | Clases selladas (`isSealed`, `getPermittedSubclasses`). Encapsulación fuerte por defecto (`--illegal-access` eliminado). | El código que dependía de acceso "ilegal" a internals ahora falla: usa `--add-opens`. |
| **18** | JEP 416: Reflection reimplementada con *method handles*. | Mejor mantenimiento y rendimiento coherente. |
| **24** | `SecurityManager` deshabilitado permanentemente (JEP 486). Class-File API estándar (JEP 484). | Ya no hay permisos de Reflection vía `SecurityManager`. La Class-File API es la vía oficial para manipular *bytecode*. |
| **25** (LTS) | Versión de soporte a largo plazo. Sin cambios disruptivos en `java.lang.reflect` respecto a 24. | Es una excelente base estable para aprender. |
| **26** | JEP 500: advertencia al modificar campos `final` con Reflection profunda; opciones `--illegal-final-field-mutation` / `--enable-final-field-mutation`. | Prepárate: evita modificar `final` por Reflection. |

### Sobre campos `final` (preparándose para el futuro)

La dirección del JDK es la **"integridad por defecto"**: que `final` signifique realmente `final`. Según el plan de JEP 500:

1. **JDK 26**: la modificación de `final` por Reflection profunda genera una **advertencia**.
2. **Versión futura**: se **lanzará una excepción** por defecto, salvo que se habilite explícitamente con opciones de línea de comandos.

Recomendación práctica: si tu código (o una librería) modifica campos `final` con Reflection, empieza a buscar alternativas (constructores, *builders*, copias de objetos, `record` con constructor canónico).

> Las novedades de versiones posteriores a la fecha de esta guía pueden haber cambiado. Consulta siempre las [notas de versión del JDK](https://openjdk.org/projects/jdk/) y el [Javadoc oficial de `java.lang.reflect` para Java 25](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/reflect/package-summary.html).

---

## 20. Rendimiento

Reflection es más lenta que el acceso directo, pero **no tanto como se cree** en el JDK moderno. Reglas prácticas:

1. **Cachea** los objetos `Class`, `Method`, `Field` y `Constructor`: buscarlos (`getDeclaredMethod`...) es lo más caro.
2. **Llama a `setAccessible(true)` una sola vez**, no en cada invocación.
3. Si vas a invocar millones de veces el mismo método, usa un **`MethodHandle` almacenado en un campo `static final`**: la JVM puede optimizarlo casi como una llamada directa.
4. No uses Reflection en bucles críticos si hay alternativa clara (interfaces, polimorfismo, `switch` con *pattern matching*).

Ejemplo de caché sencilla:

```java
class CacheMetodos {
    private static final Map<String, Method> CACHE = new java.util.concurrent.ConcurrentHashMap<>();

    static Method obtener(Class<?> clase, String nombre) {
        return CACHE.computeIfAbsent(clase.getName() + "#" + nombre, clave -> {
            try {
                Method m = clase.getMethod(nombre);
                m.setAccessible(true);
                return m;
            } catch (NoSuchMethodException e) {
                throw new IllegalArgumentException("No existe " + nombre, e);
            }
        });
    }
}
```

---

## 21. Reflection y GraalVM Native Image

Con **GraalVM Native Image** (compilación anticipada a binario nativo) el análisis de código ocurre **en tiempo de compilación**, y todo lo que se descubre solo en ejecución por Reflection puede quedar fuera del binario.

Por eso hay que **declarar** qué clases/miembros se usan por Reflection (ficheros de metadatos como `reflect-config.json`, o los *hints* de Spring AOT). Muchos frameworks (Spring Boot, Quarkus, Micronaut) lo generan automáticamente, pero es bueno saber que **"Reflection libre" y "binarios nativos" no se llevan bien sin configuración**.

---

## 22. Mini proyecto: un validador con anotaciones

Un ejemplo completo que une varias ideas: anotaciones, campos, `setAccessible` y excepciones. Guárdalo en `Validador.java` y ejecútalo con `java Validador.java`.

```java
import java.lang.annotation.*;
import java.lang.reflect.*;
import java.util.*;

public class Validador {

    // 1) Anotaciones de validación
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.FIELD)
    @interface NoVacio {
        String mensaje() default "no puede estar vacío";
    }

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.FIELD)
    @interface Minimo {
        int valor();
        String mensaje() default "valor demasiado bajo";
    }

    // 2) Clase a validar
    static class Usuario {
        @NoVacio(mensaje = "el nombre es obligatorio")
        private String nombre;

        @Minimo(valor = 18, mensaje = "debe ser mayor de edad")
        private int edad;

        Usuario(String nombre, int edad) {
            this.nombre = nombre;
            this.edad = edad;
        }
    }

    // 3) Motor de validación basado en Reflection
    static List<String> validar(Object objeto) throws IllegalAccessException {
        List<String> errores = new ArrayList<>();

        for (Field campo : objeto.getClass().getDeclaredFields()) {
            campo.setAccessible(true);
            Object valor = campo.get(objeto);

            NoVacio noVacio = campo.getAnnotation(NoVacio.class);
            if (noVacio != null && (valor == null || valor.toString().isBlank())) {
                errores.add(campo.getName() + ": " + noVacio.mensaje());
            }

            Minimo minimo = campo.getAnnotation(Minimo.class);
            if (minimo != null && valor instanceof Integer entero && entero < minimo.valor()) {
                errores.add(campo.getName() + ": " + minimo.mensaje());
            }
        }
        return errores;
    }

    public static void main(String[] args) throws Exception {
        System.out.println(validar(new Usuario("Ana", 30)));  // []
        System.out.println(validar(new Usuario("", 15)));
        // [nombre: el nombre es obligatorio, edad: debe ser mayor de edad]
    }
}
```

**Qué demuestra:** el motor `validar` **no conoce** la clase `Usuario`; descubre en ejecución qué campos tienen qué anotaciones. Así funciona Bean Validation (`@NotNull`, `@Min`...) en esencia.

> Reto: ¿puedes explicar por qué `validar` necesita `campo.setAccessible(true)` y qué pasaría si `Usuario` estuviera en un módulo que no abre su paquete?

---

## 23. Buenas prácticas

1. **Úsala solo cuando haga falta.** Si puedes resolverlo con interfaces, polimorfismo o genéricos, hazlo así.
2. **Prefiere API públicas** (`getMethods`, `getFields`) a las privadas (`getDeclared*` + `setAccessible`).
3. **Cachea** `Class`, `Method`, `Field` y `Constructor`.
4. **Gestiona las excepciones** con cuidado, sobre todo `InvocationTargetException` (mira `getCause()`).
5. **Evita modificar campos `final`** y no dependas de internals del JDK.
6. **No expongas nombres de clases/métodos que vengan de la entrada del usuario** (por ejemplo, `Class.forName(parametroHttp)`): es una vulnerabilidad grave de ejecución de código. Usa listas blancas.
7. **No imprimas *stack traces* a usuarios finales.** El tutorial de Oracle lo avisa: sus ejemplos simplifican el manejo de errores solo con fines didácticos.
8. **Declara `opens` con precisión** en `module-info.java`, preferiblemente `opens paquete to modulo`.
9. **Escribe pruebas** para el código reflexivo: el compilador no te protege de errores tipográficos en los nombres.
10. **Prefiere `record` y constructores** en vez de modificar el estado interno por Reflection.

---

## 24. Ejercicios propuestos

1. **Inspector de clases.** Dado el nombre completo de una clase (por argumento de `main`), imprime su superclase, interfaces, constructores, campos y métodos con sus modificadores.
2. **Mini "toString" automático.** Escribe `static String describir(Object o)` que devuelva `Clase[campo1=valor1, campo2=valor2]` para cualquier objeto.
3. **Copiar objetos.** Escribe `static <T> T copiar(T origen)` que cree una nueva instancia (constructor sin argumentos) y copie todos los campos no estáticos.
4. **Invocar por nombre.** Lee de consola un nombre de método y ejecútalo sobre un objeto, solo si pertenece a una lista blanca.
5. **Mini DI.** Crea una anotación `@Inyectar`. Escribe un `Contenedor.crear(Clase)` que instancie la clase e inyecte en los campos anotados instancias de otras clases.
6. **Records.** Convierte cualquier `record` a un `Map<String, Object>` usando `getRecordComponents()`.
7. **Proxy cronómetro.** Crea un proxy dinámico que mida y muestre cuánto tarda cada método de una interfaz.
8. **Cuestionario.** Explica con tus palabras: ¿qué diferencia hay entre `getMethods()` y `getDeclaredMethods()`? ¿Y entre `exports` y `opens`?

---

## 25. Resumen rápido (cheat sheet)

```text
Obtener Class      obj.getClass() | Tipo.class | Class.forName("pkg.Tipo")
Nombres            getName() | getSimpleName() | getCanonicalName()
Jerarquía          getSuperclass() | getInterfaces() | getPermittedSubclasses()
Modificadores      getModifiers() + Modifier.isPublic/isAbstract/toString
Tipos especiales   isInterface() | isEnum() | isRecord() | isSealed() | isArray()

Constructores      getConstructors() | getDeclaredConstructors()
                   getDeclaredConstructor(tipos...).newInstance(args...)

Campos             getFields() | getDeclaredFields() | getField(n) | getDeclaredField(n)
                   f.setAccessible(true); f.get(obj) | f.set(obj, v)   (static → obj = null)

Métodos            getMethods() | getDeclaredMethods() | getMethod(n, tipos...) | getDeclaredMethod(n, tipos...)
                   m.setAccessible(true); m.invoke(obj, args...)       (static → obj = null)

Anotaciones        isAnnotationPresent(A.class) | getAnnotation(A.class)   (@Retention RUNTIME)
Genéricos          getGenericType() → ParameterizedType.getActualTypeArguments()
Records            getRecordComponents() → getName() | getType() | getAccessor()
Arrays             java.lang.reflect.Array.newInstance/get/set/getLength
Proxy              Proxy.newProxyInstance(loader, interfaces, handler)
Alternativa rápida MethodHandles.lookup().findVirtual(...) | VarHandle

Excepciones clave  ClassNotFound, NoSuchMethod, NoSuchField, IllegalAccess,
                   InaccessibleObject, InvocationTarget (→ getCause())
```

---

## Para seguir aprendiendo

- [The Reflection API – Java Tutorials (Oracle)](https://docs.oracle.com/javase/tutorial/reflect/index.html) (lecciones: Classes, Members, Arrays and Enumerated Types)
- [Guide to Java Reflection – Baeldung](https://www.baeldung.com/java-reflection)
- [Javadoc de `java.lang.reflect` en Java 25](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/reflect/package-summary.html)
- [Javadoc de `java.lang.invoke` en Java 25](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/invoke/package-summary.html)
- [dev.java – Tutoriales actualizados](https://dev.java/learn/)
- [JEP 500: Prepare to Make Final Mean Final](https://openjdk.org/jeps/500)
