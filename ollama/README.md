# Local LLM with Ollama

A minimal Docker Compose setup for running a local LLM via [Ollama](https://ollama.com). The `ollama` service runs the inference server and exposes an OpenAI-compatible API on port `11434`, while `model-puller` is a one-shot service that waits for Ollama to become healthy and then downloads the configured model. Models are persisted in a named volume (`ollama_data`) so they survive container restarts and aren't re-downloaded every time.

## Usage

Start everything:

```bash
docker compose up -d
```

The default model is `qwen2.5-coder:7b`. To use a different one, override the `MODEL` variable:

```bash
MODEL=llama3.2:3b docker compose up -d
```

Or set it persistently in a `.env` file next to `compose.yaml`:

```
MODEL=qwen2.5-coder:7b
```

## Checking model-puller logs

The pull can take a while (models are several GB), so tail the logs to watch progress:

```bash
docker compose logs -f model-puller
```

Once the pull completes, the service exits with status `0` — that's expected. To pull an additional model later, update `MODEL` and re-run just that service:

```bash
MODEL=llama3.2:3b docker compose up model-puller
```

## Verifying it works

```bash
curl http://localhost:11434/api/generate -d "{\"model\": \"qwen2.5-coder:7b\", \"prompt\": \"Write a Python hello world\", \"stream\": false}"
```

## Stopping

```bash
docker compose down          # stop containers, keep the model volume
docker compose down -v       # stop and delete the volume (re-downloads models next time)
```
