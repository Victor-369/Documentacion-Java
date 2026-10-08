# Anotaciones (Annotations) en Java 25+

> **Fuentes base:** [Oracle – The Java Tutorials: Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/index.html) y [W3Schools – Java Annotations](https://www.w3schools.com/java/java_annotations.asp). El tutorial de Oracle está escrito para JDK 8; esta guía lo complementa con lo que importa en Java 25 (versión LTS).

---

## Índice

1. [¿Qué es una anotación?](#1-qué-es-una-anotación)
2. [¿Para qué sirven?](#2-para-qué-sirven)
3. [Sintaxis básica](#3-sintaxis-básica)
4. [Dónde se pueden usar](#4-dónde-se-pueden-usar)
5. [Anotaciones predefinidas de Java](#5-anotaciones-predefinidas-de-java)
6. [Meta-anotaciones](#6-meta-anotaciones)
7. [Crear tu propia anotación](#7-crear-tu-propia-anotación)
8. [Leer anotaciones en tiempo de ejecución (reflexión)](#8-leer-anotaciones-en-tiempo-de-ejecución-reflexión)
9. [Anotaciones repetibles](#9-anotaciones-repetibles)
10. [Anotaciones de tipo (type annotations)](#10-anotaciones-de-tipo-type-annotations)
11. [Anotaciones y características modernas de Java](#11-anotaciones-y-características-modernas-de-java)
12. [Reflexión en profundidad y reglas adicionales](#12-reflexión-en-profundidad-y-reglas-adicionales)
13. [Ejemplo práctico: un validador de campos](#13-ejemplo-práctico-un-validador-de-campos)
14. [Anotaciones en paquetes y módulos](#14-anotaciones-en-paquetes-y-módulos)
15. [Procesadores de anotaciones (en compilación)](#15-procesadores-de-anotaciones-en-compilación)
16. [Anotaciones en el ecosistema Java](#16-anotaciones-en-el-ecosistema-java)
17. [Errores comunes y buenas prácticas](#17-errores-comunes-y-buenas-prácticas)
18. [Preguntas frecuentes](#18-preguntas-frecuentes)
19. [Resumen rápido](#19-resumen-rápido)
20. [Ejercicios propuestos](#20-ejercicios-propuestos)
21. [Soluciones](#21-soluciones-ejercicios-3-4-5-y-7)
22. [Glosario](#22-glosario)

---

## Cómo probar los ejemplos

Desde Java 11 puedes ejecutar un archivo `.java` directamente, sin compilar antes. En Java 25 funciona así:

```bash
java MiArchivo.java
```

Si el archivo tiene varias clases, Java ejecuta la **primera clase declarada**, que debe contener el método `main`. Para ver los avisos (warnings) del compilador, usa `javac`:

```bash
javac -Xlint:all MiArchivo.java
```

---

## 1. ¿Qué es una anotación?

Una **anotación** es una *nota* que se añade al código Java. Empieza siempre con el símbolo `@`.

Es una forma de **metadatos**: información *sobre* tu programa que no forma parte de la lógica del programa en sí. Por sí sola, **una anotación no cambia lo que hace tu código**; da información extra al compilador, a herramientas o a otros programas (como frameworks) que sí la leen y actúan en consecuencia.

Una analogía: es como la etiqueta "frágil" en una caja. La etiqueta no cambia el contenido, pero quien la ve sabe cómo tratarla.

```java
@Override
public String toString() {
    return "Hola";
}
```

---

## 2. ¿Para qué sirven?

Hay tres usos principales:

| Uso | Descripción | Ejemplo |
|---|---|---|
| **Información para el compilador** | El compilador detecta errores o suprime avisos. | `@Override`, `@SuppressWarnings` |
| **Procesamiento en compilación o despliegue** | Herramientas leen las anotaciones y generan código, archivos XML, etc. | Generadores de código |
| **Procesamiento en tiempo de ejecución** | El programa lee las anotaciones mientras se ejecuta. | Frameworks como Spring, JUnit, Jakarta EE |

Si has visto `@Test` en JUnit o `@GetMapping` en Spring, ya has visto anotaciones en acción.

---

## 3. Sintaxis básica

### Anotación sin elementos

Los paréntesis se pueden omitir:

```java
@Override
void miMetodo() { }
```

### Anotación con elementos

Una anotación puede tener **elementos** (parecidos a parámetros), con nombre y valor:

```java
@Autor(nombre = "Ana García", fecha = "2026-01-15")
class MiClase { }
```

### Atajo: elemento llamado `value`

Si la anotación solo tiene un elemento y se llama `value`, puedes omitir el nombre:

```java
@SuppressWarnings(value = "unchecked")   // forma completa
@SuppressWarnings("unchecked")           // forma corta (equivalente)
void miMetodo() { }
```

### Varias anotaciones en el mismo elemento

Por convención, cada una va en su propia línea:

```java
@Autor(nombre = "Ana García")
@Deprecated
class MiClase { }
```

Si necesitas pasar varios valores en un elemento de tipo array, usa llaves:

```java
@SuppressWarnings({"unchecked", "deprecation"})
void otroMetodo() { }
```

---

## 4. Dónde se pueden usar

### Sobre declaraciones

Se pueden aplicar a clases, interfaces, enums, records, campos, métodos, constructores, parámetros, variables locales, paquetes y módulos:

```java
@Deprecated                         // sobre una clase
public class Vieja {

    @Deprecated                     // sobre un campo
    private int contador;

    @Override                       // sobre un método
    public String toString() { return "Vieja"; }

    public void metodo(@Deprecated String parametro) {   // sobre un parámetro
        @SuppressWarnings("unused") // sobre una variable local
        int sinUsar = 0;
    }
}
```

### Sobre el uso de tipos

Desde Java 8 también se pueden anotar los **usos de un tipo** (se explica en la [sección 10](#10-anotaciones-de-tipo-type-annotations)):

```java
List<@NoNulo String> nombres = new ArrayList<>();
```

---

## 5. Anotaciones predefinidas de Java

Java incluye varias anotaciones listas para usar. Las del paquete `java.lang` son las que más verás al principio.

### 5.1 `@Override`

Indica que un método **sobrescribe** un método de la superclase (o implementa uno de una interfaz). No es obligatoria, pero **es muy recomendable**: si te equivocas, el compilador te avisa.

```java
class Animal {
    void hacerSonido() {
        System.out.println("Sonido de animal");
    }
}

class Perro extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("¡Guau!");
    }
}
```

¿Qué pasa si te equivocas en el nombre del método?

```java
class Perro extends Animal {
    @Override
    void hacerSonid() {          // ¡falta una "o"!
        System.out.println("¡Guau!");
    }
}
```

El compilador responde con un error:

```text
error: method does not override or implement a method from a supertype
```

Sin `@Override`, ese código compilaría sin quejas, pero el método nunca se sobrescribiría y el programa se comportaría de forma inesperada. Por eso `@Override` previene errores silenciosos.

### 5.2 `@Deprecated`

Marca un elemento como **obsoleto**: no debería usarse porque puede desaparecer o tiene un reemplazo mejor. El compilador avisa cuando alguien lo usa.

```java
public class Calculadora {

    /**
     * @deprecated Usa {@link #sumar(int, int)} en su lugar.
     */
    @Deprecated(since = "2.0", forRemoval = true)
    public int suma(int a, int b) {
        return sumar(a, b);
    }

    public int sumar(int a, int b) {
        return a + b;
    }
}
```

Elementos útiles de `@Deprecated`:

| Elemento | Significado |
|---|---|
| `since` | Versión en la que se marcó como obsoleto. |
| `forRemoval` | Si es `true`, indica que se **eliminará** en el futuro. El aviso es más serio (categoría `removal`). |

Buena práctica: acompaña siempre `@Deprecated` con la etiqueta Javadoc `@deprecated` explicando **qué usar en su lugar**. Fíjate en la diferencia: la anotación se escribe con `D` mayúscula y la etiqueta de Javadoc con `d` minúscula.

### 5.3 `@SuppressWarnings`

Le dice al compilador que **no muestre** ciertos avisos.

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    @SuppressWarnings({"rawtypes", "unchecked"})
    public static void main(String[] args) {
        List coches = new ArrayList();   // tipo "crudo" (raw type)
        coches.add("Volvo");             // genera aviso "unchecked"
        System.out.println(coches);
    }
}
```

Los valores más habituales son `"unchecked"`, `"deprecation"`, `"rawtypes"` y `"unused"`.

> **Ojo:** suprimir un aviso es "esconder la suciedad bajo la alfombra". La mayoría de las veces es mejor **arreglar la causa**. En el ejemplo anterior, lo correcto es usar genéricos:
>
> ```java
> List<String> coches = new ArrayList<>();
> ```
>
> Usa `@SuppressWarnings` solo cuando estés seguro de que el código es correcto (por ejemplo, con código antiguo que no puedes modificar), y aplícala al **ámbito más pequeño posible** (una variable o un método, no una clase entera).

### 5.4 `@SafeVarargs`

Se aplica a un método o constructor con parámetros *varargs* genéricos y afirma que no realiza operaciones inseguras con ellos. Elimina los avisos de tipo "unchecked" asociados. Solo puede usarse en constructores y en métodos `static`, `final` o `private`.

```java
import java.util.List;

public class Utilidades {
    @SafeVarargs
    static <T> List<T> listaDe(T... elementos) {
        return List.of(elementos);
    }
}
```

### 5.5 `@FunctionalInterface`

Indica que una interfaz está pensada para ser una **interfaz funcional** (con un único método abstracto), la que se usa con expresiones lambda. Si añades un segundo método abstracto por error, el compilador te avisa.

```java
@FunctionalInterface
interface Saludo {
    String saludar(String nombre);
}

public class Main {
    public static void main(String[] args) {
        Saludo s = nombre -> "Hola, " + nombre;
        System.out.println(s.saludar("Luis"));   // Hola, Luis
    }
}
```

### 5.6 Otra que verás: `@Serial`

Disponible desde Java 14 (paquete `java.io`). Se usa en clases serializables para marcar miembros especiales como `serialVersionUID` y que el compilador verifique que están bien declarados. Es menos frecuente al empezar.

---

## 6. Meta-anotaciones

Son anotaciones que se aplican **a otras anotaciones**. Viven en `java.lang.annotation` y sirven para *configurar* tus propias anotaciones.

### 6.1 `@Retention` – ¿hasta cuándo "vive" la anotación?

| Valor de `RetentionPolicy` | Significado |
|---|---|
| `SOURCE` | Solo existe en el código fuente. El compilador la descarta. (Ej.: `@Override`) |
| `CLASS` | Se guarda en el `.class`, pero la JVM no la carga en ejecución. **Es el valor por defecto.** |
| `RUNTIME` | Se guarda en el `.class` y la JVM la carga: **se puede leer con reflexión**. |

### 6.2 `@Target` – ¿dónde se puede poner?

Restringe los elementos sobre los que se puede aplicar la anotación. Algunos valores de `ElementType`:

| Valor | Se puede aplicar a… |
|---|---|
| `TYPE` | Clases, interfaces, enums, records y anotaciones |
| `FIELD` | Campos (atributos) |
| `METHOD` | Métodos |
| `PARAMETER` | Parámetros de métodos |
| `CONSTRUCTOR` | Constructores |
| `LOCAL_VARIABLE` | Variables locales |
| `ANNOTATION_TYPE` | Otras anotaciones |
| `PACKAGE` | Paquetes |
| `MODULE` | Módulos |
| `RECORD_COMPONENT` | Componentes de un `record` |
| `TYPE_PARAMETER` | Parámetros de tipo genérico (`<T>`) |
| `TYPE_USE` | Cualquier *uso* de un tipo (ver sección 10) |

Si no pones `@Target`, la anotación puede usarse en casi cualquier declaración.

### 6.3 `@Documented`

Hace que la anotación aparezca en la documentación generada por Javadoc. Por defecto, las anotaciones no se incluyen.

### 6.4 `@Inherited`

Hace que una anotación de **clase** sea heredada por sus subclases. Por defecto no se hereda. Solo funciona sobre declaraciones de clases.

### 6.5 `@Repeatable`

Permite aplicar la misma anotación **más de una vez** sobre el mismo elemento (ver [sección 9](#9-anotaciones-repetibles)).

---

## 7. Crear tu propia anotación

Se declara con `@interface` (el `@` pegado a `interface`):

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)   // se puede leer en ejecución
@Target(ElementType.METHOD)           // solo se puede poner en métodos
@interface Prueba {
    String descripcion() default "";  // elemento con valor por defecto
    boolean activa() default true;    // otro elemento con valor por defecto
}
```

Uso:

```java
class MiClase {
    @Prueba(descripcion = "Comprueba la suma")
    void probarSuma() { }

    @Prueba(activa = false)
    void probarResta() { }

    @Prueba                       // usa los valores por defecto
    void probarDivision() { }
}
```

### Reglas de los elementos

Los elementos se declaran como métodos **sin parámetros** y sin `throws`. Su tipo solo puede ser:

- Tipos primitivos (`int`, `boolean`, `double`…)
- `String`
- `Class` (o `Class<? extends X>`)
- Un `enum`
- Otra anotación
- Un array de cualquiera de los anteriores

```java
enum Nivel { JUNIOR, SEMI_SENIOR, SENIOR }

@interface Autor {
    String nombre();                         // obligatorio (sin default)
    String[] colaboradores() default {};     // array, vacío por defecto
    Nivel nivel() default Nivel.JUNIOR;      // enum
}

@Autor(nombre = "Ana", colaboradores = {"Luis", "Marta"}, nivel = Nivel.SENIOR)
class Proyecto { }
```

Importante: **los elementos sin `default` son obligatorios**, y el valor `null` no está permitido en ningún elemento.

### Anotación "marcadora"

Una anotación sin elementos que solo sirve para *marcar* algo:

```java
@Retention(RetentionPolicy.RUNTIME)
@interface Importante { }
```

---

## 8. Leer anotaciones en tiempo de ejecución (reflexión)

Una anotación **no hace nada por sí sola**: necesita que alguien la lea. Con la API de reflexión puedes hacerlo, **siempre que la anotación tenga `@Retention(RetentionPolicy.RUNTIME)`**.

Este mini ejecutor de pruebas (parecido en espíritu a JUnit) busca los métodos marcados con `@Prueba` y los ejecuta. Guárdalo como `EjecutorDePruebas.java` y ejecútalo con `java EjecutorDePruebas.java`:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Method;

public class EjecutorDePruebas {

    public static void main(String[] args) throws Exception {
        Class<?> clase = CalculadoraTest.class;
        Object instancia = clase.getDeclaredConstructor().newInstance();

        for (Method metodo : clase.getDeclaredMethods()) {
            // 1. ¿Tiene la anotación?
            if (!metodo.isAnnotationPresent(Prueba.class)) {
                continue;
            }

            // 2. Leemos sus valores
            Prueba prueba = metodo.getAnnotation(Prueba.class);

            if (!prueba.activa()) {
                System.out.println("[OMITIDA] " + metodo.getName());
                continue;
            }

            // 3. Actuamos en consecuencia
            try {
                metodo.invoke(instancia);
                System.out.println("[OK]      " + prueba.descripcion());
            } catch (Exception e) {
                System.out.println("[FALLO]   " + prueba.descripcion());
            }
        }
    }
}

// --- Clase con pruebas ---
class CalculadoraTest {

    @Prueba(descripcion = "2 + 2 debe ser 4")
    public void sumaCorrecta() {
        if (2 + 2 != 4) throw new IllegalStateException();
    }

    @Prueba(descripcion = "Esta prueba falla a propósito")
    public void pruebaQueFalla() {
        throw new IllegalStateException("¡Error!");
    }

    @Prueba(activa = false)
    public void pruebaDesactivada() { }

    public void metodoSinAnotar() { }   // se ignora
}

// --- La anotación ---
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Prueba {
    String descripcion() default "";
    boolean activa() default true;
}
```

Salida (el orden puede variar, porque `getDeclaredMethods()` no garantiza ningún orden):

```text
[OK]      2 + 2 debe ser 4
[FALLO]   Esta prueba falla a propósito
[OMITIDA] pruebaDesactivada
```

### Métodos de reflexión más usados

| Método | Qué hace |
|---|---|
| `isAnnotationPresent(X.class)` | ¿Tiene la anotación `X`? (devuelve `boolean`) |
| `getAnnotation(X.class)` | Devuelve la anotación `X`, o `null` si no está |
| `getAnnotations()` | Devuelve todas las anotaciones del elemento |
| `getAnnotationsByType(X.class)` | Devuelve las anotaciones `X`, incluso si son repetidas |

Estos métodos existen en `Class`, `Method`, `Field`, `Constructor`, `Parameter`, etc.

---

## 9. Anotaciones repetibles

Desde Java 8 puedes poner la **misma anotación varias veces** en un elemento. Necesitas dos piezas:

1. La anotación marcada con `@Repeatable`, que indica cuál es su *contenedor*.
2. La anotación **contenedora**, con un elemento `value` que es un array de la primera.

```java
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
@Repeatable(Roles.class)          // apunta al contenedor
@interface Rol {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
@interface Roles {                // contenedor
    Rol[] value();
}

@Rol("ADMIN")
@Rol("AUDITOR")
class PanelDeControl { }

public class Main {
    public static void main(String[] args) {
        Rol[] roles = PanelDeControl.class.getAnnotationsByType(Rol.class);
        for (Rol r : roles) {
            System.out.println(r.value());   // ADMIN, AUDITOR
        }
    }
}
```

Detalle importante: para que ambas se lean en ejecución, las **dos** necesitan `@Retention(RUNTIME)`. Y usa `getAnnotationsByType` (no `getAnnotation`) para obtener las repetidas de forma cómoda.

---

## 10. Anotaciones de tipo (type annotations)

Desde Java 8, las anotaciones se pueden aplicar a **cualquier uso de un tipo**, no solo a declaraciones. Para esto la anotación debe declararse con `@Target(ElementType.TYPE_USE)`.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.List;

@Target(ElementType.TYPE_USE)
@interface NoNulo { }

public class Main {
    public static void main(String[] args) {
        // Argumento de tipo genérico
        List<@NoNulo String> nombres = new ArrayList<>();

        // Creación de objeto
        Object o = new @NoNulo Object();

        // Conversión (cast)
        String s = (@NoNulo String) "texto";

        // Declaración de excepciones
        // void metodo() throws @Critica MiExcepcion { }
    }
}
```

**¿Para qué sirve si el compilador estándar no las comprueba?** Las anotaciones de tipo son la base de los *pluggable type systems* (sistemas de tipos enchufables): herramientas externas, como el **Checker Framework**, las usan para verificar reglas más estrictas (por ejemplo, "esta variable nunca es `null`") y detectar errores **antes** de ejecutar el programa. `javac` por sí solo no impone el significado de `@NoNulo`; solo permite escribirla.

---

## 11. Anotaciones y características modernas de Java

El mecanismo básico de anotaciones es estable desde Java 8, pero hay detalles de versiones recientes que conviene conocer.

### 11.1 Records

Cuando anotas un componente de un `record`, la anotación se **propaga** a los elementos generados (campo, método de acceso, parámetro del constructor, componente del record), según lo que permita su `@Target`.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD})
@interface Sensible { }

record Usuario(String nombre, @Sensible String contrasena) { }
```

Aquí `@Sensible` se aplica tanto al campo `contrasena` como a su método `contrasena()`, porque ambos están en su `@Target`. Para que además quede en el *componente* del record (visible con `RecordComponent`), habría que añadir `ElementType.RECORD_COMPONENT` al `@Target` (ver [sección 12.3](#12-reflexión-en-profundidad-y-reglas-adicionales)).

### 11.2 `@Override` en records e interfaces

`@Override` también es válido para métodos de acceso de un record y para implementar métodos de interfaces:

```java
record Punto(int x, int y) {
    @Override
    public int x() {          // sobrescribe el accesor generado
        return Math.abs(x);
    }
}
```

### 11.3 Procesadores de anotaciones desactivados por defecto (JDK 23+)

Un *procesador de anotaciones* es una herramienta que se ejecuta **durante la compilación** (el segundo uso de la sección 2): lee anotaciones y genera código. Desde **JDK 23**, `javac` **ya no ejecuta automáticamente** los procesadores que encuentra en el classpath; hay que habilitarlos explícitamente, por ejemplo con la opción `-proc:full` (o indicándolos con `-processor` / `-processorpath`).

Si al actualizar a Java 25 un proyecto que usaba generación de código por anotaciones deja de generar clases, revisa esta configuración en tu herramienta de construcción (Maven o Gradle). La mayoría de proyectos lo resuelven en el `pom.xml` o `build.gradle`.

### 11.4 Resumen de qué es "reciente" y qué es "clásico"

| Característica | Desde |
|---|---|
| `@FunctionalInterface`, `@Repeatable`, anotaciones de tipo | Java 8 |
| `@SafeVarargs` (con soporte para métodos privados) | Java 9 |
| `@Serial`, records y su propagación de anotaciones | Java 14–16 |
| Procesamiento de anotaciones desactivado por defecto | JDK 23 |
| Class-File API estándar (`java.lang.classfile`), que permite inspeccionar anotaciones en archivos `.class` | JDK 24 |

---

## 12. Reflexión en profundidad y reglas adicionales

### 12.1 `@Inherited`: anotaciones que pasan a las subclases

Con `@Inherited`, una anotación puesta en una **clase** también "se ve" desde sus subclases. Fíjate en la diferencia entre `getAnnotations()` (incluye las heredadas) y `getDeclaredAnnotations()` (solo las declaradas directamente en esa clase):

```java
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

public class HerenciaDemo {
    public static void main(String[] args) {
        System.out.println(Base.class.isAnnotationPresent(Auditable.class));      // true
        System.out.println(Derivada.class.isAnnotationPresent(Auditable.class));  // true (heredada)
        System.out.println(Derivada.class.getDeclaredAnnotations().length);       // 0 (no la declara ella)
        System.out.println(Derivada.class.getAnnotations().length);               // 1
    }
}

@Auditable
class Base { }

class Derivada extends Base { }

@Retention(RetentionPolicy.RUNTIME)
@Inherited
@interface Auditable { }
```

Recuerda: `@Inherited` solo funciona de **clase a subclase**. No se hereda desde interfaces ni afecta a métodos o campos.

### 12.2 Anotaciones en parámetros

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;

public class ParametrosDemo {

    void saludar(@Nombre("persona") String quien, int veces) { }

    public static void main(String[] args) throws Exception {
        Method m = ParametrosDemo.class.getDeclaredMethod("saludar", String.class, int.class);
        for (Parameter p : m.getParameters()) {
            Nombre n = p.getAnnotation(Nombre.class);
            System.out.println(p.getType().getSimpleName() + " -> "
                    + (n != null ? n.value() : "sin anotación"));
        }
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
@interface Nombre {
    String value();
}
```

Salida:

```text
String -> persona
int -> sin anotación
```

### 12.3 Anotaciones en componentes de un `record`

Para poder leer la anotación directamente desde el **componente** del record, su `@Target` debe incluir `RECORD_COMPONENT`:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.RecordComponent;

public class LeerRecord {
    public static void main(String[] args) {
        for (RecordComponent c : Usuario.class.getRecordComponents()) {
            Sensible s = c.getAnnotation(Sensible.class);
            System.out.println(c.getName() + (s != null ? " -> sensible" : ""));
        }
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.RECORD_COMPONENT, ElementType.FIELD, ElementType.METHOD})
@interface Sensible { }

record Usuario(String nombre, @Sensible String contrasena) { }
```

Salida:

```text
nombre
contrasena -> sensible
```

### 12.4 Una anotación es, en el fondo, una interfaz

Cada anotación que lees por reflexión es un objeto que implementa la interfaz de tu `@interface`. Por eso puedes llamar a sus "métodos" (los elementos) y también a estos:

| Método | Qué devuelve |
|---|---|
| `anotacion.annotationType()` | La clase de la anotación (por ejemplo, `Prueba.class`) |
| `anotacion.toString()` | Una representación textual con sus valores |
| `anotacion.equals(otra)` | `true` si son del mismo tipo y tienen los mismos valores |

Además, por diseño del lenguaje:

- No puedes crear una con `new`; el valor lo construye la JVM al leerla.
- Una anotación **no puede extender** a otra ni a otra interfaz.
- Una anotación **no puede ser genérica**.
- Sus elementos no pueden lanzar excepciones ni recibir parámetros.

### 12.5 Los valores deben ser constantes de compilación

El valor de un elemento tiene que poder calcularse al **compilar**. Una constante (`static final` con valor fijo) sirve; una variable normal, no:

```java
@interface Limite {
    int value();
}

class Config {
    static final int MAXIMO = 100;     // constante: válida
    static int dinamico = 50;          // variable: NO es constante

    @Limite(MAXIMO)                    // OK
    void a() { }

    @Limite(MAXIMO * 2)                // OK: la expresión también es constante
    void b() { }

    // @Limite(dinamico)               // ERROR de compilación
    // void c() { }
}
```

### 12.6 Limitaciones de lectura

- Las anotaciones sobre **variables locales** no se pueden leer por reflexión (no hay forma de "preguntar" por una variable local en ejecución). Sirven para el compilador o herramientas de análisis.
- Para leer nombres reales de parámetros con `Parameter.getName()` hay que compilar con la opción `-parameters`; si no, verás `arg0`, `arg1`… Las anotaciones de parámetros, en cambio, se leen sin ese requisito.

---

## 13. Ejemplo práctico: un validador de campos

Este ejemplo junta casi todo lo aprendido: anotaciones propias, `record`, reflexión y `instanceof` con patrones. Es una versión muy reducida de lo que hace una librería de validación (como Jakarta Bean Validation).

Guarda el archivo como `ValidadorDemo.java` y ejecútalo con `java ValidadorDemo.java`:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

public class ValidadorDemo {

    public static void main(String[] args) throws IllegalAccessException {
        Registro correcto = new Registro("Ana", "ana@correo.com", 25);
        Registro incorrecto = new Registro("", "sin-arroba", 12);

        System.out.println("correcto:   " + validar(correcto));
        System.out.println("incorrecto: " + validar(incorrecto));
    }

    static List<String> validar(Object objeto) throws IllegalAccessException {
        List<String> errores = new ArrayList<>();

        for (Field campo : objeto.getClass().getDeclaredFields()) {
            campo.setAccessible(true);
            Object valor = campo.get(objeto);

            if (campo.isAnnotationPresent(NoVacio.class)
                    && (valor == null || valor.toString().isBlank())) {
                errores.add(campo.getName() + " no puede estar vacío");
            }

            Contiene contiene = campo.getAnnotation(Contiene.class);
            if (contiene != null && valor instanceof String texto
                    && !texto.contains(contiene.value())) {
                errores.add(campo.getName() + " debe contener " + contiene.value());
            }

            Rango rango = campo.getAnnotation(Rango.class);
            if (rango != null && valor instanceof Integer numero
                    && (numero < rango.min() || numero > rango.max())) {
                errores.add(campo.getName() + " debe estar entre "
                        + rango.min() + " y " + rango.max());
            }
        }
        return errores;
    }
}

// Los datos a validar: la anotación en un componente se propaga al campo
record Registro(
        @NoVacio String nombre,
        @NoVacio @Contiene("@") String correo,
        @Rango(min = 18, max = 120) int edad) { }

// Las anotaciones de validación
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface NoVacio { }

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface Contiene {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface Rango {
    int min();
    int max();
}
```

Salida esperada:

```text
correcto:   []
incorrecto: [nombre no puede estar vacío, correo debe contener @, edad debe estar entre 18 y 120]
```

Observa que **la lógica está separada de los datos**: `Registro` solo declara *qué* reglas tiene (con anotaciones) y `validar` decide *cómo* aplicarlas. Esa separación es justamente lo que hacen los frameworks. El orden de los errores depende del orden en que `getDeclaredFields()` devuelva los campos; la especificación no lo garantiza, aunque en la práctica suele coincidir con el orden de declaración.

---

## 14. Anotaciones en paquetes y módulos

### Paquetes: `package-info.java`

Para anotar un paquete se crea un archivo especial llamado `package-info.java` dentro de la carpeta del paquete. Contiene solo la declaración del paquete, precedida de las anotaciones (y, opcionalmente, un comentario Javadoc):

```java
// Archivo: com/ejemplo/interno/package-info.java

/**
 * Clases de uso interno de la aplicación.
 */
@AplicacionInterna
package com.ejemplo.interno;
```

La anotación debe declararse con `@Target(ElementType.PACKAGE)`. Si además tiene `@Retention(RUNTIME)`, podrás leerla con `Class.getPackage().getAnnotation(...)`.

### Módulos: `module-info.java`

Desde Java 9, `@Deprecated` (y cualquier anotación con `@Target(ElementType.MODULE)`) puede aplicarse a un módulo:

```java
// Archivo: module-info.java
@Deprecated(since = "3.0", forRemoval = true)
module mi.modulo.antiguo {
    exports com.ejemplo.api;
}
```

---

## 15. Procesadores de anotaciones (en compilación)

Hasta ahora las anotaciones se leían **mientras el programa se ejecuta**. Los *procesadores de anotaciones* las leen **mientras `javac` compila**. Con ellos se puede:

- Mostrar avisos o errores personalizados al compilar.
- Generar nuevas clases o archivos automáticamente (es lo que hacen herramientas como Lombok o MapStruct).

Para esto no hace falta `RetentionPolicy.RUNTIME`: basta con `SOURCE`.

### Un procesador mínimo

Estructura de archivos (paquete `ejemplo`):

```text
ejemplo/Importante.java
ejemplo/ProcesadorImportante.java
ejemplo/Cliente.java
```

**`ejemplo/Importante.java`**

```java
package ejemplo;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface Importante { }
```

**`ejemplo/ProcesadorImportante.java`**

```java
package ejemplo;

import java.util.Set;
import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.tools.Diagnostic;

@SupportedAnnotationTypes("ejemplo.Importante")
public class ProcesadorImportante extends AbstractProcessor {

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public boolean process(Set<? extends TypeElement> anotaciones, RoundEnvironment ronda) {
        for (Element elemento : ronda.getElementsAnnotatedWith(Importante.class)) {
            processingEnv.getMessager().printMessage(
                    Diagnostic.Kind.NOTE,
                    "Clase marcada como importante: " + elemento.getSimpleName(),
                    elemento);
        }
        return false;
    }
}
```

**`ejemplo/Cliente.java`**

```java
package ejemplo;

@Importante
public class Cliente { }
```

### Compilar y ejecutar el procesador

```bash
# 1. Compilar la anotación y el procesador (sin procesar anotaciones)
javac -proc:none -d clases ejemplo/Importante.java ejemplo/ProcesadorImportante.java

# 2. Compilar el cliente indicando explícitamente el procesador
javac -cp clases -processor ejemplo.ProcesadorImportante -d salida ejemplo/Cliente.java
```

Durante el segundo paso `javac` debería mostrar un mensaje de tipo *Note* mencionando `Cliente`.

### Opciones de `javac` relacionadas

| Opción | Efecto |
|---|---|
| `-proc:none` | No ejecuta procesadores de anotaciones. |
| `-proc:only` | Solo procesa anotaciones; no genera `.class`. |
| `-proc:full` | Ejecuta los procesadores encontrados (necesaria desde JDK 23 si no indicas `-processor`). |
| `-processor <clases>` | Indica explícitamente qué procesadores usar. |
| `-processorpath <ruta>` | Dónde buscar los procesadores. |

> **Recuerda (JDK 23+):** si no indicas ninguna de estas opciones, `javac` ya **no** busca procesadores automáticamente en el classpath. Si tienes dudas, consulta `javac --help` en tu instalación.

En un proyecto real, los procesadores se registran para que `javac` los descubra mediante el archivo `META-INF/services/javax.annotation.processing.Processor` (un mecanismo de `ServiceLoader`), y los gestores de dependencias (Maven, Gradle) se encargan de la configuración.

### Leer anotaciones desde archivos `.class`

Otra opción, más avanzada, es analizar archivos `.class` **sin cargarlos** en la JVM. Desde JDK 24 existe una API estándar para ello, la *Class-File API* (`java.lang.classfile`), que permite inspeccionar entre otras cosas las anotaciones guardadas en el bytecode. Es un tema para más adelante, pero conviene saber que existe.

---

## 16. Anotaciones en el ecosistema Java

Además de las anotaciones de Java SE, casi todos los proyectos usan anotaciones de **bibliotecas y frameworks**. Estas **no vienen con el JDK**: hay que añadir la dependencia correspondiente.

| Biblioteca | Anotaciones típicas | Para qué se usan |
|---|---|---|
| **JUnit 5** | `@Test`, `@BeforeEach`, `@Disabled` | Escribir pruebas automáticas |
| **Spring** | `@Component`, `@Autowired`, `@RestController`, `@GetMapping` | Inyección de dependencias y servicios web |
| **Jakarta Persistence (JPA)** | `@Entity`, `@Id`, `@Table` | Mapear clases a tablas de base de datos |
| **Jakarta Bean Validation** | `@NotNull`, `@Size`, `@Email` | Validar datos (como nuestro validador, pero profesional) |
| **Jackson** | `@JsonProperty`, `@JsonIgnore` | Convertir objetos a/desde JSON |
| **Lombok** | `@Getter`, `@Setter`, `@Data` | Generar código repetitivo al compilar (procesador de anotaciones) |

> En versiones recientes de Jakarta EE el paquete base es `jakarta.*`; en versiones antiguas era `javax.*`. Si ves ambos en tutoriales, es por eso.

Fíjate cómo se parece este test de JUnit a nuestro `@Prueba` de la sección 8: JUnit hace, a gran escala, exactamente lo mismo (buscar métodos anotados por reflexión y ejecutarlos).

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void sumaDosNumeros() {
        assertEquals(4, 2 + 2);
    }
}
```

---

## 17. Errores comunes y buenas prácticas

### Errores frecuentes

1. **Olvidar `@Retention(RetentionPolicy.RUNTIME)`.** El valor por defecto es `CLASS`, así que `getAnnotation(...)` devolverá `null` y tu código "no funcionará" sin dar error. Es el fallo más típico al crear anotaciones propias.
2. **Pensar que la anotación hace algo por sí sola.** `@Prueba` o `@Importante` son solo etiquetas. Necesitas código (reflexión, un framework o un procesador) que las lea.
3. **Olvidar `@Target`.** Sin él, la anotación podría colocarse en sitios sin sentido.
4. **Usar `@SuppressWarnings` para "callar" al compilador** en lugar de corregir el problema.
5. **Escribir `@Override` mal o no usarlo.** Es la protección más barata contra errores de nombre o de firma.
6. **Confundir `@deprecated` (Javadoc) con `@Deprecated` (anotación).** Lo ideal es usar **las dos** juntas.
7. **Usar `@Inherited` esperando que funcione en métodos o interfaces.** Solo afecta a clases y sus subclases.
8. **Usar un valor no constante en un elemento** (por ejemplo, una variable normal): el compilador lo rechaza.
9. **Asumir que `RECORD_COMPONENT` está incluido** en el `@Target`: si no lo pones, no podrás leer la anotación desde `getRecordComponents()`.
10. **Actualizar de JDK y que "dejen de generarse clases".** Revisa la configuración de procesadores de anotaciones (JDK 23+).

### Buenas prácticas

- Usa **siempre** `@Override` al sobrescribir métodos.
- Aplica `@SuppressWarnings` en el ámbito más pequeño posible y deja un comentario explicando por qué.
- Al crear tus propias anotaciones, declara siempre `@Retention` y `@Target` de forma explícita.
- Da valores `default` razonables a los elementos para que la anotación sea cómoda de usar.
- Usa nombres claros y, si solo hay un elemento, llámalo `value` para poder usar la sintaxis corta.
- Pon cada anotación en su propia línea sobre la declaración.
- No abuses: si una simple llamada a un método o un parámetro resuelve el problema con claridad, probablemente no necesitas una anotación propia.
- Documenta tus anotaciones con Javadoc y, si quieres que aparezcan en la documentación de quien las use, añade `@Documented`.

---

## 18. Preguntas frecuentes

**¿Las anotaciones hacen que mi programa sea más lento?**
Ponerlas no cuesta nada en ejecución si su retención es `SOURCE` o `CLASS`. Si son `RUNTIME`, leerlas por reflexión tiene un coste pequeño; en aplicaciones normales es irrelevante, pero conviene no leerlas repetidamente en bucles críticos (guarda el resultado).

**¿Puedo modificar el valor de una anotación en ejecución?**
No. Son metadatos fijados al compilar; los valores se leen, no se cambian.

**¿Una anotación puede heredar de otra?**
No. Para "combinar" varias, lo habitual es que tu código (o el framework) busque anotaciones dentro de otras anotaciones.

**¿Cuál es la diferencia entre una anotación y un comentario?**
Un comentario lo ignora el compilador; una anotación forma parte del código, la valida el compilador y puede leerse con herramientas o en ejecución.

**¿Cuál es la diferencia entre una anotación marcadora y una interfaz marcadora?**
Ambas "etiquetan" una clase (`Serializable` es una interfaz marcadora). Las anotaciones son más flexibles: se pueden poner en métodos, campos, parámetros…, y admiten elementos con valores.

**¿Necesito saber crear anotaciones para ser programador Java?**
Para el día a día, **usarlas** es lo imprescindible. Crear las tuyas es útil, pero se hace con menos frecuencia, normalmente al escribir bibliotecas o herramientas internas.

---

## 19. Resumen rápido

| Concepto | Idea clave |
|---|---|
| Anotación | Metadato que empieza por `@`; no cambia la lógica por sí misma. |
| `@Override` | Verifica que realmente sobrescribes un método. |
| `@Deprecated` | Marca algo como obsoleto (`since`, `forRemoval`). |
| `@SuppressWarnings` | Silencia avisos del compilador; úsala con moderación. |
| `@SafeVarargs` | Declara seguro un varargs genérico. |
| `@FunctionalInterface` | Garantiza que la interfaz tiene un único método abstracto. |
| `@interface` | Sirve para declarar una anotación propia. |
| `@Retention` | `SOURCE`, `CLASS` (defecto) o `RUNTIME`. |
| `@Target` | Define dónde se puede usar la anotación. |
| `@Inherited` | Hace que una anotación de clase pase a sus subclases. |
| `@Repeatable` | Permite repetir la anotación en un mismo elemento. |
| `TYPE_USE` | Permite anotar usos de tipos (`List<@X String>`). |
| `RECORD_COMPONENT` | Permite leer la anotación desde el componente del record. |
| Reflexión | `getAnnotation(...)` solo funciona con `RUNTIME`. |
| Procesadores | Leen anotaciones al compilar; en JDK 23+ hay que activarlos explícitamente. |
| Valores | Deben ser constantes de compilación y nunca `null`. |

---

## 20. Ejercicios propuestos

### Ejercicio 1 – `@Override`
Crea una clase `Figura` con un método `area()` que devuelva `0`. Crea `Circulo` que lo sobrescriba. Introduce un error de ortografía en el nombre del método sobrescrito (con `@Override`) y observa el mensaje del compilador. Corrígelo.

### Ejercicio 2 – `@Deprecated`
Crea un método `saludarViejo()` marcado como `@Deprecated(since = "1.0", forRemoval = true)` y otro `saludar()` que lo reemplace. Llama al método obsoleto desde `main` y compila con `javac -Xlint:all` para ver el aviso.

### Ejercicio 3 – Anotación propia
Crea una anotación `@Tarea` con los elementos `String responsable()` y `int prioridad() default 3`, aplicable solo a métodos y legible en ejecución. Anota tres métodos con distintas prioridades.

### Ejercicio 4 – Leer con reflexión
Escribe un programa que recorra los métodos del ejercicio 3 y muestre solo aquellos con prioridad `1` o `2`, indicando el responsable.

### Ejercicio 5 – Anotación repetible *(reto)*
Crea una anotación repetible `@Etiqueta("...")` y aplícala tres veces a una clase. Con `getAnnotationsByType` muestra todas las etiquetas por consola.

### Ejercicio 6 – Ampliar el validador *(reto)*
Parte del ejemplo de la sección 13 y añade una anotación `@LongitudMaxima(int)` para campos `String`. Aplícala al campo `nombre` con un máximo de 10 caracteres y comprueba que falla con un nombre más largo.

### Ejercicio 7 – Herencia de anotaciones
Replica el ejemplo de `@Inherited` y comprueba qué ocurre si **quitas** `@Inherited` de la anotación: ¿qué imprime ahora `Derivada.class.isAnnotationPresent(Auditable.class)`?

### Ejercicio 8 – Procesador *(avanzado)*
Modifica `ProcesadorImportante` para que, en lugar de un `NOTE`, emita un `Diagnostic.Kind.WARNING` cuando la clase marcada como `@Importante` **no** sea `public`.

### Pistas
- Ejercicio 3: usa `@Target(ElementType.METHOD)` y `@Retention(RetentionPolicy.RUNTIME)`.
- Ejercicio 4: `getDeclaredMethods()` + `getAnnotation(Tarea.class)`; comprueba que no sea `null`.
- Ejercicio 5: necesitas una anotación contenedora con un elemento `Etiqueta[] value()`.
- Ejercicio 6: copia la estructura de `Rango` y usa `texto.length()`.
- Ejercicio 8: `elemento.getModifiers()` devuelve un `Set<Modifier>`; busca `Modifier.PUBLIC`.

---

## 21. Soluciones (ejercicios 3, 4, 5 y 7)

Intenta resolverlos tú primero y compara después.

### Ejercicios 3 y 4

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Method;

public class Agenda {

    @Tarea(responsable = "Ana", prioridad = 1)
    void entregarInforme() { }

    @Tarea(responsable = "Luis")                // prioridad 3 por defecto
    void ordenarEscritorio() { }

    @Tarea(responsable = "Marta", prioridad = 2)
    void revisarCorreo() { }

    public static void main(String[] args) {
        for (Method m : Agenda.class.getDeclaredMethods()) {
            Tarea t = m.getAnnotation(Tarea.class);
            if (t != null && t.prioridad() <= 2) {
                System.out.println(m.getName() + " -> " + t.responsable()
                        + " (prioridad " + t.prioridad() + ")");
            }
        }
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Tarea {
    String responsable();
    int prioridad() default 3;
}
```

### Ejercicio 5

```java
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Etiqueta("java")
@Etiqueta("anotaciones")
@Etiqueta("junior")
public class Articulo {
    public static void main(String[] args) {
        for (Etiqueta e : Articulo.class.getAnnotationsByType(Etiqueta.class)) {
            System.out.println(e.value());
        }
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Repeatable(Etiquetas.class)
@interface Etiqueta {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
@interface Etiquetas {
    Etiqueta[] value();
}
```

### Ejercicio 7

Sin `@Inherited`, `Derivada.class.isAnnotationPresent(Auditable.class)` imprime **`false`**: la subclase no "ve" la anotación de `Base`. Solo `Base.class.isAnnotationPresent(Auditable.class)` seguirá devolviendo `true`.

---

## 22. Glosario

| Término | Significado |
|---|---|
| **Anotación** | Metadato en el código, escrito con `@`. |
| **Elemento** | Cada "campo" declarado dentro de una anotación (por ejemplo, `prioridad()`). |
| **Meta-anotación** | Anotación que se aplica a otra anotación (`@Retention`, `@Target`…). |
| **Retención (retention)** | Hasta qué fase se conserva la anotación: fuente, clase o ejecución. |
| **Reflexión (reflection)** | API de Java para inspeccionar clases, métodos y campos en ejecución. |
| **Procesador de anotaciones** | Programa que `javac` ejecuta durante la compilación para leer anotaciones. |
| **Anotación marcadora** | Anotación sin elementos que solo "etiqueta" algo. |
| **Anotación contenedora** | Anotación que agrupa varias repetidas de otra (`Roles` respecto a `Rol`). |
| **Anotación de tipo** | Anotación con `TYPE_USE`, aplicable a usos de tipos. |
| **Metadatos** | Datos sobre los datos o sobre el programa, no parte de su lógica. |

---

## Referencias

- Oracle. *The Java™ Tutorials – Lesson: Annotations*. <https://docs.oracle.com/javase/tutorial/java/annotations/index.html>
  - [Annotations Basics](https://docs.oracle.com/javase/tutorial/java/annotations/basics.html)
  - [Predefined Annotation Types](https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html)
- W3Schools. *Java Annotations*. <https://www.w3schools.com/java/java_annotations.asp>
- Oracle. *Java Language Changes* (cambios del lenguaje desde Java 9). <https://docs.oracle.com/pls/topic/lookup?ctx=en/java/javase&id=java_language_changes>
- Oracle. *JDK Release Notes*. <https://www.oracle.com/technetwork/java/javase/jdk-relnotes-index-2162236.html>
- Dev.java – tutoriales actualizados. <https://dev.java/learn/>
