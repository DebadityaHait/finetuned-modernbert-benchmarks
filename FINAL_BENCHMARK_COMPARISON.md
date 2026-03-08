# Final Benchmark Comparison (combined-prompts-v3.json)

Primary model alias used in this package: **Finetuned Modernbert**
Model URL: https://huggingface.co/zazaman/finetuned-modernbert-large

Dataset: `data/combined-prompts-v3.json` (`n=315`)

| Rank | Model | Accuracy | Precision(1) | Recall(1) | F1(1) | Time (s) |
|---|---|---:|---:|---:|---:|---:|
| 1 | `zazaman/finetuned-modernbert-large` | 0.9270 | 0.9298 | 0.8760 | 0.9021 | 15.48 |
| 2 | `nvidia/nemoguard-jailbreak-detect (run locally with Qwen 2.5:3b (via Ollama))` | 0.8570 | 0.8220 | 0.8017 | 0.8117 | 5081.30 |
| 3 | `protectai/deberta-v3-base-prompt-injection-v2` | 0.8254 | 0.7895 | 0.7438 | 0.7660 | 6.21 |
| 4 | `openai/gpt-oss-safeguard-20b` | 0.8444 | 0.9500 | 0.6281 | 0.7562 | 256.07 |
| 5 | `vijil/mbert-prompt-injection` | 0.8063 | 0.8409 | 0.6116 | 0.7081 | 5.36 |
| 6 | `meta-llama/llama-guard-4-12b` | 0.8000 | 0.9833 | 0.4876 | 0.6519 | 215.56 |
| 7 | `meta-llama/Llama-Prompt-Guard-2-86M` | 0.7714 | 0.9804 | 0.4132 | 0.5814 | 10.84 |
| 8 | `meta-llama/Prompt-Guard-86M` | 0.4286 | 0.4020 | 1.0000 | 0.5735 | 8.02 |
| 9 | `nvidia/nemoguard-jailbreak-detect (via NVIDIA API)` | 0.6190 | 1.0000 | 0.0083 | 0.0164 | 85.00 |

## Notes

- `nvidia/nemoguard-jailbreak-detect (run locally with Qwen 2.5:3b (via Ollama))` metrics are taken directly from externally run `benchmark_v3.py` artifacts and were not rerun in this package.
- Local/HF model metrics come from `results/local_hf/summary.json`.
- Groq + NVIDIA API metrics come from `results/api/summary.json`.

## Included Artifacts

- Scripts: `scripts/final_compare.py`, `scripts/api_compare_groq_nvidia_v3.py`, `scripts/benchmark_v3.py`
- Results: `results/local_hf/*`, `results/api/*`, `results/nemo_guardrails/*`
- Dataset: `data/combined-prompts-v3.json`
