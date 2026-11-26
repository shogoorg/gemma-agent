# Benchmark: Fine-Tuned Gemma & EmbeddingGemma Scripts on a Low-Spec CPU-Only PC

This document reports the execution time and memory usage of the 3 inference scripts in `apps/`, measured on a low-spec, CPU-only PC with models cached locally.

## 1. Test Environment

| Item | Value |
|---|---|
| CPU | Intel N100 (4 cores, 4 threads, AVX2, no AVX-512) |
| RAM | 3.7 GiB (+ 1.0 GiB swap) |
| GPU | None |
| OS | Ubuntu 24.04.3 LTS on WSL2 (kernel 6.6.87.2-microsoft-standard-WSL2) |
| Python | 3.13.11 |
| torch | 2.15.0 (CPU) |
| transformers | 5.23.0 |
| sentence-transformers | 6.3.0 |
| litert-lm-api | 0.19.2 |
| huggingface_hub | 1.33.0 |

## 2. Methodology

- Every script was run from `gemma-agent/` with the project virtual environment (`.venv`).
- **Total time** and **peak memory** (maximum resident set size, `MaxRSS`) were measured with `/usr/bin/time`.
- **Per-step timings** were derived by prefixing each line of stdout with a timestamp (`python -u`, unbuffered output).
- Models and assets were present in the local cache / `./assets/` directory (simulating typical steady-state execution).
- Timings on WSL2 vary with the OS page cache and background system load, so differences of roughly ±10–20 s between runs should not be over-interpreted.

## 3. Summary

| Script | Total time | Peak memory | Notes |
|---|---:|---:|---|
| `fine_tuning_embeddinggemma_with_sentence_transformers.py` | 50.1 s | 2.30 GiB | Fine-tuned text EmbeddingGemma 2 (Sentence Transformers) |
| `fine_tuning_multimodal_embeddinggemma_with_sentence_transformers.py` | 121.2 s | 2.89 GiB | Fine-tuned multimodal EmbeddingGemma 2 (Sentence Transformers) |
| `qlora_gemma_4_e2b_LiteRT.py` | 45.0 s | 1.85 GiB | QLoRA fine-tuned Gemma 4 E2B Text-to-SQL (LiteRT-LM, 2 queries) |

> Note: All three scripts run inference using already fine-tuned models (`shogoorg/my-embeddinggemma-2-finetuned`, `shogoorg/my-multimodal-embeddinggemma-2-finetuned`, `shogoorg/gemma-text-to-sql-litertlm`). On the very first run without a local cache, additional time is spent downloading model weights (e.g., 3.02 GB for `model.litertlm`).

## 4. Detailed Results

### 4.1 `fine_tuning_embeddinggemma_with_sentence_transformers.py`

Fine-tuned text EmbeddingGemma 2 (`shogoorg/my-embeddinggemma-2-finetuned`, float32), 1 query and 1 document.

| Step | Time |
|---|---:|
| Python start-up + model load + encode | 44.6 s |
| Interpreter shutdown | 5.5 s |
| **Total** | **50.1 s** |

- Similarity score: `0.8493`.
- Peak memory: **2.30 GiB**.
- Model loading accounts for almost all of the execution time.

### 4.2 `fine_tuning_multimodal_embeddinggemma_with_sentence_transformers.py`

Fine-tuned multimodal EmbeddingGemma 2 (`shogoorg/my-multimodal-embeddinggemma-2-finetuned`, float32), 1 text query and 3 images.

| Step | Time |
|---|---:|
| Python start-up + model load + encode | 107.3 s |
| Interpreter shutdown | 13.9 s |
| **Total** | **121.2 s** |

- Similarities: `[0.7091, 0.6116, 0.6654]`.
- This script loads the model without `config_kwargs`, so the audio encoder is loaded as well. It has the **highest peak memory of the three scripts (2.89 GiB)**, close to the 3.7 GiB limit of this PC.

### 4.3 `qlora_gemma_4_e2b_LiteRT.py`

QLoRA fine-tuned Gemma 4 E2B exported to `.litertlm` (`shogoorg/gemma-text-to-sql-litertlm`, 3.02 GB), running Text-to-SQL inference on the LiteRT-LM CPU backend (4 threads, `max_num_tokens=1024`, `top_k=1`).

| Step | Time |
|---|---:|
| Python start-up + resolve cached model path | ~2.0 s |
| Load LiteRT-LM engine (CPU backend) | ~10.0 s |
| Query 1 (`List all user names from Japan.`) | 20.3 s |
| Query 2 (`What is the total sales amount per product category in 2026?`) | 10.6 s |
| Engine close + interpreter shutdown | ~2.1 s |
| **Total** | **45.0 s** |

- Peak memory: **1.85 GiB** (`1,935,892 KB`, 0 swaps).
- Answers are streamed, so the first tokens appear before generation completes.
- Query 1 is slower than Query 2 (~20.3 s vs. ~10.6 s) due to initial engine warm-up on the first request.

## 5. Key Findings

1. **`qlora_gemma_4_e2b_LiteRT.py` is the fastest and most memory-efficient of the three scripts** (45.0 s total, 1.85 GiB peak, 0 swaps), even though it runs full autoregressive SQL generation for two queries.
2. **Loading the Sentence Transformers (PyTorch) models is the dominant fixed cost.** In `fine_tuning_embeddinggemma_with_sentence_transformers.py`, model loading takes the bulk of the 44.6 s runtime, whereas LiteRT-LM loads the quantized Gemma 4 E2B model in ~10 s.
3. **Memory is the tightest constraint for multimodal PyTorch workloads.** `fine_tuning_multimodal_embeddinggemma_with_sentence_transformers.py` reaches 2.89 GiB on a 3.7 GiB machine because it loads the audio encoder as well. Disabling unused encoders (`config_kwargs={"audio_config": None}`) noticeably reduces memory usage.
4. **Interpreter shutdown is much faster on LiteRT-LM** (~2.1 s) than on Sentence Transformers / PyTorch (5.5–13.9 s) because fewer large PyTorch tensors need to be freed.

## 6. Recommendations for Low-Spec PCs

- Prefer **LiteRT-LM (`.litertlm`)** for CPU-only deployment whenever a quantized export is available: it delivers lower peak memory (1.85 GiB) and faster start-up/shutdown than PyTorch.
- Keep the model/engine loaded across queries (e.g., interactive loop or service) to avoid paying the load and first-query warm-up cost repeatedly.
- For multimodal Sentence Transformers workloads, disable unused encoders (`config_kwargs={"audio_config": None}`) and close other applications to avoid swapping.
