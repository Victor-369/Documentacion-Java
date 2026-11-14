# Tests con Mockito

Guía práctica para escribir tests unitarios con **Mockito** y **JUnit 6** en proyectos con **Java 25 o superior**.

---

## Índice

1. [¿Qué es Mockito y para qué sirve?](#1-qué-es-mockito-y-para-qué-sirve)
2. [Conceptos básicos](#2-conceptos-básicos)
3. [Configuración del proyecto](#3-configuración-del-proyecto) (Maven, Gradle e IntelliJ IDEA)
4. [El código que vamos a probar](#4-el-código-que-vamos-a-probar)
5. [Tu primer test con Mockito](#5-tu-primer-test-con-mockito)
6. [Usando anotaciones](#6-usando-anotaciones)
7. [Stubbing: decidir qué devuelve un mock](#7-stubbing-decidir-qué-devuelve-un-mock)
8. [Argument matchers](#8-argument-matchers)
9. [Verificar interacciones con `verify`](#9-verificar-interacciones-con-verify)
10. [Capturar argumentos con `ArgumentCaptor`](#10-capturar-argumentos-con-argumentcaptor)
11. [Spies: objetos reales parcialmente simulados](#11-spies-objetos-reales-parcialmente-simulados)
12. [Simular métodos estáticos](#12-simular-métodos-estáticos)
13. [Estilo BDD: `given` / `then`](#13-estilo-bdd-given--then)
14. [Errores habituales y cómo resolverlos](#14-errores-habituales-y-cómo-resolverlos)
15. [Buenas prácticas](#15-buenas-prácticas)
16. [Chuleta rápida](#16-chuleta-rápida)

---

## 1. ¿Qué es Mockito y para qué sirve?

Cuando haces un **test unitario**, quieres probar **una sola clase** de forma aislada. El problema es que casi ninguna clase vive sola: suele depender de otras (un repositorio que accede a la base de datos, un cliente HTTP, un servicio de correo...).

Si en el test usas esas dependencias reales:

- El test es **lento** (accede a base de datos, red, disco...).
- El test es **frágil** (si la base de datos está caída, falla aunque tu código esté bien).
- Es **difícil provocar casos concretos** (por ejemplo, "¿qué pasa si el servidor devuelve un error?").

**Mockito** resuelve esto creando **objetos falsos** (llamados *mocks*) que sustituyen a esas dependencias. Tú decides qué devuelven y luego compruebas cómo se han usado.

> 💡 **Analogía:** imagina que estás ensayando una obra de teatro y falta un actor. En lugar de esperar, pones a alguien que lee sus frases del guion. Ese "sustituto" es el mock: no es el actor real, pero dice exactamente lo que necesitas para ensayar tu parte.

---

## 2. Conceptos básicos

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Mock** | Objeto falso que imita una clase o interfaz. Por defecto no hace nada y devuelve valores "vacíos" (`null`, `0`, `false`, listas vacías...). | `mock(UsuarioRepository.class)` |
| **Stub** | Instrucción que le das al mock para que devuelva algo concreto. | `when(repo.buscar(1L)).thenReturn(usuario)` |
| **Verify** | Comprobación de que un método del mock se ha llamado (o no), cuántas veces y con qué argumentos. | `verify(repo).guardar(usuario)` |
| **Spy** | Objeto **real** al que puedes sobrescribir solo algunos métodos. | `spy(new ArrayList<>())` |
| **SUT** | *System Under Test*: la clase que estás probando de verdad. | `UsuarioService` |

Un test típico sigue la estructura **AAA** (Arrange, Act, Assert), que en castellano sería:

1. **Preparar** → creas los mocks y defines su comportamiento.
2. **Ejecutar** → llamas al método que quieres probar.
3. **Comprobar** → revisas el resultado y/o las interacciones con los mocks.

---

## 3. Configuración del proyecto

### 3.1. Maven

#### ¿De dónde se obtiene la librería de Mockito?

**No hace falta descargar ningún `.jar` a mano.** Mockito (igual que JUnit) está publicado en **Maven Central**, el repositorio público de librerías Java que Maven usa por defecto. Basta con declarar la dependencia en el `pom.xml` y Maven la descarga automáticamente la primera vez que compilas o ejecutas los tests (`mvn test`), junto con sus dependencias transitivas (Byte Buddy, Objenesis...). Los ficheros quedan guardados en tu repositorio local, en `~/.m2/repository/org/mockito/`, y se reutilizan en todos tus proyectos.

Para encontrar las coordenadas (`groupId`, `artifactId` y `version`) y la última versión disponible:

1. Entra en **Maven Central**: <https://central.sonatype.com/> y busca `mockito-core` (o directamente <https://central.sonatype.com/artifact/org.mockito/mockito-core>).
   - Alternativa muy usada: **MVN Repository** → <https://mvnrepository.com/artifact/org.mockito/mockito-core>.
2. Elige la versión más reciente (estable, sin sufijos como `-beta` o `-RC`).
3. Copia el fragmento que aparece en la pestaña **Maven** (bloque `<dependency>...</dependency>`) y pégalo dentro de `<dependencies>` en tu `pom.xml`.
4. Repite la búsqueda para `mockito-junit-jupiter` (la integración con JUnit Jupiter, el modelo de programación de JUnit 5 y 6): <https://central.sonatype.com/artifact/org.mockito/mockito-junit-jupiter>. Usa **la misma versión** que `mockito-core`.

Otras fuentes útiles:

- Web oficial de Mockito: <https://site.mockito.org/>
- Código fuente y notas de cada versión (*releases*): <https://github.com/mockito/mockito/releases>

> 💡 Si después de editar el `pom.xml` IntelliJ no reconoce las clases de Mockito, pulsa el botón **Load Maven Changes** (icono de la "m" con flechas que aparece arriba a la derecha) o ejecuta `mvn dependency:resolve` en la terminal para forzar la descarga.

> ⚠️ Si estás detrás de un proxy o en una red corporativa, Maven puede no poder acceder a Maven Central. En ese caso hay que configurar el proxy en `~/.m2/settings.xml`.

#### Dependencias para el `pom.xml`

Con las coordenadas obtenidas, añade estas dependencias a tu `pom.xml`:

```xml
<properties>
    <maven.compiler.release>25</maven.compiler.release>
    <mockito.version>5.20.0</mockito.version>
    <junit.version>6.1.3</junit.version>
</properties>

<!-- BOM de JUnit: fija la misma versión para todos los módulos de JUnit -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.junit</groupId>
            <artifactId>junit-bom</artifactId>
            <version>${junit.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- JUnit 6 (la versión la pone el BOM) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>${mockito.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Integración de Mockito con JUnit Jupiter -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>${mockito.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

> ℹ️ Usa siempre la versión más reciente de Mockito 5.x disponible en Maven Central. Las versiones antiguas pueden no soportar el *bytecode* de Java 25.

> ⚠️ **¿Por qué el BOM?** `mockito-junit-jupiter` depende internamente de `junit-jupiter-api` de la rama 5.x. Sin el BOM, Maven podría mezclar módulos de JUnit 5 y JUnit 6 en el *classpath* y los tests fallarían al arrancar. Importando `junit-bom` en `<dependencyManagement>` todos los módulos de JUnit quedan alineados en la 6.1.3.

### 3.2. El agente de Mockito (importante en Java 21+)

Desde Java 21, la JVM avisa cuando una librería carga un **agente** de forma dinámica en tiempo de ejecución, y en futuras versiones esto dejará de estar permitido por defecto. Mockito necesita ese agente para poder simular clases `final`, métodos estáticos, etc.

Si no lo configuras, verás un aviso como este al ejecutar los tests:

```
Mockito is currently self-attaching to enable the inline-mock-maker.
This will no longer work in future releases of the JDK.
Please add Mockito as an agent to your build...
```

La solución es indicar a la JVM que cargue Mockito como agente **desde el arranque**. En Maven:

```xml
<build>
    <plugins>
        <!-- Calcula la ruta del jar de Mockito y la guarda en una propiedad -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>properties</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- Pasa el agente a la JVM al ejecutar los tests
             (JUnit 6 necesita Surefire 3.0.0 o superior) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.5.5</version>
            <configuration>
                <argLine>-javaagent:${org.mockito:mockito-core:jar}</argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

> ⚠️ Si ya usas otro plugin que modifica `argLine` (por ejemplo, JaCoCo para la cobertura), escribe `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>` para no machacar su configuración.

### 3.3. Gradle (Kotlin DSL)

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}

val mockitoAgent = configurations.create("mockitoAgent")

dependencies {
    testImplementation(platform("org.junit:junit-bom:6.1.3"))
    testImplementation("org.junit.jupiter:junit-jupiter")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
    testImplementation("org.mockito:mockito-core:5.20.0")
    testImplementation("org.mockito:mockito-junit-jupiter:5.20.0")
    mockitoAgent("org.mockito:mockito-core:5.20.0") { isTransitive = false }
}

tasks.test {
    useJUnitPlatform()
    jvmArgs("-javaagent:${mockitoAgent.asPath}")
}
```

### 3.4. Configuración en IntelliJ IDEA

IntelliJ IDEA no necesita ningún plugin para trabajar con Mockito: basta con que el proyecto tenga las dependencias en Maven o Gradle. Estos son los pasos.

#### Paso 1: instalar el JDK 25

1. Abre **File → Project Structure** (`Ctrl+Alt+Shift+S` en Windows/Linux, `⌘;` en macOS).
2. En **Project → SDK**, despliega la lista y elige **Add SDK → Download JDK...**.
3. Selecciona la versión **25** y un proveedor (por ejemplo, *Eclipse Temurin* u *Oracle OpenJDK*) y pulsa **Download**.
4. En **Language level**, elige **25** (o *SDK default*).

> ℹ️ Si ya tienes el JDK instalado en el ordenador, en lugar de *Download JDK* usa **Add JDK from disk...** y selecciona su carpeta.

#### Paso 2: crear el proyecto (si aún no lo tienes)

1. **File → New → Project...**
2. Elige **Java**, como *Build system* selecciona **Maven** o **Gradle**, y como *JDK* el **25** que acabas de instalar.
3. Pulsa **Create**.

IntelliJ crea automáticamente las carpetas `src/main/java` (código) y `src/test/java` (tests). La de tests aparece en **verde**: es ahí donde irán tus clases de test.

#### Paso 3: añadir las dependencias

1. Abre el `pom.xml` (o el `build.gradle.kts`) y pega las dependencias de las secciones [3.1](#31-maven) o [3.3](#33-gradle-kotlin-dsl).
2. Aparecerá un pequeño icono flotante con una **"m"** (Maven) o un **elefante** (Gradle) en la esquina superior derecha del editor. Púlsalo para cargar los cambios.
   - Atajo: `Ctrl+Shift+O` (Windows/Linux) o `⌘⇧I` (macOS).
   - También puedes abrir la ventana **Maven** o **Gradle** (en el lateral derecho) y pulsar el botón de **recargar**.
3. Comprueba que todo ha ido bien desplegando **External Libraries** en el panel *Project*: deberían aparecer `mockito-core`, `mockito-junit-jupiter` y `junit-jupiter`.

> 💡 Truco: dentro del `pom.xml`, pulsa `Alt+Insert` (`⌘N` en macOS) → **Dependency** para buscar y añadir dependencias sin tener que escribir el XML a mano.

#### Paso 4: configurar el agente de Mockito al ejecutar desde IntelliJ

- **Con Gradle:** no tienes que hacer nada. Por defecto IntelliJ ejecuta los tests a través de Gradle, así que se aplica el `jvmArgs` del `build.gradle.kts`. Puedes comprobarlo en **Settings → Build, Execution, Deployment → Build Tools → Gradle → Run tests using: Gradle**.

- **Con Maven:** IntelliJ ejecuta los tests con su propio lanzador de JUnit, y puede que no resuelva la propiedad `${org.mockito:mockito-core:jar}`. Tienes dos opciones:

  **Opción A (recomendada): delegar en Maven.**
  Ve a **Settings → Build, Execution, Deployment → Build Tools → Maven → Runner** y marca **Delegate IDE build/run actions to Maven**. Así IntelliJ usará exactamente la misma configuración que la línea de comandos.

  **Opción B: añadir el agente a la plantilla de JUnit.**
  1. **Run → Edit Configurations...**
  2. Abajo a la izquierda, pulsa **Edit configuration templates...** y elige **JUnit**.
  3. En **VM options** (si no ves el campo, actívalo en *Modify options → Add VM options*), escribe:
     ```
     -javaagent:$MAVEN_REPOSITORY$/org/mockito/mockito-core/5.20.0/mockito-core-5.20.0.jar
     ```
     `$MAVEN_REPOSITORY$` es una variable de IntelliJ que apunta a tu repositorio local de Maven (normalmente `~/.m2/repository`). Cambia `5.20.0` por la versión que uses.
  4. Pulsa **OK**. A partir de ahora, todos los tests nuevos que lances usarán el agente.

  > ⚠️ Las configuraciones de ejecución que ya existían no cogen la plantilla. Bórralas desde *Edit Configurations* para que se vuelvan a crear.

#### Paso 5: generar una clase de test

1. Abre la clase que quieres probar (por ejemplo, `UsuarioService`).
2. Coloca el cursor sobre el nombre de la clase y pulsa `Ctrl+Shift+T` (`⌘⇧T` en macOS) → **Create New Test...**
3. Elige **Testing library: JUnit5** (IntelliJ la llama así también para JUnit 6, porque ambas usan JUnit Jupiter), marca los métodos que quieras probar y pulsa **OK**.

IntelliJ crea `UsuarioServiceTest` en `src/test/java`, en el mismo paquete. Solo te queda añadir `@ExtendWith(MockitoExtension.class)` y los `@Mock`.

> 💡 Si escribes `when(`, `verify(` o `any(` y aparecen en rojo, pon el cursor encima y pulsa `Alt+Enter` (`⌥↩` en macOS) → **Import static method**. IntelliJ añadirá el `import static org.mockito.Mockito.*` por ti.

#### Paso 6: ejecutar los tests

- Pulsa el **triángulo verde** que aparece en el margen izquierdo, junto a la clase (ejecuta todos sus tests) o junto a un método (ejecuta solo ese test).
- Atajo: con el cursor dentro del test, `Ctrl+Shift+F10` (`⌃⇧R` en macOS).
- Para ejecutar todos los tests del proyecto: clic derecho sobre `src/test/java` → **Run 'All Tests'**.
- Para ver la cobertura: clic en el triángulo verde → **Run ... with Coverage**.

Los resultados aparecen en la ventana **Run**: en verde los tests que pasan y en rojo los que fallan. Si haces clic en un test fallido, verás el mensaje de error y podrás saltar directamente a la línea que ha fallado.

---

## 4. El código que vamos a probar

Durante toda la guía usaremos este pequeño ejemplo: un servicio que registra usuarios y les envía un correo de bienvenida.

```java
// Un record es una clase inmutable que solo guarda datos
public record Usuario(Long id, String nombre, String email) { }
```

```java
public interface UsuarioRepository {
    Optional<Usuario> buscarPorId(Long id);
    boolean existeEmail(String email);
    Usuario guardar(Usuario usuario);
}
```

```java
public interface EmailService {
    void enviar(String destinatario, String asunto, String cuerpo);
}
```

```java
public class UsuarioService {

    private final UsuarioRepository repository;
    private final EmailService emailService;

    public UsuarioService(UsuarioRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }

    public Usuario registrar(String nombre, String email) {
        if (repository.existeEmail(email)) {
            throw new IllegalArgumentException("El email ya está registrado: " + email);
        }

        var guardado = repository.guardar(new Usuario(null, nombre, email));

        emailService.enviar(
                guardado.email(),
                "¡Bienvenido!",
                "Hola " + guardado.nombre() + ", gracias por registrarte."
        );

        return guardado;
    }

    public String obtenerNombre(Long id) {
        return repository.buscarPorId(id)
                .map(Usuario::nombre)
                .orElse("Desconocido");
    }
}
```

`UsuarioService` es la clase que queremos probar. `UsuarioRepository` y `EmailService` son sus dependencias: esas son las que vamos a simular con Mockito.

---

## 5. Tu primer test con Mockito

Empezamos sin anotaciones, creando los mocks "a mano" para que se vea claro qué ocurre.

```java
import org.junit.jupiter.api.Test;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

class UsuarioServiceTest {

    @Test
    void obtenerNombre_devuelveElNombreSiElUsuarioExiste() {
        // 1. Preparar
        UsuarioRepository repository = mock(UsuarioRepository.class);
        EmailService emailService = mock(EmailService.class);
        var service = new UsuarioService(repository, emailService);

        when(repository.buscarPorId(1L))
                .thenReturn(Optional.of(new Usuario(1L, "Ana", "ana@correo.es")));

        // 2. Ejecutar
        String nombre = service.obtenerNombre(1L);

        // 3. Comprobar
        assertEquals("Ana", nombre);
    }

    @Test
    void obtenerNombre_devuelveDesconocidoSiNoExiste() {
        UsuarioRepository repository = mock(UsuarioRepository.class);
        var service = new UsuarioService(repository, mock(EmailService.class));

        when(repository.buscarPorId(99L)).thenReturn(Optional.empty());

        assertEquals("Desconocido", service.obtenerNombre(99L));
    }
}
```

**¿Qué está pasando?**

- `mock(UsuarioRepository.class)` crea un repositorio falso. No toca ninguna base de datos.
- `when(...).thenReturn(...)` le dice: "cuando te pidan el usuario con id 1, devuelve a Ana".
- El servicio no sabe que el repositorio es falso: lo usa con normalidad.

> 💡 **Dato útil:** si no defines un `when` para un método que devuelve `Optional`, Mockito devuelve `Optional.empty()` por defecto. Para colecciones devuelve colecciones vacías, para números `0` y para `boolean` devuelve `false`.

---

## 6. Usando anotaciones

Crear los mocks a mano funciona, pero se vuelve repetitivo. Con anotaciones el código queda mucho más limpio:

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)   // Activa Mockito en JUnit
class UsuarioServiceTest {

    @Mock
    UsuarioRepository repository;     // Mockito crea el mock

    @Mock
    EmailService emailService;        // Mockito crea el mock

    @InjectMocks
    UsuarioService service;           // Mockito crea el servicio e inyecta los mocks

    @Test
    void registrar_guardaElUsuarioYEnviaEmail() {
        when(repository.existeEmail("ana@correo.es")).thenReturn(false);
        when(repository.guardar(any(Usuario.class)))
                .thenReturn(new Usuario(1L, "Ana", "ana@correo.es"));

        Usuario resultado = service.registrar("Ana", "ana@correo.es");

        assertEquals(1L, resultado.id());
        verify(emailService).enviar(eq("ana@correo.es"), eq("¡Bienvenido!"), anyString());
    }
}
```

| Anotación | Para qué sirve |
|---|---|
| `@ExtendWith(MockitoExtension.class)` | Integra Mockito con JUnit (Jupiter). Inicializa los mocks antes de cada test y revisa que no haya *stubs* sin usar. |
| `@Mock` | Crea un mock del tipo del campo. |
| `@InjectMocks` | Crea una instancia real de la clase y le inyecta los `@Mock` (por constructor, si puede). |
| `@Spy` | Crea un *spy* (lo vemos más adelante). |
| `@Captor` | Crea un `ArgumentCaptor` (lo vemos más adelante). |

> ✅ **Recomendación:** aunque `@InjectMocks` es cómodo, muchos equipos prefieren crear el SUT a mano en un método `@BeforeEach`. Así, si cambias el constructor, el compilador te avisa en lugar de fallar en silencio.

```java
@BeforeEach
void setUp() {
    service = new UsuarioService(repository, emailService);
}
```

---

## 7. Stubbing: decidir qué devuelve un mock

### 7.1. Devolver un valor

```java
when(repository.existeEmail("ana@correo.es")).thenReturn(true);
```

### 7.2. Devolver valores distintos en llamadas sucesivas

```java
when(repository.existeEmail("ana@correo.es"))
        .thenReturn(false)   // primera llamada
        .thenReturn(true);   // segunda y siguientes
```

O de forma abreviada:

```java
when(repository.existeEmail("ana@correo.es")).thenReturn(false, true);
```

### 7.3. Lanzar una excepción

```java
when(repository.buscarPorId(1L))
        .thenThrow(new IllegalStateException("Base de datos no disponible"));
```

Y comprobarlo con JUnit:

```java
@Test
void registrar_fallaSiElEmailYaExiste() {
    when(repository.existeEmail("ana@correo.es")).thenReturn(true);

    var excepcion = assertThrows(
            IllegalArgumentException.class,
            () -> service.registrar("Ana", "ana@correo.es")
    );

    assertEquals("El email ya está registrado: ana@correo.es", excepcion.getMessage());
    verify(repository, never()).guardar(any());
    verifyNoInteractions(emailService);
}
```

### 7.4. Métodos `void`: `doThrow`, `doNothing`...

Un método `void` no devuelve nada, así que no puedes escribir `when(emailService.enviar(...))` (no compila). En su lugar se usa la familia `do...()`:

```java
doThrow(new RuntimeException("Servidor de correo caído"))
        .when(emailService).enviar(anyString(), anyString(), anyString());
```

Fíjate en el orden: primero **qué hacer**, luego `.when(mock)` y después **el método**.

### 7.5. Calcular la respuesta a partir de los argumentos: `thenAnswer`

A veces quieres que el mock devuelva algo que dependa de lo que recibe. Por ejemplo, simular que el repositorio asigna un id al guardar:

```java
when(repository.guardar(any(Usuario.class))).thenAnswer(invocacion -> {
    Usuario recibido = invocacion.getArgument(0);
    return new Usuario(42L, recibido.nombre(), recibido.email());
});
```

---

## 8. Argument matchers

Los *matchers* permiten decir "cualquier valor" o "un valor que cumpla esta condición" en lugar de un valor exacto.

| Matcher | Significado |
|---|---|
| `any()` | Cualquier cosa (incluido `null`) |
| `any(Usuario.class)` | Cualquier objeto de tipo `Usuario` (no `null`) |
| `anyString()`, `anyInt()`, `anyLong()`, `anyList()`... | Cualquier valor de ese tipo |
| `eq(valor)` | Exactamente ese valor |
| `isNull()`, `notNull()` | Valor nulo / no nulo |
| `startsWith("Hola")`, `contains("texto")` | Para cadenas |
| `argThat(x -> ...)` | Condición personalizada |

Ejemplo con `argThat`:

```java
verify(repository).guardar(argThat(u ->
        u.nombre().equals("Ana") && u.email().endsWith("@correo.es")
));
```

### ⚠️ La regla de oro de los matchers

**Si usas un matcher en un argumento, tienes que usar matchers en todos los argumentos.**

```java
// ❌ MAL: mezcla un valor directo con un matcher → InvalidUseOfMatchersException
verify(emailService).enviar("ana@correo.es", "¡Bienvenido!", anyString());

// ✅ BIEN: todos son matchers
verify(emailService).enviar(eq("ana@correo.es"), eq("¡Bienvenido!"), anyString());
```

---

## 9. Verificar interacciones con `verify`

`verify` sirve para comprobar que tu código ha **llamado** a una dependencia. Es útil sobre todo con métodos que no devuelven nada (como enviar un correo), porque no hay ningún resultado que comprobar con `assertEquals`.

```java
verify(emailService).enviar("ana@correo.es", "¡Bienvenido!", "Hola Ana, gracias por registrarte.");
```

### 9.1. Número de llamadas

```java
verify(repository).guardar(any());                // exactamente 1 vez (por defecto)
verify(repository, times(2)).guardar(any());      // exactamente 2 veces
verify(repository, never()).guardar(any());       // ninguna vez
verify(repository, atLeastOnce()).guardar(any()); // 1 o más veces
verify(repository, atMost(3)).guardar(any());     // como mucho 3 veces
```

### 9.2. Sin interacciones

```java
verifyNoInteractions(emailService);       // no se ha llamado a NINGÚN método del mock
verifyNoMoreInteractions(repository);     // no hay llamadas aparte de las ya verificadas
```

> ⚠️ No abuses de `verifyNoMoreInteractions`: hace los tests muy rígidos y se rompen con cualquier cambio interno sin importancia.

### 9.3. Verificar el orden de las llamadas

```java
InOrder orden = inOrder(repository, emailService);

orden.verify(repository).existeEmail("ana@correo.es");
orden.verify(repository).guardar(any());
orden.verify(emailService).enviar(anyString(), anyString(), anyString());
```

Así compruebas que primero se mira si existe el email, después se guarda y, por último, se envía el correo.

---

## 10. Capturar argumentos con `ArgumentCaptor`

A veces necesitas inspeccionar **el objeto exacto** que tu código ha pasado a un mock. Por ejemplo: ¿el servicio ha creado el `Usuario` con los datos correctos antes de guardarlo?

```java
@Captor
ArgumentCaptor<Usuario> usuarioCaptor;

@Test
void registrar_guardaUnUsuarioSinIdConLosDatosRecibidos() {
    when(repository.guardar(any())).thenReturn(new Usuario(1L, "Ana", "ana@correo.es"));

    service.registrar("Ana", "ana@correo.es");

    verify(repository).guardar(usuarioCaptor.capture());
    Usuario capturado = usuarioCaptor.getValue();

    assertNull(capturado.id());
    assertEquals("Ana", capturado.nombre());
    assertEquals("ana@correo.es", capturado.email());
}
```

También puedes crearlo sin anotación:

```java
var captor = ArgumentCaptor.forClass(String.class);
verify(emailService).enviar(anyString(), anyString(), captor.capture());
assertTrue(captor.getValue().contains("Ana"));
```

> 💡 Si el método se ha llamado varias veces, usa `captor.getAllValues()` para obtener una lista con todos los valores.

> 💡 Con los `record` de Java no siempre necesitas un captor: como implementan `equals` automáticamente, puedes verificar directamente con `verify(repository).guardar(new Usuario(null, "Ana", "ana@correo.es"))`.

---

## 11. Spies: objetos reales parcialmente simulados

Un **spy** envuelve un objeto **real**. Todos sus métodos funcionan de verdad, salvo los que tú decidas sobrescribir.

```java
@Test
void ejemploDeSpy() {
    List<String> lista = spy(new ArrayList<>());

    lista.add("uno");
    lista.add("dos");

    // El método real funciona
    assertEquals(2, lista.size());

    // Sobrescribimos solo size()
    doReturn(100).when(lista).size();
    assertEquals(100, lista.size());

    // Y también podemos verificar
    verify(lista).add("uno");
}
```

> ⚠️ Con spies, usa **`doReturn(...).when(spy).metodo()`** en lugar de `when(spy.metodo()).thenReturn(...)`. La segunda forma **ejecuta el método real** al preparar el stub, lo que puede provocar errores inesperados (por ejemplo, un `IndexOutOfBoundsException` al hacer `when(lista.get(0))` sobre una lista vacía).

**¿Cuándo usar un spy?** Pocas veces. Si necesitas simular parte de la clase que estás probando, suele ser una señal de que esa clase hace demasiadas cosas y convendría dividirla.

---

## 12. Simular métodos estáticos

Mockito permite simular métodos `static` con `mockStatic`. Es muy útil para cosas como la fecha actual o un `UUID` aleatorio.

Supongamos esta clase:

```java
public class GeneradorCodigos {
    public String nuevoCodigo() {
        return "COD-" + UUID.randomUUID();
    }
}
```

El test:

```java
@Test
void nuevoCodigo_usaUnUuidAleatorio() {
    var uuidFijo = UUID.fromString("123e4567-e89b-12d3-a456-426614174000");

    try (MockedStatic<UUID> uuidMock = mockStatic(UUID.class)) {
        uuidMock.when(UUID::randomUUID).thenReturn(uuidFijo);

        String codigo = new GeneradorCodigos().nuevoCodigo();

        assertEquals("COD-123e4567-e89b-12d3-a456-426614174000", codigo);
    }
    // Fuera del try, UUID.randomUUID() vuelve a funcionar con normalidad
}
```

> ⚠️ Usa **siempre** `try-with-resources`. Si no cierras el `MockedStatic`, el método estático seguirá simulado y afectará a otros tests.

> ✅ **Mejor alternativa cuando puedas:** en lugar de simular estáticos, inyecta la dependencia. Para fechas, por ejemplo, pasa un `java.time.Clock` al constructor y en el test usa `Clock.fixed(...)`. No necesitas Mockito para eso y el código queda más limpio.

```java
public class Saludador {
    private final Clock clock;

    public Saludador(Clock clock) { this.clock = clock; }

    public String saludo() {
        int hora = LocalTime.now(clock).getHour();
        return hora < 14 ? "Buenos días" : "Buenas tardes";
    }
}

@Test
void saludo_porLaMañana() {
    var clock = Clock.fixed(Instant.parse("2025-01-01T09:00:00Z"), ZoneOffset.UTC);
    assertEquals("Buenos días", new Saludador(clock).saludo());
}
```

---

## 13. Estilo BDD: `given` / `then`

Mockito incluye la clase `BDDMockito`, con nombres que encajan con el formato **Dado / Cuando / Entonces** (*Given / When / Then*). Hace exactamente lo mismo que `when` y `verify`, pero algunos equipos lo encuentran más legible.

```java
import static org.mockito.BDDMockito.*;

@Test
void registrar_enviaEmailDeBienvenida() {
    // Dado
    given(repository.existeEmail("ana@correo.es")).willReturn(false);
    given(repository.guardar(any())).willReturn(new Usuario(1L, "Ana", "ana@correo.es"));

    // Cuando
    service.registrar("Ana", "ana@correo.es");

    // Entonces
    then(emailService).should().enviar(eq("ana@correo.es"), eq("¡Bienvenido!"), anyString());
    then(emailService).shouldHaveNoMoreInteractions();
}
```

| Mockito clásico | BDDMockito |
|---|---|
| `when(x).thenReturn(y)` | `given(x).willReturn(y)` |
| `when(x).thenThrow(e)` | `given(x).willThrow(e)` |
| `doThrow(e).when(mock).metodo()` | `willThrow(e).given(mock).metodo()` |
| `verify(mock).metodo()` | `then(mock).should().metodo()` |
| `verify(mock, never()).metodo()` | `then(mock).should(never()).metodo()` |

Elige un estilo y úsalo de forma coherente en todo el proyecto.

---

## 14. Errores habituales y cómo resolverlos

### `UnnecessaryStubbingException`

```
Unnecessary stubbings detected.
Clean & maintainable test code requires zero unnecessary code.
```

**Causa:** has definido un `when(...)` que el test nunca llega a usar. `MockitoExtension` trabaja en modo **estricto** y te avisa porque ese código sobra (o porque tu test no está probando lo que crees).

**Solución:** borra ese `when`. Si de verdad lo necesitas (caso raro), puedes usar `lenient().when(...)`.

### `InvalidUseOfMatchersException`

**Causa:** has mezclado valores directos con matchers en la misma llamada.

**Solución:** usa `eq(...)` para los valores fijos (ver [sección 8](#8-argument-matchers)).

### `WrongTypeOfReturnValue`

**Causa:** normalmente ocurre al usar `when(spy.metodo())` sobre un spy, o al devolver un tipo incorrecto.

**Solución:** con spies, usa `doReturn(...).when(spy).metodo()`.

### `NullPointerException` en el servicio

**Causa:** los mocks no se han inicializado. Típicamente se te ha olvidado `@ExtendWith(MockitoExtension.class)`.

**Solución:** añade la anotación a la clase de test.

### `PotentialStubbingProblem`

**Causa:** has hecho `when(repo.buscarPorId(1L))`, pero el código ha llamado a `repo.buscarPorId(2L)`. Mockito detecta que los argumentos no coinciden.

**Solución:** revisa qué argumentos recibe realmente el mock. Probablemente hay un error en el test o en el código.

### Aviso de "self-attaching" del agente

**Causa:** no has configurado Mockito como agente de Java.

**Solución:** revisa la [sección 3.2](#32-el-agente-de-mockito-importante-en-java-21).

---

## 15. Buenas prácticas

1. **Simula dependencias, no la clase que pruebas.** El SUT siempre debe ser real.
2. **No simules objetos de datos.** Los `record`, DTO o entidades simples se crean con `new`. Simularlos solo complica el test.
   ```java
   // ❌ Innecesario
   Usuario u = mock(Usuario.class);
   when(u.nombre()).thenReturn("Ana");

   // ✅ Mucho más simple
   var u = new Usuario(1L, "Ana", "ana@correo.es");
   ```
3. **No simules tipos que no son tuyos** (por ejemplo, clases internas de librerías externas) si puedes evitarlo. Crea una interfaz propia que las envuelva y simula esa interfaz.
4. **Un test, un comportamiento.** Si el nombre del test necesita un "y", probablemente sean dos tests.
5. **Nombres descriptivos.** `registrar_fallaSiElEmailYaExiste` explica mucho más que `test1`. También puedes usar `@DisplayName("Falla si el email ya existe")`.
6. **Verifica lo importante, no todo.** Comprueba el resultado y las interacciones que forman parte del comportamiento esperado (por ejemplo, que se envía el email). No verifiques cada llamada interna.
7. **Prefiere comprobar resultados antes que interacciones.** Si un método devuelve algo, `assertEquals` sobre el resultado suele ser mejor que un `verify`.
8. **Si cuesta mucho preparar los mocks**, es una señal de que la clase tiene demasiadas dependencias. Piensa en dividirla.
9. **Evita `mockStatic` cuando haya alternativa**, como inyectar un `Clock` o un proveedor.

---

## 16. Chuleta rápida

```java
// ---------- Crear ----------
@ExtendWith(MockitoExtension.class)
@Mock        Dependencia dep;
@Spy         Lista lista = new ArrayList<>();
@Captor      ArgumentCaptor<Tipo> captor;
@InjectMocks ClaseAProbar sut;

var mock = mock(Dependencia.class);
var spy  = spy(objetoReal);

// ---------- Stubbing ----------
when(mock.metodo(arg)).thenReturn(valor);
when(mock.metodo(arg)).thenReturn(v1, v2, v3);
when(mock.metodo(arg)).thenThrow(new Excepcion());
when(mock.metodo(any())).thenAnswer(inv -> inv.getArgument(0));

doThrow(new Excepcion()).when(mock).metodoVoid();
doNothing().when(mock).metodoVoid();
doReturn(valor).when(spy).metodo();

lenient().when(mock.metodo()).thenReturn(valor);

// ---------- Matchers ----------
any(), any(Tipo.class), anyString(), anyInt(), anyLong(), anyList()
eq(valor), isNull(), notNull(), argThat(x -> condicion)

// ---------- Verificación ----------
verify(mock).metodo(arg);
verify(mock, times(n)).metodo(arg);
verify(mock, never()).metodo(any());
verify(mock, atLeastOnce()).metodo(any());
verifyNoInteractions(mock);
verifyNoMoreInteractions(mock);

InOrder orden = inOrder(mock1, mock2);
orden.verify(mock1).primero();
orden.verify(mock2).segundo();

// ---------- Captura ----------
verify(mock).metodo(captor.capture());
captor.getValue();
captor.getAllValues();

// ---------- Estáticos ----------
try (MockedStatic<Clase> st = mockStatic(Clase.class)) {
    st.when(Clase::metodoEstatico).thenReturn(valor);
}

// ---------- BDD ----------
given(mock.metodo()).willReturn(valor);
then(mock).should().metodo();
```

---

## Recursos

- Documentación oficial de Mockito: <https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html>
- Repositorio de Mockito: <https://github.com/mockito/mockito>
- Guía oficial de JUnit 6.1.3: <https://docs.junit.org/6.1.3/overview.html>
