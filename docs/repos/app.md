<div align="center">

# 📱 App de Reporte Ciudadano

**`licencias-marinovich/sos-salvando-patitas`** · 🌐 repo público

[![Ver repo](https://img.shields.io/badge/🌐_Ver_el_repo-público-185FA5?style=for-the-badge)](https://github.com/licencias-marinovich/sos-salvando-patitas)

</div>

---

## Qué es

El **front-end ciudadano** del sistema: la pantalla con la que cualquier persona, desde el celular, reporta un animal en condición crítica. Captura **video + fotos guiadas + geolocalización + nivel de gravedad** en un flujo de 4 pasos, y entrega un panel de administración con mapa en tiempo real.

Es **el origen del dato**: donde nace la información que después recorre todo el sistema.

**Stack:** React 19 · Vite · TypeScript · Tailwind 4 · Supabase (PostgreSQL/Auth) · Azure Blob · Leaflet.

## Su lugar en el sistema

```
📱 App (este repo) → reporte (video+fotos+geo+gravedad) → Supabase + Azure Blob
                                                              │  (roadmap de convergencia)
                                                              ▼
                                  🖥 Consola SOS (Salesforce) → 📊 Warehouse → 🌐 vitrina
```

## Modelo de datos (Supabase)

`reports` (gravedad + geo + estado) · `evidences` (fotos/videos) · `status_history` (auditoría) · `profiles` (roles).

> 💡 `evidences.file_url` (foto) + `reports.severity` (etiqueta) = **par imagen→etiqueta georreferenciado** → semilla de un dataset original para triage por visión.

## Estado

- **V0.1** — MVP funcional (este repo, público).
- **V0.2** — rediseño premium en curso (nueva directiva de diseño).

## Diagramas

- [Web pública y captura (A3.1)](../atlas/README.md#a31--sistema-web-pública)

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
