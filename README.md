# Assessment - Metroia Developer Challenge

## Contexto

En Metroia desarrollamos agentes de IA para soporte al cliente de ISPs (proveedores de internet). Nuestros agentes atienden consultas por WhatsApp, buscan informacion en bases de conocimiento, consultan datos de clientes, y transfieren a agentes humanos cuando es necesario.

Este reto simula una version simplificada de lo que hacemos en produccion.

## El Reto

Construir un agente conversacional de soporte para **FibraNet**, un ISP ficticio. El agente debe atender consultas de clientes usando RAG (Retrieval-Augmented Generation) para informacion de la empresa y herramientas (tools) para consultar datos de clientes.

## Datos Proporcionados

En la carpeta `data/` encontraras:

- `planes.txt` - Planes de internet disponibles con precios y caracteristicas
- `soporte.txt` - Informacion de soporte tecnico, horarios y canales de contacto
- `pagos.txt` - Medios de pago, fechas de vencimiento y politicas de mora
- `customers.json` - Base de datos mock de clientes con RUT, plan, estado y deuda

## Requerimientos

### 1. RAG (Retrieval-Augmented Generation)

Cargar los 3 archivos de texto como base de conocimiento:
- Dividirlos en chunks apropiados
- Generar embeddings y almacenarlos en un vector store local (FAISS, Chroma, o similar)
- Implementar busqueda por similitud para responder consultas

### 2. Herramientas del Agente

Implementar las siguientes herramientas (tools) que el agente pueda invocar:

**a) `buscar_cliente(rut: str)`**
- Busca un cliente en `customers.json` por su RUT
- Debe validar el RUT chileno usando el algoritmo MOD 11 antes de buscar
- Retorna: nombre, plan, estado, y deuda pendiente
- Si el RUT es invalido, retorna error claro
- Si no encuentra el cliente, sugiere que no esta registrado

**b) `consultar_planes(consulta: str)`**
- Busca en la base de conocimiento los planes disponibles usando RAG
- Retorna la informacion relevante encontrada

**c) `consultar_info(consulta: str)`**
- Busca informacion general (soporte, pagos, horarios) usando RAG
- Retorna la informacion relevante encontrada

### 3. Agente Conversacional

Construir el agente usando **LangChain** o **LangGraph** (a eleccion):
- El agente decide cuando usar cada herramienta segun la consulta del usuario
- Mantiene contexto de la conversacion (memoria)
- Responde en espanol de forma clara y amigable

### 4. Interfaz

- CLI interactivo: ejecutar `python main.py` para iniciar una conversacion por terminal
- El usuario escribe, el agente responde, loop hasta que el usuario escriba "salir"

### 5. Validacion de RUT (MOD 11)

El RUT chileno se valida asi:
- Tomar el cuerpo numerico (sin digito verificador)
- Multiplicar cada digito de derecha a izquierda por los factores 2, 3, 4, 5, 6, 7 (ciclicamente)
- Sumar los productos
- Calcular: 11 - (suma % 11)
- Si el resultado es 11, el digito verificador es "0"; si es 10, es "K"; otro valor es el digito directamente
- Comparar con el digito verificador proporcionado

Ejemplo: RUT 12.345.678-5 es valido.

## Conversaciones de Prueba

El agente debe manejar correctamente al menos estas conversaciones:

```
Usuario: Hola, que planes de internet tienen?
Agente: [Debe listar los 3 planes con precios]

Usuario: Quiero revisar mi cuenta, mi RUT es 11.111.111-1
Agente: [Debe mostrar datos de Maria Rodriguez, plan Premium, deuda de $49.980]

Usuario: Como puedo pagar?
Agente: [Debe explicar los medios de pago disponibles]

Usuario: Mi internet esta lento, que hago?
Agente: [Debe dar pasos de solucion desde la base de conocimiento]

Usuario: Revisa el RUT 12345679-0
Agente: [Debe indicar que el RUT es invalido]
```

## Entregables

1. Codigo fuente en el repositorio (estructura libre, pero organizada)
2. El `requirements.txt` puede ser extendido con dependencias adicionales
3. Un `.env` con la API key necesaria (no commitear, usar `.env.example` como referencia)
4. Commits descriptivos que muestren el progreso del desarrollo

## Criterios de Evaluacion

| Criterio | Peso |
|---|---|
| Funcionalidad completa (RAG + tools + agente) | 35% |
| Calidad de codigo (limpio, modular, sin codigo muerto) | 25% |
| Implementacion correcta de RAG | 15% |
| Validacion de RUT y manejo de errores | 15% |
| Git (commits claros, progresivos) | 10% |

## Stack Sugerido

- **Python 3.11+**
- **LLM**: OpenAI GPT-4o-mini (barato y suficiente) o cualquier otro modelo via LangChain
- **Embeddings**: OpenAI `text-embedding-3-small` o sentence-transformers local
- **Vector Store**: FAISS (incluido en requirements.txt)
- **Framework**: LangChain o LangGraph

## Como Empezar

```bash
git clone https://github.com/piometrowan/Assessment.git
cd Assessment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows
pip install -r requirements.txt
cp .env.example .env
# Editar .env con tu API key
```

## Entrega

Hacer fork del repositorio, desarrollar la solucion, y enviar el link del fork a **pio@metrowan.cl** junto con instrucciones para ejecutar si difieren de lo descrito.
