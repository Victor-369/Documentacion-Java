# Diagrama Entidad-Relación (ER) en bases de datos

> Resumen basado en el artículo *Entity Relationship (ER) Diagram Model with DBMS Example* de Guru99 (actualizado el 7 de octubre de 2026), con ampliaciones propias señaladas como tales.

---

## 1. Qué es un diagrama ER

Un **diagrama Entidad-Relación (ER)** es una herramienta visual para diseñar la estructura de una base de datos relacional. Define de forma precisa:

- **Entidades**: los objetos sobre los que se guardan datos.
- **Atributos**: las propiedades de esas entidades.
- **Relaciones**: cómo se asocian entre sí.
- **Cardinalidades**: cuántas instancias de una entidad se relacionan con cuántas de otra.

A simple vista se parece a un diagrama de flujo, pero usa símbolos especializados con significado propio. Su finalidad es representar la infraestructura de entidades del sistema.

### Historia

- Lo propuso **Peter Chen en 1976** en el artículo *"The Entity-Relationship Model: Toward a Unified View of Data"*.
- Su objetivo era una convención uniforme válida tanto para bases de datos relacionales como para redes, a modo de modelo conceptual que une los requisitos del mundo real con la implementación técnica.
- Desde entonces han aparecido varias notaciones: **Chen** (la original), **Crow's Foot / patas de gallo** (muy usada en herramientas modernas) y enfoques basados en **UML**. Los conceptos básicos son los mismos en todas.

### Para qué sirve

| Beneficio | Descripción |
| --- | --- |
| Comunicación visual | Lo entienden perfiles técnicos y no técnicos |
| Plano de desarrollo | Muestra cómo se conectan las tablas y qué campos tendrá cada una |
| Fácil de traducir | Se convierte directamente en tablas relacionales |
| Prevención de errores | Detecta fallos de diseño y redundancias antes de implementar |
| Documentación | Ayuda a que el equipo nuevo entienda la arquitectura |
| Análisis del sistema | Identifica todas las entidades y relaciones existentes |

---

## 2. Componentes de un diagrama ER

Todo diagrama se construye con tres componentes: **entidades, atributos y relaciones**.

Ejemplo (universidad): entidades *Estudiante*, *Curso* y *Profesor*. El estudiante tiene atributos como `Rollno`, `Name` y `DeptID`, y se relaciona con cursos y profesores.

### 2.1 Entidades

Una **entidad** es cualquier objeto del mundo real, vivo o no, que se puede identificar claramente y del que se pueden almacenar datos (persona, lugar, objeto, evento o concepto).

| Categoría | Ejemplos |
| --- | --- |
| Persona | Empleado, Estudiante, Paciente, Cliente |
| Lugar | Tienda, Edificio, Oficina, Almacén |
| Objeto | Máquina, Producto, Coche, Libro |
| Evento | Venta, Matrícula, Renovación, Transacción |
| Concepto | Cuenta, Curso, Departamento, Proyecto |

Un **conjunto de entidades** (*entity set*) agrupa entidades similares que comparten atributos (por ejemplo, todos los estudiantes forman el conjunto "Estudiante"). Se dibujan como **rectángulos** con el nombre dentro.

#### Entidades fuertes y débiles

- **Entidad fuerte**: tiene clave primaria propia y existe de forma independiente (p. ej. `Student` identificado por `Student_ID`).
- **Entidad débil**: no tiene clave primaria propia. Depende de una entidad fuerte (la *entidad propietaria*) y se identifica combinando una **clave parcial (discriminador)** con la clave primaria del propietario. Ejemplo: en un banco, `Transaction` depende de `Account`; el número de transacción por sí solo no es único en toda la base de datos, pero combinado con el número de cuenta sí lo es.

| Entidad fuerte | Entidad débil |
| --- | --- |
| Tiene clave primaria propia | No tiene clave primaria; usa clave parcial |
| Rectángulo simple | Rectángulo doble |
| Clave primaria subrayada con línea continua | Clave parcial subrayada con línea discontinua |
| Puede existir de forma independiente | Depende de la entidad propietaria |
| Se conecta con rombo simple | Se conecta con rombo doble (relación identificativa) |
| Ej.: Student, Employee, Product | Ej.: Transaction, Dependent, Order_Item |

### 2.2 Relaciones

Una **relación** es una asociación entre dos o más entidades. Suele identificarse con **verbos** ("el estudiante *asiste* a una clase", "Tom *trabaja en* el departamento de Química"). Se dibuja con un **rombo**.

### 2.3 Atributos

Un **atributo** es una propiedad que describe a una entidad o relación. Se dibuja con un **óvalo** unido a su entidad. Ejemplo para `Student`: `Student_ID`, `Name`, `Date_of_Birth`, `Email`, `Phone_Number`.

| Tipo | Descripción | Ejemplos |
| --- | --- | --- |
| Simple (atómico) | No se puede dividir más | Teléfono, SSN, email |
| Compuesto | Se descompone en subatributos | Nombre completo (nombre, segundo nombre, apellido); dirección (calle, ciudad, código postal) |
| Derivado | Se calcula a partir de otros; no se almacena directamente | Edad (a partir de la fecha de nacimiento), precio total |
| Multivalor | Puede tener varios valores para una misma entidad | Teléfonos, emails, habilidades |
| Clave | Identifica de forma única cada instancia (clave primaria) | Student_ID, Employee_ID, ISBN |

**Representación:** los atributos clave llevan el nombre **subrayado**; los derivados van en **óvalo discontinuo**; los multivalor, en **óvalo doble**.

---

## 3. Cardinalidad (tipos de relación)

La **cardinalidad** indica cuántas instancias de una entidad pueden asociarse con instancias de otra.

| Tipo | Significado | Ejemplo |
| --- | --- | --- |
| **Uno a uno (1:1)** | Un elemento de A se asocia como máximo con uno de B, y viceversa | Un estudiante tiene un ID de estudiante y cada ID pertenece a un solo estudiante |
| **Uno a muchos (1:N)** | Un elemento de A se asocia con varios de B, pero cada uno de B solo con uno de A | Una clase tiene varios estudiantes |
| **Muchos a uno (N:1)** | Varios elementos de A se asocian con uno de B | Muchos estudiantes pertenecen a la misma clase |
| **Muchos a muchos (M:N)** | Varios de A con varios de B y viceversa | Los estudiantes se asocian con varios profesores y los profesores con varios estudiantes |

---

## 4. Símbolos y notaciones

### 4.1 Notación de Chen

Usa formas geométricas (la notación original de 1976).

| Símbolo | Nombre | Representa |
| --- | --- | --- |
| Rectángulo | Entidad | Entidad fuerte (Student, Product) |
| Rectángulo doble | Entidad débil | Entidad dependiente (Transaction) |
| Elipse / óvalo | Atributo | Propiedad de una entidad (Name, ID) |
| Elipse doble | Atributo multivalor | Varios valores (teléfonos) |
| Elipse discontinua | Atributo derivado | Valor calculado (edad desde la fecha de nacimiento) |
| Rombo | Relación | Asociación entre entidades (Enrolls) |
| Rombo doble | Relación identificativa | Relación con una entidad débil |
| Línea | Enlace | Conecta los componentes |
| Texto subrayado | Clave primaria | Identificador único de la entidad |

### 4.2 Notación Crow's Foot (patas de gallo)

También llamada notación IE. Es la más común en herramientas modernas y destaca por mostrar bien el lado "muchos".

| Símbolo                          | Significado                       |
| -------------------------------- | --------------------------------- |
| Línea vertical simple (`\|`)     | Uno obligatorio (exactamente uno) |
| Círculo con línea (`O\|`)        | Uno opcional (cero o uno)         |
| Pata de gallo con línea (`>\|`)  | Muchos obligatorio (uno o más)    |
| Pata de gallo con círculo (`>O`) | Muchos opcional (cero o más)      |

### 4.3 Chen frente a Crow's Foot

| Aspecto | Chen | Crow's Foot |
| --- | --- | --- |
| Mejor para | Modelado conceptual y uso académico | Modelado lógico/físico y uso en la industria |
| Atributos | Los muestra visualmente todos | Los lista dentro del recuadro de la entidad |
| Cardinalidad | Con números (1, N, M) | Con símbolos visuales |
| Complejidad | Puede saturarse | Más compacto y limpio |
| Soporte en herramientas | Limitado en herramientas modernas | Muy extendido |

---

## 5. Cómo crear un diagrama ER (paso a paso)

**Enunciado de ejemplo (universidad):** un estudiante se matricula en cursos y debe tener al menos uno. Cada curso lo imparte un único profesor. Para mantener la calidad docente, un profesor solo puede impartir un curso.

1. **Identificar entidades**: `Student`, `Course`, `Professor`.
2. **Identificar relaciones**: el estudiante *tiene asignado* un curso; el profesor *imparte* un curso.
3. **Identificar cardinalidades**: un estudiante puede tener varios cursos; un profesor imparte solo uno.
4. **Identificar atributos** y asignar cada uno a una sola entidad (si parece pertenecer a varias, usar un modificador para hacerlo único). Después, determinar las claves primarias y, si no existe una clave natural, crearla. Para encontrar atributos se estudian los ficheros, formularios e informes de la organización y se entrevista a los interesados.

   | Entidad | Clave primaria | Atributo |
   | --- | --- | --- |
   | Student | Student_ID | StudentName |
   | Professor | Employee_ID | ProfessorName |
   | Course | Course_ID | CourseName |

   (En `Course` podrían añadirse duración, créditos, tareas, etc.; el ejemplo usa uno solo por simplicidad.)
5. **Dibujar el diagrama** con la notación elegida.

---

## 6. Buenas prácticas

- **Eliminar redundancias**: entidades, atributos o relaciones duplicados.
- **Nombres claros y consistentes**, sin abreviaturas.
- **Validar contra los requisitos**: el diagrama debe cubrir todo lo que hay que almacenar.
- **Mantenerlo simple**: mejor varios diagramas a distintos niveles que uno saturado.
- **Usar el color con moderación** y de forma consistente.
- **Documentar las suposiciones** sobre reglas de negocio.
- **Revisarlo con usuarios de negocio y equipo técnico.**
- **Control de versiones** a medida que evoluciona el diseño.

---

## 7. Diagrama ER frente a diagrama de clases UML

| Aspecto | Diagrama ER | Diagrama de clases UML |
| --- | --- | --- |
| Finalidad principal | Diseño de bases de datos | Diseño de software/objetos |
| Enfoque | Datos y relaciones | Objetos, métodos y comportamientos |
| Métodos/operaciones | No soportados | Totalmente soportados |
| Herencia | Limitada (solo en EER) | Soporte completo |
| Uso habitual | Administradores de BD y analistas de datos | Desarrolladores y arquitectos |

---

## 8. Preguntas frecuentes (resumen del artículo)

- **Dos tipos principales de ER:** *conceptual* (entidades y relaciones de alto nivel, sin detalles de implementación) y *físico* (estructuras reales, tipos de datos, claves y restricciones).
- **¿Qué notación usar?** Chen para diseño conceptual y entornos académicos; Crow's Foot para diseño lógico/físico en la industria (es la que soportan la mayoría de herramientas modernas).
- **Relaciones M:N en una base de datos real:** no se pueden implementar directamente; hay que crear una **entidad asociativa (tabla intermedia)** que las descomponga en dos relaciones 1:N, con claves foráneas hacia las dos entidades originales.
- **IA generativa y diseño de BD:** puede proponer modelos ER iniciales a partir de requisitos o de datos existentes, pero la supervisión humana sigue siendo imprescindible para validar relaciones, restricciones, coherencia lógica y ajuste a las reglas de negocio reales.
- **ER en BD en la nube:** el artículo afirma que los diagramas ER guían el aprovisionamiento, el escalado y la orquestación de consultas automatizados, y permiten a herramientas basadas en IA alinear las estructuras de datos con la carga prevista. Es una afirmación general del artículo, sin detalle técnico.

---

## 9. Ampliación: del diagrama ER a tablas MySQL

> Esta sección es una **adición propia** que no figura en el artículo de Guru99, salvo donde se indica. Aplica reglas estándar de paso de modelo conceptual a relacional.

| Elemento del ER | Se convierte en |
| --- | --- |
| Entidad | Una tabla |
| Atributo simple | Una columna |
| Atributo clave | Clave primaria (`PRIMARY KEY`) |
| Atributo derivado | Normalmente no se almacena; se calcula al consultar (el artículo indica que no se guarda directamente) |
| Atributo multivalor | Normalmente una tabla aparte relacionada con la entidad |
| Relación 1:N | Clave foránea en la tabla del lado "N" |
| Relación 1:1 | Clave foránea con restricción `UNIQUE` |
| Relación M:N | Tabla intermedia con dos claves foráneas (según el artículo) |
| Entidad débil | Clave primaria compuesta: clave del propietario + clave parcial (según la definición del artículo) |

### Ejemplo con el caso de la universidad

Un profesor imparte un solo curso (1:1 entre `professor` y `course`) y un estudiante puede estar en varios cursos (M:N, resuelta con `enrollment`):

```sql
CREATE TABLE student (
  student_id   INT UNSIGNED NOT NULL AUTO_INCREMENT,
  student_name VARCHAR(100) NOT NULL,
  PRIMARY KEY (student_id)
);

CREATE TABLE professor (
  employee_id    INT UNSIGNED NOT NULL AUTO_INCREMENT,
  professor_name VARCHAR(100) NOT NULL,
  PRIMARY KEY (employee_id)
);

CREATE TABLE course (
  course_id   INT UNSIGNED NOT NULL AUTO_INCREMENT,
  course_name VARCHAR(100) NOT NULL,
  employee_id INT UNSIGNED NOT NULL,
  PRIMARY KEY (course_id),
  UNIQUE KEY uq_course_professor (employee_id),
  FOREIGN KEY (employee_id) REFERENCES professor (employee_id)
);

CREATE TABLE enrollment (
  student_id INT UNSIGNED NOT NULL,
  course_id  INT UNSIGNED NOT NULL,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES student (student_id),
  FOREIGN KEY (course_id)  REFERENCES course (course_id)
);
```

Notas sobre el ejemplo:

- La restricción `UNIQUE` sobre `course.employee_id` es lo que impone que un profesor imparta como máximo un curso.
- La regla "un estudiante debe tener al menos un curso" (participación obligatoria) **no** queda garantizada solo con este esquema; habría que controlarla en la aplicación o con otros mecanismos.
- Los tipos de datos elegidos (`INT UNSIGNED`, `VARCHAR`) se explican en el resumen de tipos de datos MySQL.

---

## Fuentes

- Guru99: [Entity Relationship (ER) Diagram Model with DBMS Example](https://www.guru99.com/er-diagram-tutorial-dbms.html) (actualizado el 7 de octubre de 2026).
