# Tech Stack Manual

## 1. System Overview

- home/cohorts: cohort card view
- students: after joining a cohort, you can see students list
- teams: not working
- courses: list of courses, projects, books
- foundations: search tools for student's exercises

---

## 2. Services

| Service Name | Tech Stack (including version) | Purpose | Where to find |
|---|---|---|---|
|Infrastructure|docker|| docker-compose.yml |
|api | Django (Python v3.11.11) | backend API | [learn-ops-api/Pipfile](../learn-ops-api/Pipfile) ||
|client/Front-end | Node React | |[learn-ops-client/package.json](../learn-ops-client/package.json) |

---

## 3. Run Questions

### 3a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| .env | learn-ops-infrastructure/.env | POSTGRES_DB | 数据库的名称 | PostgreSQL 容器启动时用它创建数据库，API 连接时也引用这个名字 |
| .env | learn-ops-infrastructure/.env | POSTGRES_USER | 数据库登录用户名 | PostgreSQL 用它创建用户账户，DATA_SOURCE_NAME 里引用它来构建连接字符串 |
| .env | learn-ops-infrastructure/.env | DATA_SOURCE_NAME | 完整的数据库连接字符串 | postgres_exporter 容器用它连接数据库，收集 DB 性能指标暴露给 Prometheus |

### 3b. How to Start It

run 'make up'

### 3c. Where to Access It

| Service | URL | Note |
| ----------------- | --------------------- | ------------- |
| Frontend (Client) | http://localhost:3000 | React Frontend UI    |
| Backend (API)     | http://localhost:8000 Admin: http://localhost:8000/admin    | Django API; |
| Grafana           | http://localhost:3001 | Monitoring the System         |
| Prometheus        | http://localhost:9090 | Performance Index |
| PostgreSQL        | localhost:5432        | DB |
| Postgres Exporter | http://localhost:9187 | DB Index     |

### 3d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| database | 无 | 最底层基础服务，不依赖任何其他服务 |
| api | database (`service_healthy`) | API 需要读写数据库；若数据库未就绪，API 启动后无法处理任何请求 |
| client | 无（docker层面） | 前端容器本身可独立启动，但功能上需要 api 运行才有意义 |
| prometheus | api | 需要 api 已在运行，才能访问 `/metrics/metrics` 端点抓取数据 |
| grafana | prometheus | 需要 prometheus 提供数据源，否则仪表板无数据可显示 |
| postgres_exporter | database | 需要连接数据库才能收集并暴露 DB 性能指标 |

### 3e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| api (Django) | `learn-ops-api/entrypoint.sh`（容器启动脚本） / `manage.py`（Django CLI 入口） | `learn-ops-api/LearningPlatform/urls.py` |
| client (React) | `learn-ops-client/src/index.js` | App.js 或独立的 router.js / routes.js; `learn-ops-client/src/components/`（路由定义在组件树中） |
| database | 由 `postgres:16` 官方镜像内部定义，无自定义文件 | 无（数据库不处理 HTTP 请求）|
| prometheus | 由官方镜像内部定义，通过 `prometheus.yml` 配置 | 无 |
| grafana | 由官方镜像内部定义 | 无 |