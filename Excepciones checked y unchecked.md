# Excepciones checked y unchecked en Java

> Versión de referencia: **Java 25 (LTS)** o superior. Los ejemplos se han compilado con OpenJDK 25.

## 1. Introducción

Las excepciones de Java se dividen en dos categorías principales:

- **Checked exceptions** (excepciones comprobadas).
- **Unchecked exceptions** (excepciones no comprobadas).

## 2. Checked exceptions

Las **checked exceptions** representan, en general, errores que están fuera del control del programa.

Por ejemplo, el constructor de `FileInputStream` lanza `FileNotFoundException` si el archivo de entrada no existe.

Java verifica las checked exceptions **en tiempo de compilación**.

Por ello, se puede declarar una checked exception mediante la palabra clave `throws`:

```java
private static void checkedExceptionWithThrows() throws IOException {
    try (var stream = new FileInputStream("not_existing_file.txt")) {
        // leer del fichero...
    }
}
```

> `try (...)` es un **try-with-resources**: cierra el fichero automáticamente al terminar (ver [sección 6.1](#61-try-with-resources-java-7)). Como `close()` también puede lanzar `IOException`, se declara `IOException`, que incluye a `FileNotFoundException`.

También se puede gestionar una checked exception mediante un bloque `try-catch`:

```java
private static void checkedExceptionWithTryCatch() {
    try (var stream = new FileInputStream("not_existing_file.txt")) {
        // leer del fichero...
    } catch (FileNotFoundException e) {
        System.err.println("No se encuentra el fichero: " + e.getMessage());
    } catch (IOException e) {
        System.err.println("Error al cerrar el fichero: " + e.getMessage());
    }
}
```

> En una aplicación real, en lugar de `System.err` o `e.printStackTrace()` se usa un sistema de *logging* (ver [Log4j](<Log4J: Guia.md>)). Y en código nuevo se suele preferir la API NIO.2 (`Files.newInputStream(Path.of(...))`), que lanza `NoSuchFileException`, también comprobada.

Algunos ejemplos habituales de checked exceptions en Java son:

- `IOException`
- `SQLException`
- `ParseException`

La clase `Exception` es la superclase de las checked exceptions (todas sus subclases son comprobadas, **salvo** `RuntimeException` y las suyas). Por tanto, se puede crear una checked exception personalizada extendiendo `Exception`:

```java
public class IncorrectFileNameException extends Exception {
    public IncorrectFileNameException(String errorMessage) {
        super(errorMessage);
    }
}
```

## 3. Unchecked exceptions

Si un programa lanza una unchecked exception, esto refleja algún error dentro de la lógica del programa.

Por ejemplo, si se divide un número entre cero, Java lanza `ArithmeticException`:

```java
private static void divideByZero() {
    int numerator = 1;
    int denominator = 0;
    int result = numerator / denominator;
}
```

Java no verifica las unchecked exceptions durante la compilación.

Además, no es necesario declarar las unchecked exceptions mediante la palabra clave `throws`.

El código anterior no produce un error durante la compilación, pero lanza `ArithmeticException` durante la ejecución.

Algunos ejemplos habituales de unchecked exceptions en Java son:

- `NullPointerException`
- `ArrayIndexOutOfBoundsException`
- `IllegalArgumentException`

La clase `RuntimeException` es la superclase de las unchecked exceptions que se usan en las aplicaciones (la otra rama no comprobada es `Error`, ver [sección 6.6](#66-error-la-tercera-rama)). Por tanto, se puede crear una unchecked exception personalizada extendiendo `RuntimeException`:

```java
public class NullOrEmptyException extends RuntimeException {
    public NullOrEmptyException(String errorMessage) {
        super(errorMessage);
    }
}
```

## 4. Cuándo utilizar checked exceptions y unchecked exceptions

El uso de excepciones permite separar el código encargado de gestionar los errores del código normal.

Es necesario decidir qué tipo de excepción utilizar.

El criterio indicado en el artículo es:

- Si se puede esperar razonablemente que el cliente pueda recuperarse de una excepción, se debe utilizar una **checked exception**.
- Si el cliente no puede hacer nada para recuperarse de la excepción, se debe utilizar una **unchecked exception**.

### Ejemplo de checked exception

Antes de abrir un archivo, se puede validar el nombre del archivo introducido.

Si el nombre introducido por el usuario no es válido, se puede lanzar una checked exception personalizada:

```java
if (!isCorrectFileName(fileName)) {
    throw new IncorrectFileNameException("Incorrect filename : " + fileName);
}
```

De esta forma, el sistema puede recuperarse aceptando otro nombre de archivo introducido por el usuario.

### Ejemplo de unchecked exception

Si el nombre del archivo es `null` o es una cadena vacía, significa que existe un error en el código.

En este caso, se debe lanzar una unchecked exception:

```java
if (fileName == null || fileName.isEmpty()) {
    throw new NullOrEmptyException("The filename is null or empty.");
}
```

## 5. Diferencias principales

| Característica | Checked exception | Unchecked exception |
|---|---|---|
| Comprobación | En tiempo de compilación | No se comprueba en tiempo de compilación |
| Declaración con `throws` | Necesaria cuando se propaga | No es necesaria |
| Situación descrita | Error fuera del control del programa | Error dentro de la lógica del programa |
| Superclase | `Exception` (salvo `RuntimeException`) | `RuntimeException` (y `Error`) |
| Ejemplo | `FileNotFoundException` | `ArithmeticException` |

## 6. Excepciones en Java moderno (Java 25)

Las reglas de checked y unchecked no han cambiado, pero el lenguaje ofrece herramientas que hacen el código de gestión de errores más claro.

### 6.1. `try-with-resources` (Java 7)

Cualquier recurso que implemente `AutoCloseable` (ficheros, conexiones, *streams*…) se cierra automáticamente al salir del bloque, incluso si se produce una excepción. Es la forma correcta de trabajar con ficheros: los ejemplos de la sección 2 la usan.

### 6.2. *Multi-catch* (Java 7)

Si varias excepciones se tratan igual, se capturan en un solo bloque con `|`:

```java
static String leerConfig(Path ruta) {
    try {
        return Files.readString(ruta);
    } catch (NoSuchFileException | AccessDeniedException e) {
        return "";                                                    // se puede continuar sin configuración
    } catch (IOException e) {
        throw new UncheckedIOException("No se pudo leer " + ruta, e);   // se envuelve conservando la causa
    }
}
```

Los bloques `catch` van de la excepción **más concreta a la más general**: si `IOException` fuese primero, el compilador daría error porque los otros nunca se alcanzarían.

### 6.3. Variables sin nombre en `catch` (Java 22)

Cuando no necesitas la excepción capturada, puedes llamarla `_` para dejar claro que se ignora a propósito:

```java
static int parseOrDefault(String text, int defaultValue) {
    try {
        return Integer.parseInt(text);
    } catch (NumberFormatException _) {
        return defaultValue;
    }
}
```

### 6.4. Encadenar excepciones

Al convertir una checked exception en unchecked (por ejemplo, `IOException` en `UncheckedIOException`), pasa siempre la excepción original como **causa**. Así no se pierde la traza del error real.

### 6.5. Checked exceptions y lambdas

Las interfaces funcionales estándar (`Function`, `Consumer`, `Supplier`…) no declaran `throws`, así que una lambda que llama a un método con checked exceptions no compila sin tratarlas dentro. Lo habitual es capturarla dentro de la lambda y envolverla en una unchecked exception (como en 6.2). La guía [Lambdas y Streams](<Lambdas y Streams.md>) lo explica con detalle.

### 6.6. `Error`: la tercera rama

Además de `RuntimeException`, la clase `Error` y sus subclases (`OutOfMemoryError`, `StackOverflowError`…) tampoco son comprobadas. Representan problemas graves de la JVM de los que una aplicación normal no debe intentar recuperarse, por lo que no se capturan.

## 7. Conclusión

Las checked exceptions y las unchecked exceptions son las dos categorías principales de excepciones en Java.

Las **checked exceptions** se verifican durante la compilación y deben declararse o gestionarse. El artículo las relaciona con situaciones en las que el cliente puede recuperarse de la excepción.

Las **unchecked exceptions** no se verifican durante la compilación y no es necesario declararlas mediante `throws`. El artículo las relaciona con errores que se producen dentro de la lógica del programa y de los que el cliente no puede recuperarse.

La decisión entre ambos tipos se basa, por tanto, en si el código que recibe la excepción puede recuperarse razonablemente de ella.

