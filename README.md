# n8n-compose

Docker Compose setup for [n8n](https://n8n.io/) with optional [Ollama](https://ollama.com/) integration, behind Apache2 reverse proxy with HTTPS.

## Quick start

1. Choose a compose file:
   - **n8n only:** `n8n-compose/docker-compose.yml`
   - **n8n + Ollama:** `n8n+ollama-compose/docker-compose.yml`

2. Replace placeholders (exemple:`<YOUR_DOMAIN>`) in the compose file and Apache configs

3. Follow [`setup.md`](setup.md) for the full step-by-step guide


## ⚠️ Important

The Apache HTTPS config **must** include WebSocket rewrite rules, see step 7.3 in `setup.md`.


## possible development

apache2 / nginx into docker
