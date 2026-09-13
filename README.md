# AI Automation Avanzado — Proyecto Final

Proyecto integrador del curso **IA / Automatización Avanzada (CoderHouse)**.

Este repositorio contiene un único proyecto que **evoluciona módulo a módulo**: cada checkpoint parte del flujo del módulo anterior y le suma nuevas capacidades (memoria, integraciones, RAG, voz, etc.) hasta llegar al Proyecto Final Integrador (M11).

**Autor:** Tomás Violini
**Plataforma:** n8n (self-hosted vía Docker)

---

## 🎯 El proyecto

Un **agente de IA para calificación de leads comerciales** orientado a una consultora de automatización e IA que da servicios a PyMEs y concesionarias de la zona de La Plata, Argentina.

El agente recibe consultas entrantes desestructuradas, interpreta la necesidad del potencial cliente, clasifica el lead (CALIENTE / TIBIO / FRÍO) según intención de compra y urgencia, lo registra en una base de datos, y notifica al operador humano.

---

## 🗺️ Roadmap de módulos

| Módulo | Capacidad que suma | Estado |
|--------|--------------------|--------|
| **M1** | Agente base: Trigger → AI Agent + System Prompt + 1 Tool → Observabilidad | ✅ Entregado |
| M2 | Multi-agente (Manager + Workers como sub-workflows) | ⏳ Pendiente |
| M3 | Memoria y contexto (Airtable por Session_ID) | ⏳ Pendiente |
| M4 | Integraciones reales (CRM / Calendario / Workspace vía OAuth2) | ⏳ Pendiente |
| M5 | RAG / base documental (vector store) | ⏳ Pendiente |
| M6 | Voz (STT / TTS) | ⏳ Pendiente |
| … | … hasta el Proyecto Final Integrador (M11) | ⏳ Pendiente |

---

## ✅ M1 — Agente Base y Motor de Razonamiento

**Archivo:** [`checkpoint1_tomas_violini.json`](./checkpoint1_tomas_violini.json)

### Arquitectura del flujo

```
[Chat Trigger] → [AI Agent (Tools Agent)] → [Gmail — Log Observabilidad]
                       │
        ┌──────────────┼──────────────┐
   [Chat Model]                   [Tool]
   Google Gemini              Airtable — Registrar Lead
```

### Componentes

- **Trigger:** Chat Trigger — captura el mensaje inicial desestructurado del usuario.
- **AI Agent:** configurado en modo **Tools Agent**, con **máximo 6 iteraciones** como guardrail anti-loop (protege el presupuesto de tokens).
- **Chat Model:** Google Gemini (`gemini-2.5-flash`).
- **System Prompt:** estructurado de forma modular (Rol → Ámbito → Objetivo → Reglas → Escalamiento). Define un rol acotado de "Asistente de Calificación de Leads", con restricciones explícitas sobre qué acciones NO puede realizar.
- **Tool:** `Airtable — Registrar Lead`, conectada lateralmente al puerto Tools del agente (no como nodo secuencial). Incluye una **descripción semántica extensa** que le indica al modelo en qué casos de negocio activarla de forma autónoma.
- **Observabilidad:** nodo Gmail final que envía un reporte automático con el resultado de la calificación del agente (`Execution Log`).

### Modelo de datos (Airtable)

Base: `Checkpoint1 - Calificacion Leads` · Tabla: `Leads`

| Campo | Tipo |
|-------|------|
| Nombre | Texto |
| Empresa | Texto |
| Rubro | Texto |
| Necesidad | Texto largo |
| Clasificacion | Selección (CALIENTE / TIBIO / FRIO) |
| Urgencia | Selección (Alta / Media / Baja) |
| Fecha | Fecha y hora |

### Validación

El flujo fue probado manualmente vía el chat de n8n. Ante un lead comercial con datos suficientes, el agente decide de forma probabilística abrir la rama de la herramienta, registra el lead en Airtable y envía el mail de observabilidad. Ante un mensaje ambiguo sin intención de compra, no activa la herramienta.

---

## 🔧 Cómo importar y usar

1. Importar el `.json` en n8n (`Workflows → Import from File`).
2. Reemplazar las credenciales (aparecen como placeholder por seguridad):
   - **Google Gemini** — API Key de Google AI Studio.
   - **Airtable** — Personal Access Token con scopes `data.records:read`, `data.records:write`, `schema.bases:read`.
   - **Gmail** — credencial OAuth2.
3. Verificar los IDs de Base y Tabla de Airtable en el nodo `Airtable — Registrar Lead`.
4. Probar con `Open chat` y enviar un mensaje de prueba.

> ⚠️ Las credenciales **no** se incluyen en el repositorio. El `.json` exportado contiene únicamente placeholders.

---

## 🛠️ Stack

- **n8n** (self-hosted, Docker) — orquestación
- **Google Gemini** — modelo de lenguaje
- **Airtable** — base de datos
- **Gmail** — canal de observabilidad
