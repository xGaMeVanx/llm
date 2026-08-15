# Agente del Instituto Nébula — despliegue

Esta carpeta es un repositorio completo y autosuficiente. Contiene un agente que
responde preguntas sobre el Instituto Nébula usando tres herramientas, expuesto como una
API HTTP.

Al terminar esta guía vas a tener una URL pública, tuya, que funciona desde cualquier
navegador del mundo. **No necesitas haber desplegado nada antes.** Toma unos 15 minutos.

## Qué hay aquí

| Archivo            | Qué es                                                          |
|--------------------|-----------------------------------------------------------------|
| `main.py`          | Todo: el índice, las tres herramientas, el agente y la API      |
| `indice.json`      | Los documentos del Instituto ya convertidos a vectores          |
| `requirements.txt` | Las cinco librerías que necesita                                |
| `render.yaml`      | La receta que Render lee para saber cómo construir y arrancar   |

## Antes de empezar necesitas dos llaves

Son gratis las dos. Consíguelas ahora y déjalas en un bloc de notas: te las van a pedir
más adelante y no vas a querer interrumpir el proceso.

1. **GROQ_API_KEY** — en <https://console.groq.com/keys>. Crea una cuenta, entra a
   *API Keys*, botón *Create API Key*. Empieza con `gsk_`. **Cópiala en ese momento:
   Groq no te la vuelve a mostrar.**
2. **GEMINI_API_KEY** — en <https://aistudio.google.com/apikey>. Botón
   *Create API key*. Esta sí la puedes volver a ver después.

> Estas llaves son como contraseñas. No las pegues en el código, no las subas a GitHub,
> no las compartas en el chat de la clase. Si se te escapa una, bórrala desde la consola
> y crea otra: toma diez segundos.

---

## Paso 1 — Sube esta carpeta a TU GitHub

Necesitas una cuenta en <https://github.com>. Si no tienes, créala; es gratis.

1. En GitHub, arriba a la derecha, botón **+** → **New repository**.
2. Ponle un nombre, por ejemplo `agente-nebula`.
3. Déjalo **Public** (el plan gratuito de Render también funciona con repos privados,
   pero público es una cosa menos que configurar).
4. **No** marques "Add a README file". Queremos el repositorio vacío.
5. Botón **Create repository**.

GitHub te va a mostrar una pantalla con comandos. Ignórala y usa estos, desde una
terminal parada **dentro de esta carpeta `deploy/`**:

```bash
git init
git add .
git commit -m "Mi agente del Instituto Nébula"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/agente-nebula.git
git push -u origin main
```

Cambia `TU-USUARIO` por tu usuario de GitHub. Si te pide contraseña, GitHub ya no acepta
la de tu cuenta: usa un *Personal Access Token* (Settings → Developer settings → Personal
access tokens → Tokens (classic) → Generate new token, con el permiso `repo`).

Recarga la página del repositorio. Deberías ver `main.py`, `indice.json`, `render.yaml`
y este README. **Lo que no debes ver es ningún archivo `.env`.** Si lo ves, bórralo del
repositorio y rota tus llaves.

## Paso 2 — Crea la cuenta en Render

1. Entra a <https://render.com> y haz clic en **Get Started**.
2. Elige **GitHub** para registrarte. Es lo más sencillo: Render queda conectado con tu
   GitHub y va a poder leer tus repositorios.
3. GitHub te va a pedir autorizar a Render. Acepta.
4. Cuando te pregunte a qué repositorios darle acceso, puedes dar acceso a todos o solo
   a `agente-nebula`. Cualquiera de las dos está bien.

## Paso 3 — Despliega con un Blueprint

Un *Blueprint* es Render leyendo el `render.yaml` de tu repo y armando el servicio solo.
No tienes que llenar formularios de configuración.

1. En el panel de Render: **New +** → **Blueprint**.
2. Busca tu repositorio `agente-nebula` en la lista y selecciónalo. Botón **Connect**.
3. Render lee `render.yaml`, ve que hay un servicio web llamado `agente-nebula`, y —esto
   es lo importante— ve que faltan dos variables de entorno. **Te las va a pedir en un
   formulario.**
4. Pega tus dos llaves:
   - `GROQ_API_KEY` → la que empieza con `gsk_`
   - `GEMINI_API_KEY` → la de Google AI Studio
5. Botón **Apply** / **Create resources**.

> Esas dos llaves quedan guardadas en Render, **no en tu repositorio**. Eso es lo que
> significa el `sync: false` en `render.yaml`: "este valor no viene del archivo,
> pregúntaselo a la persona". Por eso el repo puede ser público sin peligro.

## Paso 4 — Espera el build

Render va a instalar las librerías y arrancar el servicio. Verás los logs en vivo. Toma
entre 2 y 5 minutos la primera vez.

Lo que estás buscando en los logs, en este orden:

```
==> Installing dependencies with pip
...
==> Build successful 🎉
==> Deploying...
INFO:     Uvicorn running on http://0.0.0.0:10000
==> Your service is live 🎉
```

Arriba de la página está tu URL, algo como
`https://agente-nebula-xxxx.onrender.com`. Cópiala.

## Paso 5 — Pruébalo

**Primero lo más simple**, pega esto en el navegador:

```
https://TU-SERVICIO.onrender.com/health
```

Debe responder `{"status":"ok"}`. Si ves eso, el servidor está vivo.

**Ahora la interfaz.** Entra a la raíz:

```
https://TU-SERVICIO.onrender.com/
```

Vas a ver una página con un campo de texto. Pregunta algo como
*"¿cuánto me sale el NBL-204 en línea con la beca de egresados?"*.
Abajo aparece la respuesta y la lista de herramientas que el agente llamó.

**Y desde la terminal**, que es como lo va a consumir el cliente de Streamlit:

```bash
curl -X POST https://TU-SERVICIO.onrender.com/preguntar \
  -H "Content-Type: application/json" \
  -d '{"pregunta": "¿Cuánto me sale el NBL-204 en línea con la beca de egresados y todavía hay lugar?"}'
```

La respuesta trae tres campos:

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

La `trayectoria` está expuesta a propósito: es lo que hace que el agente no sea una caja
negra. El cliente de Streamlit la muestra en un desplegable.

---

## Correrlo en tu máquina (opcional, pero útil para depurar)

```bash
export GROQ_API_KEY=gsk_tu_llave
export GEMINI_API_KEY=tu_llave
pip install -r requirements.txt
uvicorn main:app --reload
```

Y abre <http://localhost:8000>.

Si `pip install` te responde `error: externally-managed-environment`, tu sistema no deja
instalar paquetes de Python globalmente. Crea un entorno aislado y vuelve a intentar:

```bash
python3 -m venv .venv
source .venv/bin/activate      # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## Cuando algo sale mal

### El build falla o el servicio se queda en "Deploying" para siempre

Casi siempre es el **puerto**. Render le asigna un puerto a tu servicio en la variable
`$PORT` y espera que escuches ahí. Si en algún momento cambias el `startCommand` a algo
como `--port 8000`, Render nunca va a encontrar tu servicio y el deploy se queda colgado
hasta que expira.

El comando correcto, ya está en `render.yaml`, es:

```
uvicorn main:app --host 0.0.0.0 --port $PORT
```

`--host 0.0.0.0` también importa: si escuchas solo en `localhost`, el servicio no acepta
tráfico de afuera del contenedor.

### `model_decommissioned` en los logs de Groq

```
groq.NotFoundError: {'error': {'message': 'The model `...` has been decommissioned...'}}
```

Groq apaga modelos viejos con cierta frecuencia. Este proyecto usa
`openai/gpt-oss-120b`, que está vigente. Si algún día lo apagan, abre
<https://console.groq.com/docs/models>, busca un modelo que diga que soporta *Tool Use*,
y cambia la constante `MODELO` al principio de `main.py`. Es una línea.

### Error 429 de Gemini

```
google.genai.errors.ClientError: 429 RESOURCE_EXHAUSTED
```

Te pasaste del límite gratuito de embeddings. Cada pregunta que hace el agente consume
una llamada a `gemini-embedding-001` **solo si decide usar `buscar_en_corpus`** — los
documentos ya están embebidos en `indice.json` y no se vuelven a procesar.

Espera un minuto y vuelve a intentar. Si estás enseñando en vivo con muchas personas
pegándole al mismo servicio, es el límite por minuto lo que se satura, no una cuota
diaria agotada.

### La primera pregunta tarda un minuto y luego todo va rápido

**No es un bug.** El plan gratuito de Render duerme el servicio después de 15 minutos sin
tráfico. La siguiente petición lo despierta, y arrancar el proceso, cargar `indice.json` y
levantar numpy toma cerca de un minuto.

A partir de ahí responde en un par de segundos, hasta que se vuelva a dormir. Si estás por
demostrar algo en vivo, pégale al `/health` cinco minutos antes para tenerlo despierto.

### El agente responde pero no usa herramientas

Revisa los logs y la `trayectoria` que devuelve `/preguntar`. Si viene vacía, el modelo
decidió responder de memoria. Casi siempre se arregla en las descripciones del
`ESQUEMA_HERRAMIENTAS` de `main.py`: esas descripciones son el prompt que el modelo lee
para decidir. Sé más explícito sobre cuándo usar cada herramienta.

### Cambié el código, ¿cómo actualizo el servicio?

```bash
git add .
git commit -m "lo que cambiaste"
git push
```

Render detecta el push y redespliega solo. No hay que tocar nada en su panel.
#   l l m  
 