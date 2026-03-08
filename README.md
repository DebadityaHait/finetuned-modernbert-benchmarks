# Prompt-Injection Benchmark Package

This repository is the commit-ready benchmark bundle for comparing prompt-injection / jailbreak detectors on one shared dataset.

- Dataset: `data/combined-prompts-v3.json`
- Size: `315` prompts
- Labels:
  - `0` = benign
  - `1` = prompt injection / jailbreak

## Dataset Sources

`combined-prompts-v3.json` is a combined benchmark set built from:

- `NotInject`  
  Source: Hugging Face dataset `leolee99/NotInject` and the NotInject paper.
- `BIPIA`  
  Source: Microsoft repo `microsoft/BIPIA` (Indirect Prompt Injection benchmark).
- `WildGuard-Benign`  
  Source: benign subset of `WildGuard/WildGuardMix` from the WildGuard release.
- `PINT`  
  Source: Lakera repo `lakeraai/pint-benchmark`.
- Synthetic prompts generated with Gemini to emulate modern prompt-injection styles like Payload Splitting, Obfuscation and also difficult benign edge cases.

Primary model in this repo:

- **Finetuned Modernbert**
- https://huggingface.co/zazaman/finetuned-modernbert-large

## What This Repo Contains

- `scripts/final_compare.py`
  - Local Hugging Face benchmarking (batch inference).
- `scripts/api_compare_groq_nvidia_v3.py`
  - API benchmarking (Groq + NVIDIA), with retries and Groq key rotation.
- `scripts/benchmark_v3.py`
  - Local NeMo Guardrails benchmark (Qwen 2.5:3b via Ollama).
- `results/local_hf/*`
  - Local HF benchmark outputs.
- `results/api/*`
  - API benchmark outputs.
- `results/nemo_guardrails/*`
  - Externally generated NeMo Guardrails run outputs (included as-is).

## Final Benchmark Results (Inline)

All rows below are on the same dataset (`n=315`):

| Rank | Model | Accuracy | Precision(1) | Recall(1) | F1(1) | Time (s) |
|---|---|---:|---:|---:|---:|---:|
| 1 | `zazaman/finetuned-modernbert-large` | 0.9270 | 0.9298 | 0.8760 | 0.9021 | 15.48 |
| 2 | `nvidia/nemoguard-jailbreak-detect (run locally with Qwen 2.5:3b (via Ollama))` | 0.8570 | 0.8220 | 0.8017 | 0.8117 | 5081.30 |
| 3 | `protectai/deberta-v3-base-prompt-injection-v2` | 0.8254 | 0.7895 | 0.7438 | 0.7660 | 6.21 |
| 4 | `openai/gpt-oss-safeguard-20b (via Groq API)` | 0.8444 | 0.9500 | 0.6281 | 0.7562 | 256.07 |
| 5 | `vijil/mbert-prompt-injection` | 0.8063 | 0.8409 | 0.6116 | 0.7081 | 5.36 |
| 6 | `meta-llama/llama-guard-4-12b (via Groq API)` | 0.8000 | 0.9833 | 0.4876 | 0.6519 | 215.56 |
| 7 | `meta-llama/Llama-Prompt-Guard-2-86M` | 0.7714 | 0.9804 | 0.4132 | 0.5814 | 10.84 |
| 8 | `meta-llama/Prompt-Guard-86M` | 0.4286 | 0.4020 | 1.0000 | 0.5735 | 8.02 |
| 9 | `nvidia/nemoguard-jailbreak-detect (via NVIDIA API)` | 0.6190 | 1.0000 | 0.0083 | 0.0164 | 85.00 |

## How The Benchmarks Were Generated

### 1) Local HF benchmark

Script: `scripts/final_compare.py`

Compared locally:

- `zazaman/finetuned-modernbert-large`
- `meta-llama/Llama-Prompt-Guard-2-86M`
- `meta-llama/Prompt-Guard-86M`
- `vijil/mbert-prompt-injection`
- `protectai/deberta-v3-base-prompt-injection-v2`

Procedure:

- Tokenize + infer in batches (`max_length=512`).
- Convert model outputs to binary labels (`0/1`).
- Compute Accuracy, Precision(1), Recall(1), F1(1), TP/TN/FP/FN.
- Log prompt-level outputs and runtime.

Outputs:

- `results/local_hf/summary.json`
- `results/local_hf/FINAL_COMPARISON_REPORT.md`

### 2) API benchmark (Groq + NVIDIA)

Script: `scripts/api_compare_groq_nvidia_v3.py`

Compared via API:

- `openai/gpt-oss-safeguard-20b` (Groq)
- `meta-llama/llama-guard-4-12b` (Groq)
- `nvidia/nemoguard-jailbreak-detect` (NVIDIA API)

Also includes Finetuned Modernbert by reading existing local outputs.

Procedure:

- Prompt-by-prompt API inference.
- Groq key rotation + retries on transient/rate-limit failures.
- Response parsing into binary label.
- Same metric calculation as local benchmark.

Outputs:

- `results/api/summary.json`
- `results/api/API_COMPARISON_REPORT.md`

### 3) NeMo Guardrails local run

Script: `scripts/benchmark_v3.py`

Run context:

- Local NeMo Guardrails pipeline.
- Qwen 2.5:3b via Ollama.
- Blocking-based classification from generated response behavior.

Important:

- This run was executed externally before packaging.
- Files are included unchanged and used directly in final comparison.

Outputs:

- `results/nemo_guardrails/benchmark_v3_20260308_200307.json`
- `results/nemo_guardrails/benchmark_v3_20260308_200307.txt`

## Reproduce The Results

### Local HF

```powershell
python scripts/final_compare.py `
  --eval-path data/combined-prompts-v3.json `
  --output-dir results/local_hf `
  --prompt-guard-86m-positive all_attacks
```

### API (Groq + NVIDIA)

Create `.env` in working directory:

```text
GROQ_API_KEY=[key1,key2,key3]
NVIDIA=<nvidia_api_key>
```

Run:

```powershell
python scripts/api_compare_groq_nvidia_v3.py `
  --eval-path data/combined-prompts-v3.json `
  --existing-dir results/local_hf `
  --output-dir results/api `
  --env-path .env
```

### NeMo Guardrails local

Prerequisites:

- NeMo Guardrails config available at `./config`
- Ollama serving Qwen 2.5:3b

Run:

```powershell
python scripts/benchmark_v3.py
```

## Model Analysis (Comprehensive)

This section interprets the benchmark table in practical terms: what each model is good at, where it fails, and what trade-off profile it represents for real deployments.

### 1) Finetuned Modernbert (`zazaman/finetuned-modernbert-large`)

- Best overall on this dataset (`Accuracy=0.9270`, `F1=0.9021`).
- The key strength is balance: both precision and recall are high (`0.9298` / `0.8760`), so it catches most attacks without heavily over-blocking benign traffic.
- For binary prompt-injection detection, this is the most reliable default among all compared options here.
- Runtime (`15.48s` for 315 prompts) is slower than some smaller local baselines, but still practical for batch/offline scoring and acceptable for many online moderation paths depending on infrastructure.

### 2) NeMo Guardrails local run (`nvidia/nemoguard-jailbreak-detect` with Qwen 2.5:3b via Ollama)

- Strong quality (`Accuracy=0.8570`, `F1=0.8117`) and second-best by F1.
- Recall (`0.8017`) is notably strong, meaning it catches many true attacks.
- Main drawback is runtime: `5081.30s` total in this run (pipeline + generation overhead), which is much slower than classifier-style detectors.
- This profile can be reasonable when deep policy-driven response blocking is preferred over lightweight classifier latency, but it is costly for high-throughput production paths.

### 3) `protectai/deberta-v3-base-prompt-injection-v2`

- Good mid-tier baseline (`Accuracy=0.8254`, `F1=0.7660`).
- Precision and recall are moderately balanced (`0.7895` / `0.7438`), so behavior is less extreme than conservative API models.
- Very fast in this benchmark (`6.21s`), making it attractive when latency/throughput is a top priority and some quality loss is acceptable.
- Compared with Finetuned Modernbert, it underperforms by a meaningful margin on both F1 and accuracy.

### 4) `openai/gpt-oss-safeguard-20b` (via Groq API)

- High precision (`0.9500`) but lower recall (`0.6281`) with `F1=0.7562`.
- This means it is conservative: fewer false alarms, but more missed attacks.
- Runtime is high for API scoring (`256.07s` total here), and API behavior also adds integration complexity (response parsing/retries).
- Suitable if minimizing false positives is prioritized over maximum attack catch rate; less ideal if recall is critical.

### 5) `vijil/mbert-prompt-injection`

- Mid/low-tier in this comparison (`Accuracy=0.8063`, `F1=0.7081`).
- Precision is decent (`0.8409`) but recall is weaker (`0.6116`), showing a conservative bias similar to several other models.
- Fast runtime (`5.36s`) is a plus.
- Reasonable as a lightweight baseline, but not competitive with Finetuned Modernbert on security effectiveness.

### 6) `meta-llama/llama-guard-4-12b` (via Groq API)

- Very high precision (`0.9833`) and low recall (`0.4876`), giving `F1=0.6519`.
- In practice, it misses a substantial portion of attacks in this dataset while rarely flagging benign prompts incorrectly.
- Runtime is heavy (`215.56s`) for API-based usage.
- This profile can fit “block only when very sure” policies, but it is not strong as a primary detector if high recall is needed.

### 7) `meta-llama/Llama-Prompt-Guard-2-86M`

- Lower overall quality (`Accuracy=0.7714`, `F1=0.5814`) despite high precision (`0.9804`).
- Recall is low (`0.4132`), so many attacks are missed.
- Runtime (`10.84s`) is reasonable, but quality does not justify choosing it over better-performing alternatives in this benchmark.

### 8) `meta-llama/Prompt-Guard-86M`

- Very high recall (`1.0000`) with low precision (`0.4020`), resulting in poor accuracy (`0.4286`) and modest F1 (`0.5735`).
- This behavior indicates aggressive positive prediction: catches nearly all attacks but over-flags benign prompts heavily.
- Could be useful as a high-recall first-pass candidate generator in multi-stage pipelines, but not as a standalone blocker unless false positives are acceptable.

### 9) `nvidia/nemoguard-jailbreak-detect` (via NVIDIA API)

- Extremely conservative in this setup: perfect precision (`1.0000`) but near-zero recall (`0.0083`), hence very low `F1=0.0164`.
- It almost never predicts attack, so it misses nearly all true attacks on this dataset.
- Runtime (`85.00s`) is lower than Groq LLM safeguards here but still slower than local classifier baselines.
- As configured in this run, it is not suitable as a standalone detector for prompt-injection protection.

### Cross-model takeaways

- There is a clear precision-recall split across models:
  - Balanced and high-performing: **Finetuned Modernbert**.
  - Precision-heavy/conservative: several API safeguards.
  - Recall-heavy/aggressive: `meta-llama/Prompt-Guard-86M`.
- If your goal is best single-model protection quality on this dataset, **Finetuned Modernbert** is the strongest choice.
- If your goal is architecture design, a two-stage pipeline is viable:
  - Stage 1 high-recall detector (candidate generator),
  - Stage 2 high-precision verifier,
  but this must be tuned carefully to avoid compounding latency and integration complexity.
- Final model choice should also consider deployment constraints (local vs API), throughput targets, false-positive tolerance, and policy risk appetite.
