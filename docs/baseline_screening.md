# Baseline Screening

This note records the reviewer-facing baseline-screening outcome for the current canonical release.

## Included Runtime Baselines

The active benchmark roster keeps four pinned runtime baselines:

- `STONE`
- `SWEET`
- `EWD`
- `KGW`

These four methods remain in scope because they can be executed through a shared runtime-generation contract for source-code watermarking in LLM-based code generation: the benchmark controls orchestration, local model loading, decoding policy, release inputs, and score computation while preserving the pinned upstream watermark logic.

The exact pinned repositories and commits for those four baselines live in [`third_party/`](../third_party/).

## Excluded White-Box Methods

Two white-box source-code watermarking methods for LLM-based code generation were screened for inclusion and then excluded from the active runtime roster.

| Method | Screening outcome | Public basis used for screening | Why it is excluded from the active benchmark |
| --- | --- | --- | --- |
| `CodeIP` | Excluded after baseline screening | The public repository exposes code, but the official public artifact set does not provide the complete runtime assets required for a reproducible official-public integration path. | The benchmark requires an official-public, runtime-comparable, reproducible execution path. Without a complete official public artifact set, `CodeIP` cannot satisfy that inclusion standard. |
| `Practical and Effective Code Watermarking for Large Language Models` | Excluded after baseline screening | The public implementation path relies on training or model modification rather than a shared runtime-generation adapter contract. | The active benchmark compares methods that can be executed under the same runtime-generation contract. Training/model-modifying methods fall outside that comparison scope. |

These exclusions do not claim the methods are unimportant. They only state that the current benchmark release does not place them on the same active runtime leaderboard as `STONE`, `SWEET`, `EWD`, and `KGW`.

## Prior Work Outside The Active Roster

- `UIUC ICLR 2025 / llm-code-watermark` remains cited as prior robustness work.
- It is not treated as a redundant benchmark replacement or as an additional active baseline method in the canonical leaderboard.
