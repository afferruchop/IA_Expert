# Steering Agent: cryptodoc

**Objetivo:** Gobernar el comportamiento de los agentes durante el desarrollo de la Spec 001.

## Hooks de Comportamiento

### Pre-Tarea (before_task)

1. Leer `specs/001-pdf-builder/spec.md`.
2. Verificar que la tarea a ejecutar está listada en `tasks.md`.
3. Si requiere cambios en specs, detenerse y notificar al Architect.

### Durante la Implementación (during_code)

- Coder debe seguir el style guide de Flutter (analysis_options.yaml con `flutter_lints`).
- No usar dependencias obsoletas o inseguras (verificar con `pub outdated`).
- Cada widget debe tener su correspondiente widget test.
- **Usar exclusivamente AES-256-GCM** (no CBC). Si se usa CBC, debe ir acompañado de HMAC-SHA256.
- **Usar exclusivamente RSA-OAEP** (SHA-256, MGF1) para key wrapping. No usar PKCS#1 v1.5.
- **Usar RSA-PSS** (SHA-256) para firmas digitales.
- Al importar claves RSA, rechazar si < 2048 bits.
- Las claves públicas almacenar en archivo local (no en flutter_secure_storage) con HMAC-SHA256 para integridad.
- Incluir timestamp de creación en metadatos de claves para auditoria y rotación.

### Pre-Commit (pre_commit)

- Reviewer (`gga`) analizará:
  - Vulnerabilidades de seguridad (claves expuestas, logs en producción).
  - Desviaciones de la spec (incluyendo algoritmos correctos).
  - Cobertura mínima de tests en el código modificado.
  - Verificar que el payload firmado incluye: encrypted_pdf || wrapped_key || recipient_fingerprint.
- Si el commit no pasa, el Builder debe revertir y solicitar instrucciones al Architect.

## Personalidad de los Agentes

- **Architect:** Analítico, exhaustivo, orientado a la seguridad y escalabilidad.
- **Coder:** Preciso, limpio, adherido a principios SOLID y Clean Architecture.
- **Builder:** Eficiente, sin miedo a refactorizar, orientado a la integración continua.
- **Reviewer:** Estricto, minimalista, enfocado en la calidad y la seguridad.

## Hooks de Seguridad Adicionales (Criptografía)

### Durante la implementación

- Nunca usar generadores de números aleatorios no criptográficos (`math.Random`). Siempre `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG).
- La clave privada nunca deberá aparecer en logs, UI, debugPrint, ni en excepciones.
- Validar que las longitudes de clave cumplan los estándares (RSA ≥ 2048, preferible 4096; rechazar si < 2048).
- El PDF descifrado solo debe existir en memoria volátil. Nunca persistir en disco.
- Asegurar que el nonce de AES-GCM sea de 96 bits (12 bytes) generado con SecureRandom.

### Pre-Commit (gga)

- Escanear en busca de `print`, `debugPrint` que puedan filtrar claves o datos sensibles.
- Verificar que los tests incluyan casos con pares de claves válidos e inválidos.
- Verificar que el formato `.cryptodoc` incluye campo `version` y `algorithm_suite`.
- Verificar que la verificación de firma ocurre antes del descifrado (fail-fast).
- Validar que las claves públicas incluyen timestamp y HMAC de integridad.
- Verificar que la gestión de claves soporta identificación única entre dispositivos.
