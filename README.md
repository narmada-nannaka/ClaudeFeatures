# Claude API Features — Jupyter Notebook Demos

A hands-on demonstration suite for key Claude API capabilities, implemented as Jupyter notebooks. Each notebook is self-contained and focuses on a single feature.

## Features Covered

| Notebook | Feature | Description |
|---|---|---|
| `001_thinking_complete.ipynb` | Extended Thinking | Surface Claude's internal reasoning before a final answer |
| `002_citations.ipynb` | Citations | Get source-attributed answers grounded in a document |
| `002_images.ipynb` | Vision / Image Analysis | Analyze satellite images for structured risk assessment |
| `002_pdfsupport.ipynb` | PDF Processing | Parse and summarize PDF documents directly |
| `003_caching.ipynb` | Prompt Caching + Tool Use | Cache large system prompts and tool definitions to cut costs |
| `005_code_execution.ipynb` | Code Execution + Files API | Execute Python in a sandbox, upload/download files |

---

## Prerequisites

- Python 3.9+
- Jupyter Lab or Jupyter Notebook
- An [Anthropic API key](https://console.anthropic.com/)

---

## Setup

**1. Clone the repository**

```bash
git clone https://github.com/<your-username>/ClaudeFeatures.git
cd ClaudeFeatures
```

**2. Create and activate a virtual environment**

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

**3. Install dependencies**

```bash
pip install anthropic python-dotenv jupyter
```

**4. Configure your API key**

Copy the `.env` file template and add your key:

```bash
# .env
ANTHROPIC_API_KEY=your_api_key_here
```

> **Note:** `.env` is listed in `.gitignore` and will never be committed.

**5. Launch Jupyter**

```bash
jupyter lab
```

---

## Notebook Details

### `001_thinking_complete.ipynb` — Extended Thinking

Enables Claude's internal reasoning (`ThinkingBlock`) before it produces a final `TextBlock` answer. Useful for complex problems where showing work matters.

**Key API parameters:**
```python
thinking={"type": "enabled", "budget_tokens": 10000}
```

**What to look for:** The response object contains both a `ThinkingBlock` (redacted reasoning trace) and a `TextBlock` (final answer).

---

### `002_citations.ipynb` — Citations

Passes an Earth article as a document and asks a factual question. Claude returns its answer with inline citations — each one carries the exact quoted text, document title, and character offsets.

**Key API parameters:**
```python
{"type": "document", "source": {...}, "title": "...", "citations": {"enabled": True}}
```

**What to look for:** `CitationCharLocation` objects in the response with `cited_text`, `start_char_index`, and `end_char_index`.

---

### `002_images.ipynb` — Vision / Image Analysis

Sends a base64-encoded satellite image (`images/prop7.png`) to Claude with a structured fire risk assessment prompt. Claude identifies the residence, evaluates tree overhang and defensible space, and returns a 1–4 risk rating.

**Key API parameters:**
```python
{"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "<base64>"}}
```

**What to look for:** Structured property analysis in plain text with an explicit risk score.

---

### `002_pdfsupport.ipynb` — PDF Processing

Encodes `earth.pdf` as base64 and asks Claude to summarize it. Demonstrates that Claude can directly parse PDF content without a separate extraction step.

**Key API parameters:**
```python
{"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": "<base64>"}}
```

**What to look for:** A one-sentence summary produced from the raw PDF bytes.

---

### `003_caching.ipynb` — Prompt Caching + Tool Use

Wraps a large JavaScript-generation system prompt (~6k tokens) and four tool definitions with `cache_control` so they are reused across turns. Reduces cost significantly for multi-turn sessions with large, static context.

**Key API parameters:**
```python
# On the last tool definition and on the system prompt block:
"cache_control": {"type": "ephemeral"}
```

**What to look for:** `usage.cache_creation_input_tokens` on the first call, then `usage.cache_read_input_tokens` on subsequent calls — the second request costs far fewer tokens.

**Tools defined:**
- `add_duration_to_datetime` — date/time arithmetic
- `set_reminder` — schedule a notification
- `get_current_datetime` — current timestamp
- `db_query` — SQLite query execution

---

### `005_code_execution.ipynb` — Code Execution + Files API

Uploads `streaming.csv` (a customer churn dataset) via the Files API, asks Claude to analyze it and generate a visualization, then downloads the resulting chart and report.

**Beta headers required:**
```python
"anthropic-beta": "code-execution-2025-08-25, files-api-2025-04-14"
```

**Workflow:**
1. Upload CSV → get a `file_id`
2. Pass `file_id` in the message with a `code_execution_20250825` tool
3. Claude writes and runs Python in a sandboxed environment
4. Download the output chart (`churn_analysis_dashboard.png`) and report via `file_id`

**What to look for:** The notebook demonstrates the full lifecycle — upload, execute, retrieve — and shows how each code execution block starts with a clean state (no persistent variables between blocks).

---

## Project Structure

```
ClaudeFeatures/
├── .env                          # API key (not committed)
├── .gitignore
├── LICENSE                       # MIT
├── README.md
├── 001_thinking_complete.ipynb
├── 002_citations.ipynb
├── 002_images.ipynb
├── 002_pdfsupport.ipynb
├── 003_caching.ipynb
├── 005_code_execution.ipynb
├── earth.pdf                     # Sample document (citations + PDF demos)
├── streaming.csv                 # Sample churn dataset (code execution demo)
├── churn_analysis_dashboard.png  # Output chart from code execution demo
└── images/
    ├── prop1.png
    ├── ...
    └── prop7.png                 # Satellite images (vision demo)
```

---

## Common Patterns

All notebooks follow the same structure:

```python
from dotenv import load_dotenv
import anthropic, os

load_dotenv()
client = anthropic.Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

messages = []

def add_user_message(content):
    messages.append({"role": "user", "content": content})

def chat(**kwargs):
    response = client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=8096,
        messages=messages,
        **kwargs
    )
    messages.append({"role": "assistant", "content": response.content})
    return response
```

Feature-specific parameters (thinking, tools, caching headers) are passed as keyword arguments to `chat()`.

---

## License

MIT © 2026 Narmada Nannaka
