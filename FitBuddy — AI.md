# FitBuddy — AI Fitness Plan Generator

A complete FastAPI application for generating and refining seven-day fitness plans. It uses Jinja2-rendered HTML, SQLAlchemy + SQLite, Pydantic validation, and Google's current **Google Gen AI Python SDK**. When a Gemini key is not configured, a deterministic local demo generator keeps the entire application usable offline.

> **Model/API note:** The source brief names Gemini 1.5 and `google-generativeai`. Google's current documentation recommends `google-genai`; the older `google-generativeai` package is no longer actively maintained. FitBuddy uses the current Interactions API with configurable model IDs (`gemini-3.8-flash` for plans and `gemini-3.5-flash-lite` for short tips). Each request sets `store=False` so profile prompts are not retained as stored interactions by Google's API. Confirm model availability for your own Google AI Studio account and change the values in `.env` if needed.

## Features

- Personalized seven-day plan generation with warm-ups, exercise details, rest/recovery, and safe progression language.
- Concise nutrition/recovery guidance aligned with the selected goal.
- Feedback-based revisions; the first version remains available for comparison.
- SQLite persistence for user profiles and multiple plans.
- Coach dashboard listing profiles and plan versions; deletion removes a profile and its saved plans.
- HTML flows and JSON API endpoints, with automatically generated Swagger docs.
- Gemini requests opt out of remote interaction storage (`store=False`).
- Optional HTTP Basic protection for coach/admin pages and the user-list API.
- Built-in local demo mode so setup and feature testing do not require a Gemini key.

## Project structure

```text
fitbuddy/
├── app/
│   ├── main.py                    # FastAPI app and startup
│   ├── routes.py                  # HTML and JSON endpoints
│   ├── config.py                  # Environment configuration
│   ├── database.py                # SQLAlchemy engine/session
│   ├── models.py                  # User and WorkoutPlan tables
│   ├── schemas.py                 # Pydantic validation
│   └── services/
│       ├── gemini_client.py       # Google Gen AI SDK wrapper
│       ├── gemini_generator.py    # Seven-day plan + offline fallback
│       ├── gemini_flash_generator.py # Nutrition tip + offline fallback
│       └── updated_plan.py        # Feedback-based plan revision
├── templates/                     # Jinja2 pages
├── static/css/style.css           # Responsive UI
├── tests/test_app.py              # Functional tests
├── .vscode/                       # Debug configuration and test task
├── requirements.txt
└── .env.example
```

## Requirements

- Python **3.10+** (3.11 or 3.12 recommended)
- VS Code and the Microsoft Python extension
- Optional: a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey)

## VS Code: install and run

1. Download/unzip the project folder and open **the `fitbuddy` folder** in VS Code (`File → Open Folder…`).
2. Open the integrated terminal (`Terminal → New Terminal`). Create and activate a virtual environment:

   **Windows PowerShell**
   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   **macOS / Linux**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install dependencies:
   ```bash
   python -m pip install --upgrade pip
   python -m pip install -r requirements.txt
   ```
4. Optional Gemini setup: copy `.env.example` to `.env`, then add your key:

   ```dotenv
   GEMINI_API_KEY=your_google_ai_studio_key
   GEMINI_WORKOUT_MODEL=gemini-3.8-flash
   GEMINI_TIP_MODEL=gemini-3.5-flash-lite
   ```

   Keep `.env` private; it is excluded by `.gitignore`. If you leave the key blank, FitBuddy runs in local demo mode. To use different model IDs, edit the two `GEMINI_*_MODEL` settings.
5. Start the app from the project root:
   ```bash
   python -m uvicorn app.main:app --reload
   ```
6. Open:
   - App: <http://127.0.0.1:8000>
   - API explorer: <http://127.0.0.1:8000/docs>
   - Health check: <http://127.0.0.1:8000/health>
   - Coach dashboard: <http://127.0.0.1:8000/view-all-users>

On first startup, the SQLite database `fitbuddy.db` and its tables are created in the project root. Stop the server with `Ctrl+C`.

### VS Code run/debug

Choose **Run and Debug → FitBuddy: FastAPI** to launch with the included `.vscode/launch.json`. For tests, use the Testing panel, the task **FitBuddy: Run tests**, or the terminal command below. If VS Code asks for a Python interpreter, select the `.venv` environment created above.

## Test the application

Run the functional suite from the project root:

```bash
python -m pytest -q
```

The suite exercises the HTML form, local plan generation, database persistence, feedback revisions, JSON endpoints, validation, admin listing, and profile deletion. Tests use an isolated in-memory SQLite database.

### Manual browser test

1. Open the home page, fill in the profile, and choose a goal/intensity.
2. Submit **Create my 7-day plan**. Confirm the plan shows seven days and a nutrition/recovery tip. In demo mode the page shows a local-demo notice.
3. Submit feedback such as “Please add more gentle cardio and keep an extra rest day.” The updated plan appears, and **View the original plan** retains the first version.
4. Open **Coach view** to inspect saved members and plan versions. The delete button asks for confirmation before removing the member and their plans.

### JSON API example

Create a plan (the API returns its numeric `plan_id`):

```bash
curl -X POST http://127.0.0.1:8000/api/plans \
  -H 'Content-Type: application/json' \
  -d '{"name":"Riley Example","user_id":"RILEY-001","age":32,"weight":68.5,"goal":"general_wellness","intensity":"medium"}'
```

Revise it using the returned ID:

```bash
curl -X POST http://127.0.0.1:8000/api/plans/1/feedback \
  -H 'Content-Type: application/json' \
  -d '{"feedback":"Add gentle cardio and keep an extra rest day."}'
```

Read it with `GET /api/plans/{plan_id}`. Supported goals are `weight_loss`, `muscle_gain`, `general_wellness`, and `flexibility`; supported intensities are `low`, `medium`, and `high`.

## Optional admin protection

For local development, the coach dashboard and user-list API are open by default. Before exposing the app to a network, add a long unique password in `.env` and restart:

```dotenv
ADMIN_USERNAME=coach
ADMIN_PASSWORD=replace-with-a-long-unique-password
```

The browser prompts for HTTP Basic credentials at `/view-all-users`; API clients use the same Basic Auth for `/api/users`. **This is a local educational project, not a production-ready identity or deployment system.** Do not expose it publicly without proper user authentication, HTTPS, CSRF protection, rate limiting, backups, and a privacy review.

## Configuration reference

| Variable | Purpose | Default |
| --- | --- | --- |
| `GEMINI_API_KEY` | Google AI Studio key (server-side only) | empty; local demo mode |
| `GOOGLE_API_KEY` | Alternate key name accepted for compatibility | unused unless `GEMINI_API_KEY` is empty |
| `GEMINI_WORKOUT_MODEL` | Model for full plans and revisions | `gemini-3.8-flash` |
| `GEMINI_TIP_MODEL` | Model for short nutrition/recovery tips | `gemini-3.5-flash-lite` |
| `DATABASE_URL` | SQLAlchemy database URL | project-local SQLite database |
| `ADMIN_USERNAME` | Optional Basic Auth username | `coach` |
| `ADMIN_PASSWORD` | Optional Basic Auth password | empty (admin routes open for local-only use) |

If Gemini returns an authorization/model error, verify the key and account model access, update model names in `.env`, and restart the server. The application never sends the API key to the browser.

## Safety and privacy

FitBuddy offers general wellness suggestions, not diagnosis, treatment, or individualized clinical/nutrition advice. Its plans encourage rest, gradual progression, and stopping if pain or concerning symptoms occur. Only collect profile details you need; profiles and plans are stored locally in the SQLite file. Do not use real sensitive health information in a public or shared installation without appropriate security and consent controls.
