

crtae project in jupiter python notebook 

# Local AI Code Assistant — Project Plan
### Powered by Qwen 2.5 (local) + Optional Gemini API · Streamlit Planner UI

---

## 1. Overview

A local-first AI coding assistant that:
- Runs **Qwen 2.5** locally (via Ollama or llama.cpp) for code generation, explanation, and debugging — no internet required for core use.
- Optionally calls **Gemini API** for tasks that benefit from a larger cloud model (configured via `.env`).
- Provides a **Streamlit** front-end that doubles as a **project/task planner** — you describe a feature, the assistant breaks it into steps, generates code, and tracks progress.

---

## 2. Tech Stack

| Layer | Tool |
|---|---|
| Local LLM | Qwen 2.5 (7B/14B) via **Ollama** |
| Cloud LLM (optional) | Gemini API (`google-generativeai`) |
| UI | Streamlit |
| Config | `python-dotenv` + `.env` |
| Storage | SQLite (local task/plan history) |
| Language | Python 3.10+ |

---

## 3. Folder Structure

```
ai-code-assistant/
├── .env                      # API keys & config (never commit)
├── .env.example               # template for .env
├── .gitignore
├── requirements.txt
├── README.md
├── app.py                     # Streamlit entry point
├── config.py                  # loads .env, central settings
├── core/
│   ├── __init__.py
│   ├── qwen_client.py          # wraps Ollama/Qwen 2.5 calls
│   ├── gemini_client.py        # wraps Gemini API calls
│   ├── router.py               # picks local vs cloud model
│   └── prompts.py              # prompt templates
├── planner/
│   ├── __init__.py
│   ├── task_breaker.py         # splits a feature request into steps
│   ├── code_generator.py       # generates code per step
│   └── history_db.py           # SQLite read/write for saved plans
├── ui/
│   ├── sidebar.py               # model selector, settings
│   ├── planner_view.py          # main planning UI
│   └── code_view.py             # code output + download panel
├── data/
│   └── history.db               # created at runtime
└── tests/
    ├── test_qwen_client.py
    ├── test_planner.py
    └── test_router.py
```

---

## 4. `.env` File

```env
# --- Local Qwen 2.5 (via Ollama) ---
OLLAMA_HOST=http://localhost:11434
QWEN_MODEL=qwen2.5:7b

# --- Optional Gemini API ---
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-1.5-flash

# --- App settings ---
DEFAULT_ENGINE=local          # local | gemini
APP_TITLE=AI Code Planner
```

Load it in `config.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv()

OLLAMA_HOST = os.getenv("OLLAMA_HOST", "http://localhost:11434")
QWEN_MODEL = os.getenv("QWEN_MODEL", "qwen2.5:7b")
GEMINI_API_KEY = os.getenv("GEMINI_API_KEY")
GEMINI_MODEL = os.getenv("GEMINI_MODEL", "gemini-1.5-flash")
DEFAULT_ENGINE = os.getenv("DEFAULT_ENGINE", "local")
```

---

## 5. Core Components

### 5.1 Qwen 2.5 Client (local, via Ollama)
- Runs `qwen2.5` model locally through Ollama's REST API (`/api/generate` or `/api/chat`).
- No API key needed — fully offline once the model is pulled (`ollama pull qwen2.5`).

### 5.2 Gemini Client (optional cloud fallback)
- Used only when `DEFAULT_ENGINE=gemini` or the user toggles it in the sidebar.
- Reads `GEMINI_API_KEY` from `.env`.

### 5.3 Router
- Single interface `generate(prompt, engine="local")` that dispatches to Qwen or Gemini transparently, so the rest of the app doesn't care which model answers.

### 5.4 Planner
- `task_breaker.py`: takes a feature description → returns an ordered list of subtasks (uses the LLM with a "break this into steps" prompt).
- `code_generator.py`: for each subtask, generates code + explanation.
- `history_db.py`: saves each plan/session to SQLite so past plans can be reopened.

---

## 6. Streamlit App Flow (`app.py`)

1. **Sidebar** — choose engine (Local Qwen 2.5 / Gemini), model params (temperature, max tokens), and view saved plan history.
2. **Input box** — user describes what they want to build (e.g., "REST API for a todo app with auth").
3. **Plan generation** — app calls `task_breaker.py`, displays a numbered task list (editable checklist).
4. **Code generation per task** — click a task → generates code in an expandable code block with syntax highlighting.
5. **Export** — download full plan + code as a single `.md` file, or download individual code files.
6. **History** — sidebar lists past sessions pulled from SQLite; click to reload.

---

## 7. Setup Instructions

```bash
# 1. Install Ollama and pull Qwen 2.5
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen2.5:7b

# 2. Clone/setup project
git clone <your-repo>
cd ai-code-assistant
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
# edit .env and add GEMINI_API_KEY if you want cloud fallback

# 4. Run
streamlit run app.py
```

`requirements.txt`:
```
streamlit
python-dotenv
requests
google-generativeai
```

---

## 8. Roadmap

| Phase | Goal |
|---|---|
| 1 | Local Qwen 2.5 chat working via Ollama + basic Streamlit chat UI |
| 2 | Task breaker + code generator (planner core) |
| 3 | SQLite history + session reload |
| 4 | Gemini fallback integration via `.env` toggle |
| 5 | Export plan + code as downloadable `.md` |
| 6 | (Stretch) File-aware mode — point it at a real repo for context-aware suggestions |

---

## 9. Notes

- Keep `.env` out of version control (`.gitignore` should include it).
- Qwen 2.5 runs fully offline once pulled — good for privacy-sensitive code.
- Gemini is optional and only activates when a key is present and selected.
