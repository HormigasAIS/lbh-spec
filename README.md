# LBH Protocol Specification
## Lenguaje Binario HormigasAIS — Especificación técnica formal y auditable

<p align="center">
  <img src="https://raw.githubusercontent.com/Thrumanshow/Thrumanshow/main/logo_actualizado.svg" width="300" alt="HormigasAIS Logo">
  <br>
  <em>Digital Pheromones — Distributed Sovereignty</em>
</p>

---

## ¿Qué es LBH?

LBH (Lenguaje Binario HormigasAIS) es un protocolo de comunicación diseñado para coordinar nodos edge distribuidos con las siguientes propiedades:

- **Soberanía local:** Cada nodo valida mensajes sin depender de servicios externos.
- **Operación offline:** Funciona sin Internet activo.
- **Validación criptográfica:** Cada mensaje incluye firma HMAC-SHA256.
- **Eficiencia estructural:** Mensajes en texto plano con formato determinístico.
- **Consenso distribuido:** Quórum tipo Raft entre nodos del enjambre.

---

## 🛡️ Inmunidad Digital (v1.1.1 - Stable)

La versión actual del protocolo incluye protecciones avanzadas contra ataques de red:

- **Anti-Replay:** Implementación de Nonces de 96-bit y ventanas de tiempo (Time Windows).
- **Identidad de Nodo:** Cada mensaje incluye un `NODE_ID` único vinculado a hardware.
- **Router de Convivencia:** Capacidad de rollback dinámico entre v1.0 y v1.1.

> [!TIP]
> Puedes ver el estado de salud de la colonia en tiempo real aquí: [DASHBOARD_RESILIENCE.md](./DASHBOARD_RESILIENCE.md)

---

## Documentos de Especificación

| Documento | Descripción | Estado |
|------------|-------------|--------|
| WIRE_FORMAT.md | Formato de mensaje, firma y validación | v1.1 |
| CONSENSUS.md | Consenso BLE físico + Raft lógico | v1.0 |
| CRYPTO.md | AES-256-GCM + HMAC-SHA256 + Nonces | v1.1 |
| GOVERNANCE.md | Contratos LBH y gobernanza soberana | v1.0 |
| VERIFICATION.md | Guía de auditoría externa y Router | v1.1 |

---

## Inicio Rápido — Verificar un Mensaje LBH (v1.1)

```python
import hmac
import hashlib

def verify_lbh_v11(message: str, shared_key: bytes) -> bool:
    try:
        # Formato:
        # LBH|VER:11|ID:NODE_A|KEY:1|NONCE:XYZ|DATA:PAYLOAD|SIG:HEX

        parts = message.strip().split("|")

        # Validación mínima de estructura
        if len(parts) < 6:
            return False

        # Verificar versión
        if parts[1] != "VER:11":
            return False

        # (Ver implementación completa en core/lbh_router.py)
        return True

    except Exception:
        return False

---

## Arquitectura de Enjambre 

[Nodo Manager Alpha] <-- XOXO-BUS (LBH v1.1) --> [Nodo Escuela A16]
        |                                      |
        +----------- LBH Protocol -------------+

(Resiliencia Distribuida — El Salvador / Nicaragua)


---

## Implementación de Referencia 

La implementación oficial está registrada en Zenodo:
DOI: 10.5281/zenodo.17767205

---

## Historial 

v1.1.1 (2026-02-26): Corrección de release, blindaje de repositorio (.gitignore) y rama stable.

v1.1.0 (2026-02-26): Implementación de Anti-Replay, Nonces y LBH Router.
v1.0.0 (2026-02-22): Especificación inicial auditable.

---

## Fundador 

Cristhiam Leonardo Hernández Quiñonez (CLHQ) Organización: HormigasAIS — El Salvador / Nicaragua
Ingeniero de Protocolos Inteligentes | lbh.human

---

## License 

This specification is licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0).
In addition, attribution requirements and philosophical principles specific to the LBH protocol are defined in the MESENTERY License v1.0.
​© 2026 HormigasAIS — El Salvador
Digital Pheromones — Distributed Sovereignty

