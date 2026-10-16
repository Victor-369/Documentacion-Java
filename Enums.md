# Enumeraciones (`enum`) en Java

Guía basada en el tutorial de W3Schools sobre enums, ampliada con las novedades del lenguaje desde Java 21 en adelante.

**Convenciones de este documento**

- Los ejemplos marcados con ✅ se han compilado y ejecutado con OpenJDK 21.
- Los ejemplos marcados con ⚠️ requieren una versión posterior a 21 y **no se han podido compilar en local**; se han tomado de la especificación oficial (JEP) y se indica la fuente.
- Las secciones 1 a 6 resumen el contenido de W3Schools. Las secciones 7 a 10 son ampliaciones.

---

## 1. Qué es un enum

Un `enum` es una «clase» especial que representa un grupo de **constantes** (variables inmutables, como las `final`).

Para crearlo se usa la palabra clave `enum` (en lugar de `class` o `interface`) y se separan las constantes con comas. Por convención se escriben en mayúsculas.

```java
enum Nivel {
  BAJO,
  MEDIO,
  ALTO
}
```

Se accede a las constantes con la notación de punto:

```java
Nivel miVar = Nivel.MEDIO;
```

> *Enum* es la abreviatura de *enumeration* («enumeración»), que significa «específicamente listado».

## 2. Enum dentro de una clase

Un `enum` también puede declararse dentro de una clase:

```java
public class Main {
  enum Nivel {
    BAJO,
    MEDIO,
    ALTO
  }

  public static void main(String[] args) {
    Nivel miVar = Nivel.MEDIO;
    System.out.println(miVar);
  }
}
```

Salida ✅:

```text
MEDIO
```

## 3. Enum en una sentencia `switch`

Los enums se usan a menudo en `switch` para comprobar valores concretos:

```java
Nivel miVar = Nivel.MEDIO;

switch (miVar) {
  case BAJO:
    System.out.println("Nivel bajo");
    break;
  case MEDIO:
    System.out.println("Nivel medio");
    break;
  case ALTO:
    System.out.println("Nivel alto");
    break;
}
```

Salida ✅:

```text
Nivel medio
```

## 4. Recorrer un enum

El tipo enum tiene un método `values()` que devuelve un array con todas sus constantes. Es útil para recorrerlas:

```java
for (Nivel miVar : Nivel.values()) {
  System.out.println(miVar);
}
```

Salida ✅:

```text
BAJO
MEDIO
ALTO
```

## 5. Diferencias entre enums y clases

Un `enum` puede, igual que una `class`, tener atributos y métodos. Las diferencias son:

- Las constantes de un enum son `public`, `static` y `final` (inmutables; no se pueden sobrescribir).
- Un enum **no se puede usar para crear objetos** con `new`.
- Un enum **no puede extender otras clases**, pero **sí puede implementar interfaces**.

Estas restricciones las comprueba el compilador ✅:

```java
enum E { A }
new E();                    // error: enum classes may not be instantiated
class C extends E {}        // error: cannot inherit from final E
```

## 6. ¿Por qué y cuándo usar enums?

Úsalos cuando tengas valores que sabes que no van a cambiar: días de la semana, meses, colores, palos de una baraja, niveles, estados, etc.

---

## 7. Ampliación: lo esencial de un enum «con cuerpo»

> Estas características no son novedades de Java 21, pero son la base necesaria para entender lo que viene después. W3Schools las trata en una página aparte («Enum Constructor»).

### 7.1. Atributos, constructor y métodos

El constructor de un enum es implícitamente privado. Las constantes llaman a él al declararse:

```java
enum Planeta {
    MERCURIO(3.303e+23, 2.4397e6),
    TIERRA(5.976e+24, 6.37814e6);

    private final double masa;
    private final double radio;

    Planeta(double masa, double radio) {
        this.masa = masa;
        this.radio = radio;
    }

    double gravedad() {
        return 6.67300E-11 * masa / (radio * radio);
    }
}

System.out.printf("%.2f%n", Planeta.TIERRA.gravedad());   // 9,80 ✅
```

### 7.2. Implementar interfaces y métodos abstractos por constante

Cada constante puede aportar su propia implementación:

```java
interface Descriptible { String descripcion(); }

enum Operacion implements Descriptible {
    SUMA("+")  { int aplicar(int a, int b) { return a + b; } },
    RESTA("-") { int aplicar(int a, int b) { return a - b; } };

    private final String simbolo;
    Operacion(String simbolo) { this.simbolo = simbolo; }

    abstract int aplicar(int a, int b);
    public String descripcion() { return "Operación " + simbolo; }
}

Operacion.SUMA.aplicar(2, 3);          // 5 ✅
Operacion.RESTA.descripcion();         // "Operación -" ✅
```

### 7.3. Métodos heredados más útiles

| Método | Qué hace | Ejemplo ✅ |
|---|---|---|
| `values()` | Array con todas las constantes, en orden de declaración | `Nivel.values()` |
| `valueOf(String)` | Devuelve la constante con ese nombre exacto; lanza `IllegalArgumentException` si no existe | `Nivel.valueOf("ALTO")` → `ALTO` |
| `name()` | Nombre de la constante como `String` | `Nivel.ALTO.name()` → `"ALTO"` |
| `ordinal()` | Posición (empieza en 0) | `Nivel.ALTO.ordinal()` → `2` |
| `compareTo(E)` | Diferencia de posiciones | `Nivel.BAJO.compareTo(Nivel.ALTO)` → `-2` |

Mensaje real de `valueOf` con un nombre inexistente ✅:

```text
No enum constant EnumTests.Nivel.NO_EXISTE
```

### 7.4. `EnumMap` y `EnumSet`

Colecciones especializadas para claves o elementos de un enum ✅:

```java
EnumMap<Nivel, String> mapa = new EnumMap<>(Nivel.class);
mapa.put(Nivel.ALTO, "a");
mapa.put(Nivel.BAJO, "b");
System.out.println(mapa);                       // {BAJO=b, ALTO=a}  (ordenado por posición del enum)

EnumSet.range(Nivel.BAJO, Nivel.MEDIO);         // [BAJO, MEDIO]
EnumSet.complementOf(EnumSet.of(Nivel.BAJO));   // [MEDIO, ALTO]
```

---

## 8. Novedades desde Java 21: `switch` con patrones y enums (JEP 441)

**Java 21** convierte en definitivo el *pattern matching* para `switch` ([JEP 441](https://openjdk.org/jeps/441), estado «Closed / Delivered», versión 21). Estos son los cambios que afectan a los enums.

### 8.1. `switch` exhaustivo sin `default`

Si un `switch` **expresión** cubre todas las constantes de un enum, no hace falta `default`; y es mejor no ponerlo: si algún día se añade una constante nueva y se recompila, el compilador avisará de que falta un caso en lugar de ocultarlo.

```java
static String describir(Nivel n) {
    return switch (n) {
        case BAJO  -> "bajo";
        case MEDIO -> "medio";
        case ALTO  -> "alto";
    };
}
```

Si falta una constante, el compilador lo rechaza ✅:

```text
error: the switch expression does not cover all possible input values
```

Según el JEP, si un `switch` exhaustivo sobre un enum no tiene cláusula «total» (como `default`), el compilador **sintetiza** una que lanza una excepción, por si se añade una constante nueva sin recompilar la clase que contiene el `switch`.

### 8.2. Cambio de excepción: `MatchException`

Para alinearlo con el resto de `switch` con patrones, una expresión `switch` sobre un enum que, en tiempo de ejecución, no encuentra ninguna etiqueta aplicable lanza ahora **`MatchException`** en lugar de `IncompatibleClassChangeError`. El JEP lo califica de «cambio incompatible menor» y precisa que solo ocurre si la clase del enum cambia después de haber compilado el `switch`.

### 8.3. Constantes de enum con nombre cualificado

Antes de Java 21, en un `case` solo se podía usar el **nombre simple** de la constante y el selector debía ser del tipo enum. Ahora:

1. Se permiten **nombres cualificados** (`Palo.OROS`) como etiquetas `case`.
2. El selector **ya no tiene que ser de tipo enum** si las etiquetas son constantes cualificadas de un enum compatible con el tipo del selector. Esto permite mezclar enums con otros tipos mediante una interfaz `sealed`.

Ejemplo ✅ (adaptado del JEP, con nombres en castellano):

```java
sealed interface Clasificacion permits Palo, Tarot {}
enum Palo implements Clasificacion { OROS, COPAS, ESPADAS, BASTOS }
record Tarot(String nombre) implements Clasificacion {}

static String describir(Clasificacion c) {
    return switch (c) {
        case Palo.OROS    -> "Son oros";
        case Palo.COPAS   -> "Son copas";
        case Palo.ESPADAS -> "Son espadas";
        case Palo.BASTOS  -> "Son bastos";
        case Tarot t      -> "Es un tarot: " + t.nombre();
    };
}

describir(Palo.COPAS);               // "Son copas"
describir(new Tarot("El Loco"));     // "Es un tarot: El Loco"
```

Reglas del JEP:

- Cuando el selector **es** del tipo enum, se puede seguir usando el nombre simple (`case OROS`), e incluso cualificarlo aunque sea innecesario.
- Cuando el selector **no es** del tipo enum, el nombre **debe** estar cualificado. Usar `case TAILS` en lugar de `case Coin.TAILS` es un error de compilación.

Sin esta mejora habría que escribir un patrón con guarda por cada constante (`case Palo p when p == Palo.OROS`).

### 8.4. Guardas (`when`) y patrones de tipo con enums

Un patrón de tipo puede aplicarse a un enum, y se le puede añadir una **guarda** con `when` ✅:

```java
static String guardas(Palo p) {
    return switch (p) {
        case OROS -> "oros";
        case Palo q when q.ordinal() > 1 -> "palo alto: " + q;
        case Palo q -> "otro: " + q;
    };
}

guardas(Palo.OROS);     // "oros"
guardas(Palo.BASTOS);   // "palo alto: BASTOS"
guardas(Palo.COPAS);    // "otro: COPAS"
```

**Dominancia:** un patrón que cubre todas las constantes (`case Palo q`) *domina* a las constantes individuales, así que debe colocarse **después** de ellas. El JEP recomienda este orden: constantes primero, luego patrones con guarda, y por último los patrones sin guarda.

### 8.5. `case null`

Tradicionalmente un `switch` lanzaba `NullPointerException` si el selector era `null`. Ahora se puede tratar dentro del propio `switch` ✅:

```java
static String conNull(Nivel n) {
    return switch (n) {
        case null  -> "sin nivel";
        case BAJO  -> "bajo";
        case MEDIO -> "medio";
        case ALTO  -> "alto";
    };
}

conNull(null);          // "sin nivel"
```

Sin `case null`, el comportamiento es el de siempre: `NullPointerException` ✅. Además, el JEP permite combinarlo con el caso por defecto: `case null, default -> ...`. Un `default` por sí solo **no** captura `null`.

### 8.6. Enums que implementan una interfaz `sealed`

Un enum puede implementar una interfaz `sealed`, de modo que el compilador sepa que la lista de casos es completa:

```java
sealed interface Moneda permits Cara {}
enum Cara implements Moneda { CARA, CRUZ }

switch (moneda) {
    case Cara.CARA -> System.out.println("Cara");
    case Cara.CRUZ -> System.out.println("Cruz");
}
```

✅ Compila y funciona en Java 21.

---

## 9. Variables y patrones sin nombre (`_`)

- **Java 21:** primera versión preliminar ([JEP 443](https://openjdk.org/jeps/443)).
- **Java 22:** versión definitiva, sin cambios ([JEP 456](https://openjdk.org/jeps/456)).

Un guion bajo `_` indica que una variable o un patrón **no se va a usar**. Es útil con `switch` y enums cuando varias etiquetas comparten acción. Ejemplo del JEP con una jerarquía `sealed`:

```java
switch (ball) {
    case RedBall _   -> process(ball);
    case BlueBall _  -> process(ball);
    case GreenBall _ -> stopProcessing();
}
```

Además, JEP 456 permite varios patrones en una misma etiqueta `case` siempre que no declaren variables:

```java
case Box(RedBall _), Box(BlueBall _) -> processBox(box);
```

Otros usos de `_` (variables locales, parámetros de `catch`, parámetros de lambdas) ✅ comprobado en Java 21 con `--enable-preview --release 21`:

```java
for (var _ : List.of(1, 2, 3)) n++;
try { Integer.parseInt("x"); } catch (NumberFormatException _) { /* ... */ }
```

> En Java 21 es **solo vista previa**: sin `--enable-preview` el compilador responde `unnamed variables are a preview feature and are disabled by default`. A partir de Java 22 no hace falta ninguna opción.

## 10. Constructores flexibles en enums (Java 25, JEP 513)

[JEP 513](https://openjdk.org/jeps/513) («Flexible Constructor Bodies») es definitivo en **Java 25** (estaba en vista previa desde Java 22). Permite escribir sentencias **antes** de `super(...)` o `this(...)` en un constructor, siempre que no usen el objeto que se está construyendo.

Respecto a los enums, el JEP dice textualmente que sus constructores pueden contener invocaciones a otro constructor (`this(...)`), pero **no** invocaciones al constructor de la superclase, y que se benefician del cambio porque ahora pueden llevar sentencias antes de `this(...)`.

Ejemplo ilustrativo ⚠️ (no compilado; requiere JDK 25):

```java
enum Prioridad {
    NORMAL(1), URGENTE(5);

    private final int peso;
    private final String etiqueta;

    Prioridad(int peso) {
        if (peso < 0) throw new IllegalArgumentException("peso negativo");  // antes de this(...)
        this(peso, "P" + peso);
    }

    Prioridad(int peso, String etiqueta) {
        this.peso = peso;
        this.etiqueta = etiqueta;
    }
}
```

Reglas del JEP para ese código previo: no puede referirse a `this` (salvo asignar campos sin inicializador), puede lanzar excepciones, y **no** puede contener `return`.

## 11. Ya en preparación: tipos primitivos en patrones (Java 25, vista previa)

[JEP 507](https://openjdk.org/jeps/507) («Primitive Types in Patterns, instanceof, and switch») está en su **tercera vista previa** en Java 25. Amplía el *pattern matching* a los tipos primitivos. Es solo vista previa, así que no conviene usarlo todavía en código de producción.

---

## 12. Resumen y buenas prácticas

- Un enum es la forma idiomática de modelar un conjunto cerrado de valores.
- Con Java 21+, prefiere `switch` **expresión sin `default`** para obtener comprobación de exhaustividad en compilación.
- Para combinar enums con otros tipos, usa una interfaz `sealed` y etiquetas cualificadas (`Palo.OROS`).
- Coloca las constantes primero y los patrones genéricos al final para evitar errores de dominancia.
- Usa `case null` solo cuando `null` sea un valor legítimo de entrada.
- `EnumMap` y `EnumSet` son más compactos y rápidos que `HashMap`/`HashSet` para claves de enum.

### Tabla de versiones

| Característica | Versión | Estado |
|---|---|---|
| `switch` con patrones, constantes de enum cualificadas, `case null`, `MatchException` | Java 21 | Definitivo (JEP 441) |
| Variables y patrones sin nombre `_` | 21 (preview) → 22 | Definitivo en 22 (JEP 456) |
| Sentencias antes de `this(...)`/`super(...)` en constructores (enums incluidos) | 22 (preview) → 25 | Definitivo en 25 (JEP 513) |
| Tipos primitivos en patrones | Java 25 | Vista previa (JEP 507) |

> Java 25 es la última versión LTS vigente (septiembre de 2025); la anterior es Java 21 (septiembre de 2023).

## Fuentes

- W3Schools, [Java Enums](https://www.w3schools.com/java/java_enums.asp)
- OpenJDK, [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- OpenJDK, [JEP 456: Unnamed Variables & Patterns](https://openjdk.org/jeps/456)
- OpenJDK, [JEP 513: Flexible Constructor Bodies](https://openjdk.org/jeps/513)
- OpenJDK, [JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview)](https://openjdk.org/jeps/507)
