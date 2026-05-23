# Resumen Ejecutivo — Sistema de Automatización de Evaluaciones
## Suporta S.A.S · Taller 8 · Ingeniería de Software

---

## 1. Contexto del negocio

**Suporta S.A.S** es una empresa de capacitaciones virtuales para clientes corporativos. Opera con aproximadamente **30 empresas clientes**, cada una con un promedio de **20 empleados**, quienes deben completar una evaluación posterior a cada capacitación y obtener el **100% de respuestas correctas** para aprobar.

---

## 2. Problema identificado

El proceso anterior era completamente manual y repetitivo:

- Se enviaba un único Google Forms a todos los empleados de todas las empresas sin distinción.
- Se exportaba manualmente el Excel de respuestas al final del día.
- A diario, el equipo de Suporta filtraba por empresa y cruzaba contra la lista de empleados para determinar quién había respondido y quién no.
- El proceso era desgastante: los empleados respondían de forma escalonada, obligando a revisiones constantes sin un mecanismo de notificación automática.

**Impacto:** Alto costo operativo en tiempo humano, alta probabilidad de error, y cero visibilidad en tiempo real para los clientes corporativos.

---

## 3. Solución implementada

Se diseñó e implementó un **sistema de automatización de evaluaciones en n8n**, desplegado en **Railway**, que reemplaza completamente el proceso manual. El sistema procesa cada respuesta en el momento en que se envía, evalúa el puntaje, actualiza el estado del empleado y distribuye informes automáticamente a las partes correspondientes.

---

## 4. Arquitectura del sistema

```
Empleado                  n8n (Railway)                      Destinatarios
   │                           │                                   │
   ├─ Llena formulario ──────► Workflow 1                          │
   │                      (Procesar Respuesta)                     │
   │                           │                                   │
   │                      ┌────▼─────────────────────────┐        │
   │                      │ 1. Lee empresa y nombre        │        │
   │                      │ 2. Evalúa 10 respuestas        │        │
   │                      │ 3. Calcula puntaje (0–100%)    │        │
   │                      │ 4. Actualiza Static Data       │        │
   │                      └────────────┬─────────────────-┘        │
   │                                   │                           │
   │                           Workflow 2                          │
   │                      (Generar Informe + Email)                │
   │                           │                                   │
   │                      ┌────▼──────────────────────────┐       │
   │                      │ Informe general (todas las     ├──────►│ Jefe Suporta
   │                      │ empresas)                      │       │ juangomeper@gmail.com
   │                      │                                │       │
   │                      │ Informe empresa (solo la       ├──────►│ Contacto empresa
   │                      │ empresa del respondiente)      │       │ (email específico)
   │                      └────────────────────────────────┘       │
```

### Almacenamiento de estado

Se utiliza **Static Data de n8n** (memoria persistente del workflow) para mantener el registro de todos los empleados entre ejecuciones. Cada empleado tiene los siguientes atributos:

| Campo | Descripción |
|---|---|
| `empresa` | Nombre de la empresa cliente |
| `nombre` | Nombre completo del empleado |
| `email` | Correo del empleado |
| `estado` | `pendiente` / `completado` / `reprobado` |
| `puntaje` | Porcentaje obtenido (0–100%) |
| `fecha_respuesta` | Timestamp del envío |

---

## 5. Descripción de workflows

### Workflow 1 — Procesar Respuesta Encuesta

**Cadena A — Reset de datos (inicio de evaluación o demo)**
- Endpoint: `GET /webhook/cargar-demo`
- Carga los 30 empleados en memoria con estado `pendiente`.
- Permite reiniciar el sistema para una nueva capacitación o presentación.

**Cadena B — Procesamiento de respuesta**
- Endpoint: `POST /form/encuesta-suporta` (formulario n8n)
- El formulario presenta: dropdown de empresa → dropdown de empleados de esa empresa → 10 preguntas de evaluación (opciones A/B/C/D).
- Lógica de evaluación:
  1. Extrae el nombre del empleado desde el dropdown de su empresa.
  2. Compara las 10 respuestas contra la clave de corrección hardcodeada.
  3. Calcula: `puntaje = (respuestas_correctas / 10) × 100`
  4. Si `puntaje = 100%` → estado `completado`; si no → estado `reprobado`.
  5. Actualiza el registro en Static Data.
  6. Invoca automáticamente Workflow 2.

### Workflow 2 — Generar Informe y Enviar Email

- Endpoint interno: `POST /webhook/generar-informe`
- Recibe del Workflow 1: lista completa de empleados + empleados de la empresa activa + correo del contacto de esa empresa.
- Genera dos informes en formato HTML:
  - **Informe general**: todos los empleados de todas las empresas, agrupados por empresa.
  - **Informe de empresa**: únicamente los empleados de la empresa del respondiente.
- Envía ambos correos vía **Brevo** (API transaccional):
  - Informe general → Se resive al correo del administración Suporta.
  - Informe empresa → contacto registrado de la empresa correspondiente.

### Workflow 3 — Envío Email SMTP

Workflow de prueba utilizado durante el desarrollo para validar la integración con Brevo. Se conserva como referencia técnica. **No está activo en producción.**

---

## 6. Stack tecnológico

| Herramienta | Rol en el sistema |
|---|---|
| **n8n** | Motor de automatización de flujos |
| **Railway** | Hosting del servidor n8n (producción) |
| **Brevo** | Envío de correos transaccionales (300 emails/día — plan gratuito) |
| **n8n Form Trigger** | Formulario web hosteado directamente por n8n |
| **Static Data n8n** | Persistencia del estado de empleados entre ejecuciones |

---

## 7. Flujo de demostración

```
1. GET /webhook/cargar-demo
   → 30 empleados cargados, todos en estado PENDIENTE

2. Abrir /form/encuesta-suporta
   → Empleado selecciona empresa → selecciona su nombre → responde 10 preguntas

3. Al enviar el formulario:
   → n8n evalúa y calcula el puntaje
   → Actualiza el estado del empleado (completado o reprobado)
   → Genera informe general + informe de empresa
   → Envía ambos correos vía Brevo en tiempo real

4. El administrador recibe el informe general.
5. Contacto de empresa recibe el informe solo con sus empleados
```

---

## 8. Estado actual

El sistema se encuentra **en producción y completamente funcional**. Todos los workflows están activos. El envío de correos opera a través de Brevo con la IP de Railway autorizada.

La demo está configurada con 30 empleados ficticios distribuidos en 6 empresas, lista para ser presentada sin necesidad de datos reales.
