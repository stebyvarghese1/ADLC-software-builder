<div align="center">

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║          █████  ██████  ██      ██████                        ║
║         ██   ██ ██   ██ ██     ██                             ║
║         ███████ ██   ██ ██     ██                             ║
║         ██   ██ ██   ██ ██     ██                             ║
║         ██   ██ ██████  ███████ ██████                        ║
║                                                               ║
║           Agentic Development Lifecycle Builder               ║
║         From idea to deployed software — in one app           ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

**Turn a one-line idea into a fully-structured software project.**  
A native PyQt6 desktop app that runs five specialised AI agents — PM · Architect · Engineer · QA · DevOps —  
each building on the last, each powered by free open-source models on Hugging Face or Ollama.

<br/>

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PyQt6](https://img.shields.io/badge/PyQt6-Desktop_UI-41CD52?style=for-the-badge&logo=qt&logoColor=white)](https://riverbankcomputing.com/software/pyqt/)
[![HuggingFace](https://img.shields.io/badge/Hugging_Face-Free_Tier-FFD21E?style=for-the-badge&logo=huggingface&logoColor=black)](https://huggingface.co)
[![Ollama](https://img.shields.io/badge/Ollama-Offline_Mode-black?style=for-the-badge)](https://ollama.ai)
[![License](https://img.shields.io/badge/License-GPL_3.0-blue?style=for-the-badge)](LICENSE)
[![FOSS](https://img.shields.io/badge/Stack-100%25_FOSS-brightgreen?style=for-the-badge)](https://opensource.org)
[![Status](https://img.shields.io/badge/Status-Active_Development-orange?style=for-the-badge)]()

<br/>

[**Getting Started**](#installation) · [**How It Works**](#how-it-works) · [**Supported Models**](#supported-models) · [**Roadmap**](#roadmap) · [**Contributing**](#contributing)

</div>

---

## The pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  📋  PRD     │────▶│  🏗  Arch    │────▶│  💻  Code    │────▶│  🧪  Tests   │────▶│  🚀 Deploy   │
│              │     │              │     │              │     │              │     │              │
│ PM Agent     │     │ Architect    │     │ Engineer     │     │ QA Agent     │     │ DevOps       │
│              │     │ Agent        │     │ Agent        │     │              │     │ Agent        │
│ PRD.md       │     │ arch.md      │     │ src/**       │     │ tests/**     │     │ Dockerfile   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
       ▲                   │                    │                    │
       └───────────────────┴────────────────────┴── context flows forward ──────────▶
```

Each agent **reads summaries from every prior phase** before responding. The architect knows your PRD. The engineer knows the architecture. The QA agent knows the code. Nothing falls through the cracks.

---

## Why this exists

Most AI coding tools help you write code. That's one phase out of five.

| What you actually need | What existing tools give you |
|---|---|
| Requirements gathering + structured PRD | ❌ |
| System architecture + tech stack decisions | ❌ |
| Production-quality, multi-file code generation | ⚠️ partial |
| Comprehensive test suites with edge cases | ❌ |
| Dockerfile, CI/CD pipeline, deployment config | ❌ |
| All five phases with shared context between agents | ❌ |
| Everything local, free, and fully open-source | ❌ |

**ADLC Builder fills every gap** — in a single native desktop app that runs entirely on your machine.

---

## Features

<table>
<tr>
<td width="50%">

### Core pipeline
- **5-phase ADLC pipeline** with distinct agent personas per phase
- **Cross-phase context propagation** — each agent receives summaries of all prior phases automatically
- **Multi-language support** — Python, JS, TS, Go, Rust, Java, SQL, Bash, YAML, and more
- **Artifact detection** — code blocks are auto-logged as named files with correct extensions

</td>
<td width="50%">

### Desktop experience
- **Native PyQt6 UI** — no browser, no Electron, no web server; runs on Windows, macOS, Linux
- **Live token streaming** — QThread signals push each token to the UI without freezing
- **Syntax-highlighted code viewer** — Pygments-powered with line numbers and copy button
- **Session persistence** — SQLite saves everything; resume any project after a restart

</td>
</tr>
<tr>
<td width="50%">

### Model flexibility
- **Hugging Face free tier** — use `Qwen2.5-Coder-32B` and `DeepSeek-Coder-V2` at zero cost
- **Ollama offline mode** — fully air-gapped operation with local FOSS models
- **Any OpenAI-compatible endpoint** — swap models without changing any code
- **Per-session model config** — different model per session, never locked in

</td>
<td width="50%">

### Power-user tools
- **Custom prompt editor** — edit system prompts per phase with `{variable}` interpolation
- **Session manager** — browse, resume, rename, and export past sessions
- **ZIP artifact export** — download a full project scaffold preserving folder structure
- **Status & error handling** — rate limit detection, retry logic, offline mode indicator

</td>
</tr>
</table>

---

## How it works

### 1. You describe your idea

Type one sentence or a paragraph. The PRD agent asks clarifying questions and produces a structured `PRD.md` covering problem statement, user personas, feature requirements (MoSCoW), success metrics, and constraints.

### 2. Context flows automatically

```python
# app/agents/context.py
def build_context(phases: list[PhaseState], current_index: int) -> str:
    """Inject summaries of all completed phases into the next agent's system prompt."""
    summaries = []
    for phase in phases[:current_index]:
        if phase.messages:
            last = phase.messages[-1].content
            summaries.append(f"### {PHASES[phase.index].name}\n{last[:600]}")
    return "\n\n".join(summaries)
```

The last response from each completed phase is trimmed and prepended to the next agent's system prompt. The architect never sees a blank canvas — it sees your PRD. The engineer never guesses at the stack — it knows the architecture.

### 3. The worker thread streams every token

```python
# app/workers/hf_worker.py
class HFWorker(QThread):
    token_received = pyqtSignal(str)   # fires for every token chunk
    finished       = pyqtSignal(str)   # fires when the full response is done
    error          = pyqtSignal(str)   # fires on any API or network error

    def run(self) -> None:
        with httpx.Client(timeout=60) as client:
            with client.stream("POST", self.endpoint, json=self.payload,
                               headers=self.headers) as resp:
                for line in resp.iter_lines():
                    if line.startswith("data: "):
                        chunk = json.loads(line[6:])
                        tok = chunk["choices"][0]["delta"].get("content", "")
                        if tok:
                            self.token_received.emit(tok)   # → live chat bubble
        self.finished.emit(self._full)                      # → artifact detection
```

The UI thread never blocks. Every token appears in the chat bubble as it arrives.

### 4. Artifacts are detected and logged

Any response containing a fenced code block is automatically registered as a named artifact. File paths are inferred from the code fence language and phase context (`PRD.md`, `architecture.md`, `app/workers/hf_worker.py`, etc.). All artifacts appear in the sidebar panel and can be opened in the syntax-highlighted viewer or exported as a ZIP.

---

## Installation

### Requirements

- Python **3.11+**
- A free [Hugging Face](https://huggingface.co) account + read token — **or** [Ollama](https://ollama.ai) for local inference

### Quick start

```bash
# 1. Clone
git clone https://github.com/stebyvarghese1/ADLC-software-builder.git
cd ADLC-software-builder

# 2. Install dependencies (uv recommended — falls back to pip)
pip install uv && uv sync
# or: pip install -r requirements.txt

# 3. Configure
cp .env.example .env
# Edit .env → set HF_TOKEN and HF_MODEL

# 4. Launch
python main.py
```

> The API token lives **only in memory** — it is never written to the SQLite database, `.env` file, or any log.

### Using Ollama (fully offline)

```bash
# Pull a model
ollama pull qwen2.5-coder:32b      # best quality
ollama pull deepseek-coder-v2:16b  # strong code, lighter
ollama pull mistral:7b             # fastest, lightest

# Serve
ollama serve

# In the app: Model → Configure → switch to Ollama → endpoint: http://localhost:11434/v1
```

---

## Project structure

```
ADLC-software-builder/
│
├── main.py                        ← QApplication entry point
├── pyproject.toml                 ← Dependencies (uv / pip)
├── .env.example                   ← Environment variable template
│
├── app/
│   ├── agents/                    ── The brain ──────────────────────────────────
│   │   ├── phases.py              ← 5 PhaseConfig dataclasses with system prompts
│   │   ├── composer.py            ← Assembles [system, history, user] message list
│   │   └── context.py             ← Extracts and trims cross-phase summaries
│   │
│   ├── workers/                   ── The async layer ────────────────────────────
│   │   └── hf_worker.py           ← QThread: streams HF/Ollama tokens via signals
│   │
│   ├── models/                    ── The data contracts ─────────────────────────
│   │   ├── message.py             ← Pydantic Message (role, content, timestamp)
│   │   ├── session.py             ← Pydantic Session + PhaseState
│   │   └── artifact.py            ← Pydantic Artifact (filename, content, language)
│   │
│   ├── storage/                   ── The persistence layer ──────────────────────
│   │   ├── db.py                  ← SQLite CRUD — built-in sqlite3, zero deps
│   │   └── export.py              ← ZIP builder preserving src/ folder structure
│   │
│   └── ui/                        ── The interface ──────────────────────────────
│       ├── main_window.py         ← QMainWindow: menubar, splitter, statusbar
│       ├── phase_bar.py           ← QTabBar with done / active / pending states
│       ├── chat_widget.py         ← QScrollArea + dynamic chat bubble widgets
│       ├── artifact_panel.py      ← QListWidget with phase-coloured file icons
│       ├── code_viewer.py         ← QTextBrowser + Pygments HTML rendering
│       ├── config_dialog.py       ← QDialog: model, token, temperature, max_tokens
│       ├── prompt_editor.py       ← QPlainTextEdit per-phase system prompt editor
│       └── session_dialog.py      ← QDialog: browse, open, rename, delete sessions
│
└── tests/
    ├── conftest.py                ← Pytest fixtures and mock HF responses
    ├── test_composer.py           ← Message list assembly tests
    ├── test_context.py            ← Cross-phase context extraction tests
    ├── test_hf_worker.py          ← QThread signal emission tests (pytest-qt)
    └── test_db.py                 ← SQLite round-trip tests
```

---

## ADLC phases in detail

| # | Phase | Agent persona | Output files | System prompt focus |
|---|---|---|---|---|
| 1 | **PRD** | Product Manager | `PRD.md` | Requirements, MoSCoW, user personas, success metrics |
| 2 | **Architecture** | Senior Architect | `architecture.md` | Tech stack, data models, folder structure, API contracts |
| 3 | **Code generation** | Software Engineer | `src/**/*.{py,ts,go,...}` | Production-quality, typed, idiomatic, with error handling |
| 4 | **Testing & QA** | QA Engineer | `tests/**/*_test.*` | Unit, integration, edge cases, mocks, test data |
| 5 | **Deployment** | DevOps Engineer | `Dockerfile`, `docker-compose.yml`, `.github/workflows/ci.yml` | Container, CI/CD, env vars, health checks |

---

## Supported models

Any instruction-tuned model on Hugging Face Hub with an OpenAI-compatible `/v1/chat/completions` endpoint works.

### Hugging Face free tier

| Model | Licence | Best for | Size |
|---|---|---|---|
| `Qwen/Qwen2.5-Coder-32B-Instruct` | Apache-2.0 | All phases — **default** | 32B |
| `deepseek-ai/DeepSeek-Coder-V2-Instruct` | MIT | Code & tests | 16B MoE |
| `mistralai/Mistral-7B-Instruct-v0.3` | Apache-2.0 | PRD & architecture (fast) | 7B |
| `meta-llama/CodeLlama-34b-Instruct-hf` | LGPL-2.0 | Code generation | 34B |

### Ollama (local)

```bash
ollama pull qwen2.5-coder:32b      # best — same model as HF default
ollama pull deepseek-coder-v2:16b  # excellent code quality
ollama pull mistral:7b             # lightest, fastest
```

---

## Technology stack

| Layer | Library | Version | Licence |
|---|---|---|---|
| UI framework | PyQt6 | 6.x | GPL-3.0 |
| Threading | QThread / pyqtSignal | built-in | GPL-3.0 |
| HTTP client | httpx | ≥ 0.27 | BSD-3-Clause |
| Data validation | Pydantic | v2 | MIT |
| Session storage | sqlite3 | built-in | Public domain |
| Config | python-dotenv | ≥ 1.0 | BSD-3-Clause |
| Markdown rendering | markdown | ≥ 3.6 | BSD-3-Clause |
| Syntax highlighting | Pygments | ≥ 2.18 | BSD-2-Clause |
| JSON serialisation | orjson | ≥ 3.10 | Apache-2.0 |
| Testing | pytest + pytest-qt | latest | MIT |
| Packaging | uv | latest | MIT |
| Distribution | PyInstaller | ≥ 6 | GPL-2.0 |

> **Commercial licensing:** PyQt6 is GPL-3. For proprietary distribution, replace `pyqt6` with `pyside6` in `pyproject.toml` — PySide6 is LGPL-3 and API-identical. No other changes required.

---

## Qt signals & slots wiring

```
HFWorker ──token_received(str)──▶ ChatWidget.append_token()
         ──finished(str)────────▶ ArtifactDetector.scan()
         ──finished(str)────────▶ PhaseState.update()
         ──error(str)───────────▶ StatusBar.show_error()

PhaseBar ──phase_changed(int)───▶ ChatWidget.load_phase()
         ──phase_changed(int)───▶ ContextBuilder.rebuild()
         ──phase_advanced(int)──▶ PhaseState.mark_done()

ArtifactPanel ──artifact_selected(Artifact)──▶ CodeViewer.display()
ConfigDialog  ──config_saved(AppConfig)──────▶ HFWorker.update_config()
```

---

## Data models

```python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Literal
from uuid import uuid4

class Message(BaseModel):
    role:      Literal["user", "assistant", "system"]
    content:   str
    timestamp: datetime = Field(default_factory=datetime.utcnow)

class PhaseState(BaseModel):
    index:    int                                # 0–4
    status:   Literal["pending", "active", "done"]
    messages: list[Message] = []

class Artifact(BaseModel):
    phase_index: int
    filename:    str                             # e.g. "app/workers/hf_worker.py"
    content:     str
    language:    str | None = None               # detected from code fence

class Session(BaseModel):
    session_id: str   = Field(default_factory=lambda: str(uuid4()))
    name:       str
    phases:     list[PhaseState]                 # always length 5
    artifacts:  list[Artifact] = []
    hf_model:   str
    created_at: datetime = Field(default_factory=datetime.utcnow)

class AppConfig(BaseModel):
    backend:         Literal["huggingface", "ollama"] = "huggingface"
    hf_model:        str   = "Qwen/Qwen2.5-Coder-32B-Instruct"
    hf_token:        str   = ""                  # memory only — never persisted
    ollama_endpoint: str   = "http://localhost:11434/v1"
    max_tokens:      int   = 1200
    temperature:     float = 0.4
```

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `HF_TOKEN` | _(required for HF mode)_ | Hugging Face API read token |
| `HF_MODEL` | `Qwen/Qwen2.5-Coder-32B-Instruct` | Default model ID |
| `HF_MAX_TOKENS` | `1200` | Max tokens per completion |
| `HF_TEMPERATURE` | `0.4` | Sampling temperature (0.0–1.0) |
| `OLLAMA_ENDPOINT` | `http://localhost:11434/v1` | Ollama server base URL |
| `DB_PATH` | `~/.adlc_builder/sessions.db` | SQLite database location |
| `LOG_LEVEL` | `WARNING` | Python logging level |

All settings are also configurable at runtime via **Model → Configure** in the app.  
The API token is **never** written to disk regardless of how it is provided.

---

## Running tests

```bash
# Full suite
pytest

# Verbose with live output
pytest -v -s

# Single module
pytest tests/test_hf_worker.py

# With HTML coverage report
pytest --cov=app --cov-report=term-missing --cov-report=html

# Qt UI tests only
pytest tests/ -k "qt" --qt-app
```

Expected output on a clean run:

```
tests/test_composer.py   ....   4 passed
tests/test_context.py    ......  6 passed
tests/test_hf_worker.py  .....   5 passed
tests/test_db.py         .....   5 passed
──────────────────────────────────────────
20 passed in 1.84s
```

---

## Building a standalone binary

```bash
pip install pyinstaller

# macOS / Linux
pyinstaller --onefile --windowed \
            --name "ADLC Builder" \
            --add-data "app/ui/styles:styles" \
            main.py

# Windows
pyinstaller --onefile --windowed --noconsole ^
            --name "ADLC Builder" ^
            --icon assets/icon.ico ^
            main.py
```

| Platform | Output |
|---|---|
| Windows | `dist/ADLC Builder.exe` |
| macOS | `dist/ADLC Builder.app` |
| Linux | `dist/ADLC Builder` |

---

## Contributing

Contributions are welcome — bug fixes, new features, documentation improvements, additional test coverage.

```bash
# 1. Fork and clone
git clone https://github.com/YOUR_USERNAME/ADLC-software-builder.git
cd ADLC-software-builder

# 2. Create a branch
git checkout -b feature/your-feature-name

# 3. Make changes and run quality checks
ruff format .       # format
ruff check .        # lint
mypy app/           # type check
pytest              # tests

# 4. Push and open a pull request
git push origin feature/your-feature-name
```

For significant changes, open an issue first to discuss the approach.  
For bug reports, please include your OS, Python version, and full traceback.

---

## Roadmap

```
v1.0  ████████████████████  Core 5-phase pipeline · PyQt6 UI · HF + Ollama  ← you are here
v1.1  ░░░░░░░░░░░░░░░░░░░░  Dark / light theme toggle · custom prompt editor
v1.2  ░░░░░░░░░░░░░░░░░░░░  ZIP export · session JSON import / export
v2.0  ░░░░░░░░░░░░░░░░░░░░  Multi-model routing — different model per phase
v2.1  ░░░░░░░░░░░░░░░░░░░░  Git integration — commit artifacts directly to a repo
v2.2  ░░░░░░░░░░░░░░░░░░░░  In-app code execution sandbox (restricted subprocess)
v3.0  ░░░░░░░░░░░░░░░░░░░░  Team mode — shared sessions over local network
```

---

## Licence

Licensed under the **GNU General Public License v3.0** — see [LICENSE](LICENSE) for the full text.

**TL;DR:** Use, study, modify, and distribute freely. Modified distributions must also be GPL-3.  
For commercial / proprietary distribution, replace `pyqt6` with `pyside6` (LGPL-3, drop-in compatible).

---

## Acknowledgements

- [Hugging Face](https://huggingface.co) — free-tier inference API
- [Ollama](https://ollama.ai) — local model serving
- [Qwen team @ Alibaba Cloud](https://github.com/QwenLM/Qwen2.5-Coder) — Qwen2.5-Coder series
- [DeepSeek](https://github.com/deepseek-ai) — DeepSeek-Coder-V2
- [Riverbank Computing](https://riverbankcomputing.com) — PyQt6
- Every open-source maintainer whose library appears in this stack

---

<div align="center">

**No proprietary APIs · No usage fees · No telemetry · No lock-in**

Built with Python · PyQt6 · open-source LLMs

[⭐ Star this repo](https://github.com/stebyvarghese1/ADLC-software-builder) &nbsp;·&nbsp; [🐛 Report a bug](https://github.com/stebyvarghese1/ADLC-software-builder/issues) &nbsp;·&nbsp; [💡 Request a feature](https://github.com/stebyvarghese1/ADLC-software-builder/issues)

</div>
