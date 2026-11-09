# Patrones de diseño creacionales en Java

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los programas completos de este documento se han compilado y ejecutado con OpenJDK 25.
>
> Documentos relacionados: [Patrones estructurales](<Patrones estructurales.md>) · [Patrones de comportamiento](<Patrones comportamiento.md>)

## Índice

1. [¿Qué es un patrón de diseño?](#1-qué-es-un-patrón-de-diseño)
2. [Antes de empezar: cómo probar los ejemplos](#2-antes-de-empezar-cómo-probar-los-ejemplos)
3. [Vocabulario mínimo](#3-vocabulario-mínimo)
4. [¿Qué problema resuelven los patrones creacionales?](#4-qué-problema-resuelven-los-patrones-creacionales)
5. [Singleton](#5-singleton)
6. [Factory Method (método fábrica)](#6-factory-method-método-fábrica)
7. [Abstract Factory (fábrica abstracta)](#7-abstract-factory-fábrica-abstracta)
8. [Builder (constructor paso a paso)](#8-builder-constructor-paso-a-paso)
9. [Prototype (prototipo)](#9-prototype-prototipo)
10. [Resumen y cómo elegir](#10-resumen-y-cómo-elegir)
11. [Errores frecuentes](#11-errores-frecuentes)
12. [Ejercicios](#12-ejercicios)
13. [Glosario](#13-glosario)

---

## 1. ¿Qué es un patrón de diseño?

Cuando programas, hay problemas que aparecen una y otra vez en proyectos muy distintos: «necesito que solo exista un objeto de esta clase», «quiero crear objetos sin llenar el código de `if`», «tengo un objeto con demasiados datos para un solo constructor»…

Un **patrón de diseño** es una **solución ya probada** para uno de esos problemas repetidos. Tiene un nombre, explica el problema que resuelve y describe una forma de organizar las clases para resolverlo.

> **Analogía:** un patrón es como una **receta de cocina**. La receta de la tortilla no es *tu* tortilla: te dice los pasos, y tú la adaptas a los ingredientes que tienes. Del mismo modo, un patrón no es código para copiar y pegar, sino una idea que adaptas a tu programa.

### ¿Por qué merece la pena aprenderlos?

- **No reinventas la rueda:** otras personas ya pensaron una buena solución.
- **Hablas el mismo idioma que tu equipo:** decir «aquí usamos un *Builder*» se entiende al momento, sin explicar todo el código.
- **Reconoces el código que lees:** muchas librerías de Java usan estos patrones. Si los conoces, entiendes antes cómo funcionan.
- **Tu código aguanta mejor los cambios:** cada pieza tiene un papel claro, así que añadir algo nuevo no obliga a reescribir lo que ya funciona.

### Los tres grupos de patrones

Los patrones más conocidos se recogieron en el libro *Design Patterns* (1994), escrito por cuatro autores a los que se suele llamar *Gang of Four* (GoF, «la banda de los cuatro»). Los clasificaron según **para qué sirven**:

| Grupo | Para qué sirven | Ejemplos |
|---|---|---|
| **Creacionales** (este documento) | **Crear objetos** de forma ordenada y flexible | Singleton, Factory Method, Abstract Factory, Builder, Prototype |
| **Estructurales** | **Combinar** clases y objetos para formar estructuras más grandes | Adapter, Composite, Decorator, Facade, Proxy |
| **De comportamiento** | Organizar **cómo se comunican** los objetos y quién hace cada cosa | Strategy, Observer, Command, State, Template Method |

### Patrones y principios SOLID

**SOLID** son cinco principios para escribir código orientado a objetos fácil de mantener. **No necesitas conocerlos para seguir este documento**, pero si ya los conoces verás que muchos patrones son, en la práctica, una forma concreta de aplicarlos. Los tienes explicados en [Principios SOLID](<Principio SOLID.md>).

| Patrón | Principio que refuerza | Por qué |
|---|---|---|
| **Factory Method** | Abierto/cerrado (OCP) e inversión de dependencias (DIP) | El código pide un objeto a través de una interfaz y no hace `new` de clases concretas; añadir un tipo nuevo no obliga a tocar a quien lo usa. |
| **Abstract Factory** | Inversión de dependencias (DIP) | El programa trabaja con familias de objetos a través de interfaces, sin saber qué clases concretas hay detrás. |
| **Builder** | Responsabilidad única (SRP) | La forma de construir el objeto queda separada del propio objeto. |
| **Singleton** | ⚠️ Puede ir **en contra** de DIP | Si se abusa, cualquier clase accede a él directamente y se crean dependencias ocultas difíciles de cambiar y de probar. |

---

## 2. Antes de empezar: cómo probar los ejemplos

Cada patrón incluye un **programa completo** que puedes ejecutar. Para probarlo:

1. Instala un JDK 25 o superior (comprueba la versión con `java -version`).
2. Copia el código en un fichero de texto con extensión `.java`, por ejemplo `Singleton.java`.
3. Ejecútalo desde la terminal, en la misma carpeta:

```bash
java Singleton.java
```

Java compila y ejecuta el fichero en un solo paso.

Los ejemplos usan dos facilidades de **Java 25** pensadas para programas pequeños:

- **`void main()`**: el programa empieza aquí. No hace falta escribir `public static void main(String[] args)` ni declarar una clase alrededor.
- **`IO.println(...)`**: escribe un texto en la consola. Es lo mismo que `System.out.println(...)`, pero más corto.

> En un proyecto real (con Maven, Gradle o un IDE), cada clase iría en su propio fichero y el `main` estaría dentro de una clase. Las ideas de los patrones son exactamente las mismas.

---

## 3. Vocabulario mínimo

Estos son los términos de Java que aparecen en el documento. Si ya los conoces, puedes saltarte esta sección.

| Término | Qué significa | Ejemplo |
|---|---|---|
| **Clase** | El «molde» que describe cómo son unos objetos. | `class Coche { ... }` |
| **Objeto** (o instancia) | Una cosa concreta creada a partir de una clase. | `new Coche()` |
| **`new`** | Palabra que crea un objeto nuevo. | `Coche c = new Coche();` |
| **Constructor** | Código que se ejecuta al hacer `new` para preparar el objeto. | `Coche(String color) { ... }` |
| **Interfaz** | Una lista de métodos que una clase **promete** tener, sin decir cómo los hace. | `interface Vehiculo { void arrancar(); }` |
| **`implements`** | Indica que una clase cumple una interfaz. | `class Coche implements Vehiculo` |
| **`static`** | Algo que pertenece a la clase, no a cada objeto. Se usa sin crear objetos. | `Math.max(2, 3)` |
| **`final`** | Algo que no se puede cambiar una vez asignado. | `final int edad = 30;` |
| **`private`** | Solo se puede usar dentro de la propia clase. | `private String clave;` |
| **`enum`** | Un tipo con una lista **cerrada** de valores. | `enum Color { ROJO, VERDE }` |
| **`record`** | Una forma corta de crear una clase que solo guarda datos que no cambian. Java genera el constructor y los métodos para leer cada dato. | `record Punto(int x, int y) {}` → `p.x()` |
| **`switch` con flechas** | Elige un resultado según un valor. Con `->` no hace falta `break`. | `case ROJO -> "parar";` |
| **`List.of(...)`** | Crea una lista que no se puede modificar. | `List.of("a", "b")` |

---

## 4. ¿Qué problema resuelven los patrones creacionales?

En Java, los objetos se crean con `new`:

```java
Email aviso = new Email("ana@ejemplo.es");
```

Esto está bien en programas pequeños. Pero cuando el programa crece, aparecen problemas:

- **El `new` está repartido por todas partes.** Si mañana quieres cambiar `Email` por otra clase, tienes que buscar y modificar todos los sitios donde se crea.
- **Crear el objeto es complicado.** A veces hay que elegir entre varias clases, validar datos o pasar diez parámetros al constructor.
- **Necesitas controlar cuántos objetos existen.** Por ejemplo, que solo haya una configuración en toda la aplicación.

Los patrones creacionales **reúnen la creación de objetos en un único sitio** y la hacen más clara.

> **Analogía:** en una cafetería no entras a la cocina a preparar tu café: lo **pides** («un café con leche») y alguien que sabe hacerlo te lo entrega. Los patrones creacionales hacen lo mismo: tu código *pide* un objeto y otra pieza se encarga de *fabricarlo*.

---

## 5. Singleton

### En una frase

Garantiza que de una clase **solo exista un objeto** en todo el programa, y da una forma sencilla de llegar a él.

### Analogía

En un partido de baloncesto hay **un único marcador**. Todos (jugadores, árbitros y público) miran el mismo. Si hubiera dos marcadores con resultados distintos, nadie sabría cómo va el partido.

### El problema

Imagina una clase con la configuración de la aplicación. Si cada parte del programa hace su propio `new`, cada una tiene **su propia copia**, y los cambios de una no los ve la otra:

```java
Configuracion a = new Configuracion();
Configuracion b = new Configuracion();
a.cambiar("idioma", "en");
b.leer("idioma");   // sigue en "es": b es otro objeto distinto
```

### La solución en Java: un `enum` con un solo valor

Un `enum` tiene una lista cerrada de valores, y **Java no permite crear más con `new`**. Si el `enum` tiene un único valor, ese valor es, por definición, el único objeto que existe. Es la forma más sencilla y segura de hacer un Singleton en Java.

### Programa completo

```java
// Fichero: Singleton.java
import java.util.HashMap;
import java.util.Map;

enum Configuracion {
    INSTANCIA;   // el único objeto que existirá

    private final Map<String, String> valores = new HashMap<>();

    Configuracion() {   // se ejecuta una sola vez, la primera vez que se usa
        valores.put("idioma", "es");
        valores.put("moneda", "EUR");
    }

    String leer(String clave) {
        return valores.get(clave);
    }

    void cambiar(String clave, String valor) {
        valores.put(clave, valor);
    }
}

void main() {
    Configuracion a = Configuracion.INSTANCIA;
    Configuracion b = Configuracion.INSTANCIA;

    a.cambiar("idioma", "en");

    IO.println("Idioma leído desde b: " + b.leer("idioma"));
    IO.println("¿a y b son el mismo objeto? " + (a == b));
}
```

Salida:

```text
Idioma leído desde b: en
¿a y b son el mismo objeto? true
```

### Qué ha pasado

- `Configuracion.INSTANCIA` es siempre **el mismo objeto**, la uses desde donde la uses.
- Por eso, el cambio hecho a través de `a` se ve a través de `b`.
- `a == b` compara si dos variables apuntan **al mismo objeto**, y da `true`.
- Si intentas escribir `new Configuracion()`, el compilador lo rechaza: los `enum` no se pueden crear con `new`.

### La versión que verás en código antiguo

En muchos proyectos encontrarás el Singleton hecho con una clase normal: un constructor `private` (para que nadie haga `new` desde fuera) y un método `static` llamado `getInstance()` que devuelve siempre el mismo objeto. Funciona, pero es fácil equivocarse cuando varios hilos lo usan a la vez. En código nuevo, el `enum` es más simple y seguro.

### Cuándo usarlo

- Cuando de verdad solo debe haber **uno**: la configuración, el registro de mensajes (*log*) o una caché compartida.

### Cuándo **no** usarlo

- Para «tener a mano» cualquier objeto. Un Singleton es, en el fondo, una **variable global**: cualquier parte del programa puede cambiarlo, y eso hace más difícil encontrar errores y escribir pruebas.
- Alternativa habitual: crear **un** objeto al arrancar el programa y **pasárselo** (por el constructor) a las clases que lo necesiten. Los *frameworks* como Spring hacen exactamente esto de forma automática.

---

## 6. Factory Method (método fábrica)

### En una frase

Un **método que crea objetos por ti**: tú dices *qué* quieres y él decide *qué clase* usar.

### Analogía

En una pizzería pides «una margarita». No necesitas saber qué ingredientes lleva ni en qué orden se ponen: el pizzero lo sabe y te entrega la pizza correcta.

### El problema

Una tienda envía avisos a sus clientes por correo electrónico, SMS o notificación en el móvil. Sin patrón, cada vez que hay que enviar un aviso aparece el mismo bloque:

```java
if (canal.equals("EMAIL")) {
    aviso = new Email(destino);
} else if (canal.equals("SMS")) {
    aviso = new Sms(destino);
} else if ...
```

Si este bloque está copiado en diez sitios y se añade un canal nuevo (por ejemplo, WhatsApp), hay que cambiar los diez. Y si alguien escribe `"Emial"`, el error no se descubre hasta que el programa falla.

### La solución paso a paso

1. Define una **interfaz** con lo que todos los avisos saben hacer: `enviar(...)`.
2. Crea una clase para cada tipo de aviso que **implemente** esa interfaz.
3. Usa un **`enum`** para los canales posibles, en lugar de textos sueltos: así no hay errores de escritura.
4. Escribe **un único método** que reciba el canal y devuelva el objeto adecuado. Ese método es la «fábrica».

### Programa completo

```java
// Fichero: FactoryMethod.java

// 1. Lo que todos los avisos saben hacer
interface Notificacion {
    void enviar(String mensaje);
}

// 2. Un tipo de aviso por clase
record Email(String direccion) implements Notificacion {
    public void enviar(String mensaje) {
        IO.println("Email a " + direccion + ": " + mensaje);
    }
}

record Sms(String telefono) implements Notificacion {
    public void enviar(String mensaje) {
        IO.println("SMS a " + telefono + ": " + mensaje);
    }
}

record Push(String dispositivo) implements Notificacion {
    public void enviar(String mensaje) {
        IO.println("Push a " + dispositivo + ": " + mensaje);
    }
}

// 3. Los canales posibles: una lista cerrada
enum Canal { EMAIL, SMS, PUSH }

// 4. La fábrica: el ÚNICO sitio donde se decide qué clase se crea
class FabricaNotificaciones {
    static Notificacion crear(Canal canal, String destino) {
        return switch (canal) {
            case EMAIL -> new Email(destino);
            case SMS   -> new Sms(destino);
            case PUSH  -> new Push(destino);
        };
    }
}

void main() {
    Notificacion aviso1 = FabricaNotificaciones.crear(Canal.SMS, "600123456");
    Notificacion aviso2 = FabricaNotificaciones.crear(Canal.EMAIL, "ana@ejemplo.es");

    aviso1.enviar("Tu pedido ha salido");
    aviso2.enviar("Gracias por tu compra");
}
```

Salida:

```text
SMS a 600123456: Tu pedido ha salido
Email a ana@ejemplo.es: Gracias por tu compra
```

### Qué ha pasado

- El código de `main` **no hace ningún `new`** de `Email` o `Sms`. Solo pide un aviso a la fábrica y trabaja con la interfaz `Notificacion`.
- Si mañana se añade WhatsApp, solo cambian dos sitios: el `enum` y la fábrica. El resto del programa no se entera.
- **Ventaja extra del `switch` sobre un `enum`:** si añades `WHATSAPP` al `enum` y olvidas añadir su `case`, **el programa no compila**. El compilador te avisa del olvido antes de ejecutar nada.

### Dónde lo verás en Java

El propio Java está lleno de métodos fábrica. Fíjate en que se llaman con el nombre de la clase y no con `new`:

```java
List<String> nombres = List.of("Ana", "Luis");
LocalDate fecha      = LocalDate.of(2026, 11, 9);
Path ruta            = Path.of("datos", "clientes.txt");
```

`List.of` decide por ti qué clase de lista crear; tú solo sabes que es una `List`.

### La variante clásica

En el libro original, el método fábrica no es `static`: se declara `abstract` en una clase padre y cada clase hija decide qué objeto crea. La idea es la misma (separar *pedir* un objeto de *fabricarlo*), así que no te preocupes si la ves escrita de otra manera.

### Cuándo usarlo

- Cuando hay que elegir entre **varias clases** según un dato (un tipo, una opción del usuario, un fichero de configuración).
- Cuando quieres que el resto del código dependa de una **interfaz** y no de clases concretas.

### Cuándo **no** usarlo

- Si solo existe una clase y no hay nada que elegir, un `new` normal es más claro.

---

## 7. Abstract Factory (fábrica abstracta)

### En una frase

Una fábrica que crea **familias de objetos que combinan entre sí**, para que nunca mezcles piezas de familias distintas.

### Analogía

En una tienda de muebles eliges un **estilo**: nórdico o clásico. Si eliges nórdico, la mesa, las sillas y la estantería son todas nórdicas. No te llevarías una mesa nórdica con sillas clásicas, porque no combinan.

### El problema

Una aplicación tiene **modo claro** y **modo oscuro**. Cada pantalla usa botones y casillas de verificación. Si cada pantalla decide por su cuenta qué botón y qué casilla crear, es fácil acabar con un botón oscuro junto a una casilla clara.

### La solución paso a paso

1. Define una interfaz para cada tipo de pieza: `Boton` y `Casilla`.
2. Define una **interfaz de fábrica** con un método para crear cada pieza: `crearBoton()` y `crearCasilla()`.
3. Crea **una fábrica por familia**: `FabricaClara` y `FabricaOscura`. Cada una crea solo piezas de su estilo.
4. Las pantallas reciben **una fábrica** y le piden las piezas. Así, todas las piezas de una pantalla son siempre de la misma familia.

### Programa completo

```java
// Fichero: AbstractFactory.java

// 1. Las piezas
interface Boton   { void pintar(); }
interface Casilla { void pintar(); }

record BotonClaro()    implements Boton   { public void pintar() { IO.println("[ Botón claro ]"); } }
record BotonOscuro()   implements Boton   { public void pintar() { IO.println("[ Botón oscuro ]"); } }
record CasillaClara()  implements Casilla { public void pintar() { IO.println("[x] Casilla clara"); } }
record CasillaOscura() implements Casilla { public void pintar() { IO.println("[x] Casilla oscura"); } }

// 2. La fábrica abstracta: qué piezas se pueden pedir
interface FabricaInterfaz {
    Boton crearBoton();
    Casilla crearCasilla();
}

// 3. Una fábrica por familia
class FabricaClara implements FabricaInterfaz {
    public Boton crearBoton()     { return new BotonClaro(); }
    public Casilla crearCasilla() { return new CasillaClara(); }
}

class FabricaOscura implements FabricaInterfaz {
    public Boton crearBoton()     { return new BotonOscuro(); }
    public Casilla crearCasilla() { return new CasillaOscura(); }
}

// 4. Una pantalla: solo conoce la fábrica abstracta
class Formulario {
    private final Boton boton;
    private final Casilla casilla;

    Formulario(FabricaInterfaz fabrica) {
        this.boton = fabrica.crearBoton();
        this.casilla = fabrica.crearCasilla();
    }

    void mostrar() {
        casilla.pintar();
        boton.pintar();
    }
}

void main() {
    boolean modoOscuro = true;   // podría venir de las preferencias del usuario

    FabricaInterfaz fabrica = modoOscuro ? new FabricaOscura() : new FabricaClara();

    new Formulario(fabrica).mostrar();
}
```

Salida:

```text
[x] Casilla oscura
[ Botón oscuro ]
```

### Qué ha pasado

- La decisión «claro u oscuro» se toma **una sola vez**, al elegir la fábrica.
- `Formulario` no sabe qué familia está usando. Solo sabe pedir «un botón» y «una casilla», y recibe siempre piezas que combinan.
- Para añadir un tercer tema (por ejemplo, «alto contraste») basta con crear sus piezas y su fábrica. `Formulario` no cambia.

### Factory Method frente a Abstract Factory

| | Factory Method | Abstract Factory |
|---|---|---|
| ¿Qué crea? | **Un** objeto | Una **familia** de objetos relacionados |
| ¿Cómo? | Con un método | Con un objeto fábrica que tiene varios métodos |
| Ejemplo | «Dame una notificación por SMS» | «Dame todas las piezas del tema oscuro» |

### Cuándo usarlo

- Cuando tienes **varias familias** de objetos y las piezas de una familia deben ir siempre juntas: temas visuales, bases de datos distintas, versiones de un producto para distintos países…

### Cuándo **no** usarlo

- Si solo hay una familia, o las piezas no dependen unas de otras, es demasiada estructura. Un Factory Method o un `new` bastan.

---

## 8. Builder (constructor paso a paso)

### En una frase

Permite crear un objeto con muchos datos **paso a paso**, dando nombre a cada dato, en lugar de pasarlos todos de golpe a un constructor.

### Analogía

En una tienda de bocadillos a la carta vas indicando: «pan integral», «con pollo», «sin cebolla», «con salsa». Al final dices «ya está» y te dan el bocadillo. Cada paso tiene nombre, y te saltas los que no quieres.

### El problema

Un pedido tiene cliente, productos, dirección, si es urgente y unas notas. Con un constructor normal se escribe así:

```java
new Pedido("Ana", productos, "C/ Mayor 1", true, "");
```

¿Qué significa ese `true`? ¿Y la cadena vacía? Hay que ir a mirar el constructor para saberlo. Además, si el orden de dos `String` se cambia por error, el programa compila igual y guarda la dirección en las notas. Este problema se llama **constructor telescópico** (un constructor con demasiados parámetros).

### La solución paso a paso

1. El objeto final (`Pedido`) es un `record`: guarda los datos y no cambia.
2. Se crea una clase auxiliar, **`Builder`**, que va acumulando los datos.
3. `Builder` tiene un método por dato (`producto(...)`, `direccion(...)`, `urgente()`…). Cada método **devuelve el propio `Builder`**, para poder encadenar las llamadas.
4. Un método final, **`build()`**, comprueba que todo sea correcto y crea el `Pedido`.
5. Los datos opcionales tienen un **valor por defecto**, así que no hace falta indicarlos.

### Programa completo

```java
// Fichero: BuilderDemo.java
import java.util.ArrayList;
import java.util.List;

record Pedido(String cliente, List<String> productos, String direccion,
              boolean urgente, String notas) {

    // Punto de entrada: Pedido.builder("Ana")
    static Builder builder(String cliente) {
        return new Builder(cliente);
    }

    static class Builder {
        private final String cliente;
        private final List<String> productos = new ArrayList<>();
        private String direccion = "Recogida en tienda";   // valor por defecto
        private boolean urgente = false;                    // valor por defecto
        private String notas = "";                          // valor por defecto

        Builder(String cliente) {
            this.cliente = cliente;
        }

        Builder producto(String producto) {
            productos.add(producto);
            return this;   // devolver 'this' permite encadenar llamadas
        }

        Builder direccion(String direccion) {
            this.direccion = direccion;
            return this;
        }

        Builder urgente() {
            this.urgente = true;
            return this;
        }

        Builder notas(String notas) {
            this.notas = notas;
            return this;
        }

        Pedido build() {
            if (productos.isEmpty()) {
                throw new IllegalStateException("Un pedido necesita al menos un producto");
            }
            return new Pedido(cliente, List.copyOf(productos), direccion, urgente, notas);
        }
    }
}

void main() {
    Pedido completo = Pedido.builder("Ana")
            .producto("Teclado")
            .producto("Ratón")
            .direccion("C/ Mayor 1, Madrid")
            .urgente()
            .build();

    Pedido sencillo = Pedido.builder("Luis")
            .producto("Monitor")
            .build();   // usa los valores por defecto

    IO.println(completo);
    IO.println(sencillo);

    try {
        Pedido.builder("Eva").build();   // sin productos
    } catch (IllegalStateException e) {
        IO.println("Error: " + e.getMessage());
    }
}
```

Salida:

```text
Pedido[cliente=Ana, productos=[Teclado, Ratón], direccion=C/ Mayor 1, Madrid, urgente=true, notas=]
Pedido[cliente=Luis, productos=[Monitor], direccion=Recogida en tienda, urgente=false, notas=]
Error: Un pedido necesita al menos un producto
```

### Qué ha pasado

- Al leer `.urgente()` o `.direccion("...")` se entiende cada dato **sin mirar el constructor**.
- El pedido de Luis no indica dirección ni urgencia, y recibe los valores por defecto.
- `build()` es el lugar perfecto para **validar**: es imposible crear un pedido vacío.
- `List.copyOf(productos)` guarda una copia que no se puede modificar, para que nadie cambie los productos del pedido después de crearlo.

### Dónde lo verás en Java

- `StringBuilder`: construye un texto poco a poco con `append(...)` y lo termina con `toString()`.
- `HttpClient.newBuilder()` y `HttpRequest.newBuilder()`: configuran conexiones web paso a paso y terminan con `build()`.

```java
String saludo = new StringBuilder()
        .append("Hola, ")
        .append("Ana")
        .append("!")
        .toString();   // "Hola, Ana!"
```

### Cuándo usarlo

- Objetos con **muchos datos**, sobre todo si varios son **opcionales** o del mismo tipo (y por tanto fáciles de confundir).
- Cuando quieres **validar** el objeto antes de crearlo.

### Cuándo **no** usarlo

- Si el objeto tiene dos o tres datos obligatorios, un `record` con su constructor normal es suficiente: `new Punto(3, 4)` se entiende perfectamente.

---

## 9. Prototype (prototipo)

### En una frase

Crea objetos nuevos **copiando uno que ya existe** (el prototipo) y cambiando solo lo necesario.

### Analogía

Cuando tienes que enviar el mismo formulario a varias personas, rellenas **uno** con los datos comunes, haces **fotocopias** y en cada copia solo cambias el nombre. Es más rápido que rellenar cada formulario desde el principio.

### El problema

Una aplicación tiene una plantilla de informe con un título y una lista de secciones. Para cada informe nuevo quieres partir de la plantilla y añadirle secciones. Si «copias» así:

```java
Plantilla copia = original;   // ¡esto NO copia nada!
```

`copia` y `original` son **dos nombres para el mismo objeto**. Lo que añadas a la copia aparecerá también en el original.

### La solución: un constructor de copia

Se añade a la clase un constructor que recibe otro objeto del mismo tipo y copia sus datos. Hay que tener cuidado con los datos que son **objetos modificables**, como las listas: hay que crear **una lista nueva** con los mismos elementos. A esto se le llama **copia profunda**.

### Programa completo

```java
// Fichero: Prototype.java
import java.util.ArrayList;
import java.util.List;

class Plantilla {
    private final String titulo;
    private final List<String> secciones;

    Plantilla(String titulo, List<String> secciones) {
        this.titulo = titulo;
        this.secciones = new ArrayList<>(secciones);
    }

    // Constructor de copia: crea una lista NUEVA con los mismos elementos
    Plantilla(Plantilla otra) {
        this(otra.titulo, otra.secciones);
    }

    Plantilla copiar() {
        return new Plantilla(this);
    }

    void anadirSeccion(String seccion) {
        secciones.add(seccion);
    }

    @Override
    public String toString() {
        return titulo + " " + secciones;
    }
}

void main() {
    Plantilla original = new Plantilla("Informe mensual", List.of("Resumen", "Ventas"));

    // ❌ Esto no es una copia: las dos variables apuntan al mismo objeto
    Plantilla mismaCosa = original;
    mismaCosa.anadirSeccion("Gastos");
    IO.println("Original tras el error: " + original);

    // ✅ Copia de verdad
    Plantilla copia = original.copiar();
    copia.anadirSeccion("Previsiones");
    IO.println("Original: " + original);
    IO.println("Copia:    " + copia);
}
```

Salida:

```text
Original tras el error: Informe mensual [Resumen, Ventas, Gastos]
Original: Informe mensual [Resumen, Ventas, Gastos]
Copia:    Informe mensual [Resumen, Ventas, Gastos, Previsiones]
```

### Qué ha pasado

- `mismaCosa = original` **no crea ningún objeto**: solo da otro nombre al mismo. Por eso «Gastos» aparece en el original.
- `original.copiar()` crea un objeto nuevo **con su propia lista**. Lo que se añade a la copia («Previsiones») no afecta al original.

### Copia superficial y copia profunda

| Tipo de copia | Qué hace | Riesgo |
|---|---|---|
| **Superficial** | Copia el objeto, pero sus listas y objetos internos se **comparten** con el original. | Cambiar la lista de la copia cambia también la del original. |
| **Profunda** | Copia el objeto **y** crea copias nuevas de sus listas y objetos internos. | Ninguno de ese tipo; cada objeto es independiente. |

En el ejemplo, la línea `this.secciones = new ArrayList<>(secciones);` es la que convierte la copia en profunda.

### ¿Y el método `clone()`?

Java tiene un método `clone()` heredado de la clase `Object`, pensado para esto. **Evítalo**: hace copias superficiales por defecto, obliga a capturar excepciones y tiene reglas poco intuitivas. Un constructor de copia (o un método `copiar()`) es más claro.

### Un atajo: objetos que no cambian

Si un objeto **no se puede modificar** (por ejemplo, un `record` cuyos datos son textos y números), no hace falta copiarlo: se puede compartir sin peligro, porque nadie puede cambiarlo. Para «modificar» un dato se crea un objeto nuevo:

```java
record Punto(int x, int y) {
    Punto conX(int nuevoX) {
        return new Punto(nuevoX, y);   // objeto nuevo; el original no cambia
    }
}
```

### Cuándo usarlo

- Cuando crear un objeto desde cero es **costoso o largo**, y es más cómodo partir de uno ya preparado.
- Cuando tienes **plantillas** o configuraciones base que se repiten con pequeñas diferencias.

---

## 10. Resumen y cómo elegir

| Patrón | Problema que resuelve | Idea clave | En Java |
|---|---|---|---|
| **Singleton** | Debe existir un único objeto | Un solo objeto compartido | `enum` con un único valor |
| **Factory Method** | Elegir qué clase crear sin llenar el código de `if` | Un método decide qué objeto crear | Método `static` con `switch`; `List.of`, `LocalDate.of` |
| **Abstract Factory** | Crear piezas que deben combinar entre sí | Una fábrica por familia | Interfaz de fábrica con varios métodos |
| **Builder** | Constructores con demasiados parámetros | Construir paso a paso y terminar con `build()` | `record` + clase `Builder`; `StringBuilder` |
| **Prototype** | Crear objetos parecidos a uno existente | Copiar y modificar | Constructor de copia; objetos que no cambian |

### Preguntas para elegir

1. **¿Solo puede haber uno?** → Singleton (y piensa antes si basta con crear uno y pasarlo).
2. **¿Tengo que elegir entre varias clases según un dato?** → Factory Method.
3. **¿Tengo varias familias de piezas que deben ir juntas?** → Abstract Factory.
4. **¿El constructor tiene tantos parámetros que no se entiende?** → Builder.
5. **¿Quiero un objeto casi igual a otro que ya tengo?** → Prototype.
6. **¿Nada de lo anterior?** → Usa `new`. No hace falta ningún patrón.

---

## 11. Errores frecuentes

| Error | Por qué es un problema | Qué hacer |
|---|---|---|
| Usar Singleton para todo | Se convierte en una variable global: cualquiera lo cambia y cuesta encontrar errores. | Crear el objeto una vez y pasarlo por el constructor a quien lo necesite. |
| Usar textos (`"EMAIL"`) para elegir en una fábrica | Un error de escritura (`"Emial"`) no se detecta hasta ejecutar. | Usar un `enum`: el compilador detecta los errores. |
| Poner `default` en el `switch` de la fábrica | Si se añade un valor nuevo al `enum`, el compilador ya no avisa de que falta su `case`. | Cubrir todos los valores del `enum` sin `default`. |
| Crear un Builder para un objeto de dos datos | Mucho código para nada. | Usar un `record` con su constructor. |
| Pensar que `b = a` copia un objeto | Solo crea otro nombre para el mismo objeto. | Usar un constructor de copia. |
| Copiar una lista con `this.lista = otra.lista` | Las dos copias comparten la misma lista (copia superficial). | `this.lista = new ArrayList<>(otra.lista)`. |
| Aplicar un patrón «por si acaso» | Añade clases y complica el código sin necesidad. | Aplicarlo solo cuando aparece el problema que resuelve. |

---

## 12. Ejercicios

**Ejercicio 1 (Factory Method).** Añade un canal `WHATSAPP` al ejemplo de la sección 6. Comprueba primero qué ocurre si lo añades al `enum` pero no a la fábrica.

<details>
<summary>Solución</summary>

Al añadir `WHATSAPP` al `enum` sin tocar la fábrica, el programa **no compila**: el compilador indica que el `switch` no cubre todos los valores posibles. Para arreglarlo:

```java
record WhatsApp(String telefono) implements Notificacion {
    public void enviar(String mensaje) {
        IO.println("WhatsApp a " + telefono + ": " + mensaje);
    }
}

enum Canal { EMAIL, SMS, PUSH, WHATSAPP }

// En la fábrica:
case WHATSAPP -> new WhatsApp(destino);
```

</details>

**Ejercicio 2 (Builder).** Crea un `record Ordenador(String procesador, int memoriaGb, int discoGb, boolean tarjetaGrafica)` con un `Builder`. La memoria por defecto es 16 GB y el disco 512 GB. `build()` debe lanzar una excepción si no se ha indicado procesador.

<details>
<summary>Solución</summary>

```java
record Ordenador(String procesador, int memoriaGb, int discoGb, boolean tarjetaGrafica) {

    static Builder builder() { return new Builder(); }

    static class Builder {
        private String procesador;
        private int memoriaGb = 16;
        private int discoGb = 512;
        private boolean tarjetaGrafica = false;

        Builder procesador(String p)  { this.procesador = p; return this; }
        Builder memoriaGb(int gb)     { this.memoriaGb = gb; return this; }
        Builder discoGb(int gb)       { this.discoGb = gb; return this; }
        Builder conTarjetaGrafica()   { this.tarjetaGrafica = true; return this; }

        Ordenador build() {
            if (procesador == null) {
                throw new IllegalStateException("Falta el procesador");
            }
            return new Ordenador(procesador, memoriaGb, discoGb, tarjetaGrafica);
        }
    }
}

void main() {
    IO.println(Ordenador.builder().procesador("Ryzen 7").conTarjetaGrafica().build());
    // Ordenador[procesador=Ryzen 7, memoriaGb=16, discoGb=512, tarjetaGrafica=true]
}
```

</details>

**Ejercicio 3 (Prototype).** En el ejemplo de la sección 9, sustituye el constructor de copia por este:

```java
Plantilla(Plantilla otra) {
    this.titulo = otra.titulo;
    this.secciones = otra.secciones;   // ¿copia la lista?
}
```

Ejecuta el programa. ¿Qué muestra ahora la línea `Original:` y por qué?

<details>
<summary>Solución</summary>

Muestra `Original: Informe mensual [Resumen, Ventas, Gastos, Previsiones]`. La copia y el original **comparten la misma lista**, así que «Previsiones» aparece también en el original. Es una copia superficial. La solución es crear siempre una lista nueva: `new ArrayList<>(otra.secciones)`.

</details>

---

## 13. Glosario

| Término | Significado |
|---|---|
| **Patrón de diseño** | Solución conocida y con nombre para un problema de diseño que se repite. |
| **GoF (*Gang of Four*)** | Los cuatro autores del libro *Design Patterns* (1994), que dio nombre a los patrones clásicos. |
| **Fábrica** | Pieza de código (método u objeto) cuya única tarea es crear otros objetos. |
| **Familia de objetos** | Grupo de objetos pensados para usarse juntos (por ejemplo, todas las piezas del tema oscuro). |
| **Constructor telescópico** | Constructor con tantos parámetros que es difícil saber qué es cada uno. |
| **Encadenar llamadas** | Escribir `objeto.a().b().c()`: es posible cuando cada método devuelve el propio objeto. |
| **Copia superficial** | Copia que comparte con el original las listas y objetos internos. |
| **Copia profunda** | Copia que también duplica las listas y objetos internos. |
| **Variable global** | Dato accesible y modificable desde cualquier parte del programa. |
| **Compilador** | Programa que traduce tu código a algo que la máquina puede ejecutar, y que detecta muchos errores antes de ejecutarlo. |
