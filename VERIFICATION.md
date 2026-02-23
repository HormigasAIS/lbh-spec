# LBH Verification Guide — Guía de Auditoría Externa v1.0
Protocolo: Lenguaje Binario HormigasAIS (LBH)
Autor: Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Estado: Activo
Fecha: 2026-02-22

---

## Propósito

Este documento permite que cualquier auditor técnico externo reproduzca y verifique independientemente el protocolo LBH sin necesidad de acceso al hardware original ni de contactar al fundador.

---

## Requisitos

Python 3.10+
pip install cryptography
Dispositivo Android con Termux (verificación BLE opcional)

---

## Verificación 1 — Wire Format

Confirma que el formato de mensaje LBH es válido y que la firma es verificable.

import hmac, hashlib

def verify_lbh(message: str, shared_key: bytes) -> bool:
    try:
        parts = message.strip().split("|")
        if len(parts) != 3:
            return False

        payload = parts[0].replace("LBH_DATA:", "")
        ts      = parts[1].replace("TS:", "")
        sig_rx  = parts[2].replace("SIG:", "")

        content  = f"{payload}|{ts}".encode("utf-8")
        sig_calc = hmac.new(shared_key, content, hashlib.sha256).hexdigest()[:16]

        return hmac.compare_digest(sig_rx, sig_calc)
    except Exception:
        return False

# --- Test ---
KEY = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
msg = "LBH_DATA:INMUNIDAD_HONGO_ACTIVA|TS:1770805318|SIG:bba69499c9e516d1"

resultado = verify_lbh(msg, KEY)
print("Wire Format:", "VALIDO" if resultado else "INVALIDO")

Resultado esperado:
Wire Format: VALIDO

---

## Verificación 2 — Criptografía AES-256-GCM

Confirma que el cifrado y descifrado funcionan correctamente y que la modificación del ciphertext es detectada.

from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os, base64, hashlib

shared_key = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
aes_key    = hashlib.sha256(shared_key).digest()
payload    = b"FEROMONA_XOXO_ACTIVA"

# Cifrar
aesgcm = AESGCM(aes_key)
nonce  = os.urandom(12)
cipher = aesgcm.encrypt(nonce, payload, None)
token  = base64.b64encode(nonce + cipher).decode()

# Descifrar
raw       = base64.b64decode(token)
recovered = aesgcm.decrypt(raw[:12], raw[12:], None)

print("Cifrado/Descifrado:", "OK" if recovered == payload else "FAIL")

# Detección de tampering
try:
    corrupted = raw[:12] + bytes([raw[12] ^ 0xFF]) + raw[13:]
    aesgcm.decrypt(corrupted[:12], corrupted[12:], None)
    print("Tampering: FAIL")
except Exception:
    print("Tampering: OK (Deteccion exitosa)")

---

## Verificación 3 — Firma HMAC-SHA256

Confirma que la firma autentica mensajes válidos y rechaza mensajes alterados.

import hmac, hashlib

shared_key = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
payload    = b"LBH_STATUS:OPERATIVO_SOBERANO"

# Firmar
firma = hmac.new(shared_key, payload, hashlib.sha256).hexdigest()

# Verificar legítimo
valida = hmac.compare_digest(
    hmac.new(shared_key, payload, hashlib.sha256).hexdigest(),
    firma
)

print("Mensaje legitimo:", "ACEPTADO" if valida else "RECHAZADO")

# Verificar alterado
invalida = hmac.compare_digest(
    hmac.new(shared_key, b"LBH_STATUS:INTRUSO", hashlib.sha256).hexdigest(),
    firma
)

print("Mensaje alterado:", "ERROR" if invalida else "RECHAZADO correctamente")

---

## Verificación 4 — Lógica de Consenso BLE

El consenso BLE requiere hardware físico (A16 + A20s).

Un auditor puede verificar el algoritmo revisando:

Archivo: INCUBADORA_AIR_CITY/CENTINELA_V24_PILOTO.py
Linea 9:  OBJETIVO_BLE = "S9-DATA-IMMUNE-2026"
Linea 62: if MATCHES >= 2:
Linea 72: time.sleep(0.5)

Estos parámetros implementan el umbral anti-ruido y la frecuencia de 2 Hz.

---

## Verificación 5 — Implementación de Referencia (Zenodo)

DOI: 10.5281/zenodo.17767205
MD5 Checksum: 6c66a2f25f76abccf5c63b6971402db8

Para verificar integridad del archivo descargado:

md5sum HormigasAIS-LBH-Demo-Completa.zip

El hash calculado debe coincidir exactamente con el MD5 publicado.

---

## Resumen de Verificaciones

1  Wire Format        Script Python       No requiere hardware
2  AES-256-GCM        Script Python       No requiere hardware
3  HMAC-SHA256        Script Python       No requiere hardware
4  Consenso BLE       Revision de codigo  Hardware opcional
5  Referencia Zenodo  MD5 Checksum        No requiere hardware

---

© 2026 HormigasAIS
Cristhiam Leonardo Hernández Quiñonez
