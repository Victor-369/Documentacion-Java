# Arquitectura Criptográfica de Java (JCA) — Guía de referencia (Java 21+)

> **Fuente de referencia:** *Java Cryptography Architecture (JCA) Reference Guide* (Oracle).
> **Destinatario:** programador Java junior.
> **Versión objetivo:** Java 21 (LTS) y posteriores.

---

## 1. Introducción

La **Java Cryptography Architecture (JCA)** es la parte de la plataforma Java que ofrece una arquitectura de **proveedores** y un conjunto de **APIs** para:

- Resúmenes de mensajes (*hashes*).
- Firmas digitales.
- Certificados y su validación.
- Cifrado (simétrico y asimétrico; de bloque y de flujo).
- Generación y gestión de claves.
- Generación de números aleatorios seguros.

Gracias a ella puedes añadir seguridad a una aplicación **sin implementar tú los algoritmos**.

### 1.1. Terminología: JCA y JCE

Antes de JDK 1.4, la **JCE** (*Java Cryptography Extension*) era un producto aparte. Hoy está integrada en el JDK y usa la misma arquitectura, así que se considera parte de la JCA. En la práctica:

- `java.security.*` → clases clásicas de la JCA (hash, firmas, claves, keystore…).
- `javax.crypto.*` → clases de la antigua JCE (cifrado, MAC, acuerdo de claves…).

El JCA dentro del JDK tiene dos componentes:

1. **El marco (*framework*)**: define y soporta los servicios criptográficos (`java.security`, `javax.crypto`, `javax.crypto.spec`, `javax.crypto.interfaces`).
2. **Los proveedores** (por ejemplo `SUN`, `SunRsaSign`, `SunJCE`), que contienen las implementaciones reales.

### 1.2. Aviso importante de Oracle

La JCA facilita incorporar seguridad, pero **no enseña teoría criptográfica**. No cubre los puntos fuertes y débiles de cada algoritmo ni el diseño de protocolos. La criptografía es un tema avanzado:

> **Debes entender qué haces y por qué. No copies código al azar esperando que resuelva tu caso.** Muchas aplicaciones han tenido graves fallos de seguridad o rendimiento por elegir una herramienta o algoritmo equivocado.

### 1.3. ¿A quién va dirigida la guía original?

A programadores con experiencia que quieran **crear sus propios proveedores**. Si solo vas a **usar** los algoritmos existentes, no necesitas leer todo el detalle de la implementación de proveedores, pero sí entender los conceptos de este documento.

---

## 2. Principios de diseño

La JCA se basa en dos parejas de principios complementarios:

| Principio | Significado |
|---|---|
| **Independencia de implementación** | La aplicación pide un servicio («un SHA-256») sin saber qué proveedor lo implementa. |
| **Interoperabilidad de implementaciones** | Lo generado por un proveedor (claves, firmas) puede usarse o verificarse con otro. |
| **Independencia de algoritmo** | Se definen tipos de servicios (*engine classes*) comunes a muchos algoritmos. |
| **Extensibilidad de algoritmos** | Se pueden añadir algoritmos nuevos instalando proveedores adicionales. |

Si por alguna razón necesitas un proveedor concreto, la API también permite pedirlo por nombre.

---

## 3. Arquitectura de proveedores

### 3.1. ¿Qué es un proveedor?

Un **proveedor criptográfico** (CSP, *Cryptographic Service Provider*) es un paquete (o conjunto de paquetes) que implementa uno o más servicios criptográficos. Su clase base es `java.security.Provider`.

Cada JDK trae varios proveedores preinstalados y configurados. Entre ellos están `SUN`, `SunRsaSign`, `SunEC`, `SunJSSE` y `SunJCE`. Se reparten así sobre todo por razones históricas.

### 3.2. Orden de preferencia

Cuando pides un algoritmo **sin indicar proveedor**, la JCA recorre los proveedores instalados **por orden de preferencia** y devuelve la implementación del primero que lo ofrezca.

```java
// Sin proveedor: se usa el primero que lo implemente
MessageDigest md1 = MessageDigest.getInstance("SHA-256");

// Con proveedor concreto (no recomendado en general)
MessageDigest md2 = MessageDigest.getInstance("SHA-256", "SUN");
```

Esquema del orden de búsqueda:

```text
Petición: "MD5"
  Proveedor 1 (preferencia 1) -> ¿lo tiene? Sí -> se devuelve éste.
  Proveedor 2 (preferencia 2) -> (no se consulta)

Petición: "MD5withRSA"
  Proveedor 1 -> No
  Proveedor 2 -> Sí -> se devuelve éste.

Petición: "SHA1withRSA"
  Ningún proveedor lo ofrece -> NoSuchAlgorithmException
```

### 3.3. ¿Cuándo pedir un proveedor concreto?

Normalmente **no deberías hacerlo**. Las aplicaciones de propósito general no deben fijar proveedor, porque:

- Quedan atadas a un proveedor que puede no existir en otra implementación de Java.
- Pierden la opción de usar proveedores optimizados (aceleración hardware, PKCS#11, implementaciones nativas del SO) con mayor preferencia.

Sí tiene sentido, por ejemplo, cuando una normativa exige una implementación certificada.

Si el proveedor pedido no está instalado, se lanza `NoSuchProviderException`.

### 3.4. Nombres de algoritmo

- No distinguen mayúsculas de minúsculas: `"SHA-256"` y `"sha-256"` son equivalentes.
- Usa siempre los **nombres estándar**. Los alias (por ejemplo `"SHA1"` en lugar de `"SHA-1"`) pueden variar entre proveedores.

### 3.5. Cómo se implementa por dentro: engine + SPI

Para cada servicio existen dos clases:

- **Clase *engine*** (la que usas): `MessageDigest`, `Cipher`, `Signature`…
- **Clase SPI** (la que implementa el proveedor): `MessageDigestSpi`, `CipherSpi`, `SignatureSpi`… (siempre el mismo nombre más `Spi`).

```text
Tu código ──► Cipher (engine, métodos final) ──► CipherSpi (implementación del proveedor)
```

Cuando llamas a `Cipher.getInstance("AES")`, el marco busca en los proveedores, crea el objeto SPI y lo encapsula dentro de un objeto `Cipher`. Al llamar a `c.init(...)`, la petición se delega en `engineInit(...)` del SPI.

Un método estático que devuelve una instancia de una clase (`getInstance`) es un **método de factoría** (*factory method*).

---

## 4. Instalación y registro de proveedores

> Esto es relevante solo si usas proveedores de terceros (por ejemplo, Bouncy Castle).

### 4.1. Instalación

Añade el JAR del proveedor al *classpath* o *module path*. (El antiguo directorio `lib/ext` ya no existe desde Java 9.) Algunos proveedores de cifrado exigen que el JAR esté firmado.

### 4.2. Registro estático

Edita `<JAVA_HOME>/conf/security/java.security` y añade una línea por proveedor:

```properties
security.provider.<n>=<claseMaestra>
```

Donde `n` es la preferencia (1 = la más alta). Por ejemplo:

```properties
security.provider.8=com.empresax.provider.ProviderX
```

### 4.3. Registro dinámico

Desde código, con la clase `Security`:

```java
import java.security.Security;

Security.addProvider(new MiProveedor());          // al final de la lista
Security.insertProviderAt(new MiProveedor(), 1);  // en la posición 1
Security.removeProvider("MiProveedor");           // lo elimina
```

Notas:

- No es persistente: solo vale para esa ejecución de la JVM.
- Hoy en día solo es posible si el código es de confianza, ya que el control de permisos mediante *Security Manager* está obsoleto (véase el fichero `01`).
- Para cambiar la preferencia de un proveedor, hay que quitarlo y volver a insertarlo.

### 4.4. Consultar los proveedores instalados

```java
import java.security.Provider;
import java.security.Security;

public class ListaProveedores {
    public static void main(String[] args) {
        for (Provider p : Security.getProviders()) {
            System.out.println(p.getName() + " v" + p.getVersionStr() + " - " + p.getInfo());
        }
    }
}
```

> Desde Java 9, `getVersion()` (que devuelve `double`) está en desuso; usa `getVersionStr()`.

---

## 5. La clase `Security` y sus propiedades

`Security` solo tiene métodos estáticos y gestiona los proveedores y las propiedades de seguridad globales.

```java
String valor = Security.getProperty("keystore.type");   // "pkcs12"
Security.setProperty("clave", "valor");                 // solo en tiempo de ejecución
```

Propiedades relevantes:

| Propiedad | Uso |
|---|---|
| `security.provider.N` | Lista ordenada de proveedores |
| `jdk.security.provider.preferred` | Proveedor preferido para un algoritmo |
| `securerandom.strongAlgorithms` | Algoritmos `SecureRandom` «fuertes» |
| `keystore.type` | Tipo de keystore por defecto (`pkcs12`) |
| `jdk.tls.disabledAlgorithms` | Algoritmos TLS deshabilitados |

---

## 6. Clases *engine* del JDK

| Clase | Función |
|---|---|
| `MessageDigest` | Calcula el hash de unos datos |
| `Signature` | Firma datos y verifica firmas |
| `Cipher` | Cifra y descifra |
| `Mac` | Calcula un código de autenticación de mensaje |
| `KeyGenerator` | Genera claves secretas (simétricas) |
| `KeyPairGenerator` | Genera pares de claves pública/privada |
| `KeyFactory` | Convierte claves asimétricas ↔ especificaciones |
| `SecretKeyFactory` | Convierte claves secretas ↔ especificaciones |
| `KeyAgreement` | Acuerdo de claves (por ejemplo Diffie-Hellman, ECDH) |
| `KeyStore` | Almacén de claves y certificados |
| `SecureRandom` | Números aleatorios criptográficamente seguros |
| `AlgorithmParameters` | Gestiona parámetros de algoritmos |
| `AlgorithmParameterGenerator` | Genera parámetros para un algoritmo |
| `CertificateFactory` | Crea certificados y CRL |
| `CertPathBuilder` / `CertPathValidator` / `CertStore` | Construcción, validación y búsqueda de cadenas de certificados |
| `KEM` | Mecanismo de encapsulación de claves (nuevo en Java 21) |

### 6.1. Generador frente a factoría

Es una confusión muy común entre novatos:

- **Generador** (*generator*): crea objetos **nuevos** con contenido aleatorio. Ej.: `KeyGenerator`, `KeyPairGenerator`.
- **Factoría** (*factory*): **convierte** datos que ya existen en otro tipo de objeto. Ej.: `KeyFactory`, `CertificateFactory`.

---

## 7. `SecureRandom`: números aleatorios seguros

Genera números pseudoaleatorios criptográficamente fuertes. Para cualquier cosa sensible (claves, IV, sales, tokens) usa **siempre** `SecureRandom` y nunca `java.util.Random`.

```java
import java.security.SecureRandom;

SecureRandom sr = new SecureRandom();          // por defecto, suficiente en la mayoría de los casos
byte[] bytes = new byte[16];
sr.nextBytes(bytes);                           // rellena el array

SecureRandom fuerte = SecureRandom.getInstanceStrong(); // puede bloquearse en algunos sistemas
```

Notas:

- `setSeed(...)` **complementa** la semilla existente, no la sustituye. En general no hace falta llamarlo.
- `getInstanceStrong()` devuelve la implementación más fuerte de la plataforma, según `securerandom.strongAlgorithms`. En Linux puede bloquearse esperando entropía; úsalo para claves de larga duración, no en bucles.
- Las implementaciones DRBG siguen la norma NIST SP 800-90Ar1.

---

## 8. `MessageDigest`: hashes

Un *digest* convierte datos de cualquier tamaño en una salida de tamaño fijo. Propiedades deseables:

- Es computacionalmente inviable encontrar dos mensajes con el mismo hash.
- El hash no revela información del mensaje original.
- Cambiar un solo bit cambia el resultado por completo.

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.HexFormat;

MessageDigest md = MessageDigest.getInstance("SHA-256");  // ya viene inicializado
md.update("Parte 1 ".getBytes(StandardCharsets.UTF_8));   // se puede alimentar por trozos
md.update("Parte 2".getBytes(StandardCharsets.UTF_8));
byte[] hash = md.digest();                                // calcula y reinicia el objeto

System.out.println(HexFormat.of().formatHex(hash));
```

Tamaños de salida: MD5 = 16 bytes, SHA-1 = 20 bytes, SHA-256 = 32 bytes.

> **Importante:** MD5 y SHA-1 están rotos para usos de seguridad. Usa **SHA-256** o superior (o SHA-3).
> **No uses un hash simple para contraseñas.** Usa PBKDF2, bcrypt, scrypt o Argon2:

```java
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import java.security.SecureRandom;

char[] password = "miContraseña".toCharArray();
byte[] salt = new byte[16];
new SecureRandom().nextBytes(salt);

PBEKeySpec spec = new PBEKeySpec(password, salt, 600_000, 256); // iteraciones, bits
byte[] hash = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256")
                              .generateSecret(spec).getEncoded();
spec.clearPassword();
```

---

## 9. `Signature`: firmas digitales

Una firma digital se genera con la **clave privada** y se verifica con la **clave pública**. Garantiza **autenticidad** e **integridad**.

### 9.1. Estados de un objeto `Signature`

Es un objeto «modal»: siempre está en un estado y solo puede hacer un tipo de operación.

| Estado | Cómo se llega |
|---|---|
| `UNINITIALIZED` | Recién creado |
| `SIGN` | Tras `initSign(PrivateKey)` |
| `VERIFY` | Tras `initVerify(PublicKey)` |

### 9.2. Ejemplo completo: firmar y verificar

```java
import java.nio.charset.StandardCharsets;
import java.security.*;

public class EjemploFirma {
    public static void main(String[] args) throws Exception {
        // 1. Generar un par de claves
        KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
        kpg.initialize(3072);
        KeyPair par = kpg.generateKeyPair();

        byte[] datos = "Documento importante".getBytes(StandardCharsets.UTF_8);

        // 2. Firmar con la clave privada
        Signature firmador = Signature.getInstance("SHA256withRSA");
        firmador.initSign(par.getPrivate());
        firmador.update(datos);
        byte[] firma = firmador.sign();

        // 3. Verificar con la clave pública
        Signature verificador = Signature.getInstance("SHA256withRSA");
        verificador.initVerify(par.getPublic());
        verificador.update(datos);
        boolean valida = verificador.verify(firma);

        System.out.println("¿Firma válida? " + valida);
    }
}
```

Notas:

- Tras llamar a `sign()` o `verify()`, el objeto se reinicia al estado posterior a `initSign`/`initVerify`.
- Para firmas modernas, valora **Ed25519** (`Signature.getInstance("Ed25519")`, disponible desde Java 15).

---

## 10. `Cipher`: cifrado y descifrado

### 10.1. Simétrico frente a asimétrico

| | Simétrico | Asimétrico |
|---|---|---|
| Claves | Una sola, secreta | Par público/privado |
| Velocidad | Rápido | Lento |
| Ejemplos | AES | RSA, curvas elípticas |
| Uso típico | Cifrar datos grandes | Intercambiar claves, firmar |

En la práctica, se usa criptografía asimétrica para intercambiar una **clave simétrica pequeña**, y esa clave cifra los datos.

### 10.2. Cifrado de bloque y de flujo

- **Bloque:** procesa bloques completos; si faltan datos se rellena (*padding*, por ejemplo `PKCS5Padding`).
- **Flujo:** procesa byte a byte (o bit a bit), sin relleno.

### 10.3. Modos de operación

Con un cifrado de bloque simple, dos bloques iguales de texto en claro producen dos bloques iguales cifrados, lo que filtra información. Los modos de operación evitan esto:

| Modo | Comentario |
|---|---|
| **ECB** | Sin retroalimentación. **No usar nunca** con varios bloques. |
| **CBC** | Encadena bloques. Requiere IV aleatorio. No autentica por sí mismo. |
| **CFB / OFB** | Convierten un cifrado de bloque en uno de flujo. |
| **CTR** | Modo contador. |
| **GCM** | **Recomendado.** Cifra y autentica a la vez (AEAD). |

El **vector de inicialización (IV)** debe ser aleatorio e impredecible, pero **no secreto**.

### 10.4. Transformaciones

El nombre del algoritmo al crear un `Cipher` es una «transformación»:

```text
"algoritmo/modo/relleno"    p. ej. "AES/GCM/NoPadding"
"algoritmo"                 p. ej. "AES"   (¡se usan valores por defecto!)
```

> **Cuidado:** si solo indicas `"AES"`, el proveedor SunJCE usa `AES/ECB/PKCS5Padding`, que es inseguro. **Especifica siempre algoritmo, modo y relleno completos.**

### 10.5. Ejemplo recomendado: AES-GCM

```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.GCMParameterSpec;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.security.SecureRandom;

public class EjemploAesGcm {
    private static final int TAM_IV = 12;        // 96 bits, recomendado para GCM
    private static final int TAM_TAG_BITS = 128;

    public static void main(String[] args) throws Exception {
        KeyGenerator kg = KeyGenerator.getInstance("AES");
        kg.init(256);
        SecretKey clave = kg.generateKey();

        byte[] cifrado = cifrar(clave, "Mensaje secreto".getBytes(StandardCharsets.UTF_8));
        byte[] claro = descifrar(clave, cifrado);

        System.out.println(new String(claro, StandardCharsets.UTF_8));
    }

    static byte[] cifrar(SecretKey clave, byte[] claro) throws Exception {
        byte[] iv = new byte[TAM_IV];
        new SecureRandom().nextBytes(iv);                      // IV nuevo en cada cifrado

        Cipher c = Cipher.getInstance("AES/GCM/NoPadding");
        c.init(Cipher.ENCRYPT_MODE, clave, new GCMParameterSpec(TAM_TAG_BITS, iv));
        byte[] cifrado = c.doFinal(claro);

        // Guardamos IV + texto cifrado juntos
        return ByteBuffer.allocate(iv.length + cifrado.length).put(iv).put(cifrado).array();
    }

    static byte[] descifrar(SecretKey clave, byte[] mensaje) throws Exception {
        ByteBuffer buf = ByteBuffer.wrap(mensaje);
        byte[] iv = new byte[TAM_IV];
        buf.get(iv);
        byte[] cifrado = new byte[buf.remaining()];
        buf.get(cifrado);

        Cipher c = Cipher.getInstance("AES/GCM/NoPadding");
        c.init(Cipher.DECRYPT_MODE, clave, new GCMParameterSpec(TAM_TAG_BITS, iv));
        return c.doFinal(cifrado); // lanza AEADBadTagException si fue manipulado
    }
}
```

Reglas de oro con GCM:

- **Nunca reutilices la combinación clave + IV.** Genera un IV nuevo para cada cifrado.
- Los **datos asociados** (AAD) se añaden con `c.updateAAD(...)` **antes** de `update`/`doFinal`. Se autentican, pero no se cifran.
- Si el mensaje ha sido modificado, el descifrado falla con `AEADBadTagException`.

### 10.6. Modos de operación del `Cipher`

| Constante | Significado |
|---|---|
| `ENCRYPT_MODE` | Cifrar |
| `DECRYPT_MODE` | Descifrar |
| `WRAP_MODE` | «Envolver» una clave para transportarla |
| `UNWRAP_MODE` | Recuperar una clave envuelta |

Al descifrar hay que usar **los mismos parámetros** (IV, sal, iteraciones…) que al cifrar. Puedes obtenerlos con `cipher.getParameters()` o `cipher.getIV()`.

Inicializar un `Cipher` de nuevo equivale a crear uno nuevo: pierde todo su estado anterior.

### 10.7. Una sola llamada o varias

- `doFinal(...)`: cifra o descifra todo de una vez.
- `update(...)` repetido y `doFinal()` al final: para datos grandes o de tamaño desconocido.
- `getOutputSize(int)`: indica el tamaño necesario del buffer de salida.

### 10.8. Clases auxiliares basadas en `Cipher`

- **`CipherInputStream` / `CipherOutputStream`**: flujos que cifran o descifran mientras lees/escribes.
  - En `CipherOutputStream`, **`close()` es imprescindible**, porque llama a `doFinal()` y escribe los últimos bytes (relleno incluido). `flush()` no lo hace.
  - Usa *try-with-resources*:

```java
try (var in = new java.io.FileInputStream("claro.txt");
     var out = new javax.crypto.CipherOutputStream(
                   new java.io.FileOutputStream("cifrado.bin"), cipher)) {
    in.transferTo(out);
}
```

- **`SealedObject`**: cifra un objeto `Serializable` y guarda junto a él los parámetros necesarios para descifrarlo. Cuidado con la deserialización de datos no confiables.

---

## 11. `Mac`: códigos de autenticación de mensaje

Un MAC es como un hash, pero **incluye una clave secreta**. Solo quien la conozca puede generar o verificar el código. Se usa para comprobar integridad y autenticidad entre dos partes que comparten clave. Si se basa en una función hash, se llama **HMAC**.

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

byte[] claveBytes = ...; // al menos 32 bytes aleatorios
SecretKeySpec clave = new SecretKeySpec(claveBytes, "HmacSHA256");

Mac mac = Mac.getInstance("HmacSHA256");
mac.init(clave);
byte[] etiqueta = mac.doFinal("Mensaje".getBytes(StandardCharsets.UTF_8));
System.out.println(HexFormat.of().formatHex(etiqueta));
```

Al comparar dos MAC usa una comparación en tiempo constante: `MessageDigest.isEqual(a, b)`.

---

## 12. Claves

### 12.1. La interfaz `Key`

Es la interfaz raíz de todas las claves **opacas** (no se accede directamente a su material interno). Tiene tres métodos:

| Método | Devuelve |
|---|---|
| `getAlgorithm()` | Nombre del algoritmo (AES, RSA…) |
| `getEncoded()` | Clave codificada en un formato estándar (X.509, PKCS#8…) |
| `getFormat()` | Nombre del formato de codificación |

Subinterfaces: `PublicKey`, `PrivateKey` (sin métodos, solo para tipado seguro) y `SecretKey` (clave simétrica).

### 12.2. `KeyPair`

Contenedor de un par de claves: `getPrivate()` y `getPublic()`.

### 12.3. Representación opaca frente a transparente

- **Opaca** (`Key`): no ves los componentes internos.
- **Transparente** (`KeySpec`): puedes acceder a cada valor (por ejemplo, módulo y exponente en RSA).

| Clase / interfaz | Descripción |
|---|---|
| `KeySpec` | Interfaz marcador de especificaciones de clave |
| `EncodedKeySpec` | Clave en formato codificado (abstracta) |
| `PKCS8EncodedKeySpec` | **Clave privada** codificada en PKCS#8 |
| `X509EncodedKeySpec` | **Clave pública** codificada en X.509 |
| `SecretKeySpec` | Clave secreta a partir de bytes (implementa `SecretKey`) |

### 12.4. `KeyFactory` y `SecretKeyFactory`

Convierten entre claves (`Key`) y especificaciones (`KeySpec`), en ambos sentidos.

```java
// Reconstruir una clave pública a partir de sus bytes codificados
byte[] bytesPublica = parClaves.getPublic().getEncoded();

KeyFactory kf = KeyFactory.getInstance("RSA");
PublicKey publica = kf.generatePublic(new X509EncodedKeySpec(bytesPublica));
```

Alternativa independiente del proveedor para claves secretas:

```java
SecretKey clave = new SecretKeySpec(bytesClave, "AES");
```

### 12.5. `KeyPairGenerator` (asimétrico)

```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(3072);                // tamaño de clave
KeyPair par = kpg.generateKeyPair(); // cada llamada produce un par distinto
```

Tamaños orientativos: RSA ≥ 2048 (mejor 3072); curvas elípticas P-256 o superiores; o Ed25519 / X25519.

### 12.6. `KeyGenerator` (simétrico)

```java
KeyGenerator kg = KeyGenerator.getInstance("AES");
kg.init(256);
SecretKey clave = kg.generateKey();
```

### 12.7. `KeyAgreement`: acuerdo de claves

Permite que dos o más partes lleguen a **la misma clave compartida** sin enviarse nunca un secreto. Cada parte:

1. Crea su `KeyAgreement` (`getInstance("ECDH")`, `"X25519"`, `"DH"`…).
2. Lo inicializa con su **clave privada** (`init`).
3. Introduce la **clave pública** de la otra parte (`doPhase(clave, true)`; con `lastPhase = true` en acuerdos de dos partes).
4. Obtiene el secreto con `generateSecret()`.

```java
KeyAgreement ka = KeyAgreement.getInstance("X25519");
ka.init(miClavePrivada);
ka.doPhase(clavePublicaDelOtro, true);
byte[] secretoCompartido = ka.generateSecret();
// Pasa este secreto por una función de derivación (HKDF) antes de usarlo como clave.
```

### 12.8. Novedad: API `KEM` (Java 21)

La clase `javax.crypto.KEM` permite el **encapsulado de claves** (*Key Encapsulation Mechanism*), técnica de las que usarán los algoritmos post-cuánticos. Un lado «encapsula» un secreto con la clave pública del otro, y este lo «desencapsula» con su clave privada.

---

## 13. Gestión de claves: `KeyStore`

Un *keystore* es una base de datos de claves y certificados. Tiene dos tipos de entradas:

- **Entrada de clave** (*key entry*): clave secreta o clave privada con su cadena de certificados, protegida.
- **Entrada de certificado de confianza** (*trusted certificate entry*): certificado de una entidad en la que confías.

### 13.1. Tipos de keystore

| Tipo | Descripción | Recomendación |
|---|---|---|
| **PKCS12** | Estándar multiplataforma. **Es el predeterminado desde Java 9.** | ✅ Usar |
| **JKS** | Formato propietario antiguo de Java | ⚠️ Migrar a PKCS12 |
| **JCEKS** | Propietario, con cifrado más fuerte que JKS | ⚠️ Migrar a PKCS12 |
| **DKS** | Dominio de varios keystores presentado como uno | Casos especiales |
| **PKCS11** | Acceso a tokens, tarjetas inteligentes y HSM | Hardware criptográfico |

Los keystores de distinto tipo **no son compatibles** entre sí. El tipo por defecto se define en la propiedad `keystore.type` del archivo `java.security`.

### 13.2. Ubicación

- El keystore del usuario puede estar en cualquier fichero. Históricamente era `.keystore` en el directorio personal (`user.home`).
- Existe además un almacén global de certificados de confianza, **`cacerts`**, que consulta por defecto el gestor de confianza de TLS:

```text
<JAVA_HOME>/lib/security/cacerts
```

(La documentación antigua indica `lib/ext/cacerts`, que ya no es válido.)

Una aplicación puede ignorar `cacerts` y usar su propio *truststore*.

### 13.3. Ejemplo: leer un keystore PKCS12

```java
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.security.KeyStore;
import java.security.PrivateKey;

char[] password = System.getenv("KEYSTORE_PASS").toCharArray();

KeyStore ks = KeyStore.getInstance("PKCS12");
try (InputStream in = Files.newInputStream(Path.of("mialmacen.p12"))) {
    ks.load(in, password);
}

PrivateKey clavePrivada = (PrivateKey) ks.getKey("miclave", password);
var certificado = ks.getCertificate("miclave");
```

---

## 14. Certificados

- **`CertificateFactory`**: convierte bytes (DER/PEM) en objetos `X509Certificate` y en listas de revocación (CRL).
- **`CertPathBuilder`, `CertPathValidator`, `CertStore`**: construir, validar y buscar cadenas de certificados (forman parte de la guía de PKI de Java).

```java
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;

CertificateFactory cf = CertificateFactory.getInstance("X.509");
try (InputStream in = Files.newInputStream(Path.of("miclave.cer"))) {
    X509Certificate cert = (X509Certificate) cf.generateCertificate(in);
    System.out.println(cert.getSubjectX500Principal());
    cert.checkValidity(); // lanza excepción si está caducado
}
```

---

## 15. Excepciones habituales

| Excepción | Causa típica |
|---|---|
| `NoSuchAlgorithmException` | Algoritmo mal escrito o no disponible |
| `NoSuchProviderException` | Proveedor no instalado |
| `InvalidKeyException` | Clave inadecuada para el algoritmo o tamaño |
| `InvalidAlgorithmParameterException` | Parámetros erróneos (IV, tamaño de tag…) |
| `NoSuchPaddingException` | Relleno no soportado |
| `BadPaddingException` / `AEADBadTagException` | Datos manipulados, clave o IV incorrectos |
| `IllegalBlockSizeException` | Tamaño de datos incompatible con el cifrado de bloque |
| `SignatureException` | Error al firmar o verificar |

---

## 16. Resumen de buenas prácticas

1. Usa **`SecureRandom`** para todo valor sensible.
2. Para cifrar, usa **AES-GCM** (o ChaCha20-Poly1305) con **IV único por cifrado**.
3. Especifica siempre la transformación **completa** (`AES/GCM/NoPadding`).
4. No fijes proveedor salvo que haya una razón clara.
5. Usa **SHA-256 o superior**; evita MD5 y SHA-1.
6. Para contraseñas, usa derivación de claves con sal e iteraciones (PBKDF2, bcrypt, Argon2).
7. Usa **PKCS12** como formato de keystore.
8. No dejes claves en el código fuente.
9. Compara MAC y hashes con `MessageDigest.isEqual`.
10. Mantén tu JDK actualizado.
11. Antes de implementar algo criptográfico complejo, considera usar una biblioteca de alto nivel contrastada (por ejemplo Google Tink).

---

## 17. Glosario rápido

| Término | Definición |
|---|---|
| **AEAD** | Cifrado autenticado con datos asociados (GCM) |
| **AAD** | Datos asociados autenticados pero no cifrados |
| **CSP** | Proveedor de servicios criptográficos |
| **IV** | Vector de inicialización |
| **MAC / HMAC** | Código de autenticación de mensaje |
| **PBE** | Cifrado basado en contraseña |
| **Salt (sal)** | Valor aleatorio añadido al derivar una clave desde una contraseña |
| **SPI** | Interfaz del proveedor de servicio |
| **PKCS#8** | Formato estándar de clave privada |
| **X.509** | Formato estándar de certificado y clave pública |
