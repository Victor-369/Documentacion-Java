# Principios SOLID

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).
>
> **Conocimientos previos:** saber escribir clases, interfaces y métodos en Java. No hace falta saber nada de diseño de software ni haber oído hablar antes de SOLID: los términos técnicos se explican en la sección [Conceptos previos](#2-conceptos-previos).

## Índice

1. [¿Qué es SOLID?](#1-qué-es-solid)
2. [Conceptos previos](#2-conceptos-previos)
3. [Origen](#3-origen)
4. [S — Principio de responsabilidad única (SRP)](#4-s--principio-de-responsabilidad-única-srp)
5. [O — Principio abierto/cerrado (OCP)](#5-o--principio-abiertocerrado-ocp)
6. [L — Principio de sustitución de Liskov (LSP)](#6-l--principio-de-sustitución-de-liskov-lsp)
7. [I — Principio de segregación de interfaces (ISP)](#7-i--principio-de-segregación-de-interfaces-isp)
8. [D — Principio de inversión de dependencias (DIP)](#8-d--principio-de-inversión-de-dependencias-dip)
9. [Cómo encajan los cinco principios](#9-cómo-encajan-los-cinco-principios)
10. [SOLID y Java moderno](#10-solid-y-java-moderno)
11. [Errores frecuentes al aplicar SOLID](#11-errores-frecuentes-al-aplicar-solid)
12. [Tabla resumen](#12-tabla-resumen)
13. [Lista de comprobación](#13-lista-de-comprobación)
14. [Cómo probar los ejemplos](#14-cómo-probar-los-ejemplos)
15. [Referencias](#15-referencias)

---

## 1. ¿Qué es SOLID?

**SOLID** es un acrónimo: cada letra es la inicial de **un principio de diseño orientado a objetos**. En total son **cinco**. Su objetivo es crear software **más fácil de mantener, de entender, de ampliar y de probar**, para que la complejidad no se dispare a medida que la aplicación crece.

| Letra | Principio | En inglés | La idea en pocas palabras |
|---|---|---|---|
| **S** | Responsabilidad única | *Single Responsibility Principle* | Cada clase se encarga de una sola cosa. |
| **O** | Abierto/cerrado | *Open/Closed Principle* | Añadir funcionalidad sin tocar lo que ya funciona. |
| **L** | Sustitución de Liskov | *Liskov Substitution Principle* | Una clase hija debe poder usarse donde se usa la clase padre. |
| **I** | Segregación de interfaces | *Interface Segregation Principle* | Mejor varias interfaces pequeñas que una enorme. |
| **D** | Inversión de dependencias | *Dependency Inversion Principle* | Depender de interfaces, no de clases concretas. |

**¿Por qué debería importarte?** Al principio un programa es pequeño y cualquier diseño funciona. El problema llega cuando crece: cambias algo en un sitio y se rompe otra cosa en otro sitio que no tenía nada que ver, o no puedes probar una clase sin arrancar media aplicación. SOLID son **consejos para evitar esos problemas antes de que aparezcan**.

Aunque se explican pensando en **clases**, las ideas sirven igual para **métodos**, **paquetes** y **módulos**.

Algunos principios se parecen entre sí, pero **cada uno persigue un objetivo distinto**: se puede cumplir uno e incumplir otro. Por eso conviene entender la **meta** de cada uno, no solo memorizar la definición.

> **Importante:** SOLID son **guías, no leyes**. Aplicarlos a ciegas también produce mal código (ver [Errores frecuentes](#11-errores-frecuentes-al-aplicar-solid)).

## 2. Conceptos previos

Estos términos aparecen continuamente en la guía. Si ya los conoces, puedes saltar esta sección.

| Término | Qué significa | Ejemplo en Java |
|---|---|---|
| **Interfaz** | Un **contrato**: indica *qué* métodos debe tener una clase, pero no *cómo* funcionan. | `interface Notificador { void notificar(String msg); }` |
| **Implementación** | Una clase que cumple el contrato de una interfaz y escribe el código de sus métodos. | `class EmailNotificador implements Notificador { ... }` |
| **Abstracción** | Algo que describe *qué* se hace sin entrar en *cómo*. En Java suele ser una interfaz (a veces una clase abstracta). | `Notificador` es una abstracción; `EmailNotificador` es un detalle concreto. |
| **Herencia** | Una clase hija (`extends`) recibe los campos y métodos de una clase padre. | `class Gato extends Animal` |
| **Composición** | Una clase **tiene** otro objeto como campo y lo usa, en lugar de heredar de él. | `class Coche { private final Motor motor; }` |
| **Polimorfismo** | Una misma variable de un tipo (por ejemplo, una interfaz) puede contener objetos de clases distintas, y cada uno responde a su manera. | `Notificador n = new EmailNotificador();` y luego `n = new SmsNotificador();` |
| **Dependencia** | La clase A **depende** de la clase B si A usa B: si B cambia, A puede verse afectada. | Si `Pedido` llama a `Impresora`, `Pedido` depende de `Impresora`. |
| **Acoplamiento** | Cuánto depende una clase de otras. **Alto acoplamiento = malo**: cambiar una obliga a cambiar las otras. | Una clase que hace `new MySqlConexion()` está muy acoplada a MySQL. |
| **Cohesión** | Cuánto tienen que ver entre sí las cosas que hay dentro de una clase. **Alta cohesión = bueno**: todo lo que contiene trata del mismo tema. | Una clase `Calculadora` con `sumar` y `restar` es cohesiva; si además envía correos, no. |
| **Inyección de dependencias** | En lugar de que una clase cree sus dependencias con `new`, **se las pasan desde fuera** (normalmente por el constructor). | `new ServicioPedidos(new RepositorioPedidosJdbc())` |
| **Doble de prueba** | Una implementación **falsa o simplificada** que se usa en los tests en lugar de la real (por ejemplo, un repositorio en memoria en vez de una base de datos). | `class RepositorioFalso implements Repositorio { ... }` |

Y algunas características de Java que aparecen en los ejemplos:

| Característica | Desde | Para qué sirve |
|---|---|---|
| *Lambda* (`x -> ...`) | Java 8 | Forma corta de implementar una interfaz que tiene **un solo método** abstracto. |
| `var` | Java 10 | Deja que el compilador deduzca el tipo de una variable local: `var lista = new ArrayList<String>();`. |
| `record` | Java 16 | Clase **inmutable** (sus campos no se pueden cambiar) pensada para guardar datos. El compilador genera automáticamente el constructor, los métodos de acceso (`titulo()`, no `getTitulo()`), `equals`, `hashCode` y `toString`. |
| `sealed` / `permits` | Java 17 | Una interfaz o clase `sealed` indica **exactamente qué clases** pueden implementarla o heredar de ella. Nadie más puede. |
| `IO.println(...)` | Java 25 | Escribe una línea en la consola. Hace lo mismo que `System.out.println(...)`, pero es más corto. |

---

## 3. Origen

Los principios fueron reunidos por **Robert C. Martin** (conocido como *Uncle Bob*) en su artículo *Design Principles and Design Patterns* (2000). Poco después, **Michael Feathers** propuso el acrónimo **SOLID** para recordarlos fácilmente. Algunos principios son anteriores:

- El principio **abierto/cerrado** fue formulado por **Bertrand Meyer** en *Object-Oriented Software Construction* (1988).
- El principio de **sustitución** lleva el nombre de **Barbara Liskov**, que lo presentó en 1987.

---

## 4. S — Principio de responsabilidad única (SRP)

> **Una clase debe tener una única responsabilidad o, dicho de otro modo, un único motivo para cambiar.**

**En palabras sencillas:** cada clase debe encargarse de **una sola cosa**. Piensa en un restaurante: el cocinero cocina, el camarero sirve y el cajero cobra. Si una sola persona hiciera las tres cosas y hubiera que cambiar la forma de cobrar, podría acabar afectando también a cómo cocina.

Con las clases pasa lo mismo: si una clase tiene muchas responsabilidades, al cambiar una de ellas se pueden romper las demás sin que nos demos cuenta.

**Objetivo:** separar comportamientos para que, si un cambio introduce un error, no afecte a otros comportamientos que no tienen nada que ver.

**Ventajas:**

- **Pruebas más sencillas:** una clase que hace una sola cosa necesita muchos menos casos de prueba.
- **Menor acoplamiento:** menos funcionalidad en una clase implica menos dependencias.
- **Organización:** es más fácil encontrar algo en muchas clases pequeñas con nombres claros que en una clase gigante que lo hace todo.

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
        IO.println(titulo + " — " + autor);
        IO.println(texto);
    }
}
```

**¿Dónde está el problema?** `Libro` hace dos cosas distintas:

1. Guardar y manejar el contenido del libro.
2. Decidir **cómo se muestra** ese libro (en la consola).

Si mañana hay que imprimirlo en PDF, enviarlo por correo o cambiar el formato de salida, **habrá que modificar `Libro`**, aunque los datos del libro no hayan cambiado.

### ✅ Solución

Separamos las dos responsabilidades en dos clases:

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

Uso:

```java
var libro = new Libro("El Quijote", "Cervantes", "En un lugar de la Mancha...");
new ImpresoraLibros().imprimirEnConsola(libro);
```

**¿Qué hemos ganado?** `Libro` ya no sabe nada de cómo se presenta. `ImpresoraLibros` se encarga solo de eso y puede crecer (por ejemplo, con un método `imprimirEnPdf`) **sin tocar el libro**.

> **Nota:** hemos convertido `Libro` en un `record` porque solo guarda datos que no cambian. Fíjate en que con un `record` se accede a los campos con `libro.titulo()` (sin `get`).

> **Responsabilidad no es lo mismo que acción.** «Ser el pintor» es una responsabilidad; «pintar paredes» o «pintar puertas» son acciones que la cumplen. Una clase puede tener varios métodos siempre que todos sirvan a la misma responsabilidad.
>
> Robert C. Martin aclaró más tarde que «un motivo para cambiar» equivale a **un único actor o grupo de interesados**: si el departamento de contabilidad y el de recursos humanos pueden pedir cambios en la misma clase por motivos distintos, esa clase tiene dos responsabilidades.

### Un truco práctico

Intenta describir la clase en una frase **sin usar "y"**. Con el original sale: «guarda y manipula el texto de un libro **y** lo imprime en consola». Ese "y" marca por dónde partir la clase.

La clase del dominio (la que representa algo del negocio, como un libro) se queda con lo que el objeto **es** y lo que hace con sus propios datos. Lo que tiene que ver con el mundo exterior (mostrarlo, enviarlo, guardarlo en una base de datos) se va a otra clase.

---

## 5. O — Principio abierto/cerrado (OCP)

> **Las clases deben estar abiertas a la extensión, pero cerradas a la modificación.**

**En palabras sencillas:** debe ser posible **añadir** comportamiento nuevo **sin modificar** el código que ya funciona. Piensa en un enchufe múltiple: para conectar un aparato nuevo no abres la regleta ni cambias sus cables; simplemente lo enchufas.

¿Por qué? Cambiar el comportamiento actual de una clase afecta a todos los sitios del programa que la usan, y cada cambio puede introducir errores. Si queremos que haga más cosas, lo ideal es **añadir** código nuevo, no **modificar** el existente. La excepción evidente es **corregir errores**: eso sí hay que hacerlo en la propia clase.

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

Así no se toca `Guitarra` y el resto de la aplicación no se ve afectado. Sin embargo, la herencia **ata mucho** la clase hija a la clase padre (cualquier cambio en el padre afecta a todas sus hijas), por lo que hoy se prefiere **extender mediante interfaces** y composición, como en el siguiente ejemplo.

### ❌ Incumplimiento típico

```java
import java.math.BigDecimal;

public record Cliente(String nombre, String tipo) { }

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

> **¿Por qué `BigDecimal` y no `double`?** Porque `double` comete pequeños errores de redondeo (por ejemplo, `0.1 + 0.2` da `0.30000000000000004`). Para dinero se usa `BigDecimal`, creado a partir de un `String`.

**¿Dónde está el problema?** Cada vez que el negocio invente un descuento nuevo (estudiantes, *Black Friday*, jubilados…) habrá que **abrir esta clase y añadir otro `else if`**. Cuantos más casos, más larga, más difícil de leer y más fácil equivocarse y romper los descuentos que ya funcionaban.

### ✅ Solución con una abstracción

Creamos una interfaz que representa «una forma de calcular un descuento» y una clase por cada descuento:

```java
import java.math.BigDecimal;

public interface PoliticaDescuento {
    BigDecimal calcular(BigDecimal importe);
}

public class DescuentoVip implements PoliticaDescuento {
    @Override
    public BigDecimal calcular(BigDecimal importe) {
        return importe.multiply(new BigDecimal("0.20"));
    }
}

public class DescuentoEmpleado implements PoliticaDescuento {
    @Override
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

Uso:

```java
var calculadora = new CalculadoraDescuentos();
BigDecimal descuento = calculadora.calcular(new DescuentoVip(), new BigDecimal("100"));
IO.println(descuento); // 20.00
```

**¿Qué hemos ganado?** Para añadir un descuento de *Black Friday* basta con **crear una clase nueva** que implemente `PoliticaDescuento`; `CalculadoraDescuentos` no se toca. La clase está **cerrada** a la modificación pero **abierta** a la extensión.

Como `PoliticaDescuento` tiene un único método, también se puede implementar con una *lambda*, sin crear una clase:

```java
PoliticaDescuento blackFriday = importe -> importe.multiply(new BigDecimal("0.40"));
```

> **Matiz con `sealed`:** una interfaz `sealed` (Java 17) cierra **a propósito** la lista de clases que pueden implementarla. Es útil cuando el conjunto de variantes es fijo y conocido (por ejemplo, las figuras `Circulo`, `Rectangulo` y `Triangulo`) y se quiere que el compilador avise si nos olvidamos de tratar alguna. Si se espera que aparezcan nuevas variantes con frecuencia (como los descuentos), es mejor una interfaz normal, abierta.

---

## 6. L — Principio de sustitución de Liskov (LSP)

> **Si S es un subtipo de T, los objetos de tipo T pueden sustituirse por objetos de tipo S sin alterar ninguna de las propiedades deseables del programa.**

**En palabras sencillas:** si una clase hereda de otra (o implementa una interfaz), **debe poder usarse en cualquier lugar donde se use la clase padre (o la interfaz), sin que el programa falle ni haga cosas inesperadas**.

La clase hija debe aceptar las mismas peticiones que la clase padre y entregar el mismo tipo de resultado. Una analogía: si la clase padre es una cafetera que sirve café, que la hija sirva un capuchino es aceptable (sigue siendo café); que sirva agua, no.

**Objetivo:** garantizar la coherencia, para que la clase padre y sus hijas puedan usarse del mismo modo sin errores.

### ❌ Incumplimiento

```java
public interface Coche {
    void encenderMotor();
    void acelerar();
}

// Motor es una clase sencilla con métodos encender() y darPotencia(int)
public class CocheCombustion implements Coche {
    private final Motor motor;

    public CocheCombustion(Motor motor) {
        this.motor = motor;
    }

    @Override public void encenderMotor() { motor.encender(); }
    @Override public void acelerar()      { motor.darPotencia(1000); }
}

public class CocheElectrico implements Coche {
    @Override
    public void encenderMotor() {
        throw new UnsupportedOperationException("¡No tengo motor de combustión!");
    }

    @Override public void acelerar() { /* ... */ }
}
```

**¿Dónde está el problema?** Imagina un método que trabaja con cualquier `Coche`:

```java
void arrancarTodos(List<Coche> coches) {
    for (Coche coche : coches) {
        coche.encenderMotor();   // 💥 explota si en la lista hay un CocheElectrico
    }
}
```

Quien escribió `arrancarTodos` confió en que **todo** `Coche` sabe encender el motor, porque así lo dice la interfaz. `CocheElectrico` rompe esa promesa, así que **no es sustituible** por `Coche`.

> **Señal de alarma:** un método sobrescrito que lanza `UnsupportedOperationException`, que no hace nada (`{}`) o que exige condiciones más estrictas que el padre (por ejemplo, el padre acepta cualquier número y la hija solo positivos) suele indicar una violación de LSP.

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

    @Override public void ponerEnMarcha() { motor.encender(); }
    @Override public void acelerar()      { motor.darPotencia(1000); }
}

// Bateria es una clase sencilla con métodos activar() y suministrar(int)
public class CocheElectrico implements Vehiculo {
    private final Bateria bateria;

    public CocheElectrico(Bateria bateria) {
        this.bateria = bateria;
    }

    @Override public void ponerEnMarcha() { bateria.activar(); }
    @Override public void acelerar()      { bateria.suministrar(1000); }
}
```

**¿Qué hemos ganado?** La interfaz ahora describe lo que **todos** los vehículos pueden hacer de verdad («ponerse en marcha»), no un detalle que solo tienen algunos («encender el motor»). Cada uno lo implementa a su manera, y cualquier `Vehiculo` es intercambiable por otro.

### El ejemplo clásico: cuadrado y rectángulo

En matemáticas, un cuadrado «es un» rectángulo (con los cuatro lados iguales). Así que parece lógico hacer que `Cuadrado` herede de `Rectangulo`:

```java
class Rectangulo {
    protected int ancho, alto;
    void setAncho(int ancho) { this.ancho = ancho; }
    void setAlto(int alto)   { this.alto = alto; }
    int area() { return ancho * alto; }
}

class Cuadrado extends Rectangulo {
    // Un cuadrado debe tener ancho == alto, así que cada setter cambia los dos
    @Override void setAncho(int lado) { ancho = lado; alto = lado; }
    @Override void setAlto(int lado)  { ancho = lado; alto = lado; }
}
```

Ahora alguien usa un `Rectangulo` sin saber que en realidad es un `Cuadrado`:

```java
Rectangulo r = new Cuadrado();
r.setAncho(5);   // ancho = 5, alto = 5
r.setAlto(4);    // ancho = 4, alto = 4
IO.println(r.area());
// Quien usa un Rectangulo espera 5 × 4 = 20, pero obtiene 4 × 4 = 16
```

**Lección:** que en el mundo real algo «sea un» tipo de otra cosa **no garantiza** que la herencia sea correcta en código. Lo que importa es que la clase hija **se comporte** como espera quien usa la clase padre.

La solución habitual es no relacionarlos por herencia, o hacerlos **inmutables** (por ejemplo, con `record`), de modo que no existan *setters* cuyo comportamiento se pueda romper. Si quieres otro tamaño, creas una figura nueva:

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

`Rectangulo` y `Cuadrado` ya no heredan uno del otro: los dos son una `Figura` y cada uno calcula su área correctamente. `sealed ... permits` indica que **solo** estas dos clases pueden ser `Figura`.

---

## 7. I — Principio de segregación de interfaces (ISP)

> **Los clientes no deben verse obligados a depender de métodos que no usan.**

*(Aquí «cliente» significa «la clase que usa o implementa la interfaz», no un cliente de una tienda.)*

**En palabras sencillas:** es mejor tener **varias interfaces pequeñas** que una interfaz enorme. Así, cada clase implementa solo los métodos que de verdad le interesan. Piensa en un mando a distancia con 80 botones cuando solo usas 5: incómodo y fácil de pulsar el que no toca.

Obligar a una clase a implementar métodos que no le sirven es un desperdicio y puede provocar errores inesperados si la clase no es capaz de hacer lo que el método promete.

**Objetivo:** dividir un conjunto de acciones en conjuntos más pequeños para que cada clase implemente **solo** las que necesita.

### ❌ Incumplimiento: el cuidador de osos

```java
public interface CuidadorOsos {
    void lavarAlOso();
    void alimentarAlOso();
    void acariciarAlOso();
}
```

Lavar y alimentar a los osos está bien, pero acariciarlos es peligroso y no todos los cuidadores deberían hacerlo. Con esta interfaz, **cualquier** cuidador está obligado a implementar `acariciarAlOso()`. ¿Y qué hace un cuidador prudente con ese método? Lo deja vacío o lanza una excepción… que es justo la señal de alarma de LSP que vimos antes.

### ✅ Solución

Dividimos la interfaz grande en tres pequeñas:

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

// Una clase puede implementar varias interfaces a la vez
public class Cuidador implements LimpiadorOsos, AlimentadorOsos {
    @Override public void lavarAlOso()     { /* ... */ }
    @Override public void alimentarAlOso() { /* ... */ }
}

public class Temerario implements AcariciadorOsos {
    @Override public void acariciarAlOso() { /* ¡suerte! */ }
}
```

**¿Qué hemos ganado?** Cada clase implementa solo lo que le corresponde. Ningún método queda vacío ni lanza excepciones «porque la interfaz lo obliga».

### ISP en la biblioteca estándar de Java

El propio JDK aplica esta idea en muchos sitios. Por ejemplo, en `java.util.function` hay interfaces pequeñas y especializadas en lugar de una gran interfaz «función»:

- `Supplier<T>`: solo **produce** un valor.
- `Consumer<T>`: solo **recibe** un valor y hace algo con él.
- `Predicate<T>`: solo **evalúa** si algo es verdadero o falso.
- `Function<T, R>`: **transforma** un valor en otro.

Igualmente, `Comparable`, `AutoCloseable` o `Runnable` tienen un único método.

> **Cuidado con los métodos `default`** (Java 8): permiten añadir a una interfaz métodos con código ya escrito, sin romper las clases que ya la implementan. Son útiles, pero no son excusa para engordar la interfaz con responsabilidades que no le corresponden.

---

## 8. D — Principio de inversión de dependencias (DIP)

> 1. **Los módulos de alto nivel no deben depender de los de bajo nivel. Ambos deben depender de abstracciones.**
> 2. **Las abstracciones no deben depender de los detalles. Los detalles deben depender de las abstracciones.**

Términos:

- **Módulo de alto nivel:** la clase que decide **qué** hay que hacer: la lógica de negocio (por ejemplo, «registrar un pedido»).
- **Módulo de bajo nivel:** la herramienta técnica que se usa para hacerlo: **cómo** se hace (base de datos, teclado, envío de correos…).
- **Abstracción:** la interfaz que conecta ambas clases.
- **Detalles:** cómo funciona la herramienta por dentro.

**En palabras sencillas:** piensa en una lámpara y un enchufe. La lámpara no va **soldada** a los cables de la pared: tiene un **enchufe estándar**. La lámpara solo necesita que haya un enchufe, y la instalación eléctrica solo tiene que ofrecerlo. Si mañana enchufas un ventilador en lugar de la lámpara, la pared no cambia. El enchufe es la **abstracción**: los dos lados dependen de él, y no el uno del otro.

En código: la clase no debe estar «soldada» a la herramienta que usa, sino a la **interfaz** que permite conectar esa herramienta. Ni la clase ni la interfaz deben conocer cómo funciona la herramienta por dentro; es la herramienta la que debe cumplir el contrato de la interfaz.

**Objetivo:** reducir la dependencia de las clases de alto nivel respecto a las de bajo nivel introduciendo una interfaz entre ellas.

### ❌ Incumplimiento

```java
public class TecladoEstandar {
    public String leer() { return "texto tecleado"; }
}

public class Monitor {
    public void mostrar(String texto) { IO.println(texto); }
}

public class Ordenador {
    private final TecladoEstandar teclado;
    private final Monitor monitor;

    public Ordenador() {
        this.teclado = new TecladoEstandar();   // 👈 el Ordenador crea sus piezas
        this.monitor = new Monitor();           // 👈 y elige cuáles son
    }

    public void eco() {
        monitor.mostrar(teclado.leer());
    }
}
```

**¿Dónde está el problema?** Al crear las piezas con `new` dentro de la propia clase, `Ordenador` queda **soldado** a `TecladoEstandar` y a `Monitor`:

- Si quieres un ordenador con teclado ergonómico, tienes que **modificar** `Ordenador`.
- Para probar `Ordenador` en un test necesitas el teclado y el monitor reales; no puedes usar unos falsos.

### ✅ Solución: depender de abstracciones e inyectarlas

```java
public interface Teclado {
    String leer();
}

public interface Pantalla {
    void mostrar(String texto);
}

public class TecladoEstandar implements Teclado {
    @Override public String leer() { return "texto tecleado"; }
}

public class TecladoErgonomico implements Teclado {
    @Override public String leer() { return "texto tecleado con comodidad"; }
}

public class Monitor implements Pantalla {
    @Override public void mostrar(String texto) { IO.println(texto); }
}

public class Ordenador {
    private final Teclado teclado;
    private final Pantalla pantalla;

    // Inyección de dependencias: las piezas llegan desde fuera
    public Ordenador(Teclado teclado, Pantalla pantalla) {
        this.teclado = teclado;
        this.pantalla = pantalla;
    }

    public void eco() {
        pantalla.mostrar(teclado.leer());
    }
}
```

Quien crea el `Ordenador` decide qué piezas usar:

```java
var oficina = new Ordenador(new TecladoErgonomico(), new Monitor());
oficina.eco();
```

**¿Qué hemos ganado?** `Ordenador` solo conoce las interfaces `Teclado` y `Pantalla`. Se puede cambiar el teclado por cualquier implementación sin tocar `Ordenador`, y en los tests se le pueden pasar **dobles de prueba** (piezas falsas).

### ¿Por qué se llama «inversión»?

Sin DIP, la flecha de dependencia va del negocio al detalle técnico. Lo importante (el negocio) depende de lo que cambia (la tecnología):

```text
ServicioPedidos ──▶ RepositorioMySql
```

Con DIP, el negocio **define** la interfaz que necesita y el detalle técnico la **implementa**. La flecha que salía del detalle técnico ahora **apunta hacia** la abstracción: la dependencia se ha **invertido**.

```text
ServicioPedidos ──▶ RepositorioPedidos (interfaz) ◀── RepositorioPedidosJdbc
```

> Un *repositorio* es simplemente la clase que se encarga de **guardar y buscar** objetos (normalmente en una base de datos). *JDBC* es la API estándar de Java para hablar con bases de datos.

```java
import java.math.BigDecimal;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

public record Pedido(String id, BigDecimal total) { }

// La interfaz pertenece a la capa de negocio y habla su idioma
public interface RepositorioPedidos {
    void guardar(Pedido pedido);
    Optional<Pedido> buscarPorId(String id);   // Optional: puede que no exista
}

// Alto nivel: solo conoce la interfaz
public class ServicioPedidos {
    private final RepositorioPedidos repositorio;

    public ServicioPedidos(RepositorioPedidos repositorio) {
        this.repositorio = repositorio;
    }

    public void registrar(Pedido pedido) {
        if (pedido.total().signum() <= 0) {
            throw new IllegalArgumentException("El total debe ser positivo");
        }
        repositorio.guardar(pedido);
    }
}

// Detalle técnico, en otra capa: depende de la abstracción
public class RepositorioPedidosJdbc implements RepositorioPedidos {
    // ... implementación con JDBC contra una base de datos real
}

// Otro detalle: guarda en memoria. Ideal como doble de prueba en los tests
public class RepositorioPedidosEnMemoria implements RepositorioPedidos {
    private final Map<String, Pedido> pedidos = new HashMap<>();

    @Override
    public void guardar(Pedido pedido) {
        pedidos.put(pedido.id(), pedido);
    }

    @Override
    public Optional<Pedido> buscarPorId(String id) {
        return Optional.ofNullable(pedidos.get(id));
    }
}
```

Gracias a DIP, podemos probar la lógica de `ServicioPedidos` **sin base de datos**:

```java
var repositorio = new RepositorioPedidosEnMemoria();
var servicio = new ServicioPedidos(repositorio);

servicio.registrar(new Pedido("P-1", new BigDecimal("49.90")));

IO.println(repositorio.buscarPorId("P-1").isPresent()); // true
```

> **DIP no es lo mismo que inyección de dependencias.** DIP es el **principio** (depender de abstracciones); la inyección de dependencias es una **técnica** para cumplirlo (pasar las dependencias desde fuera, como hemos hecho con los constructores). Frameworks como Spring, Quarkus o Micronaut hacen esa inyección **automáticamente**: tú declaras qué necesita cada clase y el framework crea los objetos y se los pasa. Pero **no hace falta ningún framework** para aplicar DIP: los ejemplos de arriba son Java puro.

---

## 9. Cómo encajan los cinco principios

Los principios no viven aislados; se ayudan entre sí:

- **SRP** y la **cohesión**: una responsabilidad por clase equivale a alta cohesión.
- **OCP** se apoya en **LSP**: para extender un sistema con nuevas implementaciones sin modificarlo, esas implementaciones deben ser sustituibles de verdad.
- **ISP** facilita **LSP**: con interfaces pequeñas es menos probable que una clase tenga que «fingir» que sabe hacer algo (como el cuidador que no quiere acariciar osos).
- **DIP** reduce el **acoplamiento**: el código de negocio depende de interfaces estables, no de detalles técnicos que cambian.
- **OCP** y **DIP** usan la misma herramienta: **interfaces**. En los dos casos, el código importante habla con una interfaz y las variantes concretas se enchufan sin tocarlo.

Hay otros principios de diseño que verás a menudo junto a SOLID:

- **DRY** (*Don't Repeat Yourself*, «no te repitas»): no copies y pegues la misma lógica en varios sitios.
- **KISS** (*Keep It Simple, Stupid*, «mantenlo simple»): la solución más sencilla que funcione suele ser la mejor. SOLID nunca debe convertir un problema simple en uno complicado.
- **Tell, Don't Ask** («di, no preguntes»): en lugar de pedirle sus datos a un objeto y decidir fuera qué hacer con ellos, pídele al objeto que lo haga él.

## 10. SOLID y Java moderno

Las versiones recientes de Java incluyen características que ayudan a escribir código que cumple SOLID:

| Característica | Versión | Relación con SOLID |
|---|---|---|
| Interfaces funcionales y *lambdas* | Java 8 | Implementar interfaces pequeñas de un solo método sin escribir una clase entera (OCP, ISP, DIP). |
| Módulos de la plataforma (`module-info.java`) | Java 9 | Exportar solo las interfaces y ocultar las clases de implementación (DIP a nivel de módulo). |
| `record` | Java 16 | Datos inmutables: sin *setters* cuyo comportamiento pueda romperse (LSP) y con una única responsabilidad, representar datos (SRP). |
| `sealed` | Java 17 | Jerarquías cerradas y controladas: el compilador sabe cuáles son **todas** las clases posibles. |
| `switch` con patrones y patrones de `record` | Java 21 | Tratar jerarquías `sealed` sin cadenas de `instanceof`. |
| Cuerpos de constructor flexibles | Java 25 | Permite escribir código (por ejemplo, validar argumentos) **antes** de llamar a `super(...)`. No es un principio SOLID en sí, pero ayuda a que una clase hija no se construya en un estado inválido. |
| Ficheros fuente compactos y `IO.println` | Java 25 | No tiene que ver con SOLID, pero facilita probar los ejemplos (ver [Cómo probar los ejemplos](#14-cómo-probar-los-ejemplos)). |

Ejemplo de `switch` con patrones sobre la jerarquía `sealed` `Figura` de la sección de LSP:

```java
static String describir(Figura figura) {
    return switch (figura) {
        case Rectangulo r -> "Rectángulo de " + r.ancho() + " × " + r.alto();
        case Cuadrado c   -> "Cuadrado de lado " + c.lado();
        // No hace falta "default": el compilador sabe que no hay más tipos de Figura
    };
}
```

> **Ojo, relación con OCP:** si añades una nueva figura a la lista `permits`, tendrás que **modificar** todos los `switch` como este. Es lo contrario de lo que pide OCP, pero a cambio el compilador te avisa de cada `switch` que falta por actualizar. Por eso `sealed` encaja cuando la lista de variantes cambia poco, y una interfaz abierta encaja cuando cambia a menudo (como vimos con los descuentos).

## 11. Errores frecuentes al aplicar SOLID

- **Aplicarlos de forma dogmática.** Son guías, no leyes. Crear una interfaz para cada clase aunque solo vaya a tener una implementación, o dividir una clase de 30 líneas en cinco, suele empeorar el código (viola KISS).
- **Confundir SRP con «un método por clase».** Una responsabilidad puede necesitar varios métodos.
- **Usar herencia para todo al aplicar OCP.** Las interfaces y la composición suelen ser más flexibles.
- **Pensar que «es un» en el mundo real implica herencia correcta** (el problema del cuadrado y el rectángulo).
- **Crear interfaces que reflejan la implementación** (por ejemplo, una interfaz `RepositorioMySql` con métodos específicos de MySQL) en lugar de las necesidades de quien las usa (`RepositorioPedidos` con `guardar` y `buscarPorId`).
- **Confundir DIP con usar un framework.** Spring automatiza la inyección de dependencias, pero el principio se aplica igual sin él.

## 12. Tabla resumen

| Principio | Idea en una frase | Señal de incumplimiento | Herramienta típica |
|---|---|---|---|
| **SRP** | Un único motivo para cambiar. | La clase mezcla lógica de negocio, presentación (mostrar datos) y persistencia (guardar datos). | Extraer clases. |
| **OCP** | Añadir sin modificar. | Cadenas de `if`/`switch` que crecen con cada caso nuevo. | Interfaces y polimorfismo. |
| **LSP** | Las clases hijas deben ser sustituibles por la clase padre. | `UnsupportedOperationException` en métodos heredados. | Rediseñar la abstracción; inmutabilidad. |
| **ISP** | Interfaces pequeñas y específicas. | Métodos vacíos o que lanzan excepciones «porque la interfaz lo obliga». | Dividir interfaces. |
| **DIP** | Depender de abstracciones. | `new ClaseConcreta()` de algo técnico dentro de la lógica de negocio. | Interfaces + inyección de dependencias. |

## 13. Lista de comprobación

Cuando revises tu código, hazte estas preguntas:

- [ ] ¿Puedo describir la responsabilidad de cada clase en una frase sin «y»? *(SRP)*
- [ ] ¿Añadir un nuevo caso me obliga a modificar clases que ya funcionan? *(OCP)*
- [ ] ¿Alguna clase hija lanza excepciones o deja vacíos métodos que el padre sí implementa? *(LSP)*
- [ ] ¿Alguna clase implementa métodos de una interfaz que no necesita? *(ISP)*
- [ ] ¿La lógica de negocio crea sus dependencias técnicas con `new`? *(DIP)*
- [ ] ¿Puedo probar cada clase de forma aislada, sin base de datos ni red?
- [ ] ¿La solución sigue siendo simple? *(KISS)*

## 14. Cómo probar los ejemplos

En un proyecto real, **cada clase, interfaz o `record` público va en su propio fichero `.java`** con el mismo nombre (por ejemplo, `Libro.java`). Los fragmentos de esta guía a veces muestran varias clases juntas para que se lean de un vistazo.

Para experimentar rápido, desde Java 25 puedes meterlo todo en **un solo fichero** con un `main` sencillo, quitando la palabra `public` de las clases, y ejecutarlo directamente sin compilar antes:

```java
// Fichero: Ejemplo.java
record Libro(String titulo, String autor, String texto) { }

class ImpresoraLibros {
    void imprimirEnConsola(Libro libro) {
        IO.println(libro.titulo() + " — " + libro.autor());
        IO.println(libro.texto());
    }
}

void main() {
    var libro = new Libro("El Quijote", "Cervantes", "En un lugar de la Mancha...");
    new ImpresoraLibros().imprimirEnConsola(libro);
}
```

```bash
java Ejemplo.java
```

En este tipo de fichero (llamado *fichero fuente compacto*), las clases más usadas de `java.base` (como `List`, `Map` u `Optional`) están disponibles sin escribir `import`. En un fichero normal sí hay que importarlas.

## 15. Referencias

- Baeldung — *A Solid Guide to SOLID Principles*: <https://www.baeldung.com/solid-principles>
- Ugonna Thelma — *The S.O.L.I.D Principles in Pictures* (Medium): <https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898>
- JEP 395 — *Records* (Java 16): <https://openjdk.org/jeps/395>
- JEP 409 — *Sealed Classes* (Java 17): <https://openjdk.org/jeps/409>
- JEP 441 — *Pattern Matching for switch* (Java 21): <https://openjdk.org/jeps/441>
- JEP 512 — *Compact Source Files and Instance Main Methods* (Java 25): <https://openjdk.org/jeps/512>
- JEP 513 — *Flexible Constructor Bodies* (Java 25): <https://openjdk.org/jeps/513>
- Robert C. Martin — *Design Principles and Design Patterns* (2000).
- Bertrand Meyer — *Object-Oriented Software Construction* (1988).
