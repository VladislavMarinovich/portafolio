<div align="center">

# 📊 DW · Vitrina Pública (Transparencia)

**Proyecto 2 del Patitas Stack** · las cifras reales por caso, públicas

`salvandopatitas/fsp-data-warehouse` · 🔒 privado · + `salvandopatitas/fsp-infra` · 🔒 restringido

</div>

---

## 🟢 Nivel 1 · Visión — qué resuelve

La promesa de **transparencia radical**: cada peso donado, trazable públicamente desde la donación hasta la factura del veterinario. Este DW **calcula las cifras reales por caso** (donado, gastado, pagado, faltante) en una sola capa auditable y las publica en la web — eliminando los desfases que generaba calcular en el operativo (Salesforce).

## 🔵 Nivel 2 · Arquitectura

*(Todo público: pipeline, diccionario, reglas. El código vivo es el Nivel 3.)*

### Pipeline (medallón · 100% Azure serverless)

```mermaid
flowchart LR
    SF[("Salesforce<br/>Case · Donación · Gasto · Pago")]
    SF -->|extractor incremental · JWT| BRONZE["🥉 bronze<br/>Delta crudo"]
    BRONZE -->|SQL directo| SILVER["🥈 silver<br/>limpio + dedupe por Id"]
    SILVER -->|SQL directo| GOLD["🥇 gold<br/>ficha_caso · ficha_caso_gastos"]
    GOLD -->|POST directo del Job| WP[("🌐 WordPress<br/>vitrina pública")]
    JOB["⚙️ Azure Container App Job · cron 8h"]
    JOB -.orquesta.-> BRONZE
    JOB -.orquesta.-> SILVER
    JOB -.orquesta.-> GOLD
    JOB -.publica.-> WP
```

**Transformación por SQL directo serverless-safe** (dbt descartado: rompe con *"rename"* en Synapse serverless). **Publicación directa**: el propio Job hace el POST a la REST de WordPress (sin intermediarios). Se paga por *correr* la info, no por *tenerla*.

### Diccionario de datos — gold `ficha_caso`

| Campo | Significado |
|---|---|
| `sf_case_id` | llave de unión con WordPress (= `Case.Id`) |
| `numero_caso` · `estado_operativo` | identificación y etapa del caso padre |
| `meta` | total gastado (lo gastado es la meta de recaudo) |
| `donado` · `pagado` | Σ donaciones confirmadas · Σ pagos realizados |
| `faltante_recaudar` · `faltante_pagar` | `max(gastado−donado,0)` · deuda con proveedor |
| `pct_cubierto` · `num_donaciones` · `num_donantes` | indicadores de la vitrina |

### Reglas de negocio (cuadre de cifras)

- **Donación → caso padre**; Gasto/Pago → caso hijo (cadena Pago→Gasto→Caso).
- **Meta = total gastado.** `faltante_recaudar = max(gastado − donado, 0)`.
- **Fondo General** excluido (es la bolsa de la fundación, no un caso).
- En la vitrina aparecen **solo los casos padre** (todo consolidado).
- La web es **vitrina pasiva**: solo lee `ficha_caso`, no calcula nada.

### La infraestructura que lo sostiene (A5)

```mermaid
flowchart TD
    subgraph RG["☁️ Azure · Resource Group prod FSP"]
        KV["Key Vault<br/>(SF JWT · WP App Password)"]
        SP["Service Principal sp-fsp-dw"]
        ACR["Container Registry"]
        JOB["Container App Job<br/>caj-fsp-dw · cron 8h"]
        ADLS["ADLS Gen2<br/>(Delta bronze)"]
        SYN["Synapse serverless<br/>(vistas gold)"]
        OBS["Log Analytics"]
    end
    ACR --> JOB
    SP --> KV
    JOB --> SP
    JOB --> ADLS
    JOB --> SYN
    JOB --> OBS
```

> 🔐 **El código de infraestructura (`fsp-infra`) está restringido — a propósito.** Describe la topología exacta de producción y su postura de seguridad; exponerlo equivaldría a entregar el mapa de la superficie de ataque. Mostramos la **arquitectura** (este diagrama), no el detalle de implementación. Principio de **mínima exposición**.

## 🔒 Nivel 3 · El código

- **DW** (`fsp-data-warehouse`) → 🔒 por solicitud de acceso.

  [![Solicitar acceso](https://img.shields.io/badge/🔑_Solicitar_acceso_al_DW-abrir_solicitud-2ea44f?style=for-the-badge)](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Data+Warehouse)

- **Infraestructura** (`fsp-infra`) → 🔒 **restringido** (ver recuadro arriba). Sin acceso al código por seguridad.

📐 Más diagramas: [data platform (D2)](../atlas/README.md#d2--data-platform--dw-synapse-serverless) · [infra (A5)](../atlas/README.md#a5--infraestructura-cloud-azure) · [loop de transparencia (E1)](../atlas/README.md).

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
