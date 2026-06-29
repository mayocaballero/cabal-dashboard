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

## Dos formas de subir cambios a GitHub

| | Desde la web de GitHub | Desde terminal (git) |
|---|---|---|
| Necesita login | Sesión normal de GitHub (sin token) | Token (ya configurado, ver arriba) |
| Útil para | Cambios puntuales, un archivo, rápido | Cambios de código, varios archivos, control de versiones real |
| Actualiza la copia local | No, automáticamente no | Sí, ya queda sincronizado |

**Importante:** si se sube algo desde la web (arrastrando un archivo o "Add file → Upload files"), la copia local en `~/Documents/cabal-dashboard` se queda desactualizada hasta que se haga `git pull`. Si después de eso se sigue trabajando en local sin pulear primero, el próximo `git push` puede dar conflicto.

**Regla simple:** si se ha tocado algo desde la web, antes de volver a trabajar en terminal:
```bash
git pull
```

## Autenticación con GitHub (configurada, no tocar a menos que falle)

GitHub ya no acepta contraseña normal por HTTPS — hace falta un Personal Access Token (PAT).

- **Cómo está resuelto ahora:** el token vive incrustado directamente en la URL del remote, en `.git/config` de este repo en local. Por eso `git push` y `git pull` funcionan sin pedir usuario/contraseña cada vez.
- **Para verlo:** `git remote -v` (mostrará la URL con el token incluido — no compartir esa salida con nadie, ni pegarla en chats).
- **Tipo de token usado:** fine-grained, scope limitado solo a este repo (`mayocaballero/cabal-dashboard`), permiso "Contents: Read and write".
- **Dónde se gestionan los tokens:** `github.com/settings/tokens` → pestaña "Fine-grained tokens".
- **Si algún día deja de funcionar** (token expirado o revocado): generar uno nuevo con esos mismos permisos, y volver a correr:
  ```bash
  git remote set-url origin https://mayocaballero:NUEVO_TOKEN@github.com/mayocaballero/cabal-dashboard.git
  ```
  Mejor construir esa línea en un editor de texto aparte (Notas/TextEdit), sustituyendo solo la parte del token, y luego copiar/pegar la línea completa en terminal — pegar el token directamente en los prompts interactivos de Username/Password de git ha dado problemas en este Mac (el terminal no completaba bien el pegado).
- **Nota de identidad de commits:** los commits se firman ahora con un email genérico autogenerado (`mayo.caballero@Mays-MacBook-Air.local`). Si se quiere un email/nombre real, fijarlo con `git config --global user.name` / `user.email` — no urgente, solo cosmético.

## Pendiente / decisiones abiertas

- Decidir destino de `projects.json` (borrar vs. reutilizar).
- Evolución futura hacia "jardín digital" de verdad: fotos, texto largo. Posible camino: Cabal Webs (motor PHP ya construido, sin desplegar, vive en Google Drive en `My Drive/Cabal Studio/cabal-webs`) en lugar de seguir solo con Sheets + HTML estático.
