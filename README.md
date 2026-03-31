# Asistente Personal IA con n8n, WhatsApp, Gmail y Google Calendar

## Descripción General

Este proyecto implementa un asistente virtual inteligente para Dayanna Rocca, emprendedora y fundadora de una agencia de IA. El sistema permite interactuar por WhatsApp con un agente conversacional que, usando el modelo Gemini de Google, interpreta las instrucciones del usuario y ejecuta acciones en tres plataformas clave:

- **Gmail:** enviar correos, buscar mensajes y aplicar etiquetas.
- **Google Calendar:** crear, buscar, modificar y cancelar eventos.
- **Google Sheets:** consultar una base de datos de contactos (nombre, email, teléfono).

Todo el orquestador está construido con **n8n** (workflow automation), aprovechando sus nodos nativos y la integración con LangChain para agentes basados en LLM.

**Características principales:**

- Interacción por **WhatsApp** (texto y audio transcrito).
- Agente principal con memoria conversacional.
- Delegación inteligente a sub‑agentes (Gmail y Calendar) usando la herramienta `toolWorkflow`.
- Consulta automática de contactos en Google Sheets antes de enviar correos o agendar reuniones.
- Soporte para transcripción de notas de voz (audio) mediante Gemini.
- Registro de conversación (memoria) para mantener contexto entre mensajes.

---

## Problemática

En el día a día de un emprendedor, las tareas administrativas fragmentadas consumen tiempo valioso. Por ejemplo:

- Recibir un mensaje por WhatsApp solicitando “agenda una reunión con Juan para mañana a las 10 a.m.” obliga a abrir Calendar manualmente.
- Atender un mensaje “envía el informe a María” requiere buscar el contacto, redactar, adjuntar archivos y enviar desde Gmail.
- Coordinar agendas con invitados externos implica buscar sus correos en la base de contactos y luego crear el evento.

Este flujo manual es repetitivo, propenso a errores y no deja trazabilidad. La persona depende de recordar dónde está cada dato y de ejecutar pasos en diferentes aplicaciones.

---

## Proceso actual (AS-IS)

El diagrama AS-IS muestra el flujo manual típico:

1. El usuario escribe un mensaje en WhatsApp.
2. La persona interpreta la solicitud.
3. Abre la aplicación correspondiente (Gmail, Calendar, Sheets) y busca la información necesaria.
4. Copia y pega datos, redacta, adjunta, confirma.
5. Responde por WhatsApp confirmando la acción.

**Problemas identificados:**
- Tiempo elevado por solicitud (5‑10 minutos en promedio).
- Errores comunes: adjuntar archivo equivocado, usar correo erróneo, olvidar confirmar.
- Sin registro centralizado de lo que se hizo.
- Proceso no escalable; la persona se convierte en el cuello de botella.

---

## Proceso automatizado (TO-BE)

Con el asistente IA, el flujo se transforma:

1. El usuario envía un mensaje de WhatsApp (texto o audio).
2. El **WhatsApp Trigger** recibe el mensaje y lo envía al agente principal.
3. El agente (con LLM Gemini) interpreta la intención y decide qué herramienta usar:
   - Si es un correo invoca al **sub‑agente Gmail**.
   - Si es un evento invoca al **sub‑agente Calendar**.
   - Si necesita un contacto consulta la **Google Sheets**.
4. Las herramientas ejecutan las operaciones en las APIs reales.
5. El agente genera una respuesta en lenguaje natural y la envía de vuelta por WhatsApp.

**Beneficios obtenidos:**
- Respuesta inmediata (segundos en lugar de minutos).
- Cero errores de personalización.
- Trazabilidad completa: cada acción queda registrada en los logs de n8n y en las propias aplicaciones.
- Proceso independiente de una persona; escalable a cualquier volumen.

---

## Comparación de mejoras en los procesos

![Comparación flujos AS-IS vs TO-BE](docs/imagenes/flujo_comparativo.png)

La imagen integra ambos flujos mostrando la reducción de pasos manuales y la eliminación del trabajo repetitivo. El asistente centraliza todas las decisiones y la ejecución, liberando al usuario para tareas de mayor valor.

---

## Agentes del Sistema

El asistente funciona mediante la colaboración de tres agentes especializados. Cada uno es un workflow de n8n con un propósito claro, y se comunican entre sí usando los nodos `toolWorkflow`.

### Asistente Personal WSP (Agente Principal)
![Asistente Personal WSP](docs/imagenes/asistente_principal.png)

Es el cerebro de la automatización. Recibe los mensajes de WhatsApp (texto o audio transcrito), mantiene la memoria de la conversación y decide qué hacer. Si necesita enviar un correo o gestionar una reunión, invoca a los sub‑agentes correspondientes. También consulta la base de contactos en Google Sheets cuando se requiere un email o teléfono. Su respuesta final se envía de vuelta al usuario por WhatsApp.

### Agente Gmail
![Agente Gmail](docs/imagenes/agente_gmail.png)

Este workflow se encarga de todas las tareas relacionadas con el correo electrónico. Puede **enviar mensajes** personalizados, **buscar correos** con filtros avanzados y **aplicar etiquetas** para organizar la bandeja de entrada. Recibe instrucciones en lenguaje natural y las ejecuta usando las herramientas nativas de Gmail.

### Agente Calendario
![Agente Calendario](docs/imagenes/agente_calendario.png)

Gestiona la agenda de Google Calendar. Crea eventos con la duración adecuada, busca reuniones existentes, cancela o modifica citas. Es capaz de interpretar fechas y horarios en lenguaje natural y traducirlos a los parámetros correctos de la API.

Todos los agentes están construidos con nodos **LangChain Agent** y usan **Google Gemini** como modelo de lenguaje, lo que les permite entender instrucciones en español y actuar de forma autónoma.



## Diagrama de Secuencia (Interacción entre componentes)

![Diagrama de secuencia](docs/imagenes/secuencia.png)

El diagrama muestra el orden de llamadas cuando un usuario pide “enviar correo a María con el informe”.  
1. Usuario → WhatsApp → n8n.  
2. Agente principal interpreta, consulta contactos en Sheets, obtiene el email.  
3. Llama al sub‑agente Gmail (vía `toolWorkflow`) con las instrucciones.  
4. El sub‑agente utiliza el nodo `gmailTool` para enviar el correo.  
5. Respuesta de confirmación retorna al agente principal y se envía al usuario.

---


## Arquitectura del Sistema

### 1. Arquitectura en Capas

![Arquitectura en capas](docs/imagenes/arquitectura.png)

- **Capa de Interacción:** WhatsApp (entrada/salida).
- **Capa de Orquestación:** n8n workflows (agente principal, sub‑agentes, memoria).
- **Capa de Inteligencia:** Gemini LLM + LangChain agent.
- **Capa de Herramientas:** Conectores nativos (Gmail, Calendar, Sheets) y `toolWorkflow` para invocar sub‑workflows.
- **Capa de Datos:** Google Sheets (contactos), Gmail/Calendar (datos reales).

### 2. Diagrama de Componentes

![Diagrama de componentes](docs/imagenes/diagrama_componentes.png)

Los componentes principales son:

- **Asistente Personal WSP (workflow principal):** recibe el mensaje, maneja memoria, decide qué herramienta usar y responde.
- **Agente Gmail (sub‑workflow):** encapsula todas las operaciones de correo (enviar, buscar, etiquetar).
- **Agente Calendario (sub‑workflow):** encapsula operaciones de agenda (crear, buscar, cancelar, modificar).
- **Google Sheets Tool:** consulta la base de contactos por nombre.
- **Google Gemini Chat Model:** provee la inteligencia al agente.
- **Memory Buffer:** mantiene el contexto de la conversación.

---

## Componentes – Estructura del Proyecto

| Archivo / Carpeta | Responsabilidad |
| :--- | :--- |
| `workflows/Agente Calendario.json` | Workflow de n8n para gestionar eventos en Google Calendar. |
| `workflows/Agente Gmail.json` | Workflow de n8n para gestionar correos en Gmail. |
| `workflows/Asistente Personal WSP.json` | Workflow principal que integra WhatsApp, LLM, memoria y sub‑agentes. |
| `docs/imagenes/` | Diagramas de arquitectura, flujos y capturas. |
| `docs/configuracion.md` | Guía paso a paso para configurar credenciales y conectar las cuentas. |

---

## Configuración y Uso

1. **Importar los workflows** en una instancia de n8n (versión 1.0 o superior).
2. **Configurar credenciales**:
   - Google Calendar OAuth2 (para el calendario de Dayanna).
   - Gmail OAuth2.
   - Google Sheets OAuth2 (para la hoja de contactos).
   - WhatsApp (token de Meta, número de teléfono).
   - Google Gemini (API key).
3. **Ajustar IDs de workflows** en los nodos `toolWorkflow` para que apunten a los IDs correctos después de importar.
4. **Verificar la hoja de contactos**: debe estar pública o compartida con la cuenta que usa Google Sheets.
5. **Activar los workflows** y probar con mensajes de WhatsApp.

---

## Registro y Auditoría

Aunque los workflows actuales no incluyen un log externo, n8n guarda todas las ejecuciones en su base de datos. Puedes consultar el historial de ejecuciones de cada workflow para auditar qué acciones se realizaron, con qué datos y en qué momento. Además, cada acción en Gmail, Calendar y Sheets queda registrada en sus respectivas aplicaciones.

---

## Beneficios Finales

- **Reducción de tiempo:** de 5‑10 minutos a menos de 30 segundos por tarea.
- **Disponibilidad 24/7:** el asistente responde incluso fuera de horario laboral.
- **Escalabilidad:** se pueden agregar más sub‑agentes (Slack, Notion, etc.) sin modificar la lógica principal.
- **Centralización:** todas las automatizaciones en un solo lugar (n8n).
- **Mejora continua:** los logs permiten identificar patrones y optimizar los prompts.
