# Results

This repository keeps only release-facing, repository-tracked summary outputs under `results/`. Those tracked summaries describe the canonical `CodeMarkBench` benchmark surface for evaluating the reliability of source-code watermarking for LLM-based code generation, with exact-value tables and released submetrics as the primary evidence surface.

- `schema.json`: machine-readable schema notes for per-run `report.json` / `summary.scorecard` payloads
- `export_schema.json`: machine-readable contract for the tracked `suite_all_models_methods` summary figure/table exports
- `../docs/release_provenance.md`: publication-facing canonical matrix identity and provenance contract
- `figures/dataset_statistics/`: repository-tracked dataset and evaluation-framework figures
- `tables/dataset_statistics/`: repository-tracked dataset statistics tables
- `figures/suite_all_models_methods/`: materialized release-facing full-run figures for the canonical `140/140` single-host result
- `tables/suite_all_models_methods/`: materialized release-facing full-run tables plus the export-identity sidecar for figure-only redraw validation

Large raw per-run result trees are intentionally distributed outside git. See [`../docs/artifacts.md`](../docs/artifacts.md).
