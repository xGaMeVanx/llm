# Agente de IA

Agente conversacional que responde preguntas sobre el Instituto Nébula — cursos,
precios, becas y disponibilidad — usando herramientas reales en lugar de responder
de memoria. Está expuesto como una API HTTP (FastAPI) y listo para desplegar en Render.

## Cómo funciona

El agente recibe una pregunta y entra en un bucle de hasta 6 vueltas: el modelo
decide qué herramienta(s) llamar, las ejecuta y vuelve a decidir con el resultado,
hasta reunir la respuesta completa. Cada respuesta expone la **trayectoria** de
llamadas, para que el razonamiento no sea una caja negra.

### Herramientas

| Herramienta          | Qué hace                                                        |
|----------------------|-----------------------------------------------------------------|
| `calcular_costo`     | Desglose del precio de un curso: base, beca, subtotal, IVA, total |
| `consultar_cupo`     | Lugares disponibles de un curso (simula el servicio de inscripciones) |
| `buscar_en_corpus`   | Búsqueda semántica en los documentos del Instituto (RAG)         |

### Datos

- `indice.json` — 27 fragmentos (becas, catálogo, FAQ) embebidos con
  `gemini-embedding-001` a 768 dimensiones y normalizados.
- Umbral de similitud `0.68` — si el fragmento más parecido no lo alcanza, el agente
  admite que la pregunta no está documentada.

## Stack

- **FastAPI** + **uvicorn** — API y servidor
- **Groq** (`openai/gpt-oss-120b`) — tool calling
- **google-genai** (`gemini-embedding-001`) — embeddings
- **numpy** — similitud coseno

## Requisitos

Dos claves de API gratuitas:

- `GROQ_API_KEY` — consola de Groq
- `GEMINI_API_KEY` — Google AI Studio

Se pasan como variables de entorno; nunca se escriben en el código ni se suben al repo.

## Correr localmente

```bash
export GROQ_API_KEY=...
export GEMINI_API_KEY=...
pip install -r requirements.txt
uvicorn main:app --reload
```

Abre <http://localhost:8000>.

## API

| Endpoint    | Método | Descripción                                   |
|-------------|--------|-----------------------------------------------|
| `/`         | GET    | Página web de prueba                          |
| `/health`   | GET    | Healthcheck — responde `{"status":"ok"}`      |
| `/preguntar`| POST   | Envía una pregunta, devuelve respuesta + trayectoria |

Ejemplo:

```bash
curl -X POST http://localhost:8000/preguntar \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "¿Cuánto me sale el NBL-204 en línea con la beca de egresados?"}'
```

```json
{
  "respuesta": "...",
  "trayectoria": [
    {"vuelta": 1, "herramienta": "calcular_costo", "argumentos": {"codigo_curso": "NBL-204", "modalidad": "en línea", "tipo_beca": "egresados"}},
    {"vuelta": 1, "herramienta": "consultar_cupo", "argumentos": {"codigo_curso": "NBL-204"}}
  ],
  "vueltas": 2
}
```

## Despliegue en Render

El repo incluye `render.yaml`. En Render: **New + → Blueprint**, conecta el repositorio
y pega las dos claves cuando el formulario las pida. Render construye y publica el
servicio, y cada `git push` vuelve a desplegar.

## Estructura

| Archivo            | Contenido                                                       |
|--------------------|-----------------------------------------------------------------|
| `main.py`          | El agente completo: índice, herramientas, bucle y API           |
| `indice.json`      | Documentos del Instituto embebidos (~200 KB)                    |
| `render.yaml`      | Receta de despliegue para Render                                |
| `requirements.txt` | Dependencias                                                    |
