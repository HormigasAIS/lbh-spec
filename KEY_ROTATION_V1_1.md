# LBH Key Rotation Specification — v1.1

**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autor:** Cristhiam Leonardo Hernandez Quinonez (CLHQ)
**Estado:** Draft — Rama v1.1-dev
**Fecha:** 2026-02-26

---

## Proposito

Este documento define el mecanismo de rotacion de claves del protocolo LBH v1.1.
La rotacion de claves permite reemplazar claves comprometidas o expiradas sin
interrumpir la operacion del enjambre.

---

## Problema que Resuelve

En LBH v1.0 existe una sola shared_key global. Si esa clave se compromete:
- Todos los mensajes historicos son vulnerables
- No hay forma de revocar acceso a un nodo especifico
- El sistema debe detenerse para redistribuir la clave

LBH v1.1 resuelve esto con KEY_ID y versionado de claves por nodo.

---

## Estructura de Claves v1.1

Cada nodo mantiene un diccionario de claves indexado por KEY_ID:

    node_keys = {
        1: b"CLAVE_ACTIVA_2026_Q1",       # Clave actual
        2: b"CLAVE_ROTACION_2026_Q2"      # Pre-cargada para transicion
    }

El campo KEY en el mensaje wire indica cual clave uso el emisor para firmar.
El receptor valida con la clave correspondiente a ese KEY_ID.

---

## Proceso de Rotacion Sin Corte

    FASE 1 — Pre-distribucion (sin impacto operativo)
    El LBH-MASTER distribuye KEY_ID=2 a todos los nodos autorizados
    por canal seguro fuera de banda.
    Los nodos siguen usando KEY_ID=1 para firmar.
    Los receptores ya conocen KEY_ID=2 pero no lo usan aun.

    FASE 2 — Migracion gradual
    El LBH-MASTER indica inicio de migracion.
    Los emisores comienzan a firmar con KEY_ID=2.
    Los receptores aceptan AMBOS KEY_ID=1 y KEY_ID=2.
    Periodo de transicion recomendado: 24 horas.

    FASE 3 — Revocacion
    El LBH-MASTER confirma que todos los nodos migraron a KEY_ID=2.
    Se elimina KEY_ID=1 de todos los nodos.
    Solo KEY_ID=2 es valido a partir de este momento.

---

## Autoridad de Rotacion

Solo el LBH-MASTER (Cristhiam Leonardo Hernandez Quinonez) tiene autoridad
para iniciar una rotacion de claves. Esto esta definido en GOVERNANCE.md:

    LBH-MASTER (CLHQ): Unica entidad con autoridad para modificar
    el nucleo del protocolo y las llaves maestras de cifrado.

Ninguna rotacion automatica es valida sin validacion manual del LBH-MASTER.
Esto protege contra ataques de secuestro de clave automatizado.

---

## Escenarios de Rotacion

| Escenario | Accion | Urgencia |
|-----------|--------|----------|
| Clave comprometida | Rotacion de emergencia inmediata | Alta |
| Nodo dado de baja | Revocar KEY_ID de ese nodo | Media |
| Rotacion programada | Rotacion planificada trimestral | Baja |
| Nuevo nodo agregado | Distribuir claves activas al nuevo nodo | Media |

---

## Implementacion de Referencia

    class LBHNode:
        def __init__(self, node_id, secret_keys, authorized_nodes):
            self.node_id        = node_id
            self.secret_keys    = secret_keys        # propias
            self.authorized_nodes = authorized_nodes  # de otros nodos

        def rotate_key(self, node_id, old_key_id, new_key_id, new_key):
            # Solo el LBH-MASTER ejecuta esta funcion
            if node_id in self.authorized_nodes:
                self.authorized_nodes[node_id][new_key_id] = new_key
                # old_key_id se mantiene durante periodo de transicion
                print(f"Clave {new_key_id} registrada para {node_id}")

        def revoke_key(self, node_id, key_id):
            if node_id in self.authorized_nodes:
                if key_id in self.authorized_nodes[node_id]:
                    del self.authorized_nodes[node_id][key_id]
                    print(f"Clave {key_id} revocada para {node_id}")

---

## Limitaciones Conocidas v1.1

| Limitacion | Impacto | Plan |
|------------|---------|------|
| Distribucion fuera de banda | Requiere canal seguro manual | HKDF automatico en v1.2 |
| NonceStore no persiste | Reinicio dentro de 300s vulnerable | Persistencia en v1.2 |
| Sin Perfect Forward Secrecy | Clave comprometida expone historial | ECDH efimero en v1.2 |

---

2026 HormigasAIS - Cristhiam Leonardo Hernandez Quinonez
Rama: v1.1-dev — No fusionar a main hasta completar auditoria interna

---

## Apendice — Lista de Revocacion (CRL-LBH)

Cuando una clave es comprometida, el LBH-MASTER emite una entrada
en la lista de revocacion:

    REVOKED_KEYS = {
        "A16-Soberano-Salvador": {
            1: {
                "revoked_at": 1770805318,
                "reason": "Compromiso de clave",
                "revoked_by": "LBH-MASTER-CLHQ"
            }
        }
    }

Un nodo que recibe un mensaje firmado con KEY_ID revocado lo rechaza
inmediatamente sin importar que la firma sea matematicamente valida.

Politica de rechazo inmediato:
    if key_id in REVOKED_KEYS.get(node_id, {}):
        return False  # Clave revocada — BLOCKED

---

## Nota sobre Modelo de Autoridad

LBH es jerarquico por diseno consciente, no distribuido puro.
La autoridad del LBH-MASTER sobre rotacion de claves es una decision
deliberada para infraestructura soberana donde existe responsabilidad
humana explicita sobre cada nodo.

Esto no impide federacion futura — pero en v1.1 la autoridad de
rotacion reside en el LBH-MASTER.
