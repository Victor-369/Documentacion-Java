# Guía del desarrollador de seguridad en Java SE (Java 21+)

> **Fuente de referencia:** *Java Platform, Standard Edition Security Developer's Guide* (Oracle).
> **Destinatario:** programador Java junior.
> **Versión objetivo:** Java 21 (LTS) y posteriores.

---

## 1. ¿Qué es esta guía?

La *Security Developer's Guide* de Oracle es el manual oficial que explica la tecnología de seguridad de la plataforma Java SE: las herramientas, los algoritmos, los mecanismos y los protocolos de seguridad más habituales y cómo se implementan en Java.

La página principal solo es un índice. Este documento explica, en castellano y con ejemplos, qué áreas cubre esa guía y qué debes saber de cada una al trabajar con Java 21 o superior.

---

## 2. Mapa de la seguridad en Java

Java no tiene un único «módulo de seguridad». Es un conjunto de piezas que se complementan:

| Área | Para qué sirve | Paquetes / herramientas principales |
|---|---|---|
| **Seguridad del lenguaje y la JVM** | Tipado fuerte, verificación de *bytecode*, gestión de memoria sin punteros | La propia JVM |
| **Criptografía (JCA/JCE)** | Hashes, cifrado, firmas digitales, claves, números aleatorios | `java.security`, `javax.crypto` |
| **Infraestructura de clave pública (PKI)** | Certificados, cadenas de confianza, validación | `java.security.cert`, `CertPathValidator` |
| **Comunicaciones seguras (JSSE)** | TLS/DTLS sobre *sockets* y HTTP | `javax.net.ssl` |
| **Autenticación y autorización (JAAS)** | Identificar usuarios y decidir qué pueden hacer | `javax.security.auth` |
| **Autenticación segura (SASL, GSS-API/Kerberos)** | Intercambio de mensajes autenticados entre aplicaciones | `javax.security.sasl`, `org.ietf.jgss` |
| **Firma de código** | Garantizar el origen y la integridad de un JAR | `jarsigner`, `keytool` |
| **Serialización segura** | Evitar ataques por deserialización | `ObjectInputFilter` |
| **Control de acceso (*Security Manager*)** | Restringir permisos de código | **En desuso, ver sección 4** |

---

## 3. Conceptos básicos que debes dominar

### 3.1. Los tres pilares

- **Confidencialidad:** solo quien debe leer los datos puede hacerlo (se logra con **cifrado**).
- **Integridad:** los datos no han sido alterados (se logra con **hashes**, **MAC** y **firmas**).
- **Autenticidad:** sabemos quién es el emisor (se logra con **firmas digitales**, **certificados** y **autenticación**).

### 3.2. Vocabulario imprescindible

| Término | Significado |
|---|---|
| **Hash / *digest*** | Resumen de tamaño fijo de unos datos. No es reversible. |
| **Cifrado simétrico** | Una misma clave cifra y descifra (AES). Rápido. |
| **Cifrado asimétrico** | Par de claves: pública y privada (RSA, EC). Más lento. |
| **MAC / HMAC** | Código de autenticación de mensaje: hash con clave secreta. |
| **Firma digital** | Se firma con la clave privada y se verifica con la pública. |
| **Certificado** | Documento firmado que vincula una clave pública con una identidad. |
| **Keystore** | Almacén de claves y certificados (por defecto, formato PKCS12). |
| **Truststore** | Keystore que contiene las entidades en las que confiamos (CA). |
| **Proveedor (*provider*)** | Paquete que implementa algoritmos criptográficos. |
| **TLS** | Protocolo de transporte seguro (sucesor de SSL). |

---

## 4. Cambios importantes desde Java 17 hasta Java 21 y posteriores

Esta es la parte donde la documentación antigua puede confundirte.

### 4.1. El *Security Manager* está en desuso

- En **Java 17** (JEP 411) se marcó como obsoleto para su eliminación.
- En **Java 21** sigue existiendo, pero está **desactivado por defecto**. Para usarlo hay que activarlo explícitamente con `-Djava.security.manager=allow`. Aun así, no se recomienda.
- A partir de **Java 24** (JEP 486) se ha **desactivado de forma permanente**.

> **Consejo:** no diseñes aplicaciones nuevas pensando en políticas de permisos con `Security Manager`. Usa aislamiento a nivel de contenedor, sistema operativo y mínimos privilegios.

### 4.2. Mecanismo de extensiones eliminado

Desde Java 9 ya no existe el directorio `lib/ext`. Muchas guías antiguas siguen mencionándolo. Los proveedores de seguridad se añaden hoy mediante *classpath*/*module path* y se registran en `java.security` o dinámicamente.

### 4.3. Ubicación del archivo de configuración

El archivo principal de propiedades de seguridad es:

```text
<JAVA_HOME>/conf/security/java.security
```

Y el almacén de certificados de confianza por defecto (`cacerts`) está en:

```text
<JAVA_HOME>/lib/security/cacerts
```

### 4.4. Algoritmos y protocolos

- **TLS 1.3** está soportado y es el preferido; TLS 1.0 y 1.1 están deshabilitados.
- **MD5 y SHA-1** se consideran débiles. No los uses para seguridad (firmas, contraseñas, integridad).
- **DES y 3DES** están desaconsejados o deshabilitados. Usa **AES**.
- **RSA** de menos de 2048 bits está restringido.
- El formato de *keystore* por defecto es **PKCS12** (desde Java 9).

### 4.5. Novedades de criptografía en versiones recientes

| Versión | Novedad |
|---|---|
| Java 15 | Firmas **EdDSA** (Ed25519 y Ed448) |
| Java 17 | **HexFormat** para convertir bytes ↔ hexadecimal (`java.util.HexFormat`) |
| Java 21 | API **KEM** (`javax.crypto.KEM`), base para criptografía post-cuántica |
| Java 24 | Algoritmos post-cuánticos **ML-KEM** y **ML-DSA** |
| Java 24 | Security Manager deshabilitado de forma permanente |

---

## 5. Herramientas de línea de comandos

Vienen incluidas en el JDK.

### 5.1. `keytool`: gestión de claves y certificados

```bash
# Crear un keystore PKCS12 con un par de claves RSA
keytool -genkeypair -alias miclave -keyalg RSA -keysize 3072 \
        -validity 365 -keystore mialmacen.p12 -storetype PKCS12

# Listar el contenido del almacén
keytool -list -v -keystore mialmacen.p12

# Exportar el certificado
keytool -exportcert -alias miclave -keystore mialmacen.p12 -file miclave.cer
```

### 5.2. `jarsigner`: firma y verificación de JAR

```bash
jarsigner -keystore mialmacen.p12 miapp.jar miclave
jarsigner -verify -verbose miapp.jar
```

---

## 6. Buenas prácticas generales

1. **No inventes tu propia criptografía.** Usa algoritmos estándar y bibliotecas probadas.
2. **No copies código criptográfico sin entenderlo.** Un uso incorrecto puede dejar tu aplicación igual de vulnerable que si no cifrases nada.
3. **Elige algoritmos modernos:** AES-GCM, SHA-256 o superior, RSA ≥ 2048 (mejor 3072), curvas elípticas, Ed25519.
4. **Nunca guardes contraseñas en claro ni con un hash simple.** Usa un algoritmo de derivación con sal e iteraciones (PBKDF2, bcrypt, scrypt o Argon2).
5. **No incluyas claves ni contraseñas en el código fuente** ni en el control de versiones. Usa variables de entorno, *vaults* o *keystores*.
6. **Usa `SecureRandom`, nunca `java.util.Random`,** para cualquier valor sensible.
7. **Valida siempre las entradas** y trata los datos externos como no fiables.
8. **No deserialices datos no confiables.** Si debes hacerlo, usa filtros (`ObjectInputFilter`).
9. **Mantén el JDK actualizado.** Las actualizaciones trimestrales corrigen vulnerabilidades.
10. **No desactives la validación de certificados TLS** «para que funcione». Es un error muy habitual y grave.
11. **Aplica el principio de mínimo privilegio.**
12. **No registres datos sensibles** (contraseñas, claves, tokens) en logs.

---

## 7. Ejemplo mínimo: hash SHA-256 de un texto

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.HexFormat;

public class EjemploHash {
    public static void main(String[] args) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] hash = md.digest("Hola, mundo".getBytes(StandardCharsets.UTF_8));
        System.out.println(HexFormat.of().formatHex(hash));
    }
}
```

> Este ejemplo sirve para comprobar la integridad de datos. **No es adecuado para guardar contraseñas.**

---

## 8. Cómo seguir aprendiendo

- Fichero `02-arquitectura-criptografica-jca.md`: explica en profundidad la arquitectura JCA.
- Documentación oficial de Oracle: *Security Developer's Guide* de la versión del JDK que uses.
- Referencia de nombres estándar de algoritmos: *Java Security Standard Algorithm Names*.
- Recomendaciones de OWASP (*Cryptographic Storage Cheat Sheet*, *Password Storage Cheat Sheet*).
