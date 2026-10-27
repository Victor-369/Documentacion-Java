# Principio DRY (*Don't Repeat Yourself*)

> Versión de referencia: **Java 25 (LTS)** o superior. Todos los ejemplos usan únicamente características definitivas del lenguaje (nada en *preview*).

## Índice

1. [¿Qué es DRY?](#1-qué-es-dry)
2. [Origen del principio](#2-origen-del-principio)
3. [Duplicar código frente a duplicar conocimiento](#3-duplicar-código-frente-a-duplicar-conocimiento)
4. [Ventajas de aplicar DRY](#4-ventajas-de-aplicar-dry)
5. [Ejemplo básico: validación de correos electrónicos](#5-ejemplo-básico-validación-de-correos-electrónicos)
6. [Técnicas para eliminar la duplicación](#6-técnicas-para-eliminar-la-duplicación)
7. [Herramientas de Java moderno que ayudan a aplicar DRY](#7-herramientas-de-java-moderno-que-ayudan-a-aplicar-dry)
8. [Dónde aparece DRY en proyectos reales](#8-dónde-aparece-dry-en-proyectos-reales)
9. [Violaciones habituales de DRY](#9-violaciones-habituales-de-dry)
10. [DRY y el principio de responsabilidad única (SRP)](#10-dry-y-el-principio-de-responsabilidad-única-srp)
11. [Cuándo **no** aplicar DRY](#11-cuándo-no-aplicar-dry)
12. [Lista de comprobación](#12-lista-de-comprobación)
13. [Referencias](#13-referencias)

---

## 1. ¿Qué es DRY?

**DRY** son las siglas de *Don't Repeat Yourself* («no te repitas»). Es un principio de desarrollo de software cuyo objetivo es **evitar la duplicación de lógica**, fomentando que el código y los componentes sean reutilizables.

La idea central es sencilla: **cada regla, cálculo o dato del sistema debe estar definido en un único lugar**. Así, cuando esa regla cambia, solo hay que modificarla en un sitio y el cambio se propaga de forma coherente a todo el programa.

Su contrario se conoce informalmente como **WET** (*Write Everything Twice* o *We Enjoy Typing*): código en el que la misma lógica aparece copiada en varios puntos.

## 2. Origen del principio

El principio fue formulado por **Andy Hunt** y **Dave Thomas** en el libro *The Pragmatic Programmer* (1999). Su definición, traducida y resumida, dice que **cada pieza de conocimiento debe tener una representación única, inequívoca y autorizada dentro del sistema**.

Fíjate en que la definición habla de *conocimiento*, no de *líneas de código*. Esta distinción es muy importante y se explica en el siguiente apartado.

## 3. Duplicar código frente a duplicar conocimiento

No toda repetición de texto es una violación de DRY. Lo que DRY persigue es que **una misma decisión de negocio o técnica** no esté repartida por el código.

| Situación | ¿Viola DRY? | Motivo |
|---|---|---|
| La fórmula del IVA aparece copiada en tres servicios | **Sí** | Es la misma regla de negocio; si cambia el tipo, hay que tocar tres sitios. |
| La longitud máxima del nombre de usuario (`50`) está escrita «a mano» en el formulario, en la validación y en la base de datos | **Sí** | Es un mismo dato repetido. |
| Dos métodos tienen un `for` parecido, pero uno recorre facturas y otro recorre pedidos, con reglas que evolucionan por separado | **No necesariamente** | El parecido es casual; no representan el mismo conocimiento. |

> **Regla práctica:** pregúntate «si esta regla cambia mañana, ¿tendría que cambiarla en más de un sitio?». Si la respuesta es sí, hay duplicación de conocimiento.

## 4. Ventajas de aplicar DRY

- **Reutilización de código:** se crean componentes reutilizables y se reduce la redundancia en todo el proyecto.
- **Mantenimiento más sencillo:** al centralizar la lógica, las modificaciones se hacen en un único punto, lo que reduce los errores.
- **Mejor legibilidad:** sin repeticiones, el código es más fácil de entender y de recorrer.
- **Desarrollo más rápido:** no hay que volver a implementar la misma lógica una y otra vez.
- **Coherencia:** el comportamiento es uniforme porque la funcionalidad está encapsulada en un solo lugar.
- **Mejor colaboración:** la lógica compartida está claramente definida, lo que facilita que varias personas trabajen sin pisarse.
- **Menos errores de copiar y pegar:** un fallo no se replica en varias partes del sistema.
- **Más fácil de probar:** la lógica aislada en un componente reutilizable se prueba una sola vez con pruebas unitarias.

## 5. Ejemplo básico: validación de correos electrónicos

### ❌ Sin DRY: la misma lógica copiada

```java
public class RegistroService {

    public void registrar(String email) {
        if (email == null || !email.contains("@") || !email.contains(".")) {
            throw new IllegalArgumentException("Correo no válido: " + email);
        }
        // ... lógica de registro
    }
}

public class NewsletterService {

    public void suscribir(String email) {
        if (email == null || !email.contains("@") || !email.contains(".")) {
            throw new IllegalArgumentException("Correo no válido: " + email);
        }
        // ... lógica de suscripción
    }
}
```

Si mañana se decide que la validación debe ser más estricta, habrá que localizar **todas** las copias. Basta con olvidar una para que el sistema se comporte de forma incoherente.

### ✅ Con DRY: un tipo que representa el concepto

En Java moderno, la forma más idiomática de centralizar una regla de validación es crear un **tipo propio** con un `record`. La validación se ejecuta en el constructor compacto, de modo que **es imposible crear un `Email` no válido**.

```java
public record Email(String valor) {

    public Email {
        if (valor == null || !valor.contains("@") || !valor.contains(".")) {
            throw new IllegalArgumentException("Correo no válido: " + valor);
        }
        valor = valor.strip().toLowerCase();
    }
}
```

```java
public class RegistroService {
    public void registrar(Email email) {
        // ya sabemos que el correo es válido
    }
}

public class NewsletterService {
    public void suscribir(Email email) {
        // ya sabemos que el correo es válido
    }
}
```

Ahora la regla vive en **un único sitio**. Además, el propio compilador obliga a pasar un `Email` (y no un `String` cualquiera), por lo que nadie puede saltarse la validación por despiste.

> La comprobación de `@` y `.` es intencionadamente simple para el ejemplo. En un proyecto real conviene usar una librería de validación (por ejemplo, Jakarta Bean Validation con `@Email`).

## 6. Técnicas para eliminar la duplicación

| Técnica | Cuándo usarla | Ejemplo |
|---|---|---|
| **Extraer métodos** | La misma secuencia de instrucciones se repite en varios métodos. | `calcularTotalConIva(...)` |
| **Extraer clases o tipos** | Un concepto con reglas propias aparece en muchos sitios. | `record Email`, `record Dinero` |
| **Centralizar constantes y configuración** | Un valor «mágico» aparece en varios ficheros. | `static final int LONGITUD_MAXIMA_NOMBRE = 50;` o un fichero `application.properties` |
| **Usar composición** | Varias clases necesitan el mismo comportamiento. | Inyectar un `ValidadorDni` en quien lo necesite. |
| **Usar herencia (con moderación)** | Hay una relación «es un» real y estable. | Una clase abstracta con el esqueleto común. |
| **Modularizar** | Una parte del sistema es reutilizable por otras. | Un módulo o librería interna compartida. |

> **Composición antes que herencia.** La herencia también elimina duplicación, pero acopla mucho a las subclases con la superclase. En la mayoría de los casos, extraer un componente y reutilizarlo por composición es más flexible.

## 7. Herramientas de Java moderno que ayudan a aplicar DRY

### 7.1. `record` para conceptos con reglas

Los `record` (definitivos desde Java 16) generan automáticamente constructor, métodos de acceso, `equals`, `hashCode` y `toString`. Esto elimina el código repetitivo de las clases de datos y permite concentrar la validación en el constructor compacto, como se ha visto con `Email`.

```java
public record Dinero(java.math.BigDecimal cantidad, java.util.Currency moneda) {

    public Dinero {
        java.util.Objects.requireNonNull(cantidad);
        java.util.Objects.requireNonNull(moneda);
        if (cantidad.signum() < 0) {
            throw new IllegalArgumentException("La cantidad no puede ser negativa");
        }
    }

    public Dinero sumar(Dinero otro) {
        if (!moneda.equals(otro.moneda)) {
            throw new IllegalArgumentException("No se pueden sumar monedas distintas");
        }
        return new Dinero(cantidad.add(otro.cantidad), moneda);
    }
}
```

### 7.2. `enum` con comportamiento

Cuando una regla depende de un conjunto cerrado de opciones, un `enum` con métodos evita repetir `if`/`switch` por todo el código.

```java
public enum TipoIva {
    GENERAL(21), REDUCIDO(10), SUPERREDUCIDO(4);

    private final int porcentaje;

    TipoIva(int porcentaje) {
        this.porcentaje = porcentaje;
    }

    public java.math.BigDecimal aplicar(java.math.BigDecimal base) {
        return base.multiply(java.math.BigDecimal.valueOf(100 + porcentaje))
                   .divide(java.math.BigDecimal.valueOf(100));
    }
}
```

Si cambia un porcentaje, se modifica **solo** aquí.

### 7.3. Funciones de orden superior (lambdas)

Cuando dos métodos son iguales salvo por un pequeño paso, ese paso puede recibirse como parámetro:

```java
import java.util.List;
import java.util.function.Predicate;

public class Informes {

    // En lugar de tener contarActivos(), contarBloqueados(), contarPendientes()...
    public static <T> long contar(List<T> elementos, Predicate<T> condicion) {
        return elementos.stream().filter(condicion).count();
    }
}
```

```java
long activos    = Informes.contar(usuarios, Usuario::activo);
long bloqueados = Informes.contar(usuarios, Usuario::bloqueado);
```

### 7.4. Constructores flexibles (Java 25)

Desde Java 25 (JEP 513, *Flexible Constructor Bodies*) se pueden escribir instrucciones **antes** de llamar a `super(...)` o `this(...)`. Esto permite validar o preparar argumentos en un único constructor y reutilizarlo, sin métodos estáticos auxiliares.

```java
public class CuentaAhorro extends Cuenta {

    public CuentaAhorro(String iban, java.math.BigDecimal saldoInicial) {
        if (saldoInicial.signum() < 0) {                 // antes de super(...)
            throw new IllegalArgumentException("Saldo inicial negativo");
        }
        super(iban, saldoInicial);
    }
}
```

Antes de Java 25, esta validación tenía que hacerse después de `super(...)` (cuando el objeto padre ya estaba construido) o en un método estático auxiliar.

## 8. Dónde aparece DRY en proyectos reales

- **Clases de utilidades** para operaciones comunes: tratamiento de cadenas, formato de fechas, cálculos.
- **Validaciones y reglas de negocio** reutilizadas por varios módulos para que el comportamiento sea coherente.
- **Registro (*logging*), gestión de excepciones y configuración** compartidos, para no repetir la misma preparación ni tratar los errores de forma distinta en cada sitio.
- **Esquemas y contratos:** generar código (por ejemplo, clientes de una API a partir de una especificación OpenAPI) en lugar de escribir a mano lo mismo en dos sitios.

## 9. Violaciones habituales de DRY

- **Copiar y pegar** la misma lógica en varios métodos o clases: cada copia es un punto más que modificar cuando cambian los requisitos.
- **Repetir reglas de negocio en varias capas** (controlador, servicio y repositorio): genera incoherencias y un acoplamiento innecesario.
- **Escribir los mismos valores «a fuego»** (*hard-coded*) en distintos ficheros: actualizar se vuelve propenso a errores y difícil de rastrear.
- **Comentarios que repiten el código:** si el comentario dice exactamente lo que hace la línea siguiente, cuando el código cambie el comentario quedará desfasado.

```java
// ❌ El comentario duplica el código
// Suma 1 al contador
contador++;
```

## 10. DRY y el principio de responsabilidad única (SRP)

DRY trabaja de la mano del **principio de responsabilidad única** (la «S» de SOLID):

- **SRP** garantiza que una clase o módulo tenga **un único motivo para cambiar**.
- **DRY** garantiza que ese motivo de cambio **exista en un único lugar** del código.

Juntos reducen el «efecto dominó» cuando cambian los requisitos y hacen que la refactorización sea más segura.

## 11. Cuándo **no** aplicar DRY

Aplicar DRY a ciegas puede llevar a diseños excesivamente complejos. Conviene **no** unificar código cuando:

- **La duplicación es casual** y es probable que cada parte evolucione de forma diferente. Unificarlas crearía una abstracción que habría que «romper» más adelante.
- **La abstracción añade más complejidad** que el propio código duplicado (parámetros booleanos, `if` internos para distinguir casos, nombres genéricos que no dicen nada).
- **El código es muy pequeño o experimental**, donde la sencillez es más importante que la reutilización.

Una heurística muy extendida es la **regla de tres**: la primera vez se escribe; la segunda, se tolera la repetición; **la tercera vez, se refactoriza**. Esperar a tener tres casos ayuda a ver qué es realmente común.

```java
// ❌ DRY mal aplicado: una única función que intenta servir para todo
public String formatear(Object dato, boolean esFecha, boolean esMoneda, boolean conDecimales) {
    // ... un laberinto de if
}
```

Este tipo de métodos «navaja suiza» viola el principio **KISS** (ver el documento correspondiente) y suele ser peor que dos o tres métodos pequeños y claros.

## 12. Lista de comprobación

- [ ] ¿Hay alguna regla de negocio escrita en más de un sitio?
- [ ] ¿Hay valores «mágicos» repetidos que deberían ser constantes o configuración?
- [ ] ¿Algún concepto del dominio (correo, DNI, importe…) merece su propio `record`?
- [ ] ¿Las condiciones sobre un conjunto cerrado de opciones están en un `enum` o `sealed interface`?
- [ ] ¿La abstracción que voy a crear es más sencilla que la duplicación que elimina?
- [ ] ¿Los fragmentos que voy a unificar cambian por el mismo motivo?

## 13. Referencias

- GeeksforGeeks — *DRY Principle in Software Development*: <https://www.geeksforgeeks.org/software-engineering/dont-repeat-yourselfdry-in-software-development/>
- JEP 395 — *Records* (Java 16): <https://openjdk.org/jeps/395>
- JEP 513 — *Flexible Constructor Bodies* (Java 25): <https://openjdk.org/jeps/513>
- Andy Hunt y Dave Thomas — *The Pragmatic Programmer* (1999; 2.ª edición de 20.º aniversario, 2019).
