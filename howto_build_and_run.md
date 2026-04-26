# How To Build and Run

This document describes how to set up a development environment for this repository, how to build it, and how to run it.

It is based on the current repository layout and scripts in this checkout:

- Python package metadata: `pyproject.toml`
- Studio setup flow: `install.sh`, `studio/setup.sh`
- Package build flow: `build.sh`
- CLI entrypoint: `unsloth`

## Project Layout

This repository has two main surfaces:

1. `Unsloth Core`
   - The Python package and CLI.
   - Used for training, inference, export, and code-based workflows.

2. `Unsloth Studio`
   - The web UI.
   - Frontend: Vite + React in `studio/frontend`
   - Backend: FastAPI in `studio/backend`

## Prerequisites

## Python

The project declares:

- Python `>=3.9,<3.15`

The README recommends Python `3.13` for fresh installs. Python `3.12` should also work for development in this repo.

## Node.js

The Studio frontend declares:

- Node `^20.19.0 || >=22.12.0`

You also need `npm`.

## uv

The repository installation flow uses `uv` for Python environment creation and package installation.

Install it if needed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Recommended Development Paths

There are two reasonable ways to work on this repo.

### Option 1: Studio-focused development

Use this if you want to work on the web UI, Studio backend, or the full local app experience.

From the repo root:

```bash
cd /home/sparkleholic/Projects/unsloth
./install.sh --local
```

What this does at a high level:

- prepares a Studio environment under `~/.unsloth/studio`
- installs Python dependencies
- builds the frontend if needed
- overlays this local checkout for development

Then launch Studio:

```bash
unsloth studio -H 0.0.0.0 -p 8888
```

Open:

```text
http://localhost:8888
```

If port `8888` is already in use, the backend code will automatically move to the next available port and print the actual URL.

### Option 2: Core Python package development

Use this if you mainly want to work on the Python package and CLI.

Create a local virtual environment:

```bash
cd /home/sparkleholic/Projects/unsloth
uv venv .venv --python 3.13
source .venv/bin/activate
```

Install the package in editable mode:

```bash
uv pip install -e . --torch-backend=auto
```

If you want explicit local tooling for testing and packaging:

```bash
uv pip install pytest build fastapi uvicorn
```

Notes:

- `--torch-backend=auto` is the repo's preferred install path for matching the local hardware/backend.
- Exact GPU support depends on your system and the PyTorch/CUDA/ROCm combination selected at install time.

## Building

## Full repository build

The repository build script is:

```bash
./build.sh
```

Run it from the repo root:

```bash
cd /home/sparkleholic/Projects/unsloth
python -m pip install build
./build.sh
```

What `build.sh` does:

1. enters `studio/frontend`
2. removes old `dist/`
3. installs frontend dependencies with `bun` if available, otherwise `npm`
4. runs `npm run build`
5. validates that the generated CSS output is not truncated
6. removes old Python build artifacts
7. runs `python -m build`

Build outputs:

- frontend assets: `studio/frontend/dist/`
- Python artifacts: `dist/`

## Frontend-only build

If you only want to build the Studio frontend:

```bash
cd /home/sparkleholic/Projects/unsloth/studio/frontend
npm install
npm run build
```

Useful frontend commands:

```bash
npm run dev
npm run typecheck
npm run lint
npm run preview
```

## Running

## Run Studio

After Studio setup:

```bash
unsloth studio -H 0.0.0.0 -p 8888
```

This starts:

- the FastAPI backend
- the packaged frontend from `studio/frontend/dist`

The default Studio CLI help is available with:

```bash
unsloth studio --help
```

Useful Studio subcommands:

```bash
unsloth studio run --help
unsloth studio update --help
unsloth studio stop --help
```

## Run frontend dev server

If you are iterating on frontend code:

```bash
cd /home/sparkleholic/Projects/unsloth/studio/frontend
npm install
npm run dev
```

This gives you the Vite development server for frontend work.

Note that the production Studio path serves the built frontend bundle through the Python backend. A frontend dev server is useful for UI iteration, but it is not the same as the packaged Studio runtime.

## Run the CLI

The package exposes a top-level CLI:

```bash
unsloth --help
```

Main commands:

```bash
unsloth train --help
unsloth inference --help
unsloth export --help
unsloth list-checkpoints --help
unsloth studio --help
```

## Tests

## Installer-focused test script

The repo includes a simple test driver:

```bash
sh tests/run_all.sh
```

Before running it, make sure your environment includes `pytest`.

Example:

```bash
cd /home/sparkleholic/Projects/unsloth
source .venv/bin/activate
uv pip install pytest
sh tests/run_all.sh
```

This script runs:

- shell tests in `tests/sh`
- selected Python tests in `tests/python`

## Targeted frontend checks

For frontend-only validation:

```bash
cd /home/sparkleholic/Projects/unsloth/studio/frontend
npm install
npm run typecheck
npm run lint
npm run build
```

## Common Workflows

## Work on Studio end to end

```bash
cd /home/sparkleholic/Projects/unsloth
./install.sh --local
unsloth studio -H 0.0.0.0 -p 8888
```

## Work on frontend only

```bash
cd /home/sparkleholic/Projects/unsloth/studio/frontend
npm install
npm run dev
```

## Build distributable artifacts

```bash
cd /home/sparkleholic/Projects/unsloth
python -m pip install build
./build.sh
```

## Work on Python CLI/package

```bash
cd /home/sparkleholic/Projects/unsloth
uv venv .venv --python 3.13
source .venv/bin/activate
uv pip install -e . --torch-backend=auto
unsloth --help
```

## Notes and Caveats

- This repository has substantial hardware-specific install logic for NVIDIA, AMD, Intel, CPU-only, and macOS paths. The exact dependency resolution can differ by machine.
- Studio setup is handled more reliably through `./install.sh --local` than by manually guessing all backend dependencies.
- The frontend build depends on a valid Node/npm toolchain and uses Vite.
- The Python package build depends on `python -m build`.
- Some tests in this repository are installer-oriented or hardware-sensitive; not every test is suitable for every machine.

## Quick Start

If you just want the most practical dev command sequence for Studio:

```bash
cd /home/sparkleholic/Projects/unsloth
./install.sh --local
unsloth studio -H 0.0.0.0 -p 8888
```

If you just want the most practical command sequence for core package development:

```bash
cd /home/sparkleholic/Projects/unsloth
uv venv .venv --python 3.13
source .venv/bin/activate
uv pip install -e . --torch-backend=auto
unsloth --help
```
