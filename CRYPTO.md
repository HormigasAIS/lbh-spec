# LBH Cryptography — Especificación v1.0
Protocolo: Lenguaje Binario HormigasAIS (LBH)
Autor: Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Estado: Draft Auditable
Fecha: 2026-02-22

---

## 1. Visión General

LBH implementa dos operaciones criptográficas distintas:

| Operación | Algoritmo     | Propósito                          |
|-----------|--------------|------------------------------------|
| Cifrado   | AES-256-GCM  | Confidencialidad del payload       |
| Firma     | HMAC-SHA256  | Autenticidad e integridad mensaje  |

Ambas operaciones están implementadas en crypto/lbh_crypto.py usando la librería cryptography de Python (PyCA).

---

## 2. Derivación de Clave

A partir de una shared_key distribuida, LBH deriva la clave AES mediante SHA-256:

self.aes_key = hashlib.sha256(shared_key).digest()

Esto produce 32 bytes (256 bits) compatibles con AES-256.

Motivación:
SHA-256 garantiza longitud fija, determinismo y amplia auditabilidad.

---

## 3. Cifrado — AES-256-GCM

Implementación conceptual:

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os, base64

def cifrar(self, payload_bytes: bytes) -> str:
    aesgcm = AESGCM(self.aes_key)
    nonce  = os.urandom(12)              # 96 bits recomendado por NIST
    cipher = aesgcm.encrypt(nonce, payload_bytes, None)
    return base64.b64encode(nonce + cipher).decode()

Formato del token cifrado:

base64(
    nonce[12 bytes] +
    ciphertext[n bytes] +
    tag[16 bytes]
)

Propiedades:

Confidencialidad:
El payload es ilegible sin la clave AES.

Integridad:
Cualquier modificación del ciphertext invalida el tag GCM.

Autenticidad:
Solo poseedores de la clave pueden generar tokens válidos.

---

## 4. Descifrado

El receptor:

1. Decodifica base64.
2. Extrae los primeros 12 bytes como nonce.
3. Usa AESGCM.decrypt().
4. Si el tag es inválido, se lanza InvalidTag.

No existe decodificación silenciosa de datos corruptos.

---

## 5. Firma — HMAC-SHA256

import hmac, hashlib

def firmar(self, payload_bytes: bytes) -> str:
    return hmac.new(self.shared_key, payload_bytes, hashlib.sha256).hexdigest()

def verificar_firma(self, payload_bytes: bytes, firma_recibida: str) -> bool:
    esperada = self.firmar(payload_bytes)
    return hmac.compare_digest(esperada, firma_recibida)

Nota crítica:
hmac.compare_digest previene ataques de timing side-channel.

---

## 6. Relación con Wire Format

El Wire Format LBH integra estas capas:

LBH_DATA:<payload>|TS:<timestamp>|SIG:<hmac_sha256>

Flujo:

1. Payload original → cifrar() → Token AES-GCM.
2. Mensaje completo → firmar() → Campo SIG.

NOTA:
La versión actual del wire format no debería truncar el HMAC.
Se recomienda usar el SHA256 completo (64 hex chars).

---

## 7. Stack Criptográfico Completo

Librería: cryptography (PyCA)
Cifrado: AES-256-GCM (AEAD)
Firma: HMAC-SHA256
Nonce: os.urandom(12) (CSPRNG del sistema)
Comparación segura: hmac.compare_digest

---

## 8. Limitaciones Conocidas (v1.0)

Shared key fuera de banda:
Requiere canal seguro inicial.

Derivación simple:
Migración planificada a HKDF (RFC 5869) en v1.1.

Sin Perfect Forward Secrecy:
Compromiso de clave afecta sesiones pasadas.
ECDH planificado para v1.2.

---

## 9. Verificación Independiente (Test Básico)

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os, base64, hashlib, hmac

shared_key = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
aes_key    = hashlib.sha256(shared_key).digest()

aesgcm = AESGCM(aes_key)

payload = b"LBH_TEST"
nonce   = os.urandom(12)
cipher  = aesgcm.encrypt(nonce, payload, None)
recovered = aesgcm.decrypt(nonce, cipher, None)

firma = hmac.new(shared_key, payload, hashlib.sha256).hexdigest()

# Test válido si:
# recovered == payload
# y la firma es verificable mediante compare_digest

---

Este documento describe la capa criptográfica del protocolo LBH.

© 2026 HormigasAIS
Cristhiam Leonardo Hernández Quiñonez
