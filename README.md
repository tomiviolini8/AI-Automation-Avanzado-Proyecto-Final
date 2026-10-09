# M7 — Diseño arquitectónico de un sistema agéntico vertical (Sales Ops)

Documento de diseño (100 % no-code) que especializa el agente construido en M1–M6 para la vertical
**Sales Ops**: captación, calificación y seguimiento de oportunidades B2B de una consultora de
automatización e IA (PyMEs y concesionarias de La Plata). No agrega workflows nuevos: define cómo
evoluciona el sistema existente.

📄 Informe completo: [`PreEntrega_Modulo7_TomasViolini.pdf`](./PreEntrega_Modulo7_TomasViolini.pdf)

![Arquitectura multi-agente](./M7_arquitectura_multiagente.png)

## Contenido del informe

1. **Relevamiento y priorización**: baseline manual (20 leads/mes, 30 min por lead, 10 h/mes,
   primera respuesta ≈ 24 h, cierre 15 %) y framework de priorización (impacto 40 % · viabilidad
   no-code 30 % · adopción 30 %).
2. **Arquitectura multi-agente**: orquestador determinístico + 6 agentes especialistas
   (Router, Calificador, Redactor RAG, Voz, Seguimiento y Objeciones, Auditor de CRM y Atribución),
   fichas por agente con System Prompt y contrato JSON, mapa de permisos de mínimo privilegio y
   estrategia de Context Engineering por rol.
3. **Propuesta de valor y riesgo**: KPIs a 90 días, semáforo operativo (autónomo / aprobación humana /
   congelado y escalado) y protocolo de escalado forzado con botones interactivos en Slack ante churn,
   reembolsos, reclamos legales o cambios de precio.
4. **Scorecards**: rúbrica de calificación de 100 puntos basada en evidencia textual, playbook de acción
   por clasificación, control de calidad de borradores y versión de portafolio anonimizada.

## Red de agentes

| Agente | Rol | Estado |
|---|---|---|
| A1 Router | Clasifica la intención (taxonomía cerrada) y marca `senal_critica` | Existente (M4) |
| A2 Calificador | Score 0–100 y clasificación CALIENTE / TIBIO / FRÍO con evidencia | Existente (M4) |
| A3 Redactor RAG | Borrador con fuentes del manual comercial y regla "No sé" | Existente (M5) |
| A4 Agente de Voz | Respuestas por nota de voz en Telegram (≤ 200 caracteres) | Existente (M6) |
| A5 Seguimiento y Objeciones | Recordatorios a leads TIBIO y respuesta a objeciones | Diseño (M7) |
| A6 Auditor de CRM y Atribución | Reporte semanal de duplicados, datos faltantes y canal de origen | Diseño (M7) |

## KPIs definidos (meta a 90 días)

| KPI | Baseline | Meta |
|---|---|---|
| Tiempo hasta la primera respuesta | ≈ 24 h | < 2 h hábiles |
| Tasa de cierre | 15 % | 20 % |
| Horas mensuales de gestión comercial | 10 h | ≤ 2 h |
| Leads tibios con seguimiento en ≤ 3 días | Sin medición | 100 % |
| Resolución autónoma | 0 % | ≥ 80 % |
| Errores de precio o condiciones | Sin medición | 0 |
| Escalados críticos atendidos | — | 100 % en < 4 h hábiles |

## Principios de diseño

- Los agentes proponen y el orquestador ejecuta: ningún agente de IA tiene permisos de escritura.
- Gmail sin envío automático: toda comunicación con clientes requiere aprobación humana (HITL).
- Taxonomía de intenciones cerrada (`CALIFICAR_LEAD`, `REDACTAR_RESPUESTA`, `FALLBACK_HUMANO`);
  las situaciones críticas viajan en un campo aparte (`senal_critica`).
- Minimización de datos: cada sub-workflow recibe solo el contexto que necesita para su decisión.

## Archivos

| Archivo | Descripción |
|---|---|
| `PreEntrega_Modulo7_TomasViolini.pdf` | Informe de consultoría completo (entregable) |
| `M7_arquitectura_multiagente.png` | Diagrama de la coreografía multi-agente |

## Próximos pasos (M8+)

Implementar en n8n el agente de seguimiento (A5), el auditor de CRM (A6) y la interactividad de Slack
para los botones de escalado.
