# 📄 Resumen Ejecutivo del Proyecto Arquitectónico

## Nombre del Cliente

**Suporta S.A.S** — Empresa de capacitaciones virtuales para clientes corporativos. Opera con aproximadamente 30 empresas clientes, cada una con un promedio de 20 empleados que deben aprobar evaluaciones con el 100% de respuestas correctas tras cada capacitación.

---

## Objetivo General del Proyecto

El proyecto buscó analizar y transformar el proceso de evaluación post-capacitación de Suporta S.A.S, el cual se ejecutaba de forma completamente manual: exportación de Excel, filtrado diario por empresa y cruce contra listas de empleados. Este flujo presentaba alto costo operativo en tiempo humano, tasa de error elevada y cero visibilidad en tiempo real para los clientes corporativos.

El reto arquitectónico central fue diseñar un sistema que procesara respuestas en el momento del envío, evaluara puntajes automáticamente, mantuviera el estado de cada empleado y distribuyera informes diferenciados a múltiples destinatarios, sin intervención humana. La solución propuesta y desplegada aporta visibilidad inmediata, trazabilidad por empresa y eliminación total del trabajo operativo repetitivo.

---

## Vistas Arquitectónicas Cubiertas

| Vista                   | Alcance de la Solución                                                                 |
|-------------------------|----------------------------------------------------------------------------------------|
| Procesos de Negocio     | Modelado del flujo completo: desde el envío del formulario hasta la distribución de informes por correo, con cadena de reset para inicio de nuevas capacitaciones. |
| Información / Datos     | Modelo de entidad empleado con atributos: empresa, nombre, email, estado, puntaje y fecha de respuesta. Almacenamiento en Static Data de n8n. |
| Aplicaciones / Sistemas | Vistas C4 Nivel 1 (Contexto) y Nivel 2 (Contenedores): sistema central n8n, Formulario, Workflow 1, Workflow 2, Static Data y Brevo como sistema externo. |
| Infraestructura         | Mapa de despliegue: contenedor Docker Node.js 20 en Railway, dominio HTTPS público, dependencia de Brevo para email transaccional. Identificación de punto único de falla. |
| Seguridad               | Identificación de webhooks públicos sin autenticación (R-03), ausencia de control de acceso al endpoint de reset, y dependencia de IP para autenticación con Brevo (R-09). |
| Cumplimiento Normativo  | Identificación de ausencia de log de auditoría (R-07) y datos de empleados sin persistencia garantizada (R-01), con impacto en trazabilidad y potencial cumplimiento de Ley 1581. |

---

## Hallazgos Clave

- El **Static Data de n8n** se usa como única capa de persistencia, lo que significa que un reinicio del proceso en Railway borra todos los registros de empleados y estados activos — riesgo crítico (R-01, nivel 9).
- El sistema presenta un **punto único de falla**: toda la lógica, el formulario y el envío de correos dependen de una sola instancia n8n sin redundancia ni fallback (R-02).
- El endpoint `GET /webhook/cargar-demo` está **expuesto públicamente sin ningún mecanismo de autenticación**, lo que permite a cualquier persona con la URL borrar todos los datos de producción (R-03).
- Las **respuestas correctas están hardcodeadas** directamente en el nodo JavaScript del Workflow 1, lo que implica que cambiar las preguntas de una capacitación requiere editar código del workflow — sin versioning ni auditabilidad (R-06).
- El modelo actual de 6 empresas × 5 empleados **no escala** a la operación real de Suporta (30 empresas × 20 empleados) sin un rediseño de los dropdowns estáticos y la estructura de datos (R-08).

---

## Recomendaciones Principales

- Migrar el almacenamiento de estado de empleados de **Static Data a una base de datos externa** (PostgreSQL en Railway o Supabase), garantizando persistencia ante reinicios y soporte para transacciones concurrentes.
- Proteger el endpoint `/webhook/cargar-demo` con un **header de autenticación `X-Auth-Token`** o moverlo a una ruta interna del panel de n8n, eliminando la exposición pública.
- **Externalizar la clave de respuestas correctas** a un nodo de configuración separado o a una hoja de cálculo conectada (Google Sheets), desacoplando el contenido pedagógico de la lógica de evaluación.
- Implementar un **nodo de auditoría** al final de cada evaluación que escriba el registro histórico en Google Sheets o Airtable antes de actualizar el estado en memoria.
- Revisar el **plan de Brevo** antes del lanzamiento en producción real, dado que 600 empleados generan hasta 1.200 correos por ciclo de evaluación, superando el límite gratuito de 300/día.

---

## Reflexión Final

Este ejercicio permitió al equipo aplicar de manera práctica los conceptos de arquitectura empresarial sobre un cliente real con un problema operativo concreto y medible. El proceso de análisis estructurado — desde el modelado del problema hasta la identificación de riesgos arquitectónicos y la construcción de vistas C4 — demostró que una solución funcional en producción no equivale necesariamente a una solución arquitectónicamente sólida. Los hallazgos más valiosos no fueron los que validaron la solución, sino los que revelaron sus limitaciones reales: persistencia frágil, ausencia de autenticación, y un modelo de datos que no soporta la escala del negocio real. Esta tensión entre "funciona para la demo" y "está listo para producción" es precisamente la que la arquitectura empresarial busca hacer visible.
