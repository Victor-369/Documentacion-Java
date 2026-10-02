# Resumen: Colecciones en Java (Java Collections Framework)

> Basado en la documentación oficial de Oracle, W3Schools y fuentes actualizadas sobre las novedades introducidas hasta Java 21+.

## 1. ¿Qué es una colección?

Una colección (a veces llamada contenedor) es simplemente un objeto que agrupa múltiples elementos en una sola unidad. Las colecciones se usan para almacenar, recuperar, manipular y comunicar datos agregados, como una mano de póker, una carpeta de correo o una agenda telefónica.

## 2. ¿Qué es el "Collections Framework"?

Un framework de colecciones es una arquitectura unificada para representar y manipular colecciones, compuesta por tres piezas:

- **Interfaces**: tipos de datos abstractos que representan colecciones, permitiendo manipularlas sin depender de su implementación concreta.
- **Implementaciones**: las clases concretas (estructuras de datos reutilizables) que implementan esas interfaces.
- **Algoritmos**: métodos que realizan cómputos útiles (buscar, ordenar, etc.) sobre objetos que implementan las interfaces. Son *polimórficos*: el mismo método sirve para distintas implementaciones.

Todo esto forma parte del paquete `java.util`, y sirve para almacenar, buscar, ordenar y organizar datos de forma estandarizada.

### Beneficios principales

Reduce el esfuerzo de programación, aumenta la velocidad y calidad del programa gracias a implementaciones de alto rendimiento, permite interoperabilidad entre APIs no relacionadas, reduce el esfuerzo de aprender nuevas APIs y de diseñarlas, y fomenta la reutilización de software.

> ⚠️ **Nota importante**: el tutorial oficial de Oracle (`docs.oracle.com/javase/tutorial/...`) está escrito para **JDK 8** y no refleja las mejoras introducidas en versiones posteriores. Oracle recomienda consultar [dev.java](https://dev.java/learn/) o las notas de cada JDK para contenido actualizado — por eso este resumen añade lo introducido en Java 9–21+.

---

## 3. Jerarquía de interfaces principales

```
Iterable
  └── Collection
        ├── List           (ArrayList, LinkedList, Vector...)
        ├── Set            (HashSet, LinkedHashSet, TreeSet...)
        ├── Queue          (PriorityQueue, ArrayDeque...)
        └── Deque          (ArrayDeque, LinkedList...)

Map (NO extiende Collection)
        ├── HashMap
        ├── LinkedHashMap
        └── TreeMap        (implementa SortedMap / NavigableMap)
```

La Java Collections Framework provee interfaces como `List`, `Set` y `Map`, y clases como `ArrayList`, `HashSet`, `HashMap`, etc. que implementan esas interfaces. Un buen recurso mental: **las interfaces definen qué se puede hacer, las clases son las herramientas que lo hacen**.

### Tabla comparativa de interfaces

| Interfaz | Clases comunes | Descripción |
|---|---|---|
| `List` | `ArrayList`, `LinkedList` | Ordenada, permite duplicados |
| `Set` | `HashSet`, `TreeSet`, `LinkedHashSet` | Colección de elementos únicos |
| `Map` | `HashMap`, `TreeMap`, `LinkedHashMap` | Pares clave-valor con claves únicas |
| `Queue` / `Deque` | `PriorityQueue`, `ArrayDeque` | Procesamiento en orden (FIFO/LIFO) |

---

## 4. `List`

Colección ordenada (por índice), permite elementos duplicados y `null`.

| Implementación    | Estructura interna          | Acceso `get(i)` | Insertar/eliminar en medio | Uso típico                                                                    |
| ----------------- | --------------------------- | --------------- | -------------------------- | ----------------------------------------------------------------------------- |
| `ArrayList`       | Array dinámico              | O(1)            | O(n)                       | Acceso frecuente por índice                                                   |
| `LinkedList`      | Lista doblemente enlazada   | O(n)            | O(1) si ya tienes el nodo  | Inserciones/eliminaciones frecuentes en extremos (también implementa `Deque`) |
| `Vector` (legado) | Array dinámico sincronizado | O(1)            | O(n)                       | Código heredado; hoy se prefiere `ArrayList` + sincronización externa         |

### Ejemplo: `ArrayList`

```java
List<String> nombres = new ArrayList<>();
nombres.add("Ana");
nombres.add("Luis");
nombres.add(1, "Marta");     // inserta en la posición 1
nombres.get(0);              // "Ana"
nombres.set(0, "Ana María"); // reemplaza el elemento en el índice 0
nombres.remove("Luis");      // elimina por valor
nombres.remove(0);           // elimina por índice
System.out.println(nombres); // [Marta]
```

### Ejemplo: `LinkedList`

`LinkedList` implementa tanto `List` como `Deque`, por lo que puede usarse como lista o como cola/pila:

```java
LinkedList<String> tareas = new LinkedList<>();
tareas.add("Lavar los platos");
tareas.addFirst("Urgente: pagar factura"); // añade al inicio
tareas.addLast("Leer un libro");           // añade al final
tareas.removeFirst();                      // quita el primero
System.out.println(tareas.getFirst());     // "Lavar los platos"
System.out.println(tareas);
```

### Ejemplo: `Vector` (legado)

`Vector` funciona igual que `ArrayList`, pero todos sus métodos están sincronizados (más lento). Hoy se usa poco:

```java
Vector<Integer> numeros = new Vector<>();
numeros.add(10);
numeros.add(20);
numeros.addElement(30);   // método "clásico" heredado de Vector
System.out.println(numeros.firstElement()); // 10
System.out.println(numeros);
```

> **Nota:** `Vector` ya casi no se usa en código nuevo. Es una clase **legado** (en inglés, *legacy*), es decir, se mantiene en Java solo por compatibilidad con programas antiguos. En proyectos actuales se recomienda usar `ArrayList` en su lugar.

---

## 5. `Set`

Colección que **no permite duplicados** (basados en `equals()`/`hashCode()`).

| Implementación | Orden | Rendimiento típico | Notas |
|---|---|---|---|
| `HashSet` | Sin orden garantizado | O(1) promedio | El más rápido; usa tabla hash |
| `LinkedHashSet` | Orden de inserción | O(1) promedio | Igual que HashSet pero mantiene orden |
| `TreeSet` | Orden natural / `Comparator` | O(log n) | Implementa `NavigableSet`/`SortedSet` |

### Ejemplo: `HashSet`

```java
Set<Integer> numeros = new HashSet<>();
numeros.add(5);
numeros.add(3);
numeros.add(5); // ignorado, ya existe
System.out.println(numeros);           // orden no garantizado, p.ej. [3, 5]
System.out.println(numeros.contains(3)); // true
```

### Ejemplo: `LinkedHashSet`

Mantiene el orden en que se insertaron los elementos:

```java
Set<String> ciudades = new LinkedHashSet<>();
ciudades.add("Madrid");
ciudades.add("Bogotá");
ciudades.add("Madrid"); // ignorado
System.out.println(ciudades); // [Madrid, Bogotá]  <- respeta el orden de inserción
```

### Ejemplo: `TreeSet`

Mantiene los elementos ordenados (orden natural o con un `Comparator`):

```java
Set<String> nombres = new TreeSet<>();
nombres.add("Carlos");
nombres.add("Ana");
nombres.add("Beto");
System.out.println(nombres); // [Ana, Beto, Carlos] <- ordenado alfabéticamente

TreeSet<String> desc = new TreeSet<>(Comparator.reverseOrder());
desc.addAll(nombres);
System.out.println(desc);    // [Carlos, Beto, Ana]
```

---

## 6. `Map`

**No extiende `Collection`**, pero forma parte del framework. Almacena pares clave/valor con claves únicas.

| Implementación | Orden | Rendimiento | Notas |
|---|---|---|---|
| `HashMap` | Sin orden garantizado | O(1) promedio | La más usada |
| `LinkedHashMap` | Orden de inserción (o acceso) | O(1) promedio | Útil para caches LRU |
| `TreeMap` | Orden natural de claves | O(log n) | Implementa `NavigableMap`/`SortedMap` |

### Ejemplo: `HashMap`

```java
Map<String, Integer> edades = new HashMap<>();
edades.put("Ana", 30);
edades.put("Luis", 25);
edades.get("Ana");               // 30
edades.getOrDefault("Marta", 0); // 0, no lanza excepción
edades.putIfAbsent("Ana", 99);   // no sobrescribe, ya existe la clave
edades.forEach((k, v) -> System.out.println(k + " -> " + v)); // orden no garantizado
```

### Ejemplo: `LinkedHashMap`

Mantiene el orden de inserción (útil, por ejemplo, para implementar un caché LRU):

```java
Map<String, Integer> stock = new LinkedHashMap<>();
stock.put("Manzanas", 10);
stock.put("Peras", 5);
stock.put("Uvas", 20);
System.out.println(stock); // {Manzanas=10, Peras=5, Uvas=20} <- respeta orden de inserción
```

### Ejemplo: `TreeMap`

Mantiene las claves ordenadas y permite operaciones de navegación (`firstKey`, `lastKey`, etc.):

```java
Map<String, Integer> notas = new TreeMap<>();
notas.put("Carlos", 8);
notas.put("Ana", 9);
notas.put("Beto", 7);
System.out.println(notas);              // {Ana=9, Beto=7, Carlos=8} <- ordenado por clave
System.out.println(((TreeMap<String, Integer>) notas).firstKey()); // "Ana"
```

---

## 7. `Queue` y `Deque`

- **`Queue`**: procesamiento FIFO (primero en entrar, primero en salir). Implementación típica: `LinkedList`, `PriorityQueue` (orden por prioridad).
- **`Deque`** (*double-ended queue*): permite insertar/eliminar por ambos extremos; puede usarse como cola o como pila. Implementación típica: `ArrayDeque` (recomendada sobre `Stack`, que es legado y sincronizado).

### Ejemplo: `LinkedList` como `Queue` (FIFO)

```java
Queue<String> cola = new LinkedList<>();
cola.offer("Cliente 1"); // añade al final
cola.offer("Cliente 2");
System.out.println(cola.peek()); // "Cliente 1", sin eliminarlo
System.out.println(cola.poll()); // "Cliente 1", lo elimina y lo devuelve
System.out.println(cola);        // [Cliente 2]
```

### Ejemplo: `PriorityQueue`

Ordena automáticamente los elementos según su orden natural o un `Comparator`, no por orden de llegada:

```java
Queue<Integer> prioridad = new PriorityQueue<>();
prioridad.offer(5);
prioridad.offer(1);
prioridad.offer(3);
System.out.println(prioridad.poll()); // 1 (el menor primero)
System.out.println(prioridad.poll()); // 3

// Con Comparator para orden descendente
Queue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.addAll(List.of(5, 1, 3));
System.out.println(maxHeap.poll()); // 5
```

### Ejemplo: `ArrayDeque` como pila (LIFO)

```java
Deque<Integer> pila = new ArrayDeque<>();
pila.push(1);
pila.push(2);
pila.pop(); // 2 (LIFO)
```

### Ejemplo: `ArrayDeque` como cola (FIFO)

```java
Deque<String> colaDoble = new ArrayDeque<>();
colaDoble.addLast("primero");
colaDoble.addLast("segundo");
System.out.println(colaDoble.pollFirst()); // "primero"
```

### Ejemplo: `Stack` (legado)

```java
Stack<Integer> pilaAntigua = new Stack<>();
pilaAntigua.push(1);
pilaAntigua.push(2);
System.out.println(pilaAntigua.pop());  // 2 (LIFO)
System.out.println(pilaAntigua.peek()); // 1, sin eliminarlo
```

> **Nota:** al igual que `Vector`, `Stack` es una clase **legado** (*legacy*): es antigua, tiene métodos sincronizados que la hacen más lenta y hoy en día ya no se recomienda usarla. En su lugar, se prefiere `ArrayDeque` para implementar pilas.

---

## 8. Colecciones inmutables / "factory methods" (desde Java 9)

Desde **Java 9** existen métodos estáticos de fábrica para crear colecciones **inmutables** de forma concisa (no requieren `Collections.unmodifiableList(...)`):

```java
List<String> lista = List.of("a", "b", "c");
Set<String> set     = Set.of("x", "y", "z");
Map<String, Integer> mapa = Map.of("a", 1, "b", 2);
```

- Lanzan `UnsupportedOperationException` si intentas modificarlas.
- No permiten elementos `null`.
- `Map.of` admite hasta 10 pares; para más, usar `Map.ofEntries(...)`.

---

## 9. Novedad clave: Sequenced Collections (Java 21 — JEP 431)

Desde **Java 21**, el framework incorpora tres nuevas interfaces que resuelven una inconsistencia histórica: antes no existía una forma uniforme de acceder al primer/último elemento ni de iterar en orden inverso en toda la jerarquía. Se introducen nuevas interfaces para representar colecciones con un orden de encuentro (*encounter order*) bien definido: cada colección tiene un primer elemento, un segundo, y así sucesivamente hasta el último, además de operaciones uniformes para acceder a sus extremos y procesar los elementos en orden inverso.

Las tres nuevas interfaces son `SequencedCollection`, `SequencedSet` y `SequencedMap`.

### `SequencedCollection`

Define un nuevo método `reversed()` y promueve desde `Deque` los métodos `addFirst`, `addLast`, `getFirst`, `getLast`, `removeFirst` y `removeLast`:

```java
interface SequencedCollection<E> extends Collection<E> {
    SequencedCollection<E> reversed();
    void addFirst(E e);
    void addLast(E e);
    E getFirst();
    E getLast();
    E removeFirst();
    E removeLast();
}
```

`List` y `Deque` ahora heredan de `SequencedCollection`. Ejemplo práctico — antes de Java 21:

```java
List<String> lista = new ArrayList<>(List.of("Sam", "Alex", "Jim"));
String primero = lista.iterator().next();
String ultimo  = lista.get(lista.size() - 1);
```

Desde Java 21, con `getFirst()`/`getLast()`:

```java
String primero = lista.getFirst();
String ultimo  = lista.getLast();
List<String> invertida = lista.reversed(); // vista en orden inverso
```

### `SequencedSet`

Es un Set que también es una SequencedCollection sin elementos duplicados; hereda los métodos de SequencedCollection e incluye métodos especializados para añadir elementos manteniendo el orden. Implementado por `LinkedHashSet` y `TreeSet`.

```java
LinkedHashSet<String> set = new LinkedHashSet<>();
set.add("Sam");
set.add("Alex");
set.addFirst("Primero");
set.addLast("Ultimo");
```

### `SequencedMap`

Implementado por `LinkedHashMap` y `TreeMap`. Añade métodos como `putFirst()`, `putLast()`, `firstEntry()`, `lastEntry()`, `sequencedKeySet()`, `sequencedValues()`, `reversed()`.

### Envoltorios inmutables

La clase `Collections` incorpora nuevos métodos para crear envoltorios no modificables de los tres nuevos tipos: `Collections.unmodifiableSequencedCollection(...)`, `Collections.unmodifiableSequencedSet(...)` y `Collections.unmodifiableSequencedMap(...)`.

---

## 10. La clase utilitaria `Collections`

Contiene algoritmos estáticos (polimórficos) que operan sobre colecciones:

```java
Collections.sort(lista);
Collections.reverse(lista);
Collections.shuffle(lista);
Collections.max(lista);
Collections.min(lista);
Collections.unmodifiableList(lista);
Collections.synchronizedList(lista); // envoltorio "hilo-seguro"
```

---

## 11. Ordenar colecciones: `Comparable` vs `Comparator`

- **`Comparable<T>`**: se implementa en la propia clase (define el "orden natural") mediante `compareTo()`.
- **`Comparator<T>`**: objeto externo que define un criterio de orden alternativo mediante `compare()`.

```java
// Comparator con expresión lambda (desde Java 8)
List<String> nombres = new ArrayList<>(List.of("Carlos", "Ana", "Beto"));
nombres.sort(Comparator.naturalOrder());
nombres.sort(Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder()));
```

---

## 12. Iterar sobre colecciones

```java
// for-each (recomendado en la mayoría de casos)
for (String nombre : nombres) { System.out.println(nombre); }

// Iterator explícito (permite eliminar de forma segura durante la iteración)
Iterator<String> it = nombres.iterator();
while (it.hasNext()) {
    String n = it.next();
    if (n.equals("Ana")) it.remove();
}

// forEach + lambda
nombres.forEach(System.out::println);
```

---

## 13. Genéricos

Las colecciones modernas siempre se usan con **genéricos** (`List<String>`, no `List` "crudo"), lo que aporta seguridad de tipos en tiempo de compilación y evita *casts* manuales. Desde Java 10 puede combinarse con inferencia de tipos usando `var`:

```java
var mapa = new HashMap<String, List<Integer>>(); // el tipo se infiere
```

---

## 14. Colecciones y el Stream API (Java 8+)

Aunque no es parte del *Collections Framework* en sí, es fundamental para trabajar con colecciones hoy en día:

```java
List<String> mayores = nombres.stream()
        .filter(n -> n.length() > 3)
        .map(String::toUpperCase)
        .sorted()
        .collect(Collectors.toList());

// Desde Java 16: toList() como atajo (inmutable)
List<String> resultado = nombres.stream().map(String::toUpperCase).toList();
```

---

## 15. Colecciones concurrentes (paquete `java.util.concurrent`)

Para entornos multihilo, evitar las colecciones "sincronizadas" clásicas (`Vector`, `Hashtable`, `Collections.synchronizedX`) y preferir:

| Clase | Equivalente a | Uso |
|---|---|---|
| `ConcurrentHashMap` | `HashMap` | Mapa con alto rendimiento concurrente |
| `CopyOnWriteArrayList` | `ArrayList` | Listas con muchas lecturas, pocas escrituras |
| `ConcurrentLinkedQueue` | `Queue` | Colas sin bloqueo |
| `BlockingQueue` (y sus implementaciones) | — | Productor-consumidor |

### Ejemplo: `Hashtable` (legado)

```java
Hashtable<String, Integer> tabla = new Hashtable<>();
tabla.put("Ana", 30);
tabla.put("Luis", 25);
System.out.println(tabla.get("Ana")); // 30
```

> **Nota:** `Hashtable` es otra clase **legado** (*legacy*), tan antigua como `Vector`. También sincroniza todos sus métodos, lo que la hace lenta y, además, no permite claves ni valores `null`. En código moderno se recomienda `HashMap` (uso normal) o `ConcurrentHashMap` (uso concurrente).

### Ejemplo: `ConcurrentHashMap`

```java
Map<String, Integer> contador = new ConcurrentHashMap<>();
contador.put("visitas", 0);
contador.compute("visitas", (k, v) -> v + 1); // operación atómica, segura entre hilos
System.out.println(contador.get("visitas")); // 1
```

### Ejemplo: `CopyOnWriteArrayList`

Ideal cuando hay muchas lecturas y pocas escrituras concurrentes (cada escritura copia el array interno):

```java
List<String> suscriptores = new CopyOnWriteArrayList<>();
suscriptores.add("usuario1");
suscriptores.add("usuario2");
for (String s : suscriptores) {
    // seguro iterar aunque otro hilo modifique la lista al mismo tiempo
    System.out.println(s);
}
```

### Ejemplo: `ConcurrentLinkedQueue`

Cola no bloqueante basada en algoritmos *lock-free*:

```java
Queue<String> mensajes = new ConcurrentLinkedQueue<>();
mensajes.offer("mensaje 1");
mensajes.offer("mensaje 2");
System.out.println(mensajes.poll()); // "mensaje 1"
```

### Ejemplo: `BlockingQueue` (con `LinkedBlockingQueue`)

Usada en escenarios productor-consumidor: `take()` y `put()` bloquean el hilo si la cola está vacía o llena:

```java
BlockingQueue<String> buffer = new LinkedBlockingQueue<>(10); // capacidad máxima 10

// Hilo productor
new Thread(() -> {
    try {
        buffer.put("dato"); // espera si el buffer está lleno
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
}).start();

// Hilo consumidor
new Thread(() -> {
    try {
        String dato = buffer.take(); // espera si el buffer está vacío
        System.out.println(dato);
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
}).start();
```

---

## 16. Resumen rápido: ¿qué estructura elegir?

- Usa clases de `List` cuando te importa el orden, puede haber duplicados y quieres acceder a elementos por índice.
- Usa clases de `Set` cuando necesitas almacenar solo valores únicos.
- Usa clases de `Map` cuando necesitas almacenar pares de clave y valor, como un nombre y su número de teléfono.
- Usa `Deque`/`Queue` para pilas, colas o algoritmos tipo BFS/DFS.
- Desde Java 21, si necesitas primer/último elemento o vista invertida de forma uniforme, apóyate en las interfaces `Sequenced*`.
- Para inmutabilidad rápida, usa `List.of()`, `Set.of()`, `Map.of()` (Java 9+).
- Para concurrencia, usa el paquete `java.util.concurrent` en lugar de sincronizar manualmente.

---

## Fuentes

- Oracle Java Tutorials — *Introduction to Collections* (contenido base para JDK 8): https://docs.oracle.com/javase/tutorial/collections/intro/index.html
- W3Schools — *Java Collections Framework*: https://www.w3schools.com/java/java_collections.asp
- OpenJDK — *JEP 431: Sequenced Collections*: https://openjdk.org/jeps/431
- Oracle Docs — *Creating Sequenced Collections, Sets, and Maps*: https://docs.oracle.com/en/java/javase/25/core/creating-sequenced-collections-sets-and-maps.html
