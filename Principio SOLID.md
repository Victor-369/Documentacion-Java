# Principios SOLID

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).

## Índice

1. [¿Qué es SOLID?](#1-qué-es-solid)
2. [Origen](#2-origen)
3. [S — Principio de responsabilidad única (SRP)](#3-s--principio-de-responsabilidad-única-srp)
4. [O — Principio abierto/cerrado (OCP)](#4-o--principio-abiertocerrado-ocp)
5. [L — Principio de sustitución de Liskov (LSP)](#5-l--principio-de-sustitución-de-liskov-lsp)
6. [I — Principio de segregación de interfaces (ISP)](#6-i--principio-de-segregación-de-interfaces-isp)
7. [D — Principio de inversión de dependencias (DIP)](#7-d--principio-de-inversión-de-dependencias-dip)
8. [Cómo encajan los cinco principios](#8-cómo-encajan-los-cinco-principios)
9. [SOLID y Java moderno](#9-solid-y-java-moderno)
10. [Errores frecuentes al aplicar SOLID](#10-errores-frecuentes-al-aplicar-solid)
11. [Tabla resumen](#11-tabla-resumen)
12. [Lista de comprobación](#12-lista-de-comprobación)
13. [Referencias](#13-referencias)

---

## 1. ¿Qué es SOLID?

**SOLID** es un acrónimo que agrupa **cinco principios de diseño orientado a objetos**. Su objetivo es crear software **más fácil de mantener, de entender, de ampliar y de probar**, de modo que, a medida que la aplicación crece, su complejidad no se dispare.

| Letra | Principio | En inglés |
|---|---|---|
| **S** | Responsabilidad única | *Single Responsibility Principle* |
| **O** | Abierto/cerrado | *Open/Closed Principle* |
| **L** | Sustitución de Liskov | *Liskov Substitution Principle* |
| **I** | Segregación de interfaces | *Interface Segregation Principle* |
| **D** | Inversión de dependencias | *Dependency Inversion Principle* |

Aunque se explican pensando en **clases**, las ideas se aplican igualmente a **métodos**, **paquetes** y **módulos**.

Algunos principios se parecen entre sí, pero **no persiguen el mismo objetivo**: es posible cumplir uno e incumplir otro. Por eso conviene entender la **meta** de cada uno.

## 2. Origen

Los principios fueron reunidos por **Robert C. Martin** en su artículo *Design Principles and Design Patterns* (2000). Poco después, **Michael Feathers** propuso el acrónimo **SOLID** para recordarlos. Algunos principios son anteriores:

- El principio **abierto/cerrado** fue formulado por **Bertrand Meyer** en *Object-Oriented Software Construction* (1988).
- El principio de **sustitución** lleva el nombre de **Barbara Liskov**, que lo presentó en 1987.

---

## 3. S — Principio de responsabilidad única (SRP)

> **Una clase debe tener una única responsabilidad o, dicho de otro modo, un único motivo para cambiar.**

Si una clase tiene muchas responsabilidades, cambiar una de ellas puede afectar a las demás sin que nos demos cuenta.

**Objetivo:** separar comportamientos para que, si un cambio introduce un error, no afecte a otros comportamientos que no tienen nada que ver.

**Ventajas:**

- **Pruebas:** una clase con una sola responsabilidad necesita muchos menos casos de prueba.
- **Menor acoplamiento:** menos funcionalidad en una clase implica menos dependencias.
- **Organización:** clases pequeñas y bien organizadas son más fáciles de encontrar que clases monolíticas.

### ❌ Incumplimiento

```java
public class Libro {
    private final String titulo;
    private final String autor;
    private final String texto;

    public Libro(String titulo, String autor, String texto) {
        this.titulo = titulo;
        this.autor = autor;
        this.texto = texto;
    }

    // Relacionado con el libro: correcto
    public String reemplazarPalabra(String palabra, String reemplazo) {
        return texto.replace(palabra, reemplazo);
    }

    public boolean contienePalabra(String palabra) {
        return texto.contains(palabra);
    }

    // ❌ Responsabilidad distinta: presentar el libro en un medio concreto
    public void imprimirEnConsola() {
        System.out.println(titulo + " — " + autor);
        System.out.println(texto);
    }
}
```

Si mañana hay que imprimir en PDF, enviar por correo o cambiar el formato, **habrá que modificar `Libro`**, aunque los datos del libro no hayan cambiado.

### ✅ Solución

```java
public record Libro(String titulo, String autor, String texto) {

    public String reemplazarPalabra(String palabra, String reemplazo) {
        return texto.replace(palabra, reemplazo);
    }

    public boolean contienePalabra(String palabra) {
        return texto.contains(palabra);
    }
}
```

```java
public class ImpresoraLibros {

    public void imprimirEnConsola(Libro libro) {
        IO.println(libro.titulo() + " — " + libro.autor());
        IO.println(libro.texto());
    }
}
```

`Libro` ya no sabe nada de cómo se presenta. `ImpresoraLibros` se encarga solo de eso y puede crecer (más medios de salida) sin tocar el libro.

> **Responsabilidad no es lo mismo que acción.** «Ser el pintor» es una responsabilidad; «pintar paredes» o «pintar puertas» son acciones que la cumplen. Una clase puede tener varios métodos siempre que todos sirvan a la misma responsabilidad.
>
> Robert C. Martin aclaró más tarde que «un motivo para cambiar» equivale a **un único actor o grupo de interesados**: si el departamento de contabilidad y el de recursos humanos pueden pedir cambios en la misma clase por motivos distintos, esa clase tiene dos responsabilidades.

---

## 4. O — Principio abierto/cerrado (OCP)

> **Las clases deben estar abiertas a la extensión, pero cerradas a la modificación.**

Cambiar el comportamiento actual de una clase afecta a todos los sistemas que la usan. Si queremos que haga más cosas, lo ideal es **añadir** comportamiento, no **modificar** el existente. La excepción evidente es **corregir errores**.

**Objetivo:** ampliar el comportamiento de una clase sin alterar el que ya tiene, para no provocar errores allí donde se usa.

### Extender mediante herencia

La forma más básica es heredar:

```java
public class Guitarra {
    private final String marca;
    private final String modelo;
    private int volumen;
    // constructor, métodos...
}

public class GuitarraConLlamas extends Guitarra {
    private final String colorLlamas;
    // constructor, métodos...
}
```

Así no se toca `Guitarra` y el resto de la aplicación no se ve afectado. Sin embargo, la herencia acopla mucho la subclase a la superclase, por lo que hoy se prefiere **extender mediante abstracciones** (interfaces) y composición.

### ❌ Incumplimiento típico

```java
public class CalculadoraDescuentos {

    public BigDecimal calcular(Cliente cliente, BigDecimal importe) {
        if (cliente.tipo().equals("VIP")) {
            return importe.multiply(new BigDecimal("0.20"));
        } else if (cliente.tipo().equals("EMPLEADO")) {
            return importe.multiply(new BigDecimal("0.30"));
        }
        return BigDecimal.ZERO;
        // Cada nuevo tipo de cliente obliga a MODIFICAR esta clase
    }
}
```

### ✅ Solución con una abstracción

```java
public interface PoliticaDescuento {
    BigDecimal calcular(BigDecimal importe);
}

public class DescuentoVip implements PoliticaDescuento {
    public BigDecimal calcular(BigDecimal importe) {
        return importe.multiply(new BigDecimal("0.20"));
    }
}

public class DescuentoEmpleado implements PoliticaDescuento {
    public BigDecimal calcular(BigDecimal importe) {
        return importe.multiply(new BigDecimal("0.30"));
    }
}

public class CalculadoraDescuentos {

    public BigDecimal calcular(PoliticaDescuento politica, BigDecimal importe) {
        return politica.calcular(importe);
    }
}
```

Para añadir un descuento de *Black Friday* basta con **crear una clase nueva**; `CalculadoraDescuentos` no se toca. Como `PoliticaDescuento` tiene un único método, también se puede implementar con una *lambda*:

```java
PoliticaDescuento blackFriday = importe -> importe.multiply(new BigDecimal("0.40"));
```

> **Matiz con `sealed`:** una interfaz `sealed` (Java 17) cierra **a propósito** la jerarquía. Es útil cuando el conjunto de variantes es fijo y conocido (por ejemplo, `Circulo | Rectangulo | Triangulo`) y se quiere que el compilador compruebe que todos los casos están tratados. Si se espera que aparezcan nuevas variantes con frecuencia (como los descuentos), es mejor una interfaz abierta.

---

## 5. L — Principio de sustitución de Liskov (LSP)

> **Si S es un subtipo de T, los objetos de tipo T pueden sustituirse por objetos de tipo S sin alterar ninguna de las propiedades deseables del programa.**

Dicho de forma sencilla: **una subclase debe poder usarse en cualquier lugar donde se use su clase padre, sin que el programa falle ni se comporte de forma inesperada**.

La subclase debe aceptar las mismas peticiones y entregar el mismo resultado que la clase padre, o un resultado **del mismo tipo**. Una analogía: si la clase padre sirve café, que la hija sirva un capuchino es aceptable (es un tipo de café); que sirva agua, no.

**Objetivo:** garantizar la coherencia, para que la clase padre y sus hijas puedan usarse del mismo modo sin errores.

### ❌ Incumplimiento

```java
public interface Coche {
    void encenderMotor();
    void acelerar();
}

public class CocheCombustion implements Coche {
    private final Motor motor;

    public CocheCombustion(Motor motor) {
        this.motor = motor;
    }

    public void encenderMotor() { motor.encender(); }
    public void acelerar()      { motor.darPotencia(1000); }
}

public class CocheElectrico implements Coche {
    public void encenderMotor() {
        throw new UnsupportedOperationException("¡No tengo motor de combustión!");
    }
    public void acelerar() { /* ... */ }
}
```

Cualquier código que haga `coche.encenderMotor()` funcionará con unos coches y **explotará** con otros. `CocheElectrico` no es sustituible por `Coche`.

> **Señal de alarma:** un método sobrescrito que lanza `UnsupportedOperationException`, que no hace nada (`{}`) o que exige condiciones más estrictas que el padre suele indicar una violación de LSP.

### ✅ Solución: modelar lo que realmente es común

```java
public interface Vehiculo {
    void ponerEnMarcha();
    void acelerar();
}

public class CocheCombustion implements Vehiculo {
    private final Motor motor;

    public CocheCombustion(Motor motor) {
        this.motor = motor;
    }

    public void ponerEnMarcha() { motor.encender(); }
    public void acelerar()      { motor.darPotencia(1000); }
}

public class CocheElectrico implements Vehiculo {
    private final Bateria bateria;

    public CocheElectrico(Bateria bateria) {
        this.bateria = bateria;
    }

    public void ponerEnMarcha() { bateria.activar(); }
    public void acelerar()      { bateria.suministrar(1000); }
}
```

La abstracción describe lo que **todos** los vehículos pueden hacer de verdad. Cada uno lo implementa a su manera, y cualquier `Vehiculo` es intercambiable.

### El ejemplo clásico: cuadrado y rectángulo

Matemáticamente, un cuadrado «es un» rectángulo. Pero en código:

```java
class Rectangulo {
    protected int ancho, alto;
    void setAncho(int ancho) { this.ancho = ancho; }
    void setAlto(int alto)   { this.alto = alto; }
    int area() { return ancho * alto; }
}

class Cuadrado extends Rectangulo {
    @Override void setAncho(int lado) { ancho = lado; alto = lado; }
    @Override void setAlto(int lado)  { ancho = lado; alto = lado; }
}
```

```java
Rectangulo r = new Cuadrado();
r.setAncho(5);
r.setAlto(4);
// Quien usa un Rectangulo espera área 20, pero obtiene 16
```

La relación «es un» del mundo real **no garantiza** que la herencia sea correcta en código. La solución habitual es no relacionarlos por herencia o hacerlos **inmutables** (por ejemplo, como `record`), de modo que no existan *setters* cuyo contrato se pueda romper:

```java
sealed interface Figura permits Rectangulo, Cuadrado {
    int area();
}
record Rectangulo(int ancho, int alto) implements Figura {
    public int area() { return ancho * alto; }
}
record Cuadrado(int lado) implements Figura {
    public int area() { return lado * lado; }
}
```

---

## 6. I — Principio de segregación de interfaces (ISP)

> **Los clientes no deben verse obligados a depender de métodos que no usan.**

Las interfaces grandes deben dividirse en otras más pequeñas, de modo que cada clase solo tenga que preocuparse de los métodos que le interesan. Obligar a una clase a implementar acciones que no le sirven es un desperdicio y puede provocar errores inesperados si la clase no es capaz de realizarlas.

**Objetivo:** dividir un conjunto de acciones en conjuntos más pequeños para que cada clase ejecute **solo** las que necesita.

### ❌ Incumplimiento: el cuidador de osos

```java
public interface CuidadorOsos {
    void lavarAlOso();
    void alimentarAlOso();
    void acariciarAlOso();
}
```

Lavar y alimentar a los osos está bien, pero acariciarlos es peligroso. Con esta interfaz, cualquier cuidador está obligado a implementar `acariciarAlOso()`.

### ✅ Solución

```java
public interface LimpiadorOsos {
    void lavarAlOso();
}

public interface AlimentadorOsos {
    void alimentarAlOso();
}

public interface AcariciadorOsos {
    void acariciarAlOso();
}

public class Cuidador implements LimpiadorOsos, AlimentadorOsos {
    public void lavarAlOso()     { /* ... */ }
    public void alimentarAlOso() { /* ... */ }
}

public class Temerario implements AcariciadorOsos {
    public void acariciarAlOso() { /* ¡suerte! */ }
}
```

Cada clase implementa solo lo que le corresponde. Del mismo modo, `ImpresoraLibros` (del ejemplo de SRP) podría convertirse en una interfaz `ImpresoraLibro` con un único método `imprimir`, y tener implementaciones `ImpresoraConsola` e `ImpresoraPdf`.

### ISP en la biblioteca estándar de Java

El JDK aplica esta idea en muchos sitios. Por ejemplo, en `java.util.function` hay interfaces pequeñas y especializadas en lugar de una gran interfaz «función»: `Supplier<T>` (solo produce), `Consumer<T>` (solo consume), `Predicate<T>` (solo evalúa) y `Function<T, R>` (transforma). Igualmente, `Comparable`, `AutoCloseable` o `Runnable` tienen un único método.

> **Cuidado con los métodos `default`** (Java 8): permiten añadir métodos a una interfaz sin romper las implementaciones existentes, pero no son excusa para engordarla con responsabilidades que no le corresponden.

---

## 7. D — Principio de inversión de dependencias (DIP)

> 1. **Los módulos de alto nivel no deben depender de los de bajo nivel. Ambos deben depender de abstracciones.**
> 2. **Las abstracciones no deben depender de los detalles. Los detalles deben depender de las abstracciones.**

Términos:

- **Módulo de alto nivel:** la clase que realiza una acción usando una herramienta (la lógica de negocio).
- **Módulo de bajo nivel:** la herramienta necesaria para realizar la acción (base de datos, teclado, envío de correos…).
- **Abstracción:** la interfaz que conecta ambas clases.
- **Detalles:** cómo funciona la herramienta por dentro.

La clase no debe estar «soldada» a la herramienta que usa, sino a la **interfaz** que permite conectar esa herramienta. Ni la clase ni la interfaz deben conocer cómo funciona la herramienta; es la herramienta la que debe cumplir la especificación de la interfaz.

**Objetivo:** reducir la dependencia de las clases de alto nivel respecto a las de bajo nivel introduciendo una interfaz.

### ❌ Incumplimiento

```java
public class Ordenador {
    private final TecladoEstandar teclado;
    private final Monitor monitor;

    public Ordenador() {
        this.teclado = new TecladoEstandar();
        this.monitor = new Monitor();
    }
}
```

Al crear las dependencias con `new`, las tres clases quedan **fuertemente acopladas**: el ordenador es difícil de probar y no se puede cambiar el teclado o el monitor por otros.

### ✅ Solución: depender de abstracciones e inyectarlas

```java
public interface Teclado { }
public interface Pantalla { }

public class TecladoEstandar implements Teclado { }
public class TecladoErgonomico implements Teclado { }
public class Monitor implements Pantalla { }

public class Ordenador {
    private final Teclado teclado;
    private final Pantalla pantalla;

    public Ordenador(Teclado teclado, Pantalla pantalla) {   // inyección de dependencias
        this.teclado = teclado;
        this.pantalla = pantalla;
    }
}
```

```java
var oficina = new Ordenador(new TecladoErgonomico(), new Monitor());
```

Ahora `Ordenador` se comunica con sus dependencias a través de abstracciones. Se puede cambiar el teclado por cualquier implementación y probar la clase con dobles de prueba.

### ¿Por qué se llama «inversión»?

Sin DIP, la flecha de dependencia va del negocio al detalle técnico:

```text
ServicioPedidos ──▶ RepositorioMySql
```

Con DIP, el negocio **define** la interfaz que necesita y el detalle técnico la **implementa**. La dependencia del código fuente se **invierte**:

```text
ServicioPedidos ──▶ RepositorioPedidos (interfaz) ◀── RepositorioMySql
```

```java
// La interfaz pertenece a la capa de negocio y habla su idioma
public interface RepositorioPedidos {
    void guardar(Pedido pedido);
    Optional<Pedido> buscarPorId(String id);
}

public class ServicioPedidos {
    private final RepositorioPedidos repositorio;

    public ServicioPedidos(RepositorioPedidos repositorio) {
        this.repositorio = repositorio;
    }
}

// El detalle técnico, en otra capa, depende de la abstracción
public class RepositorioPedidosJdbc implements RepositorioPedidos {
    // ... implementación con JDBC
}
```

> **DIP no es lo mismo que inyección de dependencias.** DIP es el **principio** (depender de abstracciones); la inyección de dependencias es una **técnica** para cumplirlo (pasar las dependencias desde fuera). Frameworks como Spring, Quarkus o Micronaut automatizan esa inyección mediante un **contenedor** de inversión de control.

---

## 8. Cómo encajan los cinco principios

- **SRP** y la **cohesión**: una responsabilidad por clase equivale a alta cohesión.
- **OCP** se apoya en **LSP**: para extender un sistema con nuevas implementaciones sin modificarlo, esas implementaciones deben ser sustituibles de verdad.
- **ISP** facilita **LSP**: con interfaces pequeñas es menos probable que una clase tenga que «fingir» que sabe hacer algo.
- **DIP** reduce el **acoplamiento**: el código de negocio depende de abstracciones estables, no de detalles que cambian.
- Todos ayudan a cumplir **DRY** y **Tell, Don't Ask**, y deben aplicarse sin olvidar **KISS**.

## 9. SOLID y Java moderno

| Característica | Versión | Relación con SOLID |
|---|---|---|
| Interfaces funcionales y *lambdas* | Java 8 | Implementaciones ligeras de abstracciones pequeñas (OCP, ISP, DIP). |
| `record` | Java 16 | Valores inmutables: sin *setters* cuyo contrato pueda romperse (LSP) y con una única responsabilidad: representar datos (SRP). |
| `sealed` | Java 17 | Jerarquías cerradas y controladas; el compilador comprueba la exhaustividad. |
| `switch` con patrones y patrones de `record` | Java 21 | Tratar jerarquías cerradas sin cadenas de `instanceof`. |
| Constructores flexibles | Java 25 | Validar argumentos antes de `super(...)`, ayudando a que las subclases respeten las invariantes del padre (LSP). |
| Módulos de la plataforma (`module-info.java`) | Java 9 | Exportar solo las abstracciones y ocultar los detalles (DIP a nivel de módulo). |

## 10. Errores frecuentes al aplicar SOLID

- **Aplicarlos de forma dogmática.** Son guías, no leyes. Una interfaz para cada clase con una sola implementación, o dividir una clase de 30 líneas en cinco, suele empeorar el código (viola KISS).
- **Confundir SRP con «un método por clase».** Una responsabilidad puede necesitar varios métodos.
- **Usar herencia para todo al aplicar OCP.** Las abstracciones y la composición suelen ser más flexibles.
- **Pensar que «es un» en el mundo real implica herencia correcta** (el problema del cuadrado y el rectángulo).
- **Crear interfaces que reflejan la implementación** (`RepositorioMySql` con métodos específicos de MySQL) en lugar de las necesidades de quien las usa.

## 11. Tabla resumen

| Principio | Idea en una frase | Señal de incumplimiento | Herramienta típica |
|---|---|---|---|
| **SRP** | Un único motivo para cambiar. | La clase mezcla negocio, presentación y persistencia. | Extraer clases. |
| **OCP** | Añadir sin modificar. | Cadenas de `if`/`switch` que crecen con cada caso nuevo. | Interfaces y polimorfismo. |
| **LSP** | Las subclases deben ser sustituibles. | `UnsupportedOperationException` en métodos heredados. | Rediseñar la abstracción; inmutabilidad. |
| **ISP** | Interfaces pequeñas y específicas. | Métodos vacíos o que lanzan excepciones «porque la interfaz lo obliga». | Dividir interfaces. |
| **DIP** | Depender de abstracciones. | `new ClaseConcreta()` dentro de la lógica de negocio. | Interfaces + inyección de dependencias. |

## 12. Lista de comprobación

- [ ] ¿Puedo describir la responsabilidad de cada clase en una frase sin «y»?
- [ ] ¿Añadir un nuevo caso obliga a modificar clases existentes?
- [ ] ¿Alguna subclase lanza excepciones o deja métodos vacíos que el padre sí implementa?
- [ ] ¿Alguna clase implementa métodos de una interfaz que no necesita?
- [ ] ¿La lógica de negocio crea sus dependencias técnicas con `new`?
- [ ] ¿Puedo probar cada clase de forma aislada?
- [ ] ¿La solución sigue siendo simple (KISS)?

## 13. Referencias

- Baeldung — *A Solid Guide to SOLID Principles*: <https://www.baeldung.com/solid-principles>
- Ugonna Thelma — *The S.O.L.I.D Principles in Pictures* (Medium): <https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898>
- JEP 395 — *Records* (Java 16): <https://openjdk.org/jeps/395>
- JEP 409 — *Sealed Classes* (Java 17): <https://openjdk.org/jeps/409>
- JEP 441 — *Pattern Matching for switch* (Java 21): <https://openjdk.org/jeps/441>
- JEP 513 — *Flexible Constructor Bodies* (Java 25): <https://openjdk.org/jeps/513>
- Robert C. Martin — *Design Principles and Design Patterns* (2000).
- Bertrand Meyer — *Object-Oriented Software Construction* (1988).
