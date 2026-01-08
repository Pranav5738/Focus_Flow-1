# Focus_Flow-1
FocusFlow is a full-stack habit tracker with a Vite + React frontend and a FastAPI backend featuring JWT authentication, habit management, streak tracking, leaderboards, and analytics APIs.

## Local development

### Backend
- Install deps: `python -m pip install -r backend/requirements.txt`
- Run: `python -m uvicorn backend.server:app --host 127.0.0.1 --port 8000 --reload`

Backend defaults to a **file-backed local DB** (created at runtime) if `MONGO_URL` is not set.

#### Use MongoDB (recommended)
- Start MongoDB locally (Docker): `docker compose up -d`
- Create `backend/.env` from `backend/.env.example` and set:
	- `MONGO_URL=mongodb://localhost:27017`
	- `DB_NAME=habit_tracker` (or your preferred DB name)

The backend will automatically use MongoDB when `MONGO_URL` is reachable; otherwise it falls back to the file DB.

### Frontend
- Install deps: `cd frontend && npm install`
- Run: `cd frontend && npm run dev`

Optional environment variables:
- `VITE_BACKEND_URL` (defaults to `http://127.0.0.1:8000`)

## Deployment

### Render (Backend)

This repo includes a ready-to-use Render Blueprint at [render.yaml](render.yaml).

- Create a new Render **Web Service** from this GitHub repo (or use "Blueprint")
- Set environment variables in Render:
	- `JWT_SECRET` (required)
	- `CORS_ORIGINS` (required; your Vercel URL(s), comma-separated)
	- `MONGO_URL` (recommended; e.g., MongoDB Atlas connection string)
	- `DB_NAME` (optional; defaults to `habit_tracker`)

Start command used by Render:

`uvicorn backend.server:app --host 0.0.0.0 --port $PORT`

### Vercel (Frontend)

This repo includes a Vercel config at [vercel.json](vercel.json) that builds the Vite app in `frontend/`.

- Import the GitHub repo into Vercel
- Add an environment variable:
	- `VITE_BACKEND_URL` = your Render backend URL (example: `https://your-render-service.onrender.com`)
- Deploy

If you use React Router paths, `vercel.json` is already set up to rewrite all routes to `index.html`.

### Backend (FastAPI)

Run in production with:

`uvicorn backend.server:app --host 0.0.0.0 --port $PORT`

Required environment variables:
- `JWT_SECRET` (required in production; backend refuses to start without it)
- `CORS_ORIGINS` (comma-separated frontend origins, e.g. `https://your-frontend.com`)

Leaderboard environment variables:
- `LEADERBOARD_TZ` (optional; fixed timezone for leaderboard windows and countdowns, default `UTC`)
- `LEADERBOARD_INTERNAL_TOKEN` (optional; required only if you plan to call `POST /api/leaderboard/updateScore` for internal/admin operations)

Persistence options:
- **Recommended (production): MongoDB**
	- Set `MONGO_URL` and optionally `DB_NAME` (or `MONGO_DB_NAME`)
- **Fallback (local/demo): file-backed DB**
	- Uses `backend/data/db.json` by default
	- Override with `DATA_FILE=/path/to/db.json`
	- Not recommended for multi-instance deployments

Notes:
- If you set `CORS_ORIGINS=*`, the backend disables credentials for safety.
- The weekly leaderboard reset uses an in-process scheduler (APScheduler). For multi-instance production deployments, ensure only one instance runs the scheduler or use an external cron to trigger the reset logic (the code is idempotent).
