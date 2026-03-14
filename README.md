# 🏦 Insights — ACH Funding Assistant

Agente conversacional que guía a clientes de **Insights** para fondear su cuenta de inversión vía ACH. Construido con **Gemini 2.5 Flash** + **Gradio 6.x**.

---

## 📋 Tabla de contenidos

- [Descripción](#descripción)
- [Diseño del agente](#diseño-del-agente)
- [System Prompt](#system-prompt)
- [Implementación](#implementación)
- [Tablas de referencia](#tablas-de-referencia)
- [Demo](#demo)
- [Instalación](#instalación)
- [Análisis y síntesis](#análisis-y-síntesis)
- [Propuesta de escalabilidad (Bonus)](#propuesta-de-escalabilidad)

---

## Descripción

El agente automatiza completamente la atención a clientes que desean fondear su cuenta vía ACH. Es capaz de:

- Entender la intención del cliente
- Hacer las preguntas correctas en el orden correcto
- Inferir el routing number a partir del banco y estado del cliente
- Guiar el proceso paso a paso hasta la confirmación
- Validar información y manejar fallos con códigos Nacha (R01, R02, R03...)
- Escalar a un agente humano cuando es necesario

---

## Diseño del agente

**Propósito en una oración:**
> Guiar a un cliente de Insights paso a paso para fondear su cuenta de inversión vía ACH, desde la recopilación de datos bancarios hasta la confirmación del depósito, manejando errores y escalamientos sin intervención humana.

**Flujo de estados:**

```
INICIO
  │
  ▼
[1. RECOPILACIÓN]
  → ¿Qué banco?
  → ¿En qué estado?
  │
  ▼
[2. INFERENCIA / LOOKUP ROUTING]
  → Cruza banco + estado en tabla interna
  → Confirma routing con el cliente
  │
  ▼
[3. INSTRUCCIONES]
  → Solicita account number
  → Tipo de cuenta (checking/savings)
  → Monto a fondear
  → Velocidad (Standard vs Same-Day)
  │
  ▼
[4. CONFIRMACIÓN]
  → Resume todos los datos
  → Espera "yes" explícito
  │         │
  Sí        No → regresa a RECOPILACIÓN
  │
  ▼
[5. ÉXITO]
  → Confirma transferencia iniciada
  → Agrega disclaimer
  │
  ▼ (en cualquier punto si algo falla)
[MANEJO DE FALLOS]
  → Identifica código Nacha
  → Explica en lenguaje simple
  → Ofrece reintentar o escalar
```

---

## System Prompt

El system prompt completo está en [`app_gemini.py`](./app_gemini.py) dentro de la variable `SYSTEM_PROMPT`. Incluye:

| Sección | Propósito |
|---|---|
| Rol y tono | Define comportamiento base del LLM |
| Regla obligatoria | Fuerza banco + estado antes de cualquier dato |
| Flujo de preguntas | Script explícito para no saltarse pasos |
| Tabla de routing | Lookup interno sin APIs externas |
| Manejo de fallos | Cubre R01–R10 en lenguaje humano |
| Disclaimer | Protección legal y gestión de expectativas |

---

## Implementación

### Requisitos cumplidos

| Requisito | Estado |
|---|---|
| (a) Pregunta banco + estado antes de dar datos | ✅ |
| (b) Infiere y muestra routing correcto | ✅ |
| (c) Guía al cliente paso a paso | ✅ |
| (d) Simula y comunica fallos R01 y R03 | ✅ |
| (e) Mantiene historial conversacional entre turnos | ✅ |
| Bonus: usa API de Gemini 2.5 Flash | ✅ |

### Stack tecnológico

| Herramienta | Versión | Uso |
|---|---|---|
| Python | 3.13 | Lenguaje base |
| Gradio | 6.9 | Interfaz web de chat |
| google-generativeai | latest | Cliente API de Gemini |
| Gemini 2.5 Flash | — | Modelo LLM |
| python-dotenv | latest | Manejo seguro de API key |

---

## Tablas de referencia

Ver sección [4.1.3](#413-routing-numbers-aba-lógica-y-lookup-por-banco-y-estado) para la tabla de routing numbers y [4.1.4](#414-comparativa-ach-vs-wire-vs-debit-card) para la comparativa ACH vs Wire vs Debit Card.

---

## Demo

### Flujo exitoso completo

```
Usuario: I want to fund my Insights account
Agente:  Which bank holds the account you'd like to use?

Usuario: Bank of America
Agente:  In which U.S. state is that account registered?

Usuario: Texas
Agente:  Routing number: 111000025. Please share your account number.

Usuario: 987654321
Agente:  Account ending in ...4321 noted. Checking or savings?

Usuario: Checking
Agente:  How much would you like to deposit?

Usuario: 3000
Agente:  Standard ACH (free) or Same-Day ACH?

Usuario: Standard
Agente:  [Muestra resumen completo] Shall I proceed?

Usuario: Yes
Agente:  ✅ Transfer initiated! Funds available in 1–3 business days.
```

### Prueba de fallo R01
```
Usuario: My transfer was rejected with an R01 error
Agente:  R01 — Insufficient Funds. There weren't enough funds...
```

### Prueba de fallo R03
```
Usuario: My transfer failed with R03
Agente:  R03 — No Account / Unable to Locate. Please double-check...
```

### Escalamiento a humano
```
Usuario: I want to speak with a person
Agente:  Of course! Connecting you with a specialist. Reference: INS-XXXXX
```

---

## Instalación

```bash
# 1. Clona el repositorio
git clone https://github.com/tu-usuario/insights-ach-agent.git
cd insights-ach-agent

# 2. Instala dependencias
pip install -r requirements.txt

# 3. Configura tu API key de Gemini
cp .env.example .env
# Edita .env → GEMINI_API_KEY=tu_key_aqui

# 4. Corre el agente
python app_gemini.py

# 5. Abre en el browser
# http://127.0.0.1:7860
```

**Obtener API key gratuita:** [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)

### Estructura del proyecto

```
insights-ach-agent/
├── app_gemini.py      # Agente con Gemini 2.5 Flash + Gradio
├── app.py             # Versión alternativa sin API (lógica de estados)
├── requirements.txt   # Dependencias
├── .env.example       # Plantilla de variables de entorno
└── README.md
```

---

## Análisis y síntesis

### ¿Qué funcionó bien?
- El system prompt estructurado con secciones claras (flujo, tabla de routing, manejo de errores) le da al modelo suficiente contexto para mantener el hilo conversacional sin perderse.
- Hardcodear el saludo inicial eliminó el problema de cuota en el arranque.
- La tabla de routing interna hace al agente autosuficiente sin depender de APIs externas de lookup bancario.

### ¿Qué fue desafiante?
- La compatibilidad de Gradio 6.x rompió varios parámetros que existían en versiones anteriores (`type`, `bubble_full_width`, `avatar_images`). La solución fue usar `gr.ChatInterface` que es la API estable en esa versión.
- El formato del historial que Gradio pasa a la función de chat cambió entre versiones (de pares `[user, bot]` a lista de dicts `{"role", "content"}`), lo que causaba que Gemini perdiera el contexto y reiniciara el flujo. Se resolvió normalizando ambos formatos en la función `chat_fn`.
- Los modelos de Gemini disponibles en el plan gratuito cambian frecuentemente. Se encontró que `gemini-2.5-flash` es el más estable actualmente.

### Decisiones de diseño
- Se eligió **Gradio** sobre CLI porque permite una demo visual sin complejidad de deploy.
- Se eligió **Gemini** sobre otras alternativas por su capa gratuita y su soporte nativo de `system_instruction`, que permite separar limpiamente el prompt del código.
- La versión `app.py` (sin API) se mantuvo como fallback para demostrar que la lógica de negocio es sólida independientemente del LLM.

---

## Propuesta de escalabilidad (Bonus)

Para llevar este agente a producción de forma escalable:

### Herramientas propuestas

| Capa | Herramienta | Razón |
|---|---|---|
| Orquestación | **LangGraph** | Manejo explícito de estados del agente, fácil de auditar y extender |
| LLM | **Gemini 2.5 Flash** (API) | Bajo costo, alta velocidad, soporte de tool calling |
| Memoria | **Redis** | Persistencia de sesión entre turnos con TTL automático |
| Backend | **FastAPI** | API REST para integrar con WhatsApp, web o mobile |
| Canales | **Twilio** (WhatsApp/SMS) | Llegar al cliente donde ya está |
| Observabilidad | **LangSmith** | Trazabilidad de cada conversación para debugging y mejora continua |
| Deploy | **Cloud Run (GCP)** | Serverless, escala a cero, pago por uso |

### Flujo propuesto en producción

```
Cliente (WhatsApp / Web / App)
        │
        ▼
   Twilio / Webhook
        │
        ▼
   FastAPI Backend
        │
   ┌────┴────┐
   │ Redis   │  ← sesión del cliente (banco, estado, monto...)
   └────┬────┘
        │
   LangGraph Agent
        │
   ┌────┴────┐
   │ Gemini  │  ← LLM con system prompt
   └────┬────┘
        │
   LangSmith  ← logs y trazas
```

Este diseño permite: múltiples canales simultáneos, persistencia de sesión entre días, monitoreo de conversaciones fallidas y mejora continua del prompt basada en datos reales.
