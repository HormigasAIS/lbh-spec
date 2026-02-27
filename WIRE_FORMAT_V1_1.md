# LBH Wire Format — Especificación v1.1

**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autor:** Cristhiam Leonardo Hernández Quiñonez (CLHQ)
**Estado:** Draft — Rama v1.1-dev
**Fecha:** 2026-02-26
**Rama:** v1.1-dev

---

## Cambios respecto a v1.0

| Campo | v1.0 | v1.1 |
|-------|------|------|
| NODE_ID | No existia | Obligatorio |
| KEY_ID | No existia | Obligatorio |
| NONCE | No existia | Obligatorio 96 bits |
| HMAC | Truncado 16 chars | Completo 256 bits |
| Anti-replay | Solo timestamp | Timestamp + Nonce |
| Versionado | Sin campo VER | VER obligatorio |

---

## 1. Estructura del Mensaje v1.1

    LBH|VER:1|NODE:<node_id>|KEY:<key_id>|TS:<timestamp>|NONCE:<nonce>|DATA:<payload>|SIG:<signature>

### Campos obligatorios

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| VER | integer | Version del protocolo (actualmente 1) |
| NODE | string | Identificador unico del nodo emisor |
| KEY | integer | ID de la clave usada para firmar |
| TS | integer | Unix epoch en segundos (UTC) |
| NONCE | string hex | 96 bits aleatorios — generados por secrets.token_hex(12) |
| DATA | string UTF-8 | Payload del mensaje |
| SIG | string hex | HMAC-SHA256 completo (64 caracteres hex) |

---

## 2. Calculo de la Firma v1.1

El canonical string firmado incluye todos los campos en orden determinista:

    content = f"{VER}|{NODE}|{KEY}|{TS}|{NONCE}|{DATA}"
    SIG = HMAC-SHA256(content, shared_key).hexdigest()  # 64 chars completos

Diferencia critica con v1.0: la firma cubre NODE y KEY ademas del payload.
Esto previene suplantacion de identidad entre nodos.

---

## 3. Reglas Anti-Replay Formales

Un nodo LBH v1.1 rechaza un mensaje si cualquiera de estas condiciones se cumple:

    REGLA 1 — Ventana de tiempo:
    abs(time.time() - TS) > 300 segundos → BLOCKED

    REGLA 2 — Nonce duplicado:
    NONCE ya fue procesado dentro de la ventana activa → BLOCKED

    REGLA 3 — Nodo no autorizado:
    NODE no esta en la lista de nodos conocidos → BLOCKED

    REGLA 4 — Clave no autorizada:
    KEY no corresponde al NODE declarado → BLOCKED

    REGLA 5 — Firma invalida:
    SIG calculada != SIG recibida → BLOCKED

Todas las reglas se evaluan en orden. El sistema falla en la primera regla
que no se cumple sin revelar cual fue la razon al emisor.

---

## 4. Versionado de Clave

Cada nodo mantiene un diccionario de claves por version:

    node_keys = {
        1: b"CLAVE_ACTIVA_2026",
        2: b"CLAVE_ROTACION_2026_Q2"  # pre-cargada para rotacion
    }

El campo KEY_ID permite rotar claves sin interrumpir la operacion:
1. Se pre-distribuye la nueva clave con KEY_ID = 2
2. El emisor comienza a firmar con KEY_ID = 2
3. El receptor acepta ambas claves durante el periodo de transicion
4. Se revoca KEY_ID = 1 cuando todos los nodos migraron

---

## 5. Ejemplo Verificable v1.1

    NODE  = "A16-Soberano-Salvador"
    KEY   = 1
    TS    = 1770805318
    NONCE = "a3f1b2c4d5e6f7a8b9c0d1e2"
    DATA  = "INMUNIDAD_HONGO_ACTIVA"

    content = "1|A16-Soberano-Salvador|1|1770805318|a3f1b2c4d5e6f7a8b9c0d1e2|INMUNIDAD_HONGO_ACTIVA"
    SIG     = HMAC-SHA256(content, shared_key).hexdigest()

    Mensaje wire:
    LBH|VER:1|NODE:A16-Soberano-Salvador|KEY:1|TS:1770805318|NONCE:a3f1b2c4d5e6f7a8b9c0d1e2|DATA:INMUNIDAD_HONGO_ACTIVA|SIG:<64_hex_chars>

---

## 6. Compatibilidad con v1.0

Los nodos v1.1 pueden operar en modo compatibilidad detectando el prefijo:

    Si mensaje.startswith("LBH_DATA:") → procesar como v1.0
    Si mensaje.startswith("LBH|VER:")  → procesar como v1.1

Esto permite migracion gradual sin corte de servicio.

---

## 7. Implementacion de Referencia

La implementacion de referencia de LBH v1.1 esta en:

    core/lbh_node_v1_1.py — Motor de comunicacion soberana
    core/test_lbh_resilience.py — Suite de pruebas 5/5 vectores

Resultado verificado en produccion (2026-02-26):
    [TEST 1] Mensaje Legitimo:            PASADO
    [TEST 2] Replay Attack (Mismo Nonce): BLOQUEADO
    [TEST 3] Alteracion de Datos (Firma): BLOQUEADO
    [TEST 4] Firma con Clave Falsa:       BLOQUEADO
    [TEST 5] Mensaje Expirado (>300s):    BLOQUEADO

---

2026 HormigasAIS - Cristhiam Leonardo Hernandez Quinonez
Rama: v1.1-dev — No fusionar a main hasta completar auditoria interna
