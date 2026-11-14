# Cohesión y acoplamiento

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).

## Índice

1. [Dos medidas de calidad del diseño](#1-dos-medidas-de-calidad-del-diseño)
2. [¿Qué es el acoplamiento?](#2-qué-es-el-acoplamiento)
3. [Tipos de acoplamiento](#3-tipos-de-acoplamiento)
4. [Por qué conviene reducir el acoplamiento](#4-por-qué-conviene-reducir-el-acoplamiento)
5. [Ejemplos prácticos: de acoplamiento fuerte a débil](#5-ejemplos-prácticos-de-acoplamiento-fuerte-a-débil)
6. [Técnicas para reducir el acoplamiento](#6-técnicas-para-reducir-el-acoplamiento)
7. [Cómo detectar un acoplamiento excesivo](#7-cómo-detectar-un-acoplamiento-excesivo)
8. [El acoplamiento cero no existe](#8-el-acoplamiento-cero-no-existe)
9. [¿Qué es la cohesión?](#9-qué-es-la-cohesión)
10. [Tipos de cohesión](#10-tipos-de-cohesión)
11. [Buenas prácticas para conseguir alta cohesión](#11-buenas-prácticas-para-conseguir-alta-cohesión)
12. [Cómo comprobar la cohesión de una clase](#12-cómo-comprobar-la-cohesión-de-una-clase)
13. [Cómo se influyen mutuamente](#13-cómo-se-influyen-mutuamente)
14. [Cohesión y acoplamiento a nivel de paquete y de módulo](#14-cohesión-y-acoplamiento-a-nivel-de-paquete-y-de-módulo)
15. [Una analogía: la casa](#15-una-analogía-la-casa)
16. [Lista de comprobación](#16-lista-de-comprobación)
17. [Referencias](#17-referencias)

---

## 1. Dos medidas de calidad del diseño

| Concepto | Qué mide | Objetivo |
|---|---|---|
| **Cohesión** | Lo bien que trabajan juntos los elementos **dentro** de un módulo (clase, paquete…) para cumplir **un único propósito bien definido**. | **Alta** |
| **Acoplamiento** | El grado de interdependencia **entre** módulos. | **Bajo** |

La regla de oro del diseño orientado a objetos es: **alta cohesión y bajo acoplamiento**. Así se obtienen clases y módulos más flexibles, que se rompen menos al hacer cambios.

Ambos conceptos proceden del **diseño estructurado** de los años setenta (Larry Constantine, junto con Wayne Stevens y Glenford Myers) y siguen siendo criterios fundamentales para valorar la calidad de un diseño. En Java, un «módulo» puede ser un método, una clase, un paquete o un módulo de la plataforma (`module-info.java`).

---

## 2. ¿Qué es el acoplamiento?

El **acoplamiento** es el **grado de interdependencia entre los módulos o componentes de un sistema**.

Dicho de forma práctica: **dos piezas están acopladas si, para cambiar una, hay que tener en cuenta (o cambiar) la otra**.

| | Acoplamiento **alto** (fuerte) | Acoplamiento **bajo** (débil) |
|---|---|---|
| Dependencia | Los módulos dependen mucho unos de otros. | Los módulos son bastante independientes. |
| Efecto de un cambio | Se propaga en cadena («efecto dominó»). | Queda localizado en un módulo. |
| Riesgo | Más errores y efectos secundarios inesperados. | Menos errores; el código es modular. |
| Pruebas | Difíciles: hay que montar media aplicación. | Sencillas: se sustituyen dependencias por dobles de prueba. |
| Reutilización | Difícil: el módulo «arrastra» a otros. | Fácil: el módulo se puede llevar a otro contexto. |

El objetivo de un buen diseño es un **acoplamiento bajo**.

## 3. Tipos de acoplamiento

Los tipos se suelen ordenar **del peor (más fuerte) al mejor (más débil)**:

| # | Tipo | Descripción | Valoración |
|---|---|---|---|
| 1 | **De contenido** | Un módulo accede o modifica directamente el contenido interno de otro. | 🔴 Muy malo |
| 2 | **Común** | Varios módulos comparten variables globales. | 🔴 Malo |
| 3 | **Externo** | Los módulos dependen de un elemento externo común: un formato de fichero, una base de datos compartida, un protocolo, un dispositivo. | 🟠 A vigilar |
| 4 | **De control** | Un módulo dirige el flujo de otro pasándole información sobre **qué debe hacer** (un *flag*). | 🟠 A vigilar |
| 5 | **De sello** (*stamp*) | Se pasa una estructura de datos completa, pero el receptor solo usa una parte. | 🟡 Aceptable |
| 6 | **De datos** | Se intercambian solo los datos necesarios mediante parámetros bien definidos. | 🟢 Bueno |
| 7 | **De mensajes** | Los módulos se comunican mediante mensajes o eventos, sin dependencia directa. | 🟢 Muy bueno |

### 3.1. Acoplamiento de contenido

Ocurre cuando una clase manipula el estado interno de otra saltándose su interfaz pública. En Java el caso más evidente es usar **reflexión** para tocar campos privados, o exponer colecciones internas mutables.

```java
// ❌ La clase expone su lista interna: cualquiera puede modificarla
public class Pedido {
    private final List<Linea> lineas = new ArrayList<>();

    public List<Linea> getLineas() {
        return lineas;          // se entrega la referencia real
    }
}

pedido.getLineas().clear();     // otro módulo vacía el pedido «por dentro»
```

```java
// ✅ Se protege el estado interno
public class Pedido {
    private final List<Linea> lineas = new ArrayList<>();

    public void anadir(Linea linea) {
        lineas.add(linea);
    }

    public List<Linea> lineas() {
        return List.copyOf(lineas);   // copia inmutable
    }
}
```

### 3.2. Acoplamiento común

Varios módulos leen y escriben el mismo estado global. Cualquier módulo puede alterar el comportamiento de los demás sin que se note, lo que provoca efectos secundarios inesperados y complica la depuración.

```java
// ❌ Estado global mutable
public class Configuracion {
    public static String idioma = "es";
    public static boolean modoPruebas = false;
}
```

Alternativa: pasar la configuración como dependencia (inmutable, por ejemplo con un `record`) a quien la necesite.

```java
public record Configuracion(String idioma, boolean modoPruebas) { }

public class ServicioCorreo {
    private final Configuracion configuracion;

    public ServicioCorreo(Configuracion configuracion) {
        this.configuracion = configuracion;
    }
}
```

> Para datos que deben estar disponibles en toda una cadena de llamadas (por ejemplo, el usuario autenticado de una petición), Java 25 ofrece **`ScopedValue`** (JEP 506), que es inmutable y tiene un ámbito de vida acotado. Es una alternativa mucho más segura que las variables estáticas o `ThreadLocal`.

### 3.3. Acoplamiento externo

Los módulos dependen de algo externo que ninguno controla: el esquema de una base de datos, el formato de un CSV, la API de un tercero. Es inevitable en sistemas reales, pero conviene **aislarlo** detrás de una interfaz para que solo una parte del código conozca los detalles.

### 3.4. Acoplamiento de control

Un módulo le dice a otro **cómo** debe comportarse mediante un parámetro de control:

```java
// ❌ El llamante decide el flujo interno del método
public void exportar(List<Factura> facturas, boolean comoPdf) {
    if (comoPdf) {
        // ... generar PDF
    } else {
        // ... generar CSV
    }
}
```

```java
// ✅ Cada comportamiento es un tipo distinto
public interface Exportador {
    void exportar(List<Factura> facturas);
}

public class ExportadorPdf implements Exportador { /* ... */ }
public class ExportadorCsv implements Exportador { /* ... */ }
```

### 3.5. Acoplamiento de sello

Se pasa un objeto completo cuando solo se necesita un dato. El receptor queda acoplado a toda la estructura.

```java
// 🟡 Solo se usa el código postal, pero se depende de todo el Cliente
public double calcularEnvio(Cliente cliente) {
    return tarifas.precioPara(cliente.direccion().codigoPostal());
}

// ✅ Se pide exactamente lo necesario
public double calcularEnvio(CodigoPostal codigoPostal) {
    return tarifas.precioPara(codigoPostal);
}
```

### 3.6. Acoplamiento de datos

Es el deseable para llamadas directas: se pasan solo los datos imprescindibles, con tipos claros. Usar `record` para agrupar valores relacionados hace que estos parámetros sean explícitos e inmutables.

### 3.7. Acoplamiento de mensajes

Los módulos no se llaman directamente; publican y consumen **eventos**. El emisor ni siquiera sabe quién escucha.

```java
public record PedidoConfirmado(String idPedido, String emailCliente) { }

public interface PublicadorEventos {
    void publicar(Object evento);
}

public class ServicioPedidos {
    private final PublicadorEventos eventos;

    public ServicioPedidos(PublicadorEventos eventos) {
        this.eventos = eventos;
    }

    public void confirmar(Pedido pedido) {
        // ... lógica de confirmación
        eventos.publicar(new PedidoConfirmado(pedido.id(), pedido.emailCliente()));
    }
}
```

El servicio de correo, el de facturación o el de estadísticas pueden reaccionar al evento sin que `ServicioPedidos` los conozca. En el ecosistema Java esto se implementa, por ejemplo, con los eventos de aplicación de Spring o con sistemas de mensajería como Apache Kafka o RabbitMQ.

## 4. Por qué conviene reducir el acoplamiento

- **Mayor reutilización de código:** los módulos independientes pueden usarse en otros contextos sin grandes modificaciones.
- **Mantenimiento más sencillo:** un cambio en un módulo no afecta de forma drástica a los demás, lo que reduce el impacto de las modificaciones y de los errores.
- **Escalabilidad:** los sistemas con poco acoplamiento se adaptan y crecen de forma más eficiente.
- **Pruebas más fáciles:** se pueden sustituir las dependencias por implementaciones falsas (*mocks* o *stubs*).
- **Trabajo en paralelo:** distintas personas pueden trabajar en módulos diferentes con pocos conflictos.

## 5. Ejemplos prácticos: de acoplamiento fuerte a débil

### 5.1. `Coche` y `Motor`

```java
// ❌ Acoplamiento fuerte: Coche crea y depende de un motor concreto
class Coche {
    private final MotorGasolina motor = new MotorGasolina();

    void arrancar() { motor.arrancar(); }
    void parar()    { motor.parar(); }
}
```

Si se cambia la firma de los métodos de `MotorGasolina`, hay que cambiar `Coche`. Y si se quiere un coche eléctrico o híbrido, también hay que modificar `Coche`.

```java
// ✅ Acoplamiento débil: Coche depende de una abstracción que recibe desde fuera
interface Motor {
    void arrancar();
    void parar();
}

class MotorGasolina implements Motor {
    public void arrancar() { /* ... */ }
    public void parar()    { /* ... */ }
}

class MotorElectrico implements Motor {
    public void arrancar() { /* ... */ }
    public void parar()    { /* ... */ }
}

class Coche {
    private final Motor motor;

    Coche(Motor motor) {          // inyección de dependencias
        this.motor = motor;
    }

    void arrancar() { motor.arrancar(); }
    void parar()    { motor.parar(); }
}
```

```java
var deportivo = new Coche(new MotorGasolina());
var urbano    = new Coche(new MotorElectrico());
```

`Coche` ya no necesita cambiar para admitir nuevos tipos de motor, y cada motor puede modificarse sin tocar `Coche`.

### 5.2. Un servicio de notificaciones

```java
// ❌ Acoplamiento fuerte
public class ServicioNotificaciones {

    private final ClienteSmtp smtp = new ClienteSmtp("smtp.empresa.es", 587);

    public void avisarEnvio(Pedido pedido) {
        smtp.enviar(pedido.emailCliente(), "Tu pedido ha salido", "...");
    }
}
```

Problemas:

- `ServicioNotificaciones` **crea** su dependencia con `new`: no se puede cambiar por otra.
- Depende de una clase **concreta** (`ClienteSmtp`) y de detalles como el servidor y el puerto.
- Para probarlo hay que tener un servidor SMTP real.

```java
// ✅ Acoplamiento débil
public interface Notificador {
    void enviar(String destinatario, String asunto, String cuerpo);
}

public class NotificadorSmtp implements Notificador {
    private final ClienteSmtp smtp;

    public NotificadorSmtp(ClienteSmtp smtp) {
        this.smtp = smtp;
    }

    @Override
    public void enviar(String destinatario, String asunto, String cuerpo) {
        smtp.enviar(destinatario, asunto, cuerpo);
    }
}

public class ServicioNotificaciones {
    private final Notificador notificador;

    public ServicioNotificaciones(Notificador notificador) {   // inyección por constructor
        this.notificador = notificador;
    }

    public void avisarEnvio(Pedido pedido) {
        notificador.enviar(pedido.emailCliente(), "Tu pedido ha salido", "...");
    }
}
```

Ahora el servicio depende de una **abstracción**, la dependencia se **inyecta** desde fuera y, en una prueba, se puede usar una implementación en memoria:

```java
class NotificadorFalso implements Notificador {
    final List<String> enviados = new ArrayList<>();

    @Override
    public void enviar(String destinatario, String asunto, String cuerpo) {
        enviados.add(destinatario);
    }
}
```

## 6. Técnicas para reducir el acoplamiento

| Técnica | En qué consiste |
|---|---|
| **Programar contra interfaces** | Depender de `List` en lugar de `ArrayList`, de `Notificador` en lugar de `NotificadorSmtp`. |
| **Inyección de dependencias** | Recibir las dependencias (preferiblemente por constructor) en lugar de crearlas con `new`. Frameworks como Spring, Quarkus o Micronaut lo automatizan. |
| **Encapsulación** | Campos `private`, no exponer estructuras internas mutables, devolver copias inmutables (`List.copyOf`). |
| **Tell, Don't Ask** | Pedir a los objetos que hagan cosas en lugar de extraer sus datos (ver el documento *Tell, Don't Ask*). |
| **Ley de Deméter** | Evitar cadenas como `a.getB().getC().hacerAlgo()`. |
| **Patrones Fachada y Adaptador** | Ofrecer una interfaz unificada sobre varios objetos o adaptar una API externa a la forma que espera el sistema. |
| **Eventos y mensajería** | Comunicar módulos mediante mensajes en lugar de llamadas directas. |
| **`sealed` para jerarquías cerradas** | Declarar explícitamente qué clases pueden implementar una interfaz (`permits`), de modo que nadie fuera del diseño dependa de una jerarquía que no controla. |
| **Módulos de la plataforma Java** | Con `module-info.java`, exportar solo los paquetes públicos (`exports`) y ocultar la implementación. |

Ejemplo de encapsulación a nivel de módulo:

```java
// module-info.java
module es.empresa.facturacion {
    exports es.empresa.facturacion.api;      // visible para otros módulos
    // es.empresa.facturacion.interno NO se exporta: nadie puede acoplarse a él
}
```

## 7. Cómo detectar un acoplamiento excesivo

1. **Contar las dependencias** de una clase: si importa y usa muchas otras clases, probablemente esté muy acoplada.
2. **Contar las llamadas** entre dos clases: muchas llamadas cruzadas indican una relación demasiado estrecha.
3. **Mirar el nivel de abstracción:** si las clases interactúan a bajo nivel (campos, estructuras internas, detalles de implementación), el acoplamiento es alto.
4. **Hacer la prueba del cambio:** «si modifico esta clase, ¿cuántas tengo que recompilar o revisar?».
5. **Hacer la prueba del test:** si para probar una clase hay que levantar base de datos, red y otras diez clases, está demasiado acoplada.

Herramientas como los analizadores de dependencias de los IDE (IntelliJ IDEA, Eclipse), `jdeps` (incluida en el JDK), ArchUnit o SonarQube ayudan a medirlo.

## 8. El acoplamiento cero no existe

Un sistema sin ningún acoplamiento sería un conjunto de piezas que no colaboran. El objetivo **no** es eliminar el acoplamiento, sino:

- que sea **del tipo más débil posible** (datos o mensajes);
- que vaya **hacia abstracciones estables** (interfaces) y no hacia detalles volátiles;
- que sea **explícito** (parámetros del constructor) y no oculto (estáticos globales).

Tampoco hay que crear una interfaz para cada clase «por si acaso». Si solo existe una implementación y no hay motivos previsibles para tener otra, una interfaz extra puede violar KISS.

---

## 9. ¿Qué es la cohesión?

La **cohesión** mide lo bien que los elementos de un módulo trabajan juntos para cumplir **un único propósito bien definido**.

### 9.1. Cohesión baja

Una clase tiene **baja cohesión** cuando asume **varias responsabilidades poco relacionadas** entre sí. Consecuencias:

- El código es **difícil de entender**.
- Modificarlo genera confusión y es arriesgado: **si una clase tiene varios motivos para cambiar, tiene varios motivos para romperse**.
- Es **difícil de reutilizar**: para usar una parte hay que llevarse todo.

```java
public class ExpedienteAlumno {
    private String nombre;
    private String id;
    private String direccion;
    private double[] notas;

    public ExpedienteAlumno(String nombre, String id, String direccion, double[] notas) {
        this.nombre = nombre;
        this.id = id;
        this.direccion = direccion;
        this.notas = notas;
    }

    public double calcularNotaMedia() {
        double suma = 0;
        for (double nota : notas) {
            suma += nota;
        }
        return suma / notas.length;
    }

    public void imprimirCertificado() {
        System.out.println("Certificado académico");
        System.out.println("Nombre: " + nombre);
        System.out.println("ID: " + id);
        System.out.println("Dirección: " + direccion);
        System.out.println("Nota media: " + calcularNotaMedia());
    }

    public void enviarNotificacionPorCorreo(String email) {
        // código para enviar el correo
    }
}
```

Esta clase hace cuatro cosas distintas:

1. Guardar los datos del alumno.
2. Calcular la nota media.
3. Imprimir el certificado.
4. Enviar notificaciones por correo.

Si se añade un campo al alumno, puede afectar al cálculo, a la impresión y al envío de correos. Si se quiere reutilizar solo el cálculo de la media en otro contexto, hay que copiar la clase entera.

### 9.2. Cohesión alta

Una clase tiene **alta cohesión** cuando tiene **una única responsabilidad bien definida** y contiene toda la información y los recursos necesarios para cumplirla.

La herramienta principal para conseguirlo es el **principio de responsabilidad única** (la «S» de SOLID): **una clase debe tener un único motivo para cambiar**.

```java
import java.util.List;

public record Alumno(String id, String nombre, String direccion, String email, List<Double> notas) {

    public Alumno {
        notas = List.copyOf(notas);   // copia defensiva e inmutable
    }

    public double notaMedia() {
        return notas.stream()
                    .mapToDouble(Double::doubleValue)
                    .average()
                    .orElse(0);
    }
}
```

```java
public class GeneradorCertificado {

    public String generar(Alumno alumno) {
        return """
               Certificado académico
               Nombre: %s
               ID: %s
               Dirección: %s
               Nota media: %.2f
               """.formatted(alumno.nombre(), alumno.id(), alumno.direccion(), alumno.notaMedia());
    }
}
```

```java
public interface NotificadorAlumnos {
    void enviar(String destinatario, String mensaje);
}

public class NotificadorAlumnosCorreo implements NotificadorAlumnos {
    @Override
    public void enviar(String destinatario, String mensaje) {
        // código para enviar el correo
    }
}
```

Qué ha cambiado:

- `Alumno` contiene los datos **y** el cálculo que solo depende de ellos (`notaMedia`). Esto sigue el principio *Tell, Don't Ask*: la media se calcula donde están las notas.
- `GeneradorCertificado` solo sabe dar formato a un certificado. Además, **devuelve** un `String` en lugar de imprimir: quien lo use decide si lo muestra en consola, lo guarda en un PDF o lo envía. Esto también reduce el acoplamiento con `System.out`.
- El envío de correos está detrás de una interfaz, con lo que se puede cambiar por SMS o por una implementación falsa en las pruebas.
- Se usa un **bloque de texto** (`"""`, Java 15) con `formatted(...)` para la plantilla, más legible que varias concatenaciones.

Cada clase tiene ahora un propósito claro: es más fácil de entender, de reutilizar, de mantener y de probar, y los errores se localizan antes.

## 10. Tipos de cohesión

La clasificación clásica procede del diseño estructurado (Stevens, Myers y Constantine). Se ordena **de la peor a la mejor**:

| # | Tipo | Descripción | Ejemplo en Java |
|---|---|---|---|
| 1 | **Casual** (*coincidental*) | Tareas sin relación lógica, agrupadas por comodidad o por azar. | Una clase `Utilidades` con `formatearFecha`, `calcularIva` y `leerFichero`. |
| 2 | **Lógica** | Tareas de la misma «categoría»; quien llama elige cuál ejecutar (a menudo con un *flag*). | `procesar(tipo)` con un `switch` que hace cosas totalmente distintas según `tipo`. |
| 3 | **Temporal** | Tareas que solo tienen en común que se ejecutan **en el mismo momento** (arranque, cierre). | Un método `inicializar()` que abre la base de datos, carga traducciones y arranca una caché. |
| 4 | **Procedimental** | Tareas que deben ejecutarse **en un orden concreto**, aunque trabajen con datos distintos. | `validarFormulario(); registrarAuditoria(); mostrarPantalla();` |
| 5 | **Comunicacional** | Tareas que operan sobre **los mismos datos**. | Varios métodos que consultan y actualizan el mismo pedido. |
| 6 | **Secuencial** | La **salida** de una tarea es la **entrada** de la siguiente. | `leer → parsear → validar → guardar`, como en un *pipeline* de `Stream`. |
| 7 | **Funcional** | Todos los elementos contribuyen a **una única tarea bien definida**. | Un método `calcularLetraDni(int numero)`. |

Algunas fuentes añaden la **cohesión informacional**: un conjunto de datos junto con el grupo de operaciones que se pueden realizar sobre ellos. Es la que se observa habitualmente en las **clases bien diseñadas** (por ejemplo, el `record Alumno` anterior), mientras que la **funcional** es la propia de los **métodos** bien diseñados.

> **Objetivo práctico:** métodos con cohesión **funcional** y clases con cohesión **informacional/comunicacional**. Si encuentras cohesión casual o lógica, es una señal clara para refactorizar.

### Ejemplo de cohesión lógica y cómo mejorarla

```java
// ❌ Cohesión lógica: el llamante elige qué hacer con un parámetro
public void generarInforme(String tipo) {
    switch (tipo) {
        case "VENTAS"   -> { /* consultas y formato de ventas */ }
        case "STOCK"    -> { /* consultas y formato de stock */ }
        case "CLIENTES" -> { /* consultas y formato de clientes */ }
        default         -> throw new IllegalArgumentException(tipo);
    }
}
```

```java
// ✅ Cada informe es una clase cohesionada
public sealed interface Informe permits InformeVentas, InformeStock, InformeClientes {
    String generar();
}
```

Fíjate en que la cohesión lógica suele ir de la mano del **acoplamiento de control** (apartado 3.4): el parámetro `tipo` es precisamente un *flag* con el que el llamante dirige el flujo interno del método.

## 11. Buenas prácticas para conseguir alta cohesión

- **Una responsabilidad por clase:** todos sus métodos y datos deben estar relacionados con ella.
- **Encapsular juntos los datos y los métodos que los usan** (ver *Tell, Don't Ask*).
- **Diseñar el sistema como un conjunto de módulos pequeños y reutilizables**, cada uno con alta cohesión. Así es más fácil localizar problemas, ampliar y reutilizar.
- **Nombres precisos:** si una clase solo se puede llamar `Gestor`, `Utilidades` o `Helper`, probablemente hace demasiadas cosas.
- **Usar `record` para valores** y añadirles los métodos que dependen solo de sus componentes.
- **Usar patrones de diseño adecuados** cuando el problema lo pida (Estrategia, Fachada, etc.), sin forzarlos.

## 12. Cómo comprobar la cohesión de una clase

1. **Identifica sus responsabilidades.** ¿Cuál es su propósito principal? ¿Qué tareas realiza?
2. **Analiza sus métodos.** ¿Contribuyen todos al mismo propósito o hacen cosas sin relación?
3. **Comprueba si los métodos usan los mismos campos.** ¿Comparten funcionalidad? ¿Se apoyan unos en otros?
4. **Busca código duplicado** o métodos que hacen casi lo mismo con distintos parámetros.
5. **Evalúa si la clase tiene una responsabilidad única y clara.** Intenta describirla en una frase **sin usar la conjunción «y»**.

Si los métodos están relacionados con el mismo propósito, usan los mismos campos, comparten funcionalidad y no hay métodos irrelevantes ni código repetido, la clase tiene **alta cohesión**. Si los métodos no tienen relación, usan campos distintos y no comparten funcionalidad, la cohesión es **baja**.

> **Pista visual:** si puedes dividir los campos de una clase en dos grupos, y cada grupo de métodos solo usa uno de ellos, en realidad tienes **dos clases** dentro de una. Esta es la idea que mide la métrica **LCOM** (*Lack of Cohesion of Methods*), que calculan herramientas de análisis estático como SonarQube o los *plugins* de métricas de los IDE.

## 13. Cómo se influyen mutuamente

Cohesión y acoplamiento son conceptos **interdependientes**:

- **La alta cohesión suele ir acompañada de bajo acoplamiento.** Si los elementos de un módulo están muy relacionados y sirven a un único propósito, apenas necesitan interactuar con otros módulos.
- **El acoplamiento fuerte suele indicar baja cohesión.** Si dos módulos dependen mucho uno del otro, probablemente la responsabilidad esté repartida entre ambos y ninguno tenga un propósito claro.

```text
              Cohesión baja           Cohesión alta
            ┌─────────────────────┬─────────────────────┐
Acopl. alto │ «Bola de barro»:    │ Piezas claras pero  │
            │ todo depende de todo│ enredadas entre sí  │
            ├─────────────────────┼─────────────────────┤
Acopl. bajo │ Piezas aisladas     │ ✅ Diseño deseable   │
            │ pero confusas       │                     │
            └─────────────────────┴─────────────────────┘
```

Un síntoma típico de esta relación es la **«envidia de funcionalidad»** (*feature envy*): un método que usa más los datos de otra clase que los de la suya. Está mal ubicado: reduce la cohesión de su clase **y** aumenta el acoplamiento con la otra. Moverlo a la clase cuyos datos usa mejora ambas cosas a la vez.

## 14. Cohesión y acoplamiento a nivel de paquete y de módulo

Las dos ideas no se limitan a las clases. También se aplican a cómo se organizan los paquetes:

```text
❌ Organización por tipo técnico          ✅ Organización por funcionalidad
es.empresa.tienda                        es.empresa.tienda
├── controladores                        ├── pedidos
│   ├── PedidoController                 │   ├── PedidoController
│   └── ClienteController                │   ├── PedidoService
├── servicios                            │   └── PedidoRepository
│   ├── PedidoService                    └── clientes
│   └── ClienteService                       ├── ClienteController
└── repositorios                             ├── ClienteService
    ├── PedidoRepository                     └── ClienteRepository
    └── ClienteRepository
```

En la organización **por funcionalidad** (*package by feature*), todo lo que cambia junto está junto. Esto aumenta la cohesión de cada paquete y permite usar la visibilidad de paquete (sin `public`) para ocultar detalles internos, reduciendo el acoplamiento. Con el sistema de módulos de Java (`module-info.java`, apartado 6) se puede llevar esta idea más lejos, exportando solo los paquetes que forman la API pública.

## 15. Una analogía: la casa

- **Alta cohesión:** cada habitación de una casa tiene un propósito concreto (cocina, dormitorio, baño) y todo lo que contiene está relacionado con él. Es fácil saber qué hay en cada una y arreglar algo cuando se rompe. Una clase cohesionada es como una de esas habitaciones.
- **Bajo acoplamiento:** si queremos añadir una habitación, no queremos derribar la casa entera. Del mismo modo, añadir una funcionalidad no debería obligar a reescribir el sistema. Si las habitaciones se conectan de forma clara y bien definida, es fácil entender cómo funciona el conjunto y se reducen los efectos inesperados.

## 16. Lista de comprobación

**Acoplamiento**

- [ ] ¿Mis clases crean sus dependencias con `new` o las reciben desde fuera?
- [ ] ¿Dependo de interfaces o de clases concretas?
- [ ] ¿Expongo colecciones o estructuras internas mutables?
- [ ] ¿Hay estado global (`static` mutable)?
- [ ] ¿Hay métodos con parámetros `boolean` o de «modo» que cambian su comportamiento?
- [ ] ¿Paso objetos enteros cuando solo necesito un dato?
- [ ] ¿Hay cadenas de llamadas `a.getB().getC()...`?
- [ ] ¿Puedo probar la clase de forma aislada?

**Cohesión**

- [ ] ¿Puedo describir cada clase en una frase sin «y»?
- [ ] ¿Todos los métodos de la clase usan los campos de la clase?
- [ ] ¿Hay métodos que usan más datos de otra clase que de la suya (*feature envy*)?
- [ ] ¿Hay clases llamadas `Utilidades`, `Gestor` o `Helper` que crecen sin parar?
- [ ] ¿Los paquetes agrupan lo que cambia junto?

## 17. Referencias

- MSMK University — *Acoplamiento en Desarrollo de Software*: <https://msmk.university/acoplamiento-en-desarrollo-de-software/>
- EnjoyAlgorithms — *Cohesion and Coupling in Object Oriented Programming (OOPS)*: <https://www.enjoyalgorithms.com/blog/cohesion-and-coupling-in-oops>
- JEP 378 — *Text Blocks* (Java 15): <https://openjdk.org/jeps/378>
- JEP 395 — *Records* (Java 16): <https://openjdk.org/jeps/395>
- JEP 409 — *Sealed Classes* (Java 17): <https://openjdk.org/jeps/409>
- JEP 506 — *Scoped Values* (Java 25): <https://openjdk.org/jeps/506>
- W. P. Stevens, G. J. Myers y L. L. Constantine — *Structured Design*, IBM Systems Journal (1974).
