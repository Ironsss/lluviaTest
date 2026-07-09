# 🌧️ Lluvia CDMX — Radar meteorológico en vivo

Dashboard interactivo de **precipitación en tiempo real** para la Ciudad de México. Muestra un radar de lluvia animable sobre las 16 alcaldías, con dirección del sistema, geolocalización y una interfaz estilo iOS nativo. Todo en un solo archivo HTML, sin build, sin dependencias que instalar y sin API keys.

**Demo:** https://ironsss.github.io/lluviaTest/

![Estado](https://img.shields.io/badge/estado-en%20vivo-4cc9f0) ![Sin API key](https://img.shields.io/badge/API%20key-no%20requerida-3ad6a0) ![Licencia](https://img.shields.io/badge/licencia-MIT-blue)

---

## Características

- **Radar de lluvia real** — reflectividad de precipitación de RainViewer, la misma que usan apps meteorológicas profesionales, refrescada cada 5 minutos.
- **Línea de tiempo animable** — reproduce las últimas 2 horas de lluvia y frames de *nowcast* (pronóstico a corto plazo) para ver hacia dónde se mueve el sistema.
- **Campo vectorial de dirección** — una flecha que apunta hacia dónde se desplaza la lluvia, calculada a partir del viento a 10 m en el centro de la ciudad.
- **Mi ubicación** — botón de geolocalización con marcador "estás aquí" y círculo de precisión, para saber si te va a llover encima ahora mismo.
- **Interfaz móvil-first** — mapa a pantalla completa con *bottom sheet* deslizable, glassmorphism sutil y switches estilo iOS. Respeta el notch y la barra inferior del iPhone.
- **Modo pantalla completa** y capas conmutables (radar, flecha, nowcast, auto-actualización).
- **Tolerante a fallos** — si el viento no está disponible, el radar sigue funcionando; los errores de red se reintentan automáticamente.

---

## Cómo funciona

El dashboard combina tres fuentes de datos en un mapa de [Leaflet](https://leafletjs.com/):

```
┌─────────────────────────────────────────────┐
│  Etiquetas de calles y colonias (CARTO)      │  ← siempre encima
├─────────────────────────────────────────────┤
│  Radar de lluvia (RainViewer, translúcido)   │  ← capa intermedia
├─────────────────────────────────────────────┤
│  Mapa base oscuro sin etiquetas (CARTO)      │  ← fondo
└─────────────────────────────────────────────┘
      + flecha de dirección (SVG, viento Open-Meteo)
```

El radar se sirve como **tiles de imagen** (una sola petición ligera por refresco), lo que evita por completo los límites de cuota que aparecen al muestrear precipitación punto por punto sobre una malla.

---

## APIs consumidas

Todas gratuitas y sin API key para uso personal/educativo.

| Fuente | Endpoint | Uso | Frecuencia |
|---|---|---|---|
| **RainViewer** | `api.rainviewer.com/public/weather-maps.json` | Radar de lluvia (capa principal + timeline) | 1 petición / 5 min |
| **Open-Meteo** | `api.open-meteo.com/v1/forecast` | Viento a 10 m (flecha de dirección) | 1 petición / 5 min |
| **CARTO Basemaps** | `basemaps.cartocdn.com` | Tiles del mapa base y etiquetas | Bajo demanda (pan/zoom) |
| **Geolocation API** | `navigator.geolocation` | Ubicación del usuario (local del dispositivo) | Al tocar el botón |

### Notas sobre las APIs

- **RainViewer** ofrece resolución nativa hasta el nivel de zoom ~7 (≈1 km por celda). Al acercar más, la lluvia se ve suavizada — es normal en cualquier radar meteorológico.
- **Open-Meteo** en su tier gratuito limita a ~10 000 llamadas/día, 600/min por IP. Como GitHub Pages y las redes móviles usan IPs compartidas, esa cuota puede agotarse; por eso el viento es una fuente *secundaria* y su fallo no rompe el radar.
- El viento se consulta en **un solo punto** (centro de CDMX) para minimizar el consumo.

---

## Uso

### Opción A — abrir el demo

Entra a https://ironsss.github.io/IluviaTest/ desde cualquier navegador. En iPhone, para que se sienta como app nativa: **Compartir → Agregar a inicio**; así se abre a pantalla completa, sin barra del navegador.

### Opción B — correr localmente

```bash
git clone https://github.com/ironsss/IluviaTest.git
cd IluviaTest
# abrir index.html en el navegador, o servirlo:
python3 -m http.server 8000
# luego visita http://localhost:8000
```

> **Importante:** la geolocalización solo funciona sobre `https://` o `localhost`. Abrir el archivo con `file://` deshabilita el botón de ubicación.

---

## Despliegue en GitHub Pages

1. Sube `index.html` (y este `README.md`) a la raíz del repositorio.
2. Añade un archivo vacío llamado `.nojekyll` en la raíz — evita que Pages intente procesar el sitio con Jekyll y falle.
3. **Settings → Pages → Source:** `Deploy from a branch` → rama `main` → carpeta `/ (root)` → **Save**.
4. Espera ~1 minuto. La URL aparecerá en esa misma página.

> Si el build falla con `Conversion error: Jekyll::Converters::Scss`, es justamente el paso 2: falta el `.nojekyll`.

---

## Controles

| Control | Acción |
|---|---|
| 🎯 Botón de ubicación | Centra el mapa en tu posición y marca "estás aquí" |
| ⛶ Botón de pantalla completa | Maximiza el mapa (colapsa el panel en iOS) |
| ▶ / ❚❚ | Reproduce/pausa la animación de la lluvia |
| Deslizador de tiempo | Muévete entre frames pasados y nowcast |
| *Bottom sheet* | Arrástralo hacia arriba para ver stats, capas y leyenda |
| Switches | Activa/desactiva radar, flecha, nowcast y auto-actualización |

---

## Stack técnico

- **HTML + CSS + JavaScript vanilla** — un solo archivo, cero build.
- **[Leaflet 1.9.4](https://leafletjs.com/)** — motor de mapas (vía CDN).
- **SVG** — campo vectorial de dirección.
- **CSS moderno** — `backdrop-filter` (glassmorphism), `env(safe-area-inset-*)`, panes personalizados de Leaflet para el orden de capas.
- **Bottom sheet** con física de arrastre propia (touch events + snap points).

---

## Limitaciones conocidas

- El radar de RainViewer depende de la disponibilidad de los datos de origen; RainViewer no garantiza continuidad.
- La flecha de dirección refleja el viento del **centro** de la ciudad, no un campo por celda (RainViewer no expone viento). Para una malla completa de vectores haría falta volver a Open-Meteo con API key propia.
- La resolución del radar es meteorológica (~1 km), no calle por calle. Las calles del mapa base sí se ven nítidas a cualquier zoom.

---

## Atribución y licencias

Este proyecto usa datos de terceros bajo sus respectivas licencias:

- **Radar:** [RainViewer](https://www.rainviewer.com/) — uso personal y educativo. Se requiere mencionar la fuente con enlace.
- **Viento:** [Open-Meteo](https://open-meteo.com/) — datos bajo [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- **Mapa base:** [CARTO](https://carto.com/) y [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors.

El código de este repositorio se publica bajo licencia **MIT** (ver `LICENSE`).

---

## Autor

Desarrollado por **David García** ([@ironsss](https://github.com/ironsss)).

Contribuciones y sugerencias son bienvenidas — abre un *issue* o un *pull request*.
