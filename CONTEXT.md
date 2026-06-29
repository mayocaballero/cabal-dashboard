# Cabal Dashboard — CONTEXT.md

Documentación de cómo funciona este proyecto, para no tener que re-explicarlo en cada sesión.

## Qué es

El dashboard personal de Mayo ("Digital garden"), que muestra el estado de todos sus proyectos — artísticos, técnicos, de trabajo y financieros — en tarjetas filtrables por área.

Live en: `cabal-dashboard.pages.dev`

## Arquitectura real (confirmada line by line, no asumida)

- **Fuente de verdad del contenido:** Google Sheet
  ID: `1Um5Gs3bLQyfQrhh19N83LobYlbtRiKjVqVR6GCjjxr8`
  Publicado como CSV en una URL pública de `docs.google.com` (ver constante `CSV_URL` en `index.html`, línea ~273).
  Columnas: `title`, `area`, `area_label`, `status`, `next`, `desc`, `where`, `color`, `text_color`, `visible`.
  El campo `visible` (yes/no) controla qué aparece en el dashboard.
  Status posibles: `active`, `blocked`, `incubating`, `pending`.

- **Fuente de verdad del código:** repo GitHub `mayocaballero/cabal-dashboard`, rama `main`.

- **Despliegue:** Cloudflare Pages conectado al repo con autodeploy. Cualquier push a `main` redeploya solo, sin pasos manuales.

- **Cómo carga los datos `index.html`:**
  Hace `fetch(CSV_URL + '&t=' + Date.now())` en cada carga (el timestamp evita caché del navegador/Google).
  Cachea el resultado en `localStorage` bajo la key `cabal_v2`, así que la segunda visita en adelante es instantánea mientras actualiza en segundo plano.

- **`projects.json` (en la raíz del repo):**
  **No se usa.** No hay ningún `fetch` ni referencia a este archivo en `index.html` — se confirmó con `grep -n "fetch\|projects.json\|docs.google.com\|csv" index.html` y solo aparece el CSV_URL.
  Es un archivo huérfano: probablemente un export puntual de los datos en formato JSON (fecha interna `"updated": "2026-06-17"`, no se actualiza sola). Pendiente de decidir: borrarlo, o reutilizarlo si en el futuro se migra de Sheets a JSON nativo versionado en el propio repo.

## Diseño

Basado en The Creative Independent (thecreativeindependent.com):
- Tipografía monospace (`Courier, "Courier New", monospace`), sin bold en ningún sitio.
- Cards de 270×270px con borde punteado que se vuelve sólido al hover.
- Gap de `0.625rem` entre cards.
- Filtros como botones de texto con borde punteado.
- Vista de detalle a página completa al hacer click en un proyecto (no overlay).
- Header fijo "Cabal Studio" / subtítulo "Digital garden", sin borde inferior.
- Mobile: cards 4:3, ancho completo.

## Cómo hacer cambios

| Quiero cambiar... | Dónde lo hago |
|---|---|
| Un proyecto, su estado, descripción, visibilidad | **Google Sheet** directamente — nunca el HTML |
| Diseño, CSS, lógica de filtros, estructura | `index.html` en local → `git add` → `git commit` → `git push` → Cloudflare autodeploya |

## Setup local

Repo clonado en: `~/Documents/cabal-dashboard` (Mac de Mayo).

```bash
cd ~/Documents/cabal-dashboard
git pull        # antes de empezar, por si hay cambios remotos
# ... editar index.html ...
git add .
git commit -m "descripción del cambio"
git push
```

## Pendiente / decisiones abiertas

- Decidir destino de `projects.json` (borrar vs. reutilizar).
- Evolución futura hacia "jardín digital" de verdad: fotos, texto largo. Posible camino: Cabal Webs (motor PHP ya construido, sin desplegar, vive en Google Drive en `My Drive/Cabal Studio/cabal-webs`) en lugar de seguir solo con Sheets + HTML estático.


