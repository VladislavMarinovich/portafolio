<div align="center">

# 📊 Data Warehouse de Transparencia

**`salvandopatitas/fsp-data-warehouse`** · 🔒 repo privado

[![Solicitar acceso](https://img.shields.io/badge/🔑_Solicitar_acceso_al_DW-abrir_solicitud-2ea44f?style=for-the-badge)](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Data+Warehouse)

</div>

---

## Qué es

El **motor analítico** de la Fundación. Calcula, en una sola capa auditable, las **cifras reales por caso** (donado, gastado, pagado, faltante) que alimentan la vitrina pública — eliminando los desfases que generaban los rollups del operativo (Salesforce).

**Principio:** split **OLTP/OLAP**. El cómputo pesado NO vive en Salesforce; se calcula una vez en la capa analítica (Azure) y se publica. **Serverless siempre** (se paga por *correr* la información, no por *tenerla*).

## Stack

| Capa | Herramienta |
|------|-------------|
| Almacenamiento | Azure ADLS Gen2 · Delta Lake (incremental, particionado por día) |
| Motor de consulta | Azure Synapse **serverless** |
| Transformación | **SQL directo serverless-safe** *(dbt descartado: falla con "rename" en Synapse serverless)* |
| Orquestación | Azure Container App **Job** (cron 8h · pay-per-run) |
| Publicación | el **Job hace POST directo** a la REST de WordPress (SCF) |

## Pipeline (medallón)

```
Salesforce ──(extractor incremental · JWT)──▶ 🥉 bronze (Delta crudo)
                                                 │ SQL directo
                                                 ▼
                                              🥈 silver (limpio + dedupe por Id)
                                                 │ SQL directo
                                                 ▼
                                              🥇 gold · ficha_caso ──(POST directo del Job)──▶ WordPress
```

## Reglas de negocio (cuadre de cifras)

- **Donación → caso padre.** Gasto y Pago → caso hijo (cadena Pago→Gasto→Caso).
- **Meta = total gastado**; `faltante_recaudar = max(gastado − donado, 0)`.
- **Fondo General** = bolsa de la fundación, excluido (no es un caso).
- En la vitrina aparecen **solo los casos padre** (con todo sumado).

## Diagramas

- [Data Platform (D2)](../atlas/README.md#d2--data-platform--dw-synapse-serverless)
- [Arquitectura panorámica (A3.0)](../atlas/README.md#a30--arquitectura-tecnológica--vista-panorámica-modular) · [Infra cloud (A5)](../atlas/README.md#a5--infraestructura-cloud-azure)

## 🔑 Acceso al código

Repo privado. **[Abre una solicitud →](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Data+Warehouse)** o escribe a [vladislav@marinovich.co](mailto:vladislav@marinovich.co?subject=Acceso%20Data%20Warehouse).

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
