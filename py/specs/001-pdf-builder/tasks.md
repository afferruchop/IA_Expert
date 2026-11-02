# Tasks: Spec 001 — v2.0 (Revisado por Architect)

## Fase 1 — Primitivas Criptográficas

- [ ] 1. Configurar proyecto Flutter con `flutter create cryptodoc`.
- [ ] 2.1 Implementar clase `CryptoSymmetric`: AES-256-GCM con nonce de 96 bits (pointycastle).
- [ ] 2.2 Implementar clase `CryptoAsymmetric`: RSA-4096 key generation (FortunaRandom).
- [ ] 2.3 Implementar RSA-OAEP key wrapping/unwrapping (SHA-256, MGF1).
- [ ] 2.4 Implementar RSA-PSS signing/verification.
- [ ] 2.5 Implementar import/export de claves (DER/PEM usando asn1lib).

## Fase 2 — Formato .cryptodoc

- [ ] 3.1 Definir estructura binaria: header (magic, version, algorithm), payload cifrado.
- [ ] 3.2 Implementar serialización/deserialización del formato.
- [ ] 3.3 Validar integridad del header (rechazar versiones no soportadas).

## Fase 3 — Almacenamiento Seguro

- [ ] 4.1 Implementar almacenamiento de clave privada en `flutter_secure_storage`.
- [ ] 4.2 Implementar almacenamiento de claves públicas en archivo local + hash SHA-256.
- [ ] 4.3 Implementar directorio de contactos (key ring local con alias).

## Fase 4 — UI de Gestión de Claves

- [ ] 5.1 Pantalla "Mis claves": generar, importar (PEM/DER), exportar cifrada con passphrase.
- [ ] 5.2 Pantalla "Contactos": importar clave pública, asignar alias, mostrar fingerprint.
- [ ] 5.3 Indicador visual de estado de firma (verde/válida, rojo/inválida).

## Fase 5 — Cifrado y Visor

- [ ] 6. Integrar cifrado asimétrico en el builder de PDF.
- [ ] 7. Integrar descifrado + verificación de firma en el visor.
- [ ] 8. Implementar funcionalidad de compartir (exportar .cryptodoc).

## Fase 6 — Tests y Seguridad

- [ ] 9.1 Tests unitarios: vectores de test conocidos para AES-GCM, RSA-OAEP, RSA-PSS.
- [ ] 9.2 Tests de integración: ciclo completo (generar claves → cifrar → descifrar → verificar).
- [ ] 9.3 Tests de seguridad: claves inválidas, firma manipulada, formato corrupto.
- [ ] 9.4 Revisión de seguridad: escanear fugas de claves en logs, verificar SecureRandom.
- [ ] 9.5 Pruebas multiplataforma: Android, iOS, Windows, Linux, macOS.
