# FitBuddy – AI Fitness Plan Generator

FitBuddy is a FastAPI + Jinja2 + SQLite web app based on the supplied project documentation. It generates a structured 7-day fitness plan, a nutrition/recovery tip, and can revise the plan from user feedback.

## Architecture

- **Frontend:** HTML + Jinja2 + responsive CSS
- **Backend:** FastAPI
- **AI:** Google GenAI Python SDK (`google-genai`)
- **Workout/update model:** `GEMINI_PRO_MODEL` (default `gemini-2.5-pro`)
- **Nutrition model:** `GEMINI_FLASH_MODEL` (default `gemini-2.5-flash`)
- **Database:** SQLite + SQLAlchemy
- **Validation:** Pydantic
- **Admin:** Session-protected dashboard

## Project structure

```text
FitBuddy/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── schemas.py
│   ├── ai_models.py
│   ├── gemini_client.py
│   ├── gemini_generator.py
│   ├── gemini_flash_generator.py
│   ├── updated_plan.py
│   └── routes.py
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── result.html
│   ├── admin_login.html
│   └── all_users.html
├── static/css/style.css
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

## Windows / VS Code setup

1. Install Python 3.11+.
2. Open this folder in VS Code.
3. Open **Terminal → New Terminal**.
4. Create a virtual environment:

```powershell
py -m venv .venv
```

5. Activate it:

```powershell
.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, use Command Prompt:

```cmd
.venv\Scripts\activate
```

6. Install dependencies:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

7. Copy `.env.example` to `.env`.

For a first test, leave:

```env
DEMO_MODE=true
```

This allows the entire application to run without an API key.

For real Gemini generation, set:

```env
GEMINI_API_KEY=your_real_key
DEMO_MODE=false
```

8. Start the server:

```powershell
uvicorn app.main:app --reload
```

9. Open:

- App: http://127.0.0.1:8000
- Swagger API docs: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc
- Admin login: http://127.0.0.1:8000/admin/login

## Admin

Admin credentials are configurable with `ADMIN_USERNAME` and `ADMIN_PASSWORD`.
The built-in defaults are for local development only. Set unique, strong values
in the deployment provider before making the app public.

## Deploy publicly (Render + Neon)

FitBuddy needs a Python web service and a database. Render can run the FastAPI
app, while a Neon PostgreSQL database keeps user plans persistent across
restarts and deployments. Do not use the app's local SQLite database for a
public Render deployment: the free web-service filesystem is temporary.

1. Push this project to a GitHub repository you control.
2. Create a PostgreSQL project on Neon and copy its connection string. Keep it
   private. For SQLAlchemy, use the `postgresql+psycopg://` scheme and retain
   Neon’s `sslmode=require` parameter.
3. In Render, create a **New Web Service** and connect the GitHub repository.
   Set:
   - Runtime: **Python**
   - Build command: `pip install -r requirements.txt`
   - Start command: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
4. Add these environment variables in Render's service settings:
   - `DATABASE_URL`: the Neon connection string, using the
     `postgresql+psycopg://` scheme
   - `DEMO_MODE`: `true` to try the site without Gemini, or `false` when a key
     is configured
   - `GEMINI_API_KEY`: a Gemini API key only if `DEMO_MODE=false`
   - `SESSION_SECRET`: a long, random secret generated for this deployment
   - `SESSION_COOKIE_SECURE`: `true`
   - `ADMIN_USERNAME` and `ADMIN_PASSWORD`: unique, strong deployment
     credentials (do not use the development defaults)
5. Deploy the service. Render will provide an HTTPS URL; open it and check
   `/health` to verify the app is running.

Never put real keys or passwords in GitHub, this README, or `.env.example`.
Use the hosting provider's environment-variable settings for deployment
secrets. If a real key has ever been committed or shared, revoke it and create
a replacement before publishing.

## API examples

### Generate a plan

POST `/api/generate-workout`

```json
{
  "username": "Alex",
  "user_id": "alex01",
  "age": 21,
  "weight": 70,
  "goal": "muscle gain",
  "intensity": "medium"
}
```

### Get a saved plan

GET `/api/users/alex01`

### Submit feedback

POST `/api/submit-feedback`

```json
{
  "user_id": "alex01",
  "feedback": "Add more cardio and keep two recovery days."
}
```

## Testing

Run:

```powershell
python -m compileall -q app tests
pytest
```

The repository includes basic endpoint/database tests. Demo mode keeps tests independent of the Gemini service.

There is no separate frontend build step: FastAPI serves the Jinja2 templates
and static CSS directly. For a local production-style server, run:

```powershell
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

## Gemini model note

The supplied documentation names Gemini 1.5 Pro and Gemini Flash and uses the older `google-generativeai` package. This implementation uses Google's current `google-genai` SDK and configurable model IDs. The defaults are `gemini-2.5-pro` for plan generation/update and `gemini-2.5-flash` for quick nutrition/recovery tips.

## Safety

FitBuddy is a general wellness application, not a medical service. The prompts explicitly avoid diagnosis/treatment and advise users with medical conditions, injuries, pregnancy, or exercise restrictions to consult an appropriate healthcare professional.
