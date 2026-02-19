# Repro Tilt (Local Dev)

Centralized Tilt setup for all Repro projects (excluding SDKs). It runs core services plus demo apps, with a shared local MongoDB.

**Prereqs**
1. Docker (for Mongo)
2. Tilt
3. Node + npm (for JS apps)
4. Python (for `repro-ai`) (experimental)

**Quick Start**
1. `cd repro-tilt`
2. `tilt up`

**Core Services**
- `repro-api` — http://localhost:4010/api/docs
- `repro-ai` — http://localhost:8000/docs
- `repro-web` — http://localhost:5174

**Demo Apps**
- `repro-demo-backend` — http://localhost:3008
- `repro-demo-web` — http://localhost:5173
- `repro-demo-next` — http://localhost:3000
- `repro-demo-node-e2e` — http://localhost:3001
- `repro-demo-nest-e2e` — http://localhost:3002
- `repro-demo-react-e2e` — http://localhost:5175
- `repro-demo-nest-react-e2e` — http://localhost:5176

**Mongo**
- Connection: `mongodb://localhost:27017`
- Databases used:
  1. `repro` (core services)
  2. `taskflow` (demo backend)
  3. `repro-e2e` (demo e2e apps)

**Grouping (Labels)**
- `infra`: mongo
- `core`: repro-api, repro-ai, repro-web
- `demo`: all `repro-demo-*`

**Env Loading**
Each service loads its local `.env` and `.env.local` (if present) before running. Tilt then sets required overrides (e.g. `MONGO_URI`, `PORT`) to keep everything consistent.

**Change Ports**
Edit the `PORTS` map in `repro-tilt/Tiltfile`. All links and dependent envs derive from it.

**Common Issues**
- `EADDRINUSE`: a port is already in use. Change `PORTS` or stop the conflicting process.
- `command not found`: install deps in that repo (e.g. `npm install` or `pip install -e .`).
