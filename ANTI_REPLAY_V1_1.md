# LBH Anti-Replay Specification — v1.1

**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autor:** Cristhiam Leonardo Hernandez Quinonez (CLHQ)
**Estado:** Draft — Rama v1.1-dev
**Fecha:** 2026-02-26

---

## Proposito

Este documento define formalmente las reglas anti-replay del protocolo LBH v1.1.
Un ataque de replay ocurre cuando un mensaje valido es interceptado y reenviado
por un adversario para repetir una accion autorizada.

---

## Vectores de Ataque Cubiertos

| Vector | Descripcion | Mecanismo de Defensa |
|--------|-------------|----------------------|
| Replay simple | Mismo mensaje reenviado | Nonce unico por mensaje |
| Replay con delay | Mensaje valido enviado horas despues | Ventana de tiempo 300s |
| Suplantacion de nodo | Atacante firma con ID ajeno | NODE_ID vinculado a clave |
| Rotacion de nonce | Atacante genera nonces predecibles | secrets.token_hex(12) CSPRNG |

---

## Reglas Formales (Orden de Evaluacion)

    REGLA 1 — Formato
    El mensaje debe comenzar con "LBH|VER:1|"
    Si falla → BLOCKED (formato invalido)

    REGLA 2 — Version
    VER debe ser igual a PROTOCOL_VERSION del nodo receptor
    Si falla → BLOCKED (version incompatible)

    REGLA 3 — Nodo autorizado
    NODE debe existir en authorized_nodes del receptor
    Si falla → BLOCKED (nodo desconocido)

    REGLA 4 — Clave autorizada
    KEY debe existir en authorized_nodes[NODE]
    Si falla → BLOCKED (clave no registrada)

    REGLA 5 — Ventana de tiempo
    abs(time.time() - TS) <= 300 segundos
    Si falla → BLOCKED (mensaje expirado o del futuro)

    REGLA 6 — Nonce unico
    NONCE no debe haber sido procesado en la ventana activa
    Si pasa → registrar NONCE con timestamp actual
    Si falla → BLOCKED (replay detectado)

    REGLA 7 — Firma criptografica
    HMAC-SHA256(content, key) == SIG
    content = f"{VER}|{NODE}|{KEY}|{TS}|{NONCE}|{DATA}"
    Si falla → BLOCKED (firma invalida)

    Si todas las reglas pasan → ACCEPTED

---

## Implementacion del NonceStore

    class NonceStore:
        def __init__(self):
            self.seen = {}  # nonce -> timestamp

        def register(self, nonce: str) -> bool:
            now = time.time()

            # Limpiar nonces fuera de ventana
            expired = [n for n, ts in self.seen.items()
                      if now - ts > TIME_WINDOW_SECONDS]
            for n in expired:
                del self.seen[n]

            # Verificar duplicado
            if nonce in self.seen:
                return False  # Replay detectado

            # Registrar nonce nuevo
            self.seen[nonce] = now
            return True

---

## Generacion de Nonce

El nonce debe generarse con un CSPRNG (Cryptographically Secure
Pseudo-Random Number Generator):

    import secrets
    nonce = secrets.token_hex(12)  # 96 bits — 24 caracteres hex

No usar:
    random.random()     — No criptografico
    time.time()         — Predecible
    uuid.uuid4()        — Predecible en algunas implementaciones

---

## Limites y Consideraciones

| Parametro | Valor | Razon |
|-----------|-------|-------|
| Ventana de tiempo | 300 segundos | Balance entre latencia de red y seguridad |
| Tamano de nonce | 96 bits | Probabilidad de colision negligible en ventana |
| Almacen de nonces | En memoria | Suficiente para nodos edge de baja carga |
| Limpieza de nonces | Automatica al registrar | Sin proceso de fondo necesario |

Limitacion conocida: El NonceStore actual es en memoria.
Si el nodo se reinicia dentro de la ventana de 300s, nonces previos
se pierden y podrian ser reutilizados.
Solucion planificada v1.2: persistencia de NonceStore en archivo .lbh

---

## Evidencia de Prueba

Resultado verificado del TEST 2 en produccion (2026-02-26):

    msg_ok     = emisor.build_message("ESTADO:OPERATIVO", key_id=1)
    res_ok     = defensor.validate_message(msg_ok)      # True  — primer envio
    res_replay = defensor.validate_message(msg_ok)      # False — mismo nonce

    [TEST 2] Replay Attack (Mismo Nonce): BLOQUEADO

---

2026 HormigasAIS - Cristhiam Leonardo Hernandez Quinonez
Rama: v1.1-dev — No fusionar a main hasta completar auditoria interna
