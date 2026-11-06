# Charts de artistas

Visor al estilo de `charts_sellos`, pero por artista: escribes un nombre y muestra su discografía, con vídeos de YouTube y la información de Last.fm.

## Para qué sirve

Para explorar la discografía de un artista de un vistazo: un mosaico de portadas con vídeos de YouTube e información de Last.fm, guardado en tu servidor para no repetir consultas.

## Instalación individual

Necesitas Docker.

```bash
git clone https://github.com/volteret4/charts_artistas.git
```

```bash
cd charts_artistas && cp .env.example .env && mkdir -p data
```

```bash
docker build -t charts-artistas .
```

```bash
docker run -d --name charts-artistas --restart unless-stopped --env-file .env -e CHARTS_DATA_DIR=/app/data -v "$PWD/data:/app/data" -p 8793:8793 charts-artistas
```

Se abre en <http://localhost:8793>.

## Instalación desde REPEAT-ONE

Este módulo forma parte de [REPEAT-ONE](https://volteret4.github.io/repeat-one-web/), que permite elegir qué módulos instalar: [ver la instalación](https://github.com/volteret4/repeat-one-web#instalación). Allí se llama `charts_artistas`.

## Versión pública

Hay una [versión para navegador](https://volteret4.github.io/charts-web/artistas/) que no necesita servidor: guarda las búsquedas en tu navegador y usa tus propias claves.

## Cómo funciona

1. **Búsqueda**: se piden candidatos a MusicBrainz y Discogs. Se marca por defecto la coincidencia exacta de cada fuente, y puedes cambiarla porque hay muchos artistas homónimos.
2. **Discografía**: se juntan los dos catálogos por artista + título. De Discogs solo cuentan los discos donde el artista es el titular (`role=Main`); sus remezclas y apariciones no. MusicBrainz manda en el tipo de disco. Lo que solo está en Discogs saca el tipo del formato de su edición principal.
3. **Last.fm**: oyentes, escuchas, etiquetas, portada, wiki y canciones del disco; biografía, etiquetas y similares del artista. Con `LASTFM_USERNAME` también tus escuchas de cada disco. Se refresca a los 14 días.
4. **YouTube**: `yt-dlp` busca y guarda hasta 5 vídeos cuyo título o canal nombren disco y artista. Si un disco solo está subido pista a pista, valen los vídeos del artista que nombren una de sus canciones. Si aún no se ha buscado un disco, se busca al pulsarlo.

Al añadir un artista eliges qué secciones traer (álbumes, EP, singles, recopilatorios…). El botón **Secciones** las cambia después: lo desmarcado desaparece y, al volver a marcarlo, no se repiten las consultas ya guardadas.

Al pulsar un artista similar en la cabecera, se busca para añadirlo.

Hay dos colas independientes. La primera trae discografía y Last.fm; al terminar, el artista pasa a la de YouTube. Así, mientras se buscan los vídeos de un artista, los siguientes ya van trayendo su información. YouTube va después porque aprovecha las canciones y la duración que da Last.fm.

## Desarrollo

Sin Docker y desde esta carpeta (lee `.env` si existe):

```bash
pip install -r requirements.txt && python app.py
```

| Variable | Para qué |
|---|---|
| `LASTFM_API_KEY` | Información de Last.fm |
| `DISCOGS_TOKEN` | Opcional: sin él solo se usa MusicBrainz |
| `LASTFM_USERNAME` | Opcional: tus escuchas de cada disco |
| `YT_WORKERS` | Búsquedas de YouTube simultáneas (2 por defecto). Cada una suma unas 1700 búsquedas/h desde la misma IP |
| `SETTINGS_PASSWORD` | Panel de configuración ⚙ |
| `CHARTS_DATA_DIR` | Carpeta de datos (por defecto `data/`) |

## Si YouTube bloquea las búsquedas

Si responde «Sign in to confirm you're not a bot», la cola de YouTube se pausa 30 minutos y reintenta sola; si sigue bloqueado, la pausa se duplica hasta un máximo de 4 horas. La cola de Last.fm sigue avanzando. Si no se resuelve, exporta las cookies en formato Netscape a `data/cookies.txt` y pulsa ↻ en cualquier elemento para reintentar ya: solo se buscan los discos que faltan.

## Limitaciones

- Dos ediciones con títulos distintos («Beat» y «Beat (Remastered)») cuentan como discos distintos.
- Los iconos de «ya lo tengo» vienen del servicio `ya_lo_tengo` (puerto 8780); sin él, simplemente no salen.

## Código

- `app.py`: servidor sin dependencias web, trabajos en segundo plano y SQLite (`data/charts_artistas.db`).
- `static/app.js`: la web.
