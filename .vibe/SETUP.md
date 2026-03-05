# Vibe + Ollama Local Setup

Reproducible setup for running Vibe with a local Devstral model via Ollama.

## 1. Install Ollama

```bash
# Linux
curl -fsSL https://ollama.com/install.sh | sh

# macOS
brew install ollama

# Or download from https://ollama.com/download
```

Start the Ollama service:
```bash
# Linux (systemd)
sudo systemctl start ollama

# macOS (runs automatically after install)
# Or manually:
ollama serve
```

Verify it's running:
```bash
ollama --version
curl -s http://localhost:11434/v1/models | python3 -c "import sys,json; print(json.dumps(json.load(sys.stdin),indent=2))"
```

## 2. Pull and configure the model

```bash
# Pull the base model (~15 GB)
ollama pull devstral-small-2

# Create the medmcp-optimised variant (32K context, low temperature)
ollama create devstral-medmcp -f .vibe/Modelfile.devstral
```

Verify the model has the right settings:
```bash
ollama show devstral-medmcp --parameters
# Should show:
#   num_ctx     32768
#   temperature 0.1
```

## 3. Install the Vibe config

```bash
# Copy the tracked config to the Vibe home directory
cp .vibe/config.toml ~/.vibe/config.toml
```

Or if you already have a `~/.vibe/config.toml`, merge the `[[providers]]` and `[[models]]` sections for `ollama` and `devstral-medmcp`.

## 4. Install medmcp tools

```bash
# In the medmcp repo
cd ~/medmcp
uv sync --extra brain --extra registration --extra vibe
uv run medmcp setup   # symlinks tools + skills into ~/.vibe/
```

## 5. Switch to local model

Set `active_model = "local"` in `~/.vibe/config.toml`, or use the model picker in Vibe.

## 6. Test

```bash
# Quick API test
curl -s http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"devstral-medmcp","messages":[{"role":"user","content":"hello"}]}' \
  | python3 -m json.tool

# Run the medmcp tool-calling integration tests
cd ~/medmcp
uv run pytest tests/test_ollama_tools.py -v
```

## Files in this directory

| File | Purpose |
|------|---------|
| `config.toml` | Full Vibe config with Ollama provider + devstral-medmcp model |
| `Modelfile.devstral` | Ollama Modelfile (num_ctx=32768, temperature=0.1) |
| `SETUP.md` | This file |
