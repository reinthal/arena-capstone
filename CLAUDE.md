# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository implements **Inoculation Prompting** ([paper](https://arxiv.org/abs/2510.05024)) - a technique to improve LLM robustness by adding specific prompts during training that prevent models from learning undesirable behaviors.

The main research code lives in the `inoculation-prompting/` git submodule. Each experimental setting is independent with its own virtual environment.

## Build and Development Commands

**Package manager**: `uv` (not pip)

```bash
# Root project setup
uv sync

# Install pre-commit hooks (from inoculation-prompting/)
make hooks
```

**Linting/Formatting** (enforced via pre-commit):
- Ruff (v0.11.5) with auto-fix
- Black (25.1.0)
- nbstripout for Jupyter notebooks
- Direct commits to `main` branch are blocked

**Python version**: 3.11

## Experiment-Specific Setup

Each experiment has its own setup. Always `cd` into the experiment directory first.

### Code Reward Hacking / Reddit CMV (`code_rh_and_reddit_toxic/`)
```bash
uv venv --python=python3.11
uv pip install git+https://github.com/safety-research/safety-tooling.git@main#egg=safetytooling
uv pip install openweights inspect-ai==0.3.116 unidecode
```

Run pipeline:
```bash
uv run --env-file ../.env python -m run_pipeline --dataset_type code  # or realistic
```

Run tests:
```bash
python -m pytest test_ctg_utils.py realistic_dataset/ supervised_code/
```

### GCD Sycophancy (`gcd_sycophancy/`)
```bash
uv pip install .
cd projects
uv run --env-file ../../.env python attribute_sweep_multi_seed_run.py ip_sweep
```

### Mechanism (`mechanism/`)
```bash
git submodule update --init --recursive
uv venv --python=python3.11
uv pip install -e safety-tooling
uv pip install -r requirements.txt
python -m pair_traits.pipeline --supervision_traits 5 0 --inoculation_traits 0
```

Run tests:
```bash
cd safety-tooling && python -m pytest -n 6
```

### Spurious Correlation (`spur_corr/`)
Requires local GPU.
```bash
uv venv
uv pip install -r requirements.txt
uv run main.py --dataset cebab --concept ambiance --method very_biased ...
```

## Environment Variables

Required API keys (stored in `.env` at repo root, encrypted with SOPS):
- `HF_TOKEN` - HuggingFace
- `ANTHROPIC_API_KEY`
- `OPENAI_API_KEY`
- `OPENWEIGHTS_API_KEY` - For RunPod GPU provisioning

The `.envrc` uses direnv with SOPS to decrypt `secrets.env` automatically.

## Architecture

```
arena-capstone/
├── pyproject.toml              # Root dependencies (45+ ML packages)
├── inoculation-prompting/      # Git submodule with all experiments
│   ├── code_rh_and_reddit_toxic/
│   │   ├── run_pipeline.py     # Main entry - orchestrates training on RunPod
│   │   ├── supervised_code/    # Code reward hacking dataset/eval
│   │   └── realistic_dataset/  # Reddit CMV dataset/eval
│   ├── gcd_sycophancy/
│   │   └── projects/
│   │       ├── attribute_sweep_multi_seed_run.py  # Main entry
│   │       └── gemma_gcd/main.py                  # Training script
│   ├── mechanism/
│   │   ├── pair_traits/pipeline.py  # Main entry
│   │   └── safety-tooling/          # Nested submodule
│   └── spur_corr/
│       └── main.py             # Main entry (local GPU required)
```

## Key Dependencies

- `inspect-ai` (0.3.116): LLM evaluation framework
- `unsloth`: Fast LoRA fine-tuning
- `vllm`: Model serving
- `transformers`, `peft`, `trl`: HuggingFace training stack
- `fire`: CLI argument parsing
- `wandb`: Experiment tracking

## Git Submodules

```bash
git submodule update --init --recursive  # Initialize all submodules
```

The `inoculation-prompting/` submodule contains the main research code. The `mechanism/` experiment has a nested `safety-tooling/` submodule.
