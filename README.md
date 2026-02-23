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
- Validación criptográfica: Cada mensaje lleva una firma HMAC-SHA256.
- Eficiencia estructural: Mensajes en texto plano con formato determinístico.
- Consenso distribuido: Quórum tipo Raft entre nodos del enjambre.

---

## Documentos de Especificación

| Documento | Descripción | Estado |
|------------|------------|--------|
| WIRE_FORMAT.md | Formato de mensaje, firma y validación | v1.0 |
| CONSENSUS.md | Algoritmo de quórum entre nodos | En progreso |
| CRYPTO.md | Gestión de claves y rotación | En progreso |
| GOVERNANCE.md | Contratos LBH y gobernanza soberana | En progreso |
| VERIFICATION.md | Guía para auditoría externa | En progreso |

---

## Inicio Rápido — Verificar un Mensaje LBH

    import hmac
    import hashlib

    def verify_lbh(message: str, shared_key: bytes) -> bool:
        """
        Verifica la integridad y autenticidad de un mensaje LBH.
        Formato esperado:
        LBH_DATA:<payload>|TS:<timestamp>|SIG:<firma>
        """
        try:
            parts = message.strip().split("|")
            if len(parts) != 3:
                return False

            payload = parts[0].replace("LBH_DATA:", "")
            ts = parts[1].replace("TS:", "")
            sig_rx = parts[2].replace("SIG:", "")

            content = f"{payload}|{ts}".encode("utf-8")
            sig_calc = hmac.new(shared_key, content, hashlib.sha256).hexdigest()[:16]

            return hmac.compare_digest(sig_rx, sig_calc)

        except Exception:
            return False

---

## Arquitectura del Enjambre

[Nodo Manager Alpha] <-- XOXO-BUS --> [Nodo Escuela]
        |                                      |
        +------------ LBH Protocol ------------+
                  (Wire Format v1.0)

---

## Implementación de Referencia

La implementación oficial está registrada en Zenodo:

DOI: 10.5281/zenodo.17767205

---

## Historial

Versión 1.0.0 (2026-02-22): Especificación inicial auditable — Wire Format.

---

## Fundador

Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Organización: HormigasAIS — El Salvador
Protocolo: lbh.human

---

© 2026 HormigasAIS. Especificación bajo licencia CC-BY-4.0.
