Java: Abstract Class vs Interface — Chuleta

🧠 Idea principal



Tanto las abstract class como las interface sirven para conseguir abstracción y definir una estructura que otras clases deben seguir.



La diferencia fundamental:



ABSTRACT CLASS

&nbsp;   ↓

Base común

&nbsp;   ↓

Comparte estado + código

&nbsp;   ↓

"ES UN"



INTERFACE

&nbsp;   ↓

Contrato / comportamiento

&nbsp;   ↓

Define lo que una clase debe hacer

&nbsp;   ↓

"IMPLEMENTA / PUEDE HACER"



🟣 Abstract Class



Una abstract class es una clase que no puede instanciarse directamente.



Se utiliza como clase base para otras clases.



Puede contener:



métodos abstractos



métodos normales



atributos de instancia



constructores



métodos private



métodos protected



métodos public



métodos static



métodos final



Ejemplo:



abstract class Shape {



&nbsp;   // Método abstracto

&nbsp;   abstract double area();



&nbsp;   // Método concreto

&nbsp;   void display() {

&nbsp;       System.out.println("This is a shape");

&nbsp;   }

}





Una clase hija utiliza extends:



class Circle extends Shape {



&nbsp;   int radius = 5;



&nbsp;   @Override

&nbsp;   double area() {

&nbsp;       return 3.14 \* radius \* radius;

&nbsp;   }

}





Podemos utilizar una referencia del tipo abstracto:



Shape shape = new Circle();



shape.display();

System.out.println(shape.area());





Esto permite utilizar polimorfismo:



Shape

&nbsp; ↑

Circle



🔵 Interface



Una interface define un contrato que las clases que la implementan deben cumplir.



Ejemplo:



interface Drawable {



&nbsp;   void draw();

}





Una clase implementa la interfaz mediante implements:



class Rectangle implements Drawable {



&nbsp;   @Override

&nbsp;   public void draw() {

&nbsp;       System.out.println("Drawing Rectangle");

&nbsp;   }

}





Podemos utilizar una referencia de tipo interfaz:



Drawable d = new Rectangle();



d.draw();





Esto también permite polimorfismo:



Drawable

&nbsp;   ↑

Rectangle



📌 Métodos en una Interface



En el modelo tradicional, los métodos declarados en una interfaz son abstractos.



Pero las interfaces modernas también pueden tener:



abstract

default

static

private





Por ejemplo:



interface Example {



&nbsp;   // Abstracto

&nbsp;   void method1();



&nbsp;   // Implementación por defecto

&nbsp;   default void method2() {

&nbsp;       System.out.println("Default");

&nbsp;   }



&nbsp;   // Método estático

&nbsp;   static void method3() {

&nbsp;       System.out.println("Static");

&nbsp;   }

}





Los métodos private también son posibles desde Java 9.



⚔️ Comparación rápida

Característica	Abstract Class	Interface

Instanciable directamente	❌	❌

Métodos abstractos	✅	✅

Métodos implementados	✅	default, static, private

Variables de instancia	✅	❌

Constantes	✅	✅

Constructor	✅	❌

Estado de instancia	✅	❌

Métodos private	✅	✅ desde Java 9

Métodos protected	✅	❌

Métodos public	✅	✅

Métodos final	✅	❌

Métodos static	✅	✅

Herencia múltiple	❌	✅ mediante interfaces

Palabra clave	extends	implements

Propósito principal	Base común	Contrato / comportamiento

Acoplamiento	Mayor	Menor

🔑 extends vs implements

Abstract Class



Una clase extiende otra clase:



class Circle extends Shape {

}



Circle

&nbsp;  ↓ extends

Shape



Interface



Una clase implementa una interfaz:



class Rectangle implements Drawable {

}



Rectangle

&nbsp;   ↓ implements

Drawable



Una interfaz puede extender otra interfaz

interface AdvancedDrawable extends Drawable {

}





Resumen:



class     → extends    → class

class     → implements → interface

interface → extends    → interface



🧬 Herencia

Abstract Class: una sola clase padre



Java no permite:



class MyClass extends ClassA, ClassB { // ❌

}





Una clase solo puede extender una clase.



class MyClass extends ClassA {

}



Interface: múltiples interfaces



Una clase puede implementar varias interfaces:



class MyClass implements InterfaceA, InterfaceB, InterfaceC {

}





Esto permite combinar diferentes comportamientos:



&nbsp;             ┌→ Flyable

Bird ─────────┼→ Swimmable

&nbsp;             └→ Runnable



📦 Estado y variables

Abstract Class



Puede mantener estado:



abstract class Employee {



&nbsp;   protected String name;

&nbsp;   protected double salary;



&nbsp;   public Employee(String name, double salary) {

&nbsp;       this.name = name;

&nbsp;       this.salary = salary;

&nbsp;   }

}





Cada objeto puede tener sus propios valores:



Employee

&nbsp;├── name

&nbsp;└── salary



Interface



Las variables declaradas en una interfaz son, por defecto:



public

static

final





Por tanto, funcionan como constantes:



interface Constants {



&nbsp;   int MAX\_USERS = 100;

}





Equivale conceptualmente a:



public static final int MAX\_USERS = 100;





No sirven para mantener estado individual de cada objeto.



🏗️ Constructores

Abstract Class



Puede tener constructores:



abstract class Animal {



&nbsp;   protected String name;



&nbsp;   public Animal(String name) {

&nbsp;       this.name = name;

&nbsp;   }

}





La clase hija puede llamar al constructor mediante super():



class Dog extends Animal {



&nbsp;   public Dog(String name) {

&nbsp;       super(name);

&nbsp;   }

}



Interface



Una interfaz no tiene constructores:



interface Animal {

&nbsp;   // ❌ No constructor

}





Una interfaz no representa directamente un objeto que haya que inicializar.



🔒 Modificadores de acceso

Abstract Class



Sus métodos pueden tener diferentes niveles de acceso:



abstract class Example {



&nbsp;   private void method1() {

&nbsp;   }



&nbsp;   protected void method2() {

&nbsp;   }



&nbsp;   public void method3() {

&nbsp;   }

}



Interface



Los métodos abstractos de una interfaz son public por defecto.



interface Example {



&nbsp;   void execute();

}





Es equivalente a:



interface Example {



&nbsp;   public abstract void execute();

}





Las interfaces modernas también permiten métodos private.



🛑 Métodos final

Abstract Class



Puede tener métodos final:



abstract class Animal {



&nbsp;   final void breathe() {

&nbsp;       System.out.println("Breathing...");

&nbsp;   }

}





Una subclase no puede sobrescribirlo:



class Dog extends Animal {



&nbsp;   // ❌ No se puede sobrescribir breathe()

}



Interface



Los métodos de una interfaz no pueden declararse final.



interface Example {



&nbsp;   // ❌ No válido

&nbsp;   final void method();

}



🔄 Polimorfismo



Ambos mecanismos permiten polimorfismo.



Abstract Class

abstract class Animal {



&nbsp;   abstract void sound();

}



class Dog extends Animal {



&nbsp;   @Override

&nbsp;   void sound() {

&nbsp;       System.out.println("Woof");

&nbsp;   }

}





Uso:



Animal animal = new Dog();



animal.sound();



Interface

interface Animal {



&nbsp;   void sound();

}



class Dog implements Animal {



&nbsp;   @Override

&nbsp;   public void sound() {

&nbsp;       System.out.println("Woof");

&nbsp;   }

}





Uso:



Animal animal = new Dog();



animal.sound();





En ambos casos:



Referencia

&nbsp;   ↓

Animal

&nbsp;   ↓

Objeto real

&nbsp;   ↓

Dog



🎯 ¿Cuándo usar cada uno?

Usa abstract class cuando...



Las clases están estrechamente relacionadas y quieres compartir:



código



estado



atributos



constructores



comportamiento



Ejemplo:



&nbsp;            Vehicle

&nbsp;               ↑

&nbsp;       ┌───────┼───────┐

&nbsp;       ↓       ↓       ↓

&nbsp;      Car    Truck   Bus





Todos son vehículos y probablemente comparten características.



abstract class Vehicle {



&nbsp;   protected String brand;



&nbsp;   public Vehicle(String brand) {

&nbsp;       this.brand = brand;

&nbsp;   }



&nbsp;   public void stop() {

&nbsp;       System.out.println("Stopping...");

&nbsp;   }



&nbsp;   abstract void start();

}



🎯 Usa interface cuando...



Quieres definir un comportamiento o contrato que pueden compartir clases que no necesariamente están relacionadas por herencia.



Ejemplo:



Bird ───────→ Flyable

Airplane ───→ Flyable

Drone ──────→ Flyable





Los tres pueden volar, pero no necesariamente pertenecen a la misma jerarquía de clases.



interface Flyable {



&nbsp;   void fly();

}





Después:



class Bird implements Flyable {



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Bird flying");

&nbsp;   }

}



class Airplane implements Flyable {



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Airplane flying");

&nbsp;   }

}



🔗 Acoplamiento



Una diferencia importante señalada por GeeksforGeeks es el acoplamiento.



Abstract Class



Normalmente produce una relación más fuerte entre la clase base y sus subclases.



Employee

&nbsp;   ↑

Developer





Developer depende de la estructura de Employee.



Interface



Favorece un acoplamiento más débil.



Developer ───→ Payable

Customer  ───→ Payable

Invoice   ───→ Payable





Las clases solo necesitan cumplir el contrato de Payable.



Esto facilita cambiar las implementaciones sin cambiar el código que depende de la interfaz.



🧩 Ejemplo combinando ambas



Una situación habitual es utilizar las dos:



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

&nbsp;       System.out.println("Calculating salary...");

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void pay() {

&nbsp;       System.out.println("Paying developer...");

&nbsp;   }

}





Conceptualmente:



&nbsp;             Employee

&nbsp;                ↑

&nbsp;            Developer

&nbsp;                │

&nbsp;                ↓

&nbsp;             Payable





Developer:



es un Employee



hereda estado de Employee



hereda comportamiento de Employee



debe implementar calculateSalary()



cumple el contrato Payable



🧠 Regla mental

Abstract Class

¿Las clases están relacionadas?



&nbsp;       ↓ Sí



¿Quiero compartir estado/código?



&nbsp;       ↓ Sí



&nbsp;  ABSTRACT CLASS





Piensa:



"Es un..."



Ejemplos:



Dog → Animal

Car → Vehicle

Developer → Employee



Interface

¿Quiero definir un comportamiento?



&nbsp;       ↓ Sí



¿Pueden tenerlo clases diferentes?



&nbsp;       ↓ Sí



&nbsp;     INTERFACE





Piensa:



"Puede hacer..."



Ejemplos:



Bird → Flyable

Printer → Printable

Employee → Payable

Car → Drivable



⚡ Chuleta ultra rápida

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

│ No mantiene estado de instancia        │

│ No tiene constructor                    │

│ Variables = constantes                  │

│ Puede tener abstract/default/static      │

│ Puede tener private desde Java 9        │

│ Varias interfaces por clase             │

│ implements                              │

└─────────────────────────────────────────┘



🚀 Resumen en 10 segundos

ABSTRACT CLASS

&nbsp;   ↓

"¿QUÉ ES?"

&nbsp;   ↓

Base común

&nbsp;   ↓

Comparte estado + implementación

&nbsp;   ↓

extends



INTERFACE

&nbsp;   ↓

"¿QUÉ PUEDE HACER?"

&nbsp;   ↓

Contrato / comportamiento

&nbsp;   ↓

Menor acoplamiento

&nbsp;   ↓

implements



🎯 Regla definitiva



Abstract class = identidad + estado + comportamiento compartido.



Interface = contrato + capacidad + comportamiento común definido por contrato.



Ejemplo definitivo

&nbsp;                   Animal

&nbsp;                      ↑

&nbsp;                    Bird

&nbsp;                      │

&nbsp;                      ├────→ Flyable

&nbsp;                      │

&nbsp;                      └────→ Swimmable



abstract class Animal {

&nbsp;   protected String name;



&nbsp;   public Animal(String name) {

&nbsp;       this.name = name;

&nbsp;   }

}



interface Flyable {

&nbsp;   void fly();

}



interface Swimmable {

&nbsp;   void swim();

}



class Bird extends Animal implements Flyable, Swimmable {



&nbsp;   public Bird(String name) {

&nbsp;       super(name);

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void fly() {

&nbsp;       System.out.println("Flying");

&nbsp;   }



&nbsp;   @Override

&nbsp;   public void swim() {

&nbsp;       System.out.println("Swimming");

&nbsp;   }

}





Memoriza únicamente esto:



extends    → heredar una clase

implements → cumplir una interfaz



abstract class → "es un"

interface      → "puede hacer"





Nota: GeeksforGeeks presenta la interfaz principalmente como un contrato cuyos métodos abstractos deben implementar las clases, pero en Java moderno también existen métodos default, static y private; por eso la distinción no debe reducirse simplemente a "interfaz = solo métodos abstractos". {"fallbackMarkdown":"(GeeksforGeeks

)","reference":{"matched\_text":"","prefix":null,"start\_idx":13918,"end\_idx":13937,"safe\_urls":\["https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/","https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/?utm\_source=chatgpt.com"],"refs":\[],"alt":"(GeeksforGeeks

)","prompt\_text":null,"type":"grouped\_webpages","fallback\_items":null,"status":"done","error":null,"style":null,"items":\[{"title":"Difference Between Abstract Class and Interface in Java - GeeksforGeeks","url":"https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/?utm\_source=chatgpt.com","attribution":"GeeksforGeeks","pub\_date":1778889600,"snippet":"","attribution\_segments":null,"supporting\_websites":\[],"refs":\[{"turn\_index":1,"ref\_type":"search","ref\_index":0}],"hue":null,"attributions":null}]},"showLoginRequiredCard":false}

:::{"fallbackMarkdown":"","reference":{"matched\_text":" ","prefix":null,"start\_idx":13941,"end\_idx":13941,"safe\_urls":\[],"refs":\[],"alt":"","prompt\_text":null,"type":"sources\_footnote","sources":\[{"title":"Difference Between Abstract Class and Interface in Java - GeeksforGeeks","url":"https://www.geeksforgeeks.org/java/difference-between-abstract-class-and-interface-in-java/?utm\_source=chatgpt.com","attribution":"GeeksforGeeks"}],"has\_images":false},"showLoginRequiredCard":false}

