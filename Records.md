# Records en Java

Guía basada en el artículo de KeepCoding «¿Qué son los records en Java y cómo te ayudan a manipular datos?», ampliada y contrastada con la especificación oficial (JEP 395) y con las novedades desde Java 21.

**Convenciones de este documento**

- ✅ = ejemplo compilado y ejecutado con OpenJDK 21.
- ⚠️ = requiere una versión posterior a 21 y **no se ha podido compilar en local**; se toma de la especificación oficial (JEP).
- Las secciones 1 a 5 recogen el contenido del artículo. A partir de la 6 son ampliaciones.

---

## 1. Qué son los records

Un *record* es una clase diseñada para **reducir el código repetitivo** (*boilerplate*) al definir clases simples que solo transportan datos. Está pensado para clases **inmutables**: una vez creado el objeto, sus valores no cambian.

Cuando se trabaja con datos, sobre todo en aplicaciones con grandes volúmenes, hay que crear clases que representen esas entidades sin escribir a mano constructores, `getters`, `equals()`, `hashCode()` y `toString()`. Con un record, Java genera automáticamente todo ese código y te deja centrarte en la manipulación de los datos.

> **Precisión sobre la versión.** El artículo dice que los records llegaron «junto a Java 14». Según el JEP 395 oficial, en Java 14 fueron una **vista previa** (JEP 359), en Java 15 una segunda vista previa (JEP 384) y se hicieron **definitivos en Java 16**. Por tanto, en cualquier JDK moderno (incluido Java 21 o superior) están disponibles sin ninguna opción especial.

## 2. Declaración y uso básico

```java
public record Persona(String nombre, String apellidos, int edad) {}
```

El record `Persona` tiene tres campos (*componentes*): `nombre`, `apellidos` y `edad`. El compilador genera automáticamente:

- el **constructor** (llamado *constructor canónico*),
- los métodos **de acceso** (sin prefijo `get`: se llaman igual que el campo),
- `toString()`, `equals()` y `hashCode()`.

```java
Persona persona = new Persona("Juan", "Gómez", 25);
System.out.println(persona.nombre());   // Juan  ✅
System.out.println(persona.edad());     // 25    ✅
System.out.println(persona);            // Persona[nombre=Juan, apellidos=Gómez, edad=25]  ✅
```

`equals()` compara por **valor** ✅:

```java
persona.equals(new Persona("Juan", "Gómez", 25));   // true
```

Otro ejemplo del artículo:

```java
public record Producto(String nombre, double precio, String categoria) {}

Producto producto = new Producto("Laptop", 999.99, "Electrónica");
System.out.println(producto.nombre());   // Laptop
System.out.println(producto.precio());   // 999.99
```

> El artículo usa «Laptop»; en España se suele decir «portátil» u «ordenador portátil». El nombre es solo un valor de ejemplo.

## 3. Añadir lógica a un record

Los records son concisos, pero admiten métodos, validaciones y constructores personalizados.

### 3.1. Constructor compacto (validación)

```java
public record Producto(String nombre, double precio, String categoria) {
    public Producto {
        if (precio < 0) {
            throw new IllegalArgumentException("El precio no puede ser negativo");
        }
    }
}
```

✅ Al intentar crear `new Producto("Portátil", -1, "Electrónica")` se lanza `IllegalArgumentException: El precio no puede ser negativo`.

En un **constructor compacto** (sin lista de parámetros):

- Los parámetros se declaran implícitamente.
- No se pueden asignar los campos (`this.x = x`) dentro del cuerpo: se asignan **automáticamente al final** con los valores de los parámetros.
- Sirve para **validar** y también para **normalizar** los datos (el JEP 395 lo muestra con ambos usos).

Normalización ✅:

```java
record Usuario(String email) {
    Usuario {
        email = email.strip().toLowerCase();   // se reasigna el parámetro, no el campo
    }
}

new Usuario("  Ana@Mail.COM ");   // Usuario[email=ana@mail.com]
```

### 3.2. Métodos, constructores adicionales y miembros estáticos ✅

```java
record Rango(int min, int max) {
    Rango {
        if (min > max) throw new IllegalArgumentException(String.format("(%d,%d)", min, max));
    }
    Rango(int unico) { this(unico, unico); }          // constructor no canónico: debe delegar con this(...)
    static Rango vacio() { return new Rango(0, 0); }  // método estático de fábrica
    int longitud() { return max - min; }              // método de instancia
}
```

### 3.3. Implementar interfaces ✅

```java
interface Saludable { String saludo(); }

record Amigo(String nombre) implements Saludable {
    public String saludo() { return "Hola, " + nombre; }
}
```

## 4. ¿Cuándo usar records?

Cuando necesites clases de datos que **solo almacenan valores**, sin lógica compleja ni mutabilidad. El artículo cita:

- *Data Transfer Objects* (DTO),
- entidades de bases de datos,
- cualquier estructura que simplemente agrupe valores.

Ejemplo del artículo (comercio electrónico): en lugar de una clase con campos, constructor, *getters* y algún método, basta con:

```java
public record Producto(String nombre, double precio, String categoria) {}
```

Esto ahorra tiempo y reduce la probabilidad de errores.

> **Matiz sobre «entidades de bases de datos».** Un record es siempre `final` y todos sus campos son `final`. Eso puede chocar con los requisitos de algunos frameworks de persistencia. Comprueba la documentación de tu framework antes de usar un record como entidad; como DTO o proyección de consultas no suele haber problema.

## 5. Ventajas (resumen del artículo)

- Menos código repetitivo y menos errores.
- Inmutabilidad por defecto.
- Código más legible: la cabecera describe el estado de un vistazo.
- Permite centrarse en el tratamiento de los datos.

---

## 6. Ampliación: reglas y características de los records (JEP 395)

El [JEP 395](https://openjdk.org/jeps/395) define los records como «clases que actúan como transportadores transparentes de datos inmutables» (se pueden ver como *tuplas nominales*). Su objetivo declarado no es solo reducir código, sino **modelar datos como datos**: si la semántica es correcta, el código repetitivo «se cuida solo».

### 6.1. Qué restricciones tienen respecto a una clase normal

Comprobado con el compilador de Java 21 ✅:

| Intento | Resultado del compilador |
|---|---|
| Declarar un campo de instancia adicional (`int extra;`) | `error: field declaration must be static` |
| Hacer que un record extienda otra clase (`extends B`) | Error de sintaxis: un record no puede llevar `extends` |
| Asignar un campo en un método (`this.x = v;`) | `error: cannot assign a value to final variable x` |

Sí se permiten **campos y métodos estáticos** ✅:

```java
record A(int x) {
    static int contador = 0;
    static A of() { contador++; return new A(1); }
}
```

Y un record puede implementar interfaces, ser genérico, tener anotaciones y declarar sus propios métodos.

### 6.2. Estado descrito por la cabecera

La cabecera lista los *componentes*, que son las variables que forman el estado. Se pierde la libertad de desacoplar la API de la representación interna a cambio de mucha concisión. El JEP lo compara con esta clase equivalente:

```java
class Punto {
    private final int x;
    private final int y;
    Punto(int x, int y) { this.x = x; this.y = y; }
    int x() { return x; }
    int y() { return y; }
    // + equals, hashCode, toString
}
```

que se reduce a `record Punto(int x, int y) {}`.

### 6.3. Buenas prácticas con `equals` y los métodos de acceso

El JEP establece que, si copias un record con sus propios accesores (`new R(r1.c1(), r1.c2(), ...)`), entonces `r1.equals(r2)` debe ser `true`. Los accesores explícitos deben respetar esta regla. Un ejemplo de **mal estilo** (JEP): accesores que «ajustan en silencio» el valor devuelto.

```java
record SmallPoint(int x, int y) {
    public int x() { return this.x < 100 ? this.x : 100; }   // ¡No hagas esto!
    public int y() { return this.y < 100 ? this.y : 100; }
}
```

Para normalizar, usa el constructor compacto (sección 3.1), no los accesores.

### 6.4. Inmutabilidad *superficial*: copia defensiva ✅

Un record es inmutable solo en sus referencias: si un componente es una colección mutable, su contenido podría cambiar desde fuera. Una solución habitual es copiarla en el constructor compacto:

```java
record Pedido(String id, List<String> lineas) {
    Pedido {
        lineas = List.copyOf(lineas);
    }
}

List<String> origen = new ArrayList<>(List.of("a", "b"));
Pedido ped = new Pedido("1", origen);
origen.add("c");
System.out.println(ped);   // Pedido[id=1, lineas=[a, b]]   ← no se ve afectado
```

### 6.5. Records locales ✅

Se pueden declarar dentro de un método para agrupar valores intermedios. Son implícitamente `static`, por lo que no capturan variables del método que los contiene:

```java
record VentaMes(String comercial, double importe) {}
var ventas = List.of(new VentaMes("A", 10), new VentaMes("B", 30));
ventas.stream().max(Comparator.comparingDouble(VentaMes::importe)).get();
// VentaMes[comercial=B, importe=30.0]
```

El mismo JEP añadió enums locales e interfaces locales, también implícitamente estáticos.

### 6.6. Anotaciones sobre componentes

Al anotar un componente, la anotación se propaga a **todos los elementos a los que es aplicable** (campo, parámetro del constructor, método de acceso, componente del record), según su `@Target`. Si declaras explícitamente un accesor público o un constructor canónico no compacto, solo tendrá las anotaciones que lleve directamente.

### 6.7. Reflexión ✅

```java
Persona.class.isRecord();                 // true
Persona.class.getRecordComponents();      // [String nombre, String apellidos, int edad]
```

Todos los records heredan de `java.lang.Record`, que se importa de forma implícita. Si tu aplicación tiene su propia clase `Record` importada con comodín, puede haber ambigüedad; se resuelve importando el nombre completo.

### 6.8. Records + interfaces `sealed` (tipos de datos algebraicos)

Los records modelan «productos» y las interfaces `sealed` modelan «sumas». Juntos permiten describir jerarquías cerradas ✅:

```java
sealed interface Expr permits Const, Suma, Mult, Neg {}
record Const(int i) implements Expr {}
record Suma(Expr a, Expr b) implements Expr {}
record Mult(Expr a, Expr b) implements Expr {}
record Neg(Expr e) implements Expr {}
```

---

## 7. Novedades desde Java 21: patrones de record (JEP 440)

**Java 21** convierte en definitivos los *record patterns* ([JEP 440](https://openjdk.org/jeps/440)). Permiten **deconstruir** un record directamente en sus componentes.

### 7.1. Con `instanceof` ✅

```java
record Punto(int x, int y) {}

static void imprimirSuma(Object obj) {
    if (obj instanceof Punto(int x, int y)) {
        System.out.println(x + y);
    }
}
```

`Punto(int x, int y)` es un *patrón de record*: comprueba el tipo y extrae los componentes llamando por ti a los accesores. Los nombres de las variables no tienen que coincidir con los de los componentes. Con `var`, el compilador infiere el tipo: `Punto(var a, var b)`.

El valor `null` **no** encaja con ningún patrón de record ✅ (`nulo instanceof Punto(int x, int y)` es `false`).

### 7.2. Patrones anidados ✅

```java
enum Color { ROJO, VERDE, AZUL }
record PuntoColor(Punto p, Color c) {}
record Rectangulo(PuntoColor superiorIzq, PuntoColor inferiorDer) {}

if (r instanceof Rectangulo(PuntoColor(Punto(var x, var y), var c), var inf)) {
    System.out.println("x=" + x + " y=" + y + " c=" + c);   // x=3 y=4 c=VERDE
}
```

El patrón refleja la estructura del constructor anidado. Si algún subpatrón falla, falla todo el patrón, de modo que se centraliza el tratamiento del «no encaja» ✅:

```java
record Par(Object x, Object y) {}
Par par = new Par(42, 42);
if (par instanceof Par(String s, String t)) { /* ... */ }
else { System.out.println("No es un par de cadenas"); }
```

### 7.3. Con `switch` y jerarquías `sealed` ✅

```java
static int eval(Expr e) {
    return switch (e) {
        case Const(int i)          -> i;
        case Suma(Expr a, Expr b)  -> eval(a) + eval(b);
        case Mult(Expr a, Expr b)  -> eval(a) * eval(b);
        case Neg(Expr x)           -> -eval(x);
    };
}

eval(new Suma(new Const(2), new Mult(new Const(3), new Neg(new Const(4)))));   // -10
```

Como `Expr` es `sealed`, el compilador sabe que los cuatro casos cubren todas las posibilidades y **no hace falta `default`**.

### 7.4. Inferencia de tipos con records genéricos ✅

Si el patrón nombra un record genérico sin argumentos de tipo, estos se **infieren**:

```java
record Caja<T>(T contenido) {}

Caja<Caja<String>> cc = new Caja<>(new Caja<>("hola"));
if (cc instanceof Caja(Caja(var s))) {
    System.out.println("String " + s);   // String hola
}
```

### 7.5. Exhaustividad con patrones anidados ✅

La comprobación de exhaustividad considera los tipos de los subpatrones y las jerarquías `sealed` (ejemplo adaptado del JEP):

```java
sealed interface I permits C, D {}
record C() implements I {}
record D() implements I {}
record Pair<T>(T x, T y) {}

static String exhaustivo(Pair<I> p) {
    return switch (p) {
        case Pair<I>(I i, C c) -> "segundo es C";
        case Pair<I>(I i, D d) -> "segundo es D";
    };
}
```

Si faltase una combinación posible, el compilador lo rechazaría.

### 7.6. Qué pasa si un accesor lanza una excepción ✅

Si, durante el emparejamiento, un accesor del record falla, el patrón termina con una **`MatchException`**:

```java
record R(int i) {
    public int i() { return i / 0; }   // accesor legal pero «malo»
}

switch (obj) { case R(var i) -> ...; default -> {} }
// MatchException (causa: java.lang.ArithmeticException: / by zero)
```

### 7.7. Qué se quitó

Respecto a la vista previa de Java 20, el JEP 440 **eliminó** la posibilidad de usar patrones de record en la cabecera de un `for` mejorado. Podría reproponerse en el futuro.

---

## 8. Patrones y variables sin nombre (`_`): Java 22

- Vista previa en **Java 21** ([JEP 443](https://openjdk.org/jeps/443)).
- Definitivo en **Java 22** ([JEP 456](https://openjdk.org/jeps/456)).

Con records permite ignorar componentes que no necesitas ✅ (probado en Java 21 con `--enable-preview --release 21`):

```java
if (o instanceof PuntoColor(Punto(int x, _), _)) {
    System.out.println("x=" + x);   // x=3
}
```

- `_` como **patrón sin nombre** coincide con cualquier cosa y no declara variable. Solo puede usarse **anidado** dentro de un patrón de record, no como patrón de nivel superior.
- `_` como **variable sin nombre** (`var _`, `catch (X _)`, parámetros de lambda) indica «no se usará».
- Equivale al patrón `var _`, pero más ligero visualmente.

Sin `--enable-preview`, Java 21 rechaza este código con `unnamed variables are a preview feature and are disabled by default`.

---

## 9. Constructores flexibles y records (Java 25, JEP 513)

El [JEP 513](https://openjdk.org/jeps/513) es definitivo en **Java 25** (estuvo en vista previa en 22, 23 y 24). Permite sentencias **antes** de `this(...)`/`super(...)` en un constructor, siempre que no usen el objeto en construcción.

Para los records, el JEP indica que se **mantienen** estas restricciones:

- Un constructor **canónico** no puede contener una invocación explícita a otro constructor.
- Un constructor **no canónico** debe contener una invocación `this(...)` (y no `super(...)`).

Lo nuevo es que los constructores no canónicos **pueden llevar sentencias antes de `this(...)`** ⚠️ (no compilado; requiere JDK 25):

```java
record Rango(int min, int max) {
    Rango(int unico) {
        if (unico < 0) throw new IllegalArgumentException("valor negativo");  // antes de this(...)
        this(unico, unico);
    }
}
```

---

## 10. En preparación: tipos primitivos en patrones (Java 25, vista previa)

El [JEP 507](https://openjdk.org/jeps/507) está en su **tercera vista previa** en Java 25 y extiende los patrones a todos los tipos primitivos (en `instanceof`, `switch` y patrones). Al ser solo vista previa, evita usarlo en producción.

---

## 11. Resumen y buenas prácticas

- Usa records para **datos inmutables**: DTO, resultados de consultas, valores compuestos, claves de mapas.
- Valida y normaliza en el **constructor compacto**; no modifiques el significado de los accesores.
- Si un componente es una colección, haz una **copia defensiva** (`List.copyOf`).
- Combina records con interfaces `sealed` y `switch` con patrones para modelar y procesar jerarquías cerradas **sin `default`**.
- Con Java 22+, usa `_` para ignorar componentes que no necesites.
- No uses un record cuando necesites **mutabilidad** o herencia de clases.

### Tabla de versiones

| Característica | Versión | Estado |
|---|---|---|
| Records | 14 y 15 (preview) → 16 | Definitivo en 16 (JEP 395) |
| Patrones de record (`instanceof`, `switch`, anidados, inferencia genérica) | Java 21 | Definitivo (JEP 440) |
| `switch` con patrones | Java 21 | Definitivo (JEP 441) |
| Patrones y variables sin nombre `_` | 21 (preview) → 22 | Definitivo en 22 (JEP 456) |
| Sentencias antes de `this(...)` en constructores no canónicos | 22 (preview) → 25 | Definitivo en 25 (JEP 513) |
| Tipos primitivos en patrones | Java 25 | Vista previa (JEP 507) |

> Java 25 es la última versión LTS vigente (septiembre de 2025); la anterior es Java 21 (septiembre de 2023).

## Fuentes

- KeepCoding, [¿Qué son los records en Java y cómo te ayudan a manipular datos?](https://keepcoding.io/blog/que-son-los-records-en-java/)
- OpenJDK, [JEP 395: Records](https://openjdk.org/jeps/395)
- OpenJDK, [JEP 440: Record Patterns](https://openjdk.org/jeps/440)
- OpenJDK, [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- OpenJDK, [JEP 456: Unnamed Variables & Patterns](https://openjdk.org/jeps/456)
- OpenJDK, [JEP 513: Flexible Constructor Bodies](https://openjdk.org/jeps/513)
- OpenJDK, [JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview)](https://openjdk.org/jeps/507)
