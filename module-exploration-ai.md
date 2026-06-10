# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| `LearningAPI/` | The main Django application — contains all models, views, serializers, tests, and the business logic of the platform |
| `LearningPlatform/` | Django project-level config — holds `settings.py`, the root `urls.py`, and `wsgi.py`; the entry point that wires everything together |
| `LogViewer/` | A separate Django app dedicated to exposing runtime logs through the API, kept isolated so log-surfacing logic doesn't clutter the main app |
| `config/` | Production server configuration (nginx, gunicorn); kept outside the app code so deployment config can change without touching application logic |
| `staticfiles/` | The output directory for `manage.py collectstatic`; needed so a web server (nginx) can serve CSS/JS directly without going through Django |

## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|------------------------------------------|
| `models/` | Defines the database schema as Python classes; split into `people/`, `coursework/`, and `skill/` sub-packages so models for different domains don't pile into one file |
| `migrations/` | Stores the auto-generated versioned SQL change history; Django uses these to evolve the database schema incrementally without manually writing SQL |
| `fixtures/` | JSON seed data loaded via `manage.py loaddata`; exists so the database can be populated with realistic test or development data in one command |
| `serializers/` | Translates model instances to and from JSON; separated from views so the same serializer can be reused across multiple endpoints |
| `views/` | Contains all request handlers — each file maps HTTP verbs to the logic for a particular resource or action |
| `tests/` | Automated tests for endpoint behavior, authorization, and data integrity; isolated here so they can be discovered and run by pytest without touching production code |

## 3. What is the Pipfile?

The `Pipfile` is the dependency manifest used by `pipenv` to manage this project's Python packages. It has two sections: `[packages]` for everything needed to run the app in production, and `[dev-packages]` for tools only needed during development and testing (like `pytest` and `debugpy`). It also pins the Python version to `3.11.11` under `[requires]`, and includes a `[scripts]` section with a shortcut (`pipenv run migrate`) so common management commands don't need to be typed in full. It pairs with `Pipfile.lock` to guarantee that every environment — local, CI, production — installs the exact same versions.


## 4. Key packages

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| `django` | The core web framework — provides the ORM, URL routing, migrations system, admin interface, and middleware. Every other package in the project layers on top of it. |
| `djangorestframework` | Adds REST API tooling to Django — ViewSets, serializers, permission classes, and the browsable API. Without it, building a JSON API would mean writing all of that from scratch. |
| `django-allauth` | Handles the full authentication lifecycle: registration, login, email verification, and OAuth social login. Pinned to `0.54.0`; works alongside `dj-rest-auth` to expose auth as API endpoints rather than HTML forms. |

## 5. What does `decorators.py` do?

`decorators.py` defines two access-control decorators: `@is_instructor()` and `@is_staff()`. Each one wraps a view function. When a request comes in, the wrapper checks `request.user.groups.filter(name='Instructors')` (or `'Staff'`) before calling the real view. If the user isn't in the required group, it immediately returns a `401 Unauthorized` response with a plain-English message and never executes the view logic.

The problem it solves: without these decorators, every protected view would need to repeat the same group-check boilerplate. By centralizing the check here, any view can add role enforcement with a single decorator line instead of duplicating the `if/else` logic.


## 6. What is a serializer, and how does it fit the request/response cycle?

A serializer is the translation layer between Python objects and JSON. It sits between the view and the data in both directions: on the way **in**, it takes raw JSON from the request body, validates it, and converts it into Python-ready data; on the way **out**, it takes a model instance (or queryset) and converts it into a JSON-serializable dictionary the response can return. The `fields` list in its `Meta` class acts as a whitelist — only those fields cross the boundary, which also prevents accidental data leakage.

**Concrete example:** `NssUserSerializer` in `serializers/nssuser_serializer.py` wraps the `NssUser` model and exposes exactly five fields: `url`, `slack_handle`, `github_handle`, `mentor`, and `user`. A view returns an `NssUser` instance → the serializer converts it → the client receives clean JSON with only those five keys.


## 7. One model and what it represents

**`NssUser`** (`models/people/nssuser.py`) represents a platform-specific user profile layered on top of Django's built-in `User`. It has a `OneToOneField` back to `User`, then adds `slack_handle` (the user's Slack Member ID) and `github_handle` (their GitHub username). It also computes derived data — a `score` property that totals the student's learning records and weights them by core-skill level, and a `current_cohort` property that returns the cohort they're currently enrolled in.

The API needs to track this data because Django's `User` only stores generic identity information. This platform also needs to send Slack notifications to specific users, link GitHub repositories to students, and calculate a learning progress score — none of which exist on the built-in model.


## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View (function-based) | `notify` in `views/notify.py` | Use for one-off actions that don't represent a resource — `notify` just fires a Slack message and returns; there's no "list of notifies" to retrieve or update, so CRUD doesn't apply. |
| ViewSet | `CohortViewSet` in `views/cohort_view.py` (extends `ViewSet`); `TagViewSet` in `views/tag_view.py` (extends `ModelViewSet`) | Use when a resource needs standard CRUD operations. DRF maps `list`, `retrieve`, `create`, `update`, `destroy` to the right HTTP methods automatically. `ModelViewSet` gives all of that for free; plain `ViewSet` lets you define only the actions you need. |

## 9. What replaces templates and why?

**Serializers replace templates.** In standard Django MTV, a template's job is to take Python data and render it into something a client can consume — traditionally HTML. In this project there are no HTML templates; instead, serializers render Python model instances into JSON. The output goes straight into the HTTP response body.

This design makes sense because the React frontend is a completely separate application running on its own server. It doesn't need the server to produce HTML — it needs raw data it can render itself. Returning JSON keeps the backend client-agnostic: the same endpoints can serve the React app, a mobile client, or any future consumer without changing a line of backend code. The `django-cors-headers` package in the Pipfile is the clearest sign of this split — it only exists because the frontend and API are on different origins.
