# LBH Governance — Especificación v1.0
**Protocolo:** Lenguaje Binario HormigasAIS (LBH)
**Autoridad de Ejecución:** Cristhiam Leonardo Hernández Quiñonez (LBH-MASTER)
**Estado:** Activo - Edge Sovereignty
**Fecha:** 2026-02-22

---

## 1. Declaración de Soberanía en el Borde (Edge Sovereignty)
A diferencia de los protocolos legados, LBH v1.0 establece que la **autoridad de ejecución reside exclusivamente en el hardware físico (Edge)**. 

* **GitHub** actúa únicamente como una capa de persistencia y auditoría (Publication Layer).
* **La Red LBH** es soberana: las decisiones de estado se toman en los Nodos A16/A20s sin dependencia de entidades centralizadas o nubes extranjeras.

## 2. Jerarquía de Autoridad
La colonia opera bajo un modelo de **Autoridad Híbrida (Humano-Agente)**:

1.  **LBH-MASTER (CLHQ):** Única entidad con autoridad para modificar el núcleo del protocolo y las llaves maestras de cifrado.
2.  **Manager Alpha (Nodo Maestro):** Responsable de mantener el Quórum Raft y validar la presencia física de otros nodos.
3.  **Agentes Centinela:** Nodos de ejecución autónoma que operan bajo reglas predefinidas de inmunidad y monitoreo.

---

## 3. Modelo de Decisión LBH
Para que una acción sea considerada "Válida" en la colonia, debe cumplir con el **Triple Consenso**:

| Capa | Requisito | Validación |
| :--- | :--- | :--- |
| **Física** | Presencia BLE | Detección del pulso "S9-DATA-IMMUNE-2026" |
| **Lógica** | Quórum Raft | Consenso de mayoría (2/3) en el cluster local |
| **Cripto** | Firma HMAC | Verificación de integridad con shared_key soberana |

---

## 4. Política de Ejecución Offline
HormigasAIS es una infraestructura diseñada para la **Resiliencia Extrema**. 
* La gobernanza dicta que el sistema **debe** ser capaz de operar en aislamiento total de Internet. 
* La pérdida de conexión con el mundo exterior no degrada la autoridad local del LBH-MASTER sobre su enjambre.

## 5. Principio de Inmutabilidad en el Borde
Ningún cambio realizado en el repositorio de GitHub tiene efecto sobre la ejecución del Centinela hasta que el LBH-MASTER realice una **Validación Manual de Proximidad**. Esto protege a la colonia contra ataques de cadena de suministro o secuestro de cuentas en la nube.

---

## 6. Ética y Propósito
La gobernanza de HormigasAIS está orientada a la protección de la soberanía de datos y la automatización resiliente en entornos críticos (Air City El Salvador). 

© 2026 HormigasAIS — Cristhiam Leonardo Hernández Quiñonez
