# learn-ops-api: Service Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| `LearningAPI/` | Core Django app — holds all models, views, serializers, and tests |
| `LearningPlatform/` | Django project config — `settings.py`, root `urls.py`, `wsgi.py` |
| `LogViewer/` | Secondary app for surfacing runtime logs through an API endpoint |
| `config/` | Server-level configs (nginx, gunicorn, deployment overrides) |
| `staticfiles/` | Collected output of `manage.py collectstatic`; served by nginx in production |

## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|----------------------------------------------------------|
| `models/` | Defines database schema as Python classes; split into `people/`, `coursework/`, `skill/` sub-domains |
| `migrations/` | Auto-generated versioned SQL change history; tracks every schema evolution |
| `fixtures/` | JSON seed data loaded via `manage.py loaddata`; used for test setup and local dev |
| `serializers/` | Translates Python objects ↔ JSON; controls which fields are exposed per resource |
| `views/` | Request handlers — maps HTTP verbs to business logic for each endpoint |
| `tests/` | pytest test suite verifying endpoint behavior, data integrity, and auth rules |

## 3. What is the Pipfile?

The `Pipfile` is the dependency manifest for this project, managed by `pipenv`. It declares all required third-party packages separated into `[packages]` (production) and `[dev-packages]` (dev/test only), and pins the Python version to `3.11.11`. It replaces `requirements.txt` with clearer dev/prod separation and pairs with `Pipfile.lock` to guarantee identical environments across all machines and CI.

## 4. Key packages in the Pipfile

| Package | What functionality does it provide and why? |
|---------|----------------------------------|
| `django` | The core web framework — provides ORM, URL routing, migrations, admin interface, and middleware. Everything else is built on top of it. |
| `djangorestframework` | Adds REST API tooling on top of Django — serializers, ViewSets, permission classes, and the browsable API. Essential because this project is a pure JSON API. |
| `django-allauth` | Full authentication system — account registration, login, email verification, password reset, and OAuth social login. Paired with `dj-rest-auth` to power all user identity flows. |

## 5. What does `decorators.py` do?

`decorators.py` defines two permission guard decorators: `@is_instructor()` and `@is_staff()`. Each wraps a view function and checks whether the requesting user belongs to the `Instructors` or `Staff` Django group before allowing execution. If the check fails, it immediately returns `HTTP 401 Unauthorized`. This enforces role-based access control (RBAC) at the view level without duplicating permission logic across multiple views.

## 6. What is a serializer, and what serializers are defined here?

A serializer is the translation layer between Python model instances and JSON. It handles bidirectional conversion: Python → JSON for responses and JSON → Python + validation for incoming requests. It also controls which fields are exposed via a `fields` whitelist in its `Meta` class.

| Serializer | Fields exposed |
|------------|---------------|
| `UserSerializer` | `url`, `first_name`, `last_name`, `username`, `email`, `groups` |
| `NssUserSerializer` | `url`, `slack_handle`, `github_handle`, `mentor`, `user` |
| `CohortSerializer` | Full cohort data including nested courses, computed URLs, coaches, students |
| `MiniCohortSerializer` | `id`, `name` only — lightweight version used in search results |

## 7. Models and what they represent

| Model | Real-world thing it represents |
|-------|-------------------------------|
| `Cohort` | A student class/batch — tracks name, dates, Slack channel, active status, and enrolled members |
| `NssUser` | Extended platform user — adds `slack_handle`, `github_handle`, and cohort assignments to Django's built-in User |
| `StudentAssessment` | A single student's submission and result for a given assessment |
| `OneOnOneNote` | A private instructor note written during a 1-on-1 session with a student |
| `Assessment` | An evaluable task or project assigned within a course book |

## 8. Views vs. viewsets

| Type | Example class | File path |
|------|--------------|-----------|
| View (function-based) | `notify` | `LearningAPI/views/notify.py` |
| ViewSet | `TagViewSet` | `LearningAPI/views/tag_view.py` |
| ViewSet (custom) | `CohortViewSet` | `LearningAPI/views/cohort_view.py` |

Use a **ViewSet** when a resource needs full CRUD. Use a **plain view** when the operation is a one-off action (e.g., send a Slack notification) with no CRUD concept.

## 9. Serializers paired with their models

| Serializer | Model | Location |
|------------|-------|------|
| `CohortSerializer` | `Cohort` | `LearningAPI/views/cohort_view.py` |
| `MiniCohortSerializer` | `Cohort` | `LearningAPI/views/cohort_view.py` |
| `UserSerializer` | `django.contrib.auth.User` | `LearningAPI/serializers/` |
| `NssUserSerializer` | `NssUser` | `LearningAPI/serializers/` |
| `CapstoneSerializer` | `Capstone` | `LearningAPI/serializers/` |

## 10. What replaces the Templates and why?

**Serializers replace Templates.** In a traditional Django app, templates render Python data into HTML server-side. In this project, serializers render Python data into JSON — serving the same role but for API consumers instead of browsers.

This makes sense because the app is a **decoupled REST API** serving potentially multiple clients (web frontend, mobile, third-party tools). By returning JSON instead of HTML, the backend stays client-agnostic: any frontend framework can consume the same endpoints and handle its own rendering. The presence of `django-cors-headers` in the Pipfile confirms this design — it explicitly enables cross-origin requests from a separately hosted frontend.