# Agent Guide: aify-llamacpp-router

This guide explains how to use and extend the aify-llamacpp-router service.

## Overview

This service is an Ollama-compatible router that manages multiple aify-llamacpp sub-containers. Each sub-container runs a single LLM model. The router handles:

1. **Model registry** — parses `MODELS` env var, loads model catalog configs
2. **Container lifecycle** — spawns/stops Docker containers for each model
3. **Request routing** — proxies API requests to the correct model container
4. **API translation** — Ollama-format requests are translated to OpenAI format

## Architecture

```
Client
  │
  ├── /v1/chat/completions ──► openai_proxy.py ──► llm-{model}:8080/v1/chat/completions
  ├── /api/chat ──► ollama_compat.py ──► translates to OpenAI ──► llm-{model}:8080/v1/chat/completions
  └── /api/tags ──► ollama_compat.py ──► ModelRegistry.list_models()
```

## Adding Models

1. Create a config in `config/models/<name>.json`
2. Add the name to `MODELS` env var
3. Restart the router

The model catalog format is shared with aify-llamacpp — see its README for the JSON schema.

## Key Files

| File | Purpose |
|---|---|
| `service/main.py` | Startup: ModelRegistry → ContainerManager → MCP |
| `service/model_registry.py` | MODELS env → container definitions |
| `service/routers/openai_proxy.py` | /v1/* proxy to sub-containers |
| `service/routers/ollama_compat.py` | /api/* Ollama translation layer |
| `service/containers/manager.py` | Docker SDK container lifecycle |
| `config/models/*.json` | Model catalog |

## Extending

### Adding a new API format

Create a new router in `service/routers/` that:
1. Accepts requests in the target format
2. Resolves model → container URL via `ModelRegistry.get_model_url()`
3. Translates to OpenAI format
4. Proxies to the sub-container

### Adding static containers

Define them in `config/service.json` under `containers.definitions`. They merge with the dynamically-generated model containers.

### GPU management

Set `GPU_FRACTION_PER_MODEL` to split GPU across models. The ContainerManager tracks allocations and prevents overcommitment.
