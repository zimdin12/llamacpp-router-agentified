# aify-llamacpp-router

Ollama-like router managing aify-llamacpp sub-containers via Docker SDK. Port 11434 for Ollama drop-in compatibility.

## Project Structure

- `service/main.py` — FastAPI app, inits ModelRegistry + ContainerManager, starts model containers
- `service/model_registry.py` — Parses MODELS env var, reads config/models/*.json, generates container definitions
- `service/config.py` — Env-based config (inherited from aify-container)
- `service/containers/manager.py` — Docker SDK container lifecycle (from aify-container base)
- `service/routers/openai_proxy.py` — /v1/* proxied to correct sub-container by model name
- `service/routers/ollama_compat.py` — /api/chat, /api/generate, /api/embeddings, /api/tags, /api/show
- `service/routers/health.py` — /health, /ready (with model list), /info
- `service/routers/containers.py` — Container management API
- `config/models/*.json` — Model catalog (shared format with aify-llamacpp)
- `mcp/sse_server.py` — MCP SSE server with container management tools

## Key Patterns

- MODELS env var (comma-separated) drives which containers get spawned
- ModelRegistry.generate_container_definitions() creates ContainerManager entries dynamically
- Static containers from config/service.json merge with dynamic model containers
- All sub-containers share a single Docker volume for model file caching
- Ollama endpoints translate request/response format, proxy to OpenAI endpoints on sub-containers
- Port 11434 = Ollama default, so OpenAI/Ollama SDKs work without changes

## Dependencies

- Requires aify-llamacpp Docker image to be built and tagged
- Docker socket must be mounted for container management
