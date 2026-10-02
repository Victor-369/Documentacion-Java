# 📚 Documentación de Java

Repositorio de documentación, apuntes, ejemplos y guías sobre **Java**, creado con el objetivo de recopilar y organizar diferentes conceptos relacionados con el desarrollo en este lenguaje.

> 🚧 **Proyecto en desarrollo**
> 
> Esta documentación se encuentra actualmente **en pleno proceso de desarrollo y ampliación**. Se irán incorporando progresivamente nuevos temas, conceptos, ejemplos y buenas prácticas relacionados con Java.
> 
> El contenido actual es solo una parte de la documentación que se pretende reunir en este repositorio.

---

## 📖 Contenido

Actualmente, el repositorio cuenta con documentación sobre diferentes conceptos de Java y herramientas relacionadas con su desarrollo.

### 🧩 Programación orientada a objetos

- Clases abstractas vs. interfaces
- Interfaces vs. clases abstractas
- Colecciones
- Comparable vs. Comparator

En esta sección se recopilan conceptos fundamentales de la programación orientada a objetos en Java, así como diferentes mecanismos proporcionados por el lenguaje para trabajar con clases, interfaces y colecciones.

### ⚠️ Excepciones y gestión de errores

- Excepciones checked y unchecked
    
- Manejo eficaz de errores en Java
    

Esta sección trata sobre la gestión de excepciones y errores en aplicaciones Java, incluyendo las diferencias entre excepciones comprobadas y no comprobadas y algunas estrategias para gestionar los errores correctamente.

### 🧪 Testing y pruebas automatizadas

- JUnit - Visión general e introducción para principiantes
    
- JUnit - Aprende a escribir pruebas paso a paso
    
- AssertJ - Guía para empezar desde cero
    
- Test-Driven Development (TDD)
    

La documentación de testing recoge conceptos relacionados con las pruebas automatizadas en Java, incluyendo JUnit, AssertJ y la metodología de desarrollo dirigido por pruebas (TDD).

### 📝 Logging y registro de información

- Log4J - Gestión de registros
    
- Log4J - Guía
    

Documentación relacionada con el registro de información de las aplicaciones Java y el uso de herramientas de logging.

### 🔧 Herramientas y buenas prácticas

- Conventional Commits
    

Los **Conventional Commits** permiten establecer una estructura común para los mensajes de commit, facilitando la lectura del historial del proyecto y la automatización de determinados procesos.

---

## 🚧 Documentación en desarrollo

Este repositorio está pensado para **crecer progresivamente**.

Actualmente solo se han añadido algunos temas de Java, pero el objetivo es continuar ampliando la documentación con nuevos contenidos, ejemplos y explicaciones.

Algunos de los temas que se irán incorporando pueden estar relacionados con:

- Fundamentos de Java
    
- Variables y tipos de datos
    
- Operadores
    
- Estructuras de control
    
- Arrays
    
- Métodos
    
- Programación orientada a objetos
    
- Herencia
    
- Encapsulación
    
- Polimorfismo
    
- Clases abstractas
    
- Interfaces
    
- Enumeraciones (`enum`)
    
- Records
    
- Genéricos
    
- Colecciones
    
- Excepciones
    
- Streams
    
- Expresiones lambda
    
- `Optional`
    
- Programación funcional
    
- Entrada y salida de datos
    
- Ficheros
    
- Fechas y horas
    
- Concurrencia y _multithreading_
    
- Hilos
    
- Sincronización
    
- Testing
    
- JUnit
    
- Mockito
    
- AssertJ
    
- TDD
    
- Logging
    
- Maven
    
- Gradle
    
- JDBC
    
- Bases de datos
    
- Buenas prácticas
    
- Patrones de diseño
    
- Principios SOLID
    
- Arquitectura de aplicaciones
    
- Spring
    
- Spring Boot
    
- Y otros conceptos relacionados con el ecosistema Java
    

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

Actualmente incluye documentos relacionados con:

```text
Documentacion-Java/
├── Abstract Class vs Interface.md
├── AssertJ - Guia para empezar desde cero.md
├── Colecciones.md
├── Comparable vs Comparator.md
├── Conventional Commits.md
├── Excepciones checked y unchecked.md
├── Interface vs Abstract Class.md
├── JUnit - Aprende a escribir pruebas paso a paso.md
├── JUnit - Vision general e introduccion para principiantes.md
├── LICENSE
├── Log4J - Gestion de Registros.md
├── Log4J - Guia.md
├── Manejo Eficaz de Errores en Java: Estrategias y Buenas Practicas.md
├── README.md
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

La documentación del repositorio incluye material más detallado sobre JUnit, testing y otras herramientas relacionadas con el desarrollo en Java.

---

## 🛠️ Tecnologías y herramientas

Actualmente, el repositorio contiene documentación relacionada con:

- ☕ Java
    
- 🧪 JUnit
    
- 🔍 AssertJ
    
- 📝 Log4J
    
- 🔄 Test-Driven Development (TDD)
    
- 🌿 Git
    
- 📦 Maven
    
- 🏗️ Gradle
    
- 📝 Conventional Commits
    

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

No es necesario seguir un orden concreto, ya que cada documento está pensado para poder consultarse de manera independiente.

A medida que se incorporen nuevos temas, se irá organizando la documentación para facilitar un recorrido de aprendizaje más estructurado.

---

## 🤝 Contribuciones

Las contribuciones y sugerencias son bienvenidas.

Si encuentras un error, detectas información que pueda mejorarse o quieres proponer algún tema relacionado con Java, puedes contribuir al proyecto mediante un _Pull Request_ o plantear una propuesta.

### Proceso recomendado

1. Haz un _fork_ del repositorio.
    
2. Crea una nueva rama para tus cambios.
    
3. Realiza las modificaciones.
    
4. Comprueba que la documentación sea clara y correcta.
    
5. Realiza un _commit_ descriptivo.
    
6. Abre un _Pull Request_.
    

Para los mensajes de _commit_ se recomienda utilizar **Conventional Commits**.

---

## 📌 Estado del proyecto

> 🟡 **En desarrollo activo**

La documentación todavía **no está completa**.

Se están añadiendo nuevos temas y contenidos relacionados con Java de forma progresiva. El objetivo es construir con el tiempo una documentación cada vez más completa que abarque desde los fundamentos del lenguaje hasta conceptos más avanzados y tecnologías del ecosistema Java.

**¡El contenido seguirá creciendo! 🚀**

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **MIT**.

Consulta el archivo LICENSE para obtener más información sobre los términos de la licencia.

---

## 🔗 Repositorio

Puedes acceder al repositorio desde GitHub:

**Victor-369/Documentacion-Java**

---

⭐ Si esta documentación te resulta útil, puedes darle una estrella al repositorio.

☕ Y, sobre todo, ¡disfruta aprendiendo Java!