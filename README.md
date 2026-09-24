#  TrialScout: Agentic Clinical-Trial Matching

Patients describe illness in everyday language ("shaking hands and stiff, slow walking"); trial registries use clinical vocabulary and free-text eligibility rules. TrialScout bridges that gap: it matches a free-text patient description to **recruiting trials from ClinicalTrials.gov**, enforces age/sex constraints, flags possible exclusion conflicts, and explains each match.

>  Research demo, not medical advice. Use synthetic data only.

## Architecture

```
free text → intake (PII scrub, parse age/sex) ── missing? → clarify
          → retrieve: BM25 ⊕ fine-tuned BGE → RRF → age/sex filter → cross-encoder rerank
          → verify (exclusion-criteria screening) → explain (grounded local LLM) → report
Exposed as: FastMCP tools · FastAPI /match · Gradio UI
```

## Highlights

- **Hybrid retrieval stack** with a stage-by-stage ablation (BM25 → dense → RRF → rerank → constraints → fine-tuned).
- **From-scratch InfoNCE fine-tuning** of `bge-small-en-v1.5` with false-negative masking (same-condition trials are not treated as negatives).
- **Leak-resistant evaluation:** held-out trials *and* held-out patient phrasings; graded nDCG@10, MRR, Eligible-P@10, latency.
- **LangGraph agent** with conditional routing and guardrails: PII scrubbing, clarification instead of guessing, and a report assembled in code so the LLM cannot invent trials.
- **Shipping surfaces:** FastMCP server, FastAPI service, Gradio demo. Production path documented for PostgreSQL + pgvector.

## Results

Run the notebook to generate `ablation_results.csv` and the ablation chart, then paste your table here.

| System | nDCG@10 | Eligible-P@10 |
|---|---|---|
| BM25 | _run_ | _run_ |
| Dense (base) | _run_ | _run_ |
| Hybrid + rerank + constraints | _run_ | _run_ |
| **TrialScout (fine-tuned, full)** | _run_ | _run_ |

## Quickstart

1. Open `TrialMedicalScout.ipynb` in Google Colab.
2. `Runtime → Change runtime type → T4 GPU`.
3. `Run all` (about 10–15 min, no API keys). The Gradio cell prints a public demo link.

Set `force_synthetic=True` in `CONFIG` to run without network access to ClinicalTrials.gov.

## Limitations

- Patients are synthetic and templated; relevance labels are rule-based (registry condition tags + age/sex), not clinician-adjudicated.
- Exclusion screening is keyword-based and only *flags* items to verify.
- Intake parsing is regex-based (accuracy is measured in the notebook).

## Next steps

TREC Clinical Trials evaluation · NLI/LLM criterion-level entailment · geographic filtering · learning-to-rank from clinician feedback · PostgreSQL + pgvector backend.

## Author

**Yug Makhecha**: [Portfolio](https://yugcontact.vercel.app/) · [LinkedIn](https://www.linkedin.com/in/yug-makhecha-417601295/) · [GitHub](https://github.com/YugMakhecha17)

Data: [ClinicalTrials.gov](https://clinicaltrials.gov) public API. Models: BAAI/bge-small-en-v1.5, cross-encoder/ms-marco-MiniLM-L-6-v2, Qwen2.5-1.5B-Instruct.
