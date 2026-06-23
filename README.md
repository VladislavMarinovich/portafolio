<div align="center">

# 🐾 Patitas Stack · Portafolio de Vladislav Marinovich

**Sistemas de información *policy-grade* para el sector social de LATAM**

_De la ingesta del dato a la decisión — arquitectura, ingeniería de datos e IA aplicada al impacto_

[![Perfil](https://img.shields.io/badge/GitHub-VladislavMarinovich-181717?logo=github)](https://github.com/VladislavMarinovich)
[![Web](https://img.shields.io/badge/Marinovich_Consulting-marinovich.co-E8702A)](https://marinovich.co)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-vmarinovich-0A66C2?logo=linkedin)](https://www.linkedin.com/in/vmarinovich)

</div>

---

> **Cómo leer este portafolio — 3 niveles de profundidad:**
> 🟢 **Nivel 1 · Visión** (aquí) · qué construí, productos, potencial e investigación.
> 🔵 **Nivel 2 · Arquitectura** · cada [ficha de proyecto](#-los-proyectos) y el [atlas](docs/atlas/): diagramas, diccionario de datos, dominio y máquinas de estado (todo público).
> 🔒 **Nivel 3 · El código** · los repos privados, por **solicitud de acceso** desde cada ficha.

---

## 🎯 Qué construí

El sector de protección animal de LATAM opera **a ciegas**: sin trazabilidad del dinero, sin datos de operación, sin forma de demostrar impacto. **Diseñé y construí el sistema de información que cambia eso** — de la ingesta del dato a la decisión, **end-to-end y como arquitecto único**: cada peso donado, trazable públicamente desde la donación hasta la factura del veterinario; la operación, medida para decidir mejor.

Soy **CTO y cofundador** de la Fundación Salvando Patitas: la misión y los rescates son del equipo; **la arquitectura y el código del sistema son míos.**

**Métrica norte: vidas salvadas por peso invertido.** Marco: *design science* + sistemas de información (datos reales de campo, no de laboratorio).

## 🧠 Lo que lo vuelve investigación: la capa de decisión

Construir el sistema es **ingeniería** — necesaria, pero no es, por sí sola, la tesis. Lo que convierte esto en un **proyecto de investigación** es la **capa de inteligencia** que vive encima: decidir, bajo **incertidumbre y presupuesto finito**, *qué caso atender* para maximizar el bienestar generado. Es **teoría de colas con restricción presupuestal** + priorización ponderada por **urgencia, valor esperado y sufrimiento**.

Todo lo demás —la infraestructura de transparencia, la plataforma de reportes— es el **ecosistema que hace posible esa capa**. Esa pieza de decisión es la clave, y la original.

## 🐾 El Patitas Stack

Un sistema end-to-end: desde que un ciudadano reporta un animal en condición crítica hasta que el caso se cierra — con trazabilidad financiera por caso. Su columna vertebral es un **split OLTP/OLAP**: Salesforce decide en tiempo real, un warehouse serverless en Azure computa la historia, y la web es una vitrina pasiva de cifras reales.

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
        MEDICION["GTM → GA4 + Meta Pixel"]
    end
    SF -- "JWT Bearer" --> EXTRACT
    PUBLISH -- "POST /fsp/v1/caso/sync" --> WP
    JOB["Azure Container App Job · caj-fsp-dw (cron 8h)<br/>ACR · Key Vault · Service Principal · Log Analytics"]
    JOB -. orquesta .-> EXTRACT
    JOB -. orquesta .-> GOLD
    JOB -. orquesta .-> PUBLISH
```

📐 **[Atlas completo de arquitectura →](docs/atlas/)** · 6 diagramas (panorámica, dominio, data platform, infraestructura, web y el loop de transparencia).

## 📸 El sistema, funcionando

**Consola SOS — el cerebro operativo, corriendo en producción** *(captura real; datos sensibles difuminados):*

![Consola — seguimiento clínico del caso](docs/proyectos/img/consola-03-veterinaria.png)

> *Nota de transparencia: la **fecha de ingreso** y el indicador **hospitalizado** aún no se autocompletan al abrir el caso clínico — esa regla de integridad está en implementación. Es parte de un trabajo deliberado de **automatizar la consistencia del dato**: que ciertos valores se calculen, no se digiten a mano.*

→ más vistas del sistema en la **[ficha de la Consola](docs/proyectos/consola-sos.md)**.

**App de reportes ciudadanos — en diseño, con prototipo navegable:**

- ▶️ **[Demo navegable del flujo de 30 s →](https://vladislavmarinovich.github.io/portafolio/proyectos/prototipo-app.html)** · prototipo clickeable (sin backend)
- 📄 **[Directiva de Diseño (DD-2026-03) →](https://vladislavmarinovich.github.io/portafolio/proyectos/directiva-diseno-app.html)** · la restricción de 30 s, el flujo, los *edge states* y la *kill list*

→ detalle en la **[ficha de la App](docs/proyectos/app-sos-priorizacion.md)**.

## 🚀 Potencial

- **Infraestructura reutilizable** para el sector animalista de LATAM — un terreno **sin benchmark regional**. Lo construido para una fundación es replicable para muchas.
- **Modelo de impacto medible**: la trazabilidad pública no es solo ética, es un **mecanismo de captación** (demostrar impacto atrae recursos).
- **Camino a producto**: del sistema a medida hoy → a un paquete/SaaS para ONGs del tercer sector.

## 🔬 Líneas de investigación

- **Asignación de recursos bajo presupuesto — la capa de decisión** *(el núcleo del aporte)*: priorización de casos bajo incertidumbre, con teoría de colas + costo de oportunidad + bienestar (urgencia, valor esperado y sufrimiento) sobre datos de campo reales.
- **Trazabilidad pública *policy-grade***: cómo diseñar la transparencia de un sistema social para que sostenga la confianza y la captación de recursos (bajo Ley 1581 / Habeas Data).
- **Dataset georreferenciado original** de reportes ciudadanos del Caribe colombiano — datos que no existían.
- **Triage por visión artificial**: de la foto del reporte a una **sospecha de gravedad/condición** (decision-support, nunca diagnóstico). Por diseño, el par `evidencia (foto) ↔ diagnóstico del vet` será dato de entrenamiento etiquetado que el sistema generará cuando la App entre en operación.

## 💎 Activos que pueden emerger

- El **dataset sectorial original** (georreferenciado + clínico) — el *moat*.
- **Pares imagen→etiqueta** para entrenar modelos de triage.
- El **paquete productizado** del Patitas Stack para otras ONGs.
- La **metodología** (cómo se construye un sistema de información del tercer sector de cero).

## 🧩 Los proyectos

> **El ecosistema que alimenta la capa de decisión** — 4 piezas, un solo loop → [🧭 cómo se conecta todo](docs/como-se-conecta.md). La infraestructura (Consola, DW) es el sustrato **en producción**; la App es la puerta de entrada del dato; y sobre todas vive el **núcleo de decisión**. Cada ficha en 3 niveles (visión · arquitectura · código).

| Proyecto | Qué resuelve | Ficha |
|---|---|---|
| **1 · Consola SOS** | El cerebro operativo del rescate (Salesforce) | [📄 ficha](docs/proyectos/consola-sos.md) |
| **2 · DW · Vitrina Pública** | Cifras reales por caso → transparencia | [📄 ficha](docs/proyectos/dw-vitrina-publica.md) |
| **3 · App SOS + Consola de Priorización** | La puerta de entrada del dato + el despacho | [📄 ficha](docs/proyectos/app-sos-priorizacion.md) |
| **4 · DW · Dataset de Reportes Ciudadanos** | El activo de investigación (datos + ML) · 🚧 | [📄 ficha](docs/proyectos/dw-dataset-reportes-ciudadanos.md) |

## 🔑 Acceso al código

El código vive en repos privados; se solicita **por componente** desde su ficha (botón "Solicitar acceso"). Te reviso y te agrego como **lector**. También ofrezco, bajo invitación, **acceso de lectura a Atlassian** (Confluence + Jira) para mostrar la gestión real: backlog, runbooks, ADRs y evolución de decisiones. · 📧 [vladislav@marinovich.co](mailto:vladislav@marinovich.co)

---

<div align="center">
<sub>Construido con 🧡 por los <strong>42 casos</strong> que hemos atendido hasta hoy — y por las <strong>decenas de miles de vidas</strong> que vamos a salvar.</sub>
<br/>
<sub><a href="https://marinovich.co">Marinovich Consulting</a> · Bogotá</sub>
</div>
