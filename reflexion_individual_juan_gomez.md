# Reflexión Individual

## Nombre del Estudiante

**Juan Andrés Gómez Pérez** — Estudiante de Ingeniería de Sistemas

---

## Rol en el equipo

Desarrollador principal de los flujos de automatización en n8n: diseñé e implementé el formulario inteligente de evaluación, la lógica de calificación automática, la persistencia de estado con Static Data, y la generación y envío de informes HTML por correo a través de Brevo. Adicionalmente participé en la planeación general del proyecto, contribuyendo a definir el alcance de la solución y la arquitectura de sus componentes.

---

## Aprendizajes Clave

- Aprendí a construir flujos de automatización no triviales en n8n, pasando de simples integraciones de dos nodos a un sistema con lógica condicional, persistencia entre ejecuciones, generación de contenido dinámico y comunicación entre workflows mediante webhooks internos.
- Comprendí mejor cómo la integridad de datos puede resolverse desde el diseño del formulario: usar seis dropdowns de nombre condicionados por la empresa seleccionada eliminó por completo la posibilidad de que un empleado se registrara bajo una empresa incorrecta, sin necesidad de validación posterior.
- Practiqué la diferencia entre un sistema que "funciona en demo" y uno que está listo para producción. Construir el endpoint `/webhook/cargar-demo` me hizo pensar en cómo un sistema real necesita mecanismos de inicialización, reinicio y control de estado, no solo el flujo feliz.
- Interioricé que el Static Data de n8n es una solución pragmática pero frágil: funciona perfectamente en un contexto controlado, pero depende del ciclo de vida del proceso y del servidor, lo que lo hace inadecuado como capa de persistencia en un entorno de alta disponibilidad.

---

## Retos Superados

- Tuve dificultades con la lectura dinámica del nombre del empleado desde el formulario: como el dropdown correcto depende de cuál empresa seleccionó el usuario, el nodo de procesamiento debía resolver en tiempo de ejecución cuál de los seis campos leer. Lo resolví construyendo lógica de mapeo empresa→campo dentro del nodo principal, lo que también simplificó el resto del procesamiento al normalizar los datos antes de cualquier comparación.
- Encontré que generar dos informes HTML con estilos en línea, agrupamiento por empresa y datos dinámicos directamente dentro de un nodo de n8n requería un nivel de precisión en la construcción de strings que no es evidente al principio. Cualquier comilla mal cerrada o referencia a un campo inexistente silenciaba el nodo sin un error claro. Lo superé construyendo y probando cada bloque del HTML de forma incremental antes de integrarlos.
- Aprendí que coordinar dos workflows mediante una llamada HTTP interna (Workflow 1 llama a Workflow 2 vía POST) introduce un acoplamiento que no es obvio hasta que uno de los dos falla: si el Workflow 2 no está activo, el Workflow 1 completa su lógica de calificación sin enviar ningún correo, y el error queda silencioso para el usuario final. Resolví esto asegurando que ambos workflows permanecieran activos antes de cualquier ejecución de demo.

---

## Áreas por Mejorar

- Me gustaría reforzar mi conocimiento en patrones de mensajería asíncrona. La llamada síncrona entre Workflow 1 y Workflow 2 fue una decisión práctica, pero aprender a desacoplarlos mediante un broker de eventos o una cola haría el sistema más resiliente y escalable.
- Reconozco que debo mejorar mi criterio para anticipar los riesgos operativos de las decisiones técnicas que tomo. Algunas elecciones que hice pensando en simplicidad para la demo —como el webhook público sin autenticación o el almacenamiento en Static Data— solo revelaron su peso cuando el equipo realizó el análisis de riesgos formal. Incorporar esa mirada crítica desde el diseño, y no después, es algo que quiero desarrollar.

---

## Contribución Personal

Mi mayor aporte al equipo fue convertir el problema real de Suporta en un sistema funcional de extremo a extremo: desde el formulario que garantiza la integridad de los datos hasta el correo diferenciado que llega al jefe general y al contacto de cada empresa. Construir eso en n8n implicó tomar muchas decisiones de diseño sin una guía explícita, y creo que el resultado —un sistema en producción que funciona en la demo sin intervención manual— habla de esas decisiones. También aporté en la planeación general del proyecto, participando en la definición del alcance y en las discusiones sobre qué debía resolver la automatización antes de empezar a construirla.

---

_Esta reflexión individual hace parte de la entrega final del curso AREM - Arquitectura Empresarial - Universidad de La Sabana._
