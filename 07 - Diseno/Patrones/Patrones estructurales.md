# Patrones de diseño estructurales en Java

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los programas completos de este documento se han compilado y ejecutado con OpenJDK 25.
>
> Documentos relacionados: [Patrones creacionales](<Patrones creacionales.md>) · [Patrones de comportamiento](<Patrones comportamiento.md>)

## Índice

1. [¿Qué es un patrón de diseño?](#1-qué-es-un-patrón-de-diseño)
2. [Antes de empezar: cómo probar los ejemplos](#2-antes-de-empezar-cómo-probar-los-ejemplos)
3. [Vocabulario mínimo](#3-vocabulario-mínimo)
4. [La idea clave: envolver objetos](#4-la-idea-clave-envolver-objetos)
5. [Adapter (adaptador)](#5-adapter-adaptador)
6. [Composite (compuesto)](#6-composite-compuesto)
7. [Decorator (decorador)](#7-decorator-decorador)
8. [Facade (fachada)](#8-facade-fachada)
9. [Proxy (intermediario)](#9-proxy-intermediario)
10. [Resumen: se parecen, pero no son iguales](#10-resumen-se-parecen-pero-no-son-iguales)
11. [Errores frecuentes](#11-errores-frecuentes)
12. [Ejercicios](#12-ejercicios)
13. [Glosario](#13-glosario)

---

## 1. ¿Qué es un patrón de diseño?

Un **patrón de diseño** es una **solución ya probada** para un problema que aparece una y otra vez al programar. Tiene un nombre, explica qué problema resuelve y describe cómo organizar las clases para resolverlo.

> **Analogía:** un patrón es como una **receta de cocina**. No es tu plato concreto, sino unos pasos que adaptas a lo que tienes. Por eso un patrón no se copia y pega: se entiende la idea y se aplica a tu programa.

Los patrones clásicos se recogieron en el libro *Design Patterns* (1994), de cuatro autores conocidos como *Gang of Four* (GoF). Se agrupan según para qué sirven:

| Grupo | Para qué sirven | Ejemplos |
|---|---|---|
| **Creacionales** | **Crear objetos** de forma ordenada y flexible | Singleton, Factory Method, Builder |
| **Estructurales** (este documento) | **Combinar** clases y objetos para formar estructuras más grandes | Adapter, Composite, Decorator, Facade, Proxy |
| **De comportamiento** | Organizar **cómo se comunican** los objetos y quién hace cada cosa | Strategy, Observer, Command |

### ¿Por qué merece la pena aprenderlos?

- **No reinventas la rueda:** otras personas ya pensaron una buena solución.
- **Hablas el mismo idioma que tu equipo:** decir «aquí usamos un *Decorator*» se entiende al momento, sin explicar todo el código.
- **Reconoces el código que lees:** muchas librerías de Java usan estos patrones. Si los conoces, entiendes antes cómo funcionan.
- **Tu código aguanta mejor los cambios:** cada pieza tiene un papel claro, así que añadir algo nuevo no obliga a reescribir lo que ya funciona.

### Patrones y principios SOLID

**SOLID** son cinco principios para escribir código orientado a objetos fácil de mantener. **No necesitas conocerlos para seguir este documento**, pero si ya los conoces verás que muchos patrones son, en la práctica, una forma concreta de aplicarlos. Los tienes explicados en [Principios SOLID](<../Principios/Principio SOLID.md>).

| Patrón | Principio que refuerza | Por qué |
|---|---|---|
| **Adapter** | Abierto/cerrado (OCP) | Permite usar una clase con otra interfaz **sin modificarla**. |
| **Composite** | Abierto/cerrado (OCP) | Se pueden añadir tipos nuevos de elementos sin cambiar el código que recorre el árbol. |
| **Decorator** | Abierto/cerrado (OCP) y responsabilidad única (SRP) | Añade comportamiento sin tocar la clase original, y cada decorador se ocupa de **una sola cosa**. |
| **Facade** | Inversión de dependencias (DIP) y bajo acoplamiento | El código cliente depende de una puerta de entrada sencilla, no de todas las clases internas. |
| **Proxy** | Responsabilidad única (SRP) | El control de acceso, la caché o el registro van en el proxy, no en la clase real. |

---

## 2. Antes de empezar: cómo probar los ejemplos

Cada patrón incluye un **programa completo**. Para ejecutarlo:

1. Instala un JDK 25 o superior (comprueba la versión con `java -version`).
2. Copia el código en un fichero `.java`, por ejemplo `Adapter.java`.
3. Ejecútalo desde la terminal, en la misma carpeta:

```bash
java Adapter.java
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
| **Herencia (`extends`)** | Una clase hija recibe todo lo de su clase padre. | `class Deportivo extends Coche` |
| **Atributo** | Un dato que guarda cada objeto. | `private final String nombre;` |
| **Delegar** | Que un objeto pida a otro que haga un trabajo en su lugar. | `return motor.arrancar();` |
| **`record`** | Forma corta de crear una clase que solo guarda datos que no cambian. Java genera el constructor y un método para leer cada dato. | `record Punto(int x, int y) {}` → `p.x()` |
| **`@Override`** | Marca un método que viene de una interfaz o de la clase padre. El compilador comprueba que es correcto. | `@Override public String toString()` |
| **`Map`** | Colección que guarda parejas clave → valor. | `mapa.put("Madrid", "Soleado")` |
| **`List.of(...)`** | Crea una lista que no se puede modificar. | `List.of("a", "b")` |

---

## 4. La idea clave: envolver objetos

Casi todos los patrones estructurales se basan en lo mismo: **un objeto guarda dentro a otro objeto y le pasa (delega) parte del trabajo**. A esto se le llama **composición**.

```text
┌──────────────────────────────┐
│ Objeto exterior              │
│   ┌──────────────────────┐   │
│   │ Objeto interior      │   │   El exterior recibe la petición,
│   └──────────────────────┘   │   hace algo (o no) y se la pasa
└──────────────────────────────┘   al interior.
```

La alternativa sería la **herencia** (`extends`), pero la herencia es rígida: se decide al escribir el código y no se puede cambiar mientras el programa se ejecuta. Con la composición, puedes **cambiar o combinar piezas** como si fueran bloques de construcción.

> **Regla práctica muy citada:** «prefiere la composición a la herencia». No significa que la herencia esté prohibida, sino que, si dudas, suele ser mejor que un objeto *tenga* a otro que que *sea* otro.

Lo que diferencia a cada patrón estructural es **para qué** se envuelve el objeto:

| Patrón | Envuelve para… |
|---|---|
| Adapter | **Traducir** una interfaz que no encaja |
| Composite | **Agrupar** objetos en forma de árbol y tratarlos igual |
| Decorator | **Añadir** funcionalidad |
| Facade | **Simplificar** el uso de muchas piezas |
| Proxy | **Controlar** el acceso al objeto real |

---

## 5. Adapter (adaptador)

### En una frase

Hace que dos piezas que **no encajan** puedan trabajar juntas, poniendo entre ellas un traductor.

### Analogía

Viajas a Reino Unido con tu cargador. El enchufe de la pared tiene otra forma y no encaja. No cambias el cargador ni la pared: usas un **adaptador de enchufe**.

### El problema

Tu aplicación trabaja con temperaturas en grados **Celsius** y espera esta interfaz:

```java
interface Termometro {
    double gradosCelsius();
}
```

Compras un sensor cuya librería, que **no puedes modificar**, solo ofrece grados **Fahrenheit** y con otro nombre de método:

```java
class SensorAmericano {
    double readFahrenheit() { ... }
}
```

`SensorAmericano` no implementa `Termometro`, así que tu aplicación no puede usarlo directamente.

### La solución paso a paso

1. Crea una clase nueva, el **adaptador**, que **implemente la interfaz que tu aplicación espera** (`Termometro`).
2. El adaptador **guarda dentro** el objeto que no encaja (`SensorAmericano`).
3. Cuando le piden `gradosCelsius()`, el adaptador llama a `readFahrenheit()` y **convierte** el resultado.

### Programa completo

```java
// Fichero: Adapter.java

// Lo que espera nuestra aplicación
interface Termometro {
    double gradosCelsius();
}

// Librería externa: NO la podemos modificar
class SensorAmericano {
    double readFahrenheit() {
        return 98.6;   // en la realidad, leería el sensor
    }
}

// El adaptador: implementa lo que esperamos y traduce
class AdaptadorSensor implements Termometro {
    private final SensorAmericano sensor;

    AdaptadorSensor(SensorAmericano sensor) {
        this.sensor = sensor;
    }

    @Override
    public double gradosCelsius() {
        double fahrenheit = sensor.readFahrenheit();
        return (fahrenheit - 32) * 5 / 9;   // conversión de unidades
    }
}

// Código de la aplicación: solo conoce Termometro
void mostrarTemperatura(Termometro termometro) {
    IO.println("Temperatura: %.1f °C".formatted(termometro.gradosCelsius()));
}

void main() {
    Termometro termometro = new AdaptadorSensor(new SensorAmericano());
    mostrarTemperatura(termometro);
}
```

Salida:

```text
Temperatura: 37,0 °C
```

> El decimal aparece con coma porque `formatted` usa la configuración regional del sistema (en España, la coma). Con otra configuración podría salir `37.0`.

### Qué ha pasado

- `mostrarTemperatura` solo conoce `Termometro`. No sabe que detrás hay un sensor americano.
- Ni la aplicación ni la librería han cambiado: todo el «trabajo de traducción» está en el adaptador.
- Si mañana cambias de marca de sensor, solo escribes **otro adaptador**. El resto de la aplicación no se toca.

### Dónde lo verás en Java

- `Arrays.asList(array)` adapta un *array* para que se pueda usar como una `List`.
- `InputStreamReader` adapta un flujo de **bytes** para leerlo como **caracteres** (texto).

### Cuándo usarlo

- Para usar una **librería externa** o código antiguo cuya forma no encaja con la de tu programa.
- Para que tu código no dependa directamente de una librería de terceros: si cambias de proveedor, solo cambias el adaptador.

---

## 6. Composite (compuesto)

### En una frase

Organiza objetos en forma de **árbol** y permite tratar igual a un elemento suelto y a un grupo de elementos.

### Analogía

Una carpeta de tu ordenador puede contener ficheros **y otras carpetas**, que a su vez contienen más ficheros y carpetas. Cuando preguntas «¿cuánto ocupa esta carpeta?», el sistema suma lo que ocupa todo lo de dentro, sin importar cuántos niveles haya.

### El problema

Quieres calcular el tamaño total de una carpeta. Sin el patrón, el código tendría que preguntar en cada paso «¿esto es un fichero o una carpeta?», y escribir bucles dentro de bucles para cada nivel de profundidad. Si la estructura tiene diez niveles, el código se vuelve imposible de leer.

### La solución paso a paso

1. Define una **interfaz común** para todo lo que puede haber en el árbol: `Elemento`, con el método `tamano()`.
2. Las **hojas** (elementos sin hijos, como `Fichero`) implementan `tamano()` devolviendo su propio valor.
3. Los **compuestos** (elementos con hijos, como `Carpeta`) guardan una lista de `Elemento` y implementan `tamano()` **sumando el tamaño de sus hijos**.
4. Como una carpeta contiene `Elemento`s, puede contener tanto ficheros como otras carpetas.

```text
proyecto/            ← Carpeta (compuesto)
├── README.md        ← Fichero (hoja)
└── src/             ← Carpeta (compuesto)
    ├── Main.java    ← Fichero (hoja)
    └── Util.java    ← Fichero (hoja)
```

### Programa completo

```java
// Fichero: Composite.java
import java.util.List;

// 1. Lo común a ficheros y carpetas
interface Elemento {
    String nombre();
    long tamano();
}

// 2. Hoja: no contiene nada
record Fichero(String nombre, long bytes) implements Elemento {
    public long tamano() {
        return bytes;
    }
}

// 3. Compuesto: contiene otros elementos (ficheros o carpetas)
record Carpeta(String nombre, List<Elemento> contenido) implements Elemento {
    public long tamano() {
        long total = 0;
        for (Elemento elemento : contenido) {
            total += elemento.tamano();   // da igual si es fichero o carpeta
        }
        return total;
    }
}

void main() {
    Elemento proyecto = new Carpeta("proyecto", List.of(
            new Fichero("README.md", 1_200),
            new Carpeta("src", List.of(
                    new Fichero("Main.java", 3_400),
                    new Fichero("Util.java", 900)))));

    IO.println("Tamaño de proyecto: " + proyecto.tamano() + " bytes");

    Elemento suelto = new Fichero("notas.txt", 150);
    IO.println("Tamaño de notas.txt: " + suelto.tamano() + " bytes");
}
```

Salida:

```text
Tamaño de proyecto: 5500 bytes
Tamaño de notas.txt: 150 bytes
```

### Qué ha pasado

- Se llama a `tamano()` igual sobre una carpeta que sobre un fichero suelto: **el código que usa el árbol no necesita distinguirlos**.
- Cuando una carpeta calcula su tamaño, pide el tamaño a cada hijo. Si el hijo es otra carpeta, esta hace lo mismo con sus hijos. Así se recorre el árbol entero sin escribir un bucle por nivel. Que un método se use a sí mismo de forma indirecta así se llama **recursividad**.
- Los números se pueden escribir con guiones bajos (`1_200`) para leerlos mejor. Java los ignora: `1_200` es `1200`.

### Otra forma de escribirlo (Java moderno)

Desde Java 21, cuando la lista de tipos de nodo es fija, también se puede usar una interfaz `sealed` (que indica exactamente qué clases la implementan) y un `switch` que distingue cada tipo:

```java
sealed interface Nodo permits Archivo, Directorio {}
record Archivo(String nombre, long bytes) implements Nodo {}
record Directorio(String nombre, List<Nodo> hijos) implements Nodo {}

long tamano(Nodo nodo) {
    return switch (nodo) {
        case Archivo a    -> a.bytes();
        case Directorio d -> d.hijos().stream().mapToLong(this::tamano).sum();
    };
}
```

Las dos formas son válidas. La primera (cada clase sabe calcular su tamaño) es la clásica y la más fácil de entender al principio.

### Dónde lo verás

- Menús con submenús, organigramas de empresa, el árbol de componentes de una interfaz gráfica (una ventana contiene paneles, que contienen botones…).

### Cuándo usarlo

- Cuando tus datos tienen forma de **árbol** (un todo formado por partes, que a su vez tienen partes) y quieres tratar igual las partes sueltas y los grupos.

---

## 7. Decorator (decorador)

### En una frase

Añade funcionalidad a un objeto **envolviéndolo** en otro que tiene la misma interfaz, sin modificar la clase original.

### Analogía

En una cafetería pides un café. Puedes añadirle leche, luego nata, luego canela. Cada extra **envuelve** al anterior y suma su precio, pero el resultado sigue siendo «un café». No hace falta que exista en la carta un «café con leche, nata y canela»: se combina sobre la marcha.

### El problema

Si intentas resolverlo con herencia, necesitas una clase por cada combinación posible:

```text
Cafe
├── CafeConLeche
├── CafeConNata
├── CafeConLecheYNata
├── CafeConLecheYCanela
├── CafeConNataYCanela
└── CafeConLecheNataYCanela   ← con 3 extras ya hay 7 clases
```

Cada extra nuevo **duplica** el número de clases.

### La solución paso a paso

1. Define una interfaz común: `Cafe`, con `descripcion()` y `precio()`.
2. Crea la clase básica: `CafeSolo`.
3. Crea **un decorador por extra** (`ConLeche`, `ConNata`…). Cada decorador:
   - implementa también `Cafe`,
   - **guarda dentro** otro `Cafe`,
   - y en cada método llama al café de dentro y **añade lo suyo**.

### Programa completo

```java
// Fichero: Decorator.java

interface Cafe {
    String descripcion();
    double precio();
}

// El objeto básico
record CafeSolo() implements Cafe {
    public String descripcion() { return "Café solo"; }
    public double precio()      { return 1.20; }
}

// Decoradores: cada uno envuelve otro Cafe y añade algo
record ConLeche(Cafe cafe) implements Cafe {
    public String descripcion() { return cafe.descripcion() + " + leche"; }
    public double precio()      { return cafe.precio() + 0.30; }
}

record ConNata(Cafe cafe) implements Cafe {
    public String descripcion() { return cafe.descripcion() + " + nata"; }
    public double precio()      { return cafe.precio() + 0.50; }
}

record ConCanela(Cafe cafe) implements Cafe {
    public String descripcion() { return cafe.descripcion() + " + canela"; }
    public double precio()      { return cafe.precio() + 0.10; }
}

void mostrar(Cafe cafe) {
    IO.println("%s: %.2f €".formatted(cafe.descripcion(), cafe.precio()));
}

void main() {
    Cafe sencillo = new CafeSolo();
    Cafe completo = new ConCanela(new ConNata(new ConLeche(new CafeSolo())));
    Cafe dobleLeche = new ConLeche(new ConLeche(new CafeSolo()));

    mostrar(sencillo);
    mostrar(completo);
    mostrar(dobleLeche);
}
```

Salida:

```text
Café solo: 1,20 €
Café solo + leche + nata + canela: 2,10 €
Café solo + leche + leche: 1,80 €
```

### Qué ha pasado

- `new ConCanela(new ConNata(new ConLeche(new CafeSolo())))` se lee **de dentro hacia fuera**: un café solo, envuelto en leche, envuelto en nata, envuelto en canela.
- Al pedir el precio, cada capa pregunta a la de dentro y suma lo suyo: 1,20 + 0,30 + 0,50 + 0,10 = 2,10.
- Con 3 decoradores se puede crear **cualquier combinación**, incluso repetir un extra (doble de leche), sin crear ninguna clase más.

> **Nota sobre el dinero:** en este ejemplo se usa `double` para que el código sea corto. En una aplicación real, los importes se guardan con `BigDecimal` o en céntimos con `int`/`long`, porque `double` puede producir pequeños errores de redondeo.

### Dónde lo verás en Java

Las clases para leer ficheros son decoradores. Cada una envuelve a la anterior y le añade algo:

```java
var lector = new BufferedReader(          // añade lectura por líneas y un búfer
                 new InputStreamReader(    // convierte bytes en caracteres
                     new FileInputStream("datos.txt")));   // lee bytes del fichero
```

### Cuándo usarlo

- Cuando quieres **añadir funcionalidades opcionales** que se pueden combinar libremente.
- Cuando no puedes (o no quieres) modificar la clase original.

---

## 8. Facade (fachada)

### En una frase

Ofrece **una puerta de entrada sencilla** a un conjunto de piezas complicadas.

### Analogía

En un hotel no hablas con la persona de limpieza, la cocina, el aparcamiento y el servicio de despertador por separado. Hablas con **recepción**, y recepción se encarga de avisar a cada servicio.

### El problema

Comprar en una tienda en línea implica varios pasos, cada uno en una clase distinta: reservar el producto en el almacén, cobrar, programar el envío y avisar al cliente. Si cada pantalla de la aplicación tiene que llamar a las cuatro clases en el orden correcto, el código se repite y es fácil olvidar un paso.

### La solución paso a paso

1. Las clases del «subsistema» (`Almacen`, `Pagos`, `Envios`, `Avisos`) siguen como estaban.
2. Se crea una clase nueva, la **fachada** (`Tienda`), con un método sencillo: `comprar(...)`.
3. Ese método llama a las piezas del subsistema **en el orden correcto**.
4. El resto de la aplicación solo habla con la fachada.

### Programa completo

```java
// Fichero: Facade.java

// Subsistema: varias piezas, cada una con su tarea
class Almacen {
    void reservar(String producto) { IO.println("Almacén: reservado " + producto); }
}

class Pagos {
    void cobrar(String cliente, double importe) {
        IO.println("Pagos: cobrados %.2f € a %s".formatted(importe, cliente));
    }
}

class Envios {
    String programar(String cliente) {
        IO.println("Envíos: envío programado para " + cliente);
        return "ENV-001";
    }
}

class Avisos {
    void enviar(String cliente, String mensaje) { IO.println("Avisos: a " + cliente + " -> " + mensaje); }
}

// Fachada: una sola puerta de entrada
class Tienda {
    private final Almacen almacen = new Almacen();
    private final Pagos pagos = new Pagos();
    private final Envios envios = new Envios();
    private final Avisos avisos = new Avisos();

    String comprar(String cliente, String producto, double precio) {
        almacen.reservar(producto);
        pagos.cobrar(cliente, precio);
        String codigoEnvio = envios.programar(cliente);
        avisos.enviar(cliente, "Pedido confirmado. Envío " + codigoEnvio);
        return codigoEnvio;
    }
}

void main() {
    Tienda tienda = new Tienda();
    String envio = tienda.comprar("Ana", "Portátil", 899.00);   // una sola llamada
    IO.println("Código de envío: " + envio);
}
```

Salida:

```text
Almacén: reservado Portátil
Pagos: cobrados 899,00 € a Ana
Envíos: envío programado para Ana
Avisos: a Ana -> Pedido confirmado. Envío ENV-001
Código de envío: ENV-001
```

### Qué ha pasado

- `main` hace **una sola llamada** (`comprar`) y la fachada se encarga de los cuatro pasos en orden.
- Si mañana el proceso de compra cambia (por ejemplo, se añade una factura), solo se modifica `Tienda`.
- Las piezas del subsistema siguen disponibles por si alguna parte del programa necesita usarlas directamente.

### Cuidado

- La fachada debe **coordinar**, no hacer el trabajo de todos. Si acaba conteniendo toda la lógica del programa, se convierte en un «objeto dios»: una clase enorme que lo sabe y lo hace todo, muy difícil de mantener.
- En un proyecto real, las piezas (`Almacen`, `Pagos`…) se suelen **pasar por el constructor** en lugar de crearse con `new` dentro de la fachada. Así se pueden sustituir fácilmente, por ejemplo en las pruebas.

### Cuándo usarlo

- Cuando usar un conjunto de clases requiere **muchos pasos** y quieres ofrecer una forma sencilla de hacerlo.

---

## 9. Proxy (intermediario)

### En una frase

Pone un **sustituto** delante de un objeto para **controlar el acceso** a él. El sustituto tiene la misma interfaz que el objeto real, así que quien lo usa no nota la diferencia.

### Analogía

Para hablar con la directora de una empresa llamas a su **secretaría**. La secretaría puede contestarte directamente si ya sabe la respuesta, decidir que no tienes permiso o pasarte la llamada. Tú marcas el mismo número en cualquier caso.

### El problema

Una aplicación consulta la previsión del tiempo a un servicio externo. Cada consulta tarda varios segundos y, además, el servicio cobra por consulta. Muchos usuarios preguntan por la misma ciudad una y otra vez, así que se repite el mismo trabajo.

### La solución paso a paso

1. Hay una interfaz, `ServicioTiempo`, y la clase real que hace la consulta lenta.
2. Se crea un **proxy**, `ServicioTiempoConCache`, que **también implementa** `ServicioTiempo` y guarda dentro el servicio real.
3. El proxy guarda las respuestas en una **caché** (un `Map` ciudad → previsión).
4. Cuando le preguntan por una ciudad, mira primero en la caché. Solo si no la tiene, llama al servicio real.

### Programa completo

```java
// Fichero: Proxy.java
import java.util.HashMap;
import java.util.Map;

interface ServicioTiempo {
    String prevision(String ciudad);
}

// Objeto real: lento y costoso
class ServicioTiempoReal implements ServicioTiempo {
    @Override
    public String prevision(String ciudad) {
        IO.println("   (consultando el servicio externo para " + ciudad + "...)");
        return "Soleado en " + ciudad;
    }
}

// Proxy: misma interfaz, controla cuándo se llama al real
class ServicioTiempoConCache implements ServicioTiempo {
    private final ServicioTiempo real;
    private final Map<String, String> cache = new HashMap<>();

    ServicioTiempoConCache(ServicioTiempo real) {
        this.real = real;
    }

    @Override
    public String prevision(String ciudad) {
        if (!cache.containsKey(ciudad)) {
            cache.put(ciudad, real.prevision(ciudad));   // solo la primera vez
        }
        return cache.get(ciudad);
    }
}

void main() {
    ServicioTiempo servicio = new ServicioTiempoConCache(new ServicioTiempoReal());

    IO.println(servicio.prevision("Madrid"));
    IO.println(servicio.prevision("Madrid"));
    IO.println(servicio.prevision("Sevilla"));
    IO.println(servicio.prevision("Madrid"));
}
```

Salida:

```text
   (consultando el servicio externo para Madrid...)
Soleado en Madrid
Soleado en Madrid
   (consultando el servicio externo para Sevilla...)
Soleado en Sevilla
Soleado en Madrid
```

### Qué ha pasado

- Se ha preguntado **cuatro** veces, pero el servicio real solo se ha consultado **dos** (una por ciudad distinta).
- `main` usa `ServicioTiempo` y no sabe si habla con el servicio real o con el proxy.

### Tipos habituales de proxy

| Tipo | Qué controla | Ejemplo |
|---|---|---|
| **De caché** | Evita repetir trabajo costoso | El ejemplo anterior |
| **De protección** | Comprueba permisos antes de dejar pasar | Solo los administradores pueden borrar |
| **Virtual** | Retrasa la creación de un objeto pesado hasta que hace falta | Cargar una imagen grande solo cuando se va a mostrar |
| **Remoto** | Representa un objeto que está en otro ordenador | Llamar a un servicio de Internet como si fuera un objeto local |

### Dónde lo verás

Los *frameworks* como Spring usan proxies constantemente: por ejemplo, para abrir y cerrar transacciones de base de datos alrededor de un método sin que tú escribas ese código. Java incluso permite crear proxies mientras el programa se ejecuta, con la clase `java.lang.reflect.Proxy` (se explica en la guía de [Reflections](<../../03-Anotaciones-y-reflexion/Reflections.md>)).

### Cuándo usarlo

- Cuando necesitas **controlar** cuándo, quién o si se usa un objeto: cachés, permisos, carga perezosa o acceso remoto.

---

## 10. Resumen: se parecen, pero no son iguales

Adapter, Decorator y Proxy tienen un código **casi idéntico**: una clase que guarda otro objeto dentro y le delega el trabajo. Lo que los distingue es **la intención**:

| Patrón | ¿La interfaz de fuera es igual que la de dentro? | ¿Para qué envuelve? | Pregunta para reconocerlo |
|---|---|---|---|
| **Adapter** | **No**: la convierte en otra | Hacer que encaje | ¿Estoy traduciendo de una forma a otra? |
| **Decorator** | Sí | **Añadir** algo | ¿Estoy sumando funcionalidad? |
| **Proxy** | Sí | **Controlar** el acceso | ¿Estoy decidiendo cuándo o si se llama al objeto real? |
| **Facade** | No: ofrece una más sencilla | Simplificar muchas piezas | ¿Estoy escondiendo varios pasos detrás de uno? |
| **Composite** | Sí (hojas y grupos comparten interfaz) | Formar un árbol | ¿Tengo partes que contienen partes? |

---

## 11. Errores frecuentes

| Error | Por qué es un problema | Qué hacer |
|---|---|---|
| Modificar la librería externa en lugar de adaptarla | Al actualizar la librería se pierden los cambios. | Escribir un Adapter. |
| Crear una subclase por cada combinación de extras | El número de clases crece sin control. | Usar Decorator. |
| En un Composite, preguntar «¿es fichero o carpeta?» en el código que lo usa | Se pierde la ventaja de tratar todo igual. | Poner el comportamiento en la interfaz común (`tamano()`). |
| Meter toda la lógica en la fachada | Se convierte en una clase enorme e imposible de mantener. | La fachada solo coordina; cada pieza hace su trabajo. |
| Que el proxy o el decorador no implementen la misma interfaz que el objeto real | Ya no se pueden usar en su lugar. | Implementar siempre la misma interfaz. |
| Añadir envoltorios «por si acaso» | Cada capa hace el código más difícil de seguir. | Añadirlos solo cuando resuelvan un problema real. |

---

## 12. Ejercicios

**Ejercicio 1 (Decorator).** Añade al ejemplo del café un decorador `ConSirope` que cueste 0,40 €. Crea un café solo con sirope y nata.

<details>
<summary>Solución</summary>

```java
record ConSirope(Cafe cafe) implements Cafe {
    public String descripcion() { return cafe.descripcion() + " + sirope"; }
    public double precio()      { return cafe.precio() + 0.40; }
}

// En main:
mostrar(new ConNata(new ConSirope(new CafeSolo())));
// Café solo + sirope + nata: 2,10 €
```

No ha hecho falta modificar ninguna clase existente.

</details>

**Ejercicio 2 (Proxy).** Crea un proxy de protección para este servicio, que solo deje borrar si el usuario es `"admin"` y, si no, lance `SecurityException`:

```java
interface Documentos {
    void borrar(String usuario, String documento);
}
```

<details>
<summary>Solución</summary>

```java
class DocumentosReales implements Documentos {
    public void borrar(String usuario, String documento) {
        IO.println(documento + " borrado por " + usuario);
    }
}

class DocumentosProtegidos implements Documentos {
    private final Documentos real;

    DocumentosProtegidos(Documentos real) { this.real = real; }

    public void borrar(String usuario, String documento) {
        if (!usuario.equals("admin")) {
            throw new SecurityException(usuario + " no puede borrar documentos");
        }
        real.borrar(usuario, documento);
    }
}
```

</details>

**Ejercicio 3 (Composite).** Añade a la interfaz `Elemento` un método `int contarFicheros()`. Un fichero devuelve 1; una carpeta, la suma de lo que devuelven sus elementos. ¿Cuántos ficheros tiene `proyecto`?

<details>
<summary>Solución</summary>

```java
// En Fichero:
public int contarFicheros() { return 1; }

// En Carpeta:
public int contarFicheros() {
    int total = 0;
    for (Elemento elemento : contenido) {
        total += elemento.contarFicheros();
    }
    return total;
}
```

`proyecto` tiene **3** ficheros: `README.md`, `Main.java` y `Util.java`.

</details>

---

## 13. Glosario

| Término | Significado |
|---|---|
| **Patrón de diseño** | Solución conocida y con nombre para un problema de diseño que se repite. |
| **Composición** | Que un objeto guarde dentro otro objeto y lo use. |
| **Delegar** | Pasar una tarea a otro objeto para que la haga. |
| **Envoltorio** (*wrapper*) | Objeto que guarda otro dentro y se coloca delante de él. |
| **Subsistema** | Conjunto de clases que, juntas, realizan una tarea grande. |
| **Hoja** | En un árbol, elemento que no contiene otros (un fichero). |
| **Compuesto** | En un árbol, elemento que contiene otros (una carpeta). |
| **Recursividad** | Cuando un método se usa a sí mismo, directa o indirectamente, para resolver partes más pequeñas del problema. |
| **Caché** | Lugar donde se guardan resultados ya calculados para no repetir el trabajo. |
| **Objeto dios** | Clase que ha acumulado demasiadas responsabilidades. Es un error de diseño. |
