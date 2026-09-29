# ORCA local setup on Windows

This guide runs the current ORCA prototype locally on Windows. The application has two processes:

- FastAPI backend: `http://localhost:8001`
- React/Vite frontend: `http://localhost:8000`

Do not open `index.html` directly using `file://`. The frontend must be started using the project's development server.

## A. Install prerequisites

Install these before starting:

1. Python 3.13 or newer from [python.org](https://www.python.org/downloads/windows/). During installation, enable **Add Python to PATH**.
2. Node.js 20 or newer from [nodejs.org](https://nodejs.org/).
3. Git is optional, but useful if you cloned the project from GitHub.

Check the installations in PowerShell:

```powershell
python --version
node --version
npm --version
```

If `python` is not recognized, try `py --version` and use `py -3.13` in the commands below.

## B–C. Open PowerShell and enter the project folder

Open the extracted project folder in File Explorer, right-click an empty area, and select **Open in Terminal**.

Or navigate manually:

```powershell
cd "C:\Users\YOUR_NAME\Downloads\ORCA-Marine-Dashboard-Local"
```

Confirm that the folder contains `package.json`, `backend`, `src`, and `START_HERE.md`:

```powershell
Get-ChildItem
```

## D–F. Create a Python environment and install backend dependencies

Create the virtual environment:

```powershell
python -m venv .venv
```

If PowerShell blocks activation, allow it for this terminal only:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Activate the environment:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install the backend dependencies:

```powershell
python -m pip install --upgrade pip
python -m pip install -r backend\requirements.txt
```

## G–I. Install frontend dependencies and configure the environment

Install the Node dependencies:

```powershell
npm install
```

Create the local environment file:

```powershell
Copy-Item .env.example .env
```

Open it:

```powershell
notepad .env
```

`NVIDIA_API_KEY` is optional. If it is empty, ORCA still works using the existing deterministic prototype fallback. If you have an NVIDIA key, put it only in `.env`; never put it in frontend code or commit it to GitHub.

## J. Start the FastAPI backend

Keep the first PowerShell window open and run:

```powershell
.\.venv\Scripts\python.exe -m uvicorn backend.app.main:app --host 0.0.0.0 --port 8001
```

Verify it in another browser tab:

```text
http://localhost:8001/api/health
```

You should see a JSON response with `"status": "ok"`.

## K–L. Start the frontend and open ORCA

Open a second PowerShell window in the same project folder:

```powershell
cd "C:\Users\YOUR_NAME\Downloads\ORCA-Marine-Dashboard-Local"
.\.venv\Scripts\Activate.ps1
npm run frontend
```

Open this URL in your browser:

```text
http://localhost:8000
```

Use the demo query:

```text
Is tomorrow morning good for fishing near my location?
```

Then click **Analyze query**. The timeline should progress through Understanding, Planning, Weather, Ocean, Geospatial, Risk, Cross-source, and Decision.

## M. Stop the servers

In each PowerShell window, press:

```text
Ctrl+C
```

## N. Restart the application

Start the backend in one window:

```powershell
cd "C:\Users\YOUR_NAME\Downloads\ORCA-Marine-Dashboard-Local"
.\.venv\Scripts\Activate.ps1
npm run backend
```

Start the frontend in a second window:

```powershell
cd "C:\Users\YOUR_NAME\Downloads\ORCA-Marine-Dashboard-Local"
.\.venv\Scripts\Activate.ps1
npm run frontend
```

Then revisit `http://localhost:8000`.

## Troubleshooting

### Port 8000 or 8001 is already in use

Find the process:

```powershell
Get-NetTCPConnection -LocalPort 8000,8001 -ErrorAction SilentlyContinue
```

Stop the process using its PID:

```powershell
Stop-Process -Id YOUR_PID -Force
```

### PowerShell says script execution is disabled

Run this in the current PowerShell window, then activate the environment again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### The page loads but analysis cannot reach the backend

Confirm that the backend terminal is still running and that this URL returns JSON:

```text
http://localhost:8001/api/health
```

Restart the backend if necessary. The Vite development server proxies `/api` requests to port `8001`.

### NVIDIA is unavailable

This is supported. Leave `NVIDIA_API_KEY` empty and ORCA will show the deterministic fallback reasoning. The prototype should not remain in a loading state.

### `npm install` or Python installation fails

Check that Node.js and Python are installed and restart PowerShell. Then run:

```powershell
npm install
python -m pip install -r backend\requirements.txt
```
