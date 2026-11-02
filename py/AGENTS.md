# 🧠 Contexto Global para Agentes IA — Cryptodoc

**Versión:** 1.0  
**Última actualización:** 2026-04-29  
**Aplicable a:** Todos los agentes (Continue, Cursor, Copilot, etc.) que interactúen con este código.

---

## 🎯 Propósito del Proyecto

Cryptodoc es una aplicación **Flutter multiplataforma** que permite construir, cifrar (con criptografía asimétrica), compartir y visualizar PDFs protegidos. El formato de intercambio es `.cryptodoc`.

**Principios no negociables:**

1. **Seguridad por diseño** – Todas las decisiones criptográficas siguen estándares NIST y se implementan con librerías auditadas (`pointycastle`, `flutter_secure_storage`).
2. **Cero confianza en la red** – La aplicación funciona completamente offline; nunca se transmiten claves ni documentos descifrados.
3. **Auditabilidad** – Cada operación criptográfica debe poder ser verificada mediante tests y logging ofuscado (sin datos sensibles).
4. **Inmutabilidad algorítmica** – Solo se usan los algoritmos especificados en `CRYPTOGRAPHY_GUIDE.md` (AES-256-GCM, RSA-4096-OAEP, RSA-PSS, SHA-256).

---

## 📚 Stack Tecnológico (NO desviarse)

| Categoría                  | Tecnología específica                              | Justificación                                                 |
| -------------------------- | -------------------------------------------------- | ------------------------------------------------------------- |
| **Lenguaje**               | Dart 3.x + Flutter 3.x                             | Multiplataforma (Windows, Linux, macOS, Android, iOS)         |
| **Cripto simétrica**       | `pointycastle` – AES-256-GCM (nonce 96 bits)       | Autenticado, sin padding, seguro contra oracle                |
| **Cripto asimétrica**      | `pointycastle` – RSA-4096 con OAEP (SHA-256)       | Estándar, soporte nativo, hasta 2030+                         |
| **Firma digital**          | `pointycastle` – RSA-PSS (SHA-256)                 | Seguridad demostrable                                         |
| **Almacenamiento privado** | `flutter_secure_storage` (Keystore/Keychain)       | Las claves privadas **nunca** tocan el disco en claro         |
| **Almacenamiento público** | Archivo local (`path_provider`) + hash SHA-256     | Las claves públicas requieren integridad, no confidencialidad |
| **Formato de archivo**     | Binario propio – `.cryptodoc` (ver especificación) | Incluye magic bytes, versión, firma, etc.                     |

**Prohibiciones explícitas:**

- ❌ `math.Random()` – Usar siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- ❌ RSA < 2048 bits (rechazar en importación).
- ❌ AES en modo CBC o ECB.
- ❌ RSA PKCS#1 v1.5 para cifrado.
- ❌ SHA-1.
- ❌ Almacenar claves privadas en logs, UI, excepciones o variables persistentes.
- ❌ Guardar PDFs descifrados en disco (solo en memoria).

---

## 🧩 Directrices para Agentes IA

Cuando generes, sugieras o modifiques código, debes aplicar las siguientes reglas **automáticamente**:

### Regla 1 – Generación de números aleatorios

````dart
// ✅ Correcto
import 'package:pointycastle/random/fortuna_random.dart';
final secureRandom = FortunaRandom();
final nonce = secureRandom.nextBytes(12);

// ❌ Incorrecto
final nonce = List.generate(12, (_) => Random().nextInt(256));# 🧠 Contexto Global para Agentes IA — Cryptodoc

**Versión:** 1.0
**Última actualización:** 2026-04-29
**Aplicable a:** Todos los agentes (Continue, Cursor, Copilot, etc.) que interactúen con este código.

---

## 🎯 Propósito del Proyecto

Cryptodoc es una aplicación **Flutter multiplataforma** que permite construir, cifrar (con criptografía asimétrica), compartir y visualizar PDFs protegidos. El formato de intercambio es `.cryptodoc`.

**Principios no negociables:**

1. **Seguridad por diseño** – Todas las decisiones criptográficas siguen estándares NIST y se implementan con librerías auditadas (`pointycastle`, `flutter_secure_storage`).
2. **Cero confianza en la red** – La aplicación funciona completamente offline; nunca se transmiten claves ni documentos descifrados.
3. **Auditabilidad** – Cada operación criptográfica debe poder ser verificada mediante tests y logging ofuscado (sin datos sensibles).
4. **Inmutabilidad algorítmica** – Solo se usan los algoritmos especificados en `CRYPTOGRAPHY_GUIDE.md` (AES-256-GCM, RSA-4096-OAEP, RSA-PSS, SHA-256).

---

## 📚 Stack Tecnológico (NO desviarse)

| Categoría          | Tecnología específica                              | Justificación                                        |
| ------------------ | -------------------------------------------------- | ---------------------------------------------------- |
| **Lenguaje**       | Dart 3.x + Flutter 3.x                             | Multiplataforma (Windows, Linux, macOS, Android, iOS) |
| **Cripto simétrica** | `pointycastle` – AES-256-GCM (nonce 96 bits)      | Autenticado, sin padding, seguro contra oracle      |
| **Cripto asimétrica** | `pointycastle` – RSA-4096 con OAEP (SHA-256)     | Estándar, soporte nativo, hasta 2030+               |
| **Firma digital**  | `pointycastle` – RSA-PSS (SHA-256)                 | Seguridad demostrable                               |
| **Almacenamiento privado** | `flutter_secure_storage` (Keystore/Keychain) | Las claves privadas **nunca** tocan el disco en claro |
| **Almacenamiento público** | Archivo local (`path_provider`) + hash SHA-256 | Las claves públicas requieren integridad, no confidencialidad |
| **Formato de archivo** | Binario propio – `.cryptodoc` (ver especificación) | Incluye magic bytes, versión, firma, etc.          |

**Prohibiciones explícitas:**

- ❌ `math.Random()` – Usar siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- ❌ RSA < 2048 bits (rechazar en importación).
- ❌ AES en modo CBC o ECB.
- ❌ RSA PKCS#1 v1.5 para cifrado.
- ❌ SHA-1.
- ❌ Almacenar claves privadas en logs, UI, excepciones o variables persistentes.
- ❌ Guardar PDFs descifrados en disco (solo en memoria).

---

## 🧩 Directrices para Agentes IA

Cuando generes, sugieras o modifiques código, debes aplicar las siguientes reglas **automáticamente**:

### Regla 1 – Generación de números aleatorios
```dart
// ✅ Correcto
import 'package:pointycastle/random/fortuna_random.dart';
final secureRandom = FortunaRandom();
final nonce = secureRandom.nextBytes(12);

// ❌ Incorrecto
final nonce = List.generate(12, (_) => Random().nextInt(256));# 🧠 Contexto Global para Agentes IA — Cryptodoc

**Versión:** 1.0
**Última actualización:** 2026-04-29
**Aplicable a:** Todos los agentes (Continue, Cursor, Copilot, etc.) que interactúen con este código.

---

## 🎯 Propósito del Proyecto

Cryptodoc es una aplicación **Flutter multiplataforma** que permite construir, cifrar (con criptografía asimétrica), compartir y visualizar PDFs protegidos. El formato de intercambio es `.cryptodoc`.

**Principios no negociables:**

1. **Seguridad por diseño** – Todas las decisiones criptográficas siguen estándares NIST y se implementan con librerías auditadas (`pointycastle`, `flutter_secure_storage`).
2. **Cero confianza en la red** – La aplicación funciona completamente offline; nunca se transmiten claves ni documentos descifrados.
3. **Auditabilidad** – Cada operación criptográfica debe poder ser verificada mediante tests y logging ofuscado (sin datos sensibles).
4. **Inmutabilidad algorítmica** – Solo se usan los algoritmos especificados en `CRYPTOGRAPHY_GUIDE.md` (AES-256-GCM, RSA-4096-OAEP, RSA-PSS, SHA-256).

---

## 📚 Stack Tecnológico (NO desviarse)

| Categoría          | Tecnología específica                              | Justificación                                        |
| ------------------ | -------------------------------------------------- | ---------------------------------------------------- |
| **Lenguaje**       | Dart 3.x + Flutter 3.x                             | Multiplataforma (Windows, Linux, macOS, Android, iOS) |
| **Cripto simétrica** | `pointycastle` – AES-256-GCM (nonce 96 bits)      | Autenticado, sin padding, seguro contra oracle      |
| **Cripto asimétrica** | `pointycastle` – RSA-4096 con OAEP (SHA-256)     | Estándar, soporte nativo, hasta 2030+               |
| **Firma digital**  | `pointycastle` – RSA-PSS (SHA-256)                 | Seguridad demostrable                               |
| **Almacenamiento privado** | `flutter_secure_storage` (Keystore/Keychain) | Las claves privadas **nunca** tocan el disco en claro |
| **Almacenamiento público** | Archivo local (`path_provider`) + hash SHA-256 | Las claves públicas requieren integridad, no confidencialidad |
| **Formato de archivo** | Binario propio – `.cryptodoc` (ver especificación) | Incluye magic bytes, versión, firma, etc.          |

**Prohibiciones explícitas:**

- ❌ `math.Random()` – Usar siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- ❌ RSA < 2048 bits (rechazar en importación).
- ❌ AES en modo CBC o ECB.
- ❌ RSA PKCS#1 v1.5 para cifrado.
- ❌ SHA-1.
- ❌ Almacenar claves privadas en logs, UI, excepciones o variables persistentes.
- ❌ Guardar PDFs descifrados en disco (solo en memoria).

---

## 🧩 Directrices para Agentes IA

Cuando generes, sugieras o modifiques código, debes aplicar las siguientes reglas **automáticamente**:

### Regla 1 – Generación de números aleatorios
```dart
// ✅ Correcto
import 'package:pointycastle/random/fortuna_random.dart';
final secureRandom = FortunaRandom();
final nonce = secureRandom.nextBytes(12);

// ❌ Incorrecto
final nonce = List.generate(12, (_) => Random().nextInt(256));# 🧠 Contexto Global para Agentes IA — Cryptodoc

**Versión:** 1.0
**Última actualización:** 2026-04-29
**Aplicable a:** Todos los agentes (Continue, Cursor, Copilot, etc.) que interactúen con este código.

---

## 🎯 Propósito del Proyecto

Cryptodoc es una aplicación **Flutter multiplataforma** que permite construir, cifrar (con criptografía asimétrica), compartir y visualizar PDFs protegidos. El formato de intercambio es `.cryptodoc`.

**Principios no negociables:**

1. **Seguridad por diseño** – Todas las decisiones criptográficas siguen estándares NIST y se implementan con librerías auditadas (`pointycastle`, `flutter_secure_storage`).
2. **Cero confianza en la red** – La aplicación funciona completamente offline; nunca se transmiten claves ni documentos descifrados.
3. **Auditabilidad** – Cada operación criptográfica debe poder ser verificada mediante tests y logging ofuscado (sin datos sensibles).
4. **Inmutabilidad algorítmica** – Solo se usan los algoritmos especificados en `CRYPTOGRAPHY_GUIDE.md` (AES-256-GCM, RSA-4096-OAEP, RSA-PSS, SHA-256).

---

## 📚 Stack Tecnológico (NO desviarse)

| Categoría          | Tecnología específica                              | Justificación                                        |
| ------------------ | -------------------------------------------------- | ---------------------------------------------------- |
| **Lenguaje**       | Dart 3.x + Flutter 3.x                             | Multiplataforma (Windows, Linux, macOS, Android, iOS) |
| **Cripto simétrica** | `pointycastle` – AES-256-GCM (nonce 96 bits)      | Autenticado, sin padding, seguro contra oracle      |
| **Cripto asimétrica** | `pointycastle` – RSA-4096 con OAEP (SHA-256)     | Estándar, soporte nativo, hasta 2030+               |
| **Firma digital**  | `pointycastle` – RSA-PSS (SHA-256)                 | Seguridad demostrable                               |
| **Almacenamiento privado** | `flutter_secure_storage` (Keystore/Keychain) | Las claves privadas **nunca** tocan el disco en claro |
| **Almacenamiento público** | Archivo local (`path_provider`) + hash SHA-256 | Las claves públicas requieren integridad, no confidencialidad |
| **Formato de archivo** | Binario propio – `.cryptodoc` (ver especificación) | Incluye magic bytes, versión, firma, etc.          |

**Prohibiciones explícitas:**

- ❌ `math.Random()` – Usar siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- ❌ RSA < 2048 bits (rechazar en importación).
- ❌ AES en modo CBC o ECB.
- ❌ RSA PKCS#1 v1.5 para cifrado.
- ❌ SHA-1.
- ❌ Almacenar claves privadas en logs, UI, excepciones o variables persistentes.
- ❌ Guardar PDFs descifrados en disco (solo en memoria).

---

## 🧩 Directrices para Agentes IA

Cuando generes, sugieras o modifiques código, debes aplicar las siguientes reglas **automáticamente**:

### Regla 1 – Generación de números aleatorios
```dart
// ✅ Correcto
import 'package:pointycastle/random/fortuna_random.dart';
final secureRandom = FortunaRandom();
final nonce = secureRandom.nextBytes(12);

// ❌ Incorrecto
final nonce = List.generate(12, (_) => Random().nextInt(256));# 🧠 Contexto Global para Agentes IA — Cryptodoc

**Versión:** 1.0
**Última actualización:** 2026-04-29
**Aplicable a:** Todos los agentes (Continue, Cursor, Copilot, etc.) que interactúen con este código.

---

## 🎯 Propósito del Proyecto

Cryptodoc es una aplicación **Flutter multiplataforma** que permite construir, cifrar (con criptografía asimétrica), compartir y visualizar PDFs protegidos. El formato de intercambio es `.cryptodoc`.

**Principios no negociables:**

1. **Seguridad por diseño** – Todas las decisiones criptográficas siguen estándares NIST y se implementan con librerías auditadas (`pointycastle`, `flutter_secure_storage`).
2. **Cero confianza en la red** – La aplicación funciona completamente offline; nunca se transmiten claves ni documentos descifrados.
3. **Auditabilidad** – Cada operación criptográfica debe poder ser verificada mediante tests y logging ofuscado (sin datos sensibles).
4. **Inmutabilidad algorítmica** – Solo se usan los algoritmos especificados en `CRYPTOGRAPHY_GUIDE.md` (AES-256-GCM, RSA-4096-OAEP, RSA-PSS, SHA-256).

---

## 📚 Stack Tecnológico (NO desviarse)

| Categoría          | Tecnología específica                              | Justificación                                        |
| ------------------ | -------------------------------------------------- | ---------------------------------------------------- |
| **Lenguaje**       | Dart 3.x + Flutter 3.x                             | Multiplataforma (Windows, Linux, macOS, Android, iOS) |
| **Cripto simétrica** | `pointycastle` – AES-256-GCM (nonce 96 bits)      | Autenticado, sin padding, seguro contra oracle      |
| **Cripto asimétrica** | `pointycastle` – RSA-4096 con OAEP (SHA-256)     | Estándar, soporte nativo, hasta 2030+               |
| **Firma digital**  | `pointycastle` – RSA-PSS (SHA-256)                 | Seguridad demostrable                               |
| **Almacenamiento privado** | `flutter_secure_storage` (Keystore/Keychain) | Las claves privadas **nunca** tocan el disco en claro |
| **Almacenamiento público** | Archivo local (`path_provider`) + hash SHA-256 | Las claves públicas requieren integridad, no confidencialidad |
| **Formato de archivo** | Binario propio – `.cryptodoc` (ver especificación) | Incluye magic bytes, versión, firma, etc.          |

**Prohibiciones explícitas:**

- ❌ `math.Random()` – Usar siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- ❌ RSA < 2048 bits (rechazar en importación).
- ❌ AES en modo CBC o ECB.
- ❌ RSA PKCS#1 v1.5 para cifrado.
- ❌ SHA-1.
- ❌ Almacenar claves privadas en logs, UI, excepciones o variables persistentes.
- ❌ Guardar PDFs descifrados en disco (solo en memoria).

---

## 🧩 Directrices para Agentes IA

Cuando generes, sugieras o modifiques código, debes aplicar las siguientes reglas **automáticamente**:

### Regla 1 – Generación de números aleatorios
```dart
// ✅ Correcto
import 'package:pointycastle/random/fortuna_random.dart';
final secureRandom = FortunaRandom();
final nonce = secureRandom.nextBytes(12);

// ❌ Incorrecto
final nonce = List.generate(12, (_) => Random().nextInt(256));
````
