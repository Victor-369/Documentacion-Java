# Tipos de datos en MySQL

> Resumen basado en el artículo *Tipos de datos en MySQL para una base de datos SQL* (Diseño Web akus.net, última modificación 24/10/2025) y **contrastado con el Manual de Referencia oficial de MySQL 9.7**.
> Donde el artículo está desactualizado o es inexacto, se indica y se da el dato oficial (ver sección 8).

---

## 1. Estado de MySQL en 2026

- **MySQL 9.7 LTS** es la versión de soporte a largo plazo más reciente: disponible desde el 21 de abril de 2026, con soporte previsto hasta abril de 2034 según las fuentes consultadas (5 años de soporte *Premier* + 3 de *Extended*).
- **MySQL 8.4 LTS** sigue siendo una línea LTS soportada (última versión vista: 8.4.11, julio de 2026).
- **MySQL 8.0** llegó al fin de vida el **30 de abril de 2026**.
- **Nuevo esquema de versiones:** **MySQL 26.7.0** (julio de 2026) es la primera versión Innovation con versionado por calendario `AA.M.P` (26 = 2026, 7 = julio). La siguiente Innovation estaba programada como 26.10.0 (octubre de 2026). Las líneas LTS 8.4 y 9.7 conservan su numeración.
- Las cifras de este resumen se han verificado contra el manual de **MySQL 9.7**; no se han comparado una a una con 8.4 ni con 26.7.

---

## 2. Convenciones de la documentación oficial

- **M**: en enteros, ancho de visualización máximo; en coma flotante y decimales, número total de dígitos (precisión); en cadenas, longitud máxima.
- **D**: dígitos tras el punto decimal (escala). Máximo posible 30, pero no debería superar M−2.
- **fsp**: precisión de fracciones de segundo en `TIME`, `DATETIME` y `TIMESTAMP`. Va de 0 a 6; si se omite, es **0**.

El manual agrupa los tipos en: **numéricos, fecha y hora, cadenas (texto y binarios), espaciales y JSON**. El artículo de akus.net solo cubre numéricos, cadenas y fecha/hora.

---

## 3. Tipos numéricos

### 3.1 Enteros

| Tipo | Bytes | Mín. (con signo) | Máx. (con signo) | Máx. (`UNSIGNED`) |
| --- | --- | --- | --- | --- |
| `TINYINT` | 1 | -128 | 127 | 255 |
| `SMALLINT` | 2 | -32768 | 32767 | 65535 |
| `MEDIUMINT` | 3 | -8388608 | 8388607 | 16777215 |
| `INT` / `INTEGER` | 4 | -2147483648 | 2147483647 | 4294967295 |
| `BIGINT` | 8 | -9223372036854775808 | 9223372036854775807 | 18446744073709551615 |

- Con `UNSIGNED` el mínimo es 0 y el máximo positivo se duplica, aprox.
- **Cómo elegir:** el tipo más pequeño que cubra el rango esperado, pensando también en el crecimiento futuro. Ejemplos del artículo:
  - Edad → `TINYINT` (con `UNSIGNED` llega a 255).
  - Identificador de unos miles de artículos → `SMALLINT`.
  - 5 millones de clientes → `MEDIUMINT`; 200 millones → `INT`.
  - Identificar a toda la población mundial → `BIGINT`, porque `INT` solo llega a unos 2.000 millones.
- Hay que considerar también los **valores mínimos** (puntuaciones, saldos o temperaturas pueden ser negativos).

### 3.2 Decimales exactos: `DECIMAL` / `NUMERIC`

- Almacenan valores **exactos**; son los adecuados para **dinero** (`NUMERIC` se implementa como `DECIMAL`).
- Sintaxis `DECIMAL(M,D)`. Ejemplo: `DECIMAL(5,2)` guarda de `-999.99` a `999.99`.
- `DECIMAL(M)` equivale a `DECIMAL(M,0)`. Sin argumentos, M vale **10**.
- **Máximo de dígitos: 65.** El máximo de decimales es 30.
- Si se asigna un valor con más decimales de los permitidos, se convierte a la escala de la columna (normalmente por truncamiento, dependiente del sistema operativo).

### 3.3 Coma flotante (aproximados): `FLOAT` / `DOUBLE`

- Valores **aproximados**: 4 bytes (precisión simple) u 8 bytes (doble).
- `FLOAT(p)`: con `p` de 0 a 23 resulta una columna `FLOAT` de 4 bytes; con `p` de 24 a 53, una de 8 bytes (`DOUBLE`). El valor de `p` solo sirve para decidir el tamaño de almacenamiento.
- La sintaxis `FLOAT(M,D)` / `DOUBLE(M,D)` es una **extensión no estándar de MySQL y está obsoleta (*deprecated*)**; se espera que se elimine en una versión futura. Para portabilidad, usar `FLOAT` o `DOUBLE PRECISION` sin precisión ni dígitos.
- No conviene tratarlos como exactos en comparaciones; para importes, usar `DECIMAL`.

### 3.4 Otros

- `BIT` (tipo de valor de bits) aparece en el manual dentro de los numéricos; el artículo no lo trata.

---

## 4. Tipos de cadena

### 4.1 `CHAR` y `VARCHAR`

| | `CHAR(n)` | `VARCHAR(n)` |
| --- | --- | --- |
| Longitud | Fija; `n` de 0 a 255 | Variable; `n` de 0 a 65.535 |
| Almacenamiento | Se rellena con espacios a la derecha hasta `n` | Datos + prefijo de longitud de 1 byte (hasta 255 bytes) o 2 bytes (más de 255) |
| Espacios finales | Se eliminan al recuperar (salvo modo `PAD_CHAR_TO_FULL_LENGTH`) | Se conservan |
| Cuándo usarlo | Datos de longitud siempre igual (códigos de país, estado civil, `M`/`F`) | Longitud desconocida (nombres, direcciones, URL) |

Ejemplo oficial con un juego de caracteres de un byte (`latin1`): guardar `'ab'` ocupa 4 bytes en `CHAR(4)` y 3 en `VARCHAR(4)`; guardar `''` ocupa 4 y 1 bytes respectivamente.

**Límite real de `VARCHAR`:** el máximo efectivo depende del tamaño máximo de fila (**65.535 bytes compartidos entre todas las columnas**) y del juego de caracteres usado.

### 4.2 `BINARY` y `VARBINARY`

Equivalentes a `CHAR` y `VARCHAR` pero almacenan **bytes** en lugar de caracteres.

### 4.3 Familia `BLOB` y `TEXT`

- `BLOB` (`TINYBLOB`, `BLOB`, `MEDIUMBLOB`, `LONGBLOB`): cadenas **binarias**; se comparan y ordenan por el valor numérico de los bytes. Sirven para guardar el contenido de archivos (imágenes, ZIP).
- `TEXT` (`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`): cadenas de **caracteres**; se ordenan según la colación del juego de caracteres.
- Los cuatro tamaños de `TEXT` corresponden a los de `BLOB`. Tamaños que da el artículo:

| Tipo BLOB | Máximo |
| --- | --- |
| `TINYBLOB` | 255 bytes |
| `BLOB` | 65.535 bytes |
| `MEDIUMBLOB` | 16.777.215 bytes |
| `LONGBLOB` | 4 GB (o lo que permita el sistema operativo, según el artículo) |

  > Estas cifras vienen del artículo; no se contrastaron en esta revisión con la tabla oficial de requisitos de almacenamiento (sección 13.7 del manual).

**Particularidades que confirma el manual 9.7:**

- Para indexar columnas `BLOB`/`TEXT` hay que indicar una **longitud de prefijo** (en `CHAR`/`VARCHAR` es opcional).
- Las columnas `BLOB` y `TEXT` **no pueden tener valores `DEFAULT`**.
- Al ordenar solo se usan los primeros `max_sort_length` bytes (por defecto 1024).
- Si aparecen en el resultado de una consulta que usa tabla temporal, esta pasa a disco (más lento): conviene evitar `SELECT *` si no hacen falta.
- El tamaño transmisible entre cliente y servidor depende de la memoria y de `max_allowed_packet`.
- `LONG` y `LONG VARCHAR` se asignan a `MEDIUMTEXT` (compatibilidad).
- **Recomendación habitual (artículo):** no guardar imágenes en la base de datos en sitios grandes; guardar la **URL o ruta** en un `VARCHAR`. `BLOB` es para guardar el archivo en sí.
- `TEXT` se usa hoy menos de lo que se usaba antes de que `VARCHAR` admitiera hasta 65.535; reservarlo para textos realmente largos.

### 4.4 `ENUM`

- Cadena cuyo valor se elige de una **lista cerrada** definida al crear la tabla (literales entre comillas simples). Solo se puede elegir **un** valor.
- **Hasta 65.535 elementos distintos.**
- Internamente se guarda un **índice numérico** (empezando en 1), lo que ahorra espacio: un millón de filas con `'medium'` ocupan 1 millón de bytes frente a 6 millones en un `VARCHAR`.
- Un valor inválido se guarda como cadena vacía de error (índice 0), o provoca error si está activado el modo SQL estricto.
- Se ordena por índice, no alfabéticamente (listar los valores en orden alfabético o usar `ORDER BY CAST(col AS CHAR)`).
- No se recomienda usar números como valores, porque se confunden con el índice.

```sql
CREATE TABLE shirts (
  name VARCHAR(40),
  size ENUM('x-small', 'small', 'medium', 'large', 'x-large')
);
```

### 4.5 `SET`

- Cadena que puede tener **cero o más** valores de una lista definida al crear la tabla; los valores múltiples se separan por comas (por eso **los miembros no pueden contener comas**).
- **Hasta 64 miembros distintos.**
- Se almacena numéricamente (un bit por miembro). El orden o repetición al insertar da igual: al leerlo salen sin repetir y en el orden de la definición.
- Se busca con `FIND_IN_SET()` o `LIKE`.

### 4.6 `VECTOR`

El manual 9.7 incluye un tipo `VECTOR` dentro de los tipos de cadena. El artículo de origen no lo cubre y esta revisión no entró en su detalle: consultar la sección 13.3.5 del manual.

---

## 5. Tipos de fecha y hora

| Tipo | Formato de visualización | Rango |
| --- | --- | --- |
| `DATE` | `YYYY-MM-DD` | `1000-01-01` a `9999-12-31` (rango *soportado*: valores anteriores pueden funcionar, sin garantía) |
| `DATETIME[(fsp)]` | `YYYY-MM-DD hh:mm:ss[.fracción]` | `1000-01-01 00:00:00.000000` a `9999-12-31 23:59:59.499999` |
| `TIMESTAMP[(fsp)]` | `YYYY-MM-DD hh:mm:ss[.fracción]` | `1970-01-01 00:00:01` UTC a **`2038-01-19 03:14:07` UTC** |
| `TIME[(fsp)]` | `hh:mm:ss[.fracción]` | `-838:59:59` a `838:59:59` |
| `YEAR` | `YYYY` | `1901` a `2155`, o `0000` |

Puntos clave:

- MySQL usa el orden **año-mes-día**; en España/Latinoamérica se suele escribir día-mes-año, así que hay que convertir al insertar o mostrar.
- `fsp` (fracciones de segundo) de 0 a 6, **0 por defecto**: `DATETIME(6)` guarda microsegundos.
- **`TIMESTAMP`** se guarda como segundos desde la época Unix (`1970-01-01 00:00:00` UTC). Su límite del año **2038** es relevante para datos con fechas futuras: para esos casos usar `DATETIME`.
- **Inicialización/actualización automática:** tanto `TIMESTAMP` como `DATETIME` admiten `DEFAULT CURRENT_TIMESTAMP` y `ON UPDATE CURRENT_TIMESTAMP`, útiles para registrar la fecha de creación o de última modificación sin programar nada. El comportamiento por defecto del primer `TIMESTAMP` de una tabla depende de la variable `explicit_defaults_for_timestamp`.
- `YEAR(4)` con ancho explícito está obsoleto; usar `YEAR` sin ancho.
- `SUM()` y `AVG()` **no funcionan** con valores temporales (los convierten a número y pierden información). Solución: convertir a unidades numéricas y volver a convertir, por ejemplo `SEC_TO_TIME(SUM(TIME_TO_SEC(col)))`.
- Alternativa que cita el artículo: guardar un *timestamp* Unix generado desde PHP en un `INT` y formatearlo en la aplicación (ojo con el límite de 2038 si se usa `INT` con signo).

---

## 6. Atributos y modificadores de columna

Resumen de lo que explica el artículo (pensado para phpMyAdmin) con matices.

| Atributo | Qué hace |
| --- | --- |
| `NULL` / `NOT NULL` | Permite o prohíbe dejar el campo sin valor. El artículo dice que "por omisión todos son NOT NULL", pero eso se refiere al formulario de phpMyAdmin. En SQL, una columna que no declara `NOT NULL` admite `NULL` (comprobable con `SHOW CREATE TABLE`). |
| `DEFAULT` | Valor por defecto (texto fijo, `NULL` o `CURRENT_TIMESTAMP`). **No admitido en `BLOB` ni `TEXT`.** |
| `UNSIGNED` | Sin signo en enteros; duplica el máximo positivo. |
| `BINARY` | En `CHAR`/`VARCHAR`, hace que el orden distinga mayúsculas y minúsculas. En `TEXT` asigna la colación binaria (`_bin`) del juego de caracteres. |
| `INDEX` | Acelera búsquedas ordenando internamente por ese campo. |
| `PRIMARY KEY` | Identifica de forma única cada fila: no admite repetidos ni nulos. Puede ser un campo o una combinación (clave compuesta). |
| `AUTO_INCREMENT` | Incrementa automáticamente el valor numérico en cada alta; típico en claves primarias enteras. |
| `UNIQUE` | Impide valores repetidos sin ser clave primaria (DNI, email, nombre de usuario). |
| `FULLTEXT` | Índice para búsquedas de palabras dentro de textos largos. |

> **Sobre `FULLTEXT`:** el artículo afirma que ignora palabras de menos de cuatro caracteres y que no devuelve resultados si la palabra aparece en más del 50 % de las filas. Estos detalles **no se han contrastado** con el manual 9.7; consultar la documentación oficial de búsqueda de texto completo antes de depender de ellos.

---

## 7. Guía rápida de elección

> Tabla de **recomendaciones propias** basada en lo anterior; no figura así en el artículo.

| Dato | Tipo sugerido | Motivo |
| --- | --- | --- |
| Edad | `TINYINT UNSIGNED` | Cabe en 0–255 |
| Identificador autoincremental | `INT UNSIGNED` o `BIGINT UNSIGNED` | Según el volumen previsto |
| Dinero, precios, saldos | `DECIMAL(M,D)` | Exactitud, sin redondeos |
| Medidas científicas aproximadas | `DOUBLE` | Coma flotante, aproximado |
| Código fijo (país, `M`/`F`) | `CHAR(n)` | Longitud constante |
| Nombre, email, URL | `VARCHAR(n)` | Longitud variable |
| Texto muy largo | `TEXT` / `MEDIUMTEXT` | Sin `DEFAULT`; índices con prefijo |
| Estado de una lista cerrada | `ENUM(...)` | Compacto y legible |
| Fecha sin hora | `DATE` | |
| Fecha y hora con fechas futuras lejanas | `DATETIME` | No tiene el límite de 2038 |
| Fecha de creación/modificación | `TIMESTAMP` o `DATETIME` con `CURRENT_TIMESTAMP` | Se rellena sola |
| Número de teléfono | `VARCHAR` | No se opera aritméticamente y puede llevar `+` o ceros iniciales |

---

## 8. Correcciones respecto al artículo de akus.net

| Tema | El artículo dice | Manual oficial MySQL 9.7 |
| --- | --- | --- |
| Rango de `TIME` | `-839:59:59` a `839:59:59` | `-838:59:59` a `838:59:59` |
| Dígitos máximos de `DECIMAL` | 64 | **65** (escala máxima 30) |
| `FLOAT` / `DOUBLE` | Los decimales de `FLOAT` van de 0 a 24 y los de `DOUBLE` de 25 a 53; usa `FLOAT(6.2)` | El rango 0–23 / 24–53 es de la **precisión `p`** en bits de `FLOAT(p)`; `FLOAT(M,D)` y `DOUBLE(M,D)` están **obsoletos** |
| Rango de `TIMESTAMP` | Hasta "el año 2037"; formatos de 14, 8 o 6 dígitos | `1970-01-01 00:00:01` a **`2038-01-19 03:14:07` UTC** |
| `YEAR` | Admite 2 dígitos (70–99 → 1970–1999, 00–69 → 2000–2069) | El manual 9.7 describe `YEAR` de 4 dígitos (1901–2155 o 0000); `YEAR(4)` está obsoleto |
| Longitud de `VARCHAR` | Hasta 65.535 *caracteres* | Hasta 65.535, pero limitado por el tamaño de fila (65.535 bytes compartidos) y el juego de caracteres |
| `NOT NULL` por omisión | Todos los campos | Es el comportamiento del formulario de phpMyAdmin; en SQL, sin `NOT NULL` se admite `NULL` |
| Typo | `LONGTEX` | Es `LONGTEXT` |
| Cobertura | Solo numéricos, cadenas y fecha/hora | El manual incluye también espaciales, JSON y, en 9.7, `VECTOR` |

---

## 9. Seguir leyendo (manual oficial)

- Capítulo de tipos de datos, MySQL 9.7: <https://dev.mysql.com/doc/refman/9.7/en/data-types.html>
- Enteros: <https://dev.mysql.com/doc/refman/9.7/en/integer-types.html>
- `DECIMAL`/`NUMERIC`: <https://dev.mysql.com/doc/refman/9.7/en/fixed-point-types.html>
- `FLOAT`/`DOUBLE`: <https://dev.mysql.com/doc/refman/9.7/en/floating-point-types.html>
- Fecha y hora: <https://dev.mysql.com/doc/refman/9.7/en/date-and-time-type-syntax.html>
- `CHAR`/`VARCHAR`: <https://dev.mysql.com/doc/refman/9.7/en/char.html>
- `BLOB`/`TEXT`: <https://dev.mysql.com/doc/refman/9.7/en/blob.html>
- `ENUM`: <https://dev.mysql.com/doc/refman/9.7/en/enum.html>
- `SET`: <https://dev.mysql.com/doc/refman/9.7/en/set.html>

---

## Fuentes

- Diseño Web akus.net: [Tipos de datos en MySQL para una base de datos SQL](https://disenowebakus.net/tipos-de-datos-mysql.php) (modificado el 24/10/2025).
- Oracle: [MySQL 9.7 Reference Manual, capítulo 13 Data Types](https://dev.mysql.com/doc/refman/9.7/en/data-types.html) y secciones enlazadas arriba.
- Oracle, blog de MySQL: [Lanzamientos de julio de 2026 (9.7.2 LTS, 8.4.11 LTS y 26.7.0)](https://blogs.oracle.com/mysql/?p=5150).
- Oracle: [Anuncio de MySQL 9.7.0 LTS (21 de abril de 2026)](https://www.mysql.com/de/news-and-events/).
- Medha Cloud: [MySQL versions](https://medhacloud.com/blog/mysql-versions) (fechas de soporte de 9.7 y fin de vida de 8.0).
- Datavail: [Data Platforms Reaching End of Life in April 2026](https://www.datavail.com/blog/data-platforms-reaching-end-of-life-in-april-2026/).
- Percona: [Meet MySQL 26.7: A New Versioning Era Begins](https://www.percona.com/?p=54581).
