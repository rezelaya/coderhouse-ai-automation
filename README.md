# Proyecto Integrador — IA Automation Avanzado (CoderHouse)

**Alumno:** Rodrigo Zelaya

Agente autónomo construido en n8n que actúa como asistente personal de registro y categorización de gastos, vía Telegram. Es la primera versión (Checkpoint 1 / Módulo 1) de un proyecto que se va a ir ampliando módulo a módulo durante el curso (memoria, integraciones, RAG, voz, etc.).

## Qué hace

1. Le mandás un mensaje al bot de Telegram describiendo un gasto (ej: *"gasté 8500 en el supermercado con débito"*).
2. Un **AI Agent** (Claude Sonnet, modo Tools Agent) interpreta el mensaje, decide la categoría/subcategoría del gasto según una taxonomía fija, y completa el resto de los datos (monto, moneda, medio de pago, tipo de gasto).
3. Registra el gasto como una fila nueva en una Google Sheet.
4. Te manda un mail de confirmación con lo que quedó registrado (log de observabilidad humana).

## Por qué es un agente (y no un flujo lineal)

La categorización de un gasto a partir de una descripción en lenguaje natural es una tarea **ambigua**, que requiere criterio (¿"subte" es Movilidad > Transporte Público? ¿"Netflix" es Compras > Suscripciones?) — por eso se delega en un agente con razonamiento (ciclo ReAct), en vez de reglas fijas tipo IF/Switch. Es el criterio central que enseña la Unidad 1: *"Mantené lo determinista y delegá lo ambiguo"*.

## Componentes del workflow

| Componente | Nodo | Detalle |
|---|---|---|
| Disparador | Telegram Trigger | Captura el mensaje desestructurado del usuario |
| Cerebro | AI Agent (Tools Agent) | System Prompt modular (Rol → Ámbito → Objetivo → Reglas → Escalamiento), sin lenguaje inclusivo |
| Guardrail | Max Iterations = 8 | Límite estricto contra bucles infinitos |
| Modelo | Anthropic Chat Model (Claude Sonnet) | Razonamiento y categorización |
| Herramienta | Google Sheets Tool | Agrega la fila del gasto; descripción semántica extensa de cuándo usarla |
| Observabilidad | Gmail | Envía el resultado de cada ejecución para auditoría humana |

## Guardrails implementados

- Límite de iteraciones (5-10, seteado en 8).
- Herramienta de mínimo privilegio: la Tool solo puede agregar filas, no editar ni borrar.
- El System Prompt define explícitamente qué NO puede hacer el agente (eliminar registros, acciones financieras reales, inventar datos).

## Cómo correrlo

1. Importar `Entregables/checkpoint1_rodrigo_zelaya.json` en una instancia de n8n (local o cloud).
2. Configurar credenciales propias: Telegram Bot API, Anthropic API, Google Sheets OAuth2, Gmail OAuth2.
3. Crear una Google Sheet con las columnas: Fecha, Concepto, Descripción, Categoría, Subcategoría, Monto, Moneda, Medio de Pago, Tipo de Gasto, Frecuencia, Día del Mes, Número de Cuotas.
4. Publicar el workflow y mandarle un mensaje al bot de Telegram.

## Roadmap del proyecto integrador

- **M1 (actual):** Agente base — Trigger + AI Agent + 1 Tool + Log.
- **M2:** Multi-agente (Manager + Workers).
- **M3:** Memoria y contexto por Session_ID.
- **M4:** Integraciones reales adicionales.
- **M5:** RAG / base documental.
- **M6:** Voz (STT/TTS).
- ... hasta el Proyecto Final Integrador (M11).
