# Reproducibility Guide For A Fresh Cloud Host

This guide is the reviewer-facing recovery path for reproducing `CodeMarkBench`
after the original execution server is no longer available. The public release
is intentionally split into two durable pieces:

- GitHub companion repository: code, documentation, canonical inputs, reviewer
  workflow scripts, environment capture, and materialized summary tables/figures
- Zenodo archival artifact: raw rerun-backed matrix tree, checksums, and the
  sanitized release bundle used to rebuild the GitHub summary surface

The archived result of record is the completed canonical single-host run:
`run_count = 140`, `success_count = 140`, `failed_count = 0`, and
`execution_mode = single_host_canonical`.

## What Can Be Reproduced

Use the three levels below depending on the review need.

| Level | Requires GPU | Requires Zenodo raw artifact | Purpose |
| --- | --- | --- | --- |
| Level 1 | No | No | Inspect the shipped benchmark definition, docs, summary tables, and figures. |
| Level 2 | No | Yes | Rebuild the tracked summary tables and figures from the archived raw matrix. |
| Level 3 | Yes | No | Re-execute the full benchmark on a fresh Linux GPU host. |

Level 2 is the exact archival regeneration path for the released tables and
figures. Level 3 is the end-to-end rerun path; it should match the benchmark
contract and completed-run invariants, while wall-clock timing, host metadata,
and cache-local details may differ from the archived server.

## Level 1: Inspect The Companion Repository

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python scripts/reviewer_workflow.py browse
```

This path is enough to inspect the benchmark, result interpretation, exported
tables, and retained publication-style figures without downloading model weights
or raw matrix files.

Primary review files:

- `docs/result_interpretation.md`
- `docs/metrics.md`
- `docs/artifacts.md`
- `results/tables/suite_all_models_methods/`
- `results/figures/suite_all_models_methods/`
- `results/environment/runtime_environment.json`

## Level 2: Rebuild Summary Exports From Zenodo

After downloading the raw-results artifact, restore the raw matrix tree so that
the canonical index exists at:

```text
results/matrix/suite_all_models_methods/matrix_index.json
```

Then regenerate the summary surface:

```bash
python scripts/refresh_report_metadata.py --matrix-index results/matrix/suite_all_models_methods/matrix_index.json
python scripts/reviewer_workflow.py regenerate \
  --matrix-index results/matrix/suite_all_models_methods/matrix_index.json \
  --figure-dir results/figures/suite_all_models_methods \
  --table-dir results/tables/suite_all_models_methods
python scripts/export_dataset_statistics.py
python scripts/reviewer_workflow.py browse
```

Expected archived-run invariants:

- `run_count = 140`
- `success_count = 140`
- `failed_count = 0`
- `execution_mode = single_host_canonical`

The companion repository intentionally does not store `results/matrix/**` in
git. The raw matrix is restored only from the archival artifact.

## Level 3: Full Rerun On A Fresh 8-GPU Linux Host

Recommended execution class:

- one Linux host
- eight visible CUDA devices
- enough disk for model caches, upstream checkouts, and raw result outputs
- Python `3.10+`
- C/C++, Java, Node.js, and Go toolchains for executable validation
- Hugging Face access for the exact pinned model identifiers in `README.md`

Fresh-host setup:

```bash
git clone https://github.com/Haoyi-Zhang/CodeMarkBench.git
cd CodeMarkBench
python -m venv .venv/tosem_release
source .venv/tosem_release/bin/activate
pip install -r requirements.txt -r requirements-remote.txt
bash scripts/fetch_runtime_upstreams.sh all
python scripts/build_suite_manifests.py
```

Readiness gates:

```bash
make suite-validate
python scripts/audit_benchmarks.py --profile suite
python scripts/audit_full_matrix.py \
  --manifest configs/matrices/suite_all_models_methods.json \
  --profile suite_all_models_methods \
  --strict-hf-cache \
  --model-load-smoke \
  --runtime-smoke \
  --skip-provider-credentials
```

Formal single-host rerun:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 \
  bash scripts/remote/run_preflight.sh --formal-full-only

CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 \
  bash scripts/remote/run_formal_single_host_full.sh --command-timeout-seconds 259200
```

After completion, regenerate the publication-facing summaries:

```bash
python scripts/reviewer_workflow.py regenerate \
  --matrix-index results/matrix/suite_all_models_methods/matrix_index.json \
  --figure-dir results/figures/suite_all_models_methods \
  --table-dir results/tables/suite_all_models_methods
python scripts/export_dataset_statistics.py
python scripts/reviewer_workflow.py browse
```

For custom experiments, use a different `--profile`, matrix output root, figure
directory, and table directory. Do not overwrite the canonical
`suite_all_models_methods` release surface unless intentionally rebuilding that
formal release.

## Validation Checklist

Use these checks before relying on a restored or rerun result:

```bash
python scripts/reviewer_workflow.py browse
python -m pytest \
  tests/test_export_full_run_tables.py \
  tests/test_render_materialized_summary_figures.py \
  tests/test_reviewer_workflow.py \
  tests/test_release_bundle_contract.py \
  tests/test_validate_release_bundle.py
```

For a restored raw artifact or a fresh full rerun, also verify:

- the canonical matrix index exists under `results/matrix/suite_all_models_methods/`
- the matrix reports `140/140` successful runs with `failed_count = 0`
- `results/tables/suite_all_models_methods/suite_all_models_methods_export_identity.json`
  records the canonical manifest/profile and `single_host_canonical`
- the figure and table directories contain the materialized summary exports
- the result interpretation remains table-first: low robustness and strict zero
  diagnostics are empirical findings, not failed executions
