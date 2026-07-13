# Fuentes de datos y arquitectura de servicios — Savia Visor

Datasets oficiales colombianos utilizados en Savia Visor. Todos son de acceso público bajo licencia [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/).

---

## Chatbot IA — DeepSeek

El visor incluye un asistente conversacional embebido en la esquina inferior derecha, especializado en el contexto ambiental de Caquetá.

### Infraestructura

| Componente | Detalle |
|---|---|
| **Proveedor** | [DeepSeek](https://platform.deepseek.com) |
| **Modelo** | `deepseek-chat` (DeepSeek-V3) |
| **Endpoint** | `https://api.deepseek.com/chat/completions` |
| **Autenticación** | API Key Bearer token (header `Authorization`) |
| **Protocolo** | OpenAI-compatible REST API · JSON · HTTPS |
| **Max tokens/respuesta** | 512 |
| **Temperature** | 0.6 |
| **Streaming** | No (response completa) |
| **Ubicación en código** | `index.html` — sección `CHATBOT WIDGET` al final del body |

### Límites del plan gratuito (DeepSeek)

| Recurso | Límite aprox. |
|---|---|
| Tokens de contexto | 64 000 tokens por llamada |
| Rate limit | 60 llamadas / minuto (tier free) |
| Cuota mensual gratuita | ~500 000 tokens de entrada/salida acumulados |
| Costo adicional (deepseek-chat) | $0.014 / 1M tokens input · $0.028 / 1M tokens output (si se supera la cuota) |

El widget muestra un contador de tokens acumulados por sesión en la barra inferior del panel.

### Contexto del sistema (system prompt)

El asistente tiene instrucciones sobre:
- Zonas de deforestación detectadas por CNN ResNet50 v2.1.4 (Sentinel-2 / GEE)
- Estaciones ICA y modelo LSTM + Isolation Forest de calidad hídrica
- 44 cuencas HydroBASINS con scores de riesgo en cascada
- Infraestructura hídrica real IDEAM (210 depósitos, 461 puntos de distribución)
- Red hídrica OSM (924 segmentos) y datos abiertos datos.gov.co

### Aviso de seguridad

> La API key está embebida en el cliente (`index.html`). Esto es aceptable para demos y hackathons. **No reutilizar esta key en producción.** Para producción, mover la key a un proxy backend o variable de entorno del servidor.

### Flujo de una llamada

```
Usuario escribe mensaje
  → chatSend() construye el payload
  → fetch POST https://api.deepseek.com/chat/completions
      headers: Authorization: Bearer <key>
      body: { model, messages: [system, ...history, user], max_tokens, temperature }
  → respuesta JSON: choices[0].message.content
  → se muestra en el panel y se suma al historial de conversación
  → usage.total_tokens acumula en el contador de tokens
```

---

## 1. Depósito Agua R

Infraestructura de depósitos y reservorios de agua del territorio colombiano, publicada por el Instituto Geográfico Agustín Codazzi (IGAC).

### Metadatos

| Campo | Valor |
|---|---|
| **Publisher** | Instituto Geográfico Agustín Codazzi (IGAC) |
| **Contact** | IGAC-Admin · contactenos@igac.gov.co |
| **Homepage** | [ArcGIS Open Data](https://datos-abiertos-igac-igac-oit.hub.arcgis.com/datasets/e451ce68e01349dd94e9750930f57d60_16) |
| **Unique Identifier** | [ArcGIS Item](https://www.arcgis.com/home/item.html?id=e451ce68e01349dd94e9750930f57d60&sublayer=16) |
| **License** | [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) |
| **Issued** | 2024-12-10 |
| **Last Update** | 2025-12-17 |
| **Theme** | Geospatial |
| **Public Access** | Public |
| **Geographic Coverage** | -75.7699, 1.0591 → -75.7664, 1.0634 |
| **Tags** | altimetría · caquetá · cartografía · fotogrametría · planimetría · restitución · toponimia · valparaíso · vectorial |

### Datasets para Caquetá

28 datasets municipales unificados en `unified_deposito_agua_r.geojson` (210 polígonos).

| # | URL |
|---|---|
| 1 | https://www.datos.gov.co/dataset/Deposito-Agua-R/4m5z-e2th/about_data |
| 2 | https://www.datos.gov.co/dataset/Deposito-Agua-R/vif5-xiam/about_data |
| 3 | https://www.datos.gov.co/dataset/Deposito-Agua-R/c5uu-bmbn/about_data |
| 4 | https://www.datos.gov.co/dataset/Deposito-Agua-R/b9fi-ir4u/about_data |
| 5 | https://www.datos.gov.co/dataset/Deposito-Agua-R/3afv-kfsd/about_data |
| 6 | https://www.datos.gov.co/dataset/Deposito-Agua-R/bcnz-7xhx/about_data |
| 7 | https://www.datos.gov.co/dataset/Deposito-Agua-R/ew7g-n8rs/about_data |
| 8 | https://www.datos.gov.co/dataset/Deposito-Agua-R/9ai5-i2un/about_data |
| 9 | https://www.datos.gov.co/dataset/Deposito-Agua-R/6v7f-bi5h/about_data |
| 10 | https://www.datos.gov.co/dataset/Deposito-Agua-R/t4f3-dcjz/about_data |
| 11 | https://www.datos.gov.co/dataset/Deposito-Agua-R/q7pc-djdg/about_data |
| 12 | https://www.datos.gov.co/dataset/Deposito-Agua-R/9rni-7p75/about_data |
| 13 | https://www.datos.gov.co/dataset/Deposito-Agua-R/9jva-d7p3/about_data |
| 14 | https://www.datos.gov.co/dataset/Deposito-Agua-R/tatp-8zwn/about_data |
| 15 | https://www.datos.gov.co/dataset/Deposito-Agua-R/tybw-9hm3/about_data |
| 16 | https://www.datos.gov.co/dataset/Deposito-Agua-R/rfm4-nssj/about_data |
| 17 | https://www.datos.gov.co/dataset/Deposito-Agua-R/2nvm-jura/about_data |
| 18 | https://www.datos.gov.co/dataset/Deposito-Agua-R/e9p9-q4yn/about_data |
| 19 | https://www.datos.gov.co/dataset/Deposito-Agua-R/7s6p-chy3/about_data |
| 20 | https://www.datos.gov.co/dataset/Deposito-Agua-R/a5cp-8grf/about_data |
| 21 | https://www.datos.gov.co/dataset/Deposito-Agua-R/hpau-krwb/about_data |
| 22 | https://www.datos.gov.co/dataset/Deposito-Agua-R/9kqy-vsut/about_data |
| 23 | https://www.datos.gov.co/dataset/Deposito-Agua-R/azk6-6wu7/about_data |
| 24 | https://www.datos.gov.co/dataset/Deposito-Agua-R/sfgr-43cz/about_data |
| 25 | https://www.datos.gov.co/dataset/Deposito-Agua-R/evz5-y9m2/about_data |
| 26 | https://www.datos.gov.co/dataset/Deposito-Agua-R/874h-md4s/about_data |
| 27 | https://www.datos.gov.co/dataset/Deposito-Agua-R/vaak-j6fw/about_data |
| 28 | https://www.datos.gov.co/dataset/Deposito-Agua-R/q4jx-fnbr/about_data |

---

## 2. Punto de Distribución

Red de puntos de distribución de agua (comunitarios, institucionales y plantas de tratamiento) publicada por el IGAC.

### Metadatos

| Campo | Valor |
|---|---|
| **Publisher** | Instituto Geográfico Agustín Codazzi (IGAC) |
| **Contact** | IGAC-Admin · contactenos@igac.gov.co |
| **Homepage** | [ArcGIS Open Data](https://datos-abiertos-igac-igac-oit.hub.arcgis.com/datasets/e451ce68e01349dd94e9750930f57d60_16) |
| **Unique Identifier** | [ArcGIS Item](https://www.arcgis.com/home/item.html?id=e451ce68e01349dd94e9750930f57d60&sublayer=16) |
| **License** | [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) |
| **Issued** | 2024-12-10 |
| **Last Update** | 2025-12-17 |
| **Theme** | Geospatial |
| **Public Access** | Public |
| **Geographic Coverage** | -75.7699, 1.0591 → -75.7664, 1.0634 |
| **Tags** | altimetría · caquetá · cartografía · fotogrametría · planimetría · restitución · toponimia · valparaíso · vectorial |

### Datasets para Caquetá

27 datasets municipales unificados en `unified_punto_distribucion.geojson` (461 puntos).

| # | URL |
|---|---|
| 1 | https://www.datos.gov.co/dataset/Punto-Distribucion/s4b6-93yz/about_data |
| 2 | https://www.datos.gov.co/dataset/Punto-Distribucion/bfkj-gb92/about_data |
| 3 | https://www.datos.gov.co/dataset/Punto-Distribucion/5xpu-pt8z/about_data |
| 4 | https://www.datos.gov.co/dataset/Punto-Distribucion/p9ih-pwwx/about_data |
| 5 | https://www.datos.gov.co/dataset/Punto-Distribucion/tv8e-6z3t/about_data |
| 6 | https://www.datos.gov.co/dataset/Punto-Distribucion/bdwx-wiqs/about_data |
| 7 | https://www.datos.gov.co/dataset/Punto-Distribucion/c2n3-fxu6/about_data |
| 8 | https://www.datos.gov.co/dataset/Punto-Distribucion/rsd8-kdee/about_data |
| 9 | https://www.datos.gov.co/dataset/Punto-Distribucion/sybs-teh3/about_data |
| 10 | https://www.datos.gov.co/dataset/Punto-Distribucion/bfit-xhn4/about_data |
| 11 | https://www.datos.gov.co/dataset/Punto-Distribucion/62n5-a4dq/about_data |
| 12 | https://www.datos.gov.co/dataset/Punto-Distribucion/bsek-tgna/about_data |
| 13 | https://www.datos.gov.co/dataset/Punto-Distribucion/7unh-4zcu/about_data |
| 14 | https://www.datos.gov.co/dataset/Punto-Distribucion/25yr-3ws4/about_data |
| 15 | https://www.datos.gov.co/dataset/Punto-Distribucion/r5ag-c2zi/about_data |
| 16 | https://www.datos.gov.co/dataset/Punto-Distribucion/5rtf-5bje/about_data |
| 17 | https://www.datos.gov.co/dataset/Punto-Distribucion/wvkc-vzbm/about_data |
| 18 | https://www.datos.gov.co/dataset/Punto-Distribucion/aztc-sjn6/about_data |
| 19 | https://www.datos.gov.co/dataset/Punto-Distribucion/u85r-fpa5/about_data |
| 20 | https://www.datos.gov.co/dataset/Punto-Distribucion/7puw-gd45/about_data |
| 21 | https://www.datos.gov.co/dataset/Punto-Distribucion/qwji-thg6/about_data |
| 22 | https://www.datos.gov.co/dataset/Punto-Distribucion/phmh-ie3e/about_data |
| 23 | https://www.datos.gov.co/dataset/Punto-Distribucion/ei96-5c77/about_data |
| 24 | https://www.datos.gov.co/dataset/Punto-Distribucion/pbi7-7ydf/about_data |
| 25 | https://www.datos.gov.co/dataset/Punto-Distribucion/g4xj-64ms/about_data |
| 26 | https://www.datos.gov.co/dataset/Punto-Distribucion/h28y-di3r/about_data |
| 27 | https://www.datos.gov.co/dataset/Punto-Distribucion/299r-k8ge/about_data |

---

## 3. Data Histórica de Calidad de Agua

Dataset del IDEAM con mediciones fisicoquímicas y microbiológicas de puntos de monitoreo en cuerpos de agua colombianos.

**Dataset:** https://www.datos.gov.co/Ambiente-y-Desarrollo-Sostenible/Data-Hist-rica-de-Calidad-de-Agua/62gv-3857/about_data  
**Publisher:** IDEAM (Instituto de Hidrología, Meteorología y Estudios Ambientales)  
**Category:** Ambiente y Desarrollo Sostenible

### Cobertura para Caquetá

**521 registros** del Río Orteguaza en el municipio de Florencia. Estos registros son la fuente primaria para el entrenamiento y validación del modelo **LSTM-IF v1.8.2** de calidad hídrica.

### Variables medidas

| Categoría | Variables |
|---|---|
| **Básicas** | pH · Oxígeno Disuelto (OD) · Temperatura · Turbidez · Conductividad Eléctrica |
| **Nutrientes** | Fósforo Total · Fósforo Reactivo Disuelto · Nitrógeno Amoniacal · Nitrógeno Kjeldahl Total · Nitrógeno Total · Nitrato · Nitrito |
| **Orgánica / DQO** | Demanda Química de Oxígeno (DQO) · Carbono Orgánico Total (COT) |
| **Sólidos** | Sólidos Suspendidos Totales · Sólidos Totales |
| **Metales** | Aluminio · Cadmio · Cobre · Cromo · Hierro · Manganeso · Níquel · Plomo · Zinc (todos potencialmente biodisponibles) |
| **Plaguicidas organoclorados** | Aldrin · Dieldrin · Endrin Cetona · α/β/δ/ɣ-HCH · Heptacloro · trans-Heptacloro-endo-Epóxido · α/β-Endosulfan · Endosulfan Sulfato · p,p'-DDD · p,p'-DDE · p,p'-DDT · Metoxicloro |
| **Plaguicidas organofosforados** | Clorpirifos · Malation · Metil Paration |
| **Herbicidas / fungicidas** | Atrazina · Clorotalonil · Propanil · Ametrina |
| **Otros** | Sulfato |

**Total variables:** 48 parámetros fisicoquímicos y de contaminantes

### Esquema de columnas

| Columna | Campo API | Tipo | Descripción |
|---|---|---|---|
| NOMBRE DEL PUNTO DE MONITOREO | `nombre_del_punto_de_monitoreo` | Texto | Identificador del punto. Red de referencia nacional con prefijo `RCA`. Ejemplo: `RCA_AMAZONAS_AMA_LETICIA_NAZARETH [48017030]` |
| LATITUD | `latitud` | Número | Latitud del punto de monitoreo |
| LONGITUD | `longitud` | Número | Longitud del punto de monitoreo |
| ELEVACIÓN (m.s.n.m.) | `elevaci_n_m_s_n_m` | Número | Altura sobre el nivel del mar |
| CORRIENTE | `corriente` | Texto | Nombre de la corriente hídrica superficial |
| ZONA HIDROGRÁFICA - ZH | `zona_hidrogr_fica_zh` | Texto | Zona hidrográfica de la estación |
| SZH - Código | `szh_c_digo_rea_zona_subzona` | Número | Código de la Subzona Hidrográfica (`#Área#Zona##Subzona`) |
| Nombre Subzona Hidrográfica | `nombre_subzona_hidrogr_fica` | Texto | Subzona hidrográfica del punto |
| DEPARTAMENTO | `departamento` | Texto | Departamento del punto de monitoreo |
| MUNICIPIO | `municipio` | Texto | Municipio del punto de monitoreo |
| FECHA | `fecha` | Timestamp | Fecha de recolección de la muestra |
| PROPIEDAD OBSERVADA | `propiedad_observada` | Texto | Variable fisicoquímica o microbiológica analizada |
| RESULTADO | `resultado` | Texto | Valor medido de la variable |
| UNIDAD DEL RESULTADO | `unidad_del_resultado` | Texto | Unidad de medida del resultado |
| PROYECTO | `proyecto` | Texto | Proyecto asociado al monitoreo |
