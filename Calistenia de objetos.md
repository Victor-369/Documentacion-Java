# Calistenia de objetos en Java (actualizado a Java 25)

## ¿Qué es la calistenia de objetos?

La **calistenia de objetos** (*Object Calisthenics*) es un conjunto de **nueve reglas de programación** propuesto por **Jeff Bay** en el libro *The ThoughtWorks Anthology* (2008).

El nombre procede de la calistenia física, es decir, de los ejercicios que se hacen con el propio peso del cuerpo. La idea es la misma: son **ejercicios deliberadamente estrictos** que entrenan nuestra forma de diseñar código orientado a objetos.

El objetivo **no** es cumplir estas reglas al pie de la letra en cualquier proyecto. Lo que se busca es practicarlas, por ejemplo en una kata o en un proyecto pequeño, para coger el hábito de escribir código más **cohesivo, desacoplado, legible y fácil de mantener**.

Muchos desarrolladores que vienen de un estilo procedimental escriben "Java con clases", pero no código de verdad orientado a objetos. Estas reglas obligan a cambiar de mentalidad.

---

## ¿Qué versión de Java usamos?

Todos los ejemplos están escritos para **Java 25**, la versión con soporte a largo plazo (LTS) publicada en septiembre de 2025, y funcionan igual en las versiones posteriores.

Solo se usan características **definitivas** del lenguaje. No hace falta activar nada especial (no se usa `--enable-preview`).

Las reglas tienen casi veinte años, pero el Java moderno nos ayuda mucho a cumplirlas. Estas son las novedades que aparecen en este documento:

| Característica | ¿Qué hace, en pocas palabras? | Definitiva desde |
|----------------|-------------------------------|------------------|
| Expresiones `switch` | Un `switch` que devuelve un valor y no necesita `break` | Java 14 |
| *Records* | Clases de datos inmutables en una sola línea | Java 16 |
| `Stream.toList()` | Convierte un *stream* en una lista inmutable | Java 16 |
| Clases e interfaces selladas (`sealed`) | Indican exactamente qué clases pueden implementar un tipo | Java 17 |
| *Pattern matching* en `switch` | Un `switch` que distingue por tipo de objeto | Java 21 |
| *Record patterns* | "Desmontan" un *record* en sus componentes dentro de un `switch` o `instanceof` | Java 21 |
| Colecciones secuenciadas | Métodos como `getFirst()` y `getLast()` en las listas | Java 21 |
| Variables sin nombre (`_`) | Marcan una variable que no se va a usar | Java 22 |
| *Stream gatherers* | Operaciones intermedias a medida, como agrupar en lotes | Java 24 |
| Cuerpos de constructor flexibles | Permiten validar **antes** de llamar a `super(...)` | Java 25 |
| Ficheros fuente compactos y `IO` | Programas sin `class` ni `public static void main`, e `IO.println()` | Java 25 |
| Importación de módulos | `import module java.base;` importa de golpe todos sus paquetes | Java 25 |

---

## Las nueve reglas

### 1. Un solo nivel de indentación por método

**Qué dice:** dentro de un método solo puede haber un nivel de sangría. Nada de bucles dentro de bucles ni `if` dentro de bucles.

**Por qué:** cada nivel extra es una responsabilidad más. Si separamos cada nivel en su propio método, cada método hace **una sola cosa** y su nombre explica qué hace.

```java
// Antes: dos niveles (un if dentro de un for)
void imprimirAprobados(List<Alumno> alumnos) {
    for (Alumno alumno : alumnos) {
        if (alumno.nota() >= 5) {
            IO.println(alumno.nombre());
        }
    }
}

// Después (opción 1): extraer el nivel interior a un método
void imprimirAprobados(List<Alumno> alumnos) {
    for (Alumno alumno : alumnos) {
        imprimirSiEstaAprobado(alumno);
    }
}

// Después (opción 2): con streams no hay anidamiento
void imprimirAprobados(List<Alumno> alumnos) {
    alumnos.stream()
           .filter(Alumno::estaAprobado)
           .forEach(Alumno::imprimirNombre);
}
```

> **Novedad de Java 25:** `IO.println(...)` es un atajo de `System.out.println(...)`. La clase `IO` está en `java.lang`, así que no hay que importarla.

---

### 2. No utilizar la palabra clave `else`

**Qué dice:** prohibido escribir `else`.

**Por qué:** las cadenas de `if/else` crecen con cada caso nuevo y esconden la lógica principal. Hay tres alternativas:

1. **Retornos tempranos** (*guard clauses*): se sale del método en cuanto se sabe la respuesta.
2. **Polimorfismo**: cada clase sabe comportarse a su manera.
3. **`switch` moderno**: si hay que elegir entre varios casos, un `switch` exhaustivo es más claro que una escalera de `if/else`.

**Retorno temprano:**

```java
// Antes
String clasificar(int edad) {
    if (edad < 18) {
        return "menor";
    } else {
        return "adulto";
    }
}

// Después
String clasificar(int edad) {
    if (edad < 18) {
        return "menor";
    }
    return "adulto";
}
```

**Polimorfismo con tipos sellados y *records*:**

```java
// Antes: el código pregunta "¿qué eres?" y decide desde fuera
double area(Object forma) {
    if (forma instanceof Circulo c) {
        return Math.PI * c.radio() * c.radio();
    } else if (forma instanceof Rectangulo r) {
        return r.base() * r.altura();
    } else {
        throw new IllegalArgumentException("Forma desconocida");
    }
}

// Después: cada forma sabe calcular su propia área
sealed interface Forma permits Circulo, Rectangulo {
    double area();
}

record Circulo(double radio) implements Forma {
    public double area() {
        return Math.PI * radio * radio;
    }
}

record Rectangulo(double base, double altura) implements Forma {
    public double area() {
        return base * altura;
    }
}
```

`sealed ... permits` significa: "solo `Circulo` y `Rectangulo` pueden ser una `Forma`". Nadie más puede añadirse por sorpresa.

**`switch` con patrones**, cuando la operación no encaja dentro de las propias clases (por ejemplo, dibujarlas en pantalla):

```java
String describir(Forma forma) {
    return switch (forma) {
        case Circulo(double radio) -> "Círculo de radio " + radio;
        case Rectangulo(double base, double altura) -> "Rectángulo de " + base + " x " + altura;
    };
}
```

Fíjate en que **no hay `default`**. Como `Forma` es sellada, el compilador sabe que solo existen esos dos casos. Si mañana alguien añade `Triangulo` a la lista de `permits`, el compilador nos avisará de que este `switch` está incompleto. Con `if/else` ese error pasaría desapercibido.

---

### 3. Encapsular todos los tipos primitivos y las cadenas

**Qué dice:** si un `int`, un `BigDecimal` o un `String` tiene significado de negocio (un email, un DNI, un precio…), debe tener su propia clase.

**Por qué:**

- Evita el antipatrón *Primitive Obsession* (obsesión por los primitivos).
- Da un sitio natural a las **validaciones**: si existe un objeto `Email`, sabemos que es válido.
- El compilador nos protege: ya no se puede pasar un nombre donde se esperaba un email, porque ambos eran `String`.

Antes de Java 16 esto suponía escribir mucho código repetitivo (constructor, `equals`, `hashCode`, `toString`…). Con los ***records*** basta con una línea más la validación:

```java
// Antes: una clase completa a mano
public final class Email {
    private final String valor;

    public Email(String valor) {
        if (valor == null || !valor.contains("@")) {
            throw new IllegalArgumentException("Correo electrónico no válido");
        }
        this.valor = valor;
    }

    // ...además de equals(), hashCode() y toString()
}

// Después: un record con constructor compacto
public record Email(String valor) {
    public Email {
        Objects.requireNonNull(valor, "El correo electrónico es obligatorio");
        if (!valor.contains("@")) {
            throw new IllegalArgumentException("Correo electrónico no válido: " + valor);
        }
        valor = valor.strip().toLowerCase();
    }
}
```

El **constructor compacto** (`public Email { ... }`) se ejecuta antes de guardar los valores. Sirve para validar y también para normalizar los datos, como hacemos aquí al quitar espacios y pasar a minúsculas.

Otro ejemplo típico, el dinero. Nunca conviene representarlo con `double`, porque pierde precisión:

```java
public record Dinero(BigDecimal cantidad) {
    public static final Dinero CERO = new Dinero(BigDecimal.ZERO);

    public Dinero {
        if (cantidad.signum() < 0) {
            throw new IllegalArgumentException("El dinero no puede ser negativo");
        }
    }

    public Dinero sumar(Dinero otro) {
        return new Dinero(cantidad.add(otro.cantidad));
    }
}
```

**Novedad de Java 25: validar antes de `super(...)`.** Hasta ahora, cuando una clase heredaba de otra, la llamada a `super(...)` tenía que ser la primera línea del constructor. Eso obligaba a validar *después* de construir el padre o a usar métodos estáticos auxiliares. Ahora se puede validar primero:

```java
abstract class Identificador {
    private final String valor;

    protected Identificador(String valor) {
        this.valor = valor;
    }
}

final class Dni extends Identificador {
    Dni(String valor) {
        if (!valor.matches("\\d{8}[A-Z]")) {
            throw new IllegalArgumentException("DNI no válido: " + valor);
        }
        super(valor); // Desde Java 25 puede ir después de la validación
    }
}
```

---

### 4. Colecciones de primera clase

**Qué dice:** una clase que contiene una colección **no debe tener ninguna otra variable de instancia**. La colección se envuelve en su propia clase.

**Por qué:** las operaciones sobre la colección (calcular el total, filtrar, agrupar…) tienen un hogar claro, en lugar de estar repartidas por todo el código.

```java
public final class Pedidos {
    private final List<Pedido> pedidos;

    public Pedidos(List<Pedido> pedidos) {
        this.pedidos = List.copyOf(pedidos); // Copia inmutable: nadie puede modificarla desde fuera
    }

    public Dinero total() {
        return pedidos.stream()
                .map(Pedido::importe)
                .reduce(Dinero.CERO, Dinero::sumar);
    }

    public Pedidos pendientes() {
        return new Pedidos(pedidos.stream()
                .filter(Pedido::estaPendiente)
                .toList());
    }

    public Pedido masReciente() {
        return pedidos.getLast(); // Java 21: antes era pedidos.get(pedidos.size() - 1)
    }

    public List<Pedidos> enLotesDe(int pedidosPorLote) {
        return pedidos.stream()
                .gather(Gatherers.windowFixed(pedidosPorLote)) // Java 24: agrupa de N en N
                .map(Pedidos::new)
                .toList();
    }
}
```

Novedades de Java moderno en este ejemplo:

- `toList()` (Java 16) devuelve una lista **inmutable**, más corta que `collect(Collectors.toList())`.
- `getFirst()` y `getLast()` (Java 21) hacen el código más expresivo.
- `Gatherers.windowFixed(n)` (Java 24) parte la lista en grupos de `n` elementos sin bucles ni índices.

> **Consejo:** no devuelvas nunca la lista interna tal cual. Si alguien necesita recorrerla, ofrece métodos con significado (`total()`, `pendientes()`…) en lugar de un `getPedidos()`.

---

### 5. Un solo punto por línea

**Qué dice:** evita las cadenas de llamadas como `cliente.direccion().ciudad().nombre()`.

**Por qué:** es la **Ley de Deméter** ("habla solo con tus amigos más cercanos"). Si `Cliente` conoce la estructura interna de `Direccion` y de `Ciudad`, cualquier cambio en ellas rompe el código que usa `Cliente`. Es mejor pedirle al objeto que haga el trabajo.

```java
// Antes: conocemos las tripas de tres objetos
String ciudad = cliente.direccion().ciudad().nombre();

// Después: cada objeto delega en su colaborador directo
record Cliente(NombreCompleto nombre, Direccion direccion) {
    String nombreDeCiudad() {
        return direccion.nombreDeCiudad();
    }
}

record Direccion(Calle calle, Ciudad ciudad) {
    String nombreDeCiudad() {
        return ciudad.nombre();
    }
}

// Uso
String ciudad = cliente.nombreDeCiudad();
```

> **Ojo con los *records*:** generan automáticamente métodos para leer cada componente, lo que facilita volver a caer en las cadenas largas. La regla sigue valiendo.

> **Excepción habitual:** las APIs fluidas (`Stream`, *builders*) no incumplen la regla, porque cada llamada devuelve un objeto de la misma abstracción, no un objeto interno de otro.

---

### 6. No abreviar

**Qué dice:** usa nombres completos. Mejor `calcularTotal()` que `calcTot()`.

**Por qué:** si un nombre es tan largo que nos tienta abreviarlo, probablemente la clase o el método hace demasiadas cosas. La solución no es acortar el nombre, sino **dividir la responsabilidad**.

```java
// Antes
var c = new Cli("Ana", "a@x.es");
int cnt = 0;

// Después
var cliente = new Cliente(new NombreCompleto("Ana", "López"), new Email("ana@ejemplo.es"));
int pedidosPendientes = 0;
```

**Novedad de Java 22: variables sin nombre (`_`).** A veces una variable es obligatoria pero no la usamos, y acabamos inventando nombres como `e`, `ex` o `ignorada`. Ahora podemos decirlo claramente con un guion bajo:

```java
// Antes
try {
    return Optional.of(Integer.parseInt(texto));
} catch (NumberFormatException e) { // "e" no se usa nunca
    return Optional.empty();
}

// Después: el _ deja claro que la variable no se usa a propósito
try {
    return Optional.of(Integer.parseInt(texto));
} catch (NumberFormatException _) {
    return Optional.empty();
}

// También en lambdas
mapa.forEach((_, valor) -> IO.println(valor));
```

---

### 7. Mantener todas las entidades pequeñas

**Qué dice:** como referencia orientativa:

- Clases de **no más de 50 líneas**.
- Paquetes de **no más de 10 ficheros**.

**Por qué:** las clases pequeñas tienden de forma natural a tener una única responsabilidad y se entienden de un vistazo.

El Java moderno hace esta regla mucho más fácil:

- Un **objeto de valor** que antes ocupaba 40 o 50 líneas (campos, constructor, `equals`, `hashCode`, `toString`) ahora es un ***record* de una línea**.
- Los **ficheros fuente compactos** (Java 25) permiten escribir programas pequeños, ideales para katas, sin clase envolvente ni `public static void main(String[] args)`:

```java
// Fichero Hola.java. Se ejecuta con: java Hola.java
void main() {
    IO.println("¡Hola, calistenia!");
}
```

En un fichero compacto, además, **todo el módulo `java.base` se importa automáticamente** (`List`, `Map`, `BigDecimal`…), así que no hacen falta líneas de `import`. En ficheros normales se consigue lo mismo con `import module java.base;`.

---

### 8. No más de dos variables de instancia por clase

**Qué dice:** cada clase puede tener como máximo dos campos.

**Por qué:** es la regla más polémica, pero obliga a agrupar los datos que van juntos en objetos más pequeños y cohesivos. Con *records* anidados queda muy natural:

```java
// Antes: cinco campos sueltos
public class Persona {
    private String nombre;
    private String apellidos;
    private String calle;
    private String ciudad;
    private String codigoPostal;
}

// Después: cada record tiene como máximo dos componentes
record NombreCompleto(String nombre, String apellidos) {}
record Localidad(Ciudad ciudad, CodigoPostal codigoPostal) {}
record Direccion(Calle calle, Localidad localidad) {}
record Persona(NombreCompleto nombre, Direccion direccion) {}
```

Al agrupar, aparecen conceptos que antes estaban escondidos, como `Localidad` o `NombreCompleto`. Además, cada uno se puede reutilizar y validar por separado.

> **¿Y los `String` de `NombreCompleto`?** Que sigan existiendo no incumple la regla 3: los primitivos acaban siempre **dentro** de un objeto de valor, que es justo su sitio. Lo que la regla evita es que anden sueltos por el resto del código.

---

### 9. Sin *getters*, *setters* ni propiedades públicas

**Qué dice:** no expongas el estado interno de los objetos.

**Por qué:** aplica el principio **"Tell, don't ask"** ("di, no preguntes"). En lugar de leer el estado de un objeto para decidir desde fuera, se le dice qué tiene que hacer. Así la lógica vive junto a los datos que necesita.

```java
// Antes: la lógica de negocio queda fuera del objeto
if (cuenta.getSaldo().compareTo(importe) >= 0) {
    cuenta.setSaldo(cuenta.getSaldo().subtract(importe));
}

// Después: el objeto protege sus propias reglas
public final class Cuenta {
    private Dinero saldo;

    public Cuenta(Dinero saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void retirar(Dinero importe) {
        if (saldo.esMenorQue(importe)) {
            throw new IllegalStateException("Saldo insuficiente");
        }
        saldo = saldo.restar(importe);
    }
}

// Uso
cuenta.retirar(importe);
```

#### ¿Y los *records*? ¿No son *getters* encubiertos?

En parte, sí. Un *record* genera un método de lectura por cada componente (`email.valor()`, `dinero.cantidad()`). Para conciliarlo con esta regla, una pauta práctica:

| Situación | ¿Qué usar? |
|-----------|-----------|
| **Objetos de valor** inmutables (`Email`, `Dinero`, `Dni`) | *Records*. Son inmutables, **no tienen *setters*** y leer un valor no rompe nada. |
| **Entidades** con comportamiento y estado que cambia (`Cuenta`, `Pedido`, `Carrito`) | Clases normales con campos `private` y métodos con significado de negocio (`retirar`, `confirmar`, `añadir`). |
| Datos que entran o salen del sistema (DTO, JSON, respuestas de una API) | *Records*. Aquí la calistenia no aplica: su trabajo es precisamente transportar datos. |

La idea clave: **leer** un valor de un objeto inmutable es aceptable; **decidir fuera** del objeto lo que el objeto debería decidir, no.

---

## Ejemplo completo: un carrito de la compra

Este pequeño programa reúne casi todas las reglas. Es un **fichero fuente compacto** de Java 25: se guarda como `Carrito.java` y se ejecuta directamente con `java Carrito.java`, sin compilar antes ni crear proyecto.

```java
// Fichero Carrito.java. Se ejecuta con: java Carrito.java

// Regla 3: el dinero tiene su propio tipo, con su validación
record Dinero(BigDecimal cantidad) {
    static final Dinero CERO = new Dinero(BigDecimal.ZERO);

    Dinero {
        if (cantidad.signum() < 0) {
            throw new IllegalArgumentException("El dinero no puede ser negativo");
        }
    }

    static Dinero euros(String cantidad) {
        return new Dinero(new BigDecimal(cantidad));
    }

    Dinero sumar(Dinero otro) {
        return new Dinero(cantidad.add(otro.cantidad));
    }

    Dinero restar(Dinero otro) {
        return new Dinero(cantidad.subtract(otro.cantidad));
    }

    Dinero porcentaje(int porcentaje) {
        var resultado = cantidad.multiply(BigDecimal.valueOf(porcentaje))
                                .divide(BigDecimal.valueOf(100), 2, RoundingMode.HALF_UP);
        return new Dinero(resultado);
    }

    @Override
    public String toString() {
        return cantidad.setScale(2, RoundingMode.HALF_UP) + " €";
    }
}

// Regla 8: como máximo dos componentes
// (por brevedad, el nombre se deja como String)
record Producto(String nombre, Dinero precio) {}

// Regla 2: polimorfismo en lugar de if/else para los descuentos
sealed interface Descuento permits SinDescuento, DescuentoPorcentual {
    Dinero aplicarA(Dinero importe);
}

record SinDescuento() implements Descuento {
    public Dinero aplicarA(Dinero importe) {
        return importe;
    }
}

record DescuentoPorcentual(int porcentaje) implements Descuento {
    DescuentoPorcentual {
        if (porcentaje < 0 || porcentaje > 100) {
            throw new IllegalArgumentException("Porcentaje no válido: " + porcentaje);
        }
    }

    public Dinero aplicarA(Dinero importe) {
        return importe.restar(importe.porcentaje(porcentaje));
    }
}

// Regla 4: colección de primera clase con un único campo
// Regla 9: no hay getters ni setters, solo comportamiento
final class CarritoDeLaCompra {
    private final List<Producto> productos = new ArrayList<>();

    void añadir(Producto producto) {
        productos.add(producto);
    }

    // Regla 1: un único nivel de indentación
    Dinero total(Descuento descuento) {
        var subtotal = productos.stream()
                                .map(Producto::precio)
                                .reduce(Dinero.CERO, Dinero::sumar);
        return descuento.aplicarA(subtotal);
    }
}

// Java 25: main sin clase, sin static y sin parámetros
void main() {
    var carrito = new CarritoDeLaCompra();
    carrito.añadir(new Producto("Teclado", Dinero.euros("49.90")));
    carrito.añadir(new Producto("Ratón", Dinero.euros("19.95")));

    IO.println("Total sin descuento: " + carrito.total(new SinDescuento()));
    IO.println("Total con un 10 % de descuento: " + carrito.total(new DescuentoPorcentual(10)));
}
```

Salida del programa:

```text
Total sin descuento: 69.85 €
Total con un 10 % de descuento: 62.86 €
```

> En un fichero compacto no hacen falta `import`: `List`, `ArrayList`, `BigDecimal` y `RoundingMode` llegan gracias a la importación automática de `java.base`.

---

## Resumen de las reglas

| Nº | Regla | Principio que refuerza | Ayuda del Java moderno |
|----|-------|------------------------|------------------------|
| 1 | Un solo nivel de indentación por método | Responsabilidad única, legibilidad | *Streams*, *gatherers* |
| 2 | No utilizar `else` | Polimorfismo, flujo sencillo | `sealed`, `switch` con patrones |
| 3 | Encapsular primitivos y cadenas | Objetos de valor, validación | *Records*, constructores flexibles |
| 4 | Colecciones de primera clase | Cohesión, encapsulamiento | `toList()`, `getLast()`, *gatherers* |
| 5 | Un solo punto por línea | Ley de Deméter, bajo acoplamiento | — |
| 6 | No abreviar | Nombres expresivos, clases enfocadas | Variables sin nombre (`_`) |
| 7 | Entidades pequeñas | Responsabilidad única | *Records*, ficheros compactos |
| 8 | Máximo dos variables de instancia | Alta cohesión, descomposición | *Records* anidados |
| 9 | Sin *getters* ni *setters* | *Tell, don't ask*, encapsulamiento | *Records* inmutables (con matices) |

---

## Principios que refuerza

- **Encapsulamiento**: el estado y el comportamiento conviven en el mismo objeto.
- **Principio de responsabilidad única (SRP)** de SOLID.
- **Ley de Deméter** y **"Tell, don't ask"**, que reducen el acoplamiento.
- **Objetos de valor** (*Value Objects*), un concepto central del Diseño Guiado por el Dominio (DDD).
- **Polimorfismo** en lugar de condicionales.
- **Inmutabilidad**: los *records* y las listas inmutables evitan cambios inesperados.

---

## Cómo aplicarla en la práctica

### Como ejercicio

Lo más habitual es emplearla en **katas de código**, por ejemplo:

- El Juego de la Vida de Conway (*Game of Life*).
- La kata de los bolos (*Bowling Kata*).
- La kata del tenis (*Tennis Kata*).
- La kata del carrito de la compra (*Checkout Kata*).

El planteamiento consiste en resolver la kata aplicando las nueve reglas con rigor y, después, comparar el resultado con nuestro estilo habitual.

Con Java 25 empezar una kata es inmediato: basta con un fichero compacto con `void main()` y ejecutarlo con `java Kata.java`.

### En código de producción

En proyectos reales conviene tomarla como **orientación y no como dogma**. Algunas reglas chocan con herramientas muy extendidas en el ecosistema Java:

- **JPA/Hibernate** exige que las entidades tengan un constructor sin argumentos y no permite que sean *records*.
- **Jackson** y otras librerías de serialización se apoyaban tradicionalmente en *getters* y *setters*, aunque hoy admiten *records* sin problema.
- **Spring** y otros *frameworks* pueden imponer convenciones propias.

Por este motivo, muchos equipos aplican estas reglas sobre todo en la **capa de dominio** y mantienen más flexibles las capas de infraestructura (persistencia, controladores, DTO). Los *records* encajan muy bien en ambos lados: como objetos de valor en el dominio y como DTO en los bordes de la aplicación.

---

## Referencias

### Libros

- Bay, Jeff. "Object Calisthenics". En *The ThoughtWorks Anthology: Essays on Software Technology and Innovation*. The Pragmatic Bookshelf, 2008.
- Fowler, Martin. *Refactoring: Improving the Design of Existing Code*. Addison-Wesley, 2.ª ed., 2018.
- Evans, Eric. *Domain-Driven Design: Tackling Complexity in the Heart of Software*. Addison-Wesley, 2003.

### Propuestas de mejora de Java (JEP) citadas

- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361) (Java 14)
- [JEP 395: Records](https://openjdk.org/jeps/395) (Java 16)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409) (Java 17)
- [JEP 431: Sequenced Collections](https://openjdk.org/jeps/431) (Java 21)
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440) (Java 21)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) (Java 21)
- [JEP 456: Unnamed Variables & Patterns](https://openjdk.org/jeps/456) (Java 22)
- [JEP 485: Stream Gatherers](https://openjdk.org/jeps/485) (Java 24)
- [JEP 511: Module Import Declarations](https://openjdk.org/jeps/511) (Java 25)
- [JEP 512: Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512) (Java 25)
- [JEP 513: Flexible Constructor Bodies](https://openjdk.org/jeps/513) (Java 25)
