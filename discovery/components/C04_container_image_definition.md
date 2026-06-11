## C04 — container image definition
ID: M-009
Layer: infra
Source file: api/Dockerfile

---

**Module** — container image definition
**ID** — M-009
**Layer** — infra
**Primary Responsibility** — Define the reproducible build for the `api` container image. Establishes the Python 3.11 runtime, installs dependencies, copies source, and declares the uvicorn startup command.

**Inputs**
- `api/requirements.txt` — copied first to exploit Docker layer caching. `pip install` layer is cached when only source code changes.
- `api/` directory contents — copied after pip install step. Includes `main.py`, test files, and any other files in `api/`.

**Outputs**
Built container image with:
- Base: `python:3.11-slim`
- Working directory: `/app`
- Installed packages: fastapi, uvicorn, psycopg2-binary, pytest, requests (from requirements.txt — no version pins)
- Startup command: `["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]` (exec form — not shell form)

**Public Interface**
- `docker compose build api` or `docker compose up --build` — triggers image build
- `docker build ./api` — direct build (outside Compose context)
- Referenced by M-006: uvicorn entry point `main:app` maps to api/main.py's `app` object

**Error Behaviour**
- `RUN pip install --no-cache-dir -r requirements.txt` — STARTUP-FATAL build failure if any package cannot be resolved or installed. No version pins in requirements.txt means network availability and PyPI state at build time determine which versions are installed.
- `COPY . .` — copies all files from the build context (`./api`). Includes test files (test_errors.py, test_injection.py, test_invariants.py). Test files are present in the running container — not excluded.
- `CMD ["uvicorn", "main:app", ...]` — exec form (JSON array). STARTUP-FATAL if uvicorn is not installed or if `main.py` has a syntax error.

**Known Fragility**
- **No version pins in requirements.txt**: packages resolve to latest compatible at build time. A breaking change in fastapi, psycopg2-binary, or uvicorn would produce a broken image without any local change to the repo. This is the highest-impact silent fragility in the infra layer.
- **No non-root user**: the container runs as root. Not a vulnerability for the current development scope, but a hardening requirement for any production deployment.
- **No `.dockerignore`**: `COPY . .` copies everything in `api/`, including test files and `__pycache__/`. Test files in the production image are dead weight (not harmful, but not needed at runtime).
- **Single uvicorn worker**: CMD starts one uvicorn process with no `--workers` flag. All concurrent requests are handled by one process. Consistent with the per-request psycopg2 connection design (R-004) — adding workers without adding a connection pool would multiply connection overhead.
- `--no-cache-dir` reduces image size by not caching pip downloads, at the cost of slower rebuilds when the pip cache would otherwise have been warm.

**Change Impact**
- Changing `main:app` in CMD requires a matching rename in main.py.
- Adding `--workers N` to CMD would change concurrency model and interact with per-request psycopg2 connections.
- Adding a `USER` instruction requires ensuring the new user has read permissions on `/app`.
- Adding version pins to requirements.txt (strongly recommended) would make builds reproducible.

**Callers** — M-008 (docker-compose.yml `build: ./api`); Docker build CLI
**Calls** — M-006 (uvicorn `main:app` starts the FastAPI application)
**Integration Points Used** — None
