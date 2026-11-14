# 📚 Documentación de Java

Repositorio de documentación, apuntes, ejemplos y guías sobre **Java**, creado con el objetivo de recopilar y organizar diferentes conceptos relacionados con el desarrollo en este lenguaje.

> ☕ **Versión de referencia: Java 25 (LTS) o superior.**
>
> Toda la documentación está enfocada en Java 25 en adelante. Cuando una característica apareció en una versión anterior se indica desde cuál está disponible, y lo que solo existe en vista previa (*preview*) o en versiones posteriores (Java 26+) se señala de forma explícita. Los programas de ejemplo completos se han compilado y ejecutado con OpenJDK 25.0.5 (y las guías de pruebas, con JUnit 6.1.3 y AssertJ 3.27.7; la guía de Mockito, con Mockito 5.20.0).

> 🚧 **Proyecto en desarrollo**
>
> Esta documentación se encuentra actualmente **en pleno proceso de desarrollo y ampliación**. Se irán incorporando progresivamente nuevos temas, conceptos, ejemplos y buenas prácticas relacionados con Java.
>
> El contenido actual es solo una parte de la documentación que se pretende reunir en este repositorio.

---

## 📑 Índice

- [Contenido disponible](#-contenido-disponible)
- [Documentación en desarrollo](#-documentación-en-desarrollo)
- [Objetivos del proyecto](#-objetivos-del-proyecto)
- [Estructura actual](#️-estructura-actual)
- [Ejemplo](#-ejemplo)
- [Tecnologías y herramientas](#️-tecnologías-y-herramientas)
- [Cómo utilizar este repositorio](#-cómo-utilizar-este-repositorio)
- [¿Por dónde empezar?](#-por-dónde-empezar)
- [Contribuciones](#-contribuciones)
- [Estado del proyecto](#-estado-del-proyecto)
- [Licencia](#-licencia)

---

## 📖 Contenido disponible

Actualmente, el repositorio cuenta con documentación sobre diferentes conceptos de Java y herramientas relacionadas con su desarrollo.

### 🧩 Fundamentos del lenguaje, POO y colecciones

| Documento                                                                                                         | Qué encontrarás                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Fundamentos de Java](<01-Fundamentos-del-lenguaje/Fundamentos de Java.md>)                                       | Bases del lenguaje en Java 25+: JDK y JVM, primer programa (clásico y fichero fuente compacto con `void main()` e `IO`), variables, `var` y constantes, tipos primitivos y literales, `String`, clases envoltorio y conversiones; operadores y precedencia; `if`, `switch` (flechas, expresión con `yield` y patrones) y bucles; arrays uni y multidimensionales y la clase `Arrays`; métodos, paso por valor, sobrecarga, *varargs*, recursividad y ámbito. Incluye un ejemplo completo, errores frecuentes, chuleta, ejercicios con soluciones y glosario.                                                                                                                                                                                 |
| [Herencia, encapsulación y polimorfismo](<01-Fundamentos-del-lenguaje/Herencia, encapsulacion y polimorfismo.md>) | Los pilares de la POO en Java 25+: repaso de clases y objetos; encapsulación (modificadores de acceso, *getters*/*setters* con criterio, invariantes, inmutabilidad, copias defensivas y *records*); herencia (`extends`, `super`, orden de construcción, constructores flexibles de Java 25, sobrescritura y `@Override`, `toString`/`equals`/`hashCode`, `final`, clases abstractas, interfaces, clases selladas y composición frente a herencia); polimorfismo (tipo declarado y real, enlace dinámico, *upcasting*/*downcasting*, `instanceof` con patrones y `switch` exhaustivo con tipos sellados y patrones de *record*). Incluye un ejemplo completo de nóminas, errores frecuentes, chuleta, ejercicios con soluciones y glosario. |
| [Abstract Class vs Interface](<01-Fundamentos-del-lenguaje/Abstract Class vs Interface.md>)                       | Chuleta comparativa: constructores, estado, modificadores de acceso, herencia, métodos `final`, polimorfismo y acoplamiento. Incluye cuándo usar cada una, cómo combinarlas, lo que aporta Java moderno (tipos `sealed`, *records* que implementan interfaces y código antes de `super(...)` en Java 25), errores frecuentes, reglas mentales y una chuleta ultra rápida.                                                                                                                                                                                                                                                                                                                                                                    |
| [Colecciones](<01-Fundamentos-del-lenguaje/Colecciones.md>)                                                       | Resumen del *Java Collections Framework*: jerarquía de interfaces, `List`, `Set`, `Map`, `Queue` y `Deque`, colecciones inmutables, *Sequenced Collections* (Java 21), la clase `Collections`, genéricos (con `HashMap.newHashMap`), Stream API (`toList()` y *gatherers*) y colecciones concurrentes con hilos virtuales.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| [Comparable vs Comparator](<01-Fundamentos-del-lenguaje/Comparable vs Comparator.md>)                             | Guía de estudio para programadores (Java 25 o superior), con ejemplos compilados y ejecutados en OpenJDK 25, errores típicos, colecciones secuenciadas, `Comparator.min`/`max` (Java 26), ejercicios, chuleta de repaso y autoevaluación.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [Generics](<01-Fundamentos-del-lenguaje/Generics.md>)                                                             | Guía de genéricos: clases, interfaces y métodos genéricos, *raw types*, inferencia de tipos, tipos acotados, comodines (*wildcards*), borrado de tipos (*type erasure*), restricciones, uso con características modernas de Java, errores habituales, chuleta y ejercicios.                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [Enums](<01-Fundamentos-del-lenguaje/Enums.md>)                                                                   | Enumeraciones (`enum`): declaración, uso dentro de clases y en `switch`, recorrido de constantes, diferencias con las clases y cuándo usarlas. Ampliación con enums «con cuerpo» (atributos, constructor, interfaces), `EnumMap` y `EnumSet`, y Java moderno (`switch` con patrones, exhaustividad, `case null`, variables sin nombre `_`, constructores flexibles de Java 25 y tipos primitivos en patrones en vista previa).                                                                                                                                                                                                                                                                                                               |
| [Records](<01-Fundamentos-del-lenguaje/Records.md>)                                                               | Clases de datos inmutables: declaración, constructor compacto, métodos y miembros estáticos, interfaces, cuándo usarlos y reglas (JEP 395). Ampliación con copia defensiva, records locales, anotaciones, reflexión, interfaces `sealed`, patrones de record (Java 21), variables sin nombre (`_`), constructores flexibles (Java 25) y tipos primitivos en patrones (vista previa).                                                                                                                                                                                                                                                                                                                                                         |
| [Calistenia de objetos](<07-Diseno/Principios/Calistenia de objetos.md>)                                          | Las nueve reglas de *Object Calisthenics* (Jeff Bay) actualizadas a Java 25: un nivel de indentación, sin `else`, encapsular primitivos, colecciones de primera clase, un punto por línea, no abreviar, entidades pequeñas, dos variables de instancia y sin *getters*/*setters*. Cada regla con ejemplos antes/después apoyados en *records*, `sealed`, `switch` con patrones, *gatherers* y ficheros fuente compactos, más un ejemplo completo (carrito de la compra), tabla resumen y cómo aplicarlas en katas y en producción.                                                                                                                                                                                                           |

### 🏛️ Principios de diseño

| Documento | Qué encontrarás |
| --- | --- |
| [Cohesión y acoplamiento](<07-Diseno/Principios/Cohesion y acoplamiento.md>) | Las dos medidas clásicas de la calidad del diseño (Java 25): tipos de acoplamiento (contenido, común, externo, control, sello, datos y mensajes), ejemplos de acoplamiento fuerte a débil (`Coche` y `Motor`, servicio de notificaciones), técnicas para reducirlo y cómo detectarlo; tipos de cohesión, ejemplos de cohesión baja y alta, cómo comprobarla, relación entre ambas, nivel de paquete y módulo, analogía de la casa y lista de comprobación. |
| [Principios SOLID](<07-Diseno/Principios/Principio SOLID.md>) | Los cinco principios (SRP, OCP, LSP, ISP y DIP) con ejemplos de incumplimiento y solución en Java 25: el clásico cuadrado-rectángulo, ISP en la biblioteca estándar, por qué se llama «inversión», cómo encajan entre sí, SOLID y Java moderno, errores frecuentes, tabla resumen y lista de comprobación. |
| [Principio DRY](<07-Diseno/Principios/Principio DRY.md>) | *Don't Repeat Yourself*: origen, duplicar código frente a duplicar conocimiento, ejemplo de validación de correos, técnicas para eliminar la duplicación y herramientas de Java moderno (`record`, `enum` con comportamiento, lambdas, constructores flexibles de Java 25), violaciones habituales, relación con SRP y cuándo **no** aplicarlo. |
| [Principio KISS](<07-Diseno/Principios/Principio KISS.md>) | *Keep It Simple, Stupid*: origen, por qué simple no significa fácil ni corto, estrategias, ejemplo de conversor de temperaturas y herramientas de Java moderno que simplifican el código (ficheros fuente compactos, importación de módulos, `record`, `switch` con patrones y `sealed`, `_`, *guard clauses*, `var`), señales de complejidad y relación con otros principios. |
| [Tell, Don't Ask](<07-Diseno/Principios/Tell dont ask.md>) | «Ordena, no preguntes»: *getters*/*setters* frente a campos públicos y cómo pueden perjudicar, el principio con un ejemplo de cuenta bancaria, la Ley de Deméter, aplicación en Java moderno (métodos de acceso de los `record`, inmutabilidad, polimorfismo en lugar de preguntar por el tipo), cuándo está bien preguntar y relación con otros principios. |

### 🧱 Patrones de diseño

| Documento | Qué encontrarás |
| --- | --- |
| [Patrones creacionales](<07-Diseno/Patrones/Patrones creacionales.md>) | Qué es un patrón de diseño (con analogías), cómo probar los ejemplos con Java 25 y vocabulario mínimo. Patrones para crear objetos: Singleton (con `enum`), Factory Method, Abstract Factory, Builder (con `record`) y Prototype (copia superficial y profunda). Cada patrón con analogía de la vida real, problema, solución paso a paso, programa completo ejecutable con su salida, cuándo usarlo y cuándo no. Incluye cómo elegir, errores frecuentes, ejercicios con soluciones y glosario |
| [Patrones estructurales](<07-Diseno/Patrones/Patrones estructurales.md>) | La idea común de envolver objetos (composición frente a herencia) y los patrones Adapter (sensor en Fahrenheit), Composite (carpetas y ficheros), Decorator (café con extras), Facade (proceso de compra) y Proxy (caché), cada uno con analogía, problema, solución paso a paso y programa completo ejecutable. Incluye una tabla para distinguir los patrones que se parecen, errores frecuentes, ejercicios con soluciones y glosario |
| [Patrones de comportamiento](<07-Diseno/Patrones/Patrones comportamiento.md>) | Introducción a las *lambdas* y los patrones Strategy (gastos de envío), Observer (suscripciones), Command (editor con deshacer), State (ciclo de vida de un pedido con `enum`), Template Method (preparar bebidas) e Iterator (`Iterable` y *for-each*), cada uno con analogía, problema, solución paso a paso y programa completo ejecutable. Incluye cómo elegir, errores frecuentes, ejercicios con soluciones y glosario |

### λ Programación funcional

| Documento | Qué encontrarás |
| --- | --- |
| [Programación funcional](<02-Programacion-funcional/Programacion funcional.md>) | El paradigma funcional en Java 25+: imperativo frente a declarativo, funciones puras y efectos secundarios (núcleo funcional y capa imperativa), inmutabilidad y métodos «with» en *records*, transparencia referencial, funciones de primera clase y de orden superior, composición (`andThen`, `compose`, `Predicate`, `Comparator`), clausuras, currificación, evaluación perezosa, memoización, límites de la recursividad, tipos de datos algebraicos con `sealed` + `record` y *pattern matching*, `Optional`, errores como valores (tipo `Resultado`), streams sin efectos secundarios, *gatherers* y paralelismo. Incluye un ejemplo completo de pedidos, errores frecuentes, chuleta, ejercicios con soluciones y glosario. |
| [Lambdas y Streams](<02-Programacion-funcional/Lambdas y Streams.md>) | Guía de estudio de Java 25+: sintaxis de lambdas, interfaces funcionales, inferencia de tipos, captura de variables y referencias a métodos; *pipelines* de `Stream` (operaciones intermedias y terminales), `Collectors`, streams de primitivos, `Optional`, streams paralelos y rendimiento. Incluye Java moderno de la 16 a la 25 (`record`, `Stream.toList()`, *pattern matching*, hilos virtuales, `Gatherers`, ficheros fuente compactos con `void main()` e `import module`), errores típicos, ejercicios con soluciones y chuleta final. |

### 🧬 Anotaciones y reflexión

| Documento | Qué encontrarás |
| --- | --- |
| [Anotaciones](<03-Anotaciones-y-reflexion/Anotaciones.md>) | Guía de anotaciones en Java 25+: sintaxis, anotaciones predefinidas (`@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`…), meta-anotaciones (`@Retention`, `@Target`, `@Inherited`, `@Repeatable`), creación de anotaciones propias, lectura en tiempo de ejecución, anotaciones de tipo, uso con `record`, paquetes y módulos, procesadores de anotaciones, un validador de campos de ejemplo, errores comunes, ejercicios con soluciones y glosario. |
| [Reflections](<03-Anotaciones-y-reflexion/Reflections.md>) | Guía de la API de *Reflection* (Java 25+): obtener `Class`, inspeccionar constructores, campos y métodos, acceso a miembros privados con `setAccessible`, excepciones habituales, *records* y clases selladas, anotaciones en ejecución, genéricos y borrado de tipos, proxies dinámicos, sistema de módulos, `MethodHandle` y `VarHandle`, rendimiento, GraalVM *Native Image*, mini proyecto de validador, buenas prácticas y ejercicios. |

### 📂 Entrada/salida y ficheros

| Documento | Qué encontrarás |
| --- | --- |
| [Entrada - Salida básica](<04-Entrada-salida-y-ficheros/Entrada - Salida basica.md>) | Guía de E/S en Java 25+: flujos de bytes y de caracteres, flujos con búfer, `Scanner` y `printf`, E/S desde la línea de comandos (`Console.isTerminal()` y la clase `IO` de Java 25), *Data Streams*, serialización de objetos, ficheros con NIO.2 y E/S con hilos virtuales, además de errores frecuentes, ejercicios y recursos para seguir aprendiendo. |
| [Rutas relativas](<04-Entrada-salida-y-ficheros/Rutas relativas.md>) | Qué es una ruta relativa, notación especial (`.` y `..`), directorio de trabajo actual, `java.io.File` frente a `java.nio.file.Path`, rutas relativas al proyecto y al *classpath*, diferencias entre sistemas operativos, *Path Traversal* y errores comunes (Java 25+). |

### 🕒 Fechas y horas

| Documento | Qué encontrarás |
| --- | --- |
| [Fechas](<05-Fechas-y-horas/Fechas.md>) | API `java.time`: fecha y hora actuales, `DateTimeFormatter` y patrones, qué clase usar en cada caso, inmutabilidad, `Period`, `Duration` y `ChronoUnit`, zonas horarias e instantes, `Clock` para pruebas, *parsing* y modos de resolución, y cambios recientes que afectan a Java 25 (CLDR, eras, espacios laxos). Ejemplos verificados con OpenJDK 25. |

### ⚠️ Excepciones y gestión de errores

| Documento | Qué encontrarás |
| --- | --- |
| [Excepciones checked y unchecked](<06-Excepciones/Excepciones checked y unchecked.md>) | Diferencias entre excepciones comprobadas y no comprobadas, ejemplos con `throws`, `try-catch` y `try-with-resources`, excepciones personalizadas, cuándo usar cada tipo y Java moderno (*multi-catch*, `catch (X _)`, encadenamiento, excepciones en lambdas y la rama `Error`). |
| [Manejo eficaz de errores en Java: estrategias y buenas prácticas](<06-Excepciones/Manejo Eficaz de Errores en Java - Estrategias y Buenas Practicas.md>) | Resumen de un artículo sobre buenas prácticas, técnicas de manejo (`try-catch-finally`, *multi-catch*, `try-with-resources`), diseño de excepciones personalizadas y registro de errores, con un complemento sobre Java 25 (mensajes de `NullPointerException`, excepciones suprimidas, `_` en `catch` y resultados con tipos `sealed`). |

### 🧪 Testing y pruebas automatizadas

| Documento | Qué encontrarás |
| --- | --- |
| [JUnit: visión general e introducción](<08-Testing/JUnit - Vision general e introduccion.md>) | Traducción al castellano de la página oficial de JUnit 6.1.3, más un complemento práctico: puesta en marcha con Maven y Gradle (proyectos en Java 25), anotaciones, aserciones, errores frecuentes y glosario. |
| [JUnit: aprende a escribir pruebas paso a paso](<08-Testing/JUnit - Aprende a escribir pruebas paso a paso.md>) | Guía práctica con JUnit 6.1.3 y Java 25: primera prueba, ciclo de vida, aserciones, excepciones, pruebas parametrizadas, anidadas, repetidas y dinámicas, etiquetas, *timeouts* y buenas prácticas. |
| [AssertJ: guía para empezar desde cero](<08-Testing/AssertJ - Guia para empezar desde cero.md>) | Aserciones fluidas con `assertThat(...)`: tipos de dato, colecciones, excepciones, mensajes de error, *soft assertions*, comparación recursiva y estilo BDD. |
| [Test-Driven Development (TDD)](<08-Testing/Test-Driven Development (TDD).md>) | Guía paso a paso del ciclo Rojo → Verde → Refactor (Maven, JUnit 6 y Java 25), con dos ejemplos completos (factorial y validador de contraseñas), buenas prácticas, errores típicos y ejercicios. |
| [Mockito: aprende a escribir pruebas paso a paso](<08-Testing/Mockito - Aprende a escribir pruebas paso a paso.md>) | Tests unitarios con *mocks* usando Mockito 5 y JUnit 6.1.3 en Java 25: dónde obtener la librería y configuración (Maven, Gradle, IntelliJ y agente de Mockito), *stubbing*, *argument matchers*, `verify`, `ArgumentCaptor`, *spies*, métodos estáticos, estilo BDD, errores habituales y chuleta rápida. |

### 🔐 Seguridad y criptografía

| Documento | Qué encontrarás |
| --- | --- |
| [Guía de seguridad](<11-Seguridad/Guia de seguridad.md>) | Introducción a la *Security Developer's Guide* de Java SE (Java 25+): mapa de la seguridad en Java (JCA/JCE, PKI, JSSE, JAAS, SASL, firma de código, serialización segura), cambios de Java 17 a Java 25 (fin del *Security Manager*, ML-KEM, ML-DSA y API `KDF`), herramientas de línea de comandos (`keytool`, `jarsigner`), buenas prácticas y un ejemplo mínimo de hash SHA-256. |
| [Arquitectura criptográfica](<11-Seguridad/Arquitectura criptografica.md>) | Guía de referencia de la JCA (Java 25+): principios de diseño, proveedores, clase `Security`, clases *engine*, `SecureRandom`, `MessageDigest`, `Signature`, `Cipher`, `Mac`, claves, `KEM` con ML-KEM post-cuántico, derivación de claves con `KDF`/HKDF, `KeyStore`, certificados, excepciones habituales, buenas prácticas y glosario. |

### 🗄️ Bases de datos

| Documento | Qué encontrarás |
| --- | --- |
| [BBDD: tipos de datos](<10-Bases-de-datos/SQL/BBDD - Tipos de datos.md>) | Tipos de datos de MySQL (estado en 2026: 9.7 LTS, 8.4 LTS y fin de vida de 8.0): numéricos, cadena (`CHAR`, `VARCHAR`, `BLOB`, `TEXT`, `ENUM`, `SET`, `VECTOR`), fecha y hora, atributos de columna, guía rápida de elección, correcciones respecto al artículo original y fuentes. |
| [Entidad-Relación: diagrama](<10-Bases-de-datos/SQL/Entidad - Relacion - Diagrama.md>) | Diagrama Entidad-Relación: entidades, relaciones y atributos, cardinalidad, notaciones de Chen y *Crow's Foot*, creación paso a paso, buenas prácticas, comparación con UML, preguntas frecuentes y paso del diagrama a tablas MySQL. |
| [Entidad-Relación: PlantUML](<10-Bases-de-datos/SQL/Entidad - Relacion - PlantUML.md>) | Guía para dibujar diagramas E/R como código con PlantUML: instalación y herramientas, entidades, clases y enums, cardinalidad en notación pata de gallo, relaciones de diagrama de clases, casos típicos (1:1, 1:N, N:M, reflexiva, jerarquía, ternaria), notación de Chen, paquetes y notas, estilos y temas, macros del preprocesador, ejemplo completo de tienda online con su SQL, buenas prácticas, errores frecuentes y chuleta. |
| [NoSQL y diferencias con SQL](<10-Bases-de-datos/NoSQL/Diferencias entre nosql y sql.md>) | Qué es NoSQL y sus tipos (documentales, clave-valor, columnas anchas, grafos y multimodelo), breve historia, características, ACID frente a BASE y teorema CAP, ejemplo relacional frente a documental, diferencias entre RDBMS y NoSQL, equivalencias de conceptos y consultas SQL ↔ MongoDB (MQL), cuándo elegir cada opción, mitos habituales y preguntas frecuentes (revisado para MongoDB 9.0). |
| [MongoDB: modelado de datos y diseño](<10-Bases-de-datos/NoSQL/MongoDB - modelado de datos y diseno.md>) | Modelado de datos en MongoDB: vocabulario básico (embeber, referenciar, cardinalidad…), diferencias con el diseño relacional con ejemplos SQL ↔ MongoDB, proceso de diseño del esquema en 4 pasos, relaciones (1:1, 1:pocos, 1:N, N:M y `$lookup`), embeber o referenciar, duplicación y coherencia, validación de esquemas, índices, atomicidad y *sharding*, patrones de diseño explicados como problema → solución (*Computed*, *Bucket*, *Outlier*, *Subset*, *Extended Reference*, versionado…), antipatrones, estructuras de árbol, ejemplo de tienda en línea y lista de comprobación. |
| [MongoDB: tutoriales](<10-Bases-de-datos/NoSQL/MongoDB - tutoriales.md>) | Índice organizado de los tutoriales oficiales del manual (MongoDB 9.0) con guía rápida en `mongosh` y explicación de cada comando (equivalencias SQL, respuestas esperadas y errores típicos): CRUD, índices y `explain()`, agregación y transacciones; conjuntos de réplicas, *sharding*, administración y copias de seguridad, seguridad, patrones de desarrollo y de búsqueda de texto, novedades de la 9.0 y ruta de aprendizaje recomendada. |

### 📝 Logging y registro de información

| Documento | Qué encontrarás |
| --- | --- |
| [Log4J: gestión de registros](<09-Logging/Log4J - Gestion de Registros.md>) | Resumen conceptual: niveles de *logging* y cuándo usar cada uno, configuración, *appenders* y *layouts*. |
| [Log4J: guía](<09-Logging/Log4J - Guia.md>) | Guía práctica de Log4j 2 (2.26.1, Java 25): instalación con Maven y Gradle, `log4j2.xml`, mensajes parametrizados, escritura en ficheros (`RollingFile`), `ThreadContext`, integración con SLF4J y problemas frecuentes. |

### 🔧 Herramientas y buenas prácticas

| Documento | Qué encontrarás |
| --- | --- |
| [Conventional Commits](<12-Herramientas/Conventional Commits.md>) | Convención para escribir mensajes de *commit* claros: formato, tipos habituales (`feat`, `fix`, `docs`…), ejemplos, buenas prácticas y flujo básico. |

---

## 🚧 Documentación en desarrollo

Este repositorio está pensado para **crecer progresivamente**. Actualmente solo se han añadido algunos temas de Java y bases de datos, pero el objetivo es continuar ampliando la documentación con nuevos contenidos, ejemplos y explicaciones.

**Estado de los temas previstos:**

- [x] Clases abstractas e interfaces
- [x] Colecciones
- [x] `Comparable` y `Comparator`
- [x] Excepciones
- [x] Testing con JUnit
- [x] AssertJ
- [x] TDD
- [x] Logging con Log4j
- [x] Conventional Commits
- [x] Seguridad y criptografía (JCA)
- [x] Anotaciones
- [x] Reflexión (*Reflection*)
- [x] Fundamentos de Java
- [x] Variables y tipos de datos
- [x] Operadores
- [x] Estructuras de control
- [x] Arrays
- [x] Métodos
- [x] Herencia, encapsulación y polimorfismo
- [x] Enumeraciones (`enum`)
- [x] Records
- [x] Genéricos
- [x] Streams y expresiones lambda
- [x] `Optional`
- [x] Programación funcional
- [x] Entrada y salida de datos y ficheros
- [x] Fechas y horas
- [ ] Concurrencia, hilos y sincronización
- [x] Mockito
- [ ] Maven y Gradle
- [x] Bases de datos: tipos de datos MySQL y diagramas Entidad-Relación (incluido PlantUML)
- [x] Bases de datos NoSQL: diferencias con SQL y MongoDB (modelado de datos y tutoriales)
- [ ] JDBC y acceso a bases de datos desde Java
- [x] Calistenia de objetos (*Object Calisthenics*)
- [x] Patrones de diseño (creacionales, estructurales y de comportamiento)
- [x] Principios SOLID
- [x] Principios DRY y KISS
- [x] Cohesión y acoplamiento
- [x] *Tell, Don't Ask* y Ley de Deméter
- [ ] Arquitectura de aplicaciones
- [ ] Spring y Spring Boot
- [ ] Otros conceptos relacionados con el ecosistema Java

> 💡 **La lista anterior es orientativa y se irá ampliando a medida que avance el proyecto.**

---

## 🎯 Objetivos del proyecto

Los principales objetivos de este repositorio son:

- 📚 Crear una documentación amplia sobre Java.
- 🧠 Facilitar el aprendizaje del lenguaje.
- 🔎 Servir como referencia para consultar conceptos concretos.
- 💻 Incluir ejemplos prácticos siempre que sea posible.
- 🧪 Documentar herramientas utilizadas habitualmente en proyectos Java.
- ✅ Recopilar buenas prácticas de desarrollo.
- 📈 Ampliar progresivamente los contenidos.
- 🗂️ Mantener la información organizada y fácilmente accesible.

La intención es que el repositorio pueda servir tanto para **aprender Java** como para **consultar conceptos concretos durante el desarrollo de proyectos**.

---

## 🗂️ Estructura actual

La estructura del repositorio se irá modificando a medida que se incorporen nuevos contenidos.

```
Documentacion-Java/
├── .gitignore
├── 01-Fundamentos-del-lenguaje/
│   ├── Abstract Class vs Interface.md
│   ├── Colecciones.md
│   ├── Comparable vs Comparator.md
│   ├── Enums.md
│   ├── Fundamentos de Java.md
│   ├── Generics.md
│   ├── Herencia, encapsulacion y polimorfismo.md
│   └── Records.md
├── 02-Programacion-funcional/
│   ├── Lambdas y Streams.md
│   └── Programacion funcional.md
├── 03-Anotaciones-y-reflexion/
│   ├── Anotaciones.md
│   └── Reflections.md
├── 04-Entrada-salida-y-ficheros/
│   ├── Entrada - Salida basica.md
│   └── Rutas relativas.md
├── 05-Fechas-y-horas/
│   └── Fechas.md
├── 06-Excepciones/
│   ├── Excepciones checked y unchecked.md
│   └── Manejo Eficaz de Errores en Java - Estrategias y Buenas Practicas.md
├── 07-Diseno/
│   ├── Patrones/
│   │   ├── Patrones comportamiento.md
│   │   ├── Patrones creacionales.md
│   │   └── Patrones estructurales.md
│   └── Principios/
│       ├── Calistenia de objetos.md
│       ├── Cohesion y acoplamiento.md
│       ├── Principio DRY.md
│       ├── Principio KISS.md
│       ├── Principio SOLID.md
│       └── Tell dont ask.md
├── 08-Testing/
│   ├── AssertJ - Guia para empezar desde cero.md
│   ├── JUnit - Aprende a escribir pruebas paso a paso.md
│   ├── JUnit - Vision general e introduccion.md
│   ├── Mockito - Aprende a escribir pruebas paso a paso.md
│   └── Test-Driven Development (TDD).md
├── 09-Logging/
│   ├── Log4J - Gestion de Registros.md
│   └── Log4J - Guia.md
├── 10-Bases-de-datos/
│   ├── NoSQL/
│   │   ├── Diferencias entre nosql y sql.md
│   │   ├── MongoDB - modelado de datos y diseno.md
│   │   └── MongoDB - tutoriales.md
│   └── SQL/
│       ├── BBDD - Tipos de datos.md
│       ├── Entidad - Relacion - Diagrama.md
│       └── Entidad - Relacion - PlantUML.md
├── 11-Seguridad/
│   ├── Arquitectura criptografica.md
│   └── Guia de seguridad.md
├── 12-Herramientas/
│   └── Conventional Commits.md
├── LICENSE
└── README.md
```

Esta estructura **no es definitiva** y se irá ampliando junto con la documentación.

---

## 🧪 Ejemplo

Un ejemplo sencillo de una prueba utilizando JUnit:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void sumarDosNumeros() {
        int resultado = 2 + 3;

        assertEquals(5, resultado);
    }
}
```

Y la misma comprobación con la sintaxis fluida de AssertJ:

```java
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;

class CalculadoraAssertJTest {

    @Test
    void sumarDosNumeros() {
        int resultado = 2 + 3;

        assertThat(resultado).isEqualTo(5);
    }
}
```

La documentación del repositorio incluye material más detallado sobre [JUnit](<08-Testing/JUnit - Aprende a escribir pruebas paso a paso.md>), [AssertJ](<08-Testing/AssertJ - Guia para empezar desde cero.md>), [Mockito](<08-Testing/Mockito - Aprende a escribir pruebas paso a paso.md>) y [TDD](<08-Testing/Test-Driven Development (TDD).md>).

---

## 🛠️ Tecnologías y herramientas

Actualmente, el repositorio contiene documentación relacionada con:

- ☕ Java 25 LTS como versión de referencia (colecciones, genéricos, lambdas y streams, excepciones, clases abstractas e interfaces, ordenación, enums, records, fechas y horas, entrada/salida y rutas de ficheros)
- 🏋️ Diseño orientado a objetos (calistenia de objetos, principios SOLID, DRY, KISS y *Tell, Don't Ask*, cohesión y acoplamiento con Java 25)
- 🧱 Patrones de diseño GoF (creacionales, estructurales y de comportamiento con Java 25+)
- 🧬 Anotaciones y *Reflection* (metaprogramación en Java)
- 🗄️ Bases de datos relacionales (tipos de datos de MySQL y diagramas Entidad-Relación)
- 🍃 Bases de datos NoSQL y MongoDB (diferencias con SQL, modelado de datos, `mongosh`, CRUD, índices y agregación)
- 🌱 PlantUML (diagramas Entidad-Relación como código)
- 🔐 Seguridad y criptografía en Java (JCA/JCE)
- 🧪 JUnit 6
- 🔍 AssertJ
- 🎭 Mockito 5 (*mocks*, *stubs*, *spies* y verificación de interacciones)
- 🔄 Test-Driven Development (TDD)
- 📝 Log4j 2 (2.26.x)
- 🌿 Git (convención de *commits* con Conventional Commits)
- 📦 Maven y 🏗️ Gradle (configuración de dependencias dentro de las guías de JUnit, AssertJ, Mockito, TDD y Log4j)

Esta lista también se irá ampliando a medida que se incorporen nuevos temas.

---

## 🚀 Cómo utilizar este repositorio

Puedes clonar el repositorio utilizando Git:

```bash
git clone https://github.com/Victor-369/Documentacion-Java.git
```

Después puedes abrir los archivos `.md` con cualquier editor compatible con Markdown o consultar directamente la documentación desde GitHub.

---

## 📚 ¿Por dónde empezar?

Puedes utilizar esta documentación como material de consulta e ir avanzando progresivamente.

No es necesario seguir un orden concreto, ya que cada documento está pensado para poder consultarse de manera independiente. Aun así, si no sabes por dónde empezar, esta ruta puede servirte de orientación:

1. [Fundamentos de Java](<01-Fundamentos-del-lenguaje/Fundamentos de Java.md>): variables, operadores, estructuras de control, arrays y métodos.
2. [Herencia, encapsulación y polimorfismo](<01-Fundamentos-del-lenguaje/Herencia, encapsulacion y polimorfismo.md>): clases, objetos y los pilares de la programación orientada a objetos.
3. [Abstract Class vs Interface](<01-Fundamentos-del-lenguaje/Abstract Class vs Interface.md>) y [Colecciones](<01-Fundamentos-del-lenguaje/Colecciones.md>): base del lenguaje.
4. [Enums](<01-Fundamentos-del-lenguaje/Enums.md>) y [Records](<01-Fundamentos-del-lenguaje/Records.md>): tipos de datos propios.
5. [Generics](<01-Fundamentos-del-lenguaje/Generics.md>) y [Comparable vs Comparator](<01-Fundamentos-del-lenguaje/Comparable vs Comparator.md>): tipos genéricos y ordenación de objetos.
6. [Lambdas y Streams](<02-Programacion-funcional/Lambdas y Streams.md>): programación funcional y procesamiento de datos.
7. [Programación funcional](<02-Programacion-funcional/Programacion funcional.md>): el paradigma funcional (funciones puras, inmutabilidad, composición y tipos de datos con `sealed` + `record`).
8. [Calistenia de objetos](<07-Diseno/Principios/Calistenia de objetos.md>): reglas para practicar un diseño orientado a objetos más limpio (se apoya en *records*, `sealed` y *streams*).
9. [Cohesión y acoplamiento](<07-Diseno/Principios/Cohesion y acoplamiento.md>), [Principios SOLID](<07-Diseno/Principios/Principio SOLID.md>), [DRY](<07-Diseno/Principios/Principio DRY.md>), [KISS](<07-Diseno/Principios/Principio KISS.md>) y [Tell, Don't Ask](<07-Diseno/Principios/Tell dont ask.md>): principios de diseño para escribir código mantenible.
10. [Patrones creacionales](<07-Diseno/Patrones/Patrones creacionales.md>), [Patrones estructurales](<07-Diseno/Patrones/Patrones estructurales.md>) y [Patrones de comportamiento](<07-Diseno/Patrones/Patrones comportamiento.md>): soluciones reutilizables a problemas de diseño habituales (conviene leerlas en este orden; no hace falta conocer SOLID previamente).
11. [Excepciones checked y unchecked](<06-Excepciones/Excepciones checked y unchecked.md>) y [Manejo eficaz de errores](<06-Excepciones/Manejo Eficaz de Errores en Java - Estrategias y Buenas Practicas.md>): gestión de errores.
12. [Fechas](<05-Fechas-y-horas/Fechas.md>): fechas y horas con `java.time`.
13. [Entrada - Salida básica](<04-Entrada-salida-y-ficheros/Entrada - Salida basica.md>) y [Rutas relativas](<04-Entrada-salida-y-ficheros/Rutas relativas.md>): trabajo con datos y ficheros.
14. [Anotaciones](<03-Anotaciones-y-reflexion/Anotaciones.md>) y [Reflections](<03-Anotaciones-y-reflexion/Reflections.md>): metaprogramación (conviene leerlas en este orden).
15. [JUnit](<08-Testing/JUnit - Vision general e introduccion.md>), [AssertJ](<08-Testing/AssertJ - Guia para empezar desde cero.md>), [Mockito](<08-Testing/Mockito - Aprende a escribir pruebas paso a paso.md>) y [TDD](<08-Testing/Test-Driven Development (TDD).md>): pruebas automatizadas.
16. [Log4J](<09-Logging/Log4J - Guia.md>): registro de información.
17. [Guía de seguridad](<11-Seguridad/Guia de seguridad.md>) y [Arquitectura criptográfica](<11-Seguridad/Arquitectura criptografica.md>): seguridad y criptografía.
18. [Conventional Commits](<12-Herramientas/Conventional Commits.md>): buenas prácticas con Git.
19. [BBDD: tipos de datos](<10-Bases-de-datos/SQL/BBDD - Tipos de datos.md>), [Entidad-Relación: diagrama](<10-Bases-de-datos/SQL/Entidad - Relacion - Diagrama.md>) y [Entidad-Relación: PlantUML](<10-Bases-de-datos/SQL/Entidad - Relacion - PlantUML.md>): diseño y modelado de bases de datos relacionales.
20. [NoSQL y diferencias con SQL](<10-Bases-de-datos/NoSQL/Diferencias entre nosql y sql.md>), [MongoDB: modelado de datos y diseño](<10-Bases-de-datos/NoSQL/MongoDB - modelado de datos y diseno.md>) y [MongoDB: tutoriales](<10-Bases-de-datos/NoSQL/MongoDB - tutoriales.md>): bases de datos NoSQL con MongoDB (conviene leerlas en este orden).

A medida que se incorporen nuevos temas, se irá organizando la documentación para facilitar un recorrido de aprendizaje más estructurado.

---

## 🤝 Contribuciones

Las contribuciones y sugerencias son bienvenidas.

Si encuentras un error, detectas información que pueda mejorarse o quieres proponer algún tema relacionado con Java, puedes contribuir al proyecto mediante un *Pull Request* o plantear una propuesta en las [issues](https://github.com/Victor-369/Documentacion-Java/issues).

### Proceso recomendado

1. Haz un *fork* del repositorio.
2. Crea una nueva rama para tus cambios.
3. Realiza las modificaciones.
4. Comprueba que la documentación sea clara y correcta.
5. Realiza un *commit* descriptivo.
6. Abre un *Pull Request*.

Para los mensajes de *commit* se recomienda utilizar [Conventional Commits](<12-Herramientas/Conventional Commits.md>).

---

## 📌 Estado del proyecto

> 🟡 **En desarrollo activo**

La documentación todavía **no está completa**.

Se están añadiendo nuevos temas y contenidos relacionados con Java de forma progresiva. El objetivo es construir con el tiempo una documentación cada vez más completa que abarque desde los fundamentos del lenguaje hasta conceptos más avanzados y tecnologías del ecosistema Java.

**¡El contenido seguirá creciendo! 🚀**

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **MIT**.

Consulta el archivo [LICENSE](https://github.com/Victor-369/Documentacion-Java/blob/main/LICENSE) para obtener más información sobre los términos de la licencia.

---

## 🔗 Repositorio

Puedes acceder al repositorio desde GitHub: [**Victor-369/Documentacion-Java**](https://github.com/Victor-369/Documentacion-Java)

---

⭐ Si esta documentación te resulta útil, puedes darle una estrella al repositorio.

☕ Y, sobre todo, ¡disfruta aprendiendo Java!
