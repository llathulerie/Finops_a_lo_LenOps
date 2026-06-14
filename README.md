# Syntlas Security Net — SOC On-Premise

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
![Status](https://img.shields.io/badge/Fase_1-En_Progreso-blue)
![Stack](https://img.shields.io/badge/Stack-Wazuh_%7C_Suricata_%7C_TheHive_%7C_Shuffle_%7C_MISP-green)

## Descripción

Proyecto de **implementación y gestión de un Centro de Operaciones de Seguridad (SOC)** completamente on-premise, documentado paso a paso como referencia técnica para la comunidad de ciberseguridad.

El objetivo es construir un SOC funcional utilizando herramientas open-source, desplegado sobre infraestructura física propia, y posteriormente analizar su equivalencia en los principales proveedores cloud (Azure, AWS, GCP, OCI) con comparativas FinOps transversales.

## Stack tecnológico

| Componente | Herramienta | Versión | Rol |
|---|---|---|---|
| SIEM | Wazuh | 4.14.5 | Correlación de eventos, gestión de agentes, alertas |
| IDS | Suricata | 8.0.5 | Detección de intrusiones en red (af-packet, modo IDS) |
| Case Management | TheHive | 5.5.2 | Gestión de incidentes y casos de seguridad |
| CTIA | Cortex | 3.1.8 | Análisis automatizado de observables (IoCs) |
| SOAR | Shuffle | — | Orquestación y automatización de respuesta |
| Threat Intelligence | MISP | — | Plataforma de inteligencia de amenazas |
| OS Base | Ubuntu Server | 24.04.4 LTS | Sistema operativo de los nodos del SOC |
| Contenedores | Docker Engine + Compose | 29.x / v5.x | Despliegue de TheHive, Cortex, Shuffle, MISP |
| DNS interno | Unbound | 1.19.x | Resolución interna de servicios del SOC |
| Red inter-sedes | WireGuard | — | Túneles cifrados Hub & Spoke entre sedes |

## Arquitectura del SOC

```
┌─────────────────────────────────────────────────────────────────┐
│                       RED SOC INTERNA                           │
│                                                                 │
│  ┌──────────────────────────────────┐  ┌─────────────────────┐  │
│  │  NODO PRIMARIO (SOC Core)        │  │  NODO AUXILIAR       │  │
│  │                                  │  │                      │  │
│  │  ┌────────────┐ ┌─────────────┐  │  │  ┌───────────────┐  │  │
│  │  │  Wazuh     │ │  Suricata   │  │  │  │  Shuffle SOAR │  │  │
│  │  │  Manager   │ │  IDS        │  │  │  │               │  │  │
│  │  │  Indexer   │ │  af-packet  │  │  │  ├───────────────┤  │  │
│  │  │  Dashboard │ │             │  │  │  │  MISP         │  │  │
│  │  ├────────────┤ └──────┬──────┘  │  │  │  Threat Intel │  │  │
│  │  │  Filebeat  │        │         │  │  └───────────────┘  │  │
│  │  │  eve.json  ├────────┘         │  │                      │  │
│  │  ├────────────┤                  │  │  Wazuh Agent         │  │
│  │  │  TheHive   │ ◄── pipeline ──► │  │                      │  │
│  │  │  Cortex    │   alertas/casos  │  └─────────────────────┘  │
│  │  ├────────────┤                  │                           │
│  │  │  Unbound   │                  │                           │
│  │  │  DNS SOC   │                  │                           │
│  │  └────────────┘                  │                           │
│  └──────────────────────────────────┘                           │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  FUENTES DE DATOS                                         │  │
│  │  Agentes Wazuh:                                           │  │
│  │  • Nodo primario (auto-monitoring)                        │  │
│  │  • Nodo auxiliar                                          │  │
│  │  • Endpoints remotos vía WireGuard                        │  │
│  │  Syslog:                                                  │  │
│  │  • Router Hub (RouterOS) — firewall, VPN, login, DHCP     │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

Ver también: [`Topología Hub & Spoke genérica`](./diagramas/topologia-hubandspoke-generica.md) · [`Roadmap del proyecto`](./docs/roadmap.md) · [`Integración MikroTik → Wazuh`](./docs/integracion-mikrotik-wazuh.md)

## Progreso actual — Fase 1

| Paso | Componente | Estado |
|---|---|---|
| Bloque 1 | Recableado físico y conectividad de red | ✅ Completado |
| Bloque 2 | Bastionado de servidores, DNS interno, Docker Engine | ✅ Completado |
| Bloque 3.0 | Pre-check infraestructura | ✅ Completado |
| Bloque 3.1 | Wazuh 4.14.5 all-in-one (Manager + Indexer + Dashboard) | ✅ Completado |
| Bloque 3.2 | Tuning post-instalación Wazuh (heap, DNS) | ✅ Completado |
| Bloque 3.3 | Despliegue de agentes Wazuh (local + remotos vía VPN) | ✅ Completado |
| Bloque 3.4 | Suricata 8.0.5 IDS (af-packet, interfaz de red dedicada) | ✅ Completado |
| Bloque 3.5 | Integración Suricata → Wazuh (ingestión eve.json) | ✅ Completado |
| Bloque 3.6 | TheHive 5.5.2 + Cortex 3.1.8 (Docker Compose) | ✅ Completado |
| Bloque 3.6b | Integración router Hub (RouterOS) → Wazuh via syslog | ✅ Completado |
| Bloque 3.7 | Shuffle SOAR (Docker Compose) | 🔄 Siguiente |
| Bloque 3.8 | MISP Threat Intelligence (Docker Compose) | ⏳ Pendiente |
| Bloque 3.9 | Integración pipeline Wazuh → Shuffle → TheHive | ⏳ Pendiente |
| Bloque 3.10 | Validación end-to-end del SOC completo | ⏳ Pendiente |

## Fases del proyecto

| Fase | Descripción | Estado |
|---|---|---|
| **Fase 1** | Implantación SOC On-Premise — SIEM + IDS + Case Mgmt + SOAR + TI | `EN PROGRESO` |
| **Fase 2** | Adecuación y gestión operativa (hardening, PKI, monitorización) | `PENDIENTE` |
| **Fase 3** | Análisis e implementación en Cloud con comparativas FinOps | `PENDIENTE` |

### Fase 3 — Cobertura multi-cloud

| Proveedor | Rol | Estado |
|---|---|---|
| Microsoft Azure | Eje principal | ⏳ |
| Amazon Web Services | Equivalente | ⏳ |
| Google Cloud Platform | Equivalente | ⏳ |
| Oracle Cloud Infrastructure | Equivalente | ⏳ |

Cada sub-fase incluirá una **comparativa FinOps transversal** para evaluar costos reales frente al modelo on-premise.

## Sobre este repositorio

Este es el **repositorio público** del proyecto. Contiene documentación de referencia, decisiones arquitectónicas y diagramas sanitizados. Todo el direccionamiento IP real, credenciales, configuraciones de producción y scripts operativos se mantienen exclusivamente en el repositorio privado.

| Disponible aquí | No disponible aquí |
|---|---|
| Decisiones arquitectónicas y justificaciones | Configuraciones con direccionamiento real |
| Stack tecnológico con versiones | Credenciales, API keys, claves VPN |
| Diagramas de referencia sanitizados | Configuraciones de equipos de red |
| Comparativas FinOps (futuro) | Scripts con datos de producción |
| Guías de despliegue genéricas | Datos de agentes o endpoints reales |

## Licencia

[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Puedes compartir y adaptar este material para fines no comerciales, siempre que des crédito al autor y distribuyas las contribuciones derivadas bajo la misma licencia.

© 2026 Syntlas Security Net — Lenry Lathulerie
