}# Configuración de perfiles de cliente para ChatIA Space

Este documento define la estructura de los archivos JSON que debe tener cada cliente del sistema **ChatIA Space**. Un asistente de IA (ChatGPT, Claude, Groq, etc.) debe usar esta especificación para generar automáticamente los JSON a partir de una descripción del negocio.

## Separación entre configuración estática y datos dinámicos

- **Los archivos JSON** de esta carpeta definen la **personalidad, reglas de negocio y comportamiento** del asistente. No cambian frecuentemente.
- La información dinámica (precios, servicios concretos, productos, promociones, inventario, horarios especiales) se almacena en **Google Sheets (Excel)** y se sincroniza automáticamente. El asistente la consulta en tiempo real a través de `database.json`.
- Para actualizar precios o servicios, el cliente (o administrador) solo edita la hoja de Excel; **no necesita tocar los JSON**.

---

## Archivos requeridos (carpeta `clients/{client_id}`)

Todos los archivos deben estar en formato UTF-8 y seguir la estructura exacta que se muestra. Los campos opcionales pueden omitirse o dejarse vacíos.

### 1. `perfil.json`

Identidad del negocio y del asistente virtual.

```json
{
  "profile": {
    "business_type": "tipo de negocio (ej. educación, salud, restaurante, SaaS)",
    "agent_name": "nombre del asistente (ej. Lucy, Ana, Asistente Virtual)",
    "position": "cargo o rol (ej. Asesora Comercial, Recepcionista)",
    "company_brand": "nombre de la empresa o marca",
    "founder": "nombre del fundador (opcional)",
    "location": "ciudad, país (opcional)",
    "essence": "frase que describe la personalidad del asistente (máx 200 caracteres)"
  }
}
2. comunicacion.json
Tono, palabras permitidas/prohibidas y formato de las respuestas.

json
{
  "communication": {
    "tone": "adjetivo (profesional, cercano, divertido, formal, empático)",
    "humanity_fillers": ["frase1", "frase2", "frase3"],
    "forbidden_words": ["palabra1", "palabra2", "palabra3"],
    "preferred_words": ["palabra1", "palabra2"],
    "emojis": "poco | moderado | mucho",
    "line_breaks": true
  }
}
3. sabiduria_negocio.json
Casos de uso y soluciones típicas que ofrece el negocio (sin precios ni datos concretos).

json
{
  "use_cases": [
    { "sector": "nombre del segmento", "value": "beneficio o solución" }
  ],
  "repetitive_tasks_solutions": {
    "nombre_tarea": "descripción de la solución"
  }
}
4. tecnico_y_datos.json
Configuración de interfaz, enrutamiento de datos, reglas de calendario y políticas de registro.

json
{
  "technical": {
    "ui": {
      "welcome_buttons": ["opción1", "opción2", "opción3"],
      "pricing_buttons": ["plan1", "plan2"],
      "confirmation_buttons": ["Confirmar ✅", "Cambiar ✏️"],
      "confirmation_phrases": ["¿te parece bien?", "¿confirmamos?"],
      "whatsapp_buttons": {
        "yes_text": "Confirmar ahora ✅",
        "no_text": "Cancelar ✂️",
        "change_text": "Hacer un cambio ✏️"
      }
    },
    "data_routing": {
      "Billetera": "nuevo",
      "Seguimiento": "actualizar"
    },
    "calendar_rules": {
      "must_check_availability": true,
      "hours": "texto legible con los horarios de atención",
      "natural_dates_example": "Usa 'Este miércoles a las 4 PM'"
    },
    "log_policy": "Registrar solo datos nuevos o interés de compra. No para charla trivial."
  }
}
5. ventas_y_flujos.json
Flujo de ventas, reglas de precios, seguimiento y simulación.

json
{
  "sales_protocol": {
    "flow_order": ["paso1", "paso2", "paso3", "paso4", "paso5"],
    "price_rule": "regla sobre cuándo y cómo dar precios",
    "sales_instinct": "comportamiento ante interés del cliente"
  },
  "follow_up_logic": {
    "abandonment_minutes": 30,
    "max_attempts": 3
  },
  "simulation_mode": {}
}
6. personalizacion.json (opcional)
Reglas muy específicas o mensajes personalizados.

json
{
  "custom": {
    "extra_instructions": "texto libre con instrucciones adicionales",
    "welcome_message": "mensaje de bienvenida personalizado"
  }
}
Información mínima necesaria para generar los JSON
Para crear los archivos de un cliente, la IA debe recibir al menos lo siguiente (en cualquier formato, en lenguaje natural):

Nombre del negocio y rubro (educación, salud, restaurante, taller, etc.).

Nombre del asistente virtual (ej. "Lucy", "Ana", "Sofía") y personalidad (ej. "amable y organizada", "profesional pero cercana").

Horarios de atención para citas o reservas (días y rangos horarios). Si no aplican citas, indicarlo.

Qué servicios o productos ofrece el negocio (al menos 2 o 3 ejemplos). Estos son solo ejemplos descriptivos, no datos dinámicos. Los precios y listas concretas irán al Excel.

Reglas de venta (ej. "no dar precios sin que pregunten", "siempre preguntar el nombre antes de agendar").

Frases típicas que le gustaría usar (opcional, pero ayuda a definir el tono).

Ejemplo de entrada para la IA
"Genera los JSON para una clínica dental llamada 'Sonrisa Perfecta'. La asistente se llama 'Dra. Laura', es empática y profesional. Atienden lunes a viernes de 8am a 6pm, sábados de 9am a 1pm. Los servicios: limpieza, blanqueamiento, ortodoncia. Las citas duran 45 minutos. Regla de venta: dar precios solo si el cliente pregunta. Frases típicas: 'Tu salud dental es importante para nosotros'."

Instrucción final para la IA generadora
Con base en la información proporcionada por el usuario, genera los archivos JSON completos: perfil.json, comunicacion.json, sabiduria_negocio.json, tecnico_y_datos.json, ventas_y_flujos.json. Si falta algún dato, usa valores por defecto razonables. No incluyas precios ni servicios concretos en los JSON; esos irán al Excel. Devuelve solo los JSON, cada uno en un bloque de código separado, con el nombre del archivo como título.