
# 🛡️ Guardrails para Chatbot de Formación Profesional

Este documento describe cómo implementar **guardrails** para evitar que el chatbot responda a preguntas que no estén relacionadas con la Formación Profesional (FP).

---

## ✅ 1. Guardrails con Amazon Bedrock

### 📌 ¿Qué son los Guardrails?
Son restricciones y políticas de comportamiento que puedes configurar para controlar **qué tipo de respuestas puede o no generar el modelo** (filtrado temático, contenido ofensivo, temas prohibidos, etc.).

### 🛠️ ¿Cómo configurarlo?
Bedrock ofrece un panel gráfico para definirlos (aún en versión limitada), pero también puedes hacerlo de forma **manual usando lógica de aplicación (middleware)** o **a través del prompt** (Prompt Engineering) si aún no usas los guardrails nativos.

---

## ✅ 2. Estrategia recomendada por capas

### 🧱 Capa 1: Filtrado de pregunta
Antes de enviar la pregunta al modelo:

- Usa una **lista de palabras clave** o un modelo pequeño de clasificación para detectar si la consulta está relacionada con Formación Profesional.
- Si no lo está, responde con algo como:

> “Lo siento, este asistente solo responde dudas relacionadas con la Formación Profesional en la Comunitat Valenciana.”

Puedes usar `Amazon Comprehend`, `Amazon SageMaker`, o incluso una verificación con Bedrock:

```
¿Esta pregunta está relacionada con la Formación Profesional? Responde solo con SÍ o NO: "<pregunta del usuario>"
```

---

### 🧱 Capa 2: Restricción dentro del prompt

Diseña un prompt base que acompañe a cada petición:

```
Eres un asistente oficial de la Conselleria especializado exclusivamente en Formación Profesional. Solo puedes responder preguntas que estén directamente relacionadas con la normativa, acceso, ciclos formativos, becas, calendario académico o cualquier aspecto de la Formación Profesional en la Comunitat Valenciana. 

Si la pregunta no está relacionada con estos temas, responde educadamente que no puedes contestar y sugiere contactar con el canal oficial correspondiente.

Pregunta del usuario: {input}
```

---

### 🧱 Capa 3: Fallback controlado (Output)

Después de obtener una respuesta, evalúa si está desviada y muestra un mensaje estándar:

> “Lo siento, no puedo ayudarte con esa consulta. Este asistente está enfocado únicamente en Formación Profesional.”

---

## 🔒 Ejemplo completo de flujo defensivo

1. Usuario envía: `"¿Qué opinas del conflicto en Gaza?"`
2. Sistema verifica (Capa 1) ➜ tema no relacionado
3. Prompt no se envía al modelo o se fuerza una respuesta estándar.
4. Respuesta: `"Lo siento, este asistente solo responde preguntas sobre Formación Profesional en la Comunitat Valenciana."`

---

## 🔧 Implementación sugerida

- Middleware en Flask que filtre preguntas automáticamente.
- Prompt base para Bedrock Agent.
- Mini modelo de clasificación temática (FP / no FP).
