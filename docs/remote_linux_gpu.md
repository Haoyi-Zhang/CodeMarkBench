# Linux GPU Workflow

This document defines the formal release-facing rerun path for `CodeMarkBench`.

## Formal Result-Of-Record

The formal public execution contract is:

- host: one Linux execution host
- visible devices: `CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7`
- execution mode: `single_host_canonical`
- scheduler contract:
  - `--gpu-slots 8`
  - `--gpu-pool-mode shared`
  - `--cpu-workers 9`
  - `--retry-count 1`
  - `--command-timeout-seconds 259200`
- launch path: **A/B-free standalone preflight -> direct canonical full**

The optional two-host sharded path remains available only for engineering smoke, reviewer-safe reproduction, or throughput. It is not the publication-facing result source.

## Primary Scripts

- `scripts/remote/bootstrap_linux_gpu.sh`
- `scripts/remote/run_preflight.sh`
- `scripts/remote/run_formal_single_host_full.sh`
- `scripts/reviewer_workflow.py`
- `scripts/clean_suite_outputs.py`

The older `scripts/remote/run_suite_matrix.sh --run-full` wrapper is now engineering smoke only. It is no longer the formal release path.

## Suggested Sequence

1. Bootstrap the host and venv.
2. Refresh pinned upstream runtime checkouts if needed.
3. Run the canonical audits.
4. Run standalone preflight in formal-full-only mode.
5. Launch the detached canonical full run.
6. After `140/140 success`, regenerate summary tables and figures.

## Formal Commands

```bash
sudo apt-get update
sudo apt-get install -y build-essential git curl ca-certificates zstd nodejs npm openjdk-21-jdk golang-go
bash scripts/remote/bootstrap_linux_gpu.sh --install --venv .venv/tosem_release
source .venv/tosem_release/bin/activate
bash scripts/fetch_runtime_upstreams.sh all
python scripts/build_suite_manifests.py
python scripts/audit_benchmarks.py --profile suite
python scripts/audit_full_matrix.py --manifest configs/matrices/suite_all_models_methods.json --profile suite_all_models_methods --strict-hf-cache --model-load-smoke --runtime-smoke --skip-provider-credentials
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 bash scripts/remote/run_preflight.sh --formal-full-only
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 bash scripts/remote/run_formal_single_host_full.sh --command-timeout-seconds 259200
```

The strict-cache audit expects the pinned Hugging Face snapshots to already be
available under the configured local model cache. On a fresh host, first fill
or validate access to that cache with `python scripts/check_model_access.py
--token-env HF_ACCESS_TOKEN` and an audit pass using `--probe-hf-access`; then
rerun the strict-cache audit before preflight.

`run_preflight.sh --formal-full-only` validates:

- canonical full manifest/profile
- local cache readiness
- runtime/toolchain smoke
- environment capture
- direct-full launch readiness

It does **not** require canonical stage-A or stage-B manifests in the formal path.

## Reviewer Workflow

For reviewer convenience:

- `python scripts/reviewer_workflow.py browse`
- `python scripts/reviewer_workflow.py regenerate --matrix-index ...`
- `python scripts/reviewer_workflow.py subset ...`
- `CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 python scripts/reviewer_workflow.py full`

`reviewer_workflow.py full` delegates to the formal direct-full helper.

## Environment Capture

The environment-of-record is written to:

- `results/environment/runtime_environment.json`
- `results/environment/runtime_environment.md`

The formal rerun should refresh these files under the same eight-GPU execution class used for the release result of record.
For public release captures, pass `--public-safe-paths` so the persisted files
record the execution class and code identity without exposing host aliases or
absolute virtualenv paths.

## Cleanup Rule

Before the final formal rerun:

- stop any transitional or partial run
- clean old canonical outputs
- keep code, docs, schema, canonical inputs, venv, and model cache intact

After the full rerun:

- regenerate release-facing tables and figures
- remove temporary export intermediates and stale release-candidate residue
- keep only the final publication surface

## Optional Two-Host Sharding

The repository still supports identical-execution-class sharding for:

- engineering smoke
- reviewer-safe reproduction
- throughput experiments

That path must never be relabeled as the formal publication source. It remains subordinate to the single-host one-shot canonical rerun.
