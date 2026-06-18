<div align="center">

# 🖥️ Consola Operativa SOS

**`salvandopatitas/sf-salvando-patitas-v2`** · 🔒 repo privado

[![Solicitar acceso](https://img.shields.io/badge/🔑_Solicitar_acceso_a_la_Consola-abrir_solicitud-2ea44f?style=for-the-badge)](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Consola+Operativa+SOS)

</div>

---

## Qué es

El **cerebro operativo** de la Fundación Salvando Patitas, sobre Salesforce. Gestiona el ciclo de vida completo de cada rescate —del reporte ciudadano a la adopción— con **trazabilidad financiera por caso**: cada donación, gasto y pago queda vinculado al animal que lo originó.

**Stack:** Apex · Lightning Web Components · Flows · Chart.js · Azure Blob Storage.

## El modelo: caso padre + casos hijos

```
Caso padre (Operación de Rescate · "Chimuelo")
├── Caso hijo · Atención Veterinaria        ← gastos clínicos
├── Caso hijo · Gestión de Hogar de Paso    ← manutención
└── Caso hijo · Adopción                    ← cierre del ciclo
```

- **Donaciones** → siempre al caso padre (el donante apoya al animal, no a una intervención).
- **Gastos y pagos** → al caso hijo correspondiente (trazabilidad por etapa).
- **Rollups multinivel** consolidan todo hacia el padre en tiempo real (vía Flows, sin triggers Apex).

## La consola (LWC `caseDetailPage`)

Una sola página de registro con 5 pestañas: **Resumen** (KPIs + gráfico de evolución de donaciones), **Veterinaria** (seguimiento clínico), **Hogar de paso**, **Finanzas** (Balance General consolidado · Ingresos/Gastos/Pagos) y **Edición**.

## Decisiones de arquitectura

| Decisión | Razón |
|---|---|
| **Split OLTP/OLAP** — SF operativo + [warehouse analítico](data-warehouse.md) | SF decide en tiempo real; el warehouse computa la historia una sola vez |
| **Archivos en Azure Blob** (no SF Files) · buckets público/privado con SAS | Límite de storage de SF nonprofit + compliance Ley 1581 (PII en bucket privado) |
| **`Fecha_Cierre_Operativo__c` = única fuente de verdad del cierre** | Las fórmulas de tiempo paran de contar con cualquier motivo de cierre (adopción, fallecimiento) |
| **Salesforce aislado de integraciones salientes** | El único callout de SF es a Azure Blob (directo, vía Apex); las integraciones viven fuera del org |

## Diagramas

- [Modelo de dominio (D1)](../atlas/README.md#d1--modelo-de-dominio) — caso padre/hijo + finanzas.
- [Arquitectura panorámica (A3.0)](../atlas/README.md#a30--arquitectura-tecnológica--vista-panorámica-modular) — su lugar en el sistema.

## 🔑 Acceso al código

Repo privado. Para revisarlo: **[abre una solicitud →](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Consola+Operativa+SOS)** y te agrego como lector. O escribe a [vladislav@marinovich.co](mailto:vladislav@marinovich.co?subject=Acceso%20Consola%20SOS).

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
