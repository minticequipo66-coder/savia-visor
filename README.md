<div align="center">

<img src="assets/preview.png" alt="GovCamp 2026 · Monitor Ambiental IA — Caquetá" width="100%"/>

# SAVIA VISOR

**Monitor Ambiental IA — GovCamp 2026**  
Early detection of deforestation and water quality degradation in Caquetá, Colombia.

[![GovCamp 2026](https://img.shields.io/badge/GovCamp-2026-4ade80?style=flat-square&labelColor=020b04)](https://datos.gov.co)
[![License](https://img.shields.io/github/license/minticequipo66-coder/savia-visor?style=flat-square&labelColor=020b04&color=4ade80)](LICENSE)
[![Leaflet](https://img.shields.io/badge/Leaflet-1.9.4-22d3ee?style=flat-square&labelColor=020b04)](https://leafletjs.com)

</div>

---

## Overview

**Savia Visor** is an interactive geospatial dashboard built for the *Datos al Ecosistema 2026* competition (Ministerio TIC · datos.gov.co). It combines two AI models with real hydrological infrastructure data to provide real-time environmental surveillance of the Caquetá department (88,965 km²).

| AI Model | Architecture | Purpose |
|---|---|---|
| **ResNet50 v2.1.4** | CNN with MapBiomas fine-tuning | Deforestation detection (satellite imagery · Google Earth Engine · Sentinel-2) |
| **LSTM-IF v1.8.2** | LSTM + Isolation Forest | Water quality index prediction and anomaly detection (ICA · IDEAM) |
| **HydroBASINS model** | Isolation Forest (44 catchments) | Cascade hydrological damage scoring (0–100 risk score) |

**Team:** Bryan · Carolina · Pavel · Brahim

---

## Features

- **Boot sequence** with animated initialization status for each AI subsystem
- **Interactive Leaflet map** centered on Caquetá with dark-mode tile layer
- **KPI panel** showing live deforestation area (ha), active water anomalies, and critical basins
- **Deforestation zones** — 4 mock alert levels (CRÍTICO / ALERTA / VIGILANCIA / NORMAL) rendered as colored polygons with CNN confidence scores
- **Water quality stations** — 6 ICA monitoring points with LSTM predictions, pH, turbidity, dissolved O₂, and Isolation Forest anomaly flags
- **Hydrological cascade layer** — 44 real HydroBASINS catchments colored by risk score, clipped to the departmental boundary
- **Water infrastructure layer** — 210 reservoirs and 461 distribution points from IDEAM (toggleable)
- **River network** — 924 river and stream segments from OpenStreetMap
- **Damage zones panel** — scrollable list of critical basins with cut-boundary toggle
- **Radar sweep animation** and coordinate badge updating on mouse move

---

## Data

All geodata lives in the [`data/`](data/) directory.

### `caqueta_boundary.geojson`
**Type:** Polygon · **Size:** ~46 KB  
Official departmental boundary from the *Marco Geoestadístico Nacional (MGN)* — DANE 2025 edition. Used as the clipping mask for all other layers.  
**CRS:** WGS 84 (EPSG:4326)  
**Bounding box:** 2.938°N · -0.706°S · -76.306°W · -71.254°E

---

### `cascada_hidrologica_caqueta.geojson`
**Type:** Polygons · **Features:** 44 catchments  
HydroBASINS Pfafstetter Level-7 catchments covering Caquetá with the cascade hydrological damage model applied. Each feature carries:

| Property | Description |
|---|---|
| `HYBAS_ID` | Unique HydroBASINS catchment identifier |
| `riesgo_cascada_score` | Risk score 0–100 (deforestation + slope + exposed infrastructure + upstream count) |
| `def_acumulada_reciente_ha` | Accumulated deforestation upstream (cascade effect on downstream infrastructure) |
| `def_local_reciente_ha` | Deforestation within the catchment boundary |
| `n_infraestructura_agua` | Water infrastructure elements (reservoirs + distribution points) exposed |
| `n_cuencas_aguas_arriba` | Number of upstream feeding catchments |
| `pendiente_clase` | `suave` (Amazonian plain) or `moderada` (Andean-Amazonian transition) |
| `anomalia_riesgo` | Isolation Forest flag — `true` = statistically anomalous risk behavior |

**Source:** Pavel Ramírez · `Data PV/08_modelo_hydrobasins_standalone.ipynb` · HydroSHEDS / HydroBASINS  
**Period:** 2019–2024 · **Anomalies detected:** 11 of 44 catchments  
**Highest risk:** catchment `6070193960` (score 87.3 · 281 infrastructure elements · 40,101 ha local deforestation)

---

### `cascada_hidrologica_caqueta_trimmed.geojson`
**Type:** Polygons · **Features:** 44 catchments  
Same as above, clipped to `caqueta_boundary.geojson` using DANE geometry. Used as the rendered layer in the dashboard to avoid visual overflow outside the department.

---

### `dano_local_caqueta.geojson`
**Type:** Polygons · **Features:** 44 catchments  
Same HydroBASINS base as the cascade layer but visualized by **local deforestation** (`def_local_reciente_ha`) — damage occurring directly inside each catchment boundary, not inherited from upstream.

---

### `dano_local_caqueta_trimmed.geojson`
**Type:** Polygons · **Features:** 44 catchments  
Clipped version of `dano_local_caqueta.geojson` to the departmental boundary. Used as the default active layer in the Damage Zones panel.

---

### `unified_deposito_agua_r.geojson`
**Type:** Polygons · **Features:** 210 reservoirs  
Real water reservoir infrastructure from IDEAM / datos.gov.co. Total footprint: 99,660 m².

| `DATipo` | Count | Description |
|---|---|---|
| 3 | 127 | Minor reservoirs / jagüeyes |
| 5 | 48 | Storage tanks |
| 8 | 17 | Intake structures |
| 1 | 12 | Dams / embalses |
| 6 | 6 | Other |

**Highest concentration:** municipality of La Montañita (~1.01°N, -75.35°W).

---

### `unified_punto_distribucion.geojson`
**Type:** Points · **Features:** 461 distribution points  
Water distribution network from IDEAM / datos.gov.co.

| `PDTipo` | Count | Description |
|---|---|---|
| 2 | 399 | Community distribution points |
| 3 | 58 | Institutional distribution points |
| 1 | 4 | Treatment plants |

**Highest concentration:** municipality of Cartagena del Chairá (~1.008°N, -75.21°W).

---

### `unified_rios_caqueta.geojson`
**Type:** LineStrings · **Features:** 924 segments  
River and stream network for Caquetá from OpenStreetMap. Includes 100 named water bodies (70 rivers, 30 streams/caños).

**Major rivers:** Río Caquetá (Q171840) · Río Apaporis (Q73940) · Río Caguán (Q3458553) · Río Ajajú (Q4699515)

---

## Fuentes de datos — datos.gov.co

All open data is published under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) by Colombian government institutions. Full dataset index with URLs: [assets/datos-gov-co-sources.md](assets/datos-gov-co-sources.md).

### Depósito Agua R — IGAC

Water reservoir infrastructure (polygons) published by the **Instituto Geográfico Agustín Codazzi (IGAC)**. For Caquetá, 28 municipal datasets were downloaded and unified into [`unified_deposito_agua_r.geojson`](data/unified_deposito_agua_r.geojson) (210 reservoirs, 99,660 m² total footprint).

- **Issued:** 2024-12-10 · **Last update:** 2025-12-17
- **Tags:** altimetría · cartografía · fotogrametría · planimetría · vectorial · caquetá
- **Contact:** contactenos@igac.gov.co

### Punto de Distribución — IGAC

Water distribution point network (points) from the same IGAC publication. For Caquetá, 27 municipal datasets were merged into [`unified_punto_distribucion.geojson`](data/unified_punto_distribucion.geojson) (461 points: 399 community, 58 institutional, 4 treatment plants).

- **Issued:** 2024-12-10 · **Last update:** 2025-12-17
- **Contact:** contactenos@igac.gov.co

### Data Histórica de Calidad de Agua — IDEAM

Physicochemical and microbiological monitoring data from Colombia's national water quality network, published by the **IDEAM** (Instituto de Hidrología, Meteorología y Estudios Ambientales).

- **Dataset:** [62gv-3857](https://www.datos.gov.co/Ambiente-y-Desarrollo-Sostenible/Data-Hist-rica-de-Calidad-de-Agua/62gv-3857/about_data)
- **Caquetá coverage:** 521 records from Río Orteguaza · Florencia
- **Parameters:** 48 variables — pH, dissolved O₂, turbidity, conductivity, DQO, COT, total/reactive phosphorus, nitrogen series, 20+ pesticides (organochlorine/organophosphate/herbicide), heavy metals (Al, Cd, Cu, Cr, Fe, Mn, Ni, Pb, Zn), sulfates, solids
- **Used for:** LSTM-IF v1.8.2 training and ICA station validation

---

## Tech Stack

| Layer | Technology |
|---|---|
| Map rendering | [Leaflet.js](https://leafletjs.com) 1.9.4 |
| Styling | Tailwind CSS (CDN) |
| Fonts | Bricolage Grotesque · DM Mono (Google Fonts) |
| Geodata | GeoJSON (WGS 84) |
| Hosting | GitHub Pages |
| AI/ML app | [Streamlit Cloud](https://mintic.streamlit.app) |
| Chatbot IA | [DeepSeek](https://platform.deepseek.com) `deepseek-chat` — OpenAI-compatible API |

The entire visualizer runs as a **single `index.html`** with no build step — open it directly in any browser or deploy to GitHub Pages.

---

## Running locally

```bash
git clone https://github.com/minticequipo66-coder/savia-visor.git
cd savia-visor
# Serve with any static server, e.g.:
npx serve .
# or
python3 -m http.server 8080
```

Then open `http://localhost:8080` in your browser.

> **Note:** The GeoJSON files are fetched via relative paths — a local HTTP server is required (direct `file://` opening will be blocked by CORS).

---

## Methodology

CRISP-ML(Q) process applied across six phases:

1. **Business Understanding** — Early deforestation and water degradation detection in Caquetá
2. **Data Understanding** — IDEAM ICA datasets, MapBiomas Colombia, Google Earth Engine, HydroBASINS
3. **Data Preparation** — Temporal lags, seasonal features, 30-day Sentinel-2 composites, Pfafstetter topology
4. **Modeling** — ResNet50 (deforestation) · LSTM + Isolation Forest (water quality) · HydroBASINS cascade model
5. **Evaluation** — IoU for CNN · MAE per station for LSTM · IF calibration per catchment
6. **Deployment** — GitHub Pages (visor) · Streamlit Cloud (AI app)

---

## License

[MIT](LICENSE) · GovCamp 2026 · Ministerio TIC Colombia
