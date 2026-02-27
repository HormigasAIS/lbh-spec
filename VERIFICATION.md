# LBH Verification Guide — Guía de Auditoría Externa v1.1

**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autor:** Cristhiam Leonardo Hernández Quiñonez (CLHQ)
**Estado:** Activo | Nivel de Confianza: Soberano
**Fecha:** 2026-02-26

---

## Propósito

Este documento permite la reproducción y verificación independiente del protocolo LBH. Asegura que la **Soberanía de Datos** y la **Resiliencia** son propiedades matemáticas del código, no dependientes de la infraestructura de terceros.

---

## 1. Requisitos de Auditoría

- **Python 3.10+**
- **Librerias:** pip install cryptography
- **Entorno:** Linux / Termux / MacOS / Windows

---

## 2. Verificacion de Formato de Mensaje (Wire Format v1.0)

Valida que la estructura LBH_DATA|TS|SIG es procesable y que el truncamiento de firma a 16 caracteres hex es consistente.

    import hmac, hashlib

    def verify_lbh(message: str, shared_key: bytes) -> bool:
        try:
            parts   = message.strip().split("|")
            payload = parts[0].replace("LBH_DATA:", "")
            ts      = parts[1].replace("TS:", "")
            sig_rx  = parts[2].replace("SIG:", "")
            content = f"{payload}|{ts}".encode("utf-8")
            sig_calc = hmac.new(shared_key, content, hashlib.sha256).hexdigest()[:16]
            return hmac.compare_digest(sig_rx, sig_calc)
        except:
            return False

    KEY = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
    msg = "LBH_DATA:INMUNIDAD_HONGO_ACTIVA|TS:1770805318|SIG:bba69499c9e516d1"
    print(f"Wire Format Check: {'VALIDO' if verify_lbh(msg, KEY) else 'INVALIDO'}")

Resultado esperado: Wire Format Check: VALIDO

---

## 3. Verificacion de Capa de Cifrado (AES-256-GCM)

HormigasAIS utiliza cifrado autenticado para evitar ataques de bit-flipping en el transporte.

    from cryptography.hazmat.primitives.ciphers.aead import AESGCM
    import os, hashlib

    try:
        shared_key = b"LBH_SHARED_SECRET_32BYTES!!!!!!"
        aes_key    = hashlib.sha256(shared_key).digest()
        payload    = b"FEROMONA_XOXO_ACTIVA"
        aesgcm     = AESGCM(aes_key)
        nonce      = os.urandom(12)

        token_raw  = aesgcm.encrypt(nonce, payload, None)
        recovered  = aesgcm.decrypt(nonce, token_raw, None)
        print(f"Cifrado/Descifrado: {'OK' if recovered == payload else 'FAIL'}")

        corrupted = token_raw[:-1] + bytes([token_raw[-1] ^ 0xFF])
        try:
            aesgcm.decrypt(nonce, corrupted, None)
            print("Deteccion de Tampering: FALLO")
        except:
            print("Deteccion de Tampering: EXITOSA (Tag invalido)")

    except ImportError:
        print("Instale cryptography para ejecutar esta prueba.")

Resultado esperado:
    Cifrado/Descifrado: OK
    Deteccion de Tampering: EXITOSA (Tag invalido)

---

## 4. Suite de Resiliencia v1.1 (Anti-Replay y Logic)

Prueba de 5 vectores contra el motor soberano. Requiere core/lbh_node_v1_1.py del repositorio principal.

    import time, sys, os
    sys.path.append(os.path.expanduser("~/HormigasAIS-Nodo-Escuela/core"))
    from lbh_node_v1_1 import LBHNode

    shared_secret = b"MASTER_KEY_LBH_2026_CLHQ_Soberania"
    node_a_id     = "A16-Soberano-Salvador"
    defensor      = LBHNode("A20s-Manager", {}, {node_a_id: {1: shared_secret}})
    emisor        = LBHNode(node_a_id, {1: shared_secret}, {})

    msg_ok     = emisor.build_message("ESTADO:OPERATIVO", key_id=1)
    res_ok     = defensor.validate_message(msg_ok)
    print(f"[TEST 1] Mensaje Legitimo:            {'PASADO'    if res_ok        else 'FALLADO'}")

    res_replay = defensor.validate_message(msg_ok)
    print(f"[TEST 2] Replay Attack (Mismo Nonce): {'BLOQUEADO' if not res_replay else 'VULNERABLE'}")

    msg_tampered = msg_ok.replace("DATA:ESTADO:OPERATIVO", "DATA:ESTADO:APAGAR")
    res_tamper   = defensor.validate_message(msg_tampered)
    print(f"[TEST 3] Alteracion de Datos (Firma): {'BLOQUEADO' if not res_tamper else 'VULNERABLE'}")

    hacker_node = LBHNode(node_a_id, {1: b"CLAVE_FALSA_HACKER_12345"}, {})
    msg_hacker  = hacker_node.build_message("ESTADO:OPERATIVO", key_id=1)
    res_hacker  = defensor.validate_message(msg_hacker)
    print(f"[TEST 4] Firma con Clave Falsa:       {'BLOQUEADO' if not res_hacker else 'VULNERABLE'}")

    msg_old = msg_ok.replace(f"TS:{int(time.time())}", "TS:1600000000")
    res_old = defensor.validate_message(msg_old)
    print(f"[TEST 5] Mensaje Expirado (>300s):    {'BLOQUEADO' if not res_old   else 'VULNERABLE'}")

Resultado esperado verificado en produccion (2026-02-26):
    [TEST 1] Mensaje Legitimo:            PASADO
    [TEST 2] Replay Attack (Mismo Nonce): BLOQUEADO
    [TEST 3] Alteracion de Datos (Firma): BLOQUEADO
    [TEST 4] Firma con Clave Falsa:       BLOQUEADO
    [TEST 5] Mensaje Expirado (>300s):    BLOQUEADO

---

## 5. Verificacion de Hardware y Consenso (BLE)

Para auditoria fisica en El Salvador:

- Target: S9-DATA-IMMUNE-2026
- Umbral: MATCHES >= 2 (logica de mayoria para filtrado de ruido)
- Frecuencia: 0.5s (balance optimo entre deteccion y ahorro de bateria)

Archivo de referencia: INCUBADORA_AIR_CITY/CENTINELA_V24_PILOTO.py

---

## 6. Integridad de Referencia (Zenodo/MD5)

Verifique que su copia de trabajo coincide con el release oficial:

- DOI: 10.5281/zenodo.17767205
- MD5: 6c66a2f25f76abccf5c63b6971402db8

    md5sum HormigasAIS-LBH-Demo-Completa.zip

El hash calculado debe coincidir exactamente con el MD5 publicado.

---

## Resumen de Verificaciones

    1  Wire Format v1.0         Script Python         No requiere hardware
    2  AES-256-GCM              Script Python         No requiere hardware
    3  HMAC-SHA256              Script Python         No requiere hardware
    4  Suite Resiliencia v1.1   5/5 vectores          No requiere hardware
    5  Consenso BLE             Revision de codigo    Hardware opcional
    6  Referencia Zenodo        MD5 Checksum          No requiere hardware

---

2026 HormigasAIS - Infraestructura de Inteligencia Distribuida y Soberana.
Fundador: Cristhiam Leonardo Hernandez Quinonez.
