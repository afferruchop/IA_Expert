# Spec 001: Constructor y Visor de PDFs Encriptados (v2.0)

**Estado:** Draft — Revisado por Architect  
**Responsable:** Architect + Builder  
**Fecha:** 2026-04-28

## Resumen

Aplicación Flutter multiplataforma para construir, cifrar, compartir y visualizar PDFs protegidos con criptografía asimétrica. El formato de intercambio es `.cryptodoc`, que encapsula el PDF cifrado con AES-256-GCM, la clave simétrica envuelta con RSA-OAEP, y la firma digital del emisor con RSA-PSS.

## Requisitos Funcionales (Actualizados)

- **FR1:** Permitir al usuario escribir/pegar texto y construir un PDF desde la app.
- **FR2:** Permitir seleccionar un destinatario (por clave pública) antes de cifrar.
- **FR3:** Exportar el archivo cifrado como `.cryptodoc`.
- **FR4:** El visor debe poder abrir `.cryptodoc` desde el sistema de archivos o por "compartir".
- **FR5:** El visor desencripta en memoria y muestra el PDF (nunca guarda el PDF en claro en disco).
- **FR6:** El usuario generará un par de claves (pública/privada) dentro de la app o importará una existente (RSA‑4096). La curva elíptica P‑256 se documenta como alternativa futura.
- **FR7:** Al construir un PDF, el contenido se cifrará con una clave simétrica aleatoria (AES-256-GCM). Dicha clave simétrica se cifrará con RSA-OAEP (SHA-256, MGF1) usando la clave pública del destinatario seleccionado (o la propia si es autoconsumo).
- **FR8:** El archivo resultante (`.cryptodoc`) incluirá: header con versión y algoritmo, PDF cifrado (AES-256-GCM), clave simétrica cifrada (RSA-OAEP), firma digital del emisor (RSA-PSS sobre el payload completo), clave pública del emisor con timestamp, y fingerprint del destinatario con verificación de checksum.
- **FR9:** El visor descifrará la clave simétrica usando la clave privada del destinatario, descifrará el PDF en memoria y verificará la firma del emisor para garantizar integridad y autenticidad.

## Requisitos No Funcionales (Actualizados)

- **NFR1:** Algoritmos:
  - Simétrico: **AES‑256 en modo GCM** (autenticado, nonce de 96 bits). **No se permite CBC.** El cifrado GCM proporciona autenticación implícita del ciphertext. El nonce será generado exclusivamente con `SecureRandom` de `pointycastle`.
  - Asimétrico: **RSA‑4096 con OAEP** (SHA-256, MGF1). **No se permite PKCS#1 v1.5 para cifrado.** ECDH P‑256 se documenta como alternativa futura pero no se implementa en v1.0.
  - Firma digital: **RSA‑PSS (SHA‑256)**. El payload firmado incluye: `encrypted_pdf || wrapped_key || recipient_fingerprint || metadata_header`.
  - Hash: SHA‑256.
- **NFR2:** La aplicación debe funcionar offline sin conexión a internet.
- **NFR3:** Los archivos generados deben poder intercambiarse por cualquier medio (email, USB, nube, etc.).
- **NFR4:** El tiempo de cifrado/descifrado no debe exceder los 5 segundos en dispositivos móviles modernos (RSA-4096 puede tomar más en generación de claves; aceptable hasta 30 segundos para generación).
- **NFR5:** La generación de claves debe ser local, nunca transmitida. Las **claves privadas** se almacenan en `flutter_secure_storage`. Las **claves públicas** se almacenan en archivo local (`path_provider`) con HMAC-SHA256 de integridad usando una clave derivada (PBKDF2 con salt de 32 bytes) de la clave privada correspondiente. Las claves públicas incluirán timestamp de creación para auditoria.
- **NFR6:** La aplicación debe mostrar un indicador visual claro (icono verde/rojo) sobre el estado de la firma digital. Si la firma es inválida, el documento no se descifrará y se mostrará un mensaje de advertencia. Si la firma es válida, se mostrará la identidad del emisor (fingerprint o alias asociado).
- **NFR7:** El formato `.cryptodoc` incluirá campos `version` y `algorithm_suite` para soportar rotación de claves y migración futura de algoritmos. El visor rechazará versiones no soportadas.
- **NFR8:** La aplicación permitirá exportar la clave privada cifrada con passphrase (PBKDF2-HMAC-SHA256 con 600,000 iteraciones mínimo) para migración entre dispositivos, con confirmación explícita del usuario. La exportación incluirá metadata de creación y un identificador único de la clave para facilitar la gestión multiplataforma.
- **NFR9:** Se usará `SecureRandom` de `pointycastle` (Fortuna/CTR_DRBG) para toda generación de material criptográfico. Prohibido usar `math.Random()`.

## Stack Tecnológico (Actualizado)

- `pointycastle: ^3.7.3` para primitivas criptográficas (RSA, AES-GCM, OAEP, PSS).
- `flutter_secure_storage: ^9.2.2` para claves privadas locales.
- `asn1lib: ^1.5.0` para codificación/decodificación de claves en formato DER.
- `path_provider: ^2.1.0` para almacenamiento de claves públicas.
- `crypto: ^3.0.3` para utilidades hash complementarias.
- El resto igual (pdf/pdf_viewer, file_picker, etc.).

## Restricciones de Seguridad

1. Prohibido usar `math.Random()` — siempre `SecureRandom` de pointycastle (Fortuna/CTR_DRBG).
2. Prohibido loguear claves privadas, semillas, nonces, o datos descifrados (incluyendo `print`, `debugPrint`, o excepciones).
3. Prohibido almacenar el PDF en claro en disco (solo en memoria volátil).
4. Prohibido aceptar claves RSA < 2048 bits (rechazar en importación). Longitud recomendada: 4096 bits.
5. Obligatorio verificar firma digital **antes** de descifrar el PDF (fail-fast). Si la firma es inválida, el documento no se descifrará bajo ninguna circunstancia.
6. Prohibido exportar la clave privada sin confirmación explícita del usuario mediante un diálogo de advertencia.
7. El payload firmado debe incluir obligatoriamente: `encrypted_pdf || wrapped_key || recipient_fingerprint || metadata_header`.
8. El nonce de AES-GCM debe ser de exactamente 96 bits (12 bytes), generado con SecureRandom.
