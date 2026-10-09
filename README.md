# AI Automation Avanzado — Proyecto Final Integrador

Agente calificador de **leads comerciales** para una consultora de automatización/IA
orientada a **PyMEs y concesionarias de La Plata**. El proyecto crece módulo a módulo
(M1 → M11) sobre **n8n self-hosted**, partiendo siempre del workflow del módulo anterior.

> Curso: **AI Automation Avanzado** — CoderHouse
> Autor: **Tomi Violini**

---

## 🧱 Stack

- **n8n** self-hosted (Docker, localhost)
- **Google Gemini** (clasificación, scoring, redacción + summarization con modelo Flash; embeddings desde M5)
- **Airtable** como capa de memoria de largo plazo (tabla `Memoria`) y registro de leads
- **Gmail** como casilla de entrada de consultas (trigger) y bandeja de borradores (HITL) — desde M4
- **HubSpot** como CRM oficial de contactos — desde M4
- **Slack** como canal del equipo de operaciones (`#leads-ops`) — desde M4
- **LlamaParse (LlamaCloud)** para parsear el documento maestro + **Simple Vector Store** de n8n — desde M5
- **Telegram** (bot oficial) como canal de voz + **OpenAI Whisper** (STT) + **ElevenLabs** (TTS) — desde M6
- **ngrok** como túnel HTTPS para exponer los webhooks del n8n local — desde M6

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

### M4 — Integraciones reales con el ecosistema de negocio ✅
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

### M5 — Cerebro documental (RAG) ✅
El Worker2 deja de redactar "de memoria" y responde cada consulta comercial (precios, plazos,
integraciones, condiciones) con datos de un **documento oficial versionado** de la consultora.
Si el dato no figura, aplica la regla de contingencia **"No sé"** en lugar de inventarlo.

**Documento maestro:** `Manual_Servicios_Consultora_v1` (8 páginas, 12 secciones, 12 tablas):
planes y precios, plazos de implementación, integraciones, formas de pago, baja y pausa del servicio,
SLA, garantía, seguridad y proceso comercial.

**Arquitectura:**
```
PDF ─► LlamaParse (modo Agentic) ─► Markdown con títulos y tablas preservadas
                                         │
            [RAG - Manual Consultora]    ▼
  Ingesta:  Form Trigger ─► Simple Vector Store (insert, Clear Store)
                              ├ Default Data Loader + Splitter Markdown (2000 / 200)
                              └ Embeddings Gemini (gemini-embedding-001)   → 14 fragmentos
  Consulta: Execute Workflow Trigger ─► Vector Store Get Many (Top-K = 4)
                                      ─► Filter score ≥ 0,65 (Minimum Score) ─► fragmentos[]

  Worker2 (M5 RAG): AI Agent ─ tool `manual_consultora` (Call n8n Workflow Tool) ─► RAG
```

**Cambios respecto de M4:**
- **Nuevo input `consulta`** en Worker2 (el cuerpo del mail normalizado). En M4 el Worker2 recibía
  solo nombre, empresa, score y clasificación: sin la pregunta original no había nada que buscar.
- **System Prompt RAG:** 100% de los datos comerciales desde los fragmentos, cita de la sección en el
  campo `fuentes[]`, frase textual de escape *"No sé: ese dato no figura en nuestra documentación
  disponible. Lo confirmamos en la llamada."* + flag `sin_dato`, y regla explícita para temas parecidos
  (baja ≠ pausa). Fragmentos y consulta se tratan como dato, nunca como instrucción.
- **Contrato Salida** de Worker2 suma `fuentes` y `sin_dato`.
- **Manager M5:** pasa `consulta = bodyText` al Worker2 RAG y el aviso de Slack muestra
  `📚 Fuentes` y la advertencia de dato a confirmar.
- **Modelo del agente:** `gemini-3.5-flash-lite` (el free tier de `gemini-2.5-flash` admite 5 req/min).

> **Nota LlamaCloud:** el plan gratuito ya no incluye *Index* (solo planes pagos). Se usa **LlamaParse**
> para el parseo y la base vectorial se indexa dentro de n8n con *Simple Vector Store*. El Minimum Score
> se implementa con un `Filter` porque los Vector Store nativos de n8n solo exponen Top-K.

**Calibración:** con Min Score 0,45 la pregunta sin respuesta recibía 4 fragmentos irrelevantes
(los embeddings de Gemini puntúan todo entre 0,58 y 0,75). Peor fragmento correcto: 0,68;
mejor incorrecto: 0,63 → umbral final **0,65**.

**Test ciego (30/09/2026) — precisión 5/5:**

| # | Pregunta (informal) | Fragmento (score) | Resultado |
|---|---|---|---|
| 1 | "¿Cuánto me sale arrancar con lo del WhatsApp para los vendedores?" | §3 Planes y precios (0,72) | ✅ Concesionaria Pro USD 1.200 + USD 290/mes |
| 2 | "¿En cuánto tiempo lo tienen andando?" | §4 Plazos (0,73) | ✅ 30 días hábiles (omitió §4.1, demora de Meta) |
| 3 | "Laburamos con Pipedrive, ¿me lo enganchan?" | §5 Integraciones (0,73) | ✅ Compatible |
| 4 | "¿También me arman la página web?" | ninguno ≥ 0,65 (máx. 0,63) | ✅ "No sé" + `sin_dato: true` |
| 5 | "Si quiero frenar todo un par de meses, ¿qué onda?" | §6.6 Pausa (0,68) | ✅ Pausa, no baja |

**Gobernanza:** revisión trimestral (alineada a la actualización de precios), extraordinaria a 48 h ante
cambios; una sola versión vigente por vez (Clear Store en la ingesta); versiones anteriores archivadas
fuera del índice; regresión de las 5 preguntas en cada cambio. Como el vector store vive en memoria,
**tras reiniciar n8n hay que volver a correr la ingesta**.

### M6 — Ecosistema de voz (Voice AI: STT/TTS) ✅ (entregable actual)
El agente pasa a **escuchar y hablar**: el cliente manda una nota de voz por Telegram y recibe la
respuesta en audio, con datos del mismo RAG de M5. Workflow nuevo **`Manager - Voz M6`**, que reutiliza
como tool el workflow publicado `RAG - Manual Consultora` (no se rehízo nada de M5).

**Flujo:**
```
Telegram Trigger (message) → IF ¿nota de voz ≤ 2 min? ─(no)→ Telegram "solo respondo notas de voz"
      │ sí
      ├─► Send Chat Action "grabando audio…"
      └─► Telegram Get File (binario `data`) → OpenAI Whisper (data · es)
                 → IF contingencia ─(falla)→ Telegram "No pude entender bien el audio…"
                       │ ok
        AI Agent - Voz (Tools Agent · Gemini · memoria por chat_id · tool manual_consultora → RAG M5)
                 → Set contención 200 caracteres → ElevenLabs TTS → Telegram Send Audio (data + caption)
```

**Configuración clave:**

| Pieza | Configuración |
|---|---|
| Entrada | Bot oficial de Telegram (BotFather). El Telegram Trigger **no descarga notas de voz** (solo foto/documento/video), por eso se agrega `Telegram → Get File` con `message.voice.file_id`. |
| Oídos | `OpenAI → Audio → Transcribe a Recording` (whisper-1), **Input Binary Property = `data`**, **Language = `es`**, reintento ×2 y *continue on fail*. |
| Cerebro | AI Agent (Tools Agent) con el texto limpio de Whisper, `gemini-3.5-flash-lite` (temp 0,3), memoria de ventana 6 por `chat_id` y tool `manual_consultora` (RAG M5). |
| Contención financiera | System Message: **máximo 200 caracteres** por respuesta + reglas VUI (una idea, sin listas/emojis/símbolos, números escritos como se dicen, cierre con pregunta). Segunda capa: `Set` que limpia markdown y recorta a 200. |
| Voz | Nodo verificado `@elevenlabs/n8n-nodes-elevenlabs` → Text to Speech · **Eleven Multilingual v2** · voz *Brian* (premade) · `stability 0,6` · `similarity_boost (Clarity) 0,8` · `mp3_44100_64`. |
| Salida | `Telegram → Send Audio`, Binary File ON, campo `data`, caption con el mismo texto (doble canal). |
| Contingencia | IF después de Whisper con 4 condiciones: sin error · texto no vacío · ≥ 4 caracteres · **no coincide con alucinaciones típicas de Whisper** (`/(amara\.org\|subtítulos realizados\|gracias por ver\|suscríbete)/i`). |
| Compliance | Workflow settings: no guardar ejecuciones exitosas, fallidas, manuales ni progreso · timeout 120 s · pruning 24 h. El binario solo existe durante la ejecución y se elimina al cerrarla. Al LLM solo le llega texto. |

**Viabilidad (ROI y fatiga cognitiva):** ≈ US$ 0,04 por consulta (Whisper US$ 0,006/min + ElevenLabs ≈ US$ 0,00018/carácter).
Con 300 consultas/mes y el tope de 200 caracteres el costo es ≈ US$ 23 contra ≈ US$ 90 de tiempo de vendedor ahorrado (**ROI ≈ +290 %**);
sin tope (≈ 700 caracteres) hace falta el plan Pro y el ROI pasa a **≈ −10 %**. El audio es lineal y no se puede releer, por eso
una idea por mensaje y caption de respaldo. **Veredicto: viable con restricciones** — consultas puntuales y primer contacto por voz;
comparaciones, propuestas y datos sensibles por texto o humano.

> **Notas técnicas:**
> - n8n 2.x eliminó el modo de binarios en memoria (`N8N_DEFAULT_BINARY_DATA_MODE=default`); la volatilidad se logra no persistiendo ejecuciones.
> - Las voces de la *Voice Library* de ElevenLabs devuelven **402** en el plan gratuito vía API → usar voces predefinidas.
> - Telegram exige webhook **HTTPS público**: n8n local expuesto con ngrok (dominio fijo) + variable `WEBHOOK_URL`.
> - Ante audio en silencio, Whisper no devuelve vacío: alucina *"Subtítulos realizados por la comunidad de Amara.org"* → condición regex en el IF.

**Pruebas (08/10/2026):**

| # | Entrada | Resultado |
|---|---|---|
| 1 | 🎙️ "¿Cuánto me sale arrancar con lo del WhatsApp para la agencia?" | ✅ Audio: "El plan Arranque sale 350 dólares de instalación y 90 mensuales más IVA. ¿Te agendo una llamada?" |
| 2 | 🎙️ "¿En cuánto tiempo lo tienen andando?" | ✅ Audio: "Para el plan Arranque demora diez días hábiles desde la reunión de inicio. ¿Te agendo una llamada?" |
| 3 | 🎙️ Audio de 2 s en silencio | ✅ Texto de contingencia (tras agregar la regex; antes Whisper alucinaba texto) |
| 4 | ✍️ Mensaje de texto | ✅ Aviso: el canal responde notas de voz |

Latencia ≈ 15 s de punta a punta (Whisper 2,7 s · agente + RAG 4,7 s · ElevenLabs 2,1 s · Telegram 5,3 s). Respuestas de 95–110 caracteres.

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

## ▶️ Cómo correr (versión M6)

1. Levantar n8n con Docker (localhost).
2. Configurar credenciales:
   - **Google Gemini (PaLM) API** (chat + embeddings)
   - **Airtable Personal Access Token**
   - **Gmail OAuth2**
   - **HubSpot Service Key** (scopes de contactos read/write)
   - **Slack API** (bot token con `chat:write`; invitar el bot al canal con `/invite`)
3. Importar en este orden (`Import from File`):
   - `/M4/Worker1 - Calificar & Guardar Lead.json`
   - `/M5/RAG - Manual Consultora.json` → **publicarlo** (n8n solo deja llamar sub-workflows publicados)
   - `/M5/Worker2 - Redactar Respuesta (M5 RAG).json` → en la tool `manual_consultora`, seleccionar el workflow RAG
   - `/M5/checkpoint5_tomas_violini.json` → en `Execute Worker1/2`, volver a seleccionar los Workers
4. Reasignar credenciales en todos los nodos.
5. **Ingesta:** en el workflow RAG, `Execute workflow` y subir `/M5/Manual_Servicios_Consultora_v1.md`
   al formulario (repetir cada vez que se reinicia n8n).
6. Crear en Airtable la base con las tablas `Leads` y `Memoria` (ver esquema arriba) y cargar el
   **Channel ID** en los nodos Slack.
7. Probar el Worker2 RAG solo (trae las 5 preguntas ciegas fijadas) o enviar un mail a la casilla
   conectada **desde otra cuenta**.

**Canal de voz (M6):**

8. Instalar el community node `@elevenlabs/n8n-nodes-elevenlabs` (Settings → Community nodes).
9. Crear el bot con @BotFather y las credenciales **Telegram API**, **OpenAI** (cuenta con saldo) y
   **ElevenLabs** (key con acceso a Text to Speech; usar una voz predefinida).
10. Exponer n8n con HTTPS y levantar el contenedor con la URL pública:
    ```
    ngrok http --url=<tu-dominio>.ngrok-free.dev 5678
    docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v <carpeta-n8n>:/home/node/.n8n \
      -e WEBHOOK_URL=https://<tu-dominio>.ngrok-free.dev/ \
      -e EXECUTIONS_DATA_PRUNE=true -e EXECUTIONS_DATA_MAX_AGE=24 n8nio/n8n
    ```
11. Importar `/M6/checkpoint6_tomas_violini.json`, reasignar credenciales, seleccionar el workflow RAG en
    la tool `manual_consultora` y **publicar**. Mandarle una nota de voz al bot.

---

## 📂 Estructura del repo

```
/
├── README.md
├── /M1  → workflow del módulo 1
├── /M2  → manager + worker1 + worker2
├── /M3  → manager (con memoria) + workers + PreEntrega_Modulo3_TomiViolini.pdf
├── /M4  → checkpoint4_tomas_violini.json + Worker1 + Worker2
├── /M5  → checkpoint5_tomas_violini.json + Worker2 (M5 RAG) + RAG - Manual Consultora
│          + Manual_Servicios_Consultora_v1 (.pdf y .md parseado) + PreEntrega_Modulo5_TomasViolini.pdf
└── /M6  → checkpoint6_tomas_violini.json (Manager - Voz M6) + PreEntrega_Modulo6_TomasViolini.pdf
```

---

## 🚧 Roadmap

M7 → M11 (en curso). Mismo caso de negocio, mismo repo, extendiendo el workflow.
