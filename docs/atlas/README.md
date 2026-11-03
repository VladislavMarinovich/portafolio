# Atlas de Sistema · Salvando Patitas

Diagramas arquitectónicos del sistema de información de la Fundación Salvando Patitas.

Cada diagrama existe en **dos formatos**:

- **`.mmd` (Mermaid)** — fuente canónica, versionada en git. Renderiza nativo en GitHub y Notion.
- **`.png`** — render pulido exportado de Miro (para portafolio / presentaciones).

> **Convención de nombres:** cada carpeta usa el código del Atlas (`A3.0`, `D1`, `D2`, `E1`) para que Confluence ↔ Miro ↔ GitHub ↔ Notion hablen el mismo idioma.

> **Nota de arquitectura (jun 2026):** el Data Warehouse analítico corre **100% en Azure** (Delta Lake + Synapse serverless). Transformación por **SQL directo serverless-safe** (dbt descartado) y **publicación directa a WordPress** (el Container App Job hace el POST). Fuente de verdad verificada contra prod: Confluence SOSV2 pág 31064066 (SSoT).

---

## A3.0 · Arquitectura Tecnológica — Vista Panorámica Modular

Vista de containers (equivalente C4). Separación OLTP (Salesforce) / OLAP (Azure) / Web pública.

```mermaid
flowchart LR
    subgraph OLTP["🟦 OPERACIÓN · OLTP (Salesforce)"]
        SF["Salesforce<br/>org salvando-patitas<br/>Case · Donación · Gasto · Pago"]
        CONSOLA["Consola SOS<br/>LWC caseDetailPage"]
        SF --- CONSOLA
    end
    subgraph OLAP["🟩 ANALÍTICA · OLAP (Azure)"]
        EXTRACT["extract.py<br/>SF REST · JWT Bearer<br/>watermark incremental"]
        BRONZE["ADLS Gen2 · Delta Lake<br/>(bronze)"]
        GOLD["Synapse serverless · vistas gold<br/>ficha_caso · ficha_caso_gastos"]
        PUBLISH["Publicación directa → WP<br/>el ACJ hace el POST (SCF)"]
        EXTRACT --> BRONZE --> GOLD --> PUBLISH
    end
    subgraph WEB["🌐 WEB PÚBLICA · salvandopatitas.org"]
        WP["WordPress + Elementor<br/>GiveWP + gateway Mercado Pago<br/>casos dinámicos (cifras del DW)"]
        LEADS["Forms → Brevo (leads)"]
        MEDICION["GTM → GA4 + Meta Pixel"]
    end
    SF -- "JWT Bearer" --> EXTRACT
    PUBLISH -- "POST /fsp/v1/caso/sync" --> WP
    JOB["Azure Container App Job · caj-fsp-dw (cron 8h)<br/>ACR · Key Vault · Service Principal · Log Analytics"]
    JOB -. orquesta .-> EXTRACT
    JOB -. orquesta .-> GOLD
    JOB -. orquesta .-> PUBLISH
```

<!-- Render Miro: ![A3.0](A3.0-arquitectura-panoramica/A3.0.png) -->

---

## D1 · Modelo de dominio

Caso padre (consolidado) + casos hijos (intervenciones). Donaciones al padre; gastos/pagos al hijo; rollups en el padre.

```mermaid
erDiagram
    CASE_PADRE   ||--o{ CASE_HIJO     : "agrupa intervenciones"
    CASE_PADRE   ||--o{ DONACION      : "recibe (al padre)"
    CASE_HIJO    ||--o{ GASTO         : "genera (en el hijo)"
    CASE_HIJO    ||--o{ PAGO          : "salda (en el hijo)"
    CUENTA_FONDOS||--o{ DONACION      : "asigna ingreso"
    CUENTA_FONDOS||--o{ PAGO          : "desembolsa"
    CASE_PADRE {
        string CaseNumber
        string Animal
        date   Fecha_Rescate
        date   Fecha_Cierre_Operativo "fuente unica de cierre"
        number Total_Donado  "rollup"
        number Total_Gastado "rollup"
        number Total_Pagado  "rollup"
        number Deuda_Proveedor "rollup (faltante por pagar)"
    }
    CASE_HIJO {
        string Tipo "Veterinaria | Hogar | Adopcion"
        date   Fecha_ingreso_veterinaria
        date   Fecha_alta_medica
        date   Fecha_Ingreso_Hogar
        date   Fecha_Salida_Hogar
    }
    DONACION {
        number Monto
        string Metodo_Pago
        date   Fecha
    }
    GASTO {
        number Monto
        string Tipo_Gasto "Veterinaria | Hogar | Transporte | Administrativo | ..."
        string Comprobante_Azure "URL Blob (archivos-privado)"
    }
    PAGO {
        number Monto
        date   Fecha
    }
    CUENTA_FONDOS {
        string Nombre "Fondo General SOS | Nequi"
    }
```

<!-- Render Miro: ![D1](D1-modelo-dominio/D1.png) -->

---

## D2 · Data Platform — DW Synapse serverless

Pipeline SF → Delta Lake → Synapse (silver+gold por SQL directo, serverless-safe) → publicación directa a WordPress. dbt descartado (falla "rename" en Synapse serverless).

```mermaid
flowchart TD
    SF[("Salesforce<br/>Case · Donación · Gasto · Pago")]
    subgraph INGEST["1 · Ingesta"]
        EX["extract.py<br/>SF REST · JWT Bearer<br/>delta-rs · schema_mode merge<br/>watermark incremental"]
    end
    subgraph LAKE["2 · Lake · ADLS Gen2"]
        BR["Delta Lake · bronze"]
    end
    subgraph TRANSFORM["3 · Transformación (SQL directo · serverless-safe)"]
        RUN["Construcción silver + gold<br/>SQL directo · serverless-safe"]
        STG["silver · casos/donaciones/gastos/pagos"]
        FC["ficha_caso"]
        FCG["ficha_caso_gastos"]
        RUN --> STG --> FC & FCG
    end
    subgraph PUB["4 · Publicación"]
        PB["Publicación DIRECTA → WordPress (SCF)<br/>el job hace el POST<br/>excluye Fondo General · throttle anti-WAF"]
    end
    WP[("WordPress SCF<br/>/wp-json/fsp/v1/caso/sync")]
    SF -->|JWT Bearer| EX --> BR --> RUN
    FC  --> PB
    FCG --> PB
    PB  --> WP
    ORQ["Azure Container App Job · caj-fsp-dw · cron 8h<br/>ACR · KV · Service Principal sp-fsp-dw · Log Analytics<br/>Costo DW ≈ 0.09 USD/año (serverless)"]
    ORQ -. orquesta .-> EX
    ORQ -. orquesta .-> RUN
    ORQ -. orquesta .-> PB
```

<!-- Render Miro: ![D2](D2-data-platform/D2.png) -->

---

## E1 · Loop de Transparencia Radical · v2

SF master único + 2 puertas de entrada → aprobación → DW → vitrina pública con trazabilidad E2E.

```mermaid
flowchart LR
    subgraph ENTRADA["Puertas de entrada"]
        ONLINE["🟢 Online<br/>GiveWP + Mercado Pago"]
        OFFLINE["🟠 Offline<br/>registro manual (coordinadora)"]
    end
    APROB{"Aprobación<br/>en Salesforce"}
    SF["Salesforce · master único<br/>Donación validada"]
    subgraph DW["DW analítico · Azure"]
        SYN["Synapse gold<br/>ficha_caso"]
    end
    WP["Vitrina pública · solo muestra cifras<br/>(no origina donaciones)"]
    CIUDADANO["👤 Donante / ciudadano<br/>ve trazabilidad extremo a extremo"]
    ONLINE  --> APROB
    OFFLINE --> APROB
    APROB -- "aprobada" --> SF
    SF -- "JWT · cron 8h" --> SYN
    SYN -- "publicación directa (ACJ)" --> WP
    WP --> CIUDADANO
    %% incentivo económico, NO flujo técnico — re-entra por las puertas (siempre pasa por SF)
    CIUDADANO -. "incentiva nueva donación → re-entra por las puertas" .-> ENTRADA
```

<!-- Render Miro: ![E1](E1-loop-transparencia/E1.png) -->

---

## A5 · Infraestructura Cloud (Azure)

*Repo: `salvandopatitas/fsp-infra` (IaC · Terraform). Topología de recursos Azure que sostiene el DW.*

```mermaid
flowchart TD
    subgraph RG["☁️ Azure · Resource Group prod FSP"]
        subgraph AUTH["Identidad y secretos"]
            KV["Key Vault kv-salvandopatitas"]
            SP["Service Principal sp-fsp-dw<br/>(auth Azure: client id/secret)"]
        end
        subgraph COMPUTE["Cómputo"]
            ACR["Container Registry acrsalvandopatitas"]
            JOB["Container App Job caj-fsp-dw<br/>cron 8h · pay-per-run"]
        end
        subgraph DATA["Datos (DW)"]
            ADLS["ADLS Gen2 stsalvandopatitasdw<br/>raw · synapse (Delta bronze)"]
            SYN["Synapse serverless syn-salvandopatitas-prod<br/>DB fsp_dw (vistas gold)"]
            BLOB["Blob archivos-publico / archivos-privado<br/>(adjuntos · SAS 60min · Ley 1581)"]
        end
        OBS["Log Analytics"]
    end
    ACR -- imagen --> JOB
    SP -- lee secretos --> KV
    JOB --> SP
    JOB -- escribe Delta --> ADLS
    JOB -- "construye silver+gold (SQL directo)" --> SYN
    SYN -- OPENROWSET --> ADLS
    JOB -- logs --> OBS
    SP -- "auth Synapse" --> SYN
    SF[("Salesforce (JWT)")] -. extrae .-> JOB
    JOB -. publica .-> WP[("WordPress")]
```

<!-- Render Miro: ![A5](A5-infraestructura-cloud/A5.png) -->

---

## A3.1 · Sistema Web Pública

*Repo: `salvandopatitas/wp-child-theme`. Vitrina pasiva: publica cifras del DW, gestiona donación, leads y medición.*

```mermaid
flowchart LR
    subgraph WP["🌐 WordPress · Hostinger"]
        SNIP["Snippets FSP 01-07"]
        TPL["Template Single Caso (lee SCF)"]
        FORM["Forms Elementor"]
        GIVE["GiveWP 7"]
    end
    DW[("DW Synapse gold")] -- "ACJ: POST directo → WP REST (SCF)" --> SNIP --> TPL
    DONANTE["👤 Donante"] --> GIVE -- Checkout --> MP["Mercado Pago"]
    MP -- "IPN" --> GIVE
    FORM -- "captura de lead" --> BREVO["Brevo"]
    GTM["GTM-TCQXDTVB"] --> GA4["GA4"]
    GTM --> PIXEL["Meta Pixel"]
    WP --- GTM
```

<!-- Render Miro: ![A3.1](A3.1-web-publica/A3.1.png) -->

---

## B1 · BPMN del proceso de rescate

Flujo operativo de un rescate, del reporte ciudadano hasta la ubicación del animal. Siete carriles: el Líder de Operaciones y el Transportista aparecen dos veces porque intervienen en dos momentos distintos del proceso (rescate y ubicación).

> **Trazabilidad:** la flecha sólida es un conector que existe en el tablero de Miro. La flecha punteada se dedujo de la lógica del proceso y está pendiente de validar con operaciones. El nodo en rojo marca un hueco real: el tablero no define qué pasa con un animal que no es manipulable.

```mermaid
flowchart LR
    subgraph C1["Rescatista"]
        direction TB
        E1((" Reporta<br/>animal ")):::inicio
    end
    subgraph C2["Líder de Operaciones de Rescate"]
        direction TB
        G1{"¿Acepta<br/>el caso?"}
        F1(((" Archivar<br/>caso "))):::fin
        T1["Buscar veterinaria"]
        TM1(("Esperar<br/>6 horas")):::timer
        G2{"¿Veterinaria<br/>disponible?"}
        T2["Asignar veterinaria"]
        T3["Asignar transportista"]
    end
    subgraph C3["Transportista"]
        direction TB
        T4["Desplazarse al punto"]
        T5["Buscar animal<br/>reportado"]
        TM2(("Esperar<br/>24 horas")):::timer
        G3{"¿Encontraron<br/>al animal?"}
        G4{"¿Animal<br/>manipulable?"}
        T6["Capturar animal<br/>reportado"]
        T7["Transportar a<br/>veterinaria"]
        HUECO1["Sin definir<br/>en el tablero"]:::hueco
    end
    subgraph C4["Veterinaria"]
        direction TB
        T8["Recepción del animal"]
        T9["Diagnóstico inicial"]
        G5{"¿Requiere<br/>hospitalización?"}
        TR1[["Tratamiento y<br/>seguimiento"]]
        G6{"¿Requiere seguir<br/>hospitalización?"}
        T10["Alta médica"]
        G8{"¿Requiere<br/>hogar de paso?"}
        T11["Animal listo para<br/>adopción directa"]
        F2(((" Entregar animal<br/>a adoptante "))):::fin
        T16["Notifica al Líder de<br/>Operaciones de Rescate"]
    end
    subgraph C5["Líder de Operaciones de Rescate "]
        direction TB
        TR2[["Buscar hogar de paso"]]
        G7{"¿Hogar de paso<br/>disponible?"}
        TM3(("Esperar<br/>24 horas")):::timer
        T12["Pagar día extra<br/>en veterinaria"]
        T13["Asignar transportista"]
    end
    subgraph C6["Transportista "]
        direction TB
        T14["Desplazarse a<br/>veterinaria"]
        T15["Transportar a<br/>hogar de paso"]
    end
    subgraph C7["Hogar de paso"]
        direction TB
        T17["Recepción del animal"]
        F3(((" Animal ubicado en<br/>hogar de paso activo "))):::fin
    end
    E1 --> G1
    G1 --> F1
    G1 -.->|SÍ| T1
    T1 --> G2
    G2 -->|NO| T1
    G2 -.->|SÍ| T2
    T2 -.-> TM1
    TM1 -.-> T1
    T2 --> T3
    T3 --> T4
    T4 --> T5
    T4 --> TM2
    TM2 -.->|sin rastro del animal| F1
    T5 --> G3
    G3 -.->|NO| TM2
    G3 -.->|SÍ| G4
    G4 -.->|SÍ| T6
    G4 -.->|NO| HUECO1
    T6 --> T7
    T7 --> T8
    T8 --> T9
    T9 --> G5
    G5 -->|SÍ| TR1
    TR1 --> G6
    G6 -->|SÍ| TR1
    G5 -.->|NO| T10
    G6 -.->|NO| T10
    T10 --> G8
    G8 -->|NO| T11
    T11 --> F2
    G8 -.->|SÍ| T16
    T16 --> TR2
    TR2 --> G7
    G7 -->|NO| T12
    T12 --> TR2
    T12 -.-> TM3
    TM3 -.-> TR2
    G7 -.->|SÍ| T13
    T13 --> T14
    T14 --> T15
    T15 --> T17
    T17 --> F3
    classDef inicio fill:#ffffff,stroke:#1a1a1a,stroke-width:2px,color:#1a1a1a
    classDef fin fill:#ffffff,stroke:#1a1a1a,stroke-width:4px,color:#1a1a1a
    classDef timer fill:#ffffff,stroke:#bd0a0a,stroke-width:2px,color:#1a1a1a
    classDef hueco fill:#fff0f0,stroke:#bd0a0a,stroke-width:1px,color:#1a1a1a
```

<!-- Render Miro: ![B1](B1-bpmn-rescate/B1.png) -->

---

*Mantenedor: Vladislav Marinovich · arquitectura, código y metodología © 2026 **Vladislav Marinovich Vargas** (recursos propios) · operado por Marinovich Consulting S.A.S. · FSP recibe licencia de uso; los datos operativos son de la Fundación (Ley 1581).*
