# swegemma submission — pre-packaging checklist

Everything under `submission/` is the deliverable (zipped later). This file sits
outside that zip. Do the checks below before packaging.

## 1. Model

- `submission/configs/model.yaml` is the single source of truth and must contain
  exactly `gemma-4-31b-it-qat-w4a16-ct` for submission.
- It is pulled in by `agent.yaml` and all four files under `sub_agents/` via
  `model: !include .../model.yaml`, so all five declare the same base model.
- **Verify your compiler resolves a scalar `!include`.** Fallback if it does not:
  replace the five `model:` lines with the literal string
  `gemma-4-31b-it-qat-w4a16-ct` and delete `configs/model.yaml`.
- If you ran local tests against a different model, restore this one file (and
  nothing else) before packaging.

## 2. Adapters (added later)

- Drop weights under `adapters/<name>/` as `adapter_config.json` +
  `adapter_model.safetensors`, then uncomment/set `adapter:` in the relevant
  agent YAML(s). See `submission/adapters/README.md`.
- Keep the TOTAL unpacked size (skeleton + all adapters) under **3 GiB
  (3,221,225,472 bytes)** — measure the whole submission, not the skeleton alone.

## 3. Package hygiene

- Allowed extensions only: `.yaml .yml .md .txt .json .safetensors`; `.py` only
  under `skills/<name>/scripts/` (no skills are used here). No `.sh`, `.bin`,
  `.pt`, `.pth`, archives, or symlinks escaping the submission root.
- `eval_config.yaml` present at the submission root; exactly one root config
  (`agent.yaml`).
- Instructions < 1,000,000 chars per agent and < 10,000,000 total; each YAML
  < 50 MiB.
- No dangling references — every `!include` target and every
  `agent_tool.config_path` must exist relative to its referrer:

  | referrer | references |
  |---|---|
  | `agent.yaml` | `prompts/system.md`, `configs/model.yaml`, `configs/sampling.yaml`, and all four `sub_agents/*.yaml` |
  | `sub_agents/code_explorer.yaml` | `../prompts/explorer.md`, `../configs/model.yaml`, `../configs/sampling_analyzer.yaml` |
  | `sub_agents/test_verifier.yaml` | `../prompts/verifier.md`, `../configs/model.yaml`, `../configs/sampling_analyzer.yaml` |
  | `sub_agents/repro_builder.yaml` | `../prompts/repro.md`, `../configs/model.yaml`, `../configs/sampling_analyzer.yaml` |
  | `sub_agents/diff_critic.yaml` | `../prompts/critic.md`, `../configs/model.yaml`, `../configs/sampling_analyzer.yaml` |

- No untracked scratch files under `submission/`.

## 4. Zip

- Zip the `submission/` directory (including `adapters/`) as the final artifact.
