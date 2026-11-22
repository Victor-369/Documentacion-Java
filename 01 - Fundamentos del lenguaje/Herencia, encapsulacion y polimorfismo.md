# Herencia, encapsulación y polimorfismo en Java (Java 25+)

> **Alcance:** esta guía explica tres de los pilares de la programación orientada a objetos —encapsulación, herencia y polimorfismo— con la sintaxis actual de Java 25 (versión LTS): constructores flexibles, clases selladas, *records* y `switch` con patrones. Todos los programas completos se han compilado y ejecutado con OpenJDK 25 y su salida es la que se muestra.

---

## Índice

**Antes de empezar**

0. [Cómo probar los ejemplos](#0-cómo-probar-los-ejemplos)
1. [Los pilares de la programación orientada a objetos](#1-los-pilares-de-la-programación-orientada-a-objetos)
2. [Repaso: clases y objetos](#2-repaso-clases-y-objetos)

**Encapsulación**

3. [Qué es la encapsulación](#3-qué-es-la-encapsulación)
4. [Modificadores de acceso](#4-modificadores-de-acceso)
5. [Getters y setters](#5-getters-y-setters)
6. [Invariantes: objetos siempre válidos](#6-invariantes-objetos-siempre-válidos)
7. [Inmutabilidad y records](#7-inmutabilidad-y-records)

**Herencia**

8. [Qué es la herencia](#8-qué-es-la-herencia)
9. [Qué se hereda y qué no](#9-qué-se-hereda-y-qué-no)
10. [Constructores y super](#10-constructores-y-super)
11. [Sobrescritura de métodos](#11-sobrescritura-de-métodos)
12. [La clase Object: toString, equals y hashCode](#12-la-clase-object-tostring-equals-y-hashcode)
13. [final: impedir la herencia y la sobrescritura](#13-final-impedir-la-herencia-y-la-sobrescritura)
14. [Clases abstractas](#14-clases-abstractas)
15. [Interfaces](#15-interfaces)
16. [Clases e interfaces selladas](#16-clases-e-interfaces-selladas)
17. [Composición frente a herencia](#17-composición-frente-a-herencia)

**Polimorfismo**

18. [Qué es el polimorfismo](#18-qué-es-el-polimorfismo)
19. [Tipo declarado y tipo real](#19-tipo-declarado-y-tipo-real)
20. [Conversiones: upcasting y downcasting](#20-conversiones-upcasting-y-downcasting)
21. [Polimorfismo con colecciones y parámetros](#21-polimorfismo-con-colecciones-y-parámetros)
22. [Polimorfismo con tipos sellados y switch](#22-polimorfismo-con-tipos-sellados-y-switch)

**Para terminar**

23. [Ejemplo completo: nóminas de una empresa](#23-ejemplo-completo-nóminas-de-una-empresa)
24. [Errores frecuentes](#24-errores-frecuentes)
25. [Chuleta rápida](#25-chuleta-rápida)
26. [Ejercicios propuestos](#26-ejercicios-propuestos)
27. [Soluciones](#27-soluciones)
28. [Glosario](#28-glosario)
29. [Referencias](#29-referencias)

---

## 0. Cómo probar los ejemplos

Los programas completos usan un **fichero fuente compacto** de Java 25: un `void main()` y, debajo, las clases que necesita el ejemplo, todo en un único fichero. Se ejecutan sin compilar antes:

```bash
java Ejemplo.java
```

En un proyecto real **cada clase pública va en su propio fichero** (`Animal.java`, `Perro.java`...) y los ficheros se organizan en paquetes. Juntarlo todo en un fichero solo sirve para que los ejemplos sean fáciles de probar.

Los fragmentos cortos (sin `void main()`) muestran solo la parte relevante del código.

> 💡 Si necesitas repasar variables, métodos o `switch`, consulta antes la guía [Fundamentos de Java](<Fundamentos de Java.md>).

---

## 1. Los pilares de la programación orientada a objetos

La **programación orientada a objetos** (POO) organiza un programa en **objetos** que combinan **datos** (estado) y **comportamiento** (métodos). Se apoya en cuatro ideas:

| Pilar | Idea | Pregunta que responde |
|---|---|---|
| **Abstracción** | Quedarse con lo esencial y ocultar los detalles | ¿Qué hace este objeto? |
| **Encapsulación** | Proteger el estado interno y exponer solo una interfaz controlada | ¿Quién puede tocar estos datos y cómo? |
| **Herencia** | Crear una clase a partir de otra, reutilizando y especializando | ¿Es este objeto un tipo especial de otro? |
| **Polimorfismo** | Tratar objetos distintos de la misma forma; cada uno responde a su manera | ¿Puedo usar este objeto sin saber su clase exacta? |

Los cuatro se apoyan entre sí: la encapsulación define qué ofrece cada objeto, la herencia y las interfaces crean familias de tipos, y el polimorfismo permite trabajar con toda la familia a la vez.

---

## 2. Repaso: clases y objetos

Una **clase** es un molde que describe **qué datos** tiene un tipo de objeto y **qué sabe hacer**. Un **objeto** (o **instancia**) es un ejemplar concreto creado a partir de esa clase.

```java
class Coche {
    String matricula;       // atributo (campo): dato de cada objeto
    int kilometros;

    Coche(String matricula) {          // constructor: inicializa el objeto
        this.matricula = matricula;
        this.kilometros = 0;
    }

    void conducir(int km) {            // método: comportamiento
        kilometros += km;
    }
}
```

```java
Coche c1 = new Coche("1234 ABC");    // new crea el objeto y llama al constructor
Coche c2 = new Coche("5678 XYZ");
c1.conducir(150);
IO.println(c1.kilometros);           // 150
IO.println(c2.kilometros);           // 0: cada objeto tiene sus propios datos
```

Conceptos clave:

| Concepto | Descripción |
|---|---|
| **Atributo** (campo) | Variable que pertenece a cada objeto |
| **Método** | Función que pertenece a la clase y trabaja con los datos del objeto |
| **Constructor** | Método especial con el mismo nombre que la clase y sin tipo de retorno; se ejecuta con `new` |
| **`this`** | Referencia al objeto actual. `this.matricula = matricula` distingue el atributo del parámetro |
| **`new`** | Crea un objeto en memoria y devuelve una referencia a él |
| **Constructor por defecto** | Si no escribes ningún constructor, Java crea uno vacío sin parámetros |

Un constructor puede llamar a otro de la misma clase con `this(...)`:

```java
class Coche {
    String matricula;
    int kilometros;

    Coche(String matricula, int kilometros) {
        this.matricula = matricula;
        this.kilometros = kilometros;
    }

    Coche(String matricula) {
        this(matricula, 0);       // reutiliza el otro constructor
    }
}
```

> 💡 La clase `Coche` de este apartado tiene un problema: cualquiera puede escribir `c1.kilometros = -500;`. Resolverlo es precisamente el objetivo de la **encapsulación**.

---

## 3. Qué es la encapsulación

**Encapsular** es **ocultar el estado interno** de un objeto y permitir modificarlo **solo a través de sus métodos**, que se encargan de mantenerlo correcto.

Una analogía: un cajero automático. No puedes abrirlo y coger billetes; solo puedes usar su teclado (la interfaz pública), que comprueba tu PIN y tu saldo antes de darte dinero.

### 3.1. El problema sin encapsulación

```java
class CuentaBancaria {
    String titular;
    double saldo;          // cualquiera puede leerlo y cambiarlo
}

var cuenta = new CuentaBancaria();
cuenta.saldo = 100;
cuenta.saldo = cuenta.saldo - 500;    // ❌ saldo negativo: nadie lo impide
cuenta.saldo = 1_000_000;             // ❌ dinero que aparece de la nada
```

Las reglas del negocio («no se puede sacar más de lo que hay») quedan repartidas por todo el programa, y basta con que alguien las olvide una vez para que los datos se corrompan.

### 3.2. La solución encapsulada

```java
// CuentaEncapsulada.java
void main() {
    var cuenta = new CuentaBancaria("Ana");
    cuenta.ingresar(100);
    cuenta.retirar(30);
    IO.println(cuenta.getTitular() + ": " + cuenta.getSaldo());

    try {
        cuenta.retirar(500);
    } catch (IllegalStateException e) {
        IO.println("Error: " + e.getMessage());
    }
    IO.println("Saldo final: " + cuenta.getSaldo());
}

class CuentaBancaria {
    private final String titular;      // private: solo accesible dentro de la clase
    private double saldo;

    CuentaBancaria(String titular) {
        this.titular = titular;
    }

    public void ingresar(double importe) {
        if (importe <= 0) {
            throw new IllegalArgumentException("El importe debe ser positivo");
        }
        saldo += importe;
    }

    public void retirar(double importe) {
        if (importe <= 0) {
            throw new IllegalArgumentException("El importe debe ser positivo");
        }
        if (importe > saldo) {
            throw new IllegalStateException("Saldo insuficiente");
        }
        saldo -= importe;
    }

    public String getTitular() { return titular; }
    public double getSaldo()   { return saldo; }
}
```

```text
Ana: 70.0
Error: Saldo insuficiente
Saldo final: 70.0
```

Ahora:

- El saldo **solo cambia** mediante `ingresar` y `retirar`, que **validan** cada operación.
- `cuenta.saldo = 1_000_000;` **no compila** desde fuera de la clase.
- Si mañana hay que registrar cada movimiento, se cambia **un solo sitio**.

> 💡 En aplicaciones reales, el dinero se representa con `BigDecimal` en lugar de `double`. Aquí se usa `double` para simplificar.

### 3.3. Ventajas

- **Integridad:** el objeto nunca queda en un estado incorrecto.
- **Mantenimiento:** se puede cambiar la implementación interna sin afectar a quien usa la clase.
- **Menos acoplamiento:** el resto del código depende de *qué* hace el objeto, no de *cómo* lo hace.
- **Facilidad para depurar:** si el saldo es incorrecto, solo hay dos métodos que pueden haberlo cambiado.

---

## 4. Modificadores de acceso

Los **modificadores de acceso** indican **desde dónde** se puede usar una clase, un atributo, un constructor o un método.

| Modificador | Misma clase | Mismo paquete | Subclase (otro paquete) | Cualquier sitio |
|---|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(sin modificador)* — acceso de paquete | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

De más restrictivo a menos: `private` → paquete → `protected` → `public`.

### 4.1. Paquetes

Un **paquete** agrupa clases relacionadas, como una carpeta. Se declara en la primera línea del fichero y debe coincidir con la estructura de carpetas:

```java
// Fichero: src/main/java/com/tienda/modelo/Producto.java
package com.tienda.modelo;

public class Producto { ... }
```

Para usar una clase de otro paquete se importa:

```java
package com.tienda.app;

import com.tienda.modelo.Producto;
```

### 4.2. Recomendaciones

- **Atributos:** casi siempre `private`.
- **Métodos:** `public` solo los que forman parte de lo que la clase ofrece; los auxiliares, `private`.
- **Clases:** `public` si se usan desde otros paquetes; si no, acceso de paquete.
- **`protected`:** úsalo con moderación. Expone detalles internos a todas las subclases (y, en Java, también a todo el paquete).
- Empieza con el acceso **más restrictivo** posible y ábrelo solo cuando haga falta: es fácil hacer público algo privado, pero muy difícil hacer privado algo que ya usa medio programa.

### 4.3. Acceso a nivel de clase

Una clase de primer nivel solo puede ser `public` o de acceso de paquete. En un fichero puede haber **como máximo una clase `public`**, y el fichero debe llamarse como ella.

> 💡 Por encima de los paquetes están los **módulos** (`module-info.java`, Java 9+), que deciden qué paquetes se exportan al resto de la aplicación. Una clase `public` de un paquete no exportado no es accesible desde otros módulos.

---

## 5. Getters y setters

Los **getters** (métodos de acceso, `getX()`) devuelven el valor de un atributo y los **setters** (métodos de modificación, `setX(valor)`) lo cambian.

```java
class Producto {
    private String nombre;
    private double precio;
    private boolean disponible;

    public String getNombre() { return nombre; }

    public double getPrecio() { return precio; }

    public void setPrecio(double precio) {
        if (precio < 0) {
            throw new IllegalArgumentException("El precio no puede ser negativo");
        }
        this.precio = precio;
    }

    public boolean isDisponible() { return disponible; }    // booleanos: isX()
}
```

Convenciones:

| Tipo | Getter | Setter |
|---|---|---|
| Cualquier tipo | `getNombre()` | `setNombre(String nombre)` |
| `boolean` | `isDisponible()` | `setDisponible(boolean disponible)` |

### 5.1. Un setter no es encapsulación automática

Generar un getter y un setter para **cada** atributo sin pensar equivale casi a tenerlos públicos:

```java
// ❌ Encapsulación solo aparente
cuenta.setSaldo(cuenta.getSaldo() - 500);   // ¿quién comprueba el saldo?

// ✅ Encapsulación real: el objeto aplica sus reglas
cuenta.retirar(500);
```

Buenas prácticas:

- **No crees setters por defecto.** Créalos solo si tiene sentido cambiar ese dato desde fuera, y **valida** el valor.
- Prefiere **métodos con significado de negocio** (`retirar`, `aplicarDescuento`, `marcarComoEnviado`) en lugar de setters genéricos.
- **No devuelvas colecciones internas modificables** desde un getter (ver sección 7.2).

Este enfoque se desarrolla en la guía [Tell, Don't Ask](<../07-Diseno/Principios/Tell dont ask.md>).

---

## 6. Invariantes: objetos siempre válidos

Una **invariante** es una regla que **siempre** debe cumplirse para un objeto: «el saldo nunca es negativo», «el email contiene @», «la fecha de fin no es anterior a la de inicio».

La encapsulación permite garantizarlas en dos puntos:

1. **En el constructor**: un objeto no debe poder crearse en un estado no válido.
2. **En cada método que modifica el estado**.

```java
class Rango {
    private final int minimo;
    private final int maximo;

    Rango(int minimo, int maximo) {
        if (minimo > maximo) {
            throw new IllegalArgumentException(
                "El mínimo (%d) no puede ser mayor que el máximo (%d)".formatted(minimo, maximo));
        }
        this.minimo = minimo;
        this.maximo = maximo;
    }

    boolean contiene(int valor) {
        return valor >= minimo && valor <= maximo;
    }
}
```

Para validaciones sencillas, la biblioteca estándar ofrece `java.util.Objects`:

```java
this.nombre = Objects.requireNonNull(nombre, "El nombre es obligatorio");
```

Lanza `NullPointerException` con ese mensaje si `nombre` es `null`.

> 💡 **Fallar pronto:** es mejor lanzar una excepción en el momento en que llega un dato incorrecto que descubrir el problema mucho después, en otra parte del programa. Más información en [Excepciones checked y unchecked](<../06-Excepciones/Excepciones checked y unchecked.md>).

---

## 7. Inmutabilidad y records

Un objeto **inmutable** no puede cambiar después de crearse. Es la forma más fuerte de encapsulación: si nada cambia, nada puede romperse.

Ventajas: es más fácil de entender, se puede compartir sin miedo entre partes del programa o entre hilos, y sirve como clave de un `Map`. Ejemplos de la biblioteca estándar: `String`, `Integer`, `LocalDate`, `BigDecimal`.

### 7.1. Cómo hacer una clase inmutable

```java
public final class Punto {              // 1. final: nadie puede heredar y añadir estado mutable
    private final int x;                // 2. atributos private final
    private final int y;

    public Punto(int x, int y) {        // 3. todo se asigna en el constructor
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }     // 4. solo getters, sin setters
    public int getY() { return y; }

    public Punto desplazar(int dx, int dy) {
        return new Punto(x + dx, y + dy);   // 5. "modificar" = devolver un objeto nuevo
    }
}
```

### 7.2. Copias defensivas

`final` impide reasignar el atributo, pero **no** impide modificar el objeto al que apunta. Si un atributo es una colección o un array, hay que protegerlo:

```java
class Pedido {
    private final List<String> lineas;

    Pedido(List<String> lineas) {
        this.lineas = List.copyOf(lineas);   // copia inmutable: los cambios externos no le afectan
    }

    List<String> getLineas() {
        return lineas;                       // es inmutable: devolverla es seguro
    }
}
```

Sin `List.copyOf`, quien creó el pedido podría seguir modificando la lista original desde fuera, y quien llamase a `getLineas()` podría hacer `getLineas().clear()`.

### 7.3. Records: datos encapsulados en una línea

Un **record** (Java 16+) es una clase pensada para **transportar datos inmutables**. El compilador genera automáticamente los atributos `private final`, el constructor, los métodos de acceso, `equals`, `hashCode` y `toString`:

```java
record Punto(int x, int y) {
    Punto desplazar(int dx, int dy) {
        return new Punto(x + dx, y + dy);
    }
}

var p = new Punto(2, 3);
IO.println(p.x());          // 2: el método de acceso se llama como el componente (no getX)
IO.println(p);              // Punto[x=2, y=3]
```

Las validaciones van en el **constructor compacto**:

```java
record Rango(int minimo, int maximo) {
    Rango {
        if (minimo > maximo) {
            throw new IllegalArgumentException("mínimo > máximo");
        }
    }
}
```

Los records **no pueden heredar de otra clase** (ya heredan de `java.lang.Record`) y son implícitamente `final`, pero **sí pueden implementar interfaces**, lo que los hace muy útiles para el polimorfismo (sección 22). Más detalles en la guía de [Records](<Records.md>).

---

## 8. Qué es la herencia

La **herencia** permite crear una clase nueva (**subclase**, clase hija o derivada) a partir de otra existente (**superclase**, clase padre o base). La subclase **recibe** los atributos y métodos de la superclase, puede **añadir** los suyos y puede **cambiar** (sobrescribir) parte del comportamiento heredado.

```java
class Animal {
    protected String nombre;

    Animal(String nombre) {
        this.nombre = nombre;
    }

    void comer() {
        IO.println(nombre + " está comiendo");
    }
}

class Perro extends Animal {          // extends: Perro hereda de Animal
    Perro(String nombre) {
        super(nombre);                // llama al constructor de Animal
    }

    void ladrar() {                   // método nuevo, solo de Perro
        IO.println(nombre + " dice: ¡Guau!");
    }
}
```

```java
var toby = new Perro("Toby");
toby.comer();     // heredado de Animal → Toby está comiendo
toby.ladrar();    // propio de Perro    → Toby dice: ¡Guau!
```

```text
          ┌──────────────┐
          │    Animal    │   superclase
          ├──────────────┤
          │ nombre       │
          │ comer()      │
          └──────▲───────┘
                 │ extends
       ┌─────────┴─────────┐
┌──────┴───────┐   ┌───────┴──────┐
│    Perro     │   │     Gato     │   subclases
├──────────────┤   ├──────────────┤
│ ladrar()     │   │ maullar()    │
└──────────────┘   └──────────────┘
```

### 8.1. La relación «es un»

La herencia modela una relación **«es un»** (*is-a*): un perro **es un** animal; un coche eléctrico **es un** coche. Si la frase no tiene sentido, la herencia no es la herramienta adecuada:

- ✅ `Gerente extends Empleado`: un gerente es un empleado.
- ❌ `Coche extends Motor`: un coche **no es** un motor; **tiene** un motor (eso es composición, sección 17).

### 8.2. Reglas básicas

- **Herencia simple:** una clase solo puede heredar (`extends`) de **una** clase. Para combinar varios comportamientos se usan interfaces (sección 15).
- **Herencia en cadena:** `Cachorro extends Perro extends Animal` es válido. Cachorro hereda de ambos.
- **Todas las clases heredan de `Object`.** Si no escribes `extends`, Java añade `extends Object`. Por eso cualquier objeto tiene `toString()`, `equals()` o `hashCode()`.

---

## 9. Qué se hereda y qué no

| Elemento de la superclase | ¿Lo tiene la subclase? | ¿Puede usarlo directamente? |
|---|---|---|
| Atributos y métodos `public` | Sí | Sí |
| Atributos y métodos `protected` | Sí | Sí |
| Atributos y métodos de paquete | Sí | Solo si está en el mismo paquete |
| Atributos y métodos `private` | Sí (forman parte del objeto) | **No**; solo a través de métodos no privados |
| Constructores | **No se heredan** | Se llaman con `super(...)` |
| Miembros `static` | Accesibles a través de la subclase | Pertenecen a la clase, no se sobrescriben |

Un detalle importante: los atributos `private` de la superclase **existen** dentro del objeto de la subclase, pero la subclase no puede acceder a ellos por nombre. Debe usar los métodos públicos o protegidos que ofrezca la superclase:

```java
class Animal {
    private String nombre;                      // private: ni las subclases lo ven

    Animal(String nombre) { this.nombre = nombre; }

    protected String getNombre() { return nombre; }
}

class Perro extends Animal {
    Perro(String nombre) { super(nombre); }

    void ladrar() {
        // IO.println(nombre);                  // ❌ error: nombre has private access in Animal
        IO.println(getNombre() + ": ¡Guau!");   // ✅
    }
}
```

> 💡 Mantener los atributos `private` incluso frente a las subclases es la opción más segura: la superclase sigue controlando sus invariantes. Usa `protected` solo para métodos que las subclases realmente necesiten.

---

## 10. Constructores y super

### 10.1. super(...) para llamar al constructor de la superclase

Los constructores no se heredan. Cada subclase define los suyos y debe **inicializar la parte heredada** llamando a un constructor de la superclase con `super(...)`:

```java
class Empleado {
    private final String nombre;
    private final double salarioBase;

    Empleado(String nombre, double salarioBase) {
        this.nombre = nombre;
        this.salarioBase = salarioBase;
    }
}

class Gerente extends Empleado {
    private final double bonus;

    Gerente(String nombre, double salarioBase, double bonus) {
        super(nombre, salarioBase);    // inicializa la parte de Empleado
        this.bonus = bonus;            // inicializa la parte propia
    }
}
```

Si no escribes `super(...)`, Java añade automáticamente `super()` **sin argumentos**. Si la superclase no tiene un constructor sin parámetros, el código no compila:

```java
class Gerente extends Empleado {
    Gerente(String nombre) {
        // ❌ error: constructor Empleado in class Empleado cannot be applied to given types
    }
}
```

### 10.2. Orden de construcción

Al crear un objeto, los constructores se ejecutan **de la superclase hacia abajo**: primero se construye la parte de `Object`, luego la del padre y por último la del hijo.

```java
// OrdenConstruccion.java
void main() {
    new C();
}

class A {
    A() { IO.println("Constructor de A"); }
}

class B extends A {
    B() { IO.println("Constructor de B"); }      // super() implícito
}

class C extends B {
    C() { IO.println("Constructor de C"); }
}
```

```text
Constructor de A
Constructor de B
Constructor de C
```

### 10.3. Constructores flexibles (Java 25)

Hasta Java 24, `super(...)` tenía que ser **la primera sentencia** del constructor. Desde Java 25 (JEP 513) se puede escribir código **antes** de `super(...)`, siempre que no use el objeto que se está construyendo (no se puede llamar a sus métodos ni leer sus atributos). Es muy útil para **validar o preparar argumentos** antes de construir la superclase:

```java
class Gerente extends Empleado {
    private final double bonus;

    Gerente(String nombre, double salarioBase, double bonus) {
        if (bonus < 0) {                                       // ✅ Java 25: antes de super
            throw new IllegalArgumentException("Bonus negativo");
        }
        super(nombre, salarioBase);
        this.bonus = bonus;
    }
}
```

Antes había que recurrir a métodos `static` auxiliares para validar dentro de la propia llamada a `super(...)`. Ahora el código se lee en orden natural.

También se pueden **asignar los atributos propios** antes de `super(...)`. Esto evita un problema clásico que se explica en la sección 11.5.

### 10.4. super para acceder a la superclase

Además de llamar al constructor, `super.` permite usar un método de la superclase desde la subclase, incluso si la subclase lo ha sobrescrito:

```java
@Override
double calcularSalario() {
    return super.calcularSalario() + bonus;    // reutiliza el cálculo del padre y lo amplía
}
```

---

## 11. Sobrescritura de métodos

**Sobrescribir** (*override*) es que una subclase proporcione **su propia versión** de un método heredado, con la **misma firma** (mismo nombre y mismos tipos de parámetros).

```java
class Animal {
    String sonido() {
        return "...";
    }
}

class Perro extends Animal {
    @Override
    String sonido() {
        return "Guau";
    }
}

class Gato extends Animal {
    @Override
    String sonido() {
        return "Miau";
    }
}
```

### 11.1. Usa siempre @Override

`@Override` le dice al compilador «este método sobrescribe uno de la superclase». Si te equivocas en el nombre o en los parámetros, **el compilador da error** en lugar de crear en silencio un método nuevo:

```java
class Perro extends Animal {
    @Override
    String Sonido() { return "Guau"; }    // ❌ error: method does not override or implement
                                          //    a method from a supertype (S mayúscula)
}
```

Sin `@Override`, ese código compilaría y `sonido()` seguiría devolviendo `"..."`, un error muy difícil de encontrar.

### 11.2. Reglas de la sobrescritura

| Regla | Detalle |
|---|---|
| **Misma firma** | Mismo nombre y mismos tipos de parámetros, en el mismo orden |
| **Tipo de retorno** | El mismo o un **subtipo** (retorno covariante). Si el padre devuelve `Animal`, el hijo puede devolver `Perro` |
| **Acceso** | Igual o **más abierto**, nunca más restrictivo. Un método `public` no puede pasar a ser `protected` |
| **Excepciones *checked*** | No puede lanzar excepciones *checked* nuevas o más generales que el método original |
| **Qué no se puede sobrescribir** | Métodos `private` (no se ven), `static` (se ocultan, no se sobrescriben) y `final` |

### 11.3. Sobrescritura frente a sobrecarga

Se confunden a menudo porque los nombres se parecen:

| | Sobrescritura (*override*) | Sobrecarga (*overload*) |
|---|---|---|
| Dónde | En una **subclase** | En la **misma clase** (o en una subclase) |
| Firma | **Igual** | **Distintos parámetros** |
| Cuándo se decide qué método se ejecuta | En **ejecución**, según el objeto real | En **compilación**, según los tipos de los argumentos |
| Para qué | Cambiar el comportamiento heredado | Ofrecer variantes del mismo método |
| Anotación | `@Override` | Ninguna |

```java
class Calculadora {
    int sumar(int a, int b) { return a + b; }               // sobrecarga
    double sumar(double a, double b) { return a + b; }      // sobrecarga
}

class CalculadoraConLog extends Calculadora {
    @Override
    int sumar(int a, int b) {                               // sobrescritura
        IO.println("Sumando " + a + " y " + b);
        return super.sumar(a, b);
    }
}
```

### 11.4. Los atributos y los métodos static no son polimórficos

Solo los **métodos de instancia** se sobrescriben. Si una subclase declara un atributo o un método `static` con el mismo nombre que el padre, lo **oculta** (*hiding*), y cuál se usa depende del **tipo declarado** de la variable, no del objeto real. Es confuso y conviene evitarlo:

```java
// Ocultacion.java
void main() {
    Padre p = new Hijo();
    IO.println(p.nombre);          // atributo: depende del tipo declarado
    IO.println(p.saludar());       // método de instancia: depende del objeto real
}

class Padre {
    String nombre = "Padre";
    String saludar() { return "Hola desde Padre"; }
}

class Hijo extends Padre {
    String nombre = "Hijo";        // ⚠️ oculta el atributo del padre
    @Override
    String saludar() { return "Hola desde Hijo"; }
}
```

```text
Padre
Hola desde Hijo
```

Con atributos `private` y acceso mediante métodos, este problema desaparece.

### 11.5. No llames a métodos sobrescribibles desde un constructor

El constructor del padre se ejecuta **antes** de que se inicialicen los atributos del hijo. Si llama a un método que el hijo sobrescribe, ese método verá los atributos del hijo **todavía sin valor**:

```java
// ConstructorPeligroso.java
void main() {
    new Hijo();
}

class Padre {
    Padre() {
        presentarse();             // ⚠️ llama a la versión de Hijo
    }
    void presentarse() { IO.println("Soy el padre"); }
}

class Hijo extends Padre {
    private String nombre = "Lucas";

    @Override
    void presentarse() { IO.println("Soy " + nombre); }
}
```

```text
Soy null
```

Cuando `Padre()` llama a `presentarse()`, el inicializador `nombre = "Lucas"` del hijo todavía no se ha ejecutado, así que `nombre` vale `null`.

> 💡 Curiosidad: si el atributo fuese `private final String nombre = "Lucas";`, sí imprimiría `Soy Lucas`, porque el compilador sustituye las constantes `final` de tipo `String` o primitivo por su valor. Con cualquier otro valor (por ejemplo, uno que se calcula), el problema aparece igual.

Soluciones:

- **No llames** a métodos sobrescribibles desde un constructor; si lo necesitas, hazlos `private` o `final`.
- En Java 25, el hijo puede asignar sus atributos **antes** de `super()` gracias a los constructores flexibles:

```java
class Hijo extends Padre {
    private final String nombre;

    Hijo() {
        nombre = "Lucas";          // ✅ se asigna antes de construir Padre
        super();
    }

    @Override
    void presentarse() { IO.println("Soy " + nombre); }   // ahora imprime "Soy Lucas"
}
```

---

## 12. La clase Object: toString, equals y hashCode

Todas las clases heredan de `java.lang.Object`. Sus métodos más importantes suelen **sobrescribirse**:

| Método | Qué hace por defecto | Por qué sobrescribirlo |
|---|---|---|
| `toString()` | Devuelve algo como `Producto@1b6d3586` | Mostrar el objeto de forma legible al imprimirlo o depurar |
| `equals(Object o)` | Compara **referencias** (igual que `==`) | Que dos objetos con los mismos datos se consideren iguales |
| `hashCode()` | Un número basado en la identidad del objeto | Debe ser coherente con `equals` para funcionar en `HashMap` y `HashSet` |
| `getClass()` | Devuelve la clase real del objeto | No se puede sobrescribir (es `final`) |

### 12.1. toString

```java
@Override
public String toString() {
    return "Producto[nombre=%s, precio=%.2f]".formatted(nombre, precio);
}
```

`IO.println(producto)` y la concatenación `"Producto: " + producto` llaman a `toString()` automáticamente.

### 12.2. equals y hashCode

```java
// IgualdadProducto.java
void main() {
    var a = new Producto("TEC-01", "Teclado");
    var b = new Producto("TEC-01", "Teclado mecánico");

    IO.println(a == b);           // ¿mismo objeto?
    IO.println(a.equals(b));      // ¿mismo producto según nuestras reglas?

    Set<Producto> catalogo = new HashSet<>();
    catalogo.add(a);
    catalogo.add(b);
    IO.println(catalogo.size());  // b se considera repetido
}

class Producto {
    private final String codigo;
    private final String nombre;

    Producto(String codigo, String nombre) {
        this.codigo = codigo;
        this.nombre = nombre;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                       // mismo objeto
        if (!(o instanceof Producto otro)) return false;  // null u otro tipo
        return codigo.equals(otro.codigo);                // criterio de igualdad: el código
    }

    @Override
    public int hashCode() {
        return Objects.hash(codigo);                      // los mismos campos que equals
    }
}
```

```text
false
true
1
```

**El contrato que hay que cumplir:**

- Si `a.equals(b)` es `true`, entonces `a.hashCode() == b.hashCode()` **debe** ser `true`.
- Si sobrescribes `equals`, **sobrescribe también `hashCode`** usando los mismos campos. Si no, `HashSet` y `HashMap` se comportarán de forma incorrecta (por ejemplo, admitirán «duplicados»).
- `equals` debe ser reflexivo (`a.equals(a)`), simétrico, transitivo y devolver `false` con `null`.

> 💡 Los **records** generan `equals`, `hashCode` y `toString` correctos con todos sus componentes. Los IDE también pueden generarlos automáticamente.

---

## 13. final: impedir la herencia y la sobrescritura

`final` tiene tres usos relacionados con la herencia y la encapsulación:

| Dónde | Efecto | Ejemplo |
|---|---|---|
| **Clase** | No se puede heredar de ella | `final class Punto`, `String`, `Integer` |
| **Método** | Las subclases no pueden sobrescribirlo | `final double calcularIva()` |
| **Atributo o variable** | Solo se puede asignar una vez | `private final String id;` |

```java
final class Moneda { }
class Euro extends Moneda { }      // ❌ error: cannot inherit from final Moneda

class Cuenta {
    final void registrarMovimiento() { }
}
class CuentaJoven extends Cuenta {
    @Override
    void registrarMovimiento() { }   // ❌ error: overridden method is final
}
```

**¿Cuándo usar `final` en una clase?** Diseñar una clase para que otros hereden de ella exige pensar qué métodos pueden sobrescribirse sin romper nada. Si no lo has pensado, márcala como `final` (o usa un `record`). Siempre podrás quitarlo más adelante.

---

## 14. Clases abstractas

Una **clase abstracta** (`abstract class`) representa un concepto general que **no tiene sentido instanciar directamente**: existen perros y gatos, pero no «animales genéricos».

- **No se puede crear un objeto** de una clase abstracta con `new`.
- Puede tener **métodos abstractos**: declarados sin cuerpo, que las subclases **deben** implementar.
- Puede tener también atributos, constructores y métodos normales.

```java
abstract class Figura {
    private final String nombre;

    protected Figura(String nombre) {
        this.nombre = nombre;
    }

    abstract double area();             // sin cuerpo: cada figura calcula la suya

    String describir() {                // método normal que usa el abstracto
        return "%s con área %.2f".formatted(nombre, area());
    }
}

class Circulo extends Figura {
    private final double radio;

    Circulo(double radio) {
        super("Círculo");
        this.radio = radio;
    }

    @Override
    double area() {
        return Math.PI * radio * radio;
    }
}

class Rectangulo extends Figura {
    private final double base, altura;

    Rectangulo(double base, double altura) {
        super("Rectángulo");
        this.base = base;
        this.altura = altura;
    }

    @Override
    double area() {
        return base * altura;
    }
}
```

```java
Figura f = new Figura("algo");          // ❌ error: Figura is abstract; cannot be instantiated
Figura c = new Circulo(1);              // ✅
IO.println(c.describir());              // Círculo con área 3,14
```

Si una subclase no implementa todos los métodos abstractos, también tiene que declararse `abstract`.

El método `describir()` es un ejemplo del patrón **Template Method**: la superclase define los pasos generales y las subclases rellenan los detalles. Se explica en [Patrones de comportamiento](<../07-Diseno/Patrones/Patrones comportamiento.md>).

---

## 15. Interfaces

Una **interfaz** define un **contrato**: un conjunto de métodos que una clase se compromete a ofrecer. Responde a «qué sabe hacer», no a «qué es».

```java
interface Pagable {
    double importeAPagar();                      // abstracto (public abstract implícito)

    default String resumen() {                   // método con implementación (Java 8+)
        return "Importe: %.2f €".formatted(importeAPagar());
    }

    static Pagable gratis() {                    // método estático de la interfaz
        return () -> 0;
    }
}

class Factura implements Pagable {               // implements: cumple el contrato
    private final double base;

    Factura(double base) { this.base = base; }

    @Override
    public double importeAPagar() {              // debe ser public
        return base * 1.21;
    }
}
```

Características:

- Una clase puede **implementar varias interfaces**: `class Pato extends Animal implements Volador, Nadador`.
- Los métodos abstractos de una interfaz son implícitamente `public abstract`; al implementarlos, la clase debe declararlos `public`.
- Los atributos de una interfaz son implícitamente `public static final` (constantes).
- Puede tener métodos `default` (con implementación), `static` y `private` (Java 9+, auxiliares internos).
- Una interfaz puede **heredar** de otras con `extends`: `interface Anfibio extends Volador, Nadador`.
- Una interfaz con **un solo método abstracto** es una **interfaz funcional** y se puede implementar con una *lambda*, como en `gratis()` (ver [Lambdas y Streams](<../02-Programacion-funcional/Lambdas y Streams.md>)).

### 15.1. ¿Clase abstracta o interfaz?

| | Clase abstracta | Interfaz |
|---|---|---|
| Relación | «es un» (`Perro` es un `Animal`) | «sabe hacer» (`Factura` es `Pagable`) |
| Herencia | Una sola (`extends`) | Varias (`implements`) |
| Estado (atributos de instancia) | Sí | No (solo constantes) |
| Constructores | Sí | No |
| Cuándo | Comparten código y estado comunes | Definir capacidades que pueden tener clases sin relación entre sí |

Regla práctica: **empieza por una interfaz**; usa una clase abstracta cuando varias implementaciones necesiten compartir estado o código. La comparación completa está en [Abstract Class vs Interface](<Abstract Class vs Interface.md>).

---

## 16. Clases e interfaces selladas

Con una clase normal, **cualquiera** puede heredar de ella. Con una clase o interfaz **sellada** (`sealed`, Java 17+) eres tú quien decide **exactamente qué subclases están permitidas**.

```java
sealed interface MetodoPago permits Tarjeta, Transferencia, Bizum { }

record Tarjeta(String numero) implements MetodoPago { }
record Transferencia(String iban) implements MetodoPago { }
final class Bizum implements MetodoPago { }
```

Cada subclase permitida debe declarar cómo continúa la jerarquía:

| Modificador de la subclase | Significado |
|---|---|
| `final` | Nadie puede heredar de ella (los records ya lo son) |
| `sealed` | Sigue sellada, con su propia lista `permits` |
| `non-sealed` | Vuelve a quedar abierta a cualquier subclase |

Ventajas:

- **Modelar dominios cerrados**: un pago es una tarjeta, una transferencia o un Bizum, y nada más.
- El compilador **conoce todos los casos posibles**, lo que permite `switch` exhaustivos sin `default` (sección 22).
- Proteges invariantes: nadie puede crear una subclase que rompa tus reglas.

Si las subclases están en el mismo fichero que la clase sellada, se puede omitir `permits`.

---

## 17. Composición frente a herencia

La **composición** consiste en que un objeto **contenga** otros objetos y les **delegue** trabajo. Modela la relación **«tiene un»** (*has-a*).

```java
class Motor {
    void arrancar() { IO.println("Motor en marcha"); }
}

class Coche {
    private final Motor motor;          // Coche TIENE un Motor

    Coche(Motor motor) { this.motor = motor; }

    void arrancar() {
        motor.arrancar();               // delega en el motor
        IO.println("Coche listo");
    }
}
```

### 17.1. El problema de abusar de la herencia

Un ejemplo clásico: queremos una pila (*stack*) y heredamos de `ArrayList` para «aprovechar» su código:

```java
// ❌ Herencia mal usada
class Pila<T> extends ArrayList<T> {
    void apilar(T e) { add(e); }
    T desapilar()   { return remove(size() - 1); }
}

var pila = new Pila<String>();
pila.apilar("a");
pila.apilar("b");
pila.add(0, "c");     // 😱 se puede insertar por el fondo: ya no es una pila
pila.clear();         // y cualquier otro método de ArrayList
```

Al heredar, la pila expone **todos** los métodos públicos de `ArrayList` y su encapsulación se rompe. Con composición, la pila decide qué ofrece:

```java
// ✅ Composición
class Pila<T> {
    private final List<T> elementos = new ArrayList<>();

    void apilar(T e) { elementos.add(e); }

    T desapilar() {
        if (elementos.isEmpty()) {
            throw new IllegalStateException("Pila vacía");
        }
        return elementos.removeLast();         // Java 21+
    }

    boolean estaVacia() { return elementos.isEmpty(); }
}
```

### 17.2. ¿Cuándo heredar y cuándo componer?

**Usa herencia cuando:**

- La relación «es un» es real y estable.
- La subclase puede usarse **en cualquier sitio** donde se use la superclase sin sorpresas (**principio de sustitución de Liskov**, explicado en [Principios SOLID](<../07-Diseno/Principios/Principio SOLID.md>)).
- La superclase está diseñada para ser extendida.

**Prefiere composición cuando:**

- La relación es «tiene un» o «usa un».
- Solo quieres reutilizar **parte** del comportamiento.
- Necesitas cambiar el comportamiento en tiempo de ejecución (por ejemplo, sustituir el motor).
- La jerarquía empieza a crecer en muchas combinaciones (`CocheElectricoDescapotableAutomatico`...).

> 💡 La recomendación clásica del libro *Design Patterns* es **«favorece la composición sobre la herencia»**. No significa no usar nunca la herencia, sino no usarla solo para reutilizar código.

---

## 18. Qué es el polimorfismo

**Polimorfismo** significa «muchas formas»: **una misma operación se comporta de forma distinta según el objeto que la recibe**. Permite escribir código que trabaja con un tipo general (`Animal`, `Figura`, `Pagable`) sin saber qué clase concreta hay detrás.

```java
// Polimorfismo.java
void main() {
    Animal[] animales = {new Perro(), new Gato(), new Vaca()};
    for (Animal a : animales) {
        IO.println(a.getClass().getSimpleName() + " → " + a.sonido());
    }
}

abstract class Animal {
    abstract String sonido();
}

class Perro extends Animal {
    @Override String sonido() { return "Guau"; }
}

class Gato extends Animal {
    @Override String sonido() { return "Miau"; }
}

class Vaca extends Animal {
    @Override String sonido() { return "Muu"; }
}
```

```text
Perro → Guau
Gato → Miau
Vaca → Muu
```

El bucle **no sabe ni le importa** qué animal concreto hay en cada posición: llama a `sonido()` y cada objeto responde a su manera. Si mañana se añade `class Oveja extends Animal`, el bucle funciona sin cambiar ni una línea.

### 18.1. Tipos de polimorfismo en Java

| Tipo | Mecanismo | Cuándo se decide |
|---|---|---|
| **Polimorfismo de subtipos** (dinámico) | Sobrescritura de métodos con herencia o interfaces | En **ejecución** (*enlace dinámico*) |
| **Polimorfismo ad hoc** (estático) | Sobrecarga de métodos | En **compilación** |
| **Polimorfismo paramétrico** | Genéricos: `List<T>` funciona con cualquier `T` | En **compilación** |

Cuando se habla de «polimorfismo» sin más, casi siempre se refiere al **de subtipos**, que es el que se desarrolla en el resto de la guía. Los genéricos tienen su propia guía: [Generics](<Generics.md>).

---

## 19. Tipo declarado y tipo real

Una variable de tipo referencia tiene **dos tipos**:

- **Tipo declarado** (o estático): el que aparece en la declaración de la variable. Lo usa el **compilador**.
- **Tipo real** (o dinámico): la clase del objeto que se creó con `new`. Lo usa la **JVM** al ejecutar.

```java
Animal a = new Perro();
//  │            │
//  │            └── tipo real: Perro
//  └── tipo declarado: Animal
```

Dos reglas que lo explican todo:

1. **El compilador decide QUÉ métodos puedes llamar**, mirando el **tipo declarado**.
2. **La JVM decide QUÉ VERSIÓN se ejecuta**, mirando el **tipo real** (enlace dinámico).

```java
Animal a = new Perro();
a.sonido();       // ✅ Animal tiene sonido() → se ejecuta la versión de Perro: "Guau"
a.ladrar();       // ❌ error de compilación: Animal no tiene ladrar()
```

Aunque el objeto **sí es** un perro, el compilador solo ve un `Animal` y no permite llamar a métodos que `Animal` no declara.

### 19.1. Programar contra la abstracción

Por eso es habitual declarar las variables con el **tipo más general que sea suficiente**:

```java
List<String> nombres = new ArrayList<>();     // ✅ declarado como List
ArrayList<String> otros = new ArrayList<>();  // funciona, pero te ata a ArrayList
```

Si más adelante necesitas una `LinkedList`, solo cambias la parte derecha. Este principio se conoce como **«programa contra interfaces, no contra implementaciones»**.

---

## 20. Conversiones: upcasting y downcasting

### 20.1. Upcasting: de subclase a superclase

Convertir hacia **arriba** en la jerarquía es **automático y siempre seguro**, porque un perro siempre es un animal:

```java
Perro p = new Perro();
Animal a = p;               // upcasting implícito
Object o = p;               // también: todo es un Object
```

### 20.2. Downcasting: de superclase a subclase

Convertir hacia **abajo** requiere un *cast* explícito y **puede fallar** en ejecución si el objeto no es realmente de ese tipo:

```java
Animal a = new Perro();
Perro p = (Perro) a;        // ✅ funciona: el objeto real es un Perro

Animal b = new Gato();
Perro q = (Perro) b;        // 💥 ClassCastException: class Gato cannot be cast to class Perro
```

### 20.3. instanceof con patrones (Java 16+)

Para hacer un *downcasting* seguro, comprueba antes el tipo. Con un **patrón** se comprueba y se declara la variable en un solo paso:

```java
// ❌ Forma antigua: comprobar y luego convertir
if (a instanceof Perro) {
    Perro p = (Perro) a;
    p.ladrar();
}

// ✅ Desde Java 16
if (a instanceof Perro p) {
    p.ladrar();
}

// También combinable con condiciones
if (a instanceof Perro p && p.esCachorro()) {
    IO.println("Es un cachorro");
}
```

> ⚠️ **Señal de alerta:** si tu código está lleno de `if (x instanceof A) ... else if (x instanceof B) ...` para decidir qué hacer, probablemente ese comportamiento debería ser un **método sobrescrito** en cada clase. La excepción son las jerarquías **selladas** con `switch`, que se ven en la sección 22.

---

## 21. Polimorfismo con colecciones y parámetros

El polimorfismo brilla cuando un método o una colección trabajan con el **tipo general**:

```java
// AreasPolimorficas.java
void main() {
    List<Figura> figuras = List.of(
        new Circulo(1),
        new Rectangulo(2, 3),
        new Cuadrado(2)
    );

    for (Figura f : figuras) {
        IO.println(f.describir());
    }
    IO.println("Área total: %.2f".formatted(areaTotal(figuras)));
}

// Acepta cualquier figura, presente o futura
double areaTotal(List<? extends Figura> figuras) {
    double total = 0;
    for (Figura f : figuras) {
        total += f.area();      // cada figura calcula su área a su manera
    }
    return total;
}

abstract class Figura {
    abstract double area();

    String describir() {
        return "%s: %.2f".formatted(getClass().getSimpleName(), area());
    }
}

class Circulo extends Figura {
    private final double radio;
    Circulo(double radio) { this.radio = radio; }
    @Override double area() { return Math.PI * radio * radio; }
}

class Rectangulo extends Figura {
    private final double base, altura;
    Rectangulo(double base, double altura) { this.base = base; this.altura = altura; }
    @Override double area() { return base * altura; }
}

class Cuadrado extends Rectangulo {
    Cuadrado(double lado) { super(lado, lado); }     // reutiliza area() de Rectangulo
}
```

```text
Circulo: 3,14
Rectangulo: 6,00
Cuadrado: 4,00
Área total: 13,14
```

Observa que `Cuadrado` no sobrescribe `area()`: hereda la de `Rectangulo`, y el enlace dinámico encuentra esa versión.

> 💡 La salida usa coma decimal porque `formatted` respeta la configuración regional del sistema (aquí, española).

### 21.1. Polimorfismo con interfaces

Las interfaces permiten tratar de forma uniforme clases **sin ninguna relación de herencia**:

```java
interface Imprimible {
    String imprimir();
}

record Factura(int numero, double total) implements Imprimible {
    public String imprimir() { return "Factura nº " + numero + ": " + total + " €"; }
}

record Albaran(String cliente) implements Imprimible {
    public String imprimir() { return "Albarán para " + cliente; }
}

void imprimirTodo(List<Imprimible> documentos) {
    documentos.forEach(d -> IO.println(d.imprimir()));
}
```

`Factura` y `Albaran` no comparten superclase (salvo `Record`), pero ambos **son `Imprimible`**, y `imprimirTodo` funciona con los dos.

---

## 22. Polimorfismo con tipos sellados y switch

Java ofrece dos formas de que cada tipo tenga su propio comportamiento:

1. **Orientada a objetos:** cada clase sobrescribe un método (secciones 18 a 21).
2. **Orientada a datos (Java 21+):** una jerarquía **sellada** de *records* y un `switch` con **patrones** que trata cada caso.

```java
// PagosSellados.java
void main() {
    List<MetodoPago> pagos = List.of(
        new Tarjeta("1234567812345678", 100),
        new Transferencia("ES9121000418450200051332", 250),
        new Bizum("600123456", 20)
    );

    for (MetodoPago p : pagos) {
        IO.println(describir(p) + " | comisión: " + comision(p) + " €");
    }
}

sealed interface MetodoPago permits Tarjeta, Transferencia, Bizum { }

record Tarjeta(String numero, double importe) implements MetodoPago { }
record Transferencia(String iban, double importe) implements MetodoPago { }
record Bizum(String telefono, double importe) implements MetodoPago { }

String describir(MetodoPago pago) {
    return switch (pago) {
        case Tarjeta t -> "Tarjeta acabada en " + t.numero().substring(12);
        case Transferencia t -> "Transferencia desde " + t.iban().substring(0, 4);
        case Bizum b -> "Bizum al " + b.telefono();
    };                                   // exhaustivo: sin default
}

double comision(MetodoPago pago) {
    return switch (pago) {
        case Tarjeta(_, double importe) when importe > 50 -> 0.5;     // patrón de record
        case Tarjeta _ -> 0.25;
        case Transferencia _ -> 0;
        case Bizum _ -> 0;
    };
}
```

```text
Tarjeta acabada en 5678 | comisión: 0.5 €
Transferencia desde ES91 | comisión: 0.0 €
Bizum al 600123456 | comisión: 0.0 €
```

Novedades que aparecen en el ejemplo:

- **Exhaustividad:** como `MetodoPago` está sellada, el compilador sabe que solo hay tres casos y **no hace falta `default`**. Si se añade un cuarto método de pago, **todos los `switch` que no lo traten dejarán de compilar**, lo que evita olvidos.
- **Patrones de record** (Java 21): `case Tarjeta(_, double importe)` **descompone** el record y extrae sus componentes.
- **Patrones sin nombre** (Java 22): `_` indica que un componente o la variable no se usan.
- **Guardas** con `when` para añadir condiciones a un caso.

### 22.1. ¿Métodos sobrescritos o switch con patrones?

| | Métodos sobrescritos | Tipos sellados + `switch` |
|---|---|---|
| Añadir un **tipo** nuevo | Fácil: una clase nueva, sin tocar el resto | Hay que actualizar cada `switch` (el compilador te avisa) |
| Añadir una **operación** nueva | Hay que añadir un método a cada clase | Fácil: un método nuevo con un `switch` |
| Ideal para | Jerarquías abiertas, comportamiento propio de cada objeto | Dominios cerrados, datos que se procesan de varias formas |

Ambas formas son polimorfismo y se pueden combinar. Con jerarquías **abiertas** (no selladas), prefiere los métodos sobrescritos: un `switch` sobre un tipo abierto necesita `default` y no detecta los casos nuevos.

El enfoque orientado a datos se desarrolla en [Programación funcional](<../02-Programacion-funcional/Programacion funcional.md>), sección 15.

---

## 23. Ejemplo completo: nóminas de una empresa

Este programa reúne los tres pilares:

- **Encapsulación:** atributos `private final`, validación en constructores y ninguna forma de modificar los datos desde fuera.
- **Herencia:** `Empleado` abstracto con tres subclases; `super(...)`, constructores flexibles de Java 25 y `super.metodo()`.
- **Polimorfismo:** una lista de `Empleado` donde cada uno calcula su salario a su manera, más una interfaz para un comportamiento opcional.

```java
// Nominas.java
void main() {
    List<Empleado> plantilla = List.of(
        new EmpleadoFijo("Ana", 2_000),
        new Comercial("Bruno", 1_500, 30_000, 0.05),
        new Gerente("Carla", 3_000, 800),
        new Becario("David", 120)
    );

    IO.println("=== Nómina mensual ===");
    double total = 0;
    for (Empleado e : plantilla) {
        double salario = e.calcularSalario();          // enlace dinámico
        IO.println("%-6s %-13s %9.2f €".formatted(e.getNombre(), e.getTipo(), salario));
        total += salario;
    }
    IO.println("%-20s %9.2f €".formatted("TOTAL", total));

    IO.println();
    IO.println("=== Empleados con incentivos ===");
    for (Empleado e : plantilla) {
        if (e instanceof ConIncentivo i) {             // patrón con instanceof
            IO.println(e.getNombre() + ": " + i.describirIncentivo());
        }
    }

    try {
        new EmpleadoFijo("Eva", -100);
    } catch (IllegalArgumentException ex) {
        IO.println();
        IO.println("No se pudo dar de alta: " + ex.getMessage());
    }
}

// ── Interfaz: capacidad opcional ─────────────────────────────
interface ConIncentivo {
    String describirIncentivo();
}

// ── Superclase abstracta ─────────────────────────────────────
abstract class Empleado {
    private final String nombre;

    protected Empleado(String nombre) {
        this.nombre = Objects.requireNonNull(nombre, "El nombre es obligatorio");
    }

    public String getNombre() { return nombre; }

    public String getTipo() { return getClass().getSimpleName(); }

    public abstract double calcularSalario();

    @Override
    public String toString() {
        return "%s[%s]".formatted(getTipo(), nombre);
    }
}

// ── Subclases ────────────────────────────────────────────────
class EmpleadoFijo extends Empleado {
    private final double salarioBase;

    EmpleadoFijo(String nombre, double salarioBase) {
        if (salarioBase <= 0) {                        // Java 25: validación antes de super
            throw new IllegalArgumentException("Salario base no válido: " + salarioBase);
        }
        super(nombre);
        this.salarioBase = salarioBase;
    }

    @Override
    public double calcularSalario() {
        return salarioBase;
    }
}

class Comercial extends EmpleadoFijo implements ConIncentivo {
    private final double ventas;
    private final double porcentaje;

    Comercial(String nombre, double salarioBase, double ventas, double porcentaje) {
        super(nombre, salarioBase);
        this.ventas = ventas;
        this.porcentaje = porcentaje;
    }

    @Override
    public double calcularSalario() {
        return super.calcularSalario() + ventas * porcentaje;    // amplía el cálculo del padre
    }

    @Override
    public String describirIncentivo() {
        return "comisión del %.0f%% sobre %.2f € en ventas".formatted(porcentaje * 100, ventas);
    }
}

class Gerente extends EmpleadoFijo implements ConIncentivo {
    private final double bonus;

    Gerente(String nombre, double salarioBase, double bonus) {
        super(nombre, salarioBase);
        this.bonus = bonus;
    }

    @Override
    public double calcularSalario() {
        return super.calcularSalario() + bonus;
    }

    @Override
    public String describirIncentivo() {
        return "bonus fijo de %.2f €".formatted(bonus);
    }
}

final class Becario extends Empleado {
    private static final double PRECIO_HORA = 8.5;
    private final int horas;

    Becario(String nombre, int horas) {
        super(nombre);
        this.horas = horas;
    }

    @Override
    public double calcularSalario() {
        return horas * PRECIO_HORA;
    }
}
```

```text
=== Nómina mensual ===
Ana    EmpleadoFijo    2000,00 €
Bruno  Comercial       3000,00 €
Carla  Gerente         3800,00 €
David  Becario         1020,00 €
TOTAL                  9820,00 €

=== Empleados con incentivos ===
Bruno: comisión del 5% sobre 30000,00 € en ventas
Carla: bonus fijo de 800,00 €

No se pudo dar de alta: Salario base no válido: -100.0
```

**Para pensar:**

- Añade una clase `Autonomo` que cobre por proyecto. ¿Qué partes del programa tienes que cambiar? (Solo crear la clase: el bucle de nóminas no cambia).
- ¿Por qué `Becario` es `final` y `EmpleadoFijo` no?
- ¿Qué pasaría si `calcularSalario()` fuera `public` en `Empleado` y `protected` en `Gerente`? (No compila: no se puede reducir la visibilidad al sobrescribir).

---

## 24. Errores frecuentes

| Error | Síntoma | Solución |
|---|---|---|
| Atributos `public` | Cualquier parte del programa deja el objeto en un estado incorrecto | Atributos `private` y métodos que validen |
| Getter y setter para todo | Encapsulación aparente; la lógica acaba fuera de la clase | Métodos con significado de negocio; setters solo si son necesarios |
| Devolver una colección interna | Desde fuera se modifica el estado del objeto | `List.copyOf` o `Collections.unmodifiableList` |
| Olvidar `@Override` | Un error tipográfico crea un método nuevo en vez de sobrescribir | Pon siempre `@Override` |
| Superclase sin constructor sin parámetros | `constructor X in class X cannot be applied to given types` | Llama a `super(...)` con los argumentos adecuados |
| Reducir la visibilidad al sobrescribir | `attempting to assign weaker access privileges` | Mantén el mismo acceso o uno más abierto |
| `equals` sin `hashCode` | `HashSet` o `HashMap` admiten «duplicados» | Sobrescribe ambos con los mismos campos (o usa un `record`) |
| Llamar a un método sobrescribible desde el constructor | La subclase ve atributos a `null` o a `0` | No lo hagas, o asigna antes de `super()` (Java 25) |
| *Downcasting* sin comprobar | `ClassCastException` | `instanceof` con patrón antes de convertir |
| Cadenas de `instanceof` | Código frágil que hay que tocar con cada tipo nuevo | Método sobrescrito, o jerarquía sellada con `switch` |
| Heredar para reutilizar código | La subclase expone métodos que no debería (`Pila extends ArrayList`) | Composición |
| Jerarquías muy profundas | Difícil saber qué versión de un método se ejecuta | Pocos niveles; preferir interfaces y composición |
| Intentar instanciar una clase abstracta | `X is abstract; cannot be instantiated` | Crea una subclase concreta |
| Ocultar atributos con el mismo nombre | El valor depende del tipo de la variable, no del objeto | Atributos `private` y acceso por métodos |

---

## 25. Chuleta rápida

```java
// ── Encapsulación ────────────────────────────────────────
class Cuenta {
    private double saldo;                         // estado oculto
    public void ingresar(double i) {              // operación con validación
        if (i <= 0) throw new IllegalArgumentException();
        saldo += i;
    }
    public double getSaldo() { return saldo; }    // solo lectura
}
// private < (paquete) < protected < public

record Punto(int x, int y) { }                    // inmutable: equals, hashCode, toString

// ── Herencia ─────────────────────────────────────────────
class Animal {
    protected Animal(String nombre) { ... }
    String sonido() { return "..."; }
}
class Perro extends Animal {
    Perro(String nombre) {
        // validaciones (Java 25)
        super(nombre);                            // constructor del padre
    }
    @Override String sonido() { return "Guau"; }  // sobrescritura
    String doble() { return super.sonido() + sonido(); }
}
final class NoHeredable { }
abstract class Figura { abstract double area(); }
interface Pagable { double importe(); default String info() { ... } }
sealed interface Forma permits Circulo, Cuadrado { }

// ── Polimorfismo ─────────────────────────────────────────
Animal a = new Perro("Toby");                     // upcasting
a.sonido();                                       // "Guau": enlace dinámico
if (a instanceof Perro p) { ... }                 // downcasting seguro

String r = switch (forma) {                       // sellada: sin default
    case Circulo c -> "círculo";
    case Cuadrado(double lado) -> "cuadrado de " + lado;
};
```

| Quiero... | Uso |
|---|---|
| Ocultar un dato | `private` |
| Que no cambie nunca | `final` / `record` |
| Reutilizar una relación «es un» | `extends` |
| Definir una capacidad | `interface` + `implements` |
| Un concepto que no se instancia | `abstract class` |
| Limitar quién hereda | `sealed ... permits` |
| Impedir la herencia | `final class` |
| Reutilizar sin «es un» | Composición (un atributo del otro tipo) |

---

## 26. Ejercicios propuestos

1. **Termómetro encapsulado.** Crea una clase `Termometro` con la temperatura en grados Celsius como atributo privado. Debe impedir temperaturas por debajo de −273,15 °C y ofrecer `getCelsius()`, `getFahrenheit()` y `subir(double grados)`.
2. **Biblioteca.** Crea una clase `Libro` con título y un indicador de prestado. En lugar de `setPrestado(boolean)`, ofrece `prestar()` y `devolver()`, que lancen `IllegalStateException` si la operación no tiene sentido (prestar un libro ya prestado o devolver uno que no lo está).
3. **Vehículos.** Crea una clase abstracta `Vehiculo` con matrícula y un método abstracto `int numeroRuedas()`. Implementa `Coche`, `Moto` y `Camion` (este último recibe el número de ejes y tiene 4 ruedas por eje). Recorre una lista de vehículos y muestra la matrícula y las ruedas de cada uno.
4. **Predice la salida.** Sin ejecutarlo, escribe qué imprime este código y comprueba después tu respuesta:
   ```java
   class A {
       String nombre() { return "A"; }
       String saludo() { return "Soy " + nombre(); }
   }
   class B extends A {
       @Override String nombre() { return "B"; }
   }
   void main() {
       A x = new B();
       IO.println(x.saludo());
       IO.println(x instanceof A);
       IO.println(x.getClass().getSimpleName());
   }
   ```
5. **equals y hashCode.** Crea una clase `Dni` con un número y una letra. Dos DNI son iguales si tienen el mismo número y la misma letra. Comprueba que un `HashSet<Dni>` no admite duplicados. Después, reescríbela como `record` y compara.
6. **Composición.** Crea una clase `ListaDeLaCompra` que use internamente una `List<String>` y solo permita añadir productos no vacíos, tacharlos y consultar cuántos quedan. No debe ser posible borrar todos los productos con `clear()` desde fuera.
7. **Notificaciones con interfaz.** Define una interfaz `Notificador` con `void enviar(String mensaje)`. Impleméntala con `NotificadorEmail` y `NotificadorSms`, y escribe un método `avisarATodos(List<Notificador> canales, String mensaje)`.
8. **Formas selladas.** Define `sealed interface Forma` con los records `Circulo(double radio)`, `Rectangulo(double base, double altura)` y `Triangulo(double base, double altura)`. Escribe `double area(Forma f)` y `double perimetro(Forma f)` con `switch` exhaustivos (para el triángulo, supón que es rectángulo). Añade después `Cuadrado(double lado)` y observa qué te dice el compilador.

---

## 27. Soluciones

Hay muchas soluciones correctas; estas son una propuesta.

<details>
<summary>1. Termómetro encapsulado</summary>

```java
class Termometro {
    private static final double CERO_ABSOLUTO = -273.15;
    private double celsius;

    Termometro(double celsius) {
        validar(celsius);
        this.celsius = celsius;
    }

    public double getCelsius()    { return celsius; }
    public double getFahrenheit() { return celsius * 9 / 5 + 32; }

    public void subir(double grados) {
        validar(celsius + grados);
        celsius += grados;
    }

    private static void validar(double temperatura) {
        if (temperatura < CERO_ABSOLUTO) {
            throw new IllegalArgumentException("Por debajo del cero absoluto: " + temperatura);
        }
    }
}
```

Fíjate en que `subir` también sirve para bajar (con grados negativos) y en que la validación está en un único método privado.

</details>

<details>
<summary>2. Biblioteca</summary>

```java
class Libro {
    private final String titulo;
    private boolean prestado;

    Libro(String titulo) {
        this.titulo = Objects.requireNonNull(titulo);
    }

    public void prestar() {
        if (prestado) {
            throw new IllegalStateException(titulo + " ya está prestado");
        }
        prestado = true;
    }

    public void devolver() {
        if (!prestado) {
            throw new IllegalStateException(titulo + " no estaba prestado");
        }
        prestado = false;
    }

    public boolean isPrestado() { return prestado; }
    public String getTitulo()   { return titulo; }
}
```

</details>

<details>
<summary>3. Vehículos</summary>

```java
abstract class Vehiculo {
    private final String matricula;

    protected Vehiculo(String matricula) { this.matricula = matricula; }

    public String getMatricula() { return matricula; }

    public abstract int numeroRuedas();
}

class Coche extends Vehiculo {
    Coche(String matricula) { super(matricula); }
    @Override public int numeroRuedas() { return 4; }
}

class Moto extends Vehiculo {
    Moto(String matricula) { super(matricula); }
    @Override public int numeroRuedas() { return 2; }
}

class Camion extends Vehiculo {
    private final int ejes;

    Camion(String matricula, int ejes) {
        if (ejes < 2) throw new IllegalArgumentException("Un camión tiene al menos 2 ejes");
        super(matricula);
        this.ejes = ejes;
    }

    @Override public int numeroRuedas() { return ejes * 4; }
}

void main() {
    List<Vehiculo> flota = List.of(new Coche("1111 AAA"), new Moto("2222 BBB"), new Camion("3333 CCC", 3));
    for (Vehiculo v : flota) {
        IO.println(v.getMatricula() + ": " + v.numeroRuedas() + " ruedas");
    }
}
```

```text
1111 AAA: 4 ruedas
2222 BBB: 2 ruedas
3333 CCC: 12 ruedas
```

</details>

<details>
<summary>4. Predice la salida</summary>

```text
Soy B
true
B
```

- `saludo()` está definido solo en `A`, pero dentro llama a `nombre()`, y esa llamada **también usa enlace dinámico**: el objeto real es un `B`, así que se ejecuta `B.nombre()`.
- Un `B` **es un** `A`, así que `instanceof A` es `true`.
- `getClass()` devuelve siempre el **tipo real**, no el declarado.

</details>

<details>
<summary>5. equals y hashCode</summary>

```java
class Dni {
    private final int numero;
    private final char letra;

    Dni(int numero, char letra) {
        this.numero = numero;
        this.letra = Character.toUpperCase(letra);
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof Dni otro && numero == otro.numero && letra == otro.letra;
    }

    @Override
    public int hashCode() {
        return Objects.hash(numero, letra);
    }
}

// Versión record: equals y hashCode generados automáticamente
record DniRecord(int numero, char letra) {
    DniRecord {
        letra = Character.toUpperCase(letra);   // normalización en el constructor compacto
    }
}

void main() {
    Set<Dni> dnis = new HashSet<>();
    dnis.add(new Dni(12345678, 'z'));
    dnis.add(new Dni(12345678, 'Z'));
    IO.println(dnis.size());                   // 1

    Set<DniRecord> otros = new HashSet<>(List.of(new DniRecord(1, 'a'), new DniRecord(1, 'A')));
    IO.println(otros.size());                  // 1
}
```

</details>

<details>
<summary>6. Composición</summary>

```java
class ListaDeLaCompra {
    private final List<String> pendientes = new ArrayList<>();

    public void anadir(String producto) {
        if (producto == null || producto.isBlank()) {
            throw new IllegalArgumentException("Producto vacío");
        }
        pendientes.add(producto.strip());
    }

    public boolean tachar(String producto) {
        return pendientes.remove(producto);
    }

    public int quedan() {
        return pendientes.size();
    }

    public List<String> getPendientes() {
        return List.copyOf(pendientes);        // copia: desde fuera no se puede modificar
    }
}
```

Si `ListaDeLaCompra` heredase de `ArrayList<String>`, cualquiera podría llamar a `clear()`, `add("")` o `set(0, null)`.

</details>

<details>
<summary>7. Notificaciones con interfaz</summary>

```java
interface Notificador {
    void enviar(String mensaje);
}

class NotificadorEmail implements Notificador {
    private final String direccion;
    NotificadorEmail(String direccion) { this.direccion = direccion; }

    @Override
    public void enviar(String mensaje) {
        IO.println("Email a " + direccion + ": " + mensaje);
    }
}

class NotificadorSms implements Notificador {
    private final String telefono;
    NotificadorSms(String telefono) { this.telefono = telefono; }

    @Override
    public void enviar(String mensaje) {
        IO.println("SMS a " + telefono + ": " + mensaje);
    }
}

void avisarATodos(List<Notificador> canales, String mensaje) {
    for (Notificador n : canales) {
        n.enviar(mensaje);
    }
}

void main() {
    avisarATodos(
        List.of(new NotificadorEmail("ana@ejemplo.com"), new NotificadorSms("600111222"),
                m -> IO.println("Consola: " + m)),     // interfaz funcional: también vale una lambda
        "Tu pedido ha salido");
}
```

</details>

<details>
<summary>8. Formas selladas</summary>

```java
sealed interface Forma permits Circulo, Rectangulo, Triangulo { }

record Circulo(double radio) implements Forma { }
record Rectangulo(double base, double altura) implements Forma { }
record Triangulo(double base, double altura) implements Forma { }

double area(Forma f) {
    return switch (f) {
        case Circulo(double r) -> Math.PI * r * r;
        case Rectangulo(double b, double h) -> b * h;
        case Triangulo(double b, double h) -> b * h / 2;
    };
}

double perimetro(Forma f) {
    return switch (f) {
        case Circulo(double r) -> 2 * Math.PI * r;
        case Rectangulo(double b, double h) -> 2 * (b + h);
        case Triangulo(double b, double h) -> b + h + Math.hypot(b, h);   // triángulo rectángulo
    };
}
```

Al añadir `record Cuadrado(double lado) implements Forma` (y a `permits`), ambos métodos dejan de compilar con el error `the switch expression does not cover all possible input values`. El compilador te señala exactamente dónde falta tratar el caso nuevo.

</details>

---

## 28. Glosario

| Término | Definición |
|---|---|
| **Clase abstracta** | Clase que no se puede instanciar y puede tener métodos sin implementar |
| **Composición** | Relación «tiene un»: un objeto contiene otros y les delega trabajo |
| **Downcasting** | Conversión de una referencia del tipo padre al tipo hijo; requiere *cast* y puede fallar |
| **Encapsulación** | Ocultar el estado interno y permitir su uso solo mediante métodos controlados |
| **Enlace dinámico** | Decidir en ejecución, según el objeto real, qué versión de un método se ejecuta |
| **Herencia** | Mecanismo por el que una clase recibe atributos y métodos de otra (`extends`) |
| **Inmutable** | Objeto cuyo estado no puede cambiar después de crearse |
| **Interfaz** | Contrato con los métodos que una clase se compromete a ofrecer (`implements`) |
| **Invariante** | Regla que el estado de un objeto debe cumplir siempre |
| **Modificador de acceso** | `private`, de paquete, `protected` o `public`: desde dónde se puede usar un miembro |
| **Ocultación** (*hiding*) | Un atributo o método `static` de la subclase tapa al del padre; no es polimórfico |
| **Polimorfismo** | Capacidad de tratar objetos de distintas clases mediante un tipo común, respondiendo cada uno a su manera |
| **Clase sellada** | Clase o interfaz que limita qué subclases pueden existir (`sealed ... permits`) |
| **Sobrecarga** | Varios métodos con el mismo nombre y distintos parámetros |
| **Sobrescritura** | Nueva versión de un método heredado, con la misma firma, en una subclase |
| **Subclase / superclase** | Clase hija que hereda / clase padre de la que se hereda |
| **`super`** | Referencia a la parte de la superclase: `super(...)` llama a su constructor y `super.metodo()` a su versión del método |
| **Tipo declarado** | Tipo con el que se declara la variable; lo usa el compilador |
| **Tipo real** | Clase del objeto creado con `new`; lo usa la JVM al ejecutar |
| **Upcasting** | Conversión implícita de una referencia del tipo hijo al tipo padre |

---

## 29. Referencias

- Oracle, *The Java Tutorials*: [Classes and Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/index.html), [Controlling Access to Members of a Class](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html), [Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html), [Polymorphism](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html) y [Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html) (escritos para JDK 8; los conceptos siguen vigentes).
- [Dev.java – Learn Java](https://dev.java/learn/): tutoriales oficiales actualizados.
- [Especificación del lenguaje Java (JLS), Java SE 25](https://docs.oracle.com/javase/specs/jls/se25/html/index.html): capítulo 8 (clases) y capítulo 9 (interfaces).
- [JEP 513: Flexible Constructor Bodies](https://openjdk.org/jeps/513) (Java 25).
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409) (Java 17).
- [JEP 395: Records](https://openjdk.org/jeps/395) (Java 16) y [JEP 440: Record Patterns](https://openjdk.org/jeps/440) (Java 21).
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) (Java 21) y [JEP 456: Unnamed Variables & Patterns](https://openjdk.org/jeps/456) (Java 22).
- Joshua Bloch, *Effective Java* (3.ª ed.): «Minimize the accessibility of classes and members», «Favor composition over inheritance», «Design and document for inheritance or else prohibit it» y «Obey the general contract when overriding equals».
