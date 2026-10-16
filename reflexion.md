# Reflexión del Proyecto - Agente de Reservación

## 1. Qué casos fallaron y por qué, y qué cambiaron para corregirlos

**Preguntas guía:**
- ¿Qué casos presentaron errores durante la ejecución?
- ¿Cuál fue la causa raíz de cada fallo (falta de herramienta, error de formato, límite de capacidad, etc.)?
- Qué ajustes se realizaron en el prompt, las herramientas o el flujo para corregir el problema?
- qué casos pasaron de fallar a pasar después de la corrección?

**(Espacio para completar por el usuario)**

---

## 2. Una limitación que no pudieron resolver

**Candidatas reales del workflow para considerar:**
- Sin tools de modificar/cancelar: El agente no puede modificar ni cancelar reservaciones existentes, solo crear nuevas y consultar las existentes.
- Memoria que se pierde al reiniciar: El contexto de la conversación no se persiste entre ejecuciones diferentes del workflow.
- La suma de capacidad la hace el modelo y no una tool: La validación de si caben N personas en el local (máximo 15) se realiza en el prompt del modelo, no mediante una herramienta de cálculo explícita.

**Espacio para elegir y completar:**
_(Selecciona la limitación que consideres más relevante o añade la que observaste)_

---

## 3. Qué cambiarían para producción con usuarios reales

** ideas sugeridas a partir del workflow:**

1. Implementar una herramienta de modificación/cancelación de reservaciones para permitir cambios en Google Calendar.
2. Añadir persistencia de memoria (Memory Database) para que el contexto se preserve entre sesiones diferentes.
3. Validar la capacidad máxima mediante una herramienta/expresión explícita antes de crear reservaciones.
4. Agregar validación de grupos grandes (>10 personas) con redirección automática a contacto directo.
5. Implementar manejo de horarios festivos y cierres inesperados más robustos.
6. Agregar sistema de recordatorios automáticos (notificaciones 15 min antes de la reservación).

_(Selecciona o edita las ideas que consideres pertinentes)_

---

## 4. Qué hizo cada integrante, en concreto

_(Una línea por persona, con los datos del equipo)_

- Alan Franco Moraila: Arquitectura y Calidad - Definió la estructura del sistema, el stack por capa, las credenciales necesarias y revisó las limitaciones y calidad del sistema.
- Angel Fernando Sandoval Rodriguez: Datos e integraciones - Configuró las tools de Google (Sheets, Calendar, Gmail), el sistema de memoria, las integraciones de API y diseñó los casos de prueba.