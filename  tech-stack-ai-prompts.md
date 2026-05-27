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

Map out the dependencies between services using the `depends_on` fields. Fill in this markdown table:

| Service | Depends On | Why |
|---|---|---|

For the "Why" column: do not just restate the dependency — explain the functional reason.
For example: "The API cannot respond to requests until the database is accepting connections."
Also note if any dependency uses `condition: service_healthy` vs a plain `depends_on`, and explain what that difference means.


### 1e. Main Entry Points



## 2. Services



## 3. System Overview
