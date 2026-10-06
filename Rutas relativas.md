# Rutas relativas en Java (Java 21+)

> **Fuente de referencia:** artículo «Ruta relativa en Java», DelftStack.
> **Destinatario:** programador Java.
> **Versión objetivo:** Java 21 (LTS) y posteriores.

---

## 1. ¿Qué es una ruta relativa?

Una **ruta relativa** es una ruta incompleta: no empieza en la raíz del sistema de archivos. Se interpreta **a partir de un directorio de referencia**, que normalmente es el **directorio de trabajo actual** del proceso.

| Tipo | Ejemplo en Linux/macOS | Ejemplo en Windows |
|---|---|---|
| **Absoluta** | `/home/ana/proyecto/datos/registro.txt` | `C:\Users\ana\proyecto\datos\registro.txt` |
| **Relativa** | `datos/registro.txt` | `datos\registro.txt` |

Usamos rutas relativas para localizar archivos en el directorio actual, en un directorio padre o dentro de la misma jerarquía de carpetas del proyecto. Así el programa no depende de dónde esté instalado.

---

## 2. Notación especial

| Símbolo | Significado |
|---|---|
| `.` o `./` | Directorio **actual** |
| `..` o `../` | Directorio **padre** (un nivel por encima) |
| `../../` | Dos niveles por encima |
| `/` (al inicio) | Raíz del sistema (ruta **absoluta** en Unix) |

Ejemplo de estructura:

```text
proyecto/
├── src/
│   └── Main.java
├── datos/
│   └── registro.txt
└── config/
    └── app.properties
```

Si el directorio de trabajo es `proyecto/src`:

- `Main.java` o `./Main.java` → el fichero en el directorio actual.
- `../datos/registro.txt` → sube a `proyecto` y entra en `datos`.
- `../../otra-carpeta/x.txt` → sube dos niveles, hasta el padre de `proyecto`.

---

## 3. ¿Cuál es el «directorio actual»?

Es el directorio desde el que se **lanzó la JVM**, **no** el directorio donde está el archivo `.java` ni el `.class`. Java lo guarda en la propiedad de sistema `user.dir`.

```java
public class DirectorioActual {
    public static void main(String[] args) {
        System.out.println(System.getProperty("user.dir"));
    }
}
```

> **Error muy frecuente en principiantes:** el programa funciona al ejecutarlo desde el IDE y falla desde la terminal (o al revés), porque el directorio de trabajo es distinto. Comprueba siempre `user.dir`.

---

## 4. Ejemplos con `java.io.File`

Los ejemplos del artículo original usan la clase clásica `File`.

### 4.1. Ruta relativa a un archivo en el directorio actual

```java
import java.io.File;

public class RutaRelativaSimple {
    public static void main(String[] args) {
        String rutaArchivo = "files/record.txt";
        File archivo = new File(rutaArchivo);
        System.out.println(archivo.getPath());
    }
}
```

Salida:

```text
files/record.txt
```

### 4.2. Directorio padre con `../`

```java
import java.io.File;

public class RutaDirectorioPadre {
    public static void main(String[] args) {
        String rutaArchivo = "../files/record.txt";
        File archivo = new File(rutaArchivo);
        System.out.println(archivo.getPath());
    }
}
```

Salida:

```text
../files/record.txt
```

### 4.3. Directorio actual con `./`

```java
import java.io.File;

public class RutaDirectorioActual {
    public static void main(String[] args) {
        String rutaArchivo = "./data-files/record.txt";
        File archivo = new File(rutaArchivo);
        System.out.println(archivo.getPath());
    }
}
```

Salida:

```text
./data-files/record.txt
```

### 4.4. Dos niveles superiores con `../../`

```java
import java.io.File;

public class RutaDosNiveles {
    public static void main(String[] args) {
        String rutaArchivo = "../../data-files/record.txt";
        File archivo = new File(rutaArchivo);
        System.out.println(archivo.getPath());          // ruta tal cual se escribió
        System.out.println(archivo.getAbsolutePath());  // ruta resuelta con el directorio actual
    }
}
```

Salida de la primera línea:

```text
../../data-files/record.txt
```

La segunda línea depende de tu directorio de trabajo. Por ejemplo, desde `/home/ana/proyecto/src` sería:

```text
/home/ana/proyecto/src/../../data-files/record.txt
```

### 4.5. Qué demuestran estos ejemplos

**`getPath()` no comprueba nada: devuelve el texto de la ruta tal como la escribiste.** Crear un objeto `File` **no** comprueba que el archivo exista ni lo abre. Para saberlo:

```java
File f = new File("datos/registro.txt");
System.out.println(f.exists());      // ¿existe?
System.out.println(f.isFile());      // ¿es un fichero?
System.out.println(f.isDirectory()); // ¿es un directorio?
```

---

## 5. La forma moderna: `java.nio.file.Path` (recomendada)

Desde Java 7, la API NIO.2 (`java.nio.file`) sustituye a `File` en código nuevo. Es más potente y segura. Desde Java 11 existe `Path.of(...)`, que es la forma preferida; `Paths.get(...)` sigue funcionando.

```java
import java.nio.file.Path;

Path relativa = Path.of("datos", "registro.txt");   // construye según el SO (separador correcto)
Path padre    = Path.of("..", "datos", "registro.txt");

System.out.println(relativa);                // datos/registro.txt  (o datos\registro.txt en Windows)
System.out.println(relativa.isAbsolute());   // false
```

### 5.1. Métodos útiles

| Método | Qué hace |
|---|---|
| `toAbsolutePath()` | Convierte a absoluta usando el directorio de trabajo |
| `normalize()` | Elimina `.` y `..` redundantes |
| `toRealPath()` | Resuelve enlaces simbólicos y comprueba que existe (lanza `IOException` si no) |
| `resolve(otra)` | Une dos rutas: `base.resolve("x.txt")` |
| `resolveSibling(otra)` | Resuelve respecto al directorio padre |
| `relativize(otra)` | Calcula la ruta relativa de una ruta a otra |
| `getParent()` | Directorio padre (puede ser `null`) |
| `getFileName()` | Último elemento de la ruta |
| `startsWith(...)` | Comprueba si empieza por otra ruta |

### 5.2. Ejemplo completo

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class EjemploPath {
    public static void main(String[] args) throws IOException {
        Path ruta = Path.of("../../data-files/record.txt");

        System.out.println("Original:   " + ruta);
        System.out.println("Absoluta:   " + ruta.toAbsolutePath());
        System.out.println("Normalizada:" + ruta.toAbsolutePath().normalize());
        System.out.println("¿Existe?    " + Files.exists(ruta));
    }
}
```

Fíjate en la diferencia: `toAbsolutePath()` deja los `..` en la ruta, mientras que `normalize()` los resuelve para producir una ruta limpia.

### 5.3. Leer y escribir con `Files`

```java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

public class LeerArchivo {
    public static void main(String[] args) {
        Path ruta = Path.of("datos", "registro.txt");

        try {
            List<String> lineas = Files.readAllLines(ruta, StandardCharsets.UTF_8);
            lineas.forEach(System.out::println);

            String contenido = Files.readString(ruta); // Java 11+
        } catch (IOException e) {
            System.err.println("No se pudo leer " + ruta.toAbsolutePath() + ": " + e.getMessage());
        }
    }
}
```

Truco útil: en el mensaje de error imprime `toAbsolutePath()`. Así verás exactamente **dónde** ha buscado Java.

### 5.4. Convertir entre `File` y `Path`

```java
File f = new File("datos/registro.txt");
Path p = f.toPath();

Path p2 = Path.of("datos/registro.txt");
File f2 = p2.toFile();
```

---

## 6. Relativa a la raíz del proyecto frente a relativa al *classpath*

Hay dos conceptos distintos que se confunden mucho:

| | Ruta de **sistema de archivos** | Recurso del **classpath** |
|---|---|---|
| Se resuelve respecto a | Directorio de trabajo (`user.dir`) | Raíz del *classpath* (JAR o carpeta de clases) |
| API | `File`, `Path`, `Files` | `Class.getResourceAsStream(...)` |
| Funciona dentro de un JAR | No (el recurso no es un fichero real) | Sí |
| Típico uso | Ficheros externos, datos, logs | Ficheros dentro de `src/main/resources` |

Para ficheros que van **empaquetados en tu aplicación** (por ejemplo `src/main/resources/config.properties`), no uses `new File(...)`. Usa el *classpath*:

```java
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class LeerRecurso {
    public static void main(String[] args) throws IOException {
        try (InputStream in = LeerRecurso.class.getResourceAsStream("/config.properties")) {
            if (in == null) {
                throw new IllegalStateException("No se encuentra config.properties en el classpath");
            }
            Properties props = new Properties();
            props.load(in);
            System.out.println(props.getProperty("nombre"));
        }
    }
}
```

- Con `/` inicial, la ruta se resuelve desde la raíz del *classpath*.
- Sin `/`, se resuelve respecto al paquete de la clase.

---

## 7. Diferencias entre sistemas operativos

- **Separador:** `/` en Linux y macOS; `\` en Windows. Java acepta `/` también en Windows.
- **No escribas el separador a mano.** Usa `Path.of("datos", "registro.txt")` o `File.separator`.
- **Mayúsculas/minúsculas:** en Linux `Datos` y `datos` son carpetas distintas; en Windows y por defecto en macOS no.

```java
// Evita:
String mala = "datos\\registro.txt";   // solo funciona en Windows

// Prefiere:
Path buena = Path.of("datos", "registro.txt");
```

---

## 8. Seguridad: cuidado con `..` (*Path Traversal*)

Si construyes una ruta a partir de datos que introduce el usuario (nombre de fichero, parámetro HTTP…), un atacante podría enviar algo como `../../etc/passwd` para salir de la carpeta permitida. Esto es una vulnerabilidad conocida como *path traversal*.

Defensa recomendada: normalizar y comprobar que sigue dentro del directorio base.

```java
import java.nio.file.Path;

public class RutaSegura {
    private static final Path BASE = Path.of("datos").toAbsolutePath().normalize();

    static Path resolverSeguro(String nombreFichero) {
        Path resultado = BASE.resolve(nombreFichero).normalize();
        if (!resultado.startsWith(BASE)) {
            throw new SecurityException("Ruta fuera del directorio permitido: " + nombreFichero);
        }
        return resultado;
    }

    public static void main(String[] args) {
        System.out.println(resolverSeguro("registro.txt"));      // OK
        System.out.println(resolverSeguro("../../etc/passwd"));  // lanza SecurityException
    }
}
```

Para una protección aún más completa (enlaces simbólicos) usa `toRealPath()` sobre ambas rutas antes de comparar.

---

## 9. Errores comunes y cómo evitarlos

| Síntoma | Causa probable | Solución |
|---|---|---|
| `NoSuchFileException` / `FileNotFoundException` | El directorio de trabajo no es el que crees | Imprime `System.getProperty("user.dir")` y `ruta.toAbsolutePath()` |
| Funciona en el IDE pero no en el JAR | Usas `File` para un recurso interno | Usa `getResourceAsStream` |
| Funciona en Windows pero falla en Linux | Separadores `\` o mayúsculas/minúsculas | Usa `Path.of(...)` y respeta el nombre exacto |
| `getParent()` devuelve `null` | La ruta relativa solo tiene un elemento | Usa `toAbsolutePath().getParent()` |
| La ruta contiene `..` extraños | No está normalizada | Usa `normalize()` |

---

## 10. Resumen

1. Una ruta relativa se interpreta respecto al **directorio de trabajo** (`user.dir`), no respecto al código fuente.
2. `./` es el directorio actual, `../` el padre, `../../` dos niveles por encima.
3. `new File("ruta")` **no comprueba** si el archivo existe; `getPath()` devuelve el texto tal cual.
4. En código nuevo, prefiere **`Path.of(...)`** y **`Files`** frente a `File`.
5. Usa `toAbsolutePath()` y `normalize()` para depurar y para ver la ruta real.
6. Para recursos dentro del JAR, usa **`getResourceAsStream`**.
7. Nunca construyas rutas con datos del usuario sin **normalizar y validar**.
