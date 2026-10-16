# Reflexión del Proyecto - Agente de Reservación (Equipo AFAS)

## 1. Qué casos fallaron y por qué, y qué cambiaron para corregirlos

De los 10 casos que corrimos en Telegram, **9 pasaron el comportamiento esperado** (los 4 happy_path, los 2 de borde, la verificación de escritura y los 2 de fuera de alcance) y **1 falló: el caso de memoria (caso 7)**.

**Análisis del caso que falló (memoria):**
- Entrada: “¿Qué tal la reservación del martes pasado para 4 personas?”
- Lo esperado: que el agente usara la memoria de la conversación para recordar de qué reservación hablábamos y dar los detalles.
- Lo que pasó: el agente **no recordó** la reservación; pidió aclaración (“¿Te refieres al martes 13/10/2026?”) y solicitó el folio o el nombre para poder buscarla.
- Causa: la memoria del nodo Simple Memory está acotada a la sesión y se llavea por `chat.id`. En la prueba reiniciamos la conversación con `/reset`, por lo que el contexto anterior ya no existía; además, no había una reservación real del “martes pasado” creada en esa misma sesión, así que no había nada que recordar.
- Lo que hicimos para corregirlo: reforzamos en el system prompt la instrucción de que, cuando el cliente mencione una reservación sin identificador, el agente **siempre pida el folio o el nombre** antes de responder y **nunca inventa datos**. El agente ya se comportaba así de forma segura (no alucinó), pero dejamos explícito el flujo para que la respuesta sea consistente. Decidimos que la memoria de Simple Memory es para el hilo inmediato de la conversación, no para memoria de largo plazo.

**Conclusión:** el único fallo fue de memoria de largo plazo y quedó documentado; no fue un error de datos ni de seguridad, porque el agente prefirió preguntar en vez de inventar.

---

## 2. Una limitación que no pudimos resolver

La limitación que dejamos sin resolver es **la memoria que se pierde al reiniciar la conversación**.

El nodo Simple Memory (Buffer Window) guarda el contexto en memoria del proceso y se identifica por el `chat.id` de Telegram. Esto tiene dos consecuencias:
- Al reiniciar el workflow de n8n o la conversación, el contexto anterior se pierde.
- El agente no puede recordar reservaciones “de conversaciones pasadas” ni hacer memoria de largo plazo; por eso, cuando se refiere a una reservación, siempre tiene que pedir el folio o el nombre (como pasó en el caso 7).

Relacionadas con esta, encontramos otras dos limitaciones que el equipo decidió **no** resolver por ahora:
- **No hay tools de modificar ni cancelar:** el agente solo puede crear y consultar reservaciones. Cuando alguien pide un cambio o cancelación, el agente siempre canaliza al personal (`sandoval110906@gmail.com`).
- **La validación de capacidad la hace el modelo, no una tool:** el chequeo de que no pasen de 10 personas por reservación y de 15 en total lo razona el modelo a partir del prompt; no existe una herramienta que consulte la ocupación real del local.

Elegimos la memoria como limitación principal porque fue la que realmente observamos fallar en las pruebas (caso 7).

---

## 3. Qué cambiaríamos para producción con usuarios reales

Si este agente fuera a operar con clientes reales, haríamos estos cambios:

1. **Persistencia de memoria de largo plazo:** reemplazar Simple Memory por una base de datos de memoria (por ejemplo Postgres/Redis como store externo) para que el contexto y el historial por cliente sobrevivan a reinicios del workflow.
2. **Tools de modificar y cancelar reservaciones:** agregar operaciones sobre Google Calendar (update y delete) con validación, para no depender de que el personal haga los cambios a mano.
3. **Validación real de disponibilidad y aforo con una tool:** consultar en Calendar cuántas personas ya hay en el rango solicitado antes de crear la reservación, para respetar el máximo de 15 personas sin depender del juicio del modelo.
4. **Confirmaciones y recordatorios automáticos robustos:** enviar recordatorio (por ejemplo 24 h y 1 h antes) y guardar el folio en una hoja/base de datos para trazabilidad, además del correo actual.
5. **Observabilidad y manejo de errores:** registrar cada ejecución (folio, tool usada, resultado) y avisar al personal si falla el envío de correo o la creación del evento, en vez de solo depender de la respuesta del modelo.

---

## 4. Qué hizo cada integrante, en concreto

- **Alan Franco Moraila (Arquitectura y Calidad):** diseñó la arquitectura del agente y el stack por capa (modelo, memoria, datos, canal y acciones); definió el system prompt (rol, alcance, tono, formato, manejo de temas ajenos y guardrails); diseñó los 10 casos de prueba con su reparto (happy_path, borde, memoria, escritura y fuera de alcance); corrió y documentó los casos en Telegram, tomó las evidencias y redactó la reflexión y el README.

- **Angel Fernando Sandoval Rodriguez (Datos e integraciones):** configuró las credenciales e integraciones de n8n (Telegram bot, Google OAuth para Calendar/Sheets/Gmail y el modelo); conectó las tools al AI Agent (ObtenerMenu, Crear_reservacion, Consultar_reservaciones, Enviar_Correo y generar_folio); armó la hoja del menú y el calendario de pruebas; verificó el formato del folio y del título de los eventos, y validó en Google Calendar que las reservaciones de los casos quedaran registradas.