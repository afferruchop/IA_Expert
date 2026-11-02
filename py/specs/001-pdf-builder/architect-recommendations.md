# Recomendaciones Arquitectónicas — Criptografía Asimétrica para `.cryptodoc`

**Autor:** Architect  
**Fecha:** 2026-04-28  
**Versión:** 1.0  
**Estado:** Aprobado para revisión

---

## Índice

1. [Resumen Ejecutivo](#1-resumen-ejecutivo)
2. [Stack Criptográfico Definitivo](#2-stack-criptográfico-definitivo)
3. [Justificación de Decisiones Técnicas](#3-justificación-de-decisiones-técnicas)
4. [Flujo de Claves y Almacenamiento Detallado](#4-flujo-de-claves-y-almacenamiento-detallado)
5. [Formato de Archivo `.cryptodoc` Extendido](#5-formato-de-archivo-cryptodoc-extendido)
6. [Correcciones a la Spec](#6-correcciones-a-la-spec)
7. [Plan de Implementación Segura](#7-plan-de-implementación-segura)
8. [Apéndice: Tabla Comparativa de Librerías](#8-apéndice-tabla-comparativa-de-librerías)

---

## 1. Resumen Ejecutivo

La especificación v2.0 que incorpora criptografía asimétrica es **técnicamente viable** con las librerías propuestas (`pointycastle` + `flutter_secure_storage`). Sin embargo, se han identificado **7 riesgos de seguridad** y **3 problemas multiplataforma** que deben mitigarse antes de la implementación.

Este documento detalla las correcciones necesarias a la spec y proporciona las justificaciones técnicas de cada decisión.

### Decisiones clave

| Decisión | Opción recomendada | Justificación |
|---|---|---|
| **Cifrado simétrico** | AES-256-GCM (exclusivo) | Autenticación implícita; evita padding oracle attacks. CBC requiere HMAC adicional |
| **Cifrado asimétrico** | RSA-4096 con OAEP (SHA-256) | Soporte nativo en `pointycastle`. ECDH/P-256 como alternativa futura |
| **Firma digital** | RSA-PSS (SHA-256) o Ed25519 | PSS tiene seguridad demostrable; Ed25519 es más rápido y compacto |
| **Generación de claves** | RSA-4096 (o P-256 para ECDH) | 4096 bits cumple estándares NIST hasta 2030+ |
| **Almacenamiento privado** | `flutter_secure_storage` | Usa Keystore/Keychain nativo. No almacenar claves públicas aquí |
| **Almacenamiento público** | Archivo local + hash de integridad | Las claves públicas no requieren cifrado |

---

## 2. Stack Criptográfico Definitivo

### Dependencias `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter
  pointycastle: ^3.7.3
  flutter_secure_storage: ^9.2.2
  asn1lib: ^1.5.0        # Codificación ASN.1/DER para claves RSA
  rsa_pkcs: ^0.1.0        # Parsing PEM (opcional, para importación)
  path_provider: ^2.1.0   # Almacenamiento de claves públicas
  crypto: ^3.0.3          # Utilidades hash complementarias
```

### Mapa de algoritmos

| Función | Algoritmo | Librería | Justificación |
|---|---|---|---|
| Cifrado simétrico | AES-256-GCM | `pointycastle` | Autenticado (integridad + cifrado). Nonce de 96 bits. Sin padding |
| Cifrado asimétrico (key wrapping) | RSA-OAEP (SHA-256, MGF1) | `pointycastle` | OAEP con SHA-256 evita vulnerabilidades de padding PKCS#1 v1.5 |
| Generación par de claves | RSA 4096 bits | `pointycastle` `SecureRandom` | Estandar, compatible, soporte universal |
| Firma digital | RSA-PSS (SHA-256) | `pointycastle` | Seguridad demostrable (provably secure) |
| Hash | SHA-256 | `pointycastle` | Estándar NIST, suficiente para la vida del proyecto |
| PRNG | Fortuna (o CTR_DRBG) | `pointycastle` `FortunaRandom` | Criptográficamente seguro, reseeding automático |
| KDF (si se necesita migración) | PBKDF2-HMAC-SHA256 o Argon2id | `pointycastle` | Para proteger claves privadas exportadas con passphrase |

> **Nota:** No usar `cryptography` (dint-dev) como reemplazo completo porque **no implementa RSA-OAEP para cifrado asimétrico** (solo RSA signing). Podría usarse para firmas Ed25519 como complemento, pero añade complejidad innecesaria.

### Prohibiciones explícitas

| Práctica | Motivo |
|---|---|
| `math.Random()` | No es criptográficamente seguro |
| RSA < 2048 bits | Longitud insuficiente, vulnerable |
| AES-ECB | No oculta patrones, inseguro |
| Modo CBC sin HMAC | Vulnerable a padding oracle attack |
| RSA PKCS#1 v1.5 para cifrado | Vulnerable a ataques de Bleichenbacher |
| SHA-1 | Colisiones demostradas (SHAttered) |
| Claves privadas en logs, UI, o excepciones | Exposición de material sensible |

---

## 3. Justificación de Decisiones Técnicas

### 3.1 ¿Por qué AES-256-GCM y no CBC?

**Justificación:**

1. **Autenticación integrada:** GCM proporciona cifrado autenticado con adicional authenticated data (AAD). Un solo paso garantiza confidencialidad e integridad.
2. **Sin padding:** GCM es un modo de flujo (CTR-based), no requiere padding. Elimina por completo los ataques de *padding oracle* (CVE-2016-2107, etc.).
3. **Rendimiento:** GCM es paralelizable (contra BBC, el modo CTR de base es paralelizable). En CPUs modernas, hay aceleración por hardware (AES-NI + PCLMULQDQ), aunque en Dart puro el rendimiento depende de la implementación.

**Si se usara CBC (no recomendado):**
- Requeriría HMAC-SHA256 adicional sobre ciphertext + IV.
- El HMAC debería cubrir `IV || ciphertext || aad`.
- Mayor código, mayor superficie de ataque.

### 3.2 ¿Por qué RSA-4096 con OAEP?

- **OAEP (Optimal Asymmetric Encryption Padding):** Es un esquema de padding con seguridad demostrable bajo el modelo Random Oracle. Reemplaza al vulnerable PKCS#1 v1.5.
- **4096 bits:** Proporciona seguridad equivalente a ~128 bits simétricos (vs. 112 bits de RSA-2048). Es suficiente para documentos que deben permanecer confidenciales hasta 2030+.
- **P-256 (ECDH) como alternativa ligera:** Las claves son mucho más pequeñas (256 bits vs 4096 bits). El cifrado híbrido sería: ECDH para acuerdo de clave + AES-GCM para contenido. Sin embargo, ECDH requiere un intercambio de claves por sesión, lo cual es menos intuitivo para un modelo "cifrar para un destinatario" offline.

### 3.3 ¿Por qué RSA-PSS para firmas (y no PKCS#1 v1.5)?

- **PSS (Probabilistic Signature Scheme):** Tiene una prueba de seguridad formal. PKCS#1 v1.5 para firmas no ha sido roto en la práctica, pero su seguridad no está probada formalmente.
- **Compatibilidad:** `pointycastle` soporta `RSASigner` con PSS desde la versión 3.x.

### 3.4 ¿Por qué `flutter_secure_storage` solo para claves privadas?

- Las claves públicas, por definición, **no requieren confidencialidad**. Almacenarlas en `flutter_secure_storage` desperdicia recursos y añade dependencias innecesarias (Keychain en iOS, Keystore en Android).
- Las claves públicas necesitan **integridad** (no haber sido modificadas). Esto se logra almacenando un hash SHA-256 de la clave pública junto a ella.
- **Regla:** `flutter_secure_storage` para secretos; `shared_preferences` o archivo local para claves públicas.

---

## 4. Flujo de Claves y Almacenamiento Detallado

### 4.1 Generación de claves (FR6)

```
Usuario → [Generar par] → SecureRandom (Fortuna) → RSA.generate(4096)
                                                      │
                                                      ├─► PrivateKey → DER/ASN.1 → flutter_secure_storage
                                                      └─► PublicKey  → DER/ASN.1 → local file + SHA-256 hash
```

### 4.2 Almacenamiento

| Dato | Dónde | Formato |
|---|---|---|
| Clave privada (propia) | `flutter_secure_storage` `key: "rsa_private_key_der"` | DER binario (ASN.1) |
| Clave pública (propia) | Archivo local: `{appDir}/public_keys/self.pub` | DER o PEM |
| Claves públicas (contactos) | Archivo local: `{appDir}/public_keys/{fingerprint}.pub` | DER o PEM |
| Preferencias de usuario | `shared_preferences` | JSON |
| PDFs encriptados | Directorio documentos | `.cryptodoc` |

### 4.3 Cifrado de un PDF (FR7-FR8)

```
1. Generar clave simétrica aleatoria: AES_key = SecureRandom(32 bytes)
2. Cifrar PDF con AES_key en modo GCM:
   ┌─────────────────────────────────────┐
   │ nonce (12 bytes) || ciphertext || MAC │  ← encrypted_pdf
   └─────────────────────────────────────┘
3. Cifrar AES_key con clave pública del destinatario:
   ┌────────────────────────────────┐
   │ RSA-OAEP(AES_key, pubKey_Bob) │  ← wrapped_key
   └────────────────────────────────┘
4. Generar firma digital del emisor:
   ┌───────────────────────────────────────────────┐
   │ RSA-PSS(SHA-256(payload), privKey_Alice)      │  ← signature
   └───────────────────────────────────────────────┘
   donde payload = encrypted_pdf || wrapped_key || recipient_fingerprint || metadata
5. Ensamblar .cryptodoc:
   ┌──────────────────────────────────────────────────┐
   │ Header: version, algorithm_suite, metadata       │
   │ Body:                                             │
   │   - encrypted_pdf (GCM)                          │
   │   - wrapped_key (RSA-OAEP)                       │
   │   - signature (RSA-PSS)                          │
   │   - public_key_emitter (para verificación)       │
   └──────────────────────────────────────────────────┘
```

### 4.4 Descifrado y verificación (FR9)

```
1. Extraer encrypted_pdf, wrapped_key, signature, public_key_emitter
2. Verificar firma:
   is_valid = RSA-PSS.verify(signature, payload, pubKey_emitter)
   if !is_valid → mostrar ALERTA: "Firma inválida: el emisor no está autenticado"
3. Descifrar wrapped_key con clave privada del receptor:
   AES_key = RSA-OAEP.decrypt(wrapped_key, privKey_Bob)
4. Descifrar PDF en memoria:
   plaintext_pdf = AES-GCM.decrypt(encrypted_pdf, AES_key)
5. Mostrar PDF al usuario
```

### 4.5 Importación de clave existente (FR6)

```yaml
Formatos soportados:
  - PEM PKCS#1 (-----BEGIN RSA PRIVATE KEY-----)
  - PEM PKCS#8 (-----BEGIN PRIVATE KEY-----)
  - DER binario

Proceso:
  1. Leer archivo
  2. Parsear con rsa_pkcs o asn1lib
  3. Validar: longitud >= 2048 bits (rechazar si menor)
  4. Almacenar en flutter_secure_storage
  5. Extraer y almacenar clave pública

⚠️ Al importar, pedir confirmación al usuario:
  "Esta clave reemplazará tu clave privada actual.
   Si pierdes acceso a documentos cifrados con la anterior,
   no podrás recuperarlos."
```

---

## 5. Formato de Archivo `.cryptodoc` Extendido

Se recomienda un formato binario con header descriptivo:

```
┌─────────────────────────────────────────────────────────┐
│ Magic bytes: 0x43 0x52 0x59 0x50 0x54 0x4F  ("CRYPTO") │
│ Version:      uint8 (0x01)                              │
│ Algorithm:    uint8 (0x01 = RSA-4096-OAEP + AES-256-GCM)│
│ Flags:        uint8 (0x01 = firmado)                     │
│ Reserved:     uint8 (0x00)                              │
├─────────────────────────────────────────────────────────┤
│ Recipient fingerprint: 32 bytes (SHA-256 de pub key)    │
│ Emitter public key: variable (DER-encoded)              │
│ Emitter fingerprint: 32 bytes (SHA-256 de emitter pub)  │
├─────────────────────────────────────────────────────────┤
│ Encrypted PDF (GCM): variable                           │
│   ├─ Nonce: 12 bytes                                    │
│   ├─ Ciphertext: variable                               │
│   └─ MAC tag: 16 bytes                                  │
├─────────────────────────────────────────────────────────┤
│ Wrapped AES key (RSA-OAEP): 512 bytes (para RSA-4096)   │
├─────────────────────────────────────────────────────────┤
│ Signature (RSA-PSS): 512 bytes                          │
│   (cubre: header + recipient_fingerprint +               │
│    emitter_pubkey + encrypted_pdf + wrapped_key)        │
├─────────────────────────────────────────────────────────┤
│ Metadata: JSON opcional (nombre, fecha, etc.)            │
└─────────────────────────────────────────────────────────┘
```

### Justificación del diseño

- **Magic bytes:** Identificación rápida del tipo de archivo sin depender de extensión.
- **Version + Algorithm:** Permite *forward compatibility* y evita *downgrade attacks*.
- **Recipient fingerprint:** Permite al visor saber rápidamente qué clave privada usar (útil si se tienen múltiples claves).
- **Emitter public key incluida:** El visor puede verificar la firma inmediatamente, sin necesidad de tener la clave pública del emisor en su directorio local.
- **Firma sobre todo el payload:** Impide *ciphertext-swapping attacks* (no se puede reemplazar `wrapped_key` sin invalidar la firma).

---

## 6. Correcciones a la Spec

### Corrección 1 — NFR1: Especificar modo GCM exclusivamente

**Texto actual (spec):**
> Simétrico: AES‑256 en modo CBC o GCM (autenticado).

**Texto corregido:**
> Simétrico: AES‑256 en modo GCM (autenticado) con nonce de 96 bits. **No se permite CBC** sin HMAC-SHA256 adicional. El cifrado GCM proporciona autenticación implícita del ciphertext. El nonce será generado con `SecureRandom` de `pointycastle`.

**Justificación:** Ver sección 3.1.

### Corrección 2 — NFR1: Especificar OAEP para RSA

**Texto actual (spec):**
> Asimétrico: RSA‑4096 con OAEP o ECDH P‑256 + AES.

**Texto corregido:**
> Asimétrico: RSA‑4096 con OAEP (SHA-256, MGF1). **No se permite PKCS#1 v1.5** para cifrado. ECDH P‑256 se documenta como alternativa futura pero no se implementa en v1.0.

**Justificación:** Ver sección 3.2.

### Corrección 3 — NFR1: Especificar algoritmo de firma

**Texto actual (spec):** No menciona algoritmo de firma.

**Texto añadido:**
> Firma digital: RSA‑PSS (SHA‑256) o Ed25519. El payload firmado incluirá: `encrypted_pdf || wrapped_key || recipient_fingerprint || metadata_header`.

### Corrección 4 — Añadir NFR7: Key rotation y migración

**Texto añadido:**
> **NFR7:** El formato `.cryptodoc` incluirá un campo `version` y `algorithm_suite` para permitir rotación de claves y migración futura de algoritmos. El visor rechazará versiones no soportadas.

### Corrección 5 — Añadir NFR8: Exportación de clave privada

**Texto añadido:**
> **NFR8:** La aplicación permitirá exportar la clave privada cifrada con una passphrase (PBKDF2‑HMAC‑SHA256 con 600,000 iteraciones mínimo, o Argon2id) para migración entre dispositivos. La exportación requerirá confirmación explícita del usuario.

### Corrección 6 — NFR5: Separar almacenamiento de claves

**Texto actual (spec):**
> Las claves privadas se almacenan en `flutter_secure_storage`.

**Texto corregido:**
> Las **claves privadas** se almacenan en `flutter_secure_storage`. Las **claves públicas** se almacenan en archivo local (`path_provider`) con hash SHA-256 de integridad asociado.

### Corrección 7 — Añadir NFR9: Feedback visual de firma

**Texto añadido:**
> **NFR9:** La aplicación debe mostrar un indicador visual claro (icono verde/rojo) sobre el estado de la firma digital. Si la firma es inválida, el documento no se descifrará y se mostrará un mensaje de advertencia. Si la firma es válida, se mostrará la identidad del emisor (fingerprint o alias asociado).

---

## 7. Plan de Implementación Segura

### Fase 1 — Primitivas criptográficas (Tarea 2 actualizada)

```
- [ ] Implementar clase CryptoSymmetric:
    - AES-256-GCM (encrypt/decrypt)
    - Generación de nonce con FortunaRandom
- [ ] Implementar clase CryptoAsymmetric:
    - RSA key generation (4096 bits)
    - RSA-OAEP key wrapping/unwrapping
    - RSA-PSS signing/verification
- [ ] Import/export de claves (DER/PEM)
- [ ] Tests unitarios con vectores de test conocidos
```

### Fase 2 — Formato `.cryptodoc` (Tarea 5 actualizada)

```
- [ ] Definir estructura binaria según sección 5
- [ ] Implementar serialización/deserialización
- [ ] Validar integridad del header (magic bytes, version)
- [ ] Tests: crear y leer archivos .cryptodoc válidos e inválidos
```

### Fase 3 — Almacenamiento seguro (Tarea 4 actualizada)

```
- [ ] flutter_secure_storage para clave privada
- [ ] Archivo local + hash para claves públicas
- [ ] Directorio de contactos (key ring local)
- [ ] Tests: persistencia entre sesiones
```

### Fase 4 — UI de gestión de claves (Tarea 5 actualizada)

```
- [ ] Pantalla: "Mis claves" (generar, importar, exportar)
- [ ] Pantalla: "Contactos" (importar clave pública, alias)
- [ ] Mostrar fingerprint visual (hex abreviado o QR)
- [ ] Confirmación al reemplazar clave privada
```

### Fase 5 — Cifrado y visor (Tareas 6-7 actualizadas)

```
- [ ] Integrar cifrado en builder de PDF
- [ ] Integrar descifrado en visor con verificación de firma
- [ ] Indicadores visuales de seguridad
- [ ] Tests de integración: cifrar → descifrar → verificar
```

### Fase 6 — Revisión de seguridad

```
- [ ] Escaneo de fugas (print, debugPrint con datos sensibles)
- [ ] Verificar que SecureRandom se usa en todos los casos
- [ ] Probar con claves inválidas, corruptas, maliciosas
- [ ] Pruebas en Android, iOS, Windows, Linux, macOS
- [ ] Revisión de dependencias con pub outdated
```

---

## 8. Apéndice: Tabla Comparativa de Librerías

| Característica | `pointycastle` | `cryptography` (dint-dev) | `openpgp` (flutter) |
|---|---|---|---|
| AES-256-GCM | ✅ | ✅ | ❌ (solo PGP) |
| RSA-OAEP | ✅ | ❌ | ❌ |
| RSA-PSS | ✅ | ❌ | ❌ |
| Ed25519 | ✅ | ✅ | ✅ |
| RSA Key Gen | ✅ | ❌ | ✅ |
| ECDH | ✅ | ✅ | ✅ |
| Gestión de claves | Manual (bajo nivel) | API moderna (asincrónica) | Auto (PGP) |
| SecureRandom | ✅ Fortuna | ✅ ChaCha12 | Interno |
| Mantenimiento | ✅ Bouncy Castle | ✅ dint.dev | ⚠️ Individual |
| Multiplataforma | ✅ Dart puro | ✅ Dart puro | ⚠️ requires Go native |
| ASN.1/DER | ⚠️ parcial | ❌ | ❌ |
| **Veredicto** | **✅ Recomendado** | Parcial (solo firmas/hash) | No recomendado (sobrecarga PGP) |

### Conclusión del apéndice

`pointycastle` es la elección correcta para este proyecto porque:
1. Es la **única** que implementa **RSA-OAEP** (necesario para FR7).
2. Es **Dart puro** → funciona idéntico en todas las plataformas.
3. Proporciona **control granular** necesario para un formato propietario.
4. Soporta **ASN.1/DER** para serialización de claves.

La librería `cryptography` es superior en API/ergonomía, pero **no cubre el caso de uso de key wrapping asimétrico con RSA-OAEP**, que es central en la spec.

---

## Histórico de revisiones

| Versión | Fecha | Cambios |
|---|---|---|
| 1.0 | 2026-04-28 | Versión inicial — análisis completo y recomendaciones |

---

*Documento generado por el rol Architect. Para implementación, referirse a `tasks.md` actualizado.*
