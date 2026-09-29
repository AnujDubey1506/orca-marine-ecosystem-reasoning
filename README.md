# ORCA Marine Intelligence Dashboard

ORCA is a React + FastAPI prototype for marine decision support. It combines simulated weather, ocean, geospatial, and risk signals through a LangGraph workflow and returns a structured recommendation for a selected marine location.

> **Prototype data notice:** ORCA currently uses deterministic simulated marine data. It is not a live navigation, weather, or safety system. Verify official forecasts, local advisories, and conditions before departure.

## Project structure

```text
.
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   └── main.py              # FastAPI API, Pydantic models, LangGraph flow
│   ├── .env.example             # Server-side environment variable template
│   └── requirements.txt         # Python production dependencies
├── src/
│   ├── App.tsx                  # React dashboard and analysis state machine
│   ├── index.css
│   └── main.tsx
├── Dockerfile                   # Production image: builds React, serves through FastAPI
├── main.py                       # Optional ASGI entrypoint (`main:app`)
├── .env.example                  # Local server configuration template
├── START_HERE.md                 # Windows setup and troubleshooting guide
├── package.json                 # Frontend scripts and dependencies
├── package-lock.json
├── pyproject.toml               # Python project metadata
├── uv.lock
├── vite.config.ts
└── .github/workflows/ci.yml
```

## Database status

There is no database in the current prototype. Requests are stateless and all marine values are generated deterministically from the date, location, and activity. No database credentials, migrations, ORM, or persistent storage are required for deployment.

If persistence is added later, keep it separate from the current analysis contract and add its credentials only through the deployment platform's secret manager.

## LangGraph workflow

The backend compiles this flow:

```text
START
  → Intent / Context
  → Planner
  → Data Discovery
  → Weather ───────────────┐
  → Ocean ─────────────────┤
  → Geospatial ────────────┤
  → Evidence Visualization ┘
  → Cross-source Reasoning
  → Risk / Decision Synthesis
  → Final Response
  → END
```

The specialized agents run as parallel branches. NVIDIA synthesis is optional. If NVIDIA is not configured, unavailable, invalid, or too slow, ORCA returns a deterministic fallback response instead of leaving the request loading.

## API

### `GET /api/health`

Returns API status and whether a server-side NVIDIA key is configured.

### `POST /api/query/analyze`

Example request:

```json
{
  "query": "Is tomorrow morning good for fishing near my location?",
  "location": {
    "latitude": 19.076,
    "longitude": 72.8777,
    "label": "Selected marine location",
    "source": "map"
  },
  "timezone": "Asia/Calcutta"
}
```

The response preserves the frontend contract and includes `context`, `plan`, `agent_results`, `decision`, `evidence`, `map_layers`, `data_mode`, `reasoning_mode`, and `generated_at`.

## Run locally

### Requirements

- Node.js 20 or newer
- Python 3.13 or newer
- `uv` recommended for Python dependency management

Install dependencies:

```bash
npm ci
uv sync
```

Start the backend:

```bash
uv run uvicorn backend.app.main:app --host 0.0.0.0 --port 8001
```

In a second terminal, start the frontend:

```bash
npm run dev -- --host 0.0.0.0
```

Open `http://localhost:8000`. Vite proxies `/api` requests to the FastAPI service on port `8001`.

For the complete Windows PowerShell walkthrough, see [START_HERE.md](START_HERE.md).

## Environment variables

Copy the root local template:

```bash
cp .env.example .env
```

Available variables:

| Variable | Required | Purpose |
| --- | --- | --- |
| `NVIDIA_API_KEY` | No | Enables optional NVIDIA decision synthesis |
| `NVIDIA_BASE_URL` | No | NVIDIA-compatible API base URL |
| `NVIDIA_MODEL` | No | NVIDIA model name |
| `FRONTEND_ORIGIN` | No | CORS origin for a separately hosted frontend |

The backend loads this root `.env` file with `python-dotenv`. Never commit `.env`, API keys, or other secrets.

## Demo query

Enter this query in the ORCA interface:

```text
Is tomorrow morning good for fishing near my location?
```

The expected prototype flow is Understanding → Planning → Weather → Ocean → Geospatial → Risk → Cross-source → Decision.

## Known limitations and future upgrade path

- Marine values are deterministic simulated data, not live forecasts or advisories.
- The current prototype is stateless and has no database.
- NVIDIA synthesis is optional; deterministic fallback reasoning is used when it is unavailable.
- The map is a simulated coastal operating view.
- A future production version can add authenticated users, persistent query history, live marine data providers, and a database without changing the current API response contract.

## Production deployment with Docker

Build the image:

```bash
docker build -t orca-marine-dashboard .
```

Run it:

```bash
docker run --rm \
  -p 8001:8001 \
  -e NVIDIA_API_KEY="$NVIDIA_API_KEY" \
  orca-marine-dashboard
```

The image builds the React frontend and serves the built dashboard from FastAPI at `/`. The API remains available under `/api/*`.

This Dockerfile can be built by a hosting provider connected to a GitHub repository. Configure the platform's public port as `8001` or provide a `PORT`-aware start command if the platform assigns a different port.

## Push this project to GitHub

Create an empty repository on GitHub, then run from this project directory:

```bash
git init
git add .
git commit -m "Initial ORCA prototype"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

The included CI workflow checks the TypeScript build and Python compilation on pushes and pull requests.
