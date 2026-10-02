# Excepciones checked y unchecked en Java

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
private static void checkedExceptionWithThrows() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```

También se puede gestionar una checked exception mediante un bloque `try-catch`:

```java
private static void checkedExceptionWithTryCatch() {
    File file = new File("not_existing_file.txt");
    try {
        FileInputStream stream = new FileInputStream(file);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

Algunos ejemplos habituales de checked exceptions en Java son:

- `IOException`
- `SQLException`
- `ParseException`

La clase `Exception` es la superclase de las checked exceptions. Por tanto, se puede crear una checked exception personalizada extendiendo `Exception`:

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

La clase `RuntimeException` es la superclase de todas las unchecked exceptions. Por tanto, se puede crear una unchecked exception personalizada extendiendo `RuntimeException`:

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
| Superclase | `Exception` | `RuntimeException` |
| Ejemplo | `FileNotFoundException` | `ArithmeticException` |

## 6. Conclusión

Las checked exceptions y las unchecked exceptions son las dos categorías principales de excepciones en Java.

Las **checked exceptions** se verifican durante la compilación y deben declararse o gestionarse. El artículo las relaciona con situaciones en las que el cliente puede recuperarse de la excepción.

Las **unchecked exceptions** no se verifican durante la compilación y no es necesario declararlas mediante `throws`. El artículo las relaciona con errores que se producen dentro de la lógica del programa y de los que el cliente no puede recuperarse.

La decisión entre ambos tipos se basa, por tanto, en si el código que recibe la excepción puede recuperarse razonablemente de ella.

