# LogiAI : starter du TP (séance API)

Prédire le risque de retard d'une livraison et l'exposer par une API REST (FastAPI).

## Installation (une fois)

    python -m venv .venv
    .venv\Scripts\activate.bat        # Windows cmd  (PowerShell : .venv\Scripts\Activate.ps1)
    source .venv/bin/activate         # macOS / Linux
    pip install -r requirements.txt

## Données et modèle

    python -m logiai.dataset          # écrit data/livraisons.csv
    python -m logiai.train            # écrit models/model.joblib et models/model_info.json

## Lancer l'API

    python -m uvicorn logiai.api.main:app --reload
    # puis http://localhost:8000/docs

## Lancer l'interface Streamlit

    python -m streamlit run streamlit_app.py

## Tests

    python -m pytest

## Composants principaux

- `logiai/api/schemas.py` : validation des caractéristiques d'une livraison
- `logiai/api/main.py` : API de prédiction avec FastAPI
- `streamlit_app.py` : interface utilisateur Streamlit
