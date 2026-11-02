# Patrones de diseño de comportamiento en Java (21+)

## Introducción

Los patrones de diseño son **soluciones típicas a problemas recurrentes** en el desarrollo de software. No son código para copiar y pegar, sino conceptos generales que adaptas a tu proyecto. Además, son una forma práctica de aplicar los **principios SOLID**.

Según su intención, se clasifican en tres grandes grupos:

| Categoría | Propósito | Ejemplos |
|---|---|---|
| Creacionales | Crear objetos de forma flexible y controlada | Singleton, Factory Method, Abstract Factory |
| Estructurales | Combinar clases y objetos en estructuras mayores | Adapter, Composite, Decorator |
| **De comportamiento** | Comunicación y reparto de responsabilidades | **Observer, Strategy, Command, State, Template Method** |

Este documento trata los **patrones de comportamiento**.

---

## ¿Qué resuelven los patrones de comportamiento?

Se ocupan de **cómo se comunican y colaboran los objetos** y de **cómo se reparten las responsabilidades** entre ellos. Su objetivo es reducir el acoplamiento: que un objeto pueda pedir algo a otro sin conocer sus detalles.

Relación con SOLID:
- **Responsabilidad Única (S)**: cada algoritmo, acción o estado vive en su propia clase.
- **Abierto/Cerrado (O)**: añades una nueva estrategia, comando u observador sin tocar el código existente.
- **Inversión de Dependencias (D)**: el emisor depende de una interfaz, no del receptor concreto.

> **Nota sobre Java moderno.** Muchos patrones de comportamiento se basan en "una interfaz con un único método". En Java eso es una **interfaz funcional**, y se implementa con *lambdas* o referencias a métodos. Por eso, en Java 21+, Strategy, Command u Observer suelen requerir mucho menos código que en su forma clásica.

---

## 1. Strategy (Estrategia)

### Intención
Definir una **familia de algoritmos intercambiables**, encapsular cada uno y permitir elegirlos en tiempo de ejecución.

### Cuándo usarlo
Cuando tienes un `if/else` o `switch` que elige entre varias formas de hacer lo mismo (calcular descuentos, ordenar, validar, exportar...).

### Ejemplo clásico

```java
public interface EstrategiaDescuento {
    BigDecimal aplicar(BigDecimal precio);
}

public final class SinDescuento implements EstrategiaDescuento {
    public BigDecimal aplicar(BigDecimal precio) { return precio; }
}

public final class DescuentoPorcentaje implements EstrategiaDescuento {
    private final BigDecimal porcentaje;
    public DescuentoPorcentaje(int porcentaje) { this.porcentaje = BigDecimal.valueOf(porcentaje); }
    public BigDecimal aplicar(BigDecimal precio) {
        return precio.subtract(precio.multiply(porcentaje).divide(BigDecimal.valueOf(100)));
    }
}

public class Carrito {
    private EstrategiaDescuento descuento = new SinDescuento();

    public void setDescuento(EstrategiaDescuento descuento) { this.descuento = descuento; }

    public BigDecimal total(BigDecimal subtotal) { return descuento.aplicar(subtotal); }
}
```

### Forma moderna con *lambdas*

`EstrategiaDescuento` es una interfaz funcional, así que no necesitamos una clase por estrategia:

```java
@FunctionalInterface
public interface EstrategiaDescuento {
    BigDecimal aplicar(BigDecimal precio);

    static EstrategiaDescuento ninguno()          { return p -> p; }
    static EstrategiaDescuento fijo(String euros) { return p -> p.subtract(new BigDecimal(euros)).max(BigDecimal.ZERO); }
    static EstrategiaDescuento porcentaje(int pct) {
        return p -> p.multiply(BigDecimal.valueOf(100 - pct)).divide(BigDecimal.valueOf(100));
    }
}

// Uso
carrito.setDescuento(EstrategiaDescuento.porcentaje(20));
carrito.setDescuento(p -> p.compareTo(new BigDecimal("100")) > 0 ? p.subtract(BigDecimal.TEN) : p);
```

Ejemplo en el JDK: `Comparator` es una estrategia de ordenación.

```java
lista.sort(Comparator.comparing(Persona::apellido).thenComparing(Persona::nombre));
```

---

## 2. Observer (Observador)

### Intención
Definir una relación **uno a muchos**: cuando un objeto (sujeto) cambia de estado, **notifica automáticamente** a todos sus observadores suscritos.

### Cuándo usarlo
Eventos de interfaz gráfica, sistemas de notificaciones, actualizar varias vistas cuando cambian los datos, arquitecturas orientadas a eventos.

### Ejemplo con eventos tipados (`sealed` + `record`)

```java
// Eventos posibles: conjunto cerrado
public sealed interface EventoPedido permits PedidoCreado, PedidoEnviado, PedidoCancelado {}
public record PedidoCreado(String id, BigDecimal total) implements EventoPedido {}
public record PedidoEnviado(String id, String seguimiento) implements EventoPedido {}
public record PedidoCancelado(String id, String motivo) implements EventoPedido {}

// Sujeto
public final class PublicadorPedidos {

    private final List<Consumer<EventoPedido>> observadores = new CopyOnWriteArrayList<>();

    public Runnable suscribir(Consumer<EventoPedido> observador) {
        observadores.add(observador);
        return () -> observadores.remove(observador); // permite darse de baja
    }

    public void publicar(EventoPedido evento) {
        observadores.forEach(o -> o.accept(evento));
    }
}

// Observadores
var publicador = new PublicadorPedidos();

publicador.suscribir(evento -> switch (evento) {
    case PedidoCreado(var id, var total)       -> System.out.println("Nuevo pedido " + id + " por " + total + " €");
    case PedidoEnviado(var id, var seguimiento) -> System.out.println("Pedido " + id + " enviado: " + seguimiento);
    case PedidoCancelado(var id, var motivo)    -> System.out.println("Pedido " + id + " cancelado: " + motivo);
});

Runnable baja = publicador.suscribir(e -> registrarEnAuditoria(e));

publicador.publicar(new PedidoCreado("P-1", new BigDecimal("49.90")));
baja.run(); // el auditor deja de recibir eventos
```

### Puntos importantes
- `CopyOnWriteArrayList` permite suscribirse o darse de baja mientras se notifica, sin `ConcurrentModificationException`.
- Ofrece siempre una forma de **darse de baja**: si no, los observadores no se liberan (fugas de memoria).
- `java.util.Observable` y `Observer` están **obsoletos desde Java 9**; no los uses.
- Para flujos asíncronos con control de presión (*backpressure*) existe `java.util.concurrent.Flow` (Java 9+), base de librerías reactivas.

---

## 3. Command (Orden)

### Intención
Encapsular una **petición como un objeto**. Así puedes guardarla, ponerla en cola, registrarla o **deshacerla**.

### Cuándo usarlo
Deshacer/rehacer, colas de tareas, macros, botones y menús, registro de operaciones.

### Ejemplo: editor con deshacer

```java
public interface Comando {
    void ejecutar();
    void deshacer();
}

public final class Documento {
    private final StringBuilder texto = new StringBuilder();
    public void insertar(String s)        { texto.append(s); }
    public void borrarUltimos(int n)      { texto.setLength(texto.length() - n); }
    public String contenido()             { return texto.toString(); }
}

public record InsertarTexto(Documento doc, String texto) implements Comando {
    public void ejecutar() { doc.insertar(texto); }
    public void deshacer() { doc.borrarUltimos(texto.length()); }
}

// Invocador: ejecuta comandos y guarda el historial
public final class Editor {
    private final Deque<Comando> historial = new ArrayDeque<>();

    public void ejecutar(Comando c) {
        c.ejecutar();
        historial.push(c);
    }

    public void deshacer() {
        if (!historial.isEmpty()) historial.pop().deshacer();
    }
}

// Uso
var doc = new Documento();
var editor = new Editor();
editor.ejecutar(new InsertarTexto(doc, "Hola "));
editor.ejecutar(new InsertarTexto(doc, "mundo"));
editor.deshacer();
System.out.println(doc.contenido()); // "Hola "
```

### Comandos sin deshacer: basta con `Runnable`
Si solo necesitas "ejecutar algo más tarde", `Runnable` o `Callable` ya son comandos. Con **hilos virtuales** (Java 21) puedes lanzar miles de ellos de forma barata:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Runnable> tareas = List.of(
            () -> enviarEmail("ana@ejemplo.es"),
            () -> enviarEmail("luis@ejemplo.es"));
    tareas.forEach(executor::submit);
} // el try-with-resources espera a que terminen
```

---

## 4. State (Estado)

### Intención
Permitir que un objeto **cambie su comportamiento cuando cambia su estado interno**, como si cambiara de clase.

### Cuándo usarlo
Cuando un objeto tiene un ciclo de vida con reglas distintas según su estado (pedidos, máquinas expendedoras, semáforos, flujos de aprobación) y el código se llena de `if (estado == ...)`.

### Ejemplo moderno: estados como tipos sellados

```java
public sealed interface EstadoPedido permits Pendiente, Pagado, Enviado, Entregado, Cancelado {}
public record Pendiente() implements EstadoPedido {}
public record Pagado(String referenciaPago) implements EstadoPedido {}
public record Enviado(String seguimiento) implements EstadoPedido {}
public record Entregado() implements EstadoPedido {}
public record Cancelado(String motivo) implements EstadoPedido {}

public final class Transiciones {

    public static EstadoPedido pagar(EstadoPedido actual, String referencia) {
        return switch (actual) {
            case Pendiente p -> new Pagado(referencia);
            default -> throw new IllegalStateException("No se puede pagar en estado " + actual);
        };
    }

    public static EstadoPedido cancelar(EstadoPedido actual, String motivo) {
        return switch (actual) {           // switch exhaustivo: sin 'default'
            case Pendiente p -> new Cancelado(motivo);
            case Pagado p    -> new Cancelado(motivo);
            case Enviado e   -> throw new IllegalStateException("Ya enviado: gestiona una devolución");
            case Entregado e -> throw new IllegalStateException("Ya entregado");
            case Cancelado c -> c; // idempotente
        };
    }
}
```

Cada estado es inmutable y solo transporta los datos que tienen sentido en él (por ejemplo, solo `Enviado` tiene número de seguimiento). El compilador avisa si un `switch` exhaustivo olvida un estado.

> Desde **Java 22**, con el patrón sin nombre `_`, puedes agrupar casos: `case Pendiente _, Pagado _ -> new Cancelado(motivo);`. En Java 21 hay que escribirlos por separado, como arriba.

### Forma clásica (GoF)
En la versión original, cada estado es una clase con los mismos métodos (`pagar()`, `cancelar()`...) y el objeto contexto delega en su estado actual. Es preferible cuando cada estado tiene **mucho comportamiento propio**.

---

## 5. Template Method (Método Plantilla)

### Intención
Definir el **esqueleto de un algoritmo** en un método, dejando que ciertos pasos los implementen las subclases sin cambiar la estructura general.

### Ejemplo

```java
public abstract class ExportadorInforme {

    // Método plantilla: 'final' para que nadie altere el orden de los pasos
    public final String exportar(List<String> filas) {
        var sb = new StringBuilder();
        sb.append(cabecera());
        filas.forEach(f -> sb.append(formatearFila(f)));
        sb.append(pie());
        return sb.toString();
    }

    protected abstract String cabecera();
    protected abstract String formatearFila(String fila);
    protected String pie() { return ""; } // "gancho" opcional
}

public final class ExportadorCsv extends ExportadorInforme {
    protected String cabecera()             { return "dato\n"; }
    protected String formatearFila(String f) { return f + "\n"; }
}

public final class ExportadorHtml extends ExportadorInforme {
    protected String cabecera()             { return "<ul>\n"; }
    protected String formatearFila(String f) { return "  <li>" + f + "</li>\n"; }
    protected String pie()                  { return "</ul>\n"; }
}
```

### Alternativa con composición
Como alternativa a la herencia, los pasos variables pueden recibirse como funciones (es decir, combinarlo con Strategy):

```java
public record Exportador(String cabecera, UnaryOperator<String> fila, String pie) {
    public String exportar(List<String> filas) {
        var sb = new StringBuilder(cabecera);
        filas.forEach(f -> sb.append(fila.apply(f)));
        return sb.append(pie).toString();
    }
}

var csv = new Exportador("dato\n", f -> f + "\n", "");
```

---

## 6. Iterator (Iterador) — mención breve

Permite **recorrer una colección sin exponer su estructura interna**. En Java está integrado en el lenguaje: cualquier clase que implemente `Iterable<T>` puede usarse en un bucle *for-each*. Hoy en día, para recorridos complejos se suele usar la **API Stream**.

```java
for (String nombre : nombres) { ... }               // usa Iterator internamente
nombres.stream().filter(n -> n.startsWith("A")).toList();
```

---

## Resumen

| Patrón | Idea clave | Herramienta Java 21+ / JDK |
|---|---|---|
| Strategy | Algoritmos intercambiables | Interfaces funcionales, *lambdas*, `Comparator` |
| Observer | Notificar cambios a muchos suscriptores | `Consumer<T>`, `CopyOnWriteArrayList`, `Flow` |
| Command | Encapsular una petición como objeto | `record`, `Runnable`, hilos virtuales |
| State | Comportamiento según el estado interno | `sealed` + `record` + `switch` exhaustivo |
| Template Method | Esqueleto fijo con pasos variables | Clase abstracta con método `final`, o composición |
| Iterator | Recorrer sin exponer la estructura | `Iterable`, *for-each*, Streams |

### Consejos finales
1. Si tu interfaz tiene **un solo método**, probablemente puedas usar una *lambda* en lugar de una jerarquía de clases.
2. Usa **`sealed` + `switch` exhaustivo** para modelar estados y eventos: el compilador te avisará si olvidas un caso.
3. Prefiere **objetos inmutables** (`record`) para eventos, comandos y estados: son más fáciles de razonar y seguros entre hilos.
4. Empieza simple. Aplica un patrón cuando aparezca el problema que resuelve, no antes.
