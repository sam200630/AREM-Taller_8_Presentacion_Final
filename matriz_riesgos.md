# Matriz de Riesgos Arquitectónicos
## Sistema de Automatización de Evaluaciones — Suporta S.A.S

---

## Escala de evaluación

| Nivel | Probabilidad | Impacto |
|---|---|---|
| **1 — Bajo** | Poco probable que ocurra | Efecto menor, recuperable fácilmente |
| **2 — Medio** | Puede ocurrir en condiciones normales | Interrupción parcial del servicio |
| **3 — Alto** | Ocurrirá con certeza en producción | Pérdida de datos o fallo total del sistema |

**Nivel de riesgo = Probabilidad × Impacto**

| Rango | Clasificación |
|---|---|
| 1–2 | 🟢 Bajo |
| 3–4 | 🟡 Medio |
| 6–9 | 🔴 Alto |

---

## Matriz de riesgos

| # | Riesgo | Componente afectado | Probabilidad | Impacto | Nivel | Mitigación |
|---|---|---|:---:|:---:|:---:|---|
| R-01 | **Pérdida de estado por reinicio de proceso** — Si Railway reinicia el contenedor de n8n (deploy, crash, escasez de recursos), el Static Data se borra y se pierden todos los registros de empleados y sus estados actuales. | Static Data / n8n | 3 | 3 | 🔴 9 | Migrar el almacenamiento a una base de datos externa (PostgreSQL en Railway o Supabase). Static Data es adecuado solo para demo, no para producción. |
| R-02 | **Punto único de falla en Railway** — El sistema completo (formulario, webhooks, lógica, correos) depende de una sola instancia n8n en Railway. Si el servicio cae, no hay fallback. | Infraestructura | 2 | 3 | 🔴 6 | Configurar health checks y alertas en Railway. En producción, considerar réplica o un proveedor con SLA (Railway Pro, Render, Fly.io). |
| R-03 | **Webhooks públicos sin autenticación** — El endpoint `GET /webhook/cargar-demo` es accesible sin ningún token o credencial. Cualquier persona con la URL puede resetear todos los datos de producción. | Seguridad / Workflow 1 | 2 | 3 | 🔴 6 | Agregar header de autenticación (`X-Auth-Token`) o mover el reset a un endpoint autenticado dentro del panel de n8n. Nunca exponer este endpoint en producción. |
| R-04 | **Límite de Brevo (300 emails/día)** — En producción real con 600 empleados (30 empresas × 20 empleados), cada respuesta genera 2 correos. Pico de 1.200 correos/día supera el plan gratuito. | Brevo / Workflow 2 | 3 | 2 | 🔴 6 | Evaluar plan de pago de Brevo antes del go-live. Alternativa: agrupar correos por lotes en lugar de enviar uno por cada respuesta. |
| R-05 | **Race condition en Static Data bajo concurrencia** — Si dos empleados envían el formulario simultáneamente, ambas ejecuciones del workflow leen el estado antes de que cualquiera escriba, pudiendo sobrescribirse mutuamente. | Static Data / Workflow 1 | 2 | 2 | 🟡 4 | n8n ejecuta workflows de forma secuencial por defecto en el plan Community. Verificar configuración de concurrencia. En producción, usar una DB con transacciones atómicas. |
| R-06 | **Respuestas correctas hardcodeadas en el workflow** — Cambiar las preguntas o respuestas de una capacitación requiere editar el código JavaScript interno del nodo principal. Esto es propenso a errores humanos y no auditable. | Workflow 1 / Mantenibilidad | 3 | 2 | 🔴 6 | Externalizar la clave de respuestas a un nodo de configuración separado o a un campo en una hoja de cálculo / DB. Facilita el cambio sin tocar lógica. |
| R-07 | **Sin log de auditoría** — No existe registro histórico de quién respondió qué en evaluaciones anteriores. Si se reinicia la demo, toda la evidencia se pierde. | Trazabilidad | 3 | 2 | 🔴 6 | Agregar un nodo de escritura a Google Sheets, Airtable o una DB al final de cada evaluación, antes de actualizar el estado en memoria. |
| R-08 | **Escalabilidad del modelo de datos** — La estructura actual está diseñada para 6 empresas con 5 empleados cada una. Agregar las 30 empresas reales (~600 empleados) requiere rediseño manual del formulario y del Static Data. | Arquitectura de datos | 3 | 2 | 🔴 6 | Reemplazar los dropdowns estáticos por carga dinámica desde una fuente de datos (DB o Google Sheets). Desacoplar el catálogo de empleados del workflow. |
| R-09 | **Dependencia de IP de Railway para Brevo** — Brevo tiene autorizada la IP del servidor Railway. Si Railway cambia la IP (reasignación, cambio de región), los correos fallan sin advertencia. | Brevo / Infraestructura | 1 | 2 | 🟡 2 | Usar autenticación por API Key de Brevo (que no depende de IP) en lugar de whitelist por IP. Verificar la configuración actual del conector SMTP. |
| R-10 | **Sin manejo de errores en el flujo de email** — Si Brevo devuelve un error (límite excedido, dirección inválida), el Workflow 2 falla silenciosamente. El usuario que respondió el formulario no recibe aviso y la jefa tampoco. | Workflow 2 / Resiliencia | 2 | 2 | 🟡 4 | Agregar nodo de manejo de error (`Error Trigger`) en el Workflow 2 que notifique al administrador ante fallos de envío. |

---

## Resumen por nivel de riesgo

| Nivel | Cantidad | Riesgos |
|---|---|---|
| 🔴 Alto | 7 | R-01, R-02, R-03, R-04, R-06, R-07, R-08 |
| 🟡 Medio | 2 | R-05, R-10 |
| 🟢 Bajo | 1 | R-09 |

---

## Hoja de ruta de mitigación (prioridad para producción real)

```
CRÍTICO — Antes de lanzar en producción
├── R-01: Migrar Static Data → PostgreSQL
├── R-03: Autenticar endpoint /webhook/cargar-demo
├── R-06: Externalizar clave de respuestas
└── R-07: Agregar log de auditoría persistente

IMPORTANTE — Primer mes en producción
├── R-04: Revisar plan de Brevo
├── R-08: Cargar empleados dinámicamente desde DB
└── R-02: Configurar health checks y alertas

DEUDA TÉCNICA — Backlog
├── R-05: Validar configuración de concurrencia
├── R-09: Cambiar auth Brevo a API Key
└── R-10: Implementar Error Trigger en Workflow 2
```

---

*Documento elaborado para Taller 8 — Ingeniería de Sistemas*
