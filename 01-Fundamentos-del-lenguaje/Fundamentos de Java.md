# Fundamentos de Java (Java 25+)

> **Alcance:** esta guía recorre las bases del lenguaje —estructura de un programa, variables y tipos de datos, operadores, estructuras de control, arrays y métodos— con la sintaxis actual de Java 25 (versión LTS). Todos los programas completos se han compilado y ejecutado con OpenJDK 25.

---

## Índice

**Primeros pasos**

0. [Cómo probar los ejemplos](#0-cómo-probar-los-ejemplos)
1. [Qué es Java](#1-qué-es-java)
2. [El primer programa](#2-el-primer-programa)
3. [Anatomía de un programa Java](#3-anatomía-de-un-programa-java)

**Variables y tipos de datos**

4. [Variables](#4-variables)
5. [Tipos primitivos](#5-tipos-primitivos)
6. [Tipos referencia, String y clases envoltorio](#6-tipos-referencia-string-y-clases-envoltorio)
7. [Conversión de tipos](#7-conversión-de-tipos)
8. [Entrada por teclado](#8-entrada-por-teclado)

**Operadores**

9. [Operadores aritméticos](#9-operadores-aritméticos)
10. [Operadores de asignación, incremento y decremento](#10-operadores-de-asignación-incremento-y-decremento)
11. [Operadores relacionales y lógicos](#11-operadores-relacionales-y-lógicos)
12. [Otros operadores y precedencia](#12-otros-operadores-y-precedencia)

**Estructuras de control**

13. [Condicionales: if y else](#13-condicionales-if-y-else)
14. [La sentencia y la expresión switch](#14-la-sentencia-y-la-expresión-switch)
15. [Bucles](#15-bucles)
16. [break, continue y etiquetas](#16-break-continue-y-etiquetas)

**Arrays**

17. [Arrays unidimensionales](#17-arrays-unidimensionales)
18. [Recorrer arrays](#18-recorrer-arrays)
19. [Arrays multidimensionales](#19-arrays-multidimensionales)
20. [La clase Arrays](#20-la-clase-arrays)
21. [Los arrays son referencias](#21-los-arrays-son-referencias)

**Métodos**

22. [Definir y llamar a un método](#22-definir-y-llamar-a-un-método)
23. [Parámetros y paso por valor](#23-parámetros-y-paso-por-valor)
24. [Sobrecarga de métodos](#24-sobrecarga-de-métodos)
25. [Argumentos variables (varargs)](#25-argumentos-variables-varargs)
26. [Recursividad](#26-recursividad)
27. [Ámbito de las variables](#27-ámbito-de-las-variables)
28. [Buenas prácticas al escribir métodos](#28-buenas-prácticas-al-escribir-métodos)

**Para terminar**

29. [Ejemplo completo: estadísticas de notas](#29-ejemplo-completo-estadísticas-de-notas)
30. [Errores frecuentes](#30-errores-frecuentes)
31. [Chuleta rápida](#31-chuleta-rápida)
32. [Ejercicios propuestos](#32-ejercicios-propuestos)
33. [Soluciones](#33-soluciones)
34. [Glosario](#34-glosario)
35. [Referencias](#35-referencias)

---

## 0. Cómo probar los ejemplos

Necesitas tener instalado un **JDK 25** (por ejemplo, [OpenJDK](https://jdk.java.net/) o [Eclipse Temurin](https://adoptium.net/)). Comprueba la versión con:

```bash
java -version
```

Hay tres formas de ejecutar los ejemplos:

| Forma | Cuándo usarla | Comando |
|---|---|---|
| **Lanzador de ficheros fuente** | Programas de un solo fichero (la mayoría de esta guía) | `java Programa.java` |
| **Compilar y ejecutar** | Proyectos con varios ficheros o para ver los avisos del compilador | `javac Programa.java` y luego `java Programa` |
| **JShell** | Probar una línea suelta sin escribir un programa | `jshell` |

Los fragmentos cortos (sin `void main()`) se pueden pegar tal cual en **JShell**, la consola interactiva de Java:

```text
$ jshell
jshell> int x = 7 / 2;
x ==> 3
jshell> /exit
```

> 💡 Para ver todos los avisos del compilador, usa `javac -Xlint:all Programa.java`.

---

## 1. Qué es Java

**Java** es un lenguaje de programación de propósito general, **orientado a objetos**, **fuertemente tipado** y **compilado a bytecode**. Nació en Sun Microsystems en 1995 y hoy lo mantiene Oracle junto con la comunidad OpenJDK.

### 1.1. Cómo se ejecuta un programa Java

```text
 Programa.java  ──javac──►  Programa.class  ──java (JVM)──►  Ejecución
 (código fuente)            (bytecode)                       en Linux, Windows, macOS...
```

1. Escribes el **código fuente** en ficheros `.java`.
2. El compilador **`javac`** lo traduce a **bytecode** (ficheros `.class`), un código intermedio que no depende del sistema operativo.
3. La **JVM** (*Java Virtual Machine*) interpreta ese bytecode y, mientras el programa se ejecuta, compila a código máquina las partes que más se usan (compilación *JIT*, *Just-In-Time*).

Por eso se dice que Java es **«escribe una vez, ejecuta en cualquier parte»**: el mismo `.class` funciona en cualquier sistema que tenga una JVM.

### 1.2. JDK, JRE y JVM

| Sigla | Qué es | Qué contiene |
|---|---|---|
| **JVM** | Máquina virtual de Java | Ejecuta el bytecode |
| **JRE** | Entorno de ejecución | JVM + bibliotecas estándar. Desde Java 11 ya no se distribuye por separado |
| **JDK** | Kit de desarrollo | JRE + herramientas (`javac`, `java`, `jshell`, `javadoc`, `jar`...) |

Para programar siempre necesitas el **JDK**.

### 1.3. Versiones y LTS

Java publica una versión nueva cada **seis meses** (marzo y septiembre). Cada dos años, una de ellas es **LTS** (*Long-Term Support*), con soporte durante muchos años. Las últimas LTS son **Java 17**, **Java 21** y **Java 25** (septiembre de 2025). Las empresas suelen trabajar con versiones LTS.

### 1.4. Características principales

- **Tipado estático y fuerte:** cada variable tiene un tipo que se comprueba al compilar. Muchos errores se detectan antes de ejecutar.
- **Orientado a objetos:** el código se organiza en clases y objetos.
- **Gestión automática de memoria:** el **recolector de basura** (*garbage collector*) libera la memoria de los objetos que ya no se usan. No hay que liberar memoria a mano.
- **Biblioteca estándar enorme:** colecciones, fechas, ficheros, red, concurrencia...
- **Distingue mayúsculas de minúsculas:** `edad`, `Edad` y `EDAD` son tres nombres distintos.

---

## 2. El primer programa

### 2.1. Versión moderna: fichero fuente compacto (Java 25)

Desde Java 25 (JEP 512) puedes escribir un programa sin declarar una clase:

```java
// Hola.java
void main() {
    IO.println("¡Hola, mundo!");
}
```

```bash
$ java Hola.java
¡Hola, mundo!
```

- `void main()` es el **punto de entrada**: lo primero que se ejecuta.
- `IO.println(...)` muestra un texto y salta de línea. La clase `IO` (paquete `java.lang`, nueva en Java 25) ofrece `print`, `println` y `readln` para la consola.
- Un fichero compacto **importa automáticamente** todo el módulo `java.base`, así que puedes usar `List`, `Map`, `LocalDate`, `Arrays`, etc. sin escribir `import`.

### 2.2. Versión clásica

Es la forma que encontrarás en la mayoría de proyectos y libros, y la necesaria en cuanto el programa tiene varios ficheros:

```java
// Hola.java
public class Hola {

    public static void main(String[] args) {
        System.out.println("¡Hola, mundo!");
    }
}
```

| Elemento | Significado |
|---|---|
| `public class Hola` | Declara una clase pública llamada `Hola`. El fichero **debe llamarse** `Hola.java` |
| `public` | El método se puede llamar desde fuera de la clase (la JVM lo necesita) |
| `static` | El método pertenece a la clase; no hace falta crear un objeto para llamarlo |
| `void` | El método no devuelve nada |
| `String[] args` | Argumentos que se pasan al programa desde la línea de comandos |
| `System.out.println` | Escribe en la salida estándar y salta de línea |

Ambas versiones son válidas en Java 25. En esta guía usaremos la compacta para que los ejemplos sean más cortos; todo lo que se explica funciona igual en la versión clásica.

> 💡 `System.out.println` e `IO.println` hacen lo mismo. `IO` es más corto y además permite leer del teclado (sección 8).

---

## 3. Anatomía de un programa Java

### 3.1. Sentencias y bloques

- Una **sentencia** es una instrucción completa y termina en **punto y coma** (`;`).
- Un **bloque** es un grupo de sentencias entre **llaves** `{ }`. Los bloques delimitan clases, métodos, condicionales y bucles.

```java
void main() {                       // inicio del bloque del método
    int a = 5;                      // sentencia
    int b = 3;                      // sentencia
    if (a > b) {                    // inicio del bloque del if
        IO.println("a es mayor");
    }                               // fin del bloque del if
}                                   // fin del bloque del método
```

### 3.2. Comentarios

```java
// Comentario de una línea

/*
 * Comentario de varias líneas.
 */

/**
 * Comentario de documentación clásico (Javadoc), en HTML.
 * @param nombre nombre de la persona
 */

/// Comentario de documentación en Markdown (Java 23+).
/// Se puede usar **negrita**, `código` y listas.
/// @param nombre nombre de la persona
```

Los comentarios de documentación (`/** */` o `///`) se colocan justo antes de una clase o un método y la herramienta `javadoc` los convierte en páginas HTML.

### 3.3. Convenciones de nombres

No son obligatorias para el compilador, pero **todo el mundo las sigue** y conviene respetarlas desde el principio:

| Elemento | Convención | Ejemplo |
|---|---|---|
| Clases, interfaces, records, enums | `PascalCase` (UpperCamelCase) | `CuentaBancaria` |
| Métodos y variables | `camelCase` (lowerCamelCase) | `calcularTotal`, `precioFinal` |
| Constantes | `MAYUSCULAS_CON_GUIONES_BAJOS` | `IVA_GENERAL` |
| Paquetes | minúsculas, separadas por puntos | `com.empresa.facturacion` |

Recomendaciones:

- Usa nombres **descriptivos**: `totalConIva` mejor que `t`.
- Los métodos suelen empezar por un **verbo**: `calcular`, `obtener`, `esValido`.
- Los booleanos se leen mejor como pregunta: `esMayorDeEdad`, `tieneStock`.
- Evita tildes y la `ñ` en los identificadores (Java las admite, pero dan problemas con herramientas y teclados).

### 3.4. Palabras reservadas

Son palabras con significado especial que **no se pueden usar como nombres**: `class`, `public`, `static`, `void`, `int`, `if`, `else`, `for`, `while`, `return`, `new`, `true`, `false`, `null`, etc.

Algunas, como `var`, `record`, `yield`, `sealed` o `when`, son **palabras reservadas contextuales**: solo tienen significado especial en ciertos lugares, por eso código antiguo que las usaba como nombres sigue compilando.

---

## 4. Variables

Una **variable** es un espacio de memoria con un **nombre** y un **tipo** que guarda un valor que puede cambiar.

### 4.1. Declaración, inicialización y asignación

```java
int edad;              // declaración: tipo + nombre
edad = 30;             // asignación
int anios = 30;        // declaración e inicialización en una línea
anios = 31;            // nueva asignación: el valor anterior se pierde

int x = 1, y = 2;      // varias en una línea (mejor evitarlo: menos legible)
```

Una **variable local** (declarada dentro de un método) **debe tener valor antes de usarse**. Si no, el código no compila:

```java
int total;
IO.println(total);     // ❌ error: variable total might not have been initialized
```

### 4.2. Constantes con final

`final` impide volver a asignar la variable:

```java
final double IVA_GENERAL = 0.21;
IVA_GENERAL = 0.10;    // ❌ error: cannot assign a value to final variable
```

Las constantes de una clase se declaran normalmente como `static final`:

```java
static final int MAX_INTENTOS = 3;
```

### 4.3. Inferencia de tipos con var (Java 10+)

Con `var` el compilador **deduce el tipo** a partir del valor inicial. El tipo sigue siendo fijo: Java no se vuelve dinámico.

```java
var nombre = "Lucía";          // String
var edad = 28;                 // int
var precio = 19.95;            // double
var lista = new ArrayList<String>();   // ArrayList<String>

edad = "veintiocho";           // ❌ error: edad es int
```

Reglas:

- Solo para **variables locales** con inicialización (no para atributos ni parámetros).
- `var x;` y `var x = null;` no compilan: no hay de dónde deducir el tipo.
- Úsalo cuando el tipo sea **evidente** por la parte derecha. Si oculta información (`var r = calcular();`), mejor escribir el tipo.

### 4.4. Reglas para los identificadores

Un nombre de variable:

- Puede contener letras, dígitos, `_` y `$`.
- **No puede empezar por un dígito**: `1nota` no es válido; `nota1` sí.
- No puede ser una palabra reservada.
- `_` solo, desde Java 22, significa **variable sin nombre** (algo que se ignora; se ve en la sección 15.4).

---

## 5. Tipos primitivos

Java tiene **8 tipos primitivos**. Guardan directamente el valor (no una referencia a un objeto) y tienen un tamaño fijo en cualquier plataforma.

| Tipo | Tamaño | Rango / valores | Valor por defecto* | Ejemplo |
|---|---|---|---|---|
| `byte` | 8 bits | −128 a 127 | `0` | `byte b = 100;` |
| `short` | 16 bits | −32 768 a 32 767 | `0` | `short s = 30_000;` |
| `int` | 32 bits | −2 147 483 648 a 2 147 483 647 (≈ ±2 100 millones) | `0` | `int n = 42;` |
| `long` | 64 bits | ≈ ±9,2 × 10¹⁸ | `0L` | `long l = 8_000_000_000L;` |
| `float` | 32 bits | ≈ 6-7 dígitos decimales de precisión | `0.0f` | `float f = 3.14f;` |
| `double` | 64 bits | ≈ 15-16 dígitos decimales de precisión | `0.0` | `double d = 3.14159;` |
| `char` | 16 bits | Un carácter Unicode (0 a 65 535) | `'\u0000'` | `char c = 'A';` |
| `boolean` | — | `true` o `false` | `false` | `boolean ok = true;` |

\* El valor por defecto se aplica a **atributos** de una clase y a los **elementos de un array**. Las variables locales no tienen valor por defecto.

**¿Cuál usar?**

- Números enteros: **`int`**. Si pueden superar los 2 100 millones (milisegundos, identificadores grandes), **`long`**.
- Números decimales: **`double`**.
- Dinero: **ni `float` ni `double`** (ver sección 9.4); usa `BigDecimal` o guarda céntimos en un `long`.
- `byte` y `short` solo en casos concretos (ficheros binarios, ahorrar memoria en arrays enormes).

### 5.1. Literales

Un **literal** es un valor escrito directamente en el código.

```java
int decimal     = 255;
int hexadecimal = 0xFF;              // 255 en base 16
int binario     = 0b1111_1111;       // 255 en base 2
int octal       = 0377;              // 255 en base 8 (¡cuidado con el 0 inicial!)

int millon      = 1_000_000;         // _ como separador visual (Java 7+)
long grande     = 9_000_000_000L;    // L final: literal long
float f         = 2.5f;              // f final: literal float (sin ella es double)
double d        = 2.5;
double cientif  = 1.5e3;             // 1500.0

char letra      = 'A';               // comillas simples
char unicode    = 'ñ';          // ñ
boolean activo  = true;
```

> ⚠️ `long l = 9000000000;` **no compila**: sin la `L`, el literal es un `int` y no cabe. Usa siempre `L` mayúscula (la `l` minúscula se confunde con un `1`).

### 5.2. Secuencias de escape

Dentro de `char` y `String` se usan para representar caracteres especiales:

| Secuencia | Significado |
|---|---|
| `\n` | Salto de línea |
| `\t` | Tabulador |
| `\"` | Comilla doble |
| `\'` | Comilla simple |
| `\\` | Barra invertida |
| `\uXXXX` | Carácter Unicode por su código hexadecimal |

```java
IO.println("Nombre:\tAna\nEdad:\t30");
IO.println("Ella dijo: \"hola\"");
```

```text
Nombre:	Ana
Edad:	30
Ella dijo: "hola"
```

### 5.3. char es un número

Un `char` se guarda como un número (su código Unicode), así que admite operaciones aritméticas:

```java
char c = 'A';
IO.println(c + 1);            // 66 (se suma como número)
IO.println((char) (c + 1));   // B
IO.println('a' < 'b');        // true
```

---

## 6. Tipos referencia, String y clases envoltorio

Todo lo que **no** es un primitivo es un **tipo referencia**: clases (`String`, `Scanner`, `LocalDate`...), interfaces, arrays, records y enums.

Una variable de tipo referencia **no contiene el objeto**, sino una **referencia** (una especie de dirección) al lugar de memoria donde está el objeto.

```text
 int edad = 30;              String nombre = "Ana";

 edad                        nombre
 ┌────┐                      ┌─────┐       ┌────────────┐
 │ 30 │                      │  ●──┼──────►│ "Ana"      │  (objeto en memoria)
 └────┘                      └─────┘       └────────────┘
```

### 6.1. null

`null` significa **«no apunta a ningún objeto»**. Solo los tipos referencia pueden ser `null`. Llamar a un método sobre `null` lanza `NullPointerException`:

```java
String texto = null;
IO.println(texto.length());   // 💥 NullPointerException
```

Desde Java 14, el mensaje de error indica exactamente qué era `null`: `Cannot invoke "String.length()" because "texto" is null`. (Si el código se compiló sin información de depuración, en lugar del nombre de la variable aparece algo como `"<local1>"`; los IDE y Maven/Gradle la incluyen por defecto).

### 6.2. String

`String` representa una cadena de texto. Es tan habitual que tiene literal propio (comillas dobles) y el operador `+` para concatenar.

```java
String nombre = "Ana";
String saludo = "Hola, " + nombre + "!";      // Hola, Ana!
String info = "Edad: " + 30;                  // el número se convierte a texto
```

**Los String son inmutables**: ningún método modifica la cadena original; todos devuelven una nueva.

```java
String s = "hola";
s.toUpperCase();              // devuelve "HOLA", pero no se guarda en ningún sitio
IO.println(s);                // hola
s = s.toUpperCase();          // ahora sí
IO.println(s);                // HOLA
```

Métodos más usados:

| Método | Resultado con `s = "  Hola Mundo  "` |
|---|---|
| `s.length()` | `14` |
| `s.strip()` | `"Hola Mundo"` (quita espacios al inicio y al final) |
| `s.strip().toUpperCase()` | `"HOLA MUNDO"` |
| `s.strip().toLowerCase()` | `"hola mundo"` |
| `s.strip().charAt(0)` | `'H'` (los índices empiezan en 0) |
| `s.strip().indexOf("Mundo")` | `5` (−1 si no lo encuentra) |
| `s.strip().substring(0, 4)` | `"Hola"` (desde 0 hasta 4, **sin incluir** el 4) |
| `s.contains("Mundo")` | `true` |
| `s.strip().startsWith("Ho")` | `true` |
| `s.strip().replace("Mundo", "Java")` | `"Hola Java"` |
| `s.isBlank()` | `false` (`true` si está vacío o solo tiene espacios) |
| `s.strip().split(" ")` | array `["Hola", "Mundo"]` |
| `"ab".repeat(3)` | `"ababab"` |

#### Comparar cadenas: equals, nunca ==

```java
String a = "java";
String b = new String("java");

IO.println(a == b);                       // false: compara referencias (¿es el mismo objeto?)
IO.println(a.equals(b));                  // true: compara el contenido
IO.println("JAVA".equalsIgnoreCase(a));   // true: ignora mayúsculas
```

> ⚠️ `==` con objetos compara si son **el mismo objeto**, no si tienen el mismo contenido. Para comparar texto usa **siempre `equals`**. Un truco para evitar `NullPointerException`: pon el literal delante, `"si".equals(respuesta)`.

#### Bloques de texto (Java 15+)

Para textos de varias líneas, usa triple comilla:

```java
String json = """
        {
          "nombre": "Ana",
          "edad": 30
        }
        """;
```

La sangría común se elimina automáticamente y los saltos de línea se respetan.

#### Formatear texto

```java
String nombre = "Ana";
double saldo = 1234.5;

String linea = "%s tiene %.2f €".formatted(nombre, saldo);     // Ana tiene 1234,50 €
IO.println(String.format("%-10s|%5d|", "Total", 42));          // Total     |   42|
```

| Especificador | Significado |
|---|---|
| `%s` | Texto (cualquier objeto) |
| `%d` | Entero |
| `%f`, `%.2f` | Decimal (con 2 decimales) |
| `%c` | Carácter |
| `%b` | Booleano |
| `%n` | Salto de línea |
| `%5d`, `%-10s` | Ancho mínimo 5 alineado a la derecha / 10 a la izquierda |

> 💡 El separador decimal depende de la configuración regional del sistema: con configuración española se muestra `1234,50`; en inglés, `1234.50`. Para fijarlo: `String.format(Locale.US, "%.2f", saldo)`.

#### StringBuilder

Concatenar con `+` dentro de un bucle crea muchas cadenas intermedias. Para construir texto poco a poco, usa `StringBuilder`, que sí es modificable:

```java
var sb = new StringBuilder();
for (int i = 1; i <= 5; i++) {
    sb.append(i).append(", ");
}
sb.setLength(sb.length() - 2);    // quita la última ", "
IO.println(sb);                   // 1, 2, 3, 4, 5
```

### 6.3. Clases envoltorio (wrappers)

Cada primitivo tiene una clase equivalente: `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`, `Character` y `Boolean`. Se necesitan porque las colecciones (`List`, `Map`...) solo guardan objetos.

Java convierte automáticamente entre ambos:

```java
Integer objeto = 5;        // autoboxing: int → Integer
int primitivo = objeto;    // unboxing: Integer → int

List<Integer> numeros = new ArrayList<>();
numeros.add(10);           // autoboxing automático
```

Utilidades de las clases envoltorio:

```java
int n = Integer.parseInt("123");          // String → int
double d = Double.parseDouble("3.5");     // String → double
boolean b = Boolean.parseBoolean("true"); // String → boolean
String s = String.valueOf(42);            // int → String (o "" + 42)

Integer.MAX_VALUE;                        // 2147483647
Integer.MIN_VALUE;                        // -2147483648
Character.isDigit('7');                   // true
Character.isLetter('a');                  // true
Character.toUpperCase('a');               // 'A'
```

`Integer.parseInt("abc")` lanza `NumberFormatException`.

#### Dos trampas de los envoltorios

```java
Integer a = 127, b = 127;
IO.println(a == b);          // true  (Java reutiliza los objetos entre -128 y 127)

Integer c = 128, d = 128;
IO.println(c == d);          // false (son objetos distintos)
IO.println(c.equals(d));     // true  ✅

Integer vacio = null;
int x = vacio;               // 💥 NullPointerException al hacer unboxing
```

Conclusión: **compara envoltorios con `equals`** y usa primitivos siempre que no necesites `null` ni colecciones.

---

## 7. Conversión de tipos

### 7.1. Conversión implícita (ampliación)

Java convierte automáticamente un tipo a otro **más grande** porque no se pierde información:

```text
byte → short → int → long → float → double
              char ↗
```

```java
int entero = 100;
long largo = entero;       // ✅ automático
double real = entero;      // ✅ 100.0
```

### 7.2. Conversión explícita (casting o estrechamiento)

Para pasar a un tipo **más pequeño** hay que indicarlo con un *cast* `(tipo)`, porque se puede perder información:

```java
double precio = 9.99;
int truncado = (int) precio;           // 9: se descartan los decimales, NO redondea
long redondeado = Math.round(precio);  // 10: redondeo correcto

int grande = 300;
byte pequeno = (byte) grande;          // 44: no cabe y "da la vuelta"
```

### 7.3. Promoción en expresiones

En una operación aritmética, los operandos `byte`, `short` y `char` se convierten a `int`, y si hay mezcla de tipos el resultado es del **tipo más grande**:

```java
byte a = 10, b = 20;
byte suma = a + b;          // ❌ error: el resultado es int
byte suma2 = (byte) (a + b);// ✅

int i = 5;
double d = 2.0;
var r = i * d;              // double: 10.0
```

### 7.4. Desbordamiento (overflow)

Si un resultado no cabe en el tipo, **no hay error**: el valor da la vuelta silenciosamente.

```java
int max = Integer.MAX_VALUE;
IO.println(max + 1);                  // -2147483648 😱

long correcto = (long) max + 1;       // 2147483648 ✅
Math.addExact(max, 1);                // 💥 ArithmeticException: integer overflow
```

Usa `Math.addExact`, `Math.multiplyExact`, etc. cuando un desbordamiento deba considerarse un error.

### 7.5. Resumen de conversiones con String

| De → A | Cómo |
|---|---|
| `String` → `int` | `Integer.parseInt(s)` |
| `String` → `double` | `Double.parseDouble(s)` |
| número → `String` | `String.valueOf(n)` o `"" + n` |
| `char` → `String` | `String.valueOf(c)` |
| `String` → `char` | `s.charAt(0)` |
| `char` dígito → `int` | `c - '0'` o `Character.getNumericValue(c)` |

---

## 8. Entrada por teclado

### 8.1. Con IO.readln (Java 25)

`IO.readln(mensaje)` muestra un mensaje y devuelve la línea escrita como `String`:

```java
void main() {
    String nombre = IO.readln("¿Cómo te llamas? ");
    int edad = Integer.parseInt(IO.readln("¿Cuántos años tienes? "));
    IO.println("Hola, " + nombre + ". El año que viene tendrás " + (edad + 1) + ".");
}
```

```text
¿Cómo te llamas? Marta
¿Cuántos años tienes? 24
Hola, Marta. El año que viene tendrás 25.
```

### 8.2. Con Scanner (forma clásica)

`Scanner` lee y convierte directamente a números. Es la opción que verás en la mayoría de ejemplos anteriores a Java 25:

```java
import java.util.Scanner;

public class Entrada {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Introduce un número: ");
        int numero = sc.nextInt();
        sc.nextLine();                         // consume el salto de línea pendiente
        System.out.print("Introduce tu nombre: ");
        String nombre = sc.nextLine();
        System.out.println(nombre + " ha escrito " + numero);
    }
}
```

> ⚠️ **Trampa clásica:** `nextInt()` lee el número pero deja el **salto de línea** en el búfer, y el siguiente `nextLine()` devuelve una cadena vacía. Solución: llamar a `nextLine()` una vez más (como en el ejemplo) o leer siempre líneas completas y convertirlas con `Integer.parseInt`.

> 💡 Para más detalle sobre entrada y salida, consulta la guía [Entrada - Salida básica](<../04-Entrada-salida-y-ficheros/Entrada - Salida basica.md>).

---

## 9. Operadores aritméticos

| Operador | Operación | Ejemplo | Resultado |
|---|---|---|---|
| `+` | Suma (y concatenación con `String`) | `7 + 2` | `9` |
| `-` | Resta | `7 - 2` | `5` |
| `*` | Multiplicación | `7 * 2` | `14` |
| `/` | División | `7 / 2` | `3` ⚠️ |
| `%` | Resto (módulo) | `7 % 2` | `1` |

### 9.1. División entera

Si **los dos operandos son enteros**, el resultado es entero y **se descartan los decimales**:

```java
IO.println(7 / 2);          // 3
IO.println(7.0 / 2);        // 3.5
IO.println((double) 7 / 2); // 3.5

int a = 7, b = 2;
double media = a / b;       // 3.0 ❌ la división se hizo antes, en enteros
double bien = (double) a / b;   // 3.5 ✅
```

Dividir un entero entre cero lanza `ArithmeticException`. Con `double` no hay excepción: `1.0 / 0` es `Infinity` y `0.0 / 0` es `NaN` (*Not a Number*).

### 9.2. El operador resto

`%` devuelve el resto de la división entera. Es muy útil para:

```java
boolean esPar = n % 2 == 0;           // par o impar
int unidades = 1234 % 10;             // 4: último dígito
boolean multiploDe3 = n % 3 == 0;     // divisibilidad
int posicion = (indice + 1) % tamano; // volver al principio (lista circular)
```

Con negativos, el resultado tiene el signo del dividendo: `-7 % 3` es `-1`. Si necesitas siempre un resultado positivo, usa `Math.floorMod(-7, 3)`, que devuelve `2`.

### 9.3. La clase Math

```java
Math.abs(-5);          // 5
Math.max(3, 8);        // 8
Math.min(3, 8);        // 3
Math.pow(2, 10);       // 1024.0 (siempre double)
Math.sqrt(16);         // 4.0
Math.round(2.5);       // 3
Math.floor(2.7);       // 2.0 (hacia abajo)
Math.ceil(2.1);        // 3.0 (hacia arriba)
Math.clamp(150, 0, 100); // 100: limita un valor a un rango (Java 21+)
Math.PI;               // 3.141592653589793
```

Para números aleatorios:

```java
int dado = (int) (Math.random() * 6) + 1;               // 1 a 6
int otro = ThreadLocalRandom.current().nextInt(1, 7);   // 1 a 6 (el 7 no se incluye)
```

### 9.4. Precisión de los decimales

`float` y `double` guardan los números en binario y **muchos decimales no se pueden representar de forma exacta**:

```java
IO.println(0.1 + 0.2);            // 0.30000000000000004
IO.println(0.1 + 0.2 == 0.3);     // false
```

Consecuencias:

- **No compares `double` con `==`**. Compara con un margen: `Math.abs(a - b) < 1e-9`.
- **No uses `double` para dinero.** Usa `BigDecimal`:

```java
var a = new BigDecimal("0.1");     // ¡con String, no con 0.1!
var b = new BigDecimal("0.2");
IO.println(a.add(b));              // 0.3
```

---

## 10. Operadores de asignación, incremento y decremento

### 10.1. Asignación compuesta

| Operador | Equivale a |
|---|---|
| `x += 5` | `x = x + 5` |
| `x -= 5` | `x = x - 5` |
| `x *= 5` | `x = x * 5` |
| `x /= 5` | `x = x / 5` |
| `x %= 5` | `x = x % 5` |

Detalle: la asignación compuesta incluye un *cast* implícito.

```java
byte b = 10;
b += 5;        // ✅ compila (equivale a b = (byte) (b + 5))
b = b + 5;     // ❌ error: b + 5 es int
```

### 10.2. Incremento y decremento

`++` suma 1 y `--` resta 1. La posición importa cuando se usan **dentro de otra expresión**:

```java
int a = 5;
int b = a++;   // postincremento: primero usa el valor (b = 5), luego incrementa (a = 6)

int c = 5;
int d = ++c;   // preincremento: primero incrementa (c = 6), luego usa el valor (d = 6)
```

> 💡 Como sentencia sola (`i++;`) da igual la forma. **Evita usarlos dentro de expresiones más grandes**: el código se vuelve difícil de leer.

---

## 11. Operadores relacionales y lógicos

### 11.1. Relacionales

Comparan dos valores y devuelven un `boolean`:

| Operador | Significado |
|---|---|
| `==` | Igual a |
| `!=` | Distinto de |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual que |
| `<=` | Menor o igual que |

Recuerda: con objetos (`String`, `Integer`...) usa `equals` en lugar de `==`.

### 11.2. Lógicos

| Operador | Nombre | Es `true` cuando... |
|---|---|---|
| `&&` | Y (AND) | **ambos** son `true` |
| `\|\|` | O (OR) | **al menos uno** es `true` |
| `!` | NO (NOT) | el operando es `false` |
| `^` | O exclusivo (XOR) | **solo uno** de los dos es `true` |

Tabla de verdad:

| `a` | `b` | `a && b` | `a \|\| b` | `!a` | `a ^ b` |
|---|---|---|---|---|---|
| `true` | `true` | `true` | `true` | `false` | `false` |
| `true` | `false` | `false` | `true` | `false` | `true` |
| `false` | `true` | `false` | `true` | `true` | `true` |
| `false` | `false` | `false` | `false` | `true` | `false` |

### 11.3. Evaluación en cortocircuito

`&&` y `||` **dejan de evaluar en cuanto conocen el resultado**:

- En `a && b`, si `a` es `false`, `b` no se evalúa.
- En `a || b`, si `a` es `true`, `b` no se evalúa.

Esto permite escribir comprobaciones seguras:

```java
String texto = null;
if (texto != null && texto.length() > 3) {   // si es null, no llega a llamar a length()
    IO.println("Texto largo");
}
```

Los operadores `&` y `|` también funcionan con `boolean`, pero **evalúan siempre los dos lados**. En condiciones, usa `&&` y `||`.

### 11.4. Expresar rangos

En Java no se puede escribir `18 <= edad <= 65`. Hay que combinar dos comparaciones:

```java
boolean enEdadLaboral = edad >= 18 && edad <= 65;
```

---

## 12. Otros operadores y precedencia

### 12.1. Operador ternario

Elige entre dos valores según una condición: `condición ? valorSiTrue : valorSiFalse`.

```java
String estado = edad >= 18 ? "mayor de edad" : "menor de edad";
int maximo = a > b ? a : b;
```

Úsalo para expresiones cortas. Si anidas varios ternarios, cámbialo por un `if` o un `switch`.

### 12.2. instanceof y patrones (Java 16+)

`instanceof` comprueba si un objeto es de un tipo. Desde Java 16, además declara una variable ya convertida:

```java
Object valor = "Hola";

// Antes
if (valor instanceof String) {
    String s = (String) valor;
    IO.println(s.length());
}

// Ahora
if (valor instanceof String s) {
    IO.println(s.length());       // s ya es String
}
```

### 12.3. Operadores de bits

Operan bit a bit sobre enteros. Son poco frecuentes en aplicaciones de negocio, pero conviene reconocerlos:

| Operador | Significado | Ejemplo | Resultado |
|---|---|---|---|
| `&` | AND bit a bit | `0b1100 & 0b1010` | `0b1000` (8) |
| `\|` | OR bit a bit | `0b1100 \| 0b1010` | `0b1110` (14) |
| `^` | XOR bit a bit | `0b1100 ^ 0b1010` | `0b0110` (6) |
| `~` | Complemento | `~0` | `-1` |
| `<<` | Desplazamiento a la izquierda | `1 << 3` | `8` (multiplica por 2³) |
| `>>` | Desplazamiento a la derecha con signo | `-16 >> 2` | `-4` |
| `>>>` | Desplazamiento a la derecha sin signo | `-1 >>> 28` | `15` |

### 12.4. Precedencia de operadores

De **mayor a menor** prioridad (los de arriba se evalúan antes):

| Prioridad | Operadores |
|---|---|
| 1 | `expr++` `expr--` |
| 2 | `++expr` `--expr` `+expr` `-expr` `!` `~` `(tipo)` |
| 3 | `*` `/` `%` |
| 4 | `+` `-` |
| 5 | `<<` `>>` `>>>` |
| 6 | `<` `>` `<=` `>=` `instanceof` |
| 7 | `==` `!=` |
| 8 | `&` |
| 9 | `^` |
| 10 | `\|` |
| 11 | `&&` |
| 12 | `\|\|` |
| 13 | `? :` |
| 14 | `=` `+=` `-=` `*=` `/=` `%=` ... |

```java
int r = 2 + 3 * 4;         // 14, no 20
int s = (2 + 3) * 4;       // 20
boolean b = 5 > 3 && 2 > 4 || true;   // (false) || true → true
```

> 💡 No hace falta memorizar la tabla: **usa paréntesis** cuando haya dudas. Hacen el código más claro para quien lo lea.

Una trampa habitual con `+` y cadenas: se evalúa de izquierda a derecha.

```java
IO.println(1 + 2 + " euros");    // 3 euros
IO.println("euros: " + 1 + 2);   // euros: 12
IO.println("euros: " + (1 + 2)); // euros: 3
```

---

## 13. Condicionales: if y else

### 13.1. if, else if, else

```java
int nota = 7;

if (nota >= 9) {
    IO.println("Sobresaliente");
} else if (nota >= 7) {
    IO.println("Notable");
} else if (nota >= 5) {
    IO.println("Aprobado");
} else {
    IO.println("Suspenso");
}
```

- Las condiciones se comprueban **en orden** y solo se ejecuta el **primer** bloque cuya condición sea `true`.
- `else if` y `else` son opcionales.
- La condición debe ser un `boolean`: `if (n)` con un `int` no compila (a diferencia de C o JavaScript).

### 13.2. Usa siempre llaves

Sin llaves, el `if` solo afecta a **la siguiente sentencia**, aunque la sangría diga otra cosa:

```java
if (saldo < 0)
    IO.println("Saldo negativo");
    bloquearCuenta();          // ⚠️ se ejecuta SIEMPRE: no pertenece al if
```

### 13.3. Cláusulas de guarda

Cuando hay muchas condiciones anidadas, suele ser más claro **salir pronto** de los casos especiales:

```java
// ❌ Anidado
String procesar(String pedido) {
    if (pedido != null) {
        if (!pedido.isBlank()) {
            return "Procesando " + pedido;
        } else {
            return "Pedido vacío";
        }
    } else {
        return "Sin pedido";
    }
}

// ✅ Con cláusulas de guarda
String procesar(String pedido) {
    if (pedido == null) return "Sin pedido";
    if (pedido.isBlank()) return "Pedido vacío";
    return "Procesando " + pedido;
}
```

---

## 14. La sentencia y la expresión switch

`switch` elige un camino según el **valor** de una expresión. Funciona con `int` (y `byte`, `short`, `char`), `String`, enums y, desde Java 21, con cualquier objeto mediante patrones.

### 14.1. switch con flechas (Java 14+): la forma recomendada

```java
int dia = 3;

switch (dia) {
    case 1, 2, 3, 4, 5 -> IO.println("Laborable");
    case 6, 7 -> IO.println("Fin de semana");
    default -> IO.println("Día no válido");
}
```

- Cada `case` puede tener **varios valores** separados por comas.
- Con `->` **solo se ejecuta la rama elegida** (no hay que escribir `break`).
- `default` recoge cualquier otro valor.

### 14.2. switch como expresión

Un `switch` también puede **devolver un valor**:

```java
String mes = "febrero";

int dias = switch (mes) {
    case "abril", "junio", "septiembre", "noviembre" -> 30;
    case "febrero" -> 28;
    default -> 31;
};
```

Si una rama necesita varias sentencias, se usa un bloque con **`yield`** para devolver el valor:

```java
int precio = switch (tipoEntrada) {
    case "infantil" -> 5;
    case "adulto" -> 10;
    case "jubilado" -> {
        IO.println("Aplicando descuento");
        yield 6;
    }
    default -> throw new IllegalArgumentException("Tipo desconocido: " + tipoEntrada);
};
```

Un `switch` usado como expresión debe ser **exhaustivo**: tiene que cubrir todos los valores posibles (normalmente con `default`).

### 14.3. switch clásico con dos puntos

Es la forma original y la verás en código antiguo. Cada caso necesita `break`; si se olvida, la ejecución **continúa en el caso siguiente** (*fall-through*):

```java
switch (dia) {
    case 6:
        IO.println("Sábado");
        break;
    case 7:
        IO.println("Domingo");
        break;
    default:
        IO.println("Entre semana");
}
```

```java
int opcion = 1;
switch (opcion) {
    case 1:
        IO.println("Uno");      // se imprime
    case 2:
        IO.println("Dos");      // ⚠️ también se imprime: falta el break
        break;
    case 3:
        IO.println("Tres");
}
```

En código nuevo, **prefiere la forma con flechas**: es más corta y evita este error.

### 14.4. switch con enums

Con un enum no hace falta escribir el nombre del tipo en cada `case`, y si el `switch` es una expresión que cubre **todas** las constantes, no necesita `default`:

```java
enum Semaforo { ROJO, AMBAR, VERDE }

String accion(Semaforo s) {
    return switch (s) {
        case ROJO -> "Detente";
        case AMBAR -> "Precaución";
        case VERDE -> "Adelante";
    };                       // exhaustivo: sin default
}
```

Si mañana se añade una constante nueva al enum, el compilador avisará de que falta un caso. Más detalles en la guía de [Enums](<Enums.md>).

### 14.5. switch con patrones (Java 21+)

`switch` puede comprobar el **tipo** de un objeto, declarar una variable y añadir condiciones con `when`:

```java
String describir(Object o) {
    return switch (o) {
        case null -> "nulo";
        case Integer i when i < 0 -> "entero negativo";
        case Integer i -> "entero: " + i;
        case String s when s.isBlank() -> "texto vacío";
        case String s -> "texto de " + s.length() + " caracteres";
        default -> "otro tipo: " + o.getClass().getSimpleName();
    };
}
```

- `case null` permite tratar el `null` sin que salte `NullPointerException`.
- Los casos se evalúan **en orden**: los más específicos (con `when`) deben ir antes que los generales. Si no, el compilador da error porque nunca se alcanzarían.

> 💡 Java 25 incluye en **vista previa** (JEP 507) los patrones con tipos primitivos (`case int i when i > 10`). Al ser una característica en vista previa, hay que activarla con `--enable-preview` y puede cambiar en versiones futuras.

---

## 15. Bucles

Un **bucle** repite un bloque de código mientras se cumpla una condición.

### 15.1. while

Comprueba la condición **antes** de cada vuelta. Si es `false` desde el principio, no se ejecuta ninguna vez.

```java
int cuenta = 3;
while (cuenta > 0) {
    IO.println(cuenta);
    cuenta--;
}
IO.println("¡Despegue!");
```

```text
3
2
1
¡Despegue!
```

Úsalo cuando **no sabes cuántas vueltas** habrá (por ejemplo, leer hasta que el usuario escriba «salir»).

### 15.2. do-while

Comprueba la condición **después** de cada vuelta, así que **se ejecuta al menos una vez**. Es típico para validar datos introducidos:

```java
int numero;
do {
    numero = Integer.parseInt(IO.readln("Introduce un número entre 1 y 10: "));
} while (numero < 1 || numero > 10);
```

Fíjate en el `;` después del `while`.

### 15.3. for

Reúne en una línea la **inicialización**, la **condición** y la **actualización**. Es el bucle habitual cuando sabes cuántas vueltas habrá:

```java
for (int i = 1; i <= 5; i++) {
    IO.print(i + " ");
}
// 1 2 3 4 5
```

```text
for (inicialización; condición; actualización) {
         │              │            │
         │              │            └─ se ejecuta al final de cada vuelta
         │              └─ se comprueba antes de cada vuelta
         └─ se ejecuta una sola vez, al principio
```

Variantes:

```java
for (int i = 10; i > 0; i -= 2) { ... }       // 10, 8, 6, 4, 2
for (int i = 0, j = 10; i < j; i++, j--) { ... }  // dos variables
for (;;) { ... }                              // bucle infinito (necesita break)
```

La variable `i` solo existe dentro del bucle.

### 15.4. for-each (for mejorado)

Recorre **todos los elementos** de un array o una colección sin manejar índices:

```java
String[] frutas = {"manzana", "pera", "uva"};
for (String fruta : frutas) {
    IO.println(fruta);
}
```

Úsalo siempre que **no necesites el índice**. Limitaciones: no permite saber la posición, recorrer al revés ni sustituir elementos del array.

Si solo quieres contar las vueltas y no usas el elemento, desde Java 22 puedes indicarlo con `_`:

```java
int total = 0;
for (String _ : frutas) {
    total++;
}
```

### 15.5. ¿Qué bucle elegir?

| Situación | Bucle |
|---|---|
| Recorrer todos los elementos de un array o colección | `for-each` |
| Necesitas el índice o un número fijo de vueltas | `for` |
| Repetir mientras se cumpla una condición, sin saber cuántas vueltas | `while` |
| El bloque debe ejecutarse al menos una vez (menús, validación) | `do-while` |

### 15.6. Bucles infinitos

Si la condición nunca llega a ser `false`, el programa no termina:

```java
int i = 0;
while (i < 10) {
    IO.println(i);
    // ⚠️ falta i++: bucle infinito
}
```

Si te ocurre al ejecutar, detén el programa con **Ctrl + C**.

---

## 16. break, continue y etiquetas

- **`break`** sale del bucle inmediatamente.
- **`continue`** salta al principio de la **siguiente vuelta**.

```java
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) continue;    // salta los pares
    if (i > 7) break;            // termina al pasar de 7
    IO.print(i + " ");
}
// 1 3 5 7
```

### 16.1. Etiquetas

En bucles anidados, `break` y `continue` afectan solo al bucle **más interno**. Para salir de un bucle exterior se usa una **etiqueta**:

```java
int[][] matriz = {{1, 2, 3}, {4, -5, 6}, {7, 8, 9}};

busqueda:
for (int[] fila : matriz) {
    for (int valor : fila) {
        if (valor < 0) {
            IO.println("Encontrado un negativo: " + valor);
            break busqueda;      // sale de los dos bucles
        }
    }
}
```

Las etiquetas son poco frecuentes. Muchas veces queda más claro sacar los bucles a un **método** y usar `return`.

---

## 17. Arrays unidimensionales

Un **array** (o *arreglo*) es una estructura que guarda un **número fijo** de elementos **del mismo tipo**, a los que se accede por su **posición** (índice).

```text
 índice:    0     1     2     3     4
          ┌─────┬─────┬─────┬─────┬─────┐
 notas →  │ 7.5 │ 9.0 │ 4.0 │ 6.5 │ 8.0 │      notas.length = 5
          └─────┴─────┴─────┴─────┴─────┘
```

### 17.1. Declarar y crear

```java
int[] numeros;                      // declaración (se prefiere tipo[] nombre)
numeros = new int[5];               // creación: 5 enteros con valor 0

double[] notas = new double[3];     // declaración y creación: [0.0, 0.0, 0.0]

String[] dias = {"lunes", "martes", "miércoles"};   // creación con valores
int[] primos = new int[] {2, 3, 5, 7};             // forma larga (útil al pasar a un método)
```

Al crear un array con `new`, sus elementos toman el **valor por defecto** del tipo: `0` para números, `false` para `boolean`, `'\u0000'` para `char` y `null` para objetos.

> 💡 También se admite `int numeros[]` (estilo C), pero en Java se recomienda `int[] numeros`: el tipo es «array de int».

### 17.2. Acceder y modificar

Los índices van de **`0`** a **`length - 1`**:

```java
String[] dias = {"lunes", "martes", "miércoles"};

IO.println(dias[0]);              // lunes
IO.println(dias[dias.length - 1]);// miércoles (último elemento)
dias[1] = "MARTES";               // modificar
IO.println(dias.length);          // 3 (length es un atributo: sin paréntesis)

IO.println(dias[3]);              // 💥 ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3
```

> ⚠️ `length` sin paréntesis en arrays; `length()` con paréntesis en `String`; `size()` en colecciones. Es una fuente de errores habitual al principio.

### 17.3. El tamaño es fijo

Un array **no puede crecer ni encoger**. Para «añadir» un elemento hay que crear otro array más grande y copiar:

```java
int[] original = {1, 2, 3};
int[] ampliado = Arrays.copyOf(original, original.length + 1);  // [1, 2, 3, 0]
ampliado[3] = 4;                                                 // [1, 2, 3, 4]
```

Si necesitas una lista que cambie de tamaño, usa **`ArrayList`**:

| | Array | `ArrayList` |
|---|---|---|
| Tamaño | Fijo | Crece y decrece |
| Tipos | Primitivos y objetos | Solo objetos (`Integer`, no `int`) |
| Sintaxis | `a[i]`, `a.length` | `lista.get(i)`, `lista.size()` |
| Cuándo | Tamaño conocido, rendimiento, datos de bajo nivel | La mayoría de casos en aplicaciones |

Más información en la guía de [Colecciones](<Colecciones.md>).

---

## 18. Recorrer arrays

```java
int[] numeros = {4, 8, 15, 16, 23, 42};

// for clásico: cuando necesitas el índice
for (int i = 0; i < numeros.length; i++) {
    IO.println("Posición " + i + ": " + numeros[i]);
}

// for-each: cuando solo necesitas los valores
int suma = 0;
for (int n : numeros) {
    suma += n;
}

// Recorrido inverso
for (int i = numeros.length - 1; i >= 0; i--) {
    IO.print(numeros[i] + " ");
}
```

### 18.1. Algoritmos básicos con arrays

```java
int[] datos = {12, 5, 27, 8, 19};

// Máximo
int max = datos[0];
for (int d : datos) {
    if (d > max) max = d;
}

// Media
int suma = 0;
for (int d : datos) suma += d;
double media = (double) suma / datos.length;

// Buscar un valor (búsqueda lineal)
int buscado = 8;
int posicion = -1;
for (int i = 0; i < datos.length; i++) {
    if (datos[i] == buscado) {
        posicion = i;
        break;
    }
}

// Contar los que cumplen una condición
int mayoresDe10 = 0;
for (int d : datos) {
    if (d > 10) mayoresDe10++;
}
```

Las mismas operaciones con *streams* ocupan una línea cada una (se explican en la guía [Lambdas y Streams](<../02-Programacion-funcional/Lambdas y Streams.md>)):

```java
int max = Arrays.stream(datos).max().orElseThrow();
double media = Arrays.stream(datos).average().orElse(0);
long mayoresDe10 = Arrays.stream(datos).filter(d -> d > 10).count();
```

---

## 19. Arrays multidimensionales

Un array bidimensional es un **array de arrays**. Se usa para tablas, tableros o matrices.

```java
int[][] matriz = {
    {1, 2, 3},
    {4, 5, 6}
};

IO.println(matriz[1][2]);        // 6: fila 1, columna 2
IO.println(matriz.length);       // 2: número de filas
IO.println(matriz[0].length);    // 3: columnas de la fila 0

int[][] tablero = new int[8][8]; // 8×8 con ceros
```

```text
              columna 0  columna 1  columna 2
 fila 0  →  [    1     ,    2     ,    3     ]
 fila 1  →  [    4     ,    5     ,    6     ]
```

Recorrido con bucles anidados:

```java
for (int fila = 0; fila < matriz.length; fila++) {
    for (int col = 0; col < matriz[fila].length; col++) {
        IO.print(matriz[fila][col] + "\t");
    }
    IO.println();
}
```

### 19.1. Arrays irregulares

Como cada fila es un array independiente, **las filas pueden tener tamaños distintos**:

```java
int[][] triangulo = new int[3][];   // solo se indica el número de filas
triangulo[0] = new int[1];
triangulo[1] = new int[2];
triangulo[2] = new int[3];
```

Por eso, al recorrer, usa `matriz[fila].length` y no un número fijo de columnas.

---

## 20. La clase Arrays

`java.util.Arrays` ofrece métodos estáticos para trabajar con arrays:

```java
int[] numeros = {5, 3, 9, 1, 7};

Arrays.toString(numeros);              // "[5, 3, 9, 1, 7]"
Arrays.sort(numeros);                  // ordena el propio array: [1, 3, 5, 7, 9]
Arrays.binarySearch(numeros, 7);       // 3 (¡solo con arrays ordenados!)

int[] copia = Arrays.copyOf(numeros, 3);           // [1, 3, 5]
int[] trozo = Arrays.copyOfRange(numeros, 1, 4);   // [3, 5, 7] (el 4 no se incluye)

int[] ceros = new int[4];
Arrays.fill(ceros, -1);                // [-1, -1, -1, -1]

Arrays.equals(numeros, copia);         // false: compara contenido
Arrays.deepToString(new int[][] {{1, 2}, {3, 4}});  // "[[1, 2], [3, 4]]"

List<String> lista = Arrays.asList("a", "b", "c");  // vista de tamaño fijo como lista
```

> ⚠️ `IO.println(numeros)` **no muestra el contenido**, sino algo como `[I@1b6d3586` (tipo + código interno). Usa siempre `Arrays.toString(numeros)`.

Ordenar `String` y otros objetos también funciona; para orden descendente o criterios propios se pasa un `Comparator`:

```java
String[] nombres = {"Pedro", "ana", "Luis"};
Arrays.sort(nombres);                                // [Luis, Pedro, ana] (mayúsculas antes)
Arrays.sort(nombres, String.CASE_INSENSITIVE_ORDER); // [ana, Luis, Pedro]

Integer[] valores = {5, 3, 9};
Arrays.sort(valores, Comparator.reverseOrder());     // [9, 5, 3] (requiere Integer, no int)
```

Más sobre ordenación en la guía [Comparable vs Comparator](<Comparable vs Comparator.md>).

---

## 21. Los arrays son referencias

Un array es un **objeto**. La variable guarda una referencia, así que **asignar un array a otra variable no lo copia**: las dos apuntan al mismo array.

```java
int[] a = {1, 2, 3};
int[] b = a;          // b apunta al MISMO array
b[0] = 99;
IO.println(a[0]);     // 99 😮

int[] c = a.clone();  // copia independiente (también Arrays.copyOf)
c[1] = 50;
IO.println(a[1]);     // 2: a no cambia
```

```text
 a ──┐
     ├──► [99, 2, 3]         c ──► [99, 50, 3]
 b ──┘
```

Y por la misma razón:

```java
int[] x = {1, 2, 3};
int[] y = {1, 2, 3};
IO.println(x == y);                 // false: son dos objetos distintos
IO.println(Arrays.equals(x, y));    // true: mismo contenido
```

> ⚠️ `clone()` hace una copia **superficial**: en un array de arrays (o de objetos), copia las referencias, no los objetos internos. Para copiar una matriz hay que clonar cada fila.

---

## 22. Definir y llamar a un método

Un **método** es un bloque de código con nombre que realiza una tarea concreta. Sirve para **reutilizar** código, **dividir** un problema en partes pequeñas y dar **nombres** que expliquen qué hace cada parte.

### 22.1. Estructura

```text
 modificadores  tipoRetorno  nombre  (parámetros)
      │              │          │          │
   static          double    calcularIva (double base)  {
                                              return base * 0.21;     ← cuerpo
                                          }
```

| Parte | Descripción |
|---|---|
| **Modificadores** | `public`, `private`, `static`... (opcionales) |
| **Tipo de retorno** | Tipo del valor que devuelve, o `void` si no devuelve nada |
| **Nombre** | En `camelCase`, normalmente un verbo |
| **Parámetros** | Lista de `tipo nombre` separados por comas (puede estar vacía) |
| **Cuerpo** | Las sentencias entre llaves |

La combinación de **nombre + tipos de los parámetros** es la **firma** del método.

### 22.2. Ejemplos

```java
// Sin parámetros y sin retorno
void saludar() {
    IO.println("¡Hola!");
}

// Con parámetros y sin retorno
void saludar(String nombre) {
    IO.println("¡Hola, " + nombre + "!");
}

// Con parámetros y con retorno
int sumar(int a, int b) {
    return a + b;
}

// Devuelve un boolean: nombre en forma de pregunta
boolean esPar(int n) {
    return n % 2 == 0;
}

void main() {
    saludar();
    saludar("Carlos");
    int resultado = sumar(3, 4);       // 7
    IO.println(esPar(resultado));      // false
    IO.println(sumar(1, sumar(2, 3))); // 6: el resultado de un método como argumento
}
```

- **Parámetros**: las variables de la declaración (`a`, `b`).
- **Argumentos**: los valores concretos que se pasan al llamar (`3`, `4`).

### 22.3. return

`return` **termina el método** y devuelve un valor:

- Un método con tipo de retorno distinto de `void` **debe** devolver un valor en todos los caminos posibles.
- En un método `void` se puede usar `return;` (sin valor) para salir antes.

```java
String clasificar(int edad) {
    if (edad < 0) {
        return "edad no válida";
    }
    if (edad < 18) {
        return "menor";
    }
    return "adulto";       // sin esta línea no compilaría: "missing return statement"
}
```

### 22.4. Métodos static y de instancia

En la versión clásica de un programa, los métodos que se llaman desde `main` sin crear objetos deben ser `static`:

```java
public class Calculadora {

    static int doble(int n) {          // pertenece a la clase
        return n * 2;
    }

    public static void main(String[] args) {
        System.out.println(doble(5));      // 10
        System.out.println(Math.max(3, 9));// los métodos de Math también son static
    }
}
```

Un método **de instancia** (sin `static`) pertenece a cada objeto y puede usar sus atributos. Se llama sobre un objeto: `texto.length()`, `cuenta.ingresar(100)`. Esto se desarrolla al estudiar clases y objetos.

> 💡 En un fichero compacto de Java 25 con `void main()`, los métodos que escribes junto a `main` son métodos de instancia de una clase implícita, por eso no necesitan `static`.

---

## 23. Parámetros y paso por valor

En Java, **los argumentos siempre se pasan por valor**: el método recibe una **copia** del valor de la variable.

### 23.1. Con primitivos

El método trabaja con una copia; la variable original no cambia:

```java
void incrementar(int n) {
    n++;
    IO.println("Dentro: " + n);    // 6
}

void main() {
    int x = 5;
    incrementar(x);
    IO.println("Fuera: " + x);     // 5
}
```

### 23.2. Con referencias

Lo que se copia es **la referencia**, no el objeto. Por eso el método **puede modificar el contenido** del objeto, pero **no puede hacer que la variable original apunte a otro objeto**:

```java
void modificarContenido(int[] array) {
    array[0] = 100;               // ✅ cambia el array original
}

void reasignar(int[] array) {
    array = new int[] {7, 7, 7};  // solo cambia la copia local de la referencia
}

void main() {
    int[] datos = {1, 2, 3};
    modificarContenido(datos);
    IO.println(Arrays.toString(datos));   // [100, 2, 3]
    reasignar(datos);
    IO.println(Arrays.toString(datos));   // [100, 2, 3]: no ha cambiado
}
```

Con `String` (inmutable), ningún método puede cambiar la cadena original. Si un método debe «cambiar» un valor, lo habitual es que **lo devuelva**:

```java
String gritar(String texto) {
    return texto.toUpperCase() + "!";
}
```

---

## 24. Sobrecarga de métodos

Varios métodos pueden tener **el mismo nombre** si sus **parámetros son distintos** (en número, tipo u orden). Java elige cuál ejecutar según los argumentos:

```java
int sumar(int a, int b)            { return a + b; }
int sumar(int a, int b, int c)     { return a + b + c; }
double sumar(double a, double b)   { return a + b; }

void main() {
    IO.println(sumar(1, 2));        // 3   → versión (int, int)
    IO.println(sumar(1, 2, 3));     // 6   → versión (int, int, int)
    IO.println(sumar(1.5, 2.5));    // 4.0 → versión (double, double)
}
```

El tipo de retorno **no cuenta** para la sobrecarga: dos métodos que solo se diferencian en lo que devuelven no compilan.

`IO.println` es un ejemplo real: acepta cualquier tipo porque tiene varias versiones.

---

## 25. Argumentos variables (varargs)

Con `tipo... nombre` un método acepta **cero o más argumentos** de ese tipo. Dentro del método, el parámetro es un **array**:

```java
int sumarTodos(int... numeros) {
    int total = 0;
    for (int n : numeros) total += n;
    return total;
}

void main() {
    IO.println(sumarTodos());              // 0
    IO.println(sumarTodos(5));             // 5
    IO.println(sumarTodos(1, 2, 3, 4));    // 10
    IO.println(sumarTodos(new int[] {7, 8}));  // 15: también admite un array
}
```

Reglas:

- Solo puede haber **un** parámetro varargs y debe ser **el último**: `void registrar(String nivel, String... mensajes)`.
- Muchos métodos de la biblioteca lo usan: `String.format`, `List.of`, `Arrays.asList`...

---

## 26. Recursividad

Un método **recursivo** se llama a sí mismo. Necesita siempre:

1. Un **caso base** que termina sin volver a llamarse.
2. Un **caso recursivo** que se acerca al caso base.

```java
// factorial(5) = 5 × 4 × 3 × 2 × 1 = 120
long factorial(int n) {
    if (n <= 1) {
        return 1;                    // caso base
    }
    return n * factorial(n - 1);     // caso recursivo
}
```

```text
factorial(4)
= 4 * factorial(3)
= 4 * 3 * factorial(2)
= 4 * 3 * 2 * factorial(1)
= 4 * 3 * 2 * 1
= 24
```

Si falta el caso base o nunca se alcanza, cada llamada ocupa memoria en la **pila** hasta que se agota: `StackOverflowError`.

La recursividad es elegante para problemas que se dividen en subproblemas iguales (árboles, carpetas dentro de carpetas, algoritmos como *quicksort*). Para recorridos sencillos, un bucle suele ser más eficiente y claro.

---

## 27. Ámbito de las variables

El **ámbito** (*scope*) es la zona del código donde una variable existe. Una variable local existe **desde su declaración hasta el final del bloque `{ }`** donde se declaró:

```java
void ejemplo() {
    int a = 1;                    // existe en todo el método

    if (a > 0) {
        int b = 2;                // solo existe dentro del if
        IO.println(a + b);
    }
    // IO.println(b);             // ❌ error: b no existe aquí

    for (int i = 0; i < 3; i++) { // i solo existe dentro del for
        IO.println(i);
    }
    // IO.println(i);             // ❌ error: i no existe aquí
}
```

- Los **parámetros** existen en todo el cuerpo del método.
- **Dos métodos distintos** pueden tener variables con el mismo nombre: son independientes.
- No se puede declarar una variable local con el mismo nombre que otra que siga en ámbito:

```java
int x = 1;
if (true) {
    int x = 2;     // ❌ error: variable x is already defined
}
```

Buena práctica: declara cada variable **en el ámbito más pequeño posible** y **lo más cerca de donde se usa**.

---

## 28. Buenas prácticas al escribir métodos

- **Una sola responsabilidad.** Un método debe hacer una cosa. Si para describirlo necesitas «y», probablemente son dos métodos.
- **Cortos.** Si no cabe en la pantalla, divídelo.
- **Nombres que expliquen qué hace**: `calcularPrecioConDescuento` mejor que `calc` o `procesar`.
- **Pocos parámetros** (idealmente tres o menos). Si necesitas muchos, agrúpalos en un `record`.
- **Evita parámetros booleanos** que cambian el comportamiento: `imprimir(true)` no dice nada. Mejor dos métodos con nombres claros.
- **Devuelve resultados en lugar de modificar parámetros**: es más fácil de entender y de probar.
- **Valida las entradas** al principio (cláusulas de guarda) y lanza una excepción si no son válidas:

```java
double dividir(double a, double b) {
    if (b == 0) {
        throw new IllegalArgumentException("El divisor no puede ser 0");
    }
    return a / b;
}
```

- **Documenta** los métodos públicos con un comentario de documentación:

```java
/// Calcula el precio final aplicando un descuento.
///
/// @param precio    precio original, mayor o igual que 0
/// @param descuento porcentaje entre 0 y 100
/// @return el precio con el descuento aplicado
double aplicarDescuento(double precio, int descuento) {
    return precio * (100 - descuento) / 100;
}
```

Las guías de [Principios DRY](<../07-Diseno/Principios/Principio DRY.md>), [KISS](<../07-Diseno/Principios/Principio KISS.md>) y [Calistenia de objetos](<../07-Diseno/Principios/Calistenia de objetos.md>) profundizan en estas ideas.

---

## 29. Ejemplo completo: estadísticas de notas

Este programa reúne todo lo visto: variables, operadores, condicionales, `switch`, bucles, arrays y métodos.

```java
// EstadisticasNotas.java
// Ejecutar con: java EstadisticasNotas.java

final double NOTA_APROBADO = 5.0;

void main() {
    String[] alumnos = {"Ana", "Bruno", "Carla", "David", "Elena"};
    double[] notas = {7.5, 4.2, 9.1, 5.0, 3.8};

    IO.println("=== Notas del grupo ===");
    for (int i = 0; i < alumnos.length; i++) {
        IO.println("%-6s %5.1f  %s".formatted(alumnos[i], notas[i], calificar(notas[i])));
    }

    IO.println();
    IO.println("Media:      %.2f".formatted(media(notas)));
    IO.println("Nota máxima: %.1f (%s)".formatted(notas[posicionMaxima(notas)], alumnos[posicionMaxima(notas)]));
    IO.println("Aprobados:  %d de %d".formatted(contarAprobados(notas), notas.length));

    double[] ordenadas = notas.clone();
    Arrays.sort(ordenadas);
    IO.println("Ordenadas:  " + Arrays.toString(ordenadas));
}

String calificar(double nota) {
    if (nota < 0 || nota > 10) {
        throw new IllegalArgumentException("Nota fuera de rango: " + nota);
    }
    int parteEntera = (int) nota;
    return switch (parteEntera) {
        case 10, 9 -> "Sobresaliente";
        case 8, 7 -> "Notable";
        case 6 -> "Bien";
        case 5 -> "Suficiente";
        default -> "Insuficiente";
    };
}

double media(double[] valores) {
    double suma = 0;
    for (double v : valores) {
        suma += v;
    }
    return suma / valores.length;
}

int posicionMaxima(double[] valores) {
    int posicion = 0;
    for (int i = 1; i < valores.length; i++) {
        if (valores[i] > valores[posicion]) {
            posicion = i;
        }
    }
    return posicion;
}

int contarAprobados(double[] valores) {
    int aprobados = 0;
    for (double v : valores) {
        if (v >= NOTA_APROBADO) {
            aprobados++;
        }
    }
    return aprobados;
}
```

Salida (con configuración regional española, que usa coma decimal):

```text
=== Notas del grupo ===
Ana      7,5  Notable
Bruno    4,2  Insuficiente
Carla    9,1  Sobresaliente
David    5,0  Suficiente
Elena    3,8  Insuficiente

Media:      5,92
Nota máxima: 9,1 (Carla)
Aprobados:  3 de 5
Ordenadas:  [3.8, 4.2, 5.0, 7.5, 9.1]
```

Observa que `Arrays.toString` siempre usa punto decimal, mientras que `formatted` respeta la configuración regional.

**Ideas para ampliarlo:** leer las notas con `IO.readln`, mostrar la nota mínima, calcular la mediana o guardar los datos en un `record Alumno(String nombre, double nota)` (ver la guía de [Records](<Records.md>)).

---

## 30. Errores frecuentes

| Error | Síntoma | Solución |
|---|---|---|
| Comparar `String` con `==` | Comparaciones que fallan «sin motivo» | Usa `equals` |
| División entera inesperada | `7 / 2` da `3`; una media sale sin decimales | Convierte un operando: `(double) a / b` |
| Índice fuera de rango | `ArrayIndexOutOfBoundsException` | Los índices van de `0` a `length - 1`; usa `i < array.length`, no `<=` |
| Olvidar `break` en `switch` clásico | Se ejecutan varios casos | Usa `switch` con `->` |
| `;` detrás de `if`, `for` o `while` | `if (x > 0);` el bloque se ejecuta siempre | Quita el punto y coma |
| Usar `=` en lugar de `==` | `if (x = 5)` no compila con `int`; con `boolean` asigna en vez de comparar | `==` para comparar |
| Variable local sin inicializar | `variable might not have been initialized` | Dale un valor antes de usarla |
| Imprimir un array directamente | Sale `[I@1b6d3586` | `Arrays.toString(array)` |
| `double` para dinero | Resultados como `0.30000000000000004` | `BigDecimal` o céntimos en `long` |
| Desbordamiento de `int` | Números negativos inesperados al multiplicar | Usa `long` o `Math.multiplyExact` |
| `nextInt()` seguido de `nextLine()` | La línea leída sale vacía | Consume el salto de línea con un `nextLine()` extra |
| Recursividad sin caso base | `StackOverflowError` | Comprueba que el caso base siempre se alcanza |
| Modificar `String` sin guardar el resultado | `s.trim();` no cambia `s` | `s = s.trim();` |
| `length` / `length()` / `size()` | Error de compilación | Array: `length`; `String`: `length()`; colección: `size()` |

---

## 31. Chuleta rápida

```java
// ── Programa mínimo (Java 25) ────────────────────────────
void main() {
    IO.println("Hola");
    String linea = IO.readln("Escribe algo: ");
}

// ── Variables y tipos ────────────────────────────────────
int entero = 42;            long grande = 5_000_000_000L;
double decimal = 3.14;      float f = 3.14f;
char letra = 'A';           boolean ok = true;
String texto = "Hola";      var inferido = 10;     // int
final int CONSTANTE = 100;

// ── Conversiones ─────────────────────────────────────────
int n = Integer.parseInt("123");
String s = String.valueOf(123);
int truncado = (int) 9.99;               // 9
double real = (double) 7 / 2;            // 3.5

// ── Operadores ───────────────────────────────────────────
// + - * / %     == != < > <= >=     && || !     ?:
// += -= *= /= %=     ++ --     instanceof

// ── Condicionales ────────────────────────────────────────
if (x > 0) { ... } else if (x < 0) { ... } else { ... }

String r = switch (dia) {
    case 6, 7 -> "fin de semana";
    default -> "laborable";
};

// ── Bucles ───────────────────────────────────────────────
for (int i = 0; i < 10; i++) { ... }
for (String e : array) { ... }
while (condicion) { ... }
do { ... } while (condicion);
// break: sale del bucle · continue: siguiente vuelta

// ── Arrays ───────────────────────────────────────────────
int[] a = new int[5];
int[] b = {3, 1, 2};
int[][] m = new int[3][4];
b.length;  b[0];  b[b.length - 1];
Arrays.toString(b);  Arrays.sort(b);  Arrays.copyOf(b, 10);
Arrays.equals(a, b); Arrays.fill(a, 0);

// ── Métodos ──────────────────────────────────────────────
int sumar(int a, int b) { return a + b; }
void mostrar(String msg) { IO.println(msg); }
int total(int... nums) { ... }           // varargs
```

---

## 32. Ejercicios propuestos

1. **Conversor de temperatura.** Pide una temperatura en grados Celsius con `IO.readln` y muestra su equivalente en Fahrenheit (`F = C × 9 / 5 + 32`) con un decimal. ¿Qué pasa si escribes `C * 9 / 5` con `C` de tipo `int`?
2. **Par o impar y signo.** Dado un número entero, indica si es par o impar y si es positivo, negativo o cero.
3. **Año bisiesto.** Escribe `boolean esBisiesto(int anio)`. Un año es bisiesto si es divisible entre 4, salvo que sea divisible entre 100 y no entre 400. Comprueba 1900, 2000, 2024 y 2025.
4. **Día de la semana.** Con un `switch` como expresión, convierte un número del 1 al 7 en el nombre del día. Para cualquier otro valor, lanza `IllegalArgumentException`.
5. **Tabla de multiplicar.** Muestra la tabla de multiplicar de un número del 1 al 10 con formato `7 x 3 = 21`.
6. **FizzBuzz.** Muestra los números del 1 al 30, pero escribe «Fizz» para los múltiplos de 3, «Buzz» para los de 5 y «FizzBuzz» para los de ambos.
7. **Invertir un array.** Escribe `int[] invertir(int[] original)` que devuelva un **nuevo** array con los elementos al revés, sin modificar el original.
8. **Palíndromo.** Escribe `boolean esPalindromo(String texto)` que ignore mayúsculas y espacios. «Dábale arroz a la zorra el abad» no lo será por las tildes; prueba con «Anita lava la tina».
9. **Suma de una matriz.** Escribe `int sumarDiagonal(int[][] matriz)` que sume la diagonal principal de una matriz cuadrada.
10. **Recursividad.** Escribe una versión recursiva de `int sumarDigitos(int n)`: `sumarDigitos(1234)` debe devolver `10`.

---

## 33. Soluciones

Hay muchas soluciones correctas; estas son una propuesta.

<details>
<summary>1. Conversor de temperatura</summary>

```java
void main() {
    double celsius = Double.parseDouble(IO.readln("Grados Celsius: "));
    double fahrenheit = celsius * 9 / 5 + 32;
    IO.println("%.1f °C = %.1f °F".formatted(celsius, fahrenheit));
}
```

Si `celsius` fuera `int`, `celsius * 9 / 5` haría división entera: con 37 daría `66` en lugar de `66.6`, y el resultado sería `98` en vez de `98.6`.

</details>

<details>
<summary>2. Par o impar y signo</summary>

```java
String describir(int n) {
    String paridad = n % 2 == 0 ? "par" : "impar";
    String signo;
    if (n > 0) {
        signo = "positivo";
    } else if (n < 0) {
        signo = "negativo";
    } else {
        signo = "cero";
    }
    return n + " es " + paridad + " y " + signo;
}
```

Fíjate en que `-3 % 2` es `-1`, no `1`. Por eso se compara con `== 0` y no con `== 1`.

</details>

<details>
<summary>3. Año bisiesto</summary>

```java
boolean esBisiesto(int anio) {
    return anio % 4 == 0 && (anio % 100 != 0 || anio % 400 == 0);
}

void main() {
    for (int anio : new int[] {1900, 2000, 2024, 2025}) {
        IO.println(anio + ": " + esBisiesto(anio));
    }
}
// 1900: false · 2000: true · 2024: true · 2025: false
```

La biblioteca estándar ya lo resuelve: `java.time.Year.isLeap(anio)`.

</details>

<details>
<summary>4. Día de la semana</summary>

```java
String nombreDia(int dia) {
    return switch (dia) {
        case 1 -> "lunes";
        case 2 -> "martes";
        case 3 -> "miércoles";
        case 4 -> "jueves";
        case 5 -> "viernes";
        case 6 -> "sábado";
        case 7 -> "domingo";
        default -> throw new IllegalArgumentException("Día no válido: " + dia);
    };
}
```

</details>

<details>
<summary>5. Tabla de multiplicar</summary>

```java
void tabla(int n) {
    for (int i = 1; i <= 10; i++) {
        IO.println("%d x %2d = %3d".formatted(n, i, n * i));
    }
}
```

</details>

<details>
<summary>6. FizzBuzz</summary>

```java
void main() {
    for (int i = 1; i <= 30; i++) {
        String salida;
        if (i % 15 == 0) {
            salida = "FizzBuzz";      // primero el caso más específico
        } else if (i % 3 == 0) {
            salida = "Fizz";
        } else if (i % 5 == 0) {
            salida = "Buzz";
        } else {
            salida = String.valueOf(i);
        }
        IO.println(salida);
    }
}
```

</details>

<details>
<summary>7. Invertir un array</summary>

```java
int[] invertir(int[] original) {
    int[] resultado = new int[original.length];
    for (int i = 0; i < original.length; i++) {
        resultado[i] = original[original.length - 1 - i];
    }
    return resultado;
}

void main() {
    int[] datos = {1, 2, 3, 4};
    IO.println(Arrays.toString(invertir(datos)));   // [4, 3, 2, 1]
    IO.println(Arrays.toString(datos));             // [1, 2, 3, 4]
}
```

</details>

<details>
<summary>8. Palíndromo</summary>

```java
boolean esPalindromo(String texto) {
    String limpio = texto.replace(" ", "").toLowerCase();
    int izquierda = 0;
    int derecha = limpio.length() - 1;
    while (izquierda < derecha) {
        if (limpio.charAt(izquierda) != limpio.charAt(derecha)) {
            return false;
        }
        izquierda++;
        derecha--;
    }
    return true;
}
// esPalindromo("Anita lava la tina") → true
```

Alternativa en una línea: `new StringBuilder(limpio).reverse().toString().equals(limpio)`.

</details>

<details>
<summary>9. Suma de la diagonal</summary>

```java
int sumarDiagonal(int[][] matriz) {
    int suma = 0;
    for (int i = 0; i < matriz.length; i++) {
        suma += matriz[i][i];
    }
    return suma;
}
// sumarDiagonal(new int[][] {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}}) → 15
```

</details>

<details>
<summary>10. Suma de dígitos recursiva</summary>

```java
int sumarDigitos(int n) {
    if (n < 10) {
        return n;                          // caso base: un solo dígito
    }
    return n % 10 + sumarDigitos(n / 10);  // último dígito + el resto
}
// sumarDigitos(1234) → 4 + sumarDigitos(123) → 4 + 3 + 2 + 1 = 10
```

Para números negativos, llama a `sumarDigitos(Math.abs(n))`.

</details>

---

## 34. Glosario

| Término | Definición |
|---|---|
| **Argumento** | Valor concreto que se pasa a un método al llamarlo |
| **Array** | Estructura de tamaño fijo con elementos del mismo tipo accesibles por índice |
| **Autoboxing** | Conversión automática de un primitivo a su clase envoltorio (`int` → `Integer`) |
| **Bloque** | Conjunto de sentencias entre llaves `{ }` |
| **Bytecode** | Código intermedio (`.class`) que genera `javac` y ejecuta la JVM |
| **Casting** | Conversión explícita de un tipo a otro: `(int) 3.7` |
| **Cortocircuito** | En `&&` y `\|\|`, no evaluar el segundo operando si el primero ya decide el resultado |
| **Desbordamiento** | Resultado que no cabe en su tipo y «da la vuelta» (*overflow*) |
| **Ámbito** | Zona del código en la que una variable existe (*scope*) |
| **Firma** | Nombre de un método más los tipos de sus parámetros |
| **Inmutable** | Objeto que no puede cambiar tras crearse (por ejemplo, `String`) |
| **JDK / JVM** | Kit de desarrollo / máquina virtual que ejecuta el bytecode |
| **Literal** | Valor escrito directamente en el código: `42`, `"hola"`, `'A'`, `true` |
| **LTS** | Versión con soporte a largo plazo (17, 21, 25) |
| **Parámetro** | Variable declarada en la definición de un método |
| **Paso por valor** | El método recibe una copia del valor (o de la referencia) del argumento |
| **Primitivo** | Uno de los 8 tipos básicos que guardan directamente su valor |
| **Recursividad** | Técnica en la que un método se llama a sí mismo |
| **Referencia** | Valor que indica dónde está un objeto en memoria |
| **Sobrecarga** | Varios métodos con el mismo nombre y distintos parámetros |
| **Varargs** | Parámetro que acepta un número variable de argumentos (`int... nums`) |
| **Wrapper** | Clase envoltorio de un primitivo (`Integer`, `Double`...) |

---

## 35. Referencias

- Oracle, *The Java Tutorials*: [Language Basics](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html) (escrito para JDK 8, pero sigue siendo válido para los conceptos básicos).
- [Dev.java – Learn Java](https://dev.java/learn/): tutoriales oficiales actualizados a las últimas versiones.
- [Especificación del lenguaje Java (JLS), Java SE 25](https://docs.oracle.com/javase/specs/jls/se25/html/index.html).
- [Documentación de la API de Java SE 25](https://docs.oracle.com/en/java/javase/25/docs/api/index.html): `String`, `Math`, `Arrays`, `IO`...
- [JEP 512: Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512) (Java 25).
- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361) (Java 14) y [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) (Java 21).
- [JEP 456: Unnamed Variables & Patterns](https://openjdk.org/jeps/456) (Java 22).
- [JEP 467: Markdown Documentation Comments](https://openjdk.org/jeps/467) (Java 23).
- [JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview)](https://openjdk.org/jeps/507) (Java 25).
- W3Schools: [Java Tutorial](https://www.w3schools.com/java/) para ejercicios rápidos de repaso.
