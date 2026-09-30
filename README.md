# AI Automation Avanzado — Proyecto Final Integrador

Agente calificador de **leads comerciales** para una consultora de automatización/IA
orientada a **PyMEs y concesionarias de La Plata**. El proyecto crece módulo a módulo
(M1 → M11) sobre **n8n self-hosted**, partiendo siempre del workflow del módulo anterior.

> Curso: **AI Automation Avanzado** — CoderHouse
> Autor: **Tomi Violini**

---

## 🧱 Stack

- **n8n** self-hosted (Docker, localhost)
- **Google Gemini** (clasificación, scoring, redacción + summarization con modelo Flash)
- **Airtable** como capa de memoria de largo plazo (tabla `Memoria`) y registro de leads
- **Gmail** como casilla de entrada de consultas (trigger) y bandeja de borradores (HITL) — desde M4
- **HubSpot** como CRM oficial de contactos — desde M4
- **Slack** como canal del equipo de operaciones (`#leads-ops`) — desde M4

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

### M3 — Memoria de Largo Plazo persistente ✅
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

### M4 — Integraciones reales con el ecosistema de negocio ✅ (entregable actual)
El agente sale del chat de prueba y se conecta con **tres herramientas reales**:
**Gmail** (casilla de soporte), **HubSpot** (CRM) y **Slack** (canal de operaciones).
Se parte del Manager de M3: la memoria, el Router y los Workers se reutilizan sin rehacerse.

**Flujo:**
```
Gmail Trigger → ① IF ¿auto-reply? ─(sí)→ Stop
                     │ no
              Entrada Normalizada → [Memoria M3] → Router → Switch
   ┌───────────────────┬───────────────────────┬─────────────────────┐
CALIFICAR_LEAD     REDACTAR_RESPUESTA      FALLBACK_HUMANO
Worker1 → Worker2   Worker2                 Set Alerta → Slack 🚨 (sin borrador)
        └─────────┬────────┘
     ④ Set Limpieza Payload → Filter email válido
                  │
     ② HubSpot Search → IF ¿existe? → Actualizar / Crear contacto
                  │
     ③ Gmail Create Draft (HITL) → Set Resumen → Slack 🟢
```

**Los 4 nodos de control de la rúbrica:**

| # | Nodo | Qué previene |
|---|---|---|
| ① | `IF - ¿Es auto-reply?` inmediatamente después del trigger. Condiciones OR, *ignore case*: asunto con `auto-reply`, `automatic reply`, `out of office`, `undeliverable`, `respuesta automática`; remitente con `no-reply`, `noreply`, `mailer-daemon` o la propia casilla. Rama true → `Stop - Auto-reply`. | Bucle infinito de auto-respuestas |
| ② | `HubSpot - Buscar Contacto` (Search por email, *Always Output Data*) → `IF - ¿Existe en CRM?` → `Actualizar Contacto` / `Crear Contacto`. | Error 409 (contactos duplicados) |
| ③ | `Gmail - Crear Borrador (HITL)`: operación **Create Draft** únicamente, dentro del hilo original del cliente. No existe ningún nodo Send en el flujo. | Envíos automáticos sin revisión humana |
| ④ | `Set - Limpieza Payload` (keep only: from, nombre, empresa, subject, threadId, intención, score, clasificación, borrador) + `Filter - Email válido` (no vacío y con `@`). Además, un `Set` de resumen de una línea antes de cada nodo Slack. | Error 400 (payload mal formado) y saturación del canal con HTML/binarios |

**Otros cambios respecto de M3:**
- **Entrada por mail:** `Gmail Trigger` (sin descarga de adjuntos) reemplaza al `Chat Trigger`.
  El nodo `Entrada Normalizada` deja solo 7 campos limpios y el cuerpo sin URLs, recortado a 3000 caracteres.
- **`Session_ID` = email del remitente** (lowercase): el mismo cliente conserva su memoria entre mails.
- **Anti prompt-injection en el Router:** el correo entrante se encapsula entre
  `[INICIO EMAIL] ... [FIN EMAIL]` y se declara como dato, nunca como instrucción.
- **Regla de fallback reforzada:** si el mensaje no describe una necesidad concreta ("hola", "una consulta"),
  el Router responde `FALLBACK_HUMANO` aunque el contacto tenga historial en memoria.
- **Worker1 → Worker2 encadenados:** un lead calificado recibe automáticamente un borrador de respuesta
  acorde a su score. El `Contrato Salida` de Worker2 ahora devuelve también `score` y `clasificacion`.
- **Se eliminó el log por Gmail de M2:** enviaba mails a la misma casilla que ahora lee el trigger (riesgo de bucle).
  La notificación operativa pasa a Slack.

**Seguridad y mínimo privilegio:**

| Herramienta | Autenticación | Permisos |
|---|---|---|
| Gmail | OAuth2 (Google Cloud) | Solo Trigger (lectura) y Create Draft. Sin operación Send en el flujo. |
| HubSpot | Service Key | `crm.objects.contacts.read` + `crm.objects.contacts.write`. *Allowed HTTP Request Domains:* `api.hubapi.com` |
| Slack | Bot token emitido por el flujo OAuth2 de instalación de la app | Único scope `chat:write`. *Allowed HTTP Request Domains:* `slack.com`. Canal configurado por ID (la app no tiene `channels:read`). |

> **Nota HubSpot:** HubSpot discontinuó la creación de apps OAuth "anteriores" (legacy) para cuentas nuevas;
> las alternativas son la Service Key o apps basadas en proyectos vía CLI. Se eligió la **Service Key**,
> método recomendado actualmente por el proveedor, acotada a los dos scopes de contactos.

**Test de regresión (29/09/2026):**

| # | Caso | Resultado esperado | Estado |
|---|---|---|---|
| 0 | Notificación automática de sistema (remitente no-reply) | IF true → Stop | ✅ |
| 1 | Lead nuevo real ("Consulta automatización concesionaria") | CALIFICAR_LEAD · CALIENTE 92/100 · HubSpot **crea** contacto · borrador en el hilo · aviso 🟢 en Slack | ✅ |
| 2 | Mismo remitente, mismo mail | Memoria lo reconoce como recurrente · HubSpot **actualiza** el mismo contacto (mismo ID, sin duplicado ni 409) | ✅ |
| 3 | Mail ambiguo ("hola / una consulta") | FALLBACK_HUMANO · alerta 🚨 en Slack · sin HubSpot ni borrador | ✅ (tras ajustar el prompt del Router) |
| 4 | Mail de persona real con asunto "Out of Office" | IF true → Stop, sin ejecutar ningún nodo posterior | ✅ |

---

## 🗃️ Esquema de la base de memoria (tabla `Memoria`)

Base **Checkpoint1 - Calificacion Leads** (misma base que la tabla `Leads`).

| Campo | Tipo Airtable | Rol |
|---|---|---|
| `Session_ID` | Single line text (primario) | Clave de correlación (desde M4: email del remitente) |
| `user_name` | Long text | Nombre del lead |
| `Estado del Caso` | Single select | `NUEVO`, `EN_CALIFICACION`, `RESPUESTA_ENVIADA`, `FALLBACK_HUMANO`, `CERRADO` |
| `Resumen Consolidado` | Long text | `last_summary` (JSON del summarizer, idempotente) |
| `Datos Clave` | Long text | Indicadores accionables (rubro, presupuesto, urgencia) |
| `Fecha de Actualización` | Date (con hora) | Timestamp de última escritura |
| `msg_count` | Number (entero) | Contador de mensajes; dispara summarization al llegar a 5 y se resetea a 0 |

---

## ▶️ Cómo correr (versión M4)

1. Levantar n8n con Docker (localhost).
2. Configurar credenciales:
   - **Google Gemini (PaLM) API**
   - **Airtable Personal Access Token**
   - **Gmail OAuth2**
   - **HubSpot Service Key** (scopes de contactos read/write)
   - **Slack API** (bot token con `chat:write`; invitar el bot al canal con `/invite`)
3. Importar desde la carpeta `/M4` (`Import from File`): `Worker1`, `Worker2` y `checkpoint4_tomas_violini.json`.
   Reasignar credenciales y, si cambian los IDs, volver a seleccionar los Workers en los nodos `Execute Worker1/2`.
4. Crear en Airtable la base con las tablas `Leads` y `Memoria` (ver esquema arriba).
5. En los nodos Slack, cargar el **Channel ID** del canal de operaciones.
6. Enviar un mail a la casilla conectada **desde otra cuenta** y ejecutar el workflow.

---

## 📂 Estructura del repo

```
/
├── README.md
├── /M1  → workflow del módulo 1
├── /M2  → manager + worker1 + worker2
├── /M3  → manager (con memoria) + workers + PreEntrega_Modulo3_TomiViolini.pdf
└── /M4  → checkpoint4_tomas_violini.json + Worker1 + Worker2
```

---

## 🚧 Roadmap

M5 → M11 (en curso). Mismo caso de negocio, mismo repo, extendiendo el workflow.
