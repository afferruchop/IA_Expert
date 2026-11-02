# 📋 Registro Arquitectónico — Cryptodoc

**Propósito:** Bitácora histórica de decisiones, pruebas, errores y aciertos durante el desarrollo del proyecto.
**Formato:** Cada entrada documenta qué se hizo, por qué, el resultado de las pruebas, y las lecciones aprendidas.

---

## Entrada 001 — 2026-04-28

### Actividad

Revisión arquitectónica completa de la Spec 001 v2.0 (criptografía asimétrica).

### Archivos afectados

| Archivo                                              | Acción                                                                                 |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `specs/001-pdf-builder/spec.md`                      | ✅ Actualizado — algoritmos definitivos, restricciones de seguridad, NFRs 7-9 añadidos |
| `specs/001-pdf-builder/steering.md`                  | ✅ Actualizado — hooks de seguridad específicos para criptografía                      |
| `specs/001-pdf-builder/tasks.md`                     | ✅ Reestructurado en 6 fases con granularidad fina                                     |
| `specs/001-pdf-builder/architect-recommendations.md` | ✅ Creado — documento completo de recomendaciones y justificaciones                    |
| `CHANGELOG_ARCHITECT.md`                             | ✅ Creado — este archivo                                                               |

### Pruebas realizadas

No se ejecutaron pruebas de código (fase de diseño). Se realizaron las siguientes validaciones documentales:

| Prueba                                                                                                                             | Resultado                                                                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Viabilidad de pointycastle** → ¿Implementa RSA-OAEP?                                                                             | ✅ Sí, vía `OAEPEncoding` + `RSAEngine`                                                                                                                       |
| **Viabilidad de pointycastle** → ¿Implementa AES-256-GCM?                                                                          | ✅ Sí, vía `AEADCipher('AES/GCM/NoPadding')`                                                                                                                  |
| **Viabilidad de pointycastle** → ¿Implementa RSA-PSS?                                                                              | ✅ Sí, vía `RSASigner` con Digest y salt length                                                                                                               |
| **Viabilidad de pointycastle** → ¿SecureRandom criptográfico?                                                                      | ✅ Sí, `FortunaRandom` con reseeding automático                                                                                                               |
| **Alternativa cryptography/dint-dev** → ¿Implementa RSA-OAEP?                                                                      | ❌ **No.** Solo RSA signing. No reemplaza a pointycastle para este caso                                                                                       |
| **flutter_secure_storage** → ¿Cifra claves privadas?                                                                               | ✅ RSA OAEP + AES-GCM en Android; Keychain en iOS                                                                                                             |
| **flutter_secure_storage** → ¿Funciona igual en Windows/Linux?                                                                     | ⚠️ Parcial. Windows usa Credential Manager; Linux libsecret (no headless)                                                                                     |
| **Consistencia cruzada de documentos** → ¿spec.md, steering.md, tasks.md y recommendations.md usan mismos algoritmos/terminología? | ✅ **100% consistente.** Los 4 documentos referencian AES-256-GCM, RSA-4096-OAEP, RSA-PSS, nonce 96 bits, SecureRandom, Fortuna. Sin contradicciones entre sí |

### Decisiones clave tomadas

| Decisión                    | Opción                                      | Por qué                                                            |
| --------------------------- | ------------------------------------------- | ------------------------------------------------------------------ |
| **Cifrado simétrico**       | AES-256-GCM (NO CBC)                        | CBC no es autenticado → padding oracle attacks                     |
| **Key wrapping**            | RSA-OAEP (NO PKCS#1 v1.5)                   | OAEP tiene seguridad demostrable; v1.5 vulnerable a Bleichenbacher |
| **Firma**                   | RSA-PSS (SHA-256)                           | Seguridad demostrable (provably secure)                            |
| **Longitud RSA**            | 4096 bits                                   | 128-bit security level, cumple estándares hasta 2030+              |
| **Almacenamiento privadas** | `flutter_secure_storage`                    | Keystore/Keychain nativo                                           |
| **Almacenamiento públicas** | Archivo local + hash SHA-256                | Las públicas no requieren cifrado                                  |
| **Nonce GCM**               | 96 bits (12 bytes)                          | Longitud estándar, mínima probabilidad de colisión                 |
| **Formato `.cryptodoc`**    | Magic bytes + version + algorithm + payload | Forward compatibility, anti-downgrade                              |

### Errores detectados en la spec original

1. **NFR1 ambiguo:** "CBC o GCM" — CBC no es autenticado sin HMAC adicional
2. **NFR1 incompleto:** No especificaba algoritmo de firma
3. **NFR5 incompleto:** No diferenciaba almacenamiento de claves públicas vs privadas
4. **Ausencia de key rotation:** Sin `version`/`algorithm_suite`, migrar algoritmos en el futuro rompe todos los documentos existentes
5. **Payload de firma indefinido:** Sin especificar qué se firma, un atacante podría hacer ciphertext-swapping

### Lecciones aprendidas

1. **No asumir que una librería criptográfica "moderna" cubre todos los casos.** `cryptography` (dint-dev) es superior en API/ergonomía, pero **no implementa RSA-OAEP** — justo lo que necesita la spec. Esto refuerza la necesidad de verificar el alcance de cada librería contra los requisitos exactos, no solo por su popularidad.

2. **El diablo está en el padding.** RSA tiene tres modos (PKCS#1 v1.5, OAEP, PSS) y elegir el incorrecto (v1.5 para cifrado) introduce vulnerabilidades conocidas. La spec debe ser explícita.

3. **El almacenamiento de claves no es "todo en secure_storage".** Las claves públicas no necesitan confidencialidad, y almacenarlas en `flutter_secure_storage` añade complejidad innecesaria y dependencia de plataforma. Separar por tipo de secreto es más limpio.

4. **La firma debe cubrir el payload completo.** Si la firma solo cubre el PDF cifrado, un atacante que intercepte el `.cryptodoc` puede reemplazar `wrapped_key` y redirigir el documento. Firmar `encrypted_pdf || wrapped_key || recipient_fingerprint` previene este ataque.

---

## Plantilla para futuras entradas

```markdown
## Entrada [NÚMERO] — [FECHA]

### Actividad

[Breve descripción de qué se hizo]

### Archivos afectados

| Archivo | Acción                            |
| ------- | --------------------------------- |
| ...     | ✅/❌ Creado/Modificado/Eliminado |

### Pruebas realizadas

| Prueba | Resultado |
| ------ | --------- |
| ...    | ✅/❌/⚠️  |

### Decisiones clave tomadas

[Decisiones importantes y su justificación]

### Errores cometidos

[Qué salió mal y cómo se solucionó]

### Aciertos

[Qué salió bien y merece repetirse]

### Lecciones aprendidas

[Qué conocimiento nuevo se generó]
```
