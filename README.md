# AI Automation Avanzado — Proyecto Final Integrador

Agente calificador de **leads comerciales** para una consultora de automatización/IA
orientada a **PyMEs y concesionarias de La Plata**. El proyecto crece módulo a módulo
(M1 → M11) sobre **n8n self-hosted**, partiendo siempre del workflow del módulo anterior.

> Curso: **AI Automation Avanzado** — CoderHouse
> Autor: **Tomi Violini**

---

## 🧱 Stack

- **n8n** self-hosted (Docker, localhost)
- **Google Gemini** (clasificación + summarization con modelo Flash)
- **Airtable** como base de datos relacional (leads + memoria)
- **Gmail / Sheets** para logging y observabilidad

---

## 🗺️ Evolución por módulos

### M1 — Agente base ✅
Flujo mínimo funcional: `Chat Trigger → AI Agent (Tools Agent, Gemini) → tool Airtable`,
con log en Gmail para observabilidad. Califica un lead y lo guarda.

### M2 — Arquitectura multi-agente (Manager-Worker) ✅
- **Manager** clasifica la intención del mensaje y delega en un Worker vía
  `Execute Workflow` (Wait for child).
- **Worker 1 — Calificar & Guardar Lead:** reutiliza la lógica de M1 + Airtable.
- **Worker 2 — Redactar Respuesta:** genera un mensaje de seguimiento.
- **Log final** (Gmail/Sheets) con worker invocado + parámetros + respuesta.

**Taxonomía cerrada de intenciones:** `CALIFICAR_LEAD` · `REDACTAR_RESPUESTA` · `FALLBACK_HUMANO`

### M3 — Memoria de Largo Plazo persistente ✅ (entregable actual)
Capa de persistencia correlacionada por **`Session_ID`** que erradica la amnesia entre
ejecuciones. Se inserta en el Manager, entre el trigger y el agente.

**Circuito:**
1. **Lectura (post-trigger):** `Airtable → Search Records` filtrando por `Session_ID`
   (con *Always Output Data = ON*). Un `IF` bifurca según exista o no registro.
2. **Rama nuevo:** `Create Record` inicial limpio (`Estado del Caso = NUEVO`, `msg_count = 1`),
   sin variables vacías.
3. **Rama recurrente:** inyecta `user_name` / `Estado del Caso` / `Resumen Consolidado`
   en el **System Prompt** del agente, encapsulados con delimitadores rígidos
   `[INICIO DE CONTEXTO COMPARTIDO] ... [FIN DEL CONTEXTO COMPARTIDO]` (anti prompt-injection).
   En paralelo, un `Update` incrementa `msg_count`.
4. **Summarization:** al alcanzar 5 mensajes, un `IF` dispara un LLM económico
   (**Gemini Flash**) que devuelve un JSON `{ asunto_principal, puntos_clave[], accion_requerida }`.
   Un `Update Record` idempotente persiste el resumen y resetea `msg_count = 0`.
   Regla anti-ruido: solo resumen analítico + indicadores; prohibido HTML/logs/transcripciones crudas.

---

## 🗃️ Esquema de la base de memoria (tabla `Memoria`)

Base **Checkpoint1 - Calificacion Leads** (misma base que la tabla `Leads`).

| Campo | Tipo Airtable | Rol |
|---|---|---|
| `Session_ID` | Single line text (primario) | Clave de correlación por sesión |
| `user_name` | Long text | Nombre del lead |
| `Estado del Caso` | Single select | `NUEVO`, `EN_CALIFICACION`, `RESPUESTA_ENVIADA`, `FALLBACK_HUMANO`, `CERRADO` |
| `Resumen Consolidado` | Long text | `last_summary` (JSON del summarizer, idempotente) |
| `Datos Clave` | Long text | Indicadores accionables (rubro, presupuesto, urgencia) |
| `Fecha de Actualización` | Date (con hora) | Timestamp de última escritura |
| `msg_count` | Number (entero) | Contador de mensajes; dispara summarization al llegar a 5 y se resetea a 0 |

---

## ▶️ Cómo correr

1. Levantar n8n con Docker (localhost).
2. Configurar credenciales: **Google Gemini (PaLM) API** y **Airtable Personal Access Token**.
3. Importar los workflows (`Importar from File`) desde la carpeta del módulo.
4. Crear en Airtable la base con las tablas `Leads` y `Memoria` (ver esquema arriba).
5. Abrir el chat del `Chat Trigger` y probar.

---

## 📂 Estructura sugerida del repo

```
/
├── README.md
├── /M1  → workflow del módulo 1
├── /M2  → manager + worker1 + worker2
└── /M3  → manager (con memoria) + workers + PreEntrega_Modulo3_TomiViolini.pdf
```

---

## 🚧 Roadmap

M4 → M11 (en curso). Mismo caso de negocio, mismo repo, extendiendo el workflow.
