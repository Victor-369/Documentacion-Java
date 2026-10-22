# 📚 Documentación de Java

Repositorio de documentación, apuntes, ejemplos y guías sobre **Java**, creado con el objetivo de recopilar y organizar diferentes conceptos relacionados con el desarrollo en este lenguaje.

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

### 🧩 Programación orientada a objetos y colecciones

| Documento | Qué encontrarás                                                                                                                                                                                                                                  |
| --- | --- |
| [Abstract Class vs Interface](https://github.com/Victor-369/Documentacion-Java/blob/main/Abstract%20Class%20vs%20Interface.md) | Chuleta comparativa: constructores, estado, modificadores de acceso, herencia, métodos `final`, polimorfismo y acoplamiento. Incluye cuándo usar cada una, cómo combinarlas, errores frecuentes, reglas mentales y una chuleta ultra rápida.     |
| [Colecciones](https://github.com/Victor-369/Documentacion-Java/blob/main/Colecciones.md) | Resumen del *Java Collections Framework*: jerarquía de interfaces, `List`, `Set`, `Map`, `Queue` y `Deque`, colecciones inmutables, *Sequenced Collections* (Java 21), la clase `Collections`, genéricos, Stream API y colecciones concurrentes. |
| [Comparable vs Comparator](https://github.com/Victor-369/Documentacion-Java/blob/main/Comparable%20vs%20Comparator.md) | Guía de estudio para programadores (Java 21 o superior), con ejemplos compilados y ejecutados, errores típicos, novedades de Java 21, ejercicios, chuleta de repaso y autoevaluación.                                                                                                     |
| [Generics](https://github.com/Victor-369/Documentacion-Java/blob/main/Generics.md) | Guía de genéricos: clases, interfaces y métodos genéricos, *raw types*, inferencia de tipos, tipos acotados, comodines (*wildcards*), borrado de tipos (*type erasure*), restricciones, uso con características modernas de Java, errores habituales, chuleta y ejercicios. |
| [Enums](https://github.com/Victor-369/Documentacion-Java/blob/main/Enums.md) | Enumeraciones (`enum`): declaración, uso dentro de clases y en `switch`, recorrido de constantes, diferencias con las clases y cuándo usarlas. Ampliación con enums «con cuerpo» (atributos, constructor, interfaces), `EnumMap` y `EnumSet`, y novedades desde Java 21 (`switch` con patrones, exhaustividad, `case null`, variables sin nombre `_`, constructores flexibles de Java 25). |
| [Records](https://github.com/Victor-369/Documentacion-Java/blob/main/Records.md) | Clases de datos inmutables: declaración, constructor compacto, métodos y miembros estáticos, interfaces, cuándo usarlos y reglas (JEP 395). Ampliación con copia defensiva, records locales, anotaciones, reflexión, interfaces `sealed`, patrones de record (Java 21), variables sin nombre (`_`) y constructores flexibles (Java 25). |

### λ Programación funcional: lambdas y streams

| Documento | Qué encontrarás |
| --- | --- |
| [Lambdas y Streams](https://github.com/Victor-369/Documentacion-Java/blob/main/Lambdas%20y%20Streams.md) | Guía de estudio de Java 21+: sintaxis de lambdas, interfaces funcionales, inferencia de tipos, captura de variables y referencias a métodos; *pipelines* de `Stream` (operaciones intermedias y terminales), `Collectors`, streams de primitivos, `Optional`, streams paralelos y rendimiento. Incluye novedades de Java 21 a 25 (`record`, `Stream.toList()`, *pattern matching*, hilos virtuales, `Gatherers`), errores típicos, ejercicios con soluciones y chuleta final. |

### 🧬 Anotaciones y reflexión

| Documento | Qué encontrarás |
| --- | --- |
| [Anotaciones](https://github.com/Victor-369/Documentacion-Java/blob/main/Anotaciones.md) | Guía de anotaciones en Java 25+: sintaxis, anotaciones predefinidas (`@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`…), meta-anotaciones (`@Retention`, `@Target`, `@Inherited`, `@Repeatable`), creación de anotaciones propias, lectura en tiempo de ejecución, anotaciones de tipo, uso con `record`, paquetes y módulos, procesadores de anotaciones, un validador de campos de ejemplo, errores comunes, ejercicios con soluciones y glosario. |
| [Reflections](https://github.com/Victor-369/Documentacion-Java/blob/main/Reflections.md) | Guía de la API de *Reflection* (Java 25+): obtener `Class`, inspeccionar constructores, campos y métodos, acceso a miembros privados con `setAccessible`, excepciones habituales, *records* y clases selladas, anotaciones en ejecución, genéricos y borrado de tipos, proxies dinámicos, sistema de módulos, `MethodHandle` y `VarHandle`, rendimiento, GraalVM *Native Image*, mini proyecto de validador, buenas prácticas y ejercicios. |

### 📂 Entrada/salida y ficheros

| Documento | Qué encontrarás |
| --- | --- |
| [Entrada - Salida básica](https://github.com/Victor-369/Documentacion-Java/blob/main/Entrada%20-%20Salida%20basica.md) | Guía de E/S en Java 21+: flujos de bytes y de caracteres, flujos con búfer, `Scanner` y `printf`, E/S desde la línea de comandos, *Data Streams*, serialización de objetos y ficheros con NIO.2, además de errores frecuentes, ejercicios y recursos para seguir aprendiendo. |
| [Rutas relativas](https://github.com/Victor-369/Documentacion-Java/blob/main/Rutas%20relativas.md) | Qué es una ruta relativa, notación especial (`.` y `..`), directorio de trabajo actual, `java.io.File` frente a `java.nio.file.Path`, rutas relativas al proyecto y al *classpath*, diferencias entre sistemas operativos, *Path Traversal* y errores comunes. |

### 🕒 Fechas y horas

| Documento | Qué encontrarás |
| --- | --- |
| [Fechas](https://github.com/Victor-369/Documentacion-Java/blob/main/Fechas.md) | API `java.time`: fecha y hora actuales, `DateTimeFormatter` y patrones, qué clase usar en cada caso, inmutabilidad, `Period`, `Duration` y `ChronoUnit`, zonas horarias e instantes, `Clock` para pruebas, *parsing* y modos de resolución, y novedades desde Java 21 (CLDR, eras, espacios laxos). |

### ⚠️ Excepciones y gestión de errores

| Documento | Qué encontrarás |
| --- | --- |
| [Excepciones checked y unchecked](https://github.com/Victor-369/Documentacion-Java/blob/main/Excepciones%20checked%20y%20unchecked.md) | Diferencias entre excepciones comprobadas y no comprobadas, ejemplos con `throws` y `try-catch`, excepciones personalizadas y cuándo usar cada tipo. |
| [Manejo eficaz de errores en Java: estrategias y buenas prácticas](https://github.com/Victor-369/Documentacion-Java/blob/main/Manejo%20Eficaz%20de%20Errores%20en%20Java%20-%20Estrategias%20y%20Buenas%20Practicas.md) | Resumen de un artículo sobre buenas prácticas, técnicas de manejo (`try-catch-finally`, *multi-catch*, `try-with-resources`), diseño de excepciones personalizadas y registro de errores. |

### 🧪 Testing y pruebas automatizadas

| Documento | Qué encontrarás |
| --- | --- |
| [JUnit: visión general e introducción para principiantes](https://github.com/Victor-369/Documentacion-Java/blob/main/JUnit%3A%20Vision%20general%20e%20introduccion.md) | Traducción al castellano de la página oficial de JUnit 6.1.3, más un complemento para principiantes: puesta en marcha con Maven y Gradle, anotaciones, aserciones, errores frecuentes y glosario. |
| [JUnit: aprende a escribir pruebas paso a paso](https://github.com/Victor-369/Documentacion-Java/blob/main/JUnit%3A%20Aprende%20a%20escribir%20pruebas%20paso%20a%20paso.md) | Guía práctica: primera prueba, ciclo de vida, aserciones, excepciones, pruebas parametrizadas, anidadas, repetidas y dinámicas, etiquetas, *timeouts* y buenas prácticas. |
| [AssertJ: guía para empezar desde cero](https://github.com/Victor-369/Documentacion-Java/blob/main/AssertJ%20-%20Guia%20para%20empezar%20desde%20cero.md) | Aserciones fluidas con `assertThat(...)`: tipos de dato, colecciones, excepciones, mensajes de error, *soft assertions*, comparación recursiva y estilo BDD. |
| [Test-Driven Development (TDD)](https://github.com/Victor-369/Documentacion-Java/blob/main/Test-Driven%20Development%20%28TDD%29.md) | Guía paso a paso del ciclo Rojo → Verde → Refactor, con dos ejemplos completos (factorial y validador de contraseñas), buenas prácticas, errores típicos y ejercicios. |

### 🔐 Seguridad y criptografía

| Documento | Qué encontrarás |
| --- | --- |
| [Guía de seguridad](https://github.com/Victor-369/Documentacion-Java/blob/main/Guia%20de%20seguridad.md) | Introducción a la *Security Developer's Guide* de Java SE (Java 21+): mapa de la seguridad en Java (JCA/JCE, PKI, JSSE, JAAS, SASL, firma de código, serialización segura), cambios desde Java 17, herramientas de línea de comandos (`keytool`, `jarsigner`), buenas prácticas y un ejemplo mínimo de hash SHA-256. |
| [Arquitectura criptográfica](https://github.com/Victor-369/Documentacion-Java/blob/main/Arquitectura%20criptografica.md) | Guía de referencia de la JCA (Java 21+): principios de diseño, proveedores, clase `Security`, clases *engine*, `SecureRandom`, `MessageDigest`, `Signature`, `Cipher`, `Mac`, claves, `KeyStore`, certificados, excepciones habituales, buenas prácticas y glosario. |

### 🗄️ Bases de datos

| Documento | Qué encontrarás |
| --- | --- |
| [BBDD: tipos de datos](https://github.com/Victor-369/Documentacion-Java/blob/main/BBDD%20-%20Tipos%20de%20datos.md) | Tipos de datos de MySQL (estado en 2026: 9.7 LTS, 8.4 LTS y fin de vida de 8.0): numéricos, cadena (`CHAR`, `VARCHAR`, `BLOB`, `TEXT`, `ENUM`, `SET`, `VECTOR`), fecha y hora, atributos de columna, guía rápida de elección, correcciones respecto al artículo original y fuentes. |
| [Entidad-Relación: diagrama](https://github.com/Victor-369/Documentacion-Java/blob/main/Entidad%20-%20Relacion%3A%20Diagrama.md) | Diagrama Entidad-Relación: entidades, relaciones y atributos, cardinalidad, notaciones de Chen y *Crow's Foot*, creación paso a paso, buenas prácticas, comparación con UML, preguntas frecuentes y paso del diagrama a tablas MySQL. |
| [Entidad-Relación: PlantUML](https://github.com/Victor-369/Documentacion-Java/blob/main/Entidad%20-%20Relacion%3A%20PlantUML.md) | Guía para dibujar diagramas E/R como código con PlantUML: instalación y herramientas, entidades, clases y enums, cardinalidad en notación pata de gallo, relaciones de diagrama de clases, casos típicos (1:1, 1:N, N:M, reflexiva, jerarquía, ternaria), notación de Chen, paquetes y notas, estilos y temas, macros del preprocesador, ejemplo completo de tienda online con su SQL, buenas prácticas, errores frecuentes y chuleta. |
| [NoSQL y diferencias con SQL](https://github.com/Victor-369/Documentacion-Java/blob/main/Diferencias%20entre%20nosql%20y%20sql.md) | Qué es NoSQL y sus tipos (documentales, clave-valor, columnas anchas, grafos y multimodelo), breve historia, características, ACID frente a BASE y teorema CAP, ejemplo relacional frente a documental, diferencias entre RDBMS y NoSQL, equivalencias de conceptos y consultas SQL ↔ MongoDB (MQL), cuándo elegir cada opción, mitos habituales y preguntas frecuentes (revisado para MongoDB 9.0). |
| [MongoDB: modelado de datos y diseño](https://github.com/Victor-369/Documentacion-Java/blob/main/MongoDB%3A%20modelado%20de%20datos%20y%20diseno.md) | Modelado de datos en MongoDB: diferencias con el diseño relacional, proceso de diseño del esquema en 4 pasos, relaciones (1:1, 1:pocos, 1:N, N:M y `$lookup`), embeber o referenciar, duplicación y coherencia, validación de esquemas, índices, atomicidad y *sharding*, patrones de diseño (*Computed*, *Bucket*, *Outlier*, *Subset*, *Extended Reference*, versionado…), antipatrones, estructuras de árbol, ejemplo de tienda en línea y lista de comprobación. |
| [MongoDB: tutoriales](https://github.com/Victor-369/Documentacion-Java/blob/main/MongoDB%3A%20tutoriales.md) | Índice organizado de los tutoriales oficiales del manual (MongoDB 9.0) con guía rápida en `mongosh`: CRUD, índices, agregación y transacciones; conjuntos de réplicas, *sharding*, administración, seguridad, patrones de desarrollo y de búsqueda de texto, novedades de la 9.0 y ruta de aprendizaje recomendada. |

### 📝 Logging y registro de información

| Documento | Qué encontrarás |
| --- | --- |
| [Log4J: gestión de registros](https://github.com/Victor-369/Documentacion-Java/blob/main/Log4J%3A%20Gestion%20de%20Registros.md) | Resumen conceptual: niveles de *logging* y cuándo usar cada uno, configuración, *appenders* y *layouts*. |
| [Log4J: guía](https://github.com/Victor-369/Documentacion-Java/blob/main/Log4J%3A%20Guia.md) | Guía práctica de Log4j 2: instalación con Maven y Gradle, `log4j2.xml`, mensajes parametrizados, escritura en ficheros (`RollingFile`), `ThreadContext`, integración con SLF4J y problemas frecuentes. |

### 🔧 Herramientas y buenas prácticas

| Documento | Qué encontrarás |
| --- | --- |
| [Conventional Commits](https://github.com/Victor-369/Documentacion-Java/blob/main/Conventional%20Commits.md) | Convención para escribir mensajes de *commit* claros: formato, tipos habituales (`feat`, `fix`, `docs`…), ejemplos, buenas prácticas y flujo básico. |

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
- [ ] Fundamentos de Java
- [ ] Variables y tipos de datos
- [ ] Operadores
- [ ] Estructuras de control
- [ ] Arrays
- [ ] Métodos
- [ ] Herencia, encapsulación y polimorfismo
- [x] Enumeraciones (`enum`)
- [x] Records
- [x] Genéricos
- [x] Streams y expresiones lambda
- [x] `Optional`
- [ ] Programación funcional
- [x] Entrada y salida de datos y ficheros
- [x] Fechas y horas
- [ ] Concurrencia, hilos y sincronización
- [ ] Mockito
- [ ] Maven y Gradle
- [x] Bases de datos: tipos de datos MySQL y diagramas Entidad-Relación (incluido PlantUML)
- [x] Bases de datos NoSQL: diferencias con SQL y MongoDB (modelado de datos y tutoriales)
- [ ] JDBC y acceso a bases de datos desde Java
- [ ] Patrones de diseño
- [ ] Principios SOLID
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

La intención es que el repositorio pueda servir tanto para **aprender Java desde cero** como para **consultar conceptos concretos durante el desarrollo de proyectos**.

---

## 🗂️ Estructura actual

La estructura del repositorio se irá modificando a medida que se incorporen nuevos contenidos.

```
Documentacion-Java/
├── .gitignore
├── Abstract Class vs Interface.md
├── Anotaciones.md
├── Arquitectura criptografica.md
├── AssertJ - Guia para empezar desde cero.md
├── BBDD - Tipos de datos.md
├── Colecciones.md
├── Comparable vs Comparator.md
├── Conventional Commits.md
├── Diferencias entre nosql y sql.md
├── Entidad - Relacion: Diagrama.md
├── Entidad - Relacion: PlantUML.md
├── Entrada - Salida basica.md
├── Enums.md
├── Excepciones checked y unchecked.md
├── Fechas.md
├── Generics.md
├── Guia de seguridad.md
├── JUnit: Aprende a escribir pruebas paso a paso.md
├── JUnit: Vision general e introduccion.md
├── LICENSE
├── Lambdas y Streams.md
├── Log4J: Gestion de Registros.md
├── Log4J: Guia.md
├── Manejo Eficaz de Errores en Java - Estrategias y Buenas Practicas.md
├── MongoDB: modelado de datos y diseno.md
├── MongoDB: tutoriales.md
├── README.md
├── Records.md
├── Reflections.md
├── Rutas relativas.md
└── Test-Driven Development (TDD).md
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

La documentación del repositorio incluye material más detallado sobre [JUnit](https://github.com/Victor-369/Documentacion-Java/blob/main/JUnit%3A%20Aprende%20a%20escribir%20pruebas%20paso%20a%20paso.md), [AssertJ](https://github.com/Victor-369/Documentacion-Java/blob/main/AssertJ%20-%20Guia%20para%20empezar%20desde%20cero.md) y [TDD](https://github.com/Victor-369/Documentacion-Java/blob/main/Test-Driven%20Development%20%28TDD%29.md).

---

## 🛠️ Tecnologías y herramientas

Actualmente, el repositorio contiene documentación relacionada con:

- ☕ Java (colecciones, genéricos, lambdas y streams, excepciones, clases abstractas e interfaces, ordenación, enums, records, fechas y horas, entrada/salida y rutas de ficheros)
- 🧬 Anotaciones y *Reflection* (metaprogramación en Java)
- 🗄️ Bases de datos relacionales (tipos de datos de MySQL y diagramas Entidad-Relación)
- 🍃 Bases de datos NoSQL y MongoDB (diferencias con SQL, modelado de datos, `mongosh`, CRUD, índices y agregación)
- 🌱 PlantUML (diagramas Entidad-Relación como código)
- 🔐 Seguridad y criptografía en Java (JCA/JCE)
- 🧪 JUnit
- 🔍 AssertJ
- 🔄 Test-Driven Development (TDD)
- 📝 Log4j 2
- 🌿 Git (convención de *commits* con Conventional Commits)
- 📦 Maven y 🏗️ Gradle (configuración de dependencias dentro de las guías de JUnit, AssertJ, TDD y Log4j)

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

Si estás comenzando a aprender Java, puedes utilizar esta documentación como material de consulta e ir avanzando progresivamente.

No es necesario seguir un orden concreto, ya que cada documento está pensado para poder consultarse de manera independiente. Aun así, si no sabes por dónde empezar, esta ruta puede servirte de orientación:

1. [Abstract Class vs Interface](https://github.com/Victor-369/Documentacion-Java/blob/main/Abstract%20Class%20vs%20Interface.md) y [Colecciones](https://github.com/Victor-369/Documentacion-Java/blob/main/Colecciones.md): base del lenguaje.
2. [Enums](https://github.com/Victor-369/Documentacion-Java/blob/main/Enums.md) y [Records](https://github.com/Victor-369/Documentacion-Java/blob/main/Records.md): tipos de datos propios.
3. [Generics](https://github.com/Victor-369/Documentacion-Java/blob/main/Generics.md) y [Comparable vs Comparator](https://github.com/Victor-369/Documentacion-Java/blob/main/Comparable%20vs%20Comparator.md): tipos genéricos y ordenación de objetos.
4. [Lambdas y Streams](https://github.com/Victor-369/Documentacion-Java/blob/main/Lambdas%20y%20Streams.md): programación funcional y procesamiento de datos.
5. [Excepciones checked y unchecked](https://github.com/Victor-369/Documentacion-Java/blob/main/Excepciones%20checked%20y%20unchecked.md) y [Manejo eficaz de errores](https://github.com/Victor-369/Documentacion-Java/blob/main/Manejo%20Eficaz%20de%20Errores%20en%20Java%20-%20Estrategias%20y%20Buenas%20Practicas.md): gestión de errores.
6. [Fechas](https://github.com/Victor-369/Documentacion-Java/blob/main/Fechas.md): fechas y horas con `java.time`.
7. [Entrada - Salida básica](https://github.com/Victor-369/Documentacion-Java/blob/main/Entrada%20-%20Salida%20basica.md) y [Rutas relativas](https://github.com/Victor-369/Documentacion-Java/blob/main/Rutas%20relativas.md): trabajo con datos y ficheros.
8. [Anotaciones](https://github.com/Victor-369/Documentacion-Java/blob/main/Anotaciones.md) y [Reflections](https://github.com/Victor-369/Documentacion-Java/blob/main/Reflections.md): metaprogramación (conviene leerlas en este orden).
9. [JUnit](https://github.com/Victor-369/Documentacion-Java/blob/main/JUnit%3A%20Vision%20general%20e%20introduccion.md), [AssertJ](https://github.com/Victor-369/Documentacion-Java/blob/main/AssertJ%20-%20Guia%20para%20empezar%20desde%20cero.md) y [TDD](https://github.com/Victor-369/Documentacion-Java/blob/main/Test-Driven%20Development%20%28TDD%29.md): pruebas automatizadas.
10. [Log4J](https://github.com/Victor-369/Documentacion-Java/blob/main/Log4J%3A%20Guia.md): registro de información.
11. [Guía de seguridad](https://github.com/Victor-369/Documentacion-Java/blob/main/Guia%20de%20seguridad.md) y [Arquitectura criptográfica](https://github.com/Victor-369/Documentacion-Java/blob/main/Arquitectura%20criptografica.md): seguridad y criptografía.
12. [Conventional Commits](https://github.com/Victor-369/Documentacion-Java/blob/main/Conventional%20Commits.md): buenas prácticas con Git.
13. [BBDD: tipos de datos](https://github.com/Victor-369/Documentacion-Java/blob/main/BBDD%20-%20Tipos%20de%20datos.md), [Entidad-Relación: diagrama](https://github.com/Victor-369/Documentacion-Java/blob/main/Entidad%20-%20Relacion%3A%20Diagrama.md) y [Entidad-Relación: PlantUML](https://github.com/Victor-369/Documentacion-Java/blob/main/Entidad%20-%20Relacion%3A%20PlantUML.md): diseño y modelado de bases de datos relacionales.
14. [NoSQL y diferencias con SQL](https://github.com/Victor-369/Documentacion-Java/blob/main/Diferencias%20entre%20nosql%20y%20sql.md), [MongoDB: modelado de datos y diseño](https://github.com/Victor-369/Documentacion-Java/blob/main/MongoDB%3A%20modelado%20de%20datos%20y%20diseno.md) y [MongoDB: tutoriales](https://github.com/Victor-369/Documentacion-Java/blob/main/MongoDB%3A%20tutoriales.md): bases de datos NoSQL con MongoDB (conviene leerlas en este orden).

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

Para los mensajes de *commit* se recomienda utilizar [Conventional Commits](https://github.com/Victor-369/Documentacion-Java/blob/main/Conventional%20Commits.md).

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
