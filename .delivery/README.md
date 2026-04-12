# Delivery Pipeline

This directory contains the delivery pipeline configuration, artifacts, and memory for the game-eggs project.

## Structure

- `config.yml` — pipeline configuration
- `artifacts/` — stage outputs, organized by stage namespace
- `memory/` — lessons learned from past runs (tiered chunked system)
- `state.md` — active pipeline state (present only during in-progress runs)
- `state-archive/` — archived states from past runs

## Usage

Invoke `/delivery-team:delivery-flow` in Claude Code to start or resume a pipeline run.
