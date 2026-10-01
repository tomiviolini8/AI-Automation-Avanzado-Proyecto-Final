# Manual de Servicios y Políticas Comerciales

**Consultoría en Automatización e IA – La Plata**

Documento maestro institucional · Versión 1.0 · Vigente desde el 01/10/2026

Uso: base de conocimiento oficial del equipo comercial y del agente de IA que responde consultas de leads. Todo dato comercial que se comunique a un cliente debe surgir de este documento.

## 1. Presentación de la consultora

### 1.1 Quiénes somos

Somos una consultora de La Plata especializada en automatización de procesos e inteligencia artificial aplicada para pequeñas y medianas empresas (PyMEs) y concesionarias de vehículos. Diseñamos, implementamos y mantenemos flujos automáticos que conectan las herramientas que el cliente ya usa: correo, WhatsApp, planillas y CRM.

Trabajamos con plataformas de automatización visual (n8n) y modelos de lenguaje, siempre con supervisión humana en los puntos críticos del proceso comercial.

### 1.2 Rubros que atendemos

<table>
  <thead>
    <tr>
        <th>Rubro</th>
        <th>Casos típicos</th>
        <th>Prioridad comercial</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Concesionarias de autos y motos (0 km y usados)</td>
        <td>Seguimiento de consultas de portales y WhatsApp, agenda de test drive, recordatorios de service</td>
        <td>Alta</td>
    </tr>
    <tr>
        <td>Comercios y distribuidoras PyME</td>
        <td>Respuesta de consultas, carga de pedidos, reportes de ventas</td>
        <td>Alta</td>
    </tr>
    <tr>
        <td>Estudios profesionales (contables, jurídicos)</td>
        <td>Clasificación de correos, recordatorios de vencimientos, carga de documentación</td>
        <td>Media</td>
    </tr>
    <tr>
        <td>Inmobiliarias</td>
        <td>Calificación de interesados, agenda de visitas</td>
        <td>Media</td>
    </tr>
    <tr>
        <td>Industria y logística</td>
        <td>Integración de planillas, alertas operativas, reportes</td>
        <td>Media</td>
    </tr>
  </tbody>
</table>

### 1.3 Zona de cobertura

* **Presencial:** La Plata, Berisso, Ensenada y Gran La Plata, sin costo de traslado.

* **Presencial con viático:** CABA y resto del AMBA. Se cobra un viático fijo por visita (ver sección 4.4).

* **Remoto:** todo el país, por videollamada. La implementación completa puede hacerse 100% remota.

## 2. Catálogo de servicios

Cada servicio puede contratarse por separado o dentro de un plan (sección 3). La siguiente tabla resume el alcance de cada uno.

<table>
  <thead>
    <tr>
        <th>Código</th>
        <th>Servicio</th>
        <th>Qué incluye</th>
        <th>Qué no incluye</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>S1</td>
        <td>Diagnóstico de procesos</td>
        <td>Relevamiento de 2 reuniones, mapa del proceso actual, lista priorizada de automatizaciones con ahorro estimado de horas</td>
        <td>Implementación</td>
    </tr>
    <tr>
        <td>S2</td>
        <td>Automatización de atención (WhatsApp y correo)</td>
        <td>Respuesta inicial automática, derivación a un vendedor, registro de la consulta en planilla o CRM</td>
        <td>Costo de la API de WhatsApp Business (lo paga el cliente a Meta)</td>
    </tr>
    <tr>
        <td>S3</td>
        <td>Agente calificador de leads con IA</td>
        <td>Clasificación de cada consulta (caliente, tibio, frío), borrador de respuesta para revisión humana, alerta al equipo por Slack o correo</td>
        <td>Envío automático sin revisión humana</td>
    </tr>
    <tr>
        <td>S4</td>
        <td>Integración con CRM</td>
        <td>Alta y actualización de contactos sin duplicados, sincronización de estados</td>
        <td>Licencias del CRM</td>
    </tr>
    <tr>
        <td>S5</td>
        <td>Reportes y tableros (BI)</td>
        <td>Tablero semanal de consultas, tiempos de respuesta y conversión</td>
        <td>Auditoría contable</td>
    </tr>
    <tr>
        <td>S6</td>
        <td>Capacitación del equipo</td>
        <td>Taller de 3 horas para el equipo que usa las automatizaciones, con material escrito</td>
        <td>Capacitación en herramientas de terceros no implementadas por nosotros</td>
    </tr>
  </tbody>
</table>

## 2.1 Diagnóstico de procesos (S1)

Es el punto de entrada recomendado. El entregable es un informe con las automatizaciones posibles ordenadas por impacto y esfuerzo. Si el cliente contrata un plan dentro de los 30 días posteriores al diagnóstico, el valor del diagnóstico se descuenta del setup.

## 2.2 Automatización de atención (S2)

Conecta los canales de entrada del cliente (WhatsApp Business, correo, formularios web) para que ninguna consulta quede sin respuesta. La respuesta inicial automática se limita a confirmar la recepción y pedir datos faltantes; las respuestas comerciales pasan siempre por un vendedor.

## 2.3 Agente calificador de leads con IA (S3)

El agente lee cada consulta entrante, recuerda el historial del contacto, asigna un puntaje de 0 a 100 y redacta un borrador de respuesta. El borrador queda en la bandeja del vendedor para que lo revise y lo envíe. El agente nunca envía correos ni mensajes por su cuenta.

<table>
  <thead>
    <tr>
        <th>Clasificación</th>
        <th>Puntaje</th>
        <th>Acción recomendada</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Caliente</td>
        <td>70 a 100</td>
        <td>Contactar el mismo día hábil</td>
    </tr>
    <tr>
        <td>Tibio</td>
        <td>40 a 69</td>
        <td>Contactar dentro de 48 horas hábiles</td>
    </tr>
    <tr>
        <td>Frío</td>
        <td>0 a 39</td>
        <td>Enviar información general y seguimiento mensual</td>
    </tr>
  </tbody>
</table>

## 2.4 Integración con CRM (S4)

Se integra con los CRM listados en la sección 5. Antes de crear un contacto, el flujo busca si ya existe para evitar duplicados.

## 2.5 Reportes y tableros (S5)

Tablero con consultas recibidas, tiempo de primera respuesta, porcentaje de leads calientes y conversión por vendedor. Se actualiza en forma diaria.

## 3. Planes y precios

Los precios están expresados en dólares estadounidenses (USD) y se facturan en pesos al tipo de cambio vendedor del Banco Nación del día anterior a la emisión de la factura. No incluyen IVA.

### 3.1 Planes mensuales

<table>
  <thead>
    <tr>
        <th>Plan</th>
        <th>Para quién</th>
        <th>Setup (pago único)</th>
        <th>Abono mensual</th>
        <th>Servicios incluidos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Plan Arranque</td>
        <td>PyMEs con un solo canal de consultas</td>
        <td>USD 350</td>
        <td>USD 90</td>
        <td>S2 en un canal + S6</td>
    </tr>
    <tr>
        <td>Plan Crecimiento</td>
        <td>PyMEs con varios canales y CRM</td>
        <td>USD 750</td>
        <td>USD 180</td>
        <td>S2 en hasta 3 canales + S3 + S4 + S6</td>
    </tr>
    <tr>
        <td>Plan Concesionaria Pro</td>
        <td>Concesionarias con equipo de ventas</td>
        <td>USD 1.200</td>
        <td>USD 290</td>
        <td>S2 multicanal + S3 + S4 + S5 + S6</td>
    </tr>
  </tbody>
</table>

### 3.2 Servicios sueltos

<table>
  <thead>
    <tr>
        <th>Servicio</th>
        <th>Precio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Diagnóstico de procesos (S1)</td>
        <td>USD 150</td>
    </tr>
    <tr>
        <td>Capacitación adicional (S6), por taller de 3 horas</td>
        <td>USD 120</td>
    </tr>
    <tr>
        <td>Hora de desarrollo fuera de plan</td>
        <td>USD 35 por hora</td>
    </tr>
    <tr>
        <td>Bolsa de 10 horas de desarrollo</td>
        <td>USD 300</td>
    </tr>
  </tbody>
</table>

### 3.3 Qué incluye el abono mensual

* Monitoreo de los flujos y corrección de fallas.

* Soporte según los tiempos de la sección 7.

* Hasta 2 horas mensuales de ajustes menores (textos, reglas de derivación, campos). Las horas no usadas no se acumulan.

* Actualización de las integraciones cuando un proveedor cambia su API.

### 3.4 Costos de terceros

El abono no incluye costos de terceros: API de WhatsApp Business, licencias de CRM, uso de modelos de IA por encima de 5.000 consultas mensuales y servidores dedicados. Estos costos se informan en la propuesta y los paga el cliente directamente al proveedor.

## 4. Plazos de implementación

Los plazos se cuentan en días hábiles desde la reunión de inicio (kickoff), siempre que el cliente haya entregado los accesos necesarios.

<table>
  <thead>
    <tr>
        <th>Etapa</th>
        <th>Plan Arranque</th>
        <th>Plan Crecimiento</th>
        <th>Plan Concesionaria<br />Pro</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Relevamiento y accesos</td>
        <td>2 días</td>
        <td>3 días</td>
        <td>5 días</td>
    </tr>
    <tr>
        <td>Desarrollo e integraciones</td>
        <td>5 días</td>
        <td>10 días</td>
        <td>15 días</td>
    </tr>
    <tr>
        <td>Pruebas con casos reales</td>
        <td>2 días</td>
        <td>4 días</td>
        <td>5 días</td>
    </tr>
    <tr>
        <td>Capacitación y puesta en marcha</td>
        <td>1 día</td>
        <td>3 días</td>
        <td>5 días</td>
    </tr>
    <tr>
        <td><strong>Total estimado</strong></td>
        <td><strong>10 días hábiles</strong></td>
        <td><strong>20 días hábiles</strong></td>
        <td><strong>30 días hábiles</strong></td>
    </tr>
  </tbody>
</table>

## 4.1 Qué puede demorar la implementación

* Demora del cliente en entregar accesos o credenciales: el plazo se corre por la misma cantidad de días.

* Aprobación de la cuenta de WhatsApp Business por parte de Meta: suele tardar entre 2 y 7 días y no depende de la consultora.

* Cambios de alcance pedidos durante el desarrollo (sección 8.2).

## 4.2 Puesta en marcha acelerada

Para el Plan Arranque existe una modalidad acelerada de 5 días hábiles con un recargo del 30% sobre el setup, sujeta a disponibilidad del equipo.

## 4.3 Horario de trabajo

El equipo trabaja de lunes a viernes de 9 a 18 horas (hora de Argentina), excepto feriados nacionales.

## 4.4 Visitas presenciales

Las visitas en Gran La Plata no tienen costo adicional. En CABA y resto del AMBA se cobra un viático fijo de USD 40 por visita.

## 5. Integraciones compatibles

<table>
  <thead>
    <tr>
        <th>Herramienta</th>
        <th>Tipo</th>
        <th>Estado</th>
        <th>Observaciones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>WhatsApp Business (API oficial de Meta)</td>
        <td>Mensajería</td>
        <td>Compatible</td>
        <td>No se trabaja con WhatsApp personal ni con soluciones no oficiales</td>
    </tr>
    <tr>
        <td>Gmail y Google Workspace</td>
        <td>Correo</td>
        <td>Compatible</td>
        <td>Lectura de consultas y creación de borradores</td>
    </tr>
    <tr>
        <td>Outlook / Microsoft 365</td>
        <td>Correo</td>
        <td>Compatible</td>
        <td>Requiere permiso del administrador de la cuenta</td>
    </tr>
    <tr>
        <td>HubSpot</td>
        <td>CRM</td>
        <td>Compatible</td>
        <td>Plan gratuito o pago</td>
    </tr>
    <tr>
        <td>Pipedrive</td>
        <td>CRM</td>
        <td>Compatible</td>
        <td>—</td>
    </tr>
    <tr>
        <td>Zoho CRM</td>
        <td>CRM</td>
        <td>Compatible</td>
        <td>—</td>
    </tr>
    <tr>
        <td>Airtable y Google Sheets</td>
        <td>Planillas / base de datos</td>
        <td>Compatible</td>
        <td>Recomendado para clientes sin CRM</td>
    </tr>
    <tr>
        <td>Slack</td>
        <td>Alertas internas</td>
        <td>Compatible</td>
        <td>Canal dedicado para avisos de leads</td>
    </tr>
  </tbody>
</table>

<table>
  <thead>
    <tr>
        <th>Herramienta</th>
        <th>Tipo</th>
        <th>Estado</th>
        <th>Observaciones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>CRM específicos de concesionarias</td>
        <td>CRM</td>
        <td>A evaluar</td>
        <td>Depende de que el sistema tenga API; se confirma en el diagnóstico</td>
    </tr>
    <tr>
        <td>Sistemas de gestión ERP a medida</td>
        <td>ERP</td>
        <td>A evaluar</td>
        <td>Se cotiza por hora de desarrollo</td>
    </tr>
  </tbody>
</table>

## 6. Condiciones comerciales

### 6.1 Formas de pago

<table>
  <thead>
    <tr>
        <th>Concepto</th>
        <th>Medios aceptados</th>
        <th>Condición</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Setup</td>
        <td>Transferencia bancaria, Mercado Pago, tarjeta de crédito</td>
        <td>50% al firmar y 50% en la puesta en marcha. Con tarjeta: hasta 3 cuotas sin interés</td>
    </tr>
    <tr>
        <td>Abono mensual</td>
        <td>Transferencia bancaria, débito automático con Mercado Pago</td>
        <td>Por mes adelantado, del 1 al 10 de cada mes</td>
    </tr>
    <tr>
        <td>Horas fuera de plan</td>
        <td>Transferencia bancaria</td>
        <td>Contra entrega del trabajo</td>
    </tr>
  </tbody>
</table>

### 6.2 Facturación

Emitimos factura A o B según la condición fiscal del cliente. La factura del abono se envía por correo el primer día hábil de cada mes.

### 6.3 Actualización de precios

Los abonos se revisan cada tres meses (enero, abril, julio y octubre). Cualquier cambio se avisa por correo con 30 días de anticipación.

### 6.4 Permanencia mínima

Los planes mensuales tienen una permanencia mínima de 3 meses desde la puesta en marcha.

### 6.5 Baja definitiva del servicio

Pasada la permanencia mínima, el cliente puede dar de baja el plan avisando por correo con 30 días de anticipación. No hay penalidad. Al dar de baja, la consultora entrega la documentación de los flujos y desactiva los accesos dentro de los 5 días hábiles siguientes. La baja antes de cumplir la permanencia mínima obliga a abonar los meses restantes hasta completarla.

### 6.6 Pausa temporal del servicio

La pausa es distinta de la baja: el cliente conserva sus flujos y configuraciones sin darlos de baja. Se puede pausar el plan hasta 60 días por año calendario, avisando con 7 días de anticipación. Durante la pausa se cobra un abono reducido del 25% del valor mensual, que cubre el resguardo de la configuración. Al reactivar no se vuelve a cobrar el setup.

### 6.7 Propiedad de los desarrollos

Los flujos desarrollados para el cliente quedan instalados en cuentas a su nombre. Si el cliente da de baja el servicio, conserva los flujos y su documentación.

## 7. Soporte y niveles de servicio (SLA)

Los tiempos de respuesta se miden en horas hábiles (lunes a viernes de 9 a 18 horas) desde que el cliente reporta el problema por un canal oficial.

<table>
  <thead>
    <tr>
        <th>Prioridad</th>
        <th>Ejemplo</th>
        <th>Primera respuesta</th>
        <th>Solución objetivo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Crítica</td>
        <td>No entra ninguna consulta o el flujo está detenido</td>
        <td>2 horas hábiles</td>
        <td>8 horas hábiles</td>
    </tr>
    <tr>
        <td>Alta</td>
        <td>Falla una integración (por ejemplo el CRM) pero el resto funciona</td>
        <td>4 horas hábiles</td>
        <td>2 días hábiles</td>
    </tr>
    <tr>
        <td>Media</td>
        <td>Un texto o regla no se comporta como se esperaba</td>
        <td>1 día hábil</td>
        <td>5 días hábiles</td>
    </tr>
    <tr>
        <td>Baja</td>
        <td>Pedido de mejora o consulta de uso</td>
        <td>2 días hábiles</td>
        <td>Se agenda en el próximo ciclo de ajustes</td>
    </tr>
  </tbody>
</table>

### 7.1 Canales de soporte

* Correo de soporte: canal oficial para todas las prioridades.

* WhatsApp de soporte: solo para prioridad crítica, en horario hábil.

* Videollamada: se agenda cuando el problema lo requiere.

### 7.2 Soporte fuera de horario

El Plan Concesionaria Pro incluye guardia para prioridad crítica los sábados de 9 a 13 horas. Los demás planes no tienen soporte fuera de horario.

## 8. Garantía y cambios de alcance

### 8.1 Garantía de funcionamiento

Durante los 30 días corridos posteriores a la puesta en marcha, cualquier falla en lo implementado se corrige sin costo, aunque el cliente no tenga abono mensual. La garantía no cubre fallas causadas por cambios hechos por el cliente o por terceros en las cuentas integradas.

### 8.2 Cambios de alcance

Todo pedido que no figure en la propuesta firmada se considera cambio de alcance. Se cotiza aparte con el valor de hora de desarrollo (sección 3.2) y se informa el impacto en el plazo antes de empezar.

### 8.3 Resultados

La consultora se compromete a entregar las automatizaciones funcionando según lo acordado. No garantiza un volumen de ventas determinado, porque la conversión depende también del equipo comercial del cliente.

## 9. Seguridad y tratamiento de datos

### 9.1 Dónde quedan los datos

Los datos de los clientes finales quedan en las cuentas del propio cliente (su correo, su CRM, sus planillas). La consultora no guarda copias de las conversaciones fuera de esas cuentas.

### 9.2 Accesos

Pedimos solo los permisos mínimos necesarios para cada integración. Por ejemplo, el agente puede leer correos y crear borradores, pero no enviarlos. Al terminar el servicio se revocan todos los accesos.

### 9.3 Supervisión humana

Ninguna respuesta comercial se envía sin revisión de una persona del equipo del cliente. Los casos que el agente no puede resolver se derivan a un humano mediante una alerta.

### 9.4 Confidencialidad

Firmamos un acuerdo de confidencialidad (NDA) con cada cliente antes del relevamiento, cuando el cliente lo solicita.

## 10. Proceso comercial

Estos son los pasos desde la primera consulta hasta el inicio del proyecto:

1. Llamada inicial gratuita de 20 minutos para entender la necesidad.

2. Diagnóstico de procesos (opcional, recomendado para Plan Crecimiento y Concesionaria Pro).

3. Envío de la propuesta comercial dentro de los 5 días hábiles posteriores a la llamada o al diagnóstico.

4. Firma de la propuesta y pago del 50% del setup.

5. Reunión de inicio (kickoff) y entrega de accesos por parte del cliente.

6. Implementación según los plazos de la sección 4.

La propuesta comercial tiene una validez de 15 días corridos desde su envío.

## 11. Preguntas frecuentes

**11.1 ¿Necesito tener un CRM para contratar?**
No. Si el cliente no tiene CRM, los contactos se registran en Airtable o Google Sheets (sección 5).

**11.2 ¿El agente de IA contesta solo a los clientes?**
No. El agente redacta un borrador y una persona lo revisa y lo envía (sección 9.3).

**11.3 ¿Puedo usar mi WhatsApp personal?**
No. Solo trabajamos con la API oficial de WhatsApp Business (sección 5).

**11.4 ¿Qué pasa si una consulta es confusa o no se entiende?**
El agente no la responde: envía una alerta al equipo para que la atienda una persona.

**11.5 ¿Puedo empezar con un plan chico y después pasar a otro?**

Sí. Al pasar a un plan superior se paga solo la diferencia de setup entre ambos planes.

**11.6 ¿Trabajan fuera de La Plata?**

Sí, en forma remota en todo el país y presencial en el AMBA con viático (sección 1.3).

## 12. Control de versiones del documento

<table>
  <thead>
    <tr>
        <th>Versión</th>
        <th>Fecha</th>
        <th>Responsable</th>
        <th>Cambios</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1.0</td>
        <td>01/10/2026</td>
        <td>Socio responsable<br />comercial</td>
        <td>Versión inicial para la base de<br />conocimiento del agente</td>
    </tr>
  </tbody>
</table>