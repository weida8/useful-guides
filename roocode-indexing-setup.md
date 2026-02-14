````md
# RooCode Local Indexing Setup (Ollama + Qdrant) — Reproducible Step-by-Step Guide (macOS Intel)

This guide sets up **local codebase indexing** for RooCode using:
- **Ollama** for embeddings
- **Qdrant** for the vector database (via Docker)

Target environment: **macOS (Intel)**  
Outcome: RooCode can scan your repo, create embeddings, store them in Qdrant, and retrieve code context.

---

## Prerequisites

You will install:
- Homebrew (package manager)
- Ollama (local model server)
- Docker Desktop (to run Qdrant)
- Qdrant (vector DB)

You should have:
- Terminal access
- Admin permissions to install apps
- A repo folder to index

---

## Step 1 — Install Homebrew (skip if already installed)

Open **Terminal** and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
````

Verify:

```bash
brew --version
```

---

## Step 2 — Install and Start Ollama

### 2.1 Install Ollama

```bash
brew install ollama
```

### 2.2 Start Ollama server (keep this running)

```bash
ollama serve
```

> Leave this terminal window open.

### 2.3 Pull an embedding model (new terminal tab)

Open a **second Terminal tab** and run:

```bash
ollama pull nomic-embed-text
```

### 2.4 Verify Ollama is reachable

```bash
curl http://localhost:11434/api/tags
```

You should see JSON output listing models.

### 2.5 Verify embeddings endpoint works

```bash
curl http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model":"nomic-embed-text","prompt":"hello world"}'
```

Expected: JSON containing an `"embedding": [...]` array.

---

## Step 3 — Install Docker (required for Qdrant)

### 3.1 Install Docker Desktop

1. Download Docker Desktop for Mac (Intel)
2. Install it
3. Launch Docker Desktop and wait until it shows **Docker is running**

### 3.2 Verify Docker works

```bash
docker --version
docker ps
```

If `docker ps` runs without errors, Docker is ready.

---

## Step 4 — Run Qdrant Locally (Vector Database)

### 4.1 Start Qdrant container

```bash
docker run -d --name qdrant \
  -p 6333:6333 -p 6334:6334 \
  -v qdrant_data:/qdrant/storage \
  qdrant/qdrant
```

### 4.2 Verify Qdrant is reachable

```bash
curl http://localhost:6333
```

Expected: a response indicating Qdrant is running (often includes version info).

### 4.3 If Qdrant fails because the name already exists

Stop/remove the old container and recreate:

```bash
docker stop qdrant
docker rm qdrant
```

Then re-run Step 4.1.

### 4.4 If port 6333 is already in use

Check what’s using the port:

```bash
lsof -i :6333
```

Stop that process or run Qdrant on a different port (example maps host 6335 → container 6333):

```bash
docker run -d --name qdrant \
  -p 6335:6333 -p 6336:6334 \
  -v qdrant_data:/qdrant/storage \
  qdrant/qdrant
```

In that case, your Qdrant URL becomes `http://localhost:6335`.

---

## Step 5 — Configure RooCode

Open RooCode settings (Indexing / Retrieval / Embeddings).

### 5.1 Configure Embeddings (Ollama)

Set:

* **Embedder Provider:** `Ollama`
* **Base URL:** `http://localhost:11434`
* **Model:** `nomic-embed-text`

### 5.2 Configure Vector Database (Qdrant)

Set:

* **Qdrant URL:** `http://localhost:6333`
* **API Key:** (leave blank)

> If you changed ports in Step 4.4, use that URL instead (e.g. `http://localhost:6335`).

---

## Step 6 — Start Indexing

In RooCode:

1. Choose your project folder / repo
2. Click **Index / Build Index / Scan**
3. Wait for indexing to complete

### Recommended ignore patterns (faster + cleaner indexing)

If RooCode supports ignore patterns, add:

* `node_modules/`
* `.git/`
* `dist/`
* `build/`
* `.next/`
* `coverage/`
* `*.log`

(Optional) set a max file size (e.g. **1–2 MB**) to avoid huge generated files.

---

## Step 7 — Troubleshooting (Common “fetch failed” Fixes)

### 7.1 Health check: Can your machine reach Ollama and Qdrant?

Run:

```bash
curl http://localhost:11434/api/tags
curl http://localhost:6333
```

If these fail, the services aren’t running or ports are blocked.

### 7.2 Check Qdrant container status/logs

```bash
docker ps | grep qdrant
docker logs qdrant --tail 80
```

### 7.3 Check Ollama is listening on 11434

```bash
lsof -i :11434
```

### 7.4 If RooCode runs inside Docker / Dev Container / Remote

If RooCode is not running directly on your Mac (e.g., it’s inside Docker), `localhost` will not point to your Mac.

Use these URLs instead:

* **Ollama:** `http://host.docker.internal:11434`
* **Qdrant:** `http://host.docker.internal:6333`

Update RooCode settings accordingly.

---

## Step 8 — Optional: Make Services Auto-Start

### 8.1 Auto-restart Qdrant container after reboot

```bash
docker update --restart unless-stopped qdrant
```

### 8.2 Run Ollama as a background service

```bash
brew services start ollama
```

Verify:

```bash
curl http://localhost:11434/api/tags
```

---

## Done ✅

At this point:

* Ollama is serving embeddings on `http://localhost:11434`
* Qdrant is running on `http://localhost:6333`
* RooCode is configured to embed + store vectors locally
* Indexing should run end-to-end

If you still see errors, capture:

* RooCode error text
* Your configured URLs (Ollama + Qdrant)
* Output of:

  * `curl http://localhost:11434/api/tags`
  * `curl http://localhost:6333`
  * `docker logs qdrant --tail 80`
    and you can diagnose quickly.

```
```
