# ADLC Software Builder

> A native desktop application that guides you from a raw idea to deployed software through five AI-powered phases — entirely free and open-source.

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)
![PyQt6](https://img.shields.io/badge/PyQt6-6.x-41CD52?style=flat-square&logo=qt&logoColor=white)
![License](https://img.shields.io/badge/License-GPL--3.0-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-In%20Development-orange?style=flat-square)
![FOSS](https://img.shields.io/badge/Stack-100%25%20FOSS-brightgreen?style=flat-square)

---

## What is it?

The **ADLC Software Builder** is a native PyQt6 desktop application that orchestrates an AI agent through every phase of the software development lifecycle — from gathering requirements to writing deployment configs. Each phase has a dedicated agent persona, its own system prompt, and carries full context from all prior phases.

You bring a project idea. The agent delivers a PRD, architecture document, production code, test suites, and a deployment setup — all in one session, all on your machine, with no cloud vendor lock-in.

```
Idea → PRD → Architecture → Code → Tests → Deployment
```

---

## Features

- **5-phase ADLC pipeline** — PM, Architect, Engineer, QA, and DevOps agent personas
- **Native PyQt6 desktop UI** — runs on Windows, macOS, and Linux with no browser or server
- **Live token streaming** — responses appear token-by-token via `QThread` signals; the UI never freezes
- **Artifact panel** — every generated file is tracked, named, and viewable with syntax highlighting
- **Session persistence** — conversations and artifacts are saved to a local SQLite database and survive restarts
- **Cross-phase context** — each phase automatically receives a summary of all previous phases
- **Multi-language code generation** — Python, JavaScript, TypeScript, Go, Rust, Java, SQL, Bash, YAML
- **Syntax-highlighted code viewer** — powered by Pygments with line numbers
- **Custom prompt editor** — edit system prompts per phase with variable interpolation
- **Offline mode** — swap Hugging Face for a local Ollama instance for fully air-gapped operation
- **ZIP export** — download all session artifacts as a structured zip file
- **100% FOSS** — every dependency has an OSI-approved licence; no telemetry, no paywalls

---

## Screenshots

| Main window | Code viewer | Config dialog |
|---|---|---|
| Phase tabs, chat, artifact sidebar | Pygments-highlighted code | Model + token settings |

| Session manager | Prompt editor | Export dialog |
|---|---|---|
| Browse and resume past sessions | Customise per-phase prompts | Select and download artifacts |

---

## Requirements

- Python **3.11** or higher
- A free [Hugging Face](https://huggingface.co) account and API token
- **OR** [Ollama](https://ollama.ai) running locally for fully offline use

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/adlc-builder.git
cd adlc-builder
```

### 2. Install dependencies

Using [uv](https://github.com/astral-sh/uv) (recommended):

```bash
pip install uv
uv sync
```

Using pip:

```bash
pip install -r requirements.txt
```

### 3. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and set your Hugging Face token:

```env
HF_TOKEN=hf_your_token_here
HF_MODEL=Qwen/Qwen2.5-Coder-32B-Instruct
```

> **Note:** The API token is stored only in memory during the session. It is never written to the SQLite database or any file on disk.

### 4. Run the app

```bash
python main.py
```

---

## Using Ollama (offline mode)

If you prefer fully local inference with no external API calls:

```bash
# Install Ollama from https://ollama.ai
ollama pull qwen2.5-coder:32b

# Start Ollama server
ollama serve
```

Then in the app, open **Model → Configure**, switch the backend to **Ollama (local)**, and set the endpoint to `http://localhost:11434/v1`.

---

## Project structure

```
adlc_builder/
├── main.py                      ← QApplication entry point
├── pyproject.toml               ← Project metadata and dependencies
├── .env.example                 ← Environment variable template
│
├── app/
│   ├── agents/
│   │   ├── phases.py            ← Phase configs and agent personas
│   │   ├── composer.py          ← Builds the messages list for each API call
│   │   └── context.py           ← Extracts and summarises cross-phase context
│   │
│   ├── workers/
│   │   └── hf_worker.py         ← QThread worker for non-blocking HF API calls
│   │
│   ├── models/
│   │   ├── message.py           ← Pydantic Message model
│   │   ├── session.py           ← Pydantic Session and PhaseState models
│   │   └── artifact.py          ← Pydantic Artifact model
│   │
│   ├── storage/
│   │   ├── db.py                ← SQLite read/write (built-in sqlite3)
│   │   └── export.py            ← ZIP artifact export
│   │
│   └── ui/
│       ├── main_window.py       ← QMainWindow shell
│       ├── phase_bar.py         ← QTabBar phase navigator
│       ├── chat_widget.py       ← QScrollArea with chat bubble widgets
│       ├── artifact_panel.py    ← QListWidget artifact sidebar
│       ├── code_viewer.py       ← QTextBrowser with Pygments syntax highlighting
│       ├── config_dialog.py     ← Model and token configuration QDialog
│       ├── prompt_editor.py     ← Per-phase system prompt editor
│       └── session_dialog.py    ← Session manager QDialog
│
└── tests/
    ├── conftest.py
    ├── test_composer.py
    ├── test_context.py
    ├── test_hf_worker.py
    └── test_db.py
```

---

## ADLC phases

Each phase has a dedicated agent persona. Context from completed phases is automatically injected into subsequent ones.

| Phase | Agent | Output | Key files |
|---|---|---|---|
| **1 — PRD** | Product Manager | `PRD.md` | `phases.py` phase 0 config |
| **2 — Architecture** | Senior Architect | `architecture.md` | `phases.py` phase 1 config |
| **3 — Code generation** | Software Engineer | Source files in `src/` | `hf_worker.py`, `composer.py` |
| **4 — Testing** | QA Engineer | Test files in `tests/` | `phases.py` phase 3 config |
| **5 — Deployment** | DevOps Engineer | `Dockerfile`, `docker-compose.yml`, CI config | `phases.py` phase 4 config |

### How cross-phase context works

```python
# app/agents/context.py — simplified

def build_context(phases: list[PhaseState], current_index: int) -> str:
    summaries = []
    for phase in phases[:current_index]:
        if phase.messages:
            last_response = phase.messages[-1].content
            summaries.append(f"### {PHASES[phase.index].name}\n{last_response[:600]}")
    return "\n\n".join(summaries)
```

The last assistant message from each completed phase is trimmed to 600 characters and injected into the next phase's system prompt — so the architect knows the PRD, the engineer knows the architecture, and the QA agent knows the code.

---

## Supported models

Any instruction-tuned model on Hugging Face Hub that exposes `/v1/chat/completions` will work. Recommended free-tier models:

| Model | Licence | Strengths |
|---|---|---|
| `Qwen/Qwen2.5-Coder-32B-Instruct` | Apache-2.0 | Best overall coding quality — **default** |
| `deepseek-ai/DeepSeek-Coder-V2-Instruct` | MIT | Strong multilanguage generation |
| `mistralai/Mistral-7B-Instruct-v0.3` | Apache-2.0 | Fast, good for PRD and architecture phases |
| `meta-llama/CodeLlama-34b-Instruct-hf` | LGPL-2.0 | Meta's open code model |

For Ollama, any model pulled via `ollama pull <name>` is supported.

---

## Technology stack

| Layer | Library | Licence |
|---|---|---|
| UI framework | PyQt6 | GPL-3.0 |
| Background threads | QThread / QRunnable | GPL-3.0 |
| HTTP client | httpx | BSD-3-Clause |
| Data validation | Pydantic v2 | MIT |
| Session persistence | SQLite (built-in) | Built-in |
| Configuration | python-dotenv | BSD-3-Clause |
| Markdown rendering | markdown | BSD-3-Clause |
| Syntax highlighting | Pygments | BSD-2-Clause |
| JSON serialisation | orjson | Apache-2.0 |
| Testing | pytest + pytest-qt | MIT |
| Packaging | uv | MIT |
| Distribution | PyInstaller | GPL-2.0 |

> **Commercial use:** PyQt6 is GPL-3 licensed. For commercial/proprietary distributions, replace `PyQt6` with `PySide6` (LGPL-3) — the API is identical and no other code changes are required.

---

## Running tests

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run a specific module
pytest tests/test_composer.py

# Run with coverage
pytest --cov=app --cov-report=term-missing
```

---

## Building a standalone executable

Using PyInstaller to produce a single-file binary:

```bash
# Install PyInstaller
pip install pyinstaller

# Build
pyinstaller --onefile --windowed --name "ADLC Builder" main.py

# Output will be in dist/
```

Platform-specific notes:
- **Windows:** produces `dist/ADLC Builder.exe`
- **macOS:** produces `dist/ADLC Builder.app`
- **Linux:** produces `dist/ADLC Builder`

---

## Qt signals and slots reference

The key wiring between the UI and the worker thread:

```python
# Connecting the HF worker to the chat widget
worker = HFWorker(messages=messages, config=config)
worker.token_received.connect(chat_widget.append_token)   # streams each token
worker.finished.connect(chat_widget.on_response_complete) # triggers artifact check
worker.error.connect(status_bar.show_error)               # surfaces API errors
worker.start()
```

| Signal | Source | Connected to |
|---|---|---|
| `HFWorker.token_received(str)` | Worker thread | Chat bubble — appends token live |
| `HFWorker.finished(str)` | Worker thread | Artifact detector, phase state update |
| `HFWorker.error(str)` | Worker thread | Status bar error message |
| `PhaseBar.phase_changed(int)` | Phase tab click | Chat widget swap, context rebuild |
| `PhaseBar.phase_advanced(int)` | Next button | Mark phase done, unlock next |
| `ArtifactPanel.artifact_selected(Artifact)` | List click | Code viewer opens file |
| `ConfigDialog.config_saved(AppConfig)` | Save button | Updates global AppConfig |

---

## Data models

```python
# app/models/message.py
class Message(BaseModel):
    role: Literal["user", "assistant", "system"]
    content: str
    timestamp: datetime = Field(default_factory=datetime.utcnow)

# app/models/session.py
class PhaseState(BaseModel):
    index: int                          # 0–4
    status: Literal["pending", "active", "done"]
    messages: list[Message] = []

class Session(BaseModel):
    session_id: str = Field(default_factory=lambda: str(uuid4()))
    name: str
    phases: list[PhaseState]            # always length 5
    artifacts: list[Artifact] = []
    hf_model: str
    created_at: datetime = Field(default_factory=datetime.utcnow)

# app/models/artifact.py
class Artifact(BaseModel):
    phase_index: int
    filename: str                       # e.g. "app/workers/hf_worker.py"
    content: str
    language: str | None = None         # detected from code fence
```

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `HF_TOKEN` | — | Hugging Face API token (required for HF mode) |
| `HF_MODEL` | `Qwen/Qwen2.5-Coder-32B-Instruct` | Default model ID |
| `HF_MAX_TOKENS` | `1200` | Max tokens per API response |
| `HF_TEMPERATURE` | `0.4` | Sampling temperature |
| `OLLAMA_ENDPOINT` | `http://localhost:11434/v1` | Ollama base URL |
| `DB_PATH` | `~/.adlc_builder/sessions.db` | SQLite database path |

All variables can also be configured at runtime via the in-app Config dialog. The API token is **never** persisted to the database regardless of the source.

---

## Contributing

Contributions are welcome. Please follow these steps:

1. Fork the repository and create a feature branch
2. Make your changes with clear commit messages
3. Add or update tests for any changed behaviour
4. Run `pytest` and confirm all tests pass
5. Open a pull request with a description of the change

For large changes, please open an issue first to discuss the approach.

### Code style

```bash
# Format
ruff format .

# Lint
ruff check .

# Type check
mypy app/
```

---

## Roadmap

- [ ] v1.0 — core 5-phase pipeline, PyQt6 UI, HF + Ollama support
- [ ] v1.1 — dark/light theme toggle, custom prompt editor
- [ ] v1.2 — session JSON export/import, ZIP artifact download
- [ ] v2.0 — multi-model routing per phase (different model per agent)
- [ ] v2.1 — Git integration (commit generated files directly to a repo)
- [ ] v2.2 — in-app code execution sandbox (restricted subprocess)

---

## Licence

This project is licensed under the **GNU General Public License v3.0**.

See [LICENSE](LICENSE) for the full text.

> **Note on PyQt6:** PyQt6 is GPL-3 licensed. If you need to distribute this application under a commercial or proprietary licence, replace `pyqt6` with `pyside6` in `pyproject.toml` — PySide6 is LGPL-3 and otherwise API-compatible.

---

## Acknowledgements

- [Hugging Face](https://huggingface.co) for free-tier model inference
- [Ollama](https://ollama.ai) for local model serving
- [Qwen team](https://github.com/QwenLM/Qwen2.5-Coder) for the Qwen2.5-Coder model series
- [Riverbank Computing](https://riverbankcomputing.com) for PyQt6
- The broader open-source Python ecosystem

---

*Built with Python, PyQt6, and open-source LLMs. No proprietary APIs. No usage fees. No telemetry.*
