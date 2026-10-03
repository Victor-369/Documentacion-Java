# Entrada/Salida (E/S) en Java 21+

> **Fuente de referencia:** lección *Basic I/O* de los [Java Tutorials de Oracle](https://docs.oracle.com/javase/tutorial/essential/io/index.html).
> Este documento es una reelaboración y ampliación en castellano, con explicaciones propias, ejemplos nuevos, buenas prácticas y ejercicios pensados para quien está empezando.
>
> **Versión objetivo:** este manual está pensado para **Java 21 o superior** (21 y 25 son versiones LTS). El tutorial original de Oracle está escrito para JDK 8; aquí los ejemplos usan la API moderna (`Path.of`, `Files.readString`, `var`, `record`, bloques de texto…). Cuando una característica exige una versión posterior a la 21, se indica expresamente.

---

## Índice

1. [Introducción](#1-introducción)
2. [Flujos de E/S (streams)](#2-flujos-de-es-streams)
3. [Flujos de bytes](#3-flujos-de-bytes)
4. [Flujos de caracteres](#4-flujos-de-caracteres)
5. [Flujos con búfer (buffered)](#5-flujos-con-búfer-buffered)
6. [Lectura y formateo de texto: `Scanner` y `printf`](#6-lectura-y-formateo-de-texto-scanner-y-printf)
7. [E/S desde la línea de comandos](#7-es-desde-la-línea-de-comandos)
8. [Flujos de datos (Data Streams)](#8-flujos-de-datos-data-streams)
9. [Flujos de objetos y serialización](#9-flujos-de-objetos-y-serialización)
10. [E/S de ficheros con NIO.2](#10-es-de-ficheros-con-nio2)
    - [10.1 ¿Qué es una ruta (`Path`)?](#101-qué-es-una-ruta-path)
    - [10.2 La clase `Path`](#102-la-clase-path)
    - [10.3 Operaciones con rutas](#103-operaciones-con-rutas)
    - [10.4 Operaciones con ficheros: ideas comunes](#104-operaciones-con-ficheros-ideas-comunes)
    - [10.5 Comprobar un fichero o directorio](#105-comprobar-un-fichero-o-directorio)
    - [10.6 Borrar](#106-borrar-un-fichero-o-directorio)
    - [10.7 Copiar](#107-copiar-un-fichero-o-directorio)
    - [10.8 Mover](#108-mover-un-fichero-o-directorio)
    - [10.9 Metadatos y atributos](#109-metadatos-y-atributos)
    - [10.10 Leer, escribir y crear ficheros](#1010-leer-escribir-y-crear-ficheros)
    - [10.11 Ficheros de acceso aleatorio](#1011-ficheros-de-acceso-aleatorio)
    - [10.12 Crear y leer directorios](#1012-crear-y-leer-directorios)
    - [10.13 Enlaces simbólicos y duros](#1013-enlaces-simbólicos-y-duros)
    - [10.14 Recorrer un árbol de ficheros](#1014-recorrer-un-árbol-de-ficheros)
    - [10.15 Buscar ficheros](#1015-buscar-ficheros)
    - [10.16 Vigilar cambios en un directorio](#1016-vigilar-cambios-en-un-directorio)
    - [10.17 Otros métodos útiles](#1017-otros-métodos-útiles)
    - [10.18 Código heredado con `java.io.File`](#1018-código-heredado-con-javaiofile)
    - [10.19 E/S y concurrencia: hilos virtuales (Java 21)](#1019-es-y-concurrencia-hilos-virtuales-java-21)
11. [Resumen](#11-resumen)
12. [Errores frecuentes y buenas prácticas](#12-errores-frecuentes-y-buenas-prácticas)
13. [Preguntas y ejercicios](#13-preguntas-y-ejercicios)
14. [Para seguir aprendiendo](#14-para-seguir-aprendiendo)

---

## 1. Introducción

La **entrada/salida (E/S, o I/O en inglés)** es todo aquello que permite a un programa comunicarse con el exterior: leer del teclado, escribir en pantalla, guardar datos en un fichero, recibir información por la red, etc.

Java organiza la E/S en dos grandes bloques, que viven en paquetes distintos:

| Bloque | Paquete principal | Para qué sirve |
|---|---|---|
| **Flujos de E/S (I/O Streams)** | `java.io` | Leer y escribir datos de forma secuencial: bytes, caracteres, tipos primitivos y objetos. |
| **E/S de ficheros (File I/O, NIO.2)** | `java.nio.file` | Manejar el sistema de ficheros: rutas, copiar, mover, borrar, atributos, recorrer directorios… |

### Qué vas a aprender

- Qué es un *stream* y qué tipos existen.
- Cómo leer y escribir bytes y texto de forma eficiente.
- Cómo interpretar y dar formato a texto (`Scanner`, `printf`).
- Cómo interactuar con la consola.
- Cómo guardar tipos primitivos y objetos en binario.
- Cómo trabajar con el sistema de ficheros usando la API moderna (`Path` y `Files`).

### Requisitos previos

- Sintaxis básica de Java (clases, métodos, bucles).
- Conocer las **excepciones**, porque casi toda la E/S puede lanzar `IOException`.
- Conocer `try-catch` y, a ser posible, `try-with-resources`.

> 💡 **Consejo:** si algo no te funciona, lo primero es mirar la excepción. En E/S los errores típicos son rutas mal escritas (`FileNotFoundException`, `NoSuchFileException`) y permisos insuficientes (`AccessDeniedException`).

### Novedades desde Java 8 que se usan en este manual

Si has visto tutoriales antiguos, notarás diferencias. Estas son las más relevantes para la E/S:

| Característica | Disponible desde | Dónde se usa |
|---|---|---|
| `InputStream.readAllBytes()`, `readNBytes()`, `transferTo()`; `try-with-resources` con variables ya declaradas | Java 9 | Secciones 2, 3 y 10 |
| `var` (inferencia de tipo en variables locales) | Java 10 | Ejemplos puntuales |
| `Reader.transferTo(Writer)` | Java 10 | Sección 4 |
| `Path.of`, `Files.readString`, `Files.writeString`, `FileReader`/`FileWriter` con `Charset` | Java 11 | Secciones 4 y 10 |
| `Files.mismatch` | Java 12 | Sección 10.17 |
| Bloques de texto (`"""`) y `String.formatted()` | Java 15 | Sección 6 |
| `record` y `Stream.toList()` | Java 16 | Secciones 9 y 10 |
| UTF-8 como codificación por defecto | Java 18 | Sección 4 |
| Hilos virtuales (versión definitiva) | Java 21 | Sección 10.19 |
| `System.console()` no nulo sin terminal y `Console.isTerminal()` | Java 22 | Sección 7 |
| Clase `java.io.IO` (`IO.println`, `IO.readln`) | Java 25 | Sección 7 |

> ℹ️ **Sobre el tutorial original:** menciona el `SecurityManager` y los *applets*. Ambos están en desuso: el `SecurityManager` está marcado para eliminación desde Java 17 y desactivado de forma permanente desde Java 24, y la API de *applets* también está marcada para eliminación. Puedes ignorar esas consideraciones.

---

## 2. Flujos de E/S (streams)

### 2.1 Concepto

Un **flujo (stream)** es una abstracción que representa una **secuencia ordenada de datos** que fluye desde un origen hasta un destino. Piensa en una tubería: tú no ves de dónde viene el agua ni a dónde va, solo sabes que puedes **sacar** de ella o **meter** en ella.

- **Flujo de entrada (input stream):** el programa **lee** datos de una fuente (fichero, teclado, red, memoria…).
- **Flujo de salida (output stream):** el programa **escribe** datos en un destino (fichero, pantalla, red, memoria…).

```
 FUENTE  ──►  [ InputStream ]  ──►  PROGRAMA  ──►  [ OutputStream ]  ──►  DESTINO
(fichero)                                                                 (pantalla)
```

La gran ventaja es que el código es muy parecido sin importar el origen o el destino: lo que cambia es la clase concreta que instancias.

### 2.2 Tipos de datos que puede transportar un flujo

Un flujo puede manejar:

- **Bytes** (datos binarios en bruto).
- **Caracteres** (texto, con su codificación).
- **Tipos primitivos** y cadenas en binario.
- **Objetos** (serialización).

### 2.3 Jerarquías principales

| | Entrada | Salida |
|---|---|---|
| **Bytes** | `InputStream` | `OutputStream` |
| **Caracteres** | `Reader` | `Writer` |

Todas estas son **clases abstractas**. De ellas derivan las clases concretas (`FileInputStream`, `FileReader`, `BufferedWriter`, etc.).

### 2.4 Cerrar los flujos: obligatorio

Un flujo mantiene recursos del sistema operativo. Si no lo cierras, puedes provocar **fugas de recursos** (ficheros bloqueados, datos sin volcar al disco…).

La forma correcta desde Java 7 es **`try-with-resources`**, que cierra automáticamente cualquier objeto que implemente `AutoCloseable`:

```java
try (FileInputStream in = new FileInputStream("datos.bin")) {
    // usar 'in'
} catch (IOException e) {
    e.printStackTrace();
}
// 'in' ya está cerrado aquí, haya o no haya habido excepción
```

> ⚠️ Evita el patrón antiguo `try { ... } finally { if (in != null) in.close(); }`: es más largo y propenso a errores.

Desde Java 9 también puedes usar en el `try` una variable ya declarada, siempre que sea `final` o *efectivamente final* (no se reasigna):

```java
FileInputStream in = new FileInputStream("datos.bin");
try (in) {
    // usar 'in'
}   // se cierra aquí
```

### 2.5 Los flujos estándar

Java ofrece tres flujos ya abiertos:

| Flujo | Tipo | Uso |
|---|---|---|
| `System.in` | `InputStream` | Entrada estándar (normalmente, teclado). |
| `System.out` | `PrintStream` | Salida estándar (normalmente, pantalla). |
| `System.err` | `PrintStream` | Salida de errores (también pantalla, pero separada). |

No los cierres: son compartidos por toda la aplicación.

---

## 3. Flujos de bytes

Los **flujos de bytes** (`InputStream` / `OutputStream`) manejan datos binarios "en bruto": imágenes, audio, ficheros comprimidos, ejecutables… También sirven para texto, pero no es lo recomendable (para eso están los de caracteres).

### 3.1 Métodos fundamentales

**`InputStream`**

| Método | Descripción |
|---|---|
| `int read()` | Lee **un byte** (valor 0–255) o devuelve **-1** si se alcanzó el final. |
| `int read(byte[] b)` | Lee hasta `b.length` bytes en el array; devuelve cuántos leyó o -1 al final. |
| `byte[] readAllBytes()` | Lee todo lo que quede. |
| `byte[] readNBytes(int n)` | Lee hasta `n` bytes (o menos si se acaba el flujo). |
| `long transferTo(OutputStream out)` | Vuelca todo lo que quede en otro flujo de salida. |
| `void close()` | Cierra el flujo. |

**`OutputStream`**

| Método | Descripción |
|---|---|
| `void write(int b)` | Escribe un byte (los 8 bits menos significativos). |
| `void write(byte[] b)` | Escribe un array completo. |
| `void flush()` | Fuerza el volcado de datos pendientes. |
| `void close()` | Cierra (y vuelca) el flujo. |

> 🤔 **¿Por qué `read()` devuelve `int` y no `byte`?** Porque necesita poder devolver **-1** para indicar el final, y `byte` en Java tiene signo (-128 a 127), con lo que un byte legítimo `0xFF` se confundiría con -1. Devolviendo un `int` entre 0 y 255 se evita el problema.

### 3.2 Ejemplo: copiar un fichero byte a byte

```java
import java.io.*;

public class CopiaBytes {
    public static void main(String[] args) {
        try (FileInputStream origen = new FileInputStream("foto.jpg");
             FileOutputStream destino = new FileOutputStream("foto_copia.jpg")) {

            int b;
            while ((b = origen.read()) != -1) {
                destino.write(b);
            }
            System.out.println("Copia terminada.");

        } catch (IOException e) {
            System.err.println("Error de E/S: " + e.getMessage());
        }
    }
}
```

Este código **funciona**, pero es **lento**: cada `read()` y `write()` puede suponer una llamada al sistema operativo. Mejor leer por bloques:

```java
byte[] bloque = new byte[8192];   // 8 KiB
int leidos;
while ((leidos = origen.read(bloque)) != -1) {
    destino.write(bloque, 0, leidos);   // ¡ojo!: solo los 'leidos' bytes
}
```

> ⚠️ Error clásico: escribir `destino.write(bloque)` en lugar de `destino.write(bloque, 0, leidos)`. En la última vuelta el bloque puede no estar lleno y copiarías basura.

En Java 21 no hace falta escribir ese bucle a mano: `transferTo` lo hace por ti de forma eficiente.

```java
try (var origen = new FileInputStream("foto.jpg");
     var destino = new FileOutputStream("foto_copia.jpg")) {
    origen.transferTo(destino);
}
```

### 3.3 Comportamiento de `FileOutputStream`

- Si el fichero **no existe**, se crea.
- Si **existe**, se **sobrescribe** (se trunca a cero). Para **añadir** al final: `new FileOutputStream("fichero", true)`.

### 3.4 Cuándo NO usar flujos de bytes

Los flujos de bytes son el nivel más bajo. Si lo que manejas es **texto**, usa flujos de caracteres. Si manejas datos estructurados, usa flujos de datos u objetos (más adelante).

---

## 4. Flujos de caracteres

Los **flujos de caracteres** (`Reader` / `Writer`) trabajan con texto Unicode y se encargan de **convertir** entre caracteres y bytes según una **codificación (charset)**, como UTF-8, ISO-8859-1, etc.

### 4.1 Bytes vs. caracteres

- Un `char` en Java ocupa 16 bits y representa texto Unicode.
- En disco el texto son bytes, y la correspondencia depende de la codificación.
- Un `Reader`/`Writer` hace esa traducción por ti.

Internamente, los flujos de caracteres de fichero **usan flujos de bytes por debajo**: `FileReader` utiliza un `FileInputStream`, y `InputStreamReader` actúa como "puente" entre ambos mundos.

```
 bytes ─► FileInputStream ─► InputStreamReader (decodifica con charset) ─► chars
```

### 4.2 Ejemplo básico

```java
import java.io.*;

public class CopiaTexto {
    public static void main(String[] args) {
        try (FileReader in = new FileReader("entrada.txt");
             FileWriter out = new FileWriter("salida.txt")) {

            int c;
            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 4.3 La codificación (charset)

Desde **Java 18** ([JEP 400](https://openjdk.org/jeps/400)) la codificación por defecto de las APIs de ficheros es **UTF-8 en todas las plataformas**, de modo que `new FileReader("x.txt")` ya no depende del sistema operativo como ocurría antes (en Windows solía ser Windows-1252). Aun así, **indicar el charset explícitamente sigue siendo buena práctica**: deja clara la intención y evita sorpresas con ficheros heredados (por ejemplo, en ISO-8859-1) o con programas que se ejecuten con otra configuración:

```java
import java.nio.charset.StandardCharsets;

try (Reader in = new InputStreamReader(
             new FileInputStream("entrada.txt"), StandardCharsets.UTF_8);
     Writer out = new OutputStreamWriter(
             new FileOutputStream("salida.txt"), StandardCharsets.UTF_8)) {
    // ...
}
```

Desde Java 11 puedes escribirlo más corto: `new FileReader("x.txt", StandardCharsets.UTF_8)` y `new FileWriter("x.txt", StandardCharsets.UTF_8)`.

> 💡 Si ves caracteres raros como `Ã±` en lugar de `ñ`, casi seguro estás leyendo con una codificación distinta a la del fichero.
>
> ℹ️ La codificación de `System.out` es independiente: depende de la terminal y, desde Java 19, de la propiedad `stdout.encoding`. Un fichero puede estar bien en UTF-8 y verse mal en una consola configurada con otra codificación.

### 4.4 Lectura por líneas

Leer carácter a carácter rara vez es lo que queremos. Lo habitual es leer **línea a línea** con `BufferedReader` (siguiente sección). Para volcar un `Reader` completo en un `Writer` existe, desde Java 10, `reader.transferTo(writer)`.

---

## 5. Flujos con búfer (buffered)

### 5.1 El problema

Cada operación de lectura o escritura "sin búfer" se traduce en una petición directa al sistema operativo (disco, red…), lo cual es **muy costoso**.

### 5.2 La solución: el búfer

Los **flujos con búfer** guardan los datos en una zona de memoria intermedia:

- Al **leer**, traen un bloque grande de una vez y te van sirviendo desde memoria.
- Al **escribir**, acumulan datos en memoria y los envían al destino cuando el búfer se llena (o cuando se llama a `flush()`).

Clases:

| Bytes | Caracteres |
|---|---|
| `BufferedInputStream` | `BufferedReader` |
| `BufferedOutputStream` | `BufferedWriter` |

Se usan **envolviendo** (*decorando*) un flujo no bufereado:

```java
BufferedReader br = new BufferedReader(new FileReader("texto.txt"));
```

### 5.3 Ejemplo: leer un fichero línea a línea

```java
import java.io.*;
import java.nio.charset.StandardCharsets;

public class LeerLineas {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(new FileInputStream("poema.txt"),
                                      StandardCharsets.UTF_8))) {

            String linea;
            int numero = 1;
            while ((linea = br.readLine()) != null) {   // null = fin de fichero
                System.out.println(numero++ + ": " + linea);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 5.4 Ejemplo: escribir líneas

```java
try (BufferedWriter bw = new BufferedWriter(new FileWriter("salida.txt"))) {
    bw.write("Primera línea");
    bw.newLine();                 // salto de línea adecuado para el sistema operativo
    bw.write("Segunda línea");
    bw.newLine();
} catch (IOException e) {
    e.printStackTrace();
}
```

### 5.5 Volcar el búfer: `flush()`

- `close()` ya hace un `flush()` implícito.
- Si necesitas que los datos lleguen al destino **antes** de cerrar (por ejemplo, en una comunicación por red), llama a `flush()` manualmente.
- Algunas clases, como `PrintWriter`, permiten **autoflush** (vuelcan al encontrar un salto de línea o al llamar a `println`):

```java
PrintWriter pw = new PrintWriter(new FileWriter("log.txt"), true); // autoflush
```

> 💡 **Regla práctica:** si lees o escribes muchos datos, **usa siempre un búfer**.

---

## 6. Lectura y formateo de texto: `Scanner` y `printf`

Esta parte trata de **interpretar** texto (convertirlo en números, palabras…) y **formatear** salidas.

### 6.1 `Scanner`: dividir la entrada en *tokens*

`java.util.Scanner` divide su entrada en **tokens** (fragmentos) separados por **delimitadores**. Por defecto, el delimitador es cualquier espacio en blanco (espacios, tabuladores, saltos de línea).

```java
import java.io.*;
import java.util.Scanner;

public class SumarNumeros {
    public static void main(String[] args) throws IOException {
        double suma = 0;
        try (Scanner sc = new Scanner(new BufferedReader(new FileReader("numeros.txt")))) {
            while (sc.hasNext()) {
                if (sc.hasNextDouble()) {
                    suma += sc.nextDouble();
                } else {
                    System.out.println("Ignorado: " + sc.next());
                }
            }
        }
        System.out.println("Suma = " + suma);
    }
}
```

Métodos habituales:

| Método | Descripción |
|---|---|
| `next()` | Siguiente token como `String`. |
| `nextInt()`, `nextDouble()`, `nextBoolean()`, … | Siguiente token convertido al tipo. |
| `nextLine()` | Resto de la línea actual. |
| `hasNext()`, `hasNextInt()`, `hasNextLine()`… | Comprueban si hay más datos de ese tipo (¡úsalos antes de leer!). |
| `useDelimiter(String regex)` | Cambia el delimitador (admite expresiones regulares). |
| `useLocale(Locale)` | Configura el formato numérico esperado. |

#### Cambiar el delimitador

```java
Scanner sc = new Scanner("Ana, Luis,Marta ,Pedro");
sc.useDelimiter("\\s*,\\s*");   // coma con espacios opcionales alrededor
while (sc.hasNext()) {
    System.out.println("[" + sc.next() + "]");
}
```

#### Localización (¡importante en España!)

En España el separador decimal es la **coma** (`3,14`) y en el mundo anglosajón el **punto** (`3.14`). `Scanner` usa la configuración regional del sistema; para controlar el comportamiento, fíjala:

```java
Scanner sc = new Scanner(entrada).useLocale(Locale.US);       // lee 3.14
Scanner sc2 = new Scanner(entrada).useLocale(Locale.forLanguageTag("es-ES")); // lee 3,14
```

#### Error clásico con `nextInt()` + `nextLine()`

```java
Scanner sc = new Scanner(System.in);
System.out.print("Edad: ");
int edad = sc.nextInt();        // lee "25" pero deja el salto de línea en el búfer
System.out.print("Nombre: ");
String nombre = sc.nextLine();  // ¡lee la línea vacía que quedó!
```

**Solución:** consumir el salto sobrante con un `sc.nextLine()` extra tras `nextInt()`, o leer siempre con `nextLine()` y convertir con `Integer.parseInt(...)`.

### 6.2 Formateo: `print`, `println` y `printf`

Los objetos `System.out` y `System.err` (y cualquier `PrintStream`/`PrintWriter`) disponen de:

- `print(x)` / `println(x)`: convierten `x` a texto con `String.valueOf` y lo escriben.
- `printf(formato, args...)` / `format(...)`: escriben con **formato**.

```java
int i = 2;
double r = Math.sqrt(i);
System.out.printf("La raíz cuadrada de %d es %.4f%n", i, r);
// La raíz cuadrada de 2 es 1,4142   (con locale es-ES)
```

#### Especificadores de formato más usados

| Especificador | Significado | Ejemplo |
|---|---|---|
| `%d` | Entero decimal | `%5d` (ancho 5), `%05d` (relleno con ceros) |
| `%f` | Decimal con coma flotante | `%.2f` (2 decimales) |
| `%s` | Cadena (o `toString()`) | `%-10s` (alineada a la izquierda) |
| `%c` | Carácter | |
| `%b` | Booleano | |
| `%x` / `%X` | Hexadecimal | |
| `%e` | Notación científica | |
| `%,d` | Entero con separador de miles | `1.234.567` en `es-ES` |
| `%n` | Salto de línea **independiente de plataforma** | |
| `%%` | El símbolo `%` | |

> ⚠️ Usa `%n` en lugar de `\n` en `printf`: en Windows el salto de línea es `\r\n` y en Linux/macOS es `\n`.

#### Formatear sin imprimir

```java
String texto = String.format("%-10s|%8.2f €", "Café", 1.5);
// "Café      |    1,50 €"
```

#### `formatted()` y bloques de texto (Java 15+)

`String.formatted(...)` equivale a `String.format(this, ...)` y se combina muy bien con los **bloques de texto** (`"""`), útiles para plantillas de varias líneas:

```java
String ficha = """
        Nombre: %s
        Edad:   %d
        Nota:   %.1f
        """.formatted("Lucía", 20, 8.75);
System.out.print(ficha);
```

#### Tabla alineada

```java
String[] productos = {"Pan", "Leche", "Queso curado"};
double[] precios   = {0.85, 1.20, 12.5};
for (int i = 0; i < productos.length; i++) {
    System.out.printf("%-15s %8.2f €%n", productos[i], precios[i]);
}
```

#### Fijar el idioma del formato

```java
System.out.printf(Locale.US, "%.2f%n", 3.14159);   // 3.14
System.out.printf(Locale.forLanguageTag("es-ES"), "%.2f%n", 3.14159); // 3,14
```

---

## 7. E/S desde la línea de comandos

### 7.1 Los flujos estándar

Desde la consola, el programa puede interactuar con el usuario a través de `System.in`, `System.out` y `System.err` (ver [apartado 2.5](#25-los-flujos-estándar)).

- `System.in` es un `InputStream` de bytes: para leer texto cómodamente lo envolvemos en un `Scanner` o un `BufferedReader`.
- Se pueden **redirigir** desde el sistema operativo:

```bash
java MiPrograma < entrada.txt > salida.txt 2> errores.txt
```

También desde Java: `System.setOut(...)`, `System.setErr(...)`, `System.setIn(...)`.

> ℹ️ La codificación con la que `System.out` escribe en la terminal puede consultarse con `System.getProperty("stdout.encoding")` (Java 19+). Si ves acentos mal en la consola, revisa primero esa configuración.

```java
// Lectura típica desde teclado
try (BufferedReader teclado = new BufferedReader(new InputStreamReader(System.in))) {
    System.out.print("¿Cómo te llamas? ");
    String nombre = teclado.readLine();
    System.out.println("Hola, " + nombre);
}
```

> ⚠️ Si cierras `System.in` (por ejemplo, con el `try-with-resources` anterior), no podrás volver a leer del teclado durante el resto de la ejecución. En programas pequeños no importa; en programas mayores, ábrelo una vez y reutilízalo.

### 7.2 La clase `Console`

`java.io.Console` ofrece métodos específicos para interactuar con un terminal real: leer líneas, leer contraseñas sin eco y escribir con formato.

```java
Console consola = System.console();
if (consola == null) {
    System.err.println("No hay consola interactiva (¿IDE o E/S redirigida?).");
    System.exit(1);
}

String usuario = consola.readLine("Usuario: ");
char[] clave   = consola.readPassword("Contraseña: ");  // no se muestra al teclear

// ... comprobar credenciales ...

java.util.Arrays.fill(clave, ' ');   // borrar la contraseña de la memoria
consola.printf("Bienvenido, %s%n", usuario);
```

Puntos clave:

- **En Java 21**, `System.console()` devuelve **`null`** si la entrada o la salida no están conectadas a un terminal interactivo (por ejemplo, al ejecutar desde muchos IDE o al redirigir con `<` o `>`). Compruébalo siempre.
- **Desde Java 22**, `System.console()` devuelve normalmente un objeto `Console` aunque no haya terminal. Para saber si es interactivo existe el método `isTerminal()`. Si compilas con JDK 22 o superior, el patrón correcto es:

  ```java
  Console consola = System.console();
  if (consola != null && consola.isTerminal()) {
      // interacción completa: readLine, readPassword...
  } else {
      // sin terminal: leer de System.in o de ficheros
  }
  ```

  (Ese método no existe en JDK 21: si necesitas compilar con 21, usa solo la comprobación de `null`.)
- `readPassword` devuelve un `char[]` (y no un `String`) para poder **borrarlo de memoria** una vez usado. Los `String` son inmutables y pueden permanecer en memoria más tiempo del deseado.

#### La clase `java.io.IO` (Java 25)

Java 25 ([JEP 512](https://openjdk.org/jeps/512)) incorpora una clase de utilidad con métodos estáticos para la consola, pensada para programas pequeños y de aprendizaje:

```java
String nombre = IO.readln("¿Cómo te llamas? ");
IO.println("Hola, " + nombre);
```

Para aplicaciones reales, `Scanner`, `BufferedReader` y `Console` siguen siendo las herramientas habituales.

---

## 8. Flujos de datos (Data Streams)

Los **flujos de datos** (`DataInputStream` y `DataOutputStream`) permiten escribir y leer **tipos primitivos** (`int`, `double`, `boolean`…) y cadenas en formato **binario** y portable.

### 8.1 Métodos principales

| Escritura (`DataOutputStream`) | Lectura (`DataInputStream`) |
|---|---|
| `writeInt(int)` | `readInt()` |
| `writeDouble(double)` | `readDouble()` |
| `writeBoolean(boolean)` | `readBoolean()` |
| `writeLong(long)` | `readLong()` |
| `writeUTF(String)` | `readUTF()` |
| … | … |

### 8.2 Ejemplo: guardar y recuperar una lista de productos

```java
import java.io.*;

public class DatosProductos {
    public static void main(String[] args) throws IOException {
        String[] nombres  = {"Camiseta", "Pantalón", "Calcetines"};
        int[]    unidades = {12, 5, 40};
        double[] precios  = {14.95, 29.90, 3.50};

        // ESCRITURA
        try (DataOutputStream out = new DataOutputStream(
                new BufferedOutputStream(new FileOutputStream("productos.dat")))) {
            for (int i = 0; i < nombres.length; i++) {
                out.writeUTF(nombres[i]);
                out.writeInt(unidades[i]);
                out.writeDouble(precios[i]);
            }
        }

        // LECTURA
        try (DataInputStream in = new DataInputStream(
                new BufferedInputStream(new FileInputStream("productos.dat")))) {
            while (true) {
                String nombre = in.readUTF();
                int cantidad  = in.readInt();
                double precio = in.readDouble();
                System.out.printf("%-12s %3d uds. a %6.2f €%n", nombre, cantidad, precio);
            }
        } catch (EOFException fin) {
            // Es la forma normal de detectar el final con DataInputStream
        }
    }
}
```

### 8.3 Cosas que debes saber

- **Fin del fichero:** a diferencia de `read()`, los métodos `readXxx()` **no devuelven -1**; lanzan `EOFException`.
- **El orden importa:** hay que leer exactamente en el mismo orden y con los mismos tipos con que se escribió. El fichero no guarda "metadatos" sobre la estructura.
- **Portabilidad:** el formato es independiente de la plataforma (siempre *big-endian*).
- **Precisión monetaria:** en el ejemplo se usa `double` por simplicidad, pero **para dinero real se recomienda `BigDecimal`**, ya que `double` no representa con exactitud ciertos decimales.

---

## 9. Flujos de objetos y serialización

Los **flujos de objetos** (`ObjectInputStream` y `ObjectOutputStream`) permiten guardar un **objeto completo** en un flujo y reconstruirlo después. A este proceso se le llama **serialización** (y **deserialización** al reconstruirlo).

### 9.1 Requisitos para serializar

La clase debe implementar la interfaz marcadora `java.io.Serializable` (no tiene métodos):

```java
import java.io.Serializable;

public class Alumno implements Serializable {
    private static final long serialVersionUID = 1L;

    private String nombre;
    private int edad;
    private transient String claveTemporal;   // NO se serializa

    public Alumno(String nombre, int edad, String claveTemporal) {
        this.nombre = nombre;
        this.edad = edad;
        this.claveTemporal = claveTemporal;
    }

    @Override
    public String toString() {
        return nombre + " (" + edad + " años) clave=" + claveTemporal;
    }
}
```

- **`serialVersionUID`:** identifica la "versión" de la clase. Si cambias la clase y el identificador no coincide con el del objeto guardado, la lectura fallará con `InvalidClassException`. Declararlo explícitamente te da control.
- **`transient`:** marca atributos que **no** deben guardarse (contraseñas, cachés, recursos no serializables). Al deserializar valdrán `null`, `0`, `false`…
- **`static`:** los atributos estáticos pertenecen a la clase, no al objeto, y no se serializan.
- Todos los atributos de tipo objeto deben ser a su vez `Serializable` (o `transient`), si no, obtendrás una `NotSerializableException`.

### 9.2 Ejemplo: escribir y leer objetos

```java
import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class PruebaSerializacion {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        List<Alumno> alumnos = new ArrayList<>();
        alumnos.add(new Alumno("Lucía", 20, "abc123"));
        alumnos.add(new Alumno("Mario", 22, "zzz999"));

        // Serializar
        try (ObjectOutputStream out = new ObjectOutputStream(
                new BufferedOutputStream(new FileOutputStream("alumnos.ser")))) {
            out.writeObject(alumnos);       // ArrayList ya es Serializable
        }

        // Deserializar
        try (ObjectInputStream in = new ObjectInputStream(
                new BufferedInputStream(new FileInputStream("alumnos.ser")))) {
            @SuppressWarnings("unchecked")
            List<Alumno> leidos = (List<Alumno>) in.readObject();
            leidos.forEach(System.out::println);
            // Lucía (20 años) clave=null     ← 'transient' no se recuperó
            // Mario (22 años) clave=null
        }
    }
}
```

### 9.3 Grafos de objetos

Si serializas un objeto que referencia a otros, Java serializa **todo el grafo** de forma automática, y mantiene las referencias compartidas: si dos objetos apuntan al mismo tercer objeto, al leerlos seguirán apuntando a **una única copia**.

### 9.4 Advertencias de seguridad y diseño

- 🔒 **Nunca deserialices datos de procedencia no fiable.** La deserialización de Java es una conocida fuente de vulnerabilidades. Si necesitas intercambiar datos con terceros, valora formatos como **JSON** o **XML**. Si aun así debes deserializar, usa un **filtro** (ver [9.6](#96-filtros-de-deserialización-objectinputfilter)).
- Si tu clase es una dependencia de larga vida, el formato serializado nativo puede dificultar su evolución. Para persistencia a largo plazo, suele ser preferible una base de datos o JSON.
- Para controlar el proceso hay métodos opcionales `private void writeObject(ObjectOutputStream)` y `private void readObject(ObjectInputStream)`, o la interfaz `Externalizable`. Son temas avanzados.

### 9.5 Records y serialización

Un `record` (Java 16+) puede ser serializable si implementa `Serializable`. Es una opción muy cómoda para datos inmutables:

```java
import java.io.Serializable;

public record Producto(String nombre, double precio) implements Serializable {
    public Producto {                       // constructor compacto con validación
        if (precio < 0) {
            throw new IllegalArgumentException("El precio no puede ser negativo");
        }
    }
}
```

Diferencias respecto a una clase normal:

- Se serializa a partir de sus **componentes**, y al deserializar se invoca el **constructor canónico**: las validaciones del constructor se ejecutan también al leer, lo que aporta seguridad extra.
- Los métodos personalizados `writeObject`/`readObject` **no se usan** en los records.
- No se exige que coincida el `serialVersionUID` (por defecto es `0L`).

### 9.6 Filtros de deserialización (`ObjectInputFilter`)

Para reducir el riesgo al deserializar, puedes indicar qué clases se aceptan y fijar límites. Los patrones se evalúan en orden: lo permitido se indica tal cual y lo prohibido con `!`. El último patrón `!*` rechaza todo lo que no se haya permitido antes.

```java
import java.io.*;

ObjectInputFilter filtro = ObjectInputFilter.Config.createFilter(
        "maxbytes=1000000;maxdepth=20;"   // límites de tamaño y profundidad
      + "com.miapp.modelo.*;"             // tus clases (ajusta el paquete)
      + "java.util.ArrayList;"            // colecciones que uses
      + "!*");                            // todo lo demás, rechazado

try (ObjectInputStream in = new ObjectInputStream(
        new BufferedInputStream(new FileInputStream("alumnos.ser")))) {
    in.setObjectInputFilter(filtro);      // ANTES de llamar a readObject()
    Object o = in.readObject();
}
```

Si una clase no permitida aparece en el flujo, se lanza `InvalidClassException`. También se puede configurar un filtro global con la propiedad `jdk.serialFilter`.

> 💡 Para datos que viajan entre sistemas o se guardan a largo plazo, hoy se prefieren **formatos explícitos** (JSON, Protocol Buffers…) antes que la serialización nativa de Java.

---

## 10. E/S de ficheros con NIO.2

**NIO.2** es la API moderna de ficheros incorporada en **Java 7** (paquete `java.nio.file`). Sustituye con ventaja a la antigua clase `java.io.File`, ya que ofrece:

- Mensajes de error con excepciones claras (la clase `File` solía devolver `false` sin explicar el motivo).
- Soporte de enlaces simbólicos.
- Acceso a atributos específicos de cada sistema de ficheros.
- Recorrido de árboles de directorios y vigilancia de cambios.
- Operaciones atómicas y mejor rendimiento.

Las piezas principales son:

| Clase | Función |
|---|---|
| `Path` | Representa una **ruta** en el sistema de ficheros. |
| `Paths` / `Path.of(...)` | Crean objetos `Path`. |
| `Files` | Clase de utilidades con métodos **estáticos** para operar sobre rutas. |
| `FileSystem` / `FileSystems` | Acceso al sistema de ficheros. |

---

### 10.1 ¿Qué es una ruta (`Path`)?

Una **ruta** identifica de forma única un fichero o directorio dentro de un sistema de ficheros. Se compone de nombres separados por un **separador** que depende del sistema operativo: `/` en Linux y macOS, `\` en Windows.

#### Ruta absoluta vs. relativa

| Tipo | Ejemplo (Linux) | Ejemplo (Windows) |
|---|---|---|
| **Absoluta** (desde la raíz) | `/home/ana/docs/notas.txt` | `C:\Users\ana\docs\notas.txt` |
| **Relativa** (desde el directorio actual) | `docs/notas.txt` | `docs\notas.txt` |

Una ruta relativa se interpreta respecto al **directorio de trabajo actual** del proceso (visible con `System.getProperty("user.dir")`).

#### Símbolos especiales

- `.` : el directorio actual.
- `..` : el directorio padre.

Así, `/home/ana/docs/../fotos` equivale a `/home/ana/fotos`.

#### Enlaces simbólicos

Algunos sistemas permiten ficheros especiales que apuntan a otro fichero o directorio. Los vemos en el [apartado 10.13](#1013-enlaces-simbólicos-y-duros).

---

### 10.2 La clase `Path`

`Path` es una **interfaz** que representa una ruta. Un objeto `Path` **no tiene por qué existir realmente** en disco: es solo una secuencia de nombres. Es **inmutable**, así que sus operaciones devuelven nuevos objetos.

#### Crear rutas

```java
import java.nio.file.*;

Path p1 = Paths.get("/home/ana/docs/notas.txt");
Path p2 = Paths.get("docs", "notas.txt");          // une las partes con el separador adecuado
Path p3 = Path.of("docs", "notas.txt");            // forma recomendada desde Java 11 (Paths.get sigue funcionando)
Path p4 = Path.of(System.getProperty("user.home"), "documentos");
```

#### Información sobre una ruta

```java
Path ruta = Path.of("/home/ana/docs/notas.txt");

System.out.println(ruta.toString());        // /home/ana/docs/notas.txt
System.out.println(ruta.getFileName());     // notas.txt
System.out.println(ruta.getParent());       // /home/ana/docs
System.out.println(ruta.getRoot());         // /
System.out.println(ruta.getNameCount());    // 4  (home, ana, docs, notas.txt)
System.out.println(ruta.getName(0));        // home
System.out.println(ruta.subpath(1, 3));     // ana/docs
```

Recorrer sus elementos:

```java
for (Path elemento : ruta) {
    System.out.println(elemento);
}
```

---

### 10.3 Operaciones con rutas

Estas operaciones son **sintácticas**: manipulan la ruta como texto estructurado sin tocar el disco (salvo que se indique).

#### `normalize()`: eliminar redundancias

```java
Path p = Path.of("/home/ana/./docs/../fotos/viaje.jpg");
System.out.println(p.normalize());    // /home/ana/fotos/viaje.jpg
```

> ⚠️ `normalize()` no comprueba si los enlaces simbólicos alteran el significado real de `..`.

#### `toAbsolutePath()` y `toRealPath()`

```java
Path rel = Path.of("datos.txt");
System.out.println(rel.toAbsolutePath());   // /directorio/actual/datos.txt  (no accede al disco)

Path real = rel.toRealPath();               // exige que exista; resuelve enlaces y normaliza
```

`toRealPath()` lanza `NoSuchFileException` si el fichero no existe.

#### `resolve()`: unir rutas

```java
Path base = Path.of("/home/ana");
Path completa = base.resolve("docs/notas.txt");     // /home/ana/docs/notas.txt

// Si el argumento es absoluto, se devuelve tal cual
Path otra = base.resolve("/etc/hosts");              // /etc/hosts
```

Muy útil para construir una ruta a un fichero dentro de un directorio:

```java
Path destino = directorioDestino.resolve(origen.getFileName());
```

#### `relativize()`: ruta de una a otra

```java
Path a = Path.of("/home/ana");
Path b = Path.of("/home/ana/docs/notas.txt");
System.out.println(a.relativize(b));    // docs/notas.txt
System.out.println(b.relativize(a));    // ../..
```

Ambas rutas deben ser del mismo tipo (las dos absolutas o las dos relativas).

#### Comparar rutas

```java
Path x = Path.of("/home/ana/docs");
Path y = Path.of("/home/ana/docs/notas.txt");

x.equals(y);                 // false
y.startsWith(x);             // true
y.endsWith("notas.txt");     // true
x.compareTo(y);              // orden lexicográfico (negativo, 0 o positivo)
Files.isSameFile(x, y);      // ¿apuntan al MISMO fichero real? (accede al disco)
```

#### Conversión con `java.io.File` y URI

```java
File f = ruta.toFile();        // Path → File
Path p = f.toPath();           // File → Path
java.net.URI uri = ruta.toUri();
```

---

### 10.4 Operaciones con ficheros: ideas comunes

Antes de ver métodos concretos, conviene conocer conceptos que se repiten en muchos de ellos.

#### Liberar recursos

Muchos recursos (flujos, canales, `DirectoryStream`…) implementan `Closeable`/`AutoCloseable`. **Usa siempre `try-with-resources`.**

#### Capturar excepciones

Los métodos de `Files` lanzan `IOException` o una subclase. Algunas útiles:

| Excepción | Cuándo ocurre |
|---|---|
| `NoSuchFileException` | El fichero o directorio no existe. |
| `FileAlreadyExistsException` | Ya existe y no se permitía sobrescribir. |
| `AccessDeniedException` | Permisos insuficientes. |
| `DirectoryNotEmptyException` | Se intentó borrar un directorio con contenido. |
| `NotDirectoryException` | Se esperaba un directorio. |

```java
try {
    Files.delete(Path.of("noexiste.txt"));
} catch (NoSuchFileException e) {
    System.err.println("No existe: " + e.getFile());
} catch (DirectoryNotEmptyException e) {
    System.err.println("El directorio no está vacío: " + e.getFile());
} catch (IOException e) {
    System.err.println("Otro error: " + e);
}
```

#### Métodos encadenados y varargs

Muchos métodos aceptan un número variable de **opciones** (`OpenOption`, `CopyOption`, `LinkOption`, `FileVisitOption`…) al final:

```java
Files.copy(origen, destino, StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.COPY_ATTRIBUTES);
```

#### Operaciones atómicas

Una operación **atómica** se realiza completa o no se realiza: nadie verá un estado intermedio. Algunas, como `Files.move` con `ATOMIC_MOVE`, lo garantizan (si el sistema lo soporta).

#### Encadenamiento de métodos y enlaces

Por defecto, la mayoría de métodos **siguen** los enlaces simbólicos. Para evitarlo se pasa `LinkOption.NOFOLLOW_LINKS`.

---

### 10.5 Comprobar un fichero o directorio

```java
Path p = Path.of("datos.txt");

Files.exists(p);          // ¿existe?
Files.notExists(p);       // ¿seguro que NO existe?
Files.isDirectory(p);
Files.isRegularFile(p);
Files.isReadable(p);
Files.isWritable(p);
Files.isExecutable(p);
Files.isHidden(p);        // puede lanzar IOException
Files.isSameFile(p, otro);
```

> 🤔 **¿Por qué `exists` y `notExists` y no solo uno?** Porque hay **tres** posibles situaciones: existe, no existe y *no se puede saber* (por ejemplo, por falta de permisos). En el tercer caso, **ambos devuelven `false`**.

> ⚠️ **Condiciones de carrera (*race conditions*):** comprobar y luego actuar (`if (Files.exists(p)) { leer }`) no es seguro, porque otro proceso puede borrar el fichero justo en medio. Lo más robusto es **intentar la operación y capturar la excepción**.

---

### 10.6 Borrar un fichero o directorio

```java
Files.delete(p);               // lanza excepción si no existe o hay problemas
boolean borrado = Files.deleteIfExists(p);   // devuelve false si no existía
```

- Un **directorio** solo se puede borrar si está **vacío**. Para borrar uno con contenido hay que recorrer su árbol (ver [10.14](#1014-recorrer-un-árbol-de-ficheros)).
- Si `p` es un enlace simbólico, se borra el **enlace**, no el destino.

---

### 10.7 Copiar un fichero o directorio

```java
Files.copy(origen, destino);
```

Por defecto **falla** con `FileAlreadyExistsException` si `destino` existe. Opciones (`StandardCopyOption`):

| Opción | Efecto |
|---|---|
| `REPLACE_EXISTING` | Sobrescribe el destino si existe. |
| `COPY_ATTRIBUTES` | Copia también los atributos (fechas, permisos…). |
| `NOFOLLOW_LINKS` | Si es un enlace simbólico, copia el enlace, no su destino. |

```java
Files.copy(Path.of("a.txt"), Path.of("copias/a.txt"), StandardCopyOption.REPLACE_EXISTING);
```

Copiar un directorio con `Files.copy` crea el directorio **vacío** (no copia su contenido).

#### Copiar desde/hacia flujos

```java
// Descargar un recurso a un fichero
// (el constructor new URL(String) está obsoleto desde Java 20: se usa URI)
try (InputStream in = java.net.URI.create("https://ejemplo.org/f.txt").toURL().openStream()) {
    Files.copy(in, Path.of("f.txt"), StandardCopyOption.REPLACE_EXISTING);
}

// Volcar un fichero a la salida estándar
Files.copy(Path.of("f.txt"), System.out);
```

---

### 10.8 Mover un fichero o directorio

```java
Files.move(origen, destino);
Files.move(origen, destino, StandardCopyOption.REPLACE_EXISTING);
Files.move(origen, destino, StandardCopyOption.ATOMIC_MOVE);
```

- También sirve para **renombrar**: moverlo dentro del mismo directorio con otro nombre.
- Un directorio vacío se puede mover; uno con contenido solo si el movimiento no requiere mover físicamente los ficheros (por ejemplo, dentro del mismo volumen).
- Con `ATOMIC_MOVE`, el resto de opciones se ignoran y se lanza `AtomicMoveNotSupportedException` si no es posible.

---

### 10.9 Metadatos y atributos

Los **metadatos** son datos sobre el fichero: tamaño, fechas, propietario, permisos…

#### Métodos rápidos

```java
long bytes = Files.size(p);
var modificado = Files.getLastModifiedTime(p);
var propietario = Files.getOwner(p);
```

#### Atributos básicos agrupados (recomendado)

Leer los atributos de una vez es más eficiente que preguntar uno a uno:

```java
import java.nio.file.attribute.BasicFileAttributes;

BasicFileAttributes attr = Files.readAttributes(p, BasicFileAttributes.class);

System.out.println("Creado:       " + attr.creationTime());
System.out.println("Último acceso:" + attr.lastAccessTime());
System.out.println("Modificado:   " + attr.lastModifiedTime());
System.out.println("Tamaño:       " + attr.size());
System.out.println("¿Directorio?  " + attr.isDirectory());
System.out.println("¿Regular?     " + attr.isRegularFile());
System.out.println("¿Enlace?      " + attr.isSymbolicLink());
```

#### Vistas específicas del sistema

| Vista | Contenido |
|---|---|
| `BasicFileAttributeView` | Atributos básicos (en todos los sistemas). |
| `DosFileAttributeView` | Atributos DOS/Windows: solo lectura, oculto, sistema, archivo. |
| `PosixFileAttributeView` | Propietario, grupo y permisos estilo Unix (Linux, macOS). |
| `AclFileAttributeView` | Listas de control de acceso. |
| `FileOwnerAttributeView`, `UserDefinedFileAttributeView` | Propietario / atributos definidos por el usuario. |

Ejemplo con permisos POSIX:

```java
import java.nio.file.attribute.*;
import java.util.Set;

Set<PosixFilePermission> permisos = PosixFilePermissions.fromString("rw-r-----");
Files.setPosixFilePermissions(p, permisos);

System.out.println(PosixFilePermissions.toString(Files.getPosixFilePermissions(p)));
```

> ⚠️ En Windows las vistas POSIX lanzan `UnsupportedOperationException`. Comprueba qué vistas admite el sistema con `FileSystems.getDefault().supportedFileAttributeViews()`.

#### Modificar fechas

```java
Files.setLastModifiedTime(p, java.nio.file.attribute.FileTime.fromMillis(System.currentTimeMillis()));
```

#### Almacenes de ficheros (`FileStore`)

Información del volumen (disco, partición):

```java
FileStore almacen = Files.getFileStore(p);
System.out.println("Total:       " + almacen.getTotalSpace());
System.out.println("Usado:       " + (almacen.getTotalSpace() - almacen.getUnallocatedSpace()));
System.out.println("Disponible:  " + almacen.getUsableSpace());
```

---

### 10.10 Leer, escribir y crear ficheros

Esta es la parte que más usarás en el día a día. Hay varios niveles según el tamaño del fichero y el control que necesites:

| Necesidad | Método recomendado |
|---|---|
| Fichero **pequeño**, todo de golpe | `Files.readAllBytes`, `Files.readAllLines`, `Files.readString`, `Files.write`, `Files.writeString` |
| Fichero **grande**, texto línea a línea | `Files.newBufferedReader` / `Files.newBufferedWriter`, o `Files.lines` |
| Fichero **grande**, binario | `Files.newInputStream` / `Files.newOutputStream` |
| Acceso **no secuencial** | `SeekableByteChannel` / `FileChannel` / `RandomAccessFile` |

#### Opciones de apertura (`StandardOpenOption`)

| Opción | Descripción |
|---|---|
| `READ` | Abrir para lectura. |
| `WRITE` | Abrir para escritura. |
| `APPEND` | Escribir al final. |
| `TRUNCATE_EXISTING` | Vaciar el fichero al abrirlo para escritura. |
| `CREATE` | Crear si no existe. |
| `CREATE_NEW` | Crear; falla si ya existe. |
| `DELETE_ON_CLOSE` | Borrar al cerrar (útil para temporales). |
| `SYNC` / `DSYNC` | Escritura síncrona en disco. |

#### Fichero pequeño: todo de una vez

```java
// Leer
byte[] bytes = Files.readAllBytes(Path.of("config.bin"));
List<String> lineas = Files.readAllLines(Path.of("nombres.txt"), StandardCharsets.UTF_8);
String contenido = Files.readString(Path.of("nota.txt"));  // siempre UTF-8 si no se indica otro charset

// Escribir
Files.write(Path.of("salida.bin"), bytes);
Files.write(Path.of("salida.txt"), lineas, StandardCharsets.UTF_8);
Files.writeString(Path.of("nota.txt"), "Hola\n");

// Añadir al final
Files.writeString(Path.of("log.txt"), "Nueva entrada\n",
                  StandardOpenOption.CREATE, StandardOpenOption.APPEND);
```

> ⚠️ Estos métodos cargan **todo** el contenido en memoria. No los uses con ficheros enormes.
>
> ℹ️ Los métodos de `Files` que trabajan con texto (`readString`, `readAllLines`, `lines`, `newBufferedReader`…) usan **UTF-8** cuando no indicas charset, con independencia de la codificación por defecto de la JVM.

#### Texto grande: `BufferedReader` y `BufferedWriter`

```java
Path entrada = Path.of("grande.txt");
Path salida  = Path.of("mayusculas.txt");

try (BufferedReader br = Files.newBufferedReader(entrada, StandardCharsets.UTF_8);
     BufferedWriter bw = Files.newBufferedWriter(salida, StandardCharsets.UTF_8)) {

    String linea;
    while ((linea = br.readLine()) != null) {
        bw.write(linea.toUpperCase());
        bw.newLine();
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

#### Texto con `Files.lines` (Streams)

```java
try (Stream<String> lineas = Files.lines(Path.of("registro.log"), StandardCharsets.UTF_8)) {
    long errores = lineas.filter(l -> l.contains("ERROR")).count();
    System.out.println("Errores: " + errores);
}
```

> ⚠️ `Files.lines` mantiene el fichero abierto: **hay que cerrar el `Stream`** (con `try-with-resources`).

Si necesitas una lista, `Stream.toList()` (Java 16+) devuelve una lista inmodificable:

```java
try (var lineas = Files.lines(Path.of("registro.log"))) {
    List<String> avisos = lineas.filter(l -> l.contains("WARN")).toList();
    avisos.forEach(System.out::println);
}
```

#### Binario grande: `InputStream` / `OutputStream`

```java
try (InputStream in = Files.newInputStream(Path.of("video.mp4"));
     OutputStream out = Files.newOutputStream(Path.of("copia.mp4"))) {
    in.transferTo(out);                // copia todo el flujo de forma eficiente
}
```

(Los flujos devueltos por `Files.newInputStream`/`newOutputStream` no son bufereados: si vas a leer byte a byte, envuélvelos en `BufferedInputStream`/`BufferedOutputStream`.)

#### Crear ficheros y directorios temporales

```java
Path temporal = Files.createTempFile("informe_", ".tmp");
Path dirTemp  = Files.createTempDirectory("trabajo_");
```

El sistema decide el directorio y añade números aleatorios al nombre. Combínalo con `deleteOnExit` o `StandardOpenOption.DELETE_ON_CLOSE` si no quieres dejar basura.

#### Crear un fichero vacío

```java
Files.createFile(Path.of("vacio.txt"));   // lanza FileAlreadyExistsException si ya existe
```

#### Canales (`SeekableByteChannel`)

Un **canal** es una alternativa a los flujos que permite lectura/escritura mediante `ByteBuffer`, y (con `SeekableByteChannel`) moverse a cualquier posición. Se explica en el siguiente apartado.

---

### 10.11 Ficheros de acceso aleatorio

Un flujo normal es **secuencial**: lees o escribes de principio a fin. En un fichero de **acceso aleatorio** puedes saltar a cualquier posición (como pasar directamente a la página 50 de un libro).

#### Opción 1: `SeekableByteChannel` (NIO.2)

```java
import java.nio.ByteBuffer;
import java.nio.channels.SeekableByteChannel;

Path fichero = Path.of("datos.bin");

try (SeekableByteChannel canal = Files.newByteChannel(fichero,
        StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {

    // Escribir al principio
    canal.write(ByteBuffer.wrap("ABCDEFGHIJ".getBytes(StandardCharsets.US_ASCII)));

    // Saltar a la posición 3 y leer 4 bytes
    canal.position(3);
    ByteBuffer buf = ByteBuffer.allocate(4);
    canal.read(buf);
    buf.flip();                                          // preparar el búfer para leerlo
    System.out.println(StandardCharsets.US_ASCII.decode(buf));   // DEFG

    // Sobrescribir en la posición 0
    canal.position(0);
    canal.write(ByteBuffer.wrap("ZZ".getBytes(StandardCharsets.US_ASCII)));

    System.out.println("Tamaño: " + canal.size());       // 10
}
```

Métodos clave: `position()`, `position(long)`, `read(ByteBuffer)`, `write(ByteBuffer)`, `size()`, `truncate(long)`.

> 💡 **`ByteBuffer.flip()`:** un `ByteBuffer` tiene una posición y un límite. Tras escribir en él (por ejemplo, al leer del canal), hay que llamar a `flip()` para poder leer lo escrito.

#### Opción 2: `RandomAccessFile` (clásica)

```java
try (RandomAccessFile raf = new RandomAccessFile("datos.bin", "rw")) {   // "r" o "rw"
    raf.writeInt(100);
    raf.writeInt(200);
    raf.writeInt(300);

    raf.seek(4);                      // saltar al byte 4 (segundo entero)
    System.out.println(raf.readInt());   // 200

    raf.seek(raf.length());           // ir al final para añadir
    raf.writeInt(400);
}
```

Ideal para registros de **tamaño fijo**: el registro número `n` está en la posición `n * tamañoRegistro`.

---

### 10.12 Crear y leer directorios

#### Directorios raíz y listado de unidades

```java
for (Path raiz : FileSystems.getDefault().getRootDirectories()) {
    System.out.println(raiz);
}
```

#### Crear directorios

```java
Files.createDirectory(Path.of("nuevo"));                 // falla si ya existe o si falta el padre
Files.createDirectories(Path.of("a/b/c"));               // crea toda la cadena; no falla si existe
```

#### Listar el contenido de un directorio

**Con `DirectoryStream`** (no recursivo, eficiente con directorios enormes):

```java
Path dir = Path.of("docs");

try (DirectoryStream<Path> flujo = Files.newDirectoryStream(dir)) {
    for (Path entrada : flujo) {
        System.out.println(entrada.getFileName());
    }
} catch (IOException | DirectoryIteratorException e) {
    e.printStackTrace();
}
```

**Filtrando con *glob*:**

```java
try (DirectoryStream<Path> flujo = Files.newDirectoryStream(dir, "*.{java,class}")) {
    flujo.forEach(System.out::println);
}
```

**Filtrando con una condición propia:**

```java
DirectoryStream.Filter<Path> soloGrandes = p -> Files.isRegularFile(p) && Files.size(p) > 1_000_000;

try (DirectoryStream<Path> flujo = Files.newDirectoryStream(dir, soloGrandes)) {
    flujo.forEach(System.out::println);
}
```

**Con `Stream` (`Files.list`):**

```java
try (Stream<Path> s = Files.list(dir)) {
    s.filter(Files::isDirectory).forEach(System.out::println);
}
```

> ⚠️ No hay orden garantizado en el listado. Si lo necesitas, ordena tú mismo (`.sorted()`).

---

### 10.13 Enlaces simbólicos y duros

| Tipo | Qué es |
|---|---|
| **Enlace simbólico** (*symlink*) | Fichero especial que **contiene la ruta** a otro. Si el destino se borra, el enlace queda "roto". |
| **Enlace duro** (*hard link*) | Otro **nombre** para el mismo contenido en disco. Solo para ficheros, mismo volumen. El contenido existe mientras quede algún enlace. |

```java
Path enlace  = Path.of("atajo");
Path destino = Path.of("/home/ana/docs/notas.txt");

Files.createSymbolicLink(enlace, destino);        // puede requerir privilegios (p. ej., en Windows)
Files.createLink(Path.of("duro.txt"), destino);   // enlace duro

Files.isSymbolicLink(enlace);                     // true
Path apuntaA = Files.readSymbolicLink(enlace);    // /home/ana/docs/notas.txt
```

Para ignorar el enlace y trabajar sobre el propio enlace usa `LinkOption.NOFOLLOW_LINKS`:

```java
BasicFileAttributes a = Files.readAttributes(enlace, BasicFileAttributes.class, LinkOption.NOFOLLOW_LINKS);
```

---

### 10.14 Recorrer un árbol de ficheros

#### Con `FileVisitor` y `Files.walkFileTree`

La interfaz `FileVisitor<T>` define cuatro métodos que se invocan durante el recorrido (en profundidad):

| Método | Se invoca… |
|---|---|
| `preVisitDirectory` | Antes de visitar el contenido de un directorio. |
| `visitFile` | Al visitar un fichero. |
| `visitFileFailed` | Si no se puede acceder a un fichero o directorio. |
| `postVisitDirectory` | Después de visitar todo el contenido de un directorio. |

Cada método devuelve un `FileVisitResult`:

| Valor | Efecto |
|---|---|
| `CONTINUE` | Seguir. |
| `TERMINATE` | Terminar el recorrido. |
| `SKIP_SUBTREE` | No entrar en este directorio (solo válido en `preVisitDirectory`). |
| `SKIP_SIBLINGS` | Saltar los hermanos pendientes. |

La clase `SimpleFileVisitor` implementa todos los métodos por defecto; solo sobrescribes los que necesites.

**Ejemplo: calcular el tamaño total de un directorio**

```java
public class TamanoDirectorio extends SimpleFileVisitor<Path> {
    private long total = 0;

    @Override
    public FileVisitResult visitFile(Path fichero, BasicFileAttributes attrs) {
        total += attrs.size();
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFileFailed(Path fichero, IOException exc) {
        System.err.println("No se pudo leer: " + fichero + " (" + exc + ")");
        return FileVisitResult.CONTINUE;
    }

    public long getTotal() { return total; }

    public static void main(String[] args) throws IOException {
        TamanoDirectorio visitor = new TamanoDirectorio();
        Files.walkFileTree(Path.of("."), visitor);
        System.out.println("Total: " + visitor.getTotal() + " bytes");
    }
}
```

**Ejemplo: borrar un directorio con todo su contenido**

```java
Files.walkFileTree(Path.of("temporal"), new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path f, BasicFileAttributes a) throws IOException {
        Files.delete(f);
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path d, IOException e) throws IOException {
        if (e != null) throw e;
        Files.delete(d);              // ya está vacío
        return FileVisitResult.CONTINUE;
    }
});
```

**Ejemplo: copiar un árbol completo**

```java
Path origen = Path.of("proyecto");
Path destino = Path.of("copia_proyecto");

Files.walkFileTree(origen, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes a) throws IOException {
        Files.createDirectories(destino.resolve(origen.relativize(dir)));
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFile(Path f, BasicFileAttributes a) throws IOException {
        Files.copy(f, destino.resolve(origen.relativize(f)), StandardCopyOption.REPLACE_EXISTING);
        return FileVisitResult.CONTINUE;
    }
});
```

#### Con `Files.walk` (Streams)

Más breve para recorridos sencillos:

```java
try (Stream<Path> s = Files.walk(Path.of("src"))) {
    s.filter(p -> p.toString().endsWith(".java"))
     .forEach(System.out::println);
}
```

Puedes limitar la profundidad: `Files.walk(dir, 2)`.

#### Consideraciones

- Por defecto **no se siguen** los enlaces simbólicos. Para seguirlos, pasa `FileVisitOption.FOLLOW_LINKS` (detecta ciclos y lanza `FileSystemLoopException` en `visitFileFailed`).
- Se recorre en **profundidad** (*depth-first*).

---

### 10.15 Buscar ficheros

#### `PathMatcher` y patrones *glob* o *regex*

```java
PathMatcher m = FileSystems.getDefault().getPathMatcher("glob:*.{java,class}");

Path p = Path.of("Main.java");
System.out.println(m.matches(p.getFileName()));   // true
```

El prefijo indica la sintaxis: `glob:` o `regex:`.

#### Sintaxis *glob*

| Patrón | Significado |
|---|---|
| `*` | Cero o más caracteres dentro de **un** nivel (no cruza `/`). |
| `**` | Cero o más caracteres **cruzando** directorios. |
| `?` | Exactamente un carácter. |
| `[abc]`, `[a-z]` | Uno de los caracteres indicados / rango. |
| `{java,class}` | Cualquiera de las alternativas. |
| `\` | Escapa un carácter especial. |

Ejemplos:

- `*.html` → ficheros acabados en `.html`.
- `**.java` → `.java` en cualquier subdirectorio.
- `src/**/*Test.java`.
- `foto?.{jpg,png}` → `foto1.jpg`, `fotoA.png`…

#### Combinar `walkFileTree` y `PathMatcher`

```java
public class Buscador extends SimpleFileVisitor<Path> {
    private final PathMatcher matcher;
    private int encontrados = 0;

    public Buscador(String patron) {
        this.matcher = FileSystems.getDefault().getPathMatcher("glob:" + patron);
    }

    @Override
    public FileVisitResult visitFile(Path fichero, BasicFileAttributes attrs) {
        Path nombre = fichero.getFileName();
        if (nombre != null && matcher.matches(nombre)) {
            System.out.println(fichero);
            encontrados++;
        }
        return FileVisitResult.CONTINUE;
    }

    public static void main(String[] args) throws IOException {
        Buscador b = new Buscador("*.java");
        Files.walkFileTree(Path.of("."), b);
        System.out.println("Encontrados: " + b.encontrados);
    }
}
```

#### Alternativa: `Files.find`

```java
try (Stream<Path> s = Files.find(Path.of("."), 10,
        (ruta, attrs) -> attrs.isRegularFile() && ruta.toString().endsWith(".txt"))) {
    s.forEach(System.out::println);
}
```

---

### 10.16 Vigilar cambios en un directorio

El **servicio de vigilancia** (`WatchService`) permite que tu programa sea avisado cuando se **crean**, **modifican** o **eliminan** ficheros en uno o varios directorios. Evita tener que consultar continuamente (*polling*).

#### Pasos

1. Crear un `WatchService`.
2. Registrar cada directorio, indicando los tipos de eventos.
3. Esperar eventos en un bucle (`take()` bloquea hasta que llegue alguno).
4. Procesar los eventos.
5. Llamar a `reset()` sobre la clave para seguir recibiendo avisos.

```java
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;

public class Vigilante {
    public static void main(String[] args) throws IOException, InterruptedException {
        Path dir = Path.of("vigilado");

        try (WatchService servicio = FileSystems.getDefault().newWatchService()) {
            dir.register(servicio, ENTRY_CREATE, ENTRY_MODIFY, ENTRY_DELETE);
            System.out.println("Vigilando " + dir.toAbsolutePath() + " ...");

            while (true) {
                WatchKey clave = servicio.take();          // bloquea hasta que haya eventos

                for (WatchEvent<?> evento : clave.pollEvents()) {
                    WatchEvent.Kind<?> tipo = evento.kind();

                    if (tipo == OVERFLOW) continue;        // se perdieron eventos

                    @SuppressWarnings("unchecked")
                    Path nombre = ((WatchEvent<Path>) evento).context();
                    System.out.println(tipo.name() + ": " + nombre);
                }

                if (!clave.reset()) {                      // el directorio dejó de ser accesible
                    break;
                }
            }
        }
    }
}
```

#### Aspectos a tener en cuenta

- Los eventos son de ficheros **dentro** del directorio, **no recursivos**: para vigilar subdirectorios hay que registrarlos uno a uno.
- `ENTRY_MODIFY` puede dispararse **varias veces** para una sola edición (por ejemplo, al guardar un fichero grande).
- Alternativas a `take()`: `poll()` (devuelve `null` si no hay eventos) y `poll(tiempo, unidad)` (espera como máximo ese tiempo).
- Es ideal para recargar configuraciones, procesar ficheros que "caen" en una carpeta, etc.
- El comportamiento exacto depende del sistema operativo.

---

### 10.17 Otros métodos útiles

#### Tipo de contenido (MIME)

```java
String tipo = Files.probeContentType(Path.of("foto.jpg"));   // "image/jpeg" (puede ser null)
```

#### Comparar el contenido de dos ficheros

```java
long pos = Files.mismatch(Path.of("a.txt"), Path.of("b.txt"));   // -1 si son idénticos
```

#### El sistema de ficheros por defecto

```java
FileSystem fs = FileSystems.getDefault();
System.out.println(fs.getSeparator());            // "/" o "\"
```

#### Sistemas de ficheros alternativos: ZIP

NIO.2 permite tratar un fichero ZIP como un sistema de ficheros más:

```java
Path zip = Path.of("archivo.zip");
try (FileSystem zipfs = FileSystems.newFileSystem(zip)) {
    Path dentro = zipfs.getPath("/docs/leeme.txt");
    System.out.println(Files.readString(dentro));
}
```

#### Directorio de trabajo y directorio personal

```java
System.getProperty("user.dir");    // directorio actual
System.getProperty("user.home");   // directorio personal del usuario
System.getProperty("java.io.tmpdir");   // directorio temporal
```

---

### 10.18 Código heredado con `java.io.File`

Antes de Java 7, la clase `java.io.File` era la única manera de representar ficheros. Todavía la encontrarás en mucho código antiguo. Sus principales inconvenientes:

- Muchos métodos devuelven `false` sin indicar la causa del error.
- `rename` no funciona de forma consistente entre plataformas.
- Soporte limitado de enlaces, metadatos y permisos.
- No escala bien con directorios grandes.

#### Conversión entre ambos mundos

```java
File archivo = new File("datos.txt");
Path ruta = archivo.toPath();         // File → Path
File de_nuevo = ruta.toFile();        // Path → File
```

Esto te permite migrar de forma gradual: acepta `File` en tu API antigua, conviértelo a `Path` y trabaja con NIO.2.

#### Tabla de equivalencias

| `java.io.File` | `java.nio.file` |
|---|---|
| `new File("x")` | `Path.of("x")` / `Paths.get("x")` |
| `file.exists()` | `Files.exists(path)` |
| `file.isDirectory()` | `Files.isDirectory(path)` |
| `file.isFile()` | `Files.isRegularFile(path)` |
| `file.canRead()` / `canWrite()` / `canExecute()` | `Files.isReadable()` / `isWritable()` / `isExecutable()` |
| `file.length()` | `Files.size(path)` |
| `file.lastModified()` | `Files.getLastModifiedTime(path)` |
| `file.delete()` | `Files.delete(path)` / `deleteIfExists` |
| `file.renameTo(dest)` | `Files.move(path, dest)` |
| `file.mkdir()` | `Files.createDirectory(path)` |
| `file.mkdirs()` | `Files.createDirectories(path)` |
| `file.createNewFile()` | `Files.createFile(path)` |
| `file.list()` / `listFiles()` | `Files.newDirectoryStream(path)` / `Files.list(path)` |
| `file.getName()` | `path.getFileName()` |
| `file.getParent()` / `getParentFile()` | `path.getParent()` |
| `file.getAbsolutePath()` | `path.toAbsolutePath()` |
| `file.getCanonicalPath()` | `path.toRealPath()` |
| `file.isHidden()` | `Files.isHidden(path)` |
| `File.createTempFile(...)` | `Files.createTempFile(...)` |
| `file.toURI()` | `path.toUri()` |

> 💡 **Recomendación:** en código nuevo, usa siempre `Path` y `Files`.

---

### 10.19 E/S y concurrencia: hilos virtuales (Java 21)

Desde Java 21 ([JEP 444](https://openjdk.org/jeps/444)) los **hilos virtuales** son una característica definitiva. Son hilos muy ligeros que permiten crear, por ejemplo, un hilo por tarea sin preocuparse por el coste. Se crean con `Thread.ofVirtual()` o con `Executors.newVirtualThreadPerTaskExecutor()`.

Ejemplo: contar las líneas de varios ficheros en paralelo.

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;

public class ContarEnParalelo {
    public static void main(String[] args) throws Exception {
        var ficheros = List.of(Path.of("a.txt"), Path.of("b.txt"), Path.of("c.txt"));

        try (var ejecutor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<Long>> tareas = new ArrayList<>();
            for (Path f : ficheros) {
                tareas.add(ejecutor.submit(() -> {
                    try (var lineas = Files.lines(f)) {
                        return lineas.count();
                    }
                }));
            }
            for (int i = 0; i < ficheros.size(); i++) {
                System.out.println(ficheros.get(i) + ": " + tareas.get(i).get() + " líneas");
            }
        }   // el ejecutor se cierra y espera a que terminen las tareas
    }
}
```

Qué debes saber:

- Los hilos virtuales brillan con **E/S de red bloqueante** (sockets, HTTP): cuando el hilo virtual espera, se "desmonta" y libera el hilo de plataforma.
- Con **ficheros**, las operaciones del sistema operativo suelen bloquear el hilo de plataforma subyacente (el JDK lo compensa en parte), por lo que el beneficio es menor. Mide antes de asumir que irá más rápido.
- Sigues escribiendo código **síncrono y secuencial** (`read`, `write`, `get`), sin *callbacks*.
- No es necesario reutilizar (*poolear*) hilos virtuales: se crean y se descartan.

---

## 11. Resumen

- Un **flujo (stream)** es una secuencia de datos que va de un origen a un destino. Puede ser de **entrada** o de **salida**.
- Los **flujos de bytes** (`InputStream`/`OutputStream`) manejan datos binarios; los **de caracteres** (`Reader`/`Writer`) manejan texto y convierten según un *charset*.
- Los **flujos con búfer** reducen el número de accesos al dispositivo y mejoran mucho el rendimiento.
- `Scanner` divide la entrada en *tokens* y `printf`/`String.format` dan formato a la salida. Cuidado con el **`Locale`**.
- Desde consola disponemos de `System.in`, `System.out`, `System.err` y la clase `Console` (en Java 21 es `null` sin terminal interactivo; desde Java 22 se usa `isTerminal()`).
- Los **flujos de datos** guardan primitivos y `String`; los **de objetos** guardan objetos completos mediante **serialización** (`Serializable`, `transient`, `serialVersionUID`). Los `record` también se serializan y los **filtros** (`ObjectInputFilter`) reducen el riesgo al deserializar.
- **NIO.2** (`java.nio.file`) es la API moderna de ficheros: `Path` representa rutas y `Files` aporta operaciones (comprobar, copiar, mover, borrar, leer, escribir, atributos…).
- Para recorrer o buscar: `DirectoryStream`, `Files.walk`, `Files.walkFileTree`, `PathMatcher`. Para reaccionar a cambios: `WatchService`.
- `java.io.File` sigue existiendo por compatibilidad, pero se puede convertir con `toPath()`/`toFile()`.
- **Siempre** se cierran los recursos, preferentemente con `try-with-resources`.
- Desde Java 18 la codificación por defecto es **UTF-8**; aun así, indícala explícitamente cuando sea relevante.
- Los **hilos virtuales** (Java 21) simplifican la E/S concurrente, sobre todo la de red.

### ¿Qué clase elijo? Guía rápida

| Quiero… | Uso |
|---|---|
| Leer un fichero de texto pequeño completo | `Files.readString` / `Files.readAllLines` |
| Leer un fichero de texto grande línea a línea | `Files.newBufferedReader` o `Files.lines` |
| Escribir texto | `Files.writeString` / `Files.newBufferedWriter` |
| Copiar un fichero binario grande | `Files.copy` |
| Leer números/palabras de la entrada | `Scanner` |
| Dar formato a la salida | `printf` / `String.format` |
| Guardar primitivos en binario | `DataOutputStream` |
| Guardar objetos Java | `ObjectOutputStream` (con precaución) o JSON |
| Saltar a posiciones concretas | `SeekableByteChannel` / `RandomAccessFile` |
| Recorrer un árbol de directorios | `Files.walk` / `Files.walkFileTree` |
| Reaccionar a cambios en una carpeta | `WatchService` |

---

## 12. Errores frecuentes y buenas prácticas

### Errores típicos de principiante

1. **No cerrar los flujos.** → Usa `try-with-resources`.
2. **Capturar la excepción y no hacer nada** (`catch (IOException e) {}`) → Como mínimo, regístrala o muéstrala; mejor, trátala o propágala.
3. **Usar rutas con `\` escritas a mano.** En Java, el `\` es carácter de escape: `"C:\datos"` es un error de compilación (o `\d` inválido). Escribe `"C:\\datos"`, usa `/` (Java lo admite en Windows) o, mejor aún, `Path.of("C:", "datos")`.
4. **Dar por hecha la codificación.** Aunque desde Java 18 el valor por defecto es UTF-8, un fichero heredado puede estar en ISO-8859-1 o Windows-1252. → Indica el `charset` explícitamente.
5. **Confundir bytes leídos con tamaño del búfer** al copiar: usa `write(buf, 0, leidos)`.
6. **Leer un fichero enorme con `readAllBytes`/`readAllLines`.** → Procesa por líneas o por bloques.
7. **`nextInt()` seguido de `nextLine()`** en `Scanner` (el salto de línea sobrante).
8. **Suponer que `System.console()` no es `null`** (en Java 21 lo es sin terminal interactivo) o **no comprobar `isTerminal()`** en Java 22+.
9. **Deserializar datos no confiables**, o hacerlo sin un `ObjectInputFilter`.
10. **Comprobar con `exists()` y luego actuar** (condición de carrera) en lugar de capturar la excepción.
11. **Rutas relativas y directorio de trabajo:** el IDE y la consola pueden tener directorios de trabajo diferentes. Si algo "no se encuentra", imprime `Path.of("x").toAbsolutePath()` para ver dónde está buscando realmente.
12. **No cerrar el `Stream` de `Files.lines`, `Files.list`, `Files.walk`.** Mantienen recursos abiertos.
13. **Usar el constructor `new URL(String)`**, obsoleto desde Java 20. → Usa `URI.create(...).toURL()` o, mejor, `HttpClient`.
14. **Esperar que los hilos virtuales aceleren por sí solos la E/S de ficheros.** → Mide antes de decidir.

### Buenas prácticas

- ✅ Prefiere **NIO.2** (`Path`/`Files`) en código nuevo.
- ✅ Usa **búfer** para volúmenes de datos grandes.
- ✅ Especifica la **codificación** de los textos (aunque UTF-8 sea el valor por defecto desde Java 18).
- ✅ Usa `try-with-resources` y captura excepciones **específicas** (`NoSuchFileException`, etc.) antes que la genérica `IOException`.
- ✅ Usa `%n` en lugar de `\n` en `printf`.
- ✅ Fija el `Locale` cuando leas o formatees números.
- ✅ Para escritura crítica, considera escribir en un fichero temporal y luego `Files.move` con `ATOMIC_MOVE` para evitar dejar un fichero a medias.
- ✅ No hardcodees rutas absolutas: usa parámetros, ficheros de configuración o `user.home`.
- ✅ Separa la lógica de negocio de la E/S para facilitar las pruebas.

---

## 13. Preguntas y ejercicios

### Preguntas de repaso

1. ¿Qué diferencia hay entre `InputStream` y `Reader`?
2. ¿Por qué `InputStream.read()` devuelve un `int` y no un `byte`?
3. ¿Qué ventaja tiene un flujo con búfer? ¿Cuándo hay que llamar a `flush()`?
4. ¿Qué pasa si no cierras un `FileOutputStream`?
5. ¿Qué hace la palabra reservada `transient` en una clase serializable?
6. ¿Qué excepción indica el final del fichero al usar `DataInputStream`?
7. ¿Qué diferencia hay entre `Path.resolve` y `Path.relativize`?
8. ¿Por qué `Files.exists` y `Files.notExists` pueden devolver ambos `false`?
9. ¿Qué diferencia hay entre `Files.walk` y `Files.walkFileTree`?
10. ¿Qué ventajas tiene `Path` frente a `File`?

<details>
<summary>Respuestas breves</summary>

1. `InputStream` lee bytes; `Reader` lee caracteres y aplica una codificación.
2. Para poder devolver -1 como señal de fin de datos, distinguiéndolo de un byte válido (0–255).
3. Reduce los accesos al dispositivo. Se llama a `flush()` cuando se necesita que los datos salgan ya (antes de cerrar), por ejemplo en comunicaciones interactivas.
4. Puedes perder datos aún en el búfer y dejar recursos del sistema retenidos (fichero bloqueado).
5. Excluye ese atributo del proceso de serialización.
6. `EOFException`.
7. `resolve` une dos rutas (añade una a otra); `relativize` calcula la ruta relativa para llegar de una a otra.
8. Porque puede ser imposible determinar si existe (por falta de permisos, por ejemplo); en ese caso ninguno de los dos es verdadero.
9. `walk` devuelve un `Stream<Path>` y es más simple; `walkFileTree` usa un visitante con control fino (saltar subárboles, acciones antes/después de directorios, manejo de errores).
10. Mejor gestión de errores (excepciones descriptivas), enlaces simbólicos, atributos, rendimiento y funcionalidades como `WatchService`.
</details>

### Ejercicios prácticos

**Nivel básico**

1. **Contador de líneas.** Escribe un programa que reciba por argumento la ruta de un fichero de texto y muestre cuántas líneas, palabras y caracteres tiene.
2. **Copia con búfer.** Copia un fichero cualquiera (binario) a otro usando `FileInputStream`, `FileOutputStream` y un array de bytes. Compara el tiempo con la versión que copia byte a byte.
3. **Agenda sencilla.** Pide por teclado nombre y teléfono de varias personas con `Scanner` y guárdalos en un fichero de texto, uno por línea (`nombre;teléfono`). Después léelos y muéstralos con `printf` en forma de tabla.
4. **Suma de números.** Lee de un fichero números (uno por línea) e informa de la suma y la media. Ignora las líneas que no sean números.

**Nivel intermedio**

5. **Inventario binario.** Con `DataOutputStream` / `DataInputStream` guarda y recupera un inventario de productos (código, nombre, stock, precio). Añade una opción para listar productos con stock inferior a un valor dado.
6. **Serialización.** Crea una clase `Libro` (título, autor, año, ISBN) y una clase `Biblioteca` con una lista de libros. Serialízala a disco y recupérala. Marca algún atributo como `transient` y comprueba el efecto.
7. **Registros de acceso aleatorio.** Con `RandomAccessFile` guarda registros de tamaño fijo (por ejemplo, 4 bytes de `int` + 20 caracteres + 8 bytes de `double`). Implementa funciones para leer y modificar el registro *n* sin recorrer los anteriores.
8. **Copia de directorio.** Implementa un método que copie recursivamente un directorio completo con `Files.walkFileTree`.

**Nivel avanzado**

9. **Buscador de duplicados.** Recorre un directorio y detecta ficheros duplicados (mismo tamaño y mismo contenido; usa `Files.size` primero y `Files.mismatch` después).
10. **Registro de cambios.** Con `WatchService`, vigila una carpeta e imprime en un fichero `cambios.log` (con fecha y hora) cada fichero creado, modificado o borrado.
11. **Mini-`grep`.** Programa que reciba un texto y una carpeta, y muestre todas las líneas de todos los `.txt` y `.java` que contengan ese texto, indicando fichero y número de línea.
12. **Copia de seguridad atómica.** Escribe una utilidad que guarde un fichero de configuración escribiendo primero en un fichero temporal en el mismo directorio y finalmente lo renombre con `ATOMIC_MOVE`.
13. **Serialización segura.** Define un `record Contacto(String nombre, String email)` serializable con validación en el constructor compacto. Guárdalo y léelo usando un `ObjectInputFilter` que solo admita esa clase y `java.util.ArrayList`. Comprueba qué ocurre si intentas leer otro tipo de objeto.
14. **Análisis concurrente.** Usando `Executors.newVirtualThreadPerTaskExecutor()`, cuenta las palabras de todos los `.txt` de un directorio en paralelo y muestra el total. Compara el tiempo con la versión secuencial.

### Pistas para el ejercicio 1 (esqueleto)

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;

public class Contador {
    public static void main(String[] args) {
        if (args.length != 1) {
            System.err.println("Uso: java Contador <fichero>");
            return;
        }

        long lineas = 0, palabras = 0, caracteres = 0;

        try (BufferedReader br = Files.newBufferedReader(Path.of(args[0]), StandardCharsets.UTF_8)) {
            String linea;
            while ((linea = br.readLine()) != null) {
                lineas++;
                caracteres += linea.length();
                String limpia = linea.trim();
                if (!limpia.isEmpty()) {
                    palabras += limpia.split("\\s+").length;
                }
            }
            System.out.printf("Líneas: %d, palabras: %d, caracteres: %d%n",
                              lineas, palabras, caracteres);
        } catch (NoSuchFileException e) {
            System.err.println("No existe el fichero: " + e.getFile());
        } catch (IOException e) {
            System.err.println("Error de lectura: " + e.getMessage());
        }
    }
}
```

---

## 14. Para seguir aprendiendo

- Tutorial original (en inglés, escrito para JDK 8): <https://docs.oracle.com/javase/tutorial/essential/io/index.html>
- Tutoriales actualizados de Java: <https://dev.java/learn/>
- Documentación de la API de Java 21: `java.io`, `java.nio.file` y `java.nio.channels` en la [Javadoc de Java SE 21](https://docs.oracle.com/en/java/javase/21/docs/api/index.html).
- JEP relacionadas: [JEP 400 (UTF-8 por defecto)](https://openjdk.org/jeps/400), [JEP 444 (hilos virtuales)](https://openjdk.org/jeps/444), [JEP 512 (clase `IO`)](https://openjdk.org/jeps/512).
- Temas relacionados que conviene estudiar a continuación:
  - Excepciones y *try-with-resources* en profundidad.
  - Expresiones regulares (`java.util.regex`).
  - API `Stream` y programación funcional.
  - Redes (`java.net`, `HttpClient`), que usa los mismos flujos de E/S.
  - Formatos de intercambio de datos: JSON (Jackson, Gson), CSV, XML.
  - Persistencia con bases de datos (JDBC, JPA).
  - E/S asíncrona y no bloqueante (`java.nio.channels`, `AsynchronousFileChannel`).

---

*Documento elaborado con fines didácticos a partir de la estructura de la lección «Basic I/O» de los Java Tutorials y actualizado para Java 21 o superior. «Java» y «Oracle» son marcas registradas de Oracle y/o sus filiales.*
