Java: Interface vs Abstract Class — Chuleta

🧠 Idea principal

Interface



Define un contrato: qué puede hacer una clase.



"Esta clase puede hacer X."



Abstract Class



Define una base común: qué es una clase y qué comportamiento comparte.



"Esta clase es un X."



🔵 Interface



Una interfaz se declara con interface:



public interface Flyable {

&nbsp;   void fly();

}





Una clase implementa una interfaz con implements:



public class Bird implements Flyable {



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Flying...");

&nbsp;   }

}



Características



Define un contrato.



Una clase puede implementar varias interfaces.



No se puede hacer new de una interfaz.



Puede declarar métodos abstractos.



Puede tener métodos default.



Puede tener métodos static.



Puede tener métodos private desde Java 9.



Puede declarar constantes.



No está pensada para compartir estado de instancia.



Una clase puede implementar varias interfaces

interface Printable {

&nbsp;   void print();

}



interface Scannable {

&nbsp;   void scan();

}



class Printer implements Printable, Scannable {



&nbsp;   @Override

&nbsp;   public void print() {

&nbsp;       System.out.println("Printing...");

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void scan() {

&nbsp;       System.out.println("Scanning...");

&nbsp;   }

}



class MyClass implements InterfaceA, InterfaceB, InterfaceC {

}



🟣 Abstract Class



Una clase abstracta define una clase base que puede combinar:



comportamiento común



estado común



métodos abstractos



métodos completamente implementados



Ejemplo:



public abstract class Vehicle {



&nbsp;   protected String brand;



&nbsp;   public Vehicle(String brand) {

&nbsp;       this.brand = brand;

&nbsp;   }



&nbsp;   public abstract void start();



&nbsp;   public void stop() {

&nbsp;       System.out.println("Stopping...");

&nbsp;   }

}





Una clase hereda de ella mediante extends:



public class Car extends Vehicle {



&nbsp;   public Car(String brand) {

&nbsp;       super(brand);

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void start() {

&nbsp;       System.out.println("Car starting...");

&nbsp;   }

}



Características



Puede tener atributos de instancia.



Puede tener constructores.



Puede tener métodos abstractos.



Puede tener métodos completamente implementados.



Puede tener métodos private, protected, etc.



No se puede instanciar directamente.



Una clase solo puede extender una única clase.



class Car extends Vehicle {

}





Esto no es posible:



class Car extends Vehicle, Machine { // ❌

}



⚔️ Comparación rápida

Característica	Interface	Abstract Class

Define un contrato	✅	✅

Métodos abstractos	✅	✅

Métodos con implementación	✅	✅

Atributos de instancia	❌	✅

Constructor	❌	✅

Se puede instanciar	❌	❌

Una clase puede usar varias	✅	❌

Palabra clave	implements	extends

Compartir estado	❌	✅

Compartir implementación	Limitado	✅

Concepto principal	"puede hacer"	"es un"

🔑 Regla para decidir

Usa interface cuando...



Quieres expresar una capacidad o contrato.



Ejemplos:



Bird      → Flyable

Car       → Drivable

Document  → Printable

Payment   → Payable





Pregunta mental:



¿Qué puede hacer este objeto?



Ejemplo:



interface Flyable {

&nbsp;   void fly();

}



class Bird implements Flyable {



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Flying");

&nbsp;   }

}



Usa abstract class cuando...



Existe una relación fuerte entre las clases y quieres compartir código o estado.



Ejemplos:



Dog   → Animal

Cat   → Animal



Car   → Vehicle

Truck → Vehicle





Pregunta mental:



¿Qué es este objeto?



Ejemplo:



abstract class Animal {



&nbsp;   protected String name;



&nbsp;   public Animal(String name) {

&nbsp;       this.name = name;

&nbsp;   }



&nbsp;   public abstract void makeSound();



&nbsp;   public void sleep() {

&nbsp;       System.out.println("Sleeping...");

&nbsp;   }

}



class Dog extends Animal {



&nbsp;   public Dog(String name) {

&nbsp;       super(name);

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void makeSound() {

&nbsp;       System.out.println("Woof!");

&nbsp;   }

}



🧩 Se pueden combinar



No tienes que elegir exclusivamente una u otra.



Una clase puede heredar de una clase abstracta y, además, implementar una o varias interfaces.



interface Flyable {

&nbsp;   void fly();

}



abstract class Animal {



&nbsp;   protected String name;



&nbsp;   public Animal(String name) {

&nbsp;       this.name = name;

&nbsp;   }



&nbsp;   public abstract void makeSound();

}



class Bird extends Animal implements Flyable {



&nbsp;   public Bird(String name) {

&nbsp;       super(name);

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void makeSound() {

&nbsp;       System.out.println("Tweet!");

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Flying!");

&nbsp;   }

}





Conceptualmente:



&nbsp;       Animal

&nbsp;         ↑

&nbsp;       Bird ─────→ Flyable





Bird:



es un Animal → extends



puede hacer Flyable → implements



🧠 Truco para memorizar

INTERFACE

&nbsp;   ↓

"¿QUÉ PUEDE HACER?"

&nbsp;   ↓

CAPACIDAD / CONTRATO



ABSTRACT CLASS

&nbsp;   ↓

"¿QUÉ ES?"

&nbsp;   ↓

BASE COMÚN / HERENCIA





Ejemplo:



&nbsp;       Animal

&nbsp;         ↑

&nbsp;      ┌──┴──┐

&nbsp;     Dog   Bird

&nbsp;            │

&nbsp;            ↓

&nbsp;         Flyable





Dog es un Animal



Bird es un Animal



Bird puede hacer Flyable



⚠️ Errores frecuentes

Confundir extends e implements



Para heredar de una clase:



class Dog extends Animal {

}





Para implementar una interfaz:



class Dog implements Runnable {

}





Una interfaz puede extender otra interfaz:



interface AdvancedFlyable extends Flyable {

}



Resumen

class     → extends    → class

class     → implements → interface

interface → extends    → interface



📌 Ejemplo completo

interface Payable {

&nbsp;   void pay();

}



abstract class Employee {



&nbsp;   protected String name;



&nbsp;   public Employee(String name) {

&nbsp;       this.name = name;

&nbsp;   }



&nbsp;   public void work() {

&nbsp;       System.out.println(name + " is working");

&nbsp;   }



&nbsp;   public abstract void calculateSalary();

}



class Developer extends Employee implements Payable {



&nbsp;   public Developer(String name) {

&nbsp;       super(name);

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void calculateSalary() {

&nbsp;       System.out.println("Calculating developer salary...");

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void pay() {

&nbsp;       System.out.println("Paying developer...");

&nbsp;   }

}





Conceptualmente:



&nbsp;         Employee

&nbsp;            ↑

&nbsp;       Developer ─────→ Payable





Developer:



hereda estado de Employee



hereda comportamiento de Employee



debe implementar calculateSalary()



implementa el contrato de Payable



puede tener comportamiento específico propio



🚀 Resumen en 10 segundos

interface

&nbsp;   = contrato

&nbsp;   = capacidad

&nbsp;   = "puede hacer"

&nbsp;   = permite múltiples interfaces



abstract class

&nbsp;   = clase base

&nbsp;   = herencia

&nbsp;   = "es un"

&nbsp;   = comparte estado/código

&nbsp;   = solo una clase padre



Regla práctica

¿Quiero definir una capacidad?

&nbsp;       ↓

&nbsp;   interface



¿Quiero compartir estado o código

entre clases relacionadas?

&nbsp;       ↓

&nbsp;   abstract class



🎯 Regla definitiva



Interface = qué puede hacer.



Abstract class = qué es y qué comparte con sus hijos.

