# Patrones de diseño creacionales en Java (21+)

## Introducción: ¿qué es un patrón de diseño?

Los patrones de diseño son **soluciones típicas a problemas que aparecen a menudo** en el desarrollo de software. Funcionan como planos predefinidos que se pueden adaptar para resolver un problema de diseño recurrente en tu código.

Un patrón **no es código para copiar y pegar**, como haríamos con una función o una librería. Es un concepto general, no una implementación concreta: te da una guía para construir una solución que encaje con las necesidades de tu proyecto.

Además, los patrones son una forma práctica de aplicar los **principios SOLID**, ya que ayudan a estructurar el código de manera clara, mantenible y escalable.

### ¿Por qué aprenderlos?

Puedes trabajar años como programador sin conocer formalmente ningún patrón; de hecho, a menudo los aplicas sin saberlo. Aun así, merece la pena aprenderlos porque:

- Son una **caja de herramientas** con soluciones probadas.
- Te ayudan a **pensar mejor el diseño** aplicando principios de la orientación a objetos.
- **Facilitan la comunicación**: decir "esto lo resolvemos con una Factory" se entiende al momento.
- **Mejoran la calidad** del código: más mantenible, escalable y flexible ante cambios.

### Clasificación por intención

| Categoría | Propósito | Ejemplos |
|---|---|---|
| **Creacionales** | Crear objetos de forma flexible y controlada | Singleton, Factory Method, Abstract Factory, Builder, Prototype |
| **Estructurales** | Combinar clases y objetos en estructuras mayores | Adapter, Composite, Decorator, Facade, Proxy |
| **De comportamiento** | Comunicación y reparto de responsabilidades entre objetos | Observer, Strategy, Command, State, Template Method |

Este documento trata los **patrones creacionales**.

> **Nota sobre Java moderno.** Muchos patrones clásicos (del libro *Design Patterns* de la "Gang of Four", 1994) se escribieron pensando en un Java/C++ sin lambdas, `record`, `sealed` ni *pattern matching*. En Java 21+ varios patrones se simplifican mucho. Aquí se muestra la forma clásica cuando ayuda a entender la idea, y la forma moderna recomendada.

---

## ¿Qué resuelven los patrones creacionales?

Usar `new` directamente acopla tu código a una **clase concreta**. Si mañana necesitas otra implementación, tienes que tocar todos los sitios donde se hace `new`. Los patrones creacionales **encapsulan la creación** de objetos para que:

- El código cliente dependa de **abstracciones** (interfaces), no de clases concretas → principio de **Inversión de Dependencias (D de SOLID)**.
- Añadir nuevos tipos no obligue a modificar código existente → principio **Abierto/Cerrado (O de SOLID)**.
- La lógica de construcción compleja esté en **un único sitio** → **Responsabilidad Única (S de SOLID)**.

---

## 1. Singleton

### Intención
Garantizar que una clase tenga **una única instancia** y ofrecer un punto de acceso global a ella.

### Cuándo usarlo
Recursos compartidos de verdad únicos: configuración de la aplicación, registro de *logs*, una caché global.

### Implementación recomendada: `enum`

Es la forma más sencilla y segura: la JVM garantiza una sola instancia, es *thread-safe* y resiste la serialización y la reflexión.

```java
public enum Configuracion {
    INSTANCIA;

    private final Properties props = new Properties();

    Configuracion() {
        try (var in = Configuracion.class.getResourceAsStream("/app.properties")) {
            if (in != null) props.load(in);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }

    public String get(String clave) {
        return props.getProperty(clave);
    }
}

// Uso
String url = Configuracion.INSTANCIA.get("db.url");
```

### Alternativa: inicialización perezosa con *holder*

Si necesitas una clase normal (por ejemplo, porque hereda de otra), el idioma **Initialization-on-demand holder** es perezoso y seguro entre hilos sin usar `synchronized`:

```java
public final class Registro {

    private Registro() { }

    private static final class Holder {
        private static final Registro INSTANCIA = new Registro();
    }

    public static Registro getInstance() {
        return Holder.INSTANCIA; // la clase Holder se carga solo la primera vez
    }
}
```

### Cuidado
- El Singleton es **estado global**: dificulta los tests y oculta dependencias. Hoy en día se prefiere crear una única instancia y **inyectarla** (con un framework como Spring o a mano por el constructor).
- Evita el *double-checked locking* clásico: es fácil equivocarse y el *holder* o el `enum` lo resuelven mejor.
- Con **hilos virtuales** (Java 21), los bloques `synchronized` podían "anclar" (*pinning*) el hilo virtual a su hilo de plataforma. Esto se corrigió en **Java 24** (JEP 491), pero es otra razón para preferir `enum` o *holder*.

---

## 2. Factory Method (Método Fábrica)

### Intención
Definir un método que **crea objetos** sin que el cliente sepa qué clase concreta se instancia.

### Ejemplo moderno con `sealed` y `switch`

En Java 21, las **interfaces selladas** (`sealed`) y los **records** permiten modelar familias cerradas de tipos, y el `switch` con *pattern matching* comprueba en compilación que se cubren todos los casos.

```java
public sealed interface Notificacion permits Email, Sms, Push {
    void enviar(String mensaje);
}

public record Email(String destinatario) implements Notificacion {
    public void enviar(String mensaje) {
        System.out.println("Email a " + destinatario + ": " + mensaje);
    }
}

public record Sms(String telefono) implements Notificacion {
    public void enviar(String mensaje) {
        System.out.println("SMS a " + telefono + ": " + mensaje);
    }
}

public record Push(String dispositivo) implements Notificacion {
    public void enviar(String mensaje) {
        System.out.println("Push a " + dispositivo + ": " + mensaje);
    }
}

public enum Canal { EMAIL, SMS, PUSH }

public final class NotificacionFactory {

    private NotificacionFactory() { }

    // Método fábrica: el cliente solo conoce Notificacion
    public static Notificacion crear(Canal canal, String destino) {
        return switch (canal) {          // switch exhaustivo: sin 'default'
            case EMAIL -> new Email(destino);
            case SMS   -> new Sms(destino);
            case PUSH  -> new Push(destino);
        };
    }
}

// Uso
Notificacion n = NotificacionFactory.crear(Canal.SMS, "600123456");
n.enviar("Tu pedido ha salido");
```

Si añades un nuevo valor a `Canal` y olvidas el `case`, **el código no compila**. Es una gran ventaja frente al `default` clásico.

### Forma clásica (GoF)

En la versión original, una clase abstracta declara el método fábrica y las subclases deciden qué crean:

```java
abstract class Aplicacion {
    protected abstract Documento crearDocumento(); // método fábrica

    public void nuevoDocumento() {
        Documento d = crearDocumento();
        d.abrir();
    }
}

class AplicacionTexto extends Aplicacion {
    protected Documento crearDocumento() { return new DocumentoTexto(); }
}
```

### Fábricas estáticas en el JDK
El propio JDK usa mucho este patrón: `List.of(...)`, `Optional.of(...)`, `Path.of(...)`, `LocalDate.of(...)`, `Executors.newVirtualThreadPerTaskExecutor()`.

---

## 3. Abstract Factory (Fábrica Abstracta)

### Intención
Crear **familias de objetos relacionados** sin especificar sus clases concretas, garantizando que los objetos de una familia sean compatibles entre sí.

### Ejemplo: componentes de interfaz para distintos temas

```java
// Productos
public interface Boton { void pintar(); }
public interface Casilla { void pintar(); }

// Fábrica abstracta
public interface FabricaUI {
    Boton crearBoton();
    Casilla crearCasilla();
}

// Familia "clara"
public final class FabricaClara implements FabricaUI {
    public Boton crearBoton()     { return () -> System.out.println("Botón claro"); }
    public Casilla crearCasilla() { return () -> System.out.println("Casilla clara"); }
}

// Familia "oscura"
public final class FabricaOscura implements FabricaUI {
    public Boton crearBoton()     { return () -> System.out.println("Botón oscuro"); }
    public Casilla crearCasilla() { return () -> System.out.println("Casilla oscura"); }
}

// Cliente: solo depende de la abstracción
public class Formulario {
    private final Boton boton;
    private final Casilla casilla;

    public Formulario(FabricaUI fabrica) {
        this.boton = fabrica.crearBoton();
        this.casilla = fabrica.crearCasilla();
    }

    public void mostrar() {
        boton.pintar();
        casilla.pintar();
    }
}

// Uso
FabricaUI fabrica = modoOscuro ? new FabricaOscura() : new FabricaClara();
new Formulario(fabrica).mostrar();
```

> Como `Boton` y `Casilla` tienen un único método abstracto, son **interfaces funcionales** y se pueden implementar con *lambdas*.

### Factory Method vs. Abstract Factory

| | Factory Method | Abstract Factory |
|---|---|---|
| Crea | **Un** producto | Una **familia** de productos |
| Mecanismo | Un método (a menudo estático o sobrescrito) | Un objeto fábrica con varios métodos |
| Ejemplo JDK | `List.of()` | `DocumentBuilderFactory`, JDBC (`Connection` crea `Statement`, etc.) |

---

## 4. Builder (Constructor)

### Intención
Construir objetos complejos **paso a paso**, separando la construcción de la representación. Evita los "constructores telescópicos" con muchos parámetros.

### Ejemplo con `record`

```java
public record Pedido(String cliente, List<String> productos,
                     String direccion, boolean urgente, String notas) {

    // Constructor compacto: validación y copia defensiva
    public Pedido {
        Objects.requireNonNull(cliente, "cliente obligatorio");
        productos = List.copyOf(productos);
    }

    public static Builder builder(String cliente) {
        return new Builder(cliente);
    }

    public static final class Builder {
        private final String cliente;
        private final List<String> productos = new ArrayList<>();
        private String direccion = "Recogida en tienda";
        private boolean urgente;
        private String notas = "";

        private Builder(String cliente) { this.cliente = cliente; }

        public Builder producto(String p)    { productos.add(p); return this; }
        public Builder direccion(String d)   { this.direccion = d; return this; }
        public Builder urgente()             { this.urgente = true; return this; }
        public Builder notas(String n)       { this.notas = n; return this; }

        public Pedido build() {
            if (productos.isEmpty()) throw new IllegalStateException("Pedido vacío");
            return new Pedido(cliente, productos, direccion, urgente, notas);
        }
    }
}

// Uso: legible y sin ambigüedades
Pedido p = Pedido.builder("Ana")
        .producto("Teclado")
        .producto("Ratón")
        .direccion("C/ Mayor 1, Madrid")
        .urgente()
        .build();
```

### ¿Cuándo NO hace falta?
Si el objeto tiene pocos campos, un `record` con su constructor canónico suele bastar. Para crear variantes de un record se puede añadir un método *wither*:

```java
public record Punto(int x, int y) {
    public Punto conX(int nuevoX) { return new Punto(nuevoX, y); }
}
```

Ejemplos en el JDK: `StringBuilder`, `HttpClient.newBuilder()`, `HttpRequest.newBuilder()`, `Thread.ofVirtual()`.

---

## 5. Prototype (Prototipo)

### Intención
Crear nuevos objetos **copiando** uno existente (el prototipo), en lugar de construirlos desde cero.

### Recomendación en Java moderno
Evita `Cloneable` y `Object.clone()`: su diseño es frágil (copia superficial, excepciones comprobadas, no funciona bien con campos `final`). Usa un **constructor de copia** o un método de copia explícito:

```java
public final class Plantilla {
    private final String titulo;
    private final List<String> secciones;

    public Plantilla(String titulo, List<String> secciones) {
        this.titulo = titulo;
        this.secciones = new ArrayList<>(secciones);
    }

    // Constructor de copia (copia profunda de la lista)
    public Plantilla(Plantilla otra) {
        this(otra.titulo, otra.secciones);
    }

    public Plantilla copiar() { return new Plantilla(this); }
}
```

> Con objetos **inmutables** (como los `record` con campos inmutables) el patrón casi desaparece: no hace falta copiar algo que no puede cambiar; basta con compartirlo.

---

## Resumen

| Patrón | Problema que resuelve | Herramienta Java 21+ clave |
|---|---|---|
| Singleton | Una sola instancia global | `enum` |
| Factory Method | Ocultar qué clase concreta se crea | `sealed` + `switch` exhaustivo |
| Abstract Factory | Crear familias de objetos compatibles | Interfaces + *lambdas* |
| Builder | Construir objetos con muchos parámetros | `record` + builder anidado |
| Prototype | Crear copias de objetos existentes | Constructor de copia / inmutabilidad |

### Consejos finales
1. **No fuerces un patrón**: si un `new` simple funciona y no va a cambiar, úsalo.
2. **Programa contra interfaces**, no contra implementaciones.
3. **Prefiere la inmutabilidad** (`record`, `List.of`, `List.copyOf`): simplifica muchos patrones creacionales.
4. En aplicaciones reales, la **inyección de dependencias** (Spring, Jakarta CDI, etc.) cubre gran parte de lo que antes resolvían Singleton y Factory.
