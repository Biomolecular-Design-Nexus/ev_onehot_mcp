# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP (Model Context Protocol) server for protein fitness prediction using the EV+OneHot method — combining evolutionary coupling (EV) features from PLMC models with one-hot sequence encoding via Ridge regression. Based on [Hsu et al. 2022, Nature Biotechnology](https://github.com/chloechsu/combining-evolutionary-and-assay-labelled-data).

## Architecture

- **`src/server.py`** — FastMCP entry point. Mounts two sub-servers (`train_mcp`, `pred_mcp`) into the main `ev_onehot` MCP server.
- **`src/tools/train.py`** — `ev_onehot_train_fitness_predictor` tool: trains Ridge regression with 5-fold CV, train-test split, or external test set evaluation.
- **`src/tools/pred.py`** — `ev_onehot_predict_fitness` tool: loads a trained model and predicts fitness from CSV or sequence list.
- **`src/tools/predictor.py`** — Core ML classes (`BaseRegressionPredictor`, `OnehotRidgePredictor`, `EVPredictor`, `JointPredictor`) plus lower-level MCP tools (`train_onehot_predictor`, `train_ev_predictor`, `train_joint_predictor`).
- **`src/tools/couplings_model.py`** — EVmutation couplings model analysis tools.
- **`src/tools/util.py`** — Shared utilities (sequence validation, metrics, one-hot encoding).
- **`repo/ev_onehot/`** — Original upstream code from the paper's repository. The `src/tools/` modules import from here via `sys.path` manipulation (`REPO_PATH = PROJECT_ROOT / "repo" / "ev_onehot"`).

## Setup

```bash
bash quick_setup.sh          # Creates ./env conda environment and installs deps
bash quick_setup.sh --skip-env  # Skip environment creation if it already exists
```

Manual setup:
```bash
mamba create -p ./env python=3.10 pip -y
./env/bin/pip install -r requirements.txt
./env/bin/pip install --ignore-installed fastmcp
```

## Running

```bash
# Run MCP server directly
./env/bin/python src/server.py

# Register with Claude Code
claude mcp add ev_onehot -- ./env/bin/python src/server.py

# Or via fastmcp
fastmcp install claude-code src/server.py --python ./env/bin/python
```

## Local CLI Usage (non-MCP)

```bash
# 5-fold cross-validation
python repo/ev_onehot/train.py example/ --cross_val

# Train-test split with seed
python repo/ev_onehot/train.py example/ -s 1

# Predict fitness
python repo/ev_onehot/pred.py example --seq_path example/data.csv
```

## Docker

```bash
docker build -t ev_onehot_mcp .
docker run ev_onehot_mcp
```

## Required Input Files

A data directory must contain:
- `wt.fasta` — wild-type protein sequence in FASTA format
- `data.csv` — training data with `seq` and `log_fitness` columns
- `plmc/` — directory with `uniref100.model_params` (PLMC EVmutation model)

The PLMC model must be compatible with the wild-type sequence (validated at load time). See `example/` for a working reference dataset.

## Key Dependencies

- `evcouplings` (installed from GitHub develop branch)
- `fastmcp` — MCP server framework
- `scikit-learn` — Ridge regression
- `loguru` — logging
- `numba` — JIT compilation for couplings model

## Environment Variables

- `TRAIN_INPUT_DIR` / `TRAIN_OUTPUT_DIR` — override default `tmp/inputs` and `tmp/outputs` for training
- `PRED_INPUT_DIR` / `PRED_OUTPUT_DIR` — override defaults for prediction
