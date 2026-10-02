# Java: Abstract Class vs Interface — Chuleta

## 📑 Índice

1. [Idea principal](#-idea-principal)
2. [Abstract Class](#-abstract-class)
3. [Interface](#-interface)
4. [Comparación rápida](#️-comparación-rápida)
5. [extends vs implements](#-extends-vs-implements)
6. [Herencia](#-herencia)
7. [Estado y variables](#-estado-y-variables)
8. [Constructores](#️-constructores)
9. [Modificadores de acceso](#-modificadores-de-acceso)
10. [Métodos final](#-métodos-final)
11. [Polimorfismo](#-polimorfismo)
12. [¿Cuándo usar cada uno?](#-cuándo-usar-cada-uno)
13. [Acoplamiento](#-acoplamiento)
14. [Combinar ambas](#-combinar-ambas)
15. [Errores frecuentes](#️-errores-frecuentes)
16. [Reglas mentales](#-reglas-mentales)
17. [Chuleta ultra rápida](#-chuleta-ultra-rápida)
18. [Resumen final](#-resumen-final)

---

## 🧠 Idea principal

Tanto las `abstract class` como las `interface` sirven para conseguir **abstracción** y definir una estructura que otras clases deben seguir.

La diferencia fundamental:

| | Abstract Class | Interface |
| --- | --- | --- |
| **Qué define** | Base común | Contrato / comportamiento |
| **Qué comparte** | Estado + código | Lo que una clase debe hacer |
| **Pregunta clave** | ¿Qué **es**? | ¿Qué **puede hacer**? |
| **Relación** | "ES UN" | "IMPLEMENTA / PUEDE HACER" |

```text
ABSTRACT CLASS                          INTERFACE
      ↓                                     ↓
 Base común                         Contrato / comportamiento
      ↓                                     ↓
Comparte estado + código        Define lo que una clase debe hacer
      ↓                                     ↓
   "ES UN"                       "IMPLEMENTA / PUEDE HACER"
```

---

## 🟣 Abstract Class

Una `abstract class` es una clase que **no puede instanciarse directamente**. Se utiliza como **clase base** para otras clases.

Puede contener:

- Métodos abstractos
- Métodos normales (con implementación)
- Atributos de instancia
- Constructores
- Métodos `private`, `protected` y `public`
- Métodos `static`
- Métodos `final`

**Ejemplo:**

```java
abstract class Shape {

    // Método abstracto
    abstract double area();

    // Método concreto
    void display() {
        System.out.println("This is a shape");
    }
}
```

Una clase hija utiliza `extends`:

```java
class Circle extends Shape {

    int radius = 5;

    @Override
    double area() {
        return 3.14 * radius * radius;
    }
}
```

Podemos utilizar una referencia del tipo abstracto:

```java
Shape shape = new Circle();

shape.display();
System.out.println(shape.area());
```

Esto permite usar **polimorfismo**:

```text
Shape
  ↑
Circle
```

---

## 🔵 Interface

Una `interface` define un **contrato** que las clases que la implementan deben cumplir.

**Ejemplo:**

```java
interface Drawable {

    void draw();
}
```

Una clase implementa la interfaz mediante `implements`:

```java
class Rectangle implements Drawable {

    @Override
    public void draw() {
        System.out.println("Drawing Rectangle");
    }
}
```

Podemos utilizar una referencia de tipo interfaz:

```java
Drawable d = new Rectangle();

d.draw();
```

Esto también permite **polimorfismo**:

```text
Drawable
   ↑
Rectangle
```

### Características

- Define un contrato.
- Una clase puede implementar **varias** interfaces.
- No se puede hacer `new` de una interfaz.
- Puede declarar métodos abstractos.
- Puede tener métodos `default` y `static` (desde Java 8).
- Puede tener métodos `private` (desde Java 9).
- Puede declarar constantes.
- No está pensada para compartir estado de instancia.

### 📌 Métodos en una Interface

En el modelo tradicional, los métodos declarados en una interfaz son abstractos. Pero las interfaces modernas también pueden tener:

- `abstract`
- `default`
- `static`
- `private`

```java
interface Example {

    // Abstracto
    void method1();

    // Implementación por defecto
    default void method2() {
        System.out.println("Default");
    }

    // Método estático
    static void method3() {
        System.out.println("Static");
    }
}
```

> Los métodos `private` en interfaces son posibles desde **Java 9**.

### Una clase puede implementar varias interfaces

```java
interface Printable {
    void print();
}

interface Scannable {
    void scan();
}

class Printer implements Printable, Scannable {

    @Override
    public void print() {
        System.out.println("Printing...");
    }

    @Override
    public void scan() {
        System.out.println("Scanning...");
    }
}
```

---

## ⚔️ Comparación rápida

| Característica | Abstract Class | Interface |
| --- | :---: | :---: |
| Define un contrato | ✅ | ✅ |
| Instanciable directamente | ❌ | ❌ |
| Métodos abstractos | ✅ | ✅ |
| Métodos implementados | ✅ | ✅ `default`, `static`, `private` |
| Variables de instancia | ✅ | ❌ |
| Constantes | ✅ | ✅ |
| Constructor | ✅ | ❌ |
| Estado de instancia | ✅ | ❌ |
| Métodos `private` | ✅ | ✅ desde Java 9 |
| Métodos `protected` | ✅ | ❌ |
| Métodos `public` | ✅ | ✅ |
| Métodos `final` | ✅ | ❌ |
| Métodos `static` | ✅ | ✅ |
| Compartir implementación | ✅ | Limitado |
| Herencia múltiple | ❌ | ✅ mediante interfaces |
| Palabra clave | `extends` | `implements` |
| Propósito principal | Base común | Contrato / comportamiento |
| Concepto | "es un" | "puede hacer" |
| Acoplamiento | Mayor | Menor |

---

## 🔑 extends vs implements

### Abstract Class

Una clase **extiende** otra clase:

```java
class Circle extends Shape {

}
```

```text
Circle
  ↓ extends
Shape
```

### Interface

Una clase **implementa** una interfaz:

```java
class Rectangle implements Drawable {

}
```

```text
Rectangle
  ↓ implements
Drawable
```

Una interfaz puede **extender** otra interfaz:

```java
interface AdvancedDrawable extends Drawable {

}
```

### Resumen

```text
class     → extends    → class
class     → implements → interface
interface → extends    → interface
```

---

## 🧬 Herencia

### Abstract Class: una sola clase padre

Java **no permite**:

```java
class MyClass extends ClassA, ClassB { // ❌

}
```

Una clase solo puede extender **una** clase:

```java
class MyClass extends ClassA {

}
```

### Interface: múltiples interfaces

Una clase puede implementar varias interfaces:

```java
class MyClass implements InterfaceA, InterfaceB, InterfaceC {

}
```

Esto permite combinar diferentes comportamientos:

```text
         ┌→ Flyable
Bird ────┼→ Swimmable
         └→ Runnable
```

---

## 📦 Estado y variables

### Abstract Class

Puede mantener **estado**:

```java
abstract class Employee {

    protected String name;
    protected double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
}
```

Cada objeto puede tener sus propios valores:

```text
Employee
 ├── name
 └── salary
```

### Interface

Las variables declaradas en una interfaz son, por defecto:

- `public`
- `static`
- `final`

Por tanto, funcionan como **constantes**:

```java
interface Constants {

    int MAX_USERS = 100;
}
```

Equivale conceptualmente a:

```java
public static final int MAX_USERS = 100;
```

No sirven para mantener el estado individual de cada objeto.

---

## 🏗️ Constructores

### Abstract Class

Puede tener constructores:

```java
abstract class Animal {

    protected String name;

    public Animal(String name) {
        this.name = name;
    }
}
```

La clase hija llama al constructor mediante `super()`:

```java
class Dog extends Animal {

    public Dog(String name) {
        super(name);
    }
}
```

### Interface

Una interfaz **no tiene constructores**:

```java
interface Animal {
    // ❌ No constructor
}
```

Una interfaz no representa directamente un objeto que haya que inicializar.

---

## 🔒 Modificadores de acceso

### Abstract Class

Sus métodos pueden tener diferentes niveles de acceso:

```java
abstract class Example {

    private void method1() {
    }

    protected void method2() {
    }

    public void method3() {
    }
}
```

### Interface

Los métodos abstractos de una interfaz son `public` por defecto:

```java
interface Example {

    void execute();
}
```

Es equivalente a:

```java
interface Example {

    public abstract void execute();
}
```

Las interfaces modernas también permiten métodos `private`.

---

## 🛑 Métodos final

### Abstract Class

Puede tener métodos `final`:

```java
abstract class Animal {

    final void breathe() {
        System.out.println("Breathing...");
    }
}
```

Una subclase **no puede sobrescribirlo**:

```java
class Dog extends Animal {

    // ❌ No se puede sobrescribir breathe()
}
```

### Interface

Los métodos de una interfaz **no pueden declararse `final`**:

```java
interface Example {

    // ❌ No válido
    final void method();
}
```

---

## 🔄 Polimorfismo

Ambos mecanismos permiten polimorfismo.

### Abstract Class

```java
abstract class Animal {

    abstract void sound();
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Woof");
    }
}
```

Uso:

```java
Animal animal = new Dog();

animal.sound();
```

### Interface

```java
interface Animal {

    void sound();
}

class Dog implements Animal {

    @Override
    public void sound() {
        System.out.println("Woof");
    }
}
```

Uso:

```java
Animal animal = new Dog();

animal.sound();
```

En ambos casos:

```text
Referencia → Animal
Objeto real → Dog
```

---

## 🎯 ¿Cuándo usar cada uno?

### Usa `abstract class` cuando...

Las clases están **estrechamente relacionadas** y quieres compartir:

- Código
- Estado
- Atributos
- Constructores
- Comportamiento

Pregunta mental: **¿qué es este objeto?**

```text
Dog   → Animal
Cat   → Animal

Car   → Vehicle
Truck → Vehicle
```

**Ejemplo:**

```text
          Vehicle
             ↑
     ┌───────┼───────┐
     ↓       ↓       ↓
    Car    Truck    Bus
```

Todos son vehículos y probablemente comparten características:

```java
abstract class Vehicle {

    protected String brand;

    public Vehicle(String brand) {
        this.brand = brand;
    }

    public void stop() {
        System.out.println("Stopping...");
    }

    abstract void start();
}
```

```java
class Car extends Vehicle {

    public Car(String brand) {
        super(brand);
    }

    @Override
    void start() {
        System.out.println("Car starting...");
    }
}
```

### Usa `interface` cuando...

Quieres definir un **comportamiento o contrato** que pueden compartir clases que no necesariamente están relacionadas por herencia.

Pregunta mental: **¿qué puede hacer este objeto?**

```text
Bird     ───→ Flyable
Airplane ───→ Flyable
Drone    ───→ Flyable
```

Los tres pueden volar, pero no pertenecen a la misma jerarquía de clases:

```java
interface Flyable {

    void fly();
}

class Bird implements Flyable {

    @Override
    public void fly() {
        System.out.println("Bird flying");
    }
}

class Airplane implements Flyable {

    @Override
    public void fly() {
        System.out.println("Airplane flying");
    }
}
```

Más ejemplos de capacidades:

```text
Bird      → Flyable
Car       → Drivable
Document  → Printable
Payment   → Payable
```

---

## 🔗 Acoplamiento

Una diferencia importante es el **acoplamiento**.

### Abstract Class

Normalmente produce una relación **más fuerte** entre la clase base y sus subclases:

```text
Employee
   ↑
Developer
```

`Developer` depende de la estructura de `Employee`.

### Interface

Favorece un acoplamiento **más débil**:

```text
Developer ───→ Payable
Customer  ───→ Payable
Invoice   ───→ Payable
```

Las clases solo necesitan cumplir el contrato de `Payable`. Esto facilita cambiar las implementaciones sin cambiar el código que depende de la interfaz.

---

## 🧩 Combinar ambas

No hay que elegir exclusivamente una u otra. Una clase puede **heredar de una clase abstracta** y, además, **implementar una o varias interfaces**.

### Ejemplo 1: Employee + Payable

```java
interface Payable {

    void pay();
}

abstract class Employee {

    protected String name;

    public Employee(String name) {
        this.name = name;
    }

    public void work() {
        System.out.println(name + " is working");
    }

    public abstract void calculateSalary();
}

class Developer extends Employee implements Payable {

    public Developer(String name) {
        super(name);
    }

    @Override
    public void calculateSalary() {
        System.out.println("Calculating salary...");
    }

    @Override
    public void pay() {
        System.out.println("Paying developer...");
    }
}
```

Conceptualmente:

```text
     Employee
        ↑
    Developer ─────→ Payable
```

`Developer`:

- Es un `Employee`.
- Hereda estado y comportamiento de `Employee`.
- Debe implementar `calculateSalary()`.
- Cumple el contrato `Payable`.
- Puede tener comportamiento específico propio.

### Ejemplo 2: Animal + Flyable + Swimmable

```text
          Animal
             ↑
            Bird
             │
             ├────→ Flyable
             │
             └────→ Swimmable
```

```java
abstract class Animal {

    protected String name;

    public Animal(String name) {
        this.name = name;
    }
}

interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Bird extends Animal implements Flyable, Swimmable {

    public Bird(String name) {
        super(name);
    }

    @Override
    public void fly() {
        System.out.println("Flying");
    }

    @Override
    public void swim() {
        System.out.println("Swimming");
    }
}
```

`Bird`:

- Es un `Animal` → `extends`
- Puede volar → `implements Flyable`
- Puede nadar → `implements Swimmable`

---

## ⚠️ Errores frecuentes

| Error | Por qué falla |
| --- | --- |
| Confundir `extends` con `implements` | Se hereda de una clase con `extends` y se cumple una interfaz con `implements`. |
| `class A extends B, C` | Java solo permite heredar de **una** clase. |
| `new` sobre una clase abstracta o una interfaz | Ninguna de las dos es instanciable directamente. |
| Declarar un método `final` en una interfaz | No es válido en interfaces. |
| Esperar estado de instancia en una interfaz | Sus variables son siempre constantes (`public static final`). |
| Buscar un constructor en una interfaz | Las interfaces no tienen constructores. |
| Pensar "interfaz = solo métodos abstractos" | En Java moderno también existen `default`, `static` y `private`. |

Ejemplo de la confusión más habitual:

```java
// Heredar de una clase
class Dog extends Animal {
}

// Implementar una interfaz
class Dog implements Runnable {
}
```

---

## 🧠 Reglas mentales

### Abstract Class

```text
¿Las clases están relacionadas?
        ↓ Sí
¿Quiero compartir estado/código?
        ↓ Sí
   ABSTRACT CLASS
```

Piensa: **"Es un..."**

```text
Dog       → Animal
Car       → Vehicle
Developer → Employee
```

### Interface

```text
¿Quiero definir un comportamiento?
        ↓ Sí
¿Pueden tenerlo clases diferentes?
        ↓ Sí
      INTERFACE
```

Piensa: **"Puede hacer..."**

```text
Bird     → Flyable
Printer  → Printable
Employee → Payable
Car      → Drivable
```

### Regla práctica

```text
¿Quiero definir una capacidad?
        ↓
    interface

¿Quiero compartir estado o código entre clases relacionadas?
        ↓
  abstract class
```

---

## ⚡ Chuleta ultra rápida

```text
┌─────────────────────────────────────────┐
│          ABSTRACT CLASS                 │
├─────────────────────────────────────────┤
│ Base común                              │
│ Comparte código                         │
│ Comparte estado                         │
│ Puede tener constructor                 │
│ Puede tener atributos                   │
│ Puede tener métodos abstractos          │
│ Puede tener métodos normales            │
│ Una clase padre como máximo             │
│ extends                                 │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│              INTERFACE                  │
├─────────────────────────────────────────┤
│ Contrato / comportamiento               │
│ No mantiene estado de instancia         │
│ No tiene constructor                    │
│ Variables = constantes                  │
│ Puede tener abstract/default/static     │
│ Puede tener private desde Java 9        │
│ Varias interfaces por clase             │
│ implements                              │
└─────────────────────────────────────────┘
```

---

## 🚀 Resumen final

### En 10 segundos

```text
ABSTRACT CLASS                         INTERFACE
  = clase base                           = contrato
  = herencia                             = capacidad
  = "es un"                              = "puede hacer"
  = comparte estado/código               = menor acoplamiento
  = solo una clase padre                 = permite múltiples interfaces
  = extends                              = implements
```

### 🎯 Regla definitiva

- **Abstract class** = identidad + estado + comportamiento compartido.
- **Interface** = contrato + capacidad + comportamiento común definido por contrato.

### Memoriza únicamente esto

```text
extends    → heredar una clase
implements → cumplir una interfaz

abstract class → "es un"
interface      → "puede hacer"
```

---

> **Nota:** la interfaz se presenta principalmente como un contrato cuyos métodos abstractos deben implementar las clases, pero en Java moderno también existen métodos `default`, `static` y `private`. Por eso la distinción no debe reducirse a "interfaz = solo métodos abstractos".

📖 **Referencia:** [Difference Between Abstract Class and Interface in Java — GeeksforGeeks](https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/)
