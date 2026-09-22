# AI Automation Avanzado — Proyecto Final

Proyecto integrador del curso **IA / Automatización Avanzada (CoderHouse)**.

Este repositorio contiene un único proyecto que **evoluciona módulo a módulo**: cada checkpoint parte del flujo del módulo anterior y le suma nuevas capacidades (memoria, integraciones, RAG, voz, etc.) hasta llegar al Proyecto Final Integrador (M11).

**Autor:** Tomás Violini · **Plataforma:** n8n (self-hosted vía Docker)

---

## 🎯 El proyecto

Un **agente de IA para calificación de leads comerciales** orientado a una consultora de automatización e IA que da servicios a PyMEs y concesionarias de la zona de La Plata, Argentina.

El agente recibe consultas entrantes desestructuradas, interpreta la necesidad del potencial cliente, clasifica el lead (CALIENTE / TIBIO / FRÍO) según intención de compra y urgencia, lo registra en una base de datos, y notifica al operador humano.

---

## 📚 Roadmap de módulos

| Módulo | Estado | Qué agrega |
|--------|--------|------------|
| **M1** — Agente base | ✅ | Agente autónomo: Chat Trigger → AI Agent (Gemini) → Airtable + log |
| **M2** — Orquestación multi-agente | ✅ | Patrón Manager-Worker con sub-workflows |
| M3 → M11 | ⏳ | Memoria, integraciones, RAG, voz, … |

---

## 🧩 Módulo 1 — Agente base

Agente autónomo individual en un solo lienzo:

- **Chat Trigger** → **AI Agent** (Tools Agent, modelo Gemini) → tool **Airtable** (guardar lead)
- **Log Gmail** de observabilidad

Archivo: `checkpoint1_tomas_violini.json`

---

## 🧠 Módulo 2 — Orquestación Multi-Agente (Manager-Worker)

Se toma el agente del M1 y se lo reorganiza en el patrón **Manager-Worker** con sub-workflows independientes, rompiendo el antipatrón del "workflow mono-bloque".

### Cómo funciona

- El **Manager** recibe el mensaje, un AI Agent (Gemini) clasifica la intención en una taxonomía cerrada y un nodo Switch delega la tarea al Worker correspondiente vía **Execute Workflow** (con *Wait for child to finish*).
- **Worker 1 — Calificar & Guardar Lead:** califica el lead y lo guarda en Airtable (reutiliza la lógica del M1).
- **Worker 2 — Redactar Respuesta:** redacta un mail de seguimiento según el score.
- **Log de trazabilidad:** nodo Gmail final que registra worker invocado, parámetros y respuesta.

### Taxonomía de intenciones

`CALIFICAR_LEAD` · `REDACTAR_RESPUESTA` · `FALLBACK_HUMANO` (vía de escape ante dudas)

### Contrato de datos

Cada Worker devuelve un JSON estandarizado:

```json
{ "status": "success", "worker": "calificar_lead", "data": { "lead_id": "...", "score": 70, "clasificacion": "CALIENTE" } }
```

Ante fallos, cada Worker devuelve `{ "status": "error", ... }` mediante un Error Trigger, y el Manager continúa sin bloquearse.

### Archivos del módulo

```
/M2-multiagente
  ├── preentrega_modulo2_violini_tomas.pdf
  ├── manager_modulo2.json
  ├── worker1_calificar_lead.json
  └── worker2_redactar_respuesta.json
```

### Para importar en n8n

1. Importar primero los dos Workers, luego el Manager.
2. Reasignar credenciales en cada nodo (Gemini, Airtable, Gmail).
3. En el Manager, verificar que los nodos Execute Workflow apunten al Worker correcto y tengan *Wait for child to finish* activado.

---

## 🛠️ Stack

- n8n self-hosted (Docker, localhost)
- Google Gemini (LLM)
- Airtable (persistencia de leads)
- Gmail (observabilidad / notificaciones)
