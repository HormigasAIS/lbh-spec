LBH Wire Format — Especificación v1.0
Protocolo: Lenguaje Binario HormigasAIS (LBH)
Autor: Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Estado: Draft Auditable
Fecha: 2026-02-22
Versión: 1.0.0
1. Estructura del Mensaje
Todo mensaje LBH sigue este formato de texto plano:
LBH_DATA:<payload>|TS:<timestamp>|SIG:<signature>
Campos
Campo
Tipo
Descripción
payload
string UTF-8
Contenido del mensaje. No puede contener el carácter |
timestamp
integer
Unix epoch en segundos (UTC)
signature
string hex
HMAC-SHA256 del contenido, truncado a 16 caracteres hex
2. Cálculo de la Firma
SIG = HMAC-SHA256( payload + "|" + timestamp , shared_key )[:16]
Algoritmo: HMAC-SHA256
Entrada: concatenación de payload, pipe literal |, y timestamp como string
Clave: shared_key de 32 bytes, distribuida fuera de banda entre nodos autorizados
Salida: primeros 16 caracteres del digest hexadecimal
Ejemplo verificable
payload   = "INMUNIDAD_HONGO_ACTIVA"
timestamp = 1770805318

Mensaje resultante:
LBH_DATA:INMUNIDAD_HONGO_ACTIVA|TS:1770805318|SIG:bba69499c9e516d1
3. Validación de un Mensaje Entrante
Un nodo LBH acepta un mensaje si y solo si se cumplen las tres condiciones:
El formato cumple la estructura LBH_DATA:...|TS:...|SIG:...
El timestamp no difiere en más de 300 segundos del reloj del nodo receptor (protección anti-replay)
La SIG recalculada coincide exactamente con la recibida
Si cualquier condición falla, el mensaje va a cuarentena con veredicto LBH_VEREDICT=BLOCKED.
4. Frame de Conocimiento (LBH Frame)
Para eventos estructurados, contratos y lecciones se usa el formato extendido:
[LBH_FRAME_START]
ID=<sha256_primeros_8_chars>
DATA=<payload_en_hexadecimal>
[LBH_FRAME_END]
ID: SHA256 del contenido original, primeros 8 caracteres
DATA: contenido codificado en hexadecimal
5. Mensajes de Sistema Reservados
Payload
Significado
INMUNIDAD_HONGO_ACTIVA
Nodo en estado inmune, rechazo activo de intrusos
FEROMONA_XOXO_ACTIVA
Bus XOXO operativo, nodos sincronizados
LBH_STATUS:OPERATIVO_SOBERANO
Heartbeat de soberanía del nodo
INTRUSO_TEST
Mensaje de prueba — debe ser bloqueado por diseño
6. Verificación Independiente (Python)
Cualquier implementación externa puede verificar un mensaje LBH con este script:
import hmac
import hashlib

def verify_lbh(message: str, shared_key: bytes) -> bool:
    """
    Verifica un mensaje LBH.
    Retorna True si es autentico, False en cualquier otro caso.
    """
    try:
        parts = message.strip().split("|")
        if len(parts) != 3:
            return False

        payload  = parts[0].replace("LBH_DATA:", "")
        ts       = parts[1].replace("TS:", "")
        sig_rx   = parts[2].replace("SIG:", "")

        content  = f"{payload}|{ts}".encode("utf-8")
        sig_calc = hmac.new(shared_key, content, hashlib.sha256).hexdigest()[:16]

        return hmac.compare_digest(sig_rx, sig_calc)

    except Exception:
        return False


# --- Ejemplo de uso ---
if __name__ == "__main__":
    KEY = b"LBH_SHARED_SECRET_32BYTES!!!!!!"  # 32 bytes
    msg = "LBH_DATA:INMUNIDAD_HONGO_ACTIVA|TS:1770805318|SIG:bba69499c9e516d1"
    print("Valido:", verify_lbh(msg, KEY))
7. Limitaciones Conocidas (v1.0)
Limitación
Plan
La shared_key se distribuye fuera de banda
Rotación automática planificada para v1.1
Payload máximo de 4KB
Fragmentación planificada para v1.2
Sin cifrado de payload en tránsito
Se asume canal de red cifrado a nivel de transporte
8. Historial de Versiones
Versión
Fecha
Cambio
1.0.0
2026-02-22
Especificación inicial auditable
Este documento es la fuente de verdad del formato wire LBH.
Cualquier implementación que difiera de esta especificación no es LBH-compatible.
© 2026 HormigasAIS — Cristhiam Leonardo Hernández Quiñonez
