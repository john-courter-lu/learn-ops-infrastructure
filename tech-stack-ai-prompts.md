# Tech Stack AI Prompts

## 1. Run Questions

### 1a. Config Files

This is a multi-service Docker project. Look at the following config and environment files:

- `learn-ops-infrastructure/.env`
- `learn-ops-infrastructure/prometheus.yml`
- `learn-ops-infrastructure/docker-compose.yml`
- `learn-ops-api/.env`
- `learn-ops-client/.env`

For each file, pick at least 3 config values and fill in the markdown table under ### 1a. Config Files in tech-stack-ai.md
| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|

IMPORTANT: Write only the variable NAME (e.g. POSTGRES_USER), never the actual value or secret.
Explain what each value configures and how the running system uses it.



### 1b. How to Start It

Look at the `Makefile` in the root of the `learn-ops-infrastructure` repo.

List every `make` command available, then explain:
1. What each command does
2. When you would use it (e.g. first-time setup vs. daily startup vs. teardown)
3. How the commands differ from each other

Format the output as a short explanation followed by a code block showing the commands, and write it into the related section in tech-stack-ai.md. 


### 1c. Where to Access It

Look at `learn-ops-infrastructure/docker-compose.yml`.

For every service defined in the file, find its exposed port and fill in this markdown table in tech-stack-ai.md:

| Service | Port | URL |
|---|---|---|

The URL should be in the format `http://localhost:{port}`.
If a service exposes multiple ports, add a row for each one and note what each port is for.


### 1d. Service Dependencies

Look at `learn-ops-infrastructure/docker-compose.yml`.

Map out the dependencies between services using the `depends_on` fields. Fill in this markdown table in tech-stack-ai.md:

| Service | Depends On | Why |
|---|---|---|

For the "Why" column: do not just restate the dependency — explain the functional reason.
For example: "The API cannot respond to requests until the database is accepting connections."
Also note if any dependency uses `condition: service_healthy` vs a plain `depends_on`, and explain what that difference means.


### 1e. Main Entry Points

I have two services: a Django API (`learn-ops-api/`) and a React client (`learn-ops-client/`).

For each service, find TWO separate files:
1. The **startup file** — where the server process boots up (e.g. manage.py, entrypoint.sh, index.js)
2. The **routes/URL config file** — where incoming HTTP requests are mapped to handlers (e.g. urls.py, router.js, App.js)

These are two different files. Fill in this markdown table in tech-stack-ai.md:

| Service | Startup File | Routes / URL Config File |
|---|---|---|

For services that use official Docker images with no custom code (postgres, prometheus, grafana),
note "Built into official image" for the startup file and "N/A" for routes.



## 2. Services

Using the contents of these files:

- `learn-ops-infrastructure/docker-compose.yml` (for service names and infrastructure image versions)
- `learn-ops-api/Pipfile` (for Python/Django version)
- `learn-ops-client/package.json` (for Node/React version)

Fill in this markdown table — one row per service:

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|

For "Tech Stack (including version)": include the specific version number where available
(e.g. "PostgreSQL 16", "Django 4.x", "React 18.x"). Do not write "latest" — look up the
actual version from the files above.

For "Purpose": write one sentence describing what this service does within the system,
not what the technology is in general.

Then write the completed table into the `## 2. Services` section of `tech-stack-ai.md`,
replacing the empty placeholder row.

## 3. System Overview
Read through this project: it has a Django REST API (`learn-ops-api`), a React frontend
(`learn-ops-client`), a PostgreSQL database, and monitoring via Prometheus and Grafana.
The repo is called `learn-ops-infrastructure`.

Write a system overview for `tech-stack-ai.md` consisting of exactly 3 paragraphs:

**Paragraph 1 — What it is:**
Describe what kind of application this is (e.g. web app, internal tool, learning platform)
and what problem it is designed to solve. Base this on what you can infer from the repo
name, folder names, and any README files.

**Paragraph 2 — Main features:**
Describe the system's main features from a user's perspective — what can a user actually
do with it? Do not list technologies; describe functionality.

**Paragraph 3 — Who uses it:**
Describe who the intended users are. Are there different roles (e.g. student, instructor,
admin)? If so, explain how each role interacts with the system differently.

Write the completed overview into the `## 3. System Overview` section of `tech-stack-ai.md`,
replacing any placeholder text.