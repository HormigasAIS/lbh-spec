# LBH Upgrade Guide — Migracion v1.0 a v1.1

**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autor:** Cristhiam Leonardo Hernandez Quinonez (CLHQ)
**Estado:** Draft — Rama v1.1-dev
**Fecha:** 2026-02-26

---

## Resumen de Cambios

| Aspecto | v1.0 | v1.1 |
|---------|------|------|
| Formato wire | LBH_DATA:|TS:|SIG: | LBH|VER:|NODE:|KEY:|TS:|NONCE:|DATA:|SIG: |
| HMAC | Truncado 16 chars (64 bits) | Completo 64 chars (256 bits) |
| Anti-replay | Solo ventana de tiempo | Ventana + Nonce unico |
| Identidad | Sin NODE_ID | NODE_ID obligatorio |
| Claves | Una shared_key global | Dict por KEY_ID por nodo |
| Revocacion | No existe | CRL-LBH con rechazo inmediato |
| Compatibilidad | Solo v1.0 | Detecta y procesa v1.0 y v1.1 |

---

## Deteccion Automatica de Version

    def detect_version(message: str) -> int:
        if message.startswith("LBH_DATA:"):
            return 10   # v1.0
        elif message.startswith("LBH|VER:"):
            return 11   # v1.1
        else:
            return -1   # Desconocido — rechazar

    def route_message(message: str, node, legacy_key: bytes) -> bool:
        version = detect_version(message)
        if version == 10:
            return verify_lbh_v10(message, legacy_key)
        elif version == 11:
            return node.validate_message(message)
        return False

---

## Pasos de Migracion

    PASO 1 — Instalar lbh_node_v1_1.py
    Copiar core/lbh_node_v1_1.py al directorio core de cada nodo.
    No eliminar el verificador v1.0 todavia.

    PASO 2 — Asignar NODE_ID a cada nodo
    A16  → "A16-Soberano-Salvador"
    A20s → "A20s-Manager-Alpha"
    Formato: "<MODELO>-<ROL>-<UBICACION>"

    PASO 3 — Generar KEY_ID=1 para cada nodo
    Distribuir por canal seguro fuera de banda.
    Registrar en authorized_nodes de cada receptor.
    Solo el LBH-MASTER autoriza esta distribucion.

    PASO 4 — Periodo de transicion dual (7 dias)
    Los nodos aceptan mensajes v1.0 y v1.1 simultaneamente.
    Monitorear que todos los emisores migraron a v1.1.

    PASO 5 — Corte de v1.0
    El LBH-MASTER confirma migracion completa.
    Deshabilitar procesamiento de mensajes v1.0.
    Registrar fecha de corte en GOVERNANCE.md.

---

## Verificacion Post-Migracion

Ejecutar la suite de resiliencia en cada nodo migrado:

    python core/test_lbh_resilience.py

Resultado esperado: 5/5 PASADO o BLOQUEADO segun corresponda.
Si algun test falla, el nodo NO debe entrar en produccion.

---

## Rollback

Si la migracion falla, revertir al estado estable:

    git checkout v1.0.0

El tag v1.0.0 en main garantiza punto de retorno seguro en todo momento.

---

## Compatibilidad de Hardware

| Dispositivo | Compatible | Notas |
|-------------|-----------|-------|
| A16 Samsung | Si | Nodo Maestro — Python 3.10+ Termux |
| A20s Samsung | Si | Nodo Periferico — mismo requisito |
| Raspberry Pi | Si | Migracion directa sin cambios |
| Linux x86 | Si | Entorno de referencia para auditoria |
| Windows | Si | Requiere Python 3.10+ |

---

## Estado de la Rama v1.1-dev

Documentos completados:

    WIRE_FORMAT_V1_1.md     — Formato con NODE_ID, KEY_ID, NONCE
    ANTI_REPLAY_V1_1.md     — 7 reglas formales anti-replay
    KEY_ROTATION_V1_1.md    — Versionado, CRL-LBH, autoridad
    UPGRADE_GUIDE_V1_1.md   — Esta guia

Pendiente antes de merge a main:
    Video BLE A16-A20s — consenso fisico demostrado
    Auditoria interna de los 4 documentos
    Tag v1.1.0

---

2026 HormigasAIS - Cristhiam Leonardo Hernandez Quinonez
Rama: v1.1-dev — No fusionar a main hasta completar auditoria interna
