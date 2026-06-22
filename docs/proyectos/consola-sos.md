<div align="center">

# 🖥️ Consola Operativa SOS

**Proyecto 1 del Patitas Stack** · el cerebro operativo del rescate

`salvandopatitas/sf-salvando-patitas-v2` · 🔒 repo privado · Salesforce

</div>

---

## 🟢 Nivel 1 · Visión — qué resuelve

El **cerebro operativo** de la Fundación. Gestiona el ciclo de vida completo de cada rescate —del reporte a la adopción— con **trazabilidad financiera por caso**: cada donación, gasto y pago queda vinculado al animal que lo originó. Es la fuente operativa de verdad del sistema.

## 📸 La Consola funcionando

> Capturas reales del sistema en operación. *Nombres de donantes y proveedores difuminados por Habeas Data (Ley 1581).*

**Resumen del caso** — KPIs vitales + evolución de donaciones:

![Consola — Resumen del caso](img/consola-01-ficha.png)

**Pestaña Finanzas** — balance y movimientos por caso:

![Consola — Finanzas](img/consola-02-finanzas.png)

**Pestaña Veterinaria** — seguimiento clínico:

![Consola — Veterinaria](img/consola-03-veterinaria.png)

**Acción operativa** — registrar una donación (campos de contacto y método de pago difuminados):

![Consola — Registrar donación](img/consola-04-accion.png)

**Selector de caso hijo** — alternar entre Rescate / Veterinaria / Hogar de paso:

![Consola — Selector de caso hijo](img/consola-05-selector.png)

## 🔵 Nivel 2 · Arquitectura

*(Todo este nivel es público: modelo, máquinas de estado, reglas. El código vivo es el Nivel 3.)*

### El modelo: un solo `Case` + Record Types + `ParentId`

Un único objeto **`Case`** modela las etapas operativas (vía **Record Types**) y las relaciones jerárquicas (vía **`ParentId`**): un **caso padre (Rescate)** agrupa **casos hijos** (Veterinaria, Hogar de paso). Donaciones → al padre; gastos/pagos → al hijo; **rollups** consolidan al padre (vía Flows, sin triggers Apex).

### Máquina de estados — Caso padre (Rescate)

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Reportado
    Reportado --> Validacion
    Validacion --> Busqueda
    Busqueda --> Rescate
    Rescate --> Traslado
    Traslado --> IngresoVet
    IngresoVet --> Tratamiento
    Tratamiento --> AltaMedica
    AltaMedica --> HogarPaso
    HogarPaso --> Cerrado
    Cerrado --> [*]

    Reportado: Reportado
    Validacion: En validación
    Busqueda: En búsqueda
    Rescate: En rescate
    Traslado: Traslado
    IngresoVet: Ingreso veterinaria
    Tratamiento: En tratamiento
    AltaMedica: Alta médica
    HogarPaso: En hogar de paso
    Cerrado: Cerrado

    note right of IngresoVet: crea Case hijo Veterinaria (ParentId = Rescate)
    note right of HogarPaso: crea Case hijo Hogar de paso (ParentId = Rescate)
    note left of Tratamiento: la gestión anticipada de hogar de paso puede iniciar aquí (proceso paralelo)
```

### Máquina de estados — Caso hijo · Veterinaria

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Ingreso
    Ingreso --> Diagnostico
    Diagnostico --> Tratamiento
    Tratamiento --> Observacion
    Observacion --> Alta
    Alta --> [*]
    Ingreso: Ingreso clínico
    Diagnostico: En diagnóstico
    Tratamiento: En tratamiento
    Observacion: Observación
    Alta: Alta médica
```
*Captura: diagnóstico · estado de salud · procedimientos · hospitalización · costos · evidencia médica.*

### Máquina de estados — Caso hijo · Hogar de paso

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Buscando
    Buscando --> Evaluando
    Evaluando --> Asignado
    Asignado --> EnHogar
    EnHogar --> Finalizado
    Finalizado --> [*]
    Buscando: Buscando hogar
    Evaluando: Evaluando hogar
    Asignado: Hogar asignado
    EnHogar: En hogar de paso
    Finalizado: Finalizado
```
*Captura: hogar asignado · estado emocional · fecha ingreso/salida · costos · días en hogar (calculado).*

### Reglas de negocio (consistencia del flujo)

- No avanzar a **Veterinaria** sin Case hijo Veterinaria.
- No avanzar a **Hogar de paso** sin Case hijo Hogar.
- Veterinaria requiere **Ingreso** antes de **Tratamiento**.
- **Alta médica** requiere consistencia (datos completos).
- **No cerrar** el caso padre si hay **hijos abiertos**.
- No avanzar a **"En hogar de paso"** sin: **Alta médica** + **Hogar asignado**.

### Decisiones de arquitectura

| Decisión | Razón |
|---|---|
| **Un solo `Case` + Record Types** (no objeto custom) | Modela etapas y jerarquía con lo nativo de la plataforma |
| **Split OLTP/OLAP** — SF operativo + [warehouse analítico](dw-vitrina-publica.md) | SF decide en tiempo real; el warehouse computa la historia |
| **Archivos en Azure Blob** · buckets público/privado con SAS | Límite de storage SF nonprofit + Ley 1581 (PII en bucket privado) |
| **`Fecha_Cierre_Operativo__c` = única fuente de verdad del cierre** | Las fórmulas de tiempo paran con cualquier motivo de cierre |
| **Salesforce aislado de integraciones salientes** | El único callout de SF es a Azure Blob; las integraciones viven fuera del org |

📐 Más diagramas: [modelo de dominio (D1)](../atlas/README.md#d1--modelo-de-dominio) · [panorámica (A3.0)](../atlas/README.md#a30--arquitectura-tecnológica--vista-panorámica-modular).

## 🔒 Nivel 3 · El código

El nivel más profundo: el código vivo (Apex · LWC · Flows), por solicitud de acceso.

[![Solicitar acceso](https://img.shields.io/badge/🔑_Solicitar_acceso_a_la_Consola-abrir_solicitud-2ea44f?style=for-the-badge)](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Consola+Operativa+SOS)

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
