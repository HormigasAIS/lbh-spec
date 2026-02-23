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

- Soberanía local: Cada nodo valida mensajes sin depender de servicios externos.
- Operación offline: Funciona sin Internet activo.
- Validación criptográfica: Cada mensaje lleva firma HMAC-SHA256.
- Eficiencia estructural: Mensajes en texto plano con formato determinístico.
- Consenso distribuido: Quórum tipo Raft entre nodos del enjambre.

---

## Documentos de Especificación

| Documento | Descripción | Estado |
|------------|------------|--------|
| WIRE_FORMAT.md | Formato de mensaje, firma y validación | v1.0 |
| CONSENSUS.md | Consenso BLE físico + Raft lógico | v1.0 |
| CRYPTO.md | AES-256-GCM + HMAC-SHA256 | v1.0 |
| GOVERNANCE.md | Contratos LBH y gobernanza soberana | v1.0 |
| VERIFICATION.md | Guía de auditoría externa | v1.0 |

---

## Inicio Rápido — Verificar un Mensaje LBH

import hmac
import hashlib

def verify_lbh(message: str, shared_key: bytes) -> bool:
    try:
        parts = message.strip().split("|")
        if len(parts) != 3:
            return False

        payload = parts[0].replace("LBH_DATA:", "")
        ts = parts[1].replace("TS:", "")
        sig_rx = parts[2].replace("SIG:", "")

        content = f"{payload}|{ts}".encode("utf-8")
        sig_calc = hmac.new(
            shared_key,
            content,
            hashlib.sha256
        ).hexdigest()[:16]

        return hmac.compare_digest(sig_rx, sig_calc)

    except Exception:
        return False

---

## Arquitectura del Enjambre

[Nodo Manager Alpha] <-- XOXO-BUS --> [Nodo Escuela]
        |                                     |
        +----------- LBH Protocol -----------+
                     (Wire Format v1.0)

---

## Implementación de Referencia

La implementación oficial está registrada en Zenodo:

DOI: 10.5281/zenodo.17767205

---

## Historial

Versión 1.0.0  
Fecha: 2026-02-22  
Cambio: Especificación inicial auditable — todos los documentos.

---

## Fundador

Cristhiam Leonardo Hernández Quiñonez (CLHQ)  
Organización: HormigasAIS — El Salvador  
Protocolo: lbh.human  

---

## License

This specification is licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0).

In addition, attribution requirements and philosophical principles specific to the LBH protocol are defined in the MESENTERY License v1.0.

You may:
- Share — copy and redistribute the material in any medium or format.
- Adapt — remix, transform, and build upon the material for any purpose, including commercial use.

Under the following condition:
- Attribution — You must give appropriate credit to the original authorship, reference the LBH protocol name, and include the DOI when applicable.

For full legal terms, see the LICENSE file.

© 2026 HormigasAIS — El Salvador  
Digital Pheromones — Distributed Sovereignty
