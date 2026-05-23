# Reflexión Individual

## Nombre del Estudiante

**Samuel Rodríguez** — Estudiante de Ingeniería de Sistemas

---

## Rol en el equipo

Responsable del diseño e implementación del sistema de automatización en n8n, despliegue en Railway, integración con Brevo para el envío de correos transaccionales, y elaboración de la documentación arquitectónica: vistas C4, diagrama de secuencia, vista de despliegue y matriz de riesgos.

---

## Aprendizajes Clave

- Aprendí a estructurar un análisis arquitectónico real partiendo de un problema operativo concreto, identificando no solo lo que el sistema hace, sino lo que le falta para ser robusto en producción.
- Comprendí mejor la diferencia entre una solución funcional y una solución arquitectónicamente sólida: el sistema funciona y la demo corre perfectamente, pero la matriz de riesgos reveló que siete de diez riesgos son de nivel alto, algo que no era evidente mientras solo se miraba el flujo feliz.
- Practiqué la construcción de vistas arquitectónicas con el modelo C4, aplicando los niveles de Contexto y Contenedores sobre una arquitectura real desplegada, lo que me obligó a ser preciso en la distinción entre actores, contenedores, sistemas externos y relaciones de comunicación.
- Interioricé que la seguridad no es una capa que se agrega al final: el webhook público sin autenticación fue diseñado pensando en comodidad para la demo, y convertirlo en producción sin modificarlo sería un error de arquitectura con consecuencias directas.

---

## Retos Superados

- Tuve dificultades con la persistencia de datos en n8n, ya que el Static Data se comporta de forma distinta según el modo de ejecución y la versión del workflow. Lo resolví mediante pruebas controladas de reinicio del proceso y documentando el comportamiento real, lo que a su vez se convirtió en el riesgo R-01 de la matriz.
- Encontré que modelar la vista de Contenedores (C4 Nivel 2) requería tomar decisiones sobre qué es un "contenedor" en una plataforma de automatización como n8n, donde los workflows son la unidad de lógica pero comparten proceso, memoria y runtime. Resolví esto tratando cada workflow como un contenedor lógico independiente, con sus propios endpoints y responsabilidades definidas.
- Aprendí que documentar la arquitectura después de construir el sistema es más difícil que diseñarla antes: hay decisiones implícitas que tomé durante el desarrollo y que tuve que hacerlas explícitas al escribir las vistas, lo que en varios casos reveló inconsistencias que corregí directamente en el sistema.

---

## Áreas por Mejorar

- Me gustaría reforzar mi conocimiento en el diseño de arquitecturas orientadas a eventos y colas de mensajes, ya que el patrón actual de llamada HTTP síncrona entre Workflow 1 y Workflow 2 introduce un acoplamiento temporal que podría eliminarse con un broker como RabbitMQ o el propio sistema de eventos de n8n.
- Reconozco que debo mejorar mi criterio para separar el momento de construir del momento de documentar. En este proyecto ambas actividades fueron simultáneas, lo que funcionó, pero en un equipo más grande o en un proyecto más complejo generaría deuda documental desde el inicio.

---

## Contribución Personal

Mi mayor aporte al equipo fue dar otra perspectiva y contribuir con soluciones y diferentes puntos de vista.

---

_Esta reflexión individual hace parte de la entrega final del curso AREM - Arquitectura Empresarial - Universidad de La Sabana._
