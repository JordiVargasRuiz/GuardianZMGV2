# GuardianZMG

`GuardianZMG` es un flujo de n8n diseñado como asistente de movilidad urbana para la Zona Metropolitana de Guadalajara (ZMG). El objetivo del proyecto es ofrecer respuestas automatizadas en Telegram que integren cálculo de rutas, condiciones climáticas, análisis de tráfico y disponibilidad de transporte público.

## Visión general

Este workflow transforma una consulta de usuario en Telegram en un reporte estructurado:

1. Detecta si el mensaje corresponde a una solicitud de ruta o a una charla general.
2. Si es una ruta, solicita datos en tiempo real de transporte público, clima y tráfico.
3. Procesa la información con un agente de inteligencia artificial y herramientas internas.
4. Devuelve un mensaje de Telegram con datos relevantes y un botón de acceso directo.

El diseño enfatiza claridad, fiabilidad y contenido accionable para usuarios que necesitan movilidad dentro de Guadalajara, Zapopan, Tlaquepaque y Tonalá.

## Qué hace cada módulo

### `Telegram Trigger`

- Recibe mensajes entrantes desde Telegram.
- Extrae `chat.id` y texto del usuario.
- Inicia el flujo en n8n cuando el usuario escribe al bot.

### `Basic LLM Chain`

- Clasifica la intención del mensaje como `RUTA` o `CHARLA`.
- Evita procesar solicitudes que no sean de movilidad.
- Mejora la precisión del bot al enrutar solo las consultas relevantes al agente de ruta.

### `AI Agent`

- Actúa como orquestador central del flujo.
- Envía consultas a herramientas externas y procesa resultados.
- Genera la respuesta final en formato de texto profesional.
- Preserva lógica de negocio como: cálculo de costo, evaluación de coincidencias y estructura de reporte.

### `HTTP Request` (Google Maps Directions)

- Consulta la API de Directions con `mode=transit`.
- Devuelve rutas detalladas de transporte público.
- Utiliza parámetros como `language=es`, `region=mx` y `transit_routing_preferences=less_walking`.

### `obtener_clima`

- Obtiene el clima actual de `Guadalajara,MX`.
- Añade contexto meteorológico al reporte final.
- Permite informar sobre condiciones que pueden afectar la movilidad.

### `MiBici` / `Info_MiBici`

- Consulta el estado de la flota y las estaciones de bicicletas públicas.
- Ofrece información de movilidad intermodal al usuario.
- Permite complementar rutas con opciones de bicicleta cuando aplique.

### `Analizador_vial`

- Procesa la ruta de Google Maps y el contenido de un RSS de tráfico local.
- Extrae nombres de calles relevantes de las instrucciones de ruta.
- Identifica coincidencias entre la ruta y los incidentes reportados.
- Clasifica el impacto como `ALTO`, `MODERADO` o `NULO`.
- Genera el campo `veredictoPersonalizado` usado en el reporte.

### `calculadora_tarifas`

- Calcula el costo total del viaje en base a `11.00 MXN` por unidad de transporte.
- Aplica la regla de negocio: si hay una unidad, el costo es exactamente `11.00 MXN`.
- Devuelve precio, moneda y detalle del cálculo.

### `Send a text message`

- Envía la respuesta al chat de Telegram.
- Configura `parse_mode=Markdown` y un teclado inline con URL de rutas.
- Entrega al usuario la salida final del workflow.

## Cómo usar el flujo

### 1. Importación

- Importa `GuardianZMG.json` en tu instancia de n8n.
- El archivo contiene la definición completa del workflow y sus conexiones.

### 2. Configuración de credenciales

Configura los siguientes servicios en n8n antes de ejecutar el flujo:

- Telegram API
- Google PaLM / Gemini
- OpenWeatherMap
- Google Maps Directions API

> Nota: en el archivo exportado hay claves y credenciales de ejemplo que deben ser reemplazadas por tus propias cuentas. Nunca publiques estas credenciales.

### 3. Validación de webhook

- Verifica que el webhook de Telegram esté registrado y activo.
- Asegúrate de que el bot tenga permiso para recibir mensajes y enviar respuestas.

### 4. Prueba funcional

Envía un mensaje de ejemplo en Telegram, por ejemplo:

- `De Miramar al CUCEI`
- `Ruta desde Plaza del Sol hacia Glorieta Minerva`

El flujo generará una respuesta con:

- estado de la ruta
- veredicto de impacto vial
- costo estimado
- pasos detallados de la ruta
- disponibilidad de MiBici
- condiciones climáticas

## Requerimientos técnicos

Este workflow requiere una versión de n8n que soporte los siguientes nodos y módulos:

- `telegramTrigger`
- `telegram`
- `@n8n/n8n-nodes-langchain.agent`
- `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`
- `@n8n/n8n-nodes-langchain.chainLlm`
- `@n8n/n8n-nodes-langchain.toolCode`
- `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- `openWeatherMapTool`
- `httpRequestTool`
- `rssFeedReadTool`

Si cualquiera de estos nodos falta, el flujo no podrá ejecutarse correctamente.

## Buenas prácticas y seguridad

- Sustituye API keys y credenciales en el flujo por credenciales seguras de n8n.
- No dejes claves en texto plano dentro del repositorio.
- Usa variables de entorno o credenciales oficiales de n8n para todos los servicios.
- Revisa los permisos del bot de Telegram y limita el acceso a servicios externos.

## Consideraciones de diseño

- El flujo está diseñado para entregar información útil y accionable en una sola respuesta.
- El análisis de tráfico no inventa situaciones; solo informa impactos detectados en coincidencias reales entre la ruta y el RSS.
- El cálculo de tarifas sigue una regla fija de `11 MXN` por unidad, lo cual es necesario para mantener consistencia operativa.
- El agente utiliza la información de herramientas externas como origen de verdad para la generación del reporte.

## Archivos del proyecto

- `GuardianZMG.json`: definición completa del workflow de n8n.
- `README.md`: documentación de uso y configuración.

---

### Actualización y mantenimiento

Para modificar el flujo, realiza los cambios en n8n y exporta el JSON actualizado. Comprueba siempre el comportamiento en Telegram después de ajustar credenciales o nodos.
