# EarthTrend AI — Member 4 (AI / ML Engineer)

This package implements the Member 4 tasks described in the EarthTrend AI
hackathon plan:

1. Forecasting with Prophet.
2. Backtesting on the last three years (36 months for monthly data), with MAE.
3. Anomaly detection with STL decomposition + Isolation Forest.
4. Plain-language explanations with Claude.
5. Optional "Ask the Data" question answering.
6. FastAPI endpoints for integration with the dashboard.

## 1. Requirements

- Python 3.10–3.12 recommended.
- Internet access for installing packages and calling the Claude API.
- An Anthropic API key only for live Claude explanations. Forecasting and
  anomaly detection do not require the Claude API.

## 2. Install

From the project root:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

Then:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 3. Configure Claude

Copy `.env.example` to `.env` and set:

```env
ANTHROPIC_API_KEY=your_key_here
ANTHROPIC_MODEL=your_available_model_id
```

Use a model identifier enabled for your Anthropic account. Never commit `.env`
to GitHub. If the key or model is not configured, the explanation endpoint
returns a deterministic fallback instead of calling Claude.

## 4. Prepare NASA data

The Member 2 handoff should be a CSV with one row per month and these columns:

```csv
date,value
2000-01-01,12.4
2000-02-01,13.1
```

- `date`: month/date.
- `value`: numeric environmental variable (temperature or precipitation).
- Keep the unit documented (for example, °C or mm).
- Use real NASA data for the final demo.
- `data/sample_data.csv` is illustrative test data, not real NASA data.

If Member 2's columns have different names, pass `date_col` and `value_col`
to the Python functions, or configure the API request accordingly.

## 5. Use the Python functions directly

```python
import pandas as pd
from ai.forecasting import forecast, backtest
from ai.anomalies import detectAnomalies
from ai.explanation import explain

df = pd.read_csv("data/nasa_data.csv")

future = forecast(df, periods=36, date_col="date", value_col="value")
evaluation = backtest(df, test_months=36)
anomalies = detectAnomalies(df)

# Member 3 should provide its computed statistics dictionary.
stats = {
    "region": "Fes-Meknes",
    "variable": "Temperature",
    "trend": "Increasing",
    "p_value": 0.03,
    "forecast_mae": evaluation["mae"],
    "anomaly_count": anomalies["anomaly_count"],
}
report = explain(stats, language="Arabic")

print(future)
print(evaluation["mae"])
print(anomalies["anomalies"])
print(report)
```

For a monthly series, `periods=36` means 36 future months (three years).
Forecast values and MAE use the same unit as the input data.

## 6. Start the API

Run from the project root:

```bash
uvicorn api.server:app --reload
```

Open:

- API health: http://127.0.0.1:8000/health
- Interactive API docs: http://127.0.0.1:8000/docs

### Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Check API status |
| POST | `/forecast` | Forecast the next months |
| POST | `/backtest` | Evaluate predictions using held-out months |
| POST | `/anomalies` | Detect unusual observations |
| POST | `/explain` | Explain Member 3's statistics dictionary |
| POST | `/ask` | Optional question-answering from computed results |

Example request body for `/forecast`:

```json
{
  "data": [
    {"date": "2020-01-01", "value": 12.1},
    {"date": "2020-02-01", "value": 13.0}
  ],
  "periods": 36
}
```

The example above shows the request shape only; actual execution requires
at least 24 valid monthly observations.

Example request body for `/explain`:

```json
{
  "language": "Arabic",
  "stats": {
    "region": "Fes",
    "variable": "Temperature",
    "trend": "Increasing",
    "p_value": 0.02,
    "forecast_mae": 0.8
  }
}
```

## 7. Run tests

```bash
pytest -q
```

## 8. Important scientific limitations

- A forecast is an extrapolation of historical patterns, not a full climate model.
- A forecast should be presented with its uncertainty interval and backtest error.
- Isolation Forest identifies statistical outliers; it does not prove that a
  heatwave or drought occurred.
- Remove trend and seasonality before anomaly detection; this implementation
  uses STL decomposition for that purpose.
- A p-value does not measure the practical size or importance of a trend.
- The LLM must explain computed results only. It must not invent causes.
- NASA POWER is a gridded product and may differ from local weather-station data.

## 9. Team integration contract

- Member 2 supplies a cleaned monthly DataFrame/CSV.
- Member 3 supplies a Python dictionary containing computed statistical results.
- Member 4 returns forecast, backtest, anomaly, and explanation outputs.
- Member 5 calls the API and displays the results in the dashboard.
- Agree as a team on column names, date frequency, units, and JSON field names
  before integration.
