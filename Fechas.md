# Fechas y horas en Java (`java.time`)

Guía basada en el tutorial de W3Schools «Java Date and Time», corregida y ampliada con la documentación oficial de Oracle (Javadoc de Java 21 y 25) y con los cambios relevantes desde Java 21.

**Convenciones de este documento**

- ✅ = ejemplo compilado y ejecutado con OpenJDK 21.
- ⚠️ = requiere una versión posterior a 21 y **no se ha podido compilar en local**; se toma de las notas de versión oficiales.
- Las secciones 1 a 6 resumen W3Schools. A partir de la 7 son ampliaciones.
- Los ejemplos usan **fechas fijas** para que la salida sea reproducible. Con `now()` el resultado depende del momento y de la zona horaria del sistema.

---

## 1. La API de fecha y hora

Para trabajar con fechas y horas se usa el paquete **`java.time`** (la API de fecha y hora, disponible desde Java 8). Contiene muchas clases; las principales según W3Schools son:

| Clase | Descripción |
|---|---|
| `LocalDate` | Una fecha (año, mes, día: `yyyy-MM-dd`) |
| `LocalTime` | Una hora (hora, minuto, segundo y nanosegundos) |
| `LocalDateTime` | Fecha y hora a la vez |
| `DateTimeFormatter` | Formateador para mostrar e interpretar (*parsear*) objetos de fecha y hora |

> **Corrección importante.** W3Schools afirma que «Java no tiene una clase `Date` integrada». Es impreciso: **sí existe `java.util.Date`**, pero es la API antigua (mutable y con un diseño problemático). La documentación de Oracle indica que `java.time.Instant` es el equivalente más cercano a `java.util.Date`, y `ZonedDateTime` el más cercano a `java.util.GregorianCalendar`. En código nuevo se usa únicamente `java.time`.

## 2. Mostrar la fecha actual

```java
import java.time.LocalDate;

public class Main {
  public static void main(String[] args) {
    LocalDate miObj = LocalDate.now();   // crea un objeto fecha
    System.out.println(miObj);           // p. ej. 2026-10-16
  }
}
```

La salida tiene el formato ISO `año-mes-día`.

## 3. Mostrar la hora actual

```java
import java.time.LocalTime;

public class Main {
  public static void main(String[] args) {
    LocalTime miObj = LocalTime.now();
    System.out.println(miObj);           // p. ej. 14:30:15.123456789
  }
}
```

Muestra hora, minutos, segundos y nanosegundos. Es la hora **del sistema donde se ejecuta el programa** (si es un servidor, puede no coincidir con tu hora local).

## 4. Mostrar fecha y hora actuales

```java
import java.time.LocalDateTime;

public class Main {
  public static void main(String[] args) {
    LocalDateTime miObj = LocalDateTime.now();
    System.out.println(miObj);           // p. ej. 2026-10-16T14:30:15.123456789
  }
}
```

La **«T»** de la salida separa la fecha de la hora (formato ISO-8601).

Con valores concretos ✅:

```java
LocalDate d = LocalDate.of(1988, 9, 29);
LocalTime t = LocalTime.of(14, 30, 15);
LocalDateTime dt = LocalDateTime.of(d, t);
// 1988-09-29 | 14:30:15 | 1988-09-29T14:30:15
```

## 5. Dar formato a fecha y hora

Se usa `DateTimeFormatter` con el método `ofPattern()` para formatear o interpretar objetos de fecha y hora. El siguiente ejemplo quita la «T» y los nanosegundos ✅:

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Main {
  public static void main(String[] args) {
    LocalDateTime miFechaObj = LocalDateTime.of(1988, 9, 29, 14, 30, 15);
    System.out.println("Antes de formatear: " + miFechaObj);
    DateTimeFormatter miFormatoObj = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");

    String fechaFormateada = miFechaObj.format(miFormatoObj);
    System.out.println("Después de formatear: " + fechaFormateada);
  }
}
```

Salida ✅:

```text
Antes de formatear: 1988-09-29T14:30:15
Después de formatear: 29-09-1988 14:30:15
```

## 6. Patrones habituales (W3Schools)

`ofPattern()` acepta muchos patrones. Los de W3Schools, con su resultado real ✅ (primero en inglés, como en el tutorial, y después con la configuración regional de España):

| Patrón | Inglés (`Locale.ENGLISH`) | Español de España (`es-ES`) |
|---|---|---|
| `yyyy-MM-dd` | `1988-09-29` | `1988-09-29` |
| `dd/MM/yyyy` | `29/09/1988` | `29/09/1988` |
| `dd-MMM-yyyy` | `29-Sep-1988` | `29-sept-1988` |
| `E, MMM dd yyyy` | `Thu, Sep 29 1988` | `jue, sept 29 1988` |

> Los patrones que incluyen **nombres** de mes o de día dependen del *locale*. Las salidas en español de esta tabla son las observadas en OpenJDK 21 (con los datos CLDR); la abreviatura del mes («sept») puede variar entre versiones del JDK.

---

## 7. Ampliación: qué clase usar en cada caso

Según la documentación oficial de Oracle (paquete `java.time`):

| Clase | Qué almacena | Ejemplo ✅ | Uso típico |
|---|---|---|---|
| `Instant` | Un instante en la línea temporal (marca de tiempo numérica, en UTC) | `2011-12-03T10:15:30Z` | Registro de eventos, persistencia de un momento exacto |
| `LocalDate` | Fecha sin hora ni zona | `2026-10-16` | Cumpleaños, vencimientos, festivos |
| `LocalTime` | Hora sin fecha ni zona | `11:30` | Horarios de apertura o cierre |
| `LocalDateTime` | Fecha y hora, sin zona ni desfase | `2026-10-16T14:05:09` | Hora «de pared» dentro de una zona ya conocida |
| `ZonedDateTime` | Fecha, hora y zona horaria con sus reglas | `2026-07-01T12:00+02:00[Europe/Madrid]` | Cálculos que dependen de la zona y del horario de verano |
| `OffsetDateTime` / `OffsetTime` | Fecha-hora (u hora) con desfase respecto a UTC | `1988-09-29T14:30:15+02:00` | Protocolos de red y bases de datos |
| `Duration` | Cantidad de tiempo en segundos y nanosegundos | `PT1H45M` | Tiempos de espera, tiempo transcurrido |
| `Period` | Cantidad de tiempo en años, meses y días | `P3M5D` | Diferencias entre fechas |
| `Year`, `YearMonth`, `MonthDay` | Año; año y mes; mes y día | `2026-02`, `--12-03` | Caducidad de tarjeta (`YearMonth`), cumpleaños sin año (`MonthDay`) |
| `Month`, `DayOfWeek` | Mes o día de la semana (son `enum`) | `DECEMBER`, `TUESDAY` | |
| `Clock` / `InstantSource` | Fuente del instante actual | | Pruebas con tiempo controlado |

Recomendaciones de la propia documentación:

- Usa `LocalDate`, `LocalTime` y `LocalDateTime` siempre que puedas, porque modelan mejor el dominio y **evitan la complejidad añadida de las zonas horarias**.
- Muchas aplicaciones pueden escribirse solo con `LocalDate`, `LocalTime` e `Instant`, añadiendo la zona horaria en la capa de interfaz de usuario.
- Los tipos con desfase (`OffsetDateTime`, `OffsetTime`) están pensados sobre todo para protocolos de red y acceso a bases de datos.
- Entre sistemas (base de datos, red) se recomienda usar las clases ISO-8601 de `java.time`.
- Todas las clases son **inmutables y seguras entre hilos**.

> **Un `LocalDateTime` no es un instante.** No sabe en qué zona está, así que no identifica un momento concreto de la línea temporal. Para eso se necesita `Instant`, `ZonedDateTime` u `OffsetDateTime`.

## 8. Inmutabilidad y nombres de métodos

Como los objetos son inmutables, los métodos que «modifican» devuelven **un objeto nuevo** ✅:

```java
LocalDate hoy = LocalDate.of(2026, 10, 16);
LocalDate venc = hoy.plusDays(30);

hoy;                       // 2026-10-16   (no cambia)
venc;                      // 2026-11-15
hoy.withDayOfMonth(1);     // 2026-10-01
hoy.getDayOfWeek();        // FRIDAY
hoy.isLeapYear();          // false
```

La API usa prefijos coherentes (según el Javadoc):

| Prefijo | Significado |
|---|---|
| `of` | Método estático de fábrica |
| `parse` | Fábrica que interpreta un texto |
| `get` | Obtiene un valor |
| `is` | Comprueba una condición |
| `with` | El equivalente inmutable de un *setter* |
| `plus` / `minus` | Suma o resta una cantidad |
| `to` | Convierte a otro tipo |
| `at` | Combina con otro objeto (`fecha.atTime(hora)`) |

Ajustadores útiles ✅:

```java
hoy.with(TemporalAdjusters.next(DayOfWeek.MONDAY));    // 2026-10-19
hoy.with(TemporalAdjusters.lastDayOfMonth());          // 2026-10-31
LocalDate.of(2026, 1, 31).plusMonths(1);               // 2026-02-28  (ajusta al último día válido)
```

## 9. Diferencias entre fechas: `Period`, `Duration` y `ChronoUnit` ✅

```java
LocalDate ini = LocalDate.of(2024, 1, 15);
LocalDate fin = LocalDate.of(2024, 4, 20);

Period.between(ini, fin);                 // P3M5D  (3 meses y 5 días)
ChronoUnit.DAYS.between(ini, fin);        // 96     (días totales)

Duration du = Duration.between(LocalTime.of(10, 30), LocalTime.of(12, 15));
du;                                       // PT1H45M
du.toMinutes();                           // 105
du.toHoursPart() + ":" + du.toMinutesPart();   // 1:45
```

`Period` expresa cantidades «humanas» (años, meses, días); `Duration` expresa cantidades exactas de tiempo (horas, minutos, segundos).

## 10. Zonas horarias e instantes ✅

```java
ZonedDateTime madrid = ZonedDateTime.of(2026, 7, 1, 12, 0, 0, 0, ZoneId.of("Europe/Madrid"));
madrid;                                                    // 2026-07-01T12:00+02:00[Europe/Madrid]
madrid.withZoneSameInstant(ZoneId.of("America/New_York")); // 2026-07-01T06:00-04:00[America/New_York]

Instant i = Instant.parse("2011-12-03T10:15:30Z");
i.toEpochMilli();                                          // 1322907330000
i.atZone(ZoneId.of("Europe/Madrid")).toLocalDateTime();    // 2011-12-03T11:15:30

YearMonth.of(2026, 2).lengthOfMonth();                     // 28
MonthDay.of(12, 3);                                        // --12-03
Year.of(2024).isLeap();                                    // true
```

### Conversión con la API antigua ✅

```java
Instant i = Instant.parse("2026-10-16T08:00:00Z");
Date antiguo = Date.from(i);          // java.util.Date para una API heredada
antiguo.toInstant();                  // de vuelta a Instant
```

## 11. Probar código que depende de la hora: `Clock` e `InstantSource` ✅

Llamar a `now()` directamente hace el código difícil de probar. `Clock` (y `InstantSource`, que aparece en el Javadoc de Java 21) permiten inyectar una fuente de tiempo:

```java
Clock reloj = Clock.fixed(Instant.parse("2026-10-16T08:00:00Z"), ZoneId.of("Europe/Madrid"));
LocalDateTime.now(reloj);             // 2026-10-16T10:00

InstantSource fuente = InstantSource.fixed(Instant.parse("2026-10-16T08:00:00Z"));
fuente.instant();                     // 2026-10-16T08:00:00Z
```

---

## 12. Formateo en profundidad (`DateTimeFormatter`)

Un formateador creado desde un patrón es **inmutable y seguro entre hilos**, así que puede reutilizarse (por ejemplo, como constante). Esa es una diferencia importante con el antiguo `SimpleDateFormat`.

Tres formas de crearlo (Javadoc):

1. **Constantes predefinidas**, como `ISO_LOCAL_DATE`.
2. **Letras de patrón**, como `uuuu-MMM-dd`.
3. **Estilos localizados**, como `long` o `medium`.

### 12.1. Formateadores ISO predefinidos

| Formateador | Ejemplo (del Javadoc) |
|---|---|
| `ISO_LOCAL_DATE` | `2011-12-03` |
| `ISO_LOCAL_TIME` | `10:15:30` |
| `ISO_LOCAL_DATE_TIME` | `2011-12-03T10:15:30` |
| `ISO_OFFSET_DATE_TIME` | `2011-12-03T10:15:30+01:00` |
| `ISO_ZONED_DATE_TIME` | `2011-12-03T10:15:30+01:00[Europe/Paris]` |
| `ISO_INSTANT` | `2011-12-03T10:15:30Z` |
| `BASIC_ISO_DATE` | `20111203` |
| `ISO_ORDINAL_DATE` | `2012-337` |
| `ISO_WEEK_DATE` | `2012-W48-6` |
| `RFC_1123_DATE_TIME` | `Tue, 3 Jun 2008 11:05:30 GMT` |

Para interoperar entre sistemas se recomienda usar formatos estándar ISO siempre que sea posible.

### 12.2. Letras de patrón más usadas (Javadoc de Java 25)

Todas las letras de la `A` a la `Z` y de la `a` a la `z` están reservadas.

| Letra | Significado | Ejemplos |
|---|---|---|
| `u` | Año | `2004`; `04` |
| `y` | Año de la era | `2004`; `04` |
| `M` / `L` | Mes del año (número o texto) | `7`; `07`; `Jul`; `July` |
| `d` | Día del mes | `10` |
| `D` | Día del año | `189` |
| `E` | Día de la semana (texto) | `Tue`; `Tuesday` |
| `a` | AM/PM | `PM` |
| `H` | Hora del día (0-23) | `0` |
| `h` | Hora de AM/PM (1-12) | `12` |
| `m` | Minuto | `30` |
| `s` | Segundo | `55` |
| `S` | Fracción de segundo | `978` |
| `n` | Nanosegundo | `987654321` |
| `V` | Identificador de zona horaria | `America/Los_Angeles` |
| `z` | Nombre de zona | `Pacific Standard Time`; `PST` |
| `X` / `x` / `Z` / `O` | Desfase de zona | `-08:30`; `+0000`; `GMT+8` |
| `Y` | **Año basado en semanas** | `1996`; `96` |
| `[` `]` | Sección opcional | |
| `'` | Escapar texto literal | |

Cómo se interpreta el **número de letras**:

- **Texto:** menos de 4 letras → forma corta; exactamente 4 → forma completa; exactamente 5 → forma estrecha. Ejemplo: `MMM` → «sept», `MMMM` → «septiembre».
- **Número:** con una letra se muestra el mínimo de dígitos; con más, se rellena con ceros hasta ese ancho (`dd`, `HH`).
- **Año:** con dos letras (`yy`) se imprimen los dos últimos dígitos y, al interpretar, se toma como base el año 2000 (resultado entre 2000 y 2099).

Para escribir texto literal se recomienda **entrecomillarlo siempre con comillas simples**, para que futuras versiones no rompan tu aplicación:

```java
DateTimeFormatter.ofPattern("EEEE, d 'de' MMMM 'de' uuuu", Locale.of("es", "ES"));
// viernes, 16 de octubre de 2026 ✅
```

### 12.3. Trampa clásica: `yyyy`, `uuuu` y `YYYY` ✅

- `y` = **año de la era**. `u` = **año**. Para fechas actuales dan lo mismo, pero con el modo estricto `y` exige indicar la era (ver 13.2).
- `Y` = **año basado en semanas**: puede dar un año distinto en los últimos y primeros días del año. Casi siempre es un error usarlo al formatear una fecha normal:

```java
LocalDate fin = LocalDate.of(2025, 12, 29);
fin.format(DateTimeFormatter.ofPattern("yyyy"));               // 2025
fin.format(DateTimeFormatter.ofPattern("YYYY", Locale.US));    // 2026   ← sorpresa
```

### 12.4. Estilos localizados ✅

Dejan que el formato dependa de la configuración regional:

```java
Locale es = Locale.of("es", "ES");

DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG).withLocale(es);    // 29 de septiembre de 1988
DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).withLocale(es);  // 29 sept 1988
DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT).withLocale(es);   // 29/9/88
```

Los estilos `FULL` y `LONG` aplicados a una hora suelen necesitar una zona horaria: usa un `ZonedDateTime` o `withZone(...)`.

### 12.5. Plantillas localizadas: `ofLocalizedPattern` ✅

Disponible desde **Java 19**: construye el formato más adecuado para el *locale* a partir de una plantilla de campos:

```java
DateTimeFormatter.ofLocalizedPattern("yMMM").withLocale(Locale.US);   // «Sep 1988»
```

### 12.6. `Locale.of(...)`

Desde Java 19 se recomienda `Locale.of("es", "ES")` (o `Locale.forLanguageTag("es-ES")`). Los constructores `new Locale(...)` están marcados como obsoletos ✅ (el compilador avisa: `Locale(String,String) in Locale has been deprecated`).

---

## 13. Interpretar (*parsear*) textos

### 13.1. Con formato ISO ✅

```java
LocalDate.parse("2026-10-16");                                            // formato ISO por defecto
LocalDate.parse("29/09/1988", DateTimeFormatter.ofPattern("dd/MM/uuuu")); // 1988-09-29

try { LocalDate.parse("2023-02-30"); }
catch (DateTimeParseException e) { /* Text '2023-02-30' could not be parsed: Invalid date 'FEBRUARY 30' */ }
```

### 13.2. Modos de resolución: `SMART`, `STRICT` y `LENIENT` ✅

El interpretado se hace en dos fases: primero se lee el texto y luego se **resuelven** los campos. El modo por defecto de un formateador creado con `ofPattern` es `SMART`.

```java
// SMART (por defecto): ajusta 30 de febrero al último día válido
LocalDate.parse("30/02/2023", DateTimeFormatter.ofPattern("dd/MM/uuuu"));
// 2023-02-28

// STRICT con "uuuu": rechaza la fecha inválida
LocalDate.parse("30/02/2023",
        DateTimeFormatter.ofPattern("dd/MM/uuuu").withResolverStyle(ResolverStyle.STRICT));
// DateTimeParseException: Invalid date 'FEBRUARY 30'

// STRICT con "yyyy": falla por falta de la era
LocalDate.parse("30/02/2023",
        DateTimeFormatter.ofPattern("dd/MM/yyyy").withResolverStyle(ResolverStyle.STRICT));
// DateTimeParseException: Unable to obtain LocalDate from TemporalAccessor: {MonthOfYear=2, YearOfEra=2023, DayOfMonth=30}...
```

Moraleja: **en modo estricto usa `uuuu` y no `yyyy`**.

### 13.3. Partes opcionales y `parseBest` ✅

```java
DateTimeFormatter f = DateTimeFormatter.ofPattern("uuuu-MM-dd HH.mm[ VV]");

f.parseBest("2026-10-16 10.30 Europe/Madrid", ZonedDateTime::from, LocalDateTime::from);  // ZonedDateTime
f.parseBest("2026-10-16 10.30", ZonedDateTime::from, LocalDateTime::from);                // LocalDateTime
```

---

## 14. Novedades y cambios relevantes desde Java 21

> **Dato honesto:** en las fuentes oficiales consultadas (Javadoc de Java 21 y 25 y notas de versión) **no aparecen clases nuevas en `java.time`** posteriores a Java 21. Los cambios importantes son de **comportamiento y de datos de localización**. Si ves en algún sitio una «gran novedad de la API de fechas en Java 22–25», verifícala con el Javadoc oficial.

### 14.1. Datos CLDR 42 (Java 20) y su efecto en Java 21

Desde **Java 20** el JDK usa la versión 42 de los datos de localización Unicode CLDR. Como consecuencia, en algunos *locales* el espacio entre la hora y el marcador AM/PM pasó de ser un espacio ASCII normal (U+0020) a un **espacio estrecho sin salto (NNBSP, U+202F)**. Efectos:

- Un texto escrito a mano con un espacio normal **ya no se interpreta** con formatos localizados, porque el analizador espera el carácter nuevo.
- Las diferencias son difíciles de detectar a simple vista, porque ambos espacios se ven casi idénticos.

Recomendaciones (fuente: Inside.java, Oracle, y la wiki de Spring):

- Usa **formatos ISO** (`ISO_LOCAL_DATE_TIME`, etc.) para intercambiar datos entre sistemas.
- Reserva los formatos localizados para **mostrar** información al usuario.
- Según la wiki de Spring, en JDK 20 a 22 se podía forzar el comportamiento antiguo con `-Djava.locale.providers=COMPAT`, pero ese modo **se eliminó en JDK 23**.

### 14.2. Nombres de eras (Java 22)

Según las notas de versión de JDK 22, los nombres de las eras gregorianas que devuelven las API de `java.time.format` se obtienen ahora de los datos **CLDR** y no de los antiguos datos `COMPAT`. Por ejemplo, para el *locale* `ROOT` se devuelve «BCE»/«CE» en lugar de «BC»/«AD».

### 14.3. Interpretación «laxa» de espacios (Java 23) ⚠️

Desde **Java 23**, al interpretar textos de fecha/hora se permite una **coincidencia laxa de espacios**, para paliar el problema del NNBSP. Solo se aplica en el estilo *lenient*; en el estilo por defecto (*strict*) los espacios se siguen considerando distintos. En `java.time.format` hay que activarlo explícitamente con `parseLenient()`. Ejemplo de las notas de versión de JDK 23 ⚠️ (no compilado en local):

```java
var dtf = new DateTimeFormatterBuilder()
    .parseLenient()
    .append(DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT))
    .toFormatter(Locale.ENGLISH);
```

### 14.4. Corrección de un error de serialización en Java 25.0.x

Las notas de versión de Java SE 25.0.3 (29 de mayo de 2026) indican que se corrigió una **regresión en la serialización de objetos de `java.time`** (incluido `LocalDate`), revirtiendo el cambio JDK-8334742 (error JDK-8367031). Conviene mantener el JDK en la última actualización de su rama.

### 14.5. Contexto de versiones

- **Java 21** (septiembre de 2023) es una versión LTS.
- **Java 25** (septiembre de 2025) es la **última LTS vigente**.
- Versiones intermedias como Java 22, 23, 24 y 26 no son LTS.

---

## 15. Resumen y buenas prácticas

- Usa **`java.time`** en todo el código nuevo; deja `java.util.Date` y `Calendar` solo para interoperar con API heredadas.
- Elige el tipo más simple que modele tu caso: `LocalDate` para fechas, `Instant` para marcas de tiempo, `ZonedDateTime` solo cuando importe la zona.
- Almacena y transmite en **formato ISO** y formatea para el usuario en el último momento.
- Define los `DateTimeFormatter` como **constantes** (son inmutables y seguros entre hilos).
- Usa `uuuu` con modo estricto y **evita `YYYY`** salvo que quieras de verdad el año de semanas.
- Pasa siempre el *locale* explícitamente (`Locale.of("es", "ES")`) cuando el resultado incluya texto.
- Para pruebas, inyecta un `Clock` o `InstantSource` en lugar de llamar a `now()` en todas partes.

## Fuentes

- W3Schools, [Java Date and Time](https://www.w3schools.com/java/java_date.asp)
- Oracle, [Paquete `java.time` (Java SE 25)](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/package-summary.html)
- Oracle, [`DateTimeFormatter` (Java SE 25)](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/format/DateTimeFormatter.html)
- Oracle, [Paquete `java.time` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html)
- Oracle, [Notas de versión de JDK 22](https://www.oracle.com/java/technologies/javase/22-relnote-issues.html)
- Oracle, [Notas de versión consolidadas de JDK 23](https://www.oracle.com/java/technologies/javase/23all-relnotes.html)
- Oracle, [Notas de versión consolidadas de JDK 25](https://www.oracle.com/java/technologies/javase/25all-relnotes.html)
- Inside.java, [Quality Outreach Heads-up – JDK 20-23: Support for Unicode CLDR Version 42](https://inside.java/2024/03/29/quality-heads-up/)
- Spring Framework (wiki), [Date and Time Formatting with JDK 20 and higher](https://github.com/dancer1325/spring-framework/wiki/Date-and-Time-Formatting-with-JDK-20-and-higher/72056d18cb4a062b80419d08f709fbf3a8917fc4) (fuente secundaria: copia de la wiki)
