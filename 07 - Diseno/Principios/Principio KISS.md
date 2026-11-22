# Principio KISS (*Keep It Simple, Stupid*)

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).

## Índice

1. [¿Qué es KISS?](#1-qué-es-kiss)
2. [Origen del principio](#2-origen-del-principio)
3. [Simple no significa fácil ni corto](#3-simple-no-significa-fácil-ni-corto)
4. [¿Por qué aplicar KISS?](#4-por-qué-aplicar-kiss)
5. [Estrategias para aplicar KISS](#5-estrategias-para-aplicar-kiss)
6. [Ejemplo completo: conversor de temperaturas](#6-ejemplo-completo-conversor-de-temperaturas)
7. [Herramientas de Java moderno que simplifican el código](#7-herramientas-de-java-moderno-que-simplifican-el-código)
8. [Señales de que el código no es simple](#8-señales-de-que-el-código-no-es-simple)
9. [KISS y otros principios](#9-kiss-y-otros-principios)
10. [Lista de comprobación](#10-lista-de-comprobación)
11. [Referencias](#11-referencias)

---

## 1. ¿Qué es KISS?

**KISS** son las siglas de *Keep It Simple, Stupid* («mantenlo simple, estúpido»; a menudo suavizado como *Keep It Short and Simple*). El principio anima a **evitar la complejidad innecesaria** y a escribir código **directo y claro**.

Aunque el nombre es corto, KISS engloba muchas técnicas de refactorización orientadas a mantener la sencillez durante todo el desarrollo. Es un concepto flexible que se puede aplicar de muchas formas para conseguir un código más limpio y fácil de mantener.

## 2. Origen del principio

La expresión se atribuye a **Kelly Johnson**, ingeniero jefe de los *Skunk Works* de Lockheed, en la década de 1960. La idea era que los aviones debían poder repararse con herramientas básicas y en condiciones difíciles. Trasladado al software: **el código debe poder entenderlo y modificarlo otra persona sin esfuerzo extra**.

## 3. Simple no significa fácil ni corto

Es frecuente confundir «simple» con «pocas líneas». No es lo mismo:

| Concepto | Significado | Ejemplo |
|---|---|---|
| **Simple** | Pocas piezas, poca interacción entre ellas, intención evidente. | Un método `calcularDescuento` que hace solo eso. |
| **Fácil** | Familiar para quien lo escribe. | Copiar una solución de otro proyecto que no se entiende del todo. |
| **Corto** | Pocas líneas. | Una sola expresión de *stream* con cinco operaciones encadenadas y lambdas anidadas. |

Un código corto puede ser muy complejo de leer, y un código algo más largo puede ser mucho más simple. **El objetivo es que se entienda a la primera.**

```java
// ❌ Corto, pero obliga a pararse a descifrarlo
return a > b ? (a > c ? a : c) : (b > c ? b : c);

// ✅ Algo más largo, pero evidente
return Math.max(a, Math.max(b, c));
```

## 4. ¿Por qué aplicar KISS?

- **Mantenimiento más sencillo:** el código simple es intrínsecamente más adaptable; modificarlo y actualizarlo cuesta menos.
- **Comunicación clara:** un código fácil de leer facilita la colaboración y la comprensión entre las personas del equipo.
- **Pruebas más sencillas:** una lógica más simple reduce la complejidad de las pruebas automatizadas y mejora la fiabilidad de las pruebas unitarias y de integración.
- **Menos errores:** cuantas menos ramas, estados y casos especiales, menos sitios donde puede esconderse un fallo.

## 5. Estrategias para aplicar KISS

1. **Escribir programas más pequeños.** Métodos y clases concisos que resuelvan **un problema cada vez**.
2. **Eliminar el código que no se usa.** Métodos, clases, parámetros y variables que no sirven a ningún propósito solo añaden ruido y confusión. El control de versiones (Git) ya guarda el historial; no hace falta dejar código comentado «por si acaso».
3. **Priorizar la legibilidad.** Nombres expresivos, un nivel de abstracción por método, pocas anidaciones.
4. **Usar composición.** Combinar piezas simples ya existentes en lugar de reescribir funcionalidad.
5. **Programar de forma modular.** Dividir la aplicación en módulos que puedan funcionar de manera independiente; esto ayuda a organizar el código y lo hace más flexible.
6. **Aprovechar la biblioteca estándar.** Antes de escribir un algoritmo, comprobar si Java ya lo ofrece (`Collections`, `Stream`, `java.time`, `String`…).
7. **No adelantarse a necesidades futuras** (ver YAGNI en el apartado 9).

## 6. Ejemplo completo: conversor de temperaturas

### ❌ Versión complicada

```java
public class ConversorTemperatura {

    // 1 = Fahrenheit, 2 = Celsius, 3 = Kelvin
    public static double convertir(double temperatura, int desde, int hasta) {
        if (desde == 1 && hasta == 2) {
            return (temperatura - 32) * 5 / 9;
        } else if (desde == 2 && hasta == 1) {
            return (temperatura * 9 / 5) + 32;
        } else if (desde == 1 && hasta == 3) {
            return (temperatura - 32) * 5 / 9 + 273.15;
        } else if (desde == 3 && hasta == 1) {
            return (temperatura - 273.15) * 9 / 5 + 32;
        } else if (desde == 2 && hasta == 3) {
            return temperatura + 273.15;
        } else if (desde == 3 && hasta == 2) {
            return temperatura - 273.15;
        }
        return temperatura;
    }
}
```

Problemas:

- Los **números mágicos** (`1`, `2`, `3`) obligan a recordar qué significa cada uno. Nada impide pasar un `7`.
- Cada escala nueva multiplica las combinaciones: con 3 escalas hay 6 ramas; con 4 escalas habría 12.
- Las fórmulas se repiten (Fahrenheit → Celsius aparece dos veces), lo que además viola DRY.

### ✅ Versión simple: pasar siempre por una escala intermedia

La idea clave es convertir **primero a Celsius** y **después a la escala de destino**. Así, cada escala solo necesita saber dos cosas: cómo pasar a Celsius y cómo volver de Celsius.

```java
public enum Escala {
    CELSIUS,
    FAHRENHEIT,
    KELVIN;

    double aCelsius(double valor) {
        return switch (this) {
            case CELSIUS    -> valor;
            case FAHRENHEIT -> (valor - 32) * 5 / 9;
            case KELVIN     -> valor - 273.15;
        };
    }

    double desdeCelsius(double celsius) {
        return switch (this) {
            case CELSIUS    -> celsius;
            case FAHRENHEIT -> celsius * 9 / 5 + 32;
            case KELVIN     -> celsius + 273.15;
        };
    }
}
```

```java
public final class ConversorTemperatura {

    private ConversorTemperatura() { }

    public static double convertir(double valor, Escala desde, Escala hasta) {
        return hasta.desdeCelsius(desde.aCelsius(valor));
    }
}
```

Uso con un fichero fuente compacto (Java 25, JEP 512):

```java
void main() {
    IO.println(ConversorTemperatura.convertir(100, Escala.CELSIUS, Escala.FAHRENHEIT)); // 212.0
    IO.println(ConversorTemperatura.convertir(0, Escala.KELVIN, Escala.CELSIUS));        // -273.15
}
```

Mejoras obtenidas:

- **Tipos en lugar de números mágicos:** el compilador impide valores no válidos.
- **`switch` como expresión** con flechas (`->`): sin `break` olvidados ni caídas accidentales entre casos.
- **Exhaustividad comprobada por el compilador:** al hacer `switch` sobre un `enum` como expresión, si se añade `RANKINE` y se olvida un caso, el código **no compila**.
- **Crecimiento lineal:** una escala nueva añade dos líneas, no seis ramas.

## 7. Herramientas de Java moderno que simplifican el código

### 7.1. Ficheros fuente compactos y métodos `main` de instancia (Java 25)

Desde Java 25 (JEP 512) un programa pequeño no necesita declarar una clase, ni `public static`, ni `String[] args`:

```java
// Fichero Hola.java — se ejecuta con: java Hola.java
void main() {
    String nombre = IO.readln("¿Cómo te llamas? ");
    IO.println("Hola, " + nombre);
}
```

La clase `java.lang.IO` ofrece `println`, `print` y `readln` para entrada y salida por consola. Además, en un fichero compacto se importan automáticamente todos los paquetes exportados por el módulo `java.base` (`java.util`, `java.time`, etc.). Es ideal para *scripts*, prototipos y ejemplos.

### 7.2. Importación de módulos (Java 25)

Con JEP 511 (*Module Import Declarations*) una sola línea importa todos los paquetes públicos de un módulo:

```java
import module java.base;   // List, Map, Path, Stream, LocalDate, Function...
```

Úsalo con criterio: en proyectos grandes, las importaciones explícitas siguen dejando más claro de dónde viene cada tipo.

### 7.3. `record` en lugar de clases de datos manuales

```java
// ❌ ~40 líneas con constructor, getters, equals, hashCode y toString escritos a mano
// ✅ Una línea
public record Punto(double x, double y) { }
```

### 7.4. `switch` con patrones y `sealed` (Java 21)

Permiten sustituir cadenas de `instanceof` y conversiones explícitas:

```java
sealed interface Forma permits Circulo, Rectangulo { }
record Circulo(double radio) implements Forma { }
record Rectangulo(double ancho, double alto) implements Forma { }

static double area(Forma forma) {
    return switch (forma) {
        case Circulo(double r)              -> Math.PI * r * r;
        case Rectangulo(double an, double al) -> an * al;
    };
}
```

No hace falta `default`: como la interfaz es `sealed`, el compilador sabe que no hay más casos.

### 7.5. Variables sin nombre `_` (Java 22)

Cuando una variable es obligatoria por sintaxis pero no se usa, `_` deja clara la intención:

```java
try {
    Integer.parseInt(texto);
    return true;
} catch (NumberFormatException _) {
    return false;
}
```

### 7.6. *Guard clauses* (cláusulas de salida temprana)

Reducen la anidación devolviendo pronto en los casos especiales:

```java
// ❌ Anidación profunda
double precioFinal(Pedido pedido) {
    if (pedido != null) {
        if (!pedido.lineas().isEmpty()) {
            if (pedido.cliente().esVip()) {
                return pedido.total() * 0.9;
            } else {
                return pedido.total();
            }
        }
    }
    return 0;
}

// ✅ Salidas tempranas
double precioFinal(Pedido pedido) {
    if (pedido == null || pedido.lineas().isEmpty()) {
        return 0;
    }
    if (pedido.cliente().esVip()) {
        return pedido.total() * 0.9;
    }
    return pedido.total();
}
```

> Para importes reales usa `BigDecimal`, no `double`. Aquí se usa `double` solo para que el ejemplo sea breve.

### 7.7. Usar `var` con criterio

`var` (Java 10) evita repetir tipos evidentes, pero puede esconder información si el tipo no se deduce a simple vista:

```java
var clientes = new ArrayList<Cliente>();   // ✅ el tipo es obvio
var resultado = servicio.procesar(datos);  // ⚠️ ¿qué devuelve procesar?
```

## 8. Señales de que el código no es simple

- Métodos que no caben en una pantalla.
- Más de dos o tres niveles de sangrado.
- Parámetros booleanos que cambian por completo el comportamiento (`exportar(datos, true, false)`).
- Nombres genéricos: `Manager`, `Helper`, `Util`, `procesar`, `hacerCosas`.
- Patrones de diseño aplicados «por si acaso» cuando el problema no los necesita.
- Comentarios que explican **qué** hace un bloque enrevesado (en lugar de reescribirlo para que se entienda solo).
- Código comentado o que nunca se ejecuta.

## 9. KISS y otros principios

- **YAGNI** (*You Aren't Gonna Need It*, «no lo vas a necesitar»): no implementar funcionalidad hasta que haga falta de verdad. Es un aliado natural de KISS.
- **DRY**: eliminar duplicación suele simplificar, pero una abstracción forzada puede complicar. Si unificar código lo hace más difícil de entender, KISS gana.
- **SRP** (de SOLID): clases con una sola responsabilidad son, por naturaleza, más simples.
- **Ley de Deméter**: limitar con cuántos objetos «habla» un método reduce la complejidad de las interacciones (ver el documento *Tell, Don't Ask*).

## 10. Lista de comprobación

- [ ] ¿Se entiende lo que hace cada método leyendo solo su nombre y su cuerpo?
- [ ] ¿Hay números o cadenas «mágicas» que deberían ser `enum` o constantes?
- [ ] ¿Hay código sin usar o comentado que pueda eliminarse?
- [ ] ¿Se puede reducir la anidación con salidas tempranas?
- [ ] ¿Existe ya en la biblioteca estándar lo que estoy programando?
- [ ] ¿Estoy añadiendo algo «por si en el futuro…»?
- [ ] ¿Podría otra persona modificar este código sin pedirme ayuda?

## 11. Referencias

- CodeSignal — *KISS (Keep It Simple, Stupid) Strategies*: <https://codesignal.com/learn/courses/applying-clean-code-principles/lessons/kiss-keep-it-simple-stupid-strategies>
- JEP 512 — *Compact Source Files and Instance Main Methods* (Java 25): <https://openjdk.org/jeps/512>
- JEP 511 — *Module Import Declarations* (Java 25): <https://openjdk.org/jeps/511>
- JEP 441 — *Pattern Matching for switch* (Java 21): <https://openjdk.org/jeps/441>
- JEP 440 — *Record Patterns* (Java 21): <https://openjdk.org/jeps/440>
- JEP 456 — *Unnamed Variables & Patterns* (Java 22): <https://openjdk.org/jeps/456>
