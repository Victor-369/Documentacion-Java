# Patrones de diseño de comportamiento en Java

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los programas completos de este documento se han compilado y ejecutado con OpenJDK 25.
>
> Documentos relacionados: [Patrones creacionales](<Patrones creacionales.md>) · [Patrones estructurales](<Patrones estructurales.md>)

## Índice

1. [¿Qué es un patrón de diseño?](#1-qué-es-un-patrón-de-diseño)
2. [Antes de empezar: cómo probar los ejemplos](#2-antes-de-empezar-cómo-probar-los-ejemplos)
3. [Vocabulario mínimo](#3-vocabulario-mínimo)
4. [Lambdas en cinco minutos](#4-lambdas-en-cinco-minutos)
5. [¿Qué problema resuelven los patrones de comportamiento?](#5-qué-problema-resuelven-los-patrones-de-comportamiento)
6. [Strategy (estrategia)](#6-strategy-estrategia)
7. [Observer (observador)](#7-observer-observador)
8. [Command (orden)](#8-command-orden)
9. [State (estado)](#9-state-estado)
10. [Template Method (método plantilla)](#10-template-method-método-plantilla)
11. [Iterator (iterador)](#11-iterator-iterador)
12. [Resumen y cómo elegir](#12-resumen-y-cómo-elegir)
13. [Errores frecuentes](#13-errores-frecuentes)
14. [Ejercicios](#14-ejercicios)
15. [Glosario](#15-glosario)

---

## 1. ¿Qué es un patrón de diseño?

Un **patrón de diseño** es una **solución ya probada** para un problema que aparece una y otra vez al programar. Tiene un nombre, explica qué problema resuelve y describe cómo organizar las clases para resolverlo.

> **Analogía:** un patrón es como una **receta de cocina**. No es tu plato concreto, sino unos pasos que adaptas a lo que tienes. Por eso un patrón no se copia y pega: se entiende la idea y se aplica a tu programa.

Los patrones clásicos se recogieron en el libro *Design Patterns* (1994), de cuatro autores conocidos como *Gang of Four* (GoF). Se agrupan según para qué sirven:

| Grupo | Para qué sirven | Ejemplos |
|---|---|---|
| **Creacionales** | **Crear objetos** de forma ordenada y flexible | Singleton, Factory Method, Builder |
| **Estructurales** | **Combinar** clases y objetos para formar estructuras más grandes | Adapter, Composite, Decorator |
| **De comportamiento** (este documento) | Organizar **cómo se comunican** los objetos y quién hace cada cosa | Strategy, Observer, Command, State, Template Method, Iterator |

### ¿Por qué merece la pena aprenderlos?

- **No reinventas la rueda:** otras personas ya pensaron una buena solución.
- **Hablas el mismo idioma que tu equipo:** decir «aquí usamos un *Strategy*» se entiende al momento, sin explicar todo el código.
- **Reconoces el código que lees:** muchas librerías de Java usan estos patrones. Si los conoces, entiendes antes cómo funcionan.
- **Tu código aguanta mejor los cambios:** cada pieza tiene un papel claro, así que añadir algo nuevo no obliga a reescribir lo que ya funciona.

### Patrones y principios SOLID

**SOLID** son cinco principios para escribir código orientado a objetos fácil de mantener. **No necesitas conocerlos para seguir este documento**, pero si ya los conoces verás que muchos patrones son, en la práctica, una forma concreta de aplicarlos. Los tienes explicados en [Principios SOLID](<../Principios/Principio SOLID.md>).

| Patrón | Principio que refuerza | Por qué |
|---|---|---|
| **Strategy** | Abierto/cerrado (OCP) | Para añadir una forma nueva de hacer algo se crea otra estrategia, sin tocar el código existente ni alargar un `if`. |
| **Observer** | Inversión de dependencias (DIP) | Quien avisa solo conoce una interfaz, no las clases concretas que reciben el aviso. |
| **Command** | Responsabilidad única (SRP) | Cada acción vive en su propio objeto, separada de quien la lanza. |
| **State** | Abierto/cerrado (OCP) y responsabilidad única (SRP) | Cada estado es una clase con su propio comportamiento; desaparecen los `if (estado == ...)` repartidos por el código. |
| **Template Method** | Abierto/cerrado (OCP) | Los pasos fijos del algoritmo no cambian; las subclases solo rellenan los pasos variables. |

---

## 2. Antes de empezar: cómo probar los ejemplos

Cada patrón incluye un **programa completo**. Para ejecutarlo:

1. Instala un JDK 25 o superior (comprueba la versión con `java -version`).
2. Copia el código en un fichero `.java`, por ejemplo `Strategy.java`.
3. Ejecútalo desde la terminal, en la misma carpeta:

```bash
java Strategy.java
```

Los ejemplos usan dos facilidades de **Java 25** para programas pequeños:

- **`void main()`**: el punto de inicio del programa, sin necesidad de escribir `public static void main(String[] args)` ni una clase alrededor.
- **`IO.println(...)`**: escribe un texto en la consola (equivale a `System.out.println(...)`).

> En un proyecto real, cada clase iría en su propio fichero. Las ideas de los patrones no cambian.

---

## 3. Vocabulario mínimo

| Término | Qué significa | Ejemplo |
|---|---|---|
| **Clase** | El «molde» que describe cómo son unos objetos. | `class Coche { ... }` |
| **Objeto** | Una cosa concreta creada a partir de una clase con `new`. | `new Coche()` |
| **Interfaz** | Lista de métodos que una clase **promete** tener, sin decir cómo los hace. | `interface Vehiculo { void arrancar(); }` |
| **`implements`** | Indica que una clase cumple una interfaz. | `class Coche implements Vehiculo` |
| **Clase abstracta** | Clase que no se puede crear con `new` y que deja algunos métodos sin escribir (`abstract`) para que los completen sus clases hijas. | `abstract class Bebida { ... }` |
| **`extends`** | Indica que una clase hereda de otra (es su «hija»). | `class Te extends Bebida` |
| **`final`** (en un método) | El método no se puede cambiar en las clases hijas. | `final void preparar()` |
| **`enum`** | Tipo con una lista **cerrada** de valores. | `enum Color { ROJO, VERDE }` |
| **`record`** | Forma corta de crear una clase que solo guarda datos que no cambian. | `record Punto(int x, int y) {}` |
| **`switch` con flechas** | Elige un resultado según un valor. Con `->` no hace falta `break`. | `case ROJO -> "parar";` |
| **`List`** / **`ArrayList`** | Lista de elementos que puede crecer. | `new ArrayList<String>()` |
| **Pila** (`Deque`) | Colección en la que el último que entra es el primero que sale, como una pila de platos. | `pila.push(x)`, `pila.pop()` |

---

## 4. Lambdas en cinco minutos

Varios patrones de este documento se pueden escribir de forma muy corta con **lambdas**. Esta sección explica lo justo para entenderlas.

Una **interfaz funcional** es una interfaz con **un único método por implementar**:

```java
interface Saludo {
    String saludar(String nombre);
}
```

La forma tradicional de usarla es crear una clase que la implemente:

```java
class SaludoFormal implements Saludo {
    public String saludar(String nombre) {
        return "Buenos días, " + nombre;
    }
}

Saludo s = new SaludoFormal();
```

Una **lambda** es una forma abreviada de escribir **solo el método**, sin crear la clase:

```java
Saludo s = nombre -> "Buenos días, " + nombre;
//         ──┬───    ─────────┬────────────
//     parámetro    lo que devuelve el método
```

Se lee: «dado un `nombre`, devuelve `"Buenos días, " + nombre`». Ambas versiones se usan igual: `s.saludar("Ana")` devuelve `"Buenos días, Ana"`.

| Forma de la lambda | Significado |
|---|---|
| `x -> x * 2` | Un parámetro, devuelve un valor |
| `(a, b) -> a + b` | Dos parámetros |
| `() -> IO.println("Hola")` | Sin parámetros |
| `texto -> { IO.println(texto); }` | Con llaves, para escribir varias instrucciones |

> Si la interfaz tiene **más de un método** por implementar, no se puede usar una lambda: hay que escribir una clase.

---

## 5. ¿Qué problema resuelven los patrones de comportamiento?

Cuando un programa crece, aparecen preguntas como estas:

- «Tengo un `if` enorme que elige entre varias formas de hacer lo mismo».
- «Cuando pasa algo aquí, tienen que enterarse otras cinco partes del programa».
- «Quiero poder deshacer la última acción».
- «Este objeto se comporta de forma distinta según su situación, y el código está lleno de `if (estado == ...)`».

Los patrones de comportamiento responden a estas preguntas. Todos buscan lo mismo: que cada objeto haga **una tarea clara** y que los objetos puedan colaborar **sin conocer los detalles** unos de otros.

---

## 6. Strategy (estrategia)

### En una frase

Guarda cada forma de hacer una tarea en su propia pieza, para poder **cambiar de una a otra** fácilmente.

### Analogía

Para llegar al aeropuerto puedes ir en coche, en autobús o en taxi. El objetivo es el mismo (llegar), pero la **estrategia** cambia según el tiempo y el dinero que tengas. Un navegador GPS te deja elegir y recalcula sin que cambie nada más.

### El problema

Una tienda calcula los gastos de envío según el método elegido:

```java
int calcularEnvio(String metodo, int pesoKg) {
    if (metodo.equals("estandar")) {
        return 4 + pesoKg;
    } else if (metodo.equals("urgente")) {
        return 10 + pesoKg * 2;
    } else if (metodo.equals("recogida")) {
        return 0;
    }
    ...
}
```

Cada método nuevo (por ejemplo, «envío internacional») obliga a **modificar** este bloque, y el `if` no deja de crecer. Además, todas las formas de calcular están mezcladas en el mismo sitio.

### La solución paso a paso

1. Define una **interfaz** con la tarea: `EstrategiaEnvio`, con el método `coste(int pesoKg)`.
2. Escribe **una implementación por forma de calcular**.
3. La clase que usa el cálculo (`Pedido`) **guarda una estrategia** y le pide el coste, sin saber cuál es.
4. Para cambiar de método, se le da **otra estrategia**.

### Programa completo

```java
// Fichero: Strategy.java

// 1. La tarea que puede hacerse de varias formas
interface EstrategiaEnvio {
    int coste(int pesoKg);
}

// 2. Una clase por estrategia
class EnvioEstandar implements EstrategiaEnvio {
    public int coste(int pesoKg) { return 4 + pesoKg; }
}

class EnvioUrgente implements EstrategiaEnvio {
    public int coste(int pesoKg) { return 10 + pesoKg * 2; }
}

class RecogidaEnTienda implements EstrategiaEnvio {
    public int coste(int pesoKg) { return 0; }
}

// 3. Quien usa la estrategia no sabe cuál es
class Pedido {
    private final int pesoKg;
    private EstrategiaEnvio envio = new EnvioEstandar();   // estrategia por defecto

    Pedido(int pesoKg) {
        this.pesoKg = pesoKg;
    }

    void cambiarEnvio(EstrategiaEnvio envio) {
        this.envio = envio;
    }

    int costeEnvio() {
        return envio.coste(pesoKg);
    }
}

void main() {
    Pedido pedido = new Pedido(3);

    IO.println("Estándar: " + pedido.costeEnvio() + " €");

    pedido.cambiarEnvio(new EnvioUrgente());
    IO.println("Urgente: " + pedido.costeEnvio() + " €");

    pedido.cambiarEnvio(new RecogidaEnTienda());
    IO.println("Recogida: " + pedido.costeEnvio() + " €");

    // 4. Con una lambda: una estrategia nueva sin crear ninguna clase
    pedido.cambiarEnvio(peso -> 15 + peso * 3);   // envío internacional
    IO.println("Internacional: " + pedido.costeEnvio() + " €");
}
```

Salida:

```text
Estándar: 7 €
Urgente: 16 €
Recogida: 0 €
Internacional: 24 €
```

### Qué ha pasado

- `Pedido` no contiene ningún `if` sobre el método de envío. Solo dice «estrategia, dame el coste».
- Cambiar de método es tan sencillo como pasar otro objeto a `cambiarEnvio`.
- Como `EstrategiaEnvio` tiene un único método, también se puede crear una estrategia con una **lambda**, como el envío internacional: `peso -> 15 + peso * 3`.

### Dónde lo verás en Java

`Comparator` es una estrategia de **ordenación**: la lista sabe ordenarse, y tú le dices **con qué criterio**.

```java
List<String> nombres = new ArrayList<>(List.of("Lucía", "Ana", "Bernardo"));

nombres.sort(Comparator.naturalOrder());               // [Ana, Bernardo, Lucía]
nombres.sort(Comparator.comparing(String::length));    // [Ana, Lucía, Bernardo]
```

`String::length` es una forma aún más corta de escribir la lambda `texto -> texto.length()`. Se llama **referencia a método**.

### Cuándo usarlo

- Cuando hay **varias formas de hacer lo mismo** (calcular, ordenar, validar, exportar…) y quieres elegir una o cambiarla mientras el programa se ejecuta.
- Cuando un `if` o `switch` que elige «cómo hacer algo» no para de crecer.

---

## 7. Observer (observador)

### En una frase

Cuando un objeto cambia, **avisa automáticamente** a todos los que se han apuntado para enterarse.

### Analogía

Te **suscribes** a un canal de vídeos. Cada vez que el canal publica algo, te llega un aviso. El canal no sabe quién eres ni qué harás con el aviso: solo tiene una lista de suscriptores y avisa a todos. Puedes darte de baja cuando quieras.

### El problema

Cuando una tienda publica una oferta, hay que enviar un correo, mostrar un aviso en la aplicación del móvil y guardar un registro. Si la clase `Tienda` llama directamente a esas tres cosas, **cada vez que se quiera añadir o quitar un aviso habrá que modificar `Tienda`**, y esta clase acabará dependiendo de medio programa.

### La solución paso a paso

1. Define una interfaz para los que quieren enterarse: `Suscriptor`, con el método `recibir(String noticia)`.
2. El objeto que cambia (el **sujeto**, aquí `Tienda`) guarda una **lista de suscriptores**.
3. El sujeto ofrece métodos para **apuntarse** (`suscribir`) y **darse de baja** (`darDeBaja`).
4. Cuando ocurre algo, el sujeto **recorre la lista** y llama a `recibir` en cada suscriptor.

### Programa completo

```java
// Fichero: Observer.java
import java.util.ArrayList;
import java.util.List;

// 1. Los que quieren enterarse
interface Suscriptor {
    void recibir(String noticia);
}

// 2 y 3. El sujeto: guarda la lista y permite apuntarse y darse de baja
class Tienda {
    private final List<Suscriptor> suscriptores = new ArrayList<>();

    void suscribir(Suscriptor suscriptor) {
        suscriptores.add(suscriptor);
    }

    void darDeBaja(Suscriptor suscriptor) {
        suscriptores.remove(suscriptor);
    }

    // 4. Avisa a todos
    void publicarOferta(String oferta) {
        IO.println("Tienda publica: " + oferta);
        for (Suscriptor suscriptor : suscriptores) {
            suscriptor.recibir(oferta);
        }
    }
}

// Suscriptores concretos
class AvisoPorEmail implements Suscriptor {
    private final String direccion;

    AvisoPorEmail(String direccion) {
        this.direccion = direccion;
    }

    public void recibir(String noticia) {
        IO.println("  Email a " + direccion + ": " + noticia);
    }
}

class AvisoEnApp implements Suscriptor {
    public void recibir(String noticia) {
        IO.println("  Aviso en la app: " + noticia);
    }
}

void main() {
    Tienda tienda = new Tienda();

    Suscriptor email = new AvisoPorEmail("ana@ejemplo.es");
    tienda.suscribir(email);
    tienda.suscribir(new AvisoEnApp());
    tienda.suscribir(noticia -> IO.println("  Registro: guardada \"" + noticia + "\""));   // con una lambda

    tienda.publicarOferta("20 % en portátiles");

    tienda.darDeBaja(email);
    tienda.publicarOferta("Envío gratis este fin de semana");
}
```

Salida:

```text
Tienda publica: 20 % en portátiles
  Email a ana@ejemplo.es: 20 % en portátiles
  Aviso en la app: 20 % en portátiles
  Registro: guardada "20 % en portátiles"
Tienda publica: Envío gratis este fin de semana
  Aviso en la app: Envío gratis este fin de semana
  Registro: guardada "Envío gratis este fin de semana"
```

### Qué ha pasado

- `Tienda` no sabe qué hace cada suscriptor: solo sabe que todos tienen el método `recibir`.
- Se pueden añadir suscriptores nuevos (incluso con una lambda, como el registro) **sin tocar `Tienda`**.
- Tras darse de baja, el correo de Ana ya no recibe la segunda oferta.

### Puntos importantes

- **Ofrece siempre una forma de darse de baja.** Si un objeto se suscribe y nunca se borra de la lista, Java no puede liberar su memoria aunque ya no se use (a esto se le llama **fuga de memoria**).
- **El orden de aviso** es el orden de la lista. No conviene que un suscriptor dependa de que otro haya recibido el aviso antes.
- En Java existían las clases `java.util.Observable` y `java.util.Observer`, pero están **obsoletas** desde Java 9. No las uses: escribe tu propia interfaz, como en el ejemplo.

### Dónde lo verás

- En las interfaces gráficas: un botón avisa a sus «escuchadores» (*listeners*) cuando lo pulsan.
- En los sistemas de eventos de los *frameworks* (por ejemplo, Spring) y en las librerías de programación reactiva.

### Cuándo usarlo

- Cuando un cambio en un objeto debe provocar reacciones en **otros objetos que no quieres que conozca directamente**.

---

## 8. Command (orden)

### En una frase

Convierte una **acción** en un **objeto**, para poder guardarla, ponerla en cola, repetirla o **deshacerla**.

### Analogía

En un restaurante, el camarero apunta tu pedido en una **comanda** (un papel) y la deja en la cocina. La comanda es la petición convertida en objeto: se puede poner en cola, pasar de una persona a otra o tachar si cancelas.

### El problema

Un editor de texto tiene que permitir **deshacer** (Ctrl+Z). Si cada acción se ejecuta directamente («añade este texto»), después no queda ningún rastro de lo que se hizo, y no hay forma de deshacerlo.

### La solución paso a paso

1. Define una interfaz `Comando` con dos métodos: `ejecutar()` y `deshacer()`.
2. Crea una clase por tipo de acción (`InsertarTexto`…). Cada una guarda **los datos necesarios** para hacerse y deshacerse.
3. Un **invocador** (`Editor`) ejecuta los comandos y los guarda en un **historial** (una pila).
4. Para deshacer, el editor saca el **último** comando de la pila y llama a su `deshacer()`.

### Programa completo

```java
// Fichero: Command.java
import java.util.ArrayDeque;
import java.util.Deque;

// El objeto sobre el que se actúa
class Documento {
    private final StringBuilder texto = new StringBuilder();

    void insertar(String fragmento)  { texto.append(fragmento); }
    void borrarUltimos(int cantidad) { texto.setLength(texto.length() - cantidad); }
    String contenido()               { return texto.toString(); }
}

// 1. Una acción convertida en objeto
interface Comando {
    void ejecutar();
    void deshacer();
}

// 2. Un comando concreto: guarda lo necesario para hacerse y deshacerse
record InsertarTexto(Documento documento, String texto) implements Comando {
    public void ejecutar() { documento.insertar(texto); }
    public void deshacer() { documento.borrarUltimos(texto.length()); }
}

// 3. El invocador: ejecuta y recuerda
class Editor {
    private final Deque<Comando> historial = new ArrayDeque<>();

    void ejecutar(Comando comando) {
        comando.ejecutar();
        historial.push(comando);   // se apila
    }

    // 4. Deshacer: el último que entró es el primero que sale
    void deshacer() {
        if (!historial.isEmpty()) {
            historial.pop().deshacer();
        }
    }
}

void main() {
    Documento documento = new Documento();
    Editor editor = new Editor();

    editor.ejecutar(new InsertarTexto(documento, "Hola"));
    editor.ejecutar(new InsertarTexto(documento, ", mundo"));
    editor.ejecutar(new InsertarTexto(documento, "!!!"));
    IO.println("Texto: " + documento.contenido());

    editor.deshacer();
    IO.println("Tras deshacer: " + documento.contenido());

    editor.deshacer();
    IO.println("Tras deshacer otra vez: " + documento.contenido());
}
```

Salida:

```text
Texto: Hola, mundo!!!
Tras deshacer: Hola, mundo
Tras deshacer otra vez: Hola
```

### Qué ha pasado

- Cada acción es un objeto `InsertarTexto` que **recuerda qué texto añadió**. Por eso sabe cuántos caracteres borrar para deshacerse.
- El historial es una **pila**: `push` pone un comando encima y `pop` saca el de arriba. Así se deshace siempre la **última** acción, como con Ctrl+Z.
- Añadir una acción nueva (por ejemplo, «poner en mayúsculas») consiste en escribir otro `Comando`. `Editor` no cambia.

### Comandos sencillos: `Runnable`

Si solo necesitas guardar una acción para **ejecutarla más tarde** (sin deshacer), Java ya trae una interfaz para eso: `Runnable`, con un único método `run()`. Se puede crear con una lambda:

```java
List<Runnable> tareas = List.of(
        () -> IO.println("Enviar correo de bienvenida"),
        () -> IO.println("Generar factura"));

for (Runnable tarea : tareas) {
    tarea.run();
}
```

Así funcionan, por ejemplo, las colas de tareas que se ejecutan en segundo plano.

### Cuándo usarlo

- Para **deshacer y rehacer**.
- Para **guardar acciones** y ejecutarlas después: colas de tareas, tareas programadas, macros.
- Para **registrar** todo lo que se ha hecho (un historial de operaciones).

---

## 9. State (estado)

### En una frase

Un objeto **cambia su comportamiento según la situación (estado) en la que está**, y cada estado sabe qué se puede hacer y cuál es el siguiente.

### Analogía

Un **semáforo**: en rojo, los coches paran; en verde, pasan; en ámbar, frenan. El semáforo es el mismo, pero lo que «significa» y cuál es su siguiente color dependen de su estado actual.

### El problema

Un pedido pasa por varias fases: pendiente, pagado, enviado, entregado o cancelado. Sin el patrón, cada acción se llena de comprobaciones:

```java
void enviar() {
    if (estado.equals("PAGADO")) {
        estado = "ENVIADO";
    } else if (estado.equals("PENDIENTE")) {
        throw new IllegalStateException("Aún no está pagado");
    } else if (estado.equals("CANCELADO")) {
        ...
    }
}
// ... y lo mismo en pagar(), cancelar(), entregar()...
```

Las reglas de cada estado quedan **repartidas** por todos los métodos, y es fácil olvidar un caso o escribir mal un texto (`"PAGDO"`).

### La solución paso a paso

1. Define los estados posibles con un **`enum`**: así no puede haber estados mal escritos.
2. Da al `enum` un método por cada acción (`pagar()`, `enviar()`, `cancelar()`…). Cada método devuelve **el estado siguiente** o lanza un error si la acción no está permitida.
3. La clase `Pedido` guarda su estado actual y **delega** en él: `estado = estado.pagar();`.

Así, todas las reglas de «qué se puede hacer en cada fase» están **en un único sitio**.

### Programa completo

```java
// Fichero: State.java

// 1 y 2. Los estados y sus reglas, todo junto
enum EstadoPedido {
    PENDIENTE, PAGADO, ENVIADO, ENTREGADO, CANCELADO;

    EstadoPedido pagar() {
        return switch (this) {
            case PENDIENTE -> PAGADO;
            default -> throw new IllegalStateException("No se puede pagar un pedido " + this);
        };
    }

    EstadoPedido enviar() {
        return switch (this) {
            case PAGADO -> ENVIADO;
            default -> throw new IllegalStateException("No se puede enviar un pedido " + this);
        };
    }

    EstadoPedido entregar() {
        return switch (this) {
            case ENVIADO -> ENTREGADO;
            default -> throw new IllegalStateException("No se puede entregar un pedido " + this);
        };
    }

    EstadoPedido cancelar() {
        return switch (this) {
            case PENDIENTE, PAGADO -> CANCELADO;
            case ENVIADO, ENTREGADO -> throw new IllegalStateException("Ya ha salido: hay que pedir una devolución");
            case CANCELADO -> CANCELADO;   // cancelar dos veces no cambia nada
        };
    }
}

// 3. El pedido delega en su estado
class Pedido {
    private EstadoPedido estado = EstadoPedido.PENDIENTE;

    void pagar()    { estado = estado.pagar(); }
    void enviar()   { estado = estado.enviar(); }
    void entregar() { estado = estado.entregar(); }
    void cancelar() { estado = estado.cancelar(); }

    EstadoPedido estado() { return estado; }
}

void main() {
    Pedido pedido = new Pedido();
    IO.println("Estado inicial: " + pedido.estado());

    pedido.pagar();
    pedido.enviar();
    IO.println("Tras pagar y enviar: " + pedido.estado());

    try {
        pedido.cancelar();
    } catch (IllegalStateException e) {
        IO.println("Error: " + e.getMessage());
    }

    Pedido otro = new Pedido();
    try {
        otro.enviar();   // sin pagar
    } catch (IllegalStateException e) {
        IO.println("Error: " + e.getMessage());
    }
}
```

Salida:

```text
Estado inicial: PENDIENTE
Tras pagar y enviar: ENVIADO
Error: Ya ha salido: hay que pedir una devolución
Error: No se puede enviar un pedido PENDIENTE
```

### Qué ha pasado

- Las reglas del ciclo de vida están **todas en `EstadoPedido`**. Para saber qué se puede hacer con un pedido enviado, basta con mirar el `enum`.
- `Pedido` no tiene ni un `if`: solo pasa la petición a su estado y guarda el estado que le devuelven.
- `switch (this)` elige según el valor actual del `enum`. En `cancelar()` no hay `default`: se han escrito **todos** los estados, así que si mañana se añade uno nuevo (por ejemplo, `DEVUELTO`), **el compilador avisará** de que falta decidir qué pasa al cancelarlo.

### Diagrama de estados

```text
            pagar()          enviar()          entregar()
PENDIENTE ──────────▶ PAGADO ─────────▶ ENVIADO ──────────▶ ENTREGADO
    │                   │
    │ cancelar()        │ cancelar()
    ▼                   ▼
CANCELADO ◀─────────────┘
```

### La versión clásica

En el libro original, cada estado es una **clase** distinta que implementa una interfaz común (`EstadoPedido` con los métodos `pagar()`, `enviar()`…). Esa forma es mejor cuando cada estado tiene **mucho comportamiento propio** o necesita guardar datos (por ejemplo, el estado «enviado» con su número de seguimiento). Con estados sencillos, como aquí, el `enum` es más corto y claro.

### Cuándo usarlo

- Cuando un objeto tiene un **ciclo de vida** con reglas distintas en cada fase: pedidos, reservas, semáforos, máquinas expendedoras, procesos de aprobación…
- Cuando ves muchos `if (estado == ...)` repetidos en varios métodos.

---

## 10. Template Method (método plantilla)

### En una frase

Define **los pasos de un proceso en un orden fijo** y deja que las clases hijas decidan cómo se hacen algunos de esos pasos.

### Analogía

Preparar una bebida caliente siempre sigue los mismos pasos: **hervir agua**, **echar el ingrediente**, **servir en una taza** y, si se quiere, **añadir extras**. Para un té o un café cambian el ingrediente y los extras, pero el orden de los pasos es siempre el mismo.

### El problema

Si `Te` y `Cafe` se escriben por separado, ambas repiten «hervir agua» y «servir en taza». Si mañana se cambia cómo se sirve, hay que cambiarlo en las dos, y alguna podría acabar haciendo los pasos en otro orden.

### La solución paso a paso

1. Crea una **clase abstracta** (`Bebida`) con un método que define el orden de los pasos: `preparar()`. Ese es el **método plantilla**.
2. Márcalo como **`final`** para que ninguna clase hija pueda cambiar el orden.
3. Los pasos **comunes** se escriben en la clase abstracta (`hervirAgua()`, `servir()`).
4. Los pasos que **cambian** se declaran `abstract` (`echarIngrediente()`), y cada clase hija los escribe.
5. Los pasos **opcionales** tienen una versión por defecto que no hace nada; las hijas pueden sobrescribirla si quieren. A estos se les llama **ganchos** (*hooks*).

### Programa completo

```java
// Fichero: TemplateMethod.java

abstract class Bebida {

    // 1 y 2. El método plantilla: orden fijo, no se puede cambiar
    final void preparar() {
        IO.println("Preparando " + nombre() + ":");
        hervirAgua();
        echarIngrediente();
        servir();
        anadirExtras();
    }

    // 3. Pasos comunes
    private void hervirAgua() { IO.println("  - Hervir agua"); }
    private void servir()     { IO.println("  - Servir en una taza"); }

    // 4. Pasos que cada bebida debe definir
    abstract String nombre();
    abstract void echarIngrediente();

    // 5. Gancho: opcional, por defecto no hace nada
    void anadirExtras() { }
}

class Te extends Bebida {
    String nombre()         { return "té"; }
    void echarIngrediente() { IO.println("  - Dejar reposar la bolsita de té"); }
    @Override
    void anadirExtras()     { IO.println("  - Añadir una rodaja de limón"); }
}

class Cafe extends Bebida {
    String nombre()         { return "café"; }
    void echarIngrediente() { IO.println("  - Pasar el agua por el café molido"); }
    // no sobrescribe anadirExtras(): sin extras
}

void main() {
    new Te().preparar();
    new Cafe().preparar();
}
```

Salida:

```text
Preparando té:
  - Hervir agua
  - Dejar reposar la bolsita de té
  - Servir en una taza
  - Añadir una rodaja de limón
Preparando café:
  - Hervir agua
  - Pasar el agua por el café molido
  - Servir en una taza
```

### Qué ha pasado

- `preparar()` está escrito **una sola vez** y garantiza que los pasos siempre van en el mismo orden.
- `Te` y `Cafe` solo escriben **lo que las diferencia**.
- `Cafe` no escribe `anadirExtras()`, así que usa la versión por defecto, que no hace nada.
- `hervirAgua()` y `servir()` son `private`: las hijas no pueden cambiarlos.

### Cuidado

Este patrón se basa en la **herencia**, y una clase solo puede heredar de **una** clase padre. Si los pasos que cambian son pocos y sencillos, a veces es mejor **pasarlos como estrategias** (ver el patrón Strategy) en lugar de crear una clase hija para cada variante.

### Dónde lo verás

- Muchos *frameworks* funcionan así: el *framework* controla el orden general (el método plantilla) y tú rellenas los pasos concretos. Por ejemplo, en las pruebas con JUnit, el *framework* decide cuándo se ejecuta cada método preparado con `@BeforeEach`, `@Test` y `@AfterEach`.

### Cuándo usarlo

- Cuando varias clases siguen **el mismo proceso** y solo cambian algunos pasos.

---

## 11. Iterator (iterador)

### En una frase

Permite **recorrer** los elementos de una colección uno a uno **sin saber cómo está guardada por dentro**.

### Analogía

El mando de la tele tiene un botón de «canal siguiente». No necesitas saber cómo están guardados los canales en el televisor: solo pulsas «siguiente» hasta que no hay más.

### En Java ya está incluido

Java tiene este patrón integrado en el lenguaje. Cualquier clase que implemente la interfaz **`Iterable`** se puede recorrer con un bucle *for-each* (`for (Tipo x : coleccion)`). Las listas, los conjuntos y muchas otras colecciones ya lo hacen:

```java
List<String> nombres = List.of("Ana", "Luis", "Eva");

for (String nombre : nombres) {   // por dentro usa un iterador
    IO.println(nombre);
}
```

### Programa completo: hacer recorrible tu propia clase

```java
// Fichero: IteratorDemo.java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

// Una lista de reproducción que se puede recorrer con for-each
class ListaReproduccion implements Iterable<String> {
    private final List<String> canciones = new ArrayList<>();

    void anadir(String cancion) {
        canciones.add(cancion);
    }

    @Override
    public Iterator<String> iterator() {
        return canciones.iterator();   // quien recorre no sabe que dentro hay un ArrayList
    }
}

void main() {
    ListaReproduccion lista = new ListaReproduccion();
    lista.anadir("Canción A");
    lista.anadir("Canción B");
    lista.anadir("Canción C");

    for (String cancion : lista) {
        IO.println("Sonando: " + cancion);
    }
}
```

Salida:

```text
Sonando: Canción A
Sonando: Canción B
Sonando: Canción C
```

### Qué ha pasado

- Al implementar `Iterable`, `ListaReproduccion` se puede usar en un *for-each* como cualquier lista.
- Quien la recorre **no sabe** que por dentro hay un `ArrayList`. Si mañana se guarda de otra forma, el bucle sigue funcionando.
- Para recorridos más complejos (filtrar, transformar, contar…) Java ofrece la API **Stream**, que se explica en [Lambdas y Streams](<../../02-Programacion-funcional/Lambdas y Streams.md>).

---

## 12. Resumen y cómo elegir

| Patrón | Problema que resuelve | Idea clave | En Java |
|---|---|---|---|
| **Strategy** | Varias formas de hacer lo mismo | Cada forma es un objeto intercambiable | Interfaz + clases o lambdas; `Comparator` |
| **Observer** | Avisar a muchos cuando algo cambia | Lista de suscriptores a los que se avisa | Interfaz `Suscriptor`; *listeners* |
| **Command** | Guardar, encolar o deshacer acciones | Cada acción es un objeto | Interfaz `Comando`; `Runnable` |
| **State** | Comportamiento distinto según la fase | Cada estado sabe qué se puede hacer y cuál es el siguiente | `enum` con un método por acción |
| **Template Method** | Mismo proceso con pasos distintos | Orden fijo en la clase padre; pasos en las hijas | Clase abstracta con método `final` |
| **Iterator** | Recorrer sin conocer el interior | Pedir «el siguiente» hasta que no haya más | `Iterable` y *for-each* |

### Preguntas para elegir

1. **¿Tengo un `if` que elige entre varias formas de hacer lo mismo?** → Strategy.
2. **¿Cuando algo pasa, otros deben enterarse?** → Observer.
3. **¿Necesito deshacer, guardar o poner en cola acciones?** → Command.
4. **¿El objeto se comporta distinto según su fase y tengo `if (estado == ...)` por todas partes?** → State.
5. **¿Varias clases repiten el mismo proceso y solo cambian algunos pasos?** → Template Method.
6. **¿Quiero que mi clase se pueda recorrer con *for-each*?** → Iterator (`Iterable`).

---

## 13. Errores frecuentes

| Error | Por qué es un problema | Qué hacer |
|---|---|---|
| Elegir la estrategia con un `if` dentro de la clase que la usa | Se vuelve al problema original. | Pasar la estrategia desde fuera (por el constructor o un método). |
| Crear una clase por estrategia cuando bastaría una lambda | Mucho código para algo sencillo. | Si la interfaz tiene un solo método, usar una lambda. |
| No permitir darse de baja en un Observer | Fugas de memoria y avisos a objetos que ya no se usan. | Ofrecer siempre `darDeBaja(...)`. |
| Un comando que no guarda los datos necesarios para deshacerse | `deshacer()` no sabe qué revertir. | Guardar en el comando todo lo que necesite. |
| Representar estados con textos (`"PAGADO"`) | Errores de escritura que no detecta el compilador. | Usar un `enum`. |
| Poner `default` en todos los `switch` de estados | Si se añade un estado, el compilador no avisa de los casos olvidados. | Escribir todos los casos cuando sea posible. |
| No marcar `final` el método plantilla | Una clase hija puede cambiar el orden de los pasos. | Declararlo `final`. |
| Usar un patrón «por si acaso» | Añade clases y complica el código. | Aplicarlo solo cuando aparece el problema que resuelve. |

---

## 14. Ejercicios

**Ejercicio 1 (Strategy).** Añade al ejemplo de envíos una estrategia `EnvioGratisLigero` que sea gratis si el peso es menor de 2 kg y, si no, cueste lo mismo que el envío estándar. Hazlo primero con una clase y luego con una lambda.

<details>
<summary>Solución</summary>

```java
// Con una clase
class EnvioGratisLigero implements EstrategiaEnvio {
    public int coste(int pesoKg) {
        return pesoKg < 2 ? 0 : new EnvioEstandar().coste(pesoKg);
    }
}

// Con una lambda
pedido.cambiarEnvio(peso -> peso < 2 ? 0 : 4 + peso);
```

`condicion ? a : b` es el **operador ternario**: devuelve `a` si la condición es cierta y `b` si no.

</details>

**Ejercicio 2 (Command).** Añade un comando `Mayusculas` que convierta todo el documento a mayúsculas y que, al deshacerse, recupere el texto anterior. Pista: guarda el texto anterior en el comando antes de cambiarlo.

<details>
<summary>Solución</summary>

Primero, añade a `Documento` un método para sustituir todo el texto:

```java
void reemplazar(String nuevo) {
    texto.setLength(0);
    texto.append(nuevo);
}
```

Después, el comando:

```java
class Mayusculas implements Comando {
    private final Documento documento;
    private String anterior;   // se guarda al ejecutar

    Mayusculas(Documento documento) {
        this.documento = documento;
    }

    public void ejecutar() {
        anterior = documento.contenido();
        documento.reemplazar(anterior.toUpperCase());
    }

    public void deshacer() {
        documento.reemplazar(anterior);
    }
}
```

Aquí se usa una clase y no un `record` porque el comando necesita guardar un dato (`anterior`) que cambia al ejecutarse.

</details>

**Ejercicio 3 (State).** Añade el estado `DEVUELTO` y una acción `devolver()` que solo se pueda hacer desde `ENTREGADO`. Compila el programa **antes** de tocar `cancelar()`: ¿qué ocurre?

<details>
<summary>Solución</summary>

Al añadir `DEVUELTO` al `enum`, el `switch` de `cancelar()` deja de cubrir todos los valores y **el programa no compila** («the switch expression does not cover all possible input values»). Es justo la ventaja de no usar `default`: el compilador te recuerda decidir qué pasa con el estado nuevo.

```java
enum EstadoPedido {
    PENDIENTE, PAGADO, ENVIADO, ENTREGADO, CANCELADO, DEVUELTO;

    // ...

    EstadoPedido devolver() {
        return switch (this) {
            case ENTREGADO -> DEVUELTO;
            default -> throw new IllegalStateException("No se puede devolver un pedido " + this);
        };
    }

    EstadoPedido cancelar() {
        return switch (this) {
            case PENDIENTE, PAGADO -> CANCELADO;
            case ENVIADO, ENTREGADO -> throw new IllegalStateException("Ya ha salido: hay que pedir una devolución");
            case CANCELADO -> CANCELADO;
            case DEVUELTO -> throw new IllegalStateException("Un pedido devuelto no se puede cancelar");
        };
    }
}
```

</details>

---

## 15. Glosario

| Término | Significado |
|---|---|
| **Patrón de diseño** | Solución conocida y con nombre para un problema de diseño que se repite. |
| **Interfaz funcional** | Interfaz con un único método por implementar. Se puede implementar con una lambda. |
| **Lambda** | Forma abreviada de escribir el método de una interfaz funcional: `x -> x * 2`. |
| **Referencia a método** | Forma aún más corta de una lambda que solo llama a un método: `String::length`. |
| **Sujeto** | En Observer, el objeto que avisa cuando cambia. |
| **Suscriptor** (u observador) | En Observer, el objeto que quiere enterarse de los cambios. |
| **Invocador** | En Command, el objeto que ejecuta los comandos y guarda el historial. |
| **Pila** | Colección donde el último elemento en entrar es el primero en salir. |
| **Estado** | La situación o fase en la que está un objeto en un momento dado. |
| **Método plantilla** | Método que fija el orden de los pasos de un proceso. |
| **Gancho** (*hook*) | Paso opcional de un método plantilla que las clases hijas pueden sobrescribir. |
| **Iterador** | Objeto que entrega los elementos de una colección uno a uno. |
| **Fuga de memoria** | Memoria que el programa ya no usa pero que no se puede liberar porque algo sigue apuntando a ella. |
