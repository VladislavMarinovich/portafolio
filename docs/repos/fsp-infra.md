<div align="center">

# ☁️ Infraestructura como código

**`salvandopatitas/fsp-infra`** · 🔒 repo privado

[![Solicitar acceso](https://img.shields.io/badge/🔑_Solicitar_acceso_a_infra-abrir_solicitud-2ea44f?style=for-the-badge)](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Infraestructura)

</div>

---

## Qué es

El **Terraform** que describe y adopta la plataforma de datos de la Fundación en **Azure** (Brazil South): el Data Warehouse serverless (Synapse + Data Lake), el pipeline de sincronización (Container App Job) y su soporte (Key Vault, ACR, Service Principal, Log Analytics).

**Stack:** Terraform · Azure · 100% serverless / pay-per-use.

## Principios

| Principio | Detalle |
|---|---|
| **NO recrear producción** | La infra ya existe; el repo la **adopta** en el estado (`terraform import`), no la levanta de cero. `terraform plan` debe salir **sin cambios** antes de cualquier `apply`. |
| **Nunca apply sobre prod sin sandbox** | Todo cambio real se valida primero en un entorno de pruebas. |
| **Secretos solo en Key Vault** | Las credenciales (SF JWT, WP App Password, Service Principal) **no** viven en el repo ni en el estado; Terraform gestiona el vault y los permisos, nunca los valores. |
| **Estado remoto** | `azurerm` backend en el Data Lake (`tfstate`). |

## Recursos que describe

Resource Group · Storage Accounts (ADLS Gen2) · Synapse serverless · Key Vault (RBAC) · Container Registry · Service Principal + roles (AcrPull, KV Secrets User) · Container App Environment + Job (cron 8h) · Log Analytics.

## Diagramas

- [Infraestructura cloud (A5)](../atlas/README.md#a5--infraestructura-cloud-azure)

## 🔑 Acceso al código

Repo privado. **[Abre una solicitud →](https://github.com/VladislavMarinovich/portafolio/issues/new?template=solicitar-acceso.yml&title=%5BAcceso%5D+Infraestructura)** o escribe a [vladislav@marinovich.co](mailto:vladislav@marinovich.co?subject=Acceso%20Infraestructura).

---

<div align="center"><sub><a href="../../README.md">← Volver al portafolio</a></sub></div>
