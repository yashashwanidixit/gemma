# Adapters

LoRA adapter weights are added AFTER the skeleton. This directory holds only
this README until then; the agent YAMLs leave `adapter:` commented out.

## Expected layout

One subdirectory per adapter, each containing exactly:

```
adapters/<adapter_name>/
  adapter_config.json
  adapter_model.safetensors
```

Then set the matching agent's `adapter` field to the directory path, e.g.
`adapter: adapters/<adapter_name>`. Every agent keeps the same base model
(`gemma-4-31b-it-qat-w4a16-ct`); only the adapter may differ.

## Size budget — do not exceed

The TOTAL UNPACKED submission (skeleton + all adapter weights) must stay under
**3 GiB = 3,221,225,472 bytes**. Rough per-adapter cost for this model:

| LoRA rank | approx. size |
|-----------|--------------|
| r=16      | ~110-220 MB  |
| r=64      | ~450-900 MB  |

Budget adapter count and rank against that ceiling BEFORE final packaging —
never measure the skeleton in isolation. Only `.safetensors` is allowed for
weights (`.bin`, `.pt`, `.pth` are rejected by the packager).
