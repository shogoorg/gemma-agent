# gemma-agent

Simple ReAct agent
Agent generated with `agents-cli` version `1.13.0`

## Project Structure

```
gemma-agent/
├── app/         # Core agent code
│   ├── agent.py               # Main agent logic
│   ├── fast_api_app.py        # FastAPI Backend server
│   └── app_utils/             # App utilities and helpers
├── tests/                     # Unit, integration, and load tests
├── GEMINI.md                  # AI-assisted development guide
└── pyproject.toml             # Project dependencies
```

> 💡 **Tip:** Use [Antigravity CLI](https://antigravity.google/) for AI-assisted development - project context is pre-configured in `GEMINI.md`.

## Requirements

Before you begin, ensure you have:
- **uv**: Python package manager (used for all dependency management in this project) - [Install](https://docs.astral.sh/uv/getting-started/installation/) ([add packages](https://docs.astral.sh/uv/concepts/dependencies/) with `uv add <package>`)
- **agents-cli**: Agents CLI - Install with `uv tool install google-agents-cli`
- **Google Cloud SDK**: For GCP services - [Install](https://cloud.google.com/sdk/docs/install)


## Quick Start

Install `agents-cli` and its skills if not already installed:

```bash
uvx google-agents-cli setup
```

Install required packages:

```bash
agents-cli install
```

Test the agent with a local web server:

```bash
agents-cli playground
```

You can also use features from the [ADK](https://adk.dev/) CLI with `uv run adk`.

## Commands

| Command              | Description                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------- |
| `agents-cli install` | Install dependencies using uv                                                         |
| `agents-cli playground` | Launch local development environment                                                  |
| `agents-cli lint`    | Run code quality checks                                                               |
| `agents-cli eval`    | Evaluate agent behavior (generate, grade, analyze, and more — see `agents-cli eval --help`) |
| `uv run pytest tests/unit tests/integration` | Run unit and integration tests                                                        || [A2A Inspector](https://github.com/a2aproject/a2a-inspector) | Launch A2A Protocol Inspector                                                        |

## 🛠️ Project Management

| Command | What It Does |
|---------|--------------|
| `agents-cli scaffold enhance` | Add CI/CD pipelines and Terraform infrastructure |
| `agents-cli infra cicd` | One-command setup of entire CI/CD pipeline + infrastructure |
| `agents-cli scaffold upgrade` | Auto-upgrade to latest version while preserving customizations |

---

## Development

Edit your agent logic in `app/agent.py` and test with `agents-cli playground` - it auto-reloads on save.

## Deployment

```bash
gcloud config set project <your-project-id>
agents-cli deploy
```

To add CI/CD and Terraform, run `agents-cli scaffold enhance`.
To set up your production infrastructure, run `agents-cli infra cicd`.

## Observability

Built-in telemetry exports to Cloud Trace, BigQuery, and Cloud Logging.

## A2A Inspector

This agent supports the [A2A Protocol](https://a2a-protocol.org/). Use the [A2A Inspector](https://github.com/a2aproject/a2a-inspector) to test interoperability.
See the [A2A Inspector docs](https://github.com/a2aproject/a2a-inspector) for details.


## Fine-Tuned Model Inference Guide

Inference execution guide for fine-tuned Gemma 4 and EmbeddingGemma 2 models in `apps/` (covering text-to-text retrieval, multimodal text-to-image retrieval, and Text-to-SQL generation via LiteRT-LM).

## 1. Environment Setup

Install the required dependencies using `uv` or `agents-cli` from the project root directory (`gemma-agent`):

```bash
# Option 1: Install with uv (recommended)
uv sync

# Option 2: Install with agents-cli
agents-cli install
```

> **Note**: Run all commands from the **project root directory** (`gemma-agent`) because scripts reference local image assets under `./assets/` (e.g., `uv run python apps/fine_tuning_embeddinggemma_with_sentence_transformers.py`).

---

## 2. Running Inference Scripts

All scripts in `apps/` automatically detect whether required models and assets are cached locally.
- **First run**: Automatically downloads the necessary models and files from Hugging Face Hub.
- **Subsequent runs**: Automatically uses the local cache for offline execution.

Run all commands from the `gemma-agent` directory:

### Fine-Tuned Text-to-Text Retrieval

Evaluate text retrieval with the fine-tuned EmbeddingGemma 2 model (`shogoorg/my-embeddinggemma-2-finetuned`):

```bash
uv run python apps/fine_tuning_embeddinggemma_with_sentence_transformers.py
```

### Fine-Tuned Multimodal Retrieval (Text-to-Image)

Retrieve relevant images based on text queries with the fine-tuned multimodal EmbeddingGemma 2 model (`shogoorg/my-multimodal-embeddinggemma-2-finetuned`):

```bash
uv run python apps/fine_tuning_multimodal_embeddinggemma_with_sentence_transformers.py
```

### Fine-Tuned Text-to-SQL Generation (LiteRT-LM on CPU)

Run Text-to-SQL inference with the QLoRA fine-tuned Gemma 4 E2B model exported to `.litertlm` (`shogoorg/gemma-text-to-sql-litertlm`) on the LiteRT-LM CPU backend:

```bash
uv run python apps/qlora_gemma_4_e2b_LiteRT.py
```

You can also pass a custom schema and query (or a local `.litertlm` file via `--model-path`):

```bash
uv run python apps/qlora_gemma_4_e2b_LiteRT.py \
  --schema "CREATE TABLE users (id INT, name TEXT);" \
  --query "List all user names."
```

---

> **Benchmark**: Execution time and memory usage of all three scripts on a low-spec CPU-only PC are documented in [docs/benchmark.md](docs/benchmark.md).

---

## 3. Notebooks

The `notebooks/` directory contains the corresponding Jupyter notebooks for fine-tuning and export:

- `fine-tuning-embeddinggemma-with-sentence-transformers.ipynb`: Fine-tuning EmbeddingGemma 2 for text retrieval.
- `fine-tuning-multimodal-embeddinggemma-with-sentence-transformers.ipynb`: Fine-tuning multimodal EmbeddingGemma 2 with text-image pairs.
- `qlora-gemma-4-e2b_LiteRT.ipynb`: QLoRA fine-tuning of Gemma 4 E2B for Text-to-SQL, `.litertlm` export, and Hub upload.
