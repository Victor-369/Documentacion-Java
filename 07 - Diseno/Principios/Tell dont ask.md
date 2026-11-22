# Principio *Tell, Don't Ask* («Ordena, no preguntes»)

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).

## Índice

1. [Punto de partida: métodos de acceso y modificación](#1-punto-de-partida-métodos-de-acceso-y-modificación)
2. [¿Getters y setters o campos públicos?](#2-getters-y-setters-o-campos-públicos)
3. [Cómo los getters y setters pueden perjudicar](#3-cómo-los-getters-y-setters-pueden-perjudicar)
4. [El principio *Tell, Don't Ask*](#4-el-principio-tell-dont-ask)
5. [Ejemplo de negocio: una cuenta bancaria](#5-ejemplo-de-negocio-una-cuenta-bancaria)
6. [La Ley de Deméter](#6-la-ley-de-deméter)
7. [*Tell, Don't Ask* en Java moderno](#7-tell-dont-ask-en-java-moderno)
8. [Cuándo está bien preguntar](#8-cuándo-está-bien-preguntar)
9. [Relación con otros principios](#9-relación-con-otros-principios)
10. [Lista de comprobación](#10-lista-de-comprobación)
11. [Referencias](#11-referencias)

---

## 1. Punto de partida: métodos de acceso y modificación

Además de constructores, una clase puede ofrecer métodos para **consultar** o **modificar** sus propiedades:

- **Accesores** (*accessors* o *getters*): devuelven el valor de un campo.
- **Modificadores** (*mutators* o *setters*): cambian el valor de un campo.

```java
class Circulo {
    private double x;
    private double y;
    private double r;

    Circulo(double x, double y, double r) {
        this.x = x;
        this.y = y;
        this.r = r;
    }

    double getX() { return x; }
    void setX(double x) { this.x = x; }

    double getY() { return y; }
    void setY(double y) { this.y = y; }

    double getR() { return r; }
    void setR(double r) { this.r = r; }

    double getArea() {
        return Math.PI * r * r;
    }
}
```

| Campo | Accesor | Modificador |
|---|---|---|
| `x` | `getX` | `setX` |
| `y` | `getY` | `setY` |
| `r` | `getR` | `setR` |

## 2. ¿Getters y setters o campos públicos?

Si cada campo privado tiene su *getter* y su *setter*, ¿no sería lo mismo hacerlos `public`? No del todo. Tener métodos es **mejor** que tener campos públicos porque añade una capa de abstracción:

- Se puede **renombrar o cambiar** un campo interno sin que el código cliente lo note.
- Se pueden **validar** los valores antes de asignarlos:

```java
void setR(double r) {
    if (r <= 0) {
        throw new IllegalArgumentException("El radio debe ser positivo");
    }
    this.r = r;
}
```

Ahora bien, que sea *mejor* que un campo público no significa que sea *bueno* generar siempre un *getter* y un *setter* por cada campo.

## 3. Cómo los getters y setters pueden perjudicar

Imagina una primera versión de `Circulo` que usaba `int` porque quien la diseñó no previó que harían falta decimales:

```java
class Circulo {
    private int x;
    private int y;
    private int r;

    int getX() { return x; }
    int getY() { return y; }
    int getR() { return r; }
    // ...
}
```

Un cliente quiere saber si el punto `(px, py)` está dentro del círculo `c`:

```java
int cx = c.getX();
int cy = c.getY();
int r  = c.getR();
boolean dentro = (px - cx) * (px - cx) + (py - cy) * (py - cy) <= r * r;
```

Un día se cambia la representación interna a `double`. **El código cliente deja de compilar**, porque los accesores ahora devuelven `double` y el cliente los guardaba en `int`.

Los *getters* y *setters* **filtran detalles internos** de la clase hacia fuera. Es una forma de **acoplamiento** entre el cliente y la clase: el cliente depende de cómo está representado el círculo por dentro. Cualquier cambio interno lo rompe.

Además, la lógica «¿contiene un punto?» queda **fuera** de la clase que tiene los datos. Si otros tres clientes necesitan lo mismo, cada uno escribirá su propia versión (violando DRY), y quizá alguno con un error.

## 4. El principio *Tell, Don't Ask*

> **Dile a un objeto lo que tiene que hacer, en lugar de pedirle su estado y hacer tú el trabajo por él.**

Según Martin Fowler, este principio se asocia sobre todo a Andy Hunt y Dave Thomas (autores de *The Pragmatic Programmer*). Su idea de fondo es recordar que la orientación a objetos consiste en **agrupar los datos con las funciones que operan sobre ellos**.

### ❌ Preguntar (*ask*)

```text
Cliente ──getX()──▶ Circulo
Cliente ──getY()──▶ Circulo
Cliente ──getR()──▶ Circulo
Cliente: calcula por su cuenta si el punto está dentro
```

### ✅ Ordenar (*tell*)

Se añade el comportamiento **en la clase que tiene los datos**:

```java
boolean contiene(double px, double py) {
    double dx = px - x;
    double dy = py - y;
    return dx * dx + dy * dy <= r * r;
}
```

Y el cliente simplemente se lo pide:

```java
boolean dentro = c.contiene(px, py);
```

```text
Cliente ──contiene(px, py)──▶ Circulo
Cliente ◀──────respuesta────── Circulo
```

Supone escribir unas pocas líneas más en la clase, pero:

- La **encapsulación** queda intacta.
- Hay **menos acoplamiento**: el cliente no sabe si el círculo guarda `int`, `double` o coordenadas polares.
- La clase puede cambiar su estructura interna sin afectar a nadie.

> **Regla general:** una tarea que solo opera sobre los campos de una clase debe implementarse **dentro de esa clase**.

Un buen ejercicio de aprendizaje es diseñar las clases **sin** *getters* ni *setters* al principio, para obligarse a pensar en términos de objetos: como cliente, *decir* al objeto qué tarea realizar; como implementador, programar esa tarea como un método de la clase.

## 5. Ejemplo de negocio: una cuenta bancaria

### ❌ Versión «preguntona»

```java
public class Cuenta {
    private BigDecimal saldo;
    private boolean bloqueada;

    public BigDecimal getSaldo() { return saldo; }
    public void setSaldo(BigDecimal saldo) { this.saldo = saldo; }
    public boolean isBloqueada() { return bloqueada; }
}

public class ServicioPagos {

    public void cobrar(Cuenta cuenta, BigDecimal importe) {
        if (cuenta.isBloqueada()) {
            throw new IllegalStateException("Cuenta bloqueada");
        }
        if (cuenta.getSaldo().compareTo(importe) < 0) {
            throw new IllegalStateException("Saldo insuficiente");
        }
        cuenta.setSaldo(cuenta.getSaldo().subtract(importe));
    }
}
```

Problemas:

- Las **reglas de la cuenta** (no operar si está bloqueada, no quedar en negativo) están en `ServicioPagos`.
- Cualquier otra clase puede llamar a `setSaldo` y saltarse las reglas.
- Si otro servicio (transferencias, comisiones) necesita retirar dinero, tendrá que repetir las comprobaciones.

### ✅ Versión que «ordena»

```java
public class Cuenta {
    private BigDecimal saldo;
    private boolean bloqueada;

    public Cuenta(BigDecimal saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void retirar(BigDecimal importe) {
        if (bloqueada) {
            throw new IllegalStateException("Cuenta bloqueada");
        }
        if (saldo.compareTo(importe) < 0) {
            throw new IllegalStateException("Saldo insuficiente");
        }
        saldo = saldo.subtract(importe);
    }

    public void bloquear() {
        bloqueada = true;
    }
}

public class ServicioPagos {

    public void cobrar(Cuenta cuenta, BigDecimal importe) {
        cuenta.retirar(importe);
    }
}
```

Ahora la cuenta **protege sus propias reglas** (sus *invariantes*). Es imposible dejarla en un estado incorrecto desde fuera.

> Este paso de «objetos que solo tienen datos y servicios que tienen toda la lógica» a «objetos con datos **y** comportamiento» es la diferencia entre un **modelo de dominio anémico** y un **modelo de dominio rico**.

## 6. La Ley de Deméter

La **Ley de Deméter** (o «principio del mínimo conocimiento»), formulada en la Northeastern University a finales de los años ochenta, está muy relacionada con *Tell, Don't Ask*. Dice que un método solo debería «hablar» con:

- el propio objeto (`this`);
- sus parámetros;
- los objetos que crea;
- sus campos directos.

Es decir, **no hablar con desconocidos**. El síntoma típico de incumplirla son las cadenas de llamadas (*train wrecks*):

```java
// ❌ El cliente conoce toda la estructura interna
String ciudad = pedido.getCliente().getDireccion().getCiudad().getNombre();
```

```java
// ✅ Se pregunta a quien corresponde lo que realmente se necesita
String ciudad = pedido.ciudadDeEntrega();
```

o, aún mejor siguiendo *Tell, Don't Ask*, se ordena directamente la acción que motivaba la consulta:

```java
pedido.programarEntrega(calendarioRepartos);
```

> Las cadenas de *streams* (`lista.stream().filter(...).map(...).toList()`) **no** violan la Ley de Deméter: cada llamada devuelve un objeto del mismo «nivel» con el que se sigue trabajando, no se navega por la estructura interna de otro objeto.

## 7. *Tell, Don't Ask* en Java moderno

### 7.1. Los `record` y sus métodos de acceso

Un `record` genera automáticamente un método de acceso por cada componente (`punto.x()`, `punto.y()`). Esto **es intencionado**: un `record` es un **portador transparente de datos** y sus componentes forman parte de su contrato público.

Por eso:

- Usa `record` para **valores** cuya representación es su propia definición (`Punto`, `Rango`, `Dinero`, un DTO, un evento).
- Aun así, **añade comportamiento** al `record` cuando la operación pertenece al concepto:

```java
public record Circulo(double x, double y, double radio) {

    public Circulo {
        if (radio <= 0) {
            throw new IllegalArgumentException("El radio debe ser positivo");
        }
    }

    public boolean contiene(double px, double py) {
        double dx = px - x;
        double dy = py - y;
        return dx * dx + dy * dy <= radio * radio;
    }

    public double area() {
        return Math.PI * radio * radio;
    }
}
```

- Para **entidades con reglas y estado que cambia** (como `Cuenta`), usa una clase normal con campos privados y métodos que expresen acciones.

### 7.2. Inmutabilidad: «ordenar» devolviendo un nuevo valor

En objetos inmutables, «ordenar» significa pedir una **nueva versión** del objeto, en lugar de usar *setters*:

```java
public record Rango(int inicio, int fin) {

    public Rango {
        if (inicio > fin) {
            throw new IllegalArgumentException("inicio > fin");
        }
    }

    public Rango desplazar(int cantidad) {
        return new Rango(inicio + cantidad, fin + cantidad);
    }

    public boolean incluye(int valor) {
        return valor >= inicio && valor <= fin;
    }
}
```

### 7.3. Polimorfismo en lugar de preguntar por el tipo

Preguntar «¿de qué tipo eres?» para decidir qué hacer es otra forma de *ask*:

```java
// ❌ Se pregunta el tipo y se actúa desde fuera
if (empleado.getTipo().equals("GERENTE")) {
    sueldo = empleado.getBase() * 1.5;
} else if (empleado.getTipo().equals("BECARIO")) {
    sueldo = 800;
}
```

```java
// ✅ Se le dice al objeto que calcule su sueldo
public sealed interface Empleado permits Gerente, Becario, Tecnico {
    BigDecimal sueldo();
}
```

Cada implementación sabe calcular su propio sueldo. Es la aplicación directa del principio abierto/cerrado (ver el documento de SOLID).

> **Matiz:** el `switch` con patrones sobre una jerarquía `sealed` (Java 21) es legítimo cuando la operación **no** pertenece a los objetos (por ejemplo, convertirlos a JSON o pintarlos en pantalla desde otra capa). En ese caso es preferible a llenar las clases del dominio de responsabilidades ajenas. Elegir entre ambos enfoques es una decisión de diseño: si la operación forma parte del concepto, va dentro del objeto; si es de otra capa, puede ir fuera.

## 8. Cuándo está bien preguntar

*Tell, Don't Ask* es una guía, no una prohibición absoluta. Es razonable consultar datos cuando:

- **Se muestran en pantalla o se serializan:** una vista, un informe o un JSON necesitan leer los datos.
- **Se trata de objetos de datos puros:** DTO, eventos, mensajes, `record` de transferencia.
- **Se consulta para decidir algo que no es responsabilidad del objeto:** por ejemplo, un planificador que pregunta la fecha de un evento para ordenarlos.
- **Lo exigen herramientas o frameworks:** algunas librerías de persistencia o serialización necesitan métodos de acceso.

Lo que debe evitarse es **pedir datos para tomar una decisión que le corresponde al propio objeto y luego modificarlo desde fuera**.

## 9. Relación con otros principios

- **Encapsulación:** *Tell, Don't Ask* es, en el fondo, una forma de proteger la encapsulación.
- **Acoplamiento:** reduce el acoplamiento de contenido y de sello (ver *Cohesión y acoplamiento*).
- **Cohesión:** coloca la lógica junto a los datos que usa, lo que aumenta la cohesión informacional (ver *Cohesión y acoplamiento*).
- **DRY:** al centralizar la lógica en el objeto, se evita que cada cliente la repita.
- **SRP:** cuidado con no convertir la clase en un «cajón de sastre». Si una operación no pertenece al concepto, no debe ir en él solo por seguir *Tell, Don't Ask*.

## 10. Lista de comprobación

- [ ] ¿Estoy pidiendo datos a un objeto para hacer un cálculo que solo usa sus datos?
- [ ] ¿Hay `setX(getX() + ...)` desde fuera de la clase?
- [ ] ¿Las reglas de un objeto se comprueban fuera de él?
- [ ] ¿Hay cadenas `a.getB().getC().getD()`?
- [ ] ¿Pregunto por el tipo de un objeto para decidir qué hacer con él?
- [ ] ¿Mis *setters* existen porque hacen falta o porque el IDE los generó?
- [ ] ¿Uso `record` para valores y clases con comportamiento para entidades?

## 11. Referencias

- NUS CS2030S — *Unit 6: Tell, Don't Ask*: <https://nus-cs2030s.github.io/2425-s2/06-tell-dont-ask.html>
- Martin Fowler — *TellDontAsk*: <https://martinfowler.com/bliki/TellDontAsk.html>
- JEP 395 — *Records* (Java 16): <https://openjdk.org/jeps/395>
- JEP 409 — *Sealed Classes* (Java 17): <https://openjdk.org/jeps/409>
