# GraphRAG Project — Complete Run Guide

This guide walks you through running the GraphRAG project from scratch on Windows (also works on macOS/Linux with small command changes).

---

## What This Project Does

The project builds a **Graph RAG** (Retrieval-Augmented Generation) system:

1. Loads Wikipedia articles about a topic (default: **Imran Khan**)
2. Splits text into chunks
3. Uses **Ollama** (local LLM) to extract entities and relationships
4. Stores the knowledge graph in **Neo4j**
5. Builds hybrid search (graph + vector embeddings)
6. Answers questions using retrieved context + Ollama

**Main file to run:** `backend/GraphRagPractice.ipynb`

**Reference template:** `template.ipynb` (original OpenAI version — not used directly)

**Note:** `main.py` is a PyCharm placeholder and is **not** part of this pipeline.

---

## Prerequisites Checklist

Before you start, you need all of the following:

| Requirement | Required? | Status in your repo |
|-------------|-----------|---------------------|
| Python 3.10–3.12 (3.13 works but may have package conflicts) | Yes | You have Python 3.13 |
| Jupyter / VS Code with notebook support | Yes | Not in `requirements.txt` — install manually |
| Ollama (local LLM server) | Yes | **Not installed** on your machine yet |
| Neo4j Aura cloud database (free tier) | Yes | **Credentials not configured** |
| Internet access | Yes | For Wikipedia + Neo4j cloud |
| ~8 GB RAM minimum | Recommended | For Ollama models |

---

## What Is Missing & How to Get It

### 1. Ollama (local LLM) — **MISSING**

Ollama runs the language model and embeddings on your machine instead of OpenAI.

**How to get it:**

1. Download from [https://ollama.com/download](https://ollama.com/download)
2. Install and open a terminal
3. Verify:

```powershell
ollama --version
```

4. Pull required models (one-time download, ~2–5 GB total):

```powershell
ollama pull llama3.2
ollama pull nomic-embed-text
```

5. Start the server (usually auto-starts after install):

```powershell
ollama serve
```

6. Verify Ollama is running:

```powershell
curl http://localhost:11434/api/tags
```

You should see JSON listing your installed models.

---

### 2. Neo4j Aura Database — **MISSING (credentials)**

The graph is stored in a cloud Neo4j database. You need a free Neo4j Aura instance.

**How to get it:**

1. Go to [https://neo4j.com/cloud/aura/](https://neo4j.com/cloud/aura/)
2. Sign up / log in
3. Click **Create Instance** → choose **AuraDB Free**
4. Save these when shown (you cannot view the password again):
   - **Connection URI** (looks like `neo4j+s://xxxxx.databases.neo4j.io`)
   - **Username** (usually `neo4j`)
   - **Password**

5. Set them as environment variables (PowerShell):

```powershell
$env:NEO4J_URI = "neo4j+s://YOUR_INSTANCE.databases.neo4j.io"
$env:NEO4J_USERNAME = "neo4j"
$env:NEO4J_PASSWORD = "your-actual-password"
```

> **Tip:** To persist across sessions, add these to Windows Environment Variables (System Properties → Environment Variables) or use a `.env` file with a tool like `python-dotenv`.

---

### 3. Python Dependencies — **PARTIALLY MISSING**

Install from the project root:

```powershell
cd D:\projects\GraphRag
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt
```

**Also install these (currently missing from notebook `%pip` cell):**

```powershell
pip install langchain-neo4j jupyter ipykernel
```

| Package | Why you need it |
|---------|-----------------|
| `langchain-neo4j` | `Neo4jVector` moved out of `langchain-community` in newer versions |
| `jupyter` | To run `.ipynb` notebooks |
| `ipykernel` | So VS Code / Jupyter can use your virtual environment |

**Known import fix:** If you see:

```
ImportError: cannot import name 'Neo4jVector' from 'langchain_community.vectorstores'
```

Change the import in the notebook from:

```python
from langchain_community.vectorstores import Neo4jVector
from langchain_community.vectorstores.neo4j_vector import remove_lucene_chars
```

To:

```python
from langchain_neo4j import Neo4jVector
from langchain_neo4j.vectorstores.neo4j_vector import remove_lucene_chars
```

Also add `langchain-neo4j` to `%pip install` in the first notebook cell.

---

### 4. Jupyter Notebook Runtime — **MISSING**

**Option A — VS Code (recommended if you already use Cursor/VS Code):**

1. Install the **Jupyter** extension
2. Open `backend/GraphRagPractice.ipynb`
3. Select kernel: your `.venv` Python interpreter
4. Run cells top to bottom

**Option B — Jupyter in browser:**

```powershell
cd D:\projects\GraphRag
jupyter notebook backend/GraphRagPractice.ipynb
```

---

### 5. Optional: Graph Visualization Widget

The notebook can show an interactive graph using `yfiles_jupyter_graphs`. This is **optional** — the pipeline works without it.

- Already in `requirements.txt`
- If it fails to load, the notebook skips visualization automatically
- Works best in Jupyter; may not render in all VS Code versions

---

## Step-by-Step: Run the Project

### Step 1 — Clone / open the project

```powershell
cd D:\projects\GraphRag
```

### Step 2 — Create and activate virtual environment

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks script execution:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### Step 3 — Install all dependencies

```powershell
pip install --upgrade pip
pip install -r requirements.txt
pip install langchain-neo4j jupyter ipykernel
```

### Step 4 — Install and configure Ollama

```powershell
ollama pull llama3.2
ollama pull nomic-embed-text
```

Verify:

```powershell
ollama list
```

Expected output includes `llama3.2` and `nomic-embed-text`.

### Step 5 — Create Neo4j Aura instance

Follow [Neo4j Aura setup](#2-neo4j-aura-database--missing-credentials) above and set environment variables.

### Step 6 — Set optional environment variables

Defaults are fine for most cases. Override if needed:

```powershell
$env:OLLAMA_BASE_URL = "http://localhost:11434"
$env:OLLAMA_LLM_MODEL = "llama3.2"
$env:OLLAMA_EMBED_MODEL = "nomic-embed-text"
$env:WIKIPEDIA_QUERY = "Imran Khan"
```

### Step 7 — Open the notebook

Open `backend/GraphRagPractice.ipynb` in VS Code or Jupyter.

**Important:** After running the first `%pip install` cell, restart the kernel (Kernel → Restart) before running import cells.

### Step 8 — Run cells in order

Run every cell from top to bottom. Do not skip cells.

| Section | What happens | Expected result |
|---------|--------------|-----------------|
| Install packages | Installs LangChain, Neo4j, etc. | "Note: you may need to restart the kernel" |
| Imports | Loads all libraries | No errors (after `langchain-neo4j` fix) |
| Configuration | Sets Ollama + Neo4j settings | Prints model names and Neo4j URI |
| Connect Neo4j | `Neo4jGraph()` | `Connected to Neo4j` |
| Load Wikipedia | Fetches 3 articles | `Loaded 3 documents` |
| Split text | Chunks documents | `Split into ~7 chunks` |
| Graph extraction | Ollama extracts entities | Takes **several minutes** locally |
| Add to Neo4j | Writes graph to database | `Graph documents added to Neo4j` |
| Visualize (optional) | Shows graph widget | Interactive graph or skip message |
| Vector index | Ollama embeddings in Neo4j | `Hybrid vector index ready` |
| Fulltext index | Entity search index | No error |
| Entity chain | Structured entity extraction | Ready silently |
| Structured retriever | Test graph lookup | Prints relationship strings |
| RAG chain | Builds Q&A pipeline | Ready silently |
| Ask questions | Runs RAG queries | Natural language answers |

### Step 9 — Test with a question

Run the last cells:

```python
chain.invoke({"question": f"Who is {WIKIPEDIA_QUERY}?"})
```

Example follow-up with chat history:

```python
chain.invoke({
    "question": "When did he become prime minister?",
    "chat_history": [
        (f"Who is {WIKIPEDIA_QUERY}?", "Imran Khan is a Pakistani former cricketer and politician."),
    ],
})
```

---

## Quick Verification Commands

Run these **before** opening the notebook to confirm everything is ready:

```powershell
# Python
python --version

# Ollama running
curl http://localhost:11434/api/tags

# Neo4j env vars set
echo $env:NEO4J_URI
echo $env:NEO4J_PASSWORD

# Key imports work
python -c "from langchain_ollama import ChatOllama; from langchain_neo4j import Neo4jVector; print('OK')"
```

---

## Troubleshooting

### `ollama: command not found`

- Ollama is not installed or not in PATH
- Fix: Install from [ollama.com](https://ollama.com) and restart your terminal

### `Connection refused` on port 11434

- Ollama server is not running
- Fix: Run `ollama serve` or restart the Ollama desktop app

### `ImportError: Neo4jVector`

- Missing `langchain-neo4j` package or wrong import path
- Fix: `pip install langchain-neo4j` and update imports (see section 3 above)

### `Authentication failure` / Neo4j connection error

- Wrong URI, username, or password
- Fix: Copy credentials again from Neo4j Aura console → **Connect**

### Wikipedia returns 0 documents or JSON error

- Wikipedia blocks requests without a user-agent (already handled in notebook)
- Fix: Re-run the Wikipedia cell; check internet connection

### Graph extraction is very slow

- Normal for local Ollama — each chunk calls the LLM
- Fix: Keep `load_max_docs=3` (already set); use a smaller/faster model like `llama3.2:1b` for testing

### `with_structured_output` fails on entity extraction

- Some Ollama models do not support structured JSON output well
- Fix: Use `llama3.1` or `llama3.2`; avoid very small models

### Pip dependency conflict warnings

You may see warnings like:

```
langchain-openai requires langchain-core<1.0.0, but you have langchain-core 1.4.8
```

- These are warnings from other installed packages, not always fatal
- Fix if imports fail: create a **fresh virtual environment** and install only from `requirements.txt`

### TensorFlow warning on import

```
WARNING:tensorflow: ...
```

- Comes from an optional dependency chain — safe to ignore

---

## Project Structure

```
GraphRag/
├── backend/
│   └── GraphRagPractice.ipynb   ← Main notebook (run this)
├── template.ipynb               ← Original OpenAI reference template
├── requirements.txt             ← Python dependencies
├── RUN_GUIDE.md                 ← This file
└── main.py                      ← Unused PyCharm sample (ignore)
```

---

## Environment Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `NEO4J_URI` | *(must set)* | Neo4j Aura connection URI |
| `NEO4J_USERNAME` | `neo4j` | Neo4j username |
| `NEO4J_PASSWORD` | *(must set)* | Neo4j password |
| `OLLAMA_BASE_URL` | `http://localhost:11434` | Ollama API URL |
| `OLLAMA_LLM_MODEL` | `llama3.2` | Chat model for extraction + Q&A |
| `OLLAMA_EMBED_MODEL` | `nomic-embed-text` | Embedding model for vector search |
| `WIKIPEDIA_QUERY` | `Imran Khan` | Wikipedia search topic |

---

## Minimum Setup Summary (TL;DR)

```powershell
# 1. Setup Python env
cd D:\projects\GraphRag
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install langchain-neo4j jupyter ipykernel

# 2. Setup Ollama
ollama pull llama3.2
ollama pull nomic-embed-text

# 3. Setup Neo4j (get free Aura instance at neo4j.com/cloud/aura)
$env:NEO4J_URI = "neo4j+s://YOUR_INSTANCE.databases.neo4j.io"
$env:NEO4J_USERNAME = "neo4j"
$env:NEO4J_PASSWORD = "your-password"

# 4. Run notebook
jupyter notebook backend/GraphRagPractice.ipynb
# OR open in VS Code and run all cells
```

Fix the `Neo4jVector` import if needed, restart kernel after `%pip install`, then run all cells top to bottom.

---

## Next Steps After It Works

- Change `WIKIPEDIA_QUERY` to explore other topics
- View your graph in the [Neo4j Aura Browser](https://console.neo4j.io/)
- Try different Ollama models (`mistral`, `llama3.1`, etc.)
- Clear Neo4j data before re-ingesting the same topic (to avoid duplicate nodes)
