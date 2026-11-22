# Glosario

| Término | Definición |
|---|---|
| **Adaptador** | En arquitectura hexagonal, implementación concreta de un puerto con una tecnología (JPA, REST, email…). |
| **Agregación (MongoDB)** | Procesamiento de documentos en una cadena de etapas (filtrar, agrupar, ordenar…). |
| **Ámbito (*scope*)** | Define cuántas instancias de un *bean* crea Spring y cuánto viven (`singleton`, `prototype`…). |
| **API** | Contrato que define cómo un programa puede pedir datos u operaciones a otro. |
| **API Gateway** | Punto de entrada único que recibe las peticiones de los clientes y las reparte entre los microservicios. |
| **API REST** | API web que sigue los principios del estilo de arquitectura REST. |
| **`ApplicationContext`** | Interfaz que representa el contenedor IoC de Spring. |
| **Autoconfiguración** | Mecanismo de Spring Boot que configura automáticamente la aplicación según las librerías presentes. |
| **Bean** | Objeto creado y gestionado por el contenedor de Spring. |
| **Bean Validation** | Especificación estándar (Jakarta) para validar objetos con anotaciones como `@NotBlank` o `@Email`. |
| **BSON** | Formato binario, similar a JSON, con el que MongoDB guarda los documentos. |
| **Cabecera (*header*)** | Par `Nombre: valor` que acompaña a una petición o respuesta HTTP con información adicional. |
| **Capa (Docker)** | Resultado de cada instrucción de un Dockerfile. Las capas se guardan en caché y se reutilizan. |
| **Cascada (*cascade*)** | Propagación de una operación (guardar, borrar…) de una entidad a sus entidades relacionadas. |
| **Circuit breaker** | Patrón que deja de llamar temporalmente a un servicio que falla repetidamente. |
| **Código de estado** | Número de tres cifras en una respuesta HTTP que indica el resultado (`200`, `404`, `500`…). |
| **Colección** | En MongoDB, conjunto de documentos (equivale a una tabla). |
| **Contenedor** | Instancia en ejecución de una imagen de Docker. No confundir con el contenedor IoC de Spring. |
| **Contenedor IoC** | Componente de Spring que crea los *beans*, les inyecta sus dependencias y gestiona su ciclo de vida. |
| **Contexto de persistencia** | Conjunto de entidades que Hibernate está gestionando dentro de una transacción. |
| **Controller** | Clase de la capa web que recibe peticiones HTTP y devuelve respuestas. |
| **CRUD** | *Create, Read, Update, Delete*: las cuatro operaciones básicas sobre datos. |
| **Cuerpo (*body*)** | Parte de un mensaje HTTP que contiene los datos (por ejemplo, un JSON). |
| **Dependencia** | Objeto que una clase necesita para hacer su trabajo. También, una librería que necesita un proyecto. |
| **Deserializar** | Convertir texto (por ejemplo, JSON) en un objeto. |
| ***Dirty checking*** | Mecanismo por el que Hibernate detecta los cambios en las entidades gestionadas y los guarda al confirmar la transacción. |
| **Docker Compose** | Herramienta para definir y arrancar varios contenedores a la vez con un fichero YAML. |
| **Dockerfile** | Fichero con las instrucciones para construir una imagen de Docker. |
| **Documento** | En MongoDB, registro con estructura tipo JSON (equivale a una fila). |
| **DTO** | *Data Transfer Object*: objeto que solo transporta los datos que entran o salen de la API. |
| **`EAGER` / `LAZY`** | Carga inmediata o bajo demanda de las entidades relacionadas. |
| **Embeber** | En MongoDB, guardar datos relacionados dentro del mismo documento. |
| ***Endpoint*** | Combinación de método HTTP y URI que ofrece una API (por ejemplo, `GET /productos`). |
| **Entidad** | Objeto que representa la información tal como la aplicación la guarda internamente. En JPA, clase Java anotada con `@Entity` que se corresponde con una tabla. |
| **Escaneo de componentes** | Búsqueda automática que hace Spring de las clases anotadas con `@Component` y sus variantes. |
| **Flyway** | Herramienta de migraciones de base de datos mediante scripts SQL versionados. |
| **Framework** | Conjunto de código ya hecho que da estructura a una aplicación y resuelve problemas comunes. |
| **HATEOAS** | *Hypermedia As The Engine Of Application State*: las respuestas incluyen enlaces a las acciones posibles. |
| **Hibernate** | Implementación de JPA más utilizada; la que usa Spring Boot por defecto. |
| **HTTP / HTTPS** | Protocolo de comunicación de la web. HTTPS es HTTP cifrado con TLS. |
| **Idempotente** | Método que deja el servidor en el mismo estado tanto si se ejecuta una vez como varias. |
| **Imagen** | Plantilla inmutable que contiene una aplicación y todo lo necesario para ejecutarla. |
| **Inversión de Control (IoC)** | Principio por el que un *framework*, y no tu código, controla la creación y conexión de los objetos. |
| **Inyección de dependencias (DI)** | Técnica en la que una clase recibe sus dependencias desde fuera en lugar de crearlas. |
| **Jackson** | Librería de Java para convertir entre objetos y JSON. Spring Boot 4 usa Jackson 3. |
| **JPA / Jakarta Persistence** | Especificación estándar de Java para el mapeo objeto-relacional. |
| **JPQL** | Lenguaje de consultas de JPA, parecido a SQL pero basado en entidades. |
| **JSON** | Formato de texto para representar datos, el más usado en las APIs REST. |
| **`JsonMapper`** | Clase principal de Jackson 3 para leer y escribir JSON. |
| **Lado propietario** | En una relación JPA, el lado que contiene la clave foránea y determina cómo se guarda la relación. |
| **Maven** | Herramienta de construcción y gestión de dependencias de Java. Se configura en `pom.xml`. |
| **Método HTTP** | Acción que se quiere realizar sobre un recurso: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`… |
| **Microservicio** | Servicio pequeño e independiente, centrado en una capacidad de negocio, que se despliega por separado. |
| **Migración** | Script versionado que modifica el esquema de la base de datos de forma controlada. |
| **Mock** | Objeto falso que simula una dependencia en una prueba. |
| **`MockMvcTester`** | Herramienta de Spring para hacer peticiones HTTP simuladas en las pruebas y comprobarlas con AssertJ. |
| **Monolito** | Aplicación que se construye y despliega como una única unidad. |
| ***Multi-stage build*** | Dockerfile con varias etapas: una para compilar y otra, mínima, para ejecutar. |
| **N+1** | Problema de rendimiento en el que se lanza una consulta por cada elemento de una lista. |
| **ORM** | *Object-Relational Mapping*: técnica para traducir entre objetos y tablas relacionales. |
| ***Package by feature*** | Organización del código en paquetes por funcionalidad de negocio en lugar de por tipo técnico. |
| **Perfil** | Conjunto de configuración que se activa según el entorno (desarrollo, producción…). |
| ***Problem Details*** | Formato estándar (RFC 9457) para devolver errores en una API HTTP. En Spring, clase `ProblemDetail`. |
| **Proyección** | Consulta que devuelve solo algunos campos, normalmente en un `record`. |
| **Puerto** | En arquitectura hexagonal, interfaz definida por el núcleo que describe lo que necesita del exterior. |
| **`record`** | Tipo de clase de Java pensado para guardar datos inmutables. |
| **Recurso** | Cualquier elemento con identidad propia que expone una API REST (un producto, un usuario…). |
| **Referenciar** | En MongoDB, guardar el identificador de otro documento en lugar de embeberlo. |
| **Repositorio** (*repository*) | Componente de la capa de persistencia que guarda y recupera datos. Con Spring Data basta con declarar una interfaz: su implementación se genera automáticamente. |
| **REST** | Estilo de arquitectura para diseñar APIs web basado en recursos, métodos HTTP estándar y ausencia de estado. |
| **Seguro (método)** | Método HTTP que no modifica nada en el servidor. |
| **Serializar** | Convertir un objeto en texto (por ejemplo, JSON). |
| **Service** | Clase de la capa de negocio que contiene las reglas de la aplicación. |
| **Sin estado (*stateless*)** | Cada petición contiene toda la información necesaria; el servidor no recuerda las anteriores. |
| **Spring Boot** | Proyecto que simplifica la creación de aplicaciones Spring con autoconfiguración, *starters* y servidor embebido. |
| **Spring Data** | Proyecto de Spring que ofrece un modelo común de acceso a datos para distintas tecnologías. |
| **Spring Initializr** | Web (<https://start.spring.io>) que genera el esqueleto de un proyecto Spring Boot. |
| ***Starter*** | Dependencia de Spring Boot que agrupa todo lo necesario para una funcionalidad. |
| **TDD** | Desarrollo guiado por pruebas: ciclo rojo → verde → refactor. |
| ***Test slice*** | Prueba que arranca solo una porción de la aplicación (`@WebMvcTest`, `@DataJpaTest`…). |
| **Testcontainers** | Librería que arranca contenedores Docker desde las pruebas. |
| **Transacción** | Grupo de operaciones que se ejecutan todas o ninguna. |
| **URI / URL** | Identificador o dirección de un recurso (`https://api.tienda.com/productos/42`). |
| **Volumen** | Almacenamiento de Docker que conserva los datos aunque se elimine el contenedor. |
