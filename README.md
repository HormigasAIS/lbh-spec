LBH Protocol Specification
Lenguaje Binario HormigasAIS — Especificación técnica formal y auditable
 
¿Qué es LBH?
LBH (Lenguaje Binario HormigasAIS) es un protocolo de comunicación diseñado para coordinar nodos edge distribuidos con las siguientes propiedades:
Soberanía local — cada nodo valida mensajes sin depender de servicios externos
Operación offline — funciona sin Internet activo
Validación criptográfica — cada mensaje lleva firma HMAC-SHA256
Eficiencia extrema — mensajes en texto plano, sin overhead de protocolos pesados
Consenso distribuido — quórum tipo Raft entre nodos del enjambre
Documentos de Especificación
Documento
Descripción
Estado
WIRE_FORMAT.md
Formato de mensaje, firma y validación
✅ v1.0
CONSENSUS.md
Algoritmo de quórum entre nodos
🔄 En progreso
CRYPTO.md
Gestión de claves y rotación
🔄 En progreso
GOVERNANCE.md
Contratos LBH y gobernanza soberana
🔄 En progreso
VERIFICATION.md
Guía para auditoría externa
🔄 En progreso
Inicio Rápido — Verificar un Mensaje LBH
import hmac, hashlib

def verify_lbh(message: str, shared_key: bytes) -> bool:
    parts = message.strip().split("|")
    if len(parts) != 3:
        return False
    payload  = parts[0].replace("LBH_DATA:", "")
    ts       = parts[1].replace("TS:", "")
    sig_rx   = parts[2].replace("SIG:", "")
    content  = f"{payload}|{ts}".encode("utf-8")
    sig_calc = hmac.new(shared_key, content, hashlib.sha256).hexdigest()[:16]
    return hmac.compare_digest(sig_rx, sig_calc)
Ver especificación completa en WIRE_FORMAT.md.
Implementación de Referencia
La implementación oficial del protocolo LBH está registrada y preservada en Zenodo:
HormigasAIS-LBH-Demo
DOI: 10.5281/zenodo.17767205
Incluye: lbh-core.js, lbh-hash.js, lbh-cli.js, lbh-config.json
Arquitectura del Enjambre
[Nodo Manager Alpha] ←── XOXO-BUS ──→ [Nodo Escuela]
        │                                      │
        └──────────── LBH Protocol ────────────┘
                      (Wire Format v1.0)
Los nodos se comunican exclusivamente mediante mensajes LBH. El componente XOXO actúa como fiscalía soberana — valida, aprueba o envía a cuarentena cada mensaje que atraviesa el bus.
Ecosistema HormigasAIS
Repositorio
Descripción
HormigasAIS/HormigasAIS
Núcleo operativo (privado)
Thrumanshow/HormigasAIS-FASES
Registro histórico de fases
HormigasAIS/lbh-spec
Este repositorio — especificación formal
Historial
Versión
Fecha
Cambio
1.0.0
2026-02-22
Especificación inicial auditable — Wire Format
Fundador: Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Organización: HormigasAIS — El Salvador
Protocolo: lbh.human
© 2026 HormigasAIS. Especificación bajo licencia CC-BY-4.0.
