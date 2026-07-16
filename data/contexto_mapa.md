# Contexto del Monitor Ambiental IA — GovCamp 2026
## Caquetá, Colombia · Datos del Visor Savia

---

## 1. Proyecto

**Nombre:** Monitor Ambiental IA — GovCamp 2026  
**Concurso:** Datos al Ecosistema 2026 · Ministerio TIC de Colombia (datos.gov.co)  
**Objetivo:** Detección temprana de deforestación y degradación de calidad hídrica en tiempo real en el departamento de Caquetá.  
**URL pública:** https://mintic.streamlit.app  
**Equipo:** Bryan (CNN/deforestación), Pavel (LSTM/calidad hídrica), Brian (dashboard), Carolina (datos y documentación CRISP-ML)

---

## 2. Área de estudio — Departamento de Caquetá

**País:** Colombia  
**Departamento:** Caquetá (código DANE: 18)  
**Capital:** Florencia  
**Extensión aproximada:** 88.965 km²  
**Coordenadas del bounding box:**
- Norte: 2.938°N
- Sur: -0.706°N
- Oeste: -76.306°O
- Este: -71.254°O

**Geometría oficial:** Marco Geoestadístico Nacional (MGN) versión 2025, DANE. Archivo `caqueta_boundary.geojson` (46 KB).

---

## 3. Módulo de Monitoreo IA (datos de simulación)

> Nota: los datos de esta sección son mock/simulación mientras se integran los modelos reales. Las coordenadas y valores son aproximaciones para demostración.

### 3.1 Zonas de deforestación detectadas (CNN ResNet50 v2.1.4)

| Zona | Nivel | Hectáreas | Confianza CNN | Detectado | Coords aprox. |
|---|---|---|---|---|---|
| Sector Norte Caquetá | CRÍTICO | 8.450 ha | 94.2% | 2026-05-26 14:23 | (2.30, -75.20) a (1.80, -74.30) |
| Cuenca Río Caguán | ALERTA | 3.200 ha | 87.5% | 2026-05-26 13:55 | (1.20, -75.40) a (0.60, -74.50) |
| Corredor Andino-Amazónico | VIGILANCIA | 1.100 ha | 79.1% | 2026-05-25 09:10 | (0.30, -76.20) a (-0.30, -75.40) |
| Piedemonte Sur | NORMAL | 450 ha | 91.3% | 2026-05-24 07:30 | (-0.50, -75.90) a (-1.10, -75.10) |

**Total hectáreas afectadas (mock):** 13.200 ha  
**Escala de alertas:** CRÍTICO (rojo) → ALERTA (ámbar) → VIGILANCIA (amarillo) → NORMAL (verde)

### 3.2 Estaciones de calidad hídrica (LSTM-IF v1.8.2)

El Índice de Calidad del Agua (ICA) va de 0 a 100: MALO < 40 · REGULAR 40-60 · ACEPTABLE 60-75 · BUENO ≥ 75.

| Estación | Lat | Lng | ICA | Calidad | pH | Turbidez (NTU) | O₂ (mg/L) | Anomalía IF |
|---|---|---|---|---|---|---|---|---|
| Estación Florencia | 1.6144 | -75.6062 | 42 | REGULAR | 5.2 | 48.3 | 4.1 | ⚠ SÍ |
| Estación Río Orteguaza | 1.10 | -75.82 | 67 | ACEPTABLE | 6.8 | 22.1 | 6.2 | NO |
| Estación San Vicente | 2.1158 | -74.7673 | 55 | REGULAR | 6.4 | 31.7 | 5.5 | NO |
| Estación Puerto Rico | 1.97 | -75.17 | 78 | BUENO | 7.1 | 12.4 | 7.8 | NO |
| Estación La Montañita | 1.45 | -75.43 | 31 | MALO | 4.9 | 67.2 | 3.2 | ⚠ SÍ |
| Estación Milán | 1.02 | -75.47 | 62 | ACEPTABLE | 6.6 | 28.9 | 5.9 | NO |

**Estaciones críticas (ICA < 40):** 2 de 6 (Florencia y La Montañita)  
**Anomalías Isolation Forest activas:** 2

### 3.3 Alertas recientes (simulación)

| Hora | Zona | Tipo | Nivel |
|---|---|---|---|
| 14:23 | Sector Norte Caquetá | DEFOREST. | CRÍTICO |
| 14:20 | La Montañita (ICA: 31) | HÍDRICO | CRÍTICO |
| 14:18 | Florencia (ICA: 42) | HÍDRICO | MALO |
| 13:55 | Cuenca Río Caguán | DEFOREST. | ALERTA |
| 13:40 | San Vicente (ICA: 55) | HÍDRICO | REGULAR |
| 12:10 | Corredor Andino | DEFOREST. | VIGILANCIA |
| 11:45 | Río Orteguaza (ICA: 67) | HÍDRICO | REGULAR |

---

## 4. Cuencas Hidrológicas — Modelo de Daño Real (HydroBASINS)

**Fuente:** `Data PV/08_modelo_hydrobasins_standalone.ipynb` (Pavel Ramírez)  
**Base geográfica:** HydroBASINS — cuencas hidrográficas globales (Pfafstetter Level 7)  
**Total cuencas en Caquetá:** 44  
**Período de análisis:** 2019–2024  
**Anomalías detectadas por Isolation Forest:** 11 de 44 cuencas

### 4.1 Definición de métricas

- **def_acumulada_reciente_ha:** hectáreas de deforestación acumulada aguas arriba de la cuenca (efecto en cascada que impacta infraestructura hídrica aguas abajo).
- **def_local_reciente_ha:** hectáreas de deforestación ocurrida directamente dentro de los límites de la cuenca.
- **riesgo_cascada_score:** score de riesgo de 0 a 100 que combina deforestación, pendiente, infraestructura expuesta y número de cuencas alimentadoras aguas arriba.
- **n_infraestructura_agua:** número de elementos de infraestructura hídrica (depósitos + puntos de distribución) expuestos dentro de la cuenca.
- **n_cuencas_aguas_arriba:** cuántas cuencas drenan hacia esta cuenca (indica qué tan acumulativo es el daño).
- **pendiente_clase:** `suave` = llanura amazónica; `moderada` = transición andino-amazónica.
- **anomalia_riesgo:** flag booleano del modelo Isolation Forest. `true` = cuenca con comportamiento anómalo de riesgo.

### 4.2 Tabla completa de cuencas

| HYBAS_ID | Score | Cascada (ha) | Local (ha) | Infra expuesta | Cuencas arriba | Pendiente | Anomalía IF |
|---|---|---|---|---|---|---|---|
| 6070145730 | 37.8 | 9.461 | 2.852 | 0 | 2 | moderada | NO |
| 6070160600 | 17.5 | 427 | 427 | 0 | 0 | moderada | **SÍ** |
| 6070160610 | 0.0 | 28 | 28 | 0 | 0 | moderada | **SÍ** |
| 6070162000 | 45.3 | 29.284 | 29.284 | 0 | 0 | suave | NO |
| 6070162920 | 35.5 | 6.608 | 1.310 | 0 | 1 | moderada | NO |
| 6070165640 | 28.2 | 2.177 | 2.177 | 0 | 0 | moderada | NO |
| 6070165650 | 22.6 | 926 | 471 | 0 | 2 | moderada | NO |
| 6070167870 | 43.2 | 21.530 | 21.530 | 0 | 0 | suave | NO |
| 6070168560 | 34.0 | 5.298 | 5.298 | 0 | 0 | moderada | NO |
| 6070181980 | 59.3 | 9.257 | 9.257 | 20 | 0 | suave | **SÍ** |
| 6070181990 | 38.1 | 9.905 | 6.803 | 0 | 4 | suave | NO |
| 6070185420 | 45.4 | 29.883 | 29.883 | 0 | 0 | suave | NO |
| 6070185570 | 24.2 | 1.183 | 1.183 | 0 | 0 | suave | NO |
| 6070187060 | 43.0 | 20.753 | 20.753 | 0 | 0 | suave | NO |
| 6070187070 | 42.8 | 20.281 | 20.281 | 0 | 0 | suave | NO |
| 6070193960 | **87.3** | 40.101 | 40.101 | **281** | 0 | suave | **SÍ** |
| 6070193970 | **84.0** | 30.053 | 30.053 | **230** | 0 | suave | **SÍ** |
| 6070196300 | 40.7 | 14.558 | 14.558 | 0 | 0 | suave | NO |
| 6070196530 | 50.9 | 69.281 | 28.246 | 0 | 2 | suave | NO |
| 6070197170 | **75.3** | 48.868 | 29.705 | 42 | 6 | suave | **SÍ** |
| 6070197290 | **76.1** | 27.785 | 27.785 | 80 | 0 | suave | **SÍ** |
| 6070198740 | 51.4 | 75.034 | 4.879 | 0 | 2 | suave | NO |
| 6070201220 | **72.7** | 80.172 | 5.138 | 18 | 3 | suave | **SÍ** |
| 6070201330 | 28.2 | 2.164 | 2.164 | 0 | 0 | suave | NO |
| 6070207290 | 52.2 | 85.066 | 1.227 | 0 | 4 | suave | NO |
| 6070207540 | 19.0 | 532 | 532 | 0 | 0 | suave | NO |
| 6070208480 | 46.0 | 32.654 | 1.587 | 0 | 2 | suave | NO |
| 6070210270 | 27.2 | 1.863 | 1.863 | 0 | 0 | suave | NO |
| 6070210560 | 52.6 | 89.579 | 7.243 | 0 | 5 | suave | NO |
| 6070210920 | 53.0 | 95.673 | 19.021 | 0 | 8 | suave | NO |
| 6070211140 | 52.7 | 91.460 | 18 | 0 | 7 | suave | NO |
| 6070215410 | 52.3 | 85.788 | 190 | 0 | 6 | suave | NO |
| 6070215420 | 19.2 | 547 | 547 | 0 | 0 | suave | NO |
| 6070216330 | 57.4 | 187.748 | 615 | 0 | 17 | suave | NO |
| 6070216440 | 21.7 | 800 | 800 | 0 | 0 | suave | NO |
| 6070219140 | 4.7 | 59 | 59 | 0 | 0 | suave | **SÍ** |
| 6070219300 | 57.5 | 190.209 | 1.661 | 0 | 19 | suave | NO |
| 6070219380 | 4.4 | 57 | 57 | 0 | 0 | suave | **SÍ** |
| 6070219560 | 57.5 | 190.501 | 20 | 0 | 22 | suave | NO |
| 6070220350 | 57.5 | 190.481 | 213 | 0 | 21 | suave | NO |
| 6070220440 | 52.3 | 86.374 | 39 | 0 | 8 | suave | NO |
| 6070220500 | 57.5 | 190.983 | 426 | 0 | 24 | suave | NO |
| 6070227530 | 60.0 | **277.636** | 279 | 0 | 34 | suave | **SÍ** |
| 6070234710 | 24.6 | 1.246 | 1.246 | 0 | 0 | suave | NO |

### 4.3 Resumen estadístico de cuencas

| Métrica | Cascada acumulada | Daño local |
|---|---|---|
| Total (44 cuencas) | 2.354.271 ha | 371.810 ha |
| Máximo | 277.636 ha (cuenca 6070227530) | 40.101 ha (cuenca 6070193960) |
| Mínimo | 28 ha | 18 ha |
| Mediana | 24.657 ha | 1.762 ha |
| Cuencas con score > 70 | 5 cuencas | — |
| Cuencas con infra expuesta | 6 cuencas | — |
| Total infra expuesta | 671 elementos | — |

### 4.4 Cuencas críticas (score > 50 o anomalía)

Las 11 cuencas marcadas por Isolation Forest como anómalas son:
`6070160600`, `6070160610`, `6070181980`, `6070193960`, `6070193970`, `6070197170`, `6070197290`, `6070201220`, `6070219140`, `6070219380`, `6070227530`

Las cuencas de mayor riesgo con infraestructura expuesta:
- **6070193960**: score 87.3, 281 elementos de infraestructura, 40.101 ha locales — **ALERTA MÁXIMA**
- **6070193970**: score 84.0, 230 elementos, 30.053 ha locales — **ALERTA MÁXIMA**
- **6070197290**: score 76.1, 80 elementos, 27.785 ha locales
- **6070197170**: score 75.3, 42 elementos, 29.705 ha locales, 6 cuencas aguas arriba

---

## 5. Infraestructura Hídrica Real (IDEAM · datos.gov.co)

### 5.1 Depósitos de agua

**Total:** 210 polígonos  
**Fuente:** IDEAM / datos.gov.co  
**Área total:** 99.660 m²  
**Área mínima:** 2.3 m² · **Área máxima:** 14.057 m²

**Clasificación por tipo (DATipo):**

| Tipo | Cantidad | Descripción aprox. |
|---|---|---|
| Tipo 3 | 127 | Reservorios menores / jagüeyes |
| Tipo 5 | 48 | Tanques de almacenamiento |
| Tipo 8 | 17 | Estructuras de captación |
| Tipo 1 | 12 | Embalses / represas |
| Tipo 6 | 6 | Otros |

**Zona de mayor concentración:** municipio de La Montañita (coordenadas ~1.01°N, -75.35°O) con al menos 9 depósitos contiguos.

**Identificadores de muestra:**
- 1825600240020001 — Tipo 3 — 8.7 m² — (1.3481, -75.1102)
- 1846000540020001 — Tipo 1 — 667.3 m² — (1.0133, -75.3564) — mayor de la zona
- 1846000540020002 — Tipo 8 — 102.7 m² — (1.0138, -75.3553)

### 5.2 Puntos de distribución de agua

**Total:** 461 puntos  
**Fuente:** IDEAM / datos.gov.co

**Clasificación por tipo (PDTipo):**

| Tipo | Cantidad | Descripción aprox. |
|---|---|---|
| Tipo 2 | 399 | Puntos de distribución comunitaria |
| Tipo 3 | 58 | Puntos de distribución institucional |
| Tipo 1 | 4 | Plantas de tratamiento |

**Zona de mayor concentración:** municipio de Cartagena del Chairá (~1.008°N, -75.21°O) con cluster de más de 10 puntos contiguos.

**Identificadores de muestra (con coordenadas):**
- 18756004800201 — Tipo 2 — (1.0084, -75.2104)
- 18756004800202 — Tipo 2 — (1.0084, -75.2108)
- 18756004800210 — Tipo 2 — (1.0088, -75.2128)

---

## 6. Red Hídrica — Ríos y Cuerpos de Agua

**Total de segmentos:** 924  
**Fuente:** OpenStreetMap (extraído para Caquetá)

**Por tipo de vía hídrica:**

| Tipo | Segmentos |
|---|---|
| stream (quebrada/caño) | 479 |
| river (río) | 445 |

**Total con nombre:** 100 cuerpos de agua únicos (70 ríos, 30 quebradas/arroyos)

### 6.1 Ríos principales (con referencia Wikidata)

| Nombre | Tipo | Wikidata |
|---|---|---|
| Río Caquetá | river | Q171840 |
| Río Apaporis | river | Q73940 |
| Río Caguán | stream/river | Q3458553 |
| Río Ajajú | river | Q4699515 |

### 6.2 Ríos del departamento (listado completo)

Río Aguacaliente, Río Aguas Claras, Río Ajajú, Río Amú, Río Anaya, Río Apaporis, Río Bodoquero, Rio Bodoquerito, Río Caguán, Río Camuya, Río Caquetá, Río Caraño, Río Cuemaní, Río Cuñaré, Río Fragua, Río Fragua Grande, Río Fraguita, Río Guayas, Río Hacha, Río Honda, Río La Chocho, Río La Granada, Río La Perdiz, Río Luisa, Río Mesay, Río Negro, Río Nemal, Río Orteguaza, Río Pato, Río Peneya, Río Pescado, Río Riecito, Río Sabaleta, Río San Jorge, Río San José, Río San Juan, Río San Luis, Río San Pedro, Río Sarabando, Río Sencillo, Río Tajisa, Río Tauraré, Río Ventura, Río Yapella, Río Yaruje, Río Yarí, Río Yavilla, Río Yaya, Caño Buenos Aires, Caño Limón, Caño Majiña, Caño Negro, Caño Paujil, Caño Sararamano, Caño Tirimoní, Caño Vaupes, Caño Yanacurú, Fragua, La Perdiz

### 6.3 Quebradas y caños (listado)

Caño Aguazul, Caño Churuco, Caño Huicoco, Caño Mochilero, Caño Puño, Caño Rodolfo, Quebrada Aguanegra, Quebrada Aguas Blancas, Quebrada El Agulia, Quebrada El Berlin, Quebrada El Salado, Quebrada Encañonado, Quebrada Fragua, Quebrada Fragua Segundo, Quebrada Fragua Tercero, Quebrada Fraguita, Quebrada Garrapata, Quebrada La Castañal, Quebrada La Ceiba, Quebrada La Chocho, Quebrada La Danta, Quebrada La Esmeralda, Quebrada La Guaduala, Quebrada La Ruidosa, Quebrada La Solita, Quebrada La Yuca, Quebrada Las Doradas, Quebrada Masaya, Quebrada Méndez, Quebrada Morrocoyal, Quebrada Nutria, Quebrada San Jorge, Quebrada Santa Elena, Quebrada Sucre, Quebrada Suspisacha, Quebrada Tarqui, Quebrada del Tigre, Quebrada el Dedo, Quebrada el El Paraiso, Quebrada la Revolcosa, Quebrada la Sardina

---

## 7. Capas del mapa y su origen

| Capa | Tipo | Archivo | Fuente |
|---|---|---|---|
| Límite departamental Caquetá | Polígono | caqueta_boundary.geojson | MGN DANE 2025 |
| Zonas de deforestación | Polígonos mock | hardcoded en index.html | Simulación para demo |
| Estaciones ICA | Puntos mock | hardcoded en index.html | Simulación IDEAM |
| Cascada hidrológica (44 cuencas) | Polígonos reales | cascada_hidrologica_caqueta.geojson | Modelo Pavel / HydroBASINS |
| Cascada recortada al dpto. | Polígonos reales | cascada_hidrologica_caqueta_trimmed.geojson | Ídem + clip DANE |
| Daño local (44 cuencas) | Polígonos reales | dano_local_caqueta.geojson | Modelo Pavel / HydroBASINS |
| Daño local recortado | Polígonos reales | dano_local_caqueta_trimmed.geojson | Ídem + clip DANE |
| Depósitos de agua | Polígonos reales | unified_deposito_agua_r.geojson | IDEAM / datos.gov.co |
| Puntos de distribución | Puntos reales | unified_punto_distribucion.geojson | IDEAM / datos.gov.co |
| Ríos y quebradas | Líneas reales | unified_rios_caqueta.geojson | OpenStreetMap |

---

## 8. Modelos de IA

### 8.1 Modelo de deforestación (Bryan)

- **Arquitectura:** ResNet50 con transfer learning
- **Fine-tuning:** MapBiomas Colombia
- **Imágenes:** Google Earth Engine — composites medianas de 30 días Sentinel-2
- **Métricas:** IoU (Intersection over Union) sobre conjunto de validación
- **Validación histórica:** evento de deforestación Caquetá 2021
- **Salida:** polígono georreferenciado + nivel de alerta + confianza (%)
- **Versión en producción:** ResNet50 v2.1.4

### 8.2 Modelo de calidad hídrica (Pavel)

- **Arquitectura:** LSTM (TensorFlow/Keras) + Isolation Forest
- **Datos:** ICA (Índice de Calidad del Agua) IDEAM — descargado desde datos.gov.co
- **Criterio de estaciones:** cobertura continua > 3 años
- **Features:** lags temporales, variables estacionales, turbidez, pH, O₂ disuelto
- **Métricas:** MAE por estación, banda de confianza al 95%
- **Salida:** ICA predicho por estación + flag de anomalía Isolation Forest
- **Versión en producción:** LSTM-IF v1.8.2

### 8.3 Modelo de cuencas / cascada hidrológica (Pavel)

- **Base geográfica:** HydroBASINS Pfafstetter Level 7
- **Variable objetivo:** daño topológico acumulado aguas arriba
- **Detector de anomalías:** Isolation Forest (calibrado por cuenca)
- **Score:** 0–100 (combinación de hectáreas deforestadas, pendiente, infraestructura expuesta, cuencas alimentadoras)
- **Notebook fuente:** `Data PV/08_modelo_hydrobasins_standalone.ipynb`

---

## 9. Metodología CRISP-ML(Q)

1. **Business Understanding:** detección temprana de deforestación y degradación hídrica en Caquetá
2. **Data Understanding:** datasets IDEAM (ICA), MapBiomas Colombia, Google Earth Engine
3. **Data Preparation:** lags temporales, variables estacionales, composites satelitales 30 días
4. **Modeling:** ResNet50 (deforestación) + LSTM + Isolation Forest (calidad hídrica)
5. **Evaluation:** IoU para CNN, MAE por estación para LSTM, validación evento Caquetá 2021
6. **Deployment:** GitHub Pages (visor) + Streamlit Cloud (app)

---

## 10. Criterios del concurso GovCamp 2026

| Criterio | Puntos | Cómo se aborda |
|---|---|---|
| Uso de Datos | 20 | Dataset ICA IDEAM (datos.gov.co) + MGN DANE + OSM |
| Uso de IA | 20 | ResNet50 + LSTM + Isolation Forest |
| Impacto | 20 | Detección temprana deforestación Caquetá |
| Innovación | 15 | Fusión imágenes satelitales + hidroquímica + cuencas |
| Rigor técnico | 15 | CRISP-ML, IoU, MAE, bandas 95%, HydroBASINS |
| Diseño/Usabilidad | 10 | Dashboard interactivo + GitHub Pages |

---

## 11. Glosario técnico

- **ICA:** Índice de Calidad del Agua (0–100). Integra pH, turbidez, oxígeno disuelto y otros parámetros.
- **HYBAS_ID:** identificador único de cuenca en el sistema HydroBASINS de HydroSHEDS.
- **Isolation Forest:** algoritmo de ML para detección de anomalías sin supervisión. Detecta cuencas o estaciones con comportamiento estadísticamente atípico.
- **LSTM:** Long Short-Term Memory — red neuronal recurrente para series temporales.
- **IoU:** Intersection over Union — métrica de evaluación para modelos de segmentación de imágenes.
- **CNN:** Convolutional Neural Network — red neuronal convolucional para análisis de imágenes satelitales.
- **HydroBASINS:** producto de HydroSHEDS que divide el mundo en cuencas hidrográficas jerarquizadas (Pfafstetter).
- **MGN:** Marco Geoestadístico Nacional — geometría oficial de Colombia del DANE.
- **Sentinel-2:** satélite de la ESA con resolución 10 m/píxel, revisita 5 días. Fuente de imágenes para el módulo de deforestación.
- **MapBiomas Colombia:** proyecto de mapeo de cobertura boscosa anual usando ML. Fuente de etiquetas para fine-tuning del ResNet50.
- **pendiente_suave:** terreno de llanura amazónica (< 5% pendiente), mayor vulnerabilidad a deforestación extensiva.
- **pendiente_moderada:** zona de transición andino-amazónica (5–15%), mayor riesgo de erosión post-deforestación.
