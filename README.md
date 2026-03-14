# 🏦 Insights — ACH Funding Assistant

Agente conversacional que guía a clientes de Insights para fondear su cuenta de inversión vía ACH. Construido con **Gemini 1.5 Flash** + **Gradio**.

---

## Requisitos cumplidos

| Requisito | Estado |
|---|---|
| (a) Pregunta banco + estado antes de dar datos | ✅ |
| (b) Infiere y muestra routing correcto (lookup interno) | ✅ |
| (c) Guía al cliente paso a paso | ✅ |
| (d) Simula y comunica fallos R01 y R03 | ✅ |
| (e) Mantiene historial conversacional entre turnos | ✅ |
| Bonus: usa API de Gemini | ✅ |

---

## Instalación

```bash
# 1. Clona el repositorio
git clone https://github.com/tu-usuario/ach-funding-agent.git
cd ach-funding-agent

# 2. Instala dependencias
pip install -r requirements.txt

# 3. Configura tu API key
cp .env.example .env
# Edita .env y agrega tu GEMINI_API_KEY

# 4. Corre el agente
python app.py
```

Abre tu browser en `http://localhost:7860`

---

## Obtener API key de Gemini

1. Ve a [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
2. Crea una API key gratuita
3. Pégala en tu archivo `.env`

---

## Flujo del agente

```
GREETING → banco? → estado? → ROUTING LOOKUP → account number
→ tipo de cuenta → monto → velocidad → CONFIRMACIÓN → ÉXITO
                                              ↓
                                       MANEJO DE FALLOS
                                    (R01, R02, R03, R04...)
```

---

## Estructura del proyecto

```
ach-funding-agent/
├── app.py            # Agente completo (Gemini + Gradio)
├── requirements.txt
├── .env.example
└── README.md
```
