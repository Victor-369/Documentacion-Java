# Patrones de diseño estructurales en Java (21+)

## Introducción

Los patrones de diseño son **soluciones típicas a problemas recurrentes** en el desarrollo de software. No son código para copiar y pegar, sino conceptos generales que adaptas a tu proyecto. Además, son una forma práctica de aplicar los **principios SOLID**.

Según su intención, se clasifican en tres grandes grupos:

| Categoría | Propósito | Ejemplos |
|---|---|---|
| Creacionales | Crear objetos de forma flexible y controlada | Singleton, Factory Method, Abstract Factory |
| **Estructurales** | Combinar clases y objetos en estructuras mayores | **Adapter, Composite, Decorator, Facade, Proxy** |
| De comportamiento | Comunicación y reparto de responsabilidades | Observer, Strategy, Command |

Este documento trata los **patrones estructurales**.

---

## ¿Qué resuelven los patrones estructurales?

Describen **cómo agrupar clases y objetos** para formar estructuras más grandes, manteniéndolas **eficientes y flexibles**. Se apoyan sobre todo en una idea:

> **Composición antes que herencia.** En lugar de heredar para reutilizar comportamiento, un objeto *contiene* a otro y le delega trabajo. Esto reduce el acoplamiento y permite cambiar piezas en tiempo de ejecución.

Relación con SOLID:
- **Abierto/Cerrado (O)**: añades funcionalidad envolviendo objetos, sin modificar su código (Decorator, Proxy).
- **Segregación de Interfaces (I)**: adaptas una interfaz grande o incómoda a la que realmente necesitas (Adapter, Facade).
- **Inversión de Dependencias (D)**: el cliente trabaja con interfaces, no con implementaciones concretas.

---

## 1. Adapter (Adaptador)

### Intención
Hacer que dos interfaces **incompatibles** puedan trabajar juntas. Es como un adaptador de enchufe: no cambia el aparato ni la pared, solo los conecta.

### Cuándo usarlo
- Integrar una **librería externa** o código heredado cuya interfaz no encaja con la tuya.
- Aislar tu dominio de APIs de terceros (si cambias de proveedor, solo cambias el adaptador).

### Ejemplo

```java
// Interfaz que espera nuestra aplicación
public interface PasarelaPago {
    boolean cobrar(String cliente, BigDecimal importe);
}

// Librería externa que NO podemos modificar
public class StripeLikeClient {
    public String charge(String customerId, long cents, String currency) {
        // ... llama al servicio externo
        return "OK";
    }
}

// Adaptador: implementa nuestra interfaz y delega en la librería
public final class PasarelaStripeAdapter implements PasarelaPago {

    private final StripeLikeClient cliente;

    public PasarelaStripeAdapter(StripeLikeClient cliente) {
        this.cliente = cliente;
    }

    @Override
    public boolean cobrar(String clienteId, BigDecimal importe) {
        long centimos = importe.movePointRight(2).longValueExact();
        return "OK".equals(cliente.charge(clienteId, centimos, "EUR"));
    }
}

// Uso: el resto del código solo conoce PasarelaPago
PasarelaPago pasarela = new PasarelaStripeAdapter(new StripeLikeClient());
pasarela.cobrar("cli-42", new BigDecimal("19.99"));
```

### Adaptadores ligeros con *lambdas*
Si la interfaz destino es funcional, el adaptador puede ser una simple *lambda* o referencia a método:

```java
Function<String, Integer> longitud = String::length; // adapta String a Function
Runnable tarea = () -> servicio.procesar();          // adapta un método a Runnable
```

Ejemplos en el JDK: `Arrays.asList(...)` (array → `List`), `InputStreamReader` (bytes → caracteres).

---

## 2. Composite (Compuesto)

### Intención
Componer objetos en **estructuras de árbol** y tratar de la misma forma a los objetos individuales (hojas) y a los grupos (compuestos).

### Cuándo usarlo
Jerarquías parte-todo: sistemas de ficheros, menús con submenús, organigramas, expresiones matemáticas.

### Ejemplo moderno con `sealed` + `record`

En Java 21, una **interfaz sellada** define el conjunto cerrado de nodos, y el **`switch` con patrones de record** recorre el árbol de forma clara y exhaustiva.

```java
public sealed interface Nodo permits Fichero, Carpeta {}

public record Fichero(String nombre, long bytes) implements Nodo {}

public record Carpeta(String nombre, List<Nodo> hijos) implements Nodo {
    public Carpeta {
        hijos = List.copyOf(hijos);
    }
}

public final class Utilidades {

    // Tratamos hojas y compuestos de forma uniforme
    public static long tamano(Nodo nodo) {
        return switch (nodo) {
            case Fichero(var nombre, var bytes) -> bytes;
            case Carpeta(var nombre, var hijos) ->
                    hijos.stream().mapToLong(Utilidades::tamano).sum();
        };
    }
}

// Uso
Nodo raiz = new Carpeta("proyecto", List.of(
        new Fichero("README.md", 1_200),
        new Carpeta("src", List.of(
                new Fichero("Main.java", 3_400),
                new Fichero("Util.java", 900)
        ))
));

System.out.println(Utilidades.tamano(raiz)); // 5500
```

> Desde **Java 22** puedes usar el **patrón sin nombre** `_` para las variables que no usas: `case Fichero(_, var bytes) -> bytes;`

### Forma clásica (orientada a objetos)
También es válido que cada nodo implemente el comportamiento:

```java
public interface Componente { long tamano(); }

public record Archivo(long bytes) implements Componente {
    public long tamano() { return bytes; }
}

public record Directorio(List<Componente> hijos) implements Componente {
    public long tamano() {
        return hijos.stream().mapToLong(Componente::tamano).sum();
    }
}
```

**¿Cuál elegir?** Si los tipos de nodo son estables y las **operaciones** cambian a menudo, usa `sealed` + `switch` (añadir una operación es añadir un método). Si los **tipos** cambian a menudo, usa la forma clásica (añadir un tipo es añadir una clase).

---

## 3. Decorator (Decorador)

### Intención
**Añadir responsabilidades** a un objeto de forma dinámica, envolviéndolo en otro objeto con la misma interfaz. Es una alternativa flexible a crear subclases para cada combinación.

### Ejemplo

```java
public interface Texto {
    String contenido();
}

public record TextoPlano(String valor) implements Texto {
    public String contenido() { return valor; }
}

// Decoradores: reciben un Texto y devuelven un Texto
public record Mayusculas(Texto interno) implements Texto {
    public String contenido() { return interno.contenido().toUpperCase(); }
}

public record Negrita(Texto interno) implements Texto {
    public String contenido() { return "**" + interno.contenido() + "**"; }
}

public record Recortado(Texto interno) implements Texto {
    public String contenido() { return interno.contenido().strip(); }
}

// Uso: se combinan libremente
Texto t = new Negrita(new Mayusculas(new Recortado(new TextoPlano("  hola  "))));
System.out.println(t.contenido()); // **HOLA**
```

Con 3 decoradores tienes todas las combinaciones posibles sin crear `TextoMayusculasNegritaRecortado` y similares.

### Decoración funcional
Con interfaces funcionales, la composición de funciones es un decorador ligero:

```java
UnaryOperator<String> recortar = String::strip;
UnaryOperator<String> mayus    = String::toUpperCase;

Function<String, String> pipeline = recortar.andThen(mayus).andThen(s -> "**" + s + "**");
System.out.println(pipeline.apply("  hola  ")); // **HOLA**
```

Ejemplo clásico en el JDK: los *streams* de E/S.

```java
var lector = new BufferedReader(new InputStreamReader(new FileInputStream("datos.txt")));
```

También `Collections.unmodifiableList(...)` y `Collections.synchronizedList(...)` decoran una lista.

---

## 4. Facade (Fachada)

### Intención
Ofrecer una **interfaz simple** a un subsistema complejo. El cliente habla con la fachada y no necesita conocer los detalles internos.

### Ejemplo

```java
// Subsistema complejo
class Inventario   { void reservar(String producto) { /* ... */ } }
class Pagos        { void cobrar(String cliente, BigDecimal importe) { /* ... */ } }
class Envios       { String programar(String cliente, String producto) { return "ENV-001"; } }
class Notificador  { void avisar(String cliente, String mensaje) { /* ... */ } }

// Fachada: un único punto de entrada
public final class TiendaFacade {

    private final Inventario inventario = new Inventario();
    private final Pagos pagos = new Pagos();
    private final Envios envios = new Envios();
    private final Notificador notificador = new Notificador();

    public String comprar(String cliente, String producto, BigDecimal precio) {
        inventario.reservar(producto);
        pagos.cobrar(cliente, precio);
        String envio = envios.programar(cliente, producto);
        notificador.avisar(cliente, "Pedido confirmado. Envío: " + envio);
        return envio;
    }
}

// Uso
new TiendaFacade().comprar("Ana", "Portátil", new BigDecimal("899.00"));
```

### Cuidado
La fachada **no debe convertirse en un "objeto dios"** que lo hace todo. Debe coordinar, no contener la lógica de negocio de cada subsistema. En un proyecto real, las dependencias se pasarían por el constructor (inyección) en lugar de crearse con `new`.

---

## 5. Proxy (Intermediario)

### Intención
Proporcionar un **sustituto** de otro objeto para controlar el acceso a él. Tiene la misma interfaz que el objeto real.

### Tipos habituales
- **Virtual**: retrasa la creación de un objeto costoso hasta que se necesita (carga perezosa).
- **De protección**: comprueba permisos antes de delegar.
- **De caché**: guarda resultados para no repetir operaciones costosas.
- **Remoto**: representa un objeto que vive en otra máquina.

### Ejemplo: proxy de caché

```java
public interface ServicioTiempo {
    String prevision(String ciudad);
}

public final class ServicioTiempoReal implements ServicioTiempo {
    public String prevision(String ciudad) {
        // llamada lenta a una API externa
        return "Soleado en " + ciudad;
    }
}

public final class ServicioTiempoCache implements ServicioTiempo {

    private final ServicioTiempo real;
    private final Map<String, String> cache = new ConcurrentHashMap<>();

    public ServicioTiempoCache(ServicioTiempo real) {
        this.real = real;
    }

    @Override
    public String prevision(String ciudad) {
        return cache.computeIfAbsent(ciudad, real::prevision);
    }
}
```

### Proxy dinámico del JDK
Java permite crear proxies en tiempo de ejecución con `java.lang.reflect.Proxy`. Es la base de muchos frameworks (Spring AOP, transacciones, etc.):

```java
ServicioTiempo real = new ServicioTiempoReal();

ServicioTiempo conLog = (ServicioTiempo) Proxy.newProxyInstance(
        ServicioTiempo.class.getClassLoader(),
        new Class<?>[]{ ServicioTiempo.class },
        (proxy, metodo, args) -> {
            System.out.println("Llamando a " + metodo.getName());
            return metodo.invoke(real, args);
        });
```

### Proxy vs. Decorator
Su estructura es casi idéntica. La diferencia es la **intención**: el Decorator **añade comportamiento**; el Proxy **controla el acceso** (cuándo, quién o si se llama al objeto real).

---

## Resumen

| Patrón | Idea clave | Herramienta Java 21+ / JDK |
|---|---|---|
| Adapter | Conectar interfaces incompatibles | Clases envoltorio, *lambdas* y referencias a métodos |
| Composite | Tratar igual hojas y grupos en un árbol | `sealed` + `record` + `switch` con patrones |
| Decorator | Añadir comportamiento envolviendo | Records envoltorio, `Function.andThen` |
| Facade | Simplificar un subsistema complejo | Clase coordinadora |
| Proxy | Controlar el acceso a un objeto | `java.lang.reflect.Proxy` |

### Consejos finales
1. **Prefiere composición a herencia**: es el hilo común de todos estos patrones.
2. Los `record` son ideales para envoltorios sencillos (Decorator, Composite): concisos e inmutables.
3. Muchos patrones estructurales parecen iguales en código; **lo que los distingue es su intención**.
4. No añadas capas "por si acaso": cada envoltorio tiene un coste de complejidad.
