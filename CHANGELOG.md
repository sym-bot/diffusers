# Changelog - sym-bot/diffusers Fork

This changelog documents modifications made to the [sym-bot/diffusers](https://github.com/sym-bot/diffusers) fork compared to the upstream [huggingface/diffusers](https://github.com/huggingface/diffusers).

## [Unreleased]

### Fixed

- **Graceful fallback for attention backend import failures** (`src/diffusers/models/attention_dispatch.py`)

  Added try-except wrappers around all external attention backend imports to handle ABI mismatches and import failures gracefully. Affected backends:
  - `flash_attn` (Flash Attention)
  - `flash_attn_3` (Flash Attention 3)
  - `aiter` (AMD Instinct)
  - `sageattention` (SageAttention)
  - `flex_attention` (PyTorch Flex Attention)
  - `torch_npu` (Huawei NPU)
  - `torch_xla` (TPU/XLA)
  - `xformers` (Meta xFormers)

  **Problem**: When an external attention package is installed but compiled against a different PyTorch version, importing it causes an `OSError` with "undefined symbol" errors. The previous code used `importlib.util.find_spec()` to check package availability, which only checks if the package exists but not if it can actually be imported.

  **Solution**: Wrap actual imports in try-except blocks. On failure, log a warning and set `_CAN_USE_*` flags to `False`, allowing the code to fall back to native PyTorch SDPA (Scaled Dot-Product Attention) which requires no external dependencies.

  **Impact**: Diffusers pipelines now work correctly even when external attention packages have ABI mismatches. Users see a warning instead of a crash, and inference continues using PyTorch's built-in attention implementation.

---

## Upstream Sync

This fork is periodically synced with upstream huggingface/diffusers. The modifications above are maintained as patches on top of the upstream codebase.
