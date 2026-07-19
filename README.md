[README.md](https://github.com/user-attachments/files/30160303/README.md)
# QA Metrics Dashboard

Dashboard de métricas de QA/SDLC para **EPAM COPA · digital-products**. Es una app de una sola página HTML (`sdlc_dashboard.html`), sin backend ni build: se carga en el navegador, se suben archivos CSV exportados de Azure DevOps y calcula KPIs de calidad por sprint.

Todo el procesamiento ocurre en el cliente (JavaScript). No se envía ningún dato a un servidor.

## Cómo usarlo

Abrir `sdlc_dashboard.html` directamente en el navegador (doble clic o `open sdlc_dashboard.html`). No requiere instalación ni servidor local.

1. En la pestaña **📂 Upload**, cargar:
   - **ValidBugs CSV** — todos los bugs válidos, de todos los sprints y ambientes.
   - **ReopenedBugs CSV** (opcional) — bugs reabiertos, usado para el cálculo de Reopen Rate. Si no se carga, se usa como fallback el campo `State`.
   - **US QA Testing CSV** — uno por sprint (subir y presionar "+ Add this sprint" por cada archivo).
2. Verificar en "Detected sprints" que cada sprint tenga ✓ en Bugs y US.
3. Presionar **▶ Calculate dashboard**.
4. Explorar las pestañas: Overview, Compare, Trends, Forecast, Defect Density, Risk Heatmap.

Los umbrales de color (verde/amarillo/rojo) de cada KPI se pueden ajustar desde **⚙️ Adjust thresholds** y quedan guardados en `localStorage`, igual que un resumen de la última sesión cargada (banner de restauración).

## Columnas CSV esperadas

El parseo es tolerante a variaciones de nombre de columna, pero en general se espera:

- **ID / Id** — identificador único del work item.
- **Area Path** — se usa el último segmento tras `\` (ej. `Payments`, `Seatmap`, `Servicing`, etc.).
- **Sprint / Iteration Path** — debe contener un patrón `YYYY.MM[A-Z]` (ej. `2026.07A`) para poder mapearse al calendario de sprints predefinido.
- **Environment** (solo bugs) — Testing / UAT / Staging / Production.
- **Severity** (solo bugs) — `1 - Critical`, `2 - High`, `3 - Medium`, `4 - Low`.
- **State** — usado para reopen rate (fallback), bugs cerrados/resueltos, y US bloqueadas (`Blocked`, `Blocked UAT`).
- **Created Date / CreatedDate / Created** (solo bugs) — para el aging (0-7, 8-14, 15-30, +30 días).
- **Story Points** (solo US).
- **Ready for UAT Tracking** / **Ready for PROD Tracking** (esta última para el área `DS-Factory`) — fecha de listo para pruebas, usada para lead time y "US Ready for UAT".
- **Ready for Dev** (o similar) — usado para calcular Incomplete DoR (Definition of Ready).

## KPIs calculados

- **Defect Leakage (Prod / UAT)** — % de bugs detectados en Producción / UAT sobre el total.
- **DRE (Defect Removal Efficiency)** — % de bugs no escapados a Producción.
- **High & Critical UAT+** — bugs Severidad 1-2 detectados en UAT, Staging o Producción.
- **Valid Bugs** — total de bugs válidos del sprint.
- **Sprint Story Points** — story points totales cargados.
- **US Ready for UAT** — historias marcadas como listas para pruebas UAT/PROD.
- **Blocked US** — historias en estado bloqueado.
- **Incomplete DoR** — historias sin fecha de "Ready for Dev".
- **Defect Density** — bugs de Testing / story points, por área funcional.
- **Reopen Rate** — % de bugs reabiertos sobre bugs cerrados/resueltos.
- **Release Readiness Score (RRS)** — score compuesto: `DRE×0.40 + Test Efficiency×0.30 + Predictability×0.20 + (100 − % bloqueadas)×0.10`.

## Áreas funcionales

`Backend-Factory`, `Channels-App`, `Channel-Web`, `DS-Factory`, `Payments`, `Revenue`, `Seatmap`, `Servicing`, `Travel`.

## Sprints

El calendario de sprints (fechas de inicio/fin) está precargado en el código para todo 2026 (`2026.01A` a `2026.12B`). Para usar el dashboard en otro año o calendario, hay que actualizar los objetos `SPRINT_STARTS` / `SPRINT_ENDS` en `sdlc_dashboard.html`.

## Stack técnico

- HTML + JavaScript vanilla (sin frameworks, sin paso de build).
- [Chart.js](https://www.chartjs.org/) para gráficos (via CDN).
- [PapaParse](https://www.papaparse.com/) para parseo de CSV (via CDN).
- Persistencia de configuración y metadata de sesión en `localStorage` del navegador.

## Notas

- Los archivos CSV con datos reales (`.csv`, `.xlsx`, carpetas `2026*/`) están excluidos del repositorio vía `.gitignore` — no se versionan datos de producción.
- No hay dependencias que instalar ni comandos de build: es un único archivo HTML autocontenido.
