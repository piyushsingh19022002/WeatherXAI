# WeatherXAI

**An explainable, multilingual conversational AI system for tropical cyclone early warning and general weather forecasting.**

WeatherXAI is a proposed weather-intelligence platform that combines satellite-based cyclone analysis, rapid-intensification (RI) risk estimation, explainable predictions, and grounded weather information in one conversational interface. Its primary research contribution is explainable, uncertainty-aware RI early warning for the North Indian Ocean. Routine weather forecasting is a supporting feature that makes the same interface useful for everyday queries.

> **Project status:** This repository currently contains the project scaffold and design documentation. Forecasting, cyclone inference, alerts, and conversational features are not implemented or validated yet. Do not use this project for operational or emergency decisions.

## Goals

- Detect and classify tropical cyclones from INSAT-3D/3DR imagery.
- Estimate the risk of rapid intensification within the next 24 hours using cyclone and environmental features.
- Explain model outputs with Grad-CAM imagery and SHAP feature attributions.
- Present model confidence and uncertainty in clear, multilingual text and voice.
- Answer general weather questions using retrieved forecast data rather than generated values.
- Support a future alert workflow for relevant coastal districts, subject to validation and approval by authoritative agencies.

## Architecture

```text
INSAT imagery + track/environment data
				  |
				  v
	 Cyclone classification (ResNet-50)
				  |
				  +----> Grad-CAM visual evidence
				  |
				  v
	   RI risk model (XGBoost, 24 h)
				  |
				  +----> SHAP feature attributions
				  |
				  v
	   Structured prediction/reason objects
				  |
				  +-------------------------------+
												  v
User text/voice -> Conversation + function calls -> Multilingual response
						 |                        (Bhashini, planned)
						 +-> cyclone model results
						 +-> Open-Meteo forecast
						 +-> future alert rules
```

The language model is intended to select tools and phrase or translate returned results. It must not invent forecast values, cyclone measurements, confidence scores, or alert status. Numerical answers should be traceable to a model or data provider, with source time and uncertainty included where available.

## Repository Structure

```text
WeatherXAI/
├── backend/
│   ├── app/
│   │   ├── api/routes/        # HTTP endpoints and request routing
│   │   ├── core/              # Configuration, logging, shared application setup
│   │   ├── schemas/            # Validated API and model response types
│   │   └── services/           # Forecast, cyclone, language, and alert integrations
│   └── tests/                  # Backend unit tests
├── frontend/                  # React client
│   ├── public/                 # Static assets
│   └── src/
│       ├── components/         # Shared interface components
│       ├── features/
│       │   ├── chat/            # Text and voice conversation
│       │   ├── cyclone/         # Cyclone status, risk, and explanations
│       │   └── weather/         # General forecast views
│       ├── pages/               # Top-level screens
│       ├── services/            # Backend/API clients
│       └── types/               # Shared frontend types
├── data/
│   ├── raw/                     # Original, immutable source data (not committed)
│   ├── interim/                 # Staged or partially processed data (not committed)
│   ├── processed/               # Curated training/evaluation datasets (not committed)
│   └── metadata/                # Dataset manifests, provenance, and schemas
├── ml/
│   ├── notebooks/                # Exploration and experiments
│   ├── src/
│   │   ├── classification/       # Cyclone detection/category model
│   │   ├── ri/                   # 24-hour RI risk model
│   │   └── explainability/       # Grad-CAM and SHAP reason objects
│   └── tests/                    # Model and data-pipeline tests
├── models/
│   ├── checkpoints/              # Training checkpoints (not committed)
│   └── artifacts/                # Versioned model metadata/approved artifacts
├── docs/
│   ├── api/                      # API and tool contracts
│   ├── architecture/             # System design and decisions
│   └── data/                     # Data sources, licensing, and preparation
├── infra/docker/                 # Container and local deployment files
├── scripts/                      # Data, training, evaluation, and developer scripts
└── tests/
	├── evaluation/               # Model quality and calibration evaluation
	└── integration/              # Cross-service workflow tests
```

## Planned Components

| Component | Initial direction | Responsibility |
|---|---|---|
| Cyclone perception | PyTorch, ResNet-50 | Detect/classify cyclone structure from satellite imagery; expose Grad-CAM evidence |
| RI risk | XGBoost, SHAP | Estimate 24-hour RI risk from environmental and cyclone-derived features |
| Weather retrieval | Open-Meteo API and geocoding | Retrieve current and short-term forecast values for a location |
| Backend | FastAPI, PostgreSQL/PostGIS | Validate requests, orchestrate model/API calls, and return typed results |
| Conversational interface | LLM function calling | Route user intent to approved tools and phrase grounded results |
| Language and voice | Bhashini (planned) | Translation, speech recognition, and speech synthesis for Indian languages |
| Frontend | React, Leaflet or Mapbox | Map, forecast, cyclone explanation, and chat experience |
| Alerting | Threshold rules; delivery provider TBD | Future alert dispatch after operational validation and authorization |

These are proposed choices, not locked dependencies. Integrations may be replaced without changing the overall architecture.

## Tool Contract Sketch

The backend should expose narrow, typed operations to the conversational layer. The exact schema can evolve as data sources and models are finalized.

| Tool | Purpose |
|---|---|
| `classify_cyclone(image_id)` | Return cyclone detection/category, source timestamp, and model metadata |
| `get_ri_risk(cyclone_id)` | Return 24-hour RI risk, calibrated uncertainty, and SHAP reason object |
| `get_track_forecast(cyclone_id)` | Return track/intensity forecast with valid times and provenance |
| `get_current_forecast(location)` | Return retrieved temperature, precipitation probability, wind, and short-term forecast |

Every response should carry provenance such as source, issue/valid time, units, and model version. Missing or stale data should be reported as unavailable rather than filled in by the language model.

## Data and Safety Notes

- Candidate sources include MOSDAC INSAT-3D/3DR imagery, IMD Best Track, IBTrACS, and a carefully selected ERA5 subset. Confirm access terms, licensing, temporal alignment, and redistribution limits before use.
- Keep downloaded datasets and large weights out of Git. Store source manifests and checksums in `data/metadata/` instead.
- Separate training, validation, and test periods to reduce temporal leakage. Evaluate performance across storms, basins, and intensity ranges, not only random image splits.
- RI scores are research outputs until calibrated and independently validated. They are not official warnings.
- Any alerting workflow requires domain-expert review, robust delivery monitoring, and authorization from relevant authorities before deployment.
- Never commit API keys or personal recipient information. Use environment variables or a secrets manager for local and deployed integrations.

## Development Status

The repository is currently a structure and documentation baseline. Dependency manifests, API endpoints, model training code, UI implementation, and deployment configuration will be added in follow-up work. Once those exist, their exact setup and test commands will be documented here.