# LBH Consensus — Especificación v1.0
Protocolo: Lenguaje Binario HormigasAIS (LBH)
Autor: Cristhiam Leonardo Hernández Quiñonez (CLHQ)
Estado: Draft Auditable
Fecha: 2026-02-22

---

## 1. Visión General

LBH implementa dos capas de consenso complementarias:

| Capa    | Mecanismo               | Hardware                  | Propósito                               |
|---------|-------------------------|---------------------------|------------------------------------------|
| Capa 1  | BLE Physical Consensus  | Multi-dispositivo físico  | Presencia y soberanía de nodo            |
| Capa 2  | Raft Log Consensus      | Multi-proceso local       | Replicación de estado y entradas         |

Ambas capas operan sin Internet. La red puede estar completamente caída y el enjambre mantiene su estado soberano.

---

## 2. Capa 1 — Consenso Físico BLE

### 2.1 Arquitectura

[A16 — Nodo Maestro/Validador]
        |
        |  Bluetooth Low Energy (BLE)
        |  Scanning cada 500 ms
        |
[A20s — Nodo Periférico/Beacon]
  Broadcast: "S9-DATA-IMMUNE-2026"

El Nodo Maestro (A16) escanea activamente el entorno BLE buscando el identificador soberano del Nodo Periférico (A20s).
No hay handshake TCP/IP ni servidor intermedio — solo presencia física verificable.

### 2.2 Identificador Soberano

OBJETIVO_BLE = "S9-DATA-IMMUNE-2026"

Este identificador representa el ADN lógico del nodo periférico. Solo un dispositivo autorizado dentro de la colonia transmite este identificador.

### 2.3 Algoritmo de Consenso por Umbral

MATCHES = 0
UMBRAL  = 2        # Detecciones consecutivas requeridas
RATE    = 0.5      # Segundos entre muestras (2 Hz)

while True:
    if escanear_ble_veloz():   # Detecta "S9-DATA-IMMUNE-2026"
        MATCHES += 1
        if MATCHES >= UMBRAL:
            estado = "CONSENSO_VALIDADO"
    else:
        MATCHES = 0            # Reset por ruido o pérdida de señal
        estado = "ESCANEANDO"

    time.sleep(RATE)

Razonamiento del umbral:
Requerir 2 detecciones consecutivas reduce falsos positivos y estabiliza la señal en entornos con interferencia.

### 2.4 Frecuencia de Muestreo (2 Hz)

La pausa de 0.5 s produce estabilidad operativa y optimiza el procesador ARM del A16 sin degradar la detección.

### 2.5 Fallback Soberano

Si el A20s se desconecta:

try:
    escanear_ble_veloz()
except ConnectionError:
    print("A20 desconectado... Manteniendo soberanía local.")
    pass

El nodo maestro mantiene operación local sin depender del periférico.

---

## 3. Capa 2 — Consenso Raft (Replicación de Estado)

### 3.1 Propósito

Garantiza consistencia de estado: todos los nodos mantienen el mismo historial de entradas LBH.

### 3.2 Configuración del Cluster

Puerto 9301: Líder activo (Manager Alpha)
Puerto 9302: Seguidor 1
Puerto 9303: Seguidor 2
Quórum: 3 nodos (mayoría requerida 2/3)
Commit Index: 4 (validado en entorno local)

### 3.3 Métricas Verificadas

LÍDER: Puerto 9301
QUÓRUM: 3/3 nodos sincronizados
COMMIT: Index 4
SHA256: 73a980e3...
RSA: Verified OK

---

## 4. Tolerancia a Fallos

Escenario: A20s fuera de rango BLE  
Comportamiento: A16 mantiene soberanía local sin interrupción

Escenario: A20s vuelve al rango  
Comportamiento: Consenso BLE se restablece automáticamente en <=1 s

Escenario: Internet/WiFi caído  
Comportamiento: Operación offline sin degradación (BLE independiente)

Escenario: Nodo Raft caído  
Comportamiento: Mayoría 2/3 permite continuar operación

---

## 5. HUD — Interfaz de Consenso en Tiempo Real

┌──────────────────────────────────────┐
│ HormigasAIS Air City — LBH          │
│ FUNDADOR: Cristhiam L. Hernández Q. │
├──────────────────────────────────────┤
│ BLE:     ESCANEANDO...               │
│ TARGET:  S9-DATA-IMMUNE-2026         │
│ MATCHES: 1/2                         │
├──────────────────────────────────────┤
│ RAFT:    LÍDER: 9301 (Index: 4)      │
│ QUÓRUM:  3/3 OK                      │
└──────────────────────────────────────┘

---

## 6. Justificación de BLE como Canal Primario

LBH utiliza BLE en hardware edge por:

- Independencia de router o infraestructura externa
- Bajo consumo energético
- Verificación de presencia física además de conectividad lógica

---

## 7. Roadmap de Consenso

Fase 5 (Actual): BLE Consensus A16 <-> A20s + Raft local
Fase 6: Raft distribuido multi-host LAN
Fase 7: Raft distribuido multi-sitio (VPN/mesh)

---

© 2026 HormigasAIS
Cristhiam Leonardo Hernández Quiñonez
