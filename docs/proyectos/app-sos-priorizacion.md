<div align="center">

# 📱 App SOS + Consola de Priorización de Rescate

**Proyecto 3 del Patitas Stack** · la puerta de entrada del dato + el despacho operativo

`licencias-marinovich/sos-salvando-patitas` · 🌐 App pública · 🎨 Consola de priorización en diseño (v0.2)

</div>

---

## 🟢 Qué resuelve *(para cualquier lector)*

Hoy un animal en condición crítica en la calle **no tiene a dónde reportarse de forma trazable**. Este proyecto cierra ese hueco con dos piezas:

1. **App ciudadana** — cualquiera reporta desde el celular (video + fotos + ubicación + gravedad).
2. **Consola de Priorización** — la coordinación **ve todos los reportes en un mapa**, los **prioriza por gravedad y cercanía**, asigna unidades y, cuando procede, **convierte el reporte en un caso de rescate en Salesforce** (la [Consola SOS](consola-sos.md)).

Es el inicio del flujo: aquí **nace el dato** que después recorre todo el sistema.

## 🔵 Cómo funciona *(arquitectura)*

```
👤 Ciudadano → App (reporte) → Supabase + Azure Blob → 🗺️ Consola de Priorización
                                                              │  "Convertir en caso de rescate"
                                                              ▼
                                                    🖥️ Consola SOS (Salesforce)
```

- **App:** React 19 · Vite · TypeScript · Tailwind 4 · Supabase (Postgres/Auth) · Azure Blob · Leaflet.
- **Consola de Priorización:** vista combinada **mapa + lista**, orden por `gravedad ↓ · recencia`, filtros (gravedad, estado, tipo, fecha), cobertura multi-ciudad (Cartagena + Barranquilla). El botón **"Convertir en caso de rescate"** crea el Case en Salesforce — el puente entre el reporte ciudadano y la operación.

## ⚙️ Detalle técnico *(para ingeniería)*

### Máquina de estados del reporte

```mermaid
stateDiagram-v2
    [*] --> Pendiente: reporte ciudadano (vía App)
    Pendiente --> Asignado: asignar a unidad móvil
    Asignado --> EnGestión: rescatista en camino (ETA)
    EnGestión --> Cerrado: rescate realizado
    Pendiente --> Descartado: no procede / duplicado
    Asignado --> Descartado
    Cerrado --> [*]
    Descartado --> [*]

    note right of EnGestión
        "Convertir en caso de rescate"
        → crea el Case en Salesforce (Consola SOS)
    end note
```

Cada transición queda en `status_history` (quién, qué, cuándo) → trazabilidad completa del reporte.

### Diccionario de datos (Supabase)

| Tabla | Campos clave | Notas |
|---|---|---|
| `reports` | `id` · `animal_type` (canino/felino/ave/otro) · `severity` (baja/media/alta/crítica) · `lat`/`long` · `address` · `status` (pendiente/asignado/en_gestión/cerrado/descartado) · `anonymous_session_id` · `user_id` | reporte anónimo o autenticado |
| `evidences` | `id` · `report_id` · `file_url` · `file_type` | fotos/video en Azure Blob |
| `status_history` | `id` · `report_id` · `from`/`to` · `actor` · `at` | auditoría de transiciones |
| `profiles` | `id` · `role` (citizen / ngo_admin) | perfil extendido del usuario |

> 💡 **Para investigación:** `evidences.file_url` (foto) + `reports.severity` (etiqueta) + `lat/long` = **par imagen→etiqueta georreferenciado** → la semilla del [dataset público de reportes ciudadanos](dw-dataset-reportes-ciudadanos.md) y del triage por visión.

## 🔑 Acceso

- **App** (`sos-salvando-patitas`) → 🌐 [repo público](https://github.com/licencias-marinovich/sos-salvando-patitas).
- **Consola de Priorización** → 🎨 en diseño (v0.2). El mockup de arriba es la directiva de diseño en curso.

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
