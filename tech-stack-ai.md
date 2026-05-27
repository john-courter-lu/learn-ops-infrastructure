# Tech Stack (AI)

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| `infrastructure/.env` | [learn-ops-infrastructure/.env](./.env) | `POSTGRES_DB` | The name of the PostgreSQL database to create | Passed into the `postgres:16` container at startup; Docker uses it to initialize a blank database with this name |
| `infrastructure/.env` | [learn-ops-infrastructure/.env](./.env) | `POSTGRES_USER` | The login username for the database | Used by the `database` container to create the DB owner account, and also interpolated into `DATA_SOURCE_NAME` so the postgres_exporter can authenticate |
| `infrastructure/.env` | [learn-ops-infrastructure/.env](./.env) | `DATA_SOURCE_NAME` | A full PostgreSQL connection string (URL format) | Consumed by the `postgres_exporter` container; it uses this to connect to the database and read internal Postgres stats, which it then exposes as metrics for Prometheus to scrape |
| `prometheus.yml` | [learn-ops-infrastructure/prometheus.yml](./prometheus.yml) | `scrape_interval` | How frequently Prometheus pulls (scrapes) metrics from each target | Set globally to `15s`; every 15 seconds Prometheus sends an HTTP request to each configured target to collect the latest metric readings |
| `prometheus.yml` | [learn-ops-infrastructure/prometheus.yml](./prometheus.yml) | `metrics_path` | The URL path on the Django API where metrics are exposed | Overrides Prometheus's default `/metrics` path; Django serves its metrics at `/metrics/metrics`, so without this override, scraping would return a 404 |
| `prometheus.yml` | [learn-ops-infrastructure/prometheus.yml](./prometheus.yml) | `targets` (postgresql job) | The hostname and port of the postgres_exporter container | Tells Prometheus where to send its scrape requests for DB metrics; `postgres_exporter:9187` works because both containers share the `learningplatform` Docker network, so Docker resolves the hostname automatically |
| `docker-compose.yml` | [learn-ops-infrastructure/docker-compose.yml](./docker-compose.yml) | `depends_on` (condition: service_healthy) | Ensures the `api` container waits for the database to be ready | Without this, the API might start and try to connect to Postgres before it's finished initializing, causing crashes; Docker polls the `pg_isready` healthcheck before releasing the `api` service to start |
| `docker-compose.yml` | [learn-ops-infrastructure/docker-compose.yml](./docker-compose.yml) | `ports` ("3001:3000" for Grafana) | Maps a host port to a container port | The left side is your laptop's port, the right is the container's internal port; Grafana runs on `3000` internally but is remapped to `3001` on the host because the `client` service already uses `3000` |
| `docker-compose.yml` | [learn-ops-infrastructure/docker-compose.yml](./docker-compose.yml) | `networks: external: true` | Declares that the `learningplatform` network was created outside this Compose file | This means you must run `docker network create learningplatform` manually first; using `external: true` lets multiple Compose files or standalone containers share the same network |
| `learn-ops-api/.env` | [learn-ops-api/.env](../learn-ops-api/.env) | `LEARNING_GITHUB_CALLBACK` | The URL GitHub redirects to after a user authorizes the OAuth app | Must exactly match the callback URL registered in the GitHub OAuth App settings; after login, GitHub POSTs an auth code here so Django can exchange it for a user token |
| `learn-ops-api/.env` | [learn-ops-api/.env](../learn-ops-api/.env) | `LEARN_OPS_ALLOWED_HOSTS` | Django's security list of valid hostnames that can make requests to the API | Django rejects any HTTP request whose `Host` header isn't in this list, preventing host-header injection attacks; `api` is included so inter-container requests work |
| `learn-ops-api/.env` | [learn-ops-api/.env](../learn-ops-api/.env) | `VALKEY_HOST` | The hostname of the Valkey (Redis-compatible) cache service | Django uses this to locate the cache for storing sessions, rate-limit counters, or async task queues; Docker resolves `valkey` to the right container IP on the shared network |
| `learn-ops-api/.env` | [learn-ops-api/.env](../learn-ops-api/.env) | `DEBUG` | Toggles Django's debug mode on or off | When `True`, Django shows detailed error pages and disables some security checks; must be `False` in production or sensitive stack traces are exposed to anyone who triggers an error |
| `learn-ops-client/.env` | [learn-ops-client/.env](../learn-ops-client/.env) | `REACT_APP_API_URI` | The base URL the React frontend uses to make API calls | Every `fetch`/`axios` call in the React app is prefixed with this value; pointing it at `localhost:8000` means the browser talks directly to the Django container's exposed port |
| `learn-ops-client/.env` | [learn-ops-client/.env](../learn-ops-client/.env) | `CHOKIDAR_USEPOLLING` | Forces the file watcher to use polling instead of OS filesystem events | Needed inside Docker on Windows/WSL because native filesystem events often don't cross the container boundary; with this `true`, the dev server detects code changes and hot-reloads correctly |
| `learn-ops-client/.env` | [learn-ops-client/.env](../learn-ops-client/.env) | `GENERATE_SOURCEMAP` | Controls whether the build produces `.map` files that map minified JS back to source | Set to `false` to keep the dev bundle smaller and avoid leaking readable source code through the browser's DevTools network tab |

### 1b. How to Start It

All commands are run from inside [`learn-ops-infrastructure/`](.) using `make`. The [Makefile](./Makefile) is a thin wrapper around `docker compose` and two setup scripts — it exists so you don't have to remember the full Docker flags. Commands fall into four groups: **first-time setup**, **daily use**, **inspection**, and **teardown/reset**.

```bash
# ── First-time setup ──────────────────────────────────────────────
make setup        # Run once: creates the Docker network, copies .env files, and
                  # does any other one-time bootstrapping via scripts/setup.sh

make doctor       # Check-up: re-runs setup.sh --doctor to verify your environment
                  # is correctly configured (network exists, dependencies present, etc.)
                  # Use when something feels broken and you want a diagnosis before debugging

# ── Daily startup ─────────────────────────────────────────────────
make up           # Build and start ALL services (db, api, client, prometheus,
                  # grafana, postgres_exporter) in the background (-d = detached)

make up-api       # Build and start only the API container
                  # Use when you're working on backend code and don't need the client

make up-client-api # Build and start the API + client only (no monitoring stack)
                  # Use when you're working on the frontend and need the API responding

make restart      # Tear down then rebuild and restart all services
                  # Use after changing a Dockerfile or docker-compose.yml

# ── Inspection ────────────────────────────────────────────────────
make ps           # Show the running status of all containers (like 'docker ps')
                  # Quick check: is everything actually up?

make logs         # Stream live logs from all containers to your terminal (-f = follow)
                  # Use to watch what's happening in real time or hunt down errors

# ── Teardown & reset ──────────────────────────────────────────────
make down         # Stop and remove containers, but KEEP named volumes (database data survives)
                  # Use at end of day or to free up resources without losing your data

make teardown     # Full environment teardown via scripts/teardown.sh
                  # Use when you want to undo everything make setup did

make reset        # Stop containers AND delete all volumes (--remove-orphans cleans stale ones)
                  # ⚠️  DESTRUCTIVE: wipes the database. Use to start completely fresh.
```

**How the commands differ from each other:**

- `up` vs `up-api` vs `up-client-api` — scope. `up` starts every service defined in `docker-compose.yml`; the others start a subset. Use a subset when you want faster startup and lower resource use.
- `down` vs `reset` — data safety. `down` stops containers but leaves your database volume intact so data persists. `reset` adds `-v`, which deletes volumes — your database is wiped and migrations will re-run on next `up`.
- `restart` vs `up` — `restart` is `down` + `up` in one step. Use it when containers are already running and you need a clean rebuild; plain `up --build` on an already-running stack also works but `restart` is explicit about the cycle.
- `setup` vs `doctor` — `setup` is destructive-safe initialization (safe to run on a fresh machine). `doctor` is read-only verification — it checks without changing anything.

### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
| `database` | 5432 | http://localhost:5432 (PostgreSQL — connect via a DB client, not a browser) |
| `api` | 8000 | http://localhost:8000 (Django REST API) |
| `api` | 5678 | http://localhost:5678 (debugpy remote debugger — attach your IDE here to set breakpoints) |
| `client` | 3000 | http://localhost:3000 (React frontend) |
| `prometheus` | 9090 | http://localhost:9090 (Prometheus metrics UI) |
| `grafana` | 3001 | http://localhost:3001 (Grafana dashboards — host port remapped from container's 3000 to avoid clash with client) |
| `postgres_exporter` | 9187 | http://localhost:9187 (Postgres metrics exporter — scraped by Prometheus, rarely accessed directly) |

### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| | | |
| | | |
| | | |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| | | |
| | | |
| | | |

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| | | |

## 3. System Overview