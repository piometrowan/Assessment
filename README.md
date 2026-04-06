# Assessment - Metroia Developer Challenge

## Contexto

En Metroia construimos backends que integran multiples proveedores de LLMs para agentes de IA en produccion. Trabajamos con APIs de diferentes proveedores, streaming en tiempo real, y arquitecturas modulares que permiten intercambiar modelos facilmente.

Este reto evalua tu capacidad para leer documentacion, integrar APIs, y construir un backend funcional con streaming.

## El Reto

Construir un backend en Python que exponga una API de chat con streaming, integrando al menos **2 de los 3 proveedores gratuitos** listados abajo.

## Proveedores (todos tienen tier gratuito)

### Groq
- Consola: https://console.groq.com
- Documentacion: https://console.groq.com/docs/quickstart
- SDK: `groq`
- Modelos gratuitos: `llama-3.3-70b-versatile`, `gemma2-9b-it`, `llama-3.1-8b-instant`

### Cerebras
- Consola: https://cloud.cerebras.ai
- Documentacion: https://inference-docs.cerebras.ai/introduction
- SDK: `cerebras-cloud-sdk`
- Modelos gratuitos: `llama-3.3-70b`, `llama-3.1-8b`

### OpenRouter
- Consola: https://openrouter.ai
- Documentacion: https://openrouter.ai/docs/quickstart
- SDK: compatible con `openai` (cambiando base_url)
- Modelos gratuitos: buscar modelos con precio $0 en https://openrouter.ai/models

## Requerimientos

### 1. API Backend (FastAPI)

**`POST /chat`** - Enviar mensaje y recibir respuesta en streaming (SSE)

Request body:
```json
{
  "message": "Hola, como estas?",
  "provider": "groq",
  "model": "llama-3.3-70b-versatile",
  "session_id": "abc123"
}
```

Response: Server-Sent Events (SSE) con cada token del modelo.

**`GET /models`** - Listar modelos disponibles por proveedor

Response:
```json
{
  "groq": ["llama-3.3-70b-versatile", "gemma2-9b-it"],
  "cerebras": ["llama-3.3-70b", "llama-3.1-8b"],
  "openrouter": ["meta-llama/llama-3.3-70b-instruct:free"]
}
```

### 2. Streaming

Las respuestas del chat deben llegar token por token via SSE (Server-Sent Events). No se acepta esperar la respuesta completa y enviarla de golpe.

### 3. Memoria de Conversacion

Mantener el historial de mensajes por `session_id` (en memoria, no necesita base de datos). Si el usuario envia varios mensajes con el mismo `session_id`, el modelo debe tener contexto de los mensajes anteriores.

### 4. Manejo de Errores

- API key invalida o no configurada: error claro con el proveedor afectado
- Modelo no disponible: error indicando modelos validos
- Proveedor no soportado: error indicando proveedores disponibles
- Rate limit del proveedor: error descriptivo

### 5. Estructura del Proyecto

El codigo debe estar organizado en modulos, no todo en un solo archivo. Estructura sugerida (libre de modificar):

```
src/
  main.py          # Entry point, FastAPI app
  providers/       # Un modulo por proveedor
    groq.py
    cerebras.py
    openrouter.py
  models.py        # Schemas (Pydantic)
  memory.py        # Manejo de historial por sesion
```

## Pruebas Esperadas

El backend debe responder correctamente a estas llamadas:

```bash
# Chat con streaming (debe imprimir tokens progresivamente)
curl -N -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Que es Python en 2 oraciones?", "provider": "groq", "model": "llama-3.3-70b-versatile", "session_id": "test1"}'

# Segundo mensaje con contexto (debe recordar el mensaje anterior)
curl -N -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Y para que se usa?", "provider": "groq", "model": "llama-3.3-70b-versatile", "session_id": "test1"}'

# Listar modelos
curl http://localhost:8000/models

# Error: proveedor invalido
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "hola", "provider": "invalido", "model": "x", "session_id": "test2"}'
```

## Bonus (opcional, suma puntos)

- Frontend simple: una pagina HTML con un chat que consuma el endpoint SSE y muestre los tokens en tiempo real
- Poder comparar respuestas: enviar el mismo mensaje a 2 proveedores en paralelo y ver ambas respuestas
- Docker: `docker compose up` para levantar todo

## Entregables

1. Codigo fuente organizado en el repositorio
2. `requirements.txt` con todas las dependencias
3. `.env.example` documentando las API keys necesarias (no commitear el `.env` real)
4. Commits descriptivos que muestren el progreso del desarrollo

## Criterios de Evaluacion

| Criterio | Peso |
|---|---|
| Funcionalidad (streaming + multiples proveedores) | 30% |
| Calidad de codigo (modular, limpio, sin codigo muerto) | 25% |
| Integracion correcta de SDKs (leer docs e implementar) | 20% |
| Manejo de errores | 15% |
| Git (commits claros, progresivos) | 10% |

## Stack

- **Python 3.11+**
- **FastAPI** + **uvicorn**
- **SSE**: `sse-starlette` o streaming response nativo de FastAPI
- **SDKs**: `groq`, `cerebras-cloud-sdk`, `openai` (para OpenRouter)

## Como Empezar

```bash
# 1. Fork y clonar
git clone https://github.com/TU_USUARIO/Assessment.git
cd Assessment

# 2. Entorno virtual
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Configurar API keys
cp .env.example .env
# Editar .env con tus API keys (registrarse en los proveedores, todos son gratis)

# 5. Ejecutar
uvicorn src.main:app --reload
```

## Entrega

Hacer fork del repositorio, desarrollar la solucion, y enviar el link del fork a **pio@metrowan.cl**.
