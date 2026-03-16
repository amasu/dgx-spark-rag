# DGX Spark RAG

A fully containerized Retrieval Augmented Generation (RAG) stack optimized for NVIDIA DGX systems. This project combines Ollama (LLM server), Qdrant (vector database), SearXNG (private search engine), and AnythingLLM (web UI) into a single Docker Compose deployment.

## Features

- **Ollama**: Serve and manage large language models (like Qwen3) with GPU acceleration.
- **Qdrant**: High-performance vector database for efficient similarity search.
- **SearXNG**: Privacy-respecting metasearch engine to augment LLM knowledge with web results.
- **AnythingLLM**: Intuitive web interface for chatting with your documents and AI models.
- **NVIDIA GPU Support**: Services are configured to reserve GPUs when deployed in Docker Swarm (or with appropriate runtime configuration).

## Architecture

```
+---------------------+      +---------------------+      +---------------------+
|   Ollama (LLM)      |      |   Qdrant (Vector)   |      |   SearxNG (Search)  |
|  - Language Models  |      |  - Vector Storage   |      |  - Web Search       |
|  - GPU Accelerated  |      |  - Similarity Search|      |  - Privacy Focused  |
+----------+----------+      +----------+----------+      +----------+----------+
           \                          /                         /
            \                        /                         /
             \                      /                         /
              \                    /                         /
               \                  /                         /
                \                /                         /
                 \              /                         /
                  \            /                         /
                   \          /                         /
                    \        /                         /
                     \      /                         /
                      \    /                         /
                       \  /                         /
                        \/                         \/
                        +-------------------------------+
                        |   AnythingLLM (UI)            |
                        |   - Chat Interface            |
                        |   - Document Management       |
                        |   - Model Configuration       |
                        +-------------------------------+
```

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/) (version 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2.0+)
- NVIDIA GPU drivers (for GPU acceleration)
- Optional: NVIDIA Container Toolkit (for GPU support in standalone Compose)

> **Note**: The `deploy` section in `docker-compose.yml` is designed for Docker Swarm. For standalone Compose (`docker compose up`), GPU access may be ignored. To enable GPUs with standalone Compose, consider adding `runtime: nvidia` to each service or updating the Compose file to use `device_requests` at the service level (Compose v2.4+).

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/amasu/dgx-spark-rag.git
   cd dgx-spark-rag
   ```

2. **(Optional) Configure GPU support**:
   - For Docker Swarm: Initialize a swarm and deploy with `docker stack deploy`.
   - For standalone Compose with NVIDIA runtime: Edit `docker-compose.yml` and add `runtime: nvidia` under each service, or use the NVIDIA Container Toolkit with `device_requests`.

3. **Start the services**:
   ```bash
   docker compose pull
   docker compose up -d
   ```

4. **Pull required models** (Ollama):
   ```bash
   docker exec -it ollama ollama pull qwen3.5:latest
   docker exec -it ollama ollama pull qwen3-embedding
   ```

## Usage

Once the stack is running, access the services at:

- **AnythingLLM UI**: http://localhost:3001
- **Ollama API**: http://localhost:11434
- **Qdrant API**: http://localhost:6333
- **SearXNG**: http://localhost:8181

### Managing Models with Ollama

You can pull, list, and remove models via the Ollama CLI inside the container:

```bash
# List models
docker exec -it ollama ollama list

# Pull a model
docker exec -it ollama ollama pull <model-name>

# Remove a model
docker exec -it ollama ollama rm <model-name>
```

### Customizing SearXNG

SearXNG configuration is stored in the `./searxng` directory. Modify settings there and restart the container:

```bash
docker compose restart searxng
```

### Persistent Data

All services store data in named volumes or bind mounts:

- Ollama: `./ollama_storage`
- Qdrant: `./qdrant_storage`
- AnythingLLM: `./anythingllm_storage`
- SearXNG: `./searxng` (configuration)

## Configuration

Environment variables and volume mounts are defined in `docker-compose.yml`. Adjust as needed:

- **Ollama**: Change model storage location via `./ollama_storage` bind mount.
- **Qdrant**: Adjust storage path via `./qdrant_storage`.
- **AnythingLLM**: Storage directory and environment variables under `./anythingllm_storage`.
- **SearXNG**: Configuration files in `./searxng`.

## GPU Support

The Compose file includes `deploy.resources.reservations.devices` for NVIDIA GPUs, which is effective when deploying to a Docker Swarm. For standalone Compose, you may need to:

1. Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).
2. Add `runtime: nvidia` to each service requiring GPU access, or use:
   ```yaml
   deploy:
     resources:
       reservations:
         devices:
           - driver: nvidia
             count: 1
             capabilities: [gpu]
   ```
   (Note: This may require Compose v2.4+ and the `docker compose` command with appropriate features enabled.)

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Ollama](https://ollama.com/)
- [Qdrant](https://qdrant.tech/)
- [SearXNG](https://searxng.org/)
- [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm)

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

---

*Happy retrieval-augmented generation!*
